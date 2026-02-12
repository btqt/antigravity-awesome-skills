---
name: data-engineering-data-driven-feature
description: "Xây dựng các tính năng được hướng dẫn bởi thông tin chi tiết dữ liệu, thử nghiệm A/B và đo lường liên tục bằng cách sử dụng các tác nhân chuyên biệt để phân tích, triển khai và thử nghiệm."
---

# Phát Triển Tính Năng Dựa Trên Dữ Liệu (Data-Driven Feature Development)

Xây dựng các tính năng được hướng dẫn bởi thông tin chi tiết dữ liệu, thử nghiệm A/B và đo lường liên tục bằng cách sử dụng các tác nhân chuyên biệt để phân tích, triển khai và thử nghiệm.

[Suy nghĩ mở rộng: Quy trình làm việc này điều phối một quy trình phát triển dựa trên dữ liệu toàn diện từ phân tích dữ liệu ban đầu và hình thành giả thuyết thông qua triển khai tính năng với phân tích tích hợp, cơ sở hạ tầng thử nghiệm A/B và phân tích sau khi ra mắt. Mỗi giai đoạn tận dụng các tác nhân chuyên biệt để đảm bảo các tính năng được xây dựng dựa trên thông tin chi tiết dữ liệu, được thiết bị đúng cách để đo lường và được xác thực thông qua các thí nghiệm có kiểm soát. Quy trình làm việc nhấn mạnh các thực tiễn phân tích sản phẩm hiện đại, sự chặt chẽ thống kê trong thử nghiệm và học hỏi liên tục từ hành vi người dùng.]

## Sử dụng kỹ năng này khi

- Làm việc trên các nhiệm vụ hoặc quy trình làm việc phát triển tính năng dựa trên dữ liệu
- Cần hướng dẫn, thực tiễn tốt nhất hoặc danh sách kiểm tra cho phát triển tính năng dựa trên dữ liệu

## Không sử dụng kỹ năng này khi

- Nhiệm vụ không liên quan đến phát triển tính năng dựa trên dữ liệu
- Bạn cần một miền hoặc công cụ khác ngoài phạm vi này

## Hướng dẫn

- Làm rõ mục tiêu, ràng buộc và đầu vào cần thiết.
- Áp dụng các thực tiễn tốt nhất có liên quan và xác thực kết quả.
- Cung cấp các bước có thể hành động và xác minh.
- Nếu cần các ví dụ chi tiết, hãy mở `resources/implementation-playbook.md`.

## Giai đoạn 1: Phân tích Dữ liệu và Hình thành Giả thuyết

### 1. Phân tích Dữ liệu Thăm dò (EDA)

- Sử dụng công cụ Task với subagent_type="machine-learning-ops::data-scientist"
- Lời nhắc: "Thực hiện phân tích dữ liệu thăm dò cho tính năng: $ARGUMENTS. Phân tích dữ liệu hành vi người dùng hiện có, xác định các mẫu và cơ hội, phân đoạn người dùng theo hành vi và tính toán các chỉ số cơ sở. Sử dụng các công cụ phân tích hiện đại (Amplitude, Mixpanel, Segment) để hiểu hành trình người dùng hiện tại, phễu chuyển đổi và các mẫu tương tác."
- Đầu ra: Báo cáo EDA với trực quan hóa, phân đoạn người dùng, các mẫu hành vi, chỉ số cơ sở

### 2. Phát triển Giả thuyết Kinh doanh

- Sử dụng công cụ Task với subagent_type="business-analytics::business-analyst"
- Bối cảnh: Phát hiện EDA và các mẫu hành vi của nhà khoa học dữ liệu
- Lời nhắc: "Xây dựng các giả thuyết kinh doanh cho tính năng: $ARGUMENTS dựa trên phân tích dữ liệu. Xác định các chỉ số thành công rõ ràng, tác động mong đợi đến các KPI kinh doanh chính, phân đoạn người dùng mục tiêu và các hiệu ứng phát hiện tối thiểu. Tạo các giả thuyết có thể đo lường bằng cách sử dụng các khung như điểm ICE hoặc ưu tiên RICE."
- Đầu ra: Tài liệu giả thuyết, định nghĩa chỉ số thành công, tính toán ROI mong đợi

### 3. Thiết kế Thí nghiệm Thống kê

- Sử dụng công cụ Task với subagent_type="machine-learning-ops::data-scientist"
- Bối cảnh: Giả thuyết kinh doanh và chỉ số thành công
- Lời nhắc: "Thiết kế thí nghiệm thống kê cho tính năng: $ARGUMENTS. Tính toán kích thước mẫu cần thiết cho sức mạnh thống kê, xác định nhóm kiểm soát và điều trị, chỉ định chiến lược ngẫu nhiên hóa và lập kế hoạch cho các sửa chữa kiểm tra nhiều lần. Xem xét các phương pháp thử nghiệm A/B Bayesian để ra quyết định nhanh hơn. Thiết kế cho cả chỉ số chính và chỉ số bảo vệ (guardrail metrics)."
- Đầu ra: Tài liệu thiết kế thí nghiệm, phân tích sức mạnh, kế hoạch kiểm tra thống kê

## Giai đoạn 2: Kiến trúc Tính năng và Thiết kế Phân tích

### 4. Lập kế hoạch Kiến trúc Tính năng

- Sử dụng công cụ Task với subagent_type="data-engineering::backend-architect"
- Bối cảnh: Yêu cầu kinh doanh và thiết kế thí nghiệm
- Lời nhắc: "Thiết kế kiến trúc tính năng cho: $ARGUMENTS với khả năng thử nghiệm A/B. Bao gồm tích hợp cờ tính năng (LaunchDarkly, Split.io, hoặc Optimizely), chiến lược triển khai dần dần, ngắt mạch để an toàn và tách biệt sạch sẽ giữa logic kiểm soát và điều trị. Đảm bảo kiến trúc hỗ trợ cập nhật cấu hình thời gian thực."
- Đầu ra: Sơ đồ kiến trúc, lược đồ cờ tính năng, chiến lược triển khai

### 5. Thiết kế Thiết bị Phân tích

- Sử dụng công cụ Task với subagent_type="data-engineering::data-engineer"
- Bối cảnh: Kiến trúc tính năng và chỉ số thành công
- Lời nhắc: "Thiết kế thiết bị phân tích toàn diện cho: $ARGUMENTS. Xác định lược đồ sự kiện cho tương tác người dùng, chỉ định thuộc tính cho phân đoạn và phân tích, thiết kế theo dõi phễu và sự kiện chuyển đổi, lập kế hoạch khả năng phân tích đoàn hệ. Triển khai bằng các SDK hiện đại (Segment, Amplitude, Mixpanel) với phân loại sự kiện thích hợp."
- Đầu ra: Kế hoạch theo dõi sự kiện, lược đồ phân tích, hướng dẫn thiết bị

### 6. Kiến trúc Đường ống Dữ liệu

- Sử dụng công cụ Task với subagent_type="data-engineering::data-engineer"
- Bối cảnh: Yêu cầu phân tích và cơ sở hạ tầng dữ liệu hiện có
- Lời nhắc: "Thiết kế đường ống dữ liệu cho tính năng: $ARGUMENTS. Bao gồm truyền phát thời gian thực cho các chỉ số trực tiếp (Kafka, Kinesis), xử lý hàng loạt cho phân tích chi tiết, tích hợp kho dữ liệu (Snowflake, BigQuery), và cửa hàng tính năng cho ML nếu áp dụng. Đảm bảo quản trị dữ liệu thích hợp và tuân thủ GDPR."
- Đầu ra: Kiến trúc đường ống, thông số kỹ thuật ETL/ELT, sơ đồ luồng dữ liệu

## Giai đoạn 3: Triển khai với Thiết bị

### 7. Triển khai Backend

- Sử dụng công cụ Task với subagent_type="backend-development::backend-architect"
- Bối cảnh: Thiết kế kiến trúc và yêu cầu tính năng
- Lời nhắc: "Triển khai backend cho tính năng: $ARGUMENTS với thiết bị đầy đủ. Bao gồm kiểm tra cờ tính năng tại các điểm quyết định, theo dõi sự kiện toàn diện cho tất cả hành động người dùng, thu thập chỉ số hiệu suất, theo dõi lỗi và giám sát. Triển khai ghi nhật ký thích hợp cho phân tích thí nghiệm."
- Đầu ra: Mã backend với phân tích, tích hợp cờ tính năng, thiết lập giám sát

### 8. Triển khai Frontend

- Sử dụng công cụ Task với subagent_type="frontend-mobile-development::frontend-developer"
- Bối cảnh: API Backend và yêu cầu phân tích
- Lời nhắc: "Xây dựng frontend cho tính năng: $ARGUMENTS với theo dõi phân tích. Triển khai theo dõi sự kiện cho tất cả tương tác người dùng, tích hợp ghi lại phiên nếu áp dụng, chỉ số hiệu suất (Core Web Vitals), và ranh giới lỗi thích hợp. Đảm bảo trải nghiệm nhất quán giữa các nhóm kiểm soát và điều trị."
- Đầu ra: Mã frontend với phân tích, các biến thể thử nghiệm A/B, giám sát hiệu suất

### 9. Tích hợp Mô hình ML (nếu áp dụng)

- Sử dụng công cụ Task với subagent_type="machine-learning-ops::ml-engineer"
- Bối cảnh: Yêu cầu tính năng và đường ống dữ liệu
- Lời nhắc: "Tích hợp các mô hình ML cho tính năng: $ARGUMENTS nếu cần. Triển khai suy luận trực tuyến với độ trễ thấp, thử nghiệm A/B giữa các phiên bản mô hình, theo dõi hiệu suất mô hình và cơ chế dự phòng tự động. Thiết lập giám sát mô hình để phát hiện trôi dạt."
- Đầu ra: Đường ống ML, cơ sở hạ tầng phục vụ mô hình, thiết lập giám sát

## Giai đoạn 4: Xác thực Trước khi Ra mắt

### 10. Xác thực Phân tích

- Sử dụng công cụ Task với subagent_type="data-engineering::data-engineer"
- Bối cảnh: Các lược đồ theo dõi và sự kiện đã triển khai
- Lời nhắc: "Xác thực triển khai phân tích cho: $ARGUMENTS. Kiểm tra tất cả theo dõi sự kiện trong giai đoạn staging, xác minh chất lượng và tính đầy đủ của dữ liệu, xác thực định nghĩa phễu, đảm bảo nhận dạng người dùng và theo dõi phiên thích hợp. Chạy các bài kiểm tra đầu cuối cho đường ống dữ liệu."
- Đầu ra: Báo cáo xác thực, chỉ số chất lượng dữ liệu, phân tích phạm vi theo dõi

### 11. Thiết lập Thí nghiệm

- Sử dụng công cụ Task với subagent_type="cloud-infrastructure::deployment-engineer"
- Bối cảnh: Cờ tính năng và thiết kế thí nghiệm
- Lời nhắc: "Cấu hình cơ sở hạ tầng thí nghiệm cho: $ARGUMENTS. Thiết lập cờ tính năng với các quy tắc nhắm mục tiêu thích hợp, cấu hình phân bổ lưu lượng (bắt đầu với 5-10%), triển khai công tắc ngắt (kill switches), thiết lập cảnh báo giám sát cho các chỉ số chính. Kiểm tra logic ngẫu nhiên hóa và gán."
- Đầu ra: Cấu hình thí nghiệm, bảng điều khiển giám sát, kế hoạch triển khai

## Giai đoạn 5: Ra mắt và Thử nghiệm

### 12. Triển khai Dần dần

- Sử dụng công cụ Task với subagent_type="cloud-infrastructure::deployment-engineer"
- Bối cảnh: Cấu hình thí nghiệm và thiết lập giám sát
- Lời nhắc: "Thực thi triển khai dần dần cho tính năng: $ARGUMENTS. Bắt đầu với dogfooding nội bộ, sau đó là người dùng beta (1-5%), tăng dần đến lưu lượng mục tiêu. Giám sát tỷ lệ lỗi, chỉ số hiệu suất và các chỉ số sớm. Triển khai rollback tự động khi có bất thường."
- Đầu ra: Thực thi triển khai, cảnh báo giám sát, chỉ số sức khỏe

### 13. Giám sát Thời gian thực

- Sử dụng công cụ Task với subagent_type="observability-monitoring::observability-engineer"
- Bối cảnh: Tính năng đã triển khai và chỉ số thành công
- Lời nhắc: "Thiết lập giám sát toàn diện cho: $ARGUMENTS. Tạo bảng điều khiển thời gian thực cho các chỉ số thí nghiệm, cấu hình cảnh báo cho ý nghĩa thống kê, giám sát các chỉ số bảo vệ cho các tác động tiêu cực, theo dõi hiệu suất hệ thống và tỷ lệ lỗi. Sử dụng các công cụ như Datadog, New Relic, hoặc bảng điều khiển tùy chỉnh."
- Đầu ra: Bảng điều khiển giám sát, cấu hình cảnh báo, định nghĩa SLO

## Giai đoạn 6: Phân tích và Ra Quyết định

### 14. Phân tích Thống kê

- Sử dụng công cụ Task với subagent_type="machine-learning-ops::data-scientist"
- Bối cảnh: Dữ liệu thí nghiệm và giả thuyết ban đầu
- Lời nhắc: "Phân tích kết quả thử nghiệm A/B cho: $ARGUMENTS. Tính toán ý nghĩa thống kê với khoảng tin cậy, kiểm tra các hiệu ứng cấp phân đoạn, phân tích tác động chỉ số phụ, điều tra bất kỳ mẫu bất ngờ nào. Sử dụng cả hai cách tiếp cận frequentist và Bayesian. Tính đến việc kiểm tra nhiều lần nếu áp dụng."
- Đầu ra: Báo cáo phân tích thống kê, kiểm tra ý nghĩa, phân tích phân đoạn

### 15. Đánh giá Tác động Kinh doanh

- Sử dụng công cụ Task với subagent_type="business-analytics::business-analyst"
- Bối cảnh: Phân tích thống kê và chỉ số kinh doanh
- Lời nhắc: "Đánh giá tác động kinh doanh của tính năng: $ARGUMENTS. Tính toán ROI thực tế so với mong đợi, phân tích tác động đến các chỉ số kinh doanh chính, đánh giá chi phí-lợi ích bao gồm chi phí vận hành, dự án giá trị dài hạn. Đưa ra khuyến nghị về triển khai đầy đủ, lặp lại hoặc rollback."
- Đầu ra: Báo cáo tác động kinh doanh, phân tích ROI, tài liệu khuyến nghị

### 16. Tối ưu hóa Sau khi Ra mắt

- Sử dụng công cụ Task với subagent_type="machine-learning-ops::data-scientist"
- Bối cảnh: Kết quả ra mắt và phản hồi người dùng
- Lời nhắc: "Xác định các cơ hội tối ưu hóa cho: $ARGUMENTS dựa trên dữ liệu. Phân tích các mẫu hành vi người dùng trong nhóm điều trị, xác định các điểm ma sát trong hành trình người dùng, đề xuất cải tiến dựa trên dữ liệu, lập kế hoạch các thí nghiệm tiếp theo. Sử dụng phân tích đoàn hệ cho tác động dài hạn."
- Đầu ra: Khuyến nghị tối ưu hóa, kế hoạch thí nghiệm tiếp theo

## Tùy chọn Cấu hình

```yaml
experiment_config:
  min_sample_size: 10000
  confidence_level: 0.95
  runtime_days: 14
  traffic_allocation: "gradual" # gradual, fixed, or adaptive

analytics_platforms:
  - amplitude
  - segment
  - mixpanel

feature_flags:
  provider: "launchdarkly" # launchdarkly, split, optimizely, unleash

statistical_methods:
  - frequentist
  - bayesian

monitoring:
  - real_time_metrics: true
  - anomaly_detection: true
  - automatic_rollback: true
```

## Tiêu chí Thành công

- **Phạm vi Dữ liệu**: 100% tương tác người dùng được theo dõi với lược đồ sự kiện thích hợp
- **Hiệu lực Thí nghiệm**: Ngẫu nhiên hóa thích hợp, đủ sức mạnh thống kê, không có sự không phù hợp tỷ lệ mẫu
- **Sự chặt chẽ Thống kê**: Kiểm tra ý nghĩa rõ ràng, khoảng tin cậy thích hợp, sửa chữa kiểm tra nhiều lần
- **Tác động Kinh doanh**: Cải thiện có thể đo lường trong các chỉ số mục tiêu mà không làm giảm các chỉ số bảo vệ
- **Hiệu suất Kỹ thuật**: Không suy giảm độ trễ p95, tỷ lệ lỗi dưới 0.1%
- **Tốc độ Quyết định**: Quyết định đi/không đi rõ ràng trong thời gian chạy thí nghiệm đã lên kế hoạch
- **Kết quả Học tập**: Thông tin chi tiết được ghi lại cho sự phát triển tính năng trong tương lai

## Ghi chú Phối hợp

- Các nhà khoa học dữ liệu và nhà phân tích kinh doanh cộng tác về hình thành giả thuyết
- Các kỹ sư triển khai với phân tích là yêu cầu hạng nhất, không phải là suy nghĩ sau
- Cờ tính năng cho phép thử nghiệm an toàn mà không cần triển khai đầy đủ
- Giám sát thời gian thực cho phép lặp lại nhanh và rollback nếu cần
- Sự chặt chẽ thống kê cân bằng với tính thực tế kinh doanh và tốc độ ra thị trường
- Vòng học tập liên tục phản hồi vào chu kỳ phát triển tính năng tiếp theo

Tính năng để phát triển với cách tiếp cận dựa trên dữ liệu: $ARGUMENTS
