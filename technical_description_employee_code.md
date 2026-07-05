# Tài liệu kỹ thuật: Quản lý Chi nhánh, Phòng ban & Mã nhân viên ghép (Odoo 19 CE)

Tài liệu này mô tả chi tiết thiết kế kỹ thuật, cấu trúc cơ sở dữ liệu và các luồng xử lý nghiệp vụ cho tính năng **Mã nhân viên ghép** và **Cấu trúc phòng ban theo chi nhánh** tại DaiQuang Tech trên nền tảng Odoo 19 CE.

---

## 1. Cấu trúc Mô hình Dữ liệu (Database Schema)

### A. Chi nhánh (`hr.branch`)
Đây là một model hoàn toàn mới nhằm phân nhóm các phòng ban và nhân viên theo địa bàn hoạt động.

| Tên trường | Kiểu dữ liệu | Nhãn (Label) | Mô tả / Ràng buộc |
| :--- | :--- | :--- | :--- |
| `name` | `Char` | Tên chi nhánh | Bắt buộc nhập |
| `address` | `Char` | Địa chỉ | Mặc định: "Hà Nội" |
| `branch_counter` | `Integer` | Số thứ tự chi nhánh | Tính toán tự động theo công ty (`store=True`) |
| `branch_code` | `Char` | Mã chi nhánh | Định dạng 2 chữ số (ví dụ: `01`, `02`) từ `branch_counter` |
| `branch_alias` | `Char` | Tên viết tắt | Dùng làm định danh |
| `company_id` | `Many2one` | Công ty | Liên kết với `res.company` |

### B. Công ty (`res.company`)
Kế thừa để bổ sung mã định danh phục vụ sinh mã nhân viên.
*   `company_counter` (`Integer`): Số thứ tự công ty (default: `0`).
*   `company_code` (`Char`): Mã công ty dạng chuỗi (computed từ `company_counter`).
*   `company_alias` (`Char`): Tên viết tắt của công ty.

### C. Phòng ban (`hr.department`)
Kế thừa để gắn kết phòng ban vào chi nhánh và phục vụ mã ghép.
*   `branch_id` (`Many2one`): Liên kết tới `hr.branch`.
*   `dept_counter` (`Integer`): Số thứ tự phòng ban tự tăng trong Chi nhánh (`store=True`).
*   `dept_code` (`Char`): Mã phòng ban (computed từ `dept_counter`, e.g. `1`, `2`, `3`).
*   `dept_alias` (`Char`): Tên viết tắt phòng ban.
*   `seal_image` (`Binary`): Con dấu của phòng ban (dùng cho in ấn báo cáo).

### D. Nhân viên (`hr.employee`)
Kế thừa để triển khai cơ chế sinh mã nhân viên ghép 4 thành phần.
*   `branch_id` (`Many2one`): Liên kết tới `hr.branch` (Required).
*   `employee_counter` (`Integer`): Số thứ tự nhân viên tự tăng trong Công ty.
*   `company_code` (`Char`): Mã công ty (`store=True`, computed/inverse).
*   `branch_code` (`Char`): Mã chi nhánh (`store=True`, computed/inverse).
*   `dept_code` (`Char`): Mã phòng ban (`store=True`, computed/inverse).
*   `counter_code` (`Char`): Số thứ tự nhân viên định dạng 4 chữ số (`store=True`, computed/inverse, ví dụ: `0001`).
*   `combine_name` (`Char`): Tên hiển thị kết hợp `name` và `employee_full_name`.

---

## 2. Luật tạo Mã nhân viên ghép (Hierarchical Employee Code Rule)

Mã nhân viên hiển thị trên giao diện là sự kết hợp của 4 ô trường nhập liệu song song:
```
[Company Code] - [Branch Code] - [Dept Code] - [Counter Code]
```
Ví dụ: `1 01 2 0015` có nghĩa là:
*   `1`: Công ty có mã số 1.
*   `01`: Chi nhánh thứ nhất (ví dụ: Văn phòng Hà Nội).
*   `2`: Phòng ban thứ hai (ví dụ: Phòng Kế toán).
*   `0015`: Nhân viên số thứ tự 15 được tạo trong công ty.

### Cơ chế tính toán tự động (Compute & Inverse):
1.  **Tính toán (`@api.depends`):**
    *   `company_code` được lấy từ `company_id.company_code`.
    *   `branch_code` được lấy từ `branch_id.branch_code`.
    *   `dept_code` được lấy từ `department_id.dept_code`.
    *   `counter_code` tự động điền các số 0 ở đầu dựa trên số nguyên `employee_counter` để đủ 4 chữ số.
2.  **Cơ chế nhập tay đồng bộ ngược (`inverse`):**
    *   Nếu người dùng trực tiếp sửa `company_code`, Odoo sẽ tìm kiếm công ty có mã tương ứng và gán vào `company_id`.
    *   Nếu sửa `branch_code`, Odoo tìm chi nhánh phù hợp để gán vào `branch_id`.
    *   Nếu sửa `dept_code`, Odoo tìm phòng ban phù hợp trong chi nhánh để gán vào `department_id`.
    *   Nếu sửa `counter_code`, Odoo cập nhật lại `employee_counter`.
3.  **Hàm cảnh báo và ràng buộc (`@api.onchange`):**
    *   Kiểm tra sự tồn tại của mã công ty, chi nhánh, phòng ban. Nếu không tồn tại, hiển thị hộp thoại Cảnh báo (Warning pop-up).
    *   Kiểm tra trùng lặp số thứ tự: Nếu gán `counter_code` đã tồn tại cho nhân viên khác trong cùng công ty, cảnh báo trùng lặp sẽ hiển thị để ngăn chặn sai sót dữ liệu.

---

## 3. Quy tắc tìm kiếm và hiển thị nhân viên

*   **Tên hiển thị (`display_name`):** 
    Ghi đè hàm `_compute_display_name()` trong Odoo 19 để trả về định dạng: 
    `"Tên viết tắt (Họ tên đầy đủ)"` (Ví dụ: `Nguyen Van A (Nguyễn Văn An)`).
*   **Tìm kiếm nhanh (`_name_search`):**
    Cho phép tìm kiếm nhân viên theo cả tên thường, họ tên đầy đủ (`employee_full_name`), và mã nhân viên ngẫu nhiên (`employees_codes`).
