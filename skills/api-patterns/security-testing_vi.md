# Kiểm thử Bảo mật API

> Các nguyên tắc để kiểm thử bảo mật API. Top 10 API OWASP, kiểm thử xác thực, phân quyền.

---

## Top 10 Bảo mật API của OWASP

| Lỗ hổng                        | Trọng tâm Kiểm thử                          |
| ------------------------------ | ------------------------------------------- |
| **API1: BOLA**                 | Truy cập tài nguyên của người dùng khác     |
| **API2: Broken Auth**          | JWT, phiên, thông tin đăng nhập             |
| **API3: Property Auth**        | Gán hàng loạt (Mass assignment), lộ dữ liệu |
| **API4: Resource Consumption** | Giới hạn tốc độ, DoS                        |
| **API5: Function Auth**        | Các điểm cuối quản trị, bỏ qua vai trò      |
| **API6: Business Flow**        | Lạm dụng logic, tự động hóa                 |
| **API7: SSRF**                 | Truy cập mạng nội bộ                        |
| **API8: Misconfiguration**     | Các điểm cuối debug, CORS                   |
| **API9: Inventory**            | API ẩn (Shadow APIs), phiên bản cũ          |
| **API10: Unsafe Consumption**  | Tin cậy API bên thứ ba                      |

---

## Kiểm thử Xác thực

### Kiểm thử JWT

| Kiểm tra          | Kiểm thử gì                         |
| ----------------- | ----------------------------------- |
| Thuật toán        | Không, nhầm lẫn thuật toán          |
| Bí mật            | Bí mật yếu, brute force             |
| Tuyên bố (Claims) | Hết hạn, người phát hành, đối tượng |
| Chữ ký            | Thao túng, tiêm nhiễm khóa          |

### Kiểm thử Phiên (Session)

| Kiểm tra    | Kiểm thử gì                      |
| ----------- | -------------------------------- |
| Tạo         | Khả năng dự đoán                 |
| Lưu trữ     | Bảo mật phía máy khách           |
| Hết hạn     | Thực thi thời gian chờ (Timeout) |
| Vô hiệu hóa | Hiệu quả đăng xuất               |

---

## Kiểm thử Phân quyền

| Loại Kiểm thử | Cách tiếp cận                               |
| ------------- | ------------------------------------------- |
| **Ngang**     | Truy cập dữ liệu của người dùng ngang hàng  |
| **Dọc**       | Truy cập các chức năng có đặc quyền cao hơn |
| **Ngữ cảnh**  | Truy cập ngoài phạm vi cho phép             |

### Kiểm thử BOLA/IDOR

1. Xác định ID tài nguyên trong các yêu cầu
2. Bắt yêu cầu với phiên của người dùng A
3. Phát lại với phiên của người dùng B
4. Kiểm tra quyền truy cập trái phép

---

## Kiểm thử Xác thực Đầu vào

| Loại Tiêm nhiễm | Trọng tâm Kiểm thử |
| --------------- | ------------------ |
| SQL             | Thao túng truy vấn |
| NoSQL           | Truy vấn tài liệu  |
| Lệnh (Command)  | Lệnh hệ thống      |
| LDAP            | Truy vấn thư mục   |

**Cách tiếp cận:** Kiểm thử tất cả các tham số, thử ép kiểu, kiểm thử ranh giới, kiểm tra thông báo lỗi.

---

## Kiểm thử Giới hạn Tốc độ

| Khía cạnh  | Kiểm tra                           |
| ---------- | ---------------------------------- |
| Sự tồn tại | Có giới hạn nào không?             |
| Bỏ qua     | Headers, xoay vòng IP              |
| Phạm vi    | Theo người dùng, theo IP, toàn cầu |

**Các kỹ thuật bỏ qua:** X-Forwarded-For, các phương thức HTTP khác nhau, biến thể chữ hoa/thường, đánh phiên bản API.

---

## Bảo mật GraphQL

| Kiểm thử                    | Trọng tâm           |
| --------------------------- | ------------------- |
| Tự kiểm tra (Introspection) | Tiết lộ lược đồ     |
| Hàng loạt (Batching)        | DoS truy vấn        |
| Lồng nhau (Nesting)         | DoS dựa trên độ sâu |
| Phân quyền                  | Truy cập cấp trường |

---

## Danh sách kiểm tra Kiểm thử Bảo mật

**Xác thực:**

- [ ] Kiểm thử việc bỏ qua
- [ ] Kiểm tra độ mạnh thông tin đăng nhập
- [ ] Xác minh bảo mật token

**Phân quyền:**

- [ ] Kiểm thử BOLA/IDOR
- [ ] Kiểm tra leo thang đặc quyền
- [ ] Xác minh quyền truy cập chức năng

**Đầu vào:**

- [ ] Kiểm thử tất cả các tham số
- [ ] Kiểm tra tiêm nhiễm

**Cấu hình:**

- [ ] Kiểm tra CORS
- [ ] Xác minh tiêu đề
- [ ] Kiểm thử xử lý lỗi

---

> **Ghi nhớ:** API là xương sống của các ứng dụng hiện đại. Hãy kiểm thử chúng như cách những kẻ tấn công sẽ làm.
