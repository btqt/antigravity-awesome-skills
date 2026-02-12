# Hướng dẫn Chọn Mẫu

> Cây quyết định để chọn các mẫu kiến trúc.

## Cây Quyết định Chính

```
BẮT ĐẦU: Mối quan tâm CHÍNH của bạn là gì?

┌─ Phức tạp Truy cập Dữ liệu?
│  ├─ CAO (truy vấn phức tạp, cần kiểm thử)
│  │  → Mẫu Repository + Unit of Work
│  │  XÁC THỰC: Nguồn dữ liệu có thay đổi thường xuyên không?
│  │     ├─ CÓ → Repository xứng đáng với sự gián tiếp
│  │     └─ KHÔNG → Cân nhắc truy cập ORM trực tiếp đơn giản hơn
│  └─ THẤP (CRUD đơn giản, cơ sở dữ liệu đơn lẻ)
│     → ORM trực tiếp (Prisma, Drizzle)
│     Đơn giản hơn = Tốt hơn, Nhanh hơn
│
├─ Phức tạp Quy tắc Nghiệp vụ?
│  ├─ CAO (logic tên miền, quy tắc thay đổi theo ngữ cảnh)
│  │  → Domain-Driven Design (DDD)
│  │  XÁC THỰC: Bạn có chuyên gia tên miền trong nhóm không?
│  │     ├─ CÓ → Full DDD (Aggregates, Value Objects)
│  │     └─ KHÔNG → Partial DDD (thực thể phong phú, ranh giới rõ ràng)
│  └─ THẤP (chủ yếu là CRUD, xác thực đơn giản)
│     → Mẫu Transaction Script
│     Đơn giản hơn = Tốt hơn, Nhanh hơn
│
├─ Cần Mở rộng Độc lập?
│  ├─ CÓ (các thành phần khác nhau mở rộng khác nhau)
│  │  → Microservices XỨNG ĐÁNG với sự phức tạp
│  │  YÊU CẦU (TẤT CẢ phải đúng):
│  │    - Ranh giới tên miền rõ ràng
│  │    - Nhóm > 10 nhà phát triển
│  │    - Nhu cầu mở rộng khác nhau cho mỗi dịch vụ
│  │  NẾU KHÔNG ĐÁP ỨNG TẤT CẢ → Modular Monolith thay thế
│  └─ KHÔNG (mọi thứ mở rộng cùng nhau)
│     → Modular Monolith
│     Có thể trích xuất dịch vụ sau khi được chứng minh là cần thiết
│
└─ Yêu cầu Thời gian thực?
   ├─ CAO (cập nhật ngay lập tức, đồng bộ đa người dùng)
   │  → Kiến trúc Hướng sự kiện (Event-Driven Architecture)
   │  → Hàng đợi Tin nhắn (RabbitMQ, Redis, Kafka)
   │  XÁC THỰC: Bạn có thể xử lý tính nhất quán cuối cùng không?
   │     ├─ CÓ → Hướng sự kiện hợp lệ
   │     └─ KHÔNG → Đồng bộ với polling
   └─ THẤP (tính nhất quán cuối cùng chấp nhận được)
      → Đồng bộ (REST/GraphQL)
      Đơn giản hơn = Tốt hơn, Nhanh hơn
```

## 3 Câu hỏi (Trước BẤT KỲ Mẫu nào)

1. **Vấn đề Được giải quyết**: Mẫu này giải quyết vấn đề CỤ THỂ nào?
2. **Giải pháp Đơn giản hơn**: Có giải pháp đơn giản hơn không?
3. **Phức tạp Trì hoãn**: Chúng ta có thể thêm cái này SAU KHI cần thiết không?

## Cờ Đỏ (Anti-patterns)

| Mẫu             | Anti-pattern                   | Giải pháp Thay thế Đơn giản hơn                    |
| --------------- | ------------------------------ | -------------------------------------------------- |
| Microservices   | Tách sớm (Premature splitting) | Bắt đầu monolith, trích xuất sau                   |
| Clean/Hexagonal | Trừu tượng hóa quá mức         | Cụ thể trước, giao diện sau                        |
| Event Sourcing  | Kỹ thuật quá mức               | Nhật ký kiểm toán chỉ thêm (Append-only audit log) |
| CQRS            | Phức tạp không cần thiết       | Mô hình đơn lẻ                                     |
| Repository      | YAGNI cho CRUD đơn giản        | Truy cập ORM trực tiếp                             |
