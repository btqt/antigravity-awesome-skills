# Các Chiến lược Đánh phiên bản

> Lập kế hoạch cho sự phát triển API ngay từ ngày đầu tiên.

## Các Yếu tố Quyết định

| Chiến lược   | Triển khai          | Đánh đổi                                  |
| ------------ | ------------------- | ----------------------------------------- |
| **URI**      | /v1/users           | Rõ ràng, dễ caching                       |
| **Tiêu đề**  | Accept-Version: 1   | URL sạch hơn, khó khám phá hơn            |
| **Truy vấn** | ?version=1          | Dễ thêm, lộn xộn                          |
| **Không**    | Phát triển cẩn thận | Tốt nhất cho nội bộ, rủi ro cho công khai |

## Triết lý Đánh phiên bản

```
Cân nhắc:
├── API Công khai? → Đánh phiên bản trong URI
├── Chỉ nội bộ? → Có thể không cần đánh phiên bản
├── GraphQL? → Thường không có phiên bản (phát triển ngược lược đồ)
├── tRPC? → Các kiểu thực thi khả năng tương thích
```
