---
name: nextjs-static
description: Nguyên tắc mẫu trang tĩnh Next.js. Landing pages, portfolios, marketing.
---

# Mẫu Trang Tĩnh Next.js

## Tech Stack

| Thành phần | Công nghệ                              |
| ---------- | -------------------------------------- |
| Framework  | Next.js 14 (Xuất tĩnh - Static Export) |
| Ngôn ngữ   | TypeScript                             |
| Styling    | Tailwind CSS                           |
| Hoạt hình  | Framer Motion                          |
| Biểu tượng | Lucide React                           |
| SEO        | Next SEO                               |

---

## Cấu trúc Thư mục

```
project-name/
├── src/
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx      # Landing
│   │   ├── about/
│   │   ├── contact/
│   │   └── blog/
│   ├── components/
│   │   ├── layout/       # Header, Footer
│   │   ├── sections/     # Hero, Features, CTA
│   │   └── ui/
│   └── lib/
├── content/              # Nội dung Markdown
├── public/
└── next.config.js
```

---

## Cấu hình Xuất tĩnh (Static Export)

```javascript
// next.config.js
const nextConfig = {
  output: "export",
  images: { unoptimized: true },
  trailingSlash: true,
};
```

---

## Các Phần của Trang Đích

| Phần         | Mục đích                |
| ------------ | ----------------------- |
| Hero         | Dòng tiêu đề chính, CTA |
| Features     | Lợi ích sản phẩm        |
| Testimonials | Bằng chứng xã hội       |
| Pricing      | Các gói giá             |
| CTA          | Chuyển đổi cuối cùng    |

---

## Các Mẫu Hoạt hình

| Mẫu           | Sử dụng                              |
| ------------- | ------------------------------------ |
| Fade up       | Nội dung xuất hiện từ dưới lên       |
| Stagger       | Các mục danh sách xuất hiện lần lượt |
| Scroll reveal | Khi cuộn vào vùng nhìn thấy          |
| Hover         | Phản hồi tương tác                   |

---

## Các bước Thiết lập

1. `npx create-next-app {{name}} --typescript --tailwind --app`
2. Cài đặt: `npm install framer-motion lucide-react next-seo`
3. Cấu hình xuất tĩnh
4. Tạo các phần (sections)
5. `npm run dev`

---

## Triển khai

| Nền tảng        | Phương pháp           |
| --------------- | --------------------- |
| Vercel          | Tự động               |
| Netlify         | Tự động               |
| GitHub Pages    | nhánh gh-pages        |
| Bất kỳ host nào | Tải lên thư mục `out` |

---

## Thực hành Tốt nhất

- Xuất tĩnh để đạt hiệu năng tối đa
- Framer Motion cho các hoạt hình cao cấp
- Thiết kế mobile-first responsive
- Metadata SEO trên mọi trang
