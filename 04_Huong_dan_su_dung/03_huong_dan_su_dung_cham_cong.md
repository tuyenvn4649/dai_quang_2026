# HƯỚNG DẪN SỬ DỤNG
## PHÂN HỆ: KẾT NỐI MÁY CHẤM CÔNG ZK ADMS VÀ LOGIC TÍNH CÔNG 24H (Odoo 19 CE)
**Dự án:** Nâng cấp & Chuẩn hóa Hệ thống Nhân sự Đại Quang
**Đối tượng hướng dẫn:** Bộ phận Hành chính Nhân sự (HR Admin), Tổ trưởng sản xuất và Nhân viên vận hành thiết bị
**Phiên bản tài liệu:** v1.0
**Ngày biên soạn:** 2026-07-05

---

## 1. QUY TRÌNH ĐĂNG KÝ VÂN TAY & ĐỒNG BỘ NHÂN SỰ MỚI

Khi tuyển dụng nhân viên mới, HR thực hiện quy trình sau để đảm bảo máy chấm công nhận diện chính xác:

### Bước 1: Khai báo Mã PIN trên Odoo
1.  Truy cập menu **Employees (Nhân sự) ➔ Employees** và mở hồ sơ nhân sự mới.
2.  Tại Tab **"Thông tin Pháp lý & Hợp đồng"**, điền mã **PIN chấm công** (Mặc định hệ thống tự điền bằng số mã nhân viên rút gọn).
3.  Lưu hồ sơ.

### Bước 2: Đăng ký Vân tay trên thiết bị tại Chi nhánh
1.  Đến trực tiếp máy chấm công ZKTeco tại xưởng/chi nhánh.
2.  Nhấn giữ nút **M/OK** để vào menu quản trị máy.
3.  Chọn **Người dùng mới (New User)**.
4.  Tại mục **ID người dùng (User ID)**: Nhập chính xác mã **PIN chấm công** đã cấp trên Odoo ở Bước 1.
5.  Chọn **Đăng ký Vân tay (Fingerprint)** và cho nhân viên tiến hành quét vân tay 3 lần để máy nhận diện. Nhấn **OK** để lưu lại trên máy.

### Bước 3: Đồng bộ tự động về Odoo
1.  Máy chấm công sử dụng giao thức ADMS tự động đẩy mẫu vân tay mã hóa về lưu trữ trên Odoo.
2.  HR kiểm tra lại trên Odoo bằng cách mở hồ sơ nhân viên ➔ Tab **"Sinh trắc học & Vân tay"**. Nếu bảng hiển thị dòng mẫu ngón tay đã đăng ký nghĩa là đồng bộ thành công.

---

## 2. QUẢN LÝ THIẾT BỊ CHẤM CÔNG TRÊN ODOO (`hr.zk.device`)

HR hoặc Admin hệ thống có thể theo dõi và điều khiển máy chấm công từ xa:

1.  Truy cập menu **Attendance (Chấm công) ➔ Configuration ➔ Devices (Thiết bị)**.
2.  Quan sát cột **Trạng thái**:
    *   🟢 **Đang hoạt động (Online):** Thiết bị đang kết nối mạng internet bình thường và sẵn sàng đẩy dữ liệu.
    *   🔴 **Mất kết nối (Offline):** Thiết bị mất mạng hoặc mất điện. HR cần thông báo kỹ thuật chi nhánh kiểm tra ngay để tránh gián đoạn dữ liệu chấm công.
3.  **Đẩy lệnh từ xa (Device Action):**
    *   *Chuyển nhân viên sang chi nhánh khác:* Chọn nhân viên, nhấn **"Đẩy dữ liệu sang máy mới"**, chọn máy chấm công đích. Hệ thống tự động truyền mẫu vân tay sang máy chi nhánh mới mà nhân viên không cần phải quét lại vân tay.
    *   *Xóa nhân sự nghỉ việc:* Khi chuyển trạng thái nhân viên thành "Đã nghỉ việc", Odoo sẽ tự động đẩy lệnh xóa vân tay của nhân viên đó trên toàn bộ máy chấm công.

---

## 3. TRA CỨU SỰ KIỆN QUÉT THẺ THÔ (`hr.attendance.event`)

Khi nhân viên thắc mắc về thời gian quét thẻ thực tế (ví dụ: *"Tôi nhớ đã quét thẻ lúc 17:05 nhưng bảng công chỉ ghi nhận về 17:00"*):

1.  Truy cập menu **Attendance ➔ Attendance Events (Sự kiện quét thô)**.
2.  Sử dụng bộ lọc nhập **Tên nhân viên** hoặc mã **PIN**.
3.  Quan sát cột **Thời gian quét thô**: Đây là mốc thời gian chính xác từng giây mà máy chấm công ghi nhận khi nhân viên quét vân tay (chưa qua xử lý làm tròn).

---

## 4. XEM VÀ KIỂM TRA BẢNG CÔNG NGÀY CHUYÊN SÂU (`hr.attendance.day`)

Hệ thống tự động thực hiện gộp ca kíp 24h và làm tròn 15 phút để sinh ra bảng công ngày:

### 4.1. Quan sát quy tắc làm tròn 15 phút thực tế:
Truy cập menu **Attendance ➔ Daily Attendance (Công ngày)**, hệ thống hiển thị:
*   *Lượt quét vào thô:* 07:51 ➔ *Giờ vào tính công (Đã làm tròn):* 08:00 (Làm tròn lên).
*   *Lượt quét ra thô:* 17:05 ➔ *Giờ ra tính công (Đã làm tròn):* 17:00 (Làm tròn xuống).
*   *Số giờ làm việc:* 9.0 giờ.
*   *Hệ số ngày công:* 1.0 công.

### 4.2. Kiểm tra ca đêm gối ngày:
Đối với nhân viên làm ca kíp (ví dụ: vào ca lúc 22:00 ngày 05/07 và ra ca lúc 06:00 sáng ngày 06/07):
*   Hệ thống tự động nhận diện lượt check-out lúc 06:00 sáng nằm trong chu kỳ 24h của ca làm việc bắt đầu ngày 05/07.
*   Bảng công ngày chỉ tạo duy nhất 1 dòng cho ngày **05/07** với giờ vào `22:00`, giờ ra `06:00` (đã quy đổi). Hệ thống không tạo dòng chấm công thừa cho ngày 06/07.

### 4.3. Báo cáo bảng công Pivot (Month-end check)
1.  Tại giao diện **Daily Attendance**, nhấn biểu tượng **Pivot View** (biểu tượng bảng ma trận) ở góc phải.
2.  Hệ thống hiển thị ma trận công tổng hợp của toàn bộ nhân viên theo từng ngày trong tháng.
3.  HR rà soát các ô có giá trị `0.0` công (nghỉ việc, nghỉ không phép hoặc quên quét thẻ) để yêu cầu giải trình trước khi chốt bảng công lương.

---

## 5. ĐIỀU CHỈNH & BỔ SUNG CÔNG THỦ CÔNG (GIẢI TRÌNH QUÊN QUÉT THẺ)

Trường hợp nhân viên quên quét thẻ hoặc đi công tác bên ngoài, HR thực hiện bổ sung công bằng tay:

1.  Truy cập menu **Attendance ➔ Daily Attendance**.
2.  Tìm ngày công bị thiếu của nhân sự hoặc nhấn **New (Tạo mới)**.
3.  Chọn tên **Nhân viên**, nhập **Ngày công**, chọn **Ca làm việc**.
4.  Nhập giờ vào/ra mong muốn vào trường **Giờ vào tính công (Đã làm tròn)** và **Giờ ra tính công (Đã làm tròn)**.
5.  Tại ô **Ghi chú**, điền lý do bổ sung (ví dụ: *Đi công tác gặp khách hàng / Quên quét thẻ có xác nhận của tổ trưởng*).
6.  Nhấn **Save (Lưu)**. Hệ thống ghi nhận công bổ sung và đánh dấu nhãn *"Điều chỉnh thủ công"* để phục vụ hậu kiểm.

---
