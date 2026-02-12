---
name: avalonia-zafiro-development
description: Các kỹ năng bắt buộc, các quy ước và các quy tắc hành vi cho việc phát triển Avalonia UI sử dụng bộ công cụ Zafiro.
---

# Phát triển Avalonia Zafiro

Kỹ năng này định nghĩa các quy ước bắt buộc và các quy tắc hành vi để phát triển các ứng dụng đa nền tảng với Avalonia UI và bộ công cụ Zafiro. Các quy tắc này ưu tiên khả năng bảo trì, tính đúng đắn và cách tiếp cận chức năng-phản ứng (functional-reactive).

## Các Trụ cột Cốt lõi

1.  **MVVM Chức năng-Phản ứng**: Logic MVVM thuần túy sử dụng DynamicData và ReactiveUI.
2.  **An toàn & Có thể dự đoán**: Xử lý lỗi rõ ràng với các kiểu `Result` và tránh dùng exception để điều khiển luồng.
3.  **Sự xuất sắc trên Đa nền tảng**: Các ViewModel độc lập hoàn toàn với Avalonia và ưu tiên cấu thành hơn kế thừa (composition-over-inheritance).
4.  **Ưu tiên Zafiro**: Tận dụng các lớp trừu tượng và các trình trợ giúp có sẵn của Zafiro để tránh sự dư thừa.

## Các Hướng dẫn

- [Kỹ năng Kỹ thuật Cốt lõi & Kiến trúc](core-technical-skills.md): Các kỹ năng cơ bản và các nguyên tắc kiến trúc.
- [Tiêu chuẩn Đặt tên & Lập trình](naming-standards.md): Các quy tắc đặt tên, các trường (fields) và xử lý lỗi.
- [Các quy tắc Avalonia, Zafiro & Reactive](avalonia-reactive-rules.md): Các hướng dẫn cụ thể cho UI, tích hợp Zafiro và các đường dẫn (pipelines) DynamicData.
- [Các phím tắt Zafiro](zafiro-shortcuts.md): Các ánh xạ ngắn gọn cho các hoạt động Rx/Zafiro phổ biến.
- [Các Mẫu thiết kế Phổ biến](patterns.md): Các mẫu nâng cao như `RefreshableCollection` và Validation (Xác thực).

## Quy trình Trước khi Viết Mã

1.  **Tìm kiếm Trước**: Tìm kiếm trong codebase các triển khai tương tự hoặc các trình trợ giúp Zafiro hiện có.
2.  **Các Extension có thể tái sử dụng**: Nếu thiếu một trình trợ giúp, hãy đề xuất một phương thức extension mới có thể tái sử dụng thay vì đưa trực tiếp các logic phức tạp vào mã nguồn.
3.  **Đường dẫn Phản ứng (Reactive Pipelines)**: Đảm bảo các toán tử DynamicData được sử dụng thay cho Rx thuần túy khi có thể áp dụng.
