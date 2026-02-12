---
name: backtesting-frameworks
description: Xây dựng các hệ thống backtesting mạnh mẽ cho các chiến lược giao dịch với việc xử lý đúng đắn các lỗi look-ahead bias, survivorship bias, và chi phí giao dịch. Sử dụng khi phát triển các thuật toán giao dịch, xác thực chiến lược, hoặc xây dựng cơ sở hạ tầng backtesting.
---

# Backtesting Frameworks

Xây dựng các hệ thống backtesting cấp production mạnh mẽ, tránh các cạm bẫy phổ biến và tạo ra các ước tính hiệu suất chiến lược đáng tin cậy.

## Sử dụng kỹ năng này khi

- Phát triển backtest cho chiến lược giao dịch
- Xây dựng cơ sở hạ tầng backtesting
- Xác thực hiệu suất và độ bền vững của chiến lược
- Tránh các sai lệch (biases) phổ biến trong backtesting
- Thực hiện phân tích walk-forward

## Không sử dụng kỹ năng này khi

- Bạn cần thực hiện giao dịch trực tiếp hoặc tư vấn đầu tư
- Chất lượng dữ liệu lịch sử không xác định hoặc không đầy đủ
- Nhiệm vụ chỉ là tóm tắt hiệu suất nhanh

## Hướng dẫn

- Định nghĩa giả thuyết, vũ trụ (universe), khung thời gian, và tiêu chí đánh giá.
- Xây dựng đường ống dữ liệu point-in-time và mô hình chi phí thực tế.
- Triển khai logic mô phỏng và thực thi dựa trên sự kiện (event-driven).
- Sử dụng phân chia train/validation/test và kiểm thử walk-forward.
- Nếu cần ví dụ chi tiết, mở `resources/implementation-playbook.md`.

## An toàn

- Không trình bày backtest như là sự đảm bảo cho hiệu suất trong tương lai.
- Tránh cung cấp lời khuyên tài chính hoặc đầu tư.

## Tài nguyên

- `resources/implementation-playbook.md` cho các mẫu và ví dụ chi tiết.
