---
name: cal-com-automation
description: "Tự động hóa các tác vụ Cal.com qua Rube MCP (Composio): quản lý đặt chỗ, kiểm tra tính khả dụng, cấu hình webhooks, và xử lý nhóm. Luôn tìm kiếm công cụ trước để có schemas hiện tại."
requires:
  mcp: [rube]
---

# Tự động hóa Cal.com qua Rube MCP

Tự động hóa các hoạt động lập lịch Cal.com thông qua bộ công cụ Cal của Composio qua Rube MCP.

## Điều kiện tiên quyết

- Rube MCP phải được kết nối (RUBE_SEARCH_TOOLS có sẵn)
- Kết nối Cal.com đang hoạt động qua `RUBE_MANAGE_CONNECTIONS` với toolkit `cal`
- Luôn gọi `RUBE_SEARCH_TOOLS` trước để lấy schemas công cụ hiện tại

## Thiết lập

**Lấy Rube MCP**: Thêm `https://rube.app/mcp` làm máy chủ MCP trong cấu hình khách của bạn. Không cần khóa API — chỉ cần thêm endpoint và nó hoạt động.

1. Xác minh Rube MCP có sẵn bằng cách xác nhận `RUBE_SEARCH_TOOLS` phản hồi
2. Gọi `RUBE_MANAGE_CONNECTIONS` với toolkit `cal`
3. Nếu kết nối không ACTIVE, làm theo liên kết xác thực được trả về để hoàn tất xác thực Cal.com
4. Xác nhận trạng thái kết nối hiển thị ACTIVE trước khi chạy bất kỳ quy trình làm việc nào

## Quy trình làm việc Cốt lõi

### 1. Quản lý Đặt chỗ (Manage Bookings)

**Khi nào sử dụng**: Người dùng muốn liệt kê, tạo, hoặc xem xét các đặt chỗ

**Trình tự công cụ**:

1. `CAL_FETCH_ALL_BOOKINGS` - Liệt kê tất cả các đặt chỗ với bộ lọc [Bắt buộc]
2. `CAL_POST_NEW_BOOKING_REQUEST` - Tạo một đặt chỗ mới [Tùy chọn]

**Các tham số chính để liệt kê**:

- `status`: Lọc theo trạng thái đặt chỗ ('upcoming', 'recurring', 'past', 'cancelled', 'unconfirmed')
- `afterStart`: Lọc các đặt chỗ sau ngày này (ISO 8601)
- `beforeEnd`: Lọc các đặt chỗ trước ngày này (ISO 8601)

**Các tham số chính để tạo**:

- `eventTypeId`: ID loại sự kiện cho đặt chỗ
- `start`: Thời gian bắt đầu đặt chỗ (ISO 8601)
- `end`: Thời gian kết thúc đặt chỗ (ISO 8601)
- `name`: Tên người tham dự
- `email`: Email người tham dự
- `timeZone`: Múi giờ người tham dự (định dạng IANA)
- `language`: Mã ngôn ngữ người tham dự
- `metadata`: Đối tượng metadata bổ sung

**Cạm bẫy**:

- Bộ lọc ngày sử dụng định dạng ISO 8601 với múi giờ (ví dụ: '2024-01-15T09:00:00Z')
- `eventTypeId` phải tham chiếu đến một loại sự kiện hợp lệ, đang hoạt động
- Tạo đặt chỗ yêu cầu khớp với một khe thời gian có sẵn; kiểm tra tính khả dụng trước
- Múi giờ phải là chuỗi múi giờ IANA hợp lệ (ví dụ: 'America/New_York')
- Giá trị bộ lọc trạng thái là các chuỗi cụ thể; giá trị không hợp lệ trả về kết quả trống

### 2. Kiểm tra Tính khả dụng (Check Availability)

**Khi nào sử dụng**: Người dùng muốn tìm thời gian rảnh/bận hoặc các khe đặt chỗ có sẵn

**Trình tự công cụ**:

1. `CAL_RETRIEVE_CALENDAR_BUSY_TIMES` - Lấy các khối thời gian bận [Bắt buộc]
2. `CAL_GET_AVAILABLE_SLOTS_INFO` - Lấy các khe có sẵn cụ thể [Bắt buộc]

**Các tham số chính**:

- `dateFrom`: Ngày bắt đầu kiểm tra tính khả dụng (YYYY-MM-DD)
- `dateTo`: Ngày kết thúc kiểm tra tính khả dụng (YYYY-MM-DD)
- `eventTypeId`: Loại sự kiện để kiểm tra các khe
- `timeZone`: Múi giờ cho phản hồi tính khả dụng
- `loggedInUsersTz`: Múi giờ của người dùng đang yêu cầu

**Cạm bẫy**:

- Thời gian bận hiển thị khi người dùng KHÔNG có sẵn
- Các khe có sẵn là cụ thể cho thời lượng và cấu hình của loại sự kiện
- Phạm vi ngày nên hợp lý (không phải trước nhiều tháng) để có kết quả chính xác
- Múi giờ ảnh hưởng đến cách hiển thị các khe; luôn chỉ định rõ ràng
- Tính khả dụng phản ánh các tích hợp lịch (Google Calendar, Outlook, v.v.)

### 3. Cấu hình Webhooks

**Khi nào sử dụng**: Người dùng muốn thiết lập hoặc quản lý thông báo webhook cho các sự kiện đặt chỗ

**Trình tự công cụ**:

1. `CAL_RETRIEVE_WEBHOOKS_LIST` - Liệt kê các webhook hiện có [Bắt buộc]
2. `CAL_GET_WEBHOOK_BY_ID` - Lấy chi tiết webhook cụ thể [Tùy chọn]
3. `CAL_UPDATE_WEBHOOK_BY_ID` - Cập nhật cấu hình webhook [Tùy chọn]
4. `CAL_DELETE_WEBHOOK_BY_ID` - Xóa một webhook [Tùy chọn]

**Các tham số chính**:

- `id`: ID Webhook cho các hoạt động GET/UPDATE/DELETE
- `subscriberUrl`: URL endpoint của Webhook
- `eventTriggers`: Mảng các loại sự kiện để kích hoạt
- `active`: Liệu webhook có đang hoạt động hay không
- `secret`: Bí mật ký webhook

**Cạm bẫy**:

- URL Webhook phải là các endpoint HTTPS có thể truy cập công khai
- Các trình kích hoạt sự kiện bao gồm: 'BOOKING_CREATED', 'BOOKING_RESCHEDULED', 'BOOKING_CANCELLED', v.v.
- Các webhook không hoạt động sẽ không kích hoạt; chuyển đổi `active` để bật/tắt
- Bí mật webhook được sử dụng để xác minh chữ ký payload

### 4. Quản lý Nhóm (Manage Teams)

**Khi nào sử dụng**: Người dùng muốn tạo, xem, hoặc quản lý các nhóm và loại sự kiện nhóm

**Trình tự công cụ**:

1. `CAL_GET_TEAMS_LIST` - Liệt kê tất cả các nhóm [Bắt buộc]
2. `CAL_GET_TEAM_INFORMATION_BY_TEAM_ID` - Lấy chi tiết nhóm cụ thể [Tùy chọn]
3. `CAL_CREATE_TEAM_IN_ORGANIZATION` - Tạo một nhóm mới [Tùy chọn]
4. `CAL_RETRIEVE_TEAM_EVENT_TYPES` - Liệt kê các loại sự kiện cho một nhóm [Tùy chọn]

**Các tham số chính**:

- `teamId`: Định danh nhóm
- `name`: Tên nhóm (để tạo)
- `slug`: Định danh nhóm thân thiện với URL

**Cạm bẫy**:

- Việc tạo nhóm có thể yêu cầu quyền cấp tổ chức
- Các loại sự kiện nhóm tách biệt với các loại sự kiện cá nhân
- Slug nhóm phải an toàn cho URL và duy nhất trong tổ chức

### 5. Quản lý Tổ chức (Organization Management)

**Khi nào sử dụng**: Người dùng muốn xem chi tiết tổ chức

**Trình tự công cụ**:

1. `CAL_GET_ORGANIZATION_ID` - Lấy ID tổ chức [Bắt buộc]

**Các tham số chính**: (không có bắt buộc)

**Cạm bẫy**:

- ID tổ chức là cần thiết cho việc tạo nhóm và các hoạt động cấp tổ chức
- Không phải tất cả tài khoản Cal.com đều có tổ chức; các gói cá nhân có thể trả về lỗi

## Các Mẫu Phổ biến

### Quy trình Tạo Đặt chỗ

```
1. Gọi CAL_GET_AVAILABLE_SLOTS_INFO để tìm các khe mở
2. Trình bày thời gian có sẵn cho người dùng
3. Gọi CAL_POST_NEW_BOOKING_REQUEST với khe đã chọn
4. Xác nhận phản hồi tạo đặt chỗ
```

### Giải quyết ID

**Tên nhóm -> ID nhóm**:

```
1. Gọi CAL_GET_TEAMS_LIST
2. Tìm nhóm theo tên trong phản hồi
3. Trích xuất trường id
```

### Thiết lập Webhook

```
1. Gọi CAL_RETRIEVE_WEBHOOKS_LIST để kiểm tra các hook hiện có
2. Tạo hoặc cập nhật webhook với các trình kích hoạt mong muốn
3. Xác minh webhook kích hoạt trên đặt chỗ thử nghiệm
```

## Các Cạm bẫy Đã biết

**Định dạng Ngày/Giờ**:

- Thời gian đặt chỗ: ISO 8601 với múi giờ (ví dụ: '2024-01-15T09:00:00Z')
- Ngày khả dụng: định dạng YYYY-MM-DD
- Luôn chỉ định múi giờ rõ ràng để tránh nhầm lẫn

**Loại Sự kiện**:

- ID loại sự kiện là số nguyên
- Loại sự kiện xác định thời lượng, địa điểm, và quy tắc đặt chỗ
- Các loại sự kiện bị vô hiệu hóa không thể chấp nhận đặt chỗ mới

**Quyền hạn**:

- Các hoạt động nhóm yêu cầu tư cách thành viên nhóm hoặc quyền admin
- Các hoạt động tổ chức yêu cầu quyền cấp tổ chức
- Quản lý webhook yêu cầu mức truy cập phù hợp

**Giới hạn Tốc độ**:

- API Cal.com có giới hạn tốc độ cho mỗi khóa API
- Thực hiện backoff khi gặp phản hồi 429

## Tham khảo Nhanh

| Tác vụ            | Slug Công cụ                        | Tham số Chính                        |
| ----------------- | ----------------------------------- | ------------------------------------ |
| Liệt kê đặt chỗ   | CAL_FETCH_ALL_BOOKINGS              | status, afterStart, beforeEnd        |
| Tạo đặt chỗ       | CAL_POST_NEW_BOOKING_REQUEST        | eventTypeId, start, end, name, email |
| Lấy thời gian bận | CAL_RETRIEVE_CALENDAR_BUSY_TIMES    | dateFrom, dateTo                     |
| Lấy khe có sẵn    | CAL_GET_AVAILABLE_SLOTS_INFO        | eventTypeId, dateFrom, dateTo        |
| Liệt kê webhooks  | CAL_RETRIEVE_WEBHOOKS_LIST          | (không có)                           |
| Lấy webhook       | CAL_GET_WEBHOOK_BY_ID               | id                                   |
| Cập nhật webhook  | CAL_UPDATE_WEBHOOK_BY_ID            | id, subscriberUrl, eventTriggers     |
| Xóa webhook       | CAL_DELETE_WEBHOOK_BY_ID            | id                                   |
| Liệt kê nhóm      | CAL_GET_TEAMS_LIST                  | (không có)                           |
| Lấy nhóm          | CAL_GET_TEAM_INFORMATION_BY_TEAM_ID | teamId                               |
| Tạo nhóm          | CAL_CREATE_TEAM_IN_ORGANIZATION     | name, slug                           |
| Loại sự kiện nhóm | CAL_RETRIEVE_TEAM_EVENT_TYPES       | teamId                               |
| Lấy ID tổ chức    | CAL_GET_ORGANIZATION_ID             | (không có)                           |
