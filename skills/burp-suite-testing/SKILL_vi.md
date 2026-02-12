---
name: Burp Suite Web Application Testing
description: Kỹ năng này nên được sử dụng khi người dùng yêu cầu "chặn lưu lượng HTTP", "sửa đổi yêu cầu web", "sử dụng Burp Suite để kiểm thử", "thực hiện quét lỗ hổng web", "kiểm thử với Burp Repeater", "phân tích lịch sử HTTP", hoặc "cấu hình proxy để kiểm thử web". Nó cung cấp hướng dẫn toàn diện để sử dụng các tính năng cốt lõi của Burp Suite cho kiểm thử bảo mật ứng dụng web.
metadata:
  author: zebbern
  version: "1.1"
---

# Kiểm thử Ứng dụng Web với Burp Suite

## Mục đích

Thực hiện kiểm thử bảo mật ứng dụng web toàn diện bằng cách sử dụng bộ công cụ tích hợp của Burp Suite, bao gồm chặn và sửa đổi lưu lượng HTTP, phân tích và phát lại yêu cầu, quét lỗ hổng tự động, và các quy trình kiểm thử thủ công. Kỹ năng này cho phép phát hiện và khai thác có hệ thống các lỗ hổng ứng dụng web thông qua phương pháp kiểm thử dựa trên proxy.

## Đầu vào / Điều kiện tiên quyết

### Công cụ Yêu cầu

- Burp Suite Community hoặc Professional Edition đã được cài đặt
- Trình duyệt nhúng của Burp hoặc trình duyệt bên ngoài đã được cấu hình
- URL ứng dụng web mục tiêu
- Thông tin xác thực hợp lệ để kiểm thử đã xác thực (nếu áp dụng)

### Thiết lập Môi trường

- Burp Suite được khởi chạy với dự án tạm thời hoặc có tên
- Proxy listener hoạt động trên 127.0.0.1:8080 (mặc định)
- Trình duyệt được cấu hình để sử dụng Burp proxy (hoặc sử dụng trình duyệt của Burp)
- Chứng chỉ CA được cài đặt để chặn HTTPS

### So sánh Phiên bản

| Tính năng  | Community | Professional |
| ---------- | --------- | ------------ |
| Proxy      | ✓         | ✓            |
| Repeater   | ✓         | ✓            |
| Intruder   | Hạn chế   | Đầy đủ       |
| Scanner    | ✗         | ✓            |
| Extensions | ✓         | ✓            |

## Đầu ra / Giao nộp

### Đầu ra Chính

- Các yêu cầu/phản hồi HTTP đã bị chặn và sửa đổi
- Báo cáo quét lỗ hổng với tư vấn khắc phục
- Lịch sử HTTP và tài liệu site map
- Bằng chứng khái niệm (PoC) khai thác cho các lỗ hổng được xác định

## Quy trình làm việc Cốt lõi

### Giai đoạn 1: Chặn Lưu lượng HTTP

#### Khởi chạy Trình duyệt của Burp

Điều hướng đến trình duyệt tích hợp để tích hợp proxy liền mạch:

1. Mở Burp Suite và tạo/mở dự án
2. Đi tới tab **Proxy > Intercept**
3. Nhấp **Open Browser** để khởi chạy trình duyệt được cấu hình sẵn
4. Định vị các cửa sổ để xem cả Burp và trình duyệt cùng lúc

#### Cấu hình Chặn (Interception)

Kiểm soát các yêu cầu nào được bắt:

```
Proxy > Intercept > Intercept is on/off toggle

Khi ON: Các yêu cầu tạm dừng để xem xét/sửa đổi
Khi OFF: Các yêu cầu đi qua, được ghi vào lịch sử
```

#### Chặn và Chuyển tiếp Yêu cầu

Xử lý lưu lượng bị chặn:

1. Đặt công tắc chặn thành **Intercept on**
2. Điều hướng đến URL mục tiêu trong trình duyệt
3. Quan sát yêu cầu được giữ trong tab Proxy > Intercept
4. Xem xét nội dung yêu cầu (headers, parameters, body)
5. Nhấp **Forward** để gửi yêu cầu đến máy chủ
6. Tiếp tục chuyển tiếp các yêu cầu tiếp theo cho đến khi trang tải

#### Xem Lịch sử HTTP

Truy cập nhật ký lưu lượng đầy đủ:

1. Đi tới tab **Proxy > HTTP history**
2. Nhấp vào bất kỳ mục nào để xem yêu cầu/phản hồi đầy đủ
3. Sắp xếp bằng cách nhấp vào tiêu đề cột (# cho thứ tự thời gian)
4. Sử dụng bộ lọc để tập trung vào lưu lượng liên quan

### Giai đoạn 2: Sửa đổi Yêu cầu

#### Chặn và Sửa đổi

Thay đổi tham số yêu cầu trước khi chuyển tiếp:

1. Bật chặn: **Intercept on**
2. Kích hoạt yêu cầu mục tiêu trong trình duyệt
3. Tìm tham số cần sửa đổi trong yêu cầu bị chặn
4. Chỉnh sửa giá trị trực tiếp trong trình chỉnh sửa yêu cầu
5. Nhấp **Forward** để gửi yêu cầu đã sửa đổi

#### Các Mục tiêu Sửa đổi Phổ biến

| Mục tiêu         | Ví dụ          | Mục đích                     |
| ---------------- | -------------- | ---------------------------- |
| Tham số giá      | `price=1`      | Kiểm thử logic nghiệp vụ     |
| ID người dùng    | `userId=admin` | Kiểm thử kiểm soát truy cập  |
| Giá trị số lượng | `qty=-1`       | Kiểm thử xác thực đầu vào    |
| Trường ẩn        | `isAdmin=true` | Kiểm thử leo thang đặc quyền |

#### Ví dụ: Thao tác Giá

```http
POST /cart HTTP/1.1
Host: target.com
Content-Type: application/x-www-form-urlencoded

productId=1&quantity=1&price=100

# Sửa đổi thành:
productId=1&quantity=1&price=1
```

Kết quả: Mục được thêm vào giỏ hàng với giá đã sửa đổi.

### Giai đoạn 3: Thiết lập Phạm vi Mục tiêu (Target Scope)

#### Định nghĩa Phạm vi

Tập trung kiểm thử vào mục tiêu cụ thể:

1. Đi tới **Target > Site map**
2. Nhấp chuột phải vào máy chủ mục tiêu trong bảng điều khiển bên trái
3. Chọn **Add to scope**
4. Khi được nhắc, nhấp **Yes** để loại trừ lưu lượng ngoài phạm vi

#### Lọc theo Phạm vi

Loại bỏ nhiễu từ lịch sử HTTP:

1. Nhấp vào bộ lọc hiển thị phía trên lịch sử HTTP
2. Chọn **Show only in-scope items**
3. Lịch sử hiện chỉ hiển thị lưu lượng trang web mục tiêu

#### Lợi ích Phạm vi

- Giảm lộn xộn từ các yêu cầu của bên thứ ba
- Ngăn chặn việc kiểm thử vô tình các trang web ngoài phạm vi
- Cải thiện hiệu quả quét
- Tạo báo cáo sạch hơn

### Giai đoạn 4: Sử dụng Burp Repeater

#### Gửi Yêu cầu đến Repeater

Chuẩn bị yêu cầu cho kiểm thử thủ công:

1. Xác định yêu cầu thú vị trong lịch sử HTTP
2. Nhấp chuột phải vào yêu cầu và chọn **Send to Repeater**
3. Đi tới tab **Repeater** để truy cập yêu cầu

#### Sửa đổi và Gửi lại

Kiểm thử các đầu vào khác nhau một cách hiệu quả:

```
1. Xem yêu cầu trong tab Repeater
2. Sửa đổi giá trị tham số
3. Nhấp Send để gửi yêu cầu
4. Xem xét phản hồi trong bảng điều khiển bên phải
5. Sử dụng các mũi tên điều hướng để xem lại lịch sử yêu cầu
```

#### Quy trình Kiểm thử Repeater

```
Yêu cầu Gốc:
GET /product?productId=1 HTTP/1.1

Test 1: productId=2    → Phản hồi sản phẩm hợp lệ
Test 2: productId=999  → Phản hồi Not Found
Test 3: productId='    → Phản hồi Lỗi/ngoại lệ
Test 4: productId=1 OR 1=1 → Kiểm thử SQL injection
```

#### Phân tích Phản hồi

Tìm kiếm các chỉ báo về lỗ hổng:

- Thông báo lỗi tiết lộ stack traces
- Tiết lộ thông tin framework/phiên bản
- Độ dài phản hồi khác nhau cho thấy lỗi logic
- Sự khác biệt về thời gian gợi ý blind injection
- Dữ liệu không mong muốn trong phản hồi

### Giai đoạn 5: Chạy Quét Tự động

#### Khởi chạy Quét Mới

Bắt đầu quét lỗ hổng (Chỉ bản Professional):

1. Đi tới tab **Dashboard**
2. Nhấp **New scan**
3. Nhập URL mục tiêu vào trường **URLs to scan**
4. Cấu hình cài đặt quét

#### Tùy chọn Cấu hình Quét

| Chế độ      | Mô tả                     | Thời lượng |
| ----------- | ------------------------- | ---------- |
| Lightweight | Tổng quan cấp cao         | ~15 phút   |
| Fast        | Kiểm tra lỗ hổng nhanh    | ~30 phút   |
| Balanced    | Quét toàn diện tiêu chuẩn | ~1-2 giờ   |
| Deep        | Kiểm thử kỹ lưỡng         | Vài giờ    |

#### Giám sát Tiến trình Quét

Theo dõi hoạt động quét:

1. Xem trạng thái tác vụ trong **Dashboard**
2. Xem **Target > Site map** cập nhật trong thời gian thực
3. Kiểm tra tab **Issues** để tìm các lỗ hổng được phát hiện

#### Xem xét Các Vấn đề Được xác định

Phân tích kết quả quét:

1. Chọn tác vụ quét trong Dashboard
2. Đi tới tab **Issues**
3. Nhấp vào vấn đề để xem:
   - **Advisory**: Mô tả và khắc phục
   - **Request**: Yêu cầu HTTP kích hoạt
   - **Response**: Phản hồi máy chủ hiển thị lỗ hổng

### Giai đoạn 6: Tấn công Intruder

#### Cấu hình Intruder

Thiết lập tấn công tự động:

1. Gửi yêu cầu đến Intruder (nhấp chuột phải > Send to Intruder)
2. Đi tới tab **Intruder**
3. Định nghĩa các vị trí payload bằng cách sử dụng dấu §
4. Chọn loại tấn công

#### Các Loại Tấn công

| Loại          | Mô tả                           | Trường hợp Sử dụng              |
| ------------- | ------------------------------- | ------------------------------- |
| Sniper        | Một vị trí, lặp lại payload     | Fuzzing một tham số             |
| Battering ram | Cùng một payload cho mọi vị trí | Kiểm thử thông tin xác thực     |
| Pitchfork     | Lặp lại payload song song       | Các cặp tên người dùng:mật khẩu |
| Cluster bomb  | Tất cả các kết hợp payload      | Brute force đầy đủ              |

#### Cấu hình Payloads

```
Positions Tab:
POST /login HTTP/1.1
...
username=§admin§&password=§password§

Payloads Tab:
Set 1: admin, user, test, guest
Set 2: password, 123456, admin, letmein
```

#### Phân tích Kết quả

Xem xét đầu ra tấn công:

- Sắp xếp theo độ dài phản hồi để tìm điểm bất thường
- Lọc theo mã trạng thái cho các lần thử thành công
- Sử dụng grep để tìm kiếm các chuỗi cụ thể
- Xuất kết quả để làm tài liệu

## Tham khảo Nhanh

### Phím tắt Bàn phím

| Hành động           | Windows/Linux | macOS |
| ------------------- | ------------- | ----- |
| Chuyển tiếp yêu cầu | Ctrl+F        | Cmd+F |
| Bỏ yêu cầu          | Ctrl+D        | Cmd+D |
| Gửi đến Repeater    | Ctrl+R        | Cmd+R |
| Gửi đến Intruder    | Ctrl+I        | Cmd+I |
| Bật/tắt chặn        | Ctrl+T        | Cmd+T |

### Payloads Kiểm thử Phổ biến

```
# SQL Injection
' OR '1'='1
' OR '1'='1'--
1 UNION SELECT NULL--

# XSS
<script>alert(1)</script>
"><img src=x onerror=alert(1)>
javascript:alert(1)

# Path Traversal
../../../etc/passwd
..\..\..\..\windows\win.ini

# Command Injection
; ls -la
| cat /etc/passwd
`whoami`
```

### Mẹo Sửa đổi Yêu cầu

- Nhấp chuột phải để có các tùy chọn menu ngữ cảnh
- Sử dụng decoder để mã hóa/giải mã
- So sánh các yêu cầu bằng công cụ Comparer
- Lưu các yêu cầu thú vị vào dự án

## Ràng buộc và Lan can (Guardrails)

### Ranh giới Hoạt động

- Chỉ kiểm thử các ứng dụng được ủy quyền
- Cấu hình phạm vi để ngăn chặn kiểm thử vô tình ngoài phạm vi
- Giới hạn tốc độ quét để tránh từ chối dịch vụ
- Tài liệu hóa tất cả các phát hiện và hành động

### Hạn chế Kỹ thuật

- Community Edition thiếu trình quét tự động
- Một số trang web có thể chặn lưu lượng proxy
- HSTS/ghim chứng chỉ có thể yêu cầu cấu hình thêm
- Quét nặng có thể kích hoạt chặn WAF

### Các Thực hành Tốt nhất

- Luôn đặt phạm vi mục tiêu trước khi kiểm thử rộng rãi
- Sử dụng trình duyệt của Burp để chặn đáng tin cậy
- Lưu dự án thường xuyên để bảo quản công việc
- Xem xét kết quả quét thủ công để tìm dương tính giả

## Ví dụ

### Ví dụ 1: Kiểm thử Logic Nghiệp vụ

**Kịch bản**: Thao tác giá thương mại điện tử

1. Thêm mục vào giỏ hàng bình thường, chặn yêu cầu
2. Xác định tham số `price=9999` trong POST body
3. Sửa đổi thành `price=1`
4. Chuyển tiếp yêu cầu
5. Hoàn tất thanh toán ở mức giá đã thao tác

**Phát hiện**: Máy chủ tin tưởng giá trị giá do máy khách cung cấp.

### Ví dụ 2: Bỏ qua Xác thực

**Kịch bản**: Kiểm thử biểu mẫu đăng nhập

1. Gửi thông tin xác thực hợp lệ, bắt yêu cầu trong Repeater
2. Gửi đến Repeater để kiểm thử
3. Thử: `username=admin' OR '1'='1'--`
4. Quan sát phản hồi đăng nhập thành công

**Phát hiện**: SQL injection trong xác thực.

### Ví dụ 3: Tiết lộ Thông tin

**Kịch bản**: Thu thập thông tin dựa trên lỗi

1. Điều hướng đến trang sản phẩm, quan sát tham số `productId`
2. Gửi yêu cầu đến Repeater
3. Thay đổi `productId=1` thành `productId=test`
4. Quan sát lỗi dài dòng tiết lộ phiên bản framework

**Phát hiện**: Apache Struts 2.5.12 bị tiết lộ trong stack trace.

## Khắc phục sự cố

### Trình duyệt Không Kết nối Qua Proxy

- Xác minh proxy listener đang hoạt động (Proxy > Options)
- Kiểm tra cài đặt proxy trình duyệt trỏ đến 127.0.0.1:8080
- Đảm bảo không có tường lửa chặn kết nối cục bộ
- Sử dụng trình duyệt nhúng của Burp để thiết lập đáng tin cậy

### Chặn HTTPS Thất bại

- Cài đặt chứng chỉ Burp CA trong trình duyệt/hệ thống
- Điều hướng đến http://burp để tải xuống chứng chỉ
- Thêm chứng chỉ vào trusted roots
- Khởi động lại trình duyệt sau khi cài đặt

### Hiệu suất Chậm

- Giới hạn phạm vi để giảm xử lý
- Vô hiệu hóa các tiện ích mở rộng không cần thiết
- Tăng kích thước Java heap trong các tùy chọn khởi động
- Đóng các tab và tính năng Burp không sử dụng

### Yêu cầu Không Bị Chặn

- Xác minh "Intercept on" đã được bật
- Kiểm tra các quy tắc chặn không lọc mục tiêu
- Đảm bảo trình duyệt đang sử dụng Burp proxy
- Xác minh mục tiêu không sử dụng giao thức không được hỗ trợ
