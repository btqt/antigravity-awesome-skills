---
name: flutter-app
description: Nguyên tắc mẫu ứng dụng di động Flutter. Riverpod, Go Router, kiến trúc sạch.
---

# Mẫu Ứng dụng Flutter

## Tech Stack

| Thành phần              | Công nghệ    |
| ----------------------- | ------------ |
| Framework               | Flutter 3.x  |
| Ngôn ngữ                | Dart 3.x     |
| Trạng thái (State)      | Riverpod 2.0 |
| Điều hướng (Navigation) | Go Router    |
| HTTP                    | Dio          |
| Lưu trữ                 | Hive         |

---

## Cấu trúc Thư mục

```
project_name/
├── lib/
│   ├── main.dart
│   ├── app.dart
│   ├── core/
│   │   ├── constants/
│   │   ├── theme/
│   │   ├── router/
│   │   └── utils/
│   ├── features/
│   │   ├── auth/
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   └── home/
│   ├── shared/
│   │   ├── widgets/
│   │   └── providers/
│   └── services/
│       ├── api/
│       └── storage/
├── test/
└── pubspec.yaml
```

---

## Các tầng Kiến trúc

| Tầng         | Nội dung                    |
| ------------ | --------------------------- |
| Presentation | Màn hình, Widget, Providers |
| Domain       | Entities, Use Cases         |
| Data         | Repositories, Models        |

---

## Các Gói Chính

| Gói                 | Mục đích           |
| ------------------- | ------------------ |
| flutter_riverpod    | Quản lý trạng thái |
| riverpod_annotation | Tạo mã             |
| go_router           | Điều hướng         |
| dio                 | HTTP client        |
| freezed             | Mô hình bất biến   |
| hive                | Lưu trữ cục bộ     |

---

## Các bước Thiết lập

1. `flutter create {{name}} --org com.{{bundle}}`
2. Cập nhật `pubspec.yaml`
3. `flutter pub get`
4. Chạy tạo mã: `dart run build_runner build`
5. `flutter run`

---

## Thực hành Tốt nhất

- Cấu trúc thư mục ưu tiên tính năng (Feature-first)
- Riverpod cho trạng thái, mẫu React Query cho trạng thái máy chủ
- Freezed cho các lớp dữ liệu bất biến
- Go Router cho điều hướng khai báo
- Theming Material 3
