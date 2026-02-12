# Các quy tắc Avalonia, Zafiro & Reactive

## Các quy tắc Avalonia UI

- **Avalonia Nghiêm ngặt**: Không bao giờ sử dụng `System.Drawing`; luôn luôn sử dụng các kiểu của Avalonia.
- **ViewModel Thuần túy**: Các ViewModel **không bao giờ** được tham chiếu tới các kiểu của Avalonia.
- **Ưu tiên Binding hơn Code-Behind**: Logic nên được điều khiển bởi các binding.
- **DataTemplate**: Ưu tiên các `DataTemplate` rõ ràng và các `DataContext` có định kiểu.
- **VisualState**: Tránh sử dụng `VisualStates` trừ khi thực sự bắt buộc.

## Các Hướng dẫn Zafiro

- **Ưu tiên các Lớp trừu tượng (Abstractions)**: Luôn tìm kiếm các trình trợ giúp, phương thức extension và các lớp trừu tượng có sẵn của Zafiro trước khi triển khai lại các logic.
- **Xác thực (Validation)**: Sử dụng `ValidationRule` của Zafiro và các extension xác thực thay vì các logic reactive tự chế (ad-hoc).

## DynamicData & Các quy tắc Reactive

### Cách tiếp cận Bắt buộc

- **Ưu tiên Toán tử**: Luôn ưu tiên các toán tử **DynamicData** (`Connect`, `Filter`, `Transform`, `Sort`, `Bind`, `DisposeMany`) hơn các toán tử Rx thuần túy khi làm việc với các collection.
- **Đường dẫn (Pipelines) dễ đọc**: Xây dựng và duy trì các đường dẫn dưới dạng một chuỗi duy nhất, dễ đọc.
- **Vòng đời (Lifecycle)**: Sử dụng `DisposeWith` để quản lý vòng đời.
- **Đăng ký tối thiểu (Minimal Subscriptions)**: Các đăng ký nên ở mức tối thiểu, tập trung và chỉ dành riêng cho các tác dụng phụ (side-effects).

### Các mẫu Chống thiết kế Bị cấm

- **Nguồn tự chế (Ad-hoc Sources)**: KHÔNG tạo mới `SourceList` / `SourceCache` một cách tùy tiện cho các vấn đề cục bộ.
- **Logic trong Subscribe**: KHÔNG đặt logic nghiệp vụ bên trong `Subscribe`.
- **Sử dụng sai Toán tử**: KHÔNG sử dụng các toán tử của `System.Reactive` nếu đã có một toán tử DynamicData tương đương.

### Các Mẫu thiết kế chuẩn (Canonical Patterns)

**Xác thực các Dynamic Collection:**

```csharp
this.ValidationRule(
        StagesSource
            .Connect()
            .FilterOnObservable(stage => stage.IsValid)
            .IsEmpty(),
        b => !b,
        _ => "Stages are not valid")
    .DisposeWith(Disposables);
```

**Lọc các giá trị Null:**
Sử dụng `WhereNotNull()` trong các đường dẫn reactive.

```csharp
this.WhenAnyValue(x => x.DurationPreset).WhereNotNull()
```
