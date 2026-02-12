---
name: conductor-manage
description: "Quản lý vòng đời theo dõi: lưu trữ, khôi phục, xóa, đổi tên và dọn dẹp"
metadata:
  argument-hint: "[--archive | --restore | --delete | --rename | --list | --cleanup]"
---

# Quản lý Theo dõi (Track Manager)

Quản lý vòng đời theo dõi hoàn chỉnh bao gồm lưu trữ, khôi phục, xóa, đổi tên và dọn dẹp các tạo tác mồ côi.

## Sử dụng kỹ năng này khi

- Lưu trữ, khôi phục, đổi tên hoặc xóa các theo dõi Conductor
- Liệt kê trạng thái theo dõi hoặc làm sạch các tạo tác mồ côi
- Quản lý vòng đời theo dõi qua các trạng thái hoạt động, hoàn thành và lưu trữ

## Không sử dụng kỹ năng này khi

- Conductor chưa được khởi tạo trong kho lưu trữ
- Bạn thiếu quyền sửa đổi siêu dữ liệu theo dõi hoặc tệp
- Nhiệm vụ không liên quan đến quản lý theo dõi Conductor

## Hướng dẫn

- Xác minh cấu trúc `conductor/` và các tệp cần thiết trước khi tiếp tục.
- Xác định chế độ hoạt động từ các đối số hoặc lời nhắc tương tác.
- Xác nhận các hành động phá hủy (xóa/dọn dẹp) trước khi áp dụng.
- Cập nhật `tracks.md` và siêu dữ liệu một cách nhất quán.
- Nếu cần các bước chi tiết, hãy mở `resources/implementation-playbook.md`.

## An toàn

- Sao lưu dữ liệu theo dõi trước các hoạt động xóa.
- Tránh xóa các theo dõi đã lưu trữ mà không có sự chấp thuận rõ ràng.

## Tài nguyên

- `resources/implementation-playbook.md` cho các chế độ, lời nhắc và quy trình làm việc chi tiết.
