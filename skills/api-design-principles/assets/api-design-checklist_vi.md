# Danh sách Kiểm tra Thiết kế API (API Design Checklist)

## Đánh giá Trước khi Triển khai

### Thiết kế Tài nguyên (Resource Design)

- [ ] Tài nguyên là danh từ, không phải động từ
- [ ] Tên số nhiều cho các tập hợp (collections)
- [ ] Đặt tên nhất quán trên tất cả các endpoint
- [ ] Phân cấp tài nguyên rõ ràng (tránh lồng nhau sâu > 2 cấp)
- [ ] Tất cả các hoạt động CRUD được ánh xạ đúng vào các phương thức HTTP

### Các Phương thức HTTP

- [ ] GET để truy xuất (an toàn, idempotent)
- [ ] POST để tạo mới
- [ ] PUT để thay thế toàn bộ (idempotent)
- [ ] PATCH để cập nhật một phần
- [ ] DELETE để xóa (idempotent)

### Mã trạng thái (Status Codes)

- [ ] 200 OK cho GET/PATCH/PUT thành công
- [ ] 201 Created cho POST
- [ ] 204 No Content cho DELETE
- [ ] 400 Bad Request cho các yêu cầu sai định dạng
- [ ] 401 Unauthorized khi thiếu xác thực
- [ ] 403 Forbidden khi không đủ quyền hạn
- [ ] 404 Not Found khi không tìm thấy tài nguyên
- [ ] 422 Unprocessable Entity cho các lỗi xác thực
- [ ] 429 Too Many Requests cho việc giới hạn tốc độ (rate limiting)
- [ ] 500 Internal Server Error cho lỗi máy chủ

### Phân trang (Pagination)

- [ ] Tất cả các endpoint tập hợp đều được phân trang
- [ ] Quy định kích thước trang mặc định (ví dụ: 20)
- [ ] Quy định kích thước trang tối đa (ví dụ: 100)
- [ ] Bao gồm siêu dữ liệu phân trang (tổng số, số trang, v.v.)
- [ ] Chọn mẫu phân trang dựa trên con trỏ (cursor-based) hoặc offset-based

### Lọc & Sắp xếp (Filtering & Sorting)

- [ ] Sử dụng tham số truy vấn (query parameters) để lọc
- [ ] Hỗ trợ tham số sắp xếp (sort)
- [ ] Tham số tìm kiếm (search) để tìm kiếm toàn văn
- [ ] Hỗ trợ lựa chọn trường (sparse fieldsets)

### Quản lý Phiên bản (Versioning)

- [ ] Xác định chiến lược phiên bản (URL/header/query)
- [ ] Bao gồm phiên bản trong tất cả các endpoint
- [ ] Tài liệu hóa chính sách ngừng hỗ trợ (deprecation policy)

### Xử lý Lỗi

- [ ] Định dạng phản hồi lỗi nhất quán
- [ ] Thông báo lỗi chi tiết
- [ ] Lỗi xác thực ở cấp độ trường
- [ ] Mã lỗi để client xử lý
- [ ] Dấu thời gian (timestamps) trong phản hồi lỗi

### Xác thực & Phân quyền (Authentication & Authorization)

- [ ] Xác định phương pháp xác thực (Bearer token, API key)
- [ ] Kiểm tra phân quyền trên tất cả các endpoint
- [ ] Sử dụng đúng 401 so với 403
- [ ] Xử lý việc hết hạn token

### Giới hạn Tốc độ (Rate Limiting)

- [ ] Xác định giới hạn tốc độ cho mỗi endpoint/người dùng
- [ ] Bao gồm các header giới hạn tốc độ
- [ ] Mã trạng thái 429 khi vượt quá giới hạn
- [ ] Cung cấp header Retry-After

### Tài liệu

- [ ] Tạo đặc tả OpenAPI/Swagger
- [ ] Tài liệu hóa tất cả các endpoint
- [ ] Cung cấp ví dụ yêu cầu/phản hồi
- [ ] Tài liệu hóa các phản hồi lỗi
- [ ] Tài liệu hóa luồng xác thực

### Kiểm thử

- [ ] Kiểm thử đơn vị (unit tests) cho logic nghiệp vụ
- [ ] Kiểm thử tích hợp (integration tests) cho các endpoint
- [ ] Kiểm thử các tình huống lỗi
- [ ] Bao phủ các trường hợp biên (edge cases)
- [ ] Kiểm thử hiệu suất cho các endpoint nặng

### Bảo mật

- [ ] Xác thực đầu vào trên tất cả các trường
- [ ] Ngăn chặn SQL injection
- [ ] Ngăn chặn XSS
- [ ] Cấu hình CORS chính xác
- [ ] Bắt buộc sử dụng HTTPS
- [ ] Dữ liệu nhạy cảm không nằm trong URL
- [ ] Không để lộ thông tin bí mật (secrets) trong phản hồi

### Hiệu suất

- [ ] Tối ưu hóa truy vấn cơ sở dữ liệu
- [ ] Ngăn chặn truy vấn N+1
- [ ] Xác định chiến lược lưu bộ nhớ đệm (caching)
- [ ] Thiết lập các header cache phù hợp
- [ ] Phân trang các phản hồi lớn

### Giám sát

- [ ] Triển khai ghi nhật ký (logging)
- [ ] Cấu hình theo dõi lỗi
- [ ] Thu thập các chỉ số hiệu suất
- [ ] Có sẵn endpoint kiểm tra sức khỏe (health check)
- [ ] Cấu hình cảnh báo cho các lỗi

## Các Kiểm tra Đặc thù cho GraphQL

### Thiết kế Lược đồ (Schema Design)

- [ ] Sử dụng phương pháp tiếp cận Schema-first
- [ ] Các kiểu dữ liệu được định nghĩa đúng
- [ ] Quyết định trường nào non-null so với nullable
- [ ] Sử dụng Interfaces/unions một cách phù hợp
- [ ] Định nghĩa các custom scalars

### Truy vấn (Queries)

- [ ] Giới hạn độ sâu của truy vấn (query depth limiting)
- [ ] Phân tích độ phức tạp của truy vấn
- [ ] Sử dụng DataLoaders để ngăn chặn N+1
- [ ] Chọn mẫu phân trang (Relay/offset)

### Mutations

- [ ] Định nghĩa các kiểu đầu vào (input types)
- [ ] Các kiểu payload kèm theo lỗi
- [ ] Hỗ trợ phản hồi lạc quan (optimistic response)
- [ ] Cân nhắc tính Idempotency

### Hiệu suất

- [ ] Sử dụng DataLoader cho tất cả các mối quan hệ
- [ ] Kích hoạt gom nhóm truy vấn (query batching)
- [ ] Cân nhắc Persisted queries
- [ ] Triển khai phản hồi bộ nhớ đệm (response caching)

### Tài liệu

- [ ] Tài liệu hóa tất cả các trường
- [ ] Đánh dấu các trường ngừng hỗ trợ (deprecations)
- [ ] Cung cấp ví dụ
- [ ] Kích hoạt tính năng Schema introspection
