---
name: astro-static
description: Nguyên tắc mẫu trang tĩnh Astro. Web tập trung vào nội dung, blogs, tài liệu.
---

# Mẫu Trang Tĩnh Astro

## Tech Stack

| Thành phần | Công nghệ                 |
| ---------- | ------------------------- |
| Framework  | Astro 4.x                 |
| Nội dung   | MDX + Content Collections |
| Styling    | Tailwind CSS              |
| Tích hợp   | Sitemap, RSS, SEO         |
| Đầu ra     | Static/SSG                |

---

## Cấu trúc Thư mục

```
project-name/
├── src/
│   ├── components/      # .astro components
│   ├── content/         # MDX content
│   │   ├── blog/
│   │   └── config.ts    # Schemas bộ sưu tập
│   ├── layouts/         # Layouts trang
│   ├── pages/           # Định tuyến dựa trên tệp
│   └── styles/
├── public/              # Tài sản tĩnh (Static assets)
├── astro.config.mjs
└── package.json
```

---

## Các Khái niệm Chính

| Khái niệm                                 | Mô tả                                 |
| ----------------------------------------- | ------------------------------------- |
| Bộ sưu tập Nội dung (Content Collections) | Nội dung an toàn kiểu với schema Zod  |
| Kiến trúc Islands                         | Hydration một phần cho tính tương tác |
| Mặc định Zero JS                          | HTML tĩnh trừ khi cần thiết           |
| Hỗ trợ MDX                                | Markdown với các thành phần           |

---

## Các bước Thiết lập

1. `npm create astro@latest {{name}}`
2. Thêm tích hợp: `npx astro add mdx tailwind sitemap`
3. Cấu hình `astro.config.mjs`
4. Tạo các bộ sưu tập nội dung
5. `npm run dev`

---

## Triển khai

| Nền tảng         | Phương pháp           |
| ---------------- | --------------------- |
| Vercel           | Tự động phát hiện     |
| Netlify          | Tự động phát hiện     |
| Cloudflare Pages | Tự động phát hiện     |
| GitHub Pages     | Build + deploy action |

---

## Thực hành Tốt nhất

- Sử dụng Bộ sưu tập Nội dung cho an toàn kiểu
- Tận dụng tạo tĩnh (static generation)
- Thêm islands chỉ khi cần thiết
- Tối ưu hóa hình ảnh với Astro Image
