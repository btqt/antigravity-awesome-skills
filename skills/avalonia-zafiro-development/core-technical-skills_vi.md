# Kỹ năng Kỹ thuật Cốt lõi & Kiến trúc

## Chuyên môn Bắt buộc

Nhà phát triển phải sở hữu chuyên môn vững vàng về:

- **C# và .NET hiện đại**: Sử dụng các tính năng mới nhất của ngôn ngữ và framework.
- **Avalonia UI**: Để phát triển giao diện người dùng (UI) đa nền tảng.
- **Kiến trúc MVVM**: Duy trì sự tách biệt nghiêm ngặt giữa giao diện người dùng và logic nghiệp vụ.
- **Clean Code & Kiến trúc Sạch (Clean Architecture)**: Tập trung vào khả năng bảo trì và luồng phụ thuộc hướng vào bên trong.
- **Lập trình Chức năng trong C#**: Đón nhận tính không thay đổi (immutability) và các mẫu chức năng.
- **Lập trình Phản ứng (Reactive Programming)**: Có chuyên môn về DynamicData và System.Reactive.

## Các Nguyên tắc Kiến trúc

- **MVVM Thuần túy**: Bắt buộc cho tất cả các mã nguồn UI. Logic phải độc lập với các mối quan tâm về giao diện người dùng.
- **Ưu tiên Cấu thành hơn Kế thừa (Composition over Inheritance)**: Ưu tiên các khối xây dựng module hơn là các hệ thống phân cấp kế thừa sâu.
- **Luồng Phụ thuộc hướng vào trong**: Các lớp trừu tượng (abstractions) không được phụ thuộc vào các triển khai (implementations).
- **Tính không thay đổi (Immutability)**: Ưu tiên các cấu trúc không thay đổi khi thực tế để đảm bảo khả năng dự đoán.
- **Các Public API ổn định**: Thiết kế các API một cách cẩn thận để đảm bảo sự ổn định và rõ ràng lâu dài.
