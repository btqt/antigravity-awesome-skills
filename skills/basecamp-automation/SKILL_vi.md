---
name: basecamp-automation
description: "Tự động hóa quản lý dự án Basecamp, to-dos, tin nhắn, con người, và tổ chức danh sách to-do thông qua Rube MCP (Composio). Luôn tìm kiếm công cụ trước để có schema hiện tại."
requires:
  mcp: [rube]
---

# Tự động hóa Basecamp thông qua Rube MCP

Tự động hóa các hoạt động Basecamp bao gồm quản lý dự án, tạo danh sách việc cần làm (to-do list), quản lý tác vụ, đăng bài lên bảng tin (message board), quản lý con người, và tổ chức nhóm to-do thông qua bộ công cụ Basecamp của Composio.

## Điều kiện tiên quyết

- Rube MCP phải được kết nối (RUBE_SEARCH_TOOLS khả dụng)
- Kết nối Basecamp đang hoạt động qua `RUBE_MANAGE_CONNECTIONS` với toolkit `basecamp`
- Luôn gọi `RUBE_SEARCH_TOOLS` trước tiên để lấy schema công cụ hiện tại

## Cài đặt

**Lấy Rube MCP**: Thêm `https://rube.app/mcp` làm máy chủ MCP trong cấu hình client của bạn. Không cần API key — chỉ cần thêm endpoint và nó sẽ hoạt động.

1. Xác minh Rube MCP khả dụng bằng cách xác nhận `RUBE_SEARCH_TOOLS` phản hồi
2. Gọi `RUBE_MANAGE_CONNECTIONS` với toolkit `basecamp`
3. Nếu kết nối không ACTIVE, làm theo liên kết xác thực được trả về để hoàn tất xác thực Basecamp OAuth
4. Xác nhận trạng thái kết nối hiển thị ACTIVE trước khi chạy bất kỳ quy trình làm việc nào

## Các Quy trình làm việc Cốt lõi

### 1. Quản lý Danh sách To-Do và Tác vụ

**Khi nào sử dụng**: Người dùng muốn tạo danh sách to-do, thêm tác vụ, hoặc tổ chức công việc trong một dự án Basecamp

**Trình tự công cụ**:

1. `BASECAMP_GET_PROJECTS` - Liệt kê các dự án để tìm bucket_id mục tiêu [Tiên quyết]
2. `BASECAMP_GET_BUCKETS_TODOSETS` - Lấy to-do set trong một dự án [Tiên quyết]
3. `BASECAMP_GET_BUCKETS_TODOSETS_TODOLISTS` - Liệt kê các danh sách to-do hiện có để tránh trùng lặp [Tùy chọn]
4. `BASECAMP_POST_BUCKETS_TODOSETS_TODOLISTS` - Tạo danh sách to-do mới trong một to-do set [Bắt buộc để tạo danh sách]
5. `BASECAMP_GET_BUCKETS_TODOLISTS` - Lấy chi tiết của một danh sách to-do cụ thể [Tùy chọn]
6. `BASECAMP_POST_BUCKETS_TODOLISTS_TODOS` - Tạo một mục to-do trong danh sách to-do [Bắt buộc để tạo tác vụ]
7. `BASECAMP_CREATE_TODO` - Công cụ thay thế để tạo to-do riêng lẻ [Thay thế]
8. `BASECAMP_GET_BUCKETS_TODOLISTS_TODOS` - Liệt kê các to-do trong một danh sách to-do [Tùy chọn]

**Tham số chính để tạo danh sách to-do**:

- `bucket_id`: Project/bucket ID số nguyên (từ GET_PROJECTS)
- `todoset_id`: To-do set ID số nguyên (từ GET_BUCKETS_TODOSETS)
- `name`: Tiêu đề của danh sách to-do (bắt buộc)
- `description`: Mô tả định dạng HTML (hỗ trợ Rich text)

**Tham số chính để tạo to-do**:

- `bucket_id`: Project/bucket ID số nguyên
- `todolist_id`: To-do list ID số nguyên
- `content`: Nội dung to-do (bắt buộc)
- `description`: Chi tiết HTML về to-do
- `assignee_ids`: Mảng các ID người được giao (số nguyên)
- `due_on`: Ngày đến hạn định dạng `YYYY-MM-DD`
- `starts_on`: Ngày bắt đầu định dạng `YYYY-MM-DD`
- `notify`: Boolean để thông báo cho người được giao (mặc định là false)
- `completion_subscriber_ids`: ID người được thông báo khi hoàn thành

**Cạm bẫy**:

- Một dự án (bucket) có thể chứa nhiều to-do sets; chọn sai `todoset_id` sẽ tạo danh sách trong phần sai
- Luôn kiểm tra các danh sách to-do hiện có trước khi tạo để tránh tên gần giống
- Payload thành công bao gồm URLs hướng người dùng (`app_url`, `app_todos_url`); ưu tiên trả về những cái này hơn là raw ID
- Tất cả ID (`bucket_id`, `todoset_id`, `todolist_id`) đều là số nguyên, không phải chuỗi
- Mô tả chỉ hỗ trợ định dạng HTML, không phải Markdown

### 2. Đăng và Quản lý Tin nhắn

**Khi nào sử dụng**: Người dùng muốn đăng tin nhắn lên bảng tin dự án hoặc cập nhật tin nhắn hiện có

**Trình tự công cụ**:

1. `BASECAMP_GET_PROJECTS` - Tìm dự án mục tiêu và bucket_id [Tiên quyết]
2. `BASECAMP_GET_MESSAGE_BOARD` - Lấy ID bảng tin cho dự án [Tiên quyết]
3. `BASECAMP_CREATE_MESSAGE` - Tạo tin nhắn mới trên bảng [Bắt buộc]
4. `BASECAMP_POST_BUCKETS_MESSAGE_BOARDS_MESSAGES` - Công cụ tạo tin nhắn thay thế [Dự phòng]
5. `BASECAMP_GET_MESSAGE` - Đọc tin nhắn cụ thể theo ID [Tùy chọn]
6. `BASECAMP_PUT_BUCKETS_MESSAGES` - Cập nhật tin nhắn hiện có [Tùy chọn]

**Tham số chính**:

- `bucket_id`: Project/bucket ID số nguyên
- `message_board_id`: Message board ID số nguyên (từ GET_MESSAGE_BOARD)
- `subject`: Tiêu đề tin nhắn (bắt buộc)
- `content`: Thân tin nhắn HTML
- `status`: Đặt thành `"active"` để xuất bản ngay lập tức
- `category_id`: Phân loại loại tin nhắn (tùy chọn)
- `subscriptions`: Mảng các ID người cần thông báo; bỏ qua để thông báo cho tất cả thành viên dự án

**Cạm bẫy**:

- `status="draft"` có thể tạo ra lỗi HTTP 400; sử dụng `status="active"` như tùy chọn đáng tin cậy
- `bucket_id` và `message_board_id` phải thuộc về cùng một dự án; không khớp sẽ thất bại hoặc định tuyến sai
- Nội dung tin nhắn chỉ hỗ trợ thẻ HTML; không phải Markdown
- Cập nhật qua `PUT_BUCKETS_MESSAGES` thay thế toàn bộ thân tin nhắn -- bao gồm nội dung đầy đủ đã sửa, không chỉ là phần khác biệt
- Ưu tiên `app_url` từ phản hồi cho các liên kết xác nhận hướng người dùng
- Cả `CREATE_MESSAGE` và `POST_BUCKETS_MESSAGE_BOARDS_MESSAGES` đều làm cùng một việc; sử dụng CREATE_MESSAGE trước và dự phòng sang POST nếu nó thất bại

### 3. Quản lý Con người và Quyền truy cập

**Khi nào sử dụng**: Người dùng muốn liệt kê mọi người, quản lý quyền truy cập dự án, hoặc thêm người dùng mới

**Trình tự công cụ**:

1. `BASECAMP_GET_PEOPLE` - Liệt kê tất cả những người hiển thị với người dùng hiện tại [Bắt buộc]
2. `BASECAMP_GET_PROJECTS` - Tìm dự án mục tiêu [Tiên quyết]
3. `BASECAMP_LIST_PROJECT_PEOPLE` - Liệt kê những người trong một dự án cụ thể [Bắt buộc]
4. `BASECAMP_GET_PROJECTS_PEOPLE` - Thay thế để liệt kê thành viên dự án [Thay thế]
5. `BASECAMP_PUT_PROJECTS_PEOPLE_USERS` - Cấp hoặc thu hồi quyền truy cập dự án [Bắt buộc cho thay đổi quyền truy cập]

**Tham số chính cho PUT_PROJECTS_PEOPLE_USERS**:

- `project_id`: Project ID số nguyên
- `grant`: Mảng các ID người (số nguyên) để thêm vào dự án
- `revoke`: Mảng các ID người (số nguyên) để xóa khỏi dự án
- `create`: Mảng các đối tượng với `name`, `email_address`, và tùy chọn `company_name`, `title` cho người dùng mới
- Ít nhất một trong `grant`, `revoke`, hoặc `create` phải được cung cấp

**Cạm bẫy**:

- Person ID là số nguyên; luôn giải quyết tên thành ID qua GET_PEOPLE trước
- `project_id` cho quản lý con người cũng giống như `bucket_id` cho các hoạt động khác
- `LIST_PROJECT_PEOPLE` và `GET_PROJECTS_PEOPLE` gần như giống hệt nhau; sử dụng cái nào cũng được
- Tạo người dùng qua `create` cũng cấp cho họ quyền truy cập dự án trong một bước

### 4. Tổ chức To-Do với Nhóm

**Khi nào sử dụng**: Người dùng muốn tổ chức các to-do trong một danh sách thành các nhóm được mã màu

**Trình tự công cụ**:

1. `BASECAMP_GET_PROJECTS` - Tìm dự án mục tiêu [Tiên quyết]
2. `BASECAMP_GET_BUCKETS_TODOLISTS` - Lấy chi tiết danh sách to-do [Tiên quyết]
3. `BASECAMP_GET_TODOLIST_GROUPS` - Liệt kê các nhóm hiện có trong danh sách to-do [Tùy chọn]
4. `BASECAMP_GET_BUCKETS_TODOLISTS_GROUPS` - Liệt kê nhóm thay thế [Thay thế]
5. `BASECAMP_POST_BUCKETS_TODOLISTS_GROUPS` - Tạo nhóm mới trong danh sách to-do [Bắt buộc]
6. `BASECAMP_CREATE_TODOLIST_GROUP` - Công cụ tạo nhóm thay thế [Thay thế]

**Tham số chính**:

- `bucket_id`: Project/bucket ID số nguyên
- `todolist_id`: To-do list ID số nguyên
- `name`: Tiêu đề nhóm (bắt buộc)
- `color`: Định danh màu trực quan -- một trong các: `white`, `red`, `orange`, `yellow`, `green`, `blue`, `aqua`, `purple`, `gray`, `pink`, `brown`
- `status`: Bộ lọc để liệt kê -- `"archived"` hoặc `"trashed"` (bỏ qua cho các nhóm hoạt động)

**Cạm bẫy**:

- `POST_BUCKETS_TODOLISTS_GROUPS` và `CREATE_TODOLIST_GROUP` gần như giống hệt nhau; sử dụng cái nào cũng được
- Giá trị màu phải từ bảng màu cố định; giá trị hex/rgb tùy ý không được hỗ trợ
- Nhóm là các phần con trong một danh sách to-do, không phải thực thể độc lập

### 5. Duyệt và Kiểm tra Dự án

**Khi nào sử dụng**: Người dùng muốn liệt kê các dự án, lấy chi tiết dự án, hoặc khám phá cấu trúc dự án

**Trình tự công cụ**:

1. `BASECAMP_GET_PROJECTS` - Liệt kê tất cả các dự án đang hoạt động [Bắt buộc]
2. `BASECAMP_GET_PROJECT` - Lấy chi tiết toàn diện cho một dự án cụ thể [Tùy chọn]
3. `BASECAMP_GET_PROJECTS_BY_PROJECT_ID` - Lấy chi tiết dự án thay thế [Thay thế]

**Tham số chính**:

- `status`: Lọc theo `"archived"` hoặc `"trashed"`; bỏ qua cho các dự án đang hoạt động
- `project_id`: Project ID số nguyên để lấy chi tiết

**Cạm bẫy**:

- Các dự án được sắp xếp theo thứ tự tạo gần đây nhất trước
- Phản hồi bao gồm mảng `dock` với các công cụ (todoset, message_board, v.v.) và ID của chúng
- Sử dụng ID công cụ dock để tìm `todoset_id`, `message_board_id`, v.v. cho các hoạt động tiếp theo

## Các Mẫu Phổ biến

### Giải quyết ID (ID Resolution)

Basecamp sử dụng cấu trúc ID phân cấp. Luôn giải quyết từ trên xuống dưới:

- **Project (bucket_id)**: `BASECAMP_GET_PROJECTS` -- tìm theo tên, lấy `id`
- **To-do set (todoset_id)**: Tìm trong dự án dock hoặc qua `BASECAMP_GET_BUCKETS_TODOSETS`
- **Message board (message_board_id)**: Tìm trong dự án dock hoặc qua `BASECAMP_GET_MESSAGE_BOARD`
- **To-do list (todolist_id)**: `BASECAMP_GET_BUCKETS_TODOSETS_TODOLISTS`
- **Con người (person_id)**: `BASECAMP_GET_PEOPLE` hoặc `BASECAMP_LIST_PROJECT_PEOPLE`
- Lưu ý: `bucket_id` và `project_id` tham chiếu đến cùng một thực thể trong các ngữ cảnh khác nhau

### Phân trang

Basecamp sử dụng phân trang dựa trên trang trên các endpoint liệt kê:

- Headers phản hồi hoặc body có thể chỉ ra còn trang khả dụng
- `GET_PROJECTS`, `GET_BUCKETS_TODOSETS_TODOLISTS`, và các endpoint liệt kê trả về kết quả phân trang
- Tiếp tục fetch cho đến khi không còn kết quả nào được trả về

### Định dạng Nội dung

- Tất cả các trường văn bản phong phú (rich text) sử dụng HTML, không phải Markdown
- Bọc nội dung trong các thẻ `<div>`; sử dụng `<strong>`, `<em>`, `<ul>`, `<ol>`, `<li>`, `<a>`, v.v.
- Ví dụ: `<div><strong>Quan trọng:</strong> Hoàn thành trước thứ Sáu</div>`

## Các Cạm bẫy Đã biết

### Định dạng ID

- Tất cả ID Basecamp là số nguyên, không phải chuỗi hay UUIDs
- `bucket_id` = `project_id` (cùng thực thể, tên tham số khác nhau giữa các công cụ)
- To-do set IDs, to-do list IDs, và message board IDs được tìm thấy trong mảng `dock` của dự án
- Person IDs là số nguyên; giải quyết tên thông qua `GET_PEOPLE` trước khi thực hiện các hoạt động

### Trường Status

- `status="draft"` cho tin nhắn có thể gây ra lỗi HTTP 400; luôn sử dụng `status="active"`
- Bộ lọc trạng thái Project/to-do list: `"archived"`, `"trashed"`, hoặc bỏ qua cho active

### Định dạng Nội dung

- Chỉ HTML, không bao giờ là Markdown
- Cập nhật thay thế toàn bộ body, không phải diff một phần
- Các thẻ HTML không hợp lệ có thể bị loại bỏ âm thầm

### Giới hạn Tốc độ (Rate Limits)

- Basecamp API có giới hạn tốc độ; giãn cách các yêu cầu liên tiếp nhanh
- Các dự án lớn với nhiều to-do nên được phân trang cẩn thận

### Xử lý URL

- Ưu tiên `app_url` từ phản hồi API cho các liên kết hướng người dùng
- Không tự xây dựng URLs Basecamp thủ công từ IDs

## Tham khảo Nhanh

| Tác vụ                    | Tool Slug                                       | Tham số chính                                        |
| ------------------------- | ----------------------------------------------- | ---------------------------------------------------- |
| Liệt kê dự án             | `BASECAMP_GET_PROJECTS`                         | `status`                                             |
| Lấy dự án                 | `BASECAMP_GET_PROJECT`                          | `project_id`                                         |
| Lấy chi tiết dự án        | `BASECAMP_GET_PROJECTS_BY_PROJECT_ID`           | `project_id`                                         |
| Lấy to-do set             | `BASECAMP_GET_BUCKETS_TODOSETS`                 | `bucket_id`, `todoset_id`                            |
| Liệt kê danh sách to-do   | `BASECAMP_GET_BUCKETS_TODOSETS_TODOLISTS`       | `bucket_id`, `todoset_id`                            |
| Lấy danh sách to-do       | `BASECAMP_GET_BUCKETS_TODOLISTS`                | `bucket_id`, `todolist_id`                           |
| Tạo danh sách to-do       | `BASECAMP_POST_BUCKETS_TODOSETS_TODOLISTS`      | `bucket_id`, `todoset_id`, `name`                    |
| Tạo to-do                 | `BASECAMP_POST_BUCKETS_TODOLISTS_TODOS`         | `bucket_id`, `todolist_id`, `content`                |
| Tạo to-do (alt)           | `BASECAMP_CREATE_TODO`                          | `bucket_id`, `todolist_id`, `content`                |
| Liệt kê to-do             | `BASECAMP_GET_BUCKETS_TODOLISTS_TODOS`          | `bucket_id`, `todolist_id`                           |
| Liệt kê nhóm to-do        | `BASECAMP_GET_TODOLIST_GROUPS`                  | `bucket_id`, `todolist_id`                           |
| Tạo nhóm to-do            | `BASECAMP_POST_BUCKETS_TODOLISTS_GROUPS`        | `bucket_id`, `todolist_id`, `name`, `color`          |
| Tạo nhóm to-do (alt)      | `BASECAMP_CREATE_TODOLIST_GROUP`                | `bucket_id`, `todolist_id`, `name`                   |
| Lấy bảng tin              | `BASECAMP_GET_MESSAGE_BOARD`                    | `bucket_id`, `message_board_id`                      |
| Tạo tin nhắn              | `BASECAMP_CREATE_MESSAGE`                       | `bucket_id`, `message_board_id`, `subject`, `status` |
| Tạo tin nhắn (alt)        | `BASECAMP_POST_BUCKETS_MESSAGE_BOARDS_MESSAGES` | `bucket_id`, `message_board_id`, `subject`           |
| Lấy tin nhắn              | `BASECAMP_GET_MESSAGE`                          | `bucket_id`, `message_id`                            |
| Cập nhật tin nhắn         | `BASECAMP_PUT_BUCKETS_MESSAGES`                 | `bucket_id`, `message_id`                            |
| Liệt kê tất cả mọi người  | `BASECAMP_GET_PEOPLE`                           | (không có)                                           |
| Liệt kê người trong dự án | `BASECAMP_LIST_PROJECT_PEOPLE`                  | `project_id`                                         |
| Quản lý quyền truy cập    | `BASECAMP_PUT_PROJECTS_PEOPLE_USERS`            | `project_id`, `grant`, `revoke`, `create`            |
