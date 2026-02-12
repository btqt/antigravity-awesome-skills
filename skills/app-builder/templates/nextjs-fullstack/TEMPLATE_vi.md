---
name: nextjs-fullstack
description: Nguyên tắc mẫu Next.js full-stack. App Router, Prisma, Tailwind.
---

# Mẫu Next.js Full-Stack

## Tech Stack

| Thành phần       | Công nghệ               |
| ---------------- | ----------------------- |
| Framework        | Next.js 14 (App Router) |
| Ngôn ngữ         | TypeScript              |
| Cơ sở dữ liệu    | PostgreSQL + Prisma     |
| Styling          | Tailwind CSS            |
| Xác thực         | Clerk (tùy chọn)        |
| Xác thực dữ liệu | Zod                     |

---

## Cấu trúc Thư mục

```
project-name/
├── prisma/
│   └── schema.prisma
├── src/
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── globals.css
│   │   └── api/
│   ├── components/
│   │   └── ui/
│   ├── lib/
│   │   ├── db.ts        # Prisma client
│   │   └── utils.ts
│   └── types/
├── .env.example
└── package.json
```

---

## Các Khái niệm Chính

| Khái niệm         | Mô tả                        |
| ----------------- | ---------------------------- |
| Server Components | Mặc định, fetch dữ liệu      |
| Server Actions    | Xử lý Form mutations         |
| Route Handlers    | Các điểm cuối API            |
| Prisma            | ORM an toàn kiểu (Type-safe) |

---

## Biến Môi trường

| Biến                | Mục đích       |
| ------------------- | -------------- |
| DATABASE_URL        | Kết nối Prisma |
| NEXT_PUBLIC_APP_URL | URL Công khai  |

---

## Các bước Thiết lập

1. `npx create-next-app {{name}} --typescript --tailwind --app`
2. `npm install prisma @prisma/client zod`
3. `npx prisma init`
4. Cấu hình schema
5. `npm run db:push`
6. `npm run dev`

---

## Thực hành Tốt nhất

- Server Components theo mặc định
- Server Actions cho các mutations
- Prisma cho DB an toàn kiểu
- Zod cho xác thực dữ liệu
- Edge runtime ở nơi có thể
