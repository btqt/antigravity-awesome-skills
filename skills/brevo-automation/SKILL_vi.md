---
name: brevo-automation
description: "Tự động hóa các tác vụ Brevo (Sendinblue) thông qua Rube MCP (Composio): quản lý chiến dịch email, tạo/chỉnh sửa mẫu, theo dõi người gửi, và giám sát hiệu suất chiến dịch. Luôn tìm kiếm công cụ trước để có schema hiện tại."
requires:
  mcp: [rube]
---

# Tự động hóa Brevo thông qua Rube MCP

Tự động hóa các hoạt động tiếp thị qua email Brevo (trước đây là Sendinblue) thông qua bộ công cụ Brevo của Composio qua Rube MCP.

## Điều kiện tiên quyết

- Rube MCP phải được kết nối (RUBE_SEARCH_TOOLS khả dụng)
- Kết nối Brevo đang hoạt động qua `RUBE_MANAGE_CONNECTIONS` với toolkit `brevo`
- Luôn gọi `RUBE_SEARCH_TOOLS` trước tiên để lấy schema công cụ hiện tại

## Cài đặt

**Lấy Rube MCP**: Thêm `https://rube.app/mcp` làm máy chủ MCP trong cấu hình client của bạn. Không cần API key — chỉ cần thêm endpoint và nó sẽ hoạt động.

1. Xác minh Rube MCP khả dụng bằng cách xác nhận `RUBE_SEARCH_TOOLS` phản hồi
2. Gọi `RUBE_MANAGE_CONNECTIONS` với toolkit `brevo`
3. Nếu kết nối không ACTIVE, làm theo liên kết xác thực được trả về để hoàn tất xác thực Brevo
4. Xác nhận trạng thái kết nối hiển thị ACTIVE trước khi chạy bất kỳ quy trình làm việc nào

## Các Quy trình làm việc Cốt lõi

### 1. Quản lý Chiến dịch Email

**Khi nào sử dụng**: Người dùng muốn liệt kê, xem xét, hoặc cập nhật các chiến dịch email

**Trình tự công cụ**:

1. `BREVO_LIST_EMAIL_CAMPAIGNS` - Liệt kê tất cả các chiến dịch với bộ lọc [Bắt buộc]
2. `BREVO_UPDATE_EMAIL_CAMPAIGN` - Cập nhật nội dung hoặc cài đặt chiến dịch [Tùy chọn]

**Tham số chính cho danh sách**:

- `type`: Loại chiến dịch ('classic' hoặc 'trigger')
- `status`: Trạng thái chiến dịch ('suspended', 'archive', 'sent', 'queued', 'draft', 'inProcess', 'inReview')
- `startDate`/`endDate`: Bộ lọc phạm vi ngày (định dạng YYYY-MM-DDTHH:mm:ss.SSSZ)
- `statistics`: Loại thống kê để bao gồm ('globalStats', 'linksStats', 'statsByDomain')
- `limit`: Kết quả mỗi trang (tối đa 100, mặc định 50)
- `offset`: Offset phân trang
- `sort`: Thứ tự sắp xếp ('asc' hoặc 'desc')
- `excludeHtmlContent`: Đặt `true` để giảm kích thước phản hồi

**Tham số chính cho cập nhật**:

- `campaign_id`: ID chiến dịch dạng số (bắt buộc)
- `name`: Tên chiến dịch
- `subject`: Dòng chủ đề email
- `htmlContent`: Nội dung email HTML (loại trừ lẫn nhau với `htmlUrl`)
- `htmlUrl`: URL đến nội dung HTML
- `sender`: Đối tượng Sender với `name`, `email`, hoặc `id`
- `recipients`: Đối tượng với `listIds` và `exclusionListIds`
- `scheduledAt`: Thời gian gửi được lên lịch (YYYY-MM-DDTHH:mm:ss.SSSZ)

**Cạm bẫy**:

- `startDate` và `endDate` được yêu cầu cùng nhau; cung cấp cả hai hoặc không cái nào
- Bộ lọc ngày chỉ hoạt động khi `status` không được truyền hoặc đặt là 'sent'
- `htmlContent` và `htmlUrl` loại trừ lẫn nhau
- Email `sender` của chiến dịch phải là người gửi đã được xác minh trong Brevo
- Các trường A/B testing (`subjectA`, `subjectB`, `splitRule`, `winnerCriteria`) yêu cầu `abTesting: true`
- `scheduledAt` sử dụng định dạng ISO 8601 đầy đủ với múi giờ

### 2. Tạo và Quản lý Mẫu Email

**Khi nào sử dụng**: Người dùng muốn tạo, chỉnh sửa, liệt kê, hoặc xóa các mẫu email

**Trình tự công cụ**:

1. `BREVO_GET_ALL_EMAIL_TEMPLATES` - Liệt kê tất cả các mẫu [Bắt buộc]
2. `BREVO_CREATE_OR_UPDATE_EMAIL_TEMPLATE` - Tạo mẫu mới hoặc cập nhật mẫu hiện có [Bắt buộc]
3. `BREVO_DELETE_EMAIL_TEMPLATE` - Xóa một mẫu không hoạt động [Tùy chọn]

**Tham số chính cho danh sách**:

- `templateStatus`: Lọc các mẫu hoạt động (`true`) hoặc không hoạt động (`false`)
- `limit`: Kết quả mỗi trang (tối đa 1000, mặc định 50)
- `offset`: Offset phân trang
- `sort`: Thứ tự sắp xếp ('asc' hoặc 'desc')

**Tham số chính cho tạo/cập nhật**:

- `templateId`: Bao gồm để cập nhật; bỏ qua để tạo mới
- `templateName`: Tên hiển thị mẫu (bắt buộc khi tạo)
- `subject`: Dòng chủ đề email (bắt buộc khi tạo)
- `htmlContent`: Nội dung mẫu HTML (tối thiểu 10 ký tự; sử dụng cái này hoặc `htmlUrl`)
- `sender`: Đối tượng Sender với `name` và `email`, hoặc `id` (bắt buộc khi tạo)
- `replyTo`: Địa chỉ email trả lời (Reply-to)
- `isActive`: Kích hoạt hoặc hủy kích hoạt mẫu
- `tag`: Thẻ danh mục cho mẫu

**Cạm bẫy**:

- Khi `templateId` được cung cấp, công cụ cập nhật; khi bỏ qua, nó tạo mới
- Để tạo, `templateName`, `subject`, và `sender` là bắt buộc
- `htmlContent` phải có ít nhất 10 ký tự
- Cá nhân hóa mẫu sử dụng cú pháp `{{contact.ATTRIBUTE}}`
- Chỉ các mẫu không hoạt động mới có thể bị xóa
- `htmlContent` và `htmlUrl` loại trừ lẫn nhau

### 3. Quản lý Người gửi

**Khi nào sử dụng**: Người dùng muốn xem danh tính người gửi được ủy quyền

**Trình tự công cụ**:

1. `BREVO_GET_ALL_SENDERS` - Liệt kê tất cả người gửi đã xác minh [Bắt buộc]

**Tham số chính**: (không có bắt buộc)

**Cạm bẫy**:

- Người gửi phải được xác minh trước khi họ có thể được sử dụng trong các chiến dịch hoặc mẫu
- Xác minh người gửi được thực hiện thông qua giao diện web Brevo, không phải qua API
- ID người gửi có thể được sử dụng trong các trường `sender.id` cho các chiến dịch và mẫu

### 4. Cấu hình Chiến dịch A/B Testing

**Khi nào sử dụng**: Người dùng muốn thiết lập hoặc sửa đổi cài đặt A/B test trên một chiến dịch

**Trình tự công cụ**:

1. `BREVO_LIST_EMAIL_CAMPAIGNS` - Tìm chiến dịch mục tiêu [Tiên quyết]
2. `BREVO_UPDATE_EMAIL_CAMPAIGN` - Cấu hình cài đặt A/B test [Bắt buộc]

**Tham số chính**:

- `campaign_id`: Chiến dịch để cấu hình
- `abTesting`: Đặt thành `true` để bật A/B testing
- `subjectA`: Dòng chủ đề cho biến thể A
- `subjectB`: Dòng chủ đề cho biến thể B
- `splitRule`: Phần trăm phân chia cho thử nghiệm (1-99)
- `winnerCriteria`: 'open' hoặc 'click' để xác định người chiến thắng
- `winnerDelay`: Số giờ chờ trước khi chọn người chiến thắng (1-168)

**Cạm bẫy**:

- A/B testing phải được bật (`abTesting: true`) trước khi thiết lập các trường biến thể
- `splitRule` là tỷ lệ phần trăm liên hệ nhận biến thể A
- `winnerDelay` xác định thời gian thử nghiệm trước khi gửi người chiến thắng cho các liên hệ còn lại
- Chỉ hoạt động với loại chiến dịch 'classic'

## Các Mẫu Phổ biến

### Vòng đời Chiến dịch

```
1. Tạo chiến dịch (trạng thái: draft)
2. Thiết lập người nhận (listIds)
3. Cấu hình nội dung (htmlContent hoặc htmlUrl)
4. Tùy chọn lên lịch (scheduledAt)
5. Gửi hoặc lên lịch qua giao diện người dùng Brevo (cập nhật API có thể đặt scheduledAt)
```

### Phân trang

- Sử dụng `limit` (kích thước trang) và `offset` (chỉ mục bắt đầu)
- Giới hạn mặc định là 50; tối đa thay đổi theo endpoint (100 cho chiến dịch, 1000 cho mẫu)
- Tăng `offset` thêm `limit` mỗi trang
- Kiểm tra `count` trong phản hồi để xác định tổng số có sẵn

### Cá nhân hóa Mẫu

```
- Tên: {{contact.FIRSTNAME}}
- Họ: {{contact.LASTNAME}}
- Thuộc tính tùy chỉnh: {{contact.CUSTOM_ATTRIBUTE}}
- Liên kết gương (Mirror link): {{mirror}}
- Liên kết hủy đăng ký: {{unsubscribe}}
```

## Các Cạm bẫy Đã biết

**Định dạng Ngày**:

- Tất cả các ngày sử dụng ISO 8601 với mili giây: YYYY-MM-DDTHH:mm:ss.SSSZ
- Truyền múi giờ trong định dạng ngày giờ để có kết quả chính xác
- `startDate` và `endDate` phải được sử dụng cùng nhau

**Xác minh Người gửi**:

- Tất cả email người gửi phải được xác minh trong Brevo trước khi sử dụng
- Người gửi chưa xác minh gây ra lỗi tạo/cập nhật chiến dịch
- Sử dụng GET_ALL_SENDERS để kiểm tra người gửi đã xác minh có sẵn

**Giới hạn Tốc độ**:

- API Brevo có giới hạn tốc độ theo gói tài khoản
- Triển khai backoff trên các phản hồi 429
- Các hoạt động mẫu có giới hạn thấp hơn so với các hoạt động đọc

**Phân tích Phản hồi**:

- Dữ liệu phản hồi có thể được lồng dưới `data` hoặc `data.data`
- Phân tích phòng thủ với các mẫu dự phòng
- ID chiến dịch và mẫu là các số nguyên

## Tham khảo Nhanh

| Tác vụ              | Tool Slug                             | Tham số chính                              |
| ------------------- | ------------------------------------- | ------------------------------------------ |
| Liệt kê chiến dịch  | BREVO_LIST_EMAIL_CAMPAIGNS            | type, status, limit, offset                |
| Cập nhật chiến dịch | BREVO_UPDATE_EMAIL_CAMPAIGN           | campaign_id, subject, htmlContent          |
| Liệt kê mẫu         | BREVO_GET_ALL_EMAIL_TEMPLATES         | templateStatus, limit, offset              |
| Tạo mẫu             | BREVO_CREATE_OR_UPDATE_EMAIL_TEMPLATE | templateName, subject, htmlContent, sender |
| Cập nhật mẫu        | BREVO_CREATE_OR_UPDATE_EMAIL_TEMPLATE | templateId, htmlContent                    |
| Xóa mẫu             | BREVO_DELETE_EMAIL_TEMPLATE           | templateId                                 |
| Liệt kê người gửi   | BREVO_GET_ALL_SENDERS                 | (none)                                     |
