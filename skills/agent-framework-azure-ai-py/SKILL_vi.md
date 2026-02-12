---
name: agent-framework-azure-ai-py
description: Xây dựng các đại lý Azure AI Foundry bằng Microsoft Agent Framework Python SDK (agent-framework-azure-ai). Sử dụng khi tạo các đại lý bền vững với AzureAIAgentsProvider, sử dụng các công cụ được lưu trữ (trình thông dịch mã, tìm kiếm tệp, tìm kiếm web), tích hợp các máy chủ MCP, quản lý các luồng hội thoại hoặc triển khai các phản hồi phát trực tuyến (streaming). Bao gồm các công cụ hàm (function tools), đầu ra có cấu trúc và đại lý đa công cụ.
package: agent-framework-azure-ai
---

# Đại lý lưu trữ trên Azure với Agent Framework

Xây dựng các đại lý bền vững (persistent agents) trên Azure AI Foundry bằng Microsoft Agent Framework Python SDK.

## Kiến trúc

```
Câu hỏi của người dùng → AzureAIAgentsProvider → Azure AI Agent Service (Bền vững)
                    ↓
              Agent.run() / Agent.run_stream()
                    ↓
              Công cụ: Functions | Được lưu trữ (Code/Search/Web) | MCP
                    ↓
              AgentThread (lưu trữ hội thoại bền vững)
```

## Cài đặt

```bash
# Toàn bộ framework (khuyên dùng)
pip install agent-framework --pre

# Hoặc chỉ gói dành riêng cho Azure
pip install agent-framework-azure-ai --pre
```

## Biến môi trường

```bash
export AZURE_AI_PROJECT_ENDPOINT="https://<project>.services.ai.azure.com/api/projects/<project-id>"
export AZURE_AI_MODEL_DEPLOYMENT_NAME="gpt-4o-mini"
export BING_CONNECTION_ID="your-bing-connection-id"  # Dành cho tìm kiếm web
```

## Xác thực

```python
from azure.identity.aio import AzureCliCredential, DefaultAzureCredential

# Khi phát triển
credential = AzureCliCredential()

# Khi chạy thực tế (Production)
credential = DefaultAzureCredential()
```

## Quy trình công việc cốt lõi

### Đại lý Cơ bản

```python
import asyncio
from agent_framework.azure import AzureAIAgentsProvider
from azure.identity.aio import AzureCliCredential

async def main():
    async with (
        AzureCliCredential() as credential,
        AzureAIAgentsProvider(credential=credential) as provider,
    ):
        agent = await provider.create_agent(
            name="MyAgent",
            instructions="Bạn là một trợ lý hữu ích.",
        )

        result = await agent.run("Xin chào!")
        print(result.text)

asyncio.run(main())
```

### Đại lý với Công cụ Hàm (Function Tools)

```python
from typing import Annotated
from pydantic import Field
from agent_framework.azure import AzureAIAgentsProvider
from azure.identity.aio import AzureCliCredential

def get_weather(
    location: Annotated[str, Field(description="Tên thành phố để lấy thông tin thời tiết")],
) -> str:
    """Lấy thời tiết hiện tại cho một địa điểm."""
    return f"Thời tiết tại {location}: 72°F, có nắng"

def get_current_time() -> str:
    """Lấy thời gian UTC hiện tại."""
    from datetime import datetime, timezone
    return datetime.now(timezone.utc).strftime("%Y-%m-%d %H:%M:%S UTC")

async def main():
    async with (
        AzureCliCredential() as credential,
        AzureAIAgentsProvider(credential=credential) as provider,
    ):
        agent = await provider.create_agent(
            name="WeatherAgent",
            instructions="Bạn giúp trả lời các câu hỏi về thời tiết và thời gian.",
            tools=[get_weather, get_current_time],  # Truyền các hàm trực tiếp
        )

        result = await agent.run("Thời tiết ở Seattle thế nào?")
        print(result.text)
```

### Đại lý với Công cụ được Lưu trữ (Hosted Tools)

```python
from agent_framework import (
    HostedCodeInterpreterTool,
    HostedFileSearchTool,
    HostedWebSearchTool,
)
from agent_framework.azure import AzureAIAgentsProvider
from azure.identity.aio import AzureCliCredential

async def main():
    async with (
        AzureCliCredential() as credential,
        AzureAIAgentsProvider(credential=credential) as provider,
    ):
        agent = await provider.create_agent(
            name="MultiToolAgent",
            instructions="Bạn có thể thực thi mã, tìm kiếm tệp và tìm kiếm trên web.",
            tools=[
                HostedCodeInterpreterTool(),
                HostedWebSearchTool(name="Bing"),
            ],
        )

        result = await agent.run("Tính giai thừa của 20 bằng Python")
        print(result.text)
```

### Phản hồi phát trực tuyến (Streaming Responses)

```python
async def main():
    async with (
        AzureCliCredential() as credential,
        AzureAIAgentsProvider(credential=credential) as provider,
    ):
        agent = await provider.create_agent(
            name="StreamingAgent",
            instructions="Bạn là một trợ lý hữu ích.",
        )

        print("Đại lý: ", end="", flush=True)
        async for chunk in agent.run_stream("Kể cho tôi một câu chuyện ngắn"):
            if chunk.text:
                print(chunk.text, end="", flush=True)
        print()
```

### Luồng hội thoại (Conversation Threads)

```python
from agent_framework.azure import AzureAIAgentsProvider
from azure.identity.aio import AzureCliCredential

async def main():
    async with (
        AzureCliCredential() as credential,
        AzureAIAgentsProvider(credential=credential) as provider,
    ):
        agent = await provider.create_agent(
            name="ChatAgent",
            instructions="Bạn là một trợ lý hữu ích.",
            tools=[get_weather],
        )

        # Tạo luồng mới để lưu trữ hội thoại bền vững
        thread = agent.get_new_thread()

        # Lượt thứ nhất
        result1 = await agent.run("Thời tiết ở Seattle thế nào?", thread=thread)
        print(f"Đại lý: {result1.text}")

        # Lượt thứ hai - ngữ cảnh được duy trì
        result2 = await agent.run("Còn ở Portland thì sao?", thread=thread)
        print(f"Đại lý: {result2.text}")

        # Lưu lại thread ID để tiếp tục sau này
        print(f"ID hội thoại: {thread.conversation_id}")
```

### Đầu ra có cấu trúc (Structured Outputs)

```python
from pydantic import BaseModel, ConfigDict
from agent_framework.azure import AzureAIAgentsProvider
from azure.identity.aio import AzureCliCredential

class WeatherResponse(BaseModel):
    model_config = ConfigDict(extra="forbid")

    location: str
    temperature: float
    unit: str
    conditions: str

async def main():
    async with (
        AzureCliCredential() as credential,
        AzureAIAgentsProvider(credential=credential) as provider,
    ):
        agent = await provider.create_agent(
            name="StructuredAgent",
            instructions="Cung cấp thông tin thời tiết ở định dạng có cấu trúc.",
            response_format=WeatherResponse,
        )

        result = await agent.run("Thời tiết ở Seattle?")
        weather = WeatherResponse.model_validate_json(result.text)
        print(f"{weather.location}: {weather.temperature}°{weather.unit}")
```

## Các phương thức của Provider

| Phương thức           | Mô tả                                                   |
| --------------------- | ------------------------------------------------------- |
| `create_agent()`      | Tạo đại lý mới trên dịch vụ Azure AI                    |
| `get_agent(agent_id)` | Lấy đại lý hiện có theo ID                              |
| `as_agent(sdk_agent)` | Bọc đối tượng SDK Agent (không thực hiện cuộc gọi HTTP) |

## Tham khảo nhanh các Công cụ được Lưu trữ

| Công cụ                     | Import                                                  | Mục đích                          |
| --------------------------- | ------------------------------------------------------- | --------------------------------- |
| `HostedCodeInterpreterTool` | `from agent_framework import HostedCodeInterpreterTool` | Thực thi mã Python                |
| `HostedFileSearchTool`      | `from agent_framework import HostedFileSearchTool`      | Tìm kiếm trong kho lưu trữ vector |
| `HostedWebSearchTool`       | `from agent_framework import HostedWebSearchTool`       | Tìm kiếm web bằng Bing            |
| `HostedMCPTool`             | `from agent_framework import HostedMCPTool`             | MCP do dịch vụ quản lý            |
| `MCPStreamableHTTPTool`     | `from agent_framework import MCPStreamableHTTPTool`     | MCP do ứng dụng khách quản lý     |

## Ví dụ hoàn chỉnh

```python
import asyncio
from typing import Annotated
from pydantic import BaseModel, Field
from agent_framework import (
    HostedCodeInterpreterTool,
    HostedWebSearchTool,
    MCPStreamableHTTPTool,
)
from agent_framework.azure import AzureAIAgentsProvider
from azure.identity.aio import AzureCliCredential


def get_weather(
    location: Annotated[str, Field(description="Tên thành phố")],
) -> str:
    """Lấy thời tiết cho một địa điểm."""
    return f"Thời tiết tại {location}: 72°F, có nắng"


class AnalysisResult(BaseModel):
    summary: str
    key_findings: list[str]
    confidence: float


async def main():
    async with (
        AzureCliCredential() as credential,
        MCPStreamableHTTPTool(
            name="Docs MCP",
            url="https://learn.microsoft.com/api/mcp",
        ) as mcp_tool,
        AzureAIAgentsProvider(credential=credential) as provider,
    ):
        agent = await provider.create_agent(
            name="ResearchAssistant",
            instructions="Bạn là một trợ lý nghiên cứu với nhiều năng lực khác nhau.",
            tools=[
                get_weather,
                HostedCodeInterpreterTool(),
                HostedWebSearchTool(name="Bing"),
                mcp_tool,
            ],
        )

        thread = agent.get_new_thread()

        # Không phát trực tuyến (Non-streaming)
        result = await agent.run(
            "Tìm kiếm các thực hành tốt nhất về Python và tóm tắt lại",
            thread=thread,
        )
        print(f"Phản hồi: {result.text}")

        # Phát trực tuyến (Streaming)
        print("\nStreaming: ", end="")
        async for chunk in agent.run_stream("Tiếp tục với các ví dụ", thread=thread):
            if chunk.text:
                print(chunk.text, end="", flush=True)
        print()

        # Đầu ra có cấu trúc
        result = await agent.run(
            "Phân tích các kết quả tìm thấy",
            thread=thread,
            response_format=AnalysisResult,
        )
        analysis = AnalysisResult.model_validate_json(result.text)
        print(f"\nĐộ tin cậy: {analysis.confidence}")


if __name__ == "__main__":
    asyncio.run(main())
```

## Quy tắc chung

- Luôn sử dụng các trình quản lý ngữ cảnh bất đồng bộ: `async with provider:`
- Truyền các hàm trực tiếp vào tham số `tools=` (tự động chuyển đổi thành AIFunction)
- Sử dụng `Annotated[type, Field(description=...)]` cho các tham số của hàm
- Sử dụng `get_new_thread()` cho các cuộc hội thoại nhiều lượt
- Ưu tiên `HostedMCPTool` cho MCP do dịch vụ quản lý, và `MCPStreamableHTTPTool` cho MCP do ứng dụng khách quản lý

## Các tệp tham chiếu

- [references/tools.md](references/tools.md): Các mẫu công cụ được lưu trữ chi tiết
- [references/mcp.md](references/mcp.md): Tích hợp MCP (lưu trữ + cục bộ)
- [references/threads.md](references/threads.md): Quản lý Thread và hội thoại
- [references/advanced.md](references/advanced.md): OpenAPI, trích dẫn, đầu ra có cấu trúc
