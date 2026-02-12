---
name: close-automation
description: "Tự động hóa các tác vụ Close CRM qua Rube MCP (Composio): tạo khách hàng tiềm năng, quản lý cuộc gọi/SMS, xử lý tác vụ và theo dõi ghi chú. Luôn tìm kiếm công cụ trước để biết lược đồ hiện tại."
requires:
  mcp: [rube]
---

# Tự động hóa Close CRM qua Rube MCP

Tự động hóa các hoạt động Close CRM thông qua bộ công cụ Close của Composio qua Rube MCP.

## Điều kiện tiên quyết

- Rube MCP phải được kết nối (RUBE_SEARCH_TOOLS có sẵn)
- Kết nối Close hoạt động qua `RUBE_MANAGE_CONNECTIONS` với bộ công cụ `close`
- Luôn gọi `RUBE_SEARCH_TOOLS` trước để lấy lược đồ công cụ hiện tại

## Thiết lập

**Lấy Rube MCP**: Thêm `https://rube.app/mcp` làm máy chủ MCP trong cấu hình máy khách của bạn. Không cần khóa API — chỉ cần thêm endpoint và nó sẽ hoạt động.

1. Xác minh Rube MCP có sẵn bằng cách xác nhận `RUBE_SEARCH_TOOLS` phản hồi
2. Gọi `RUBE_MANAGE_CONNECTIONS` với bộ công cụ `close`
3. Nếu kết nối không ACTIVE, hãy làm theo liên kết xác thực được trả về để hoàn tất xác thực API Close
4. Xác nhận trạng thái kết nối hiển thị ACTIVE trước khi chạy bất kỳ quy trình làm việc nào

## Các Quy trình làm việc Cốt lõi

### 1. Tạo và Quản lý Khách hàng tiềm năng (Leads)

**Khi nào sử dụng**: Người dùng muốn tạo khách hàng tiềm năng mới hoặc quản lý hồ sơ khách hàng tiềm năng hiện có

**Trình tự công cụ**:

1. `CLOSE_CREATE_LEAD` - Tạo một khách hàng tiềm năng mới trong Close [Bắt buộc]

**Các tham số chính**:

- `name`: Tên khách hàng tiềm năng/công ty
- `contacts`: Mảng các đối tượng liên hệ được liên kết với khách hàng tiềm năng
- `custom`: Giá trị trường tùy chỉnh dưới dạng cặp khóa-giá trị
- `status_id`: ID trạng thái khách hàng tiềm năng

**Cạm bẫy**:

- Khách hàng tiềm năng trong Close đại diện cho các công ty/tổ chức, không phải cá nhân
- Các liên hệ được lồng trong khách hàng tiềm năng; tạo khách hàng tiềm năng trước, sau đó các liên hệ được bao gồm
- Các khóa trường tùy chỉnh sử dụng ID trường tùy chỉnh (ví dụ: 'custom.cf_XXX'), không phải tên hiển thị
- Phát hiện khách hàng tiềm năng trùng lặp không tự động; kiểm tra trước khi tạo

### 2. Ghi lại Cuộc gọi

**Khi nào sử dụng**: Người dùng muốn ghi nhật ký hoạt động cuộc gọi điện thoại với một khách hàng tiềm năng

**Trình tự công cụ**:

1. `CLOSE_CREATE_CALL` - Ghi nhật ký hoạt động cuộc gọi [Bắt buộc]

**Các tham số chính**:

- `lead_id`: ID của khách hàng tiềm năng được liên kết
- `contact_id`: ID của liên hệ được gọi
- `direction`: 'outbound' (đi) hoặc 'inbound' (đến)
- `status`: Trạng thái cuộc gọi ('completed', 'no-answer', 'busy', v.v.)
- `duration`: Thời lượng cuộc gọi tính bằng giây
- `note`: Ghi chú cuộc gọi

**Cạm bẫy**:

- lead_id là bắt buộc; cuộc gọi phải được liên kết với một khách hàng tiềm năng
- Thời lượng tính bằng giây, không phải phút
- Hướng cuộc gọi ảnh hưởng đến báo cáo và phân tích
- contact_id là tùy chọn nhưng được khuyến nghị để theo dõi

### 3. Gửi tin nhắn SMS

**Khi nào sử dụng**: Người dùng muốn gửi hoặc ghi nhật ký tin nhắn SMS qua Close

**Trình tự công cụ**:

1. `CLOSE_CREATE_SMS` - Gửi hoặc ghi nhật ký tin nhắn SMS [Bắt buộc]

**Các tham số chính**:

- `lead_id`: ID của khách hàng tiềm năng được liên kết
- `contact_id`: ID của liên hệ
- `direction`: 'outbound' hoặc 'inbound'
- `text`: Nội dung tin nhắn SMS
- `status`: Trạng thái tin nhắn

**Cạm bẫy**:

- Chức năng SMS yêu cầu tích hợp điện thoại/SMS Close phải được cấu hình
- lead_id là bắt buộc cho tất cả các hoạt động SMS
- SMS đi có thể yêu cầu số gửi đã được xác minh
- Giới hạn độ dài tin nhắn có thể áp dụng tùy thuộc vào nhà mạng

### 4. Quản lý Tác vụ

**Khi nào sử dụng**: Người dùng muốn tạo hoặc quản lý các tác vụ theo dõi

**Trình tự công cụ**:

1. `CLOSE_CREATE_TASK` - Tạo một tác vụ mới [Bắt buộc]

**Các tham số chính**:

- `lead_id`: ID khách hàng tiềm năng được liên kết
- `text`: Mô tả tác vụ
- `date`: Ngày đến hạn cho tác vụ
- `assigned_to`: ID người dùng của người được giao
- `is_complete`: Liệu tác vụ đã hoàn thành chưa

**Cạm bẫy**:

- Các tác vụ được liên kết với khách hàng tiềm năng, không phải liên hệ
- Định dạng ngày nên tuân theo ISO 8601
- assigned_to yêu cầu ID người dùng Close, không phải email hoặc tên
- Các tác vụ không có ngày sẽ xuất hiện trong phần 'không có ngày đến hạn'

### 5. Quản lý Ghi chú

**Khi nào sử dụng**: Người dùng muốn thêm hoặc truy xuất ghi chú về khách hàng tiềm năng

**Trình tự công cụ**:

1. `CLOSE_GET_NOTE` - Truy xuất một ghi chú cụ thể [Bắt buộc]

**Các tham số chính**:

- `note_id`: ID của ghi chú cần truy xuất

**Cạm bẫy**:

- Ghi chú được liên kết với khách hàng tiềm năng
- ID ghi chú là bắt buộc để truy xuất; tìm kiếm khách hàng tiềm năng trước để tìm tham chiếu ghi chú
- Ghi chú hỗ trợ văn bản thuần túy và định dạng cơ bản

### 6. Xóa Hoạt động

**Khi nào sử dụng**: Người dùng muốn xóa bản ghi cuộc gọi hoặc các hoạt động khác

**Trình tự công cụ**:

1. `CLOSE_DELETE_CALL` - Xóa một hoạt động cuộc gọi [Bắt buộc]

**Các tham số chính**:

- `call_id`: ID của cuộc gọi cần xóa

**Cạm bẫy**:

- Việc xóa là vĩnh viễn và không thể hoàn tác
- Chỉ người tạo cuộc gọi hoặc quản trị viên mới có thể xóa cuộc gọi
- Xóa cuộc gọi sẽ xóa nó khỏi tất cả các báo cáo và dòng thời gian

## Các Mẫu Phổ biến

### Mối quan hệ Khách hàng tiềm năng và Liên hệ

```
Mô hình dữ liệu Close:
- Lead = Công ty/Tổ chức
  - Contact = Người (lồng trong Lead)
  - Activity = Cuộc gọi, SMS, Email, Ghi chú (được liên kết với Lead)
  - Task = Mục theo dõi (được liên kết với Lead)
  - Opportunity = Thỏa thuận (được liên kết với Lead)
```

### Giải quyết ID

**Lead ID**:

```
1. Tìm kiếm khách hàng tiềm năng bằng API tìm kiếm Close
2. Trích xuất lead_id từ kết quả (định dạng: 'lead_XXXXXXXXXXXXX')
3. Sử dụng lead_id trong tất cả các lệnh gọi tạo hoạt động
```

**Contact ID**:

```
1. Truy xuất chi tiết khách hàng tiềm năng để lấy các liên hệ lồng nhau
2. Trích xuất contact_id (định dạng: 'cont_XXXXXXXXXXXXX')
3. Sử dụng trong các hoạt động cuộc gọi/SMS để theo dõi chính xác
```

### Mẫu Ghi nhật ký Hoạt động

```
1. Xác định lead_id và tùy chọn contact_id
2. Tạo hoạt động (cuộc gọi, SMS, ghi chú) với lead_id
3. Bao gồm siêu dữ liệu liên quan (thời lượng, hướng, trạng thái)
4. Tạo tác vụ theo dõi nếu cần
```

## Các Cạm bẫy Đã biết

**Định dạng ID**:

- Lead IDs: 'lead_XXXXXXXXXXXXX'
- Contact IDs: 'cont_XXXXXXXXXXXXX'
- Activity IDs thay đổi theo loại: 'acti_XXXXXXXXXXXXX', 'call_XXXXXXXXXXXXX'
- Custom field IDs: 'custom.cf_XXXXXXXXXXXXX'
- Luôn sử dụng chuỗi ID đầy đủ

**Giới hạn Tốc độ**:

- API Close có giới hạn tốc độ dựa trên gói của bạn
- Thực hiện độ trễ giữa các hoạt động hàng loạt
- Theo dõi tiêu đề phản hồi cho trạng thái giới hạn tốc độ
- Phản hồi 429 yêu cầu chờ (backoff)

**Trường Tùy chỉnh**:

- Các trường tùy chỉnh được tham chiếu bởi ID API của chúng, không phải tên hiển thị
- Các trạng thái khách hàng tiềm năng khác nhau có thể có các trường tùy chỉnh bắt buộc khác nhau
- Các loại trường tùy chỉnh (văn bản, số, ngày, danh sách thả xuống) thực thi định dạng giá trị

**Toàn vẹn Dữ liệu**:

- Khách hàng tiềm năng là thực thể chính; liên hệ và hoạt động được liên kết với khách hàng tiềm năng
- Xóa một khách hàng tiềm năng có thể ảnh hưởng đến các liên hệ và hoạt động của nó
- Các hoạt động hàng loạt nên xác thực ID trước khi thực hiện

## Tham khảo Nhanh

| Nhiệm vụ                 | Slug Công cụ      | Các Tham số Chính                    |
| ------------------------ | ----------------- | ------------------------------------ |
| Tạo khách hàng tiềm năng | CLOSE_CREATE_LEAD | name, contacts, custom               |
| Ghi nhật ký cuộc gọi     | CLOSE_CREATE_CALL | lead_id, direction, status, duration |
| Gửi SMS                  | CLOSE_CREATE_SMS  | lead_id, text, direction             |
| Tạo tác vụ               | CLOSE_CREATE_TASK | lead_id, text, date, assigned_to     |
| Lấy ghi chú              | CLOSE_GET_NOTE    | note_id                              |
| Xóa cuộc gọi             | CLOSE_DELETE_CALL | call_id                              |
