---
name: chrome-extension
description: Nguyên tắc mẫu Tiện ích mở rộng Chrome. Manifest V3, React, TypeScript.
---

# Mẫu Tiện ích mở rộng Chrome

## Tech Stack

| Thành phần | Công nghệ          |
| ---------- | ------------------ |
| Manifest   | V3                 |
| UI         | React 18           |
| Ngôn ngữ   | TypeScript         |
| Styling    | Tailwind CSS       |
| Bundler    | Vite               |
| Lưu trữ    | Chrome Storage API |

---

## Cấu trúc Thư mục

```
project-name/
├── src/
│   ├── popup/           # Popup tiện ích mở rộng
│   ├── options/         # Trang tùy chọn
│   ├── background/      # Service worker
│   ├── content/         # Content scripts
│   ├── components/
│   ├── hooks/
│   └── lib/
│       ├── storage.ts   # Tiện ích lưu trữ Chrome
│       └── messaging.ts # Truyền tin nhắn
├── public/
│   ├── icons/
│   └── manifest.json
└── package.json
```

---

## Các Khái niệm Manifest V3

| Thành phần      | Mục đích             |
| --------------- | -------------------- |
| Service Worker  | Xử lý nền            |
| Content Scripts | Tiêm vào trang       |
| Popup           | Giao diện người dùng |
| Options Page    | Cài đặt              |

---

## Quyền hạn (Permissions)

| Quyền            | Sử dụng                |
| ---------------- | ---------------------- |
| storage          | Lưu dữ liệu người dùng |
| activeTab        | Truy cập tab hiện tại  |
| scripting        | Tiêm script            |
| host_permissions | Truy cập trang web     |

---

## Các bước Thiết lập

1. `npm create vite {{name}} -- --template react-ts`
2. Thêm Chrome types: `npm install -D @types/chrome`
3. Cấu hình Vite cho multi-entry
4. Tạo manifest.json
5. `npm run dev` (chế độ theo dõi)
6. Tải trong Chrome: `chrome://extensions` → Load unpacked

---

## Mẹo Trợ giúp Phát triển

| Nhiệm vụ                  | Phương pháp                             |
| ------------------------- | --------------------------------------- |
| Gỡ lỗi Popup              | Chuột phải icon → Inspect               |
| Gỡ lỗi Background         | Trang Tiện ích mở rộng → Service worker |
| Gỡ lỗi Content            | DevTools console trên trang             |
| Tải lại Nóng (Hot Reload) | `npm run dev` với watch                 |

---

## Thực hành Tốt nhất

- Sử dụng truyền tin nhắn an toàn kiểu
- Bọc các API Chrome trong promise
- Giảm thiểu quyền hạn
- Xử lý ngoại tuyến (offline) khéo léo
