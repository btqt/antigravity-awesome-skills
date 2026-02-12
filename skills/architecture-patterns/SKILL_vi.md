---
name: architecture-patterns
description: Triển khai các mẫu kiến trúc backend đã được chứng minh bao gồm Kiến trúc Sạch (Clean Architecture), Kiến trúc Lục giác (Hexagonal Architecture), và Thiết kế Hướng Tên miền (Domain-Driven Design). Sử dụng khi kiến trúc các hệ thống backend phức tạp hoặc tái cấu trúc các ứng dụng hiện có để có khả năng bảo trì tốt hơn.
---

# Các Mẫu Kiến trúc

Làm chủ các mẫu kiến trúc backend đã được chứng minh bao gồm Kiến trúc Sạch, Kiến trúc Lục giác, và Thiết kế Hướng Tên miền để xây dựng các hệ thống dễ bảo trì, dễ kiểm thử, và có khả năng mở rộng.

## Sử dụng kỹ năng này khi

- Thiết kế hệ thống backend mới từ đầu
- Tái cấu trúc các ứng dụng monolith để có khả năng bảo trì tốt hơn
- Thiết lập các tiêu chuẩn kiến trúc cho nhóm của bạn
- Di chuyển từ kiến trúc ghép nối chặt chẽ sang lỏng lẻo
- Triển khai các nguyên tắc thiết kế hướng tên miền
- Tạo các codebase dễ kiểm thử và mockable
- Lập kế hoạch phân tách microservices

## Không sử dụng kỹ năng này khi

- Bạn chỉ cần các bản tái cấu trúc nhỏ, cục bộ
- Hệ thống chủ yếu là frontend mà không có thay đổi kiến trúc backend
- Bạn cần chi tiết triển khai mà không cần thiết kế kiến trúc

## Hướng dẫn

1. Làm rõ ranh giới tên miền, ràng buộc, và mục tiêu mở rộng.
2. Chọn một mẫu kiến trúc phù hợp với độ phức tạp của tên miền.
3. Xác định ranh giới module, giao diện, và quy tắc phụ thuộc.
4. Cung cấp các bước di chuyển và kiểm tra xác thực.

Tham khảo `resources/implementation-playbook.md` để biết các mẫu chi tiết, danh sách kiểm tra, và template.

## Tài nguyên

- `resources/implementation-playbook.md` cho các mẫu chi tiết, danh sách kiểm tra, và template.
