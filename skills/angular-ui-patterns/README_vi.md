# Các mẫu UI Angular (Angular UI Patterns)

Các mẫu UI hiện đại để xây dựng các ứng dụng Angular mạnh mẽ được tối ưu hóa cho các tác nhân AI và LLM.

## Tổng quan

Kỹ năng này bao gồm các mẫu UI thiết yếu cho:

- **Trạng thái Tải (Loading States)** - Cây quyết định giữa Skeleton và Spinner
- **Xử lý Lỗi** - Phân cấp ranh giới lỗi và khôi phục
- **Tiết lộ Lũy tiến (Progressive Disclosure)** - Sử dụng `@defer` để kết xuất lười (lazy rendering)
- **Hiển thị Dữ liệu** - Xử lý các trạng thái trống, đang tải và lỗi
- **Các mẫu Form** - Trạng thái gửi và phản hồi xác thực
- **Các mẫu Dialog/Modal** - Quản lý vòng đời dialog đúng cách

## Các Nguyên tắc Cốt lõi

1. **Không bao giờ hiển thị UI cũ (stale UI)** - Chỉ hiển thị trạng thái đang tải khi không có dữ liệu
2. **Hiển thị tất cả các lỗi** - Không bao giờ để lỗi xảy ra trong im lặng
3. **Cập nhật lạc quan (Optimistic updates)** - Cập nhật UI trước khi máy chủ xác nhận
4. **Tiết lộ lũy tiến** - Sử dụng `@defer` để tải nội dung không quan trọng
5. **Giảm cấp mượt mà (Graceful degradation)** - Dự phòng cho các tính năng bị lỗi

## Cấu trúc

Tệp `SKILL.md` bao gồm:

1. **Quy tắc Vàng** - Các mẫu không thể thương lượng cần tuân theo
2. **Cây Quyết định** - Khi nào sử dụng skeleton so với spinner
3. **Ví dụ Mã** - Các triển khai đúng so với sai
4. **Anti-patterns** - Các lỗi phổ biến cần tránh

## Tham khảo Nhanh

```html
<!-- Mẫu template Angular cho các trạng thái dữ liệu -->
@if (error()) {
<app-error-state [error]="error()" (retry)="load()" />
} @else if (loading() && !data()) {
<app-skeleton-state />
} @else if (!data()?.length) {
<app-empty-state message="Không tìm thấy mục nào" />
} @else {
<app-data-display [data]="data()" />
}
```

## Phiên bản

Phiên bản hiện tại: 1.0.0 (Tháng 2 năm 2026)

## Tham khảo

- [Angular @defer](https://angular.dev/guide/defer)
- [Angular Templates](https://angular.dev/guide/templates)
