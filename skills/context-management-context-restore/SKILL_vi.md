---
name: context-management-context-restore
description: "Sử dụng khi làm việc với khôi phục bối cảnh quản lý bối cảnh"
---

# Khôi phục Bối cảnh: Tái tạo Bộ nhớ Ngữ nghĩa Nâng cao (Context Restoration)

## Sử dụng kỹ năng này khi

- Làm việc trên các nhiệm vụ hoặc quy trình làm việc khôi phục bối cảnh: tái tạo bộ nhớ ngữ nghĩa nâng cao
- Cần hướng dẫn, thực tiễn tốt nhất hoặc danh sách kiểm tra cho khôi phục bối cảnh: tái tạo bộ nhớ ngữ nghĩa nâng cao

## Không sử dụng kỹ năng này khi

- Nhiệm vụ không liên quan đến khôi phục bối cảnh: tái tạo bộ nhớ ngữ nghĩa nâng cao
- Bạn cần một miền hoặc công cụ khác ngoài phạm vi này

## Hướng dẫn

- Làm rõ mục tiêu, ràng buộc và đầu vào cần thiết.
- Áp dụng các thực tiễn tốt nhất có liên quan và xác thực kết quả.
- Cung cấp các bước có thể hành động và xác minh.
- Nếu cần các ví dụ chi tiết, hãy mở `resources/implementation-playbook.md`.

## Tuyên bố Vai trò

Chuyên gia Khôi phục Bối cảnh tập trung vào việc truy xuất và tái cấu trúc bối cảnh thông minh, nhận thức ngữ nghĩa trên các quy trình làm việc AI đa tác nhân phức tạp. Chuyên về việc bảo tồn và tái cấu trúc kiến thức dự án với độ trung thực cao và mất thông tin tối thiểu.

## Tổng quan Bối cảnh

Công cụ Khôi phục Bối cảnh là một hệ thống quản lý bộ nhớ tinh vi được thiết kế để:

- Khôi phục và tái cấu trúc bối cảnh dự án trên các quy trình làm việc AI phân tán
- Cho phép sự liên tục liền mạch trong các dự án phức tạp, chạy dài
- Cung cấp khả năng tái tạo bối cảnh thông minh, nhận thức ngữ nghĩa
- Duy trì tính toàn vẹn kiến thức lịch sử và khả năng truy xuất quyết định

## Yêu cầu Cốt lõi và Tham số

### Tham số Đầu vào

- `context_source`: Vị trí lưu trữ bối cảnh chính (cơ sở dữ liệu vector, hệ thống tệp)
- `project_identifier`: Không gian tên dự án duy nhất
- `restoration_mode`:
  - `full`: Khôi phục bối cảnh hoàn toàn
  - `incremental`: Cập nhật bối cảnh một phần
  - `diff`: So sánh và hợp nhất các phiên bản bối cảnh
- `token_budget`: Số token bối cảnh tối đa để khôi phục (mặc định: 8192)
- `relevance_threshold`: Ngưỡng tương tự ngữ nghĩa cho các thành phần bối cảnh (mặc định: 0.75)

## Các Chiến lược Truy xuất Bối cảnh Nâng cao

### 1. Tìm kiếm Vector Ngữ nghĩa

- Sử dụng các mô hình nhúng đa chiều cho việc truy xuất bối cảnh
- Sử dụng độ tương tự cosine và các kỹ thuật phân cụm vector
- Hỗ trợ nhúng đa phương thức (văn bản, mã, sơ đồ kiến trúc)

```python
def semantic_context_retrieve(project_id, query_vector, top_k=5):
    """Semantically retrieve most relevant context vectors"""
    vector_db = VectorDatabase(project_id)
    matching_contexts = vector_db.search(
        query_vector,
        similarity_threshold=0.75,
        max_results=top_k
    )
    return rank_and_filter_contexts(matching_contexts)
```

### 2. Lọc và Xếp hạng Mức độ Liên quan

- Triển khai chấm điểm mức độ liên quan đa giai đoạn
- Xem xét sự suy giảm theo thời gian, độ tương tự ngữ nghĩa và tác động lịch sử
- Trọng số động của các thành phần bối cảnh

```python
def rank_context_components(contexts, current_state):
    """Rank context components based on multiple relevance signals"""
    ranked_contexts = []
    for context in contexts:
        relevance_score = calculate_composite_score(
            semantic_similarity=context.semantic_score,
            temporal_relevance=context.age_factor,
            historical_impact=context.decision_weight
        )
        ranked_contexts.append((context, relevance_score))

    return sorted(ranked_contexts, key=lambda x: x[1], reverse=True)
```

### 3. Các Mẫu Tái tạo Bối cảnh

- Triển khai tải bối cảnh gia tăng
- Hỗ trợ tái cấu trúc bối cảnh một phần và toàn bộ
- Quản lý ngân sách token một cách linh hoạt

```python
def rehydrate_context(project_context, token_budget=8192):
    """Intelligent context rehydration with token budget management"""
    context_components = [
        'project_overview',
        'architectural_decisions',
        'technology_stack',
        'recent_agent_work',
        'known_issues'
    ]

    prioritized_components = prioritize_components(context_components)
    restored_context = {}

    current_tokens = 0
    for component in prioritized_components:
        component_tokens = estimate_tokens(component)
        if current_tokens + component_tokens <= token_budget:
            restored_context[component] = load_component(component)
            current_tokens += component_tokens

    return restored_context
```

### 4. Tái cấu trúc Trạng thái Phiên

- Tái cấu trúc trạng thái quy trình làm việc của tác nhân
- Bảo tồn các dấu vết quyết định và bối cảnh lý luận
- Hỗ trợ lịch sử cộng tác đa tác nhân

### 5. Hợp nhất Bối cảnh và Giải quyết Xung đột

- Triển khai các chiến lược hợp nhất ba chiều
- Phát hiện và giải quyết các xung đột ngữ nghĩa
- Duy trì nguồn gốc và khả năng truy xuất quyết định

### 6. Tải Bối cảnh Gia tăng

- Hỗ trợ tải lười (lazy loading) các thành phần bối cảnh
- Triển khai truyền phát bối cảnh cho các dự án lớn
- Cho phép mở rộng bối cảnh động

### 7. Xác thực Bối cảnh và Kiểm tra Tính toàn vẹn

- Chữ ký bối cảnh mật mã
- Xác minh tính nhất quán ngữ nghĩa
- Kiểm tra khả năng tương thích phiên bản

### 8. Tối ưu hóa Hiệu suất

- Triển khai các cơ chế bộ nhớ đệm hiệu quả
- Sử dụng các cấu trúc dữ liệu xác suất cho việc lập chỉ mục bối cảnh
- Tối ưu hóa các thuật toán tìm kiếm vector

## Quy trình làm việc Tham khảo

### Quy trình 1: Tiếp tục Dự án

1. Truy xuất bối cảnh dự án gần đây nhất
2. Xác thực bối cảnh so với cơ sở mã hiện tại
3. Khôi phục có chọn lọc các thành phần liên quan
4. Tạo tóm tắt tiếp tục

### Quy trình 2: Chuyển giao Kiến thức Liên dự án

1. Trích xuất các vector ngữ nghĩa từ dự án nguồn
2. Ánh xạ và chuyển giao kiến thức liên quan
3. Thích ứng bối cảnh với miền của dự án đích
4. Xác thực khả năng chuyển giao kiến thức

## Ví dụ Sử dụng

```bash
# Full context restoration
context-restore project:ai-assistant --mode full

# Incremental context update
context-restore project:web-platform --mode incremental

# Semantic context query
context-restore project:ml-pipeline --query "model training strategy"
```

## Các Mẫu Tích hợp

- Đường ống RAG (Thế hệ Tăng cường Truy xuất)
- Điều phối quy trình làm việc đa tác nhân
- Hệ thống học tập liên tục
- Quản lý kiến thức doanh nghiệp

## Lộ trình Tương lai

- Hỗ trợ nhúng đa phương thức nâng cao
- Các thuật toán tìm kiếm vector lấy cảm hứng từ lượng tử
- Tái cấu trúc bối cảnh tự phục hồi
- Các chiến lược bối cảnh học tập thích ứng
