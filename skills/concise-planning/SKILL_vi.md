---
name: concise-planning
description: Sử dụng khi người dùng yêu cầu kế hoạch cho một nhiệm vụ mã hóa, để tạo ra một danh sách kiểm tra rõ ràng, có thể hành động và nguyên tử.
---

# Lập kế hoạch Ngắn gọn

## Mục tiêu

Biến yêu cầu của người dùng thành **một kế hoạch hành động duy nhất** với các bước nguyên tử.

## Quy trình làm việc

### 1. Quét Bối cảnh

- Đọc `README.md`, tài liệu và các tệp mã có liên quan.
- Xác định các ràng buộc (ngôn ngữ, khuôn khổ, kiểm thử).

### 2. Tương tác Tối thiểu

- Hỏi **tối đa 1–2 câu hỏi** và chỉ khi thực sự chặn.
- Đưa ra các giả định hợp lý cho những ẩn số không chặn.

### 3. Tạo Kế hoạch

Sử dụng cấu trúc sau:

- **Cách tiếp cận**: 1-3 câu về cái gì và tại sao.
- **Phạm vi**: Các gạch đầu dòng cho "Trong" và "Ngoài".
- **Các mục Hành động**: Một danh sách 6-10 nhiệm vụ nguyên tử, có thứ tự (Động từ đầu tiên).
- **Xác thực**: Ít nhất một mục để kiểm thử.

## Mẫu Kế hoạch

```markdown
# Kế hoạch

<Cách tiếp cận cấp cao>

## Phạm vi

- Trong:
- Ngoài:

## Các mục Hành động

[ ] <Bước 1: Khám phá>
[ ] <Bước 2: Triển khai>
[ ] <Bước 3: Triển khai>
[ ] <Bước 4: Xác thực/Kiểm thử>
[ ] <Bước 5: Triển khai/Cam kết>

## Câu hỏi Mở

- <Câu hỏi 1 (tối đa 3)>
```

## Hướng dẫn Danh sách kiểm tra

- **Nguyên tử**: Mỗi bước phải là một đơn vị công việc logic duy nhất.
- **Động từ đầu tiên**: "Thêm...", "Tái cấu trúc...", "Xác minh...".
- **Cụ thể**: Nêu tên các tệp hoặc mô-đun cụ thể khi có thể.
