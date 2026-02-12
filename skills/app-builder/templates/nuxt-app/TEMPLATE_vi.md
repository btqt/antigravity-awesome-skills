---
name: nuxt-app
description: Mẫu Nuxt 3 full-stack. Vue 3, Pinia, Tailwind, Prisma.
---

# Mẫu Nuxt 3 Full-Stack

## Tech Stack

| Thành phần         | Công nghệ               |
| ------------------ | ----------------------- |
| Framework          | Nuxt 3                  |
| Ngôn ngữ           | TypeScript              |
| UI                 | Vue 3 (Composition API) |
| Trạng thái (State) | Pinia                   |
| Cơ sở dữ liệu      | PostgreSQL + Prisma     |
| Styling            | Tailwind CSS            |
| Xác thực dữ liệu   | Zod                     |

---

## Cấu trúc Thư mục

```
project-name/
├── prisma/
│   └── schema.prisma
├── server/
│   ├── api/
│   │   └── [resource]/
│   │       └── index.ts
│   └── utils/
│       └── db.ts         # Prisma client
├── composables/
│   └── useAuth.ts
├── stores/
│   └── user.ts           # Pinia store
├── components/
│   └── ui/
├── pages/
│   ├── index.vue
│   └── [...slug].vue
├── layouts/
│   └── default.vue
├── assets/
│   └── css/
│       └── main.css
├── .env.example
├── nuxt.config.ts
└── package.json
```

---

## Các Khái niệm Chính

| Khái niệm                   | Mô tả                          |
| --------------------------- | ------------------------------ |
| Tự động nhập (Auto-imports) | Components, composables, utils |
| Định tuyến dựa trên tệp     | pages/ → routes                |
| Server Routes               | server/api/ → API endpoints    |
| Composables                 | Logic phản ứng tái sử dụng     |
| Pinia                       | Quản lý trạng thái             |

---

## Biến Môi trường

| Biến                | Mục đích       |
| ------------------- | -------------- |
| DATABASE_URL        | Kết nối Prisma |
| NUXT_PUBLIC_APP_URL | URL Công khai  |

---

## Các bước Thiết lập

1. `npx nuxi@latest init {{name}}`
2. `cd {{name}}`
3. `npm install @pinia/nuxt @prisma/client prisma zod`
4. `npm install -D @nuxtjs/tailwindcss`
5. Thêm modules vào `nuxt.config.ts`:
   ```ts
   modules: ["@pinia/nuxt", "@nuxtjs/tailwindcss"];
   ```
6. `npx prisma init`
7. Cấu hình schema
8. `npx prisma db push`
9. `npm run dev`

---

## Thực hành Tốt nhất

- Sử dụng `<script setup>` cho các thành phần
- Composables cho logic tái sử dụng
- Pinia stores trong thư mục `stores/`
- Server routes cho logic API
- Tự động nhập để mã nguồn sạch sẽ
- TypeScript cho an toàn kiểu
- Xem `@[skills/vue-expert]` để biết các mẫu Vue
