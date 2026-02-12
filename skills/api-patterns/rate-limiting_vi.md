# Các Nguyên tắc Giới hạn Tốc độ

> Bảo vệ API của bạn khỏi lạm dụng và quá tải.

## Tại sao cần Giới hạn Tốc độ

```
Bảo vệ chống lại:
├── Tấn công brute force
├── Cạn kiệt tài nguyên
├── Vượt quá chi phí (nếu trả tiền theo mức sử dụng)
└── Sử dụng không công bằng
```

## Lựa chọn Chiến lược

| Loại               | Cách thức                                | Khi nào              |
| ------------------ | ---------------------------------------- | -------------------- |
| **Token bucket**   | Cho phép bùng nổ, làm đầy theo thời gian | Hầu hết các API      |
| **Sliding window** | Phân phối mượt mà                        | Giới hạn nghiêm ngặt |
| **Fixed window**   | Bộ đếm đơn giản cho mỗi cửa sổ           | Nhu cầu cơ bản       |

## Tiêu đề Phản hồi

```
Bao gồm trong tiêu đề:
├── X-RateLimit-Limit (số yêu cầu tối đa)
├── X-RateLimit-Remaining (số yêu cầu còn lại)
├── X-RateLimit-Reset (khi nào giới hạn được đặt lại)
└── Trả về 429 khi vượt quá
```
