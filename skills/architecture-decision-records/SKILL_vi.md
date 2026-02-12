---
name: architecture-decision-records
description: Viết và duy trì Hồ sơ Quyết định Kiến trúc (ADRs) tuân theo các thực hành tốt nhất cho tài liệu quyết định kỹ thuật. Sử dụng khi ghi lại các quyết định kỹ thuật quan trọng, xem xét các lựa chọn kiến trúc trong quá khứ, hoặc thiết lập quy trình ra quyết định.
---

# Hồ sơ Quyết định Kiến trúc (ADR)

Các mẫu toàn diện để tạo, duy trì và quản lý Hồ sơ Quyết định Kiến trúc (ADRs) nắm bắt ngữ cảnh và cơ sở lý luận đằng sau các quyết định kỹ thuật quan trọng.

## Sử dụng kỹ năng này khi

- Đưa ra các quyết định kiến trúc quan trọng
- Ghi lại các lựa chọn công nghệ
- Ghi lại các đánh đổi thiết kế
- Giới thiệu thành viên mới trong nhóm
- Xem xét các quyết định lịch sử
- Thiết lập quy trình ra quyết định

## Không sử dụng kỹ năng này khi

- Bạn chỉ cần ghi lại các chi tiết triển khai nhỏ
- Thay đổi là một bản vá nhỏ hoặc bảo trì định kỳ
- Không có quyết định kiến trúc nào để nắm bắt

## Hướng dẫn

1. Nắm bắt ngữ cảnh quyết định, các ràng buộc và các yếu tố thúc đẩy (drivers).
2. Ghi lại các tùy chọn được xem xét với các đánh đổi.
3. Ghi lại quyết định, cơ sở lý luận và hậu quả.
4. Liên kết các ADR liên quan và cập nhật trạng thái theo thời gian.

## Các Khái niệm Cốt lõi

### 1. ADR là gì?

Một Hồ sơ Quyết định Kiến trúc nắm bắt:

- **Ngữ cảnh**: Tại sao chúng ta cần đưa ra quyết định
- **Quyết định**: Chúng ta đã quyết định gì
- **Hậu quả**: Điều gì xảy ra sau đó

### 2. Khi nào Viết một ADR

| Viết ADR                         | Bỏ qua ADR             |
| -------------------------------- | ---------------------- |
| Áp dụng framework mới            | Nâng cấp phiên bản nhỏ |
| Lựa chọn công nghệ cơ sở dữ liệu | Sửa lỗi                |
| Mẫu thiết kế API                 | Chi tiết triển khai    |
| Kiến trúc bảo mật                | Bảo trì định kỳ        |
| Mẫu tích hợp                     | Thay đổi cấu hình      |

### 3. Vòng đời ADR

```
Đề xuất → Được chấp nhận → Bị phản đối → Bị thay thế
              ↓
           Bị từ chối
```

## Các Mẫu

### Mẫu 1: ADR Tiêu chuẩn (Định dạng MADR)

```markdown
# ADR-0001: Sử dụng PostgreSQL làm Cơ sở dữ liệu Chính

## Trạng thái

Được chấp nhận

## Ngữ cảnh

Chúng ta cần chọn một cơ sở dữ liệu chính cho nền tảng thương mại điện tử mới của mình. Hệ thống sẽ xử lý:

- ~10,000 người dùng đồng thời
- Danh mục sản phẩm phức tạp với các danh mục phân cấp
- Xử lý giao dịch cho đơn hàng và thanh toán
- Tìm kiếm toàn văn (full-text search) cho sản phẩm
- Truy vấn không gian địa lý cho định vị cửa hàng

Nhóm có kinh nghiệm với MySQL, PostgreSQL, và MongoDB. Chúng ta cần tuân thủ ACID cho các giao dịch tài chính.

## Các Yếu tố Thúc đẩy Quyết định

- **Phải có tuân thủ ACID** cho xử lý thanh toán
- **Phải hỗ trợ truy vấn phức tạp** cho báo cáo
- **Nên hỗ trợ tìm kiếm toàn văn** để giảm độ phức tạp cơ sở hạ tầng
- **Nên hỗ trợ tốt JSON** cho các thuộc tính sản phẩm linh hoạt
- **Sự quen thuộc của nhóm** giảm thời gian làm quen

## Các Tùy chọn Được xem xét

### Tùy chọn 1: PostgreSQL

- **Ưu điểm**: Tuân thủ ACID, hỗ trợ JSON xuất sắc (JSONB), tích hợp tìm kiếm toàn văn, PostGIS cho không gian địa lý, nhóm có kinh nghiệm
- **Nhược điểm**: Thiết lập sao chép (replication) phức tạp hơn một chút so với MySQL

### Tùy chọn 2: MySQL

- **Ưu điểm**: Rất quen thuộc với nhóm, sao chép đơn giản, cộng đồng lớn
- **Nhược điểm**: Hỗ trợ JSON yếu hơn, không tích hợp tìm kiếm toàn văn (cần Elasticsearch), không có không gian địa lý nếu không có tiện ích mở rộng

### Tùy chọn 3: MongoDB

- **Ưu điểm**: Lược đồ linh hoạt, JSON gốc, mở rộng theo chiều ngang
- **Nhược điểm**: Không có ACID cho giao dịch đa tài liệu (tại thời điểm quyết định), nhóm có kinh nghiệm hạn chế, yêu cầu kỷ luật thiết kế lược đồ

## Quyết định

Chúng ta sẽ sử dụng **PostgreSQL 15** làm cơ sở dữ liệu chính.

## Cơ sở lý luận

PostgreSQL cung cấp sự cân bằng tốt nhất về:

1. **Tuân thủ ACID** thiết yếu cho giao dịch thương mại điện tử
2. **Khả năng tích hợp sẵn** (tìm kiếm toàn văn, JSONB, PostGIS) giảm độ phức tạp cơ sở hạ tầng
3. **Sự quen thuộc của nhóm** với cơ sở dữ liệu SQL giảm đường cong học tập
4. **Hệ sinh thái trưởng thành** với công cụ và hỗ trợ cộng đồng xuất sắc

Độ phức tạp nhỏ trong sao chép được bù đắp bởi việc giảm các dịch vụ bổ sung (không cần Elasticsearch riêng biệt).

## Hậu quả

### Tích cực

- Một cơ sở dữ liệu duy nhất xử lý giao dịch, tìm kiếm và truy vấn không gian địa lý
- Giảm độ phức tạp vận hành (ít dịch vụ phải quản lý hơn)
- Đảm bảo tính nhất quán mạnh mẽ cho dữ liệu tài chính
- Nhóm có thể tận dụng chuyên môn SQL hiện có

### Tiêu cực

- Cần học các tính năng đặc thù của PostgreSQL (JSONB, cú pháp tìm kiếm toàn văn)
- Giới hạn mở rộng theo chiều dọc có thể yêu cầu read replicas sớm hơn
- Một số thành viên trong nhóm cần đào tạo đặc thù về PostgreSQL

### Rủi ro

- Tìm kiếm toàn văn có thể không mở rộng tốt như các công cụ tìm kiếm chuyên dụng
- Giảm nhẹ: Thiết kế cho khả năng thêm Elasticsearch nếu cần

## Ghi chú Triển khai

- Sử dụng JSONB cho các thuộc tính sản phẩm linh hoạt
- Triển khai connection pooling với PgBouncer
- Thiết lập streaming replication cho read replicas
- Sử dụng tiện ích mở rộng pg_trgm cho tìm kiếm mờ (fuzzy search)

## Các Quyết định Liên quan

- ADR-0002: Chiến lược Caching (Redis) - bổ sung cho lựa chọn cơ sở dữ liệu
- ADR-0005: Kiến trúc Tìm kiếm - có thể thay thế nếu cần Elasticsearch

## Tham khảo

- [Tài liệu PostgreSQL JSON](https://www.postgresql.org/docs/current/datatype-json.html)
- [Tìm kiếm Toàn văn PostgreSQL](https://www.postgresql.org/docs/current/textsearch.html)
- Nội bộ: Điểm chuẩn hiệu suất trong `/docs/benchmarks/database-comparison.md`
```

### Mẫu 2: ADR Nhẹ (Lightweight ADR)

```markdown
# ADR-0012: Áp dụng TypeScript cho Phát triển Frontend

**Trạng thái**: Được chấp nhận
**Ngày**: 15-01-2024
**Người quyết định**: @alice, @bob, @charlie

## Ngữ cảnh

Codebase React của chúng ta đã phát triển lên hơn 50 component với các báo cáo lỗi gia tăng liên quan đến sai lệch kiểu prop và lỗi undefined. PropTypes chỉ cung cấp kiểm tra tại runtime.

## Quyết định

Áp dụng TypeScript cho tất cả mã frontend mới. Di chuyển mã hiện có dần dần.

## Hậu quả

**Tốt**: Bắt lỗi kiểu tại thời điểm biên dịch, hỗ trợ IDE tốt hơn, mã tự tài liệu hóa (self-documenting).

**Xấu**: Đường cong học tập cho nhóm, chậm lại ban đầu, tăng độ phức tạp build.

**Giảm nhẹ**: Các buổi đào tạo TypeScript, cho phép áp dụng dần dần với `allowJs: true`.
```

### Mẫu 3: Định dạng Y-Statement

```markdown
# ADR-0015: Lựa chọn API Gateway

Trong ngữ cảnh **xây dựng kiến trúc microservices**,
đối mặt với **nhu cầu quản lý API tập trung, xác thực và giới hạn tốc độ**,
chúng tôi quyết định chọn **Kong Gateway**
và chống lại **AWS API Gateway và giải pháp Nginx tùy chỉnh**,
để đạt được **sự độc lập với nhà cung cấp, khả năng mở rộng plugin, và sự quen thuộc của nhóm với Lua**,
chấp nhận rằng **chúng tôi cần tự quản lý cơ sở hạ tầng Kong**.
```

### Mẫu 4: ADR cho Sự phản đối (Deprecation)

```markdown
# ADR-0020: Phản đối MongoDB để ủng hộ PostgreSQL

## Trạng thái

Được chấp nhận (Thay thế ADR-0003)

## Ngữ cảnh

ADR-0003 (2021) đã chọn MongoDB cho lưu trữ hồ sơ người dùng do nhu cầu linh hoạt lược đồ. Kể từ đó:

- Các giao dịch đa tài liệu của MongoDB vẫn là vấn đề cho trường hợp sử dụng của chúng ta
- Lược đồ của chúng ta đã ổn định và hiếm khi thay đổi
- Chúng ta hiện có chuyên môn PostgreSQL từ các dịch vụ khác
- Duy trì hai cơ sở dữ liệu làm tăng gánh nặng vận hành

## Quyết định

Phản đối MongoDB và di chuyển hồ sơ người dùng sang PostgreSQL.

## Kế hoạch Di chuyển

1. **Giai đoạn 1** (Tuần 1-2): Tạo lược đồ PostgreSQL, kích hoạt ghi kép (dual-write)
2. **Giai đoạn 2** (Tuần 3-4): Điền lại (Backfill) dữ liệu lịch sử, xác thực tính nhất quán
3. **Giai đoạn 3** (Tuần 5): Chuyển đọc sang PostgreSQL, giám sát
4. **Giai đoạn 4** (Tuần 6): Loại bỏ ghi MongoDB, ngừng hoạt động

## Hậu quả

### Tích cực

- Công nghệ cơ sở dữ liệu duy nhất giảm độ phức tạp vận hành
- Giao dịch ACID cho dữ liệu người dùng
- Nhóm có thể tập trung chuyên môn PostgreSQL

### Tiêu cực

- Nỗ lực di chuyển (~4 tuần)
- Rủi ro vấn đề dữ liệu trong quá trình di chuyển
- Mất một số tính linh hoạt của lược đồ

## Bài học Rút ra

Tài liệu từ kinh nghiệm ADR-0003:

- Lợi ích linh hoạt lược đồ đã bị đánh giá quá cao
- Chi phí vận hành của nhiều cơ sở dữ liệu đã bị đánh giá thấp
- Cân nhắc bảo trì dài hạn trong các quyết định công nghệ
```

### Mẫu 5: Kiểu Yêu cầu Bình luận (RFC)

```markdown
# RFC-0025: Áp dụng Event Sourcing cho Quản lý Đơn hàng

## Tóm tắt

Đề xuất áp dụng mẫu event sourcing cho tên miền quản lý đơn hàng để cải thiện khả năng kiểm toán, cho phép truy vấn thời gian, và hỗ trợ phân tích kinh doanh.

## Động lực

Thách thức hiện tại:

1. Yêu cầu kiểm toán cần lịch sử đơn hàng đầy đủ
2. Truy vấn "Trạng thái đơn hàng tại thời điểm X là gì?" là không thể
3. Nhóm phân tích cần luồng sự kiện cho bảng điều khiển thời gian thực
4. Tái tạo trạng thái đơn hàng cho hỗ trợ khách hàng là thủ công

## Thiết kế Chi tiết

### Kho Sự kiện (Event Store)
```

OrderCreated { orderId, customerId, items[], timestamp }
OrderItemAdded { orderId, item, timestamp }
OrderItemRemoved { orderId, itemId, timestamp }
PaymentReceived { orderId, amount, paymentId, timestamp }
OrderShipped { orderId, trackingNumber, timestamp }

```

### Projections

- **CurrentOrderState**: Materialized view cho truy vấn
- **OrderHistory**: Dòng thời gian đầy đủ cho kiểm toán
- **DailyOrderMetrics**: Tổng hợp phân tích

### Công nghệ

- Event Store: EventStoreDB (xây dựng theo mục đích, xử lý projections)
- Thay thế được xem xét: Kafka + dịch vụ projection tùy chỉnh

## Nhược điểm

- Đường cong học tập cho nhóm
- Tăng độ phức tạp so với CRUD
- Cần thiết kế sự kiện cẩn thận (bất biến sau khi lưu trữ)
- Tăng trưởng lưu trữ (sự kiện không bao giờ bị xóa)

## Các Thay thế

1. **Bảng kiểm toán**: Đơn giản hơn nhưng không cho phép truy vấn thời gian
2. **CDC từ DB hiện có**: Phức tạp, không thay đổi mô hình dữ liệu
3. **Lai (Hybrid)**: Event source chỉ cho thay đổi trạng thái đơn hàng

## Các Câu hỏi Chưa giải quyết

- [ ] Chiến lược phiên bản hóa lược đồ sự kiện
- [ ] Chính sách lưu giữ cho sự kiện
- [ ] Tần suất snapshot cho hiệu suất

## Kế hoạch Triển khai

1. Nguyên mẫu với loại đơn hàng đơn lẻ (2 tuần)
2. Đào tạo nhóm về event sourcing (1 tuần)
3. Triển khai đầy đủ và di chuyển (4 tuần)
4. Giám sát và tối ưu hóa (liên tục)

## Tham khảo

- [Event Sourcing bởi Martin Fowler](https://martinfowler.com/eaaDev/EventSourcing.html)
- [Tài liệu EventStoreDB](https://www.eventstore.com/docs)
```

## Quản lý ADR

### Cấu trúc Thư mục

```
docs/
├── adr/
│   ├── README.md           # Chỉ mục và hướng dẫn
│   ├── template.md         # Mẫu ADR của nhóm
│   ├── 0001-use-postgresql.md
│   ├── 0002-caching-strategy.md
│   ├── 0003-mongodb-user-profiles.md  # [BỊ PHẢN ĐỐI]
│   └── 0020-deprecate-mongodb.md      # Thay thế 0003
```

### Chỉ mục ADR (README.md)

```markdown
# Hồ sơ Quyết định Kiến trúc

Thư mục này chứa các Hồ sơ Quyết định Kiến trúc (ADRs) cho [Tên Dự án].

## Chỉ mục

| ADR                                   | Tiêu đề                                    | Trạng thái     | Ngày       |
| ------------------------------------- | ------------------------------------------ | -------------- | ---------- |
| [0001](0001-use-postgresql.md)        | Sử dụng PostgreSQL làm Cơ sở dữ liệu Chính | Được chấp nhận | 2024-01-10 |
| [0002](0002-caching-strategy.md)      | Chiến lược Caching với Redis               | Được chấp nhận | 2024-01-12 |
| [0003](0003-mongodb-user-profiles.md) | MongoDB cho Hồ sơ Người dùng               | Bị phản đối    | 2023-06-15 |
| [0020](0020-deprecate-mongodb.md)     | Phản đối MongoDB                           | Được chấp nhận | 2024-01-15 |

## Tạo một ADR Mới

1. Sao chép `template.md` thành `NNNN-title-with-dashes.md`
2. Điền vào mẫu
3. Gửi PR để xem xét
4. Cập nhật chỉ mục này sau khi phê duyệt

## Trạng thái ADR

- **Đề xuất**: Đang thảo luận
- **Được chấp nhận**: Đã quyết định, đang triển khai
- **Bị phản đối**: Không còn liên quan
- **Bị thay thế**: Được thay thế bởi ADR khác
- **Bị từ chối**: Đã xem xét nhưng không được thông qua
```

### Tự động hóa (adr-tools)

```bash
# Cài đặt adr-tools
brew install adr-tools

# Khởi tạo thư mục ADR
adr init docs/adr

# Tạo ADR mới
adr new "Use PostgreSQL as Primary Database"

# Thay thế một ADR
adr new -s 3 "Deprecate MongoDB in Favor of PostgreSQL"

# Tạo mục lục
adr generate toc > docs/adr/README.md

# Liên kết các ADR liên quan
adr link 2 "Complements" 1 "Is complemented by"
```

## Quy trình Xem xét

```markdown
## Danh sách Kiểm tra Xem xét ADR

### Trước khi Gửi

- [ ] Ngữ cảnh giải thích rõ ràng vấn đề
- [ ] Tất cả các tùy chọn khả thi đã được xem xét
- [ ] Ưu/nhược điểm cân bằng và trung thực
- [ ] Hậu quả (tích cực và tiêu cực) được ghi lại
- [ ] Các ADR liên quan được liên kết

### Trong khi Xem xét

- [ ] Ít nhất 2 kỹ sư cấp cao đã xem xét
- [ ] Các nhóm bị ảnh hưởng đã được tham vấn
- [ ] Các ý nghĩa bảo mật đã được xem xét
- [ ] Các ý nghĩa chi phí được ghi lại
- [ ] Khả năng đảo ngược được đánh giá

### Sau khi Chấp nhận

- [ ] Chỉ mục ADR được cập nhật
- [ ] Nhóm được thông báo
- [ ] Vé triển khai được tạo
- [ ] Tài liệu liên quan được cập nhật
```

## Thực hành Tốt nhất

### Nên làm

- **Viết ADR sớm** - Trước khi bắt đầu triển khai
- **Giữ cho ngắn gọn** - Tối đa 1-2 trang
- **Trung thực về các đánh đổi** - Bao gồm các nhược điểm thực sự
- **Liên kết các quyết định liên quan** - Xây dựng đồ thị quyết định
- **Cập nhật trạng thái** - Phản đối khi bị thay thế

### Không nên làm

- **Không thay đổi các ADR đã được chấp nhận** - Viết cái mới để thay thế
- **Không bỏ qua ngữ cảnh** - Người đọc tương lai cần nền tảng
- **Không che giấu thất bại** - Các quyết định bị từ chối có giá trị
- **Không mơ hồ** - Quyết định cụ thể, hậu quả cụ thể
- **Không quên triển khai** - ADR không có hành động là lãng phí

## Tài nguyên

- [Ghi lại các Quyết định Kiến trúc (Michael Nygard)](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
- [Mẫu MADR](https://adr.github.io/madr/)
- [Tổ chức GitHub ADR](https://adr.github.io/)
- [adr-tools](https://github.com/npryce/adr-tools)
