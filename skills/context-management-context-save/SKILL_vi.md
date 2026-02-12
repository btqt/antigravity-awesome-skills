---
name: context-management-context-save
description: "Sử dụng khi làm việc với lưu bối cảnh quản lý bối cảnh"
---

# Công cụ Lưu Bối cảnh: Chuyên gia Quản lý Bối cảnh Thông minh (Context Save Tool)

## Sử dụng kỹ năng này khi

- Làm việc trên các nhiệm vụ hoặc quy trình làm việc của công cụ lưu bối cảnh: chuyên gia quản lý bối cảnh thông minh
- Cần hướng dẫn, thực tiễn tốt nhất hoặc danh sách kiểm tra cho công cụ lưu bối cảnh: chuyên gia quản lý bối cảnh thông minh

## Không sử dụng kỹ năng này khi

- Nhiệm vụ không liên quan đến công cụ lưu bối cảnh: chuyên gia quản lý bối cảnh thông minh
- Bạn cần một miền hoặc công cụ khác ngoài phạm vi này

## Hướng dẫn

- Làm rõ mục tiêu, ràng buộc và đầu vào cần thiết.
- Áp dụng các thực tiễn tốt nhất có liên quan và xác thực kết quả.
- Cung cấp các bước có thể hành động và xác minh.
- Nếu cần các ví dụ chi tiết, hãy mở `resources/implementation-playbook.md`.

## Vai trò và Mục đích

Một chuyên gia kỹ thuật bối cảnh ưu tú tập trung vào việc bảo tồn bối cảnh toàn diện, ngữ nghĩa và thích ứng động trên các quy trình làm việc AI. Công cụ này điều phối các chiến lược nắm bắt, tuần tự hóa và truy xuất bối cảnh nâng cao để duy trì kiến thức tổ chức và cho phép cộng tác đa phiên liền mạch.

## Tổng quan Quản lý Bối cảnh

Công cụ Lưu Bối cảnh là một giải pháp kỹ thuật bối cảnh tinh vi được thiết kế để:

- Nắm bắt trạng thái và kiến thức dự án toàn diện
- Cho phép truy xuất bối cảnh ngữ nghĩa
- Hỗ trợ điều phối quy trình làm việc đa tác nhân
- Bảo tồn các quyết định kiến trúc và sự phát triển dự án
- Tạo điều kiện chuyển giao kiến thức thông minh

## Yêu cầu và Xử lý Tham số

### Tham số Đầu vào

- `$PROJECT_ROOT`: Đường dẫn tuyệt đối đến gốc dự án
- `$CONTEXT_TYPE`: Độ chi tiết của việc nắm bắt bối cảnh (tối thiểu, tiêu chuẩn, toàn diện)
- `$STORAGE_FORMAT`: Định dạng lưu trữ ưu tiên (json, markdown, vector)
- `$TAGS`: Các thẻ ngữ nghĩa tùy chọn cho phân loại bối cảnh

## Các Chiến lược Trích xuất Bối cảnh

### 1. Xác định Thông tin Ngữ nghĩa

- Trích xuất các mẫu kiến trúc cấp cao
- Nắm bắt lý do ra quyết định
- Xác định các mối quan tâm cắt ngang và phụ thuộc
- Ánh xạ các cấu trúc kiến thức ngầm

### 2. Các Mẫu Tuần tự hóa Trạng thái

- Sử dụng JSON Schema cho biểu diễn có cấu trúc
- Hỗ trợ các mô hình bối cảnh lồng nhau, phân cấp
- Triển khai tuần tự hóa an toàn kiểu
- Cho phép tái cấu trúc bối cảnh không mất mát

### 3. Quản lý Bối cảnh Đa phiên

- Tạo dấu vân tay bối cảnh duy nhất
- Hỗ trợ kiểm soát phiên bản cho các tạo tác bối cảnh
- Triển khai phát hiện trôi dạt bối cảnh
- Tạo khả năng diff ngữ nghĩa

### 4. Các Kỹ thuật Nén Bối cảnh

- Sử dụng các thuật toán nén nâng cao
- Hỗ trợ các chế độ nén có mất mát và không mất mát
- Triển khai giảm token ngữ nghĩa
- Tối ưu hóa hiệu quả lưu trữ

### 5. Tích hợp Cơ sở dữ liệu Vector

Cơ sở dữ liệu Vector được Hỗ trợ:

- Pinecone
- Weaviate
- Qdrant

Các Tính năng Tích hợp:

- Tạo nhúng ngữ nghĩa
- Xây dựng chỉ mục vector
- Truy xuất bối cảnh dựa trên độ tương tự
- Ánh xạ kiến thức đa chiều

### 6. Xây dựng Biểu đồ Kiến thức

- Trích xuất siêu dữ liệu quan hệ
- Tạo các biểu diễn bản thể học
- Hỗ trợ liên kết kiến thức liên miền
- Cho phép mở rộng bối cảnh dựa trên suy luận

### 7. Lựa chọn Định dạng Lưu trữ

Các Định dạng được Hỗ trợ:

- JSON có cấu trúc
- Markdown với frontmatter
- Protocol Buffers
- MessagePack
- YAML với chú thích ngữ nghĩa

## Ví dụ Mã

### 1. Trích xuất Bối cảnh

```python
def extract_project_context(project_root, context_type='standard'):
    context = {
        'project_metadata': extract_project_metadata(project_root),
        'architectural_decisions': analyze_architecture(project_root),
        'dependency_graph': build_dependency_graph(project_root),
        'semantic_tags': generate_semantic_tags(project_root)
    }
    return context
```

### 2. Lược đồ Tuần tự hóa Trạng thái

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "project_name": { "type": "string" },
    "version": { "type": "string" },
    "context_fingerprint": { "type": "string" },
    "captured_at": { "type": "string", "format": "date-time" },
    "architectural_decisions": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "decision_type": { "type": "string" },
          "rationale": { "type": "string" },
          "impact_score": { "type": "number" }
        }
      }
    }
  }
}
```

### 3. Thuật toán Nén Bối cảnh

```python
def compress_context(context, compression_level='standard'):
    strategies = {
        'minimal': remove_redundant_tokens,
        'standard': semantic_compression,
        'comprehensive': advanced_vector_compression
    }
    compressor = strategies.get(compression_level, semantic_compression)
    return compressor(context)
```

## Quy trình làm việc Tham khảo

### Quy trình 1: Nắm bắt Bối cảnh Giới thiệu Dự án

1. Phân tích cấu trúc dự án
2. Trích xuất các quyết định kiến trúc
3. Tạo nhúng ngữ nghĩa
4. Lưu trữ trong cơ sở dữ liệu vector
5. Tạo tóm tắt markdown

### Quy trình 2: Quản lý Bối cảnh Phiên Chạy Dài

1. Định kỳ nắm bắt ảnh chụp nhanh bối cảnh
2. Phát hiện các thay đổi kiến trúc quan trọng
3. Phiên bản và lưu trữ bối cảnh
4. Cho phép khôi phục bối cảnh có chọn lọc

## Khả năng Tích hợp Nâng cao

- Đồng bộ hóa bối cảnh thời gian thực
- Khả năng di chuyển bối cảnh đa nền tảng
- Tuân thủ các tiêu chuẩn quản lý kiến thức doanh nghiệp
- Hỗ trợ biểu diễn bối cảnh đa phương thức

## Hạn chế và Cân nhắc

- Thông tin nhạy cảm phải được loại trừ rõ ràng
- Nắm bắt bối cảnh có chi phí tính toán
- Yêu cầu cấu hình cẩn thận để có hiệu suất tối ưu

## Lộ trình Tương lai

- Cải thiện nén bối cảnh dựa trên ML
- Tăng cường chuyển giao kiến thức liên miền
- Chỉnh sửa bối cảnh cộng tác thời gian thực
- Hệ thống đề xuất bối cảnh dự đoán
