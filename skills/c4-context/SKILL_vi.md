---
name: c4-context
description: Chuyên gia tài liệu cấp Ngữ cảnh C4. Tạo biểu đồ ngữ cảnh hệ thống cấp cao, tài liệu hóa chân dung (personas), hành trình người dùng, tính năng hệ thống, và các phụ thuộc bên ngoài. Tổng hợp tài liệu container và thành phần với tài liệu hệ thống để tạo kiến trúc cấp ngữ cảnh toàn diện. Sử dụng khi tạo tài liệu ngữ cảnh hệ thống C4 cấp cao nhất.
metadata:
  model: sonnet
---

# C4 Cấp độ Ngữ cảnh: Ngữ cảnh Hệ thống

## Sử dụng kỹ năng này khi

- Làm việc trên các nhiệm vụ hoặc quy trình làm việc c4 context level: ngữ cảnh hệ thống
- Cần hướng dẫn, thực tiễn tốt nhất hoặc danh sách kiểm tra cho c4 context level: ngữ cảnh hệ thống

## Không sử dụng kỹ năng này khi

- Nhiệm vụ không liên quan đến c4 context level: ngữ cảnh hệ thống
- Bạn cần một miền hoặc công cụ khác ngoài phạm vi này

## Hướng dẫn

- Làm rõ mục tiêu, ràng buộc và đầu vào cần thiết.
- Áp dụng các thực tiễn tốt nhất có liên quan và xác thực kết quả.
- Cung cấp các bước có thể hành động và xác minh.
- Nếu cần các ví dụ chi tiết, hãy mở `resources/implementation-playbook.md`.

## Tổng quan Hệ thống

### Mô tả Ngắn

[Mô tả một câu về những gì hệ thống làm]

### Mô tả Dài

[Mô tả chi tiết về mục đích hệ thống, khả năng, và những vấn đề nó giải quyết]

## Chân dung (Personas)

### [Tên Persona]

- **Loại**: [Người dùng Con người / Người dùng Lập trình / Hệ thống Bên ngoài]
- **Mô tả**: [Persona này là ai và họ cần gì]
- **Mục tiêu**: [Những gì persona này muốn đạt được]
- **Các tính năng chính được sử dụng**: [Danh sách các tính năng persona này sử dụng]

## Tính năng Hệ thống

### [Tên Tính năng]

- **Mô tả**: [Tính năng này làm gì]
- **Người dùng**: [Những persona nào sử dụng tính năng này]
- **Hành trình Người dùng**: [Liên kết đến bản đồ hành trình người dùng]

## Hành trình Người dùng

### Hành trình [Tên Tính năng] - [Tên Persona]

1. [Bước 1]: [Mô tả]
2. [Bước 2]: [Mô tả]
3. [Bước 3]: [Mô tả]
   ...

### Hành trình Tích hợp [Hệ thống Bên ngoài]

1. [Bước 1]: [Mô tả]
2. [Bước 2]: [Mô tả]
   ...

## Hệ thống Bên ngoài và Phụ thuộc

### [Tên Hệ thống Bên ngoài]

- **Loại**: [Cơ sở dữ liệu, API, Dịch vụ, Hàng đợi Tin nhắn, v.v.]
- **Mô tả**: [Hệ thống bên ngoài này cung cấp gì]
- **Loại Tích hợp**: [API, Sự kiện, Chuyển tệp, v.v.]
- **Mục đích**: [Tại sao hệ thống phụ thuộc vào cái này]

## Biểu đồ Ngữ cảnh Hệ thống

[Biểu đồ Mermaid hiển thị hệ thống, người dùng, và các hệ thống bên ngoài]

## Tài liệu Liên quan

- [Tài liệu Container](./c4-container.md)
- [Tài liệu Thành phần](./c4-component.md)

````

## Mẫu Biểu đồ Ngữ cảnh

Theo [mô hình C4](https://c4model.com/diagrams/system-context), biểu đồ Ngữ cảnh Hệ thống hiển thị hệ thống như một hộp ở trung tâm, được bao quanh bởi người dùng và các hệ thống khác mà nó tương tác. Trọng tâm là **con người (tác nhân, vai trò, chân dung) và hệ thống phần mềm** thay vì công nghệ, giao thức và các chi tiết cấp thấp khác.

Sử dụng cú pháp Mermaid C4 phù hợp:

```mermaid
C4Context
    title System Context Diagram

    Person(user, "User", "Uses the system to accomplish their goals")
    System(system, "System Name", "Provides features X, Y, and Z")
    System_Ext(external1, "External System 1", "Provides service A")
    System_Ext(external2, "External System 2", "Provides service B")
    SystemDb(externalDb, "External Database", "Stores data")

    Rel(user, system, "Uses")
    Rel(system, external1, "Uses", "API")
    Rel(system, external2, "Sends events to")
    Rel(system, externalDb, "Reads from and writes to")
````

**Nguyên tắc Chính** (từ [c4model.com](https://c4model.com/diagrams/system-context)):

- Tập trung vào **con người và hệ thống phần mềm**, không phải công nghệ
- Hiển thị **ranh giới hệ thống** rõ ràng
- Bao gồm tất cả **người dùng** (con người và lập trình)
- Bao gồm tất cả **hệ thống bên ngoài** mà hệ thống tương tác
- Giữ cho nó **thân thiện với các bên liên quan** - dễ hiểu đối với khán giả phi kỹ thuật
- Tránh hiển thị công nghệ, giao thức, hoặc chi tiết cấp thấp

## Ví dụ Tương tác

- "Tạo tài liệu cấp Ngữ cảnh C4 cho hệ thống"
- "Xác định tất cả các personas và tạo bản đồ hành trình người dùng cho các tính năng chính"
- "Tài liệu hóa các hệ thống bên ngoài và tạo biểu đồ ngữ cảnh hệ thống"
- "Phân tích tài liệu hệ thống và tạo tài liệu ngữ cảnh toàn diện"
- "Lập bản đồ hành trình người dùng cho tất cả các tính năng chính bao gồm người dùng lập trình"

## Phân biệt Chính

- **vs C4-Container agent**: Cung cấp cái nhìn hệ thống cấp cao; Tác nhân Container tập trung vào kiến trúc triển khai
- **vs C4-Component agent**: Tập trung vào ngữ cảnh hệ thống; Tác nhân Component tập trung vào cấu trúc thành phần logic
- **vs C4-Code agent**: Cung cấp tổng quan thân thiện với các bên liên quan; Tác nhân Code cung cấp chi tiết mã kỹ thuật

## Ví dụ Đầu ra

Khi tạo tài liệu ngữ cảnh, cung cấp:

- Mô tả hệ thống rõ ràng (ngắn và dài)
- Tài liệu persona toàn diện (con người và lập trình)
- Danh sách tính năng hoàn chỉnh với mô tả
- Bản đồ hành trình người dùng chi tiết cho tất cả các tính năng chính
- Tài liệu hệ thống bên ngoài và phụ thuộc hoàn chỉnh
- Biểu đồ ngữ cảnh Mermaid hiển thị hệ thống, người dùng, và hệ thống bên ngoài
- Liên kết đến tài liệu container và thành phần
- Tài liệu thân thiện với các bên liên quan dễ hiểu đối với khán giả phi kỹ thuật
- Định dạng tài liệu nhất quán
