---
name: prompt-engineering
description: "Hướng dẫn chuyên sâu về các pattern prompt engineering, best practices, và các kỹ thuật tối ưu hóa (optimization). Sử dụng khi người dùng muốn cải thiện prompt, học các chiến lược prompting, hoặc debug hành vi của Agent."
risk: unknown
source: community
date_added: "2026-02-27"
---

# Prompt Engineering Patterns

Các kỹ thuật prompt engineering nâng cao để tối đa hóa hiệu suất (performance), độ tin cậy, và khả năng kiểm soát của LLM.

## Các chức năng cốt lõi (Core Capabilities)

### 1. Few-Shot Learning

Dạy model bằng cách đưa ra các ví dụ thay vì giải thích các quy tắc. Bao gồm 2-5 cặp input-output để minh họa hành vi mong muốn. Sử dụng khi bạn cần định dạng nhất quán, các mô hình suy luận cụ thể, hoặc việc xử lý các edge cases. Nhiều ví dụ hơn sẽ cải thiện độ chính xác nhưng tiêu tốn token—hãy cân bằng dựa trên độ phức tạp của task.

**Ví dụ:**

```markdown
Extract key information from support tickets:

Input: "My login doesn't work and I keep getting error 403"
Output: {"issue": "authentication", "error_code": "403", "priority": "high"}

Input: "Feature request: add dark mode to settings"
Output: {"issue": "feature_request", "error_code": null, "priority": "low"}

Now process: "Can't upload files larger than 10MB, getting timeout"
```

### 2. Chain-of-Thought Prompting

Yêu cầu quy trình suy luận từng bước trước khi đưa ra câu trả lời cuối cùng. Thêm "Let's think step by step" (zero-shot) hoặc bao gồm các dấu vết suy luận ví dụ (few-shot). Sử dụng cho các vấn đề phức tạp đòi hỏi logic nhiều bước, suy luận toán học, hoặc khi bạn cần xác minh quá trình suy nghĩ của model. Cải thiện độ chính xác trên các analytical tasks từ 30-50%.

**Ví dụ:**

```markdown
Analyze this bug report and determine root cause.

Think step by step:

1. What is the expected behavior?
2. What is the actual behavior?
3. What changed recently that could cause this?
4. What components are involved?
5. What is the most likely root cause?

Bug: "Users can't save drafts after the cache update deployed yesterday"
```

### 3. Prompt Optimization

Cải thiện các prompt một cách có hệ thống thông qua thử nghiệm (testing) và tinh chỉnh. Bắt đầu đơn giản, đo lường performance (độ chính xác, tính nhất quán, lượng token sử dụng), sau đó lặp lại (iterate). Thử nghiệm trên các input đa dạng bao gồm cả edge cases. Sử dụng A/B testing để so sánh các biến thể. Cực kỳ quan trọng đối với các prompt trên môi trường production, nơi tính nhất quán và chi phí là những yếu tố đáng lưu tâm.

**Ví dụ:**

```markdown
Version 1 (Simple): "Summarize this article"
→ Result: Inconsistent length, misses key points

Version 2 (Add constraints): "Summarize in 3 bullet points"
→ Result: Better structure, but still misses nuance

Version 3 (Add reasoning): "Identify the 3 main findings, then summarize each"
→ Result: Consistent, accurate, captures key information
```

### 4. Template Systems

Xây dựng các cấu trúc prompt có thể tái sử dụng với các biến, các phần có điều kiện (conditional sections), và các modular components. Sử dụng cho các cuộc hội thoại nhiều lượt (multi-turn conversations), tương tác dựa trên role, hoặc khi áp dụng cùng một pattern cho các inputs khác nhau. Giảm thiểu sự trùng lặp và đảm bảo tính nhất quán trên các task tương tự nhau.

**Ví dụ:**

```python
# Reusable code review template
template = """
Review this {language} code for {focus_area}.

Code:
{code_block}

Provide feedback on:
{checklist}
"""

# Usage
prompt = template.format(
    language="Python",
    focus_area="security vulnerabilities",
    code_block=user_code,
    checklist="1. SQL injection\n2. XSS risks\n3. Authentication"
)
```

### 5. System Prompt Design

Thiết lập hành vi tổng thể và các ràng buộc duy trì xuyên suốt cuộc hội thoại. Định nghĩa role của model, mức độ chuyên môn, định dạng output, và các nguyên tắc an toàn (safety guidelines). Sử dụng system prompts cho các chỉ dẫn ổn định không thay đổi sau mỗi lượt, giải phóng lượng token của tin nhắn người dùng dành cho nội dung biến đổi (variable content).

**Ví dụ:**

```markdown
System: You are a senior backend engineer specializing in API design.

Rules:

- Always consider scalability and performance
- Suggest RESTful patterns by default
- Flag security concerns immediately
- Provide code examples in Python
- Use early return pattern

Format responses as:

1. Analysis
2. Recommendation
3. Code example
4. Trade-offs
```

## Key Patterns

### Progressive Disclosure

Bắt đầu với các prompt đơn giản, chỉ thêm mức độ phức tạp khi cần thiết:

1. **Level 1**: Chỉ dẫn trực tiếp (Direct instruction)
   - "Summarize this article"

2. **Level 2**: Thêm ràng buộc (Add constraints)
   - "Summarize this article in 3 bullet points, focusing on key findings"

3. **Level 3**: Thêm quá trình suy luận (Add reasoning)
   - "Read this article, identify the main findings, then summarize in 3 bullet points"

4. **Level 4**: Thêm ví dụ (Add examples)
   - Bao gồm 2-3 ví dụ tóm tắt kèm theo các cặp input-output

### Instruction Hierarchy

```
[System Context] → [Task Instruction] → [Examples] → [Input Data] → [Output Format]
```

### Error Recovery

Xây dựng các prompt có khả năng xử lý thất bại một cách nhẹ nhàng:

- Bao gồm các fallback instructions
- Yêu cầu các điểm số về độ tự tin (confidence scores)
- Yêu cầu các cách diễn giải thay thế (alternative interpretations) khi không chắc chắn
- Chỉ định cách báo hiệu việc thiếu thông tin

## Best Practices

1. **Cụ thể (Be Specific)**: Các prompt mơ hồ tạo ra kết quả không nhất quán
2. **Minh họa, không chỉ diễn giải (Show, Don't Tell)**: Các ví dụ thường mang lại hiệu quả cao hơn là các phần mô tả
3. **Test Extensively**: Đánh giá dựa trên các input đa dạng, mang tính đại diện
4. **Iterate Rapidly**: Những thay đổi nhỏ cũng có thể mang lại tác động lớn
5. **Monitor Performance**: Theo dõi các metrics trên môi trường production
6. **Version Control**: Hãy đối xử với các prompt giống như code bằng cách thiết lập versioning thích hợp
7. **Document Intent**: Giải thích mục đích đằng sau tại sao các prompt lại được cấu trúc như vậy

## Common Pitfalls

- **Over-engineering**: Bắt đầu ngay với các prompt phức tạp thay vì thử nghiệm các prompt đơn giản trước
- **Example pollution**: Sử dụng các ví dụ không hoàn toàn khớp với target task
- **Context overflow**: Vượt qua giới hạn token limit với số lượng lớn ví dụ dư thừa
- **Ambiguous instructions**: Chừa lại khoảng hờ khiến model có thể hiểu và diễn giải theo nhiều cách khác nhau
- **Ignoring edge cases**: Không kiểm thử testing trên những tập input dị biệt hoặc nằm trong giới hạn ranh giới (boundary inputs)

## Khi nào nên sử dụng (When to Use)

Skill này được áp dụng để thực thi các workflow hoặc các action được mô tả trong phần tổng quan.
