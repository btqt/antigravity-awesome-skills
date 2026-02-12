---
name: avalonia-layout-zafiro
description: Các hướng dẫn cho bố cục Avalonia UI hiện đại sử dụng Zafiro.Avalonia, nhấn mạnh vào các style dùng chung, các component generic, và tránh sự dư thừa XAML.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Bố cục Avalonia với Zafiro.Avalonia

> Nắm vững các bố cục Avalonia UI hiện đại, sạch sẽ và dễ bảo trì.
> **Tập trung vào các container mang tính ngữ nghĩa, các style dùng chung và XAML tối giản.**

## 🎯 Quy tắc Đọc Có chọn lọc

**CHỈ đọc các tệp có liên quan đến thách thức về bố cục!**

---

## 📑 Bản đồ Nội dung

| Tệp             | Mô tả                                                              | Khi nào cần Đọc                               |
| --------------- | ------------------------------------------------------------------ | --------------------------------------------- |
| `themes.md`     | Tổ chức chủ đề (theme) và các style dùng chung                     | Thiết lập hoặc tinh chỉnh các chủ đề ứng dụng |
| `containers.md` | Các container ngữ nghĩa (`HeaderedContainer`, `EdgePanel`, `Card`) | Cấu trúc các view và bố cục                   |
| `icons.md`      | Cách dùng icon với `IconExtension` và `IconOptions`                | Thêm và tùy chỉnh các icon                    |
| `behaviors.md`  | `Xaml.Interaction.Behaviors` và việc tránh dùng Converter          | Triển khai các tương tác phức tạp             |
| `components.md` | Các component generic và việc tránh lồng nhau                      | Tạo các phần tử UI có thể tái sử dụng         |

---

## 🔗 Dự án Liên quan (Triển khai Mẫu)

Đối với một ví dụ thực tế, hãy tham khảo dự án **Angor**:
`/mnt/fast/Repos/angor/src/Angor/Avalonia/Angor.Avalonia.sln`

---

## ✅ Danh sách kiểm tra cho Bố cục Sạch sẽ

- [ ] **Đã sử dụng các container ngữ nghĩa?** (ví dụ: `HeaderedContainer` thay vì `Border` với tiêu đề thủ công)
- [ ] **Đã tránh các thuộc tính dư thừa?** Sử dụng các style dùng chung trong các tệp `axaml`.
- [ ] **Đã giảm thiểu việc lồng nhau?** Làm phẳng các bố cục bằng cách sử dụng `EdgePanel` hoặc các component generic.
- [ ] **Icon thông qua extension?** Sử dụng `{Icon fa-name}` và `IconOptions` để định dạng style.
- [ ] **Sử dụng Behavior thay vì code-behind?** Sử dụng `Interaction.Behaviors` cho logic giao diện người dùng (UI-logic).
- [ ] **Đã tránh dùng Converter?** Ưu tiên các thuộc tính ViewModel hoặc Behavior trừ khi thực sự cần thiết.

---

## ❌ Các mẫu Chống thiết kế (Anti-Patterns)

**KHÔNG NÊN:**

- Sử dụng các màu sắc hoặc kích thước được mã hóa cứng (literals) trong các view.
- Tạo việc lồng nhau quá sâu của `Grid` và `StackPanel`.
- Lặp lại các thuộc tính hình ảnh trên nhiều phần tử (hãy dùng Styles).
- Sử dụng `IValueConverter` cho các logic đơn giản thuộc về ViewModel.

**NÊN:**

- Sử dụng `DynamicResource` cho màu sắc và bút vẽ (brushes).
- Trích xuất các bố cục lặp lại vào các component generic.
- Tận dụng các bảng (panels) đặc thù của `Zafiro.Avalonia` như `EdgePanel` cho các mẫu UI phổ biến.
