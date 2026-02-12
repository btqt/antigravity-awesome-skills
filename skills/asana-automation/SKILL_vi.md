---
name: asana-automation
description: "Tự động hóa các tác vụ Asana thông qua Rube MCP (Composio): tác vụ, dự án, phần (sections), nhóm, không gian làm việc. Luôn tìm kiếm các công cụ trước để biết các lược đồ (schemas) hiện tại."
requires:
  mcp: [rube]
---

# Tự động hóa Asana qua Rube MCP

Tự động hóa các hoạt động của Asana thông qua bộ công cụ Asana của Composio qua Rube MCP.

## Điều kiện tiên quyết

- Rube MCP phải được kết nối (có sẵn `RUBE_SEARCH_TOOLS`)
- Kết nối Asana đang hoạt động thông qua `RUBE_MANAGE_CONNECTIONS` với bộ công cụ `asana`
- Luôn gọi `RUBE_SEARCH_TOOLS` trước để lấy các lược đồ công cụ hiện tại

## Thiết lập

**Tải Rube MCP**: Thêm `https://rube.app/mcp` làm máy chủ MCP trong cấu hình ứng dụng của bạn. Không cần khóa API — chỉ cần thêm endpoint và nó sẽ hoạt động.

1. Xác minh Rube MCP có sẵn bằng cách xác nhận `RUBE_SEARCH_TOOLS` phản hồi
2. Gọi `RUBE_MANAGE_CONNECTIONS` với bộ công cụ `asana`
3. Nếu kết nối không ở trạng thái ACTIVE, hãy làm theo liên kết xác thực được trả về để hoàn tất OAuth của Asana
4. Xác nhận trạng thái kết nối hiển thị ACTIVE trước khi chạy bất kỳ quy trình công việc nào

## Quy trình công việc cốt lõi

### 1. Quản lý Tác vụ

**Khi nào sử dụng**: Người dùng muốn tạo, tìm kiếm, liệt kê hoặc tổ chức các tác vụ

**Trình tự công cụ**:

1. `ASANA_GET_MULTIPLE_WORKSPACES` - Lấy ID không gian làm việc [Điều kiện tiên quyết]
2. `ASANA_SEARCH_TASKS_IN_WORKSPACE` - Tìm kiếm các tác vụ [Tùy chọn]
3. `ASANA_GET_TASKS_FROM_A_PROJECT` - Liệt kê các tác vụ dự án [Tùy chọn]
4. `ASANA_CREATE_A_TASK` - Tạo một tác vụ mới [Tùy chọn]
5. `ASANA_GET_A_TASK` - Lấy chi tiết tác vụ [Tùy chọn]
6. `ASANA_CREATE_SUBTASK` - Tạo một tác vụ con [Tùy chọn]
7. `ASANA_GET_TASK_SUBTASKS` - Liệt kê các tác vụ con [Tùy chọn]

**Các tham số chính**:

- `workspace`: GID của không gian làm việc (bắt buộc để tìm kiếm/tạo)
- `projects`: Mảng các GID dự án để thêm tác vụ vào
- `name`: Tên tác vụ
- `notes`: Mô tả tác vụ
- `assignee`: Người thực hiện (GID người dùng hoặc email)
- `due_on`: Ngày hết hạn (YYYY-MM-DD)

**Các bẫy phổ biến**:

- GID không gian làm việc là bắt buộc cho hầu hết các hoạt động; hãy lấy nó trước
- GID tác vụ được trả về dưới dạng chuỗi, không phải số nguyên
- Tìm kiếm nằm trong phạm vi không gian làm việc, không phải phạm vi dự án

### 2. Quản lý Dự án và Các Phần (Sections)

**Khi nào sử dụng**: Người dùng muốn tạo dự án, quản lý các phần hoặc tổ chức các tác vụ

**Trình tự công cụ**:

1. `ASANA_GET_WORKSPACE_PROJECTS` - Liệt kê các dự án trong không gian làm việc [Tùy chọn]
2. `ASANA_GET_A_PROJECT` - Lấy chi tiết dự án [Tùy chọn]
3. `ASANA_CREATE_A_PROJECT` - Tạo một dự án mới [Tùy chọn]
4. `ASANA_GET_SECTIONS_IN_PROJECT` - Liệt kê các phần [Tùy chọn]
5. `ASANA_CREATE_SECTION_IN_PROJECT` - Tạo một phần mới [Tùy chọn]
6. `ASANA_ADD_TASK_TO_SECTION` - Di chuyển tác vụ vào phần [Tùy chọn]
7. `ASANA_GET_TASKS_FROM_A_SECTION` - Liệt kê các tác vụ trong phần [Tùy chọn]

**Các tham số chính**:

- `project_gid`: GID dự án
- `name`: Tên dự án hoặc tên phần
- `workspace`: GID không gian làm việc để tạo
- `task`: GID tác vụ để gán vào phần
- `section`: GID phần

**Các bẫy phổ biến**:

- Các dự án thuộc về không gian làm việc; cần GID không gian làm việc để tạo
- Các phần được sắp xếp theo thứ tự trong một dự án
- `DUPLICATE_PROJECT` tạo một bản sao với tùy chọn bao gồm các tác vụ

### 3. Quản lý Nhóm và Người dùng

**Khi nào sử dụng**: Người dùng muốn liệt kê các nhóm, thành viên nhóm hoặc người dùng trong không gian làm việc

**Trình tự công cụ**:

1. `ASANA_GET_TEAMS_IN_WORKSPACE` - Liệt kê các nhóm trong không gian làm việc [Tùy chọn]
2. `ASANA_GET_USERS_FOR_TEAM` - Liệt kê các thành viên nhóm [Tùy chọn]
3. `ASANA_GET_USERS_FOR_WORKSPACE` - Liệt kê tất cả người dùng trong không gian làm việc [Tùy chọn]
4. `ASANA_GET_CURRENT_USER` - Lấy người dùng hiện đã xác thực [Tùy chọn]
5. `ASANA_GET_MULTIPLE_USERS` - Lấy chi tiết của nhiều người dùng [Tùy chọn]

**Các tham số chính**:

- `workspace_gid`: GID không gian làm việc
- `team_gid`: GID nhóm

**Các bẫy phổ biến**:

- Người dùng nằm trong phạm vi không gian làm việc
- Tư cách thành viên nhóm yêu cầu GID nhóm

### 4. Hoạt động Song song

**Khi nào sử dụng**: Người dùng cần thực hiện các hoạt động hàng loạt một cách hiệu quả

**Trình tự công cụ**:

1. `ASANA_SUBMIT_PARALLEL_REQUESTS` - Thực thi nhiều lệnh gọi API song song [Bắt buộc]

**Các tham số chính**:

- `actions`: Mảng các đối tượng hành động với phương thức, đường dẫn và dữ liệu

**Các bẫy phổ biến**:

- Mỗi hành động phải là một lệnh gọi API Asana hợp lệ
- Các yêu cầu cá nhân thất bại không làm hoàn tác các yêu cầu đã thành công

## Các mẫu Phổ biến

### Giải quyết ID (ID Resolution)

**Tên không gian làm việc -> GID**:

1. Gọi `ASANA_GET_MULTIPLE_WORKSPACES`
2. Tìm không gian làm việc theo tên
3. Trích xuất trường `gid`

**Tên dự án -> GID**:

1. Gọi `ASANA_GET_WORKSPACE_PROJECTS` với GID không gian làm việc
2. Tìm dự án theo tên
3. Trích xuất trường `gid`

### Phân trang

- Asana sử dụng phân trang dựa trên con trỏ với tham số `offset`
- Kiểm tra `next_page` trong phản hồi
- Truyền `offset` từ `next_page.offset` cho yêu cầu tiếp theo

## Các bẫy đã biết

**Định dạng GID**:

- Tất cả ID của Asana là chuỗi (GIDs), không phải số nguyên
- GID là các định danh duy nhất toàn cầu (globally unique identifiers)

**Phạm vi Không gian làm việc**:

- Hầu hết các hoạt động yêu cầu một ngữ cảnh không gian làm việc
- Các tác vụ, dự án và người dùng nằm trong phạm vi không gian làm việc

## Tham khảo Nhanh

| Nhiệm vụ                       | Tool Slug                       | Tham số chính             |
| ------------------------------ | ------------------------------- | ------------------------- |
| Liệt kê không gian làm việc    | ASANA_GET_MULTIPLE_WORKSPACES   | (không có)                |
| Tìm kiếm tác vụ                | ASANA_SEARCH_TASKS_IN_WORKSPACE | workspace, text           |
| Tạo tác vụ                     | ASANA_CREATE_A_TASK             | workspace, name, projects |
| Lấy tác vụ                     | ASANA_GET_A_TASK                | task_gid                  |
| Tạo tác vụ con                 | ASANA_CREATE_SUBTASK            | parent, name              |
| Liệt kê tác vụ con             | ASANA_GET_TASK_SUBTASKS         | task_gid                  |
| Tác vụ dự án                   | ASANA_GET_TASKS_FROM_A_PROJECT  | project_gid               |
| Liệt kê dự án                  | ASANA_GET_WORKSPACE_PROJECTS    | workspace                 |
| Tạo dự án                      | ASANA_CREATE_A_PROJECT          | workspace, name           |
| Lấy dự án                      | ASANA_GET_A_PROJECT             | project_gid               |
| Nhân bản dự án                 | ASANA_DUPLICATE_PROJECT         | project_gid               |
| Liệt kê các phần               | ASANA_GET_SECTIONS_IN_PROJECT   | project_gid               |
| Tạo phần                       | ASANA_CREATE_SECTION_IN_PROJECT | project_gid, name         |
| Thêm vào phần                  | ASANA_ADD_TASK_TO_SECTION       | section, task             |
| Tác vụ trong phần              | ASANA_GET_TASKS_FROM_A_SECTION  | section_gid               |
| Liệt kê nhóm                   | ASANA_GET_TEAMS_IN_WORKSPACE    | workspace_gid             |
| Thành viên nhóm                | ASANA_GET_USERS_FOR_TEAM        | team_gid                  |
| Người dùng không gian làm việc | ASANA_GET_USERS_FOR_WORKSPACE   | workspace_gid             |
| Người dùng hiện tại            | ASANA_GET_CURRENT_USER          | (không có)                |
| Yêu cầu song song              | ASANA_SUBMIT_PARALLEL_REQUESTS  | actions                   |
