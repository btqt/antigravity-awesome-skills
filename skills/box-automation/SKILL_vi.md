---
name: box-automation
description: "Tự động hóa các hoạt động lưu trữ đám mây Box bao gồm tải lên/tải xuống tệp, tìm kiếm, quản lý thư mục, chia sẻ, cộng tác, và truy vấn metadata thông qua Rube MCP (Composio). Luôn tìm kiếm công cụ trước để có schema hiện tại."
requires:
  mcp: [rube]
---

# Tự động hóa Box thông qua Rube MCP

Tự động hóa các hoạt động Box bao gồm tải lên/tải xuống tệp, tìm kiếm nội dung, quản lý thư mục, cộng tác, truy vấn metadata, và yêu cầu chữ ký thông qua bộ công cụ Box của Composio.

## Điều kiện tiên quyết

- Rube MCP phải được kết nối (RUBE_SEARCH_TOOLS khả dụng)
- Kết nối Box đang hoạt động qua `RUBE_MANAGE_CONNECTIONS` với toolkit `box`
- Luôn gọi `RUBE_SEARCH_TOOLS` trước tiên để lấy schema công cụ hiện tại

## Cài đặt

**Lấy Rube MCP**: Thêm `https://rube.app/mcp` làm máy chủ MCP trong cấu hình client của bạn. Không cần API key — chỉ cần thêm endpoint và nó sẽ hoạt động.

1. Xác minh Rube MCP khả dụng bằng cách xác nhận `RUBE_SEARCH_TOOLS` phản hồi
2. Gọi `RUBE_MANAGE_CONNECTIONS` với toolkit `box`
3. Nếu kết nối không ACTIVE, làm theo liên kết xác thực được trả về để hoàn tất xác thực Box OAuth
4. Xác nhận trạng thái kết nối hiển thị ACTIVE trước khi chạy bất kỳ quy trình làm việc nào

## Các Quy trình làm việc Cốt lõi

### 1. Tải lên và Tải xuống Tệp

**Khi nào sử dụng**: Người dùng muốn tải lên tệp vào Box hoặc tải xuống tệp từ đó

**Trình tự công cụ**:

1. `BOX_SEARCH_FOR_CONTENT` - Tìm thư mục mục tiêu nếu đường dẫn không xác định [Tiên quyết]
2. `BOX_GET_FOLDER_INFORMATION` - Xác minh thư mục tồn tại và lấy folder_id [Tiên quyết]
3. `BOX_LIST_ITEMS_IN_FOLDER` - Duyệt nội dung thư mục và khám phá ID tệp [Tùy chọn]
4. `BOX_UPLOAD_FILE` - Tải lên một tệp vào thư mục cụ thể [Bắt buộc cho tải lên]
5. `BOX_DOWNLOAD_FILE` - Tải xuống một tệp theo file_id [Bắt buộc cho tải xuống]
6. `BOX_CREATE_ZIP_DOWNLOAD` - Đóng gói nhiều tệp/thư mục vào một tệp zip [Tùy chọn]

**Tham số chính**:

- `parent_id`: ID thư mục đích tải lên (sử dụng `"0"` cho thư mục gốc)
- `file`: Đối tượng FileUploadable với `s3key`, `mimetype`, và `name` cho tải lên
- `file_id`: Định danh tệp duy nhất cho tải xuống
- `version`: ID phiên bản tệp tùy chọn để tải xuống các phiên bản cụ thể
- `fields`: Danh sách các thuộc tính cần trả về, phân tách bằng dấu phẩy

**Cạm bẫy**:

- Tải lên thư mục có tên tệp hiện có có thể kích hoạt hành vi xung đột; quyết định ghi đè hay đổi tên
- Các tệp trên 50MB nên sử dụng API tải lên phân đoạn (chunk upload) (không khả dụng qua các công cụ tiêu chuẩn)
- Phần `attributes` của tải lên phải đến trước phần `file` nếu không bạn sẽ nhận lỗi HTTP 400 với `metadata_after_file_contents`
- ID tệp và ID thư mục là chuỗi số có thể trích xuất từ URL ứng dụng web Box (ví dụ: `https://*.app.box.com/files/123` cho file_id `"123"`)

### 2. Tìm kiếm và Duyệt Nội dung

**Khi nào sử dụng**: Người dùng muốn tìm tệp, thư mục, hoặc liên kết web theo tên, nội dung, hoặc metadata

**Trình tự công cụ**:

1. `BOX_SEARCH_FOR_CONTENT` - Tìm kiếm toàn văn bản trên tệp, thư mục, và liên kết web [Bắt buộc]
2. `BOX_LIST_ITEMS_IN_FOLDER` - Duyệt nội dung của một thư mục cụ thể [Tùy chọn]
3. `BOX_GET_FILE_INFORMATION` - Lấy metadata chi tiết cho một tệp cụ thể [Tùy chọn]
4. `BOX_GET_FOLDER_INFORMATION` - Lấy metadata chi tiết cho một thư mục cụ thể [Tùy chọn]
5. `BOX_QUERY_FILES_FOLDERS_BY_METADATA` - Tìm kiếm theo giá trị mẫu metadata [Tùy chọn]
6. `BOX_LIST_RECENTLY_ACCESSED_ITEMS` - Liệt kê các mục truy cập gần đây [Tùy chọn]

**Tham số chính**:

- `query`: Chuỗi tìm kiếm hỗ trợ toán tử (`""` khớp chính xác, `AND`, `OR`, `NOT` - chỉ chữ hoa)
- `type`: Lọc theo `"file"`, `"folder"`, hoặc `"web_link"`
- `ancestor_folder_ids`: Giới hạn tìm kiếm trong các thư mục cụ thể (ID phân tách bằng dấu phẩy)
- `file_extensions`: Lọc theo loại tệp (phân tách bằng dấu phẩy, không có dấu chấm)
- `content_types`: Tìm kiếm trong `"name"`, `"description"`, `"file_content"`, `"comments"`, `"tags"`
- `created_at_range` / `updated_at_range`: Bộ lọc ngày dưới dạng timestamp RFC3339 phân tách bằng dấu phẩy
- `limit`: Kết quả mỗi trang (mặc định 30)
- `offset`: Offset phân trang (tối đa 10000)
- `folder_id`: Cho `LIST_ITEMS_IN_FOLDER` (sử dụng `"0"` cho gốc)

**Cạm bẫy**:

- Các truy vấn với offset > 10000 bị từ chối với HTTP 400
- `BOX_SEARCH_FOR_CONTENT` yêu cầu tham số `query` hoặc `mdfilters`
- Các bộ lọc cấu hình sai có thể âm thầm bỏ qua các mục mong đợi; xác thực với các truy vấn thử nghiệm nhỏ trước
- Các toán tử Boolean (`AND`, `OR`, `NOT`) phải viết hoa
- `BOX_LIST_ITEMS_IN_FOLDER` yêu cầu phân trang qua `marker` hoặc `offset`/`usemarker`; danh sách một phần là phổ biến
- Các thư mục tiêu chuẩn sắp xếp các mục theo loại trước (thư mục trước tệp trước liên kết web)

### 3. Quản lý Thư mục

**Khi nào sử dụng**: Người dùng muốn tạo, cập nhật, di chuyển, sao chép, hoặc xóa thư mục

**Trình tự công cụ**:

1. `BOX_GET_FOLDER_INFORMATION` - Xác minh thư mục tồn tại và kiểm tra quyền [Tiên quyết]
2. `BOX_CREATE_FOLDER` - Tạo thư mục mới [Bắt buộc để tạo]
3. `BOX_UPDATE_FOLDER` - Đổi tên, di chuyển, hoặc cập nhật cài đặt thư mục [Bắt buộc để cập nhật]
4. `BOX_COPY_FOLDER` - Sao chép thư mục sang vị trí mới [Tùy chọn]
5. `BOX_DELETE_FOLDER` - Di chuyển thư mục vào thùng rác [Bắt buộc để xóa]
6. `BOX_PERMANENTLY_REMOVE_FOLDER` - Xóa vĩnh viễn thư mục đã xóa [Tùy chọn]

**Tham số chính**:

- `name`: Tên thư mục (không có `/`, `\`, khoảng trắng cuối cùng, hoặc `.`/`..`)
- `parent__id`: ID thư mục cha (sử dụng `"0"` cho gốc)
- `folder_id`: ID thư mục mục tiêu cho các hoạt động
- `parent.id`: ID thư mục đích cho di chuyển thông qua `BOX_UPDATE_FOLDER`
- `recursive`: Đặt `true` để xóa các thư mục không rỗng
- `shared_link`: Đối tượng với `access`, `password`, `permissions` để tạo liên kết chia sẻ trên thư mục
- `description`, `tags`: Các trường metadata tùy chọn

**Cạm bẫy**:

- `BOX_DELETE_FOLDER` di chuyển vào thùng rác theo mặc định; sử dụng `BOX_PERMANENTLY_REMOVE_FOLDER` để xóa vĩnh viễn
- Các thư mục không rỗng yêu cầu `recursive: true` để xóa
- Thư mục gốc (ID `"0"`) không thể sao chép hoặc xóa
- Tên thư mục không được chứa `/`, `\`, ASCII không in được, hoặc khoảng trắng cuối cùng
- Di chuyển thư mục yêu cầu đặt `parent.id` thông qua `BOX_UPDATE_FOLDER`

### 4. Chia sẻ Tệp và Quản lý Cộng tác

**Khi nào sử dụng**: Người dùng muốn chia sẻ tệp, quản lý quyền truy cập, hoặc xử lý cộng tác

**Trình tự công cụ**:

1. `BOX_GET_FILE_INFORMATION` - Lấy chi tiết tệp và trạng thái chia sẻ hiện tại [Tiên quyết]
2. `BOX_LIST_FILE_COLLABORATIONS` - Liệt kê những người có quyền truy cập vào tệp [Bắt buộc]
3. `BOX_UPDATE_COLLABORATION` - Thay đổi cấp độ truy cập hoặc chấp nhận/từ chối lời mời [Bắt buộc]
4. `BOX_GET_COLLABORATION` - Lấy chi tiết của một cộng tác cụ thể [Tùy chọn]
5. `BOX_UPDATE_FILE` - Tạo liên kết chia sẻ, khóa tệp, hoặc cập nhật quyền [Tùy chọn]
6. `BOX_UPDATE_FOLDER` - Tạo liên kết chia sẻ trên thư mục [Tùy chọn]

**Tham số chính**:

- `collaboration_id`: Định danh cộng tác duy nhất
- `role`: Cấp độ truy cập (`"editor"`, `"viewer"`, `"co-owner"`, `"owner"`, `"previewer"`, `"uploader"`, `"viewer uploader"`, `"previewer uploader"`)
- `status`: `"accepted"`, `"pending"`, hoặc `"rejected"` cho lời mời cộng tác
- `file_id`: Tệp để chia sẻ hoặc quản lý
- `lock__access`: Đặt thành `"lock"` để khóa tệp
- `permissions__can__download`: `"company"` hoặc `"open"` cho quyền tải xuống

**Cạm bẫy**:

- Chỉ một số vai trò nhất định mới có thể mời cộng tác viên; không đủ quyền sẽ gây ra lỗi ủy quyền
- `can_view_path` làm tăng thời gian tải cho trang "All Files" của người được mời; giới hạn ở 1000 mỗi người dùng
- Hết hạn cộng tác yêu cầu bật cài đặt quản trị doanh nghiệp
- Tên tham số lồng nhau sử dụng dấu gạch dưới kép (ví dụ, `lock__access`, `parent__id`)

### 5. Yêu cầu Chữ ký Box (Box Sign Requests)

**Khi nào sử dụng**: Người dùng muốn quản lý yêu cầu chữ ký tài liệu

**Trình tự công cụ**:

1. `BOX_LIST_BOX_SIGN_REQUESTS` - Liệt kê tất cả các yêu cầu chữ ký [Bắt buộc]
2. `BOX_GET_BOX_SIGN_REQUEST_BY_ID` - Lấy chi tiết của một yêu cầu chữ ký cụ thể [Tùy chọn]
3. `BOX_CANCEL_BOX_SIGN_REQUEST` - Hủy một yêu cầu chữ ký đang chờ xử lý [Tùy chọn]

**Tham số chính**:

- `sign_request_id`: UUID của yêu cầu chữ ký
- `shared_requests`: Đặt `true` để bao gồm các yêu cầu mà người dùng là cộng tác viên (không phải chủ sở hữu)
- `senders`: Lọc theo email người gửi (yêu cầu `shared_requests: true`)
- `limit` / `marker`: Tham số phân trang

**Cạm bẫy**:

- Yêu cầu Box Sign được bật cho tài khoản doanh nghiệp
- Các tệp chữ ký đã xóa hoặc thư mục cha đã xóa khiến yêu cầu không xuất hiện trong danh sách
- Chỉ người tạo mới có thể hủy yêu cầu chữ ký
- Trạng thái yêu cầu chữ ký bao gồm: `converting`, `created`, `sent`, `viewed`, `signed`, `declined`, `cancelled`, `expired`, `error_converting`, `error_sending`

## Các Mẫu Phổ biến

### Giải quyết ID (ID Resolution)

Box sử dụng ID chuỗi số cho tất cả các thực thể:

- **Thư mục gốc**: Luôn là ID `"0"`
- **File ID từ URL**: `https://*.app.box.com/files/123` đưa ra file_id `"123"`
- **Folder ID từ URL**: `https://*.app.box.com/folder/123` đưa ra folder_id `"123"`
- **Tìm kiếm để lấy ID**: Sử dụng `BOX_SEARCH_FOR_CONTENT` để tìm các mục, sau đó trích xuất ID từ kết quả
- **ETag**: Sử dụng `if_match` với ETag của tệp cho các hoạt động xóa đồng thời an toàn

### Phân trang

Box hỗ trợ hai phương pháp phân trang:

- **Dựa trên Offset**: Sử dụng `offset` + `limit` (max offset 10000)
- **Dựa trên Marker**: Đặt `usemarker: true` và theo dõi `marker` từ phản hồi (ưu tiên cho bộ dữ liệu lớn)
- Luôn phân trang đến khi hoàn thành để tránh kết quả một phần

### Tham số Lồng nhau

Các công cụ Box sử dụng ký hiệu gạch dưới kép cho các đối tượng lồng nhau:

- `parent__id` cho tham chiếu thư mục cha
- `lock__access`, `lock__expires__at`, `lock__is__download__prevented` cho khóa tệp
- `permissions__can__download` cho quyền tải xuống

## Các Cạm bẫy Đã biết

### Định dạng ID

- Tất cả ID là chuỗi số (ví dụ, `"123456"`, không phải số nguyên)
- Thư mục gốc luôn là `"0"`
- ID tệp và thư mục có thể được trích xuất từ URL ứng dụng web Box

### Giới hạn Tốc độ (Rate Limits)

- Box API có giới hạn tốc độ cho mỗi endpoint
- Các hoạt động tìm kiếm và liệt kê nên sử dụng phân trang một cách có trách nhiệm
- Các hoạt động hàng loạt nên bao gồm độ trễ giữa các yêu cầu

### Đặc điểm quái dị của Tham số

- Tham số `fields` thay đổi hình dạng phản hồi: khi được chỉ định, chỉ trả về đại diện mini + các trường được yêu cầu
- Tìm kiếm yêu cầu `query` hoặc `mdfilters`; cả hai đều tùy chọn riêng lẻ nhưng một trong hai phải có mặt
- `BOX_UPDATE_FILE` với `lock` đặt thành `null` sẽ xóa khóa (nếu API thô hỗ trợ)
- Định dạng trường `from` truy vấn metadata: `enterprise_{enterprise_id}.templateKey` hoặc `global.templateKey`

### Quyền hạn

- Xóa sẽ thất bại nếu không đủ quyền; luôn xử lý phản hồi lỗi
- Vai trò cộng tác xác định những hoạt động nào được phép
- Cài đặt doanh nghiệp có thể hạn chế một số tùy chọn chia sẻ nhất định

## Tham khảo Nhanh

| Tác vụ                 | Tool Slug                             | Tham số chính                          |
| ---------------------- | ------------------------------------- | -------------------------------------- |
| Tìm kiếm nội dung      | `BOX_SEARCH_FOR_CONTENT`              | `query`, `type`, `ancestor_folder_ids` |
| Liệt kê mục thư mục    | `BOX_LIST_ITEMS_IN_FOLDER`            | `folder_id`, `limit`, `marker`         |
| Lấy thông tin tệp      | `BOX_GET_FILE_INFORMATION`            | `file_id`, `fields`                    |
| Lấy thông tin thư mục  | `BOX_GET_FOLDER_INFORMATION`          | `folder_id`, `fields`                  |
| Tải lên tệp            | `BOX_UPLOAD_FILE`                     | `file`, `parent_id`                    |
| Tải xuống tệp          | `BOX_DOWNLOAD_FILE`                   | `file_id`                              |
| Tạo thư mục            | `BOX_CREATE_FOLDER`                   | `name`, `parent__id`                   |
| Cập nhật thư mục       | `BOX_UPDATE_FOLDER`                   | `folder_id`, `name`, `parent`          |
| Sao chép thư mục       | `BOX_COPY_FOLDER`                     | `folder_id`, `parent__id`              |
| Xóa thư mục            | `BOX_DELETE_FOLDER`                   | `folder_id`, `recursive`               |
| Xóa thư mục vĩnh viễn  | `BOX_PERMANENTLY_REMOVE_FOLDER`       | folder_id                              |
| Cập nhật tệp           | `BOX_UPDATE_FILE`                     | `file_id`, `name`, `parent__id`        |
| Xóa tệp                | `BOX_DELETE_FILE`                     | `file_id`, `if_match`                  |
| Liệt kê cộng tác       | `BOX_LIST_FILE_COLLABORATIONS`        | `file_id`                              |
| Cập nhật cộng tác      | `BOX_UPDATE_COLLABORATION`            | `collaboration_id`, `role`             |
| Lấy cộng tác           | `BOX_GET_COLLABORATION`               | `collaboration_id`                     |
| Truy vấn theo metadata | `BOX_QUERY_FILES_FOLDERS_BY_METADATA` | `from`, `ancestor_folder_id`, `query`  |
| Liệt kê bộ sưu tập     | `BOX_LIST_ALL_COLLECTIONS`            | (none)                                 |
| Liệt kê mục bộ sưu tập | `BOX_LIST_COLLECTION_ITEMS`           | `collection_id`                        |
| Liệt kê yêu cầu chữ ký | `BOX_LIST_BOX_SIGN_REQUESTS`          | `limit`, `marker`                      |
| Lấy yêu cầu chữ ký     | `BOX_GET_BOX_SIGN_REQUEST_BY_ID`      | `sign_request_id`                      |
| Hủy yêu cầu chữ ký     | `BOX_CANCEL_BOX_SIGN_REQUEST`         | `sign_request_id`                      |
| Mục gần đây            | `BOX_LIST_RECENTLY_ACCESSED_ITEMS`    | (none)                                 |
| Tạo tải xuống zip      | `BOX_CREATE_ZIP_DOWNLOAD`             | item IDs                               |
