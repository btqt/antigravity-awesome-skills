---
name: bamboohr-automation
description: "Tự động hóa các tác vụ BambooHR thông qua Rube MCP (Composio): nhân viên, nghỉ phép, phúc lợi, người phụ thuộc, cập nhật nhân viên. Luôn tìm kiếm công cụ trước để có schema hiện tại."
requires:
  mcp: [rube]
---

# Tự động hóa BambooHR thông qua Rube MCP

Tự động hóa các hoạt động nhân sự BambooHR thông qua bộ công cụ BambooHR của Composio qua Rube MCP.

## Điều kiện tiên quyết

- Rube MCP phải được kết nối (RUBE_SEARCH_TOOLS khả dụng)
- Kết nối BambooHR đang hoạt động qua `RUBE_MANAGE_CONNECTIONS` với toolkit `bamboohr`
- Luôn gọi `RUBE_SEARCH_TOOLS` trước tiên để lấy schema công cụ hiện tại

## Cài đặt

**Lấy Rube MCP**: Thêm `https://rube.app/mcp` làm máy chủ MCP trong cấu hình client của bạn. Không cần API key — chỉ cần thêm endpoint và nó sẽ hoạt động.

1. Xác minh Rube MCP khả dụng bằng cách xác nhận `RUBE_SEARCH_TOOLS` phản hồi
2. Gọi `RUBE_MANAGE_CONNECTIONS` với toolkit `bamboohr`
3. Nếu kết nối không ACTIVE, làm theo liên kết xác thực được trả về để hoàn tất xác thực BambooHR
4. Xác nhận trạng thái kết nối hiển thị ACTIVE trước khi chạy bất kỳ quy trình làm việc nào

## Các Quy trình làm việc Cốt lõi

### 1. Liệt kê và Tìm kiếm Nhân viên

**Khi nào sử dụng**: Người dùng muốn tìm nhân viên hoặc lấy toàn bộ danh bạ nhân viên

**Trình tự công cụ**:

1. `BAMBOOHR_GET_ALL_EMPLOYEES` - Lấy danh bạ nhân viên [Bắt buộc]
2. `BAMBOOHR_GET_EMPLOYEE` - Lấy thông tin chi tiết cho một nhân viên cụ thể [Tùy chọn]

**Tham số chính**:

- Cho GET_ALL_EMPLOYEES: Không có tham số bắt buộc; trả về danh bạ
- Cho GET_EMPLOYEE:
  - `id`: Employee ID (số)
  - `fields`: Danh sách các trường cần trả về, phân tách bằng dấu phẩy (ví dụ: 'firstName,lastName,department,jobTitle')

**Cạm bẫy**:

- Employee ID là số nguyên
- GET_ALL_EMPLOYEES trả về thông tin danh bạ cơ bản; sử dụng GET_EMPLOYEE để có đầy đủ chi tiết
- Tham số `fields` kiểm soát các trường được trả về; bỏ qua nó có thể trả về dữ liệu tối thiểu
- Các trường phổ biến: firstName, lastName, department, division, jobTitle, workEmail, status
- Nhân viên không hoạt động/đã nghỉ việc có thể được bao gồm; kiểm tra trường `status`

### 2. Theo dõi Thay đổi Nhân viên

**Khi nào sử dụng**: Người dùng muốn phát hiện các thay đổi dữ liệu nhân viên gần đây để đồng bộ hóa hoặc kiểm toán

**Trình tự công cụ**:

1. `BAMBOOHR_EMPLOYEE_GET_CHANGED` - Lấy danh sách nhân viên có thay đổi gần đây [Bắt buộc]

**Tham số chính**:

- `since`: Chuỗi ngày giờ ISO 8601 cho ngưỡng phát hiện thay đổi
- `type`: Loại thay đổi cần kiểm tra (ví dụ: 'inserted', 'updated', 'deleted')

**Cạm bẫy**:

- Tham số `since` là bắt buộc; sử dụng định dạng ISO 8601 (ví dụ: '2024-01-15T00:00:00Z')
- Trả về ID của nhân viên đã thay đổi, không phải toàn bộ dữ liệu nhân viên
- Phải gọi GET_EMPLOYEE riêng cho từng nhân viên đã thay đổi để lấy chi tiết
- Hữu ích cho các quy trình đồng bộ hóa gia tăng; cache timestamp lần đồng bộ cuối cùng

### 3. Quản lý Nghỉ phép (Time-Off)

**Khi nào sử dụng**: Người dùng muốn xem số dư nghỉ phép, yêu cầu nghỉ phép, hoặc quản lý các yêu cầu

**Trình tự công cụ**:

1. `BAMBOOHR_GET_META_TIME_OFF_TYPES` - Liệt kê các loại nghỉ phép có sẵn [Tiên quyết]
2. `BAMBOOHR_GET_TIME_OFF_BALANCES` - Kiểm tra số dư hiện tại [Tùy chọn]
3. `BAMBOOHR_GET_TIME_OFF_REQUESTS` - Liệt kê các yêu cầu hiện có [Tùy chọn]
4. `BAMBOOHR_CREATE_TIME_OFF_REQUEST` - Gửi yêu cầu mới [Tùy chọn]
5. `BAMBOOHR_UPDATE_TIME_OFF_REQUEST` - Sửa đổi hoặc phê duyệt/từ chối yêu cầu [Tùy chọn]

**Tham số chính**:

- Cho số dư: `employeeId`, time-off type ID
- Cho yêu cầu: `start`, `end` (phạm vi ngày), `employeeId`
- Cho tạo mới:
  - `employeeId`: Nhân viên cần yêu cầu
  - `timeOffTypeId`: Type ID từ GET_META_TIME_OFF_TYPES
  - `start`: Ngày bắt đầu (YYYY-MM-DD)
  - `end`: Ngày kết thúc (YYYY-MM-DD)
  - `amount`: Số ngày/giờ
  - `notes`: Ghi chú tùy chọn cho yêu cầu
- Cho cập nhật: `requestId`, `status` ('approved', 'denied', 'cancelled')

**Cạm bẫy**:

- Time-off type ID là số; giải quyết thông qua GET_META_TIME_OFF_TYPES trước
- Định dạng ngày là 'YYYY-MM-DD' cho ngày bắt đầu và kết thúc
- Số dư có thể tính bằng giờ hoặc ngày tùy thuộc vào cấu hình công ty
- Cập nhật trạng thái yêu cầu đòi hỏi quyền phù hợp (quản lý/admin)
- Tạo yêu cầu KHÔNG tự động phê duyệt nó; cần bước phê duyệt riêng

### 4. Cập nhật Thông tin Nhân viên

**Khi nào sử dụng**: Người dùng muốn sửa đổi dữ liệu hồ sơ nhân viên

**Trình tự công cụ**:

1. `BAMBOOHR_GET_EMPLOYEE` - Lấy dữ liệu nhân viên hiện tại [Tiên quyết]
2. `BAMBOOHR_UPDATE_EMPLOYEE` - Cập nhật các trường nhân viên [Bắt buộc]

**Tham số chính**:

- `id`: Employee ID (số, bắt buộc)
- Các cặp trường-giá trị cho các trường cần cập nhật (ví dụ: `department`, `jobTitle`, `workPhone`)

**Cạm bẫy**:

- Chỉ các trường có trong yêu cầu mới được cập nhật; các trường khác giữ nguyên
- Một số trường là chỉ đọc và không thể cập nhật qua API
- Tên trường phải khớp chính xác với tên trường mong đợi của BambooHR
- Các cập nhật được kiểm toán; thay đổi xuất hiện trong lịch sử thay đổi của nhân viên
- Xác minh giá trị hiện tại với GET_EMPLOYEE trước khi cập nhật để tránh ghi đè

### 5. Quản lý Người phụ thuộc và Phúc lợi

**Khi nào sử dụng**: Người dùng muốn xem người phụ thuộc của nhân viên hoặc chi tiết bảo hiểm phúc lợi

**Trình tự công cụ**:

1. `BAMBOOHR_DEPENDENTS_GET_ALL` - Liệt kê tất cả người phụ thuộc [Bắt buộc]
2. `BAMBOOHR_BENEFIT_GET_COVERAGES` - Lấy chi tiết bảo hiểm phúc lợi [Tùy chọn]

**Tham số chính**:

- Cho người phụ thuộc: Bộ lọc `employeeId` tùy chọn
- Cho phúc lợi: Phụ thuộc vào schema; kiểm tra RUBE_SEARCH_TOOLS cho các tham số hiện tại

**Cạm bẫy**:

- Dữ liệu người phụ thuộc bao gồm PII nhạy cảm; xử lý cẩn thận
- Bảo hiểm phúc lợi có thể bao gồm nhiều loại gói cho mỗi nhân viên
- Không phải tất cả các gói BambooHR đều bao gồm quản trị phúc lợi; kiểm tra tính năng tài khoản
- Quyền truy cập dữ liệu phụ thuộc vào quyền của API key

## Các Mẫu Phổ biến

### Giải quyết ID (ID Resolution)

**Tên nhân viên -> Employee ID**:

```
1. Gọi BAMBOOHR_GET_ALL_EMPLOYEES
2. Tìm nhân viên theo tên trong kết quả danh bạ
3. Trích xuất id (số) cho các hoạt động chi tiết
```

**Tên loại nghỉ phép -> Type ID**:

```
1. Gọi BAMBOOHR_GET_META_TIME_OFF_TYPES
2. Tìm loại theo tên (ví dụ: 'Vacation', 'Sick Leave')
3. Trích xuất id cho các yêu cầu nghỉ phép
```

### Mẫu Đồng bộ Gia tăng (Incremental Sync Pattern)

Để giữ cho các hệ thống bên ngoài đồng bộ với BambooHR:

```
1. Lưu last_sync_timestamp
2. Gọi BAMBOOHR_EMPLOYEE_GET_CHANGED với since=last_sync_timestamp
3. Với mỗi employee ID đã thay đổi, gọi BAMBOOHR_GET_EMPLOYEE
4. Xử lý cập nhật trong hệ thống bên ngoài
5. Cập nhật last_sync_timestamp
```

### Quy trình Nghỉ phép

```
1. GET_META_TIME_OFF_TYPES -> tìm type ID
2. GET_TIME_OFF_BALANCES -> xác minh số dư khả dụng
3. CREATE_TIME_OFF_REQUEST -> gửi yêu cầu
4. UPDATE_TIME_OFF_REQUEST -> phê duyệt/từ chối (hành động của quản lý)
```

## Các Cạm bẫy Đã biết

**Employee IDs**:

- Luôn là số nguyên
- Giải quyết tên thành ID qua GET_ALL_EMPLOYEES
- Nhân viên đã nghỉ việc vẫn giữ ID của họ

**Định dạng Ngày**:

- Ngày nghỉ phép: 'YYYY-MM-DD'
- Phát hiện thay đổi: ISO 8601 với múi giờ
- Định dạng không nhất quán giữa các endpoint; kiểm tra schema của từng endpoint

**Quyền hạn**:

- Quyền của API key xác định các trường và hoạt động có thể truy cập
- Một số hoạt động yêu cầu quyền truy cập cấp admin hoặc quản lý
- Phê duyệt nghỉ phép yêu cầu quyền vai trò phù hợp

**Dữ liệu Nhạy cảm**:

- Dữ liệu nhân viên bao gồm PII (tên, địa chỉ, SSN, v.v.)
- Xử lý tất cả các phản hồi với các biện pháp bảo mật thích hợp
- Dữ liệu người phụ thuộc đặc biệt nhạy cảm

**Giới hạn Tốc độ (Rate Limits)**:

- BambooHR API có giới hạn tốc độ cho mỗi API key
- Các hoạt động hàng loạt (bulk) nên được điều tiết
- GET_ALL_EMPLOYEES hiệu quả hơn các cuộc gọi GET_EMPLOYEE riêng lẻ

**Phân tích Phản hồi**:

- Dữ liệu phản hồi có thể được lồng dưới key `data`
- Các trường nhân viên thay đổi dựa trên tham số `fields`
- Các trường trống có thể bị bỏ qua hoặc trả về là null
- Phân tích phòng thủ với fallbacks

## Tham khảo Nhanh

| Tác vụ                     | Tool Slug                        | Tham số chính                         |
| -------------------------- | -------------------------------- | ------------------------------------- |
| Liệt kê tất cả nhân viên   | BAMBOOHR_GET_ALL_EMPLOYEES       | (không có)                            |
| Lấy chi tiết nhân viên     | BAMBOOHR_GET_EMPLOYEE            | id, fields                            |
| Theo dõi thay đổi          | BAMBOOHR_EMPLOYEE_GET_CHANGED    | since, type                           |
| Loại nghỉ phép             | BAMBOOHR_GET_META_TIME_OFF_TYPES | (không có)                            |
| Số dư nghỉ phép            | BAMBOOHR_GET_TIME_OFF_BALANCES   | employeeId                            |
| Liệt kê yêu cầu nghỉ phép  | BAMBOOHR_GET_TIME_OFF_REQUESTS   | start, end, employeeId                |
| Tạo yêu cầu nghỉ phép      | BAMBOOHR_CREATE_TIME_OFF_REQUEST | employeeId, timeOffTypeId, start, end |
| Cập nhật yêu cầu nghỉ phép | BAMBOOHR_UPDATE_TIME_OFF_REQUEST | requestId, status                     |
| Cập nhật nhân viên         | BAMBOOHR_UPDATE_EMPLOYEE         | id, (các cập nhật trường)             |
| Liệt kê người phụ thuộc    | BAMBOOHR_DEPENDENTS_GET_ALL      | employeeId                            |
| Bảo hiểm phúc lợi          | BAMBOOHR_BENEFIT_GET_COVERAGES   | (kiểm tra schema)                     |
