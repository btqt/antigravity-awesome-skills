---
name: api-documentation-generator
description: "Tạo tài liệu API toàn diện, thân thiện với nhà phát triển từ mã, bao gồm các điểm cuối, tham số, ví dụ và các thực hành tốt nhất"
---

# Trình tạo Tài liệu API

## Tổng quan

Tự động tạo tài liệu API rõ ràng, toàn diện từ mã nguồn của bạn. Kỹ năng này giúp bạn tạo tài liệu chuyên nghiệp bao gồm mô tả điểm cuối, ví dụ yêu cầu/phản hồi, chi tiết xác thực, xử lý lỗi và hướng dẫn sử dụng.

Hoàn hảo cho API REST, API GraphQL và API WebSocket.

## Khi nào nên sử dụng kỹ năng này

- Sử dụng khi bạn cần lập tài liệu cho một API mới
- Sử dụng khi cập nhật tài liệu API hiện có
- Sử dụng khi API của bạn thiếu tài liệu rõ ràng
- Sử dụng khi giới thiệu các nhà phát triển mới vào API của bạn
- Sử dụng khi chuẩn bị tài liệu API cho người dùng bên ngoài
- Sử dụng khi tạo thông số kỹ thuật OpenAPI/Swagger

## Cách nó hoạt động

### Bước 1: Phân tích Cấu trúc API

Đầu tiên, tôi sẽ kiểm tra mã nguồn API của bạn để hiểu:

- Các điểm cuối và tuyến đường có sẵn
- Các phương thức HTTP (GET, POST, PUT, DELETE, v.v.)
- Tham số yêu cầu và cấu trúc thân (body)
- Định dạng phản hồi và mã trạng thái
- Yêu cầu xác thực và ủy quyền
- Các mẫu xử lý lỗi

### Bước 2: Tạo Tài liệu Điểm cuối

Đối với mỗi điểm cuối, tôi sẽ tạo tài liệu bao gồm:

**Chi tiết Điểm cuối:**

- Phương thức HTTP và đường dẫn URL
- Mô tả ngắn gọn về những gì nó làm
- Yêu cầu xác thực
- Thông tin giới hạn tốc độ (nếu có)

**Đặc tả Yêu cầu:**

- Tham số đường dẫn
- Tham số truy vấn
- Tiêu đề yêu cầu
- Lược đồ thân yêu cầu (với các kiểu và quy tắc xác thực)

**Đặc tả Phản hồi:**

- Phản hồi thành công (mã trạng thái + cấu trúc thân)
- Phản hồi lỗi (tất cả các mã lỗi có thể xảy ra)
- Tiêu đề phản hồi

**Ví dụ Mã:**

- Lệnh cURL
- JavaScript/TypeScript (fetch/axios)
- Python (requests)
- Các ngôn ngữ khác khi cần thiết

### Bước 3: Thêm Hướng dẫn Sử dụng

Tôi sẽ bao gồm:

- Hướng dẫn bắt đầu
- Thiết lập xác thực
- Các trường hợp sử dụng phổ biến
- Các thực hành tốt nhất
- Chi tiết giới hạn tốc độ
- Các mẫu phân trang
- Các tùy chọn lọc và sắp xếp

### Bước 4: Lập Tài liệu Xử lý Lỗi

Tài liệu lỗi rõ ràng bao gồm:

- Tất cả các mã lỗi có thể xảy ra
- Định dạng thông báo lỗi
- Hướng dẫn khắc phục sự cố
- Các kịch bản lỗi phổ biến và giải pháp

### Bước 5: Tạo Các Ví dụ Tương tác

Nếu có thể, tôi sẽ cung cấp:

- Bộ sưu tập Postman
- Đặc tả OpenAPI/Swagger
- Các ví dụ mã tương tác
- Phản hồi mẫu

## Các ví dụ

### Ví dụ 1: Tài liệu Điểm cuối API REST

```markdown
## Tạo Người dùng

Tạo một tài khoản người dùng mới.

**Điểm cuối:** `POST /api/v1/users`

**Xác thực:** Bắt buộc (Bearer token)

**Thân Yêu cầu:**
\`\`\`json
{
"email": "user@example.com", // Bắt buộc: Địa chỉ email hợp lệ
"password": "SecurePass123!", // Bắt buộc: Tối thiểu 8 ký tự, 1 chữ hoa, 1 số
"name": "John Doe", // Bắt buộc: 2-50 ký tự
"role": "user" // Tùy chọn: "user" hoặc "admin" (mặc định: "user")
}
\`\`\`

**Phản hồi Thành công (201 Created):**
\`\`\`json
{
"id": "usr_1234567890",
"email": "user@example.com",
"name": "John Doe",
"role": "user",
"createdAt": "2026-01-20T10:30:00Z",
"emailVerified": false
}
\`\`\`

**Phản hồi Lỗi:**

- `400 Bad Request` - Dữ liệu đầu vào không hợp lệ
  \`\`\`json
  {
  "error": "VALIDATION_ERROR",
  "message": "Invalid email format",
  "field": "email"
  }
  \`\`\`

- `409 Conflict` - Email đã tồn tại
  \`\`\`json
  {
  "error": "EMAIL_EXISTS",
  "message": "An account with this email already exists"
  }
  \`\`\`

- `401 Unauthorized` - Thiếu token xác thực hoặc không hợp lệ

**Ví dụ Yêu cầu (cURL):**
\`\`\`bash
curl -X POST https://api.example.com/api/v1/users \
 -H "Authorization: Bearer YOUR_TOKEN" \
 -H "Content-Type: application/json" \
 -d '{
"email": "user@example.com",
"password": "SecurePass123!",
"name": "John Doe"
}'
\`\`\`
```

### Ví dụ 2: Tài liệu API GraphQL

```markdown
## Truy vấn Người dùng

Lấy thông tin người dùng theo ID.

**Truy vấn:**
\`\`\`graphql
query GetUser($id: ID!) {
user(id: $id) {
id
email
name
role
createdAt
posts {
id
title
publishedAt
}
}
}
\`\`\`

**Biến:**
\`\`\`json
{
"id": "usr_1234567890"
}
\`\`\`

**Phản hồi:**
\`\`\`json
{
"data": {
"user": {
"id": "usr_1234567890",
"email": "user@example.com",
"name": "John Doe",
"role": "user",
"createdAt": "2026-01-20T10:30:00Z",
"posts": [
{
"id": "post_123",
"title": "My First Post",
"publishedAt": "2026-01-21T14:00:00Z"
}
]
}
}
}
\`\`\`
```

### Ví dụ 3: Tài liệu Xác thực

```markdown
## Xác thực

Tất cả các yêu cầu API đều yêu cầu xác thực bằng Bearer tokens.

### Lấy Token

**Điểm cuối:** `POST /api/v1/auth/login`

**Yêu cầu:**
\`\`\`json
{
"email": "user@example.com",
"password": "your-password"
}
\`\`\`

**Phản hồi:**
\`\`\`json
{
"token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
"expiresIn": 3600,
"refreshToken": "refresh_token_here"
}
\`\`\`
```

## Các Thực hành Tốt nhất

### ✅ Nên làm

- **Nhất quán** - Sử dụng định dạng giống nhau cho tất cả các điểm cuối
- **Bao gồm Ví dụ** - Cung cấp các ví dụ mã hoạt động bằng nhiều ngôn ngữ
- **Lập tài liệu Lỗi** - Liệt kê tất cả các mã lỗi có thể có và ý nghĩa của chúng
- **Hiển thị Dữ liệu Thực** - Sử dụng dữ liệu ví dụ thực tế, không phải "foo" và "bar"
- **Giải thích Tham số** - Mô tả chức năng của từng tham số và các ràng buộc của nó
- **Đánh phiên bản API của bạn** - Bao gồm số phiên bản trong URL (/api/v1/)
- **Thêm Dấu thời gian** - Hiển thị thời điểm tài liệu được cập nhật lần cuối
- **Liên kết các Điểm cuối Liên quan** - Giúp người dùng khám phá chức năng liên quan
- **Bao gồm Giới hạn Tốc độ** - Lập tài liệu bất kỳ chính sách giới hạn tốc độ nào
- **Cung cấp Bộ sưu tập Postman** - Giúp kiểm tra API của bạn dễ dàng

### ❌ Không nên làm

- **Đừng Bỏ qua Các Trường hợp Lỗi** - Người dùng cần biết những gì có thể sai sót
- **Đừng Sử dụng Mô tả Mơ hồ** - "Lấy dữ liệu" không hữu ích
- **Đừng Quên Xác thực** - Luôn lập tài liệu các yêu cầu xác thực
- **Đừng Bỏ qua Các Trường hợp Biên** - Lập tài liệu phân trang, lọc, sắp xếp
- **Đừng Để Các Ví dụ Bị Hỏng** - Kiểm tra tất cả các ví dụ mã
- **Đừng Sử dụng Thông tin Lỗi thời** - Giữ tài liệu đồng bộ với mã
- **Đừng Phức tạp hóa** - Giữ cho nó đơn giản và dễ quét
- **Đừng Quên Tiêu đề Phản hồi** - Lập tài liệu các tiêu đề quan trọng

## Cấu trúc Tài liệu

### Các Phần Được Khuyến nghị

1. **Giới thiệu**
   - API làm gì
   - URL cơ sở
   - Phiên bản API
   - Liên hệ hỗ trợ

2. **Xác thực**
   - Cách xác thực
   - Quản lý token
   - Các thực hành tốt nhất về bảo mật

3. **Bắt đầu Nhanh**
   - Ví dụ đơn giản để bắt đầu
   - Hướng dẫn trường hợp sử dụng phổ biến

4. **Các điểm cuối**
   - Được tổ chức theo tài nguyên
   - Chi tiết đầy đủ cho mỗi điểm cuối

5. **Mô hình Dữ liệu**
   - Định nghĩa lược đồ
   - Mô tả trường
   - Quy tắc xác thực

6. **Xử lý Lỗi**
   - Tham chiếu mã lỗi
   - Định dạng phản hồi lỗi
   - Hướng dẫn khắc phục sự cố

7. **Giới hạn Tốc độ**
   - Giới hạn và hạn ngạch
   - Các tiêu đề cần kiểm tra
   - Xử lý lỗi giới hạn tốc độ

8. **Nhật ký Thay đổi**
   - Lịch sử phiên bản API
   - Các thay đổi phá vỡ
   - Thông báo ngừng hỗ trợ

9. **SDKs và Công cụ**
   - Thư viện máy khách chính thức
   - Bộ sưu tập Postman
   - Đặc tả OpenAPI

## Các Cạm bẫy Phổ biến

### Vấn đề: Tài liệu Không đồng bộ

**Triệu chứng:** Các ví dụ không hoạt động, tham số sai, điểm cuối trả về dữ liệu khác
**Giải pháp:**

- Tạo tài liệu từ các bình luận/chú thích mã
- Sử dụng các công cụ như Swagger/OpenAPI
- Thêm các kiểm tra API để xác thực tài liệu
- Xem xét tài liệu với mỗi thay đổi API

### Vấn đề: Thiếu Tài liệu Lỗi

**Triệu chứng:** Người dùng không biết cách xử lý lỗi, vé hỗ trợ tăng
**Giải pháp:**

- Lập tài liệu mọi mã lỗi có thể xảy ra
- Cung cấp thông báo lỗi rõ ràng
- Bao gồm các bước khắc phục sự cố
- Hiển thị phản hồi lỗi ví dụ

### Vấn đề: Các Ví dụ Không hoạt động

**Triệu chứng:** Người dùng không thể bắt đầu, sự thất vọng tăng lên
**Giải pháp:**

- Kiểm tra mọi ví dụ mã
- Sử dụng các điểm cuối thực, hoạt động
- Bao gồm các ví dụ hoàn chỉnh (không phải các đoạn trích)
- Cung cấp môi trường sandbox

## Công cụ và Định dạng

### OpenAPI/Swagger

Tạo tài liệu tương tác:

```yaml
openapi: 3.0.0
info:
  title: My API
  version: 1.0.0
paths:
  /users:
    post:
      summary: Create a new user
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/CreateUserRequest"
```

### Bộ sưu tập Postman

Xuất bộ sưu tập để kiểm tra dễ dàng:

```json
{
  "info": {
    "name": "My API",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "Create User",
      "request": {
        "method": "POST",
        "url": "{{baseUrl}}/api/v1/users"
      }
    }
  ]
}
```

## Các kỹ năng liên quan

- `@doc-coauthoring` - Để viết tài liệu cộng tác
- `@copywriting` - Để có mô tả rõ ràng, thân thiện với người dùng
- `@test-driven-development` - Để đảm bảo hành vi API khớp với tài liệu
- `@systematic-debugging` - Để khắc phục sự cố API

## Tài nguyên Bổ sung

- [OpenAPI Specification](https://swagger.io/specification/)
- [REST API Best Practices](https://restfulapi.net/)
- [GraphQL Documentation](https://graphql.org/learn/)
- [API Design Patterns](https://www.apiguide.com/)
- [Postman Documentation](https://learning.postman.com/docs/)

---

**Mẹo Chuyên nghiệp:** Giữ tài liệu API của bạn càng gần mã càng tốt. Sử dụng các công cụ tạo tài liệu từ các bình luận mã để đảm bảo chúng luôn đồng bộ!
