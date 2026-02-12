---
name: application-performance-performance-optimization
description: "Tối ưu hóa hiệu suất ứng dụng đầu cuối (end-to-end) với profiling, khả năng quan sát (observability), và tinh chỉnh backend/frontend. Sử dụng khi phối hợp tối ưu hóa hiệu suất trên toàn bộ stack."
---

Tối ưu hóa hiệu suất ứng dụng đầu cuối (end-to-end) sử dụng các agent chuyên biệt về hiệu suất và tối ưu hóa:

[Tư duy mở rộng: Quy trình làm việc này điều phối một quá trình tối ưu hóa hiệu suất toàn diện trên toàn bộ stack ứng dụng. Bắt đầu với việc profiling sâu và thiết lập mức cơ sở (baseline), quy trình tiến triển qua các tối ưu hóa có mục tiêu trong từng lớp hệ thống, xác thực các cải tiến thông qua kiểm thử tải, và thiết lập giám sát liên tục để duy trì hiệu suất. Mỗi giai đoạn xây dựng dựa trên thông tin chi tiết từ các giai đoạn trước, tạo ra một chiến lược tối ưu hóa dựa trên dữ liệu giải quyết các điểm nghẽn thực sự thay vì các cải tiến lý thuyết. Quy trình nhấn mạnh các thực hành khả năng quan sát hiện đại, các chỉ số hiệu suất tập trung vào người dùng, và các chiến lược tối ưu hóa hiệu quả về chi phí.]

## Sử dụng kỹ năng này khi

- Phối hợp tối ưu hóa hiệu suất trên backend, frontend, và cơ sở hạ tầng
- Thiết lập các mức cơ sở và profiling để xác định điểm nghẽn
- Thiết kế kiểm thử tải, ngân sách hiệu suất, hoặc kế hoạch năng lực (capacity plans)
- Xây dựng khả năng quan sát cho các mục tiêu hiệu suất và độ tin cậy

## Không sử dụng kỹ năng này khi

- Nhiệm vụ là một bản sửa lỗi nhỏ cục bộ không có mục tiêu hiệu suất rộng hơn
- Không có quyền truy cập vào các dữ liệu metrics, tracing, hoặc profiling
- Yêu cầu không liên quan đến hiệu suất hoặc khả năng mở rộng

## Hướng dẫn

1. Xác nhận các mục tiêu hiệu suất, ràng buộc, và chỉ số mục tiêu.
2. Thiết lập các mức cơ sở với profiling, tracing, và dữ liệu người dùng thực.
3. Thực thi các tối ưu hóa theo giai đoạn trên toàn bộ stack với tác động có thể đo lường.
4. Xác thực các cải tiến và đặt ra các rào chắn (guardrails) để ngăn chặn sự suy giảm (regressions).

## An toàn

- Tránh kiểm thử tải trên môi trường production mà không có sự chấp thuận và biện pháp bảo vệ.
- Triển khai các thay đổi hiệu suất dần dần với kế hoạch khôi phục (rollback plans).

## Giai đoạn 1: Profiling Hiệu suất & Mức cơ sở

### 1. Profiling Hiệu suất Toàn diện

- Sử dụng công cụ Task với subagent_type="performance-engineer"
- Prompt: "Profile application performance comprehensively for: $ARGUMENTS. Generate flame graphs for CPU usage, heap dumps for memory analysis, trace I/O operations, and identify hot paths. Use APM tools like DataDog or New Relic if available. Include database query profiling, API response times, and frontend rendering metrics. Establish performance baselines for all critical user journeys."
- Ngữ cảnh: Điều tra hiệu suất ban đầu
- Đầu ra: Hồ sơ hiệu suất chi tiết với biểu đồ flame, phân tích bộ nhớ, xác định điểm nghẽn, các chỉ số cơ sở

### 2. Đánh giá Stack Khả năng quan sát (Observability Stack Assessment)

- Sử dụng công cụ Task với subagent_type="observability-engineer"
- Prompt: "Assess current observability setup for: $ARGUMENTS. Review existing monitoring, distributed tracing with OpenTelemetry, log aggregation, and metrics collection. Identify gaps in visibility, missing metrics, and areas needing better instrumentation. Recommend APM tool integration and custom metrics for business-critical operations."
- Ngữ cảnh: Hồ sơ hiệu suất từ bước 1
- Đầu ra: Báo cáo đánh giá khả năng quan sát, các khoảng trống trong đo lường (instrumentation), đề xuất giám sát

### 3. Phân tích Trải nghiệm Người dùng

- Sử dụng công cụ Task với subagent_type="performance-engineer"
- Prompt: "Analyze user experience metrics for: $ARGUMENTS. Measure Core Web Vitals (LCP, FID, CLS), page load times, time to interactive, and perceived performance. Use Real User Monitoring (RUM) data if available. Identify user journeys with poor performance and their business impact."
- Ngữ cảnh: Các mức cơ sở hiệu suất từ bước 1
- Đầu ra: Báo cáo hiệu suất UX, phân tích Core Web Vitals, đánh giá tác động người dùng

## Giai đoạn 2: Tối ưu hóa Database & Backend

### 4. Tối ưu hóa Hiệu suất Cơ sở dữ liệu

- Sử dụng công cụ Task với subagent_type="database-cloud-optimization::database-optimizer"
- Prompt: "Optimize database performance for: $ARGUMENTS based on profiling data: {context_from_phase_1}. Analyze slow query logs, create missing indexes, optimize execution plans, implement query result caching with Redis/Memcached. Review connection pooling, prepared statements, and batch processing opportunities. Consider read replicas and database sharding if needed."
- Ngữ cảnh: Các điểm nghẽn hiệu suất từ giai đoạn 1
- Đầu ra: Các truy vấn được tối ưu hóa, chỉ mục mới, chiến lược caching, cấu hình connection pool

### 5. Tối ưu hóa Backend Code & API

- Sử dụng công cụ Task với subagent_type="backend-development::backend-architect"
- Prompt: "Optimize backend services for: $ARGUMENTS targeting bottlenecks: {context_from_phase_1}. Implement efficient algorithms, add application-level caching, optimize N+1 queries, use async/await patterns effectively. Implement pagination, response compression, GraphQL query optimization, and batch API operations. Add circuit breakers and bulkheads for resilience."
- Ngữ cảnh: Các tối ưu hóa cơ sở dữ liệu từ bước 4, dữ liệu profiling từ giai đoạn 1
- Đầu ra: Backend code được tối ưu hóa, triển khai caching, cải tiến API, các mẫu resilience

### 6. Tối ưu hóa Microservices & Hệ thống Phân tán

- Sử dụng công cụ Task với subagent_type="performance-engineer"
- Prompt: "Optimize distributed system performance for: $ARGUMENTS. Analyze service-to-service communication, implement service mesh optimizations, optimize message queue performance (Kafka/RabbitMQ), reduce network hops. Implement distributed caching strategies and optimize serialization/deserialization."
- Ngữ cảnh: Các tối ưu hóa backend từ bước 5
- Đầu ra: Cải tiến giao tiếp dịch vụ, tối ưu hóa hàng đợi tin nhắn, thiết lập distributed caching

## Giai đoạn 3: Tối ưu hóa Frontend & CDN

### 7. Tối ưu hóa Frontend Bundle & Loading

- Sử dụng công cụ Task với subagent_type="frontend-developer"
- Prompt: "Optimize frontend performance for: $ARGUMENTS targeting Core Web Vitals: {context_from_phase_1}. Implement code splitting, tree shaking, lazy loading, and dynamic imports. Optimize bundle sizes with webpack/rollup analysis. Implement resource hints (prefetch, preconnect, preload). Optimize critical rendering path and eliminate render-blocking resources."
- Ngữ cảnh: Phân tích UX từ giai đoạn 1, tối ưu hóa backend từ giai đoạn 2
- Đầu ra: Các bundle được tối ưu hóa, triển khai lazy loading, cải thiện Core Web Vitals

### 8. Tối ưu hóa CDN & Edge

- Sử dụng công cụ Task với subagent_type="cloud-infrastructure::cloud-architect"
- Prompt: "Optimize CDN and edge performance for: $ARGUMENTS. Configure CloudFlare/CloudFront for optimal caching, implement edge functions for dynamic content, set up image optimization with responsive images and WebP/AVIF formats. Configure HTTP/2 and HTTP/3, implement Brotli compression. Set up geographic distribution for global users."
- Ngữ cảnh: Các tối ưu hóa frontend từ bước 7
- Đầu ra: Cấu hình CDN, quy tắc edge caching, thiết lập nén, tối ưu hóa địa lý

### 9. Tối ưu hóa Mobile & Progressive Web App

- Sử dụng công cụ Task với subagent_type="frontend-mobile-development::mobile-developer"
- Prompt: "Optimize mobile experience for: $ARGUMENTS. Implement service workers for offline functionality, optimize for slow networks with adaptive loading. Reduce JavaScript execution time for mobile CPUs. Implement virtual scrolling for long lists. Optimize touch responsiveness and smooth animations. Consider React Native/Flutter specific optimizations if applicable."
- Ngữ cảnh: Các tối ưu hóa frontend từ các bước 7-8
- Đầu ra: Code tối ưu hóa cho mobile, triển khai PWA, chức năng offline

## Giai đoạn 4: Kiểm thử Tải & Xác thực

### 10. Kiểm thử Tải Toàn diện

- Sử dụng công cụ Task với subagent_type="performance-engineer"
- Prompt: "Conduct comprehensive load testing for: $ARGUMENTS using k6/Gatling/Artillery. Design realistic load scenarios based on production traffic patterns. Test normal load, peak load, and stress scenarios. Include API testing, browser-based testing, and WebSocket testing if applicable. Measure response times, throughput, error rates, and resource utilization at various load levels."
- Ngữ cảnh: Tất cả các tối ưu hóa từ giai đoạn 1-3
- Đầu ra: Kết quả kiểm thử tải, hiệu suất dưới tải, điểm gãy, phân tích khả năng mở rộng

### 11. Kiểm thử Hồi quy Hiệu suất (Performance Regression Testing)

- Sử dụng công cụ Task với subagent_type="performance-testing-review::test-automator"
- Prompt: "Create automated performance regression tests for: $ARGUMENTS. Set up performance budgets for key metrics, integrate with CI/CD pipeline using GitHub Actions or similar. Create Lighthouse CI tests for frontend, API performance tests with Artillery, and database performance benchmarks. Implement automatic rollback triggers for performance regressions."
- Ngữ cảnh: Kết quả kiểm thử tải từ bước 10, các chỉ số cơ sở từ giai đoạn 1
- Đầu ra: Bộ kiểm thử hiệu suất, tích hợp CI/CD, hệ thống ngăn chặn hồi quy

## Giai đoạn 5: Giám sát & Tối ưu hóa Liên tục

### 12. Thiết lập Giám sát Production

- Sử dụng công cụ Task với subagent_type="observability-engineer"
- Prompt: "Implement production performance monitoring for: $ARGUMENTS. Set up APM with DataDog/New Relic/Dynatrace, configure distributed tracing with OpenTelemetry, implement custom business metrics. Create Grafana dashboards for key metrics, set up PagerDuty alerts for performance degradation. Define SLIs/SLOs for critical services with error budgets."
- Ngữ cảnh: Các cải tiến hiệu suất từ tất cả các giai đoạn trước
- Đầu ra: Dashboards giám sát, quy tắc cảnh báo, định nghĩa SLI/SLO, sổ tay vận hành (runbooks)

### 13. Tối ưu hóa Hiệu suất Liên tục

- Sử dụng công cụ Task với subagent_type="performance-engineer"
- Prompt: "Establish continuous optimization process for: $ARGUMENTS. Create performance budget tracking, implement A/B testing for performance changes, set up continuous profiling in production. Document optimization opportunities backlog, create capacity planning models, and establish regular performance review cycles."
- Ngữ cảnh: Thiết lập giám sát từ bước 12, tất cả công việc tối ưu hóa trước đó
- Đầu ra: Theo dõi ngân sách hiệu suất, backlog tối ưu hóa, lập kế hoạch năng lực, quy trình xem xét

## Các Tùy chọn Cấu hình

- **performance_focus**: "latency" | "throughput" | "cost" | "balanced" (mặc định: "balanced")
- **optimization_depth**: "quick-wins" | "comprehensive" | "enterprise" (mặc định: "comprehensive")
- **tools_available**: ["datadog", "newrelic", "prometheus", "grafana", "k6", "gatling"]
- **budget_constraints**: Đặt chi phí chấp nhận được tối đa cho các thay đổi cơ sở hạ tầng
- **user_impact_tolerance**: "zero-downtime" | "maintenance-window" | "gradual-rollout"

## Tiêu chí Thành công

- **Thời gian Phản hồi**: P50 < 200ms, P95 < 1s, P99 < 2s cho các endpoint quan trọng
- **Core Web Vitals**: LCP < 2.5s, FID < 100ms, CLS < 0.1
- **Thông lượng**: Hỗ trợ 2x tải đỉnh hiện tại với tỷ lệ lỗi < 1%
- **Hiệu suất Database**: Truy vấn P95 < 100ms, không có truy vấn > 1s
- **Sử dụng Tài nguyên**: CPU < 70%, Memory < 80% dưới tải bình thường
- **Hiệu quả Chi phí**: Hiệu suất trên mỗi đô la được cải thiện tối thiểu 30%
- **Phạm vi Giám sát**: 100% các đường dẫn quan trọng được đo lường (instrumented) với cảnh báo

Mục tiêu tối ưu hóa hiệu suất: $ARGUMENTS
