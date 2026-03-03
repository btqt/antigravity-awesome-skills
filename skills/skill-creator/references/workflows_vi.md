# Các Pattern của Workflow (Workflow Patterns)

## Workflow tuần tự (Sequential Workflows)

Đối với các tác vụ phức tạp, hãy chia nhỏ các hoạt động thành các bước tuần tự, rõ ràng. Thường sẽ hữu ích nếu cung cấp cho Claude một cái nhìn tổng quan về quy trình ở phần đầu của SKILL.md:

```markdown
Điền vào một form PDF bao gồm các bước sau:

1. Phân tích form (chạy analyze_form.py)
2. Tạo field mapping (chỉnh sửa fields.json)
3. Validate mapping (chạy validate_fields.py)
4. Điền vào form (chạy fill_form.py)
5. Xác minh kết quả (run verify_output.py)
```

## Workflow có điều kiện (Conditional Workflows)

Đối với các tác vụ có logic rẽ nhánh, hãy hướng dẫn Claude qua các điểm quyết định:

```markdown
1. Xác định loại sửa đổi:
   **Đang tạo nội dung mới?** → Làm theo "Creation workflow" bên dưới
   **Đang sửa đổi nội dung hiện có?** → Làm theo "Editing workflow" bên dưới

2. Creation workflow: [các bước]
3. Editing workflow: [các bước]
```
