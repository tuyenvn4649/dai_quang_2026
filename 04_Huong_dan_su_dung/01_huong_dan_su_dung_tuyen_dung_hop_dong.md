# HƯỚNG DẪN SỬ DỤNG
## PHÂN HỆ: TUYỂN DỤNG VÀ QUẢN LÝ HỢP ĐỒNG LAO ĐỘNG (Odoo 19 CE)
**Dự án:** Nâng cấp & Chuẩn hóa Hệ thống Nhân sự Đại Quang
**Đối tượng hướng dẫn:** Bộ phận Tuyển dụng (HR Recruitment) và Hành chính Nhân sự (HR Admin)
**Phiên bản tài liệu:** v1.0
**Ngày biên soạn:** 2026-07-05

---

## 1. GIỚI THIỆU CHUNG
Tài liệu này hướng dẫn Chuyên viên Nhân sự thực hiện trọn vẹn quy trình từ lúc ứng viên trúng tuyển, lập thỏa thuận lương, duyệt và phân phối thư mời nhận việc (qua Email/Zalo), chuyển đổi ứng viên thành nhân viên chính thức đến khâu phê duyệt và quản lý hợp đồng lao động trên hệ thống Odoo 19 CE.

---

## 2. QUY TRÌNH 1: TIẾP NHẬN ỨNG VIÊN & THỎA THUẬN LƯƠNG

### Bước 1: Khởi tạo/Mở hồ sơ ứng viên
1.  Truy cập menu **Recruitment (Tuyển dụng) ➔ Applications (Ứng viên) ➔ All Applications**.
2.  Nhấn nút **New (Tạo mới)** hoặc click vào hồ sơ ứng viên hiện có.
3.  Nhập thông tin cơ bản: *Họ tên ứng viên, Email, Số điện thoại*.

### Bước 2: Nhập thông tin pháp lý & Tải hồ sơ đính kèm
1.  Kéo xuống phần **Notebook** ở nửa dưới màn hình, chọn Tab **"Thông tin Pháp lý & Hồ sơ đính kèm"**.
2.  Điền số **CCCD/CMND**, **Ngày cấp**, **Nơi cấp**.
3.  Chọn đúng **Chi nhánh tuyển dụng** (Trường bắt buộc để hệ thống phân cấp mã nhân sự sau này).
4.  Tải lên các file ảnh/PDF: *Mặt trước CCCD, Mặt sau CCCD, Sơ yếu lý lịch, Giấy khám sức khỏe* tại các ô tương ứng.

### Bước 3: Khai báo cấu trúc lương thỏa thuận
1.  Chuyển sang Tab **"Thỏa thuận Lương đề xuất"**.
2.  Nhập các số tiền thỏa thuận:
    *   **Lương cơ bản đề xuất** (Lương đóng BHXH).
    *   **Lương trách nhiệm đề xuất** (nếu có).
    *   **Lương kinh nghiệm/tay nghề đề xuất** (nếu có).
3.  Tại bảng **Phụ cấp hàng tháng**, nhấn *Add a line (Thêm một dòng)* để chọn loại phụ cấp (ví dụ: Ăn trưa, xăng xe) và nhập số tiền tương ứng.
4.  Nhấn **Save (Lưu)**. Hệ thống sẽ tự động:
    *   Cộng dồn tính toán tổng mức **Lương Gross đề xuất**.
    *   Trích khấu trừ tạm tính bảo hiểm của NLĐ để ra mức **Lương Net đề xuất**.
    *   Tự động dịch số tiền Net/Gross thành chữ tiếng Việt (ví dụ: *"Mười lăm triệu đồng chẵn"*) hiển thị ngay trên màn hình.

---

## 3. QUY TRÌNH 2: PHÊ DUYỆT & PHÂN PHỐI THƯ MỜI NHẬN VIỆC (OFFER LETTER)

### Bước 1: Chọn mẫu thư mời nhận việc
1.  Chuyển sang Tab **"Ký duyệt & In Thư Tuyển dụng"**.
2.  Tại trường **Mẫu Thư Tuyển Dụng**, chọn mẫu tương ứng:
    *   *Mẫu 1 - Thư mời nhận việc Khối Văn phòng.*
    *   *Mẫu 2 - Thư mời nhận việc Khối Nhà máy/Sản xuất.*
    *   *Mẫu 3 - Thư mời thử việc.*

### Bước 2: In kiểm tra bản PDF
1.  Nhìn lên thanh công cụ (Header) của form Ứng viên, nhấn nút **"In Thư Tuyển Dụng"**.
2.  Hệ thống sẽ kết xuất tự động toàn bộ nội dung mẫu kèm chữ ký, con dấu của Đại Quang thành file PDF và tự động tải xuống máy tính của bạn. Mở file để rà soát câu từ và số tiền lương đã chính xác chưa.

### Bước 3: Gửi Thư mời nhận việc qua Email
1.  Nếu ứng viên có địa chỉ Email, nút **"Gửi Email Thư Mời"** sẽ xuất hiện trên Header.
2.  HR nhấn nút này. Hệ thống tự động tạo email chúc mừng, đính kèm file PDF Thư mời nhận việc và gửi trực tiếp tới hòm thư ứng viên.
3.  Kiểm tra nhật ký tại phần **Chatter** (cột bên phải) để xác nhận hệ thống báo gửi email thành công.

### Bước 4: Gửi tin nhắn thông báo qua Zalo ZNS
1.  Nếu ứng viên có số điện thoại di động, nút **"Gửi Zalo Thư Mời"** sẽ xuất hiện.
2.  Nhấn nút **"Gửi Zalo Thư Mời"**. Odoo sẽ gọi Zalo OA của Đại Quang để gửi tin nhắn ZNS xác nhận trúng tuyển cùng tóm tắt mức lương Net/Gross trực tiếp vào số Zalo của ứng viên.
3.  Hệ thống sẽ ghi nhận log thành công hoặc báo lỗi (nếu số điện thoại không dùng Zalo) tại mục Chatter.

---

## 4. QUY TRÌNH 3: CHUYỂN ỨNG VIÊN THÀNH NHÂN VIÊN & KÍCH HOẠT HỢP ĐỒNG

### Bước 1: 1-Click Tạo Nhân viên chính thức
1.  Khi ứng viên xác nhận đồng ý nhận việc, nhấn nút **"Create Employee (Tạo Nhân viên)"** trên header hồ sơ tuyển dụng.
2.  Hệ thống tự động đóng hồ sơ tuyển dụng và mở ra form Nhân sự mới (`hr.employee`) với toàn bộ thông tin cá nhân, CCCD, ảnh chụp, file đính kèm được đồng bộ 100%.
3.  Nhấn **Save (Lưu)**. Mã nhân viên sẽ tự động sinh theo cấu trúc phân cấp (ví dụ: `DQ-HN-NS-0012`).

### Bước 2: Kiểm tra Hợp đồng lao động nháp
1.  Cùng lúc với thao tác tạo nhân viên, hệ thống đã tự động tạo một bản ghi **Hợp đồng lao động nháp** liên kết với nhân sự đó.
2.  Để kiểm tra, truy cập menu **Employee ➔ Contracts** hoặc click vào nút **Contracts (Hợp đồng)** ngay trên form Nhân viên.
3.  Mở hợp đồng ở trạng thái **Draft (Nháp)**. Kiểm tra các thông tin:
    *   Lương cơ bản (`wage`), phụ cấp trách nhiệm, phụ cấp tay nghề đã được ánh xạ tự động đúng với thỏa thuận lúc tuyển dụng.
    *   Bảng phụ cấp phúc lợi được đồng bộ chi tiết.

### Bước 3: Cấu hình Bảo hiểm & Kích hoạt Hợp đồng
1.  Tại giao diện hợp đồng, kiểm tra các Checkbox bảo hiểm bắt buộc: **Đóng BHXH**, **Đóng BHYT**, **Đóng BHTN**. Tích chọn hoặc bỏ chọn dựa trên đối tượng lao động (ví dụ: Lao động thử việc không tích đóng bảo hiểm).
2.  Kiểm tra trường **Mức lương đóng bảo hiểm (insurance_salary_base)**. Hệ thống tự động cộng dồn ba khoản lương đóng BHXH theo luật.
3.  Sau khi rà soát toàn bộ thông tin, thay đổi trạng thái hợp đồng từ **Draft (Nháp)** sang **Running (Đang chạy)**. Từ thời điểm này, hệ thống sẽ sử dụng thông tin hợp đồng này làm căn cứ tính công và chạy bảng lương hàng tháng cho nhân viên.

---

## 5. CÁC LƯU Ý QUAN TRỌNG & XỬ LÝ SỰ CỐ

### 5.1. Nút "Gửi Email" hoặc "Gửi Zalo" không xuất hiện?
*   *Nguyên nhân:* Do hồ sơ ứng viên bị thiếu thông tin Email hoặc Số điện thoại.
*   *Khắc phục:* HR bổ sung chính xác Email (phục vụ nút Gửi Email) và Số điện thoại di động (phục vụ nút Gửi Zalo) rồi nhấn Lưu. Các nút bấm sẽ tự động hiển thị lại.

### 5.2. Tin nhắn Zalo báo gửi thất bại?
*   *Cách kiểm tra:* Admin truy cập menu **Settings ➔ Technical ➔ Zalo Message Logs** để xem lịch sử lỗi chi tiết.
*   *Lỗi phổ biến:* Mã lỗi của Zalo báo số điện thoại không đăng ký sử dụng Zalo, hoặc tài khoản Zalo của ứng viên chặn nhận tin nhắn từ người lạ. Trường hợp này HR sẽ liên hệ hoặc gửi Email PDF thay thế.

### 5.3. Thay đổi mức giảm trừ thuế TNCN khi Nhà nước đổi luật?
*   *Hướng dẫn:* Truy cập **Settings ➔ Human Resources ➔ Payroll & Contract Configuration**.
*   Thay đổi số tiền tại ô *Giảm trừ bản thân* và *Giảm trừ người phụ thuộc* sang mức mới, nhấn **Save**. Hệ thống sẽ tự động áp dụng mức cấu hình mới này cho toàn bộ bảng lương tính toán kể từ thời điểm đó mà không cần chỉnh sửa code.

---
