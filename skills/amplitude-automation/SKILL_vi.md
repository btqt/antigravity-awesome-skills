---
name: amplitude-automation
description: "Tự động hóa các nhiệm vụ Amplitude thông qua Rube MCP (Composio): sự kiện, hoạt động người dùng, các nhóm (cohorts), định danh người dùng. Luôn tìm kiếm các công cụ trước để có các lược đồ (schemas) hiện tại."
requires:
  mcp: [rube]
---

# Tự động hóa Amplitude qua Rube MCP

Tự động hóa phân tích sản phẩm Amplitude thông qua bộ công cụ Amplitude của Composio thông qua Rube MCP.

## Điều kiện tiên quyết

- Rube MCP phải được kết nối (có sẵn `RUBE_SEARCH_TOOLS`)
- Kết nối Amplitude đang hoạt động thông qua `RUBE_MANAGE_CONNECTIONS` với bộ công cụ `amplitude`
- Luôn gọi `RUBE_SEARCH_TOOLS` trước để lấy lược đồ công cụ hiện tại

## Thiết lập

**Cài đặt Rube MCP**: Thêm `https://rube.app/mcp` làm máy chủ MCP trong cấu hình ứng dụng khách của bạn. Không cần mã khóa API — chỉ cần thêm điểm cuối (endpoint) và nó sẽ hoạt động.

1. Kiểm tra xem Rube MCP có sẵn sàng không bằng cách xác nhận `RUBE_SEARCH_TOOLS` có phản hồi.
2. Gọi `RUBE_MANAGE_CONNECTIONS` với bộ công cụ `amplitude`.
3. Nếu kết nối không ở trạng thái ACTIVE (đang hoạt động), hãy làm theo liên kết xác thực được trả về để hoàn tất xác thực Amplitude.
4. Xác nhận trạng thái kết nối hiển thị ACTIVE trước khi chạy bất kỳ quy trình công việc nào.

## Quy trình công việc cốt lõi

### 1. Gửi các Sự kiện

**Khi nào cần sử dụng**: Người dùng muốn theo dõi các sự kiện hoặc gửi dữ liệu sự kiện đến Amplitude.

**Trình tự công cụ**:

1. `AMPLITUDE_SEND_EVENTS` - Gửi một hoặc nhiều sự kiện đến Amplitude [Bắt buộc]

**Các tham số chính**:

- `events`: Mảng các đối tượng sự kiện, mỗi đối tượng bao gồm:
  - `event_type`: Tên sự kiện (ví dụ: 'page_view', 'purchase')
  - `user_id`: Định danh người dùng duy nhất (bắt buộc nếu không có `device_id`)
  - `device_id`: Định danh thiết bị (bắt buộc nếu không có `user_id`)
  - `event_properties`: Đối tượng chứa các thuộc tính sự kiện tùy chỉnh
  - `user_properties`: Đối tượng chứa các thuộc tính người dùng cần thiết lập
  - `time`: Dấu thời gian sự kiện (timestamp) tính bằng mili giây kể từ thời điểm epoch

**Các lỗi thường gặp**:

- Mỗi sự kiện yêu cầu ít nhất một trong hai thông tin `user_id` hoặc `device_id`.
- `event_type` là bắt buộc cho mọi sự kiện; không được để trống.
- `time` phải tính bằng mili giây (số epoch có 13 chữ số), không phải giây.
- Có giới hạn theo lô; hãy kiểm tra lược đồ để biết số lượng sự kiện tối đa mỗi yêu cầu.
- Các sự kiện được xử lý bất đồng bộ; phản hồi API thành công không có nghĩa là dữ liệu có thể truy vấn được ngay lập tức.

### 2. Lấy hoạt động của người dùng

**Khi nào cần sử dụng**: Người dùng muốn xem lịch sử sự kiện của một người dùng cụ thể.

**Trình tự công cụ**:

1. `AMPLITUDE_FIND_USER` - Tìm người dùng bằng ID hoặc thuộc tính [Điều kiện tiên quyết]
2. `AMPLITUDE_GET_USER_ACTIVITY` - Lấy dòng sự kiện của người dùng [Bắt buộc]

**Các tham số chính**:

- `user`: ID người dùng nội bộ của Amplitude (lấy từ `FIND_USER`)
- `offset`: Độ lệch phân trang cho danh sách sự kiện
- `limit`: Số lượng sự kiện tối đa cần trả về

**Các lỗi thường gặp**:

- Tham số `user` yêu cầu ID người dùng nội bộ của Amplitude, KHÔNG PHẢI `user_id` trong ứng dụng của bạn.
- Phải gọi `FIND_USER` trước để ánh xạ `user_id` của bạn sang ID nội bộ của Amplitude.
- Hoạt động được trả về theo trình tự thời gian ngược lại theo mặc định.
- Lịch sử hoạt động lớn yêu cầu phân trang thông qua `offset`.

### 3. Tìm kiếm và Định danh người dùng

**Khi nào cần sử dụng**: Người dùng muốn tra cứu người dùng hoặc thiết lập các thuộc tính người dùng.

**Trình tự công cụ**:

1. `AMPLITUDE_FIND_USER` - Tìm kiếm người dùng bằng các định danh khác nhau [Bắt buộc]
2. `AMPLITUDE_IDENTIFY` - Thiết lập hoặc cập nhật các thuộc tính người dùng [Tùy chọn]

**Các tham số chính**:

- Đối với `FIND_USER`:
  - `user`: Từ khóa tìm kiếm (user_id, email, hoặc ID Amplitude)
- Đối với `IDENTIFY`:
  - `user_id`: Định danh người dùng trong ứng dụng của bạn
  - `device_id`: Định danh thiết bị (thay thế cho `user_id`)
  - `user_properties`: Đối tượng chứa các thao tác `$set`, `$unset`, `$add`, `$append`

**Các lỗi thường gặp**:

- `FIND_USER` tìm kiếm trên cả user_id, device_id và ID Amplitude.
- `IDENTIFY` sử dụng các thao tác thuộc tính đặc biệt (`$set`, `$unset`, `$add`, `$append`).
- `$set` ghi đè các giá trị hiện có; `$setOnce` chỉ thiết lập nếu chưa có giá trị.
- Cần ít nhất `user_id` hoặc `device_id` cho `IDENTIFY`.
- Các thay đổi thuộc tính người dùng có tính nhất quán cuối cùng (eventually consistent); không có tác dụng ngay lập tức.

### 4. Quản lý các nhóm (Cohorts)

**Khi nào cần sử dụng**: Người dùng muốn liệt kê các nhóm, xem chi tiết nhóm hoặc cập nhật thành viên trong nhóm.

**Trình tự công cụ**:

1. `AMPLITUDE_LIST_COHORTS` - Liệt kê tất cả các nhóm đã lưu [Bắt buộc]
2. `AMPLITUDE_GET_COHORT` - Lấy thông tin nhóm chi tiết [Tùy chọn]
3. `AMPLITUDE_UPDATE_COHORT_MEMBERSHIP` - Thêm/xóa người dùng khỏi một nhóm [Tùy chọn]
4. `AMPLITUDE_CHECK_COHORT_STATUS` - Kiểm tra trạng thái thao tác nhóm bất đồng bộ [Tùy chọn]

**Các tham số chính**:

- Đối với `LIST_COHORTS`: Không yêu cầu tham số
- Đối với `GET_COHORT`: `cohort_id` (lấy từ kết quả danh sách)
- Đối với `UPDATE_COHORT_MEMBERSHIP`:
  - `cohort_id`: ID của nhóm mục tiêu
  - `memberships`: Đối tượng chứa các mảng `add` và/hoặc `remove` các ID người dùng
- Đối với `CHECK_COHORT_STATUS`: `request_id` từ phản hồi cập nhật

**Các lỗi thường gặp**:

- Yêu cầu `cohort_id` cho tất cả các hoạt động cụ thể đến nhóm.
- `UPDATE_COHORT_MEMBERSHIP` là bất đồng bộ; sử dụng `CHECK_COHORT_STATUS` để xác minh.
- Cần `request_id` từ phản hồi cập nhật để kiểm tra trạng thái.
- Số lượng thay đổi thành viên tối đa mỗi yêu cầu có thể bị giới hạn; hãy chia nhỏ các bản cập nhật lớn.
- Chỉ các nhóm hành vi (behavioral cohorts) mới hỗ trợ cập nhật thành viên qua API.

### 5. Duyệt các Danh mục sự kiện

**Khi nào cần sử dụng**: Người dùng muốn khám phá các loại sự kiện và danh mục có sẵn trong Amplitude.

**Trình tự công cụ**:

1. `AMPLITUDE_GET_EVENT_CATEGORIES` - Liệt kê tất cả các danh mục sự kiện [Bắt buộc]

**Các tham số chính**:

- Không yêu cầu tham số; trả về tất cả các danh mục sự kiện đã được cấu hình.

**Các lỗi thường gặp**:

- Các danh mục được cấu hình trong giao diện người dùng của Amplitude; API chỉ cung cấp quyền đọc.
- Tên sự kiện trong các danh mục có phân biệt hoa thường.
- Sử dụng các danh mục này để xác thực các giá trị `event_type` trước khi gửi sự kiện.

## Các mẫu phổ biến

### Ánh xạ ID

**user_id của ứng dụng -> ID nội bộ của Amplitude**:

```
1. Gọi AMPLITUDE_FIND_USER với user = your_user_id
2. Trích xuất ID người dùng nội bộ của Amplitude từ phản hồi
3. Sử dụng ID nội bộ này cho AMPLITUDE_GET_USER_ACTIVITY
```

**Tên nhóm -> ID nhóm**:

```
1. Gọi AMPLITUDE_LIST_COHORTS
2. Tìm nhóm theo tên trong kết quả trả về
3. Trích xuất ID để thực hiện các thao tác trên nhóm
```

### Các thao tác Thuộc tính người dùng

`AMPLITUDE_IDENTIFY` hỗ trợ các thao tác thuộc tính sau:

- `$set`: Thiết lập giá trị thuộc tính (ghi đè nếu đã tồn tại)
- `$setOnce`: Chỉ thiết lập nếu thuộc tính chưa có giá trị
- `$add`: Tăng giá trị thuộc tính dạng số
- `$append`: Thêm vào thuộc tính dạng danh sách
- `$unset`: Xóa hoàn toàn thuộc tính

Ví dụ cấu trúc:

```json
{
  "user_properties": {
    "$set": { "plan": "premium", "company": "Acme" },
    "$add": { "login_count": 1 }
  }
}
```

### Mẫu thao tác bất đồng bộ (Async)

Đối với cập nhật thành viên nhóm:

```
1. Gọi AMPLITUDE_UPDATE_COHORT_MEMBERSHIP -> lấy request_id
2. Gọi AMPLITUDE_CHECK_COHORT_STATUS với request_id
3. Lặp lại bước 2 cho đến khi trạng thái là 'complete' (hoàn thành) hoặc 'error' (lỗi)
```

## Các lỗi thường gặp đã biết

**ID người dùng**:

- Amplitude có ID người dùng nội bộ riêng, tách biệt với ID trong ứng dụng của bạn.
- `FIND_USER` giúp ánh xạ ID của bạn sang ID nội bộ của Amplitude.
- `GET_USER_ACTIVITY` yêu cầu ID nội bộ của Amplitude, không phải `user_id` của bạn.

**Dấu thời gian sự kiện (Event Timestamps)**:

- Phải tính bằng mili giây kể từ epoch (13 chữ số).
- Nếu dùng giây (10 chữ số), chúng sẽ được hiểu là các ngày rất cũ trong quá khứ.
- Bỏ qua dấu thời gian sẽ sử dụng thời gian máy chủ nhận được sự kiện.

**Giới hạn tốc độ (Rate Limits)**:

- Việc nạp sự kiện có giới hạn băng thông trên mỗi dự án.
- Hãy gom nhóm các sự kiện khi có thể để giảm số lượng cuộc gọi API.
- Cập nhật thành viên nhóm có giới hạn xử lý bất đồng bộ.

**Phân tích phản hồi**:

- Dữ liệu phản hồi có thể nằm trong khóa `data`.
- Hoạt động người dùng trả về các sự kiện theo trình tự thời gian ngược lại.
- Danh sách nhóm có thể bao gồm cả các nhóm đã được lưu trữ (archived); hãy kiểm tra trường trạng thái (status).
- Phân tích dữ liệu một cách cẩn thận với các phương án dự phòng cho các trường tùy chọn.

## Tham khảo nhanh

| Nhiệm vụ                 | Tên công cụ (Slug)                 | Các tham số chính        |
| ------------------------ | ---------------------------------- | ------------------------ |
| Gửi sự kiện              | AMPLITUDE_SEND_EVENTS              | events (mảng)            |
| Tìm người dùng           | AMPLITUDE_FIND_USER                | user                     |
| Lấy hoạt động người dùng | AMPLITUDE_GET_USER_ACTIVITY        | user, offset, limit      |
| Định danh người dùng     | AMPLITUDE_IDENTIFY                 | user_id, user_properties |
| Liệt kê các nhóm         | AMPLITUDE_LIST_COHORTS             | (không có)               |
| Lấy thông tin nhóm       | AMPLITUDE_GET_COHORT               | cohort_id                |
| Cập nhật thành viên nhóm | AMPLITUDE_UPDATE_COHORT_MEMBERSHIP | cohort_id, memberships   |
| Kiểm tra trạng thái nhóm | AMPLITUDE_CHECK_COHORT_STATUS      | request_id               |
| Liệt kê danh mục sự kiện | AMPLITUDE_GET_EVENT_CATEGORIES     | (không có)               |
