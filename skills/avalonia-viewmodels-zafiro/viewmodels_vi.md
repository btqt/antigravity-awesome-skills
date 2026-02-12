# ViewModels & Lệnh (ViewModels & Commands)

Trong một ứng dụng dựa trên Zafiro, các ViewModel nên mang tính chức năng (functional), phản ứng (reactive) và có khả năng phục hồi (resilient).

## Các Reactive ViewModel

Sử dụng `ReactiveObject` làm lớp cơ sở. Các thuộc tính nên được định nghĩa bằng thuộc tính `[Reactive]` (từ ReactiveUI.SourceGenerators) để ngắn gọn.

```csharp
public partial class MyViewModel : ReactiveObject
{
    [Reactive] private string name;
    [Reactive] private bool isBusy;
}
```

### Quan sát và Chuyển đổi

Sử dụng `WhenAnyValue` để phản ứng với các thay đổi thuộc tính:

```csharp
this.WhenAnyValue(x => x.Name)
    .Select(name => !string.IsNullOrEmpty(name))
    .ToPropertyEx(this, x => x.CanSubmit);
```

## Các Lệnh Nâng cao (Enhanced Commands)

Zafiro sử dụng `IEnhancedCommand`, giúp mở rộng `ICommand` và `IReactiveCommand` với các siêu dữ liệu bổ sung như `Name` và `Text`.

### Tạo một Lệnh

Sử dụng `ReactiveCommand.Create` hoặc `ReactiveCommand.CreateFromTask` và sau đó gọi `Enhance()`.

```csharp
public IEnhancedCommand Submit { get; }
public MyViewModel()
{
    Submit = ReactiveCommand.CreateFromTask(OnSubmit, canSubmit)
        .Enhance(text: "Submit Data", name: "SubmitCommand");
}
```

### Xử lý Lỗi

Sử dụng `HandleErrorsWith` để tự động chuyển hướng các lỗi của lệnh tới `NotificationService`.

```csharp
Submit.HandleErrorsWith(uiServices.NotificationService, "Submission Failed")
    .DisposeWith(disposable);
```

## Các Disposable

Luôn sử dụng một `CompositeDisposable` để quản lý các đăng ký (subscriptions) và vòng đời của lệnh.

```csharp
public class MyViewModel : ReactiveObject, IDisposable
{
    private readonly CompositeDisposable disposables = new();

    public void Dispose() => disposables.Dispose();
}
```

> [!TIP]
> Sử dụng `.DisposeWith(disposables)` trên bất kỳ đăng ký observable hoặc lệnh nào để đảm bảo việc dọn dẹp đúng cách.
