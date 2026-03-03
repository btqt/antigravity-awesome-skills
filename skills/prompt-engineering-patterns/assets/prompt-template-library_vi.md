# Prompt Template Library

## Classification Templates (Các Template phân loại)

### Sentiment Analysis

```
Phân loại sentiment của văn bản sau thành Tích cực (Positive), Tiêu cực (Negative), hoặc Trung lập (Neutral).

Text: {text}

Sentiment:
```

### Intent Detection

```
Xác định intent của người dùng từ tin nhắn sau.

Các intent có thể: {intent_list}

Message: {message}

Intent:
```

### Topic Classification

```
Phân loại bài báo sau vào một trong các category này: {categories}

Article:
{article}

Category:
```

## Extraction Templates (Các Template trích xuất)

### Named Entity Recognition

```
Trích xuất tất cả các named entities từ văn bản và phân loại chúng.

Text: {text}

Entities (JSON format):
{
  "persons": [],
  "organizations": [],
  "locations": [],
  "dates": []
}
```

### Structured Data Extraction

```
Trích xuất thông tin có cấu trúc (structured information) từ mẩu tin tuyển dụng.

Job Posting:
{posting}

Extracted Information (JSON):
{
  "title": "",
  "company": "",
  "location": "",
  "salary_range": "",
  "requirements": [],
  "responsibilities": []
}
```

## Generation Templates (Các Template tạo nội dung)

### Email Generation

```
Viết một email {email_type} chuyên nghiệp.

To: {recipient}
Context: {context}
Các điểm chính cần bao gồm:
{key_points}

Email:
Subject:
Body:
```

### Code Generation

```
Generate {language} code cho task sau:

Task: {task_description}

Requirements:
{requirements}

Bao gồm:
- Error handling
- Input validation
- Inline comments

Code:
```

### Creative Writing

```
Viết một câu chuyện {style} dài {length} từ về {topic}.

Bao gồm các yếu tố này:
- {element_1}
- {element_2}
- {element_3}

Story:
```

## Transformation Templates (Các Template chuyển đổi)

### Summarization

```
Tóm tắt văn bản sau trong {num_sentences} câu.

Text:
{text}

Summary:
```

### Translation with Context

```
Dịch văn bản {source_lang} sau sang {target_lang}.

Context: {context}
Tone: {tone}

Text: {text}

Translation:
```

### Format Conversion

```
Chuyển đổi {source_format} sau thành {target_format}.

Input:
{input_data}

Output ({target_format}):
```

## Analysis Templates (Các Template phân tích)

### Code Review

```
Review đoạn code sau cho các vấn đề:
1. Bugs và errors
2. Các vấn đề về performance
3. Security vulnerabilities (Các lỗ hổng bảo mật)
4. Sự vi phạm best practice

Code:
{code}

Review:
```

### SWOT Analysis

```
Thực hiện một SWOT analysis cho: {subject}

Context: {context}

Analysis:
Strengths:
-

Weaknesses:
-

Opportunities:
-

Threats:
-
```

## Question Answering Templates (Các Template hỏi đáp)

### RAG Template

```
Trả lời câu hỏi dựa trên context được cung cấp. Nếu context không chứa đủ thông tin, hãy báo như vậy.

Context:
{context}

Question: {question}

Answer:
```

### Multi-Turn Q&A

```
Lịch sử cuộc hội thoại trước đó (Previous conversation):
{conversation_history}

Câu hỏi mới (New question): {question}

Answer (tiếp tục một cách tự nhiên từ cuộc hội thoại):
```

## Specialized Templates (Các Template chuyên biệt)

### SQL Query Generation

```
Generate một SQL query cho request sau.

Database schema:
{schema}

Request: {request}

SQL Query:
```

### Regex Pattern Creation

```
Tạo một Regex pattern để match: {requirement}

Các test cases CẦN match (positive_examples):
{positive_examples}

Các test cases KHÔNG ĐƯỢC match (negative_examples):
{negative_examples}

Regex pattern:
```

### API Documentation

```
Generate API documentation cho function này:

Code:
{function_code}

Documentation (tuân theo định dạng {doc_format}):
```

## Sử dụng các template này bằng cách điền vào các {variables}
