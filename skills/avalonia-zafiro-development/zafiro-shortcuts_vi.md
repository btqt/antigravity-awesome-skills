# Các phím tắt Zafiro Reactive

Sử dụng các phương thức mở rộng Zafiro này để thay thế các mẫu Reactive và DynamicData tiêu chuẩn, dài dòng hơn.

## Các Trợ giúp Observable Chung

| Mẫu Tiêu chuẩn                               | Phím tắt Zafiro      |
| :------------------------------------------- | :------------------- |
| `Replay(1).RefCount()`                       | `ReplayLastActive()` |
| `Select(_ => Unit.Default)`                  | `ToSignal()`         |
| `Select(b => !b)`                            | `Not()`              |
| `Where(b => b).ToSignal()`                   | `Trues()`            |
| `Where(b => !b).ToSignal()`                  | `Falses()`           |
| `Select(x => x is null)`                     | `Null()`             |
| `Select(x => x is not null)`                 | `NotNull()`          |
| `Select(string.IsNullOrWhiteSpace)`          | `NullOrWhitespace()` |
| `Select(s => !string.IsNullOrWhiteSpace(s))` | `NotNullOrEmpty()`   |

## Các Mở rộng Result & Maybe

| Mẫu Tiêu chuẩn                                 | Phím tắt Zafiro |
| :--------------------------------------------- | :-------------- |
| `Where(r => r.IsSuccess).Select(r => r.Value)` | `Successes()`   |
| `Where(r => r.IsFailure).Select(r => r.Error)` | `Failures()`    |
| `Where(m => m.HasValue).Select(m => m.Value)`  | `Values()`      |
| `Where(m => !m.HasValue).ToSignal()`           | `Empties()`     |

## Quản lý Vòng đời

| Mô tả                                          | Phương thức                |
| :--------------------------------------------- | :------------------------- |
| Dispose mục trước đó trước khi phát ra mục mới | `DisposePrevious()`        |
| Quản lý vòng đời trong một disposable          | `DisposeWith(disposables)` |

## Lệnh & Tương tác

| Mô tả                                         | Phương thức                             |
| :-------------------------------------------- | :-------------------------------------- |
| Thêm metadata/văn bản vào một ReactiveCommand | `Enhance(text, name)`                   |
| Tự động hiển thị lỗi trong UI                 | `HandleErrorsWith(notificationService)` |

> [!TIP]
> Luôn kiểm tra `Zafiro.Reactive.ObservableMixin` và `Zafiro.CSharpFunctionalExtensions.ObservableExtensions` trước khi viết logic Rx tùy chỉnh.
