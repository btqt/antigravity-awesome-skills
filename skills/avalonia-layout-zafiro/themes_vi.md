# Tổ chức Chủ đề và các Style dùng chung

Việc tổ chức chủ đề (theme) hiệu quả là chìa khóa để tránh sự dư thừa XAML và đảm bảo tính nhất quán về hình ảnh.

## 🏗️ Cấu trúc

Hãy tuân theo mẫu từ dự án Angor:

1.  **Colors & Brushes**: Định nghĩa trong một tệp `Colors.axaml` riêng biệt. Sử dụng `DynamicResource` để hỗ trợ việc chuyển đổi chủ đề.
2.  **Styles**: Nhóm các style theo danh mục (ví dụ: `Buttons.axaml`, `Containers.axaml`, `Typography.axaml`).
3.  **App-wide Theme**: Tập hợp tất cả các style trong một tệp `Theme.axaml` chính.

## 🎨 Tránh sự dư thừa

Thay vì thiết lập các thuộc tính trực tiếp trên các phần tử:

```xml
<!-- ❌ TỆ: Các thuộc tính dư thừa -->
<HeaderedContainer CornerRadius="10" BorderThickness="1" BorderBrush="Blue" Background="LightBlue" />
<HeaderedContainer CornerRadius="10" BorderThickness="1" BorderBrush="Blue" Background="LightBlue" />

<!-- ✅ TỐT: Sử dụng Class và Style -->
<HeaderedContainer Classes="BlueSection" />
<HeaderedContainer Classes="BlueSection" />
```

Hãy định nghĩa style trong một tệp `axaml` dùng chung:

```xml
<Style Selector="HeaderedContainer.BlueSection">
    <Setter Property="CornerRadius" Value="10" />
    <Setter Property="BorderThickness" Value="1" />
    <Setter Property="BorderBrush" Value="{DynamicResource Accent}" />
    <Setter Property="Background" Value="{DynamicResource SurfaceSubtle}" />
</Style>
```

## 🧩 Các Icon và Tài nguyên dùng chung

Tập trung các định nghĩa icon và các tài nguyên dùng chung khác trong `Icons.axaml` và đưa chúng vào `MergedDictionaries` của chủ đề của bạn hoặc tệp `App.axaml`.

```xml
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <MergeResourceInclude Source="UI/Themes/Styles/Containers.axaml" />
            <MergeResourceInclude Source="UI/Shared/Resources/Icons.axaml" />
        </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
</Application.Resources>
```
