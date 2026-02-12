---
name: comprehensive-review-full-review
description: "Sử dụng khi làm việc với đánh giá toàn diện đánh giá đầy đủ"
---

## Sử dụng kỹ năng này khi

- Làm việc trên các nhiệm vụ hoặc quy trình đánh giá toàn diện đánh giá đầy đủ
- Cần hướng dẫn, thực tiễn tốt nhất hoặc danh sách kiểm tra cho đánh giá toàn diện đánh giá đầy đủ

## Không sử dụng kỹ năng này khi

- Nhiệm vụ không liên quan đến đánh giá toàn diện đánh giá đầy đủ
- Bạn cần một miền hoặc công cụ khác ngoài phạm vi này

## Hướng dẫn

- Làm rõ mục tiêu, ràng buộc và đầu vào cần thiết.
- Áp dụng các thực tiễn tốt nhất có liên quan và xác thực kết quả.
- Cung cấp các bước có thể hành động và xác minh.
- Nếu cần các ví dụ chi tiết, hãy mở `resources/implementation-playbook.md`.

Điều phối đánh giá mã đa chiều toàn diện bằng cách sử dụng các tác nhân đánh giá chuyên biệt

[Tư duy mở rộng: Quy trình làm việc này thực hiện đánh giá mã thấu đáo bằng cách điều phối nhiều tác nhân chuyên biệt theo các giai đoạn tuần tự. Mỗi giai đoạn xây dựng dựa trên các phát hiện trước đó để tạo ra một đánh giá toàn diện bao gồm chất lượng mã, bảo mật, hiệu suất, kiểm thử, tài liệu và các thực tiễn tốt nhất. Quy trình làm việc tích hợp các công cụ đánh giá hỗ trợ AI hiện đại, phân tích tĩnh, quét bảo mật và các số liệu chất lượng tự động. Kết quả được hợp nhất thành phản hồi có thể hành động với hướng dẫn ưu tiên và khắc phục rõ ràng. Cách tiếp cận theo giai đoạn đảm bảo bao phủ kỹ lưỡng trong khi duy trì hiệu quả thông qua việc thực thi tác nhân song song nếu thích hợp.]

## Tùy chọn Cấu hình Đánh giá

- **--security-focus**: Ưu tiên các lỗ hổng bảo mật và tuân thủ OWASP
- **--performance-critical**: Nhấn mạnh các nút thắt hiệu suất và các vấn đề về khả năng mở rộng
- **--tdd-review**: Bao gồm tuân thủ TDD và xác minh kiểm thử trước (test-first)
- **--ai-assisted**: Bật các công cụ đánh giá hỗ trợ AI (Copilot, Codium, Bito)
- **--strict-mode**: Đánh giá thất bại đối với bất kỳ vấn đề quan trọng nào được tìm thấy
- **--metrics-report**: Tạo bảng điều khiển số liệu chất lượng chi tiết
- **--framework [name]**: Áp dụng các thực tiễn tốt nhất cụ thể cho khuôn khổ (React, Spring, Django, v.v.)

## Giai đoạn 1: Đánh giá Chất lượng Mã & Kiến trúc

Sử dụng công cụ Task để điều phối các tác nhân chất lượng và kiến trúc song song:

### 1A. Phân tích Chất lượng Mã

- Sử dụng công cụ Task với subagent_type="code-reviewer"
- Lời nhắc: "Thực hiện đánh giá chất lượng mã toàn diện cho: $ARGUMENTS. Phân tích độ phức tạp của mã, chỉ số khả năng bảo trì, nợ kỹ thuật, trùng lặp mã, quy ước đặt tên và tuân thủ các nguyên tắc Mã sạch. Tích hợp với SonarQube, CodeQL và Semgrep để phân tích tĩnh. Kiểm tra các mùi mã (code smells), các mẫu chống đối (anti-patterns) và vi phạm các nguyên tắc SOLID. Tạo các số liệu độ phức tạp cyclomatic và xác định các cơ hội tái cấu trúc."
- Đầu ra mong đợi: Các số liệu chất lượng, kho lưu trữ mùi mã, các đề xuất tái cấu trúc
- Bối cảnh: Phân tích cơ sở mã ban đầu, không phụ thuộc vào các giai đoạn khác

### 1B. Đánh giá Kiến trúc & Thiết kế

- Sử dụng công cụ Task với subagent_type="architect-review"
- Lời nhắc: "Đánh giá các mô hình thiết kế kiến trúc và tính toàn vẹn cấu trúc trong: $ARGUMENTS. Đánh giá ranh giới microservices, thiết kế API, lược đồ cơ sở dữ liệu, quản lý phụ thuộc và tuân thủ các nguyên tắc Thiết kế Hướng Tên miền (DDD). Kiểm tra các phụ thuộc vòng, ghép nối không phù hợp, thiếu sự trừu tượng và trôi dạt kiến trúc. Xác minh tuân thủ các tiêu chuẩn kiến trúc doanh nghiệp và các mẫu gốc đám mây."
- Đầu ra mong đợi: Đánh giá kiến trúc, phân tích mẫu thiết kế, các đề xuất cấu trúc
- Bối cảnh: Chạy song song với phân tích chất lượng mã

## Giai đoạn 2: Đánh giá Bảo mật & Hiệu suất

Sử dụng công cụ Task với các tác nhân bảo mật và hiệu suất, kết hợp các phát hiện của Giai đoạn 1:

### 2A. Đánh giá Lỗ hổng Bảo mật

- Sử dụng công cụ Task với subagent_type="security-auditor"
- Lời nhắc: "Thực hiện kiểm toán bảo mật toàn diện trên: $ARGUMENTS. Thực hiện phân tích OWASP Top 10, quét lỗ hổng phụ thuộc với Snyk/Trivy, phát hiện bí mật với GitLeaks, đánh giá xác thực đầu vào, đánh giá xác thực/ủy quyền và đánh giá triển khai mật mã. Bao gồm các phát hiện từ đánh giá kiến trúc Giai đoạn 1: {phase1_architecture_context}. Kiểm tra SQL injection, XSS, CSRF, giải mã không an toàn và các vấn đề bảo mật cấu hình."
- Đầu ra mong đợi: Báo cáo lỗ hổng, danh sách CVE, ma trận rủi ro bảo mật, các bước khắc phục
- Bối cảnh: Kết hợp các lỗ hổng kiến trúc được xác định trong Giai đoạn 1B

### 2B. Phân tích Hiệu suất & Khả năng mở rộng

- Sử dụng công cụ Task với subagent_type="application-performance::performance-engineer"
- Lời nhắc: "Tiến hành phân tích hiệu suất và đánh giá khả năng mở rộng cho: $ARGUMENTS. Lập hồ sơ mã cho các điểm nóng CPU/bộ nhớ, phân tích hiệu suất truy vấn cơ sở dữ liệu, đánh giá các chiến lược bộ nhớ đệm, xác định các vấn đề N+1, đánh giá gộp kết nối và đánh giá các mẫu xử lý không đồng bộ. Xem xét các phát hiện kiến trúc từ Giai đoạn 1: {phase1_architecture_context}. Kiểm tra rò rỉ bộ nhớ, tranh chấp tài nguyên và tắc nghẽn dưới tải."
- Đầu ra mong đợi: Các số liệu hiệu suất, phân tích tắc nghẽn, các đề xuất tối ưu hóa
- Bối cảnh: Sử dụng thông tin chi tiết về kiến trúc để xác định các vấn đề hiệu suất hệ thống

## Giai đoạn 3: Đánh giá Kiểm thử & Tài liệu

Sử dụng công cụ Task để đánh giá chất lượng kiểm thử và tài liệu:

### 3A. Phân tích Bao phủ & Chất lượng Kiểm thử

- Sử dụng công cụ Task với subagent_type="unit-testing::test-automator"
- Lời nhắc: "Đánh giá chiến lược và triển khai kiểm thử cho: $ARGUMENTS. Phân tích bao phủ kiểm thử đơn vị, tính đầy đủ của kiểm thử tích hợp, các kịch bản kiểm thử đầu cuối, tuân thủ kim tự tháp kiểm thử và khả năng bảo trì kiểm thử. Đánh giá các số liệu chất lượng kiểm thử bao gồm mật độ khẳng định, cách ly kiểm thử, sử dụng giả lập (mock) và tính chập chờn. Xem xét các yêu cầu kiểm thử bảo mật và hiệu suất từ Giai đoạn 2: {phase2_security_context}, {phase2_performance_context}. Xác minh các thực tiễn TDD nếu cờ --tdd-review được đặt."
- Đầu ra mong đợi: Báo cáo bao phủ, các số liệu chất lượng kiểm thử, phân tích khoảng trống kiểm thử
- Bối cảnh: Kết hợp các yêu cầu kiểm thử bảo mật và hiệu suất từ Giai đoạn 2

### 3B. Đánh giá Tài liệu & Đặc tả API

- Sử dụng công cụ Task với subagent_type="code-documentation::docs-architect"
- Lời nhắc: "Đánh giá tính đầy đủ và chất lượng tài liệu cho: $ARGUMENTS. Đánh giá tài liệu mã nội tuyến, tài liệu API (OpenAPI/Swagger), hồ sơ quyết định kiến trúc (ADR), tính đầy đủ của README, hướng dẫn triển khai và runbook. Xác minh tài liệu phản ánh triển khai thực tế dựa trên tất cả các phát hiện giai đoạn trước: {phase1_context}, {phase2_context}. Kiểm tra tài liệu lỗi thời, thiếu ví dụ và giải thích không rõ ràng."
- Đầu ra mong đợi: Báo cáo bao phủ tài liệu, danh sách không nhất quán, các đề xuất cải thiện
- Bối cảnh: Tham chiếu chéo tất cả các phát hiện trước đó để đảm bảo độ chính xác của tài liệu

## Giai đoạn 4: Tuân thủ Thực tiễn Tốt nhất & Tiêu chuẩn

Sử dụng công cụ Task để xác minh các thực tiễn tốt nhất cụ thể theo khuôn khổ và ngành:

### 4A. Thực tiễn Tốt nhất về Khuôn khổ & Ngôn ngữ

- Sử dụng công cụ Task với subagent_type="framework-migration::legacy-modernizer"
- Lời nhắc: "Xác minh tuân thủ các thực tiễn tốt nhất về khuôn khổ và ngôn ngữ cho: $ARGUMENTS. Kiểm tra các mẫu JavaScript/TypeScript hiện đại, thực tiễn tốt nhất về React hooks, tuân thủ Python PEP, các mẫu doanh nghiệp Java, mã thành ngữ Go, hoặc các quy ước cụ thể theo khuôn khổ (dựa trên cờ --framework). Đánh giá quản lý gói, cấu hình xây dựng, xử lý môi trường và thực tiễn triển khai. Bao gồm tất cả các vấn đề chất lượng từ các giai đoạn trước: {all_previous_contexts}."
- Đầu ra mong đợi: Báo cáo tuân thủ thực tiễn tốt nhất, các đề xuất hiện đại hóa
- Bối cảnh: Tổng hợp tất cả các phát hiện trước đó để hướng dẫn cụ thể theo khuôn khổ

### 4B. Đánh giá Thực tiễn CI/CD & DevOps

- Sử dụng công cụ Task với subagent_type="cicd-automation::deployment-engineer"
- Lời nhắc: "Đánh giá đường ống CI/CD và thực tiễn DevOps cho: $ARGUMENTS. Đánh giá tự động hóa xây dựng, tích hợp tự động hóa kiểm thử, chiến lược triển khai (blue-green, canary), cơ sở hạ tầng dưới dạng mã, thiết lập giám sát/khả năng quan sát và quy trình phản ứng sự cố. Đánh giá bảo mật đường ống, quản lý tạo tác và khả năng quay lại (rollback). Xem xét tất cả các vấn đề được xác định trong các giai đoạn trước ảnh hưởng đến việc triển khai: {all_critical_issues}."
- Đầu ra mong đợi: Đánh giá đường ống, đánh giá mức độ trưởng thành DevOps, các đề xuất tự động hóa
- Bối cảnh: Tập trung vào việc vận hành các bản sửa lỗi cho tất cả các vấn đề đã xác định

## Tạo Báo cáo Hợp nhất

Biên soạn tất cả các đầu ra giai đoạn thành báo cáo đánh giá toàn diện:

### Các Vấn đề Quan trọng (P0 - Phải Sửa Ngay lập tức)

- Lỗ hổng bảo mật với CVSS > 7.0
- Rủi ro mất hoặc hỏng dữ liệu
- Bỏ qua xác thực/ủy quyền
- Các mối đe dọa ổn định sản xuất
- Vi phạm tuân thủ (GDPR, PCI DSS, SOC2)

### Ưu tiên Cao (P1 - Sửa Trước Khi Phát hành Tiếp theo)

- Tắc nghẽn hiệu suất ảnh hưởng đến trải nghiệm người dùng
- Thiếu bao phủ kiểm thử quan trọng
- Các mẫu chống đối kiến trúc gây ra nợ kỹ thuật
- Các phụ thuộc lỗi thời với các lỗ hổng đã biết
- Các vấn đề chất lượng mã ảnh hưởng đến khả năng bảo trì

### Ưu tiên Trung bình (P2 - Lên kế hoạch cho Sprint Tiếp theo)

- Tối ưu hóa hiệu suất không quan trọng
- Khoảng trống và sự không nhất quán của tài liệu
- Cơ hội tái cấu trúc mã
- Cải thiện chất lượng kiểm thử
- Cải tiến tự động hóa DevOps

### Ưu tiên Thấp (P3 - Theo dõi trong Backlog)

- Vi phạm hướng dẫn phong cách
- Các vấn đề mùi mã nhỏ
- Cập nhật tài liệu nên có (nice-to-have)
- Cải tiến thẩm mỹ

## Tiêu chí Thành công

Đánh giá được coi là thành công khi:

- Tất cả các lỗ hổng bảo mật quan trọng được xác định và ghi lại
- Các tắc nghẽn hiệu suất được lập hồ sơ với các đường dẫn khắc phục
- Các khoảng trống bao phủ kiểm thử được lập bản đồ với các đề xuất ưu tiên
- Các rủi ro kiến trúc được đánh giá với các chiến lược giảm thiểu
- Tài liệu phản ánh trạng thái triển khai thực tế
- Tuân thủ thực tiễn tốt nhất về khuôn khổ được xác minh
- Đường ống CI/CD hỗ trợ triển khai an toàn mã đã xem xét
- Phản hồi rõ ràng, có thể hành động được cung cấp cho tất cả các phát hiện
- Bảng điều khiển số liệu hiển thị xu hướng cải thiện
- Nhóm có kế hoạch hành động ưu tiên rõ ràng để khắc phục

Mục tiêu: $ARGUMENTS
