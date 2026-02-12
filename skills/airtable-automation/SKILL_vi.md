---
name: airtable-automation
description: "Tự động hóa các nhiệm vụ Airtable thông qua Rube MCP (Composio): bản ghi, cơ sở dữ liệu (bases), bảng, trường, chế độ xem. Luôn tìm kiếm các công cụ trước để có các lược đồ (schemas) hiện tại."
requires:
  mcp: [rube]
---

# Tự động hóa Airtable qua Rube MCP

Tự động hóa các hoạt động Airtable thông qua bộ công cụ Airtable của Composio thông qua Rube MCP.

## Điều kiện tiên quyết

- Rube MCP phải được kết nối (có sẵn `RUBE_SEARCH_TOOLS`)
- Kết nối Airtable đang hoạt động thông qua `RUBE_MANAGE_CONNECTIONS` với bộ công cụ `airtable`
- Luôn gọi `RUBE_SEARCH_TOOLS` trước để lấy lược đồ công cụ hiện tại

## Thiết lập

**Cài đặt Rube MCP**: Thêm `https://rube.app/mcp` làm máy chủ MCP trong cấu hình ứng dụng khách của bạn. Không cần mã khóa API — chỉ cần thêm điểm cuối (endpoint) và nó sẽ hoạt động.

1. Kiểm tra xem Rube MCP có sẵn sàng không bằng cách xác nhận `RUBE_SEARCH_TOOLS` có phản hồi.
2. Gọi `RUBE_MANAGE_CONNECTIONS` với bộ công cụ `airtable`.
3. Nếu kết nối không ở trạng thái ACTIVE (đang hoạt động), hãy làm theo liên kết xác thực được trả về để hoàn tất xác thực Airtable.
4. Xác nhận trạng thái kết nối hiển thị ACTIVE trước khi chạy bất kỳ quy trình công việc nào.

## Quy trình công việc cốt lõi

### 1. Tạo và Quản lý các Bản ghi (Records)

**Khi nào cần sử dụng**: Người dùng muốn tạo, đọc, cập nhật hoặc xóa các bản ghi.

**Trình tự công cụ**:

1. `AIRTABLE_LIST_BASES` - Khám phá các cơ sở dữ liệu (bases) có sẵn [Điều kiện tiên quyết]
2. `AIRTABLE_GET_BASE_SCHEMA` - Kiểm tra cấu trúc bảng [Điều kiện tiên quyết]
3. `AIRTABLE_LIST_RECORDS` - Liệt kê/lọc các bản ghi [Tùy chọn]
4. `AIRTABLE_CREATE_RECORD` / `AIRTABLE_CREATE_RECORDS` - Tạo các bản ghi [Tùy chọn]
5. `AIRTABLE_UPDATE_RECORD` / `AIRTABLE_UPDATE_MULTIPLE_RECORDS` - Cập nhật các bản ghi [Tùy chọn]
6. `AIRTABLE_DELETE_RECORD` / `AIRTABLE_DELETE_MULTIPLE_RECORDS` - Xóa các bản ghi [Tùy chọn]

**Các tham số chính**:

- `baseId`: ID của Base (bắt đầu bằng 'app', ví dụ: 'appXXXXXXXXXXXXXX')
- `tableIdOrName`: ID của bảng (bắt đầu bằng 'tbl') hoặc tên bảng
- `fields`: Đối tượng ánh xạ tên trường với giá trị
- `recordId`: ID của bản ghi (bắt đầu bằng 'rec') để cập nhật/xóa
- `filterByFormula`: Công thức Airtable để lọc
- `typecast`: Đặt thành true để tự động chuyển đổi kiểu dữ liệu

**Các lỗi thường gặp**:

- `pageSize` giới hạn ở 100; sử dụng phân trang bằng offset; thay đổi bộ lọc giữa các trang có thể làm bỏ sót hoặc lặp lại các dòng.
- `CREATE_RECORDS` giới hạn cứng 10 bản ghi mỗi yêu cầu; hãy chia nhỏ nếu nhập số lượng lớn.
- Tên trường có PHÂN BIỆT HOA THƯỜNG và phải khớp chính xác với lược đồ.
- Lỗi 422 `UNKNOWN_FIELD_NAME` khi tên trường sai; 403 cho các vấn đề về quyền hạn.
- `INVALID_MULTIPLE_CHOICE_OPTIONS` có thể yêu cầu `typecast=true`.

### 2. Tìm kiếm và Lọc các Bản ghi

**Khi nào cần sử dụng**: Người dùng muốn tìm các bản ghi cụ thể bằng công thức.

**Trình tự công cụ**:

1. `AIRTABLE_GET_BASE_SCHEMA` - Xác minh tên trường và kiểu dữ liệu [Điều kiện tiên quyết]
2. `AIRTABLE_LIST_RECORDS` - Truy vấn với `filterByFormula` [Bắt buộc]
3. `AIRTABLE_GET_RECORD` - Lấy thông tin chi tiết đầy đủ của bản ghi [Tùy chọn]

**Các tham số chính**:

- `filterByFormula`: Công thức Airtable (ví dụ: `{Status}='Done'`)
- `sort`: Mảng các đối tượng sắp xếp
- `fields`: Mảng các tên trường cần trả về
- `maxRecords`: Tổng số bản ghi tối đa trên tất cả các trang
- `offset`: Con trỏ phân trang từ phản hồi trước đó

**Các lỗi thường gặp**:

- Tên trường trong công thức phải được bao trong `{}` và khớp chính xác với lược đồ.
- Giá trị chuỗi phải được đặt trong dấu nháy: `{Status}='Active'` chứ không phải `{Status}=Active`.
- Lỗi 422 `INVALID_FILTER_BY_FORMULA` do sai cú pháp hoặc trường không tồn tại.
- Giới hạn tốc độ của Airtable: khoảng 5 yêu cầu/giây trên mỗi base; xử lý lỗi 429 với `Retry-After`.

### 3. Quản lý các Trường và Lược đồ (Schema)

**Khi nào cần sử dụng**: Người dùng muốn tạo hoặc sửa đổi các trường trong bảng.

**Trình tự công cụ**:

1. `AIRTABLE_GET_BASE_SCHEMA` - Kiểm tra lược đồ hiện tại [Điều kiện tiên quyết]
2. `AIRTABLE_CREATE_FIELD` - Tạo một trường mới [Tùy chọn]
3. `AIRTABLE_UPDATE_FIELD` - Đổi tên/mô tả một trường [Tùy chọn]
4. `AIRTABLE_UPDATE_TABLE` - Cập nhật siêu dữ liệu (metadata) của bảng [Tùy chọn]

**Các tham số chính**:

- `name`: Tên trường
- `type`: Kiểu trường (singleLineText, number, singleSelect, v.v.)
- `options`: Các tùy chọn cụ thể theo kiểu (ví dụ: các lựa chọn cho singleSelect, độ chính xác cho number)
- `description`: Mô tả trường

**Các lỗi thường gặp**:

- `UPDATE_FIELD` chỉ thay đổi tên/mô tả, KHÔNG thay đổi kiểu/tùy chọn; hãy tạo một trường thay thế và di dời dữ liệu.
- Các trường tính toán (công thức, rollup, lookup) không thể được tạo qua API.
- Lỗi 422 khi các tùy chọn kiểu bị thiếu hoặc sai định dạng.

### 4. Quản lý Bình luận

**Khi nào cần sử dụng**: Người dùng muốn xem hoặc thêm bình luận vào các bản ghi.

**Trình tự công cụ**:

1. `AIRTABLE_LIST_COMMENTS` - Liệt kê các bình luận trên một bản ghi [Bắt buộc]

**Các tham số chính**:

- `baseId`: ID của Base
- `tableIdOrName`: Định danh bảng
- `recordId`: ID của bản ghi (17 ký tự, bắt đầu với 'rec')
- `pageSize`: Số bình luận mỗi trang (tối đa 100)

**Các lỗi thường gặp**:

- ID bản ghi phải chính xác 17 ký tự và bắt đầu bằng 'rec'.

## Các mẫu phổ biến

### Cú pháp Công thức Airtable

**So sánh**:

- `{Status}='Done'` - Bằng
- `{Priority}>1` - Lớn hơn
- `{Name}!=''` - Không rỗng

**Hàm**:

- `AND({A}='x', {B}='y')` - Cả hai điều kiện
- `OR({A}='x', {A}='y')` - Một trong hai điều kiện
- `FIND('test', {Name})>0` - Chứa văn bản
- `IS_BEFORE({Due Date}, TODAY())` - So sánh ngày tháng

**Quy tắc thoát ký tự (Escape)**:

- Dấu nháy đơn trong giá trị: viết hai lần (`{Name}='John''s Company'`)

### Phân trang

- Thiết lập `pageSize` (tối đa 100)
- Kiểm tra phản hồi để lấy chuỗi `offset`
- Chuyển `offset` nguyên văn sang yêu cầu tiếp theo
- Giữ nguyên các bộ lọc/sắp xếp/chế độ xem giữa các trang

## Các lỗi thường gặp đã biết

**Định dạng ID**:

- Base IDs: `appXXXXXXXXXXXXXX` (17 ký tự)
- Table IDs: `tblXXXXXXXXXXXXXX` (17 ký tự)
- Record IDs: `recXXXXXXXXXXXXXX` (17 ký tự)
- Field IDs: `fldXXXXXXXXXXXXXX` (17 ký tự)

**Giới hạn theo lô**:

- `CREATE_RECORDS`: tối đa 10 bản ghi mỗi yêu cầu
- `UPDATE_MULTIPLE_RECORDS`: tối đa 10 bản ghi mỗi yêu cầu
- `DELETE_MULTIPLE_RECORDS`: tối đa 10 bản ghi mỗi yêu cầu

## Tham khảo nhanh

| Nhiệm vụ               | Tên công cụ (Slug)               | Các tham số chính                       |
| ---------------------- | -------------------------------- | --------------------------------------- |
| Liệt kê các base       | AIRTABLE_LIST_BASES              | (không có)                              |
| Lấy lược đồ            | AIRTABLE_GET_BASE_SCHEMA         | baseId                                  |
| Liệt kê bản ghi        | AIRTABLE_LIST_RECORDS            | baseId, tableIdOrName                   |
| Lấy bản ghi            | AIRTABLE_GET_RECORD              | baseId, tableIdOrName, recordId         |
| Tạo bản ghi            | AIRTABLE_CREATE_RECORD           | baseId, tableIdOrName, fields           |
| Tạo nhiều bản ghi      | AIRTABLE_CREATE_RECORDS          | baseId, tableIdOrName, records          |
| Cập nhật bản ghi       | AIRTABLE_UPDATE_RECORD           | baseId, tableIdOrName, recordId, fields |
| Cập nhật nhiều bản ghi | AIRTABLE_UPDATE_MULTIPLE_RECORDS | baseId, tableIdOrName, records          |
| Xóa bản ghi            | AIRTABLE_DELETE_RECORD           | baseId, tableIdOrName, recordId         |
| Tạo trường             | AIRTABLE_CREATE_FIELD            | baseId, tableIdOrName, name, type       |
| Cập nhật trường        | AIRTABLE_UPDATE_FIELD            | baseId, tableIdOrName, fieldId          |
| Cập nhật bảng          | AIRTABLE_UPDATE_TABLE            | baseId, tableIdOrName, name             |
| Liệt kê bình luận      | AIRTABLE_LIST_COMMENTS           | baseId, tableIdOrName, recordId         |
