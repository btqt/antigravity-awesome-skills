---
name: backend-development-feature-development
description: "Điều phối phát triển tính năng backend đầu cuối (end-to-end) từ yêu cầu đến triển khai. Sử dụng khi phối hợp phân phối tính năng đa giai đoạn giữa các nhóm và dịch vụ."
---

Điều phối phát triển tính năng đầu cuối từ yêu cầu đến triển khai production:

[Suy nghĩ mở rộng: Quy trình làm việc này điều phối các tác nhân chuyên biệt thông qua các giai đoạn phát triển tính năng toàn diện - từ khám phá và lập kế hoạch đến triển khai, kiểm thử và đưa vào hoạt động. Mỗi giai đoạn xây dựng dựa trên các đầu ra trước đó, đảm bảo việc phân phối tính năng nhất quán. Quy trình hỗ trợ nhiều phương pháp phát triển (truyền thống, TDD/BDD, DDD), các mức độ phức tạp của tính năng, và các chiến lược triển khai hiện đại bao gồm feature flags, rollouts dần dần, và phát triển ưu tiên khả năng quan sát (observability-first). Các tác nhân nhận ngữ cảnh chi tiết từ các giai đoạn trước để duy trì sự nhất quán và chất lượng trong suốt vòng đời phát triển.]

## Sử dụng kỹ năng này khi

- Phối hợp phân phối tính năng đầu cuối giữa backend, frontend, và dữ liệu
- Quản lý yêu cầu, kiến trúc, triển khai, kiểm thử, và rollout
- Lập kế hoạch thay đổi đa dịch vụ với nhu cầu triển khai và giám sát
- Thống nhất các nhóm về phạm vi, rủi ro, và các chỉ số thành công

## Không sử dụng kỹ năng này khi

- Nhiệm vụ là một thay đổi backend nhỏ, biệt lập hoặc sửa lỗi
- Bạn chỉ cần một tác vụ chuyên gia đơn lẻ, không phải toàn bộ quy trình làm việc
- Không có triển khai hoặc phối hợp chéo nhóm

## Hướng dẫn

1. Xác nhận phạm vi tính năng, chỉ số thành công, và các ràng buộc.
2. Chọn phương pháp luận và định nghĩa đầu ra của các giai đoạn.
3. Điều phối triển khai, kiểm thử, và xác thực bảo mật.
4. Chuẩn bị kế hoạch rollout, giám sát, và tài liệu.

## An toàn

- Tránh thay đổi production mà không có phê duyệt và kế hoạch rollback.
- Xác thực di chuyển dữ liệu và feature flags trong staging trước.

## Các Tùy chọn Cấu hình

### Phương pháp Phát triển

- **traditional**: Phát triển tuần tự với kiểm thử sau khi triển khai
- **tdd**: Phát triển Hướng Kiểm thử (Test-Driven Development) với chu trình red-green-refactor
- **bdd**: Phát triển Hướng Hành vi (Behavior-Driven Development) với kiểm thử dựa trên kịch bản
- **ddd**: Thiết kế Hướng Miền (Domain-Driven Design) với bounded contexts và aggregates

### Độ phức tạp Tính năng

- **simple**: Dịch vụ đơn lẻ, tích hợp tối thiểu (1-2 ngày)
- **medium**: Đa dịch vụ, tích hợp trung bình (3-5 ngày)
- **complex**: Chéo miền, tích hợp mở rộng (1-2 tuần)
- **epic**: Thay đổi kiến trúc lớn, đa nhóm (2+ tuần)

### Chiến lược Triển khai

- **direct**: Rollout ngay lập tức cho tất cả người dùng
- **canary**: Rollout dần dần bắt đầu với 5% lưu lượng
- **feature-flag**: Kích hoạt có kiểm soát thông qua các công tắc tính năng (feature toggles)
- **blue-green**: Triển khai không downtime với rollback tức thì
- **a-b-test**: Phân chia lưu lượng để thử nghiệm và đo lường

## Giai đoạn 1: Khám phá & Lập kế hoạch Yêu cầu

1. **Phân tích Kinh doanh & Yêu cầu**
   - Sử dụng công cụ Task với subagent_type="business-analytics::business-analyst"
   - Prompt: "Analyze feature requirements for: $ARGUMENTS. Define user stories, acceptance criteria, success metrics, and business value. Identify stakeholders, dependencies, and risks. Create feature specification document with clear scope boundaries."
   - Đầu ra mong đợi: Tài liệu yêu cầu với user stories, chỉ số thành công, đánh giá rủi ro
   - Ngữ cảnh: Yêu cầu tính năng ban đầu và ngữ cảnh kinh doanh

2. **Thiết kế Kiến trúc Kỹ thuật**
   - Sử dụng công cụ Task với subagent_type="comprehensive-review::architect-review"
   - Prompt: "Design technical architecture for feature: $ARGUMENTS. Using requirements: [include business analysis from step 1]. Define service boundaries, API contracts, data models, integration points, and technology stack. Consider scalability, performance, and security requirements."
   - Đầu ra mong đợi: Tài liệu thiết kế kỹ thuật với sơ đồ kiến trúc, đặc tả API, mô hình dữ liệu
   - Ngữ cảnh: Yêu cầu kinh doanh, kiến trúc hệ thống hiện tại

3. **Đánh giá Khả thi & Rủi ro**
   - Sử dụng công cụ Task với subagent_type="security-scanning::security-auditor"
   - Prompt: "Assess security implications and risks for feature: $ARGUMENTS. Review architecture: [include technical design from step 2]. Identify security requirements, compliance needs, data privacy concerns, and potential vulnerabilities."
   - Đầu ra mong đợi: Đánh giá bảo mật với ma trận rủi ro, danh sách kiểm tra tuân thủ, chiến lược giảm thiểu
   - Ngữ cảnh: Thiết kế kỹ thuật, yêu cầu quy định

## Giai đoạn 2: Triển khai & Phát triển

4. **Triển khai Dịch vụ Backend**
   - Sử dụng công cụ Task với subagent_type="backend-architect"
   - Prompt: "Implement backend services for: $ARGUMENTS. Follow technical design: [include architecture from step 2]. Build RESTful/GraphQL APIs, implement business logic, integrate with data layer, add resilience patterns (circuit breakers, retries), implement caching strategies. Include feature flags for gradual rollout."
   - Đầu ra mong đợi: Các dịch vụ backend với APIs, logic kinh doanh, tích hợp cơ sở dữ liệu, feature flags
   - Ngữ cảnh: Thiết kế kỹ thuật, hợp đồng API, mô hình dữ liệu

5. **Triển khai Frontend**
   - Sử dụng công cụ Task với subagent_type="frontend-mobile-development::frontend-developer"
   - Prompt: "Build frontend components for: $ARGUMENTS. Integrate with backend APIs: [include API endpoints from step 4]. Implement responsive UI, state management, error handling, loading states, and analytics tracking. Add feature flag integration for A/B testing capabilities."
   - Đầu ra mong đợi: Các thành phần frontend với tích hợp API, quản lý trạng thái, analytics
   - Ngữ cảnh: Backend APIs, thiết kế UI/UX, user stories

6. **Tích hợp & Đường ống Dữ liệu**
   - Sử dụng công cụ Task với subagent_type="data-engineering::data-engineer"
   - Prompt: "Build data pipelines for: $ARGUMENTS. Design ETL/ELT processes, implement data validation, create analytics events, set up data quality monitoring. Integrate with product analytics platforms for feature usage tracking."
   - Đầu ra mong đợi: Đường ống dữ liệu, sự kiện analytics, kiểm tra chất lượng dữ liệu
   - Ngữ cảnh: Yêu cầu dữ liệu, nhu cầu analytics, hạ tầng dữ liệu hiện tại

## Giai đoạn 3: Kiểm thử & Đảm bảo Chất lượng

7. **Bộ Kiểm thử Tự động**
   - Sử dụng công cụ Task với subagent_type="unit-testing::test-automator"
   - Prompt: "Create comprehensive test suite for: $ARGUMENTS. Write unit tests for backend: [from step 4] and frontend: [from step 5]. Add integration tests for API endpoints, E2E tests for critical user journeys, performance tests for scalability validation. Ensure minimum 80% code coverage."
   - Đầu ra mong đợi: Bộ kiểm thử với unit, integration, E2E, và performance tests
   - Ngữ cảnh: Mã triển khai, tiêu chí chấp nhận, yêu cầu kiểm thử

8. **Xác thực Bảo mật**
   - Sử dụng công cụ Task với subagent_type="security-scanning::security-auditor"
   - Prompt: "Perform security testing for: $ARGUMENTS. Review implementation: [include backend and frontend from steps 4-5]. Run OWASP checks, penetration testing, dependency scanning, and compliance validation. Verify data encryption, authentication, and authorization."
   - Đầu ra mong đợi: Kết quả kiểm thử bảo mật, báo cáo lỗ hổng, hành động khắc phục
   - Ngữ cảnh: Mã triển khai, yêu cầu bảo mật

9. **Tối ưu hóa Hiệu năng**
   - Sử dụng công cụ Task với subagent_type="application-performance::performance-engineer"
   - Prompt: "Optimize performance for: $ARGUMENTS. Analyze backend services: [from step 4] and frontend: [from step 5]. Profile code, optimize queries, implement caching, reduce bundle sizes, improve load times. Set up performance budgets and monitoring."
   - Đầu ra mong đợi: Các cải tiến hiệu năng, báo cáo tối ưu hóa, chỉ số hiệu năng
   - Ngữ cảnh: Mã triển khai, yêu cầu hiệu năng

## Giai đoạn 4: Triển khai & Giám sát

10. **Chiến lược & Đường ống Triển khai**
    - Sử dụng công cụ Task với subagent_type="deployment-strategies::deployment-engineer"
    - Prompt: "Prepare deployment for: $ARGUMENTS. Create CI/CD pipeline with automated tests: [from step 7]. Configure feature flags for gradual rollout, implement blue-green deployment, set up rollback procedures. Create deployment runbook and rollback plan."
    - Đầu ra mong đợi: Đường ống CI/CD, cấu hình triển khai, quy trình rollback
    - Ngữ cảnh: Bộ kiểm thử, yêu cầu hạ tầng, chiến lược triển khai

11. **Observability & Giám sát**
    - Sử dụng công cụ Task với subagent_type="observability-monitoring::observability-engineer"
    - Prompt: "Set up observability for: $ARGUMENTS. Implement distributed tracing, custom metrics, error tracking, and alerting. Create dashboards for feature usage, performance metrics, error rates, and business KPIs. Set up SLOs/SLIs with automated alerts."
    - Đầu ra mong đợi: Bảng điều khiển giám sát, cảnh báo, định nghĩa SLO, hạ tầng observability
    - Ngữ cảnh: Triển khai tính năng, chỉ số thành công, yêu cầu vận hành

12. **Tài liệu & Chuyển giao Kiến thức**
    - Sử dụng công cụ Task với subagent_type="documentation-generation::docs-architect"
    - Prompt: "Generate comprehensive documentation for: $ARGUMENTS. Create API documentation, user guides, deployment guides, troubleshooting runbooks. Include architecture diagrams, data flow diagrams, and integration guides. Generate automated changelog from commits."
    - Đầu ra mong đợi: Tài liệu API, hướng dẫn người dùng, runbooks, tài liệu kiến trúc
    - Ngữ cảnh: Đầu ra của tất cả các giai đoạn trước

## Tham số Thực thi

### Tham số Bắt buộc

- **--feature**: Tên tính năng và mô tả
- **--methodology**: Cách tiếp cận phát triển (traditional|tdd|bdd|ddd)
- **--complexity**: Mức độ phức tạp của tính năng (simple|medium|complex|epic)

### Tham số Tùy chọn

- **--deployment-strategy**: Cách tiếp cận triển khai (direct|canary|feature-flag|blue-green|a-b-test)
- **--test-coverage-min**: Ngưỡng bao phủ kiểm thử tối thiểu (mặc định: 80%)
- **--performance-budget**: Yêu cầu hiệu năng (ví dụ: <200ms thời gian phản hồi)
- **--rollout-percentage**: Tỷ lệ phần trăm rollout ban đầu cho triển khai dần dần (mặc định: 5%)
- **--feature-flag-service**: Nhà cung cấp feature flag (launchdarkly|split|unleash|custom)
- **--analytics-platform**: Tích hợp analytics (segment|amplitude|mixpanel|custom)
- **--monitoring-stack**: Công cụ observability (datadog|newrelic|grafana|custom)

## Tiêu chí Thành công

- Tất cả các tiêu chí chấp nhận từ yêu cầu kinh doanh đều được đáp ứng
- Độ bao phủ kiểm thử vượt quá ngưỡng tối thiểu (mặc định 80%)
- Quét bảo mật không cho thấy lỗ hổng nghiêm trọng nào
- Hiệu năng đáp ứng ngân sách và SLO đã xác định
- Feature flags được cấu hình cho rollout có kiểm soát
- Giám sát và cảnh báo hoạt động đầy đủ
- Tài liệu hoàn chỉnh và được phê duyệt
- Triển khai thành công lên production với khả năng rollback
- Product analytics theo dõi việc sử dụng tính năng
- Các số liệu A/B test được cấu hình (nếu áp dụng)

## Chiến lược Rollback

Nếu sự cố phát sinh trong hoặc sau khi triển khai:

1. Vô hiệu hóa feature flag ngay lập tức (< 1 phút)
2. Chuyển đổi lưu lượng Blue-green (< 5 phút)
3. Rollback triển khai đầy đủ thông qua CI/CD (< 15 phút)
4. Rollback di chuyển cơ sở dữ liệu nếu cần (phối hợp với nhóm dữ liệu)
5. Post-mortem sự cố và sửa lỗi trước khi tái triển khai

Mô tả tính năng: $ARGUMENTS
