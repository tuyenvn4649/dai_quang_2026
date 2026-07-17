# TÀI LIỆU ĐẶC TẢ THIẾT KẾ KỸ THUẬT (TSD)
## PHÂN HỆ: QUẢN LÝ PHƯƠNG TIỆN VÀ CHI PHÍ VẬN HÀNH ĐỘI XE (Odoo 19 CE)
**Dự án:** Nâng cấp & Chuẩn hóa Hệ thống Nhân sự Đại Quang
**Phiên bản tài liệu:** v1.0
**Ngày biên soạn:** 2026-07-06

---

## 1. CẤU TRÚC MODULE & QUAN HỆ PHỤ THUỘC

### 1.1. Tên module dự kiến
`daiquang_fleet_management`

### 1.2. Danh sách phụ thuộc (Dependencies)
```python
{
    'name': 'Dai Quang Fleet & Expense Management Customization',
    'version': '19.0.1.0.0',
    'category': 'Human Resources/Fleet',
    'depends': [
        'fleet',
        'branch', # quản lý xe theo chi nhánh
    ],
    'data': [
        'security/ir.model.access.csv',
        'views/fleet_vehicle_views.xml',
        'views/fleet_vehicle_log_fuel_views.xml',
        'views/fleet_vehicle_log_services_views.xml',
        'data/cron_data.xml',
    ],
    'installable': True,
    'application': False,
}
```

---

## 2. THIẾT KẾ CƠ SỞ DỮ LIỆU (DATABASE SCHEMA)

### 2.1. Kế thừa Hồ sơ xe (`fleet.vehicle`)
Bổ sung các trường lưu trữ thông số định mức xăng, ngày đăng kiểm và hạn thay dầu máy:

| Tên trường (Field Name) | Kiểu dữ liệu | Mô tả |
| :--- | :--- | :--- |
| `fuel_limit` | Float | Định mức tiêu thụ chuẩn quy định ($L_0$ lít/100km) |
| `last_oil_change_odometer`| Integer | Số Km tại lần thay dầu máy gần nhất |
| `next_oil_change_odometer`| Integer | Compute, Số Km phải thay dầu lần kế tiếp (`last_oil_change_odometer + 5000`) |
| `oil_change_needed` | Boolean | Compute, `store=True`. Báo đỏ nếu Km hiện tại vượt mốc thay dầu |
| `last_inspection_date` | Date | Ngày đăng kiểm gần nhất |
| `inspection_cycle` | Integer | Chu kỳ đăng kiểm của xe (Số ngày, ví dụ: 180 hoặc 365) |
| `next_inspection_date` | Date | Compute, `store=True`. Ngày đăng kiểm tiếp theo |
| `inspection_needed` | Boolean | Compute, `store=True`. Báo đỏ trước ngày đăng kiểm 15 ngày |

### 2.2. Kế thừa Nhật ký đổ nhiên liệu (`fleet.vehicle.log.fuel`)
Bổ sung các trường tính toán tỷ lệ tiêu hao nhiên liệu thực tế:

| Tên trường (Field Name) | Kiểu dữ liệu | Mô tả |
| :--- | :--- | :--- |
| `fuel_consumption_rate` | Float | Compute, `store=True`. Lượng tiêu hao thực tế (lít/100km) |
| `alert_level` | Selection | Compute, `store=True`. Mức độ cảnh báo (`green`, `yellow`, `red`) |

### 2.3. Kế thừa Nhật ký Chi phí & Dịch vụ xe (`fleet.vehicle.log.services`)
Bổ sung phân nhóm chi phí độc lập theo 4 nhóm nghiệp vụ của Đại Quang:

| Tên trường (Field Name) | Kiểu dữ liệu | Cấu hình | Mô tả |
| :--- | :--- | :--- | :--- |
| `cost_type` | Selection | `'fuel'` (Xăng dầu), `'maintenance'` (Bảo dưỡng/Sửa chữa), `'inspection'` (Đăng kiểm), `'toll'` (Phí cầu đường/BOT) | Phân loại chi phí vận hành xe |

---

## 3. LẬP TRÌNH NGHIỆP VỤ & LỚP XỬ LÝ (LOGIC METHODS)

### 3.1. Thuật toán tính tiêu thụ thực tế & Đưa ra Cảnh báo xăng dầu
Khi tài xế lưu một phiếu đổ xăng mới, hệ thống tự động tìm phiếu đổ xăng liền trước của cùng xe để tính khoảng cách Km di chuyển, từ đó tính ra Lít/100km thực tế và đưa ra cảnh báo màu sắc:

```python
from odoo import api, fields, models, _
from odoo.exceptions import UserError

class FleetVehicleLogFuel(models.Model):
    _inherit = 'fleet.vehicle.log.fuel'

    fuel_consumption_rate = fields.Float(
        string='Tiêu hao thực tế (L/100km)', 
        compute='_compute_consumption_rate', 
        store=True
    )
    alert_level = fields.Selection([
        ('green', 'Bình thường'),
        ('yellow', 'Cảnh báo Vàng (Vượt 10-20%)'),
        ('red', 'Báo động Đỏ (Vượt >20%)')
    ], string='Mức cảnh báo', compute='_compute_consumption_rate', store=True, default='green')

    @api.depends('odometer', 'amount', 'vehicle_id', 'vehicle_id.fuel_limit')
    def _compute_consumption_rate(self):
        for log in self:
            if not log.vehicle_id or not log.odometer or log.amount <= 0:
                log.fuel_consumption_rate = 0.0
                log.alert_level = 'green'
                continue

            # 1. Tìm phiếu đổ xăng liền trước của cùng xe
            prev_log = self.search([
                ('vehicle_id', '=', log.vehicle_id.id),
                ('odometer', '<', log.odometer),
                ('id', '!=', log.id)
            ], order='odometer desc', limit=1)

            if not prev_log:
                # Không có phiếu trước đó -> Chưa đủ cơ sở tính định mức
                log.fuel_consumption_rate = 0.0
                log.alert_level = 'green'
                continue

            # 2. Tính số Km di chuyển giữa 2 lần đổ xăng đầy bình
            distance = log.odometer - prev_log.odometer
            if distance <= 0:
                log.fuel_consumption_rate = 0.0
                log.alert_level = 'green'
                continue

            # 3. Tính toán lượng tiêu thụ thực tế (Số lít xăng / Khoảng cách * 100)
            # Giả định trường 'liter' lưu số lít đổ đầy (nếu dùng trường volume mặc định của Odoo)
            liters = log.volume
            rate = (liters / distance) * 100.0
            log.fuel_consumption_rate = rate

            # 4. So sánh với định mức xăng quy định của xe (vehicle_id.fuel_limit)
            limit = log.vehicle_id.fuel_limit
            if not limit or limit <= 0:
                log.alert_level = 'green'
                continue

            # Áp dụng logic cảnh báo
            if rate <= limit * 1.1:
                log.alert_level = 'green'
            elif rate <= limit * 1.2:
                log.alert_level = 'yellow'
            else:
                log.alert_level = 'red'
```

### 3.2. Cấu hình Cảnh báo nhắc thay dầu máy & Hạn đăng kiểm
Tính toán tự động các hạn bảo dưỡng và đăng kiểm dựa trên số Km hiện tại của xe:

```python
class FleetVehicle(models.Model):
    _inherit = 'fleet.vehicle'

    next_oil_change_odometer = fields.Integer(
        string='Mốc thay dầu kế tiếp (Km)', 
        compute='_compute_oil_change', 
        store=True
    )
    oil_change_needed = fields.Boolean(
        string='Cần thay dầu máy', 
        compute='_compute_oil_change', 
        store=True, 
        default=False
    )
    
    next_inspection_date = fields.Date(
        string='Ngày đăng kiểm tiếp theo', 
        compute='_compute_inspection', 
        store=True
    )
    inspection_needed = fields.Boolean(
        string='Sắp hết hạn đăng kiểm', 
        compute='_compute_inspection', 
        store=True, 
        default=False
    )

    @api.depends('last_oil_change_odometer', 'odometer')
    def _compute_oil_change(self):
        for vehicle in self:
            vehicle.next_oil_change_odometer = vehicle.last_oil_change_odometer + 5000
            # Cảnh báo nếu số km hiện tại lớn hơn hoặc bằng mốc thay dầu kế tiếp
            vehicle.oil_change_needed = vehicle.odometer >= vehicle.next_oil_change_odometer

    @api.depends('last_inspection_date', 'inspection_cycle')
    def _compute_inspection(self):
        today = fields.Date.today()
        for vehicle in self:
            if vehicle.last_inspection_date and vehicle.inspection_cycle:
                next_date = vehicle.last_inspection_date + timedelta(days=vehicle.inspection_cycle)
                vehicle.next_inspection_date = next_date
                # Cảnh báo trước ngày hết hạn 15 ngày
                vehicle.inspection_needed = (next_date - today).days <= 15
            else:
                vehicle.next_inspection_date = False
                vehicle.inspection_needed = False

    @api.model
    def cron_check_fleet_reminders(self):
        """
        Cron Job chạy hàng đêm cập nhật lại trạng thái cảnh báo đăng kiểm
        khi ngày hiện tại (fields.Date.today()) thay đổi.
        """
        vehicles = self.search([('last_inspection_date', '!=', False)])
        vehicles._compute_inspection()
```

---

## 4. ĐẶC TẢ GIAO DIỆN & TƯƠNG THÍCH ODOO 19

### 4.1. Kế thừa Form Nhật ký Nhiên liệu hiển thị Cảnh báo Màu sắc
```xml
<record id="fleet_vehicle_log_fuel_view_form_inherit_dq" model="ir.ui.view">
    <field name="name">fleet.vehicle.log.fuel.form.inherit.dq</field>
    <field name="model">fleet.vehicle.log.fuel</field>
    <field name="inherit_id" ref="fleet.fleet_vehicle_log_fuel_view_form"/>
    <field name="arch" type="xml">
        <xpath expr="//group" position="inside">
            <group string="Phân tích định mức (Đổ đầy bình)">
                <field name="fuel_consumption_rate" readonly="1"/>
                <!-- Hiển thị màu sắc nổi bật bằng CSS widget trạng thái -->
                <field name="alert_level" decoration-success="alert_level == 'green'" decoration-warning="alert_level == 'yellow'" decoration-danger="alert_level == 'red'" widget="badge"/>
            </group>
        </xpath>
    </field>
</record>
```

### 4.2. Kế thừa Kanban Hồ sơ Xe hiển thị Badge Cảnh báo Đăng kiểm/Thay dầu
Tương thích với cú pháp Kanban mới của Odoo 19:

```xml
<record id="fleet_vehicle_view_kanban_inherit_dq" model="ir.ui.view">
    <field name="name">fleet.vehicle.kanban.inherit.dq</field>
    <field name="model">fleet.vehicle</field>
    <field name="inherit_id" ref="fleet.fleet_vehicle_view_kanban"/>
    <field name="arch" type="xml">
        <xpath expr="//templates" position="before">
            <field name="oil_change_needed"/>
            <field name="inspection_needed"/>
        </xpath>
        <xpath expr="//div[contains(@class, 'oe_kanban_details')]" position="inside">
            <div class="mt-2">
                <t t-if="record.oil_change_needed.raw_value">
                    <span class="badge bg-danger"><i class="fa fa-wrench"/> Cần thay dầu máy (5,000 km+)</span>
                </t>
                <t t-if="record.inspection_needed.raw_value">
                    <span class="badge bg-danger ms-1"><i class="fa fa-calendar"/> Sắp hết hạn Đăng kiểm (15 ngày)</span>
                </t>
            </div>
        </xpath>
    </field>
</record>
```

---
