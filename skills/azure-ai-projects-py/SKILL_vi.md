---
name: azure-ai-projects-py
description: Xây dựng các ứng dụng AI sử dụng Azure AI Projects Python SDK (azure-ai-projects). Sử dụng khi làm việc với các client dự án Foundry, tạo agent có phiên bản với PromptAgentDefinition, chạy đánh giá, quản lý kết nối/triển khai/bộ dữ liệu/chỉ mục, hoặc sử dụng các client tương thích OpenAI. Đây là SDK Foundry cấp cao - cho các hoạt động agent cấp thấp, sử dụng skill azure-ai-agents-python.
package: azure-ai-projects
---

# Azure AI Projects Python SDK (Foundry SDK)

Xây dựng các ứng dụng AI trên Microsoft Foundry sử dụng SDK `azure-ai-projects`.

## Cài đặt

```bash
pip install azure-ai-projects azure-identity
```

## Biến Môi trường

```bash
AZURE_AI_PROJECT_ENDPOINT="https://<resource>.services.ai.azure.com/api/projects/<project>"
AZURE_AI_MODEL_DEPLOYMENT_NAME="gpt-4o-mini"
```

## Xác thực

```python
import os
from azure.identity import DefaultAzureCredential
from azure.ai.projects import AIProjectClient

credential = DefaultAzureCredential()
client = AIProjectClient(
    endpoint=os.environ["AZURE_AI_PROJECT_ENDPOINT"],
    credential=credential,
)
```

## Tổng quan về Hoạt động Client

| Hoạt động            | Truy cập         | Mục đích                            |
| -------------------- | ---------------- | ----------------------------------- |
| `client.agents`      | `.agents.*`      | CRUD Agent, versions, threads, runs |
| `client.connections` | `.connections.*` | Liệt kê/lấy các kết nối dự án       |
| `client.deployments` | `.deployments.*` | Liệt kê triển khai mô hình          |
| `client.datasets`    | `.datasets.*`    | Quản lý bộ dữ liệu                  |
| `client.indexes`     | `.indexes.*`     | Quản lý chỉ mục                     |
| `client.evaluations` | `.evaluations.*` | Chạy đánh giá                       |
| `client.red_teams`   | `.red_teams.*`   | Các hoạt động Red team              |

## Hai Cách tiếp cận Client

### 1. AIProjectClient (Native Foundry)

```python
from azure.ai.projects import AIProjectClient

client = AIProjectClient(
    endpoint=os.environ["AZURE_AI_PROJECT_ENDPOINT"],
    credential=DefaultAzureCredential(),
)

# Sử dụng các hoạt động gốc Foundry
agent = client.agents.create_agent(
    model=os.environ["AZURE_AI_MODEL_DEPLOYMENT_NAME"],
    name="my-agent",
    instructions="Bạn rất hữu ích.",
)
```

### 2. Client Tương thích OpenAI

```python
# Lấy client tương thích OpenAI từ dự án
openai_client = client.get_openai_client()

# Sử dụng API OpenAI tiêu chuẩn
response = openai_client.chat.completions.create(
    model=os.environ["AZURE_AI_MODEL_DEPLOYMENT_NAME"],
    messages=[{"role": "user", "content": "Xin chào!"}],
)
```

## Các Hoạt động Agent

### Tạo Agent (Cơ bản)

```python
agent = client.agents.create_agent(
    model=os.environ["AZURE_AI_MODEL_DEPLOYMENT_NAME"],
    name="my-agent",
    instructions="Bạn là một trợ lý hữu ích.",
)
```

### Tạo Agent với Công cụ

```python
from azure.ai.agents import CodeInterpreterTool, FileSearchTool

agent = client.agents.create_agent(
    model=os.environ["AZURE_AI_MODEL_DEPLOYMENT_NAME"],
    name="tool-agent",
    instructions="Bạn có thể thực thi mã và tìm kiếm tệp.",
    tools=[CodeInterpreterTool(), FileSearchTool()],
)
```

### Agent Có phiên bản với PromptAgentDefinition

```python
from azure.ai.projects.models import PromptAgentDefinition

# Tạo một agent có phiên bản
agent_version = client.agents.create_version(
    agent_name="customer-support-agent",
    definition=PromptAgentDefinition(
        model=os.environ["AZURE_AI_MODEL_DEPLOYMENT_NAME"],
        instructions="Bạn là một chuyên gia hỗ trợ khách hàng.",
        tools=[],  # Thêm công cụ khi cần
    ),
    version_label="v1.0",
)
```

Xem [references/agents.md](references/agents.md) để biết chi tiết các mẫu agent.

## Tổng quan về Công cụ

| Công cụ          | Lớp                       | Trường hợp Sử dụng             |
| ---------------- | ------------------------- | ------------------------------ |
| Code Interpreter | `CodeInterpreterTool`     | Thực thi Python, tạo tệp       |
| File Search      | `FileSearchTool`          | RAG trên tài liệu đã tải lên   |
| Bing Grounding   | `BingGroundingTool`       | Tìm kiếm web (yêu cầu kết nối) |
| Azure AI Search  | `AzureAISearchTool`       | Tìm kiếm chỉ mục của bạn       |
| Function Calling | `FunctionTool`            | Gọi các hàm Python của bạn     |
| OpenAPI          | `OpenApiTool`             | Gọi REST APIs                  |
| MCP              | `McpTool`                 | Máy chủ Model Context Protocol |
| Memory Search    | `MemorySearchTool`        | Tìm kiếm kho bộ nhớ agent      |
| SharePoint       | `SharepointGroundingTool` | Tìm kiếm nội dung SharePoint   |

Xem [references/tools.md](references/tools.md) để biết tất cả các mẫu công cụ.

## Luồng Thread và Message

```python
# 1. Tạo thread
thread = client.agents.threads.create()

# 2. Thêm tin nhắn
client.agents.messages.create(
    thread_id=thread.id,
    role="user",
    content="Thời tiết thế nào?",
)

# 3. Tạo và xử lý run
run = client.agents.runs.create_and_process(
    thread_id=thread.id,
    agent_id=agent.id,
)

# 4. Lấy phản hồi
if run.status == "completed":
    messages = client.agents.messages.list(thread_id=thread.id)
    for msg in messages:
        if msg.role == "assistant":
            print(msg.content[0].text.value)
```

## Kết nối (Connections)

```python
# Liệt kê tất cả kết nối
connections = client.connections.list()
for conn in connections:
    print(f"{conn.name}: {conn.connection_type}")

# Lấy kết nối cụ thể
connection = client.connections.get(connection_name="my-search-connection")
```

Xem [references/connections.md](references/connections.md) để biết các mẫu kết nối.

## Triển khai (Deployments)

```python
# Liệt kê các triển khai mô hình có sẵn
deployments = client.deployments.list()
for deployment in deployments:
    print(f"{deployment.name}: {deployment.model}")
```

Xem [references/deployments.md](references/deployments.md) để biết các mẫu triển khai.

## Bộ dữ liệu và Chỉ mục

```python
# Liệt kê bộ dữ liệu
datasets = client.datasets.list()

# Liệt kê chỉ mục
indexes = client.indexes.list()
```

Xem [references/datasets-indexes.md](references/datasets-indexes.md) để biết các hoạt động dữ liệu.

## Đánh giá (Evaluation)

```python
# Sử dụng OpenAI client cho evals
openai_client = client.get_openai_client()

# Tạo đánh giá với các bộ đánh giá tích hợp sẵn
eval_run = openai_client.evals.runs.create(
    eval_id="my-eval",
    name="quality-check",
    data_source={
        "type": "custom",
        "item_references": [{"item_id": "test-1"}],
    },
    testing_criteria=[
        {"type": "fluency"},
        {"type": "task_adherence"},
    ],
)
```

Xem [references/evaluation.md](references/evaluation.md) để biết các mẫu đánh giá.

## Client Bất đồng bộ (Async Client)

```python
from azure.ai.projects.aio import AIProjectClient

async with AIProjectClient(
    endpoint=os.environ["AZURE_AI_PROJECT_ENDPOINT"],
    credential=DefaultAzureCredential(),
) as client:
    agent = await client.agents.create_agent(...)
    # ... các hoạt động async
```

Xem [references/async-patterns.md](references/async-patterns.md) để biết các mẫu bất đồng bộ.

## Kho Bộ nhớ (Memory Stores)

```python
# Tạo kho bộ nhớ cho agent
memory_store = client.agents.create_memory_store(
    name="conversation-memory",
)

# Gắn vào agent để có bộ nhớ bền vững
agent = client.agents.create_agent(
    model=os.environ["AZURE_AI_MODEL_DEPLOYMENT_NAME"],
    name="memory-agent",
    tools=[MemorySearchTool()],
    tool_resources={"memory": {"store_ids": [memory_store.id]}},
)
```

## Thực hành Tốt nhất

1. **Sử dụng context managers** cho client async: `async with AIProjectClient(...) as client:`
2. **Dọn dẹp agent** khi xong: `client.agents.delete_agent(agent.id)`
3. **Sử dụng `create_and_process`** cho các run đơn giản, **streaming** cho UX thời gian thực
4. **Sử dụng agent có phiên bản** cho triển khai sản xuất
5. **Ưu tiên kết nối** cho tích hợp dịch vụ bên ngoài (AI Search, Bing, v.v.)

## So sánh SDK

| Tính năng          | `azure-ai-projects`     | `azure-ai-agents`      |
| ------------------ | ----------------------- | ---------------------- |
| Cấp độ             | Cấp cao (Foundry)       | Cấp thấp (Agents)      |
| Client             | `AIProjectClient`       | `AgentsClient`         |
| Phiên bản hóa      | `create_version()`      | Không có sẵn           |
| Kết nối            | Có                      | Không                  |
| Triển khai         | Có                      | Không                  |
| Bộ dữ liệu/Chỉ mục | Có                      | Không                  |
| Đánh giá           | Qua OpenAI client       | Không                  |
| Khi nào dùng       | Tích hợp đầy đủ Foundry | Ứng dụng agent độc lập |

## Tệp Tham khảo

- [references/agents.md](references/agents.md): Các hoạt động agent với PromptAgentDefinition
- [references/tools.md](references/tools.md): Tất cả các công cụ agent với ví dụ
- [references/evaluation.md](references/evaluation.md): Tổng quan về hoạt động đánh giá
- [references/built-in-evaluators.md](references/built-in-evaluators.md): Tham chiếu đầy đủ bộ đánh giá tích hợp
- [references/custom-evaluators.md](references/custom-evaluators.md): Các mẫu bộ đánh giá dựa trên mã và prompt
- [references/connections.md](references/connections.md): Các hoạt động kết nối
- [references/deployments.md](references/deployments.md): Liệt kê triển khai
- [references/datasets-indexes.md](references/datasets-indexes.md): Các hoạt động bộ dữ liệu và chỉ mục
- [references/async-patterns.md](references/async-patterns.md): Sử dụng client async
- [references/api-reference.md](references/api-reference.md): Tham chiếu API đầy đủ cho tất cả 373 SDK exports (v2.0.0b4)
- [scripts/run_batch_evaluation.py](scripts/run_batch_evaluation.py): Công cụ CLI cho đánh giá hàng loạt
