---
name: canva-automation
description: "Tự động hóa các tác vụ Canva qua Rube MCP (Composio): thiết kế, xuất khẩu, thư mục, mẫu thương hiệu, tự động điền. Luôn tìm kiếm công cụ trước để có schemas hiện tại."
requires:
  mcp: [rube]
---

# Tự động hóa Canva qua Rube MCP

Tự động hóa các hoạt động thiết kế Canva thông qua bộ công cụ Canva của Composio qua Rube MCP.

## Điều kiện tiên quyết

- Rube MCP phải được kết nối (RUBE_SEARCH_TOOLS có sẵn)
- Kết nối Canva đang hoạt động qua `RUBE_MANAGE_CONNECTIONS` với toolkit `canva`
- Luôn gọi `RUBE_SEARCH_TOOLS` trước để lấy schemas công cụ hiện tại

## Thiết lập

**Lấy Rube MCP**: Thêm `https://rube.app/mcp` làm máy chủ MCP trong cấu hình khách của bạn. Không cần khóa API — chỉ cần thêm endpoint và nó hoạt động.

1. Xác minh Rube MCP có sẵn bằng cách xác nhận `RUBE_SEARCH_TOOLS` phản hồi
2. Gọi `RUBE_MANAGE_CONNECTIONS` với toolkit `canva`
3. Nếu kết nối không ACTIVE, làm theo liên kết xác thực được trả về để hoàn tất OAuth Canva
4. Xác nhận trạng thái kết nối hiển thị ACTIVE trước khi chạy bất kỳ quy trình làm việc nào

## Quy trình làm việc Cốt lõi

### 1. Liệt kê và Duyệt Các thiết kế

**Khi nào sử dụng**: Người dùng muốn tìm các thiết kế hiện có hoặc duyệt thư viện Canva của họ

**Trình tự công cụ**:

1. `CANVA_LIST_USER_DESIGNS` - Liệt kê tất cả các thiết kế với các bộ lọc tùy chọn [Bắt buộc]

**Các tham số chính**:

- `query`: Thuật ngữ tìm kiếm để lọc thiết kế theo tên
- `continuation`: Token phân trang từ phản hồi trước
- `ownership`: Lọc theo 'owned', 'shared', hoặc 'any'
- `sort_by`: Trường sắp xếp (ví dụ: 'modified_at', 'title')

**Cạm bẫy**:

- Kết quả được phân trang; theo dõi token `continuation` cho đến khi vắng mặt
- Các thiết kế đã xóa có thể vẫn xuất hiện trong thời gian ngắn; kiểm tra trạng thái thiết kế
- Tìm kiếm dựa trên chuỗi con, không phải khớp mờ

### 2. Tạo và Thiết kế

**Khi nào sử dụng**: Người dùng muốn tạo một thiết kế Canva mới từ đầu hoặc từ một mẫu

**Trình tự công cụ**:

1. `CANVA_ACCESS_USER_SPECIFIC_BRAND_TEMPLATES_LIST` - Duyệt các mẫu thương hiệu có sẵn [Tùy chọn]
2. `CANVA_CREATE_CANVA_DESIGN_WITH_OPTIONAL_ASSET` - Tạo một thiết kế mới [Bắt buộc]

**Các tham số chính**:

- `design_type`: Loại thiết kế (ví dụ: 'Presentation', 'Poster', 'SocialMedia')
- `title`: Tên cho thiết kế mới
- `asset_id`: Tài sản tùy chọn để bao gồm trong thiết kế
- `width` / `height`: Kích thước tùy chỉnh bằng pixel

**Cạm bẫy**:

- Loại thiết kế phải khớp chính xác với các loại được xác định trước của Canva
- Kích thước tùy chỉnh có giới hạn tối thiểu và tối đa
- Tài sản phải được tải lên trước qua CANVA_CREATE_ASSET_UPLOAD_JOB trước khi tham chiếu

### 3. Tải lên Tài sản

**Khi nào sử dụng**: Người dùng muốn tải lên hình ảnh hoặc tệp lên Canva để sử dụng trong thiết kế

**Trình tự công cụ**:

1. `CANVA_CREATE_ASSET_UPLOAD_JOB` - Khởi tạo việc tải lên tài sản [Bắt buộc]
2. `CANVA_FETCH_ASSET_UPLOAD_JOB_STATUS` - Thăm dò cho đến khi tải lên hoàn tất [Bắt buộc]

**Các tham số chính**:

- `name`: Tên hiển thị cho tài sản
- `url`: URL công khai của tệp để tải lên (đối với tải lên dựa trên URL)
- `job_id`: ID công việc tải lên được trả về từ bước 1 (để thăm dò trạng thái)

**Cạm bẫy**:

- Tải lên là không đồng bộ; bạn PHẢI thăm dò trạng thái công việc cho đến khi nó hoàn tất
- Các định dạng được hỗ trợ bao gồm PNG, JPG, SVG, MP4, GIF
- Giới hạn kích thước tệp được áp dụng; các tệp lớn có thể mất nhiều thời gian hơn để xử lý
- `job_id` từ CREATE trả về ID cần thiết cho việc thăm dò trạng thái
- Các giá trị trạng thái: 'in_progress', 'success', 'failed'

### 4. Xuất khẩu Thiết kế

**Khi nào sử dụng**: Người dùng muốn tải xuống hoặc xuất khẩu một thiết kế Canva dưới dạng PDF, PNG, hoặc định dạng khác

**Trình tự công cụ**:

1. `CANVA_LIST_USER_DESIGNS` - Tìm thiết kế để xuất khẩu [Điều kiện tiên quyết]
2. `CANVA_CREATE_CANVA_DESIGN_EXPORT_JOB` - Bắt đầu quy trình xuất khẩu [Bắt buộc]
3. `CANVA_GET_DESIGN_EXPORT_JOB_RESULT` - Thăm dò cho đến khi xuất khẩu hoàn tất và lấy URL tải xuống [Bắt buộc]

**Các tham số chính**:

- `design_id`: ID của thiết kế để xuất khẩu
- `format`: Định dạng xuất khẩu ('pdf', 'png', 'jpg', 'svg', 'mp4', 'gif', 'pptx')
- `pages`: Số trang cụ thể để xuất khẩu (mảng)
- `quality`: Chất lượng xuất khẩu ('regular', 'high')
- `job_id`: ID công việc xuất khẩu để thăm dò trạng thái

**Cạm bẫy**:

- Xuất khẩu là không đồng bộ; bạn PHẢI thăm dò kết quả công việc cho đến khi nó hoàn tất
- URL tải xuống từ các xuất khẩu đã hoàn thành hết hạn sau một thời gian giới hạn
- Các thiết kế lớn với nhiều trang mất nhiều thời gian hơn để xuất khẩu
- Không phải tất cả các định dạng đều hỗ trợ tất cả các loại thiết kế (ví dụ: MP4 chỉ dành cho hoạt hình)
- Khoảng thời gian thăm dò: đợi 2-3 giây giữa các lần kiểm tra trạng thái

### 5. Tổ chức với Thư mục

**Khi nào sử dụng**: Người dùng muốn tạo thư mục hoặc tổ chức các thiết kế vào thư mục

**Trình tự công cụ**:

1. `CANVA_POST_FOLDERS` - Tạo một thư mục mới [Bắt buộc]
2. `CANVA_MOVE_ITEM_TO_SPECIFIED_FOLDER` - Di chuyển các thiết kế vào thư mục [Tùy chọn]

**Các tham số chính**:

- `name`: Tên thư mục
- `parent_folder_id`: Thư mục cha cho tổ chức lồng nhau
- `item_id`: ID của thiết kế hoặc tài sản để di chuyển
- `folder_id`: ID thư mục đích

**Cạm bẫy**:

- Tên thư mục phải là duy nhất trong cùng một thư mục cha
- Di chuyển các mục giữa các thư mục cập nhật vị trí của chúng ngay lập tức
- Các thư mục cấp gốc không có parent_folder_id

### 6. Tự động điền từ Mẫu Thương hiệu

**Khi nào sử dụng**: Người dùng muốn tạo các thiết kế bằng cách điền các trình giữ chỗ mẫu thương hiệu với dữ liệu

**Trình tự công cụ**:

1. `CANVA_ACCESS_USER_SPECIFIC_BRAND_TEMPLATES_LIST` - Liệt kê các mẫu thương hiệu có sẵn [Bắt buộc]
2. `CANVA_INITIATE_CANVA_DESIGN_AUTOFILL_JOB` - Bắt đầu tự động điền với dữ liệu [Bắt buộc]

**Các tham số chính**:

- `brand_template_id`: ID của mẫu thương hiệu để sử dụng
- `title`: Tiêu đề cho thiết kế được tạo
- `data`: Ánh xạ khóa-giá trị của tên trình giữ chỗ đến các giá trị thay thế

**Cạm bẫy**:

- Các trình giữ chỗ mẫu phải khớp chính xác (phân biệt chữ hoa chữ thường)
- Tự động điền là không đồng bộ; thăm dò để hoàn tất
- Chỉ các mẫu thương hiệu hỗ trợ tự động điền, không phải các thiết kế thông thường
- Các giá trị dữ liệu phải khớp với loại mong đợi cho mỗi trình giữ chỗ (văn bản, URL hình ảnh)

## Các Mẫu Phổ biến

### Mẫu Công việc Không đồng bộ (Async Job Pattern)

Nhiều hoạt động Canva là không đồng bộ:

```
1. Khởi tạo công việc (tải lên, xuất khẩu, tự động điền) -> lấy job_id
2. Thăm dò endpoint trạng thái với job_id mỗi 2-3 giây
3. Kiểm tra trạng thái 'success' hoặc 'failed'
4. Khi thành công, trích xuất kết quả (asset_id, download_url, design_id)
```

### Giải quyết ID

**Tên thiết kế -> ID thiết kế**:

```
1. Gọi CANVA_LIST_USER_DESIGNS với query=design_name
2. Tìm thiết kế khớp trong kết quả
3. Trích xuất trường id
```

**Tên mẫu thương hiệu -> ID mẫu**:

```
1. Gọi CANVA_ACCESS_USER_SPECIFIC_BRAND_TEMPLATES_LIST
2. Tìm mẫu theo tên
3. Trích xuất brand_template_id
```

### Phân trang (Pagination)

- Kiểm tra phản hồi cho token `continuation`
- Chuyển token trong tham số `continuation` của yêu cầu tiếp theo
- Tiếp tục cho đến khi `continuation` vắng mặt hoặc trống

## Các Cạm bẫy Đã biết

**Hoạt động Không đồng bộ**:

- Tải lên, xuất khẩu, và tự động điền đều là không đồng bộ
- Luôn thăm dò trạng thái công việc; không giả định hoàn thành ngay lập tức
- URL tải xuống từ xuất khẩu hết hạn; sử dụng chúng ngay

**Quản lý Tài sản**:

- Tài sản phải được tải lên trước khi chúng có thể được sử dụng trong thiết kế
- Công việc tải lên phải đạt trạng thái 'success' trước khi asset_id hợp lệ
- Các định dạng được hỗ trợ thay đổi; kiểm tra tài liệu Canva cho các giới hạn hiện tại

**Giới hạn Tốc độ**:

- API Canva có giới hạn tốc độ cho mỗi endpoint
- Thực hiện backoff theo cấp số nhân cho các hoạt động hàng loạt
- Các hoạt động lô khi có thể để giảm các cuộc gọi API

**Phân tích Phản hồi**:

- Dữ liệu phản hồi có thể được lồng dưới khóa `data`
- Phản hồi trạng thái công việc bao gồm các trường khác nhau dựa trên trạng thái hoàn thành
- Phân tích một cách phòng thủ với các dự phòng cho các trường tùy chọn

## Tham khảo Nhanh

| Tác vụ                | Slug Công cụ                                    | Tham số Chính           |
| --------------------- | ----------------------------------------------- | ----------------------- |
| Liệt kê thiết kế      | CANVA_LIST_USER_DESIGNS                         | query, continuation     |
| Tạo thiết kế          | CANVA_CREATE_CANVA_DESIGN_WITH_OPTIONAL_ASSET   | design_type, title      |
| Tải lên tài sản       | CANVA_CREATE_ASSET_UPLOAD_JOB                   | name, url               |
| Kiểm tra tải lên      | CANVA_FETCH_ASSET_UPLOAD_JOB_STATUS             | job_id                  |
| Xuất khẩu thiết kế    | CANVA_CREATE_CANVA_DESIGN_EXPORT_JOB            | design_id, format       |
| Lấy xuất khẩu         | CANVA_GET_DESIGN_EXPORT_JOB_RESULT              | job_id                  |
| Tạo thư mục           | CANVA_POST_FOLDERS                              | name, parent_folder_id  |
| Di chuyển vào thư mục | CANVA_MOVE_ITEM_TO_SPECIFIED_FOLDER             | item_id, folder_id      |
| Liệt kê mẫu           | CANVA_ACCESS_USER_SPECIFIC_BRAND_TEMPLATES_LIST | (không có)              |
| Tự động điền mẫu      | CANVA_INITIATE_CANVA_DESIGN_AUTOFILL_JOB        | brand_template_id, data |
