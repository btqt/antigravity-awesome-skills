# Các Nguyên tắc Định dạng Phản hồi

> Nhất quán là chìa khóa - chọn một định dạng và tuân thủ nó.

## Các Mẫu Phổ biến

```
Chọn một:
├── Envelope pattern ({ success, data, error })
├── Dữ liệu trực tiếp (chỉ trả về tài nguyên)
└── HAL/JSON:API (hypermedia)
```

## Phản hồi Lỗi

```
Bao gồm:
├── Mã lỗi (để xử lý bằng chương trình)
├── Thông báo người dùng (để hiển thị)
├── Chi tiết (để gỡ lỗi, lỗi cấp trường)
├── ID yêu cầu (để hỗ trợ)
└── KHÔNG bao gồm chi tiết nội bộ (bảo mật!)
```

## Các Loại Phân trang

| Loại       | Tốt nhất cho                | Đánh đổi                        |
| ---------- | --------------------------- | ------------------------------- |
| **Offset** | Đơn giản, có thể nhảy trang | Hiệu năng trên tập dữ liệu lớn  |
| **Cursor** | Tập dữ liệu lớn             | Không thể nhảy đến trang cụ thể |
| **Keyset** | Hiệu năng quan trọng        | Yêu cầu khóa có thể sắp xếp     |

### Các câu hỏi Lựa chọn

1. Tập dữ liệu lớn đến mức nào?
2. Người dùng có cần nhảy đến các trang cụ thể không?
3. Dữ liệu có thường xuyên thay đổi không?
