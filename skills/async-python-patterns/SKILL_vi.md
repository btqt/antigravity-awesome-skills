---
name: async-python-patterns
description: Làm chủ Python asyncio, lập trình đồng thời (concurrent programming), và các mẫu async/await cho các ứng dụng hiệu suất cao. Sử dụng khi xây dựng các API không đồng bộ, các hệ thống đồng thời, hoặc các ứng dụng bị giới hạn bởi I/O (I/O-bound) yêu cầu các hoạt động không chặn (non-blocking).
---

# Các mẫu Python Bất đồng bộ (Async Python Patterns)

Hướng dẫn toàn diện để triển khai các ứng dụng Python không đồng bộ sử dụng asyncio, các mẫu lập trình đồng thời, và async/await để xây dựng các hệ thống hiệu suất cao, không chặn.

## Sử dụng kỹ năng này khi

- Xây dựng các web API không đồng bộ (FastAPI, aiohttp, Sanic)
- Triển khai các hoạt động I/O đồng thời (cơ sở dữ liệu, tệp, mạng)
- Tạo các công cụ thu thập web (web scrapers) với các yêu cầu đồng thời
- Phát triển các ứng dụng thời gian thực (máy chủ WebSocket, hệ thống chat)
- Xử lý nhiều tác vụ độc lập đồng thời
- Xây dựng các microservices với giao tiếp không đồng bộ
- Tối ưu hóa các khối lượng công việc bị giới hạn bởi I/O
- Triển khai các tác vụ nền không đồng bộ và hàng đợi

## Không sử dụng kỹ năng này khi

- Khối lượng công việc bị giới hạn bởi CPU (CPU-bound) với ít I/O.
- Một kịch bản (script) đồng bộ đơn giản là đủ.
- Môi trường chạy (runtime) không thể hỗ trợ việc sử dụng asyncio/vòng lặp sự kiện (event loop).

## Hướng dẫn

- Làm rõ các đặc điểm của khối lượng công việc (I/O so với CPU), các mục tiêu và ràng buộc thời gian chạy.
- Chọn các mẫu đồng thời (tác vụ, gather, hàng đợi, pools) với các quy tắc hủy bỏ (cancellation).
- Thêm thời gian chờ (timeouts), áp lực ngược (backpressure), và xử lý lỗi có cấu trúc.
- Bao gồm hướng dẫn kiểm thử và debug cho các đường dẫn mã không đồng bộ.
- Nếu cần các ví dụ chi tiết, hãy mở `resources/implementation-playbook.md`.

Tham khảo `resources/implementation-playbook.md` để biết các mẫu và ví dụ chi tiết.

## Tài nguyên

- `resources/implementation-playbook.md` cho các mẫu và ví dụ chi tiết.
