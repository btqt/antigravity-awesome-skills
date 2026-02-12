---
name: azure-eventhub-py
description: |
  Azure Event Hubs SDK cho Python streaming. Sử dụng cho nhập sự kiện thông lượng cao (high-throughput event ingestion), producers, consumers, và checkpointing.
  Triggers: "event hubs", "EventHubProducerClient", "EventHubConsumerClient", "streaming", "partitions".
package: azure-eventhub
---

# Azure Event Hubs SDK cho Python

Nền tảng streaming dữ liệu lớn cho nhập sự kiện thông lượng cao.

## Cài đặt

```bash
pip install azure-eventhub azure-identity
# Cho checkpointing với blob storage
pip install azure-eventhub-checkpointstoreblob-aio
```

## Biến Môi trường

```bash
EVENT_HUB_FULLY_QUALIFIED_NAMESPACE=<namespace>.servicebus.windows.net
EVENT_HUB_NAME=my-eventhub
STORAGE_ACCOUNT_URL=https://<account>.blob.core.windows.net
CHECKPOINT_CONTAINER=checkpoints
```

## Xác thực

```python
from azure.identity import DefaultAzureCredential
from azure.eventhub import EventHubProducerClient, EventHubConsumerClient

credential = DefaultAzureCredential()
namespace = "<namespace>.servicebus.windows.net"
eventhub_name = "my-eventhub"

# Producer
producer = EventHubProducerClient(
    fully_qualified_namespace=namespace,
    eventhub_name=eventhub_name,
    credential=credential
)

# Consumer
consumer = EventHubConsumerClient(
    fully_qualified_namespace=namespace,
    eventhub_name=eventhub_name,
    consumer_group="$Default",
    credential=credential
)
```

## Các Loại Client

| Client                   | Mục đích                         |
| ------------------------ | -------------------------------- |
| `EventHubProducerClient` | Gửi sự kiện đến Event Hub        |
| `EventHubConsumerClient` | Nhận sự kiện từ Event Hub        |
| `BlobCheckpointStore`    | Theo dõi tiến trình của consumer |

## Gửi Sự kiện

```python
from azure.eventhub import EventHubProducerClient, EventData
from azure.identity import DefaultAzureCredential

producer = EventHubProducerClient(
    fully_qualified_namespace="<namespace>.servicebus.windows.net",
    eventhub_name="my-eventhub",
    credential=DefaultAzureCredential()
)

with producer:
    # Tạo lô (xử lý giới hạn kích thước)
    event_data_batch = producer.create_batch()

    for i in range(10):
        try:
            event_data_batch.add(EventData(f"Event {i}"))
        except ValueError:
            # Lô đã đầy, gửi và tạo lô mới
            producer.send_batch(event_data_batch)
            event_data_batch = producer.create_batch()
            event_data_batch.add(EventData(f"Event {i}"))

    # Gửi phần còn lại
    producer.send_batch(event_data_batch)
```

### Gửi đến Partition Cụ thể

```python
# Theo partition ID
event_data_batch = producer.create_batch(partition_id="0")

# Theo partition key (consistent hashing)
event_data_batch = producer.create_batch(partition_key="user-123")
```

## Nhận Sự kiện

### Nhận Đơn giản

```python
from azure.eventhub import EventHubConsumerClient

def on_event(partition_context, event):
    print(f"Partition: {partition_context.partition_id}")
    print(f"Data: {event.body_as_str()}")
    partition_context.update_checkpoint(event)

consumer = EventHubConsumerClient(
    fully_qualified_namespace="<namespace>.servicebus.windows.net",
    eventhub_name="my-eventhub",
    consumer_group="$Default",
    credential=DefaultAzureCredential()
)

with consumer:
    consumer.receive(
        on_event=on_event,
        starting_position="-1",  # Bắt đầu luồng (stream)
    )
```

### Với Blob Checkpoint Store (Sản xuất)

```python
from azure.eventhub import EventHubConsumerClient
from azure.eventhub.extensions.checkpointstoreblob import BlobCheckpointStore
from azure.identity import DefaultAzureCredential

checkpoint_store = BlobCheckpointStore(
    blob_account_url="https://<account>.blob.core.windows.net",
    container_name="checkpoints",
    credential=DefaultAzureCredential()
)

consumer = EventHubConsumerClient(
    fully_qualified_namespace="<namespace>.servicebus.windows.net",
    eventhub_name="my-eventhub",
    consumer_group="$Default",
    credential=DefaultAzureCredential(),
    checkpoint_store=checkpoint_store
)

def on_event(partition_context, event):
    print(f"Received: {event.body_as_str()}")
    # Checkpoint sau khi xử lý
    partition_context.update_checkpoint(event)

with consumer:
    consumer.receive(on_event=on_event)
```

## Async Client

```python
from azure.eventhub.aio import EventHubProducerClient, EventHubConsumerClient
from azure.identity.aio import DefaultAzureCredential
import asyncio

async def send_events():
    credential = DefaultAzureCredential()

    async with EventHubProducerClient(
        fully_qualified_namespace="<namespace>.servicebus.windows.net",
        eventhub_name="my-eventhub",
        credential=credential
    ) as producer:
        batch = await producer.create_batch()
        batch.add(EventData("Async event"))
        await producer.send_batch(batch)

async def receive_events():
    async def on_event(partition_context, event):
        print(event.body_as_str())
        await partition_context.update_checkpoint(event)

    async with EventHubConsumerClient(
        fully_qualified_namespace="<namespace>.servicebus.windows.net",
        eventhub_name="my-eventhub",
        consumer_group="$Default",
        credential=DefaultAzureCredential()
    ) as consumer:
        await consumer.receive(on_event=on_event)

asyncio.run(send_events())
```

## Thuộc tính Sự kiện

```python
event = EventData("My event body")

# Thiết lập thuộc tính
event.properties = {"custom_property": "value"}
event.content_type = "application/json"

# Đọc thuộc tính (khi nhận)
print(event.body_as_str())
print(event.sequence_number)
print(event.offset)
print(event.enqueued_time)
print(event.partition_key)
```

## Lấy Thông tin Event Hub

```python
with producer:
    info = producer.get_eventhub_properties()
    print(f"Name: {info['name']}")
    print(f"Partitions: {info['partition_ids']}")

    for partition_id in info['partition_ids']:
        partition_info = producer.get_partition_properties(partition_id)
        print(f"Partition {partition_id}: {partition_info['last_enqueued_sequence_number']}")
```

## Thực hành Tốt nhất

1. **Sử dụng lô (batches)** để gửi nhiều sự kiện
2. **Sử dụng checkpoint store** trong sản xuất để xử lý tin cậy
3. **Sử dụng async client** cho các kịch bản thông lượng cao
4. **Sử dụng partition keys** để giao hàng theo thứ tự trong một partition
5. **Xử lý giới hạn kích thước lô** — bắt ValueError khi lô đầy
6. **Sử dụng context managers** (`with`/`async with`) để dọn dẹp đúng cách
7. **Thiết lập consumer groups phù hợp** cho các ứng dụng khác nhau

## Tệp Tham khảo

| Tệp                                                        | Nội dung                                                             |
| ---------------------------------------------------------- | -------------------------------------------------------------------- |
| [references/checkpointing.md](references/checkpointing.md) | Các mẫu checkpoint store, blob checkpointing, chiến lược checkpoint  |
| [references/partitions.md](references/partitions.md)       | Quản lý partition, cân bằng tải, vị trí bắt đầu                      |
| [scripts/setup_consumer.py](scripts/setup_consumer.py)     | CLI cho thông tin Event Hub, thiết lập consumer, và gửi/nhận sự kiện |
