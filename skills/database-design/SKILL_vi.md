---
name: database-design
description: Các nguyên tắc và ra quyết định thiết kế cơ sở dữ liệu. Thiết kế lược đồ, chiến lược lập chỉ mục, lựa chọn ORM, cơ sở dữ liệu serverless.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Thiết Kế Cơ Sở Dữ Liệu (Database Design)

> **Học cách TƯ DUY, không sao chép các mẫu SQL.**

## 🎯 Quy tắc Đọc Chọn lọc

**Chỉ đọc các tệp CÓ LIÊN QUAN đến yêu cầu!** Kiểm tra bản đồ nội dung, tìm những gì bạn cần.

| Tệp                     | Mô tả                                 | Khi nào Đọc          |
| ----------------------- | ------------------------------------- | -------------------- |
| `database-selection.md` | PostgreSQL vs Neon vs Turso vs SQLite | Chọn cơ sở dữ liệu   |
| `orm-selection.md`      | Drizzle vs Prisma vs Kysely           | Chọn ORM             |
| `schema-design.md`      | Chuẩn hóa, PKs, mối quan hệ           | Thiết kế lược đồ     |
| `indexing.md`           | Loại chỉ mục, chỉ mục tổng hợp        | Tinh chỉnh hiệu suất |
| `optimization.md`       | N+1, EXPLAIN ANALYZE                  | Tối ưu hóa truy vấn  |
| `migrations.md`         | Di chuyển an toàn, serverless DBs     | Thay đổi lược đồ     |

---

## ⚠️ Nguyên tắc Cốt lõi

- HỎI người dùng về sở thích cơ sở dữ liệu khi không rõ ràng
- Chọn cơ sở dữ liệu/ORM dựa trên BỐI CẢNH
- Đừng mặc định dùng PostgreSQL cho mọi thứ

---

## Danh sách Kiểm tra Quyết định

Trước khi thiết kế lược đồ:

- [ ] Đã hỏi người dùng về sở thích cơ sở dữ liệu chưa?
- [ ] Đã chọn cơ sở dữ liệu cho bối cảnh NÀY chưa?
- [ ] Đã xem xét môi trường triển khai chưa?
- [ ] Đã lập kế hoạch chiến lược chỉ mục chưa?
- [ ] Đã xác định các loại mối quan hệ chưa?

---

## Các Mẫu Chống (Anti-Patterns)

❌ Mặc định dùng PostgreSQL cho các ứng dụng đơn giản (SQLite có thể đủ)
❌ Bỏ qua lập chỉ mục
❌ Sử dụng SELECT \* trong sản xuất
❌ Lưu trữ JSON khi dữ liệu có cấu trúc tốt hơn
❌ Bỏ qua truy vấn N+1
