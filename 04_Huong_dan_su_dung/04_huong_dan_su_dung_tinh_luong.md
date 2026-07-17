# HƯỚNG DẪN SỬ DỤNG
## PHÂN HỆ: TÍNH LƯƠNG LAI TINH GỌN VÀ PHÂN PHỐI PHIẾU LƯƠNG BẢO MẬT (Odoo 19 CE)
**Dự án:** Nâng cấp & Chuẩn hóa Hệ thống Nhân sự Đại Quang
**Đối tượng hướng dẫn:** Bộ phận Hành chính Nhân sự (HR Admin) và Kế toán tiền lương
**Phiên bản tài liệu:** v1.0
**Ngày biên soạn:** 2026-07-05

---

## 1. QUY TRÌNH ĐỒNG BỘ CÔNG & TÍNH LƯƠNG HÀNG THÁNG

Hệ thống tự động liên kết dữ liệu ngày công để tính lương nhanh chóng:

### Các bước thực hiện:
1.  Truy cập menu **Payroll (Lương) ➔ Payslip Batches (Đợt tính lương)**.
2.  Nhấn nút **New (Tạo mới)** để khởi tạo đợt tính lương cho tháng (Ví dụ: *Bảng lương tháng 07/2026*).
3.  Chọn khoảng thời gian tính lương (Từ ngày 01 đến ngày cuối tháng).
4.  Nhấn nút **Generate Payslips (Tạo phiếu lương hàng loạt)**.
    *   *Cơ chế tự động:* Odoo sẽ tự động quét bảng công ngày (`hr.attendance.day`) của từng nhân sự, đếm tổng số ngày công thực tế và giờ tăng ca (OT) trong tháng, đồng bộ dữ liệu vào phiếu lương và áp dụng dưới 15 quy tắc lương cốt lõi để tính toán ra kết quả Gross/Net.
5.  Click vào từng Phiếu lương cá nhân để kiểm tra chéo các dòng tiền.

---

## 2. KẾT XUẤT BẢNG LƯƠNG TỔNG HỢP EXCEL

Trước khi thực hiện chi trả lương, HR/Kế toán cần xuất bảng lương ra file Excel để trình duyệt Ban Giám đốc:

### Các bước thực hiện:
1.  Tại giao diện đợt tính lương đã hoàn thành, nhìn lên thanh Header, nhấn nút **"Xuất Bảng Lương Excel"**.
2.  Hệ thống tự động khởi tạo và tải xuống file Excel bảng lương tổng hợp đã được định dạng chuẩn:
    *   Tiêu đề bảng lương nổi bật ở dòng đầu.
    *   Cột số tiền được căn phải và có dấu ngăn cách hàng nghìn (ví dụ: `12,500,000`).
    *   Cột thông tin tài khoản ngân hàng của nhân viên phục vụ việc chuyển khoản ngân hàng hàng loạt.
    *   Dòng dưới cùng tự động tính tổng (SUM) toàn bộ quỹ lương và các khoản bảo hiểm/thuế.
3.  HR in hoặc đính kèm file Excel này trình Kế toán trưởng và Ban Giám đốc Đại Quang phê duyệt.

---

## 3. PHÂN PHỐI PHIẾU LƯƠNG BẢO MẬT QUA EMAIL (PDF MÃ HÓA)

Khi bảng lương được phê duyệt, HR thực hiện gửi phiếu lương bảo mật cho nhân sự văn phòng qua email:

### Các bước thực hiện:
1.  Tại giao diện đợt tính lương, nhấn nút **"Gửi Email Phiếu Lương Hàng Loạt"**.
2.  Hệ thống tự động chạy ngầm, xuất PDF phiếu lương của từng nhân viên và tiến hành **mã hóa bảo vệ bằng mật khẩu**.
3.  **Hướng dẫn nhân viên cách mở file:**
    *   Khi nhân viên mở email và click vào file PDF đính kèm, điện thoại/máy tính sẽ yêu cầu nhập mật khẩu bảo mật.
    *   Mật khẩu mặc định là **Ngày tháng năm sinh** của chính nhân viên đó (Ví dụ: Nhân viên sinh ngày 12 tháng 08 năm 1990 ➔ Mật khẩu mở file là: `12081990`).
    *   *Mục đích:* Tránh rò rỉ thông tin thu nhập khi nhân viên mở nhầm hòm thư hoặc dùng chung máy tính với người khác.

---

## 4. PHÂN PHỐI PHIẾU LƯƠNG QUA TINH NHẮN ZALO OA (XEM NHANH BẰNG TOKEN)

Đối với công nhân sản xuất tại nhà máy không sử dụng email thường xuyên, HR thực hiện gửi qua Zalo:

### Các bước thực hiện:
1.  Tại giao diện đợt tính lương, nhấn nút **"Gửi Zalo Phiếu Lương Hàng Loạt"**.
2.  Hệ thống Odoo gọi API Zalo OA để gửi tin nhắn ZNS thông báo nhận lương tháng tới số điện thoại đăng ký Zalo của từng nhân viên.
3.  **Cách nhân viên xem phiếu lương:**
    *   Nhân viên mở tin nhắn Zalo của Đại Quang, nhấn vào nút **"Xem Phiếu Lương"**.
    *   Trình duyệt web điện thoại tự động hiển thị trực tiếp file PDF phiếu lương chi tiết mà nhân viên không cần đăng nhập tài khoản Odoo.
    *   *Tính bảo mật:* Đường link xem nhanh này chứa token bảo mật ngẫu nhiên và chỉ có giá trị truy cập trong vòng **48 giờ** kể từ lúc gửi tin nhắn. Sau 48 giờ link sẽ tự động bị khóa để bảo mật thông tin.

---
