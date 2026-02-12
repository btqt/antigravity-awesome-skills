# Các Container Ngữ nghĩa (Semantic Containers)

Sử dụng đúng container cho từng loại dữ liệu sẽ đơn giản hóa XAML và cải thiện khả năng bảo trì. `Zafiro.Avalonia` cung cấp các control chuyên biệt cho các mẫu bố cục phổ biến.

## 📦 HeaderedContainer

Ưu tiên dùng `HeaderedContainer` thay vì một `Border` hoặc `Grid` khi một phần (section) cần có tiêu đề hoặc phần đầu (header).

```xml
<HeaderedContainer Header="Security Settings" Classes="WizardSection">
    <StackPanel>
        <!-- Nội dung ở đây -->
    </StackPanel>
</HeaderedContainer>
```

### Các thuộc tính Chính:

- `Header`: Nội dung hoặc chuỗi văn bản cho phần đầu.
- `HeaderBackground`: Brush cho khu vực phần đầu.
- `ContentPadding`: Padding cho khu vực nội dung.

## ↔️ EdgePanel

Sử dụng `EdgePanel` để đặt các phần tử ở các cạnh của một container mà không cần các định nghĩa `Grid` phức tạp.

```xml
<EdgePanel StartContent="{Icon fa-wallet}"
           Content="Wallet Balance"
           EndContent="$1,234.00" />
```

### Các Khe cắm (Slots):

- `StartContent`: Căn lề trái (hoặc bắt đầu).
- `Content`: Lấp đầy khoảng trống còn lại ở giữa.
- `EndContent`: Căn lề phải (hoặc kết thúc).

## 📇 Card

Một container đơn giản để nhóm các thông tin liên quan, thường được sử dụng bên trong `HeaderedContainer` hoặc như một phần tử độc lập trong một danh sách.

```xml
<Card Header="Enter recipient address:">
    <TextBox Text="{Binding Address}" />
</Card>
```

## 📐 Các Thực hành Tốt nhất

- Sử dụng `Classes` để áp dụng các biến thể theo chủ đề (ví dụ: `Classes="Section"`, `Classes="Highlight"`).
- Tùy chỉnh các phần bên trong của các container bằng cách sử dụng template trong các style của bạn khi cần thiết, thay vì lồng thêm nhiều control khác.
