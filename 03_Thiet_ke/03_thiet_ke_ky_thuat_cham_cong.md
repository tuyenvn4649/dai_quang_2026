# TÀI LIỆU ĐẶC TẢ THIẾT KẾ KỸ THUẬT (TSD)
## PHÂN HỆ: KẾT NỐI MÁY CHẤM CÔNG ZK ADMS VÀ LOGIC TÍNH CÔNG 24H (Odoo 19 CE)
**Dự án:** Nâng cấp & Chuẩn hóa Hệ thống Nhân sự Đại Quang
**Phiên bản tài liệu:** v1.0
**Ngày biên soạn:** 2026-07-05

---

## 1. KIẾN TRÚC HỆ THỐNG & ĐƯỜNG TRUYỀN API (ADMS CONTROLLER)

Phân hệ sử dụng mô hình đẩy dữ liệu (Push SDK). Thiết bị chấm công đóng vai trò HTTP Client tự động gửi các truy vấn đến Odoo Web Server (HTTP Server) thông qua các Route mở rộng không yêu cầu xác thực phiên đăng nhập (`auth='none'`).

### 1.1. Các API Endpoints định nghĩa trên Odoo
*   **Nhận log chấm công & Vân tay (`/iclock/cdata`):**
    *   *Phương thức:* `GET` (để thiết lập cấu hình ban đầu cho máy) và `POST` (để nhận dữ liệu).
    *   *Nội dung nhận:* Log quét thẻ thô (`ATTLOG`) và Nhật ký thao tác vân tay (`OPERLOG` / `FPID`).
*   **Đẩy tập lệnh từ Odoo xuống máy (`/iclock/getrequest`):**
    *   *Phương thức:* `GET`. Máy chấm công định kỳ gửi yêu cầu để lấy các lệnh chờ thực thi (Thêm vân tay mới, xóa nhân sự...).
*   **Xác nhận thực thi lệnh (`/iclock/devicecmd`):**
    *   *Phương thức:* `POST`. Máy chấm công phản hồi trạng thái thực hiện lệnh (Thành công / Thất bại) về Odoo.

---

## 2. THIẾT KẾ MÔ HÌNH DỮ LIỆU (DATABASE SCHEMA)

### 2.1. Cấu hình Máy chấm công (`hr.zk.device`)
```python
class HrZkDevice(models.Model):
    _name = 'hr.zk.device'
    _description = 'ZKTeco ADMS Device'

    name = fields.Char(required=True, string='Tên máy chấm công')
    serial_number = fields.Char(required=True, string='Số Serial (SN)', index=True, copy=False)
    branch_id = fields.Many2one('res.branch', string='Chi nhánh liên kết', required=True)
    last_activity = fields.Datetime(string='Lần kết nối gần nhất')
    state = fields.Selection([
        ('online', 'Đang hoạt động'),
        ('offline', 'Mất kết nối')
    ], string='Trạng thái', compute='_compute_state', store=True)
```

### 2.2. Dữ liệu Mẫu vân tay (`hr.zk.fingerprint.template`)
```python
class HrZkFingerprintTemplate(models.Model):
    _name = 'hr.zk.fingerprint.template'
    _description = 'ZK Fingerprint Template'
    _rec_name = 'employee_id'

    employee_id = fields.Many2one('hr.employee', string='Nhân sự liên kết', ondelete='cascade', required=True)
    pin = fields.Char(string='Mã PIN chấm công', required=True, index=True)
    finger_id = fields.Integer(string='ID Ngón tay (0-9)', required=True)
    template_data = fields.Text(string='Dữ liệu vân tay mã hóa', required=True) # Chuỗi Base64
```

### 2.3. Sự kiện quét thẻ thô (`hr.attendance.event`)
```python
class HrAttendanceEvent(models.Model):
    _name = 'hr.attendance.event'
    _description = 'Raw Attendance Event'
    _order = 'timestamp desc'

    device_id = fields.Many2one('hr.zk.device', string='Thiết bị chấm công')
    pin = fields.Char(string='Mã PIN thô', index=True, required=True)
    employee_id = fields.Many2one('hr.employee', string='Nhân viên', index=True)
    timestamp = fields.Datetime(string='Thời gian quét thô (UTC)', required=True)
    event_type = fields.Selection([
        ('0', 'Check-In'),
        ('1', 'Check-Out'),
        ('15', 'Quét tự do / Định danh')
    ], string='Kiểu quét thô', default='15')
```

### 2.4. Bảng công ngày xử lý (`hr.attendance.day`)
Lưu trữ kết quả công đã được gộp ca và làm tròn:

```python
class HrAttendanceDay(models.Model):
    _name = 'hr.attendance.day'
    _description = 'Processed Daily Attendance'
    _rec_name = 'employee_id'
    _order = 'date desc, employee_id'

    employee_id = fields.Many2one('hr.employee', string='Nhân viên', required=True, index=True, ondelete='cascade')
    date = fields.Date(string='Ngày công', required=True, index=True)
    shift_id = fields.Many2one('hr.shift', string='Ca làm việc áp dụng')
    
    check_in_raw = fields.Datetime(string='Lượt quét vào thô')
    check_out_raw = fields.Datetime(string='Lượt quét ra thô')
    
    check_in_rounded = fields.Datetime(string='Giờ vào tính công (Đã làm tròn)')
    check_out_rounded = fields.Datetime(string='Giờ ra tính công (Đã làm tròn)')
    
    work_hours = fields.Float(string='Số giờ làm việc thực tế', compute='_compute_hours', store=True)
    attendance_coef = fields.Float(string='Hệ số ngày công (ví dụ: 1.0, 0.5)', default=0.0)
    ot_hours = fields.Float(string='Số giờ tăng ca (OT)', default=0.0)
```

---

## 3. LẬP TRÌNH LOGIC NGHIỆP VỤ (COMPUTATION LOGIC)

### 3.1. Thuật toán làm tròn giờ 15 phút (15-Minute Rounding)
Viết hàm helper chuyển múi giờ sang GMT+7, thực hiện làm tròn giờ quét theo block 15 phút trước khi ghi nhận tính toán:

```python
import pytz
from datetime import datetime, timedelta

def round_attendance_time(dt_utc, is_checkin=True):
    """
    dt_utc: Datetime đối tượng ở dạng UTC
    is_checkin: True -> Làm tròn LÊN mốc 15 phút gần nhất
                False -> Làm tròn XUỐNG mốc 15 phút gần nhất
    """
    if not dt_utc:
        return False
        
    # 1. Chuyển sang múi giờ Việt Nam (Asia/Ho_Chi_Minh) để làm tròn
    tz_vietnam = pytz.timezone('Asia/Ho_Chi_Minh')
    dt_local = dt_utc.astimezone(tz_vietnam)
    
    minute = dt_local.minute
    minute_rem = minute % 15
    
    if is_checkin:
        # Làm tròn lên mốc 15 phút gần nhất
        if minute_rem > 0:
            dt_rounded = dt_local + timedelta(minutes=(15 - minute_rem))
        else:
            dt_rounded = dt_local
        # Đặt lại giây và microsecond về 0
        dt_rounded = dt_rounded.replace(second=0, microsecond=0)
    else:
        # Làm tròn xuống mốc 15 phút gần nhất
        dt_rounded = dt_local - timedelta(minutes=minute_rem)
        dt_rounded = dt_rounded.replace(second=0, microsecond=0)
        
    # 2. Convert ngược lại múi giờ UTC để lưu vào database
    return dt_rounded.astimezone(pytz.utc).replace(tzinfo=None)
```

### 3.2. Thuật toán khớp ca làm việc chu kỳ 24h (24-Hour Cycle Matching)
Hàm quét qua các sự kiện quét thẻ thô của nhân viên trong ngày để ghép cặp check-in/out theo chu kỳ ca kíp:

```python
class HrAttendanceDay(models.Model):
    _inherit = 'hr.attendance.day'

    @api.model
    def process_attendance_events_to_day(self, employee_id, checkin_event):
        """
        employee_id: Nhân viên cần tính công
        checkin_event: Bản ghi hr.attendance.event (Check-in bắt đầu ca)
        """
        # 1. Định nghĩa chu kỳ 24h kể từ lúc check-in
        start_time_utc = checkin_event.timestamp
        end_time_utc = start_time_utc + timedelta(hours=24)
        
        # 2. Tìm tất cả các lượt quét của nhân viên trong chu kỳ 24h này
        events = self.env['hr.attendance.event'].search([
            ('employee_id', '=', employee_id),
            ('timestamp', '>=', start_time_utc),
            ('timestamp', '<=', end_time_utc)
        ], order='timestamp asc')
        
        if not events:
            return False
            
        # Lượt quét đầu tiên là check-in
        first_scan = events[0].timestamp
        # Lượt quét cuối cùng là check-out
        last_scan = events[-1].timestamp if len(events) > 1 else False
        
        # 3. Áp dụng làm tròn 15 phút
        in_rounded = round_attendance_time(first_scan, is_checkin=True)
        out_rounded = round_attendance_time(last_scan, is_checkin=False) if last_scan else False
        
        # 4. Xác định ngày tính công (dựa trên ngày bắt đầu ca theo giờ địa phương)
        tz_vn = pytz.timezone('Asia/Ho_Chi_Minh')
        local_date = first_scan.astimezone(tz_vn).date()
        
        # 5. Lưu kết quả vào hr.attendance.day
        day_record = self.search([
            ('employee_id', '=', employee_id),
            ('date', '=', local_date)
        ], limit=1)
        
        vals = {
            'employee_id': employee_id,
            'date': local_date,
            'check_in_raw': first_scan,
            'check_out_raw': last_scan,
            'check_in_rounded': in_rounded,
            'check_out_rounded': out_rounded,
        }
        
        if day_record:
            day_record.write(vals)
        else:
            self.create(vals)
```

---

## 4. XỬ LÝ GIAO THỨC ADMS TRÊN CONTROLLER (ADMS WEB CONTROLLER)

Đoạn mã Python Controller kế thừa xử lý luồng nhận dữ liệu trực tiếp từ máy chấm công ZKTeco gửi lên:

```python
import logging
from odoo import http
from odoo.http import request

_logger = logging.getLogger(__name__)

class ZkAdmsController(http.Controller):

    @http.route('/iclock/cdata', type='http', auth='none', methods=['POST', 'GET'])
    def iclock_cdata(self, **kwargs):
        """
        Nhận gói tin ATTLOG (dữ liệu quẹt thẻ) đẩy lên từ máy chấm công.
        """
        req = request.httprequest
        # Lấy serial number của máy gửi lên qua query string
        sn = kwargs.get('SN')
        device = request.env['hr.zk.device'].sudo().search([('serial_number', '=', sn)], limit=1)
        if not device:
            _logger.error("ADMS Device with SN %s not found", sn)
            return "OK" # Vẫn trả về OK để máy không bị treo gửi lại
            
        # Cập nhật kết nối của máy
        device.write({'last_activity': fields.Datetime.now()})
        
        if req.method == 'POST':
            # Nội dung log thô nằm ở body
            raw_data = req.data.decode('utf-8')
            lines = raw_data.split('\n')
            for line in lines:
                if not line.strip():
                    continue
                # Định dạng ATTLOG: PIN\tDATETIME\tSTATUS\t...
                parts = line.split('\t')
                if len(parts) >= 2:
                    pin = parts[0]
                    dt_str = parts[1]
                    # Chuyển đổi chuỗi thời gian của máy sang UTC Datetime
                    # Ví dụ: 2026-07-05 23:25:00
                    try:
                        dt_naive = datetime.strptime(dt_str, '%Y-%m-%d %H:%M:%S')
                        # Máy đẩy lên là giờ địa phương (Asia/Ho_Chi_Minh) -> Convert sang UTC
                        tz_vn = pytz.timezone('Asia/Ho_Chi_Minh')
                        dt_local = tz_vn.localize(dt_naive)
                        dt_utc = dt_local.astimezone(pytz.utc).replace(tzinfo=None)
                        
                        # Tìm nhân viên tương ứng với mã PIN
                        emp = request.env['hr.employee'].sudo().search([
                            ('pin', '=', pin)
                        ], limit=1)
                        
                        # Tạo log sự kiện thô
                        request.env['hr.attendance.event'].sudo().create({
                            'device_id': device.id,
                            'pin': pin,
                            'employee_id': emp.id if emp else False,
                            'timestamp': dt_utc,
                            'event_type': parts[2] if len(parts) > 2 else '15'
                        })
                    except Exception as e:
                        _logger.error("Failed to parse ADMS log line: %s, Error: %s", line, str(e))
                        
        return "GBY" # Trả về lệnh GBY (Goodbye) để máy chấm công biết Odoo đã nhận thành công
```

---
