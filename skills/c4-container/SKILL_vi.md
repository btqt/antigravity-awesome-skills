---
name: c4-container
description: Chuyên gia tài liệu cấp Container C4. Tổng hợp tài liệu cấp Thành phần thành kiến trúc cấp Container, ánh xạ các thành phần tới các đơn vị triển khai, tài liệu hóa giao diện container dưới dạng API, và tạo biểu đồ container. Sử dụng khi tổng hợp các thành phần thành các container triển khai và tài liệu hóa kiến trúc triển khai hệ thống.
metadata:
  model: sonnet
---

# C4 Cấp độ Container: Triển khai Hệ thống

## Sử dụng kỹ năng này khi

- Làm việc trên các nhiệm vụ hoặc quy trình làm việc c4 container level: triển khai hệ thống
- Cần hướng dẫn, thực tiễn tốt nhất hoặc danh sách kiểm tra cho c4 container level: triển khai hệ thống

## Không sử dụng kỹ năng này khi

- Nhiệm vụ không liên quan đến c4 container level: triển khai hệ thống
- Bạn cần một miền hoặc công cụ khác ngoài phạm vi này

## Hướng dẫn

- Làm rõ mục tiêu, ràng buộc và đầu vào cần thiết.
- Áp dụng các thực tiễn tốt nhất có liên quan và xác thực kết quả.
- Cung cấp các bước có thể hành động và xác minh.
- Nếu cần các ví dụ chi tiết, hãy mở `resources/implementation-playbook.md`.

## Container

### [Tên Container]

- **Tên**: [Tên Container]
- **Mô tả**: [Mô tả ngắn về mục đích container và triển khai]
- **Loại**: [Ứng dụng Web, API, Cơ sở dữ liệu, Hàng đợi Tin nhắn, v.v.]
- **Công nghệ**: [Công nghệ chính: Node.js, Python, PostgreSQL, Redis, v.v.]
- **Triển khai**: [Docker, Kubernetes, Dịch vụ Đám mây, v.v.]

## Mục đích

[Mô tả chi tiết về những gì container này làm và cách nó được triển khai]

## Thành phần

Container này triển khai các thành phần sau:

- [Tên Thành phần]: [Mô tả]
  - Tài liệu: [c4-component-name.md](./c4-component-name.md)

## Giao diện

### [Tên API/Giao diện]

- **Giao thức**: [REST/GraphQL/gRPC/Events/v.v.]
- **Mô tả**: [Giao diện này cung cấp gì]
- **Thông số kỹ thuật**: [Liên kết đến tệp OpenAPI/Swagger/API Spec]
- **Endpoints**:
  - `GET /api/resource` - [Mô tả]
  - `POST /api/resource` - [Mô tả]

## Phụ thuộc

### Các Container được sử dụng

- [Tên Container]: [Cách nó được sử dụng, giao thức giao tiếp]

### Các hệ thống bên ngoài

- [Hệ thống Bên ngoài]: [Cách nó được sử dụng, loại tích hợp]

## Cơ sở hạ tầng

- **Cấu hình Triển khai**: [Liên kết đến Dockerfile, K8s manifest, v.v.]
- **Mở rộng**: [Chiến lược mở rộng ngang/dọc]
- **Tài nguyên**: [CPU, bộ nhớ, yêu cầu lưu trữ]

## Biểu đồ Container

Sử dụng cú pháp Mermaid C4Container phù hợp:

```mermaid
C4Container
    title Container Diagram for [System Name]

    Person(user, "User", "Uses the system")
    System_Boundary(system, "System Name") {
        Container(webApp, "Web Application", "Spring Boot, Java", "Provides web interface")
        Container(api, "API Application", "Node.js, Express", "Provides REST API")
        ContainerDb(database, "Database", "PostgreSQL", "Stores data")
        Container_Queue(messageQueue, "Message Queue", "RabbitMQ", "Handles async messaging")
    }
    System_Ext(external, "External System", "Third-party service")

    Rel(user, webApp, "Uses", "HTTPS")
    Rel(webApp, api, "Makes API calls to", "JSON/HTTPS")
    Rel(api, database, "Reads from and writes to", "SQL")
    Rel(api, messageQueue, "Publishes messages to")
    Rel(api, external, "Uses", "API")
```

```

**Nguyên tắc Chính** (từ [c4model.com](https://c4model.com/diagrams/container)):

- Hiển thị **các lựa chọn công nghệ cấp cao** (đây là nơi chi tiết công nghệ thuộc về)
- Hiển thị cách **phân phối trách nhiệm** qua các container
- Bao gồm **các loại container**: Ứng dụng, Cơ sở dữ liệu, Hàng đợi Tin nhắn, Hệ thống Tệp, v.v.
- Hiển thị **giao thức giao tiếp** giữa các container
- Bao gồm **các hệ thống bên ngoài** mà các container tương tác

```

## Mẫu Thông số kỹ thuật API

Đối với mỗi API container, tạo thông số kỹ thuật OpenAPI/Swagger:

```yaml
openapi: 3.1.0
info:
  title: [Container Name] API
  description: [API description]
  version: 1.0.0
servers:
  - url: https://api.example.com
    description: Production server
paths:
  /api/resource:
    get:
      summary: [Operation summary]
      description: [Operation description]
      parameters:
        - name: param1
          in: query
          schema:
            type: string
      responses:
        '200':
          description: [Response description]
          content:
            application/json:
              schema:
                type: object
```

## Ví dụ Tương tác

- "Tổng hợp tất cả các thành phần thành các container dựa trên định nghĩa triển khai"
- "Ánh xạ các thành phần API tới các container và tài liệu hóa API của chúng dưới dạng thông số kỹ thuật OpenAPI"
- "Tạo tài liệu cấp container cho kiến trúc microservices"
- "Tài liệu hóa các giao diện container dưới dạng thông số kỹ thuật Swagger/OpenAPI"
- "Phân tích Kubernetes manifests và tạo tài liệu container"

## Phân biệt Chính

- **vs C4-Component agent**: Ánh xạ các thành phần tới các đơn vị triển khai; Tác nhân Component tập trung vào nhóm logic
- **vs C4-Context agent**: Cung cấp chi tiết cấp container; Tác nhân Context tạo các biểu đồ hệ thống cấp cao
- **vs C4-Code agent**: Tập trung vào kiến trúc triển khai; Tác nhân Code tài liệu hóa các phần tử mã riêng lẻ

## Ví dụ Đầu ra

Khi tổng hợp các container, cung cấp:

- Ranh giới container rõ ràng với lý do triển khai
- Tên container mô tả và đặc điểm triển khai
- Tài liệu API hoàn chỉnh với thông số kỹ thuật OpenAPI/Swagger
- Liên kết đến tất cả các thành phần chứa trong đó
- Biểu đồ container Mermaid hiển thị kiến trúc triển khai
- Liên kết đến cấu hình triển khai (Dockerfiles, K8s manifests, v.v.)
- Yêu cầu cơ sở hạ tầng và cân nhắc mở rộng
- Định dạng tài liệu nhất quán trên tất cả các container
