---
name: Broken Authentication Testing
description: Kỹ năng này nên được sử dụng khi người dùng yêu cầu "kiểm tra các lỗ hổng xác thực bị hỏng", "đánh giá bảo mật quản lý phiên", "thực hiện kiểm tra credential stuffing", "đánh giá chính sách mật khẩu", "kiểm tra session fixation", hoặc "xác định lỗi bỏ qua xác thực". Nó cung cấp các kỹ thuật toàn diện để xác định các điểm yếu về xác thực và quản lý phiên trong các ứng dụng web.
metadata:
  author: zebbern
  version: "1.1"
---

# Kiểm thử Xác thực Bị hỏng (Broken Authentication Testing)

## Mục đích

Xác định và khai thác các lỗ hổng xác thực và quản lý phiên trong các ứng dụng web. Xác thực bị hỏng liên tục xếp hạng trong Top 10 OWASP và có thể dẫn đến chiếm đoạt tài khoản, trộm cắp danh tính, và truy cập trái phép vào các hệ thống nhạy cảm. Kỹ năng này bao gồm các phương pháp kiểm thử cho chính sách mật khẩu, xử lý phiên, xác thực đa yếu tố (MFA), và quản lý thông tin xác thực.

## Điều kiện tiên quyết

### Kiến thức Yêu cầu

- Giao thức HTTP và cơ chế phiên
- Các loại xác thực (SFA, 2FA, MFA)
- Xử lý Cookie và token
- Các framework xác thực phổ biến

### Công cụ Yêu cầu

- Burp Suite Professional hoặc Community
- Hydra hoặc các công cụ brute-force tương tự
- Wordlists tùy chỉnh để kiểm thử thông tin xác thực
- Công cụ phát triển trình duyệt (Browser developer tools)

### Truy cập Yêu cầu

- URL ứng dụng mục tiêu
- Thông tin xác thực tài khoản kiểm thử
- Ủy quyền bằng văn bản để kiểm thử

## Đầu ra và Giao nộp

1. **Báo cáo Đánh giá Xác thực** - Tài liệu hóa tất cả các lỗ hổng được xác định
2. **Kết quả Kiểm thử Thông tin xác thực** - Kết quả tấn công brute-force và từ điển
3. **Phân tích Bảo mật Phiên** - Đánh giá tính ngẫu nhiên của token và thời gian chờ
4. **Khuyến nghị Khắc phục** - Hướng dẫn củng cố bảo mật

## Quy trình làm việc Cốt lõi

### Giai đoạn 1: Phân tích Cơ chế Xác thực

Hiểu kiến trúc xác thực của ứng dụng:

```
# Xác định loại xác thực
- Dựa trên mật khẩu (forms, basic auth, digest)
- Dựa trên token (JWT, OAuth, API keys)
- Dựa trên chứng chỉ (mutual TLS)
- Đa yếu tố (SMS, TOTP, hardware tokens)

# Bản đồ các endpoint xác thực
/login, /signin, /authenticate
/register, /signup
/forgot-password, /reset-password
/logout, /signout
/api/auth/*, /oauth/*
```

Thu thập và phân tích các yêu cầu xác thực:

```http
POST /login HTTP/1.1
Host: target.com
Content-Type: application/x-www-form-urlencoded

username=test&password=test123
```

### Giai đoạn 2: Kiểm thử Chính sách Mật khẩu

Đánh giá các yêu cầu và việc thực thi mật khẩu:

```bash
# Kiểm thử độ dài tối thiểu (a, ab, abcdefgh)
# Kiểm thử độ phức tạp (password, password1, Password1!)
# Kiểm thử các mật khẩu yếu phổ biến (123456, password, qwerty, admin)
# Kiểm thử tên người dùng làm mật khẩu (admin/admin, test/test)
```

Tài liệu hóa các lỗ hổng chính sách: Độ dài tối thiểu <8, không có độ phức tạp, cho phép mật khẩu phổ biến, tên người dùng làm mật khẩu.

### Giai đoạn 3: Liệt kê Thông tin xác thực (Credential Enumeration)

Kiểm thử các lỗ hổng liệt kê tên người dùng:

```bash
# So sánh phản hồi cho tên người dùng hợp lệ vs không hợp lệ
# Không hợp lệ: "Invalid username" vs Hợp lệ: "Invalid password"
# Kiểm tra sự khác biệt về thời gian, mã phản hồi, thông báo đăng ký
```

```
# Đặt lại mật khẩu
"Email sent if account exists" (an toàn)
"No account with that email" (rò rỉ thông tin)

# Phản hồi API
{"error": "user_not_found"}
{"error": "invalid_password"}
```

### Giai đoạn 4: Kiểm thử Brute Force

Kiểm thử khóa tài khoản và giới hạn tốc độ:

```bash
# Sử dụng Hydra cho xác thực dựa trên form
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
  target.com http-post-form \
  "/login:username=^USER^&password=^PASS^:Invalid credentials"

# Sử dụng Burp Intruder
1. Thu thập yêu cầu đăng nhập
2. Gửi đến Intruder
3. Đặt vị trí payload trên trường mật khẩu
4. Tải wordlist
5. Bắt đầu tấn công
6. Phân tích độ dài/mã phản hồi
```

Kiểm tra các biện pháp bảo vệ:

```bash
# Khóa tài khoản
- Sau bao nhiêu lần thử?
- Thời gian khóa?
- Thông báo khóa?

# Giới hạn tốc độ (Rate limiting)
- Giới hạn yêu cầu mỗi phút?
- Dựa trên IP hay tài khoản?
- Bỏ qua qua headers (X-Forwarded-For)?

# CAPTCHA
- Sau các lần thử thất bại?
- Có thể bỏ qua dễ dàng không?
```

### Giai đoạn 5: Credential Stuffing

Kiểm thử với các thông tin xác thực bị rò rỉ đã biết:

```bash
# Credential stuffing khác với brute force
# Sử dụng các cặp email:password đã biết từ các vụ vi phạm dữ liệu

# Sử dụng Burp Intruder với tấn công Pitchfork
1. Đặt tên người dùng và mật khẩu làm vị trí
2. Tải danh sách email làm payload 1
3. Tải danh sách mật khẩu làm payload 2 (các cặp khớp)
4. Phân tích các lần đăng nhập thành công

# Tránh phát hiện
- Tốc độ yêu cầu chậm
- Xoay vòng IP nguồn
- Ngẫu nhiên hóa user agents
- Thêm độ trễ giữa các lần thử
```

### Giai đoạn 6: Kiểm thử Quản lý Phiên

Phân tích bảo mật token phiên:

```bash
# Thu thập cookie phiên
Cookie: SESSIONID=abc123def456

# Kiểm thử đặc điểm token
1. Entropy - Nó có đủ ngẫu nhiên không?
2. Độ dài - Độ dài đủ (128+ bits)?
3. Khả năng dự đoán - Các mẫu tuần tự?
4. Cờ bảo mật - HttpOnly, Secure, SameSite?
```

Phân tích token phiên:

```python
#!/usr/bin/env python3
import requests
import hashlib

# Thu thập nhiều token phiên
tokens = []
for i in range(100):
    response = requests.get("https://target.com/login")
    token = response.cookies.get("SESSIONID")
    tokens.append(token)

# Phân tích các mẫu
# Kiểm tra các mức tăng tuần tự
# Tính toán entropy
# Tìm kiếm các thành phần dấu thời gian
```

### Giai đoạn 7: Kiểm thử Session Fixation

Kiểm thử xem phiên có được tạo lại sau khi xác thực không:

```bash
# Bước 1: Lấy phiên trước khi đăng nhập
GET /login HTTP/1.1
Response: Set-Cookie: SESSIONID=abc123

# Bước 2: Đăng nhập với cùng phiên đó
POST /login HTTP/1.1
Cookie: SESSIONID=abc123
username=valid&password=valid

# Bước 3: Kiểm tra xem phiên có thay đổi không
# LỖ HỔNG nếu SESSIONID vẫn là abc123
# AN TOÀN nếu phiên mới được gán sau khi đăng nhập
```

Kịch bản tấn công:

```bash
# Quy trình của kẻ tấn công:
1. Kẻ tấn công truy cập trang web, lấy phiên: SESSIONID=attacker_session
2. Kẻ tấn công gửi liên kết cho nạn nhân với phiên cố định:
   https://target.com/login?SESSIONID=attacker_session
3. Nạn nhân đăng nhập với phiên của kẻ tấn công
4. Kẻ tấn công hiện có phiên đã xác thực
```

### Giai đoạn 8: Kiểm thử Hết hạn Phiên (Session Timeout)

Xác minh các chính sách hết hạn phiên:

```bash
# Kiểm thử timeout nhàn rỗi (idle timeout)
1. Đăng nhập và ghi lại cookie phiên
2. Chờ không hoạt động (15, 30, 60 phút)
3. Cố gắng sử dụng phiên
4. Kiểm tra xem phiên còn hợp lệ không

# Kiểm thử timeout tuyệt đối (absolute timeout)
1. Đăng nhập và liên tục sử dụng phiên
2. Kiểm tra xem có bị buộc đăng xuất sau thời gian thiết lập không (8 giờ, 24 giờ)

# Kiểm thử chức năng đăng xuất
1. Đăng nhập và ghi lại phiên
2. Nhấp đăng xuất
3. Cố gắng sử dụng lại cookie phiên cũ
4. Phiên nên bị vô hiệu hóa phía máy chủ
```

### Giai đoạn 9: Kiểm thử Xác thực Đa yếu tố (MFA)

Đánh giá bảo mật triển khai MFA:

```bash
# OTP brute force
- 4 chữ số OTP = 10,000 tổ hợp
- 6 chữ số OTP = 1,000,000 tổ hợp
- Kiểm thử giới hạn tốc độ trên endpoint OTP

# Kỹ thuật bỏ qua OTP
- Bỏ qua bước MFA bằng truy cập URL trực tiếp
- Sửa đổi phản hồi để chỉ ra MFA đã qua
- Gửi OTP null/rỗng
- Sử dụng lại OTP hợp lệ trước đó

# Tấn công hạ cấp phiên bản API (ví dụ crAPI)
# Nếu /api/v3/check-otp có giới hạn tốc độ, thử các phiên bản cũ hơn:
POST /api/v2/check-otp
{"otp": "1234"}
# Các phiên bản API cũ hơn có thể thiếu kiểm soát bảo mật

# Sử dụng Burp để kiểm thử OTP
1. Thu thập yêu cầu xác minh OTP
2. Gửi đến Intruder
3. Đặt trường OTP làm vị trí payload
4. Sử dụng payload số (0000-9999)
5. Kiểm tra bỏ qua thành công
```

Kiểm thử đăng ký MFA:

```bash
# Đăng ký bắt buộc
- Có thể bỏ qua MFA trong quá trình thiết lập không?
- Có thể truy cập mã sao lưu mà không cần xác minh không?

# Quy trình khôi phục
- Có thể tắt MFA chỉ qua email không?
- Tiềm năng social engineering?
```

### Giai đoạn 10: Kiểm thử Đặt lại Mật khẩu

Phân tích bảo mật đặt lại mật khẩu:

```bash
# Bảo mật token
1. Yêu cầu đặt lại mật khẩu
2. Thu thập liên kết đặt lại
3. Phân tích token:
   - Độ dài và tính ngẫu nhiên
   - Thời gian hết hạn
   - Thực thi sử dụng một lần
   - Ràng buộc tài khoản

# Thao tác token
https://target.com/reset?token=abc123&user=victim
# Thử thay đổi tham số người dùng trong khi sử dụng token hợp lệ

# Host header injection
POST /forgot-password HTTP/1.1
Host: attacker.com
email=victim@email.com
# Email đặt lại có thể chứa tên miền của kẻ tấn công
```

## Tham khảo Nhanh

### Các Loại Lỗ hổng Phổ biến

| Lỗ hổng                        | Rủi ro       | Phương pháp Kiểm thử                  |
| ------------------------------ | ------------ | ------------------------------------- |
| Mật khẩu yếu                   | Cao          | Kiểm thử chính sách, tấn công từ điển |
| Không khóa tài khoản           | Cao          | Kiểm thử brute force                  |
| Liệt kê tên người dùng         | Trung bình   | Phân tích phản hồi khác biệt          |
| Session fixation               | Cao          | So sánh phiên trước/sau đăng nhập     |
| Token phiên yếu                | Cao          | Phân tích entropy                     |
| Không timeout phiên            | Trung bình   | Kiểm thử phiên thời gian dài          |
| Đặt lại mật khẩu không an toàn | Cao          | Phân tích token, bỏ qua quy trình     |
| Bỏ qua MFA                     | Nghiêm trọng | Truy cập trực tiếp, thao tác phản hồi |

### Payload Kiểm thử Thông tin xác thực

```bash
# Thông tin xác thực mặc định
admin:admin
admin:password
admin:123456
root:root
test:test
user:user

# Mật khẩu phổ biến
123456
password
12345678
qwerty
abc123
password1
admin123

# Cơ sở dữ liệu thông tin xác thực bị rò rỉ
- Have I Been Pwned dataset
- SecLists passwords
- Danh sách mục tiêu tùy chỉnh
```

### Cờ Cookie Phiên

| Cờ       | Mục đích              | Lỗ hổng nếu Thiếu            |
| -------- | --------------------- | ---------------------------- |
| HttpOnly | Ngăn chặn truy cập JS | XSS có thể đánh cắp phiên    |
| Secure   | Chỉ HTTPS             | Gửi qua HTTP                 |
| SameSite | Bảo vệ CSRF           | Yêu cầu chéo trang được phép |
| Path     | Phạm vi URL           | Phơi bày rộng hơn            |
| Domain   | Phạm vi tên miền      | Truy cập tên miền phụ        |
| Expires  | Thời gian sống        | Các phiên bền vững           |

### Headers Bỏ qua Giới hạn Tốc độ

```http
X-Forwarded-For: 127.0.0.1
X-Real-IP: 127.0.0.1
X-Originating-IP: 127.0.0.1
X-Client-IP: 127.0.0.1
X-Remote-IP: 127.0.0.1
True-Client-IP: 127.0.0.1
```

## Ràng buộc và Hạn chế

### Yêu cầu Pháp lý

- Chỉ kiểm thử với ủy quyền bằng văn bản rõ ràng
- Tránh kiểm thử với thông tin xác thực bị rò rỉ thực tế
- Không truy cập tài khoản người dùng thực tế
- Tài liệu hóa tất cả các hoạt động kiểm thử

### Hạn chế Kỹ thuật

- CAPTCHA có thể ngăn chặn kiểm thử tự động
- Giới hạn tốc độ ảnh hưởng đến thời gian brute force
- MFA làm tăng đáng kể độ khó tấn công
- Một số lỗ hổng yêu cầu tương tác của nạn nhân

### Cân nhắc Phạm vi

- Tài khoản kiểm thử có thể hoạt động khác với sản phẩm
- Một số tính năng có thể bị vô hiệu hóa trong môi trường kiểm thử
- Xác thực bên thứ ba có thể nằm ngoài phạm vi
- Kiểm thử sản phẩm yêu cầu thận trọng hơn

## Ví dụ

### Ví dụ 1: Bỏ qua Khóa Tài khoản

**Kịch bản:** Kiểm thử xem khóa tài khoản có thể bị bỏ qua không

```bash
# Bước 1: Xác định ngưỡng khóa
# Thử 5 mật khẩu sai cho tài khoản admin
# Kết quả: "Account locked for 30 minutes"

# Bước 2: Kiểm thử bỏ qua qua xoay vòng IP
# Sử dụng header X-Forwarded-For
POST /login HTTP/1.1
X-Forwarded-For: 192.168.1.1
username=admin&password=attempt1

# Tăng IP cho mỗi lần thử
X-Forwarded-For: 192.168.1.2
# Tiếp tục cho đến khi thành công hoặc xác nhận bị chặn

# Bước 3: Kiểm thử bỏ qua qua thao tác chữ hoa thường
username=Admin (vs admin)
username=ADMIN
# Một số hệ thống coi đây là các tài khoản khác nhau
```

### Ví dụ 2: Tấn công Token JWT

**Kịch bản:** Khai thác triển khai JWT yếu

```bash
# Bước 1: Thu thập token JWT
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoidGVzdCJ9.signature

# Bước 2: Giải mã và phân tích
# Header: {"alg":"HS256","typ":"JWT"}
# Payload: {"user":"test","role":"user"}

# Bước 3: Thử tấn công thuật toán "none"
# Thay đổi header thành: {"alg":"none","typ":"JWT"}
# Xóa chữ ký
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ.

# Bước 4: Gửi token đã sửa đổi
Authorization: Bearer eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiYWRtaW4ifQ.
```

### Ví dụ 3: Khai thác Token Đặt lại Mật khẩu

**Kịch bản:** Kiểm thử chức năng đặt lại mật khẩu

```bash
# Bước 1: Yêu cầu đặt lại cho tài khoản kiểm thử
POST /forgot-password
email=test@example.com

# Bước 2: Thu thập liên kết đặt lại
https://target.com/reset?token=a1b2c3d4e5f6

# Bước 3: Kiểm thử tính chất token
# Sử dụng lại: Thử sử dụng cùng một token hai lần
# Hết hạn: Chờ 24+ giờ và thử lại
# Sửa đổi: Thay đổi các ký tự trong token

# Bước 4: Kiểm thử thao tác tham số người dùng
https://target.com/reset?token=a1b2c3d4e5f6&email=admin@example.com
# Kiểm tra xem mật khẩu của admin có thể được đặt lại bằng token của người dùng kiểm thử không
```

## Khắc phục sự cố

| Vấn đề                            | Giải pháp                                                                                          |
| --------------------------------- | -------------------------------------------------------------------------------------------------- |
| Brute force quá chậm              | Xác định phạm vi giới hạn tốc độ; xoay vòng IP; thêm độ trễ; sử dụng wordlists mục tiêu            |
| Phân tích phiên không kết luận    | Thu thập 1000+ token; sử dụng công cụ thống kê; kiểm tra dấu thời gian; so sánh các tài khoản      |
| MFA không thể bị bỏ qua           | Tài liệu hóa là an toàn; kiểm thử cơ chế sao lưu/khôi phục; kiểm tra MFA fatigue; xác minh đăng ký |
| Khóa tài khoản ngăn chặn kiểm thử | Yêu cầu nhiều tài khoản kiểm thử; kiểm thử ngưỡng trước; sử dụng thời gian chậm hơn                |
