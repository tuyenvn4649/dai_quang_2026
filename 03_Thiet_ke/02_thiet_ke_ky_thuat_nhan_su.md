# TÀI LIỆU ĐẶC TẢ THIẾT KẾ KỸ THUẬT (TSD)
## PHÂN HỆ: QUẢN LÝ NHÂN SỰ VÀ SỨC KHỎE ĐỊNH KỲ (Odoo 19 CE)
**Dự án:** Nâng cấp & Chuẩn hóa Hệ thống Nhân sự Đại Quang
**Phiên bản tài liệu:** v1.0
**Ngày biên soạn:** 2026-07-05

---

## 1. CẤU TRÚC MODULE & QUAN HỆ PHỤ THUỘC

### 1.1. Tên module dự kiến
`daiquang_hr_health`

### 1.2. Danh sách phụ thuộc (Dependencies)
```python
{
    'name': 'Dai Quang HR Employee & Health Management',
    'version': '19.0.1.0.0',
    'category': 'Human Resources',
    'depends': [
        'hr',
        'branch', # module chi nhánh hiện hành
    ],
    'data': [
        'security/ir.model.access.csv',
        'security/hr_security.xml',
        'views/hr_department_views.xml',
        'views/hr_employee_views.xml',
        'data/cron_data.xml',
    ],
    'installable': True,
    'application': False,
}
```

---

## 2. THIẾT KẾ CƠ SỞ DỮ LIỆU (DATABASE SCHEMA)

### 2.1. Kế thừa Phòng ban (`hr.department`)
Bổ sung trường lưu trữ các tài khoản được xem dữ liệu chéo:

| Tên trường (Field Name) | Kiểu dữ liệu | Cấu hình | Mô tả |
| :--- | :--- | :--- | :--- |
| `viewers_ids` | Many2many | `res.users`, relation table: `hr_dept_viewers_rel` | Danh sách tài khoản được xem nhân viên thuộc phòng ban này |

### 2.2. Lịch sử Khám sức khỏe (`employee.health`)
Bảng ghi nhận nhật ký khám sức khỏe của từng nhân sự:

| Tên trường | Kiểu dữ liệu | Cấu hình | Mô tả |
| :--- | :--- | :--- | :--- |
| `employee_id` | Many2one | `hr.employee`, `ondelete='cascade'`, `required=True` | Nhân viên khám sức khỏe |
| `date` | Date | `required=True`, `default=fields.Date.context_today` | Ngày thực hiện khám |
| `hospital` | Char | | Cơ sở y tế thực hiện |
| `health_type` | Selection | `'type_1'` (Loại I), `'type_2'` (Loại II), `'type_3'` (Loại III), `'type_4'` (Loại IV), `'type_5'` (Loại V) | Kết quả phân loại sức khỏe |
| `notes` | Text | | Ghi chú bệnh lý hoặc hạn chế làm việc |
| `attachment_ids` | Many2many | `ir.attachment` | File đính kèm kết quả y tế |

### 2.3. Kế thừa Nhân viên (`hr.employee`)
Mở rộng liên kết sức khỏe và các trường compute lưu trữ (`store=True`):

| Tên trường | Kiểu dữ liệu | Cấu hình | Mô tả |
| :--- | :--- | :--- | :--- |
| `health_ids` | One2many | `employee.health`, `employee_id` | Lịch sử các đợt khám |
| `last_health_date` | Date | Compute, `store=True` | Ngày khám sức khỏe gần nhất |
| `last_health_type` | Selection| Compute, `store=True` | Kết quả phân loại sức khỏe gần nhất |
| `health_status` | Selection| Compute, `store=True` | Trạng thái y tế (ok, warning, danger, none) |

*   **Các giá trị của `health_status`:**
    *   `'ok'`: Đã khám đầy đủ (Thời gian từ ngày khám gần nhất đến hiện tại < 6 tháng).
    *   `'warning'`: Quá hạn 6 tháng chưa khám (Thời gian từ 6 tháng đến < 12 tháng).
    *   `'danger'`: Quá hạn 12 tháng chưa khám (Thời gian $\ge 12$ tháng).
    *   `'none'`: Chưa từng khám sức khỏe.

---

## 3. LẬP TRÌNH NGHIỆP VỤ & PHÂN QUYỀN TRUY CẬP (LOGIC & SECURITY)

### 3.1. Thuật toán tính toán Trạng thái Y tế (Compute & Cron Job)
Do trường trạng thái y tế phụ thuộc vào ngày hiện tại (`fields.Date.today()`), để đảm bảo tính chính xác hàng ngày và hỗ trợ **Group-by / Filter** hiệu quả với thuộc tính `store=True`:
1.  Viết hàm compute tính toán dựa trên ngày khám gần nhất.
2.  Thiết lập **Cron Job chạy hàng đêm** quét qua tất cả nhân viên để kích hoạt tính toán lại trạng thái, cập nhật dữ liệu lưu trữ vật lý trong DB.

```python
from datetime import datetime
from dateutil.relativedelta import relativedelta
from odoo import api, fields, models

class HrEmployee(models.Model):
    _inherit = 'hr.employee'

    health_ids = fields.One2many('employee.health', 'employee_id', string='Lịch sử khám sức khỏe')
    last_health_date = fields.Date(string='Ngày khám gần nhất', compute='_compute_health_data', store=True)
    last_health_type = fields.Selection([
        ('type_1', 'Loại I'),
        ('type_2', 'Loại II'),
        ('type_3', 'Loại III'),
        ('type_4', 'Loại IV'),
        ('type_5', 'Loại V'),
    ], string='Phân loại sức khỏe gần nhất', compute='_compute_health_data', store=True)
    
    health_status = fields.Selection([
        ('ok', 'Đã khám đầy đủ'),
        ('warning', 'Quá hạn 6 tháng chưa khám'),
        ('danger', 'Quá hạn 12 tháng chưa khám'),
        ('none', 'Chưa từng khám'),
    ], string='Trạng thái khám sức khỏe', compute='_compute_health_data', store=True, default='none')

    @api.depends('health_ids.date', 'health_ids.health_type')
    def _compute_health_data(self):
        # Tối ưu hóa hiệu năng, xử lý theo recordset hiện hành
        today = fields.Date.today()
        for emp in self:
            if emp.health_ids:
                # Sắp xếp lịch sử để tìm đợt khám gần nhất
                sorted_health = emp.health_ids.sorted(key=lambda r: r.date, reverse=True)
                latest = sorted_health[0]
                emp.last_health_date = latest.date
                emp.last_health_type = latest.health_type
                
                # Tính khoảng cách tháng
                diff_months = relativedelta(today, latest.date).years * 12 + relativedelta(today, latest.date).months
                if diff_months < 6:
                    emp.health_status = 'ok'
                elif diff_months < 12:
                    emp.health_status = 'warning'
                else:
                    emp.health_status = 'danger'
            else:
                emp.last_health_date = False
                emp.last_health_type = False
                emp.health_status = 'none'

    @api.model
    def cron_recalculate_health_status(self):
        """
        Cron Job chạy hàng đêm (00:30) để cập nhật lại trạng thái health_status
        do sự thay đổi của ngày hiện tại (fields.Date.today()).
        """
        employees = self.search([('health_ids', '!=', False)])
        # Kích hoạt tính toán lại bằng cách ghi đè hoặc gọi hàm compute
        employees._compute_health_data()
```

### 3.2. Phân quyền xem dữ liệu chéo bằng Record Rule
Cấu hình Record Rule cho phép người dùng thuộc danh sách `viewers_ids` của phòng ban được xem nhân viên thuộc phòng ban đó (hỗ trợ đệ quy xem cả phòng ban con bằng toán tử `child_of`):

```xml
<record id="rule_hr_employee_department_viewer" model="ir.rule">
    <field name="name">HR Department Viewers Access Rule</field>
    <field name="model_id" ref="hr.model_hr_employee"/>
    <field name="domain_force">
        ['|', '|',
            ('department_id.viewers_ids', 'in', user.id),
            ('department_id.parent_id.viewers_ids', 'in', user.id),
            ('user_id', '=', user.id)
        ]
    </field>
    <field name="groups" eval="[(4, ref('base.group_user'))]"/>
    <field name="perm_read" eval="True"/>
    <field name="perm_write" eval="False"/>
    <field name="perm_create" eval="False"/>
    <field name="perm_unlink" eval="False"/>
</record>
```

---

## 4. ĐẶC TẢ GIAO DIỆN & TƯƠNG THÍCH ODOO 19

### 4.1. Kế thừa View Tìm kiếm (`hr.view_employee_filter`)
Tích hợp sẵn bộ lọc nhanh chuẩn xác theo múi giờ trên Odoo 19:

```xml
<record id="view_employee_filter_inherit_health" model="ir.ui.view">
    <field name="name">hr.employee.search.inherit.health</field>
    <field name="model">hr.employee</field>
    <field name="inherit_id" ref="hr.view_employee_filter"/>
    <field name="arch" type="xml">
        <!-- Chèn bộ lọc sức khỏe vào khối search -->
        <xpath expr="//filter[@name='inactive']" position="before">
            <separator/>
            <filter string="Quá 6 tháng chưa khám" name="health_warning" domain="[('health_status', '=', 'warning')]"/>
            <filter string="Quá 12 tháng chưa khám" name="health_danger" domain="[('health_status', '=', 'danger')]"/>
            <filter string="Chưa từng khám sức khỏe" name="health_none" domain="[('health_status', '=', 'none')]"/>
            <separator/>
        </xpath>
        <!-- Thêm tính năng Gom nhóm (Group by) theo trạng thái sức khỏe -->
        <xpath expr="//group" position="inside">
            <filter string="Trạng thái sức khỏe" name="group_by_health_status" context="{'group_by': 'health_status'}"/>
        </xpath>
    </field>
</record>
```

### 4.2. Kế thừa Form phòng ban để thiết lập Viewers
```xml
<record id="view_department_form_inherit_viewers" model="ir.ui.view">
    <field name="name">hr.department.form.inherit.viewers</field>
    <field name="model">hr.department</field>
    <field name="inherit_id" ref="hr.view_department_form"/>
    <field name="arch" type="xml">
        <xpath expr="//field[@name='parent_id']" position="after">
            <field name="viewers_ids" widget="many2many_tags" placeholder="Chọn tài khoản được quyền xem dữ liệu phòng này..."/>
        </xpath>
    </field>
</record>
```

### 4.3. Kế thừa giao diện Kanban Nhân viên (Tương thích Odoo 19)
Khắc phục lỗi xung đột giao diện Kanban bằng cách sử dụng thẻ XML kế thừa tương thích với cú pháp của Odoo 19 (nằm trong khối cấu trúc `<main>` hoặc thẻ `<div class="oe_kanban_global_click">`):

```xml
<record id="hr_kanban_view_simplified_inherit_health" model="ir.ui.view">
    <field name="name">hr.employee.kanban.inherit.health</field>
    <field name="model">hr.employee</field>
    <field name="inherit_id" ref="hr.hr_employee_kanban_view"/>
    <field name="arch" type="xml">
        <!-- Chèn biến check trạng thái sức khỏe -->
        <xpath expr="//templates" position="before">
            <field name="health_status"/>
            <field name="last_health_type"/>
        </xpath>
        <!-- Chèn thẻ cảnh báo y tế dạng Ribbon/Badge trực quan bằng CSS Bootstrap 5 -->
        <xpath expr="//div[contains(@class, 'oe_kanban_details')]" position="inside">
            <div class="mt-2">
                <t t-if="record.health_status.raw_value == 'warning'">
                    <span class="badge bg-warning text-dark"><i class="fa fa-exclamation-triangle"/> Cảnh báo y tế: 6 tháng chưa khám</span>
                </t>
                <t t-if="record.health_status.raw_value == 'danger' or record.health_status.raw_value == 'none'">
                    <span class="badge bg-danger"><i class="fa fa-times-circle"/> Quá hạn khám sức khỏe (12T+)</span>
                </t>
                <t t-if="record.last_health_type.raw_value in ['type_4', 'type_5']">
                    <span class="badge bg-danger ms-1" title="Sức khỏe Loại IV/V - Hạn chế lao động nặng!"><i class="fa fa-heartbeat"/> Sức khỏe yếu</span>
                </t>
            </div>
        </xpath>
    </field>
</record>
```

---
