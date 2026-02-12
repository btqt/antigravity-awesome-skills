---
name: address-github-comments
description: Sử dụng khi bạn cần xử lý các nhận xét đánh giá (review) hoặc nhận xét vấn đề (issue) trên một Pull Request GitHub đang mở bằng cách sử dụng gh CLI.
---

# Giải quyết các nhận xét trên GitHub

## Tổng quan

Giải quyết hiệu quả các nhận xét đánh giá PR hoặc phản hồi vấn đề bằng GitHub CLI (`gh`). Kỹ năng này đảm bảo tất cả các phản hồi được xử lý một cách có hệ thống.

## Điều kiện tiên quyết

Đảm bảo `gh` đã được xác thực.

```bash
gh auth status
```

Nếu chưa đăng nhập, hãy chạy `gh auth login`.

## Quy trình công việc

### 1. Kiểm tra các nhận xét

Lấy các nhận xét cho PR của nhánh hiện tại.

```bash
gh pr view --comments
```

Hoặc sử dụng một tập lệnh tùy chỉnh nếu có để liệt kê các luồng (threads) nhận xét.

### 2. Phân loại và lập kế hoạch

- Liệt kê các nhận xét và các luồng đánh giá.
- Đề xuất cách khắc phục cho mỗi nhận xét.
- **Đợi xác nhận của người dùng** về những nhận xét nào cần xử lý trước nếu có nhiều nhận xét.

### 3. Áp dụng các thay đổi

Áp dụng các thay đổi mã cho các nhận xét đã chọn.

### 4. Phản hồi các nhận xét

Sau khi đã sửa xong, hãy phản hồi các luồng nhận xét là đã giải quyết.

```bash
gh pr comment <PR_NUMBER> --body "Đã được xử lý trong commit mới nhất."
```

## Các lỗi thường gặp

- **Áp dụng các thay đổi mà không hiểu ngữ cảnh**: Luôn đọc mã xung quanh của một nhận xét.
- **Không xác minh xác thực**: Kiểm tra `gh auth status` trước khi bắt đầu.
