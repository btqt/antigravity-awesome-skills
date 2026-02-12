---
name: activecampaign-automation
description: "Tự động hóa các tác vụ ActiveCampaign qua Rube MCP (Composio): quản lý danh bạ, thẻ, đăng ký danh sách, ghi danh tự động và tác vụ. Luôn tìm kiếm các công cụ trước để xem lược đồ hiện tại."
requires:
  mcp: [rube]
---

# Tự động hóa ActiveCampaign qua Rube MCP

Tự động hóa CRM ActiveCampaign và các hoạt động tiếp thị tự động thông qua bộ công cụ ActiveCampaign của Composio qua Rube MCP.

## Điều kiện tiên quyết

- Rube MCP phải được kết nối (RUBE_SEARCH_TOOLS có sẵn)
- Kết nối ActiveCampaign đang hoạt động qua `RUBE_MANAGE_CONNECTIONS` với bộ công cụ `active_campaign`
- Luôn gọi `RUBE_SEARCH_TOOLS` trước để lấy lược đồ công cụ hiện tại

## Thiết lập

**Tải Rube MCP**: Thêm `https://rube.app/mcp` làm máy chủ MCP trong cấu hình ứng dụng khách của bạn. Không cần khóa API — chỉ cần thêm điểm cuối và nó sẽ hoạt động.

1. Xác minh Rube MCP có sẵn bằng cách xác nhận `RUBE_SEARCH_TOOLS` phản hồi
2. Gọi `RUBE_MANAGE_CONNECTIONS` với bộ công cụ `active_campaign`
3. Nếu trạng thái kết nối không phải là ACTIVE, hãy nhấp vào liên kết xác thực được trả về để hoàn tất xác thực ActiveCampaign
4. Xác nhận trạng thái kết nối hiển thị ACTIVE trước khi chạy bất kỳ quy trình công việc nào

## Quy trình công việc cốt lõi

### 1. Tạo và Tìm kiếm Danh bạ

**Khi nào nên sử dụng**: Người dùng muốn tạo danh bạ mới hoặc tra cứu danh bạ hiện có

**Trình tự công cụ**:

1. `ACTIVE_CAMPAIGN_FIND_CONTACT` - Tìm kiếm một liên hệ hiện có [Tùy chọn]
2. `ACTIVE_CAMPAIGN_CREATE_CONTACT` - Tạo một liên hệ mới [Bắt buộc]

**Các tham số chính để tìm kiếm**:

- `email`: Tìm kiếm theo địa chỉ email
- `id`: Tìm kiếm theo ID liên hệ ActiveCampaign
- `phone`: Tìm kiếm theo số điện thoại

**Các tham số chính để tạo**:

- `email`: Địa chỉ email liên hệ (bắt buộc)
- `first_name`: Tên liên hệ
- `last_name`: Họ liên hệ
- `phone`: Số điện thoại liên hệ
- `organization_name`: Tổ chức của liên hệ
- `job_title`: Chức danh công việc của liên hệ
- `tags`: Danh sách các thẻ cách nhau bằng dấu phẩy để áp dụng

**Các bẫy phổ biến**:

- `email` là trường bắt buộc duy nhất để tạo liên hệ
- Tìm kiếm điện thoại sử dụng tham số tìm kiếm chung nội bộ; nó có thể trả về các kết quả khớp một phần
- Khi kết hợp `email` và `phone` trong FIND_CONTACT, các kết quả sẽ được lọc ở phía ứng dụng khách
- Các thẻ được cung cấp trong khi tạo sẽ được áp dụng ngay lập tức
- Tạo một liên hệ với một email hiện có có thể cập nhật liên hệ hiện có đó

### 2. Quản lý Thẻ Liên hệ

**Khi nào nên sử dụng**: Người dùng muốn thêm hoặc xóa thẻ khỏi danh bạ

**Trình tự công cụ**:

1. `ACTIVE_CAMPAIGN_FIND_CONTACT` - Tìm liên hệ theo email hoặc ID [Điều kiện tiên quyết]
2. `ACTIVE_CAMPAIGN_MANAGE_CONTACT_TAG` - Thêm hoặc xóa thẻ [Bắt buộc]

**Các tham số chính**:

- `action`: 'Add' hoặc 'Remove' (bắt buộc)
- `tags`: Tên thẻ dưới dạng chuỗi cách nhau bằng dấu phẩy hoặc mảng chuỗi (bắt buộc)
- `contact_id`: ID liên hệ (cung cấp tham số này hoặc contact_email)
- `contact_email`: Địa chỉ email liên hệ (thay thế cho contact_id)

**Các bẫy phổ biến**:

- Các giá trị `action` được viết hoa chữ cái đầu: 'Add' hoặc 'Remove' (không phải chữ thường)
- Các thẻ có thể là một chuỗi cách nhau bằng dấu phẩy ('tag1, tag2') hoặc một mảng (['tag1', 'tag2'])
- Phải cung cấp `contact_id` hoặc `contact_email`; `contact_id` được ưu tiên hơn
- Thêm một thẻ chưa tồn tại sẽ tự động tạo ra nó
- Xóa một thẻ không tồn tại sẽ không thực hiện gì (không báo lỗi)

### 3. Quản lý Đăng ký Danh sách

**Khi nào nên sử dụng**: Người dùng muốn cho liên hệ đăng ký hoặc hủy đăng ký khỏi các danh sách

**Trình tự công cụ**:

1. `ACTIVE_CAMPAIGN_FIND_CONTACT` - Tìm liên hệ [Điều kiện tiên quyết]
2. `ACTIVE_CAMPAIGN_MANAGE_LIST_SUBSCRIPTION` - Đăng ký hoặc hủy đăng ký [Bắt buộc]

**Các tham số chính**:

- `action`: 'subscribe' hoặc 'unsubscribe' (bắt buộc)
- `list_id`: Chuỗi ID danh sách bằng số (bắt buộc)
- `email`: Địa chỉ email liên hệ (cung cấp tham số này hoặc contact_id)
- `contact_id`: Chuỗi ID liên hệ bằng số (thay thế cho email)

**Các bẫy phổ biến**:

- Các giá trị `action` được viết thường: 'subscribe' hoặc 'unsubscribe'
- `list_id` là một chuỗi số (ví dụ: '2'), không phải là tên danh sách
- ID danh sách có thể được lấy qua điểm cuối GET /api/3/lists (không khả dụng dưới dạng công cụ Composio; hãy sử dụng giao diện người dùng ActiveCampaign)
- Nếu cả `email` và `contact_id` đều được cung cấp, `contact_id` sẽ được ưu tiên hơn
- Hủy đăng ký sẽ thay đổi trạng thái thành '2' (unsubscribed) nhưng bản ghi mối quan hệ vẫn tồn tại

### 4. Thêm Liên hệ vào Quy trình Tự động (Automations)

**Khi nào nên sử dụng**: Người dùng muốn ghi danh một liên hệ vào một quy trình tự động hóa

**Trình tự công cụ**:

1. `ACTIVE_CAMPAIGN_FIND_CONTACT` - Xác minh liên hệ tồn tại [Điều kiện tiên quyết]
2. `ACTIVE_CAMPAIGN_ADD_CONTACT_TO_AUTOMATION` - Ghi danh liên hệ vào quy trình tự động [Bắt buộc]

**Các tham số chính**:

- `contact_email`: Email của liên hệ để ghi danh (bắt buộc)
- `automation_id`: ID của quy trình tự động mục tiêu (bắt buộc)

**Các bẫy phổ biến**:

- Liên hệ phải đã tồn tại trong ActiveCampaign
- Các quy trình tự động chỉ có thể được tạo thông qua giao diện người dùng ActiveCampaign, không phải qua API
- `automation_id` phải tham chiếu đến một quy trình tự động đang hoạt động và hiện có
- Công cụ thực hiện quy trình hai bước: tra cứu liên hệ theo email, sau đó ghi danh
- ID tự động hóa có thể được tìm thấy trong giao diện người dùng ActiveCampaign hoặc qua GET /api/3/automations

### 5. Tạo Tác vụ Liên hệ

**Khi nào nên sử dụng**: Người dùng muốn tạo các tác vụ theo dõi liên quan đến danh bạ

**Trình tự công cụ**:

1. `ACTIVE_CAMPAIGN_FIND_CONTACT` - Tìm liên hệ để liên kết tác vụ [Điều kiện tiên quyết]
2. `ACTIVE_CAMPAIGN_CREATE_CONTACT_TASK` - Tạo tác vụ [Bắt buộc]

**Các tham số chính**:

- `relid`: ID liên hệ để liên kết tác vụ (bắt buộc)
- `duedate`: Ngày đến hạn ở định dạng ISO 8601 kèm theo múi giờ (bắt buộc, ví dụ: '2025-01-15T14:30:00-05:00')
- `dealTasktype`: ID loại tác vụ dựa trên các loại có sẵn (bắt buộc)
- `title`: Tiêu đề tác vụ
- `note`: Mô tả/nội dung tác vụ
- `assignee`: ID người dùng để giao tác vụ
- `edate`: Ngày kết thúc ở định dạng ISO 8601 (phải muộn hơn duedate)
- `status`: 0 cho chưa hoàn thành, 1 cho đã hoàn thành

**Các bẫy phổ biến**:

- `duedate` phải là một chuỗi ngày giờ ISO 8601 hợp lệ với độ lệch múi giờ; KHÔNG sử dụng các giá trị giữ chỗ
- `edate` phải muộn hơn `duedate`
- `dealTasktype` là một ID chuỗi tham chiếu đến các loại tác vụ được cấu hình trong ActiveCampaign
- `relid` là ID liên hệ bằng số, không phải địa chỉ email
- `assignee` là ID người dùng; hãy giải quyết tên người dùng thành ID qua giao diện người dùng ActiveCampaign

## Các mẫu phổ biến

### Quy trình Tra cứu Liên hệ

```
1. Gọi ACTIVE_CAMPAIGN_FIND_CONTACT với email
2. Nếu tìm thấy, trích xuất ID liên hệ cho các hoạt động tiếp theo
3. Nếu không tìm thấy, tạo liên hệ với ACTIVE_CAMPAIGN_CREATE_CONTACT
4. Sử dụng ID liên hệ cho các thẻ, đăng ký hoặc tự động hóa
```

### Gắn thẻ Liên hệ hàng loạt

```
1. Đối với mỗi liên hệ, hãy gọi ACTIVE_CAMPAIGN_MANAGE_CONTACT_TAG
2. Sử dụng contact_email để tránh các cuộc gọi tra cứu riêng biệt
3. Thực hiện theo đợt với độ trễ hợp lý để tôn trọng các giới hạn tốc độ (rate limits)
```

### Giải quyết ID

**Email liên hệ -> ID liên hệ**:

```
1. Gọi ACTIVE_CAMPAIGN_FIND_CONTACT với email
2. Trích xuất id từ phản hồi
```

## Các bẫy đã biết

**Viết hoa Hành động**:

- Hành động thẻ: 'Add', 'Remove' (viết hoa chữ cái đầu)
- Hành động đăng ký: 'subscribe', 'unsubscribe' (viết thường)
- Ghi lộn cách viết hoa sẽ gây ra lỗi

**Các loại ID**:

- ID liên hệ: chuỗi số (ví dụ: '123')
- ID danh sách: chuỗi số
- ID tự động hóa: chuỗi số
- Tất cả các ID nên được truyền dưới dạng chuỗi, không phải số nguyên

**Tự động hóa (Automations)**:

- Không thể tạo tự động hóa qua API; chỉ có thể ghi danh
- Tự động hóa phải ở trạng thái hoạt động để chấp nhận các liên hệ mới
- Việc ghi danh một liên hệ đã có trong quy trình tự động có thể không có tác dụng

**Giới hạn Tốc độ (Rate Limits)**:

- API ActiveCampaign có giới hạn tốc độ cho mỗi tài khoản
- Triển khai chiến lược chờ (backoff) khi gặp phản hồi 429
- Các hoạt động hàng loạt nên được giãn cách một cách thích hợp

**Phân tích phản hồi**:

- Dữ liệu phản hồi có thể được lồng dưới `data` hoặc `data.data`
- Phân tích một cách cẩn thận với các mẫu dự phòng
- Tìm kiếm liên hệ có thể trả về nhiều kết quả; hãy so khớp theo email để có độ chính xác

## Tham khảo nhanh

| Tác vụ               | Công cụ                                   | Các tham số chính                   |
| -------------------- | ----------------------------------------- | ----------------------------------- |
| Tìm liên hệ          | ACTIVE_CAMPAIGN_FIND_CONTACT              | email, id, phone                    |
| Tạo liên hệ          | ACTIVE_CAMPAIGN_CREATE_CONTACT            | email, first_name, last_name, tags  |
| Thêm/xóa thẻ         | ACTIVE_CAMPAIGN_MANAGE_CONTACT_TAG        | action, tags, contact_email         |
| Đăng ký/Hủy đăng ký  | ACTIVE_CAMPAIGN_MANAGE_LIST_SUBSCRIPTION  | action, list_id, email              |
| Thêm vào tự động hóa | ACTIVE_CAMPAIGN_ADD_CONTACT_TO_AUTOMATION | contact_email, automation_id        |
| Tạo tác vụ           | ACTIVE_CAMPAIGN_CREATE_CONTACT_TASK       | relid, duedate, dealTasktype, title |
