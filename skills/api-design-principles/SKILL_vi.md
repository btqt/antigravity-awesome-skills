---
name: api-design-principles
description: Làm chủ các nguyên tắc thiết kế API REST và GraphQL để xây dựng các API trực quan, có khả năng mở rộng và dễ bảo trì, làm hài lòng các nhà phát triển. Sử dụng khi thiết kế các API mới, xem xét các thông số kỹ thuật API hoặc thiết lập các tiêu chuẩn thiết kế API.
---

# Các Nguyên tắc Thiết kế API

Làm chủ các nguyên tắc thiết kế API REST và GraphQL để xây dựng các API trực quan, có khả năng mở rộng và dễ bảo trì, làm hài lòng các nhà phát triển và đứng vững trước thử thách của thời gian.

## Sử dụng kỹ năng này khi

- Thiết kế các API REST hoặc GraphQL mới
- Tái cấu trúc các API hiện có để có khả năng sử dụng tốt hơn
- Thiết lập các tiêu chuẩn thiết kế API cho nhóm của bạn
- Xem xét các thông số kỹ thuật API trước khi triển khai
- Di chuyển giữa các mô hình API (REST sang GraphQL, v.v.)
- Tạo tài liệu API thân thiện với nhà phát triển
- Tối ưu hóa API cho các trường hợp sử dụng cụ thể (di động, tích hợp bên thứ ba)

## Không sử dụng kỹ năng này khi

- Bạn chỉ cần hướng dẫn triển khai cho một framework cụ thể
- Bạn đang làm công việc chỉ liên quan đến cơ sở hạ tầng mà không có hợp đồng API
- Bạn không thể thay đổi hoặc tạo phiên bản cho các giao diện công khai

## Hướng dẫn

1. Xác định người tiêu dùng, trường hợp sử dụng và các ràng buộc.
2. Chọn kiểu API và mô hình tài nguyên hoặc loại.
3. Chỉ rõ lỗi, phiên bản, phân trang và chiến lược xác thực.
4. Xác thực bằng các ví dụ và xem xét tính nhất quán.

Tham khảo `resources/implementation-playbook_vi.md` để biết các chi tiết mẫu, danh sách kiểm tra và mẫu.

## Tài nguyên

- `resources/implementation-playbook_vi.md` để biết các chi tiết mẫu, danh sách kiểm tra và mẫu.
