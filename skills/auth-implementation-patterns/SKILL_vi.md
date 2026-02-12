---
name: auth-implementation-patterns
description: Làm chủ các mẫu xác thực (authentication) và ủy quyền (authorization) bao gồm JWT, OAuth2, quản lý phiên (session management), và RBAC để xây dựng các hệ thống kiểm soát truy cập an toàn, có thể mở rộng. Sử dụng khi triển khai các hệ thống xác thực, bảo mật API, hoặc debug các vấn đề bảo mật.
---

# Các mẫu Triển khai Xác thực & Ủy quyền

Xây dựng các hệ thống xác thực và ủy quyền an toàn, có thể mở rộng bằng cách sử dụng các mẫu tiêu chuẩn ngành và các thực hành tốt nhất hiện đại.

## Sử dụng kỹ năng này khi

- Triển khai các hệ thống xác thực người dùng
- Bảo mật các API REST hoặc GraphQL
- Thêm đăng nhập OAuth2/mạng xã hội hoặc SSO (đăng nhập một lần)
- Thiết kế quản lý phiên hoặc RBAC (kiểm soát truy cập dựa trên vai trò)
- Debug các vấn đề về xác thực hoặc ủy quyền

## Không sử dụng kỹ năng này khi

- Bạn chỉ cần nội dung giao diện (UI copy) hoặc định dạng trang đăng nhập
- Nhiệm vụ chỉ liên quan đến cơ sở hạ tầng mà không quan tâm đến định danh
- Bạn không thể thay đổi các chính sách xác thực hoặc lưu trữ thông tin đăng nhập

## Hướng dẫn

- Định nghĩa người dùng, người thuê (tenants), các luồng (flows) và các ràng buộc của mô hình đe dọa.
- Chọn chiến lược xác thực (phiên, JWT, OIDC) và vòng đời của token.
- Thiết kế mô hình ủy quyền và các điểm thực thi chính sách.
- Lập kế hoạch lưu trữ bí mật (secrets storage), xoay vòng (rotation), ghi nhật ký (logging) và các yêu cầu kiểm tra (audit).
- Nếu cần các ví dụ chi tiết, hãy mở `resources/implementation-playbook.md`.

## An toàn

- Không bao giờ ghi nhật ký (log) các bí mật, token hoặc thông tin đăng nhập.
- Thực thi quyền tối thiểu (least privilege) và lưu trữ an toàn cho các khóa (keys).

## Tài nguyên

- `resources/implementation-playbook.md` để biết các mẫu và ví dụ chi tiết.
