# Cách dùng Icon

`Zafiro.Avalonia` đơn giản hóa việc quản lý icon bằng cách sử dụng một markup extension chuyên biệt và các tùy chọn định dạng style.

## 🛠️ IconExtension

Sử dụng markup extension `{Icon}` để dễ dàng nhúng các icon từ các thư viện như FontAwesome.

```xml
<!-- Tham số theo vị trí (Positional parameter) -->
<Button Content="{Icon fa-wallet}" />

<!-- Tham số theo tên (Named parameter) -->
<ContentControl Content="{Icon Source=fa-gear}" />
```

## 🎨 IconOptions

`IconOptions` cho phép bạn tùy chỉnh các icon mà không cần bao bọc chúng một cách thủ công trong các control khác. Nó thường được sử dụng trong các style để mang lại một cái nhìn nhất quán.

```xml
<Style Selector="HeaderedContainer /template/ ContentPresenter#Header EdgePanel /template/ ContentControl#StartContent">
    <Setter Property="IconOptions.Size" Value="20" />
    <Setter Property="IconOptions.Fill" Value="{DynamicResource Accent}" />
    <Setter Property="IconOptions.Padding" Value="10" />
    <Setter Property="IconOptions.CornerRadius" Value="10" />
</Style>
```

### Các thuộc tính Phổ biến:

- `IconOptions.Size`: Thiết lập chiều rộng và chiều cao của icon.
- `IconOptions.Fill`: Màu sắc/bút vẽ (brush) của icon.
- `IconOptions.Background`: Bút vẽ nền cho container của icon.
- `IconOptions.Padding`: Padding bên trong container của icon.
- `IconOptions.CornerRadius`: Bán kính góc nếu có sử dụng nền.

## 📁 Các Tài nguyên Icon dùng chung

Định nghĩa các icon như là các tài nguyên (resources) để tái sử dụng trong toàn bộ ứng dụng.

```xml
<ResourceDictionary xmlns="https://github.com/avaloniaui">
    <Icon x:Key="fa-wallet" Source="fa-wallet" />
</ResourceDictionary>
```

Sau đó sử dụng chúng với `StaticResource` nếu chúng đã được định nghĩa:

```xml
<Button Content="{StaticResource fa-wallet}" />
```

Tuy nhiên, extension `{Icon ...}` thường được ưu tiên hơn vì sự ngắn gọn và khả năng tạo các instance icon mới một cách nhanh chóng.
