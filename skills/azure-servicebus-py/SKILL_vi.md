---
name: azure-servicebus-py
description: |
  Azure Service Bus SDK cho Python messaging. Sử dụng cho queues, topics, subscriptions, và các mẫu nhắn tin doanh nghiệp.
  Triggers: "service bus", "ServiceBusClient", "queue", "topic", "subscription", "message broker".
package: azure-servicebus
---

# Azure Service Bus SDK cho Python

Nhắn tin doanh nghiệp cho giao tiếp đám mây đáng tin cậy với queues và pub/sub topics.

## Cài đặt

```bash
pip install azure-servicebus azure-identity
```

## Biến Môi trường

```bash
SERVICEBUS_FULLY_QUALIFIED_NAMESPACE=<namespace>.servicebus.windows.net
SERVICEBUS_QUEUE_NAME=myqueue
SERVICEBUS_TOPIC_NAME=mytopic
SERVICEBUS_SUBSCRIPTION_NAME=mysubscription
```

## Xác thực

```python
from azure.identity import DefaultAzureCredential
from azure.servicebus import ServiceBusClient

credential = DefaultAzureCredential()
namespace = "<namespace>.servicebus.windows.net"

client = ServiceBusClient(
    fully_qualified_namespace=namespace,
    credential=credential
)
```

## Các Loại Client

| Client               | Mục đích        | Lấy Từ                                                        |
| -------------------- | --------------- | ------------------------------------------------------------- |
| `ServiceBusClient`   | Quản lý kết nối | Khởi tạo trực tiếp                                            |
| `ServiceBusSender`   | Gửi tin nhắn    | `client.get_queue_sender()` / `get_topic_sender()`            |
| `ServiceBusReceiver` | Nhận tin nhắn   | `client.get_queue_receiver()` / `get_subscription_receiver()` |

## Gửi Tin nhắn (Async)

```python
import asyncio
from azure.servicebus.aio import ServiceBusClient
from azure.servicebus import ServiceBusMessage
from azure.identity.aio import DefaultAzureCredential

async def send_messages():
    credential = DefaultAzureCredential()

    async with ServiceBusClient(
        fully_qualified_namespace="<namespace>.servicebus.windows.net",
        credential=credential
    ) as client:
        sender = client.get_queue_sender(queue_name="myqueue")

        async with sender:
            # Tin nhắn đơn lẻ
            message = ServiceBusMessage("Hello, Service Bus!")
            await sender.send_messages(message)

            # Lô tin nhắn
            messages = [ServiceBusMessage(f"Message {i}") for i in range(10)]
            await sender.send_messages(messages)

            # Message batch (để kiểm soát kích thước)
            batch = await sender.create_message_batch()
            for i in range(100):
                try:
                    batch.add_message(ServiceBusMessage(f"Batch message {i}"))
                except ValueError:  # Batch đầy
                    await sender.send_messages(batch)
                    batch = await sender.create_message_batch()
                    batch.add_message(ServiceBusMessage(f"Batch message {i}"))
            await sender.send_messages(batch)

asyncio.run(send_messages())
```

## Nhận Tin nhắn (Async)

```python
async def receive_messages():
    credential = DefaultAzureCredential()

    async with ServiceBusClient(
        fully_qualified_namespace="<namespace>.servicebus.windows.net",
        credential=credential
    ) as client:
        receiver = client.get_queue_receiver(queue_name="myqueue")

        async with receiver:
            # Nhận theo lô
            messages = await receiver.receive_messages(
                max_message_count=10,
                max_wait_time=5  # giây
            )

            for msg in messages:
                print(f"Received: {str(msg)}")
                await receiver.complete_message(msg)  # Xóa khỏi hàng đợi

asyncio.run(receive_messages())
```

## Các Chế độ Nhận

| Chế độ                 | Hành vi                                 | Trường hợp Sử dụng                          |
| ---------------------- | --------------------------------------- | ------------------------------------------- |
| `PEEK_LOCK` (mặc định) | Tin nhắn bị khóa, phải hoàn thành/từ bỏ | Xử lý đáng tin cậy                          |
| `RECEIVE_AND_DELETE`   | Xóa ngay lập tức khi nhận               | Giao hàng nhiều nhất một lần (At-most-once) |

```python
from azure.servicebus import ServiceBusReceiveMode

receiver = client.get_queue_receiver(
    queue_name="myqueue",
    receive_mode=ServiceBusReceiveMode.RECEIVE_AND_DELETE
)
```

## Giải quyết Tin nhắn (Message Settlement)

```python
async with receiver:
    messages = await receiver.receive_messages(max_message_count=1)

    for msg in messages:
        try:
            # Xử lý tin nhắn...
            await receiver.complete_message(msg)  # Thành công - xóa khỏi hàng đợi
        except ProcessingError:
            await receiver.abandon_message(msg)  # Thử lại sau
        except PermanentError:
            await receiver.dead_letter_message(
                msg,
                reason="ProcessingFailed",
                error_description="Could not process"
            )
```

| Hành động               | Hiệu ứng                                                |
| ----------------------- | ------------------------------------------------------- |
| `complete_message()`    | Xóa khỏi hàng đợi (thành công)                          |
| `abandon_message()`     | Giải phóng khóa, thử lại ngay lập tức                   |
| `dead_letter_message()` | Di chuyển đến hàng đợi dead-letter                      |
| `defer_message()`       | Đặt sang một bên, nhận theo số thứ tự (sequence number) |

## Topics và Subscriptions

```python
# Gửi đến topic
sender = client.get_topic_sender(topic_name="mytopic")
async with sender:
    await sender.send_messages(ServiceBusMessage("Topic message"))

# Nhận từ subscription
receiver = client.get_subscription_receiver(
    topic_name="mytopic",
    subscription_name="mysubscription"
)
async with receiver:
    messages = await receiver.receive_messages(max_message_count=10)
```

## Sessions (FIFO)

```python
# Gửi với session
message = ServiceBusMessage("Session message")
message.session_id = "order-123"
await sender.send_messages(message)

# Nhận từ session cụ thể
receiver = client.get_queue_receiver(
    queue_name="session-queue",
    session_id="order-123"
)

# Nhận từ session có sẵn tiếp theo
from azure.servicebus import NEXT_AVAILABLE_SESSION
receiver = client.get_queue_receiver(
    queue_name="session-queue",
    session_id=NEXT_AVAILABLE_SESSION
)
```

## Tin nhắn được Lên lịch

```python
from datetime import datetime, timedelta, timezone

message = ServiceBusMessage("Scheduled message")
scheduled_time = datetime.now(timezone.utc) + timedelta(minutes=10)

# Lên lịch tin nhắn
sequence_number = await sender.schedule_messages(message, scheduled_time)

# Hủy tin nhắn đã lên lịch
await sender.cancel_scheduled_messages(sequence_number)
```

## Dead-Letter Queue

```python
from azure.servicebus import ServiceBusSubQueue

# Nhận từ dead-letter queue
dlq_receiver = client.get_queue_receiver(
    queue_name="myqueue",
    sub_queue=ServiceBusSubQueue.DEAD_LETTER
)

async with dlq_receiver:
    messages = await dlq_receiver.receive_messages(max_message_count=10)
    for msg in messages:
        print(f"Dead-lettered: {msg.dead_letter_reason}")
        await dlq_receiver.complete_message(msg)
```

## Sync Client (cho các script đơn giản)

```python
from azure.servicebus import ServiceBusClient, ServiceBusMessage
from azure.identity import DefaultAzureCredential

with ServiceBusClient(
    fully_qualified_namespace="<namespace>.servicebus.windows.net",
    credential=DefaultAzureCredential()
) as client:
    with client.get_queue_sender("myqueue") as sender:
        sender.send_messages(ServiceBusMessage("Sync message"))

    with client.get_queue_receiver("myqueue") as receiver:
        for msg in receiver:
            print(str(msg))
            receiver.complete_message(msg)
```

## Thực hành Tốt nhất

1. **Sử dụng async client** cho khối lượng công việc production
2. **Sử dụng context managers** (`async with`) để dọn dẹp đúng cách
3. **Hoàn thành tin nhắn** sau khi xử lý thành công
4. **Sử dụng dead-letter queue** cho các tin nhắn bị lỗi (poison messages)
5. **Sử dụng sessions** để xử lý theo thứ tự FIFO
6. **Sử dụng message batches** cho các kịch bản thông lượng cao
7. **Đặt `max_wait_time`** để tránh chặn vô hạn

## Các File Tham khảo

| File                                                       | Nội dung                                                                 |
| ---------------------------------------------------------- | ------------------------------------------------------------------------ |
| [references/patterns.md](references/patterns.md)           | Competing consumers, sessions, mẫu retry, request-response, transactions |
| [references/dead-letter.md](references/dead-letter.md)     | Xử lý DLQ, poison messages, chiến lược xử lý lại                         |
| [scripts/setup_servicebus.py](scripts/setup_servicebus.py) | CLI để quản lý queue/topic/subscription và giám sát DLQ                  |
