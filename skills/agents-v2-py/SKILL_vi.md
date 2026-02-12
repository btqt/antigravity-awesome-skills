---
name: agents-v2-py
description: |
  Xây dựng các Đại lý Foundry dựa trên container bằng Azure AI Projects SDK với ImageBasedHostedAgentDefinition.
  Sử dụng khi tạo các đại lý được lưu trữ chạy mã tùy chỉnh trong Azure AI Foundry với hình ảnh container của riêng bạn.
  Từ khóa kích hoạt: "ImageBasedHostedAgentDefinition", "hosted agent", "container agent", "Foundry Agent",
  "create_version", "ProtocolVersionRecord", "AgentProtocol.RESPONSES", "custom agent image".
package: azure-ai-projects
---

# Đại lý lưu trữ Azure AI (Python)

Xây dựng các đại lý lưu trữ dựa trên container bằng `ImageBasedHostedAgentDefinition` từ Azure AI Projects SDK.

## Cài đặt

```bash
pip install azure-ai-projects>=2.0.0b3 azure-identity
```

**Phiên bản SDK tối thiểu:** Yêu cầu bản `2.0.0b3` trở lên để hỗ trợ đại lý lưu trữ (hosted agent).

## Biến môi trường

```bash
AZURE_AI_PROJECT_ENDPOINT=https://<resource>.services.ai.azure.com/api/projects/<project>
```

## Điều kiện tiên quyết

Trước khi tạo các đại lý lưu trữ:

1. **Container Image** - Xây dựng và đẩy lên Azure Container Registry (ACR)
2. **Quyền Pull ACR** - Cấp vai trò `AcrPull` cho danh tính được quản lý của dự án trên ACR đó
3. **Capability Host** - Máy chủ năng lực cấp tài khoản với cài đặt `enablePublicHostingEnvironment=true`
4. **Phiên bản SDK** - Đảm bảo `azure-ai-projects>=2.0.0b3`

## Xác thực

Luôn sử dụng `DefaultAzureCredential`:

```python
from azure.identity import DefaultAzureCredential
from azure.ai.projects import AIProjectClient

credential = DefaultAzureCredential()
client = AIProjectClient(
    endpoint=os.environ["AZURE_AI_PROJECT_ENDPOINT"],
    credential=credential
)
```

## Quy trình công việc cốt lõi

### 1. Import các thư viện cần thiết

```python
import os
from azure.identity import DefaultAzureCredential
from azure.ai.projects import AIProjectClient
from azure.ai.projects.models import (
    ImageBasedHostedAgentDefinition,
    ProtocolVersionRecord,
    AgentProtocol,
)
```

### 2. Tạo Đại lý Lưu trữ (Hosted Agent)

```python
client = AIProjectClient(
    endpoint=os.environ["AZURE_AI_PROJECT_ENDPOINT"],
    credential=DefaultAzureCredential()
)

agent = client.agents.create_version(
    agent_name="my-hosted-agent",
    definition=ImageBasedHostedAgentDefinition(
        container_protocol_versions=[
            ProtocolVersionRecord(protocol=AgentProtocol.RESPONSES, version="v1")
        ],
        cpu="1",
        memory="2Gi",
        image="myregistry.azurecr.io/my-agent:latest",
        tools=[{"type": "code_interpreter"}],
        environment_variables={
            "AZURE_AI_PROJECT_ENDPOINT": os.environ["AZURE_AI_PROJECT_ENDPOINT"],
            "MODEL_NAME": "gpt-4o-mini"
        }
    )
)

print(f"Đã tạo đại lý: {agent.name} (phiên bản: {agent.version})")
```

### 3. Liệt kê các phiên bản Đại lý

```python
versions = client.agents.list_versions(agent_name="my-hosted-agent")
for version in versions:
    print(f"Phiên bản: {version.version}, Trạng thái: {version.state}")
```

### 4. Xóa một phiên bản Đại lý

```python
client.agents.delete_version(
    agent_name="my-hosted-agent",
    version=agent.version
)
```

## Các tham số của ImageBasedHostedAgentDefinition

| Tham số                       | Kiểu                          | Bắt buộc | Mô tả                                                     |
| ----------------------------- | ----------------------------- | -------- | --------------------------------------------------------- |
| `container_protocol_versions` | `list[ProtocolVersionRecord]` | Có       | Các phiên bản giao thức mà đại lý hỗ trợ                  |
| `image`                       | `str`                         | Có       | Đường dẫn đầy đủ của container image (registry/image:tag) |
| `cpu`                         | `str`                         | Không    | Phân bổ CPU (ví dụ: "1", "2")                             |
| `memory`                      | `str`                         | Không    | Phân bổ bộ nhớ (ví dụ: "2Gi", "4Gi")                      |
| `tools`                       | `list[dict]`                  | Không    | Các công cụ có sẵn cho đại lý                             |
| `environment_variables`       | `dict[str, str]`              | Không    | Các biến môi trường cho container                         |

## Các phiên bản Giao thức (Protocol Versions)

Tham số `container_protocol_versions` chỉ định các giao thức mà đại lý của bạn hỗ trợ:

```python
from azure.ai.projects.models import ProtocolVersionRecord, AgentProtocol

# Giao thức RESPONSES - các phản hồi chuẩn của đại lý
container_protocol_versions=[
    ProtocolVersionRecord(protocol=AgentProtocol.RESPONSES, version="v1")
]
```

**Các giao thức khả dụng:**
| Giao thức | Mô tả |
|----------|-------------|
| `AgentProtocol.RESPONSES` | Giao thức phản hồi chuẩn cho các tương tác của đại lý |

## Phân bổ Tài nguyên

Chỉ định CPU và bộ nhớ cho container của bạn:

```python
definition=ImageBasedHostedAgentDefinition(
    container_protocol_versions=[...],
    image="myregistry.azurecr.io/my-agent:latest",
    cpu="2",      # 2 nhân CPU
    memory="4Gi"  # 4 GiB bộ nhớ
)
```

**Giới hạn Tài nguyên:**
| Tài nguyên | Tối thiểu | Tối đa | Mặc định |
|----------|-----|-----|---------|
| CPU | 0.5 | 4 | 1 |
| Bộ nhớ | 1Gi | 8Gi | 2Gi |

## Cấu hình Công cụ (Tools)

Thêm các công cụ cho đại lý lưu trữ của bạn:

### Trình thông dịch mã (Code Interpreter)

```python
tools=[{"type": "code_interpreter"}]
```

### Các công cụ MCP

```python
tools=[
    {"type": "code_interpreter"},
    {
        "type": "mcp",
        "server_label": "my-mcp-server",
        "server_url": "https://my-mcp-server.example.com"
    }
]
```

### Nhiều công cụ kết hợp

```python
tools=[
    {"type": "code_interpreter"},
    {"type": "file_search"},
    {
        "type": "mcp",
        "server_label": "custom-tool",
        "server_url": "https://custom-tool.example.com"
    }
]
```

## Biến môi trường

Truyền cấu hình vào container của bạn:

```python
environment_variables={
    "AZURE_AI_PROJECT_ENDPOINT": os.environ["AZURE_AI_PROJECT_ENDPOINT"],
    "MODEL_NAME": "gpt-4o-mini",
    "LOG_LEVEL": "INFO",
    "CUSTOM_CONFIG": "giá trị"
}
```

**Lời khuyên:** Không bao giờ ghi cứng các bí mật (secrets). Hãy sử dụng biến môi trường hoặc Azure Key Vault.

## Ví dụ hoàn chỉnh

```python
import os
from azure.identity import DefaultAzureCredential
from azure.ai.projects import AIProjectClient
from azure.ai.projects.models import (
    ImageBasedHostedAgentDefinition,
    ProtocolVersionRecord,
    AgentProtocol,
)

def create_hosted_agent():
    """Tạo một đại lý lưu trữ với container image tùy chỉnh."""

    client = AIProjectClient(
        endpoint=os.environ["AZURE_AI_PROJECT_ENDPOINT"],
        credential=DefaultAzureCredential()
    )

    agent = client.agents.create_version(
        agent_name="data-processor-agent",
        definition=ImageBasedHostedAgentDefinition(
            container_protocol_versions=[
                ProtocolVersionRecord(
                    protocol=AgentProtocol.RESPONSES,
                    version="v1"
                )
            ],
            image="myregistry.azurecr.io/data-processor:v1.0",
            cpu="2",
            memory="4Gi",
            tools=[
                {"type": "code_interpreter"},
                {"type": "file_search"}
            ],
            environment_variables={
                "AZURE_AI_PROJECT_ENDPOINT": os.environ["AZURE_AI_PROJECT_ENDPOINT"],
                "MODEL_NAME": "gpt-4o-mini",
                "MAX_RETRIES": "3"
            }
        )
    )

    print(f"Đã tạo đại lý lưu trữ: {agent.name}")
    print(f"Phiên bản: {agent.version}")
    print(f"Trạng thái: {agent.state}")

    return agent

if __name__ == "__main__":
    create_hosted_agent()
```

## Mẫu lập trình bất đồng bộ (Async Pattern)

```python
import os
from azure.identity.aio import DefaultAzureCredential
from azure.ai.projects.aio import AIProjectClient
from azure.ai.projects.models import (
    ImageBasedHostedAgentDefinition,
    ProtocolVersionRecord,
    AgentProtocol,
)

async def create_hosted_agent_async():
    """Tạo một đại lý lưu trữ bất đồng bộ."""

    async with DefaultAzureCredential() as credential:
        async with AIProjectClient(
            endpoint=os.environ["AZURE_AI_PROJECT_ENDPOINT"],
            credential=credential
        ) as client:
            agent = await client.agents.create_version(
                agent_name="async-agent",
                definition=ImageBasedHostedAgentDefinition(
                    container_protocol_versions=[
                        ProtocolVersionRecord(
                            protocol=AgentProtocol.RESPONSES,
                            version="v1"
                        )
                    ],
                    image="myregistry.azurecr.io/async-agent:latest",
                    cpu="1",
                    memory="2Gi"
                )
            )
            return agent
```

## Các lỗi thường gặp

| Lỗi                           | Nguyên nhân                      | Giải pháp                                                  |
| ----------------------------- | -------------------------------- | ---------------------------------------------------------- |
| `ImagePullBackOff`            | Quyền pull ACR bị từ chối        | Cấp vai trò `AcrPull` cho danh tính được quản lý của dự án |
| `InvalidContainerImage`       | Không tìm thấy hình ảnh          | Kiểm tra lại đường dẫn và tag của image trong ACR          |
| `CapabilityHostNotFound`      | Chưa cấu hình capability host    | Tạo một capability host cấp tài khoản                      |
| `ProtocolVersionNotSupported` | Phiên bản giao thức không hợp lệ | Sử dụng `AgentProtocol.RESPONSES` với phiên bản `"v1"`     |

## Các thực hành tốt nhất

1. **Gắn phiên bản cho Image** - Sử dụng các tag cụ thể, không dùng `latest` khi triển khai thực tế.
2. **Tối thiểu hóa Tài nguyên** - Bắt đầu với mức CPU/bộ nhớ tối thiểu, và tăng dần nếu cần.
3. **Biến môi trường** - Sử dụng cho tất cả các cấu hình, không bao giờ ghi cứng.
4. **Xử lý lỗi** - Bao bọc việc tạo đại lý trong các khối try/except.
5. **Dọn dẹp** - Xóa các phiên bản đại lý không còn sử dụng để giải phóng tài nguyên.

## Các liên kết tham khảo

- [Azure AI Projects SDK](https://pypi.org/project/azure-ai-projects/)
- [Tài liệu về Hosted Agents](https://learn.microsoft.com/azure/ai-services/agents/how-to/hosted-agents)
- [Azure Container Registry](https://learn.microsoft.com/azure/container-registry/)
