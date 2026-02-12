---
name: clerk-auth
description: "Các mẫu chuyên gia cho triển khai xác thực Clerk, middleware, tổ chức, webhook và đồng bộ hóa người dùng. Sử dụng khi: thêm xác thực, xác thực clerk, xác thực người dùng, đăng nhập, đăng ký."
source: vibeship-spawner-skills (Apache 2.0)
---

# Xác thực Clerk

## Các Mẫu

### Thiết lập Next.js App Router

Thiết lập Clerk hoàn chỉnh cho Next.js 14/15 App Router.

Bao gồm ClerkProvider, các biến môi trường, và các thành phần đăng nhập/đăng ký cơ bản.

Các thành phần chính:

- ClerkProvider: Bao bọc ứng dụng cho bối cảnh xác thực
- <SignIn />, <SignUp />: Các biểu mẫu xác thực được tạo sẵn
- <UserButton />: Menu người dùng với quản lý phiên

### Bảo vệ Tuyến đường Middleware

Bảo vệ các tuyến đường sử dụng clerkMiddleware và createRouteMatcher.

Thực tiễn tốt nhất:

- Tệp middleware.ts duy nhất tại thư mục gốc dự án
- Sử dụng createRouteMatcher cho các nhóm tuyến đường
- auth.protect() cho bảo vệ rõ ràng
- Tập trung tất cả logic xác thực trong middleware

### Xác thực Server Component

Truy cập trạng thái xác thực trong Server Components sử dụng auth() và currentUser().

Các hàm chính:

- auth(): Trả về userId, sessionId, orgId, claims
- currentUser(): Trả về đối tượng User đầy đủ
- Cả hai đều yêu cầu clerkMiddleware phải được cấu hình

## ⚠️ Các Cạnh Sắc (Sharp Edges)

| Vấn đề | Mức độ nghiêm trọng | Giải pháp    |
| ------ | ------------------- | ------------ |
| Vấn đề | nghiêm trọng        | Xem tài liệu |
| Vấn đề | cao                 | Xem tài liệu |
| Vấn đề | cao                 | Xem tài liệu |
| Vấn đề | cao                 | Xem tài liệu |
| Vấn đề | trung bình          | Xem tài liệu |
| Vấn đề | trung bình          | Xem tài liệu |
| Vấn đề | trung bình          | Xem tài liệu |
| Vấn đề | trung bình          | Xem tài liệu |
