---
name: rag-implementation
description: "Quy trình triển khai RAG (Retrieval-Augmented Generation) bao gồm lựa chọn embedding, thiết lập vector Database, chiến lược chunking và tối ưu hóa truy xuất."
category: granular-workflow-bundle
risk: safe
source: personal
date_added: "2026-02-27"
---

# Quy trình triển khai RAG

## Tổng quan

Quy trình chuyên sâu để triển khai các hệ thống RAG (Retrieval-Augmented Generation) bao gồm lựa chọn mô hình embedding, thiết lập vector Database, chiến lược chunking, tối ưu hóa truy xuất, và đánh giá.

## Khi nào nên sử dụng Quy trình này

Sử dụng quy trình này khi:

- Xây dựng các ứng dụng được hỗ trợ bởi RAG
- Triển khai tìm kiếm ngữ nghĩa (semantic search)
- Tạo AI dựa trên kiến thức nền tảng (knowledge-grounded AI)
- Thiết lập hệ thống Hỏi đáp (Q&A) tài liệu
- Tối ưu hóa chất lượng truy xuất

## Các giai đoạn của Quy trình

### Giai đoạn 1: Phân tích Yêu cầu

#### Kỹ năng cần Gọi

- `ai-product` - Thiết kế sản phẩm AI
- `rag-engineer` - Kỹ thuật RAG

#### Hành động

1. Xác định use case (trường hợp sử dụng)
2. Nhận dạng các nguồn dữ liệu
3. Đặt ra các yêu cầu về độ chính xác
4. Xác định mục tiêu về độ trễ (latency targets)
5. Lên kế hoạch cho các số liệu đánh giá (evaluation metrics)

#### Prompt sao chép-dán

```
Use @ai-product to define RAG application requirements
```

### Giai đoạn 2: Lựa chọn Embedding

#### Kỹ năng cần Gọi

- `embedding-strategies` - Lựa chọn Embedding
- `rag-engineer` - Các kỹ thuật RAG (RAG patterns)

#### Hành động

1. Đánh giá các mô hình embedding
2. Kiểm tra sự phù hợp với lĩnh vực (domain relevance)
3. Đo lường chất lượng embedding
4. Cân nhắc chi phí/độ trễ
5. Chọn mô hình

#### Prompt sao chép-dán

```
Use @embedding-strategies to select optimal embedding model
```

### Giai đoạn 3: Thiết lập Vector Database

#### Kỹ năng cần Gọi

- `vector-database-engineer` - Vector Database
- `similarity-search-patterns` - Tìm kiếm tương tự (Similarity search)

#### Hành động

1. Chọn vector Database
2. Thiết kế schema
3. Cấu hình các index
4. Thiết lập kết nối
5. Kiểm thử các truy vấn (queries)

#### Prompt sao chép-dán

```
Use @vector-database-engineer to set up vector database
```

### Giai đoạn 4: Chiến lược Chunking

#### Kỹ năng cần Gọi

- `rag-engineer` - Chiến lược chunking
- `rag-implementation` - Triển khai RAG

#### Hành động

1. Chọn kích thước chunk
2. Triển khai chunking
3. Thêm xử lý trùng lặp (overlap handling)
4. Tạo metadata
5. Kiểm thử chất lượng truy xuất

#### Prompt sao chép-dán

```
Use @rag-engineer to implement chunking strategy
```

### Giai đoạn 5: Triển khai Truy xuất

#### Kỹ năng cần Gọi

- `similarity-search-patterns` - Tìm kiếm tương tự (Similarity search)
- `hybrid-search-implementation` - Tìm kiếm kết hợp (Hybrid search)

#### Hành động

1. Triển khai vector search
2. Thêm keyword search
3. Cấu hình hybrid search
4. Thiết lập reranking
5. Tối ưu hóa độ trễ

#### Prompt sao chép-dán

```
Use @similarity-search-patterns to implement retrieval
```

```
Use @hybrid-search-implementation to add hybrid search
```

### Giai đoạn 6: Tích hợp LLM

#### Kỹ năng cần Gọi

- `llm-application-dev-ai-assistant` - Tích hợp LLM
- `llm-application-dev-prompt-optimize` - Tối ưu hóa Prompt

#### Hành động

1. Chọn nhà cung cấp LLM
2. Thiết kế template cho Prompt
3. Triển khai tiêm ngữ cảnh (context injection)
4. Thêm xử lý trích dẫn (citation handling)
5. Kiểm thử chất lượng văn bản được tạo

#### Prompt sao chép-dán

```
Use @llm-application-dev-ai-assistant to integrate LLM
```

### Giai đoạn 7: Caching

#### Kỹ năng cần Gọi

- `prompt-caching` - Caching Prompt
- `rag-engineer` - Tối ưu hóa RAG

#### Hành động

1. Triển khai response caching
2. Thiết lập embedding cache
3. Cấu hình TTL
4. Thêm cache invalidation (Vô hiệu hóa cache)
5. Theo dõi tỷ lệ hit cache (hit rates)

#### Prompt sao chép-dán

```
Use @prompt-caching to implement RAG caching
```

### Giai đoạn 8: Đánh giá

#### Kỹ năng cần Gọi

- `llm-evaluation` - Đánh giá LLM
- `evaluation` - Đánh giá AI

#### Hành động

1. Xác định các số liệu đánh giá (evaluation metrics)
2. Tạo tập dữ liệu kiểm thử (test dataset)
3. Đo lường độ chính xác của truy xuất
4. Đánh giá chất lượng sinh văn bản (generation quality)
5. Lặp lại quá trình để cải thiện

#### Prompt sao chép-dán

```
Use @llm-evaluation to evaluate RAG system
```

## Kiến trúc RAG

```
User Query -> Embedding -> Vector Search -> Retrieved Docs -> LLM -> Response
                |              |              |              |
             Model        Vector DB     Chunk Store    Prompt + Context
```

## Quality Gates

- [ ] Đã chọn mô hình embedding
- [ ] Đã cấu hình Vector DB
- [ ] Đã triển khai chunking
- [ ] Truy xuất hoạt động tốt
- [ ] Đã tích hợp LLM
- [ ] Đánh giá đạt yêu cầu

## Các Gói Quy trình Liên quan

- `ai-ml` - Phát triển AI/ML
- `ai-agent-development` - AI agents
- `database` - Vector Database
