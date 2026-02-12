# Xây dựng các Component Generic

Việc giảm bớt sự lồng nhau và độ phức tạp đạt được bằng cách chia nhỏ các view thành các component generic (tổng quát), có thể tái sử dụng.

## 🧊 Các Component Generic

Thay vì xây dựng các view lớn và phức tạp, hãy trích xuất các mẫu lặp đi lặp lại vào các `UserControl` nhỏ.

### Ví dụ: Một "Summary Item" generic

Thay vì lặp lại một `Grid` với các nhãn và giá trị:

```xml
<!-- ❌ TỆ: Grid lặp lại -->
<Grid ColumnDefinitions="*,Auto">
   <TextBlock Text="Total:" />
   <TextBlock Grid.Column="1" Text="{Binding Total}" />
</Grid>
```

Hãy tạo một component generic (hoặc sử dụng `EdgePanel` với một Style):

```xml
<!-- ✅ TỐT: Sử dụng một control hoặc style chuyên biệt -->
<EdgePanel StartContent="Total:" EndContent="{Binding Total}" Classes="SummaryItem" />
```

## 📉 Làm phẳng các Bố cục (Flattening Layouts)

Tránh việc lồng nhau quá sâu. XAML lồng nhau sâu rất khó đọc và có thể ảnh hưởng đến hiệu năng.

- **StackPanel so với Grid**: Sử dụng `StackPanel` (với `Spacing`) cho các bố cục tuyến tính đơn giản.
- **EdgePanel**: Tuyệt vời cho các hàng kiểu "Nhãn - Giá trị" hoặc "Icon - Văn bản - Hành động".
- **UniformGrid**: Sử dụng cho các lưới mà tất cả các ô có cùng kích thước.

## 🔧 Độ chi tiết của Component (Component Granularity)

- **Nguyên tử (Atomical)**: Các control nhỏ như các nút hoặc icon tùy chỉnh.
- **Phân tử (Molecular)**: Các nhóm nguyên tử như một `HeaderedContainer` với nội dung cụ thể.
- **Cơ thể (Organisms)**: Các phần cấp cao hơn của một trang.

Hãy hướng tới các component đủ generic để có thể tái sử dụng nhưng cũng đủ cụ thể để đơn giản hóa đáng kể view cha.
