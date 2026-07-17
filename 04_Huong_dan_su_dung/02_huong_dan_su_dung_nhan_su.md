# HƯỚNG DẪN SỬ DỤNG
## PHÂN HỆ: QUẢN LÝ NHÂN SỰ VÀ SỨC KHỎE ĐỊNH KỲ (Odoo 19 CE)
**Dự án:** Nâng cấp & Chuẩn hóa Hệ thống Nhân sự Đại Quang
**Đối tượng hướng dẫn:** Bộ phận Hành chính Nhân sự (HR Admin), Cán bộ Y tế và Trưởng bộ phận
**Phiên bản tài liệu:** v1.0
**Ngày biên soạn:** 2026-07-05

---

## 1. THIẾT LẬP PHÒNG BAN & PHÂN QUYỀN XEM DỮ LIỆU (VIEWERS)

Chức năng này giúp phân quyền cho Trưởng bộ phận / Tổ trưởng tổ sản xuất được xem hồ sơ và tình trạng sức khỏe của nhân viên trực thuộc tổ mình quản lý mà không cần cấp quyền truy cập toàn bộ hệ thống nhân sự.

### Các bước thực hiện:
1.  Truy cập menu **Employees (Nhân sự) ➔ Departments (Phòng ban)**.
2.  Click chọn phòng ban/tổ đội cần thiết lập (Ví dụ: *Tổ Hàn - Chi nhánh HN*).
3.  Tại trường **Người xem (Viewers)**, chọn các tài khoản của Trưởng bộ phận/Tổ trưởng tương ứng.
4.  Nhấn **Save (Lưu)**. 
    *   *Kết quả:* Tài khoản được chọn khi đăng nhập sẽ nhìn thấy danh sách nhân sự của phòng ban này trong menu Employees của họ.

---

## 2. QUY TRÌNH CẬP NHẬT KẾT QUẢ KHÁM SỨC KHỎE ĐỊNH KỲ

Khi công ty tổ chức khám sức khỏe định kỳ hoặc nhân viên nộp Giấy khám sức khỏe mới, cán bộ HR thực hiện nhập liệu vào hệ thống:

### Các bước thực hiện:
1.  Truy cập menu **Employees ➔ Employees** và mở hồ sơ nhân sự cần cập nhật.
2.  Kéo xuống phần Notebook, chọn Tab **"Thông tin Y tế & Sức khỏe"**.
3.  Tại bảng **Lịch sử khám sức khỏe**, nhấn **Add a line (Thêm một dòng)** và điền:
    *   **Ngày khám:** Nhập ngày ghi trên giấy khám.
    *   **Cơ sở y tế:** Tên bệnh viện / phòng khám thực hiện.
    *   **Phân loại sức khỏe:** Chọn phân loại từ **Loại I** (Rất khỏe) đến **Loại V** (Rất yếu) theo kết luận của bác sĩ.
    *   **Ghi chú:** Ghi nhận các bệnh lý cụ thể hoặc khuyến cáo của bác sĩ (ví dụ: *Hạn chế làm ca đêm, không bê vác vật nặng trên 20kg*).
4.  Nhấn biểu tượng **Đính kèm (Attachment)** trên dòng đó để tải lên file scan PDF hoặc ảnh chụp Giấy khám sức khỏe.
5.  Nhấn **Save (Lưu)** hồ sơ nhân viên.
    *   *Kết quả:* Hệ thống tự động ghi nhận **Ngày khám gần nhất**, **Phân loại sức khỏe gần nhất** và tự động tính toán lại **Trạng thái y tế** của nhân sự đó.

---

## 3. THEO DÕI HỆ THỐNG CẢNH BÁO Y TẾ TRÊN GIAO DIỆN KANBAN

HR và Trưởng bộ phận truy cập menu **Employees ➔ Employees** chọn chế độ hiển thị **Kanban** (biểu tượng thẻ) để quan sát nhanh các cảnh báo màu sắc tự động:

*   🟢 **Thẻ không có nhãn cảnh báo đỏ/cam:** Nhân viên đã khám sức khỏe đầy đủ (thời gian dưới 6 tháng).
*   🟡 **Nhãn Cam ("Cảnh báo y tế: 6 tháng chưa khám"):** Nhân viên đã quá hạn 6 tháng kể từ đợt khám gần nhất. HR cần lưu ý đưa vào danh sách chuẩn bị khám đợt tiếp theo.
*   🔴 **Nhãn Đỏ ("Quá hạn khám sức khỏe (12T+)"):** Nhân viên đã quá hạn 12 tháng chưa khám hoặc chưa từng cập nhật thông tin y tế trên hệ thống. Đây là trường hợp báo động cần bố trí khám gấp.
*   ⚠️ **Nhãn Đỏ ("Sức khỏe yếu"):** Hiển thị khi phân loại sức khỏe gần nhất của nhân sự là **Loại IV** hoặc **Loại V**. Trưởng bộ phận cần rà soát để chuyển nhân viên sang vị trí công việc nhẹ nhàng hơn, tránh rủi ro tai nạn lao động.

---

## 4. TRUY XUẤT BÁO CÁO & SỬ DỤNG BỘ LỌC NHANH (QUICK FILTERS)

Để lập danh sách nhân sự đi khám sức khỏe đợt mới hoặc xuất báo cáo gửi Ban Giám đốc, HR sử dụng thanh tìm kiếm (Search bar):

### 4.1. Sử dụng Bộ lọc nhanh (Filters)
Click vào nút **Filters (Bộ lọc)** trên thanh tìm kiếm và chọn:
*   **Quá 6 tháng chưa khám:** Hệ thống lọc ra danh sách nhân sự cần lên kế hoạch khám định kỳ.
*   **Quá 12 tháng chưa khám:** Hệ thống lọc ra danh sách vi phạm thời hạn khám y tế.
*   **Chưa từng khám sức khỏe:** Lọc ra nhân sự mới vào làm chưa nộp hồ sơ y tế.

### 4.2. Sử dụng Gom nhóm (Group by)
1.  Click vào nút **Group by (Gom nhóm)** trên thanh tìm kiếm.
2.  Chọn **Trạng thái sức khỏe**.
3.  Hệ thống sẽ gom nhóm toàn bộ nhân sự công ty thành 4 cột trực quan: *Đã khám đầy đủ*, *Quá hạn 6 tháng*, *Quá hạn 12 tháng*, và *Chưa từng khám*.
4.  HR có thể chuyển sang giao diện danh sách (List View) và nhấn nút **Export (Xuất Excel)** ở góc trái để tải file báo cáo tổng hợp.

---
