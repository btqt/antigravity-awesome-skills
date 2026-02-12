---
name: datadog-automation
description: "Tự động hóa các tác vụ Datadog qua Rube MCP (Composio): truy vấn chỉ số, tìm kiếm nhật ký, quản lý monitor/bảng điều khiển, tạo sự kiện và thời gian chết. Luôn tìm kiếm công cụ trước để có lược đồ hiện tại."
requires:
  mcp: [rube]
---

# Tự Động Hóa Datadog qua Rube MCP (Datadog Automation via Rube MCP)

Tự động hóa các hoạt động giám sát và khả năng quan sát Datadog thông qua bộ công cụ Datadog của Composio qua Rube MCP.

## Điều kiện tiên quyết

- Rube MCP phải được kết nối (RUBE_SEARCH_TOOLS có sẵn)
- Kết nối Datadog đang hoạt động qua `RUBE_MANAGE_CONNECTIONS` với bộ công cụ `datadog`
- Luôn gọi `RUBE_SEARCH_TOOLS` trước để lấy lược đồ công cụ hiện tại

## Thiết lập

**Lấy Rube MCP**: Thêm `https://rube.app/mcp` làm máy chủ MCP trong cấu hình máy khách của bạn. Không cần khóa API — chỉ cần thêm điểm cuối và nó hoạt động.

1. Xác minh Rube MCP có sẵn bằng cách xác nhận `RUBE_SEARCH_TOOLS` phản hồi
2. Gọi `RUBE_MANAGE_CONNECTIONS` với bộ công cụ `datadog`
3. Nếu kết nối không TRẠNG THÁI HOẠT ĐỘNG, hãy làm theo liên kết xác thực được trả về để hoàn tất xác thực Datadog
4. Xác nhận trạng thái kết nối hiển thị HOẠT ĐỘNG trước khi chạy bất kỳ quy trình làm việc nào

## Các Quy trình làm việc Cốt lõi

### 1. Truy vấn và Khám phá Chỉ số

**Khi nào sử dụng**: Người dùng muốn truy vấn dữ liệu chỉ số hoặc liệt kê các chỉ số có sẵn

**Trình tự công cụ**:

1. `DATADOG_LIST_METRICS` - Liệt kê tên các chỉ số có sẵn [Tùy chọn]
2. `DATADOG_QUERY_METRICS` - Truy vấn dữ liệu chuỗi thời gian chỉ số [Bắt buộc]

**Các tham số chính**:

- `query`: Chuỗi truy vấn chỉ số Datadog (ví dụ: `avg:system.cpu.user{host:web01}`)
- `from`: Dấu thời gian bắt đầu (giây Unix epoch)
- `to`: Dấu thời gian kết thúc (giây Unix epoch)
- `q`: Chuỗi tìm kiếm để liệt kê các chỉ số

**Các cạm bẫy**:

- Cú pháp truy vấn tuân theo định dạng truy vấn chỉ số của Datadog: `aggregation:metric_name{tag_filters}`
- `from` và `to` là dấu thời gian Unix epoch tính bằng giây, không phải mili giây
- Các tổng hợp hợp lệ: `avg`, `sum`, `min`, `max`, `count`
- Bộ lọc thẻ sử dụng dấu ngoặc nhọn: `{host:web01,env:prod}`
- Khoảng thời gian không được vượt quá giới hạn lưu giữ của Datadog cho loại chỉ số đó

### 2. Tìm kiếm và Phân tích Nhật ký

**Khi nào sử dụng**: Người dùng muốn tìm kiếm các mục nhật ký hoặc liệt kê các chỉ mục nhật ký

**Trình tự công cụ**:

1. `DATADOG_LIST_LOG_INDEXES` - Liệt kê các chỉ mục nhật ký có sẵn [Tùy chọn]
2. `DATADOG_SEARCH_LOGS` - Tìm kiếm nhật ký với truy vấn và bộ lọc [Bắt buộc]

**Các tham số chính**:

- `query`: Truy vấn tìm kiếm nhật ký sử dụng cú pháp truy vấn nhật ký Datadog
- `from`: Thời gian bắt đầu (ISO 8601 hoặc dấu thời gian Unix)
- `to`: Thời gian kết thúc (ISO 8601 hoặc dấu thời gian Unix)
- `sort`: Thứ tự sắp xếp ('asc' hoặc 'desc')
- `limit`: Số lượng mục nhật ký trả về

**Các cạm bẫy**:

- Truy vấn nhật ký sử dụng cú pháp tìm kiếm nhật ký của Datadog: `service:web status:error`
- Tìm kiếm bị giới hạn ở các nhật ký được lưu giữ trong thời gian lưu giữ đã cấu hình
- Các tập kết quả lớn yêu cầu phân trang; kiểm tra mã thông báo con trỏ/trang
- Các chỉ mục nhật ký kiểm soát định tuyến và lưu giữ; lọc theo chỉ mục nếu biết

### 3. Quản lý Monitor

**Khi nào sử dụng**: Người dùng muốn tạo, cập nhật, tắt tiếng, hoặc kiểm tra monitor

**Trình tự công cụ**:

1. `DATADOG_LIST_MONITORS` - Liệt kê tất cả monitor với bộ lọc [Bắt buộc]
2. `DATADOG_GET_MONITOR` - Lấy chi tiết monitor cụ thể [Tùy chọn]
3. `DATADOG_CREATE_MONITOR` - Tạo một monitor mới [Tùy chọn]
4. `DATADOG_UPDATE_MONITOR` - Cập nhật cấu hình monitor [Tùy chọn]
5. `DATADOG_MUTE_MONITOR` - Tắt tiếng một monitor tạm thời [Tùy chọn]
6. `DATADOG_UNMUTE_MONITOR` - Bật lại tiếng một monitor đã tắt tiếng [Tùy chọn]

**Các tham số chính**:

- `monitor_id`: ID monitor dạng số
- `name`: Tên hiển thị monitor
- `type`: Loại monitor ('metric alert', 'service check', 'log alert', 'query alert', v.v.)
- `query`: Truy vấn monitor xác định điều kiện cảnh báo
- `message`: Tin nhắn thông báo với @mentions
- `tags`: Mảng các chuỗi thẻ
- `thresholds`: Các giá trị ngưỡng cảnh báo (`critical`, `warning`, `ok`)

**Các cạm bẫy**:

- `type` của monitor phải khớp với loại truy vấn; không khớp sẽ gây ra lỗi tạo
- `message` hỗ trợ @mentions cho thông báo (ví dụ: `@slack-channel`, `@pagerduty`)
- Ngưỡng thay đổi theo loại monitor; monitor chỉ số cần `critical` tối thiểu
- Tắt tiếng một monitor sẽ ngăn chặn thông báo nhưng monitor vẫn đánh giá
- ID Monitor là số nguyên

### 4. Quản lý Bảng điều khiển (Dashboards)

**Khi nào sử dụng**: Người dùng muốn liệt kê, xem, cập nhật, hoặc xóa bảng điều khiển

**Trình tự công cụ**:

1. `DATADOG_LIST_DASHBOARDS` - Liệt kê tất cả bảng điều khiển [Bắt buộc]
2. `DATADOG_GET_DASHBOARD` - Lấy định nghĩa bảng điều khiển đầy đủ [Tùy chọn]
3. `DATADOG_UPDATE_DASHBOARD` - Cập nhật bố cục hoặc tiện ích bảng điều khiển [Tùy chọn]
4. `DATADOG_DELETE_DASHBOARD` - Xóa một bảng điều khiển (không thể đảo ngược) [Tùy chọn]

**Các tham số chính**:

- `dashboard_id`: Chuỗi định danh bảng điều khiển
- `title`: Tiêu đề bảng điều khiển
- `layout_type`: 'ordered' (lưới) hoặc 'free' (định vị tự do)
- `widgets`: Mảng các đối tượng định nghĩa tiện ích
- `description`: Mô tả bảng điều khiển

**Các cạm bẫy**:

- ID Bảng điều khiển là chuỗi chữ và số (ví dụ: 'abc-def-ghi'), không phải số
- `layout_type` không thể thay đổi sau khi tạo; phải tạo lại bảng điều khiển
- Định nghĩa tiện ích là các đối tượng lồng nhau phức tạp; lấy bảng điều khiển hiện có trước để hiểu cấu trúc
- DELETE là vĩnh viễn; không có hoàn tác

### 5. Tạo Sự kiện và Quản lý Thời gian chết

**Khi nào sử dụng**: Người dùng muốn đăng sự kiện hoặc lên lịch thời gian chết bảo trì

**Trình tự công cụ**:

1. `DATADOG_LIST_EVENTS` - Liệt kê các sự kiện hiện có [Tùy chọn]
2. `DATADOG_CREATE_EVENT` - Đăng một sự kiện mới [Bắt buộc]
3. `DATADOG_CREATE_DOWNTIME` - Lên lịch một thời gian chết bảo trì [Tùy chọn]

**Các tham số chính cho sự kiện**:

- `title`: Tiêu đề sự kiện
- `text`: Văn bản nội dung sự kiện (hỗ trợ markdown)
- `alert_type`: Mức độ nghiêm trọng của sự kiện ('error', 'warning', 'info', 'success')
- `tags`: Mảng các chuỗi thẻ

**Các tham số chính cho thời gian chết**:

- `scope`: Phạm vi thẻ cho thời gian chết (ví dụ: `host:web01`)
- `start`: Thời gian bắt đầu (Unix epoch)
- `end`: Thời gian kết thúc (Unix epoch; bỏ qua nếu vô thời hạn)
- `message`: Mô tả thời gian chết
- `monitor_id`: Monitor cụ thể cho thời gian chết (tùy chọn, bỏ qua nếu dựa trên phạm vi)

**Các cạm bẫy**:

- `text` sự kiện hỗ trợ định dạng markdown của Datadog bao gồm @mentions
- Phạm vi thời gian chết sử dụng cú pháp thẻ: `host:web01`, `env:staging`
- Bỏ qua `end` tạo ra thời gian chết vô thời hạn; luôn đặt thời gian kết thúc cho bảo trì
- `monitor_id` thời gian chết thu hẹp vào một monitor đơn lẻ; phạm vi áp dụng cho tất cả monitor phù hợp

### 6. Quản lý Máy chủ (Hosts) và Dấu vết (Traces)

**Khi nào sử dụng**: Người dùng muốn liệt kê các máy chủ cơ sở hạ tầng hoặc kiểm tra các dấu vết phân tán

**Trình tự công cụ**:

1. `DATADOG_LIST_HOSTS` - Liệt kê tất cả máy chủ đang báo cáo [Bắt buộc]
2. `DATADOG_GET_TRACE_BY_ID` - Lấy một dấu vết phân tán cụ thể [Tùy chọn]

**Các tham số chính**:

- `filter`: Chuỗi bộ lọc tìm kiếm máy chủ
- `sort_field`: Sắp xếp máy chủ theo trường (ví dụ: 'name', 'apps', 'cpu')
- `sort_dir`: Hướng sắp xếp ('asc' hoặc 'desc')
- `trace_id`: ID dấu vết phân tán để tra cứu dấu vết

**Các cạm bẫy**:

- Danh sách máy chủ bao gồm tất cả các máy chủ báo cáo cho Datadog trong cửa sổ lưu giữ
- ID Dấu vết là các chuỗi số dài; đảm bảo khớp chính xác
- Các máy chủ ngừng báo cáo được giữ lại trong một khoảng thời gian được cấu hình trước khi xóa

## Các Mẫu Phổ biến

### Cú pháp Truy vấn Monitor

**Cảnh báo chỉ số**:

```
avg(last_5m):avg:system.cpu.user{env:prod} > 90
```

**Cảnh báo nhật ký**:

```
logs("service:web status:error").index("main").rollup("count").last("5m") > 10
```

### Lọc Thẻ

- Thẻ sử dụng định dạng `key:value`: `host:web01`, `env:prod`, `service:api`
- Nhiều thẻ: `{host:web01,env:prod}` (logic AND)
- Ký tự đại diện: `host:web*`

### Phân trang

- Sử dụng `page` và `page_size` hoặc phân trang dựa trên offset tùy thuộc vào điểm cuối
- Kiểm tra phản hồi để biết tổng số lượng nhằm xác định xem còn trang nào không
- Tiếp tục cho đến khi tất cả kết quả được truy xuất

## Các Cạm bẫy Đã biết

**Dấu thời gian**:

- Hầu hết các điểm cuối sử dụng giây Unix epoch (không phải mili giây)
- Một số điểm cuối chấp nhận ISO 8601; kiểm tra lược đồ công cụ
- Khoảng thời gian nên hợp lý (không phải dữ liệu của nhiều năm)

**Cú pháp Truy vấn**:

- Truy vấn chỉ số: `aggregation:metric{tags}`
- Truy vấn nhật ký: các cặp `field:value`
- Truy vấn monitor thay đổi theo loại; kiểm tra tài liệu Datadog

**Giới hạn Tốc độ**:

- Datadog API có giới hạn tốc độ trên mỗi điểm cuối
- Triển khai backoff trên các phản hồi 429
- Xử lý hàng loạt khi có thể

## Tham khảo Nhanh

| Tác vụ                   | Slug Công cụ             | Tham số Chính                |
| ------------------------ | ------------------------ | ---------------------------- |
| Truy vấn chỉ số          | DATADOG_QUERY_METRICS    | query, from, to              |
| Liệt kê chỉ số           | DATADOG_LIST_METRICS     | q                            |
| Tìm kiếm nhật ký         | DATADOG_SEARCH_LOGS      | query, from, to, limit       |
| Liệt kê chỉ mục nhật ký  | DATADOG_LIST_LOG_INDEXES | (không có)                   |
| Liệt kê monitor          | DATADOG_LIST_MONITORS    | tags                         |
| Lấy monitor              | DATADOG_GET_MONITOR      | monitor_id                   |
| Tạo monitor              | DATADOG_CREATE_MONITOR   | name, type, query, message   |
| Cập nhật monitor         | DATADOG_UPDATE_MONITOR   | monitor_id                   |
| Tắt tiếng monitor        | DATADOG_MUTE_MONITOR     | monitor_id                   |
| Bật tiếng monitor        | DATADOG_UNMUTE_MONITOR   | monitor_id                   |
| Liệt kê bảng điều khiển  | DATADOG_LIST_DASHBOARDS  | (không có)                   |
| Lấy bảng điều khiển      | DATADOG_GET_DASHBOARD    | dashboard_id                 |
| Cập nhật bảng điều khiển | DATADOG_UPDATE_DASHBOARD | dashboard_id, title, widgets |
| Xóa bảng điều khiển      | DATADOG_DELETE_DASHBOARD | dashboard_id                 |
| Liệt kê sự kiện          | DATADOG_LIST_EVENTS      | start, end                   |
| Tạo sự kiện              | DATADOG_CREATE_EVENT     | title, text, alert_type      |
| Tạo thời gian chết       | DATADOG_CREATE_DOWNTIME  | scope, start, end            |
| Liệt kê máy chủ          | DATADOG_LIST_HOSTS       | filter, sort_field           |
| Lấy dấu vết              | DATADOG_GET_TRACE_BY_ID  | trace_id                     |
