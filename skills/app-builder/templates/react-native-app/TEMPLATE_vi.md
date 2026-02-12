---
name: react-native-app
description: Nguyên tắc mẫu ứng dụng di động React Native. Expo, TypeScript, navigation.
---

# Mẫu Ứng dụng React Native

## Tech Stack

| Thành phần              | Công nghệ             |
| ----------------------- | --------------------- |
| Framework               | React Native + Expo   |
| Ngôn ngữ                | TypeScript            |
| Điều hướng (Navigation) | Expo Router           |
| Trạng thái (State)      | Zustand + React Query |
| Styling                 | NativeWind            |
| Kiểm thử                | Jest + RNTL           |

---

## Cấu trúc Thư mục

```
project-name/
├── app/                 # Expo Router (dựa trên tệp)
│   ├── _layout.tsx      # Root layout
│   ├── index.tsx        # Trang chủ
│   ├── (tabs)/          # Điều hướng tab
│   └── [id].tsx         # Route động
├── components/
│   ├── ui/              # Tái sử dụng
│   └── features/
├── hooks/
├── lib/
│   ├── api.ts
│   └── storage.ts
├── store/
├── constants/
└── app.json
```

---

## Các Mẫu Điều hướng

| Mẫu    | Sử dụng                    |
| ------ | -------------------------- |
| Stack  | Phân cấp trang             |
| Tabs   | Điều hướng dưới cùng       |
| Drawer | Menu bên                   |
| Modal  | Màn hình lớp phủ (Overlay) |

---

## Quản lý Trạng thái

| Loại              | Công cụ          |
| ----------------- | ---------------- |
| Cục bộ (Local)    | Zustand          |
| Máy chủ (Server)  | React Query      |
| Forms             | React Hook Form  |
| Lưu trữ (Storage) | Expo SecureStore |

---

## Các Gói Chính

| Gói                   | Mục đích                |
| --------------------- | ----------------------- |
| expo-router           | Định tuyến dựa trên tệp |
| zustand               | Trạng thái cục bộ       |
| @tanstack/react-query | Trạng thái máy chủ      |
| nativewind            | Tailwind styling        |
| expo-secure-store     | Lưu trữ an toàn         |

---

## Các bước Thiết lập

1. `npx create-expo-app {{name}} -t expo-template-blank-typescript`
2. `npx expo install expo-router react-native-safe-area-context`
3. Cài đặt state: `npm install zustand @tanstack/react-query`
4. `npx expo start`

---

## Thực hành Tốt nhất

- Expo Router cho điều hướng
- Zustand cho trạng thái cục bộ, React Query cho trạng thái máy chủ
- NativeWind cho styling nhất quán
- Expo SecureStore cho các token
- Kiểm thử trên cả iOS và Android
