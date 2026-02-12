---
name: confluence-automation
description: "Tự động hóa tạo trang Confluence, tìm kiếm nội dung, quản lý không gian, nhãn và điều hướng phân cấp thông qua Rube MCP (Composio). Luôn tìm kiếm công cụ trước để có lược đồ hiện tại."
requires:
  mcp: [rube]
---

# Tự động hóa Confluence qua Rube MCP

Tự động hóa các hoạt động Confluence bao gồm tạo và cập nhật trang, tìm kiếm nội dung với CQL, quản lý không gian, gắn thẻ nhãn và điều hướng phân cấp trang thông qua bộ công cụ Confluence của Composio.

## Điều kiện tiên quyết

- Rube MCP phải được kết nối (`RUBE_SEARCH_TOOLS` có sẵn)
- Kết nối Confluence đang hoạt động qua `RUBE_MANAGE_CONNECTIONS` với bộ công cụ `confluence`
- Luôn gọi `RUBE_SEARCH_TOOLS` trước để lấy lược đồ công cụ hiện tại

## Thiết lập

**Nhận Rube MCP**: Thêm `https://rube.app/mcp` làm máy chủ MCP trong cấu hình máy khách của bạn. Không cần khóa API — chỉ cần thêm điểm cuối và nó hoạt động.

1. Xác minh Rube MCP có sẵn bằng cách xác nhận `RUBE_SEARCH_TOOLS` phản hồi
2. Gọi `RUBE_MANAGE_CONNECTIONS` với bộ công cụ `confluence`
3. Nếu kết nối không ACTIVE, hãy làm theo liên kết xác thực trả về để hoàn tất Confluence OAuth
4. Xác nhận trạng thái kết nối hiển thị ACTIVE trước khi chạy bất kỳ quy trình làm việc nào

## Quy trình làm việc Cốt lõi

### 1. Tạo và Cập nhật Trang

**Khi nào sử dụng**: Người dùng muốn tạo tài liệu mới hoặc cập nhật các trang Confluence hiện có

**Trình tự công cụ**:

1. `CONFLUENCE_GET_SPACES` - Liệt kê các không gian để tìm ID không gian đích [Điều kiện tiên quyết]
2. `CONFLUENCE_SEARCH_CONTENT` - Tìm trang hiện có để tránh trùng lặp hoặc xác định vị trí cha [Tùy chọn]
3. `CONFLUENCE_GET_PAGE_BY_ID` - Lấy nội dung trang hiện tại và số phiên bản trước khi cập nhật [Điều kiện tiên quyết cho cập nhật]
4. `CONFLUENCE_CREATE_PAGE` - Tạo trang mới trong một không gian [Bắt buộc cho tạo mới]
5. `CONFLUENCE_UPDATE_PAGE` - Cập nhật trang hiện có với nội dung mới và phiên bản tăng dần [Bắt buộc cho cập nhật]
6. `CONFLUENCE_ADD_CONTENT_LABEL` - Gắn thẻ trang với các nhãn sau khi tạo [Tùy chọn]

**Các tham số chính**:

- `spaceId`: ID không gian hoặc khóa (ví dụ: `"DOCS"`, `"12345678"`) -- các khóa không gian được tự động chuyển đổi thành ID
- `title`: Tiêu đề trang (phải là duy nhất trong một không gian)
- `parentId`: ID trang cha để tạo các trang con; bỏ qua để đặt dưới trang chủ không gian
- `body.storage.value`: Nội dung HTML/XHTML ở định dạng lưu trữ Confluence
- `body.storage.representation`: Phải là `"storage"` cho các hoạt động tạo
- `version.number`: Đối với cập nhật, phải là phiên bản hiện tại + 1
- `version.message`: Mô tả thay đổi tùy chọn

**Cạm bẫy**:

- Confluence thực thi tiêu đề trang duy nhất trên mỗi không gian; tạo trang có tiêu đề trùng lặp sẽ thất bại
- `UPDATE_PAGE` yêu cầu `version.number` được đặt thành phiên bản hiện tại + 1; luôn lấy phiên bản hiện tại trước với `GET_PAGE_BY_ID`
- Nội dung phải ở định dạng lưu trữ Confluence (XHTML), không phải văn bản thuần túy hoặc Markdown
- `CREATE_PAGE` sử dụng `body.storage.value` trong khi `UPDATE_PAGE` sử dụng `body.value` với `body.representation`
- `GET_PAGE_BY_ID` yêu cầu ID dạng số (long ID), không phải UUID hoặc chuỗi

### 2. Tìm kiếm Nội dung

**Khi nào sử dụng**: Người dùng muốn tìm trang, bài đăng blog hoặc nội dung trên khắp Confluence

**Trình tự công cụ**:

1. `CONFLUENCE_SEARCH_CONTENT` - Tìm kiếm từ khóa với xếp hạng mức độ liên quan thông minh [Bắt buộc]
2. `CONFLUENCE_CQL_SEARCH` - Tìm kiếm nâng cao sử dụng Ngôn ngữ Truy vấn Confluence (CQL) [Thay thế]
3. `CONFLUENCE_GET_PAGE_BY_ID` - Hydrate nội dung đầy đủ cho các kết quả tìm kiếm đã chọn [Tùy chọn]
4. `CONFLUENCE_GET_PAGES` - Duyệt các trang được sắp xếp theo ngày khi mức độ liên quan tìm kiếm yếu [Dự phòng]

**Các tham số chính cho SEARCH_CONTENT**:

- `query`: Văn bản tìm kiếm khớp với tiêu đề trang với xếp hạng thông minh
- `spaceKey`: Giới hạn tìm kiếm trong một không gian cụ thể
- `limit`: Kết quả tối đa (mặc định 25, tối đa 250)
- `start`: Độ lệch phân trang (dựa trên 0)

**Các tham số chính cho CQL_SEARCH**:

- `cql`: Chuỗi truy vấn CQL (ví dụ: `text ~ "API docs" AND space = DOCS AND type = page`)
- `expand`: Các thuộc tính phân tách bằng dấu phẩy (ví dụ: `content.space`, `content.body.storage`)
- `excerpt`: `highlight`, `indexed`, hoặc `none`
- `limit`: Kết quả tối đa (tối đa 250; giảm xuống 25-50 khi sử dụng mở rộng nội dung)

**Các toán tử và trường CQL**:

- Trường: `text`, `title`, `label`, `space`, `type`, `creator`, `lastModified`, `created`, `ancestor`
- Toán tử: `=`, `!=`, `~` (chứa), `!~`, `>`, `<`, `>=`, `<=`, `IN`, `NOT IN`
- Hàm: `currentUser()`, `now("-7d")`, `now("-30d")`
- Ví dụ: `title ~ "meeting" AND lastModified > now("-7d") ORDER BY lastModified DESC`

**Cạm bẫy**:

- `CONFLUENCE_SEARCH_CONTENT` lấy tối đa 300 trang và áp dụng lọc phía máy khách -- không phải là tìm kiếm toàn văn thực sự
- `CONFLUENCE_CQL_SEARCH` là tìm kiếm toàn văn thực sự; sử dụng `text ~ "term"` cho tìm kiếm nội dung thân bài
- HTTP 429 giới hạn tốc độ có thể xảy ra; điều tiết xuống ~2 yêu cầu/giây với backoff
- Sử dụng mở rộng nội dung trong CQL_SEARCH có thể giảm kết quả tối đa xuống 25-50
- Lập chỉ mục tìm kiếm không ngay lập tức; các trang mới tạo có thể không xuất hiện

### 3. Quản lý Không gian

**Khi nào sử dụng**: Người dùng muốn liệt kê, tạo hoặc kiểm tra các không gian Confluence

**Trình tự công cụ**:

1. `CONFLUENCE_GET_SPACES` - Liệt kê tất cả các không gian với bộ lọc tùy chọn [Bắt buộc]
2. `CONFLUENCE_GET_SPACE_BY_ID` - Lấy siêu dữ liệu chi tiết cho một không gian cụ thể [Tùy chọn]
3. `CONFLUENCE_CREATE_SPACE` - Tạo không gian mới với khóa và tên [Tùy chọn]
4. `CONFLUENCE_GET_SPACE_PROPERTIES` - Truy xuất siêu dữ liệu tùy chỉnh được lưu trữ dưới dạng thuộc tính không gian [Tùy chọn]
5. `CONFLUENCE_GET_SPACE_CONTENTS` - Liệt kê trang, bài đăng blog hoặc tệp đính kèm trong một không gian [Tùy chọn]
6. `CONFLUENCE_GET_LABELS_FOR_SPACE` - Liệt kê nhãn trên một không gian [Tùy chọn]

**Các tham số chính**:

- `key`: Khóa không gian -- chỉ chữ và số, không có gạch dưới hoặc gạch nối (ví dụ: `DOCS`, `PROJECT1`)
- `name`: Tên không gian dễ đọc
- `type`: `global` hoặc `personal`
- `status`: `current` (hoạt động) hoặc `archived`
- `spaceKey`: Cho GET_SPACE_CONTENTS, lọc theo khóa không gian
- `id`: ID không gian dạng số cho GET_SPACE_BY_ID (KHÔNG phải khóa không gian)

**Cạm bẫy**:

- Khóa không gian chỉ được chứa chữ và số (không có gạch dưới, gạch nối hoặc ký tự đặc biệt)
- `GET_SPACE_BY_ID` yêu cầu ID không gian dạng số, không phải khóa không gian; sử dụng `GET_SPACES` để tìm ID số
- URL không gian có thể nhấp có thể cần lắp ráp: nối `_links.webui` (tương đối) với `_links.base`
- Phân trang mặc định là 25; đặt `limit` một cách rõ ràng (tối đa 200 cho không gian)

### 4. Điều hướng Phân cấp Trang và Nhãn

**Khi nào sử dụng**: Người dùng muốn khám phá cây trang, trang con, tổ tiên hoặc quản lý nhãn

**Trình tự công cụ**:

1. `CONFLUENCE_SEARCH_CONTENT` - Tìm ID trang đích [Điều kiện tiên quyết]
2. `CONFLUENCE_GET_CHILD_PAGES` - Liệt kê con trực tiếp của trang cha [Bắt buộc]
3. `CONFLUENCE_GET_PAGE_ANCESTORS` - Lấy chuỗi tổ tiên đầy đủ cho một trang [Tùy chọn]
4. `CONFLUENCE_GET_LABELS_FOR_PAGE` - Liệt kê nhãn trên một trang cụ thể [Tùy chọn]
5. `CONFLUENCE_ADD_CONTENT_LABEL` - Thêm nhãn vào một trang [Tùy chọn]
6. `CONFLUENCE_GET_LABELS_FOR_SPACE_CONTENT` - Liệt kê nhãn trên toàn bộ nội dung trong một không gian [Tùy chọn]
7. `CONFLUENCE_GET_PAGE_VERSIONS` - Kiểm toán lịch sử chỉnh sửa cho một trang [Tùy chọn]

**Các tham số chính**:

- `id`: ID trang cho trang con, tổ tiên, nhãn và phiên bản
- `cursor`: Con trỏ phân trang mờ cho GET_CHILD_PAGES (từ `_links.next`)
- `limit`: Mục trên mỗi trang (tối đa 250 cho trang con)
- `sort`: Tùy chọn sắp xếp trang con: `id`, `-id`, `created-date`, `-created-date`, `modified-date`, `-modified-date`, `child-position`, `-child-position`

**Cạm bẫy**:

- `GET_CHILD_PAGES` chỉ trả về con trực tiếp, không phải hậu duệ lồng nhau; đệ quy cho cây đầy đủ
- Phân trang cho GET_CHILD_PAGES sử dụng phân trang dựa trên con trỏ (không phải start/limit)
- Xác minh ID trang chính xác từ tìm kiếm trước khi sử dụng làm cha; tìm kiếm có thể trả về các tiêu đề tương tự
- `GET_PAGE_VERSIONS` yêu cầu ID trang, không phải số phiên bản

## Các Mẫu Phổ biến

### Phân giải ID

Luôn phân giải tên dễ đọc thành ID trước khi thực hiện:

- **Khóa không gian -> ID Không gian**: `CONFLUENCE_GET_SPACES` với bộ lọc `spaceKey`, hoặc `CREATE_PAGE` chấp nhận khóa không gian trực tiếp
- **Tiêu đề trang -> ID Trang**: `CONFLUENCE_SEARCH_CONTENT` với tham số `query`, sau đó trích xuất ID trang
- **ID Không gian từ URL**: Trích xuất ID số từ URL Confluence hoặc sử dụng GET_SPACES

### Phân trang

Confluence sử dụng hai kiểu phân trang:

- **Dựa trên Offset** (hầu hết các điểm cuối): `start` (độ lệch dựa trên 0) + `limit` (kích thước trang). Tăng `start` thêm `limit` cho đến khi kết quả trả về ít hơn `limit`.
- **Dựa trên Cursor** (GET_CHILD_PAGES, GET_PAGES): Sử dụng `cursor` từ `_links.next` trong phản hồi. Tiếp tục cho đến khi không còn liên kết `next`.

### Định dạng Nội dung

- Các trang sử dụng định dạng lưu trữ Confluence (XHTML), không phải Markdown
- Các phần tử cơ bản: `<p>`, `<h1>`-`<h6>`, `<strong>`, `<em>`, `<code>`, `<ul>`, `<ol>`, `<li>`
- Bảng: Cấu trúc `<table><tbody><tr><th>` / `<td>`
- Macro: `<ac:structured-macro ac:name="code">` cho khối mã, v.v.
- Luôn bọc nội dung trong các thẻ XHTML thích hợp

## Các Cạm bẫy Đã biết

### Định dạng ID

- ID Không gian là số (ví dụ: `557060`); khóa không gian là chuỗi ngắn (ví dụ: `DOCS`)
- ID Trang là giá trị số dài (long) cho GET_PAGE_BY_ID; một số công cụ chấp nhận định dạng UUID
- `GET_SPACE_BY_ID` yêu cầu ID số, không phải khóa không gian
- `GET_PAGE_BY_ID` nhận số nguyên, không phải chuỗi

### Giới hạn Tốc độ

- HTTP 429 có thể xảy ra trên các điểm cuối tìm kiếm; tôn trọng tiêu đề Retry-After
- Điều tiết xuống ~2 yêu cầu/giây với backoff theo cấp số nhân và jitter
- Mở rộng nội dung trong CQL_SEARCH giảm giới hạn kết quả xuống 25-50

### Định dạng Nội dung

- Nội dung phải là định dạng lưu trữ Confluence (XHTML), không phải Markdown hoặc văn bản thuần túy
- XHTML không hợp lệ sẽ khiến tạo/cập nhật trang thất bại
- `CREATE_PAGE` lồng nội dung dưới `body.storage.value`; `UPDATE_PAGE` sử dụng `body.value` + `body.representation`

### Xung đột Phiên bản

- Cập nhật yêu cầu số phiên bản tiếp theo chính xác (hiện tại + 1)
- Chỉnh sửa đồng thời có thể gây ra xung đột phiên bản; luôn lấy phiên bản hiện tại ngay trước khi cập nhật
- Thay đổi tiêu đề trong khi cập nhật vẫn phải là duy nhất trong không gian

## Tham khảo Nhanh

| Tác vụ                 | Slug Công cụ                      | Tham số Chính                    |
| ---------------------- | --------------------------------- | -------------------------------- |
| Liệt kê không gian     | `CONFLUENCE_GET_SPACES`           | `type`, `status`, `limit`        |
| Lấy không gian theo ID | `CONFLUENCE_GET_SPACE_BY_ID`      | `id`                             |
| Tạo không gian         | `CONFLUENCE_CREATE_SPACE`         | `key`, `name`, `type`            |
| Nội dung không gian    | `CONFLUENCE_GET_SPACE_CONTENTS`   | `spaceKey`, `type`, `status`     |
| Thuộc tính không gian  | `CONFLUENCE_GET_SPACE_PROPERTIES` | `id`, `key`                      |
| Tìm kiếm nội dung      | `CONFLUENCE_SEARCH_CONTENT`       | `query`, `spaceKey`, `limit`     |
| Tìm kiếm CQL           | `CONFLUENCE_CQL_SEARCH`           | `cql`, `expand`, `limit`         |
| Liệt kê trang          | `CONFLUENCE_GET_PAGES`            | `spaceId`, `sort`, `limit`       |
| Lấy trang theo ID      | `CONFLUENCE_GET_PAGE_BY_ID`       | `id` (integer)                   |
| Tạo trang              | `CONFLUENCE_CREATE_PAGE`          | `title`, `spaceId`, `body`       |
| Cập nhật trang         | `CONFLUENCE_UPDATE_PAGE`          | `id`, `title`, `body`, `version` |
| Xóa trang              | `CONFLUENCE_DELETE_PAGE`          | `id`                             |
| Trang con              | `CONFLUENCE_GET_CHILD_PAGES`      | `id`, `limit`, `sort`            |
| Tổ tiên trang          | `CONFLUENCE_GET_PAGE_ANCESTORS`   | `id`                             |
| Nhãn trang             | `CONFLUENCE_GET_LABELS_FOR_PAGE`  | `id`                             |
| Thêm nhãn              | `CONFLUENCE_ADD_CONTENT_LABEL`    | content ID, label                |
| Phiên bản trang        | `CONFLUENCE_GET_PAGE_VERSIONS`    | `id`                             |
| Nhãn không gian        | `CONFLUENCE_GET_LABELS_FOR_SPACE` | space ID                         |
