---
name: coda-automation
description: "Tự động hóa các tác vụ Coda qua Rube MCP (Composio): quản lý tài liệu, trang, bảng, hàng, công thức, quyền và xuất bản. Luôn tìm kiếm công cụ trước để biết lược đồ hiện tại."
requires:
  mcp: [rube]
---

# Tự động hóa Coda qua Rube MCP

Tự động hóa các hoạt động tài liệu và dữ liệu Coda thông qua bộ công cụ Coda của Composio qua Rube MCP.

## Điều kiện tiên quyết

- Rube MCP phải được kết nối (RUBE_SEARCH_TOOLS có sẵn)
- Kết nối Coda hoạt động qua `RUBE_MANAGE_CONNECTIONS` với bộ công cụ `coda`
- Luôn gọi `RUBE_SEARCH_TOOLS` trước để lấy lược đồ công cụ hiện tại

## Thiết lập

**Lấy Rube MCP**: Thêm `https://rube.app/mcp` làm máy chủ MCP trong cấu hình máy khách của bạn. Không cần khóa API — chỉ cần thêm endpoint và nó sẽ hoạt động.

1. Xác minh Rube MCP có sẵn bằng cách xác nhận `RUBE_SEARCH_TOOLS` phản hồi
2. Gọi `RUBE_MANAGE_CONNECTIONS` với bộ công cụ `coda`
3. Nếu kết nối không ACTIVE, hãy làm theo liên kết xác thực được trả về để hoàn tất xác thực Coda
4. Xác nhận trạng thái kết nối hiển thị ACTIVE trước khi chạy bất kỳ quy trình làm việc nào

## Các Quy trình làm việc Cốt lõi

### 1. Tìm kiếm và Duyệt Tài liệu

**Khi nào sử dụng**: Người dùng muốn tìm, liệt kê hoặc kiểm tra các tài liệu Coda

**Trình tự công cụ**:

1. `CODA_SEARCH_DOCS` hoặc `CODA_LIST_AVAILABLE_DOCS` - Tìm tài liệu [Bắt buộc]
2. `CODA_RESOLVE_BROWSER_LINK` - Giải quyết URL Coda thành ID doc/page/table [Thay thế]
3. `CODA_LIST_PAGES` - Liệt kê các trang trong một tài liệu [Tùy chọn]
4. `CODA_GET_A_PAGE` - Lấy chi tiết trang cụ thể [Tùy chọn]

**Các tham số chính**:

- `query`: Thuật ngữ tìm kiếm để tìm tài liệu
- `isOwner`: Lọc các tài liệu do người dùng sở hữu
- `docId`: ID tài liệu cho các hoạt động trang
- `pageIdOrName`: Định danh hoặc tên trang
- `url`: URL trình duyệt cho các hoạt động giải quyết

**Cạm bẫy**:

- ID tài liệu là các chuỗi chữ và số (ví dụ: 'AbCdEfGhIj')
- `CODA_RESOLVE_BROWSER_LINK` là cách tốt nhất để chuyển đổi URL Coda thành API ID
- Tên trang có thể không duy nhất trong một tài liệu; ưu tiên ID trang
- Kết quả tìm kiếm bao gồm các tài liệu được chia sẻ với người dùng, không chỉ các tài liệu sở hữu

### 2. Làm việc với Bảng và Dữ liệu

**Khi nào sử dụng**: Người dùng muốn đọc, ghi hoặc truy vấn dữ liệu bảng

**Trình tự công cụ**:

1. `CODA_LIST_TABLES` - Liệt kê các bảng trong một tài liệu [Điều kiện tiên quyết]
2. `CODA_LIST_COLUMNS` - Lấy định nghĩa cột cho một bảng [Điều kiện tiên quyết]
3. `CODA_LIST_TABLE_ROWS` - Liệt kê tất cả các hàng với bộ lọc tùy chọn [Bắt buộc]
4. `CODA_SEARCH_ROW` - Tìm kiếm các hàng cụ thể theo truy vấn [Thay thế]
5. `CODA_GET_A_ROW` - Lấy một hàng cụ thể theo ID [Tùy chọn]
6. `CODA_UPSERT_ROWS` - Chèn hoặc cập nhật các hàng trong một bảng [Tùy chọn]
7. `CODA_GET_A_COLUMN` - Lấy chi tiết của một cột cụ thể [Tùy chọn]

**Các tham số chính**:

- `docId`: ID tài liệu chứa bảng
- `tableIdOrName`: Định danh hoặc tên bảng
- `query`: Truy vấn lọc để tìm kiếm hàng
- `rows`: Mảng các đối tượng hàng cho các hoạt động upsert
- `keyColumns`: ID cột được sử dụng để khớp trong quá trình upsert
- `sortBy`: Cột để sắp xếp kết quả
- `useColumnNames`: Sử dụng tên cột thay vì ID trong dữ liệu hàng

**Cạm bẫy**:

- Tên bảng có thể chứa khoảng trắng; mã hóa URL nếu cần
- `CODA_UPSERT_ROWS` thực hiện chèn nếu không khớp trên `keyColumns`, cập nhật nếu tìm thấy khớp
- `keyColumns` phải tham chiếu đến các cột có giá trị duy nhất để upsert đáng tin cậy
- ID cột khác với tên cột; liệt kê các cột trước để ánh xạ tên sang ID
- `useColumnNames: true` cho phép sử dụng tên dễ đọc trong dữ liệu hàng
- Giá trị dữ liệu hàng phải khớp với loại cột (văn bản, số, ngày, v.v.)

### 3. Quản lý Công thức

**Khi nào sử dụng**: Người dùng muốn liệt kê hoặc đánh giá các công thức trong một tài liệu

**Trình tự công cụ**:

1. `CODA_LIST_FORMULAS` - Liệt kê tất cả các công thức được đặt tên trong một tài liệu [Bắt buộc]
2. `CODA_GET_A_FORMULA` - Lấy giá trị hiện tại của một công thức cụ thể [Tùy chọn]

**Các tham số chính**:

- `docId`: ID tài liệu
- `formulaIdOrName`: Định danh hoặc tên công thức

**Cạm bẫy**:

- Các công thức là các tính toán được đặt tên được xác định trong tài liệu
- Giá trị công thức được tính toán phía máy chủ; kết quả phản ánh trạng thái hiện tại
- Tên công thức phân biệt chữ hoa chữ thường

### 4. Xuất Nội dung Tài liệu

**Khi nào sử dụng**: Người dùng muốn xuất một tài liệu hoặc trang sang HTML hoặc Markdown

**Trình tự công cụ**:

1. `CODA_BEGIN_CONTENT_EXPORT` - Bắt đầu một công việc xuất [Bắt buộc]
2. `CODA_CONTENT_EXPORT_STATUS` - Thăm dò trạng thái xuất cho đến khi hoàn tất [Bắt buộc]

**Các tham số chính**:

- `docId`: ID tài liệu để xuất
- `outputFormat`: Định dạng xuất ('html' hoặc 'markdown')
- `pageIdOrName`: Trang cụ thể để xuất (tùy chọn, bỏ qua để xuất toàn bộ tài liệu)
- `requestId`: ID yêu cầu xuất để thăm dò trạng thái

**Cạm bẫy**:

- Xuất là không đồng bộ; thăm dò trạng thái cho đến khi `status` là 'complete'
- Các tài liệu lớn có thể mất thời gian đáng kể để xuất
- URL xuất trong phản hồi đã hoàn thành là tạm thời; tải xuống ngay lập tức
- Thăm dò quá thường xuyên có thể chạm giới hạn tốc độ; sử dụng khoảng thời gian 2-5 giây

### 5. Quản lý Quyền và Chia sẻ

**Khi nào sử dụng**: Người dùng muốn xem hoặc quản lý quyền truy cập tài liệu

**Trình tự công cụ**:

1. `CODA_GET_SHARING_METADATA` - Xem cài đặt chia sẻ hiện tại [Bắt buộc]
2. `CODA_GET_ACL_SETTINGS` - Lấy cài đặt danh sách kiểm soát truy cập [Tùy chọn]
3. `CODA_ADD_PERMISSION` - Cấp quyền truy cập cho người dùng hoặc email [Tùy chọn]

**Các tham số chính**:

- `docId`: ID tài liệu
- `access`: Mức độ quyền ('readonly', 'write', 'comment')
- `principal`: Đối tượng với email hoặc ID người dùng của người nhận
- `suppressEmail`: Có bỏ qua email thông báo chia sẻ hay không

**Cạm bẫy**:

- Các mức độ quyền: 'readonly', 'write', 'comment'
- Thêm quyền sẽ gửi email thông báo theo mặc định; sử dụng `suppressEmail` để ngăn chặn
- Không thể xóa quyền qua API trong mọi trường hợp; kiểm tra cài đặt ACL

### 6. Xuất bản và Tùy chỉnh Tài liệu

**Khi nào sử dụng**: Người dùng muốn xuất bản một tài liệu hoặc quản lý tên miền tùy chỉnh

**Trình tự công cụ**:

1. `CODA_PUBLISH_DOC` - Xuất bản một tài liệu công khai [Bắt buộc]
2. `CODA_UNPUBLISH_DOC` - Hủy xuất bản một tài liệu [Tùy chọn]
3. `CODA_ADD_CUSTOM_DOMAIN` - Thêm tên miền tùy chỉnh cho tài liệu đã xuất bản [Tùy chọn]
4. `CODA_GET_DOC_CATEGORIES` - Lấy các danh mục tài liệu để khám phá [Tùy chọn]

**Các tham số chính**:

- `docId`: ID tài liệu
- `slug`: Slug URL tùy chỉnh cho tài liệu đã xuất bản
- `categoryIds`: ID danh mục cho khả năng khám phá

**Cạm bẫy**:

- Xuất bản làm cho tài liệu có thể truy cập được bởi bất kỳ ai có liên kết
- Tên miền tùy chỉnh yêu cầu cấu hình DNS
- Hủy xuất bản xóa quyền truy cập công khai nhưng giữ lại quyền truy cập đã chia sẻ

## Các Mẫu Phổ biến

### Giải quyết ID

**Doc URL -> Doc ID**:

```
1. Gọi CODA_RESOLVE_BROWSER_LINK với URL Coda
2. Trích xuất docId từ phản hồi
```

**Table name -> Table ID**:

```
1. Gọi CODA_LIST_TABLES với docId
2. Tìm bảng theo tên, trích xuất id
```

**Column name -> Column ID**:

```
1. Gọi CODA_LIST_COLUMNS với docId và tableIdOrName
2. Tìm cột theo tên, trích xuất id
```

### Phân trang

- Coda sử dụng phân trang dựa trên con trỏ với `pageToken`
- Kiểm tra phản hồi cho `nextPageToken`
- Truyền dưới dạng `pageToken` trong yêu cầu tiếp theo cho đến khi vắng mặt
- Kích thước trang mặc định thay đổi theo endpoint

### Mẫu Upsert Hàng

```
1. Gọi CODA_LIST_COLUMNS để lấy ID cột
2. Xây dựng các đối tượng hàng với khóa ID cột và giá trị
3. Đặt keyColumns thành (các) cột định danh duy nhất
4. Gọi CODA_UPSERT_ROWS với rows và keyColumns
```

## Các Cạm bẫy Đã biết

**Định dạng ID**:

- ID Tài liệu: chuỗi chữ và số
- ID Bảng/cột/hàng: chuỗi có tiền tố (ví dụ: 'grid-abc', 'c-xyz')
- Sử dụng RESOLVE_BROWSER_LINK để chuyển đổi URL thành ID

**Loại Dữ liệu**:

- Giá trị hàng phải khớp với loại cột
- Cột ngày mong đợi định dạng ISO 8601
- Cột chọn/chọn nhiều mong đợi các giá trị tùy chọn chính xác
- Cột người mong đợi địa chỉ email

**Giới hạn Tốc độ**:

- API Coda có giới hạn tốc độ trên mỗi mã thông báo
- Thực hiện chờ (backoff) trên các phản hồi 429
- Các hoạt động hàng loạt qua UPSERT_ROWS hiệu quả hơn so với cập nhật riêng lẻ

## Tham khảo Nhanh

| Nhiệm vụ              | Slug Công cụ               | Các Tham số Chính                                 |
| --------------------- | -------------------------- | ------------------------------------------------- |
| Tìm kiếm tài liệu     | CODA_SEARCH_DOCS           | query                                             |
| Liệt kê tài liệu      | CODA_LIST_AVAILABLE_DOCS   | isOwner                                           |
| Giải quyết URL        | CODA_RESOLVE_BROWSER_LINK  | url                                               |
| Liệt kê trang         | CODA_LIST_PAGES            | docId                                             |
| Lấy trang             | CODA_GET_A_PAGE            | docId, pageIdOrName                               |
| Liệt kê bảng          | CODA_LIST_TABLES           | docId                                             |
| Liệt kê cột           | CODA_LIST_COLUMNS          | docId, tableIdOrName                              |
| Liệt kê hàng          | CODA_LIST_TABLE_ROWS       | docId, tableIdOrName                              |
| Tìm kiếm hàng         | CODA_SEARCH_ROW            | docId, tableIdOrName, query                       |
| Lấy hàng              | CODA_GET_A_ROW             | docId, tableIdOrName, rowIdOrName                 |
| Upsert hàng           | CODA_UPSERT_ROWS           | docId, tableIdOrName, rows, keyColumns            |
| Lấy cột               | CODA_GET_A_COLUMN          | docId, tableIdOrName, columnIdOrName              |
| Nhấn nút              | CODA_PUSH_A_BUTTON         | docId, tableIdOrName, rowIdOrName, columnIdOrName |
| Liệt kê công thức     | CODA_LIST_FORMULAS         | docId                                             |
| Lấy công thức         | CODA_GET_A_FORMULA         | docId, formulaIdOrName                            |
| Bắt đầu xuất          | CODA_BEGIN_CONTENT_EXPORT  | docId, outputFormat                               |
| Trạng thái xuất       | CODA_CONTENT_EXPORT_STATUS | docId, requestId                                  |
| Lấy chia sẻ           | CODA_GET_SHARING_METADATA  | docId                                             |
| Thêm quyền            | CODA_ADD_PERMISSION        | docId, access, principal                          |
| Xuất bản tài liệu     | CODA_PUBLISH_DOC           | docId, slug                                       |
| Hủy xuất bản tài liệu | CODA_UNPUBLISH_DOC         | docId                                             |
| Liệt kê gói           | CODA_LIST_PACKS            | (không có)                                        |
