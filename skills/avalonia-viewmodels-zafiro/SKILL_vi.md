---
name: avalonia-viewmodels-zafiro
description: Các mẫu tạo ViewModel và Wizard tối ưu cho Avalonia sử dụng Zafiro và ReactiveUI.
---

# Avalonia ViewModel với Zafiro

Kỹ năng này cung cấp một tập hợp các thực hành tốt nhất và các mẫu (patterns) để tạo ViewModel, Wizard và quản lý điều hướng trong các ứng dụng Avalonia, tận dụng sức mạnh của **ReactiveUI** và bộ công cụ **Zafiro**.

## Các Nguyên tắc Cốt lõi

1.  **Tiếp cận Chức năng-Phản ứng (Functional-Reactive)**: Sử dụng ReactiveUI (`ReactiveObject`, `WhenAnyValue`, v.v.) để xử lý trạng thái và logic.
2.  **Lệnh nâng cao (Enhanced Commands)**: Sử dụng `IEnhancedCommand` để quản lý lệnh tốt hơn, bao gồm báo cáo tiến độ và các thuộc tính name/text.
3.  **Mẫu Wizard**: Triển khai các luồng phức tạp sử dụng `SlimWizard` và `WizardBuilder` theo cách tiếp cận khai báo và dễ bảo trì.
4.  **Tự động Khám phá Section**: Sử dụng thuộc tính `[Section]` để đăng ký và tự động khám phá các phần của giao diện người dùng.
5.  **Cấu trúc Sạch sẽ (Clean Composition)**: Ánh xạ ViewModel tới View bằng cách sử dụng `DataTypeViewLocator` và quản lý các phụ thuộc trong `CompositionRoot`.

## Các Hướng dẫn

- [ViewModels & Commands](viewmodels.md): Tạo các ViewModel mạnh mẽ và xử lý các lệnh.
- [Wizards & Flows](wizards.md): Xây dựng các wizard nhiều bước với `SlimWizard`.
- [Navigation & Sections](navigation_sections.md): Quản lý điều hướng và UI dựa trên các section.
- [Composition & Mapping](composition.md): Các thực hành tốt nhất cho việc kết nối View-ViewModel và DI.

## Ví dụ Tham khảo

Đối với các triển khai thực tế, hãy tham khảo dự án **Angor**:

- `CreateProjectFlowV2.cs`: Một ví dụ tuyệt vời về việc xây dựng Wizard phức tạp.
- `HomeViewModel.cs`: ViewModel section đơn giản sử dụng các lệnh functional-reactive.
