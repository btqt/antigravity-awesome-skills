---
name: clickup-automation
description: "Tự động hóa quản lý dự án ClickUp bao gồm các tác vụ, space, thư mục, danh sách, bình luận và hoạt động nhóm qua Rube MCP (Composio). Luôn tìm kiếm công cụ trước để biết lược đồ hiện tại."
requires:
  mcp: [rube]
---

# Tự động hóa ClickUp qua Rube MCP

Tự động hóa quy trình quản lý dự án ClickUp bao gồm tạo và cập nhật tác vụ, điều hướng phân cấp không gian làm việc, bình luận và quản lý thành viên nhóm thông qua bộ công cụ ClickUp của Composio.

## Điều kiện tiên quyết

- Rube MCP phải được kết nối (RUBE_SEARCH_TOOLS có sẵn)
- Kết nối ClickUp hoạt động qua `RUBE_MANAGE_CONNECTIONS` với bộ công cụ `clickup`
- Luôn gọi `RUBE_SEARCH_TOOLS` trước để lấy lược đồ công cụ hiện tại

## Thiết lập

**Lấy Rube MCP**: Thêm `https://rube.app/mcp` làm máy chủ MCP trong cấu hình máy khách của bạn. Không cần khóa API — chỉ cần thêm endpoint và nó sẽ hoạt động.

1. Xác minh Rube MCP có sẵn bằng cách xác nhận `RUBE_SEARCH_TOOLS` phản hồi
2. Gọi `RUBE_MANAGE_CONNECTIONS` với bộ công cụ `clickup`
3. Nếu kết nối không ACTIVE, hãy làm theo liên kết xác thực được trả về để hoàn tất OAuth ClickUp
4. Xác nhận trạng thái kết nối hiển thị ACTIVE trước khi chạy bất kỳ quy trình làm việc nào

## Các Quy trình làm việc Cốt lõi

### 1. Tạo và Quản lý Tác vụ

**Khi nào sử dụng**: Người dùng muốn tạo tác vụ, tác vụ con, cập nhật thuộc tính tác vụ hoặc liệt kê các tác vụ trong danh sách ClickUp.

**Trình tự công cụ**:

1. `CLICKUP_GET_AUTHORIZED_TEAMS_WORKSPACES` - Lấy ID không gian làm việc/nhóm [Điều kiện tiên quyết]
2. `CLICKUP_GET_SPACES` - Liệt kê các space trong không gian làm việc [Điều kiện tiên quyết]
3. `CLICKUP_GET_FOLDERS` - Liệt kê các thư mục trong một space [Điều kiện tiên quyết]
4. `CLICKUP_GET_FOLDERLESS_LISTS` - Lấy các danh sách không nằm trong thư mục [Tùy chọn]
5. `CLICKUP_GET_LIST` - Xác thực danh sách và kiểm tra các trạng thái có sẵn [Điều kiện tiên quyết]
6. `CLICKUP_CREATE_TASK` - Tạo một tác vụ trong danh sách đích [Bắt buộc]
7. `CLICKUP_CREATE_TASK` (với `parent`) - Tạo tác vụ con dưới một tác vụ cha [Tùy chọn]
8. `CLICKUP_UPDATE_TASK` - Sửa đổi trạng thái tác vụ, người được giao, ngày tháng, mức độ ưu tiên [Tùy chọn]
9. `CLICKUP_GET_TASK` - Truy xuất chi tiết tác vụ đầy đủ [Tùy chọn]
10. `CLICKUP_GET_TASKS` - Liệt kê tất cả các tác vụ trong một danh sách với bộ lọc [Tùy chọn]
11. `CLICKUP_DELETE_TASK` - Xóa vĩnh viễn một tác vụ [Tùy chọn]

**Các tham số chính cho CLICKUP_CREATE_TASK**:

- `list_id`: ID danh sách đích (số nguyên, bắt buộc)
- `name`: Tên tác vụ (chuỗi, bắt buộc)
- `description`: Mô tả chi tiết tác vụ
- `status`: Phải khớp chính xác (phân biệt chữ hoa chữ thường) với tên trạng thái được cấu hình trong danh sách đích
- `priority`: 1 (Khẩn cấp), 2 (Cao), 3 (Bình thường), 4 (Thấp)
- `assignees`: Mảng ID người dùng (số nguyên)
- `due_date`: Dấu thời gian Unix tính bằng mili giây
- `parent`: Chuỗi ID tác vụ cha để tạo tác vụ con
- `tags`: Mảng chuỗi tên thẻ
- `time_estimate`: Thời gian ước tính tính bằng mili giây

**Cạm bẫy**:

- `status` phân biệt chữ hoa chữ thường và phải khớp với trạng thái hiện có trong danh sách; sử dụng `CLICKUP_GET_LIST` để kiểm tra các trạng thái có sẵn
- `due_date` và `start_date` là dấu thời gian Unix tính bằng **mili giây**, không phải giây
- `parent` của tác vụ con phải là một tác vụ (không phải tác vụ con khác) trong cùng một danh sách
- `notify_all` kích hoạt thông báo cho người theo dõi; đặt thành false cho các hoạt động hàng loạt
- Thử lại có thể tạo ra các bản sao; theo dõi ID tác vụ đã tạo để tránh tạo lại
- `custom_item_id` cho các mốc quan trọng (ID 1) tuân theo hạn ngạch gói không gian làm việc

### 2. Điều hướng Phân cấp Không gian làm việc

**Khi nào sử dụng**: Người dùng muốn duyệt hoặc quản lý cấu trúc không gian làm việc ClickUp (Workspaces > Spaces > Folders > Lists).

**Trình tự công cụ**:

1. `CLICKUP_GET_AUTHORIZED_TEAMS_WORKSPACES` - Liệt kê tất cả các không gian làm việc có thể truy cập [Bắt buộc]
2. `CLICKUP_GET_SPACES` - Liệt kê các space trong một không gian làm việc [Bắt buộc]
3. `CLICKUP_GET_SPACE` - Lấy chi tiết cho một space cụ thể [Tùy chọn]
4. `CLICKUP_GET_FOLDERS` - Liệt kê các thư mục trong một space [Bắt buộc]
5. `CLICKUP_GET_FOLDER` - Lấy chi tiết cho một thư mục cụ thể [Tùy chọn]
6. `CLICKUP_CREATE_FOLDER` - Tạo một thư mục mới trong một space [Tùy chọn]
7. `CLICKUP_GET_FOLDERLESS_LISTS` - Liệt kê các danh sách không nằm trong bất kỳ thư mục nào [Bắt buộc]
8. `CLICKUP_GET_LIST` - Lấy chi tiết danh sách bao gồm trạng thái và trường tùy chỉnh [Tùy chọn]

**Các tham số chính**:

- `team_id`: ID không gian làm việc từ GET_AUTHORIZED_TEAMS_WORKSPACES (bắt buộc cho spaces)
- `space_id`: ID Space (bắt buộc cho thư mục và danh sách không thư mục)
- `folder_id`: ID thư mục (bắt buộc cho GET_FOLDER)
- `list_id`: ID danh sách (bắt buộc cho GET_LIST)
- `archived`: Bộ lọc boolean cho các mục đã lưu trữ/đang hoạt động

**Cạm bẫy**:

- Phân cấp ClickUp là: Workspace (Team) > Space > Folder > List > Task
- Các danh sách có thể tồn tại trực tiếp dưới Spaces (không có thư mục) hoặc bên trong Folders
- Phải sử dụng `CLICKUP_GET_FOLDERLESS_LISTS` để tìm các danh sách không nằm trong thư mục; `CLICKUP_GET_FOLDERS` chỉ trả về các thư mục
- `team_id` trong API ClickUp đề cập đến ID Không gian làm việc, không phải nhóm người dùng

### 3. Thêm Bình luận vào Tác vụ

**Khi nào sử dụng**: Người dùng muốn thêm bình luận, xem xét các bình luận hiện có hoặc quản lý các luồng bình luận trên tác vụ.

**Trình tự công cụ**:

1. `CLICKUP_GET_TASK` - Xác minh tác vụ tồn tại và lấy task_id [Điều kiện tiên quyết]
2. `CLICKUP_CREATE_TASK_COMMENT` - Thêm một bình luận mới vào tác vụ [Bắt buộc]
3. `CLICKUP_GET_TASK_COMMENTS` - Liệt kê các bình luận hiện có trên tác vụ [Tùy chọn]
4. `CLICKUP_UPDATE_COMMENT` - Chỉnh sửa văn bản bình luận, người được giao hoặc trạng thái giải quyết [Tùy chọn]

**Các tham số chính cho CLICKUP_CREATE_TASK_COMMENT**:

- `task_id`: Chuỗi ID tác vụ (bắt buộc)
- `comment_text`: Nội dung bình luận với hỗ trợ định dạng ClickUp (bắt buộc)
- `assignee`: ID người dùng để giao bình luận cho (bắt buộc)
- `notify_all`: true/false cho thông báo người theo dõi (bắt buộc)

**Các tham số chính cho CLICKUP_GET_TASK_COMMENTS**:

- `task_id`: Chuỗi ID tác vụ (bắt buộc)
- `start` / `start_id`: Phân trang cho các bình luận cũ hơn (tối đa 25 mỗi trang)

**Cạm bẫy**:

- `CLICKUP_CREATE_TASK_COMMENT` yêu cầu cả bốn trường: `task_id`, `comment_text`, `assignee`, và `notify_all`
- `assignee` trên một bình luận giao bình luận (không phải tác vụ) cho người dùng đó
- Các bình luận được phân trang ở mức 25 mỗi trang; sử dụng `start` (Unix ms) và `start_id` cho các trang cũ hơn
- `CLICKUP_UPDATE_COMMENT` yêu cầu cả bốn trường: `comment_id`, `comment_text`, `assignee`, `resolved`

### 4. Quản lý Thành viên Nhóm và Phân công

**Khi nào sử dụng**: Người dùng muốn xem các thành viên không gian làm việc, kiểm tra việc sử dụng ghế hoặc tra cứu chi tiết người dùng.

**Trình tự công cụ**:

1. `CLICKUP_GET_AUTHORIZED_TEAMS_WORKSPACES` - Liệt kê các không gian làm việc và lấy team_id [Bắt buộc]
2. `CLICKUP_GET_WORKSPACE_SEATS` - Kiểm tra việc sử dụng ghế (thành viên so với khách) [Bắt buộc]
3. `CLICKUP_GET_TEAMS` - Liệt kê các nhóm người dùng trong không gian làm việc [Tùy chọn]
4. `CLICKUP_GET_USER` - Lấy chi tiết cho một người dùng cụ thể (Chỉ doanh nghiệp) [Tùy chọn]
5. `CLICKUP_GET_CUSTOM_ROLES` - Liệt kê các vai trò quyền tùy chỉnh [Tùy chọn]

**Các tham số chính**:

- `team_id`: ID không gian làm việc (bắt buộc cho tất cả các hoạt động nhóm)
- `user_id`: ID người dùng cụ thể cho GET_USER
- `group_ids`: ID nhóm được phân tách bằng dấu phẩy để lọc các nhóm

**Cạm bẫy**:

- `CLICKUP_GET_WORKSPACE_SEATS` trả về số lượng ghế, không phải chi tiết thành viên; phân biệt thành viên với khách
- `CLICKUP_GET_TEAMS` trả về các nhóm người dùng, không phải thành viên không gian làm việc; nhóm trống không có nghĩa là không có thành viên
- `CLICKUP_GET_USER` chỉ có sẵn trên Gói Doanh nghiệp ClickUp
- Phải lặp lại các truy vấn ghế không gian làm việc cho mỗi không gian làm việc trong các thiết lập nhiều không gian làm việc

### 5. Lọc và Truy vấn Tác vụ

**Khi nào sử dụng**: Người dùng muốn tìm các tác vụ với các bộ lọc cụ thể (trạng thái, người được giao, ngày tháng, thẻ, trường tùy chỉnh).

**Trình tự công cụ**:

1. `CLICKUP_GET_TASKS` - Lọc các tác vụ trong một danh sách với nhiều tiêu chí [Bắt buộc]
2. `CLICKUP_GET_TASK` - Lấy chi tiết đầy đủ cho các tác vụ riêng lẻ [Tùy chọn]

**Các tham số chính cho CLICKUP_GET_TASKS**:

- `list_id`: ID danh sách (số nguyên, bắt buộc)
- `statuses`: Mảng các chuỗi trạng thái để lọc
- `assignees`: Mảng các chuỗi ID người dùng
- `tags`: Mảng các chuỗi tên thẻ
- `due_date_gt` / `due_date_lt`: Dấu thời gian Unix tính bằng ms cho phạm vi ngày
- `include_closed`: Boolean để bao gồm các tác vụ đã đóng
- `subtasks`: Boolean để bao gồm các tác vụ con
- `order_by`: "id", "created", "updated", hoặc "due_date"
- `page`: Số trang bắt đầu từ 0 (tối đa 100 tác vụ mỗi trang)

**Cạm bẫy**:

- Chỉ các tác vụ có danh sách trang chủ khớp với `list_id` mới được trả về; các tác vụ trong danh sách con không được bao gồm
- Bộ lọc ngày sử dụng dấu thời gian Unix tính bằng mili giây
- Các chuỗi trạng thái phải khớp chính xác; sử dụng mã hóa URL cho khoảng trắng (ví dụ: "to%20do")
- Đánh số trang bắt đầu từ 0; mỗi trang trả về tối đa 100 tác vụ
- Bộ lọc `custom_fields` chấp nhận một mảng các chuỗi JSON, không phải đối tượng

## Các Mẫu Phổ biến

### Giải quyết ID

Luôn giải quyết tên thành ID thông qua phân cấp:

- **Tên không gian làm việc -> team_id**: `CLICKUP_GET_AUTHORIZED_TEAMS_WORKSPACES` và khớp theo tên
- **Tên Space -> space_id**: `CLICKUP_GET_SPACES` với `team_id`
- **Tên thư mục -> folder_id**: `CLICKUP_GET_FOLDERS` với `space_id`
- **Tên danh sách -> list_id**: Điều hướng thư mục hoặc sử dụng `CLICKUP_GET_FOLDERLESS_LISTS`
- **Tên tác vụ -> task_id**: `CLICKUP_GET_TASKS` với `list_id` và khớp theo tên

### Phân trang

- `CLICKUP_GET_TASKS`: Dựa trên trang với `page` bắt đầu từ 0, tối đa 100 tác vụ mỗi trang
- `CLICKUP_GET_TASK_COMMENTS`: Sử dụng `start` (Unix ms) và `start_id` cho phân trang dựa trên con trỏ, tối đa 25 mỗi trang
- Tiếp tục tìm nạp cho đến khi phản hồi trả về ít mục hơn kích thước trang

## Các Cạm bẫy Đã biết

### Định dạng ID

- ID Không gian làm việc/Nhóm là các số nguyên lớn
- ID Space, thư mục và danh sách là các số nguyên
- ID Tác vụ là các chuỗi chữ và số (ví dụ: "9hz", "abc123")
- ID Người dùng là các số nguyên
- ID Bình luận là các số nguyên

### Giới hạn Tốc độ

- ClickUp thực thi giới hạn tốc độ; tạo tác vụ hàng loạt có thể kích hoạt phản hồi 429
- Tôn trọng tiêu đề `Retry-After` khi hiện diện
- Đặt `notify_all=false` cho các hoạt động hàng loạt để giảm tải thông báo

### Những điều kỳ lạ của tham số

- `team_id` trong API có nghĩa là ID Không gian làm việc, không phải một nhóm người dùng
- `status` trên các tác vụ phân biệt chữ hoa chữ thường và cụ thể cho danh sách
- Ngày tháng là dấu thời gian Unix tính bằng **mili giây** (nhân giây với 1000)
- `priority` là một số nguyên 1-4 (1=Khẩn cấp, 4=Thấp), không phải chuỗi
- `CLICKUP_CREATE_TASK_COMMENT` đánh dấu `assignee` và `notify_all` là bắt buộc
- Để xóa mô tả tác vụ, truyền một khoảng trắng `" "` đến `CLICKUP_UPDATE_TASK`

### Quy tắc Phân cấp

- Tác vụ con không được là tác vụ con của chính nó
- Tác vụ con phải nằm trong cùng một danh sách
- Các danh sách có thể không có thư mục (trực tiếp trong một Space) hoặc bên trong một Folder
- Bảng mục con không được hỗ trợ bởi CLICKUP_CREATE_TASK

## Tham khảo Nhanh

| Nhiệm vụ                    | Slug Công cụ                              | Các Tham số Chính                        |
| --------------------------- | ----------------------------------------- | ---------------------------------------- |
| Liệt kê không gian làm việc | `CLICKUP_GET_AUTHORIZED_TEAMS_WORKSPACES` | (không có)                               |
| Liệt kê spaces              | `CLICKUP_GET_SPACES`                      | `team_id`                                |
| Lấy chi tiết space          | `CLICKUP_GET_SPACE`                       | `space_id`                               |
| Liệt kê thư mục             | `CLICKUP_GET_FOLDERS`                     | `space_id`                               |
| Lấy chi tiết thư mục        | `CLICKUP_GET_FOLDER`                      | `folder_id`                              |
| Tạo thư mục                 | `CLICKUP_CREATE_FOLDER`                   | `space_id`, `name`                       |
| Danh sách không thư mục     | `CLICKUP_GET_FOLDERLESS_LISTS`            | `space_id`                               |
| Lấy chi tiết danh sách      | `CLICKUP_GET_LIST`                        | `list_id`                                |
| Tạo tác vụ                  | `CLICKUP_CREATE_TASK`                     | `list_id`, `name`, `status`, `assignees` |
| Cập nhật tác vụ             | `CLICKUP_UPDATE_TASK`                     | `task_id`, `status`, `priority`          |
| Lấy tác vụ                  | `CLICKUP_GET_TASK`                        | `task_id`, `include_subtasks`            |
| Liệt kê tác vụ              | `CLICKUP_GET_TASKS`                       | `list_id`, `statuses`, `page`            |
| Xóa tác vụ                  | `CLICKUP_DELETE_TASK`                     | `task_id`                                |
| Thêm bình luận              | `CLICKUP_CREATE_TASK_COMMENT`             | `task_id`, `comment_text`, `assignee`    |
| Liệt kê bình luận           | `CLICKUP_GET_TASK_COMMENTS`               | `task_id`, `start`, `start_id`           |
| Cập nhật bình luận          | `CLICKUP_UPDATE_COMMENT`                  | `comment_id`, `comment_text`, `resolved` |
| Ghế không gian làm việc     | `CLICKUP_GET_WORKSPACE_SEATS`             | `team_id`                                |
| Liệt kê nhóm người dùng     | `CLICKUP_GET_TEAMS`                       | `team_id`                                |
| Lấy chi tiết người dùng     | `CLICKUP_GET_USER`                        | `team_id`, `user_id`                     |
| Vai trò tùy chỉnh           | `CLICKUP_GET_CUSTOM_ROLES`                | `team_id`                                |
