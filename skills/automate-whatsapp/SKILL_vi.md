---
name: automate-whatsapp
description: "Xây dựng các hệ thống tự động hóa WhatsApp với Kapso workflows: cấu hình các trigger WhatsApp, chỉnh sửa đồ thị quy trình công việc (workflow graphs), quản lý việc thực thi, triển khai các hàm, và sử dụng cơ sở dữ liệu/tích hợp cho trạng thái. Sử dụng khi tự động hóa các cuộc hội thoại WhatsApp và xử lý sự kiện."
source: "https://github.com/gokapso/agent-skills/tree/master/skills/automate-whatsapp"
risk: safe
---

# Tự động hóa WhatsApp (Automate WhatsApp)

## Khi nào sử dụng

Sử dụng kỹ năng này để xây dựng và chạy các hệ thống tự động hóa WhatsApp: CRUD quy trình công việc, chỉnh sửa đồ thị, các trigger, thực thi, quản lý hàm, tích hợp ứng dụng, và các hoạt động cơ sở dữ liệu D1.

## Thiết lập

Các biến môi trường:

- `KAPSO_API_BASE_URL` (chỉ host, không có `/platform/v1`)
- `KAPSO_API_KEY`

## Cách thực hiện

### Chỉnh sửa đồ thị quy trình công việc (workflow graph)

1. Lấy đồ thị: `node scripts/get-graph.js <workflow_id>` (lưu ý `lock_version`)
2. Chỉnh sửa JSON (xem các quy tắc đồ thị bên dưới)
3. Xác thực: `node scripts/validate-graph.js --definition-file <path>`
4. Cập nhật: `node scripts/update-graph.js <workflow_id> --expected-lock-version <n> --definition-file <path>`
5. Lấy lại để xác nhận

Đối với các chỉnh sửa nhỏ, hãy sử dụng `edit-graph.js` với `--old-file` và `--new-file` thay thế.

Nếu bạn gặp xung đột `lock_version`: hãy lấy lại, áp dụng lại các thay đổi, thử lại với `lock_version` mới.

### Quản lý các trigger

1. Liệt kê: `node scripts/list-triggers.js <workflow_id>`
2. Tạo: `node scripts/create-trigger.js <workflow_id> --trigger-type <type> --phone-number-id <id>`
3. Bật/Tắt: `node scripts/update-trigger.js --trigger-id <id> --active true|false`
4. Xóa: `node scripts/delete-trigger.js --trigger-id <id>`

Đối với các trigger `inbound_message`, trước tiên hãy chạy `node scripts/list-whatsapp-phone-numbers.js` để lấy `phone_number_id`.

### Debug việc thực thi

1. Liệt kê: `node scripts/list-executions.js <workflow_id>`
2. Kiểm tra: `node scripts/get-execution.js <execution-id>`
3. Lấy giá trị: `node scripts/get-context-value.js <execution-id> --variable-path vars.foo`
4. Sự kiện: `node scripts/list-execution-events.js <execution-id>`

### Tạo và triển khai một hàm (function)

1. Viết mã với chữ ký handler (xem quy tắc hàm bên dưới)
2. Tạo: `node scripts/create-function.js --name <name> --code-file <path>`
3. Triển khai: `node scripts/deploy-function.js --function-id <id>`
4. Xác minh: `node scripts/get-function.js --function-id <id>`

### Thiết lập nút agent với tích hợp ứng dụng

1. Tìm mô hình: `node scripts/list-provider-models.js`
2. Tìm tài khoản: `node scripts/list-accounts.js --app-slug <slug>` (sử dụng `pipedream_account_id`)
3. Tìm hành động: `node scripts/search-actions.js --query <word> --app-slug <slug>` (action_id = key)
4. Tạo tích hợp: `node scripts/create-integration.js --action-id <id> --app-slug <slug> --account-id <id> --configured-props <json>`
5. Thêm các công cụ vào nút agent thông qua `flow_agent_app_integration_tools`

### CRUD Cơ sở dữ liệu

1. Liệt kê các bảng: `node scripts/list-tables.js`
2. Truy vấn: `node scripts/query-rows.js --table <name> --filters <json>`
3. Tạo/cập nhật/xóa bằng các script dòng (row scripts)

## Quy tắc Đồ thị (Graph rules)

- Chính xác một nút bắt đầu với `id` = `start`
- Không bao giờ thay đổi ID của các nút hiện có
- Sử dụng `{node_type}_{timestamp_ms}` cho ID các nút mới
- Các nút không phải nút quyết định (Non-decide nodes) có 0 hoặc 1 cạnh `next` đi ra
- Các nhãn cạnh quyết định phải khớp với `conditions[].label`
- Các khóa của cạnh là `source`/`target`/`label` (không phải `from`/`to`)

Để biết chi tiết đầy đủ về lược đồ (schema), hãy xem `references/graph-contract.md`.

## Quy tắc Hàm (Function rules)

```js
async function handler(request, env) {
  // Phân tích đầu vào
  const body = await request.json();
  // Sử dụng env.KV và env.DB khi cần
  return new Response(JSON.stringify({ result: "ok" }));
}
```

- KHÔNG sử dụng `export`, `export default`, hoặc các hàm arrow (arrow functions)
- Trả về một đối tượng `Response`

## Ngữ cảnh Thực thi (Execution context)

Luôn sử dụng cấu trúc này:

- `vars` - các biến do người dùng định nghĩa
- `system` - các biến hệ thống
- `context` - dữ liệu kênh
- `metadata` - siêu dữ liệu yêu cầu

## Các Scripts

### Quy trình công việc (Workflows)

| Script                        | Mục đích                                           |
| ----------------------------- | -------------------------------------------------- |
| `list-workflows.js`           | Liệt kê các quy trình công việc (chỉ siêu dữ liệu) |
| `get-workflow.js`             | Lấy siêu dữ liệu quy trình công việc               |
| `create-workflow.js`          | Tạo một quy trình công việc                        |
| `update-workflow-settings.js` | Cập nhật cài đặt quy trình công việc               |

### Đồ thị (Graph)

| Script              | Mục đích                                      |
| ------------------- | --------------------------------------------- |
| `get-graph.js`      | Lấy đồ thị quy trình công việc + lock_version |
| `edit-graph.js`     | Sửa vá đồ thị thông qua thay thế chuỗi        |
| `update-graph.js`   | Thay thế toàn bộ đồ thị                       |
| `validate-graph.js` | Xác thực cấu trúc đồ thị cục bộ               |

### Các Trigger

| Script                           | Mục đích                                        |
| -------------------------------- | ----------------------------------------------- |
| `list-triggers.js`               | Liệt kê các trigger cho một quy trình công việc |
| `create-trigger.js`              | Tạo một trigger                                 |
| `update-trigger.js`              | Bật/tắt một trigger                             |
| `delete-trigger.js`              | Xóa một trigger                                 |
| `list-whatsapp-phone-numbers.js` | Liệt kê các số điện thoại để thiết lập trigger  |

### Việc thực thi (Executions)

| Script                       | Mục đích                         |
| ---------------------------- | -------------------------------- |
| `list-executions.js`         | Liệt kê việc thực thi            |
| `get-execution.js`           | Lấy chi tiết thực thi            |
| `get-context-value.js`       | Đọc giá trị từ ngữ cảnh thực thi |
| `update-execution-status.js` | Ép buộc trạng thái thực thi      |
| `resume-execution.js`        | Tiếp tục việc thực thi đang chờ  |
| `list-execution-events.js`   | Liệt kê các sự kiện thực thi     |

### Các Hàm (Functions)

| Script                         | Mục đích                        |
| ------------------------------ | ------------------------------- |
| `list-functions.js`            | Liệt kê các hàm của dự án       |
| `get-function.js`              | Lấy chi tiết hàm + mã nguồn     |
| `create-function.js`           | Tạo một hàm                     |
| `update-function.js`           | Cập nhật mã nguồn hàm           |
| `deploy-function.js`           | Triển khai hàm vào runtime      |
| `invoke-function.js`           | Gọi hàm với tải trọng (payload) |
| `list-function-invocations.js` | Liệt kê các lần gọi hàm         |

### Tích hợp ứng dụng (App integrations)

| Script                    | Mục đích                                 |
| ------------------------- | ---------------------------------------- |
| `list-apps.js`            | Tìm kiếm các ứng dụng tích hợp           |
| `search-actions.js`       | Tìm kiếm các hành động (action_id = key) |
| `get-action-schema.js`    | Lấy lược đồ JSON của hành động           |
| `list-accounts.js`        | Liệt kê các tài khoản đã kết nối         |
| `create-connect-token.js` | Tạo liên kết kết nối OAuth               |
| `configure-prop.js`       | Giải quyết remote_options cho một prop   |
| `reload-props.js`         | Tải lại các prop động                    |
| `list-integrations.js`    | Liệt kê các tích hợp đã lưu              |
| `create-integration.js`   | Tạo một tích hợp                         |
| `update-integration.js`   | Cập nhật một tích hợp                    |
| `delete-integration.js`   | Xóa một tích hợp                         |

### Cơ sở dữ liệu (Databases)

| Script           | Mục đích                             |
| ---------------- | ------------------------------------ |
| `list-tables.js` | Liệt kê các bảng D1                  |
| `get-table.js`   | Lấy lược đồ bảng + các dòng mẫu      |
| `query-rows.js`  | Truy vấn các dòng với bộ lọc         |
| `create-row.js`  | Tạo một dòng                         |
| `update-row.js`  | Cập nhật các dòng                    |
| `upsert-row.js`  | Cập nhật hoặc chèn (upsert) một dòng |
| `delete-row.js`  | Xóa các dòng                         |

### OpenAPI

| Script                | Mục đích                                  |
| --------------------- | ----------------------------------------- |
| `openapi-explore.mjs` | Khám phá OpenAPI (search/op/schema/where) |

Cài đặt các thành phần phụ thuộc (một lần):

```bash
npm i
```

Các ví dụ:

```bash
node scripts/openapi-explore.mjs --spec workflows search "variables"
node scripts/openapi-explore.mjs --spec workflows op getWorkflowVariables
node scripts/openapi-explore.mjs --spec platform op queryDatabaseRows
```

## Lưu ý

- Ưu tiên các đường dẫn tệp hơn JSON nội dòng (`--definition-file`, `--code-file`)
- `action_id` giống với `key` từ `search-actions`
- `--account-id` sử dụng `pipedream_account_id` từ `list-accounts`
- Việc biến CRUD (`variables-set.js`, `variables-delete.js`) bị chặn - Platform API không hỗ trợ nó
- Việc thực thi SQL thô không được hỗ trợ thông qua Platform API

## Tham khảo

Đọc trước khi chỉnh sửa:

- [references/graph-contract.md](references/graph-contract.md) - Lược đồ đồ thị, các trường được tính toán so với các trường có thể chỉnh sửa, lock_version
- [references/node-types.md](references/node-types.md) - Các loại nút và hình dáng cấu hình
- [references/workflow-overview.md](references/workflow-overview.md) - Luồng thực thi và các trạng thái

Các tham khảo khác:

- [references/execution-context.md](references/execution-context.md) - Cấu trúc ngữ cảnh và thay thế biến
- [references/triggers.md](references/triggers.md) - Các loại trigger và thiết lập
- [references/app-integrations.md](references/app-integrations.md) - Tích hợp ứng dụng và variable_definitions
- [references/functions-reference.md](references/functions-reference.md) - Quản lý hàm
- [references/functions-payloads.md](references/functions-payloads.md) - Hình dáng tải trọng cho các hàm
- [references/databases-reference.md](references/databases-reference.md) - Các hoạt động cơ sở dữ liệu

## Các Tài sản (Assets)

| Tệp                                                 | Mô tả                                    |
| --------------------------------------------------- | ---------------------------------------- |
| `workflow-linear.json`                              | Quy trình công việc tuyến tính tối thiểu |
| `workflow-decision.json`                            | Quy trình công việc phân nhánh tối thiểu |
| `workflow-agent-simple.json`                        | Quy trình công việc agent tối thiểu      |
| `workflow-customer-support-intake-agent.json`       | Agent tiếp nhận hỗ trợ khách hàng        |
| `workflow-interactive-buttons-decide-function.json` | Các nút tương tác + quyết định (hàm)     |
| `workflow-interactive-buttons-decide-ai.json`       | Các nút tương tác + quyết định (AI)      |
| `workflow-api-template-wait-agent.json`             | API trigger + template + agent           |
| `function-decide-route-interactive-buttons.json`    | Hàm cho việc điều hướng nút              |
| `agent-app-integration-example.json`                | Nút agent với tích hợp ứng dụng          |

## Các kỹ năng liên quan

- `integrate-whatsapp` - Onboarding, webhooks, nhắn tin, mẫu (templates), luồng (flows)
- `observe-whatsapp` - Debugging, logs, kiểm tra sức khỏe (health checks)

<!-- FILEMAP:BEGIN -->

```text
[automate-whatsapp file map]|root: .
|.:{package.json,SKILL.md}
|assets:{agent-app-integration-example.json,databases-example.json,function-decide-route-interactive-buttons.json,functions-example.json,workflow-agent-simple.json,workflow-api-template-wait-agent.json,workflow-customer-support-intake-agent.json,workflow-decision.json,workflow-interactive-buttons-decide-ai.json,workflow-interactive-buttons-decide-function.json,workflow-linear.json}
|references:{app-integrations.md,databases-reference.md,execution-context.md,function-contracts.md,functions-payloads.md,functions-reference.md,graph-contract.md,node-types.md,triggers.md,workflow-overview.md,workflow-reference.md}
|scripts:{configure-prop.js,create-connect-token.js,create-function.js,create-integration.js,create-row.js,create-trigger.js,create-workflow.js,delete-integration.js,delete-row.js,delete-trigger.js,deploy-function.js,edit-graph.js,get-action-schema.js,get-context-value.js,get-execution-event.js,get-execution.js,get-function.js,get-graph.js,get-table.js,get-workflow.js,invoke-function.js,list-accounts.js,list-apps.js,list-execution-events.js,list-executions.js,list-function-invocations.js,list-functions.js,list-integrations.js,list-provider-models.js,list-tables.js,list-triggers.js,list-whatsapp-phone-numbers.js,list-workflows.js,openapi-explore.mjs,query-rows.js,reload-props.js,resume-execution.js,search-actions.js,update-execution-status.js,update-function.js,update-graph.js,update-integration.js,update-row.js,update-trigger.js,update-workflow-settings.js,upsert-row.js,validate-graph.js,variables-delete.js,variables-list.js,variables-set.js}
|scripts/lib/databases:{args.js,filters.js,kapso-api.js}
|scripts/lib/functions:{args.js,kapso-api.js}
|scripts/lib/workflows:{args.js,kapso-api.js,result.js}
```

<!-- FILEMAP:END -->
