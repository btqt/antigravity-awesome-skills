---
name: bitbucket-automation
description: "Tự động hóa kho lưu trữ Bitbucket, pull requests, nhánh, issues, và quản lý không gian làm việc thông qua Rube MCP (Composio). Luôn tìm kiếm công cụ trước để có schema hiện tại."
requires:
  mcp: [rube]
---

# Tự động hóa Bitbucket thông qua Rube MCP

Tự động hóa các hoạt động Bitbucket bao gồm quản lý kho lưu trữ, quy trình làm việc pull request, hoạt động nhánh, theo dõi vấn đề (issue tracking), và quản trị không gian làm việc thông qua bộ công cụ Bitbucket của Composio.

## Điều kiện tiên quyết

- Rube MCP phải được kết nối (RUBE_SEARCH_TOOLS khả dụng)
- Kết nối Bitbucket đang hoạt động qua `RUBE_MANAGE_CONNECTIONS` với toolkit `bitbucket`
- Luôn gọi `RUBE_SEARCH_TOOLS` trước tiên để lấy schema công cụ hiện tại

## Cài đặt

**Lấy Rube MCP**: Thêm `https://rube.app/mcp` làm máy chủ MCP trong cấu hình client của bạn. Không cần API key — chỉ cần thêm endpoint và nó sẽ hoạt động.

1. Xác minh Rube MCP khả dụng bằng cách xác nhận `RUBE_SEARCH_TOOLS` phản hồi
2. Gọi `RUBE_MANAGE_CONNECTIONS` với toolkit `bitbucket`
3. Nếu kết nối không ACTIVE, làm theo liên kết xác thực được trả về để hoàn tất xác thực Bitbucket OAuth
4. Xác nhận trạng thái kết nối hiển thị ACTIVE trước khi chạy bất kỳ quy trình làm việc nào

## Các Quy trình làm việc Cốt lõi

### 1. Quản lý Pull Requests

**Khi nào sử dụng**: Người dùng muốn tạo, đánh giá, hoặc kiểm tra pull requests

**Trình tự công cụ**:

1. `BITBUCKET_LIST_WORKSPACES` - Khám phá các không gian làm việc có thể truy cập [Tiên quyết]
2. `BITBUCKET_LIST_REPOSITORIES_IN_WORKSPACE` - Tìm kho lưu trữ mục tiêu [Tiên quyết]
3. `BITBUCKET_LIST_BRANCHES` - Xác minh nhánh nguồn và đích tồn tại [Tiên quyết]
4. `BITBUCKET_CREATE_PULL_REQUEST` - Tạo PR mới với tiêu đề, nhánh nguồn, và người đánh giá tùy chọn [Bắt buộc]
5. `BITBUCKET_LIST_PULL_REQUESTS` - Liệt kê các PR được lọc theo trạng thái (OPEN, MERGED, DECLINED) [Tùy chọn]
6. `BITBUCKET_GET_PULL_REQUEST` - Lấy đầy đủ chi tiết của một PR cụ thể theo ID [Tùy chọn]
7. `BITBUCKET_GET_PULL_REQUEST_DIFF` - Lấy unified diff để đánh giá mã [Tùy chọn]
8. `BITBUCKET_GET_PULL_REQUEST_DIFFSTAT` - Lấy các tệp đã thay đổi với số dòng thêm/xóa [Tùy chọn]

**Tham số chính**:

- `workspace`: Workspace slug hoặc UUID (bắt buộc cho tất cả hoạt động)
- `repo_slug`: Tên kho lưu trữ thân thiện với URL
- `source_branch`: Nhánh có thay đổi để hợp nhất
- `destination_branch`: Nhánh đích (mặc định là nhánh chính của repo nếu bỏ qua)
- `reviewers`: Danh sách các đối tượng với trường `uuid` để gán người đánh giá
- `state`: Bộ lọc cho LIST_PULL_REQUESTS - `OPEN`, `MERGED`, hoặc `DECLINED`
- `max_chars`: Giới hạn cắt ngắn cho GET_PULL_REQUEST_DIFF để xử lý diff lớn

**Cạm bẫy**:

- `reviewers` mong đợi một mảng các đối tượng với khóa `uuid`, KHÔNG phải tên người dùng: `[{"uuid": "{...}"}]`
- Định dạng UUID phải bao gồm dấu ngoặc nhọn: `{123e4567-e89b-12d3-a456-426614174000}`
- `destination_branch` mặc định là nhánh chính của repo nếu bỏ qua, có thể không phải là `main`
- `pull_request_id` là số nguyên cho các hoạt động GET/DIFF nhưng trả về như một phần của danh sách PR
- Diff lớn có thể làm quá tải ngữ cảnh; luôn đặt `max_chars` (ví dụ, 50000) trên GET_PULL_REQUEST_DIFF

### 2. Quản lý Kho lưu trữ và Không gian làm việc

**Khi nào sử dụng**: Người dùng muốn liệt kê, tạo, hoặc xóa kho lưu trữ hoặc khám phá không gian làm việc

**Trình tự công cụ**:

1. `BITBUCKET_LIST_WORKSPACES` - Liệt kê tất cả các không gian làm việc có thể truy cập [Bắt buộc]
2. `BITBUCKET_LIST_REPOSITORIES_IN_WORKSPACE` - Liệt kê repos với bộ lọc BBQL tùy chọn [Bắt buộc]
3. `BITBUCKET_CREATE_REPOSITORY` - Tạo repo mới với ngôn ngữ, quyền riêng tư, và cài đặt dự án [Tùy chọn]
4. `BITBUCKET_DELETE_REPOSITORY` - Xóa vĩnh viễn kho lưu trữ (không thể đảo ngược) [Tùy chọn]
5. `BITBUCKET_LIST_WORKSPACE_MEMBERS` - Liệt kê thành viên để gán người đánh giá hoặc kiểm tra quyền truy cập [Tùy chọn]

**Tham số chính**:

- `workspace`: Workspace slug (tìm qua LIST_WORKSPACES)
- `repo_slug`: Tên thân thiện với URL để tạo/xóa
- `q`: Truy vấn lọc BBQL (ví dụ: `name~"api"`, `project.key="PROJ"`, `is_private=true`)
- `role`: Lọc repos theo vai trò người dùng: `member`, `contributor`, `admin`, `owner`
- `sort`: Trường sắp xếp với tiền tố `-` tùy chọn cho giảm dần (ví dụ: `-updated_on`)
- `is_private`: Boolean cho khả năng hiển thị kho lưu trữ (mặc định là `true`)
- `project_key`: Bitbucket project key; bỏ qua để sử dụng dự án cũ nhất của workspace

**Cạm bẫy**:

- `BITBUCKET_DELETE_REPOSITORY` là **không thể đảo ngược** và không ảnh hưởng đến forks
- Giá trị chuỗi BBQL PHẢI được bao trong dấu ngoặc kép: `name~"my-repo"` không phải `name~my-repo`
- `repository` KHÔNG phải là trường BBQL hợp lệ; sử dụng `name` thay thế
- Phân trang mặc định là 10 kết quả; đặt `pagelen` rõ ràng cho danh sách đầy đủ
- `CREATE_REPOSITORY` mặc định là riêng tư; đặt `is_private: false` cho repos công khai

### 3. Quản lý Issues

**Khi nào sử dụng**: Người dùng muốn tạo, cập nhật, liệt kê, hoặc bình luận về các vấn đề trong kho lưu trữ

**Trình tự công cụ**:

1. `BITBUCKET_LIST_ISSUES` - Liệt kê issues với bộ lọc tùy chọn cho trạng thái, ưu tiên, loại, người được gán [Bắt buộc]
2. `BITBUCKET_CREATE_ISSUE` - Tạo issue mới với tiêu đề, nội dung, mức độ ưu tiên và loại [Bắt buộc]
3. `BITBUCKET_UPDATE_ISSUE` - Sửa đổi thuộc tính issue (trạng thái, ưu tiên, người được gán, v.v.) [Tùy chọn]
4. `BITBUCKET_CREATE_ISSUE_COMMENT` - Thêm bình luận markdown vào issue hiện có [Tùy chọn]
5. `BITBUCKET_DELETE_ISSUE` - Xóa vĩnh viễn một issue [Tùy chọn]

**Tham số chính**:

- `issue_id`: Định danh chuỗi cho issue
- `title`, `content`: Bắt buộc để tạo
- `kind`: `bug`, `enhancement`, `proposal`, hoặc `task`
- `priority`: `trivial`, `minor`, `major`, `critical`, hoặc `blocker`
- `state`: `new`, `open`, `resolved`, `on hold`, `invalid`, `duplicate`, `wontfix`, `closed`
- `assignee`: Tên người dùng Bitbucket cho CREATE; `assignee_account_id` (UUID) cho UPDATE
- `due_on`: Chuỗi ngày định dạng ISO 8601

**Cạm bẫy**:

- Issue tracker phải được bật trên kho lưu trữ (`has_issues: true`) hoặc các cuộc gọi API sẽ thất bại
- `CREATE_ISSUE` sử dụng `assignee` (chuỗi tên người dùng), nhưng `UPDATE_ISSUE` sử dụng `assignee_account_id` (UUID) -- chúng là các trường khác nhau
- `DELETE_ISSUE` là vĩnh viễn không có hoàn tác
- Các giá trị `state` bao gồm khoảng trắng: `"on hold"` không phải `"on_hold"`
- Lọc theo `assignee` trong LIST_ISSUES sử dụng account ID, không phải tên người dùng; sử dụng chuỗi `"null"` cho chưa được gán

### 4. Quản lý Nhánh

**Khi nào sử dụng**: Người dùng muốn tạo nhánh hoặc khám phá cấu trúc nhánh

**Trình tự công cụ**:

1. `BITBUCKET_LIST_BRANCHES` - Liệt kê các nhánh với bộ lọc BBQL tùy chọn và sắp xếp [Bắt buộc]
2. `BITBUCKET_CREATE_BRANCH` - Tạo nhánh mới từ một hàm băm commit cụ thể [Bắt buộc]

**Tham số chính**:

- `name`: Tên nhánh không có tiền tố `refs/heads/` (ví dụ: `feature/new-login`)
- `target_hash`: Hàm băm commit SHA1 đầy đủ để rẽ nhánh từ đó (phải tồn tại trong repo)
- `q`: Bộ lọc BBQL (ví dụ: `name~"feature/"`, `name="main"`)
- `sort`: Sắp xếp theo `name` hoặc `-target.date` (ngày commit giảm dần)
- `pagelen`: 1-100 kết quả mỗi trang (mặc định là 10)

**Cạm bẫy**:

- `CREATE_BRANCH` yêu cầu một hàm băm commit đầy đủ, KHÔNG phải tên nhánh làm `target_hash`
- KHÔNG bao gồm tiền tố `refs/heads/` trong tên nhánh
- Tên nhánh phải tuân theo quy ước đặt tên Bitbucket (chữ và số, `/`, `.`, `_`, `-`)
- Giá trị chuỗi BBQL cần dấu ngoặc kép: `name~"feature/"` không phải `name~feature/`

### 5. Đánh giá Pull Requests với Bình luận

**Khi nào sử dụng**: Người dùng muốn thêm bình luận đánh giá vào pull requests, bao gồm bình luận mã nội dòng

**Trình tự công cụ**:

1. `BITBUCKET_GET_PULL_REQUEST` - Lấy chi tiết PR và xác minh nó tồn tại [Tiên quyết]
2. `BITBUCKET_GET_PULL_REQUEST_DIFF` - Xem xét các thay đổi mã thực tế [Tiên quyết]
3. `BITBUCKET_GET_PULL_REQUEST_DIFFSTAT` - Lấy danh sách các tệp đã thay đổi [Tùy chọn]
4. `BITBUCKET_CREATE_PULL_REQUEST_COMMENT` - Đăng bình luận đánh giá [Bắt buộc]

**Tham số chính**:

- `pull_request_id`: ID chuỗi của PR
- `content_raw`: Văn bản bình luận định dạng Markdown
- `content_markup`: Mặc định là `markdown`; cũng hỗ trợ `plaintext`
- `inline`: Đối tượng với `path`, `from`, `to` cho bình luận mã nội dòng
- `parent_comment_id`: ID số nguyên cho các câu trả lời theo luồng cho các bình luận hiện có

**Cạm bẫy**:

- `pull_request_id` là một chuỗi trong CREATE_PULL_REQUEST_COMMENT nhưng là một số nguyên trong GET_PULL_REQUEST
- Bình luận nội dòng yêu cầu tối thiểu `inline.path`; `from`/`to` là số dòng tùy chọn
- `parent_comment_id` tạo ra một câu trả lời theo luồng; bỏ qua cho các bình luận cấp cao nhất
- Số dòng trong bình luận nội dòng tham chiếu đến diff, không phải tệp nguồn

## Các Mẫu Phổ biến

### Giải quyết ID (ID Resolution)

Luôn giải quyết tên người đọc được thành ID trước khi thao tác:

- **Workspace**: `BITBUCKET_LIST_WORKSPACES` để lấy workspace slugs
- **Repository**: `BITBUCKET_LIST_REPOSITORIES_IN_WORKSPACE` với bộ lọc `q` để tìm repo slugs
- **Branch**: `BITBUCKET_LIST_BRANCHES` để xác minh sự tồn tại của nhánh trước khi tạo PR
- **Members**: `BITBUCKET_LIST_WORKSPACE_MEMBERS` để lấy UUIDs cho gán người đánh giá

### Phân trang

Bitbucket sử dụng phân trang dựa trên trang (không dựa trên con trỏ):

- Sử dụng tham số `page` (bắt đầu từ 1) và `pagelen` (mục mỗi trang)
- Kích thước trang mặc định thường là 10; đặt `pagelen` rõ ràng (tối đa 50 cho PRs, 100 cho cái khác)
- Kiểm tra phản hồi cho URL `next` hoặc tổng số lượng để xác định xem còn trang nào không
- Luôn lặp qua tất cả các trang cho kết quả hoàn chỉnh

### Lọc BBQL

Ngôn ngữ Truy vấn Bitbucket có sẵn trên các endpoint danh sách:

- Giá trị chuỗi PHẢI sử dụng dấu ngoặc kép: `name~"pattern"`
- Toán tử: `=` (chính xác), `~` (chứa), `!=` (không bằng), `>`, `>=`, `<`, `<=`
- Kết hợp với `AND` / `OR`: `name~"api" AND is_private=true`

## Các Cạm bẫy Đã biết

### Định dạng ID

- Workspace: chuỗi slug (ví dụ: `my-workspace`) hoặc UUID trong dấu ngoặc nhọn (`{uuid}`)
- UUID người đánh giá phải bao gồm dấu ngoặc nhọn: `{123e4567-e89b-12d3-a456-426614174000}`
- ID issue là chuỗi; ID PR là số nguyên trong một số công cụ, chuỗi trong công cụ khác
- Hàm băm commit phải là SHA1 đầy đủ (40 ký tự)

### Đặc điểm quái dị của Tham số

- `assignee` vs `assignee_account_id`: CREATE_ISSUE sử dụng tên người dùng, UPDATE_ISSUE sử dụng UUID
- Giá trị `state` cho issues bao gồm khoảng trắng: `"on hold"`, không phải `"on_hold"`
- Bỏ qua `destination_branch` mặc định là nhánh chính của repo, không phải `main` theo nghĩa đen
- BBQL `repository` không phải là trường hợp lệ -- sử dụng `name`

### Giới hạn Tốc độ (Rate Limits)

- Bitbucket Cloud API có giới hạn tốc độ; các hoạt động hàng loạt lớn nên bao gồm độ trễ
- Các yêu cầu được phân trang tính vào giới hạn tốc độ; giảm thiểu việc lấy trang không cần thiết

### Các Hoạt động Phá hủy

- `BITBUCKET_DELETE_REPOSITORY` là không thể đảo ngược và không xóa forks
- `BITBUCKET_DELETE_ISSUE` là vĩnh viễn không có tùy chọn khôi phục
- Luôn xác nhận với người dùng trước khi thực hiện các hoạt động xóa

## Tham khảo Nhanh

| Tác vụ               | Tool Slug                                  | Tham số chính                                              |
| -------------------- | ------------------------------------------ | ---------------------------------------------------------- |
| Liệt kê workspaces   | `BITBUCKET_LIST_WORKSPACES`                | `q`, `sort`                                                |
| Liệt kê repos        | `BITBUCKET_LIST_REPOSITORIES_IN_WORKSPACE` | `workspace`, `q`, `role`                                   |
| Tạo repo             | `BITBUCKET_CREATE_REPOSITORY`              | `workspace`, `repo_slug`, `is_private`                     |
| Xóa repo             | `BITBUCKET_DELETE_REPOSITORY`              | `workspace`, `repo_slug`                                   |
| Liệt kê nhánh        | `BITBUCKET_LIST_BRANCHES`                  | `workspace`, `repo_slug`, `q`                              |
| Tạo nhánh            | `BITBUCKET_CREATE_BRANCH`                  | `workspace`, `repo_slug`, `name`, `target_hash`            |
| Liệt kê PRs          | `BITBUCKET_LIST_PULL_REQUESTS`             | `workspace`, `repo_slug`, `state`                          |
| Tạo PR               | `BITBUCKET_CREATE_PULL_REQUEST`            | `workspace`, `repo_slug`, `title`, `source_branch`         |
| Lấy chi tiết PR      | `BITBUCKET_GET_PULL_REQUEST`               | `workspace`, `repo_slug`, `pull_request_id`                |
| Lấy PR diff          | `BITBUCKET_GET_PULL_REQUEST_DIFF`          | `workspace`, `repo_slug`, `pull_request_id`, `max_chars`   |
| Lấy PR diffstat      | `BITBUCKET_GET_PULL_REQUEST_DIFFSTAT`      | `workspace`, `repo_slug`, `pull_request_id`                |
| Bình luận trên PR    | `BITBUCKET_CREATE_PULL_REQUEST_COMMENT`    | `workspace`, `repo_slug`, `pull_request_id`, `content_raw` |
| Liệt kê issues       | `BITBUCKET_LIST_ISSUES`                    | `workspace`, `repo_slug`, `state`, `priority`              |
| Tạo issue            | `BITBUCKET_CREATE_ISSUE`                   | `workspace`, `repo_slug`, `title`, `content`               |
| Cập nhật issue       | `BITBUCKET_UPDATE_ISSUE`                   | `workspace`, `repo_slug`, `issue_id`                       |
| Bình luận trên issue | `BITBUCKET_CREATE_ISSUE_COMMENT`           | `workspace`, `repo_slug`, `issue_id`, `content`            |
| Xóa issue            | `BITBUCKET_DELETE_ISSUE`                   | `workspace`, `repo_slug`, `issue_id`                       |
| Liệt kê thành viên   | `BITBUCKET_LIST_WORKSPACE_MEMBERS`         | `workspace`                                                |
