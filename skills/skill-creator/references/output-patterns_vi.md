# Các Pattern về Output (Output Patterns)

Sử dụng các pattern này khi các skill cần tạo ra kết quả (output) nhất quán và chất lượng cao.

## Pattern về Template (Template Pattern)

Cung cấp các template cho định dạng đầu ra (output format). Hãy điều chỉnh mức độ chặt chẽ phù hợp với nhu cầu của bạn.

**Đối với các yêu cầu nghiêm ngặt (như phản hồi API hoặc định dạng dữ liệu):**

```markdown
## Cấu trúc báo cáo (Report structure)

LUÔN LUÔN sử dụng chính xác cấu trúc template này:

# [Tiêu đề phân tích]

## Tóm tắt nội dung (Executive summary)

[Tổng quan một đoạn về các phát hiện chính]

## Các phát hiện chính (Key findings)

- Phát hiện 1 với dữ liệu hỗ trợ
- Phát hiện 2 với dữ liệu hỗ trợ
- Phát hiện 3 với dữ liệu hỗ trợ

## Các đề xuất (Recommendations)

1. Đề xuất hành động cụ thể
2. Đề xuất hành động cụ thể
```

**Đối với các hướng dẫn linh hoạt (khi việc thích nghi là hữu ích):**

```markdown
## Cấu trúc báo cáo (Report structure)

Dưới đây là một định dạng mặc định hợp lý, nhưng hãy sử dụng phán đoán tốt nhất của bạn:

# [Tiêu đề phân tích]

## Tóm tắt nội dung (Executive summary)

[Tổng quan]

## Các phát hiện chính (Key findings)

[Điều chỉnh các phần dựa trên những gì bạn khám phá được]

## Các đề xuất (Recommendations)

[Điều chỉnh cho phù hợp với ngữ cảnh cụ thể]

Điều chỉnh các phần khi cần thiết cho loại phân tích cụ thể.
```

## Pattern về Ví dụ (Examples Pattern)

Đối với các skill mà chất lượng đầu ra (output) phụ thuộc vào việc xem các ví dụ, hãy cung cấp các cặp input/output:

```markdown
## Định dạng thông điệp commit (Commit message format)

Tạo các thông điệp commit theo các ví dụ sau:

**Ví dụ 1:**
Input: Added user authentication with JWT tokens
Output:
```

feat(auth): implement JWT-based authentication

Add login endpoint and token validation middleware

```

**Ví dụ 2:**
Input: Fixed bug where dates displayed incorrectly in reports
Output:
```

fix(reports): correct date formatting in timezone conversion

Use UTC timestamps consistently across report generation

```

Tuân theo style này: type(scope): mô tả ngắn gọn, sau đó là giải thích chi tiết.
```

Các ví dụ giúp Claude hiểu phong cách và mức độ chi tiết mong muốn rõ ràng hơn là chỉ mô tả đơn thuần.
