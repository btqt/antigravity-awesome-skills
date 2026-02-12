---
name: debugging-strategies
description: Làm chủ các kỹ thuật gỡ lỗi có hệ thống, công cụ lập hồ sơ và phân tích nguyên nhân gốc rễ để theo dõi lỗi hiệu quả trên bất kỳ cơ sở mã hoặc ngăn xếp công nghệ nào. Sử dụng khi điều tra lỗi, vấn đề hiệu suất hoặc hành vi không mong muốn.
---

# Các Chiến Lược Gỡ Lỗi (Debugging Strategies)

Biến việc gỡ lỗi từ phỏng đoán bực bội thành giải quyết vấn đề có hệ thống với các chiến lược đã được chứng minh, công cụ mạnh mẽ và phương pháp luận bài bản.

## Sử dụng kỹ năng này khi

- Theo dõi các lỗi khó nắm bắt
- Điều tra các vấn đề hiệu suất
- Gỡ lỗi các sự cố sản xuất
- Phân tích crash dumps hoặc stack traces
- Gỡ lỗi các hệ thống phân tán

## Không sử dụng kỹ năng này khi

- Không có vấn đề tái hiện được hoặc triệu chứng quan sát được
- Nhiệm vụ hoàn toàn là phát triển tính năng
- Bạn không thể truy cập nhật ký, dấu vết hoặc tín hiệu thời gian chạy

## Hướng dẫn

- Tái hiện vấn đề và thu thập nhật ký, dấu vết và chi tiết môi trường.
- Hình thành giả thuyết và thiết kế các thí nghiệm có kiểm soát.
- Thu hẹp phạm vi với tìm kiếm nhị phân và thiết bị đo (instrumentation) có mục tiêu.
- Tài liệu hóa các phát hiện và xác minh bản sửa lỗi.
- Nếu cần các playbook chi tiết, hãy mở `resources/implementation-playbook.md`.

## Tài nguyên

- `resources/implementation-playbook.md` cho các mẫu gỡ lỗi chi tiết và danh sách kiểm tra.
