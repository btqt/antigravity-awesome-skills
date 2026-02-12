# Cấu trúc & Ánh xạ (Composition & Mapping)

Đảm bảo các ViewModel của bạn được khởi tạo đúng cách và được ánh xạ tới các View tương ứng là điều quan trọng đối với một ứng dụng dễ bảo trì.

## Ánh xạ ViewModel tới View

Zafiro sử dụng `DataTypeViewLocator` để tự động ánh xạ ViewModel tới View dựa trên kiểu dữ liệu của chúng.

### Tích hợp trong App.axaml

Đăng ký `DataTypeViewLocator` trong các data template của ứng dụng:

```xml
<Application.DataTemplates>
    <DataTypeViewLocator />
    <DataTemplateInclude Source="avares://Zafiro.Avalonia/DataTemplates.axaml" />
</Application.DataTemplates>
```

### Việc Đăng ký

Các ánh xạ có thể được đăng ký trên toàn cục hoặc cục bộ. Thực hành phổ biến trong các dự án Zafiro là sử dụng các quy ước đặt tên hoặc các đăng ký rõ ràng được tạo ra bởi các source generator.

## Gốc Cấu tạo (Composition Root)

Sử dụng một `CompositionRoot` trung tâm để quản lý việc tiêm phụ thuộc (dependency injection) và đăng ký dịch vụ.

```csharp
public static class CompositionRoot
{
    public static IShellViewModel CreateMainViewModel(Control topLevelView)
    {
        var services = new ServiceCollection();

        services
            .AddViewModels()
            .AddUIServices(topLevelView);

        var serviceProvider = services.BuildServiceProvider();
        return serviceProvider.GetRequiredService<IShellViewModel>();
    }
}
```

### Đăng ký các ViewModel

Đăng ký các ViewModel với các phạm vi (scopes) phù hợp (Transient, Scoped, hoặc Singleton).

```csharp
public static IServiceCollection AddViewModels(this IServiceCollection services)
{
    return services
        .AddTransient<IHomeSectionViewModel, HomeSectionSectionViewModel>()
        .AddSingleton<IShellViewModel, ShellViewModel>();
}
```

## Tiêm View (View Injection)

Sử dụng trình trợ giúp `Connect` (nếu có) hoặc khởi tạo thủ công trong `OnFrameworkInitializationCompleted`:

```csharp
public override void OnFrameworkInitializationCompleted()
{
    this.Connect(
        () => new ShellView(),
        view => CompositionRoot.CreateMainViewModel(view),
        () => new MainWindow());

    base.OnFrameworkInitializationCompleted();
}
```

> [!TIP]
> Sử dụng `ActivatorUtilities.CreateInstance` khi bạn cần khởi tạo thủ công một lớp trong khi vẫn giải quyết các phụ thuộc của nó từ `IServiceProvider`.
