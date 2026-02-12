# Wizard & Luồng (Wizards & Flows)

Các quy trình nhiều bước phức tạp được xử lý bằng mẫu `SlimWizard`. Điều này cung cấp một cách khai báo để định nghĩa các bước, logic điều hướng và kết quả cuối cùng.

## Định nghĩa một Wizard

Sử dụng `WizardBuilder` để định nghĩa các bước. Mỗi bước tương ứng với một ViewModel.

```csharp
SlimWizard<string> wizard = WizardBuilder
    .StartWith(() => new Step1ViewModel(data))
        .NextUnit()
        .WhenValid()
    .Then(prevResult => new Step2ViewModel(prevResult))
        .NextCommand(vm => vm.CustomNextCommand)
    .Then(result => new SuccessViewModel("Done!"))
        .Next((_, s) => s, "Finish")
    .WithCompletionFinalStep();
```

### Các Quy tắc Điều hướng

- **NextUnit()**: Chuyển tiếp khi một tín hiệu đơn giản được phát ra.
- **NextCommand()**: Chuyển tiếp khi một lệnh cụ thể trong ViewModel thực thi thành công.
- **WhenValid()**: Chờ cho đến khi việc xác thực của ViewModel hiện tại vượt qua trước khi cho phép điều hướng.
- **Always()**: Luôn cho phép điều hướng.

## Tích hợp Điều hướng

Wizard được điều hướng bằng cách sử dụng một `INavigator`:

```csharp
public async Task CreateSomething()
{
    var wizard = BuildWizard();
    var result = await wizard.Navigate(navigator);
    // Xử lý kết quả
}
```

## Cấu hình Bước

- **WithCompletionFinalStep()**: Đánh dấu wizard là kết thúc khi bước cuối cùng hoàn thành.
- **WithCommitFinalStep()**: Thường được sử dụng cho các wizard thực hiện hành động "Lưu" hoặc "Triển khai" cuối cùng.

> [!NOTE]
> `SlimWizard` tự động xử lý lệnh "Back" (Quay lại), mang lại trải nghiệm người dùng nhất quán giữa các luồng khác nhau.
