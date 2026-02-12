---
name: azure-eventgrid-py
description: |
  Azure Event Grid SDK cho Python. Sử dụng để xuất bản sự kiện (publishing events), xử lý CloudEvents, và các kiến trúc hướng sự kiện.
  Triggers: "event grid", "EventGridPublisherClient", "CloudEvent", "EventGridEvent", "publish events".
package: azure-eventgrid
---

# Azure Event Grid SDK cho Python

Dịch vụ định tuyến sự kiện để xây dựng các ứng dụng hướng sự kiện với ngữ nghĩa pub/sub.

## Cài đặt

```bash
pip install azure-eventgrid azure-identity
```

## Biến Môi trường

```bash
EVENTGRID_TOPIC_ENDPOINT=https://<topic-name>.<region>.eventgrid.azure.net/api/events
EVENTGRID_NAMESPACE_ENDPOINT=https://<namespace>.<region>.eventgrid.azure.net
```

## Xác thực

```python
from azure.identity import DefaultAzureCredential
from azure.eventgrid import EventGridPublisherClient

credential = DefaultAzureCredential()
endpoint = "https://<topic-name>.<region>.eventgrid.azure.net/api/events"

client = EventGridPublisherClient(endpoint, credential)
```

## Các Loại Sự kiện

| Định dạng         | Class            | Trường hợp sử dụng                           |
| ----------------- | ---------------- | -------------------------------------------- |
| Cloud Events 1.0  | `CloudEvent`     | Tiêu chuẩn, khả năng tương tác (khuyên dùng) |
| Event Grid Schema | `EventGridEvent` | Định dạng gốc của Azure                      |

## Xuất bản CloudEvents

```python
from azure.eventgrid import EventGridPublisherClient, CloudEvent
from azure.identity import DefaultAzureCredential

client = EventGridPublisherClient(endpoint, DefaultAzureCredential())

# Sự kiện đơn
event = CloudEvent(
    type="MyApp.Events.OrderCreated",
    source="/myapp/orders",
    data={"order_id": "12345", "amount": 99.99}
)
client.send(event)

# Nhiều sự kiện
events = [
    CloudEvent(
        type="MyApp.Events.OrderCreated",
        source="/myapp/orders",
        data={"order_id": f"order-{i}"}
    )
    for i in range(10)
]
client.send(events)
```

## Xuất bản EventGridEvents

```python
from azure.eventgrid import EventGridEvent
from datetime import datetime, timezone

event = EventGridEvent(
    subject="/myapp/orders/12345",
    event_type="MyApp.Events.OrderCreated",
    data={"order_id": "12345", "amount": 99.99},
    data_version="1.0"
)

client.send(event)
```

## Thuộc tính Sự kiện

### Thuộc tính CloudEvent

```python
event = CloudEvent(
    type="MyApp.Events.ItemCreated",      # Bắt buộc: loại sự kiện
    source="/myapp/items",                 # Bắt buộc: nguồn sự kiện
    data={"key": "value"},                 # Payload sự kiện
    subject="items/123",                   # Tùy chọn: chủ đề/đường dẫn
    datacontenttype="application/json",   # Tùy chọn: loại nội dung
    dataschema="https://schema.example",  # Tùy chọn: URL schema
    time=datetime.now(timezone.utc),      # Tùy chọn: dấu thời gian
    extensions={"custom": "value"}         # Tùy chọn: thuộc tính tùy chỉnh
)
```

### Thuộc tính EventGridEvent

```python
event = EventGridEvent(
    subject="/myapp/items/123",            # Bắt buộc: chủ đề
    event_type="MyApp.ItemCreated",        # Bắt buộc: loại sự kiện
    data={"key": "value"},                 # Bắt buộc: payload sự kiện
    data_version="1.0",                    # Bắt buộc: phiên bản schema
    topic="/subscriptions/.../topics/...", # Tùy chọn: tự động thiết lập
    event_time=datetime.now(timezone.utc)  # Tùy chọn: dấu thời gian
)
```

## Async Client

```python
from azure.eventgrid.aio import EventGridPublisherClient
from azure.identity.aio import DefaultAzureCredential

async def publish_events():
    credential = DefaultAzureCredential()

    async with EventGridPublisherClient(endpoint, credential) as client:
        event = CloudEvent(
            type="MyApp.Events.Test",
            source="/myapp",
            data={"message": "hello"}
        )
        await client.send(event)

import asyncio
asyncio.run(publish_events())
```

## Namespace Topics (Event Grid Namespaces)

Cho Event Grid Namespaces (pull delivery):

```python
from azure.eventgrid.aio import EventGridPublisherClient

# Namespace endpoint (khác với custom topic)
namespace_endpoint = "https://<namespace>.<region>.eventgrid.azure.net"
topic_name = "my-topic"

async with EventGridPublisherClient(
    endpoint=namespace_endpoint,
    credential=DefaultAzureCredential()
) as client:
    await client.send(
        event,
        namespace_topic=topic_name
    )
```

## Thực hành Tốt nhất

1. **Sử dụng CloudEvents** cho các ứng dụng mới (tiêu chuẩn ngành)
2. **Gửi lô sự kiện** khi xuất bản nhiều sự kiện
3. **Bao gồm subject có ý nghĩa** để lọc
4. **Sử dụng async client** cho các kịch bản thông lượng cao
5. **Xử lý thử lại** — Event Grid có cơ chế thử lại tích hợp sẵn
6. **Thiết lập loại sự kiện phù hợp** để định tuyến và lọc
