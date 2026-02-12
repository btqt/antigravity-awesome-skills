---
name: calendly-automation
description: "Tự động hóa lập lịch Calendly, quản lý sự kiện, theo dõi người được mời, kiểm tra tính khả dụng, và quản trị tổ chức qua Rube MCP (Composio). Luôn tìm kiếm công cụ trước để có schemas hiện tại."
requires:
  mcp: [rube]
---

# Tự động hóa Calendly qua Rube MCP

Tự động hóa các hoạt động Calendly bao gồm liệt kê sự kiện, quản lý người được mời, tạo liên kết lập lịch, truy vấn tính khả dụng, và quản trị tổ chức thông qua bộ công cụ Calendly của Composio.

## Điều kiện tiên quyết

- Rube MCP phải được kết nối (RUBE_SEARCH_TOOLS có sẵn)
- Kết nối Calendly đang hoạt động qua `RUBE_MANAGE_CONNECTIONS` với toolkit `calendly`
- Luôn gọi `RUBE_SEARCH_TOOLS` trước để lấy schemas công cụ hiện tại
- Nhiều hoạt động yêu cầu URI Calendly của người dùng, lấy được qua `CALENDLY_GET_CURRENT_USER`

## Thiết lập

**Lấy Rube MCP**: Thêm `https://rube.app/mcp` làm máy chủ MCP trong cấu hình khách của bạn. Không cần khóa API — chỉ cần thêm endpoint và nó hoạt động.

1. Xác minh Rube MCP có sẵn bằng cách xác nhận `RUBE_SEARCH_TOOLS` phản hồi
2. Gọi `RUBE_MANAGE_CONNECTIONS` với toolkit `calendly`
3. Nếu kết nối không ACTIVE, làm theo liên kết xác thực được trả về để hoàn tất OAuth Calendly
4. Xác nhận trạng thái kết nối hiển thị ACTIVE trước khi chạy bất kỳ quy trình làm việc nào

## Quy trình làm việc Cốt lõi

### 1. Liệt kê và Xem Các sự kiện Đã lên lịch

**Khi nào sử dụng**: Người dùng muốn xem các sự kiện Calendly sắp tới, trong quá khứ, hoặc được lọc

**Trình tự công cụ**:

1. `CALENDLY_GET_CURRENT_USER` - Lấy URI người dùng đã xác thực và URI tổ chức [Điều kiện tiên quyết]
2. `CALENDLY_LIST_EVENTS` - Liệt kê các sự kiện theo phạm vi người dùng, tổ chức, hoặc nhóm [Bắt buộc]
3. `CALENDLY_GET_EVENT` - Lấy thông tin chi tiết cho một sự kiện cụ thể bằng UUID [Tùy chọn]

**Các tham số chính**:

- `user`: URI Calendly API đầy đủ (ví dụ: `https://api.calendly.com/users/{uuid}`) - KHÔNG PHẢI `"me"`
- `organization`: URI tổ chức đầy đủ cho các truy vấn phạm vi tổ chức
- `status`: `"active"` hoặc `"canceled"`
- `min_start_time` / `max_start_time`: Dấu thời gian UTC (ví dụ: `2024-01-01T00:00:00.000000Z`)
- `invitee_email`: Lọc sự kiện theo email người được mời (chỉ lọc, không phải phạm vi)
- `sort`: `"start_time:asc"` hoặc `"start_time:desc"`
- `count`: Kết quả mỗi trang (mặc định 20)
- `page_token`: Token phân trang từ phản hồi trước

**Cạm bẫy**:

- Chính xác MỘT trong các phạm vi `user`, `organization`, hoặc `group` phải được cung cấp - bỏ qua hoặc kết hợp các phạm vi sẽ thất bại
- Tham số `user` yêu cầu URI API đầy đủ, không phải `"me"` - sử dụng `CALENDLY_GET_CURRENT_USER` trước
- `invitee_email` là bộ lọc, không phải phạm vi; bạn vẫn cần một trong use/organization/group
- Phân trang sử dụng `count` + `page_token`; lặp lại cho đến khi `page_token` vắng mặt để có kết quả đầy đủ
- Quyền quản trị có thể cần thiết cho các truy vấn phạm vi tổ chức hoặc nhóm

### 2. Quản lý Người được mời Sự kiện

**Khi nào sử dụng**: Người dùng muốn xem ai đã đặt chỗ cho các sự kiện hoặc lấy chi tiết người được mời

**Trình tự công cụ**:

1. `CALENDLY_LIST_EVENTS` - Tìm (các) sự kiện mục tiêu [Điều kiện tiên quyết]
2. `CALENDLY_LIST_EVENT_INVITEES` - Liệt kê tất cả người được mời cho một sự kiện cụ thể [Bắt buộc]
3. `CALENDLY_GET_EVENT_INVITEE` - Lấy thông tin chi tiết cho một người được mời [Tùy chọn]

**Các tham số chính**:

- `uuid`: UUID Sự kiện (cho `LIST_EVENT_INVITEES`)
- `event_uuid` + `invitee_uuid`: Cả hai đều bắt buộc cho `GET_EVENT_INVITEE`
- `email`: Lọc người được mời theo địa chỉ email
- `status`: `"active"` hoặc `"canceled"`
- `sort`: `"created_at:asc"` hoặc `"created_at:desc"`
- `count`: Kết quả mỗi trang (mặc định 20)

**Cạm bẫy**:

- Tham số `uuid` cho `CALENDLY_LIST_EVENT_INVITEES` là UUID sự kiện, không phải UUID người được mời
- Phân trang bằng `page_token` cho đến khi vắng mặt để có danh sách người được mời đầy đủ
- Người được mời đã hủy bị loại trừ theo mặc định; sử dụng `status: "canceled"` để xem họ

### 3. Tạo Liên kết Lập lịch và Kiểm tra Tính khả dụng

**Khi nào sử dụng**: Người dùng muốn tạo liên kết đặt chỗ hoặc kiểm tra các khe thời gian có sẵn

**Trình tự công cụ**:

1. `CALENDLY_GET_CURRENT_USER` - Lấy URI người dùng [Điều kiện tiên quyết]
2. `CALENDLY_LIST_USER_S_EVENT_TYPES` - Liệt kê các loại sự kiện có sẵn [Bắt buộc]
3. `CALENDLY_LIST_EVENT_TYPE_AVAILABLE_TIMES` - Kiểm tra các khe có sẵn cho một loại sự kiện [Tùy chọn]
4. `CALENDLY_CREATE_SCHEDULING_LINK` - Tạo liên kết lập lịch dùng một lần [Bắt buộc]
5. `CALENDLY_LIST_USER_AVAILABILITY_SCHEDULES` - Xem lịch khả dụng của người dùng [Tùy chọn]

**Các tham số chính**:

- `owner`: URI loại sự kiện (ví dụ: `https://api.calendly.com/event_types/{uuid}`)
- `owner_type`: `"EventType"` (mặc định)
- `max_event_count`: Phải chính xác là `1` cho các liên kết dùng một lần
- `start_time` / `end_time`: Dấu thời gian UTC cho các truy vấn tính khả dụng (phạm vi tối đa 7 ngày)
- `active`: Boolean để lọc các loại sự kiện hoạt động/không hoạt động
- `user`: URI người dùng cho danh sách loại sự kiện

**Cạm bẫy**:

- `CALENDLY_CREATE_SCHEDULING_LINK` có thể trả về 403 nếu token thiếu quyền hoặc URI chủ sở hữu không hợp lệ
- `CALENDLY_LIST_EVENT_TYPE_AVAILABLE_TIMES` yêu cầu dấu thời gian UTC và phạm vi tối đa 7 ngày; chia nhỏ các tìm kiếm dài hơn
- Kết quả thời gian có sẵn KHÔNG được phân trang - tất cả kết quả được trả về trong một phản hồi
- URI loại sự kiện phải là URI API đầy đủ (ví dụ: `https://api.calendly.com/event_types/...`)

### 4. Hủy Sự kiện

**Khi nào sử dụng**: Người dùng muốn hủy một sự kiện Calendly đã lên lịch

**Trình tự công cụ**:

1. `CALENDLY_LIST_EVENTS` - Tìm sự kiện để hủy [Điều kiện tiên quyết]
2. `CALENDLY_GET_EVENT` - Xác nhận chi tiết sự kiện trước khi hủy [Điều kiện tiên quyết]
3. `CALENDLY_LIST_EVENT_INVITEES` - Kiểm tra ai sẽ bị ảnh hưởng [Tùy chọn]
4. `CALENDLY_CANCEL_EVENT` - Hủy sự kiện [Bắt buộc]

**Các tham số chính**:

- `uuid`: UUID sự kiện để hủy
- `reason`: Lý do hủy tùy chọn (có thể được bao gồm trong thông báo cho người được mời)

**Cạm bẫy**:

- Hủy bỏ là KHÔNG THỂ ĐẢO NGƯỢC - luôn xác nhận với người dùng trước khi gọi
- Việc hủy có thể kích hoạt thông báo cho người được mời
- Chỉ các sự kiện đang hoạt động mới có thể bị hủy; các sự kiện đã hủy trả về lỗi
- Lấy xác nhận rõ ràng của người dùng trước khi thực thi `CALENDLY_CANCEL_EVENT`

### 5. Quản lý Tổ chức và Lời mời

**Khi nào sử dụng**: Người dùng muốn mời thành viên, quản lý tổ chức, hoặc xử lý lời mời tổ chức

**Trình tự công cụ**:

1. `CALENDLY_GET_CURRENT_USER` - Lấy ngữ cảnh người dùng và tổ chức [Điều kiện tiên quyết]
2. `CALENDLY_GET_ORGANIZATION` - Lấy chi tiết tổ chức [Tùy chọn]
3. `CALENDLY_LIST_ORGANIZATION_INVITATIONS` - Kiểm tra các lời mời hiện có [Tùy chọn]
4. `CALENDLY_CREATE_ORGANIZATION_INVITATION` - Gửi lời mời tổ chức [Bắt buộc]
5. `CALENDLY_REVOKE_USER_S_ORGANIZATION_INVITATION` - Thu hồi lời mời đang chờ xử lý [Tùy chọn]
6. `CALENDLY_REMOVE_USER_FROM_ORGANIZATION` - Xóa thành viên [Tùy chọn]

**Các tham số chính**:

- `uuid`: UUID Tổ chức
- `email`: Địa chỉ email của người dùng cần mời
- `status`: Lọc lời mời theo `"pending"`, `"accepted"`, hoặc `"declined"`

**Cạm bẫy**:

- Chỉ chủ sở hữu/quản trị viên tổ chức mới có thể quản lý lời mời và xóa bỏ; người khác sẽ gặp lỗi ủy quyền
- Các lời mời đang hoạt động trùng lặp cho cùng một email bị từ chối - kiểm tra các lời mời hiện có trước
- Chủ sở hữu tổ chức không thể bị xóa qua `CALENDLY_REMOVE_USER_FROM_ORGANIZATION`
- Trạng thái lời mời bao gồm pending, accepted, declined, và revoked - xử lý từng cái một cách thích hợp

## Các Mẫu Phổ biến

### Giải quyết ID

Calendly sử dụng URI API đầy đủ làm định danh, không phải ID đơn giản:

- **URI người dùng hiện tại**: `CALENDLY_GET_CURRENT_USER` trả về `resource.uri` (ví dụ: `https://api.calendly.com/users/{uuid}`)
- **URI tổ chức**: Tìm thấy trong phản hồi người dùng hiện tại tại `resource.current_organization`
- **UUID sự kiện**: Trích xuất từ URI sự kiện hoặc danh sách phản hồi
- **URI loại sự kiện**: Từ phản hồi `CALENDLY_LIST_USER_S_EVENT_TYPES`

Quan trọng: Không bao giờ sử dụng `"me"` làm tham số người dùng trong các endpoint danh sách/bộ lọc. Luôn giải quyết thành URI đầy đủ trước.

### Phân trang

Hầu hết các endpoint danh sách của Calendly sử dụng phân trang dựa trên token:

- Đặt `count` cho kích thước trang (mặc định 20)
- Theo dõi `page_token` từ `pagination.next_page_token` cho đến khi vắng mặt
- Sắp xếp với định dạng `field:direction` (ví dụ: `start_time:asc`, `created_at:desc`)

### Xử lý Thời gian

- Tất cả các dấu thời gian phải ở định dạng UTC: `yyyy-MM-ddTHH:mm:ss.ffffffZ`
- Sử dụng `min_start_time` / `max_start_time` để lọc phạm vi ngày trên các sự kiện
- Các truy vấn thời gian có sẵn có phạm vi tối đa 7 ngày; chia nhỏ các tìm kiếm dài hơn thành nhiều cuộc gọi

## Các Cạm bẫy Đã biết

### Định dạng URI

- Tất cả các tham chiếu thực thể sử dụng URI API Calendly đầy đủ (ví dụ: `https://api.calendly.com/users/{uuid}`)
- Không bao giờ chuyển UUID trần trụi nơi URI được mong đợi, và không bao giờ chuyển `"me"` đến các endpoint danh sách
- Trích xuất UUID từ URI khi các công cụ mong đợi tham số UUID (ví dụ: `CALENDLY_GET_EVENT`)

### Yêu cầu Phạm vi

- `CALENDLY_LIST_EVENTS` yêu cầu chính xác một phạm vi (người dùng, tổ chức, hoặc nhóm) - không nhiều hơn, không ít hơn
- Các truy vấn phạm vi tổ chức/nhóm có thể yêu cầu đặc quyền quản trị
- Phạm vi token xác định các hoạt động nào khả dụng; lỗi 403 cho thấy không đủ quyền

### Mối quan hệ Dữ liệu

- Các sự kiện có người được mời (người tham dự đã đặt chỗ)
- Các loại sự kiện xác định các trang lập lịch (thời lượng, quy tắc khả dụng)
- Các tổ chức chứa người dùng và nhóm
- Các liên kết lập lịch được gắn với các loại sự kiện, không trực tiếp với các sự kiện

### Giới hạn Tốc độ

- API Calendly có giới hạn tốc độ; tránh các vòng lặp chặt chẽ trên các tập dữ liệu lớn
- Phân trang có trách nhiệm và thêm độ trễ cho các hoạt động hàng loạt

## Tham khảo Nhanh

| Tác vụ                   | Slug Công cụ                                     | Tham số Chính                              |
| ------------------------ | ------------------------------------------------ | ------------------------------------------ |
| Lấy người dùng hiện tại  | `CALENDLY_GET_CURRENT_USER`                      | (không có)                                 |
| Lấy người dùng theo UUID | `CALENDLY_GET_USER`                              | `uuid`                                     |
| Liệt kê sự kiện          | `CALENDLY_LIST_EVENTS`                           | `user`, `status`, `min_start_time`         |
| Lấy chi tiết sự kiện     | `CALENDLY_GET_EVENT`                             | `uuid`                                     |
| Hủy sự kiện              | `CALENDLY_CANCEL_EVENT`                          | `uuid`, `reason`                           |
| Liệt kê người được mời   | `CALENDLY_LIST_EVENT_INVITEES`                   | `uuid`, `status`, `email`                  |
| Lấy người được mời       | `CALENDLY_GET_EVENT_INVITEE`                     | `event_uuid`, `invitee_uuid`               |
| Liệt kê loại sự kiện     | `CALENDLY_LIST_USER_S_EVENT_TYPES`               | `user`, `active`                           |
| Lấy loại sự kiện         | `CALENDLY_GET_EVENT_TYPE`                        | `uuid`                                     |
| Kiểm tra tính khả dụng   | `CALENDLY_LIST_EVENT_TYPE_AVAILABLE_TIMES`       | URI loại sự kiện, `start_time`, `end_time` |
| Tạo liên kết lập lịch    | `CALENDLY_CREATE_SCHEDULING_LINK`                | `owner`, `max_event_count`                 |
| Liệt kê lịch khả dụng    | `CALENDLY_LIST_USER_AVAILABILITY_SCHEDULES`      | URI người dùng                             |
| Lấy tổ chức              | `CALENDLY_GET_ORGANIZATION`                      | `uuid`                                     |
| Mời vào tổ chức          | `CALENDLY_CREATE_ORGANIZATION_INVITATION`        | `uuid`, `email`                            |
| Liệt kê lời mời tổ chức  | `CALENDLY_LIST_ORGANIZATION_INVITATIONS`         | `uuid`, `status`                           |
| Thu hồi lời mời tổ chức  | `CALENDLY_REVOKE_USER_S_ORGANIZATION_INVITATION` | UUID tổ chức, UUID lời mời                 |
| Xóa khỏi tổ chức         | `CALENDLY_REMOVE_USER_FROM_ORGANIZATION`         | UUID thành viên                            |
