---
name: airflow-dag-patterns
description: Xây dựng các Apache Airflow DAG sẵn sàng cho sản xuất với các thực hành tốt nhất cho operators, sensors, kiểm thử và triển khai. Sử dụng khi tạo các đường dẫn dữ liệu (data pipelines), điều phối quy trình công việc hoặc lập lịch cho các công việc hàng loạt (batch jobs).
---

# Các mẫu Apache Airflow DAG (Apache Airflow DAG Patterns)

Các mẫu sẵn sàng cho sản xuất cho Apache Airflow bao gồm thiết kế DAG, operators, sensors, kiểm thử và các chiến lược triển khai.

## Sử dụng kỹ năng này khi

- Tạo hệ điều phối đường dẫn dữ liệu với Airflow
- Thiết kế cấu trúc DAG và các phụ thuộc
- Triển khai các operator và sensor tùy chỉnh
- Kiểm thử các Airflow DAG cục bộ
- Thiết lập Airflow trong môi trường sản xuất
- Gỡ lỗi các lần chạy DAG bị thất bại

## Không sử dụng kỹ năng này khi

- Bạn chỉ cần một công việc cron đơn giản hoặc tập lệnh shell
- Airflow không nằm trong ngăn xếp công cụ của bạn
- Nhiệm vụ không liên quan đến điều phối quy trình công việc

## Hướng dẫn

1. Xác định các nguồn dữ liệu, lịch trình và các phụ thuộc.
2. Thiết kế các tác vụ có tính lặp lại (idempotent) với quyền sở hữu và cơ chế thử lại rõ ràng.
3. Triển khai các DAG với khả năng quan sát (observability) và các hook cảnh báo.
4. Xác thực trong môi trường dàn dựng (staging) và ghi chép tài liệu hướng dẫn vận hành (runbooks).

Tham khảo `resources/implementation-playbook_vi.md` để biết chi tiết về các mẫu, danh sách kiểm tra và các mẫu (templates).

## An toàn

- Tránh thay đổi lịch trình DAG sản xuất mà không được phê duyệt.
- Kiểm tra kỹ các lần chạy bù (backfills) và thử lại để ngăn ngừa việc trùng lặp dữ liệu.

## Tài nguyên

- `resources/implementation-playbook_vi.md` cho các mẫu chi tiết, danh sách kiểm tra và các mẫu (templates).
