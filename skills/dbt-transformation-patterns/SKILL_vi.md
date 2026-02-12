---
name: dbt-transformation-patterns
description: Làm chủ dbt (công cụ xây dựng dữ liệu) cho kỹ thuật phân tích với tổ chức mô hình, kiểm thử, tài liệu hóa và chiến lược gia tăng. Sử dụng khi xây dựng các chuyển đổi dữ liệu, tạo mô hình dữ liệu hoặc triển khai các thực tiễn tốt nhất về kỹ thuật phân tích.
---

# Các Mẫu Chuyển Đổi dbt (dbt Transformation Patterns)

Các mẫu sẵn sàng cho sản xuất cho dbt (công cụ xây dựng dữ liệu) bao gồm tổ chức mô hình, chiến lược kiểm thử, tài liệu hóa và xử lý gia tăng.

## Sử dụng kỹ năng này khi

- Xây dựng các đường ống chuyển đổi dữ liệu với dbt
- Tổ chức các mô hình vào các lớp staging, intermediate, và marts
- Triển khai kiểm tra chất lượng dữ liệu và tài liệu hóa
- Tạo các mô hình gia tăng cho các bộ dữ liệu lớn
- Thiết lập cấu trúc dự án dbt và các quy ước

## Không sử dụng kỹ năng này khi

- Dự án không sử dụng dbt hoặc quy trình làm việc dựa trên kho dữ liệu
- Bạn chỉ cần các truy vấn SQL ad-hoc
- Không có quyền truy cập vào dữ liệu nguồn hoặc lược đồ

## Hướng dẫn

- Xác định các lớp mô hình, đặt tên và quyền sở hữu.
- Triển khai kiểm thử, tài liệu hóa và kiểm tra độ mới.
- Chọn cụ thể hóa (materializations) và chiến lược gia tăng.
- Tối ưu hóa các lần chạy với bộ chọn (selectors) và quy trình làm việc CI.
- Nếu cần các mẫu chi tiết, hãy mở `resources/implementation-playbook.md`.

## Tài nguyên

- `resources/implementation-playbook.md` cho các mẫu dbt chi tiết và ví dụ.
