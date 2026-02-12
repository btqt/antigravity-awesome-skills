---
name: antigravity-workflows
description: "Điều phối nhiều kỹ năng Antigravity thông qua các quy trình làm việc được hướng dẫn để phân phối SaaS MVP, kiểm toán bảo mật, xây dựng tác nhân AI và QA trình duyệt."
source: self
risk: none
---

# Quy trình làm việc Antigravity

Sử dụng kỹ năng này để biến một mục tiêu phức tạp thành một chuỗi các lời gọi kỹ năng được hướng dẫn.

## Khi nào nên sử dụng kỹ năng này

Sử dụng kỹ năng này khi:

- Người dùng muốn kết hợp nhiều kỹ năng mà không cần chọn thủ công từng kỹ năng.
- Mục tiêu bao gồm nhiều giai đoạn (ví dụ: lập kế hoạch, xây dựng, kiểm thử, vận chuyển).
- Người dùng yêu cầu thực hiện theo thực hành tốt nhất cho các kịch bản phổ biến như:
  - Vận chuyển một SaaS MVP
  - Chạy kiểm toán bảo mật web
  - Xây dựng hệ thống tác nhân AI
  - Triển khai tự động hóa trình duyệt và E2E QA

## Nguồn sự thật của Quy trình làm việc

Đọc các quy trình làm việc theo thứ tự này:

1. `docs/WORKFLOWS.md` cho các sách hướng dẫn dễ đọc cho con người.
2. `data/workflows.json` cho dữ liệu meta quy trình làm việc có thể đọc được bằng máy.

## Cách chạy kỹ năng này

1. Xác định kết quả cụ thể của người dùng.
2. Đề xuất 1-2 quy trình làm việc phù hợp nhất.
3. Yêu cầu người dùng chọn một.
4. Thực hiện từng bước:
   - Thông báo bước hiện tại và tạo tác (artifact) mong đợi.
   - Gọi các kỹ năng được đề xuất cho bước đó.
   - Xác minh các tiêu chí hoàn thành trước khi chuyển sang bước tiếp theo.
5. Khi kết thúc, cung cấp:
   - Các tạo tác đã hoàn thành
   - Bằng chứng xác thực
   - Các rủi ro còn lại và hành động tiếp theo

## Định tuyến Quy trình làm việc Mặc định

- Yêu cầu phân phối sản phẩm -> `ship-saas-mvp`
- Yêu cầu đánh giá bảo mật -> `security-audit-web-app`
- Yêu cầu sản phẩm Agent/LLM -> `build-ai-agent-system`
- Yêu cầu kiểm thử E2E/trình duyệt -> `qa-browser-automation`

## Các lời nhắc Sao chép-Dán

```text
Use @antigravity-workflows to run the "Ship a SaaS MVP" workflow for my project idea.
```

```text
Use @antigravity-workflows and execute a full "Security Audit for a Web App" workflow.
```

```text
Use @antigravity-workflows to guide me through "Build an AI Agent System" with checkpoints.
```

```text
Use @antigravity-workflows to execute the "QA and Browser Automation" workflow and stabilize flaky tests.
```

## Hạn chế

- Kỹ năng này điều phối; nó không thay thế các kỹ năng chuyên biệt.
- Nó phụ thuộc vào sự sẵn có cục bộ của các kỹ năng được tham chiếu.
- Nó không đảm bảo thành công nếu không có quyền truy cập môi trường, thông tin đăng nhập hoặc cơ sở hạ tầng cần thiết.
- Đối với tự động hóa trình duyệt cụ thể cho stack bằng Go, `go-playwright` có thể yêu cầu kỹ năng tương ứng phải có trong kho kỹ năng cục bộ của bạn.

## Các kỹ năng liên quan

- `concise-planning`
- `brainstorming`
- `workflow-automation`
- `verification-before-completion`
