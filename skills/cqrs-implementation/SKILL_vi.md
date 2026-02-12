---
name: cqrs-implementation
description: Triển khai Command Query Responsibility Segregation cho kiến trúc có thể mở rộng. Sử dụng khi tách biệt các mô hình đọc và ghi, tối ưu hóa hiệu suất truy vấn hoặc xây dựng các hệ thống nguồn sự kiện (event-sourced).
---

# Triển Khai CQRS (CQRS Implementation)

Hướng dẫn toàn diện để triển khai các mẫu CQRS (Command Query Responsibility Segregation - Phân tách Trách nhiệm Truy vấn Lệnh).

## Sử dụng kỹ năng này khi

- Tách biệt các mối quan tâm đọc và ghi
- Mở rộng đọc độc lập với ghi
- Xây dựng các hệ thống nguồn sự kiện
- Tối ưu hóa các kịch bản truy vấn phức tạp
- Cần các mô hình dữ liệu đọc/ghi khác nhau
- Yêu cầu báo cáo hiệu suất cao

## Không sử dụng kỹ năng này khi

- Miền đơn giản và CRUD là đủ
- Bạn không thể vận hành các mô hình đọc/ghi riêng biệt
- Yêu cầu sự nhất quán ngay lập tức mạnh mẽ ở mọi nơi

## Hướng dẫn

- Xác định khối lượng công việc đọc/ghi và nhu cầu nhất quán.
- Xác định các mô hình lệnh và truy vấn với ranh giới rõ ràng.
- Triển khai các dự đoán mô hình đọc và đồng bộ hóa.
- Xác thực hiệu suất, phục hồi và các chế độ thất bại.
- Nếu cần các mẫu chi tiết, hãy mở `resources/implementation-playbook.md`.

## Tài nguyên

- `resources/implementation-playbook.md` cho các mẫu và mẫu CQRS chi tiết.
