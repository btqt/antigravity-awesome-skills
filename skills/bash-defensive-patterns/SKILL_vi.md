---
name: bash-defensive-patterns
description: Nắm vững các kỹ thuật lập trình Bash phòng thủ cho các script cấp production. Sử dụng khi viết các shell script mạnh mẽ, đường ống CI/CD, hoặc các tiện ích hệ thống yêu cầu khả năng chịu lỗi và an toàn.
---

# Các Mẫu Bash Phòng thủ (Bash Defensive Patterns)

Hướng dẫn toàn diện để viết các Bash script sẵn sàng cho production sử dụng các kỹ thuật lập trình phòng thủ, xử lý lỗi, và các thực hành tốt nhất về an toàn để ngăn chặn các cạm bẫy phổ biến và đảm bảo độ tin cậy.

## Sử dụng kỹ năng này khi

- Viết các script tự động hóa production
- Xây dựng các script đường ống CI/CD
- Tạo các tiện ích quản trị hệ thống
- Phát triển tự động hóa triển khai có khả năng chịu lỗi
- Viết các script phải xử lý các trường hợp biên một cách an toàn
- Xây dựng các thư viện shell script có thể bảo trì
- Triển khai logging và giám sát toàn diện
- Tạo các script phải hoạt động trên các nền tảng khác nhau

## Không sử dụng kỹ năng này khi

- Bạn cần một lệnh shell ad-hoc đơn lẻ, không phải một script
- Môi trường mục tiêu yêu cầu strict POSIX sh only
- Nhiệm vụ không liên quan đến shell scripting hoặc tự động hóa

## Hướng dẫn

1. Xác nhận shell mục tiêu, OS, và môi trường thực thi.
2. Bật strict mode và các mặc định an toàn ngay từ đầu.
3. Xác thực đầu vào, trích dẫn (quote) các biến, và xử lý tệp an toàn.
4. Thêm logging, error traps, và các kiểm thử cơ bản.

## An toàn

- Tránh các lệnh phá hủy mà không có xác nhận hoặc cờ dry-run.
- Không chạy script dưới quyền root trừ khi thực sự bắt buộc.

Tham khảo `resources/implementation-playbook.md` cho các mẫu, danh sách kiểm tra, và mẫu chi tiết.

## Tài nguyên

- `resources/implementation-playbook.md` cho các mẫu, danh sách kiểm tra, và mẫu chi tiết.
