# Các Workflows của Antigravity (Antigravity Workflows)

> Các cẩm nang thực thi (playbooks) dạng Workflow giúp điều phối hiệu quả nhiều skills khác nhau và giảm thiểu tới mức tối đa các trở ngại (friction).

## Workflow là gì?

Một workflow đại diện cho một lộ trình thực thi có tính định hướng dẫn dắt theo từng bước (guided, step-by-step) kết hợp sử dụng nhiều skills để đạt được một thành quả cụ thể.

- Các **Bundles** chỉ điểm cho bạn biết những skills nào là mang tính liên quan thiết thực đối với một vai trò nhất định.
- Các **Workflows** chỉ dẫn bạn cách sử dụng chính những skills đó theo một trình tự cụ thể để hoàn thành một mục tiêu thực tiễn.

Nếu ví bundles như là hộp đồ nghề (toolbox) của bạn, thì workflows chính là cẩm nang thực thi (execution playbook) của bạn lúc làm việc.

---

## Cách sử dụng Workflows

1. Cài đặt repository một lần duy nhất (`npx antigravity-awesome-skills`).
2. Lựa chọn một workflow tương ứng với mục tiêu trước mắt của bạn.
3. Thực thi các bước theo đúng trình tự và tiến hành gọi (invoke) các skills được liệt kê trong từng bước tương ứng.
4. Lưu giữ lại các kết quả đầu ra (output artifacts) ở mỗi một bước (chẳng hạn như bản plan, các decisions, các files tests và validation evidence).

Bạn hoàn toàn có thể kết hợp các workflows với các bundles từ file [BUNDLES.md](BUNDLES.md) khi bạn mong muốn mức độ bao phủ (coverage) rộng hơn.

---

## Workflow: Ra mắt một MVP dạng SaaS (Ship a SaaS MVP)

Xây dựng và tiến hành ra mắt (ship) một sản phẩm SaaS nhỏ gọn nhưng được định hướng đạt chuẩn môi trường production.

**Các bundles liên quan:** `Essentials`, `Full-Stack Developer`, `QA & Testing`, `DevOps & Cloud`

### Các điều kiện báo trước (Prerequisites)

- Kho repository ở local và runtime đã được cấu hình ổn định.
- Xác định rõ ràng bài toán (problem) của user và phạm vi (scope) của MVP.
- Đã chọn lựa một mục tiêu deployment cơ bản.

### Các Bước (Steps)

1. **Hoạch định phạm vi (Plan the scope)**
   - **Mục tiêu (Goal):** Định nghĩa các đường ranh giới của MVP và các tiêu chí chấp thuận (acceptance criteria).
   - **Skills:** [`@brainstorming`](../skills/brainstorming/), [`@concise-planning`](../skills/concise-planning/), [`@writing-plans`](../skills/writing-plans/)
   - **Ví dụ về Prompt:** `Sử dụng @concise-planning để xác định các milestones và acceptance criteria cho MVP SaaS của tôi.`

2. **Xây dựng backend và API (Build backend and API)**
   - **Mục tiêu (Goal):** Implement các thực thể (entities) cốt lõi, APIs, và thiết lập đường cơ sở bảo mật (auth baseline).
   - **Skills:** [`@backend-dev-guidelines`](../skills/backend-dev-guidelines/), [`@api-patterns`](../skills/api-patterns/), [`@database-design`](../skills/database-design/)
   - **Ví dụ về Prompt:** `Sử dụng @backend-dev-guidelines để tạo ra các API và services cho lĩnh vực thanh toán (billing domain).`

3. **Xây dựng frontend (Build frontend)**
   - **Mục tiêu (Goal):** Ra mắt luồng người dùng (user flow) cốt lõi bám sát cùng các trạng thái (states) UX biểu hiện rõ ràng.
   - **Skills:** [`@frontend-developer`](../skills/frontend-developer/), [`@react-patterns`](../skills/react-patterns/), [`@frontend-design`](../skills/frontend-design/)
   - **Ví dụ về Prompt:** `Sử dụng @frontend-developer để implement phần onboarding, giao diện trạng thái rỗng (empty state) và dashboard sơ khởi.`

4. **Kiểm tra và Xác thực (Test and validate)**
   - **Mục tiêu (Goal):** Bao phủ việc kiểm thử các kịch bản hành trình user (user journeys) có cấp độ đặc biệt nghiêm trọng trước khi bắt tay thực hiện release.
   - **Skills:** [`@test-driven-development`](../skills/test-driven-development/), [`@browser-automation`](../skills/browser-automation/), `@go-playwright` (tùy chọn, đối với bộ stack Go)
   - **Ví dụ về Prompt:** `Sử dụng @browser-automation để tạo lập các bài kiểm tra E2E test cho tiến trình luồng signup và checkout.`
   - **Lưu ý đối với Go:** Nếu dự án QA và các công cụ (tooling) đang dùng Go, hãy ưu tiên dùng `@go-playwright`.

5. **Release An Toàn (Ship safely)**
   - **Mục tiêu (Goal):** Release hệ thống đi liền với quy chuẩn khả năng quan sát (observability) và lên lộ trình dự phòng sẵn kế hoạch rollback.
   - **Skills:** [`@deployment-procedures`](../skills/deployment-procedures/), [`@observability-engineer`](../skills/observability-engineer/)
   - **Ví dụ về Prompt:** `Sử dụng @deployment-procedures tạo thành một checklist dành riêng cho các quy trình release có bao gồm kế hoạch rollback.`

---

## Workflow: Rà soát Bảo mật cho một Ứng dụng Web (Security Audit for a Web App)

Chạy đợt kiểm tra quy chuẩn chuyên sâu về Security từ khâu ấn định khoanh vùng phạm vi (scope) cho đến khâu quy chiếu hiệu năng ngăn chặn hệ lụy (remediation validation).

**Các bundles liên quan:** `Security Engineer`, `Security Developer`, `Observability & Monitoring`

### Các điều kiện báo trước (Prerequisites)

- Nhận được uỷ quyền rõ ràng (explicit authorization) dùng cho mục đích testing.
- Tài liệu quy định danh sách các điểm mục tiêu thuộc vùng In-scope đã được soạn sẵn.
- Trích lập được dữ liệu thông tin về Logging cùng yếu tố môi trường (environment details).

### Các Bước (Steps)

1. **Khắc suất Phạm vi & Tiến hành Mô hình hóa Mối đe dọa (Define scope and threat model)**
   - **Mục tiêu (Goal):** Định vị ra các loại tài nguyên giá trị (assets), các đường ranh giới mức độ tín nhiệm (trust boundaries), và thiết lập các sơ đồ tấn công (attack paths).
   - **Skills:** [`@ethical-hacking-methodology`](../skills/ethical-hacking-methodology/), [`@threat-modeling-expert`](../skills/threat-modeling-expert/), [`@attack-tree-construction`](../skills/attack-tree-construction/)
   - **Ví dụ về Prompt:** `Sử dụng @threat-modeling-expert để vẽ nên bản đồ đường bộ đối với nhóm tài nguyên (assets) có tính critical và trust boundaries trong web app của tôi.`

2. **Rà soát chứng thực quyền danh và cơ chế điều hướng truy cập (Review auth and access control)**
   - **Mục tiêu (Goal):** Săn tìm thủ thuật đánh cắp tài khoản (account takeover) và những khiếm khuyết phát sinh ở khâu phân quyền.
   - **Skills:** [`@broken-authentication`](../skills/broken-authentication/), [`@auth-implementation-patterns`](../skills/auth-implementation-patterns/), [`@idor-testing`](../skills/idor-testing/)
   - **Ví dụ về Prompt:** `Sử dụng @idor-testing nhằm tiến hành rà xác minh hiện trạng có hay không những loại truy cập trái thẩm quyền trên các cấu trúc nền tảng API endpoint đa ứng dụng (multitenant).`

3. **Thẩm định Bảo mật Dữ liệu đầu vào và Kết cấu API (Assess API and input security)**
   - **Mục tiêu (Goal):** Trích xuất vạch trần những lỗ hổng dạng injection hoặc từ các tầng API ở mức đe dọa tổn thương cao.
   - **Skills:** [`@api-security-best-practices`](../skills/api-security-best-practices/), [`@api-fuzzing-bug-bounty`](../skills/api-fuzzing-bug-bounty/), [`@top-web-vulnerabilities`](../skills/top-web-vulnerabilities/)
   - **Ví dụ về Prompt:** `Sử dụng @api-security-best-practices để audit trọn bộ cấu trúc mạng endpoint của bên auth, trang thiết lập billing và quyền chức admin.`

4. **Biện pháp Phòng ngừa và Rà soát đối chiếu (Harden and verify)**
   - **Mục tiêu (Goal):** Chuyển hóa toàn bộ thành tích rà soát (findings) trở thành các bản sửa lỗi (fixes) cũng như xác thực các vật chứng của công tác giảm thiểu (mitigation).
   - **Skills:** [`@security-auditor`](../skills/security-auditor/), [`@sast-configuration`](../skills/sast-configuration/), [`@verification-before-completion`](../skills/verification-before-completion/)
   - **Ví dụ về Prompt:** `Sử dụng @verification-before-completion để ra sát hạch và xác nhận rằng hệ thống phương án mitigation nay đã đi vào hoạt động trơn tru.`

---

## Workflow: Xây dựng Một Mạng Lưới AI Agent (Build an AI Agent System)

Thiết kế ra quy mô và đi vào vận hành một con agent đạt chuẩn production-grade mang theo độ tin cậy được thể hiện thành các số lượng đong đếm.

**Các bundles liên quan:** `Agent Architect`, `LLM Application Developer`, `Data Engineering`

### Các điều kiện báo trước (Prerequisites)

- Chọn một định dạng Use case khoanh vùng cực hẹp (Narrow use case) song hành với kết quả (outcomes) mà vẫn đảm bảo có thể đong đo cân đếm.
- Cắp theo khóa quyền truy xuất vào nhà cung cấp API Model (model provider) và kèm lấy phương tiện quản lý bằng hệ thống quan sát hệ điều hành (observability tooling).
- Xây dọn trước sẵn lấy một hệ sinh thái Cơ sở Dữ liệu mẫu Dataset ban sơ hay cẩm nang nền tảng Knowledge corpus.

### Các Bước (Steps)

1. **Kháng định Hành vi Mục tiêu tiêu chuẩn và Hệ KPIs (Define target behavior and KPIs)**
   - **Mục tiêu (Goal):** Tranh thủ đánh mốc đặt đường cơ sở các ngưỡng của chất lượng tiến hóa (quality), trị số trễ phát dữ liệu (latency), kèm luôn định lý trị số chệch cự ly sập luồng khống chế (failure thresholds) rủi ro.
   - **Skills:** [`@ai-agents-architect`](../skills/ai-agents-architect/), [`@agent-evaluation`](../skills/agent-evaluation/), [`@product-manager-toolkit`](../skills/product-manager-toolkit/)
   - **Ví dụ về Prompt:** `Sử dụng @agent-evaluation giúp tui quy định mức benchmark giới hạn và xác suất bảng tiêu chí nhận diện cái mức độ thành công đối của bé agent.`

2. **Dàn Kiến trúc Trích Lọc và Ổ Nhớ Cache (Design retrieval and memory)**
   - **Mục tiêu (Goal):** Xây dựng nên kết cấu hạ tầng lưu lưu của hệ nền trích vấn điểm tệp truy cứu thông tin (retrieval) lẫn với context architecture cứng cựa.
   - **Skills:** [`@llm-app-patterns`](../skills/llm-app-patterns/), [`@rag-implementation`](../skills/rag-implementation/), [`@vector-database-engineer`](../skills/vector-database-engineer/)
   - **Ví dụ về Prompt:** `Sử dụng @rag-implementation thiết kế vẽ dàn pipeline xử lý tác vụ chunking, nhét cả embedding, và lấy cái cữ retrieval.`

3. **Củng Cố Cấu Cơ Cấu Quản Trị Hệ Điều Hành Máy (Implement orchestration)**
   - **Mục tiêu (Goal):** Đồng bộ cấu kết vận hành Implement mạng luới xử lý có rào chắn cố định (deterministic orchestration) cộng gộp cơ số quy cách phân tuyến viền ngoài danh giới công cụ sử dụng (tool boundaries).
   - **Skills:** [`@langgraph`](../skills/langgraph/), [`@mcp-builder`](../skills/mcp-builder/), [`@workflow-automation`](../skills/workflow-automation/)
   - **Ví dụ về Prompt:** `Sử dụng @langgraph để implement cái luồng biểu đồ agent graph với sự cơ chế tự fallback (chuyển phòng vệ an toàn) cùng kèm bộ can hiệp ngắt quản ở nút nhạy bén human-in-the-loop.`

4. **Sát Sạch Thẩm Định Năng Kiểm Đảo Chéo (Evaluate and iterate)**
   - **Mục tiêu (Goal):** Lấp che gia tăng củng cố lôi vớt chỗ nào bị weak points nhờ một mạng khung quy trình quay vòng tuần hoàn hệ thống (structured loop).
   - **Skills:** [`@agent-evaluation`](../skills/agent-evaluation/), [`@langfuse`](../skills/langfuse/), [`@kaizen`](../skills/kaizen/)
   - **Ví dụ về Prompt:** `Sử dụng @kaizen nhằm gài thứ tự sự ưu tiên (prioritize) dành dọn các vết chỉnh đọng cấu hình cho số hỏng mớ failure modes trích lấy lộ xuất bên sau khi thực nghiệm báo bài qua test.`

---

## Workflow: Công tác QA và Bộ Tự vận hành trên Trình Duyệt (QA and Browser Automation)

Tạo lập khả năng tự vận hành Browser automation chống chịu tốt kết hợp phương thức tính trước quy trình có rào quy củ cứng deterministic execution thông phần đường luồng ống dẫn của bộ CI.

**Các bundles liên quan:** `QA & Testing`, `Full-Stack Developer`

### Các điều kiện báo trước (Prerequisites)

- Đã sẵn đầy các hạng mục test environments cùng nguồn dẫn chốt truy khóa bảo tồn (stable credentials).
- Những trạm luồng lưu dẫn đi qua đường user journeys trọng tâm cốt yếu nào đã được soi kiểm đích thị định tính sẵn trơn.
- Dàn cữ cho CI pipeline đã sẵn sàng xài rải.

### Các Bước (Steps)

1. **Chuẩn bị Dọn Chiến Lược Quy Cách Ngầm Cho Testing (Prepare test strategy)**
   - **Mục tiêu (Goal):** Nhồi quy phạm lên những vùng journeys, móc rào dữ kiện đi kèm (fixtures), và các khoanh vùng cảnh giới mô phỏng (execution environments).
   - **Skills:** [`@e2e-testing-patterns`](../skills/e2e-testing-patterns/), [`@test-driven-development`](../skills/test-driven-development/)
   - **Ví dụ về Prompt:** `Sử dụng @e2e-testing-patterns nhằm định tính ấn định cho xuất ra tập suite dành test mã bộ E2E dù tinh nhã thu gọn nhất có thể cơ mà rặt gắt trúng đích đem lại cao hiệu chấn mức ảnh hưởng (high impact).`

2. **Áp Khung Cho Browser Tests Hoạt Động (Implement browser tests)**
   - **Mục tiêu (Goal):** Xây đúc một thảm cấu trúc test coverage phòng bị bọc vách chắc chắn mạnh bạo trọn dùng hệ phím điều hướng ổn định cứng cáp (stable selectors).
   - **Skills:** [`@browser-automation`](../skills/browser-automation/), `@go-playwright` (có cũng được không ép (optional), chèn theo cho mảng công cụ Go stack)
   - **Ví dụ về Prompt:** `Sử dụng @go-playwright cho chu kì implement khả năng browser automation thả phịch vô nội dung 1 project lác đác đang nằm với Go.`

3. **Thanh Lọc Lại và Cô Tinh Nhám Dọn Lỗ Đen (Triage and harden)**
   - **Mục tiêu (Goal):** Dọn chùi nạo tận phế thái thứ cái lũ nết xấu của cấu đồ vận trớn chập chờn (flaky behavior) và trói chốt cứng ngắt áp khả năng giữ tính đồng lập (repeatability).
   - **Skills:** [`@systematic-debugging`](../skills/systematic-debugging/), [`@test-fixing`](../skills/test-fixing/), [`@verification-before-completion`](../skills/verification-before-completion/)
   - **Ví dụ về Prompt:** `Sử dụng @systematic-debugging đánh rới nhãn phân phân khu vực và đi giải vây xé rào đánh lùng cho nát dọn gọn êm nhẹm thứ flakiness nhục trân hay xuất phát đe trễ nơi ngục ải CI kia.`

---

## Tính Chất Máy Quét Mở Đọc Của Cả Ngành Lũ Workflows (Machine-Readable Workflows)

Đối đãi mục đích nắn chỉnh của tooling (công cụ mọt mã chế tạo) và cấu hình nướng tự hành (automation), mọi siêu luồng mảng tin của data trong gốc rễ mang danh metadata được đặt trải lộ hẳn cho bên tại đây luôn ở [data/workflows.json](../data/workflows.json).
