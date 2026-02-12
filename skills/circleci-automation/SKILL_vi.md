---
name: circleci-automation
description: "Tự động hóa các tác vụ CircleCI qua Rube MCP (Composio): kích hoạt đường ống, giám sát quy trình làm việc/công việc, truy xuất hiện vật và siêu dữ liệu kiểm thử. Luôn tìm kiếm công cụ trước để biết lược đồ hiện tại."
requires:
  mcp: [rube]
---

# Tự động hóa CircleCI qua Rube MCP

Tự động hóa các hoạt động CI/CD của CircleCI thông qua bộ công cụ CircleCI của Composio qua Rube MCP.

## Điều kiện tiên quyết

- Rube MCP phải được kết nối (RUBE_SEARCH_TOOLS có sẵn)
- Kết nối CircleCI hoạt động qua `RUBE_MANAGE_CONNECTIONS` với bộ công cụ `circleci`
- Luôn gọi `RUBE_SEARCH_TOOLS` trước để lấy lược đồ công cụ hiện tại

## Thiết lập

**Lấy Rube MCP**: Thêm `https://rube.app/mcp` làm máy chủ MCP trong cấu hình máy khách của bạn. Không cần khóa API — chỉ cần thêm endpoint và nó sẽ hoạt động.

1. Xác minh Rube MCP có sẵn bằng cách xác nhận `RUBE_SEARCH_TOOLS` phản hồi
2. Gọi `RUBE_MANAGE_CONNECTIONS` với bộ công cụ `circleci`
3. Nếu kết nối không ACTIVE, hãy làm theo liên kết xác thực được trả về để hoàn tất xác thực CircleCI
4. Xác nhận trạng thái kết nối hiển thị ACTIVE trước khi chạy bất kỳ quy trình làm việc nào

## Các Quy trình làm việc Cốt lõi

### 1. Kích hoạt một Đường ống (Pipeline)

**Khi nào sử dụng**: Người dùng muốn bắt đầu một lần chạy đường ống CI/CD mới

**Trình tự công cụ**:

1. `CIRCLECI_TRIGGER_PIPELINE` - Kích hoạt một đường ống mới trên một dự án [Bắt buộc]
2. `CIRCLECI_LIST_WORKFLOWS_BY_PIPELINE_ID` - Giám sát các quy trình làm việc kết quả [Tùy chọn]

**Các tham số chính**:

- `project_slug`: Định danh dự án theo định dạng `gh/org/repo` hoặc `bb/org/repo`
- `branch`: Nhánh Git để chạy đường ống (loại trừ lẫn nhau với tag)
- `tag`: Thẻ Git để chạy đường ống (loại trừ lẫn nhau với branch)
- `parameters`: Các cặp khóa-giá trị tham số đường ống

**Cạm bẫy**:

- Định dạng `project_slug` là `{vcs}/{org}/{repo}` (ví dụ: `gh/myorg/myrepo`)
- `branch` và `tag` loại trừ lẫn nhau; cung cấp cả hai sẽ gây ra lỗi
- Các tham số đường ống phải khớp với các tham số được xác định trong `.circleci/config.yml`
- Việc kích hoạt trả về ID đường ống; quy trình làm việc bắt đầu không đồng bộ

### 2. Giám sát Đường ống và Quy trình làm việc

**Khi nào sử dụng**: Người dùng muốn kiểm tra trạng thái của các đường ống hoặc quy trình làm việc

**Trình tự công cụ**:

1. `CIRCLECI_LIST_PIPELINES_FOR_PROJECT` - Liệt kê các đường ống gần đây cho một dự án [Bắt buộc]
2. `CIRCLECI_LIST_WORKFLOWS_BY_PIPELINE_ID` - Liệt kê các quy trình làm việc trong một đường ống [Bắt buộc]
3. `CIRCLECI_GET_PIPELINE_CONFIG` - Xem cấu hình đường ống đã sử dụng [Tùy chọn]

**Các tham số chính**:

- `project_slug`: Định danh dự án theo định dạng `{vcs}/{org}/{repo}`
- `pipeline_id`: UUID của một đường ống cụ thể
- `branch`: Lọc các đường ống theo tên nhánh
- `page_token`: Con trỏ phân trang cho trang kết quả tiếp theo

**Cạm bẫy**:

- ID đường ống là UUID, không phải ID số
- Các quy trình làm việc kế thừa ID đường ống; một đường ống duy nhất có thể có nhiều quy trình làm việc
- Các trạng thái quy trình làm việc bao gồm: success, running, not_run, failed, error, failing, on_hold, canceled, unauthorized
- `page_token` được trả về trong phản hồi để phân trang; tiếp tục cho đến khi không còn

### 3. Kiểm tra Chi tiết Công việc (Job Details)

**Khi nào sử dụng**: Người dùng muốn đi sâu vào chi tiết thực thi của một công việc cụ thể

**Trình tự công cụ**:

1. `CIRCLECI_LIST_WORKFLOWS_BY_PIPELINE_ID` - Tìm quy trình làm việc chứa công việc [Điều kiện tiên quyết]
2. `CIRCLECI_GET_JOB_DETAILS` - Lấy thông tin công việc chi tiết [Bắt buộc]

**Các tham số chính**:

- `project_slug`: Định danh dự án
- `job_number`: Số công việc (không phải UUID)

**Cạm bẫy**:

- Số công việc là số nguyên, không phải UUID (không giống như ID đường ống và quy trình làm việc)
- Chi tiết công việc bao gồm loại trình thực thi, tính song song, thời gian bắt đầu/kết thúc và trạng thái
- Các trạng thái công việc: success, running, not_run, failed, retried, timedout, infrastructure_fail, canceled

### 4. Truy xuất Hiện vật Xây dựng (Build Artifacts)

**Khi nào sử dụng**: Người dùng muốn tải xuống hoặc liệt kê các hiện vật được tạo bởi một công việc

**Trình tự công cụ**:

1. `CIRCLECI_GET_JOB_DETAILS` - Xác nhận công việc đã hoàn thành thành công [Điều kiện tiên quyết]
2. `CIRCLECI_GET_JOB_ARTIFACTS` - Liệt kê tất cả các hiện vật từ công việc [Bắt buộc]

**Các tham số chính**:

- `project_slug`: Định danh dự án
- `job_number`: Số công việc

**Cạm bẫy**:

- Các hiện vật chỉ có sẵn sau khi công việc hoàn thành
- Mỗi hiện vật có một `path` và `url` để tải xuống
- URL hiện vật có thể yêu cầu tiêu đề xác thực để tải xuống
- Các hiện vật lớn có thể có giới hạn kích thước tải xuống

### 5. Xem xét Kết quả Kiểm thử

**Khi nào sử dụng**: Người dùng muốn kiểm tra kết quả kiểm thử cho một công việc cụ thể

**Trình tự công cụ**:

1. `CIRCLECI_GET_JOB_DETAILS` - Xác minh công việc đã chạy kiểm thử [Điều kiện tiên quyết]
2. `CIRCLECI_GET_TEST_METADATA` - Truy xuất kết quả kiểm thử và siêu dữ liệu [Bắt buộc]

**Các tham số chính**:

- `project_slug`: Định danh dự án
- `job_number`: Số công việc

**Cạm bẫy**:

- Siêu dữ liệu kiểm thử yêu cầu công việc đã tải lên kết quả kiểm thử (định dạng JUnit XML)
- Nếu không có kết quả kiểm thử nào được tải lên, phản hồi sẽ trống
- Siêu dữ liệu kiểm thử bao gồm các trường classname, name, result, message, và run_time
- Các kiểm thử thất bại bao gồm thông báo lỗi trong trường `message`

## Các Mẫu Phổ biến

### Định dạng Project Slug

```
Định dạng: {vcs_type}/{org_name}/{repo_name}
- GitHub:    gh/myorg/myrepo
- Bitbucket: bb/myorg/myrepo
```

### Phân cấp Pipeline -> Workflow -> Job

```
1. Gọi CIRCLECI_LIST_PIPELINES_FOR_PROJECT để lấy các ID đường ống
2. Gọi CIRCLECI_LIST_WORKFLOWS_BY_PIPELINE_ID với pipeline_id
3. Trích xuất số công việc từ chi tiết quy trình làm việc
4. Gọi CIRCLECI_GET_JOB_DETAILS với job_number
```

### Phân trang

- Kiểm tra phản hồi cho trường `next_page_token`
- Truyền mã thông báo dưới dạng `page_token` trong yêu cầu tiếp theo
- Tiếp tục cho đến khi `next_page_token` vắng mặt hoặc null

## Các Cạm bẫy Đã biết

**Định dạng ID**:

- Pipeline ID: UUID (ví dụ: `5034460f-c7c4-4c43-9457-de07e2029e7b`)
- Workflow ID: UUID
- Số công việc: Số nguyên (ví dụ: `123`)
- KHÔNG trộn lẫn UUID và số nguyên giữa các endpoint khác nhau

**Project Slugs**:

- Phải bao gồm tiền tố VCS: `gh/` cho GitHub, `bb/` cho Bitbucket
- Tên tổ chức và kho lưu trữ phân biệt chữ hoa chữ thường
- Định dạng slug không chính xác gây ra lỗi 404

**Giới hạn Tốc độ**:

- API CircleCI có giới hạn tốc độ trên mỗi endpoint
- Thực hiện chờ theo cấp số nhân (exponential backoff) trên các phản hồi 429
- Tránh bỏ phiếu nhanh; sử dụng khoảng thời gian hợp lý (5-10 giây)

## Tham khảo Nhanh

| Nhiệm vụ                       | Slug Công cụ                           | Các Tham số Chính                |
| ------------------------------ | -------------------------------------- | -------------------------------- |
| Kích hoạt đường ống            | CIRCLECI_TRIGGER_PIPELINE              | project_slug, branch, parameters |
| Liệt kê các đường ống          | CIRCLECI_LIST_PIPELINES_FOR_PROJECT    | project_slug, branch             |
| Liệt kê các quy trình làm việc | CIRCLECI_LIST_WORKFLOWS_BY_PIPELINE_ID | pipeline_id                      |
| Lấy cấu hình đường ống         | CIRCLECI_GET_PIPELINE_CONFIG           | pipeline_id                      |
| Lấy chi tiết công việc         | CIRCLECI_GET_JOB_DETAILS               | project_slug, job_number         |
| Lấy hiện vật công việc         | CIRCLECI_GET_JOB_ARTIFACTS             | project_slug, job_number         |
| Lấy siêu dữ liệu kiểm thử      | CIRCLECI_GET_TEST_METADATA             | project_slug, job_number         |
