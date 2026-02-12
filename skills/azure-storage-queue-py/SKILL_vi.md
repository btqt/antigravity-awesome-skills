---
name: azure-storage-queue-py
description: |
  Azure Queue Storage SDK cho Python. Sử dụng để xếp hàng tin nhắn (message queuing) đáng tin cậy, phân phối tác vụ, và xử lý bất đồng bộ.
  Triggers: "queue storage", "QueueServiceClient", "QueueClient", "message queue", "dequeue".
package: azure-storage-queue
---

# Azure Queue Storage SDK cho Python

Xếp hàng tin nhắn đơn giản, chi phí thấp cho giao tiếp bất đồng bộ.

## Cài đặt

```bash
pip install azure-storage-queue azure-identity
```

## Biến Môi trường

```bash
AZURE_STORAGE_ACCOUNT_URL=https://<account>.queue.core.windows.net
```

## Xác thực

```python
from azure.identity import DefaultAzureCredential
from azure.storage.queue import QueueServiceClient, QueueClient

credential = DefaultAzureCredential()
account_url = "https://<account>.queue.core.windows.net"

# Service client
service_client = QueueServiceClient(account_url=account_url, credential=credential)

# Queue client
queue_client = QueueClient(account_url=account_url, queue_name="myqueue", credential=credential)
```

## Các Hoạt động Queue

```python
# Tạo queue
service_client.create_queue("myqueue")

# Lấy queue client
queue_client = service_client.get_queue_client("myqueue")

# Xóa queue
service_client.delete_queue("myqueue")

# Liệt kê queues
for queue in service_client.list_queues():
    print(queue.name)
```

## Gửi Tin nhắn

```python
# Gửi tin nhắn (chuỗi)
queue_client.send_message("Hello, Queue!")

# Gửi với các tùy chọn
queue_client.send_message(
    content="Delayed message",
    visibility_timeout=60,  # Ẩn trong 60 giây
    time_to_live=3600       # Hết hạn trong 1 giờ
)

# Gửi JSON
import json
data = {"task": "process", "id": 123}
queue_client.send_message(json.dumps(data))
```

## Nhận Tin nhắn

```python
# Nhận tin nhắn (làm cho chúng ẩn tạm thời)
messages = queue_client.receive_messages(
    messages_per_page=10,
    visibility_timeout=30  # 30 giây để xử lý
)

for message in messages:
    print(f"ID: {message.id}")
    print(f"Content: {message.content}")
    print(f"Dequeue count: {message.dequeue_count}")

    # Xử lý tin nhắn...

    # Xóa sau khi xử lý
    queue_client.delete_message(message)
```

## Peek Tin nhắn

```python
# Peek mà không ẩn (không ảnh hưởng đến visibility)
messages = queue_client.peek_messages(max_messages=5)

for message in messages:
    print(message.content)
```

## Cập nhật Tin nhắn

```python
# Gia hạn visibility hoặc cập nhật nội dung
messages = queue_client.receive_messages()
for message in messages:
    # Gia hạn timeout (cần thêm thời gian)
    queue_client.update_message(
        message,
        visibility_timeout=60
    )

    # Cập nhật nội dung và timeout
    queue_client.update_message(
        message,
        content="Updated content",
        visibility_timeout=60
    )
```

## Xóa Tin nhắn

```python
# Xóa sau khi xử lý thành công
messages = queue_client.receive_messages()
for message in messages:
    try:
        # Xử lý...
        queue_client.delete_message(message)
    except Exception:
        # Tin nhắn sẽ hiển thị lại sau khi timeout
        pass
```

## Xóa Hết Queue (Clear Queue)

```python
# Xóa tất cả tin nhắn
queue_client.clear_messages()
```

## Thuộc tính Queue

```python
# Lấy thuộc tính queue
properties = queue_client.get_queue_properties()
print(f"Approximate message count: {properties.approximate_message_count}")

# Đặt/lấy metadata
queue_client.set_queue_metadata(metadata={"environment": "production"})
properties = queue_client.get_queue_properties()
print(properties.metadata)
```

## Async Client

```python
from azure.storage.queue.aio import QueueServiceClient, QueueClient
from azure.identity.aio import DefaultAzureCredential

async def queue_operations():
    credential = DefaultAzureCredential()

    async with QueueClient(
        account_url="https://<account>.queue.core.windows.net",
        queue_name="myqueue",
        credential=credential
    ) as client:
        # Gửi
        await client.send_message("Async message")

        # Nhận
        async for message in client.receive_messages():
            print(message.content)
            await client.delete_message(message)

import asyncio
asyncio.run(queue_operations())
```

## Base64 Encoding

```python
from azure.storage.queue import QueueClient, BinaryBase64EncodePolicy, BinaryBase64DecodePolicy

# Cho dữ liệu nhị phân
queue_client = QueueClient(
    account_url=account_url,
    queue_name="myqueue",
    credential=credential,
    message_encode_policy=BinaryBase64EncodePolicy(),
    message_decode_policy=BinaryBase64DecodePolicy()
)

# Gửi bytes
queue_client.send_message(b"Binary content")
```

## Thực hành Tốt nhất

1. **Xóa tin nhắn sau khi xử lý** để ngăn chặn việc xử lý lại
2. **Đặt visibility timeout phù hợp** dựa trên thời gian xử lý
3. **Xử lý `dequeue_count`** để phát hiện tin nhắn bị lỗi (poison message)
4. **Sử dụng async client** cho các kịch bản thông lượng cao
5. **Sử dụng `peek_messages`** để giám sát mà không ảnh hưởng đến queue
6. **Đặt `time_to_live`** để ngăn chặn tin nhắn cũ
7. **Cân nhắc Service Bus** cho các tính năng nâng cao (sessions, topics)
