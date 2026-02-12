# Lựa chọn Kiểu API (2025)

> REST vs GraphQL vs tRPC - Cái nào trong trường hợp nào?

## Cây Quyết định

```
Ai là người tiêu dùng API?
│
├── API Công khai / Nhiều nền tảng
│   └── REST + OpenAPI (tương thích rộng nhất)
│
├── Nhu cầu dữ liệu phức tạp / Nhiều frontend
│   └── GraphQL (truy vấn linh hoạt)
│
├── TypeScript frontend + backend (monorepo)
│   └── tRPC (an toàn kiểu end-to-end)
│
├── Thời gian thực / Hướng sự kiện
│   └── WebSocket + AsyncAPI
│
└── Microservices nội bộ
    └── gRPC (hiệu năng) hoặc REST (đơn giản)
```

## So sánh

| Yếu tố                    | REST               | GraphQL           | tRPC               |
| ------------------------- | ------------------ | ----------------- | ------------------ |
| **Tốt nhất cho**          | API Công khai      | Ứng dụng phức tạp | TS monorepos       |
| **Đường cong học tập**    | Thấp               | Trung bình        | Thấp (nếu TS)      |
| **Lấy quá nhiều/thiếu**   | Phổ biến           | Đã giải quyết     | Đã giải quyết      |
| **An toàn kiểu**          | Thủ công (OpenAPI) | Dựa trên lược đồ  | Tự động            |
| **Lưu trữ đệm (Caching)** | HTTP gốc           | Phức tạp          | Dựa trên máy khách |

## Các câu hỏi Lựa chọn

1. Ai là người tiêu dùng API?
2. Frontend có phải là TypeScript không?
3. Các mối quan hệ dữ liệu phức tạp đến mức nào?
4. Lưu trữ đệm có quan trọng không?
5. API công khai hay nội bộ?
