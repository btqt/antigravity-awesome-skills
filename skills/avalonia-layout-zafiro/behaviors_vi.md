# Các Tương tác và Logic

Để giữ cho XAML sạch sẽ và dễ bảo trì, hãy giảm thiểu logic trong các view và tránh sử dụng quá nhiều converter.

## 🎭 Xaml.Interaction.Behaviors

Sử dụng `Interaction.Behaviors` để xử lý logic liên quan đến UI mà không thuộc về ViewModel, chẳng hạn như quản lý focus, hiệu ứng hoạt họa (animations), hoặc xử lý sự kiện chuyên biệt.

```xml
<TextBox Text="{Binding Address}">
    <Interaction.Behaviors>
        <UntouchedClassBehavior />
    </Interaction.Behaviors>
</TextBox>
```

### Tại sao nên sử dụng Behavior?

- **Tính đóng gói (Encapsulation)**: Logic UI được chứa trong một lớp behavior có thể tái sử dụng.
- **XAML Sạch sẽ**: Tránh dùng code-behind và các trigger XAML phức tạp.
- **Khả năng kiểm thử**: Các behavior có thể được kiểm thử độc lập với View.

## 🚫 Tránh dùng Converter

Các converter thường dẫn đến logic "ma thuật" bị ẩn giấu trong XAML. Bất cứ khi nào có thể, hãy ưu tiên:

1.  **Các thuộc tính ViewModel**: Hãy để ViewModel cung cấp định dạng dữ liệu cuối cùng (ví dụ: một `string` được định dạng để hiển thị).
2.  **MultiBinding**: Sử dụng cho các tổ hợp logic đơn giản (And/Or) trực tiếp trong XAML.
3.  **Behavior**: Đối với các tương tác phức tạp hơn liên quan đến trạng thái hoặc sự kiện.

### Khi nào nên sử dụng Converter?

Chỉ sử dụng chúng khi việc chuyển đổi hoàn toàn mang tính hình ảnh và có khả năng tái sử dụng cao trong các ngữ cảnh khác nhau (ví dụ: `BoolToOpacityConverter`).

## 🧩 Các Tương tác được Đơn giản hóa

Nếu bạn thấy mình cần một converter hoặc behavior phức tạp, hãy cân nhắc xem liệu component đó có thể được đơn giản hóa hay không, hoặc liệu data model có thể được điều chỉnh để làm cho việc binding của view trực tiếp hơn hay không.
