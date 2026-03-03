---
name: prompt-engineering-patterns
description: "Làm chủ các kỹ thuật prompt engineering nâng cao để tối đa hóa performance, độ tin cậy và khả năng kiểm soát của LLM trong môi trường production. Sử dụng khi tối ưu hóa prompt, cải thiện output của LLM, hoặc thiết kế production..."
risk: unknown
source: community
date_added: "2026-02-27"
---

# Prompt Engineering Patterns

Làm chủ các kỹ thuật prompt engineering nâng cao để tối đa hóa performance, độ tin cậy và khả năng kiểm soát của LLM.

## Không sử dụng skill này khi (Do not use this skill when)

- Task không liên quan đến prompt engineering patterns
- Bạn cần một domain hoặc tool khác nằm ngoài phạm vi này

## Hướng dẫn (Instructions)

- Làm rõ mục tiêu, các constraint và các input cần thiết.
- Áp dụng các best practice phù hợp và validate kết quả đầu ra.
- Cung cấp các bước actionable và cách thức verification.
- Nếu cần các ví dụ chi tiết, hãy mở `resources/implementation-playbook.md`.

## Sử dụng skill này khi (Use this skill when)

- Design các prompt phức tạp cho các ứng dụng LLM trên production
- Tối ưu hóa performance và độ nhất quán của prompt
- Triển khai các pattern suy luận có cấu trúc (chain-of-thought, tree-of-thought)
- Xây dựng các hệ thống few-shot learning với khả năng chọn lọc ví dụ linh hoạt (dynamic example selection)
- Tạo ra các template prompt có thể tái sử dụng với tính năng nội suy biến (variable interpolation)
- Debugging và tinh chỉnh các prompt tạo ra output không nhất quán
- Triển khai system prompts cho các AI assistant chuyên biệt

## Các chức năng cốt lõi (Core Capabilities)

### 1. Few-Shot Learning

- Các chiến lược example selection (tương đồng ngữ nghĩa, lấy mẫu đa dạng)
- Cân bằng example count với các constraint của context window
- Xây dựng các demonstration hiệu quả với các cặp input-output
- Truy xuất ví dụ linh hoạt (Dynamic example retrieval) từ các knowledge base
- Xử lý các edge cases thông qua việc chọn loc ví dụ mang tính chiến lược

### 2. Chain-of-Thought Prompting

- Khơi gợi khả năng suy luận step-by-step
- Zero-shot CoT với câu lệnh "Let's think step by step"
- Few-shot CoT với các dấu vết suy luận
- Các kỹ thuật tự nhất quán (self-consistency techniques) (sampling nhiều luồng suy luận)
- Các bước verification và validation

### 3. Prompt Optimization

- Tiến trình làm việc lặp lại tinh chỉnh (Iterative refinement workflows)
- A/B testing các biến thể prompt
- Đo lường các metric performance của prompt (độ chính xác, độ nhất quán, latency)
- Giảm thiểu token usage trong khi vẫn giữ nguyên chất lượng
- Xử lý các edge cases và các tình huống lỗi (failure modes)

### 4. Template Systems

- Variable interpolation và định dạng format
- Các conditional prompt section
- Multi-turn conversation templates
- Xây dựng prompt dựa trên role
- Các thành phần prompt mang tính modular

### 5. System Prompt Design

- Thiết lập hành vi của model và các ràng buộc (constraints)
- Định nghĩa định dạng và cấu trúc output
- Thiết lập role và expertise chuyên môn
- Các guideline an toàn và chính sách nội dung (content policies)
- Cài đặt context và thông tin background

## Bắt đầu nhanh (Quick Start)

```python
from prompt_optimizer import PromptTemplate, FewShotSelector

# Define a structured prompt template
template = PromptTemplate(
    system="You are an expert SQL developer. Generate efficient, secure SQL queries.",
    instruction="Convert the following natural language query to SQL:\n{query}",
    few_shot_examples=True,
    output_format="SQL code block with explanatory comments"
)

# Configure few-shot learning
selector = FewShotSelector(
    examples_db="sql_examples.jsonl",
    selection_strategy="semantic_similarity",
    max_examples=3
)

# Generate optimized prompt
prompt = template.render(
    query="Find all users who registered in the last 30 days",
    examples=selector.select(query="user registration date filter")
)
```

## Các Pattern chính (Key Patterns)

### Progressive Disclosure (Tiết lộ lũy tiến)

Bắt đầu với các prompt đơn giản, chỉ thêm mức độ phức tạp khi cần thiết:

1. **Level 1**: Chỉ dẫn trực tiếp (Direct instruction)
   - "Summarize this article"

2. **Level 2**: Thêm ràng buộc (Add constraints)
   - "Summarize this article in 3 bullet points, focusing on key findings"

3. **Level 3**: Thêm quá trình suy luận (Add reasoning)
   - "Read this article, identify the main findings, then summarize in 3 bullet points"

4. **Level 4**: Thêm ví dụ (Add examples)
   - Bao gồm 2-3 ví dụ tóm tắt kèm theo các cặp input-output

### Cây cấu trúc chỉ dẫn (Instruction Hierarchy)

```
[System Context] → [Task Instruction] → [Examples] → [Input Data] → [Output Format]
```

### Phục hồi lỗi (Error Recovery)

Xây dựng các prompt có khả năng xử lý thất bại một cách nhẹ nhàng:

- Bao gồm các fallback instructions
- Yêu cầu các điểm số về độ tự tin (confidence scores)
- Yêu cầu các alternative interpretations (diễn giải thay thế) khi không chắc chắn
- Chỉ định cách báo hiệu khi thiếu thông tin

## Best Practices

1. **Cụ thể (Be Specific)**: Các prompt mơ hồ tạo ra kết quả không nhất quán
2. **Minh họa, không chỉ diễn giải (Show, Don't Tell)**: Các ví dụ thường mang lại hiệu quả cao hơn là các phần mô tả
3. **Test Extensively**: Đánh giá dựa trên các input đa dạng, đại diện
4. **Iterate Rapidly**: Những thay đổi nhỏ mang lại tác động lớn
5. **Monitor Performance**: Theo dõi các metric trên môi trường production
6. **Version Control**: Đối xử với các prompt giống như code với versioning chuẩn chỉnh
7. **Document Intent**: Giải thích mục đích đằng sau tại sao các prompt lại được cấu trúc như vậy

## Những Cạm bẫy Thường gặp (Common Pitfalls)

- **Over-engineering**: Bắt đầu ngay với các prompt phức tạp thay vì thử nghiệm các prompt đơn giản trước
- **Example pollution**: Sử dụng các ví dụ không khớp với target task
- **Context overflow**: Vượt qua token limits do quá nhiều ví dụ dư thừa
- **Ambiguous instructions**: Chừa lại khoảng hờ khiến model diễn giải theo nhiều cách
- **Ignoring edge cases**: Không kiểm thử testing trên input dị biệt hoặc input thuộc phần biên (boundary)

## Integration Patterns

### Với RAG Systems

```python
# Combine retrieved context with prompt engineering
prompt = f"""Given the following context:
{retrieved_context}

{few_shot_examples}

Question: {user_question}

Provide a detailed answer based solely on the context above. If the context doesn't contain enough information, explicitly state what's missing."""
```

### Với Validation

```python
# Add self-verification step
prompt = f"""{main_task_prompt}

After generating your response, verify it meets these criteria:
1. Answers the question directly
2. Uses only information from provided context
3. Cites specific sources
4. Acknowledges any uncertainty

If verification fails, revise your response."""
```

## Performance Optimization

### Token Efficiency (Hiệu quả Token)

- Loại bỏ các từ và cụm từ dư thừa
- Sử dụng nhất quán các từ viết tắt sau lần định nghĩa đầu tiên
- Gộp chung các instruction tương tự
- Chuyển các nội dung ổn định sang system prompts

### Latency Reduction (Giảm độ trễ)

- Tối thiểu hóa độ dài của prompt mà không làm giảm chất lượng
- Sử dụng cơ chế streaming cho các output dài
- Chạy cache đối chiếu cho các prompt prefixes phổ biến
- Batch các request tương tự với nhau khi có thể

## Tài nguyên (Resources)

- **references/few-shot-learning.md**: Tìm hiểu sâu vào cách chọn lọc ví dụ và xây dựng
- **references/chain-of-thought.md**: Các kỹ thuật khơi gợi suy luận logic nâng cao
- **references/prompt-optimization.md**: Tiến trình luân hồi đánh bóng prompt
- **references/prompt-templates.md**: Các pattern cho template tải sử dụng
- **references/system-prompts.md**: Thiết kế prompt ở cấp độ System
- **assets/prompt-template-library.md**: Thư viện prompt templates thực chiến
- **assets/few-shot-examples.json**: Bộ dataset ví dụ được tinh tuyển
- **scripts/optimize-prompt.py**: Tool tối ưu hóa prompt tự động

## Thước đo Thành công (Success Metrics)

Theo dõi các KPI sau cho prompt của bạn:

- **Accuracy**: Tính chính xác của output
- **Consistency**: Tính tái lặp kết quả khớp qua hàng loạt input trơn mượt
- **Latency**: Thời gian phản hồi (P50, P95, P99)
- **Token Usage**: Token trung bình trên mỗi request
- **Success Rate**: Phần trăm của số lượng output hợp lệ (valid outputs)
- **User Satisfaction**: Chỉ số xếp hạng (Ratings) và feedback

## Các bước Tiếp theo (Next Steps)

1. Review lại prompt template library với những pattern phổ biến
2. Thử nghiệm với few-shot learning định hướng vào use case cụ thể của bản thân
3. Áp dụng chuẩn prompt versioning và A/B testing
4. Cài đặt các pipeline đánh giá tự động
5. Tài liệu hóa các quyết định chọn lọc đối với prompt engineering cũng như bài học rút ra
