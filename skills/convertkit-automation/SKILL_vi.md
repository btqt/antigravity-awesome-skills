---
name: convertkit-automation
description: "Tự động hóa các tác vụ ConvertKit (Kit) qua Rube MCP (Composio): quản lý người đăng ký, thẻ, chương trình phát sóng và thống kê phát sóng. Luôn tìm kiếm công cụ trước để có lược đồ hiện tại."
requires:
  mcp: [rube]
---

# Tự Động Hóa ConvertKit (Kit) qua Rube MCP

Tự động hóa các hoạt động tiếp thị qua email của ConvertKit (hiện được gọi là Kit) thông qua bộ công cụ Kit của Composio qua Rube MCP.

## Điều kiện tiên quyết

- Rube MCP phải được kết nối (RUBE_SEARCH_TOOLS có sẵn)
- Kết nối Kit hoạt động qua `RUBE_MANAGE_CONNECTIONS` với bộ công cụ `kit`
- Luôn gọi `RUBE_SEARCH_TOOLS` trước để nhận các lược đồ công cụ hiện tại

## Thiết lập

**Nhận Rube MCP**: Thêm `https://rube.app/mcp` làm máy chủ MCP trong cấu hình máy khách của bạn. Không cần khóa API — chỉ cần thêm điểm cuối và nó sẽ hoạt động.

1. Xác minh Rube MCP có sẵn bằng cách xác nhận `RUBE_SEARCH_TOOLS` phản hồi
2. Gọi `RUBE_MANAGE_CONNECTIONS` với bộ công cụ `kit`
3. Nếu kết nối không ACTIVE, hãy làm theo liên kết xác thực được trả về để hoàn tất xác thực Kit
4. Xác nhận trạng thái kết nối hiển thị ACTIVE trước khi chạy bất kỳ quy trình làm việc nào

## Các Quy trình làm việc Cốt lõi

### 1. Liệt kê và Tìm kiếm Người đăng ký

**Khi nào sử dụng**: Người dùng muốn duyệt, tìm kiếm hoặc lọc người đăng ký email

**Trình tự công cụ**:

1. `KIT_LIST_SUBSCRIBERS` - Liệt kê người đăng ký với các bộ lọc và phân trang [Bắt buộc]

**Các tham số chính**:

- `status`: Lọc theo trạng thái ('active' hoặc 'inactive')
- `email_address`: Email chính xác để tìm kiếm
- `created_after`/`created_before`: Bộ lọc phạm vi ngày (YYYY-MM-DD)
- `updated_after`/`updated_before`: Bộ lọc phạm vi ngày (YYYY-MM-DD)
- `sort_field`: Sắp xếp theo 'id', 'cancelled_at', hoặc 'updated_at'
- `sort_order`: 'asc' hoặc 'desc'
- `per_page`: Kết quả mỗi trang (tối thiểu 1)
- `after`/`before`: Chuỗi con trỏ cho phân trang
- `include_total_count`: Đặt thành 'true' để nhận tổng số người đăng ký

**Cạm bẫy**:

- Nếu `sort_field` là 'cancelled_at', `status` phải được đặt thành 'cancelled'
- Bộ lọc ngày sử dụng định dạng YYYY-MM-DD (không có thành phần thời gian)
- `email_address` là khớp chính xác; tìm kiếm email một phần không được hỗ trợ
- Phân trang sử dụng cách tiếp cận dựa trên con trỏ với các chuỗi con trỏ `after`/`before`
- `include_total_count` là chuỗi 'true', không phải boolean

### 2. Quản lý Thẻ Người đăng ký

**Khi nào sử dụng**: Người dùng muốn gắn thẻ người đăng ký để phân đoạn

**Trình tự công cụ**:

1. `KIT_LIST_SUBSCRIBERS` - Tìm ID người đăng ký theo email [Tiền đề]
2. `KIT_TAG_SUBSCRIBER` - Liên kết người đăng ký với một thẻ [Bắt buộc]
3. `KIT_LIST_TAG_SUBSCRIBERS` - Liệt kê người đăng ký cho một thẻ cụ thể [Tùy chọn]

**Các tham số chính cho việc gắn thẻ**:

- `tag_id`: ID thẻ dạng số (bắt buộc)
- `subscriber_id`: ID người đăng ký dạng số (bắt buộc)

**Cạm bẫy**:

- Cả `tag_id` và `subscriber_id` phải là số nguyên dương
- ID thẻ phải tham chiếu đến các thẻ hiện có; thẻ được tạo qua giao diện web Kit
- Gắn thẻ một người đăng ký đã được gắn thẻ là idempotent (không có lỗi)
- ID người đăng ký được trả về từ LIST_SUBSCRIBERS; sử dụng bộ lọc `email_address` để tìm người đăng ký cụ thể

### 3. Hủy đăng ký một Người đăng ký

**Khi nào sử dụng**: Người dùng muốn hủy đăng ký một người đăng ký khỏi tất cả các liên lạc

**Trình tự công cụ**:

1. `KIT_LIST_SUBSCRIBERS` - Tìm ID người đăng ký [Tiền đề]
2. `KIT_DELETE_SUBSCRIBER` - Hủy đăng ký người đăng ký [Bắt buộc]

**Các tham số chính**:

- `id`: ID người đăng ký (bắt buộc, số nguyên dương)

**Cạm bẫy**:

- Điều này hủy đăng ký người đăng ký vĩnh viễn khỏi TẤT CẢ các liên lạc email
- Dữ liệu lịch sử của người đăng ký được giữ lại nhưng họ sẽ không còn nhận được email nữa
- Hoạt động là idempotent; hủy đăng ký một người đăng ký đã hủy đăng ký thành công mà không có lỗi
- Trả về phản hồi trống (HTTP 204 No Content) khi thành công
- ID người đăng ký phải tồn tại; ID không tồn tại trả về 404

### 4. Liệt kê và Xem Chương trình phát sóng

**Khi nào sử dụng**: Người dùng muốn duyệt các chương trình phát sóng email hoặc nhận chi tiết của một chương trình cụ thể

**Trình tự công cụ**:

1. `KIT_LIST_BROADCASTS` - Liệt kê tất cả các chương trình phát sóng với phân trang [Bắt buộc]
2. `KIT_GET_BROADCAST` - Nhận thông tin chi tiết cho một chương trình phát sóng cụ thể [Tùy chọn]
3. `KIT_GET_BROADCAST_STATS` - Nhận thống kê hiệu suất cho một chương trình phát sóng [Tùy chọn]

**Các tham số chính cho việc liệt kê**:

- `per_page`: Kết quả mỗi trang (1-500)
- `after`/`before`: Chuỗi con trỏ cho phân trang
- `include_total_count`: Đặt thành 'true' cho tổng số lượng

**Các tham số chính cho chi tiết**:

- `id`: ID chương trình phát sóng (bắt buộc, số nguyên dương)

**Cạm bẫy**:

- `per_page` tối đa là 500 cho các chương trình phát sóng
- Thống kê phát sóng chỉ có sẵn cho các chương trình phát sóng đã gửi
- Các bản nháp phát sóng sẽ không có thống kê
- ID chương trình phát sóng là số nguyên

### 5. Xóa một Chương trình phát sóng

**Khi nào sử dụng**: Người dùng muốn xóa vĩnh viễn một chương trình phát sóng

**Trình tự công cụ**:

1. `KIT_LIST_BROADCASTS` - Tìm chương trình phát sóng để xóa [Tiền đề]
2. `KIT_GET_BROADCAST` - Xác minh đó là chương trình phát sóng chính xác [Tùy chọn]
3. `KIT_DELETE_BROADCAST` - Xóa vĩnh viễn chương trình phát sóng [Bắt buộc]

**Các tham số chính**:

- `id`: ID chương trình phát sóng (bắt buộc)

**Cạm bẫy**:

- Việc xóa là vĩnh viễn và không thể hoàn tác
- Xóa một chương trình phát sóng đã gửi sẽ xóa nó nhưng không hủy gửi email
- Xác nhận ID chương trình phát sóng trước khi xóa

## Các Mẫu Phổ biến

### Tra cứu Người đăng ký theo Email

```
1. Call KIT_LIST_SUBSCRIBERS with email_address='user@example.com'
2. Extract subscriber ID from the response
3. Use ID for tagging, unsubscribing, or other operations
```

### Phân trang

Kit sử dụng phân trang dựa trên con trỏ:

- Kiểm tra phản hồi cho giá trị con trỏ `after`
- Truyền con trỏ dưới dạng tham số `after` trong yêu cầu tiếp theo
- Tiếp tục cho đến khi không còn con trỏ nào được trả về
- Sử dụng `include_total_count: 'true'` để theo dõi tiến độ

### Phân đoạn Dựa trên Thẻ

```
1. Create tags in Kit web UI
2. Use KIT_TAG_SUBSCRIBER to assign tags to subscribers
3. Use KIT_LIST_TAG_SUBSCRIBERS to view subscribers per tag
```

## Các Cạm bẫy Đã biết

**Định dạng ID**:

- ID người đăng ký: số nguyên dương (ví dụ: 3887204736)
- ID thẻ: số nguyên dương
- ID chương trình phát sóng: số nguyên dương
- Tất cả ID là số, không phải chuỗi

**Giá trị Trạng thái**:

- Trạng thái người đăng ký: 'active', 'inactive', 'cancelled'
- Một số hoạt động bị hạn chế bởi trạng thái (ví dụ: sắp xếp theo cancelled_at yêu cầu status='cancelled')

**Tham số Chuỗi vs Boolean**:

- `include_total_count` là chuỗi 'true', không phải boolean true
- `sort_order` là chuỗi enum: 'asc' hoặc 'desc'

**Giới hạn Tốc độ**:

- Kit API có giới hạn tốc độ cho mỗi tài khoản
- Triển khai backoff trên phản hồi 429
- Các hoạt động hàng loạt nên được điều chỉnh tốc độ phù hợp

**Phân tích Phản hồi**:

- Dữ liệu phản hồi có thể được lồng dưới `data` hoặc `data.data`
- Phân tích phòng thủ với các mẫu dự phòng
- Giá trị con trỏ là chuỗi mờ; sử dụng chính xác như được trả về

## Tham khảo Nhanh

| Nhiệm vụ                       | Slug Công cụ             | Tham số Chính                   |
| ------------------------------ | ------------------------ | ------------------------------- |
| Liệt kê người đăng ký          | KIT_LIST_SUBSCRIBERS     | status, email_address, per_page |
| Gắn thẻ người đăng ký          | KIT_TAG_SUBSCRIBER       | tag_id, subscriber_id           |
| Liệt kê người đăng ký thẻ      | KIT_LIST_TAG_SUBSCRIBERS | tag_id                          |
| Hủy đăng ký                    | KIT_DELETE_SUBSCRIBER    | id                              |
| Liệt kê chương trình phát sóng | KIT_LIST_BROADCASTS      | per_page, after                 |
| Nhận chương trình phát sóng    | KIT_GET_BROADCAST        | id                              |
| Nhận thống kê phát sóng        | KIT_GET_BROADCAST_STATS  | id                              |
| Xóa chương trình phát sóng     | KIT_DELETE_BROADCAST     | id                              |
