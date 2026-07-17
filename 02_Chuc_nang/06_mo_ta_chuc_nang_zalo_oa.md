# TÀI LIỆU MÔ TẢ CHỨC NĂNG (FSD)
## MODULE ĐỘC LẬP: KẾT NỐI API ZALO OFFICIAL ACCOUNT (daiquang_zalo_api)
**Dự án:** Nâng cấp & Chuẩn hóa Hệ thống Nhân sự Đại Quang
**Phiên bản tài liệu:** v1.0
**Ngày biên soạn:** 2026-07-05

---

## 1. TỔNG QUAN HỆ THỐNG

### 1.1. Bối cảnh & Mục tiêu
Để nâng cao trải nghiệm ứng viên và nhân sự, Công ty Đại Quang có nhu cầu gửi các thông báo tự động (thư mời nhận việc, phiếu lương, thông báo khuyết công, thông báo sinh nhật...) trực tiếp qua ứng dụng Zalo - nền tảng nhắn tin phổ biến nhất Việt Nam. 
Để tránh việc các phân hệ phải tự lập trình API riêng lẻ, hệ thống thiết kế một **Module kết nối trung gian độc lập (`daiquang_zalo_api`)**. Module này đóng vai trò cấu hình tập trung các tham số kết nối Zalo Official Account (OA), quản lý xác thực OAuth 2.0 và cung cấp cổng dịch vụ API dùng chung cho tất cả các phân hệ khác (Tuyển dụng, Chấm công, Bảng lương).

### 1.2. Phạm vi chức năng
*   **Cấu hình Zalo OA:** Thiết lập App ID, Secret Key, Access Token, Refresh Token tập trung.
*   **Cơ chế Tự động Duy trì Kết nối (OAuth 2.0 Auto-refresh):** Tự động làm mới Access Token định kỳ (tránh hết hạn sau 25 giờ).
*   **Quản lý mẫu ZNS (Zalo Notification Service):** Đăng ký và đồng bộ các biểu mẫu tin nhắn được duyệt từ Zalo.
*   **Nhật ký gửi tin (Logging System):** Theo dõi trạng thái gửi tin (Thành công, Thất bại) và nội dung chi tiết.

---

## 2. LUỒNG NGHIỆP VỤ TỔNG THỂ (WORKFLOW)

```
[Phân hệ Tuyển dụng / Lương / Chấm công]
                 │
                 ▼ (Gọi hàm dùng chung)
    [Module daiquang_zalo_api] ───(Kiểm tra/Refresh Token)───► [Zalo OAuth API]
                 │
                 ▼ (Format SĐT dạng 84xxx & Gửi ZNS)
         [Zalo Open API]
                 │
                 ▼ (Đẩy tin nhắn)
        [Điện thoại ứng viên]
```

---

## 3. MÔ TẢ CHỨC NĂNG CHI TIẾT

### 3.1. Giao diện cấu hình tham số Zalo OA
Hệ thống cung cấp một trang cấu hình độc lập tại **Settings ➔ Technical ➔ Zalo OA Configuration**:
*   **Thông tin ứng dụng (App Credentials):** App ID, Secret Key của ứng dụng tạo trên Zalo Developer.
*   **Thông tin định danh OA:** Official Account ID (OA ID) của Đại Quang.
*   **Khóa xác thực (Tokens):** Access Token (thời hạn 25 giờ) và Refresh Token (thời hạn 3 tháng).
*   **Nút "Kiểm tra kết nối":** Cho phép HR/Admin gửi thử 1 tin nhắn test đến số điện thoại quản trị để xác nhận kết nối thành công.

### 3.2. Cơ chế tự động duy trì Token (Cron Job OAuth 2.0)
Do Access Token của Zalo chỉ có hiệu lực trong 25 giờ và Refresh Token chỉ dùng được một lần để sinh cặp Token mới:
*   Hệ thống thiết lập một tiến trình tự động chạy ngầm (Cron Job) kiểm tra thời gian hết hạn của Access Token.
*   Khi Access Token còn hạn dưới 6 giờ, hệ thống tự động gọi API Zalo để đổi Refresh Token lấy cặp Access/Refresh Token mới và cập nhật ngược lại vào phần thiết lập.
*   *Lợi ích:* Đảm bảo kênh kết nối Zalo luôn hoạt động liên tục 24/7 mà không cần Admin phải thao tác thủ công hàng ngày.

### 3.3. Quản lý Mẫu tin nhắn Zalo ZNS (`zalo.zns.template`)
*   Khai báo ID mẫu tin nhắn (`Template ID`) được cấp bởi Zalo Cloud.
*   Định nghĩa danh mục các tham số động tương ứng của mẫu (Ví dụ: `customer_name`, `position`, `salary_amount`).
*   Cho phép liên kết ánh xạ các tham số này với các trường thông tin trên Odoo (ví dụ: `customer_name` mapping với trường `partner_name` của Ứng viên).

### 3.4. Nhật ký gửi tin nhắn (`zalo.message.log`)
Tất cả các tin nhắn gửi đi đều được lưu vết chi tiết:
*   **Thông tin ghi nhận:** Số điện thoại nhận, Loại tin nhắn (Thư tuyển dụng, Phiếu lương...), Nội dung gửi, Ngày gửi, Trạng thái (Thành công / Lỗi).
*   **Lưu vết lỗi chi tiết:** Nếu gửi thất bại, hệ thống lưu lại mã lỗi của Zalo (ví dụ: Mã 130 - Số điện thoại không sử dụng Zalo, Mã 108 - Sai chữ ký xác thực...) để HR có hướng xử lý thủ công.

---

## 4. YÊU CẦU PHÂN QUYỀN

*   **System Administrator (Admin):** Có toàn quyền cấu hình tham số kết nối, đổi Refresh Token bằng tay và xem toàn bộ nhật ký gửi tin nhắn lỗi.
*   **HR Users:** Không được quyền xem trang cấu hình Zalo OA; chỉ được quyền kích hoạt hành động gửi tin nhắn (nút bấm gửi trên Form ứng viên/phần phiếu lương) và xem trạng thái gửi tin trên Chatter.

---
