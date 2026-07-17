# HƯỚNG DẪN SỬ DỤNG
## PHÂN HỆ: QUẢN LÝ PHƯƠNG TIỆN VÀ CHI PHÍ VẬN HÀNH ĐỘI XE (Odoo 19 CE)
**Dự án:** Nâng cấp & Chuẩn hóa Hệ thống Nhân sự Đại Quang
**Đối tượng hướng dẫn:** Bộ phận Hành chính (Fleet Admin), Tài xế công ty và Kế toán chi phí
**Phiên bản tài liệu:** v1.0
**Ngày biên soạn:** 2026-07-06

---

## 1. THIẾT LẬP THÔNG SỐ XE BAN ĐẦU

Để hệ thống tính toán chính xác định mức và đưa ra cảnh báo tự động, Fleet Admin thực hiện khai báo các thông số ban đầu của xe:

### Các bước thực hiện:
1.  Truy cập menu **Fleet (Đội xe) ➔ Vehicles (Phương tiện)** và chọn xe cần thiết lập.
2.  Tại Tab **"Cấu hình định mức & bảo dưỡng"**, khai báo:
    *   **Định mức tiêu thụ chuẩn (L/100km):** Nhập mức tiêu thụ quy định của xe (Ví dụ: Xe tải chở hàng định mức là `12.0` L/100km).
    *   **Số Km thay dầu gần nhất:** Số Km hiển thị trên đồng hồ tại thời điểm thay dầu máy gần nhất.
    *   **Ngày đăng kiểm gần nhất:** Ngày đăng kiểm ghi trên sổ đăng kiểm của xe.
    *   **Chu kỳ đăng kiểm (Ngày):** Tần suất đăng kiểm (Ví dụ: xe mới là `365` ngày, xe cũ là `180` ngày).
3.  Nhấn **Save (Lưu)**.

---

## 2. QUY TRÌNH TÀI XẾ NHẬP PHIẾU XĂNG (MOBILE / PORTAL)

Yêu cầu bắt buộc để công thức tính tiêu thụ nhiên liệu hoạt động chính xác: **Mỗi lần đổ xăng bắt buộc phải đổ đầy bình xăng.**

### Các bước thực hiện (trên điện thoại di động):
1.  Đăng nhập Odoo trên điện thoại, truy cập **Fleet ➔ Fuel Logs (Nhật ký xăng dầu)**.
2.  Nhấn nút **New (Tạo mới)** và nhập:
    *   **Phương tiện:** Chọn biển số xe đang đi.
    *   **Số Km hiện tại (Odometer):** Nhập chỉ số hiển thị trên mặt đồng hồ xe lúc đang ở cây xăng.
    *   **Số lít xăng đã đổ đầy (Volume):** Nhập số lít xăng ghi trên cột bơm xăng (Số lít thực tế để đổ đầy bình).
    *   **Tổng số tiền:** Nhập số tiền thanh toán ghi trên hóa đơn.
3.  Chụp ảnh hóa đơn xăng và đính kèm vào phần file đính kèm.
4.  Nhấn **Save (Lưu)**.
    *   *Cơ chế tự động:* Hệ thống sẽ so sánh chỉ số Km này với chỉ số Km của lần đổ xăng đầy bình trước đó để tính ra mức tiêu hao thực tế (L/100km) và hiển thị nhãn cảnh báo màu sắc tương ứng.

---

## 3. GIÁM SÁT CẢNH BÁO TIÊU HAO NHIÊN LIỆU (DÀNH CHO KẾ TOÁN/ADMIN)

Kế toán và Fleet Admin truy cập danh sách phiếu xăng để kiểm soát hao hụt nhiên liệu:

1.  Truy cập menu **Fleet ➔ Fuel Logs**.
2.  Quan sát cột **Mức cảnh báo**:
    *   🟢 **Bình thường (Màu Xanh):** Xe chạy đúng định mức (hoặc vượt dưới 10%).
    *   🟡 **Cảnh báo Vàng:** Xe chạy hao xăng vượt định mức từ 10% - 20%. Admin cần nhắc nhở tài xế chú ý cung đường đi hoặc cách vận hành xe.
    *   🔴 **Báo động Đỏ:** Tiêu hao thực tế vượt định mức chuẩn > 20%. Kế toán cần yêu cầu tài xế giải trình hành trình, hoặc kiểm tra rò rỉ bình chứa xăng, hoặc đưa xe đi sửa chữa vì máy móc hỏng gây tốn nhiên liệu.

---

## 4. XỬ LÝ CẢNH BÁO BẢO DƯỠNG THAY DẦU MÁY & ĐĂNG KIỂM

Hệ thống tự động theo dõi Km xe chạy và ngày tháng để hiển thị cảnh báo đỏ trên Dashboard Đội xe:

### 4.1. Cách xử lý khi có cảnh báo "Cần thay dầu máy (5,000 km+)"
1.  Khi xe chạy quá 5,000 km tính từ lần thay dầu trước, thẻ xe hiển thị nhãn đỏ cảnh báo. Tài xế đưa xe đi thay dầu máy tại gara.
2.  Sau khi thay dầu xong, HR mở hồ sơ xe trên Odoo, cập nhật trường **"Số Km thay dầu gần nhất"** thành chỉ số Km hiện tại trên đồng hồ xe.
3.  Nhấn **Save**. Nhãn cảnh báo đỏ sẽ tự động biến mất và mốc nhắc tiếp theo được tự động cộng thêm 5,000 km.

### 4.2. Cách xử lý khi có cảnh báo "Sắp hết hạn Đăng kiểm (15 ngày)"
1.  Trước ngày hết hạn đăng kiểm 15 ngày, thẻ xe hiển thị nhãn cảnh báo đỏ. HR mang xe đi đăng kiểm tại trạm kiểm định.
2.  Sau khi hoàn thành và nhận sổ đăng kiểm mới, mở hồ sơ xe trên Odoo, cập nhật trường **"Ngày đăng kiểm gần nhất"** thành ngày đăng kiểm mới thực tế.
3.  Nhấn **Save**. Hệ thống tự động xóa nhãn cảnh báo đỏ và tự động tính ngày hết hạn cho chu kỳ đăng kiểm tiếp theo.

---

## 5. PHÂN LOẠI CHI PHÍ XE & BÁO CÁO TỔNG HỢP

Để phục vụ phân tích tài chính chi tiết của Đại Quang:

1.  Khi kế toán hoặc Admin nhập hóa đơn chi phí xe tại menu **Fleet ➔ Services (Nhật ký dịch vụ)**, bắt buộc chọn đúng **Phân loại chi phí**:
    *   *Xăng dầu:* Các hóa đơn xăng dầu.
    *   *Bảo dưỡng/Sửa chữa:* Thay dầu, sửa lốp, bảo dưỡng máy (không bao gồm đăng kiểm).
    *   *Đăng kiểm:* Phí kiểm định xe, phí đường bộ.
    *   *Vé cầu đường (BOT):* Chi phí cầu đường VETC/ePass.
2.  Truy cập menu **Fleet ➔ Reporting ➔ Costs (Báo cáo chi phí)** để xem biểu đồ cột/tròn tổng hợp chi phí của toàn bộ đội xe theo từng tháng và theo từng đầu xe.

---
