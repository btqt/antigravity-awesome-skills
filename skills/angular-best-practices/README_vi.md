# Thực hành Tốt nhất cho Angular (Angular Best Practices)

Tối ưu hóa hiệu suất và các thực hành tốt nhất cho ứng dụng Angular được tối ưu hóa cho các tác nhân AI và LLM.

## Tổng quan

Kỹ năng này cung cấp các hướng dẫn hiệu suất được ưu tiên trên các lĩnh vực:

- **Phát hiện Thay đổi (Change Detection)** - Chiến lược OnPush, Signals, ứng dụng Zoneless
- **Hoạt động Bất đồng bộ (Async Operations)** - Tránh hiện tượng waterfalls, SSR preloading
- **Tối ưu hóa Bundle** - Lazy loading, `@defer`, tree-shaking
- **Hiệu suất Kết xuất (Rendering Performance)** - TrackBy, virtual scrolling, CDK
- **SSR & Hydration** - Các mẫu kết xuất phía máy chủ
- **Tối ưu hóa Template** - Các chỉ thị cấu trúc (structural directives), pipe memoization
- **Quản lý Trạng thái** - Các mẫu phản ứng hiệu quả
- **Quản lý Bộ nhớ** - Dọn dẹp Subscription, detached refs

## Cấu trúc

Tệp `SKILL.md` được tổ chức theo mức độ ưu tiên:

1. **Ưu tiên Tối quan trọng (Critical Priority)** - Cải thiện hiệu suất lớn nhất (phát hiện thay đổi, bất đồng bộ)
2. **Ưu tiên Cao (High Priority)** - Tác động đáng kể (bundles, kết xuất)
3. **Ưu tiên Trung bình (Medium Priority)** - Cải thiện đáng chú ý (SSR, templates)
4. **Ưu tiên Thấp (Low Priority)** - Cải thiện dần dần (bộ nhớ, dọn dẹp)

Mỗi quy tắc bao gồm:

- ❌ **SAI (WRONG)** - Những gì không nên làm
- ✅ **ĐÚNG (CORRECT)** - Mẫu được đề xuất
- 📝 **Tại sao (Why)** - Giải thích về tác động

## Danh sách Kiểm tra Tham khảo Nhanh

**Cho các Component mới:**

- [ ] Sử dụng `ChangeDetectionStrategy.OnPush`
- [ ] Sử dụng Signals cho trạng thái phản ứng
- [ ] Sử dụng `@defer` cho nội dung không quan trọng
- [ ] Sử dụng `trackBy` cho các vòng lặp `*ngFor`
- [ ] Không có subscription nào mà không được dọn dẹp

**Cho các Đánh giá Hiệu suất:**

- [ ] Không có async waterfalls (lấy dữ liệu song song)
- [ ] Các tuyến đường (routes) được tải chậm (lazy-loaded)
- [ ] Các thư viện lớn được tách mã (code-split)
- [ ] Hình ảnh sử dụng `NgOptimizedImage`

## Phiên bản

Phiên bản hiện tại: 1.0.0 (Tháng 2 năm 2026)

## Tham khảo

- [Hiệu suất Angular](https://angular.dev/guide/performance)
- [Zoneless Angular](https://angular.dev/guide/zoneless)
- [Angular SSR](https://angular.dev/guide/ssr)
