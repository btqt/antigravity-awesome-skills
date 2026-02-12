---
name: code-review-checklist
description: "Danh sách kiểm tra toàn diện để thực hiện đánh giá mã kỹ lưỡng bao gồm chức năng, bảo mật, hiệu suất và khả năng bảo trì"
---

# Danh sách Kiểm tra Đánh giá Mã

## Tổng quan

Cung cấp một danh sách kiểm tra có hệ thống để thực hiện đánh giá mã kỹ lưỡng. Kỹ năng này giúp người đánh giá đảm bảo chất lượng mã, bắt lỗi, xác định các vấn đề bảo mật và duy trì tính nhất quán trên toàn bộ cơ sở mã.

## Khi nào Sử dụng Kỹ năng Này

- Sử dụng khi đánh giá các yêu cầu kéo (pull requests)
- Sử dụng khi thực hiện kiểm toán mã
- Sử dụng khi thiết lập các tiêu chuẩn đánh giá mã cho một nhóm
- Sử dụng khi đào tạo các nhà phát triển mới về thực tiễn đánh giá mã
- Sử dụng khi bạn muốn đảm bảo không bỏ sót điều gì trong các đánh giá
- Sử dụng khi tạo tài liệu đánh giá mã

## Cách Thức Hoạt Động

### Bước 1: Hiểu Bối cảnh

Trước khi đánh giá mã, tôi sẽ giúp bạn hiểu:

- Mã này giải quyết vấn đề gì?
- Các yêu cầu là gì?
- Những tệp nào đã được thay đổi và tại sao?
- Có các vấn đề hoặc vé (tickets) liên quan không?
- Chiến lược kiểm thử là gì?

### Bước 2: Đánh giá Chức năng

Kiểm tra xem mã có hoạt động chính xác không:

- Nó có giải quyết vấn đề đã nêu không?
- Các trường hợp biên có được xử lý không?
- Xử lý lỗi có phù hợp không?
- Có bất kỳ lỗi logic nào không?
- Nó có khớp với các yêu cầu không?

### Bước 3: Đánh giá Chất lượng Mã

Đánh giá khả năng bảo trì mã:

- Mã có dễ đọc và rõ ràng không?
- Tên có mô tả không?
- Nó có được cấu trúc đúng cách không?
- Các hàm/phương thức có tập trung không?
- Có sự phức tạp không cần thiết không?

### Bước 4: Đánh giá Bảo mật

Kiểm tra các vấn đề bảo mật:

- Đầu vào có được xác thực không?
- Dữ liệu nhạy cảm có được bảo vệ không?
- Có rủi ro SQL injection không?
- Xác thực/ủy quyền có chính xác không?
- Các phụ thuộc có an toàn không?

### Bước 5: Đánh giá Hiệu suất

Tìm kiếm các vấn đề hiệu suất:

- Có các vòng lặp không cần thiết không?
- Truy cập cơ sở dữ liệu có được tối ưu hóa không?
- Có rò rỉ bộ nhớ không?
- Bộ nhớ đệm có được sử dụng phù hợp không?
- Có vấn đề truy vấn N+1 không?

### Bước 6: Đánh giá Kiểm thử

Xác minh bao phủ kiểm thử:

- Có các bài kiểm tra cho mã mới không?
- Các bài kiểm tra có bao phủ các trường hợp biên không?
- Các bài kiểm tra có ý nghĩa không?
- Tất cả các bài kiểm tra có vượt qua không?
- Bao phủ kiểm thử có đầy đủ không?

## Ví dụ

### Ví dụ 1: Danh sách Kiểm tra Đánh giá Chức năng

```markdown
## Đánh giá Chức năng

### Yêu cầu

- [ ] Mã giải quyết vấn đề đã nêu
- [ ] Tất cả các tiêu chí chấp nhận được đáp ứng
- [ ] Các trường hợp biên được xử lý
- [ ] Các trường hợp lỗi được xử lý
- [ ] Đầu vào người dùng được xác thực

### Logic

- [ ] Không có lỗi logic hoặc lỗi (bug)
- [ ] Các điều kiện là chính xác (không có lỗi lệch một - off-by-one errors)
- [ ] Các vòng lặp kết thúc chính xác
- [ ] Đệ quy có các trường hợp cơ sở phù hợp
- [ ] Quản lý trạng thái là chính xác

### Xử lý Lỗi

- [ ] Các lỗi được bắt phù hợp
- [ ] Thông báo lỗi rõ ràng và hữu ích
- [ ] Các lỗi không lộ thông tin nhạy cảm
- [ ] Các hoạt động thất bại được quay lại (rolled back)
- [ ] Ghi nhật ký là phù hợp

### Các Vấn đề Ví dụ để Bắt:

**❌ Tệ - Thiếu xác thực:**
\`\`\`javascript
function createUser(email, password) {
// Không xác thực!
return db.users.create({ email, password });
}
\`\`\`

**✅ Tốt - Xác thực đúng cách:**
\`\`\`javascript
function createUser(email, password) {
if (!email || !isValidEmail(email)) {
throw new Error('Invalid email address');
}
if (!password || password.length < 8) {
throw new Error('Password must be at least 8 characters');
}
return db.users.create({ email, password });
}
\`\`\`
```

### Ví dụ 2: Danh sách Kiểm tra Đánh giá Bảo mật

```markdown
## Đánh giá Bảo mật

### Xác thực Đầu vào

- [ ] Tất cả đầu vào người dùng được xác thực
- [ ] SQL injection được ngăn chặn (sử dụng truy vấn tham số hóa)
- [ ] XSS được ngăn chặn (escape đầu ra)
- [ ] Bảo vệ CSRF được áp dụng
- [ ] Tải lên tệp được xác thực (loại, kích thước, nội dung)

### Xác thực & Ủy quyền

- [ ] Xác thực được yêu cầu ở nơi cần thiết
- [ ] Các kiểm tra ủy quyền có hiện diện
- [ ] Mật khẩu được băm (không bao giờ lưu văn bản thuần túy)
- [ ] Các phiên được quản lý an toàn
- [ ] Mã thông báo hết hạn phù hợp

### Bảo vệ Dữ liệu

- [ ] Dữ liệu nhạy cảm được mã hóa
- [ ] Khóa API không được mã hóa cứng
- [ ] Biến môi trường được sử dụng cho các bí mật
- [ ] Dữ liệu cá nhân tuân theo các quy định về quyền riêng tư
- [ ] Thông tin xác thực cơ sở dữ liệu được bảo mật

### Phụ thuộc

- [ ] Không có phụ thuộc dễ bị tổn thương đã biết
- [ ] Các phụ thuộc được cập nhật
- [ ] Các phụ thuộc không cần thiết được loại bỏ
- [ ] Các phiên bản phụ thuộc được ghim

### Các Vấn đề Ví dụ để Bắt:

**❌ Tệ - Rủi ro SQL injection:**
\`\`\`javascript
const query = \`SELECT \* FROM users WHERE email = '\${email}'\`;
db.query(query);
\`\`\`

**✅ Tốt - Truy vấn tham số hóa:**
\`\`\`javascript
const query = 'SELECT \* FROM users WHERE email = $1';
db.query(query, [email]);
\`\`\`

**❌ Tệ - Bí mật được mã hóa cứng:**
\`\`\`javascript
const API_KEY = 'sk_live_abc123xyz';
\`\`\`

**✅ Tốt - Biến môi trường:**
\`\`\`javascript
const API_KEY = process.env.API_KEY;
if (!API_KEY) {
throw new Error('API_KEY environment variable is required');
}
\`\`\`
```

### Ví dụ 3: Danh sách Kiểm tra Đánh giá Chất lượng Mã

```markdown
## Đánh giá Chất lượng Mã

### Khả năng Đọc

- [ ] Mã dễ hiểu
- [ ] Tên biến mang tính mô tả
- [ ] Tên hàm giải thích những gì chúng làm
- [ ] Logic phức tạp có bình luận
- [ ] Các số ma thuật được thay thế bằng hằng số

### Cấu trúc

- [ ] Các hàm nhỏ và tập trung
- [ ] Mã tuân theo nguyên tắc DRY (Đừng Lặp lại Chính mình)
- [ ] Phân tách các mối quan tâm đúng cách
- [ ] Phong cách mã nhất quán
- [ ] Không có mã chết hoặc mã bị bình luận

### Khả năng Bảo trì

- [ ] Mã là mô-đun và có thể tái sử dụng
- [ ] Phụ thuộc là tối thiểu
- [ ] Các thay đổi tương thích ngược
- [ ] Các thay đổi phá vỡ được ghi lại
- [ ] Nợ kỹ thuật được ghi chú

### Các Vấn đề Ví dụ để Bắt:

**❌ Tệ - Đặt tên không rõ ràng:**
\`\`\`javascript
function calc(a, b, c) {
return a \* b + c;
}
\`\`\`

**✅ Tốt - Đặt tên mô tả:**
\`\`\`javascript
function calculateTotalPrice(quantity, unitPrice, tax) {
return quantity \* unitPrice + tax;
}
\`\`\`

**❌ Tệ - Hàm làm quá nhiều việc:**
\`\`\`javascript
function processOrder(order) {
// Validate order
if (!order.items) throw new Error('No items');

// Calculate total
let total = 0;
for (let item of order.items) {
total += item.price \* item.quantity;
}

// Apply discount
if (order.coupon) {
total \*= 0.9;
}

// Process payment
const payment = stripe.charge(total);

// Send email
sendEmail(order.email, 'Order confirmed');

// Update inventory
updateInventory(order.items);

return { orderId: order.id, total };
}
\`\`\`

**✅ Tốt - Phân tách các mối quan tâm:**
\`\`\`javascript
function processOrder(order) {
validateOrder(order);
const total = calculateOrderTotal(order);
const payment = processPayment(total);
sendOrderConfirmation(order.email);
updateInventory(order.items);

return { orderId: order.id, total };
}
\`\`\`
```

## Thực tiễn Tốt nhất

### ✅ Nên Làm

- **Đánh giá Thay đổi Nhỏ** - Các PR nhỏ hơn dễ đánh giá kỹ lưỡng hơn
- **Kiểm tra Tests Trước** - Xác minh tests vượt qua và bao phủ mã mới
- **Chạy Mã** - Kiểm thử nó cục bộ khi có thể
- **Đặt Câu hỏi** - Đừng giả định, hãy hỏi để làm rõ
- **Mang tính Xây dựng** - Đề xuất cải tiến, đừng chỉ chỉ trích
- **Tập trung vào Các vấn đề Quan trọng** - Đừng soi mói các vấn đề phong cách nhỏ
- **Sử dụng Công cụ Tự động** - Linters, formatters, máy quét bảo mật
- **Đánh giá Tài liệu** - Kiểm tra xem tài liệu có được cập nhật không
- **Cân nhắc Hiệu suất** - Nghĩ về quy mô và hiệu quả
- **Kiểm tra Hồi quy** - Đảm bảo chức năng hiện có vẫn hoạt động

### ❌ Không Nên Làm

- **Đừng Phê duyệt Mà Không Đọc** - Thực sự đọc mã
- **Đừng Mơ hồ** - Cung cấp phản hồi cụ thể với các ví dụ
- **Đừng Bỏ qua Bảo mật** - Các vấn đề bảo mật là rất quan trọng
- **Đừng Bỏ qua Tests** - Mã không được kiểm thử sẽ gây ra vấn đề
- **Đừng Thô lỗ** - Hãy tôn trọng và chuyên nghiệp
- **Đừng Đóng dấu Hờ hững** - Mọi đánh giá nên thêm giá trị
- **Đừng Đánh giá Khi Mệt** - Bạn sẽ bỏ lỡ các vấn đề quan trọng
- **Đừng Quên Bối cảnh** - Hiểu bức tranh lớn hơn

## Danh sách Kiểm tra Đánh giá Hoàn chỉnh

### Trước Đánh giá

- [ ] Đọc mô tả PR và các vấn đề được liên kết
- [ ] Hiểu vấn đề nào đang được giải quyết
- [ ] Kiểm tra xem tests có vượt qua trong CI/CD không
- [ ] Kéo nhánh và chạy nó cục bộ

### Chức năng

- [ ] Mã giải quyết vấn đề đã nêu
- [ ] Các trường hợp biên được xử lý
- [ ] Xử lý lỗi là phù hợp
- [ ] Đầu vào người dùng được xác thực
- [ ] Không có lỗi logic

### Bảo mật

- [ ] Không có lỗ hổng SQL injection
- [ ] Không có lỗ hổng XSS
- [ ] Xác thực/ủy quyền là chính xác
- [ ] Dữ liệu nhạy cảm được bảo vệ
- [ ] Không có bí mật được mã hóa cứng

### Hiệu suất

- [ ] Không có truy vấn cơ sở dữ liệu không cần thiết
- [ ] Không có vấn đề truy vấn N+1
- [ ] Các thuật toán hiệu quả được sử dụng
- [ ] Không có rò rỉ bộ nhớ
- [ ] Bộ nhớ đệm được sử dụng phù hợp

### Chất lượng Mã

- [ ] Mã dễ đọc và rõ ràng
- [ ] Tên mang tính mô tả
- [ ] Các hàm tập trung và nhỏ
- [ ] Không có mã trùng lặp
- [ ] Tuân theo các quy ước dự án

### Tests

- [ ] Mã mới có tests
- [ ] Tests bao phủ các trường hợp biên
- [ ] Tests có ý nghĩa
- [ ] Tất cả tests vượt qua
- [ ] Bao phủ test là đầy đủ

### Tài liệu

- [ ] Bình luận mã giải thích tại sao, không phải cái gì
- [ ] Tài liệu API được cập nhật
- [ ] README được cập nhật nếu cần
- [ ] Các thay đổi phá vỡ được ghi lại
- [ ] Hướng dẫn di chuyển được cung cấp nếu cần

### Git

- [ ] Thông báo cam kết rõ ràng
- [ ] Không có xung đột hợp nhất
- [ ] Nhánh được cập nhật với main
- [ ] Không có tệp không cần thiết được cam kết
- [ ] .gitignore được cấu hình đúng cách

## Các Cạm bẫy Phổ biến

### Vấn đề: Thiếu Trường hợp Biên

**Triệu chứng:** Mã hoạt động cho đường dẫn hạnh phúc nhưng thất bại trên các trường hợp biên
**Giải pháp:** Đặt câu hỏi "Chuyện gì xảy ra nếu...?"

- Chuyện gì xảy ra nếu đầu vào là null?
- Chuyện gì xảy ra nếu mảng trống?
- Chuyện gì xảy ra nếu người dùng không được xác thực?
- Chuyện gì xảy ra nếu yêu cầu mạng thất bại?

### Vấn đề: Lỗ hổng Bảo mật

**Triệu chứng:** Mã để lộ rủi ro bảo mật
**Giải pháp:** Sử dụng danh sách kiểm tra bảo mật

- Chạy máy quét bảo mật (npm audit, Snyk)
- Kiểm tra OWASP Top 10
- Xác thực tất cả đầu vào
- Sử dụng truy vấn tham số hóa
- Không bao giờ tin tưởng đầu vào người dùng

### Vấn đề: Bao phủ Test Kém

**Triệu chứng:** Mã mới không có tests hoặc tests không đầy đủ
**Giải pháp:** Yêu cầu tests cho tất cả mã mới

- Unit tests cho các hàm
- Integration tests cho các tính năng
- Tests trường hợp biên
- Tests trường hợp lỗi

### Vấn đề: Mã Không rõ ràng

**Triệu chứng:** Người đánh giá không thể hiểu mã làm gì
**Giải pháp:** Yêu cầu cải tiến

- Tên biến tốt hơn
- Bình luận giải thích
- Các hàm nhỏ hơn
- Cấu trúc rõ ràng

## Mẫu Bình luận Đánh giá

### Yêu cầu Thay đổi

```markdown
**Vấn đề:** [Mô tả vấn đề]

**Mã hiện tại:**
\`\`\`javascript
// Hiển thị mã có vấn đề
\`\`\`

**Sửa chữa đề xuất:**
\`\`\`javascript
// Hiển thị mã cải tiến
\`\`\`

**Tại sao:** [Giải thích tại sao điều này tốt hơn]
```

### Đặt Câu hỏi

```markdown
**Câu hỏi:** [Câu hỏi của bạn]

**Bối cảnh:** [Tại sao bạn hỏi]

**Gợi ý:** [Nếu bạn có]
```

### Khen ngợi Mã Tốt

```markdown
**Tuyệt vời!** [Điều bạn thích]

Điều này thật tuyệt vì [giải thích tại sao]
```

## Các Kỹ năng Liên quan

- `@requesting-code-review` - Chuẩn bị mã cho đánh giá
- `@receiving-code-review` - Xử lý phản hồi đánh giá
- `@systematic-debugging` - Gỡ lỗi các vấn đề tìm thấy trong đánh giá
- `@test-driven-development` - Đảm bảo mã có tests

## Tài nguyên Bổ sung

- [Google Code Review Guidelines](https://google.github.io/eng-practices/review/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Code Review Best Practices](https://github.com/thoughtbot/guides/tree/main/code-review)
- [How to Review Code](https://www.kevinlondon.com/2015/05/05/code-review-best-practices.html)

---

**Mẹo Chuyên nghiệp:** Sử dụng mẫu danh sách kiểm tra cho mọi đánh giá để đảm bảo tính nhất quán và kỹ lưỡng. Tùy chỉnh nó cho nhu cầu cụ thể của nhóm bạn!
