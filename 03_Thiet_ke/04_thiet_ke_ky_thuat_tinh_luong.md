# TÀI LIỆU ĐẶC TẢ THIẾT KẾ KỸ THUẬT (TSD)
## PHÂN HỆ: TÍNH LƯƠNG LAI TINH GỌN VÀ PHÂN PHỐI PHIẾU LƯƠNG BẢO MẬT (Odoo 19 CE)
**Dự án:** Nâng cấp & Chuẩn hóa Hệ thống Nhân sự Đại Quang
**Phiên bản tài liệu:** v1.0
**Ngày biên soạn:** 2026-07-05

---

## 1. KIẾN TRÚC MODULE & QUAN HỆ PHỤ THUỘC

### 1.1. Tên module dự kiến
`daiquang_hr_payroll_hybrid`

### 1.2. Danh sách phụ thuộc (Dependencies)
```python
{
    'name': 'Dai Quang Hybrid Payroll & Secure Payslip Distribution',
    'version': '19.0.1.0.0',
    'category': 'Human Resources/Payroll',
    'depends': [
        'hr_payroll', # module bảng lương gốc
        'daiquang_attendance_2026', # lấy số liệu công
        'daiquang_zalo_api', # gửi Zalo ZNS
    ],
    'data': [
        'security/ir.model.access.csv',
        'views/hr_payslip_views.xml',
        'views/hr_payslip_run_views.xml',
        'report/hr_payslip_report_templates.xml',
    ],
    'installable': True,
    'application': False,
}
```

---

## 2. THIẾT KẾ CƠ SỞ DỮ LIỆU (DATABASE SCHEMA)

### 2.1. Kế thừa Phiếu lương (`hr.payslip`)
Bổ sung các trường lưu trữ thông tin công đồng bộ từ phân hệ Chấm công và bảo mật Portal:

| Tên trường (Field Name) | Kiểu dữ liệu | Mô tả |
| :--- | :--- | :--- |
| `actual_work_days` | Float | Số ngày công thực tế trong tháng |
| `ot_hours` | Float | Tổng số giờ tăng ca (OT) |
| `secure_token` | Char | Mã token bảo mật dùng để xem nhanh trên Portal (không cần login) |
| `token_expiry` | Datetime | Thời gian hết hạn của secure_token (48 giờ kể từ lúc sinh) |

---

## 3. LẬP TRÌNH NGHIỆP VỤ & PHƯƠNG THỨC XỬ LÝ (LOGIC METHODS)

### 3.1. Kết xuất Excel Bảng lương tổng hợp (`openpyxl`)
Phương thức xử lý tạo và định dạng file Excel chuyên nghiệp bằng thư viện `openpyxl` viết trực tiếp trên Odoo:

```python
import io
import openpyxl
from openpyxl.styles import Font, Alignment, PatternFill, Border, Side
from odoo import models, fields, api

class HrPayslipRun(models.Model):
    _inherit = 'hr.payslip.run'

    def action_export_payroll_excel(self):
        self.ensure_one()
        wb = openpyxl.Workbook()
        ws = wb.active
        ws.title = "Bảng lương Đại Quang"
        
        # 1. Định nghĩa Styles
        font_title = Font(name='Arial', size=16, bold=True)
        font_header = Font(name='Arial', size=11, bold=True, color='FFFFFF')
        font_body = Font(name='Arial', size=11)
        font_total = Font(name='Arial', size=11, bold=True)
        
        fill_header = PatternFill(start_color='1F4E78', end_color='1F4E78', fill_type='solid') # Xanh Navy đậm
        fill_total = PatternFill(start_color='D9E1F2', end_color='D9E1F2', fill_type='solid')
        
        thin_border = Border(
            left=Side(style='thin', color='BFBFBF'),
            right=Side(style='thin', color='BFBFBF'),
            top=Side(style='thin', color='BFBFBF'),
            bottom=Side(style='thin', color='BFBFBF')
        )
        
        # 2. Tạo tiêu đề bảng lương
        ws.merge_cells('A1:L1')
        ws['A1'] = f"BẢNG LƯƠNG TỔNG HỢP - THÁNG {self.date_start.strftime('%m/%Y')}"
        ws['A1'].font = font_title
        ws['A1'].alignment = Alignment(horizontal='center')
        
        # 3. Khai báo Headers
        headers = [
            "Mã NV", "Họ Tên", "Phòng ban", "Số tài khoản", 
            "Công thực tế", "Lương cơ bản", "Phụ cấp trách nhiệm", 
            "Phụ cấp tay nghề", "Lương tăng ca", "Bảo hiểm NLĐ (10.5%)", 
            "Thuế TNCN", "Thực nhận NET"
        ]
        ws.append([]) # Dòng trống
        ws.append(headers)
        
        # Format Headers
        for col_idx in range(1, len(headers) + 1):
            cell = ws.cell(row=3, column=col_idx)
            cell.font = font_header
            cell.fill = fill_header
            cell.alignment = Alignment(horizontal='center', vertical='center')
            cell.border = thin_border
            
        # 4. Duyệt các phiếu lương để điền dữ liệu
        row_num = 4
        for slip in self.slip_ids:
            # Lấy chi tiết số tiền từ các dòng kết quả tính lương (hr.payslip.line)
            basic = sum(slip.line_ids.filtered(lambda l: l.code == 'BASIC').mapped('total'))
            alw_resp = sum(slip.line_ids.filtered(lambda l: l.code == 'ALW_RESP').mapped('total'))
            alw_exp = sum(slip.line_ids.filtered(lambda l: l.code == 'ALW_EXP').mapped('total'))
            ot = sum(slip.line_ids.filtered(lambda l: l.code == 'OVERTIME').mapped('total'))
            ins = sum(slip.line_ids.filtered(lambda l: l.code == 'INS_EE').mapped('total'))
            tax = sum(slip.line_ids.filtered(lambda l: l.code == 'TAX_TNCN').mapped('total'))
            net = sum(slip.line_ids.filtered(lambda l: l.code == 'NET').mapped('total'))
            
            row_data = [
                slip.employee_id.employee_code or '',
                slip.employee_id.name,
                slip.employee_id.department_id.name or '',
                slip.employee_id.bank_account_id.acc_number or '',
                slip.actual_work_days,
                basic, alw_resp, alw_exp, ot, ins, tax, net
            ]
            ws.append(row_data)
            
            # Format dòng dữ liệu
            for col_idx in range(1, len(row_data) + 1):
                cell = ws.cell(row=row_num, column=col_idx)
                cell.font = font_body
                cell.border = thin_border
                if col_idx in [1, 3, 4]:
                    cell.alignment = Alignment(horizontal='center')
                elif col_idx > 4:
                    cell.alignment = Alignment(horizontal='right')
                    if col_idx > 5:
                        cell.number_format = '#,##0' # Định dạng tiền tệ VND
            row_num += 1
            
        # 5. Dòng tổng cộng (Total)
        ws.cell(row=row_num, column=2, value="Tổng cộng").font = font_total
        ws.cell(row=row_num, column=2).fill = fill_total
        for col_idx in range(5, len(headers) + 1):
            col_letter = openpyxl.utils.get_column_letter(col_idx)
            cell = ws.cell(row=row_num, column=col_idx, value=f"=SUM({col_letter}4:{col_letter}{row_num-1})")
            cell.font = font_total
            cell.fill = fill_total
            cell.border = thin_border
            cell.alignment = Alignment(horizontal='right')
            if col_idx > 5:
                cell.number_format = '#,##0'
                
        # Tự động co giãn độ rộng cột
        for col in ws.columns:
            max_len = max(len(str(cell.value or '')) for cell in col)
            col_letter = openpyxl.utils.get_column_letter(col[0].column)
            ws.column_dimensions[col_letter].width = max(max_len + 3, 12)
            
        # 6. Trả về file Excel dạng binary để tải xuống
        fp = io.BytesIO()
        wb.save(fp)
        fp.seek(0)
        data = fp.read()
        fp.close()
        
        # (Tạo attachment tải xuống tương tự như logic ở module Tuyển dụng)
```

### 3.2. Thuật toán Mã hóa Mật khẩu file PDF Phiếu lương
Sử dụng thư viện `pypdf` để tiến hành khóa file PDF phiếu lương bằng mật khẩu (Ngày sinh nhân sự `DDMMYYYY`):

```python
import base64
import io
from pypdf import PdfReader, PdfWriter
from odoo.exceptions import UserError

class HrPayslip(models.Model):
    _inherit = 'hr.payslip'

    def action_generate_encrypted_payslip_pdf(self):
        self.ensure_one()
        # 1. Sinh file PDF gốc bằng QWeb Report
        report = self.env.ref('hr_payroll.action_report_payslip')
        pdf_content, dummy = report._render_qweb_pdf(self.ids)
        
        # 2. Đọc file PDF vừa sinh bằng PdfReader
        reader = PdfReader(io.BytesIO(pdf_content))
        writer = PdfWriter()
        for page in reader.pages:
            writer.add_page(page)
            
        # 3. Lấy mật khẩu (Ngày sinh nhân viên định dạng DDMMYYYY)
        if not self.employee_id.birthday:
            raise UserError(f"Nhân viên {self.employee_id.name} chưa cập nhật ngày sinh để cấu hình mật khẩu PDF!")
            
        birthday_str = self.employee_id.birthday.strftime('%d%m%Y')
        
        # 4. Mã hóa bảo mật file bằng mật khẩu sinh nhật
        writer.encrypt(user_password=birthday_str, owner_password=None, use_128bit=True)
        
        # 5. Lưu và xuất kết quả nhị phân
        output_stream = io.BytesIO()
        writer.write(output_stream)
        encrypted_pdf = output_stream.getvalue()
        output_stream.close()
        
        return encrypted_pdf
```

### 3.3. Xây dựng Controller xem Phiếu lương trên Portal bằng Token
Controller mở cổng Portal không yêu cầu đăng nhập, xác thực bằng Token bảo mật để nhân viên xem nhanh phiếu lương từ link Zalo:

```python
from datetime import datetime
from odoo import http
from odoo.http import request

class PortalPayslipController(http.Controller):

    @http.route('/my/payslip/<int:payslip_id>', type='http', auth='none', methods=['GET'])
    def view_portal_payslip(self, payslip_id, token=None, **kwargs):
        """
        Cho phép truy cập xem phiếu lương nếu mã token truyền lên trùng khớp và chưa hết hạn.
        """
        # Sử dụng quyền sudo vì auth='none' để đọc dữ liệu
        payslip = request.env['hr.payslip'].sudo().browse(payslip_id)
        if not payslip.exists():
            return "Phiếu lương không tồn tại."
            
        # Kiểm tra tính toàn vẹn của Token và thời hạn hiệu lực (48 tiếng)
        if not token or payslip.secure_token != token:
            return "Mã truy cập không hợp lệ. Vui lòng liên hệ bộ phận HR."
            
        if payslip.token_expiry < datetime.now():
            return "Đường link đã hết hạn truy cập (Hiệu lực tối đa 48 giờ)."
            
        # Xuất giao diện HTML hoặc PDF phiếu lương tương ứng cho nhân viên xem trên Mobile
        report = request.env.ref('hr_payroll.action_report_payslip').sudo()
        pdf_content, dummy = report._render_qweb_pdf(payslip.ids)
        
        return request.make_response(pdf_content, headers=[
            ('Content-Type', 'application/pdf'),
            ('Content-Disposition', f'inline; filename=Payslip_{payslip.employee_id.name}.pdf')
        ])
```

---
