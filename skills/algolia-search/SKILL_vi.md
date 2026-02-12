---
name: algolia-search
description: "Các mẫu chuyên gia để triển khai tìm kiếm Algolia, chiến lược lập chỉ mục, React InstantSearch và tinh chỉnh mức độ liên quan. Sử dụng khi: thêm tính năng tìm kiếm vào, algolia, instantsearch, search api, chức năng tìm kiếm."
source: vibeship-spawner-skills (Apache 2.0)
---

# Tích hợp Tìm kiếm Algolia (Algolia Search Integration)

## Các mẫu (Patterns)

### React InstantSearch với Hooks

Thiết lập React InstantSearch hiện đại sử dụng hooks cho tính năng tìm kiếm gợi ý (type-ahead search).

Sử dụng gói `react-instantsearch-hooks-web` với client `algoliasearch`.
Các Widgets là những thành phần có thể được tùy chỉnh bằng `classnames`.

Các hooks chính:

- `useSearchBox`: Xử lý ô nhập tìm kiếm
- `useHits`: Truy cập kết quả tìm kiếm
- `useRefinementList`: Lọc theo thuộc tính (Facet filtering)
- `usePagination`: Phân trang kết quả
- `useInstantSearch`: Truy cập toàn bộ trạng thái

### Kết xuất phía máy chủ (SSR) với Next.js

Tích hợp SSR cho Next.js với gói `react-instantsearch-nextjs`.

Sử dụng `<InstantSearchNext>` thay vì `<InstantSearch>` để hỗ trợ SSR.
Hỗ trợ cả Pages Router và App Router (đang thử nghiệm).

Các cân nhắc chính:

- Thiết lập `dynamic = 'force-dynamic'` để luôn có kết quả mới nhất.
- Xử lý đồng bộ hóa URL với thuộc tính `routing`.
- Sử dụng `getServerState` để lấy trạng thái ban đầu.

### Đồng bộ hóa Dữ liệu và Lập chỉ mục

Các chiến lược lập chỉ mục để giữ cho Algolia đồng bộ với dữ liệu của bạn.

Ba phương pháp chính:

1. Lập chỉ mục lại toàn bộ (Full Reindexing) - Thay thế toàn bộ chỉ mục (tốn kém).
2. Cập nhật toàn bộ bản ghi (Full Record Updates) - Thay thế từng bản ghi riêng lẻ.
3. Cập nhật một phần (Partial Updates) - Chỉ cập nhật các thuộc tính cụ thể.

Thực hành tốt nhất:

- Gom nhóm các bản ghi (lý tưởng: 10MB, 1.000 - 10.000 bản ghi mỗi nhóm).
- Sử dụng cập nhật tăng cường (incremental updates) khi có thể.
- `partialUpdateObjects` cho các thay đổi chỉ ở cấp độ thuộc tính.
- Tránh sử dụng `deleteBy` (tốn kém tài nguyên tính toán).

## ⚠️ Các điểm cần lưu ý (Sharp Edges)

| Vấn đề | Mức độ nghiêm trọng | Giải pháp    |
| ------ | ------------------- | ------------ |
| Vấn đề | Nghiêm trọng        | Xem tài liệu |
| Vấn đề | Cao                 | Xem tài liệu |
| Vấn đề | Trung bình          | Xem tài liệu |
| Vấn đề | Trung bình          | Xem tài liệu |
| Vấn đề | Trung bình          | Xem tài liệu |
| Vấn đề | Trung bình          | Xem tài liệu |
| Vấn đề | Trung bình          | Xem tài liệu |
| Vấn đề | Trung bình          | Xem tài liệu |
