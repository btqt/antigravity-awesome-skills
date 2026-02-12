# Tiêu chuẩn Đặt tên & Lập trình

## Các Tiêu chuẩn Chung

- **Tên Rõ ràng**: Ưu tiên sự rõ ràng hơn là sự khéo léo.
- **Hậu tố Async**: KHÔNG sử dụng hậu tố `Async` trong tên phương thức, ngay cả khi chúng trả về `Task`.
- **Trường Riêng tư**: KHÔNG sử dụng tiền tố `_` cho các trường riêng tư (private fields).
- **Trạng thái Tĩnh**: Tránh trạng thái tĩnh (static state) trừ khi được biện minh và ghi chép rõ ràng.
- **Thiết kế Phương thức**: Giữ các phương thức nhỏ, diễn đạt rõ ràng và có độ phức tạp cyclomatic thấp.

## Xử lý Lỗi

- **Result & Maybe**: Sử dụng các kiểu từ **CSharpFunctionalExtensions** để kiểm soát luồng và xử lý lỗi.
- **Ngoại lệ (Exceptions)**: Dành riêng cho các tình huống thực sự ngoại lệ, không thể phục hồi.
- **Ranh giới**: Không bao giờ cho phép các ngoại lệ rò rỉ qua các ranh giới kiến trúc.
