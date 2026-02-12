# Các Nguyên tắc GraphQL

> Truy vấn linh hoạt cho dữ liệu phức tạp, kết nối với nhau.

## Khi nào nên sử dụng

```
✅ Phù hợp:
├── Dữ liệu phức tạp, liên kết với nhau
├── Nhiều nền tảng frontend
├── Khách hàng cần truy vấn linh hoạt
├── Yêu cầu dữ liệu phát triển
└── Giảm thiểu việc lấy quá nhiều dữ liệu là quan trọng

❌ Ít phù hợp:
├── Các hoạt động CRUD đơn giản
├── Tải lên tệp nặng
├── Caching HTTP là quan trọng
└── Nhóm không quen thuộc với GraphQL
```

## Các Nguyên tắc Thiết kế Lược đồ

```
Nguyên tắc:
├── Tư duy theo đồ thị, không phải điểm cuối
├── Thiết kế cho khả năng phát triển (không có phiên bản)
├── Sử dụng connections cho phân trang
├── Cụ thể với các loại (không dùng "data" chung chung)
└── Xử lý nullability một cách chu đáo
```

## Cân nhắc Bảo mật

```
Bảo vệ chống lại:
├── Tấn công độ sâu truy vấn → Đặt độ sâu tối đa
├── Độ phức tạp truy vấn → Tính toán chi phí
├── Lạm dụng hàng loạt (Batching abuse) → Giới hạn kích thước lô
├── Tự kiểm tra (Introspection) → Vô hiệu hóa trong sản xuất
```
