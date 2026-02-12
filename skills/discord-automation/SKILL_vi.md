---
name: discord-automation
description: "Tự động hóa các tác vụ Discord qua Rube MCP (Composio): tin nhắn, kênh, vai trò, webhooks, phản ứng. Luôn luôn tìm kiếm công cụ trước để có lược đồ hiện tại."
requires:
  mcp: [rube]
---

# Tự Động Hóa Discord qua Rube MCP

Tự động hóa các hoạt động Discord thông qua bộ công cụ Discord/Discordbot của Composio qua Rube MCP.

## Điều kiện Tiên quyết

- Rube MCP phải được kết nối (RUBE_SEARCH_TOOLS khả dụng)
- Kết nối Discord hoạt động qua `RUBE_MANAGE_CONNECTIONS` với bộ công cụ `discord` và `discordbot`
- Luôn gọi `RUBE_SEARCH_TOOLS` trước để lấy lược đồ công cụ hiện tại

## Thiết lập

**Lấy Rube MCP**: Thêm `https://rube.app/mcp` làm máy chủ MCP trong cấu hình khách của bạn. Không cần khóa API — chỉ cần thêm điểm cuối và nó sẽ hoạt động.

1. Xác minh Rube MCP khả dụng bằng cách xác nhận `RUBE_SEARCH_TOOLS` phản hồi
2. Gọi `RUBE_MANAGE_CONNECTIONS` với bộ công cụ `discordbot` (hoạt động bot) hoặc `discord` (hoạt động người dùng)
3. Nếu kết nối không ACTIVE, hãy làm theo liên kết xác thực được trả về để hoàn tất xác thực Discord
4. Xác nhận trạng thái kết nối hiển thị ACTIVE trước khi chạy bất kỳ quy trình làm việc nào

## Các Quy trình Làm việc Cốt lõi

### 1. Gửi Tin nhắn

**Khi nào sử dụng**: Người dùng muốn gửi tin nhắn đến kênh hoặc DM

**Trình tự công cụ**:

1. `DISCORD_LIST_MY_GUILDS` - Liệt kê các guild mà bot tham gia [Tiên quyết]
2. `DISCORDBOT_LIST_GUILD_CHANNELS` - Liệt kê các kênh trong một guild [Tiên quyết]
3. `DISCORDBOT_CREATE_MESSAGE` - Gửi tin nhắn [Bắt buộc]
4. `DISCORDBOT_UPDATE_MESSAGE` - Chỉnh sửa tin nhắn đã gửi [Tùy chọn]

**Tham số chính**:

- `channel_id`: ID snowflake của kênh
- `content`: Văn bản tin nhắn (tối đa 2000 ký tự)
- `embeds`: Mảng các đối tượng embed cho nội dung phong phú
- `guild_id`: ID Guild để liệt kê kênh

**Cạm bẫy**:

- Bot phải có quyền SEND_MESSAGES trong kênh
- Việc gửi tần suất cao có thể chạm giới hạn tốc độ mỗi tuyến; tôn trọng tiêu đề Retry-After
- Chỉ tin nhắn được gửi bởi cùng một bot mới có thể được chỉnh sửa

### 2. Gửi Tin nhắn Trực tiếp (DM)

**Khi nào sử dụng**: Người dùng muốn DM cho một người dùng Discord

**Trình tự công cụ**:

1. `DISCORDBOT_CREATE_DM` - Tạo hoặc lấy kênh DM [Bắt buộc]
2. `DISCORDBOT_CREATE_MESSAGE` - Gửi tin nhắn đến kênh DM [Bắt buộc]

**Tham số chính**:

- `recipient_id`: ID snowflake của người dùng để DM
- `channel_id`: ID kênh DM từ CREATE_DM

**Cạm bẫy**:

- Không thể DM cho người dùng đã tắt DM hoặc đã chặn bot
- CREATE_DM trả về kênh hiện có nếu đã tồn tại

### 3. Quản lý Vai trò

**Khi nào sử dụng**: Người dùng muốn tạo, gán, hoặc xóa vai trò

**Trình tự công cụ**:

1. `DISCORDBOT_CREATE_GUILD_ROLE` - Tạo vai trò mới [Tùy chọn]
2. `DISCORDBOT_ADD_GUILD_MEMBER_ROLE` - Gán vai trò cho thành viên [Tùy chọn]
3. `DISCORDBOT_DELETE_GUILD_ROLE` - Xóa vai trò [Tùy chọn]
4. `DISCORDBOT_GET_GUILD_MEMBER` - Lấy chi tiết thành viên [Tùy chọn]
5. `DISCORDBOT_UPDATE_GUILD_MEMBER` - Cập nhật thành viên (vai trò, nick, v.v.) [Tùy chọn]

**Tham số chính**:

- `guild_id`: ID snowflake của Guild
- `user_id`: ID snowflake của người dùng
- `role_id`: ID snowflake của vai trò
- `name`: Tên vai trò
- `permissions`: Giá trị quyền bitwise
- `color`: Số nguyên màu RGB

**Cạm bẫy**:

- Gán vai trò yêu cầu quyền MANAGE_ROLES
- Vai trò đích phải thấp hơn trong phân cấp so với vai trò cao nhất của bot
- DELETE xóa vĩnh viễn vai trò khỏi tất cả thành viên

### 4. Quản lý Webhooks

**Khi nào sử dụng**: Người dùng muốn tạo hoặc sử dụng webhooks cho tích hợp bên ngoài

**Trình tự công cụ**:

1. `DISCORDBOT_GET_GUILD_WEBHOOKS` / `DISCORDBOT_LIST_CHANNEL_WEBHOOKS` - Liệt kê webhooks [Tùy chọn]
2. `DISCORDBOT_CREATE_WEBHOOK` - Tạo webhook mới [Tùy chọn]
3. `DISCORDBOT_EXECUTE_WEBHOOK` - Gửi tin nhắn qua webhook [Tùy chọn]
4. `DISCORDBOT_UPDATE_WEBHOOK` - Cập nhật cài đặt webhook [Tùy chọn]

**Tham số chính**:

- `webhook_id`: ID Webhook
- `webhook_token`: Mã thông báo bí mật Webhook
- `channel_id`: Kênh để tạo webhook
- `name`: Tên Webhook
- `content`/`embeds`: Nội dung tin nhắn để thực thi

**Cạm bẫy**:

- Mã thông báo Webhook là bí mật; xử lý an toàn
- Webhooks có thể đăng bài với tên người dùng và ảnh đại diện tùy chỉnh cho mỗi tin nhắn
- Cần quyền MANAGE_WEBHOOKS để tạo

### 5. Quản lý Phản ứng

**Khi nào sử dụng**: Người dùng muốn xem hoặc quản lý phản ứng tin nhắn

**Trình tự công cụ**:

1. `DISCORDBOT_LIST_MESSAGE_REACTIONS_BY_EMOJI` - Liệt kê người dùng đã phản ứng [Tùy chọn]
2. `DISCORDBOT_DELETE_ALL_MESSAGE_REACTIONS` - Xóa tất cả phản ứng [Tùy chọn]
3. `DISCORDBOT_DELETE_ALL_MESSAGE_REACTIONS_BY_EMOJI` - Xóa phản ứng emoji cụ thể [Tùy chọn]
4. `DISCORDBOT_DELETE_USER_MESSAGE_REACTION` - Xóa phản ứng của người dùng cụ thể [Tùy chọn]

**Tham số chính**:

- `channel_id`: ID Kênh
- `message_id`: ID snowflake của tin nhắn
- `emoji_name`: Emoji được mã hóa URL hoặc `name:id` cho emoji tùy chỉnh
- `user_id`: ID người dùng để xóa phản ứng cụ thể

**Cạm bẫy**:

- Emoji Unicode phải được mã hóa URL (ví dụ: '%F0%9F%91%8D' cho ngón tay cái lên)
- Emoji tùy chỉnh sử dụng định dạng `name:id`
- DELETE_ALL yêu cầu quyền MANAGE_MESSAGES

## Các Mẫu Phổ biến

### Snowflake IDs

Discord sử dụng snowflake IDs (số nguyên 64-bit dưới dạng chuỗi) cho tất cả các thực thể:

- Guilds, kênh, người dùng, vai trò, tin nhắn, webhooks

### Bitfields Quyền

Quyền được kết hợp bằng cách sử dụng bitwise OR:

- SEND_MESSAGES = 0x800
- MANAGE_ROLES = 0x10000000
- MANAGE_MESSAGES = 0x2000
- ADMINISTRATOR = 0x8

### Phân trang

- Hầu hết các điểm cuối danh sách hỗ trợ tham số `limit`, `before`, `after`
- Tin nhắn: tối đa 100 mỗi yêu cầu
- Phản ứng: tối đa 100 mỗi yêu cầu, sử dụng `after` để phân trang

## Các Cạm bẫy Đã biết

**Mã thông báo Bot vs Người dùng**:

- Bộ công cụ `discordbot` sử dụng mã thông báo bot; `discord` sử dụng OAuth người dùng
- Các hoạt động bot được ưu tiên cho tự động hóa

**Giới hạn Tốc độ**:

- Discord thực thi giới hạn tốc độ mỗi tuyến
- Tôn trọng tiêu đề `Retry-After` trên các phản hồi 429

## Tham khảo Nhanh

| Tác vụ              | Slug Công cụ                               | Tham số Chính                      |
| ------------------- | ------------------------------------------ | ---------------------------------- |
| Liệt kê guilds      | DISCORD_LIST_MY_GUILDS                     | (không có)                         |
| Liệt kê kênh        | DISCORDBOT_LIST_GUILD_CHANNELS             | guild_id                           |
| Gửi tin nhắn        | DISCORDBOT_CREATE_MESSAGE                  | channel_id, content                |
| Chỉnh sửa tin nhắn  | DISCORDBOT_UPDATE_MESSAGE                  | channel_id, message_id             |
| Lấy tin nhắn        | DISCORDBOT_LIST_MESSAGES                   | channel_id, limit                  |
| Tạo DM              | DISCORDBOT_CREATE_DM                       | recipient_id                       |
| Tạo vai trò         | DISCORDBOT_CREATE_GUILD_ROLE               | guild_id, name                     |
| Gán vai trò         | DISCORDBOT_ADD_GUILD_MEMBER_ROLE           | guild_id, user_id, role_id         |
| Xóa vai trò         | DISCORDBOT_DELETE_GUILD_ROLE               | guild_id, role_id                  |
| Lấy thành viên      | DISCORDBOT_GET_GUILD_MEMBER                | guild_id, user_id                  |
| Cập nhật thành viên | DISCORDBOT_UPDATE_GUILD_MEMBER             | guild_id, user_id                  |
| Lấy guild           | DISCORDBOT_GET_GUILD                       | guild_id                           |
| Tạo webhook         | DISCORDBOT_CREATE_WEBHOOK                  | channel_id, name                   |
| Thực thi webhook    | DISCORDBOT_EXECUTE_WEBHOOK                 | webhook_id, webhook_token          |
| Liệt kê webhooks    | DISCORDBOT_GET_GUILD_WEBHOOKS              | guild_id                           |
| Lấy phản ứng        | DISCORDBOT_LIST_MESSAGE_REACTIONS_BY_EMOJI | channel_id, message_id, emoji_name |
| Xóa phản ứng        | DISCORDBOT_DELETE_ALL_MESSAGE_REACTIONS    | channel_id, message_id             |
| Kiểm tra auth       | DISCORDBOT_TEST_AUTH                       | (không có)                         |
| Lấy kênh            | DISCORDBOT_GET_CHANNEL                     | channel_id                         |
