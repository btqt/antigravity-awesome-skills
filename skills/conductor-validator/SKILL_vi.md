---
name: conductor-validator
description: Xác thực các tạo tác dự án Conductor về tính đầy đủ, nhất quán và chính xác. Sử dụng sau khi thiết lập, khi chẩn đoán sự cố, hoặc trước khi triển khai để xác minh bối cảnh dự án.
allowed-tools: Read Glob Grep Bash
metadata:
  model: opus
  color: cyan
---

# Kiểm tra xem thư mục conductor có tồn tại không

ls -la conductor/

# Tìm tất cả các thư mục theo dõi

ls -la conductor/tracks/

# Kiểm tra các tệp bắt buộc

ls conductor/index.md conductor/product.md conductor/tech-stack.md conductor/workflow.md conductor/tracks.md

```

## Sử dụng kỹ năng này khi

- Làm việc trên các nhiệm vụ hoặc quy trình làm việc kiểm tra xem thư mục conductor có tồn tại không
- Cần hướng dẫn, thực tiễn tốt nhất hoặc danh sách kiểm tra để kiểm tra xem thư mục conductor có tồn tại không

## Không sử dụng kỹ năng này khi

- Nhiệm vụ không liên quan đến việc kiểm tra xem thư mục conductor có tồn tại không
- Bạn cần một miền hoặc công cụ khác ngoài phạm vi này

## Hướng dẫn

- Làm rõ mục tiêu, ràng buộc và đầu vào cần thiết.
- Áp dụng các thực tiễn tốt nhất có liên quan và xác thực kết quả.
- Cung cấp các bước có thể hành động và xác minh.
- Nếu cần các ví dụ chi tiết, hãy mở `resources/implementation-playbook.md`.

## Khớp Mẫu (Pattern Matching)

**Dấu hiệu trạng thái trong tracks.md:**

```

- [ ] Tên Theo dõi # Chưa bắt đầu
- [~] Tên Theo dõi # Đang tiến hành
- [x] Tên Theo dõi # Hoàn thành

```

**Dấu hiệu nhiệm vụ trong plan.md:**

```

- [ ] Mô tả nhiệm vụ # Chờ xử lý
- [~] Mô tả nhiệm vụ # Đang tiến hành
- [x] Mô tả nhiệm vụ # Hoàn thành

```

**Mẫu ID Theo dõi:**

```

<type>_<name>_<YYYYMMDD>
Ví dụ: feature_user_auth_20250115

```

```
