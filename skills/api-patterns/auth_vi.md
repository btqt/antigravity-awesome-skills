# Các Mẫu Xác thực

> Chọn mẫu xác thực dựa trên trường hợp sử dụng.

## Hướng dẫn Lựa chọn

| Mẫu           | Tốt nhất cho                                |
| ------------- | ------------------------------------------- |
| **JWT**       | Không trạng thái (Stateless), microservices |
| **Session**   | Web truyền thống, đơn giản                  |
| **OAuth 2.0** | Tích hợp bên thứ ba                         |
| **API Keys**  | Máy chủ đến máy chủ, API công khai          |
| **Passkey**   | Hiện đại không mật khẩu (2025+)             |

## Các nguyên tắc JWT

```
Quan trọng:
├── Luôn xác minh chữ ký
├── Kiểm tra hết hạn
├── Bao gồm tuyên bố tối thiểu (minimal claims)
├── Sử dụng hết hạn ngắn + refresh tokens
└── Không bao giờ lưu trữ dữ liệu nhạy cảm trong JWT
```
