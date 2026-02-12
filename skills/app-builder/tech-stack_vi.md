# Lựa chọn Tech Stack (2025)

> Các lựa chọn công nghệ mặc định và thay thế cho các ứng dụng web.

## Stack Mặc định (Ứng dụng Web - 2025)

```yaml
Frontend:
  framework: Next.js 16 (Ổn định)
  language: TypeScript 5.7+
  styling: Tailwind CSS v4
  state: React 19 Actions / Server Components
  bundler: Turbopack (Ổn định cho Dev)

Backend:
  runtime: Node.js 23
  framework: Next.js API Routes / Hono (cho Edge)
  validation: Zod / TypeBox

Database:
  primary: PostgreSQL
  orm: Prisma / Drizzle
  hosting: Supabase / Neon

Auth:
  provider: Auth.js (v5) / Clerk

Monorepo:
  tool: Turborepo 2.0
```

## Các Tùy chọn Thay thế

| Nhu cầu                    | Mặc định | Thay thế                     |
| -------------------------- | -------- | ---------------------------- |
| Thời gian thực (Real-time) | -        | Supabase Realtime, Socket.io |
| Lưu trữ tệp                | -        | Cloudinary, S3               |
| Thanh toán                 | Stripe   | LemonSqueezy, Paddle         |
| Email                      | -        | Resend, SendGrid             |
| Tìm kiếm                   | -        | Algolia, Typesense           |
