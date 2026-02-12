# Scaffolding Dự án

> Cấu trúc thư mục và các tệp cốt lõi cho các dự án mới.

---

## Cấu trúc Next.js Full-Stack (Tối ưu hóa 2025)

```
project-name/
├── src/
│   ├── app/                        # Chỉ chứa Routes (tầng mỏng)
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── globals.css
│   │   ├── (auth)/                 # Nhóm Route - trang xác thực
│   │   │   ├── login/page.tsx
│   │   │   └── register/page.tsx
│   │   ├── (dashboard)/            # Nhóm Route - layout bảng điều khiển
│   │   │   ├── layout.tsx
│   │   │   └── page.tsx
│   │   └── api/
│   │       └── [resource]/route.ts
│   │
│   ├── features/                   # Module dựa trên tính năng
│   │   ├── auth/
│   │   │   ├── components/
│   │   │   ├── hooks/
│   │   │   ├── actions.ts          # Server Actions
│   │   │   ├── queries.ts          # Data fetching
│   │   │   └── types.ts
│   │   ├── products/
│   │   │   ├── components/
│   │   │   ├── actions.ts
│   │   │   └── queries.ts
│   │   └── cart/
│   │       └── ...
│   │
│   ├── shared/                     # Tiện ích chia sẻ
│   │   ├── components/ui/          # Thành phần UI tái sử dụng
│   │   ├── lib/                    # Utils, helpers
│   │   └── hooks/                  # Global hooks
│   │
│   └── server/                     # Mã chỉ chạy trên Server
│       ├── db/                     # Database client (Prisma)
│       ├── auth/                   # Cấu hình Auth
│       └── services/               # Tích hợp API bên ngoài
│
├── prisma/
│   ├── schema.prisma
│   ├── migrations/
│   └── seed.ts
│
├── public/
├── .env.example
├── .env.local
├── package.json
├── tailwind.config.ts
├── tsconfig.json
└── README.md
```

---

## Các Nguyên tắc Cấu trúc

| Nguyên tắc                  | Triển khai                                                                   |
| --------------------------- | ---------------------------------------------------------------------------- |
| **Cô lập tính năng**        | Mỗi tính năng trong `features/` với các thành phần, hooks, actions riêng     |
| **Phân tách Server/Client** | Mã chỉ dành cho Server trong `server/`, ngăn chặn import nhầm lẫn bên Client |
| **Routes mỏng**             | `app/` chỉ dành cho định tuyến, logic nằm trong `features/`                  |
| **Nhóm Route**              | `(groupName)/` để chia sẻ layout mà không ảnh hưởng đến URL                  |
| **Mã chia sẻ**              | `shared/` cho UI và tiện ích thực sự tái sử dụng                             |

---

## Các tệp Cốt lõi

| Tệp                    | Mục đích                                                        |
| ---------------------- | --------------------------------------------------------------- |
| `package.json`         | Các phụ thuộc (Dependencies)                                    |
| `tsconfig.json`        | TypeScript + đường dẫn đại diện (path aliases) (`@/features/*`) |
| `tailwind.config.ts`   | Cấu hình Tailwind                                               |
| `.env.example`         | Mẫu biến môi trường                                             |
| `README.md`            | Tài liệu dự án                                                  |
| `.gitignore`           | Quy tắc Git ignore                                              |
| `prisma/schema.prisma` | Lược đồ cơ sở dữ liệu                                           |

---

## Đường dẫn Đại diện (tsconfig.json)

```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"],
      "@/features/*": ["./src/features/*"],
      "@/shared/*": ["./src/shared/*"],
      "@/server/*": ["./src/server/*"]
    }
  }
}
```

---

## Khi nào dùng cái gì

| Nhu cầu                | Vị trí                        |
| ---------------------- | ----------------------------- |
| Trang/tuyến đường mới  | `app/(group)/page.tsx`        |
| Thành phần tính năng   | `features/[name]/components/` |
| Server action          | `features/[name]/actions.ts`  |
| Lấy dữ liệu            | `features/[name]/queries.ts`  |
| Nút/input tái sử dụng  | `shared/components/ui/`       |
| Truy vấn cơ sở dữ liệu | `server/db/`                  |
| Gọi API bên ngoài      | `server/services/`            |
