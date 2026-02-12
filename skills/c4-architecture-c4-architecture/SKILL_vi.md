---
name: c4-architecture-c4-architecture
description: "Tạo tài liệu kiến trúc C4 toàn diện cho một kho lưu trữ/cơ sở mã (codebase) hiện có sử dụng cách tiếp cận phân tích từ dưới lên (bottom-up)."
---

# Quy trình làm việc Tài liệu Kiến trúc C4

Tạo tài liệu kiến trúc C4 toàn diện cho một kho lưu trữ/cơ sở mã hiện có sử dụng cách tiếp cận phân tích từ dưới lên.

[Suy nghĩ mở rộng: Quy trình làm việc này thực hiện một quy trình tài liệu kiến trúc C4 hoàn chỉnh theo mô hình C4 (Context, Container, Component, Code). Nó sử dụng cách tiếp cận từ dưới lên, bắt đầu từ các thư mục mã sâu nhất và làm việc hướng lên trên, đảm bảo mọi phần tử mã được tài liệu hóa trước khi tổng hợp thành các khái niệm trừu tượng cấp cao hơn. Quy trình làm việc phối hợp bốn tác nhân C4 chuyên biệt (Code, Component, Container, Context) để tạo ra một bộ tài liệu kiến trúc hoàn chỉnh phục vụ cả các bên liên quan kỹ thuật và phi kỹ thuật.]

## Sử dụng kỹ năng này khi

- Làm việc trên các nhiệm vụ hoặc quy trình làm việc c4 architecture documentation workflow
- Cần hướng dẫn, thực tiễn tốt nhất hoặc danh sách kiểm tra cho c4 architecture documentation workflow

## Không sử dụng kỹ năng này khi

- Nhiệm vụ không liên quan đến c4 architecture documentation workflow
- Bạn cần một miền hoặc công cụ khác ngoài phạm vi này

## Hướng dẫn

- Làm rõ mục tiêu, ràng buộc và đầu vào cần thiết.
- Áp dụng các thực tiễn tốt nhất có liên quan và xác thực kết quả.
- Cung cấp các bước có thể hành động và xác minh.
- Nếu cần các ví dụ chi tiết, hãy mở `resources/implementation-playbook.md`.

## Tổng quan

Quy trình làm việc này tạo tài liệu kiến trúc C4 toàn diện theo [mô hình C4 chính thức](https://c4model.com/diagrams) bằng cách:

1. **Cấp độ Mã (Code Level)**: Phân tích mọi thư mục con từ dưới lên để tạo tài liệu cấp mã
2. **Cấp độ Thành phần (Component Level)**: Tổng hợp tài liệu mã thành các thành phần logic trong các container
3. **Cấp độ Container**: Ánh xạ các thành phần tới các container triển khai với tài liệu API (hiển thị các lựa chọn công nghệ cấp cao)
4. **Cấp độ Ngữ cảnh (Context Level)**: Tạo ngữ cảnh hệ thống cấp cao với chân dung người dùng (personas) và hành trình người dùng (tập trung vào con người và hệ thống phần mềm, không phải công nghệ)

**Lưu ý**: Theo [mô hình C4](https://c4model.com/diagrams), bạn không cần sử dụng tất cả 4 cấp độ biểu đồ - biểu đồ ngữ cảnh hệ thống và container là đủ cho hầu hết các nhóm phát triển phần mềm. Quy trình làm việc này tạo tất cả các cấp độ cho sự đầy đủ, nhưng các nhóm có thể chọn cấp độ nào để sử dụng.

Tất cả tài liệu được ghi vào một thư mục `C4-Documentation/` mới trong gốc kho lưu trữ.

## Giai đoạn 1: Tài liệu Cấp Mã (Phân tích Từ dưới lên)

### 1.1 Khám phá Tất cả Thư mục con

- Sử dụng tìm kiếm cơ sở mã để xác định tất cả các thư mục con trong kho lưu trữ
- Sắp xếp các thư mục theo độ sâu (sâu nhất trước) để xử lý từ dưới lên
- Lọc ra các thư mục phi mã phổ biến (node_modules, .git, build, dist, v.v.)
- Tạo danh sách các thư mục để xử lý

### 1.2 Xử lý Mỗi Thư mục (Từ dưới lên)

Đối với mỗi thư mục, bắt đầu từ sâu nhất:

- Sử dụng công cụ Task với subagent_type="c4-architecture::c4-code"
- Prompt: |
  Phân tích mã trong thư mục: [directory_path]

  Tạo tài liệu cấp Mã C4 toàn diện theo cấu trúc này:
  1. **Phần Tổng quan**:
     - Tên: [Tên mô tả cho thư mục mã này]
     - Mô tả: [Mô tả ngắn về những gì mã này làm]
     - Vị trí: [Liên kết đến đường dẫn thư mục thực tế so với gốc repo]
     - Ngôn ngữ: [Ngôn ngữ lập trình chính được sử dụng]
     - Mục đích: [Mã này hoàn thành điều gì]
  2. **Phần Các phần tử Mã**:
     - Tài liệu hóa tất cả các hàm/phương thức với chữ ký đầy đủ:
       - Tên hàm, tham số (với kiểu), kiểu trả về
       - Mô tả về những gì mỗi hàm làm
       - Vị trí (đường dẫn tệp và số dòng)
       - Phụ thuộc (hàm này phụ thuộc vào cái gì)
     - Tài liệu hóa tất cả các lớp/mô-đun:
       - Tên lớp, mô tả, vị trí
       - Các phương thức và chữ ký của chúng
       - Phụ thuộc
  3. **Phần Phụ thuộc**:
     - Phụ thuộc nội bộ (mã khác trong repo này)
     - Phụ thuộc bên ngoài (thư viện, framework, dịch vụ)
  4. **Phần Mối quan hệ**:
     - Biểu đồ Mermaid tùy chọn nếu các mối quan hệ phức tạp

  Lưu đầu ra dưới dạng: C4-Documentation/c4-code-[directory-name].md
  Sử dụng tên thư mục đã làm sạch (thay thế / bằng -, loại bỏ ký tự đặc biệt) cho tên tệp.

  Đảm bảo tài liệu bao gồm:
  - Chữ ký hàm hoàn chỉnh với tất cả các tham số và kiểu
  - Liên kết đến vị trí mã nguồn thực tế
  - Tất cả các phụ thuộc (nội bộ và bên ngoài)
  - Tên và mô tả rõ ràng, mô tả

- Đầu ra mong đợi: tệp c4-code-<directory-name>.md trong C4-Documentation/
- Ngữ cảnh: Tất cả các tệp trong thư mục và các thư mục con của nó

**Lặp lại cho mỗi thư mục con** cho đến khi tất cả các thư mục có các tệp c4-code-\*.md tương ứng.

## Giai đoạn 2: Tổng hợp Cấp Thành phần

### 2.1 Phân tích Tất cả Tài liệu Cấp Mã

- Thu thập tất cả các tệp c4-code-\*.md đã tạo trong Giai đoạn 1
- Phân tích cấu trúc mã, phụ thuộc và mối quan hệ
- Xác định ranh giới thành phần logic dựa trên:
  - Ranh giới miền (chức năng kinh doanh liên quan)
  - Ranh giới kỹ thuật (framework, thư viện được chia sẻ)
  - Ranh giới tổ chức (quyền sở hữu nhóm, nếu rõ ràng)

### 2.2 Tạo Tài liệu Thành phần

Đối với mỗi thành phần được xác định:

- Sử dụng công cụ Task với subagent_type="c4-architecture::c4-component"
- Prompt: |
  Tổng hợp các tệp tài liệu cấp Mã C4 sau đây thành một thành phần logic:

  Các tệp mã để phân tích:
  [Danh sách đường dẫn tệp c4-code-*.md]

  Tạo tài liệu cấp Thành phần C4 toàn diện theo cấu trúc này:
  1. **Phần Tổng quan**:
     - Tên: [Tên thành phần - mô tả và có ý nghĩa]
     - Mô tả: [Mô tả ngắn về mục đích thành phần]
     - Loại: [Ứng dụng, Dịch vụ, Thư viện, v.v.]
     - Công nghệ: [Công nghệ chính được sử dụng]
  2. **Phần Mục đích**:
     - Mô tả chi tiết về những gì thành phần này làm
     - Những vấn đề nó giải quyết
     - Vai trò của nó trong hệ thống
  3. **Phần Tính năng Phần mềm**:
     - Liệt kê tất cả các tính năng phần mềm được cung cấp bởi thành phần này
     - Mỗi tính năng với một mô tả ngắn gọn
  4. **Phần Các phần tử Mã**:
     - Liệt kê tất cả các tệp c4-code-\*.md chứa trong thành phần này
     - Liên kết đến mỗi tệp với một mô tả ngắn gọn
  5. **Phần Giao diện**:
     - Tài liệu hóa tất cả các giao diện thành phần:
       - Tên giao diện
       - Giao thức (REST, GraphQL, gRPC, Events, v.v.)
       - Mô tả
       - Hoạt động (chữ ký hàm, endpoints, v.v.)
  6. **Phần Phụ thuộc**:
     - Các thành phần được sử dụng (các thành phần khác mà cái này phụ thuộc vào)
     - Các hệ thống bên ngoài (cơ sở dữ liệu, API, dịch vụ)
  7. **Biểu đồ Thành phần**:
     - Biểu đồ Mermaid hiển thị thành phần này và các mối quan hệ của nó

  Lưu đầu ra dưới dạng: C4-Documentation/c4-component-[component-name].md
  Sử dụng tên thành phần đã làm sạch cho tên tệp.

- Đầu ra mong đợi: tệp c4-component-<name>.md cho mỗi thành phần
- Ngữ cảnh: Tất cả các tệp c4-code-\*.md liên quan cho thành phần này

### 2.3 Tạo Chỉ mục Thành phần Chính (Master Component Index)

- Sử dụng công cụ Task với subagent_type="c4-architecture::c4-component"
- Prompt: |
  Tạo một chỉ mục thành phần chính liệt kê tất cả các thành phần trong hệ thống.

  Dựa trên tất cả các tệp c4-component-\*.md đã tạo, tạo:
  1. **Phần Thành phần Hệ thống**:
     - Liệt kê tất cả các thành phần với:
       - Tên thành phần
       - Mô tả ngắn
       - Liên kết đến tài liệu thành phần
  2. **Biểu đồ Mối quan hệ Thành phần**:
     - Biểu đồ Mermaid hiển thị tất cả các thành phần và mối quan hệ của chúng
     - Hiển thị phụ thuộc giữa các thành phần
     - Hiển thị phụ thuộc hệ thống bên ngoài

  Lưu đầu ra dưới dạng: C4-Documentation/c4-component.md

- Đầu ra mong đợi: Tệp c4-component.md chính
- Ngữ cảnh: Tất cả các tệp c4-component-\*.md

## Giai đoạn 3: Tổng hợp Cấp Container

### 3.1 Phân tích Thành phần và Định nghĩa Triển khai

- Xem xét tất cả các tệp c4-component-\*.md
- Tìm kiếm các định nghĩa triển khai/cơ sở hạ tầng:
  - Dockerfiles
  - Kubernetes manifests (deployments, services, v.v.)
  - Docker Compose files
  - Terraform/CloudFormation configs
  - Định nghĩa dịch vụ đám mây (AWS Lambda, Azure Functions, v.v.)
  - Định nghĩa đường ống CI/CD

### 3.2 Ánh xạ Thành phần tới Container

- Sử dụng công cụ Task với subagent_type="c4-architecture::c4-container"
- Prompt: |
  Tổng hợp các thành phần thành các container dựa trên định nghĩa triển khai.

  Tài liệu thành phần:
  [Danh sách tất cả các đường dẫn tệp c4-component-*.md]

  Định nghĩa triển khai tìm thấy:
  [Danh sách các tệp config triển khai: Dockerfiles, K8s manifests, v.v.]

  Tạo tài liệu cấp Container C4 toàn diện theo cấu trúc này:
  1. **Phần Container** (cho mỗi container):
     - Tên: [Tên Container]
     - Mô tả: [Mô tả ngắn về mục đích container và triển khai]
     - Loại: [Ứng dụng Web, API, Cơ sở dữ liệu, Hàng đợi Tin nhắn, v.v.]
     - Công nghệ: [Công nghệ chính: Node.js, Python, PostgreSQL, v.v.]
     - Triển khai: [Docker, Kubernetes, Dịch vụ Đám mây, v.v.]
  2. **Phần Mục đích** (cho mỗi container):
     - Mô tả chi tiết về những gì container này làm
     - Cách nó được triển khai
     - Vai trò của nó trong hệ thống
  3. **Phần Thành phần** (cho mỗi container):
     - Liệt kê tất cả các thành phần được triển khai trong container này
     - Liên kết đến tài liệu thành phần
  4. **Phần Giao diện** (cho mỗi container):
     - Tài liệu hóa tất cả các API và giao diện container:
       - Tên API/Giao diện
       - Giao thức (REST, GraphQL, gRPC, Events, v.v.)
       - Mô tả
       - Liên kết đến tệp Thông số kỹ thuật OpenAPI/Swagger/API
       - Danh sách các endpoints/hoạt động
  5. **Thông số kỹ thuật API**:
     - Đối với mỗi API container, tạo thông số kỹ thuật OpenAPI 3.1+
     - Lưu dưới dạng: C4-Documentation/apis/[container-name]-api.yaml
     - Bao gồm:
       - Tất cả các endpoints với phương thức (GET, POST, v.v.)
       - Schemas yêu cầu/phản hồi
       - Yêu cầu xác thực
       - Phản hồi lỗi
  6. **Phần Phụ thuộc** (cho mỗi container):
     - Các container được sử dụng (các container khác mà cái này phụ thuộc vào)
     - Các hệ thống bên ngoài (cơ sở dữ liệu, API bên thứ ba, v.v.)
     - Giao thức giao tiếp
  7. **Phần Cơ sở hạ tầng** (cho mỗi container):
     - Liên kết đến config triển khai (Dockerfile, K8s manifest, v.v.)
     - Chiến lược mở rộng
     - Yêu cầu tài nguyên (CPU, bộ nhớ, lưu trữ)
  8. **Biểu đồ Container**:
     - Biểu đồ Mermaid hiển thị tất cả các container và mối quan hệ của chúng
     - Hiển thị giao thức giao tiếp
     - Hiển thị phụ thuộc hệ thống bên ngoài

  Lưu đầu ra dưới dạng: C4-Documentation/c4-container.md

- Đầu ra mong đợi: c4-container.md với tất cả các container và thông số kỹ thuật API
- Ngữ cảnh: Tất cả tài liệu thành phần và định nghĩa triển khai

## Giai đoạn 4: Tài liệu Cấp Ngữ cảnh

### 4.1 Phân tích Tài liệu Hệ thống

- Xem xét tài liệu container và thành phần
- Tìm kiếm tài liệu hệ thống:
  - Tệp README
  - Tài liệu kiến trúc
  - Tài liệu yêu cầu
  - Tài liệu thiết kế
  - Tệp kiểm thử (để hiểu hành vi hệ thống)
  - Tài liệu API
  - Tài liệu người dùng

### 4.2 Tạo Tài liệu Ngữ cảnh

- Sử dụng công cụ Task với subagent_type="c4-architecture::c4-context"
- Prompt: |
  Tạo tài liệu cấp Ngữ cảnh C4 toàn diện cho hệ thống.

  Tài liệu Container: C4-Documentation/c4-container.md
  Tài liệu Thành phần: C4-Documentation/c4-component.md
  Tài liệu Hệ thống: [Danh sách README, tài liệu kiến trúc, yêu cầu, v.v.]
  Tệp kiểm thử: [Danh sách tệp kiểm thử hiển thị hành vi hệ thống]

  Tạo tài liệu cấp Ngữ cảnh C4 toàn diện theo cấu trúc này:
  1. **Phần Tổng quan Hệ thống**:
     - Mô tả Ngắn: [Mô tả một câu về những gì hệ thống làm]
     - Mô tả Dài: [Mô tả chi tiết về mục đích hệ thống, khả năng, vấn đề giải quyết]
  2. **Phần Chân dung (Personas)**:
     - Cho mỗi persona (người dùng con người và người dùng "lập trình"):
       - Tên Persona
       - Loại (Người dùng Con người / Người dùng Lập trình / Hệ thống Bên ngoài)
       - Mô tả (họ là ai, họ cần gì)
       - Mục tiêu (những gì họ muốn đạt được)
       - Các tính năng chính được sử dụng
  3. **Phần Tính năng Hệ thống**:
     - Cho mỗi tính năng cấp cao:
       - Tên tính năng
       - Mô tả (tính năng này làm gì)
       - Người dùng (persona nào sử dụng tính năng này)
       - Liên kết đến bản đồ hành trình người dùng
  4. **Phần Hành trình Người dùng**:
     - Cho mỗi tính năng chính và persona:
       - Tên hành trình: [Tên Tính năng] - [Tên Persona] Journey
       - Hành trình từng bước:
         1. [Bước 1]: [Mô tả]
         2. [Bước 2]: [Mô tả]
            ...
       - Bao gồm tất cả các điểm tiếp xúc hệ thống
     - Đối với người dùng lập trình (hệ thống bên ngoài, API):
       - Hành trình tích hợp với quy trình từng bước
  5. **Phần Hệ thống Bên ngoài và Phụ thuộc**:
     - Cho mỗi hệ thống bên ngoài:
       - Tên hệ thống
       - Loại (Cơ sở dữ liệu, API, Dịch vụ, Hàng đợi Tin nhắn, v.v.)
       - Mô tả (những gì nó cung cấp)
       - Loại tích hợp (API, Sự kiện, Chuyển tệp, v.v.)
       - Mục đích (tại sao hệ thống phụ thuộc vào cái này)
  6. **Biểu đồ Ngữ cảnh Hệ thống**:
     - Biểu đồ Mermaid C4Context hiển thị:
       - Hệ thống (như một hộp ở trung tâm)
       - Tất cả các personas (người dùng) xung quanh nó
       - Tất cả các hệ thống bên ngoài xung quanh nó
       - Mối quan hệ và luồng dữ liệu
       - Sử dụng ký hiệu C4Context cho biểu đồ C4 phù hợp
  7. **Phần Tài liệu Liên quan**:
     - Liên kết đến tài liệu container
     - Liên kết đến tài liệu thành phần

  Lưu đầu ra dưới dạng: C4-Documentation/c4-context.md

  Đảm bảo tài liệu là:
  - Dễ hiểu đối với các bên liên quan phi kỹ thuật
  - Tập trung vào mục đích hệ thống, người dùng và các mối quan hệ bên ngoài
  - Bao gồm bản đồ hành trình người dùng toàn diện
  - Xác định tất cả các hệ thống bên ngoài và phụ thuộc

- Đầu ra mong đợi: c4-context.md với ngữ cảnh hệ thống hoàn chỉnh
- Ngữ cảnh: Tất cả tài liệu container, thành phần và hệ thống

## Tùy chọn Cấu hình

- `target_directory`: Thư mục gốc để phân tích (mặc định: gốc kho lưu trữ hiện tại)
- `exclude_patterns`: Các mẫu để loại trừ (mặc định: node_modules, .git, build, dist, v.v.)
- `output_directory`: Nơi ghi tài liệu C4 (mặc định: C4-Documentation/)
- `include_tests`: Có phân tích tệp kiểm thử cho ngữ cảnh hay không (mặc định: true)
- `api_format`: Định dạng cho thông số kỹ thuật API (mặc định: openapi)

## Tiêu chí Thành công

- ✅ Mọi thư mục con đều có tệp c4-code-\*.md tương ứng
- ✅ Tất cả tài liệu cấp mã bao gồm chữ ký hàm hoàn chỉnh
- ✅ Các thành phần được nhóm logic với ranh giới rõ ràng
- ✅ Tất cả các thành phần có tài liệu giao diện
- ✅ Chỉ mục thành phần chính được tạo với biểu đồ mối quan hệ
- ✅ Các container ánh xạ tới các đơn vị triển khai thực tế
- ✅ Tất cả các API container được tài liệu hóa với thông số kỹ thuật OpenAPI/Swagger
- ✅ Biểu đồ container hiển thị kiến trúc triển khai
- ✅ Ngữ cảnh hệ thống bao gồm tất cả các personas (con người và lập trình)
- ✅ Hành trình người dùng được tài liệu hóa cho tất cả các tính năng chính
- ✅ Tất cả các hệ thống bên ngoài và phụ thuộc được xác định
- ✅ Biểu đồ ngữ cảnh hiển thị hệ thống, người dùng và hệ thống bên ngoài
- ✅ Tài liệu được tổ chức trong thư mục C4-Documentation/

## Cấu trúc Đầu ra

```
C4-Documentation/
├── c4-code-*.md              # Tài liệu cấp mã (một cho mỗi thư mục)
├── c4-component-*.md          # Tài liệu cấp thành phần (một cho mỗi thành phần)
├── c4-component.md            # Chỉ mục thành phần chính
├── c4-container.md            # Tài liệu cấp container
├── c4-context.md              # Tài liệu cấp ngữ cảnh
└── apis/                      # Thông số kỹ thuật API
    ├── [container]-api.yaml   # Thông số kỹ thuật OpenAPI cho mỗi container
    └── ...
```

## Ghi chú Phối hợp

- **Xử lý từ dưới lên**: Xử lý các thư mục từ sâu nhất đến nông nhất
- **Tổng hợp tăng dần**: Mỗi cấp độ xây dựng dựa trên tài liệu của cấp độ trước đó
- **Bao phủ hoàn toàn**: Mọi thư mục phải có tài liệu cấp mã trước khi tổng hợp
- **Tính nhất quán liên kết**: Tất cả các tệp tài liệu liên kết với nhau một cách thích hợp
- **Tài liệu API**: Các API container phải có thông số kỹ thuật OpenAPI/Swagger
- **Thân thiện với bên liên quan**: Tài liệu ngữ cảnh nên dễ hiểu đối với các bên liên quan phi kỹ thuật
- **Biểu đồ Mermaid**: Sử dụng ký hiệu Mermaid C4 phù hợp cho tất cả các biểu đồ

## Ví dụ Sử dụng

```bash
/c4-architecture:c4-architecture
```

Điều này sẽ:

1. Đi qua tất cả các thư mục con từ dưới lên
2. Tạo c4-code-\*.md cho mỗi thư mục
3. Tổng hợp thành các thành phần
4. Ánh xạ tới các container với tài liệu API
5. Tạo ngữ cảnh hệ thống với personas và hành trình

Tất cả tài liệu được ghi vào: C4-Documentation/
