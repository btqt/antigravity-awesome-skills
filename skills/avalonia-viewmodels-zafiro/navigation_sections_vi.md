# Điều hướng & Các Section (Navigation & Sections)

Zafiro cung cấp các lớp trừu tượng mạnh mẽ để quản lý việc điều hướng toàn ứng dụng và các phần giao diện người dùng (UI) theo module.

## Điều hướng với INavigator

Interface `INavigator` được sử dụng để chuyển đổi giữa các view hoặc viewmodel khác nhau.

```csharp
public class MyViewModel(INavigator navigator)
{
    public async Task GoToDetails()
    {
        await navigator.Navigate(() => new DetailsViewModel());
    }
}
```

## Các UI Section

Các section là các phần module của giao diện người dùng (như các tab hoặc các mục thanh bên) có thể được tự động đăng ký.

### Thuộc tính [Section]

Các ViewModel dự định trở thành các section nên được đánh dấu bằng thuộc tính `[Section]`.

```csharp
[Section("Wallet", icon: "fa-wallet")]
public class WalletSectionViewModel : IWalletSectionViewModel
{
    // ...
}
```

### Việc Đăng ký tự động

Trong `CompositionRoot`, các section có thể được tự động đăng ký:

```csharp
services.AddAnnotatedSections(logger);
services.AddSectionsFromAttributes(logger);
```

### Chuyển đổi các Section

Bạn có thể chuyển đổi section hiện đang hoạt động thông qua `IShellViewModel`:

```csharp
shellViewModel.SetSection("Browse");
```

> [!IMPORTANT]
> Tham số `icon` trong thuộc tính `[Section]` hỗ trợ các FontAwesome icon (ví dụ: `fa-home`) khi được cấu hình với `ProjektankerIconControlProvider`.
