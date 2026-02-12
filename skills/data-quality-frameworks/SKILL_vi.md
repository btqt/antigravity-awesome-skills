---
name: data-quality-frameworks
description: Triển khai xác thực chất lượng dữ liệu với Great Expectations, dbt tests, và hợp đồng dữ liệu. Sử dụng khi xây dựng đường ống chất lượng dữ liệu, triển khai quy tắc xác thực, hoặc thiết lập hợp đồng dữ liệu.
---

# Các Khung Chất Lượng Dữ Liệu (Data Quality Frameworks)

Các mẫu sản xuất để triển khai chất lượng dữ liệu với Great Expectations, dbt tests, và hợp đồng dữ liệu để đảm bảo đường ống dữ liệu đáng tin cậy.

## Sử dụng kỹ năng này khi

- Triển khai kiểm tra chất lượng dữ liệu trong đường ống
- Thiết lập xác thực Great Expectations
- Xây dựng bộ kiểm tra dbt toàn diện
- Thiết lập hợp đồng dữ liệu giữa các nhóm
- Giám sát các chỉ số chất lượng dữ liệu
- Tự động hóa xác thực dữ liệu trong CI/CD

## Không sử dụng kỹ năng này khi

- Các nguồn dữ liệu không được xác định hoặc không khả dụng
- Bạn không thể sửa đổi quy tắc xác thực hoặc lược đồ
- Nhiệm vụ không liên quan đến chất lượng dữ liệu hoặc hợp đồng

## Hướng dẫn

- Xác định các bộ dữ liệu quan trọng và các khía cạnh chất lượng.
- Xác định kỳ vọng/kiểm tra và quy tắc hợp đồng.
- Tự động hóa xác thực trong CI/CD và lên lịch kiểm tra.
- Thiết lập cảnh báo, quyền sở hữu và các bước khắc phục.
- Nếu cần các mẫu chi tiết, hãy mở `resources/implementation-playbook.md`.

## An toàn

- Tránh chặn các đường ống quan trọng mà không có kế hoạch dự phòng.
- Xử lý dữ liệu nhạy cảm một cách an toàn trong đầu ra xác thực.

## Tài nguyên

- `resources/implementation-playbook.md` cho các khung chi tiết, mẫu và ví dụ.
