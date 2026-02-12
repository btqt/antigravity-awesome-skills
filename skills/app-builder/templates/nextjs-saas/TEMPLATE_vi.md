---
name: nextjs-saas
description: Nguyên tắc mẫu Next.js SaaS. Auth, thanh toán, email.
---

# Mẫu Next.js SaaS

## Tech Stack

| Thành phần    | Công nghệ                                             |
| ------------- | ----------------------------------------------------- |
| Framework     | Next.js 14 (App Router)                               |
| Auth          | NextAuth.js v5                                        |
| Thanh toán    | Stripe                                                |
| Cơ sở dữ liệu | PostgreSQL + Prisma                                   |
| Email         | Resend                                                |
| UI            | Tailwind (HỎI NGƯỜI DÙNG: shadcn/Headless UI/Custom?) |

---

## Cấu trúc Thư mục

```
project-name/
├── prisma/
├── src/
│   ├── app/
│   │   ├── (auth)/      # Login, register
│   │   ├── (dashboard)/ # Protected routes
│   │   ├── (marketing)/ # Landing, pricing
│   │   └── api/
│   │       ├── auth/[...nextauth]/
│   │       └── webhooks/stripe/
│   ├── components/
│   │   ├── auth/
│   │   ├── billing/
│   │   └── dashboard/
│   ├── lib/
│   │   ├── auth.ts      # NextAuth config
│   │   ├── stripe.ts    # Stripe client
│   │   └── email.ts     # Resend client
│   └── config/
│       └── subscriptions.ts
└── package.json
```

---

## Các Tính năng SaaS

| Tính năng                        | Triển khai           |
| -------------------------------- | -------------------- |
| Auth                             | NextAuth + OAuth     |
| Đăng ký (Subscriptions)          | Stripe Checkout      |
| Cổng thanh toán (Billing Portal) | Stripe Portal        |
| Webhooks                         | Stripe events        |
| Email                            | Giao dịch qua Resend |

---

## Lược đồ Cơ sở dữ liệu

| Model   | Trường                                      |
| ------- | ------------------------------------------- |
| User    | id, email, stripeCustomerId, subscriptionId |
| Account | Dữ liệu OAuth provider                      |
| Session | Các phiên người dùng                        |

---

## Biến Môi trường

| Biến                  | Mục đích   |
| --------------------- | ---------- |
| DATABASE_URL          | Prisma     |
| NEXTAUTH_SECRET       | Auth       |
| STRIPE_SECRET_KEY     | Thanh toán |
| STRIPE_WEBHOOK_SECRET | Webhooks   |
| RESEND_API_KEY        | Email      |

---

## Các bước Thiết lập

1. `npx create-next-app {{name}} --typescript --tailwind --app`
2. Cài đặt: `npm install next-auth @auth/prisma-adapter stripe resend`
3. Thiết lập Stripe products/prices
4. Cấu hình môi trường
5. `npm run db:push`
6. `npm run stripe:listen` (webhooks)
7. `npm run dev`

---

## Thực hành Tốt nhất

- Nhóm Route để phân tách layout
- Stripe webhooks để đồng bộ đăng ký
- NextAuth với Prisma adapter
- Email templates với React Email
