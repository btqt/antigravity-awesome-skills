---
name: api-patterns
description: Các nguyên tắc thiết kế API và ra quyết định. Chọn lựa REST vs GraphQL vs tRPC, định dạng phản hồi, đánh phiên bản, phân trang.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Các Mẫu API

> Các nguyên tắc thiết kế API và ra quyết định cho năm 2025.
> **Học cách TƯ DUY, không sao chép các mẫu cố định.**

## 🎯 Quy tắc Đọc Chọn lọc

**Chỉ đọc các tệp liên quan đến yêu cầu!** Kiểm tra bản đồ nội dung, tìm những gì bạn cần.

---

## 📑 Bản đồ Nội dung

| Tệp                   | Mô tả                                                    | Khi nào nên đọc             |
| --------------------- | -------------------------------------------------------- | --------------------------- |
| `api-style.md`        | Cây quyết định REST vs GraphQL vs tRPC                   | Chọn loại API               |
| `rest.md`             | Đặt tên tài nguyên, phương thức HTTP, mã trạng thái      | Thiết kế API REST           |
| `response.md`         | Mẫu bao bì (Envelope pattern), định dạng lỗi, phân trang | Cấu trúc phản hồi           |
| `graphql.md`          | Thiết kế lược đồ, khi nào nên sử dụng, bảo mật           | Cân nhắc GraphQL            |
| `trpc.md`             | TypeScript monorepo, an toàn kiểu                        | Dự án TS fullstack          |
| `versioning.md`       | Đánh phiên bản URI/Header/Query                          | Lập kế hoạch phát triển API |
| `auth.md`             | JWT, OAuth, Passkey, API Keys                            | Chọn mẫu xác thực           |
| `rate-limiting.md`    | Token bucket, cửa sổ trượt                               | Bảo vệ API                  |
| `documentation.md`    | Thực hành tốt nhất OpenAPI/Swagger                       | Tài liệu                    |
| `security-testing.md` | OWASP API Top 10, kiểm thử auth/authz                    | Kiểm toán bảo mật           |

---

## 🔗 Các kỹ năng liên quan

| Nhu cầu          | Kỹ năng                         |
| ---------------- | ------------------------------- |
| Triển khai API   | `@[skills/backend-development]` |
| Cấu trúc dữ liệu | `@[skills/database-design]`     |
| Chi tiết bảo mật | `@[skills/security-hardening]`  |

---

## ✅ Danh sách kiểm tra Quyết định

Trước khi thiết kế API:

- [ ] **Đã hỏi người dùng về người tiêu dùng API chưa?**
- [ ] **Đã chọn kiểu API cho ngữ cảnh NÀY chưa?** (REST/GraphQL/tRPC)
- [ ] **Đã xác định định dạng phản hồi nhất quán chưa?**
- [ ] **Đã lập kế hoạch chiến lược đánh phiên bản chưa?**
- [ ] **Đã xem xét nhu cầu xác thực chưa?**
- [ ] **Đã lập kế hoạch giới hạn tốc độ chưa?**
- [ ] **Đã xác định cách tiếp cận tài liệu chưa?**

---

## ❌ Các Phản mẫu (Anti-Patterns)

**KHÔNG:**

- Mặc định chọn REST cho mọi thứ
- Sử dụng động từ trong các điểm cuối REST (/getUsers)
- Trả về định dạng phản hồi không nhất quán
- Để lộ lỗi nội bộ cho khách hàng
- Bỏ qua giới hạn tốc độ

**NÊN:**

- Chọn kiểu API dựa trên ngữ cảnh
- Hỏi về yêu cầu của khách hàng
- Lập tài liệu kỹ lưỡng
- Sử dụng mã trạng thái thích hợp

---

## Kịch bản (Script)

| Script                     | Mục đích               | Lệnh                                             |
| -------------------------- | ---------------------- | ------------------------------------------------ |
| `scripts/api_validator.py` | Xác thực điểm cuối API | `python scripts/api_validator.py <project_path>` |
