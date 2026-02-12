---
name: azure-monitor-ingestion-py
description: |
  Azure Monitor Ingestion SDK cho Python. Sử dụng để gửi nhật ký tùy chỉnh (custom logs) đến Log Analytics workspace thông qua Logs Ingestion API.
  Triggers: "azure-monitor-ingestion", "LogsIngestionClient", "custom logs", "DCR", "data collection rule", "Log Analytics".
package: azure-monitor-ingestion
---

# Azure Monitor Ingestion SDK cho Python

Gửi nhật ký tùy chỉnh đến Azure Monitor Log Analytics workspace sử dụng Logs Ingestion API.

## Cài đặt

```bash
pip install azure-monitor-ingestion
pip install azure-identity
```

## Biến Môi trường

```bash
# Data Collection Endpoint (DCE)
AZURE_DCE_ENDPOINT=https://<dce-name>.<region>.ingest.monitor.azure.com

# Data Collection Rule (DCR) immutable ID
AZURE_DCR_RULE_ID=dcr-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Tên Stream từ DCR
AZURE_DCR_STREAM_NAME=Custom-MyTable_CL
```

## Điều kiện Tiên quyết

Trước khi sử dụng SDK này, bạn cần:

1. **Log Analytics Workspace** — Đích cho logs của bạn
2. **Data Collection Endpoint (DCE)** — Endpoint nhập liệu
3. **Data Collection Rule (DCR)** — Định nghĩa schema và đích đến
4. **Custom Table** — Trong Log Analytics (được tạo qua DCR hoặc thủ công)

## Xác thực

```python
from azure.monitor.ingestion import LogsIngestionClient
from azure.identity import DefaultAzureCredential
import os

client = LogsIngestionClient(
    endpoint=os.environ["AZURE_DCE_ENDPOINT"],
    credential=DefaultAzureCredential()
)
```

## Upload Custom Logs

```python
from azure.monitor.ingestion import LogsIngestionClient
from azure.identity import DefaultAzureCredential
import os

client = LogsIngestionClient(
    endpoint=os.environ["AZURE_DCE_ENDPOINT"],
    credential=DefaultAzureCredential()
)

rule_id = os.environ["AZURE_DCR_RULE_ID"]
stream_name = os.environ["AZURE_DCR_STREAM_NAME"]

logs = [
    {"TimeGenerated": "2024-01-15T10:00:00Z", "Computer": "server1", "Message": "Application started"},
    {"TimeGenerated": "2024-01-15T10:01:00Z", "Computer": "server1", "Message": "Processing request"},
    {"TimeGenerated": "2024-01-15T10:02:00Z", "Computer": "server2", "Message": "Connection established"}
]

client.upload(rule_id=rule_id, stream_name=stream_name, logs=logs)
```

## Upload từ File JSON

```python
import json

with open("logs.json", "r") as f:
    logs = json.load(f)

client.upload(rule_id=rule_id, stream_name=stream_name, logs=logs)
```

## Xử lý Lỗi Tùy chỉnh

Xử lý các lỗi một phần với callback:

```python
failed_logs = []

def on_error(error):
    print(f"Upload failed: {error.error}")
    failed_logs.extend(error.failed_logs)

client.upload(
    rule_id=rule_id,
    stream_name=stream_name,
    logs=logs,
    on_error=on_error
)

# Thử lại các logs thất bại
if failed_logs:
    print(f"Retrying {len(failed_logs)} failed logs...")
    client.upload(rule_id=rule_id, stream_name=stream_name, logs=failed_logs)
```

## Bỏ qua Lỗi

```python
def ignore_errors(error):
    pass  # Lặng lẽ bỏ qua các lỗi upload

client.upload(
    rule_id=rule_id,
    stream_name=stream_name,
    logs=logs,
    on_error=ignore_errors
)
```

## Async Client

```python
import asyncio
from azure.monitor.ingestion.aio import LogsIngestionClient
from azure.identity.aio import DefaultAzureCredential

async def upload_logs():
    async with LogsIngestionClient(
        endpoint=endpoint,
        credential=DefaultAzureCredential()
    ) as client:
        await client.upload(
            rule_id=rule_id,
            stream_name=stream_name,
            logs=logs
        )

asyncio.run(upload_logs())
```

## Sovereign Clouds

```python
from azure.identity import AzureAuthorityHosts, DefaultAzureCredential
from azure.monitor.ingestion import LogsIngestionClient

# Azure Government
credential = DefaultAzureCredential(authority=AzureAuthorityHosts.AZURE_GOVERNMENT)
client = LogsIngestionClient(
    endpoint="https://example.ingest.monitor.azure.us",
    credential=credential,
    credential_scopes=["https://monitor.azure.us/.default"]
)
```

## Hành vi Batching

SDK tự động:

- Chia logs thành các chunks từ 1MB trở xuống
- Nén từng chunk với gzip
- Upload các chunks song song

Không cần batching thủ công cho các tập hợp log lớn.

## Các Loại Client

| Client                      | Mục đích                    |
| --------------------------- | --------------------------- |
| `LogsIngestionClient`       | Sync client để upload logs  |
| `LogsIngestionClient` (aio) | Async client để upload logs |

## Các Khái niệm Chính

| Khái niệm        | Mô tả                                                          |
| ---------------- | -------------------------------------------------------------- |
| **DCE**          | Data Collection Endpoint — ingestion URL                       |
| **DCR**          | Data Collection Rule — định nghĩa schema, chuyển đổi, đích đến |
| **Stream**       | Luồng dữ liệu được đặt tên trong một DCR                       |
| **Custom Table** | Bảng đích trong Log Analytics (kết thúc bằng `_CL`)            |

## Định dạng Tên Stream DCR

Tên Stream tuân theo các mẫu:

- `Custom-<TableName>_CL` — Cho các bảng tùy chỉnh
- `Microsoft-<TableName>` — Cho các bảng có sẵn (built-in)

## Thực hành Tốt nhất

1. **Sử dụng DefaultAzureCredential** cho xác thực
2. **Xử lý lỗi nhẹ nhàng** — sử dụng `on_error` callback cho các lỗi một phần
3. **Bao gồm TimeGenerated** — Trường bắt buộc cho tất cả các logs
4. **Khớp schema DCR** — Các trường log phải khớp với định nghĩa cột trong DCR
5. **Sử dụng async client** cho các kịch bản thông lượng cao
6. **Batch uploads** — SDK xử lý batching, nhưng hãy gửi các chunks hợp lý
7. **Giám sát ingestion** — Kiểm tra Log Analytics cho trạng thái ingestion
8. **Sử dụng context manager** — Đảm bảo dọn dẹp client đúng cách
