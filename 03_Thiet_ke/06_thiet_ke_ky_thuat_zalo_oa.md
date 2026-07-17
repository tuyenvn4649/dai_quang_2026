# TÀI LIỆU ĐẶC TẢ THIẾT KẾ KỸ THUẬT (TSD)
## MODULE ĐỘC LẬP: KẾT NỐI API ZALO OFFICIAL ACCOUNT (daiquang_zalo_api)
**Dự án:** Nâng cấp & Chuẩn hóa Hệ thống Nhân sự Đại Quang
**Phiên bản tài liệu:** v1.0
**Ngày biên soạn:** 2026-07-05

---

## 1. KIẾN TRÚC KỸ THUẬT & API ENDPOINTS

### 1.1. Zalo Developer API Endpoints
*   **Endpoint Lấy/Làm mới Token (OAuth v4):**
    `POST https://oauth.zaloapp.com/v4/oa/access_token`
    *   *Headers:* `Content-Type: application/x-www-form-urlencoded`, `secret_key: <Zalo_App_Secret>`
    *   *Payload:* `code=<authorization_code>&app_id=<Zalo_App_Id>&grant_type=authorization_code` (khi init) hoặc `refresh_token=<Refresh_Token>&app_id=<Zalo_App_Id>&grant_type=refresh_token` (khi refresh).
*   **Endpoint Gửi tin nhắn ZNS:**
    `POST https://business.openapi.zalo.me/message/template`
    *   *Headers:* `Content-Type: application/json`, `access_token: <Access_Token>`

---

## 2. THIẾT KẾ MÔ HÌNH DỮ LIỆU (DATABASE SCHEMA)

### 2.1. Cấu hình Tham số Zalo (`zalo.provider`)
Lưu trữ thông tin xác thực kết nối ứng dụng:

| Tên trường (Field Name) | Kiểu dữ liệu | Mô tả |
| :--- | :--- | :--- |
| `name` | Char | Tên cấu hình (Ví dụ: Đại Quang Zalo OA) |
| `zalo_app_id` | Char | App ID của Zalo Developer |
| `zalo_app_secret` | Char | Secret Key của ứng dụng |
| `zalo_oa_id` | Char | Official Account ID |
| `access_token` | Char | Access Token hiện hành |
| `refresh_token` | Char | Refresh Token hiện hành |
| `token_expiry` | Datetime | Thời gian hết hạn của Access Token |

### 2.2. Danh mục Mẫu ZNS (`zalo.zns.template`)
Lưu cấu trúc các mẫu tin nhắn:

| Tên trường | Kiểu dữ liệu | Mô tả |
| :--- | :--- | :--- |
| `name` | Char | Tên mẫu |
| `template_id` | Char | ID mẫu tin nhắn do Zalo cấp |
| `param_ids` | One2many | Liên kết `zalo.zns.template.param` |

*   **Bảng tham số mẫu (`zalo.zns.template.param`):**
    *   `template_id`: Many2one (`zalo.zns.template`, `ondelete='cascade'`)
    *   `name`: Char (Tên tham số, ví dụ: `candidate_name`)
    *   `field_id`: Many2one (`ir.model.fields`) - Lựa chọn trường mapping tự động trên Odoo.

### 2.3. Nhật ký gửi tin (`zalo.message.log`)
Lưu vết để kiểm toán và gỡ lỗi:

| Tên trường | Kiểu dữ liệu | Mô tả |
| :--- | :--- | :--- |
| `phone` | Char | Số điện thoại nhận |
| `message_type` | Selection | 'recruitment' (Tuyển dụng), 'payroll' (Lương), 'attendance' (Chấm công) |
| `res_model` | Char | Tên model liên kết (Ví dụ: `hr.applicant`) |
| `res_id` | Integer | ID bản ghi liên kết |
| `payload` | Text | Nội dung JSON payload gửi đi |
| `status` | Selection | 'success' (Thành công), 'error' (Lỗi) |
| `error_message` | Text | Thông điệp lỗi chi tiết nhận về từ Zalo |
| `send_date` | Datetime | Thời gian thực hiện gửi |

---

## 3. LẬP TRÌNH NGHIỆP VỤ & LỚP XỬ LÝ (LOGIC METHODS)

### 3.1. Phương thức Tự động Refresh Token (Cron Job)
Hệ thống tự động chạy ngầm mỗi 6 giờ để kiểm tra và cập nhật cặp Access/Refresh Token mới:

```python
import requests
import logging
from datetime import datetime, timedelta
from odoo import api, fields, models

_logger = logging.getLogger(__name__)

class ZaloProvider(models.Model):
    _name = 'zalo.provider'
    _description = 'Zalo OA Connection Provider'

    name = fields.Char(required=True)
    zalo_app_id = fields.Char(required=True)
    zalo_app_secret = fields.Char(required=True)
    zalo_oa_id = fields.Char(required=True)
    access_token = fields.Char()
    refresh_token = fields.Char()
    token_expiry = fields.Datetime()

    @api.model
    def cron_refresh_zalo_tokens(self):
        # Tìm các nhà cung cấp Zalo có token sắp hết hạn (trong vòng 6 tiếng nữa)
        expiry_threshold = datetime.now() + timedelta(hours=6)
        providers = self.search([('token_expiry', '<=', expiry_threshold)])
        for provider in providers:
            provider.action_refresh_tokens()

    def action_refresh_tokens(self):
        self.ensure_one()
        url = "https://oauth.zaloapp.com/v4/oa/access_token"
        headers = {
            'Content-Type': 'application/x-www-form-urlencoded',
            'secret_key': self.zalo_app_secret
        }
        data = {
            'refresh_token': self.refresh_token,
            'app_id': self.zalo_app_id,
            'grant_type': 'refresh_token'
        }
        try:
            response = requests.post(url, headers=headers, data=data, timeout=10)
            res_data = response.json()
            if 'access_token' in res_data:
                self.write({
                    'access_token': res_data['access_token'],
                    'refresh_token': res_data['refresh_token'],
                    'token_expiry': datetime.now() + timedelta(seconds=int(res_data['expires_in']))
                })
                _logger.info("Zalo OA Token refreshed successfully for provider: %s", self.name)
            else:
                _logger.error("Failed to refresh Zalo Token: %s", res_data)
        except Exception as e:
            _logger.error("Exception occurred while refreshing Zalo Token: %s", str(e))
```

### 3.2. Hàm gửi tin nhắn ZNS dùng chung cho toàn hệ thống
Hàm này được thiết kế để các module khác gọi trực tiếp khi có sự kiện cần nhắn tin:

```python
class ZaloProvider(models.Model):
    _inherit = 'zalo.provider'

    def send_zns_message(self, phone, template_id, template_data, res_model=False, res_id=False, msg_type='recruitment'):
        """
        phone: Số điện thoại nhận (Định dạng gốc: 0906238262 hoặc định dạng quốc tế)
        template_id: ID mẫu tin nhắn Zalo
        template_data: Dict chứa các tham số mẫu (Ví dụ: {'candidate_name': 'Nguyen Van A'})
        """
        self.ensure_one()
        # 1. Định dạng lại số điện thoại sang chuẩn Zalo yêu cầu (84xxx thay cho 0xxx)
        formatted_phone = self._format_phone_vietnam(phone)
        
        # 2. Chuẩn bị payload gửi đi
        url = "https://business.openapi.zalo.me/message/template"
        headers = {
            'Content-Type': 'application/json',
            'access_token': self.access_token
        }
        payload = {
            "phone": formatted_phone,
            "template_id": template_id,
            "template_data": template_data
        }
        
        # 3. Thực hiện gọi API
        status = 'error'
        error_msg = ''
        try:
            response = requests.post(url, headers=headers, json=payload, timeout=10)
            res_json = response.json()
            if res_json.get('error') == 0:
                status = 'success'
            else:
                error_msg = f"Error Code {res_json.get('error')}: {res_json.get('message')}"
        except Exception as e:
            error_msg = str(e)
            
        # 4. Ghi nhận lịch sử gửi tin (Logging)
        self.env['zalo.message.log'].create({
            'phone': phone,
            'message_type': msg_type,
            'res_model': res_model,
            'res_id': res_id,
            'payload': str(payload),
            'status': status,
            'error_message': error_msg,
            'send_date': fields.Datetime.now()
        })
        
        return status == 'success'

    def _format_phone_vietnam(self, phone):
        if not phone:
            return ""
        # Loại bỏ khoảng trắng và ký tự đặc biệt
        phone = ''.join(c for c in phone if c.isdigit())
        if phone.startswith('0'):
            return '84' + phone[1:]
        elif phone.startswith('84'):
            return phone
        return '84' + phone
```

---

## 3.3. TÍCH HỢP GỬI THƯ MỜI NHẬN VIỆC TRÊN MODULE TUYỂN DỤNG

### Kế thừa `hr.applicant` để bổ sung chức năng gửi Email và Zalo:
*   **Gửi Email:** Kiểm tra nếu ứng viên có Email, tự động tạo email nháp, kết xuất PDF thư tuyển dụng đính kèm và gửi đi.
*   **Gửi Zalo:** Lấy thông tin ứng tuyển truyền vào template Zalo OA ZNS và gọi hàm gửi tin nhắn của module `daiquang_zalo_api`.

```python
class HrApplicant(models.Model):
    _inherit = 'hr.applicant'

    def action_send_offer_via_email(self):
        self.ensure_one()
        if not self.email_from:
            raise UserError(_("Ứng viên không có thông tin Email. Không thể gửi thư tuyển dụng!"))
            
        # 1. Tạo file PDF thư tuyển dụng
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
        
        # 3. Tạo và gửi Email sử dụng mail server cấu hình trên Odoo
        mail_values = {
            'subject': _('Thư mời nhận việc - Công ty Đại Quang'),
            'body_html': _('<p>Chào %s,</p><p>Đại Quang trân trọng gửi tới bạn Thư mời nhận việc chi tiết được đính kèm trong email này.</p>') % self.partner_name,
            'email_to': self.email_from,
            'attachment_ids': [(6, 0, [attachment.id])]
        }
        mail = self.env['mail.mail'].create(mail_values)
        mail.send()
        
        # Ghi nhận log chatter
        self.message_post(body=_("Đã gửi Thư mời nhận việc PDF qua Email cho ứng viên: %s") % self.email_from)

    def action_send_offer_via_zalo(self):
        self.ensure_one()
        if not self.partner_mobile and not self.partner_phone:
            raise UserError(_("Ứng viên không có số điện thoại. Không thể gửi Zalo!"))
            
        # Tìm cấu hình Zalo OA đang hoạt động
        zalo_provider = self.env['zalo.provider'].search([], limit=1)
        if not zalo_provider:
            raise UserError(_("Chưa cấu hình thông số kết nối Zalo OA trên hệ thống!"))
            
        # Chuẩn bị dữ liệu tham số động cho mẫu ZNS
        template_id = "ZNS_TEMPLATE_OFFER_ID" # ID mẫu đăng ký trên Zalo Cloud
        template_data = {
            "candidate_name": self.partner_name,
            "job_position": self.job_id.name,
            "salary_gross": f"{self.salary_gross_proposed:,.0f} VNĐ",
            "working_branch": self.branch_id.name
        }
        
        phone = self.partner_mobile or self.partner_phone
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
            raise UserError(_("Gửi tin nhắn Zalo thất bại. Vui lòng kiểm tra nhật ký kỹ thuật!"))
```

---
