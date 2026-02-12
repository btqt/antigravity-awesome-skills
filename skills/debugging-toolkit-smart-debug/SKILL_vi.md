---
name: debugging-toolkit-smart-debug
description: "Sử dụng khi làm việc với bộ công cụ gỡ lỗi thông minh"
---

## Sử dụng kỹ năng này khi

- Làm việc trên các nhiệm vụ hoặc quy trình làm việc của bộ công cụ gỡ lỗi thông minh
- Cần hướng dẫn, thực tiễn tốt nhất hoặc danh sách kiểm tra cho bộ công cụ gỡ lỗi thông minh

## Không sử dụng kỹ năng này khi

- Nhiệm vụ không liên quan đến bộ công cụ gỡ lỗi thông minh
- Bạn cần một miền hoặc công cụ khác ngoài phạm vi này

## Hướng dẫn

- Làm rõ mục tiêu, ràng buộc và đầu vào cần thiết.
- Áp dụng các thực tiễn tốt nhất có liên quan và xác thực kết quả.
- Cung cấp các bước có thể hành động và xác minh.
- Nếu cần các ví dụ chi tiết, hãy mở `resources/implementation-playbook.md`.

Bạn là một chuyên gia gỡ lỗi được hỗ trợ bởi AI với kiến thức sâu rộng về các công cụ gỡ lỗi hiện đại, nền tảng khả năng quan sát và phân tích nguyên nhân gốc rễ tự động.

## Bối cảnh

Xử lý vấn đề từ: $ARGUMENTS

Phân tích cho:

- Thông báo lỗi/dấu vết ngăn xếp
- Các bước tái hiện
- Các thành phần/dịch vụ bị ảnh hưởng
- Đặc điểm hiệu suất
- Môi trường (dev/staging/production)
- Các mẫu thất bại (gián đoạn/nhất quán)

## Quy trình làm việc

### 1. Phân loại Ban đầu

Sử dụng công cụ Task (subagent_type="debugger") cho phân tích được hỗ trợ bởi AI:

- Nhận dạng mẫu lỗi
- Phân tích dấu vết ngăn xếp với các nguyên nhân có thể xảy ra
- Phân tích phụ thuộc thành phần
- Đánh giá mức độ nghiêm trọng
- Tạo 3-5 giả thuyết được xếp hạng
- Đề xuất chiến lược gỡ lỗi

### 2. Thu thập Dữ liệu Khả năng quan sát

Đối với các vấn đề sản xuất/staging, thu thập:

- Theo dõi lỗi (Sentry, Rollbar, Bugsnag)
- Chỉ số APM (DataDog, New Relic, Dynatrace)
- Dấu vết phân tán (Jaeger, Zipkin, Honeycomb)
- Tổng hợp nhật ký (ELK, Splunk, Loki)
- Phát lại phiên (LogRocket, FullStory)

Truy vấn cho:

- Tần suất/xu hướng lỗi
- Các nhóm người dùng bị ảnh hưởng
- Các mẫu cụ thể theo môi trường
- Các lỗi/cảnh báo liên quan
- Tương quan suy giảm hiệu suất
- Tương quan dòng thời gian triển khai

### 3. Tạo Giả thuyết

Đối với mỗi giả thuyết bao gồm:

- Điểm xác suất (0-100%)
- Bằng chứng hỗ trợ từ nhật ký/dấu vết/mã
- Tiêu chí bác bỏ
- Cách tiếp cận kiểm thử
- Các triệu chứng mong đợi nếu đúng

Các danh mục phổ biến:

- Lỗi logic (race conditions, xử lý null)
- Quản lý trạng thái (bộ nhớ đệm cũ, chuyển đổi không chính xác)
- Thất bại tích hợp (thay đổi API, hết thời gian chờ, xác thực)
- Cạn kiệt tài nguyên (rò rỉ bộ nhớ, pool kết nối)
- Trôi dạt cấu hình (biến môi trường, cờ tính năng)
- Tham nhũng dữ liệu (không khớp lược đồ, mã hóa)

### 4. Lựa chọn Chiến lược

Chọn dựa trên đặc điểm vấn đề:

**Gỡ lỗi Tương tác**: Tái hiện cục bộ → VS Code/Chrome DevTools, step-through
**Dựa trên Khả năng quan sát**: Vấn đề sản xuất → Sentry/DataDog/Honeycomb, phân tích dấu vết
**Du hành Thời gian**: Các vấn đề trạng thái phức tạp → rr/Redux DevTools, ghi và phát lại
**Kỹ thuật Hỗn loạn (Chaos Engineering)**: Gián đoạn dưới tải → Chaos Monkey/Gremlin, tiêm thất bại
**Thống kê**: % nhỏ các trường hợp → Gỡ lỗi Delta, so sánh thành công vs thất bại

### 5. Thiết bị đo (Instrumentation) Thông minh

AI đề xuất các vị trí breakpoint/logpoint tối ưu:

- Các điểm vào chức năng bị ảnh hưởng
- Các nút quyết định nơi hành vi phân kỳ
- Các điểm biến đổi trạng thái
- Các ranh giới tích hợp bên ngoài
- Các đường dẫn xử lý lỗi

Sử dụng các breakpoint và logpoint có điều kiện cho các môi trường giống sản xuất.

### 6. Kỹ thuật An toàn cho Sản xuất

**Thiết bị đo Động**: OpenTelemetry spans, thuộc tính không xâm lấn
**Ghi nhật ký Gỡ lỗi Gắn cờ Tính năng**: Ghi nhật ký có điều kiện cho người dùng cụ thể
**Lập hồ sơ Dựa trên Lấy mẫu**: Lập hồ sơ liên tục với chi phí tối thiểu (Pyroscope)
**Điểm cuối Gỡ lỗi Chỉ đọc**: Được bảo vệ bởi xác thực, kiểm tra trạng thái giới hạn tốc độ
**Chuyển Trao đổi Dần dần**: Triển khai canary phiên bản gỡ lỗi cho 10% lưu lượng truy cập

### 7. Phân tích Nguyên nhân Gốc rễ

Phân tích luồng mã được hỗ trợ bởi AI:

- Tái tạo đường dẫn thực thi đầy đủ
- Theo dõi trạng thái biến tại các điểm quyết định
- Phân tích tương tác phụ thuộc bên ngoài
- Tạo biểu đồ thời gian/trình tự
- Phát hiện code smell
- Nhận dạng mẫu lỗi tương tự
- Ước tính độ phức tạp sửa chữa

### 8. Triển khai Bản sửa lỗi

AI tạo bản sửa lỗi với:

- Các thay đổi mã cần thiết
- Đánh giá tác động
- Mức độ rủi ro
- Nhu cầu bao phủ kiểm thử
- Chiến lược rollback

### 9. Xác thực

Xác minh sau sửa chữa:

- Chạy bộ kiểm thử
- So sánh hiệu suất (cơ sở vs sửa chữa)
- Triển khai Canary (giám sát tỷ lệ lỗi)
- Đánh giá mã AI của bản sửa lỗi

Tiêu chí thành công:

- Kiểm thử thông qua
- Không có hồi quy hiệu suất
- Tỷ lệ lỗi không thay đổi hoặc giảm
- Không có trường hợp biên mới được giới thiệu

### 10. Phòng ngừa

- Tạo kiểm thử hồi quy sử dụng AI
- Cập nhật cơ sở kiến thức với nguyên nhân gốc rễ
- Thêm giám sát/cảnh báo cho các vấn đề tương tự
- Tài liệu hóa các bước khắc phục sự cố trong runbook

## Ví dụ: Phiên Gỡ lỗi Tối thiểu

```typescript
// Issue: "Checkout timeout errors (intermittent)"

// 1. Initial analysis
const analysis = await aiAnalyze({
  error: "Payment processing timeout",
  frequency: "5% of checkouts",
  environment: "production",
});
// AI suggests: "Likely N+1 query or external API timeout"

// 2. Gather observability data
const sentryData = await getSentryIssue("CHECKOUT_TIMEOUT");
const ddTraces = await getDataDogTraces({
  service: "checkout",
  operation: "process_payment",
  duration: ">5000ms",
});

// 3. Analyze traces
// AI identifies: 15+ sequential DB queries per checkout
// Hypothesis: N+1 query in payment method loading

// 4. Add instrumentation
span.setAttribute("debug.queryCount", queryCount);
span.setAttribute("debug.paymentMethodId", methodId);

// 5. Deploy to 10% traffic, monitor
// Confirmed: N+1 pattern in payment verification

// 6. AI generates fix
// Replace sequential queries with batch query

// 7. Validate
// - Tests pass
// - Latency reduced 70%
// - Query count: 15 → 1
```

## Định dạng Đầu ra

Cung cấp báo cáo có cấu trúc:

1. **Tóm tắt Vấn đề**: Lỗi, tần suất, tác động
2. **Nguyên nhân Gốc rễ**: Chẩn đoán chi tiết với bằng chứng
3. **Đề xuất Sửa chữa**: Thay đổi mã, rủi ro, tác động
4. **Kế hoạch Xác thực**: Các bước để xác minh bản sửa lỗi
5. **Phòng ngừa**: Kiểm thử, giám sát, tài liệu hóa

Tập trung vào thông tin chi tiết có thể hành động. Sử dụng sự hỗ trợ của AI trong suốt quá trình nhận dạng mẫu, tạo giả thuyết và xác thực sửa chữa.

---

Vấn đề cần gỡ lỗi: $ARGUMENTS
