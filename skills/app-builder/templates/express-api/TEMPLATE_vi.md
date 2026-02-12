---
name: express-api
description: Các nguyên tắc mẫu Express.js REST API. TypeScript, Prisma, JWT.
---

# Mẫu Express.js API

## Tech Stack

| Thành phần          | Công nghệ           |
| ------------------- | ------------------- |
| Runtime             | Node.js 20+         |
| Framework           | Express.js          |
| Ngôn ngữ            | TypeScript          |
| Cơ sở dữ liệu       | PostgreSQL + Prisma |
| Xác thực dữ liệu    | Zod                 |
| Xác thực người dùng | JWT + bcrypt        |

---

## Cấu trúc Thư mục

```
project-name/
├── prisma/
│   └── schema.prisma
├── src/
│   ├── app.ts           # Cấu hình Express
│   ├── config/          # Môi trường
│   ├── routes/          # Xử lý định tuyến
│   ├── controllers/     # Logic nghiệp vụ
│   ├── services/        # Truy cập dữ liệu
│   ├── middleware/
│   │   ├── auth.ts      # Xác minh JWT
│   │   ├── error.ts     # Xử lý lỗi
│   │   └── validate.ts  # Xác thực Zod
│   ├── schemas/         # Các schema Zod
│   └── utils/
└── package.json
```

---

## Middleware Stack

| Thứ tự | Middleware                    |
| ------ | ----------------------------- |
| 1      | helmet (bảo mật)              |
| 2      | cors                          |
| 3      | morgan (ghi nhật ký)          |
| 4      | phân tích body (body parsing) |
| 5      | các tuyến đường (routes)      |
| 6      | xử lý lỗi (error handler)     |

---

## Định dạng Phản hồi API

| Loại       | Cấu trúc                               |
| ---------- | -------------------------------------- |
| Thành công | `{ success: true, data: {...} }`       |
| Lỗi        | `{ error: "message", details: [...] }` |

---

## Các bước Thiết lập

1. Tạo thư mục dự án
2. `npm init -y`
3. Cài đặt các phụ thuộc: `npm install express prisma zod bcrypt jsonwebtoken`
4. Cấu hình Prisma
5. `npm run db:push`
6. `npm run dev`

---

## Thực hành Tốt nhất

- Kiến trúc phân lớp (routes → controllers → services)
- Xác thực tất cả đầu vào với Zod
- Xử lý lỗi tập trung
- Cấu hình dựa trên môi trường
- Sử dụng Prisma để truy cập DB an toàn kiểu
