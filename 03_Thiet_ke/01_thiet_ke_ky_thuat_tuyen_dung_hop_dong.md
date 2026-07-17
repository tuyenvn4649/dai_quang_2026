# TÀI LIỆU ĐẶC TẢ THIẾT KẾ KỸ THUẬT (TSD)
## PHÂN HỆ: TUYỂN DỤNG VÀ QUẢN LÝ HỢP ĐỒNG LAO ĐỘNG (Odoo 19 CE)
**Dự án:** Nâng cấp & Chuẩn hóa Hệ thống Nhân sự Đại Quang
**Phiên bản tài liệu:** v1.0
**Ngày biên soạn:** 2026-07-05

---

## 1. CẤU TRÚC MODULE & QUAN HỆ PHỤ THUỘC

### 1.1. Tên module dự kiến
`daiquang_hr_recruitment`

### 1.2. Danh sách phụ thuộc (Dependencies)
```python
{
    'name': 'Dai Quang HR Recruitment & Contract Automation',
    'version': '19.0.1.0.0',
    'category': 'Human Resources',
    'depends': [
        'hr_recruitment',
        'hr_contract',
        'branch', # module chi nhánh của hệ thống
    ],
    'data': [
        'security/ir.model.access.csv',
        'views/res_config_settings_views.xml',
        'views/hr_applicant_views.xml',
        'views/hr_employee_views.xml',
        'views/hr_contract_views.xml',
        'data/mail_template_data.xml',
        'data/decimal_precision_data.xml',
    ],
    'installable': True,
    'application': False,
}
```

---

## 2. THIẾT KẾ CƠ SỞ DỮ LIỆU (DATABASE SCHEMA)

### 2.1. Kế thừa mô hình Ứng viên (`hr.applicant`)
Bổ sung các trường thông tin phục vụ thỏa thuận lương, in thư tuyển dụng và hồ sơ pháp lý:

| Tên trường (Field Name) | Kiểu dữ liệu (Data Type) | Chi tiết / Cấu hình |
| :--- | :--- | :--- |
| `id_card_number` | Char | Số CCCD/CMND |
| `id_card_date` | Date | Ngày cấp |
| `id_card_place` | Char | Nơi cấp |
| `id_card_front_img` | Binary | Ảnh mặt trước CCCD |
| `id_card_back_img` | Binary | Ảnh mặt sau CCCD |
| `medical_check_img` | Binary | File ảnh/PDF Giấy khám sức khỏe |
| `branch_id` | Many2one | Liên kết `res.branch`, `required=True` |
| `salary_basic_proposed` | Float | Lương cơ bản đề xuất (đóng bảo hiểm) |
| `salary_responsible_proposed`| Float | Lương trách nhiệm đề xuất |
| `salary_experience_proposed` | Float | Lương tay nghề/kinh nghiệm đề xuất |
| `alw_ids` | One2many | Liên kết `hr.applicant.allowance`, `applicant_id` |
| `reward_ids` | One2many | Liên kết `hr.applicant.reward`, `applicant_id` |
| `cutdown_ids` | One2many | Liên kết `hr.applicant.cutdown`, `applicant_id` |
| `salary_gross_proposed` | Float | Compute, Lương Gross đề xuất |
| `salary_net_proposed` | Float | Compute, Lương Net đề xuất |
| `str_salary_gross_proposed` | Char | Compute, Lương Gross bằng chữ |
| `str_salary_net_proposed` | Char | Compute, Lương Net bằng chữ |
| `boss_ids` | Many2many | Liên kết `res.users`, Bộ phận ký duyệt |
| `seal_image` | Binary | Con dấu công ty |
| `offer_template_id` | Many2one | Liên kết `mail.template`, filter domain applicant |

### 2.2. Các bảng phụ cấu trúc lương Ứng viên
*   **Chi tiết phụ cấp (`hr.applicant.allowance`):**
    *   `applicant_id`: Many2one (`hr.applicant`, `ondelete='cascade'`)
    *   `name`: Char (Tên khoản phụ cấp, ví dụ: Phụ cấp ăn trưa)
    *   `amount`: Float (Số tiền)
*   **Chi tiết thưởng thỏa thuận (`hr.applicant.reward`):**
    *   `applicant_id`: Many2one (`hr.applicant`, `ondelete='cascade'`)
    *   `name`: Char (Tên khoản thưởng, ví dụ: Thưởng chuyên cần)
    *   `amount`: Float (Số tiền)
*   **Chi tiết giảm trừ thỏa thuận (`hr.applicant.cutdown`):**
    *   `applicant_id`: Many2one (`hr.applicant`, `ondelete='cascade'`)
    *   `name`: Char (Tên khoản giảm trừ)
    *   `amount`: Float (Số tiền)

### 2.3. Kế thừa Cấu hình Hệ thống (`res.company` / `res.config.settings`)
Bổ sung các tham số thuế lũy tiến và giảm trừ gia cảnh có thể tùy biến động:

| Tên trường (Field Name) | Kiểu dữ liệu | Mô tả |
| :--- | :--- | :--- |
| `pit_personal_deduction` | Float | Giảm trừ bản thân (Mặc định: 11,000,000) |
| `pit_dependent_deduction`| Float | Giảm trừ người phụ thuộc (Mặc định: 4,400,000) |
| `rate_bhxh_company` | Float | Tỷ lệ BHXH công ty trả (Mặc định: 17.5) |
| `rate_bhyt_company` | Float | Tỷ lệ BHYT công ty trả (Mặc định: 3.0) |
| `rate_bhtn_company` | Float | Tỷ lệ BHTN công ty trả (Mặc định: 1.0) |
| `rate_kpcd_company` | Float | Tỷ lệ Kinh phí Công đoàn (Mặc định: 2.0) |
| `rate_bhxh_employee` | Float | Tỷ lệ BHXH NLĐ đóng (Mặc định: 8.0) |
| `rate_bhyt_employee` | Float | Tỷ lệ BHYT NLĐ đóng (Mặc định: 1.5) |
| `rate_bhtn_employee` | Float | Tỷ lệ BHTN NLĐ đóng (Mặc định: 1.0) |

### 2.4. Kế thừa mô hình Hợp đồng lao động (`hr.contract`)
*   `salary_responsible`: Float (Phụ cấp trách nhiệm)
*   `salary_experience`: Float (Phụ cấp tay nghề/kinh nghiệm)
*   `has_social_insurance`: Boolean (Đóng BHXH, mặc định True)
*   `has_health_insurance`: Boolean (Đóng BHYT, mặc định True)
*   `has_unemployment_insurance`: Boolean (Đóng BHTN, mặc định True)
*   `insurance_salary_base`: Float, Compute, Store=True (Mức lương tính đóng BHXH)
*   `contract_allowance_ids`: One2many (`hr.contract.allowance`, `contract_id`)

---

## 3. THIẾT KẾ PHƯƠNG THỨC NGHIỆP VỤ & LẬP TRÌNH (LOGIC METHODS)

### 3.1. Tính toán lương đề xuất & Hàm dịch số tiền sang chữ
Tập trung cải tiến hàm compute để loại bỏ lỗi hiệu năng $O(N^2)$, chỉ lặp qua danh sách `self` đang hiển thị trên màn hình:

```python
from odoo import api, fields, models
# Thư viện dịch số thành chữ tùy biến hoặc viết hàm helper nội bộ
from odoo.tools.translate import _

class HrApplicant(models.Model):
    _inherit = 'hr.applicant'

    @api.depends('salary_basic_proposed', 'salary_responsible_proposed', 
                 'salary_experience_proposed', 'alw_ids', 'reward_ids', 'cutdown_ids')
    def _compute_proposed_salary(self):
        # Tối ưu hóa hiệu năng: Chỉ duyệt qua các bản ghi trong self
        for applicant in self:
            total_alw = sum(applicant.alw_ids.mapped('amount'))
            total_reward = sum(applicant.reward_ids.mapped('amount'))
            total_cutdown = sum(applicant.cutdown_ids.mapped('amount'))
            
            # Tính Gross đề xuất
            applicant.salary_gross_proposed = (
                applicant.salary_basic_proposed +
                applicant.salary_responsible_proposed +
                applicant.salary_experience_proposed +
                total_alw + total_reward
            )
            
            # Khấu trừ bảo hiểm NLĐ đề xuất (tạm tính theo tỷ lệ cấu hình)
            ins_base = (
                applicant.salary_basic_proposed +
                applicant.salary_responsible_proposed +
                applicant.salary_experience_proposed
            )
            rates = applicant.company_id.rate_bhxh_employee + applicant.company_id.rate_bhyt_employee + applicant.company_id.rate_bhtn_employee
            insurance_deduct = ins_base * (rates / 100.0)
            
            # Tính Net đề xuất
            applicant.salary_net_proposed = applicant.salary_gross_proposed - total_cutdown - insurance_deduct
            
            # Chuyển đổi số thành chữ
            applicant.str_salary_gross_proposed = applicant._amount_to_vietnamese_text(applicant.salary_gross_proposed)
            applicant.str_salary_net_proposed = applicant._amount_to_vietnamese_text(applicant.salary_net_proposed)

    def _amount_to_vietnamese_text(self, amount):
        # Thuật toán dịch số tiền thành chữ tiếng Việt chuẩn hóa
        # Đầu ra ví dụ: "Mười lăm triệu năm trăm nghìn đồng chẵn"
        # (Chi tiết thuật toán sẽ được hiện thực hóa trong code python)
        pass
```

### 3.2. Đồng bộ 1-Click chuyển đổi Ứng viên sang Nhân sự (`hr.employee`)
Kế thừa phương thức chuyển ứng viên thành nhân viên của Odoo 19:
*   Đồng bộ thông tin CCCD, ngày sinh, giới tính và Chi nhánh bắt buộc.
*   Tự động sinh mã nhân sự theo chi nhánh và phòng ban.

```python
class HrApplicant(models.Model):
    _inherit = 'hr.applicant'

    def create_employee_from_applicant(self):
        # 1. Kế thừa phương thức chuẩn của Odoo
        res = super(HrApplicant, self).create_employee_from_applicant()
        
        # 2. Lấy ID nhân viên mới được tạo
        employee_id = res.get('res_id') or self.emp_id.id
        if employee_id:
            employee = self.env['hr.employee'].browse(employee_id)
            # Đồng bộ các trường thông tin cá nhân và CCCD
            employee.write({
                'branch_id': self.branch_id.id, # Đồng bộ chi nhánh bắt buộc
                'id_card_number': self.id_card_number,
                'id_card_date': self.id_card_date,
                'id_card_place': self.id_card_place,
                'id_card_front_img': self.id_card_front_img,
                'id_card_back_img': self.id_card_back_img,
            })
            
            # Copy file đính kèm y tế và CV sang hồ sơ nhân viên
            attachments = self.env['ir.attachment'].search([
                ('res_model', '=', 'hr.applicant'),
                ('res_id', '=', self.id)
            ])
            for att in attachments:
                att.copy({
                    'res_model': 'hr.employee',
                    'res_id': employee.id
                })
            
            # Tự động sinh mã nhân sự phân cấp cho nhân viên
            employee._generate_employee_code()
            
            # 3. Tự động khởi tạo Hợp đồng nháp (Draft Contract)
            self._create_draft_contract(employee)
            
        return res

    def _create_draft_contract(self, employee):
        contract_obj = self.env['hr.contract']
        # Ánh xạ các mức lương từ ứng viên sang hợp đồng nháp
        contract_vals = {
            'name': f'Hợp đồng lao động - {employee.name}',
            'employee_id': employee.id,
            'wage': self.salary_basic_proposed,
            'salary_responsible': self.salary_responsible_proposed,
            'salary_experience': self.salary_experience_proposed,
            'has_social_insurance': True,
            'has_health_insurance': True,
            'has_unemployment_insurance': True,
            'state': 'draft', # Trạng thái nháp
        }
        contract = contract_obj.create(contract_vals)
        
        # Đồng bộ các phụ cấp ngoài lương sang hợp đồng
        allowance_obj = self.env['hr.contract.allowance']
        for alw in self.alw_ids:
            allowance_obj.create({
                'contract_id': contract.id,
                'name': alw.name,
                'amount': alw.amount,
            })
```

### 3.3. Thuật toán tự động tạo Mã nhân sự (`_generate_employee_code`)
Mã nhân sự sinh tự động dựa trên mã chi nhánh, mã phòng ban và số đếm tuần tự:

```python
class HrEmployee(models.Model):
    _inherit = 'hr.employee'

    def _generate_employee_code(self):
        for emp in self:
            if not emp.employee_code:
                company_code = emp.company_id.code or 'DQ'
                branch_code = emp.branch_id.code or 'VP'
                dept_code = emp.department_id.code or 'NS'
                
                # Tìm số đếm tuần tự cao nhất thuộc tổ hợp Company + Branch + Dept
                prefix = f"{company_code}-{branch_code}-{dept_code}-"
                last_emp = self.search([
                    ('employee_code', '=like', f"{prefix}%")
                ], order='employee_code desc', limit=1)
                
                next_seq = 1
                if last_emp:
                    try:
                        last_seq = int(last_emp.employee_code.replace(prefix, ''))
                        next_seq = last_seq + 1
                    except ValueError:
                        pass
                
                emp.employee_code = f"{prefix}{next_seq:04d}"
```

### 3.4. Cấu hình & Tính Lương trích đóng Bảo hiểm
Định nghĩa trường tính toán mức trích nộp bảo hiểm bắt buộc trên hợp đồng:

```python
class HrContract(models.Model):
    _inherit = 'hr.contract'

    @api.depends('wage', 'salary_responsible', 'salary_experience', 
                 'has_social_insurance', 'has_health_insurance', 'has_unemployment_insurance')
    def _compute_insurance_salary_base(self):
        for contract in self:
            if contract.has_social_insurance or contract.has_health_insurance or contract.has_unemployment_insurance:
                # Căn cứ đóng bảo hiểm = Lương cơ bản + Phụ cấp trách nhiệm + Phụ cấp tay nghề/kinh nghiệm
                contract.insurance_salary_base = contract.wage + contract.salary_responsible + contract.salary_experience
            else:
                contract.insurance_salary_base = 0.0
```

---

## 4. ĐẶC TẢ GIAO DIỆN & TƯƠNG THÍCH ODOO 19

### 4.1. Kế thừa Form Ứng viên (`hr_recruitment.hr_applicant_view_form`)
Sử dụng thẻ `<notebook>` để chia tách các Tab thông tin bằng Bootstrap 5 responsive:

```xml
<record id="hr_applicant_view_form_inherit_dq" model="ir.ui.view">
    <field name="name">hr.applicant.view.form.inherit.dq</field>
    <field name="model">hr.applicant</field>
    <field name="inherit_id" ref="hr_recruitment.hr_applicant_view_form"/>
    <field name="arch" type="xml">
        <!-- Chèn tab thông tin thỏa thuận lương và hồ sơ y tế/CCCD -->
        <xpath expr="//sheet" position="inside">
            <notebook>
                <page string="Thông tin Pháp lý & Hồ sơ đính kèm">
                    <group>
                        <group string="Căn cước Công dân">
                            <field name="id_card_number"/>
                            <field name="id_card_date"/>
                            <field name="id_card_place"/>
                        </group>
                        <group string="Chi nhánh tuyển dụng">
                            <field name="branch_id" options="{'no_create': True}"/>
                        </group>
                    </group>
                    <group>
                        <group string="Mặt trước CCCD">
                            <field name="id_card_front_img" widget="image" class="oe_avatar"/>
                        </group>
                        <group string="Mặt sau CCCD">
                            <field name="id_card_back_img" widget="image" class="oe_avatar"/>
                        </group>
                    </group>
                </page>
                <page string="Thỏa thuận Lương đề xuất">
                    <group>
                        <group string="Các Mức lương Đề xuất">
                            <field name="salary_basic_proposed"/>
                            <field name="salary_responsible_proposed"/>
                            <field name="salary_experience_proposed"/>
                        </group>
                        <group string="Tổng hợp (Quy đổi)">
                            <field name="salary_gross_proposed" readonly="1"/>
                            <field name="str_salary_gross_proposed" readonly="1"/>
                            <field name="salary_net_proposed" readonly="1"/>
                            <field name="str_salary_net_proposed" readonly="1"/>
                        </group>
                    </group>
                    <group>
                        <field name="alw_ids" widget="one2many_list">
                            <tree editable="bottom">
                                <field name="name"/>
                                <field name="amount" sum="Tổng Phụ cấp"/>
                            </tree>
                        </field>
                    </group>
                </page>
                <page string="Ký duyệt & In Thư Tuyển dụng">
                    <group>
                        <group>
                            <field name="offer_template_id" options="{'no_create': True}"/>
                            <field name="boss_ids" widget="many2many_tags"/>
                        </group>
                        <group>
                            <field name="seal_image" widget="image" class="oe_avatar"/>
                        </group>
                    </group>
                </page>
            </notebook>
        </xpath>
        <!-- Thêm các nút In và Gửi thư tuyển dụng trên header -->
        <xpath expr="//header" position="inside">
            <button name="action_print_offer_letter" string="In Thư Tuyển Dụng" type="object" class="btn-primary" attrs="{'invisible': [('offer_template_id', '=', False)]}"/>
            <button name="action_send_offer_via_email" string="Gửi Email Thư Mời" type="object" class="btn-secondary" attrs="{'invisible': ['|', ('offer_template_id', '=', False), ('email_from', '=', False)]}"/>
            <button name="action_send_offer_via_zalo" string="Gửi Zalo Thư Mời" type="object" class="btn-success" attrs="{'invisible': ['|', ('offer_template_id', '=', False), ('partner_mobile', '=', False)]}"/>
        </xpath>
    </field>
</record>
```

### 4.2. In Thư mời nhận việc PDF từ Mail Template
Nút bấm "In Thư Tuyển Dụng" sẽ gọi phương thức kết xuất HTML Mail Template thành PDF:

```python
class HrApplicant(models.Model):
    _inherit = 'hr.applicant'

    def action_print_offer_letter(self):
        self.ensure_one()
        if not self.offer_template_id:
            raise UserError(_("Vui lòng chọn Mẫu thư tuyển dụng trước khi in!"))
            
        # Renders the mail template body to HTML with variables
        template = self.offer_template_id
        rendered_body = template._render_field('body_html', self.ids)[self.id]
        
        # Odoo API to generate PDF report from HTML body string
        pdf_content, dummy = self.env['ir.actions.report']._run_wkhtmltopdf([rendered_body])
        
        # Saves or downloads the PDF
        attachment_vals = {
            'name': f'Offer_Letter_{self.partner_name or self.name}.pdf',
            'type': 'binary',
            'datas': base64.b64encode(pdf_content),
            'res_model': 'hr.applicant',
            'res_id': self.id,
            'mimetype': 'application/pdf'
        }
        attachment = self.env['ir.attachment'].create(attachment_vals)
        
        return {
            'type': 'ir.actions.act_url',
            'url': f'/web/content/{attachment.id}?download=true',
            'target': 'new',
        }
```

### 4.3. Các phương thức gửi Thư mời nhận việc qua Email và Zalo (ZNS)
Tích hợp gửi tự động trực tiếp từ hồ sơ Ứng viên:

```python
class HrApplicant(models.Model):
    _inherit = 'hr.applicant'

    def action_send_offer_via_email(self):
        self.ensure_one()
        if not self.email_from:
            raise UserError(_("Ứng viên không có thông tin Email!"))
            
        # 1. Tạo file PDF thư mời
        rendered_body = self.offer_template_id._render_field('body_html', self.ids)[self.id]
        pdf_content, dummy = self.env['ir.actions.report']._run_wkhtmltopdf([rendered_body])
        
        # 2. Tạo attachment đính kèm email
        attachment = self.env['ir.attachment'].create({
            'name': f'Thu_Moi_Nhan_Viec_{self.partner_name or self.name}.pdf',
            'type': 'binary',
            'datas': base64.b64encode(pdf_content),
            'res_model': 'hr.applicant',
            'res_id': self.id,
            'mimetype': 'application/pdf'
        })
        
        # 3. Gửi email
        mail_values = {
            'subject': _('Thư mời nhận việc - Công ty Đại Quang'),
            'body_html': _('<p>Chào %s,</p><p>Đại Quang trân trọng gửi tới bạn Thư mời nhận việc chi tiết được đính kèm trong email này.</p>') % self.partner_name,
            'email_to': self.email_from,
            'attachment_ids': [(6, 0, [attachment.id])]
        }
        mail = self.env['mail.mail'].create(mail_values)
        mail.send()
        self.message_post(body=_("Đã gửi Thư mời nhận việc PDF qua Email cho ứng viên: %s") % self.email_from)

    def action_send_offer_via_zalo(self):
        self.ensure_one()
        phone = self.partner_mobile or self.partner_phone
        if not phone:
            raise UserError(_("Ứng viên không có số điện thoại di động!"))
            
        # Tìm cấu hình Zalo OA đang hoạt động
        zalo_provider = self.env['zalo.provider'].search([], limit=1)
        if not zalo_provider:
            raise UserError(_("Chưa cấu hình thông số kết nối Zalo OA trên hệ thống!"))
            
        # Dữ liệu động cho mẫu ZNS
        template_id = "ZNS_TEMPLATE_OFFER_ID"
        template_data = {
            "candidate_name": self.partner_name,
            "job_position": self.job_id.name,
            "salary_gross": f"{self.salary_gross_proposed:,.0f} VNĐ",
            "working_branch": self.branch_id.name
        }
        
        success = zalo_provider.send_zns_message(
            phone=phone,
            template_id=template_id,
            template_data=template_data,
            res_model='hr.applicant',
            res_id=self.id,
            msg_type='recruitment'
        )
        
        if success:
            self.message_post(body=_("Đã gửi tin nhắn Zalo ZNS thông báo nhận việc thành công đến số: %s") % phone)
        else:
            raise UserError(_("Gửi tin nhắn Zalo thất bại! Vui lòng kiểm tra log."))
```

---
