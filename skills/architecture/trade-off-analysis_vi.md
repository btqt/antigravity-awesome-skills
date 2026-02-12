# Phân tích Đánh đổi & ADR

> Ghi lại mọi quyết định kiến trúc với các đánh đổi.

## Khung Ra quyết định

Đối với MỖI thành phần kiến trúc, hãy ghi lại:

```markdown
## Hồ sơ Quyết định Kiến trúc (ADR)

### Ngữ cảnh

- **Vấn đề**: [Chúng ta đang giải quyết vấn đề gì?]
- **Ràng buộc**: [Quy mô nhóm, quy mô hệ thống, dòng thời gian, ngân sách]

### Các Tùy chọn Được xem xét

| Tùy chọn   | Ưu điểm   | Nhược điểm | Độ phức tạp | Khi nào Hợp lệ |
| ---------- | --------- | ---------- | ----------- | -------------- |
| Tùy chọn A | Lợi ích 1 | Chi phí 1  | Thấp        | [Điều kiện]    |
| Tùy chọn B | Lợi ích 2 | Chi phí 2  | Cao         | [Điều kiện]    |

### Quyết định

**Đã chọn**: [Tùy chọn B]

### Cơ sở lý luận

1. [Lý do 1 - gắn với ràng buộc]
2. [Lý do 2 - gắn với yêu cầu]

### Các Đánh đổi Được chấp nhận

- [Những gì chúng ta đang từ bỏ]
- [Tại sao điều này có thể chấp nhận được]

### Hậu quả

- **Tích cực**: [Lợi ích chúng ta đạt được]
- **Tiêu cực**: [Chi phí/rủi ro chúng ta chấp nhận]
- **Giảm nhẹ**: [Cách chúng ta giải quyết tiêu cực]

### Kích hoạt Xem xét lại

- [Khi nào xem xét lại quyết định này]
```

## Mẫu ADR

```markdown
# ADR-[XXX]: [Tiêu đề Quyết định]

## Trạng thái

Đề xuất (Proposed) | Được chấp nhận (Accepted) | Bị phản đối (Deprecated) | Bị thay thế bởi [ADR-YYY] (Superseded by)

## Ngữ cảnh

[Vấn đề là gì? Ràng buộc là gì?]

## Quyết định

[Chúng ta đã chọn gì - hãy cụ thể]

## Cơ sở lý luận

[Tại sao - gắn với yêu cầu và ràng buộc]

## Đánh đổi

[Những gì chúng ta đang từ bỏ - hãy trung thực]

## Hậu quả

- **Tích cực**: [Lợi ích]
- **Tiêu cực**: [Chi phí]
- **Giảm nhẹ**: [Cách giải quyết]
```

## Lưu trữ ADR

```
docs/
└── architecture/
    ├── adr-001-use-nextjs.md
    ├── adr-002-postgresql-over-mongodb.md
    └── adr-003-adopt-repository-pattern.md
```
