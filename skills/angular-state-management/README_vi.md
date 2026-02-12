# Quản lý Trạng thái trong Angular (Angular State Management)

Các mẫu quản lý trạng thái hoàn chỉnh cho ứng dụng Angular được tối ưu hóa cho các tác nhân AI và LLM.

## Tổng quan

Kỹ năng này cung cấp các khung ra quyết định và các mẫu triển khai cho:

- **Dịch vụ dựa trên Signal (Signal-based Services)** - Trạng thái nhẹ cho dữ liệu dùng chung
- **NgRx SignalStore** - Trạng thái phạm vi tính năng (feature-scoped) với các giá trị computed
- **NgRx Store** - Quản lý trạng thái toàn cục quy mô doanh nghiệp
- **RxJS ComponentStore** - Trạng thái cấp thành phần có tính phản ứng
- **Trạng thái Forms** - Các mẫu form reactive và template-driven

## Cấu trúc

Tệp `SKILL.md` được tổ chức thành:

1. **Các danh mục Trạng thái** - Trạng thái cục bộ, dùng chung, toàn cục, máy chủ, URL và form
2. **Tiêu chí Lựa chọn** - Cây quyết định để chọn giải pháp phù hợp
3. **Các mẫu Triển khai** - Các ví dụ hoàn chỉnh cho từng cách tiếp cận
4. **Hướng dẫn Di chuyển** - Chuyển từ BehaviorSubject sang Signals
5. **Các mẫu Kết nối (Bridging Patterns)** - Tích hợp Signals với RxJS

## Khi nào nên sử dụng mỗi Mẫu

- **Dịch vụ Signal**: Trạng thái UI dùng chung (chủ đề, sở thích người dùng)
- **NgRx SignalStore**: Trạng thái tính năng với các giá trị computed
- **NgRx Store**: Các phụ thuộc chéo tính năng phức tạp
- **ComponentStore**: Các hoạt động bất đồng bộ trong phạm vi thành phần
- **Reactive Forms**: Trạng thái form với xác thực

## Phiên bản

Phiên bản hiện tại: 1.0.0 (Tháng 2 năm 2026)

## Tham khảo

- [Angular Signals](https://angular.dev/guide/signals)
- [NgRx](https://ngrx.io)
- [NgRx SignalStore](https://ngrx.io/guide/signals)
