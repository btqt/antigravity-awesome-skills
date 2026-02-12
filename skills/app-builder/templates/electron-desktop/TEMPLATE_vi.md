---
name: electron-desktop
description: Nguyên tắc mẫu ứng dụng máy tính để bàn Electron. Đa nền tảng, React, TypeScript.
---

# Mẫu Ứng dụng Máy tính để bàn Electron

## Tech Stack

| Thành phần | Công nghệ               |
| ---------- | ----------------------- |
| Framework  | Electron 28+            |
| UI         | React 18                |
| Ngôn ngữ   | TypeScript              |
| Styling    | Tailwind CSS            |
| Bundler    | Vite + electron-builder |
| IPC        | Giao tiếp an toàn kiểu  |

---

## Cấu trúc Thư mục

```
project-name/
├── electron/
│   ├── main.ts          # Tiến trình chính
│   ├── preload.ts       # Script tải trước
│   └── ipc/             # Trình xử lý IPC
├── src/
│   ├── App.tsx
│   ├── components/
│   │   ├── TitleBar.tsx # Thanh tiêu đề tùy chỉnh
│   │   └── ...
│   └── hooks/
├── public/
└── package.json
```

---

## Mô hình Tiến trình

| Tiến trình | Vai trò                    |
| ---------- | -------------------------- |
| Main       | Node.js, truy cập hệ thống |
| Renderer   | Chromium, React UI         |
| Preload    | Cầu nối, cô lập ngữ cảnh   |

---

## Các Khái niệm Chính

| Khái niệm              | Mục đích             |
| ---------------------- | -------------------- |
| contextBridge          | Phơi bày API an toàn |
| ipcMain/ipcRenderer    | Giao tiếp tiến trình |
| nodeIntegration: false | Bảo mật              |
| contextIsolation: true | Bảo mật              |

---

## Các bước Thiết lập

1. `npm create vite {{name}} -- --template react-ts`
2. Cài đặt: `npm install -D electron electron-builder vite-plugin-electron`
3. Tạo thư mục `electron/`
4. Cấu hình tiến trình chính
5. `npm run electron:dev`

---

## Các mục tiêu Build

| Nền tảng | Đầu ra         |
| -------- | -------------- |
| Windows  | NSIS, Portable |
| macOS    | DMG, ZIP       |
| Linux    | AppImage, DEB  |

---

## Thực hành Tốt nhất

- Sử dụng script tải trước cho cầu nối main/renderer
- IPC an toàn kiểu với các trình xử lý được đánh kiểu
- Thanh tiêu đề tùy chỉnh để có cảm giác native
- Xử lý trạng thái cửa sổ (maximize, minimize)
- Tự động cập nhật với electron-updater
