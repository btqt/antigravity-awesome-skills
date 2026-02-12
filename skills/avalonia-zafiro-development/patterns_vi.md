# Các Mẫu phổ biến trong Angor/Zafiro

## Refreshable Collections (Bộ sưu tập có thể làm mới)

Mẫu `RefreshableCollection` được sử dụng để quản lý các danh sách có thể được làm mới thông qua một lệnh (command), duy trì một `SourceCache`/`SourceList` nội bộ và hiển thị ra một `ReadOnlyObservableCollection`.

### Triển khai

```csharp
var refresher = RefreshableCollection.Create(
        () => GetDataTask(),
        model => model.Id)
    .DisposeWith(disposable);

LoadData = refresher.Refresh;
Items = refresher.Items;
```

### Lợi ích

- **Tự động Tải**: Xử lý việc thực thi lệnh và kết quả.
- **Cập nhật Hiệu quả**: Sử dụng `EditDiff` nội bộ để cập nhật các mục mà không cần xóa danh sách.
- **Thân thiện với UI**: Hiển thị `Items` dưới dạng `ReadOnlyObservableCollection` phù hợp để ràng buộc (binding).

## Mẫu Xác thực Bắt buộc

Khi xác thực các bộ sưu tập động, luôn sử dụng phần mở rộng xác thực Zafiro:

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

## Đường ống Xử lý Lỗi

Thay vì `Subscribe` thủ công, hãy sử dụng `HandleErrorsWith` để chuyển lỗi trực tiếp đến người dùng:

```csharp
LoadProjects.HandleErrorsWith(uiServices.NotificationService, "Could not load projects");
```
