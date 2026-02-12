---
name: monorepo-turborepo
description: Nguyên tắc mẫu Turborepo monorepo. pnpm workspaces, các gói chia sẻ.
---

# Mẫu Turborepo Monorepo

## Tech Stack

| Thành phần        | Công nghệ                |
| ----------------- | ------------------------ |
| Hệ thống Xây dựng | Turborepo                |
| Quản lý Gói       | pnpm                     |
| Ứng dụng          | Next.js, Express         |
| Gói (Packages)    | Shared UI, Config, Types |
| Ngôn ngữ          | TypeScript               |

---

## Cấu trúc Thư mục

```
project-name/
├── apps/
│   ├── web/             # Ứng dụng Next.js
│   ├── api/             # Express API
│   └── docs/            # Tài liệu
├── packages/
│   ├── ui/              # Thành phần chia sẻ
│   ├── config/          # ESLint, TS, Tailwind
│   ├── types/           # Kiểu chia sẻ (Shared types)
│   └── utils/           # Tiện ích chia sẻ
├── turbo.json
├── pnpm-workspace.yaml
└── package.json
```

---

## Các Khái niệm Chính

| Khái niệm  | Mô tả                       |
| ---------- | --------------------------- |
| Workspaces | pnpm-workspace.yaml         |
| Pipeline   | Đồ thị tác vụ turbo.json    |
| Caching    | Caching tác vụ từ xa/cục bộ |
| Phụ thuộc  | `workspace:*` protocol      |

---

## Đường ống Turbo (Turbo Pipeline)

| Tác vụ | Phụ thuộc vào                       |
| ------ | ----------------------------------- |
| build  | ^build (các phụ thuộc trước)        |
| dev    | cache: false, liên tục (persistent) |
| lint   | ^build                              |
| test   | ^build                              |

---

## Các bước Thiết lập

1. Tạo thư mục gốc
2. `pnpm init`
3. Tạo pnpm-workspace.yaml
4. Tạo turbo.json
5. Thêm các ứng dụng và gói
6. `pnpm install`
7. `pnpm dev`

---

## Các Lệnh Phổ biến

| Lệnh                                | Mô tả                       |
| ----------------------------------- | --------------------------- |
| `pnpm dev`                          | Chạy tất cả ứng dụng        |
| `pnpm build`                        | Xây dựng tất cả             |
| `pnpm --filter @name/web dev`       | Chạy ứng dụng cụ thể        |
| `pnpm --filter @name/web add axios` | Thêm phụ thuộc vào ứng dụng |

---

## Thực hành Tốt nhất

- Các cấu hình chia sẻ trong packages/config
- Các kiểu chia sẻ trong packages/types
- Các gói nội bộ với `workspace:*`
- Sử dụng Turbo remote caching cho CI
