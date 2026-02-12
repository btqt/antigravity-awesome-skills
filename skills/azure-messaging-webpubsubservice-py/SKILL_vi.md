---
name: azure-messaging-webpubsubservice-py
description: |
  Azure Web PubSub Service SDK cho Python. Sử dụng cho nhắn tin thời gian thực, kết nối WebSocket, và các mẫu pub/sub.
  Triggers: "azure-messaging-webpubsubservice", "WebPubSubServiceClient", "real-time", "WebSocket", "pub/sub".
package: azure-messaging-webpubsubservice
---

# Azure Web PubSub Service SDK cho Python

Nhắn tin thời gian thực với kết nối WebSocket ở quy mô lớn.

## Cài đặt

```bash
# Service SDK (phía server)
pip install azure-messaging-webpubsubservice

# Client SDK (cho Python WebSocket clients)
pip install azure-messaging-webpubsubclient
```

## Biến Môi trường

```bash
AZURE_WEBPUBSUB_CONNECTION_STRING=Endpoint=https://<name>.webpubsub.azure.com;AccessKey=...
AZURE_WEBPUBSUB_HUB=my-hub
```

## Service Client (Phía Server)

### Xác thực

```python
from azure.messaging.webpubsubservice import WebPubSubServiceClient

# Connection string
client = WebPubSubServiceClient.from_connection_string(
    connection_string=os.environ["AZURE_WEBPUBSUB_CONNECTION_STRING"],
    hub="my-hub"
)

# Entra ID
from azure.identity import DefaultAzureCredential

client = WebPubSubServiceClient(
    endpoint="https://<name>.webpubsub.azure.com",
    hub="my-hub",
    credential=DefaultAzureCredential()
)
```

### Tạo Client Access Token

```python
# Token cho người dùng ẩn danh
token = client.get_client_access_token()
print(f"URL: {token['url']}")

# Token với user ID
token = client.get_client_access_token(
    user_id="user123",
    roles=["webpubsub.sendToGroup", "webpubsub.joinLeaveGroup"]
)

# Token với nhóm
token = client.get_client_access_token(
    user_id="user123",
    groups=["group1", "group2"]
)
```

### Gửi đến Tất cả Clients

```python
# Gửi text
client.send_to_all(message="Hello everyone!", content_type="text/plain")

# Gửi JSON
client.send_to_all(
    message={"type": "notification", "data": "Hello"},
    content_type="application/json"
)
```

### Gửi đến User

```python
client.send_to_user(
    user_id="user123",
    message="Hello user!",
    content_type="text/plain"
)
```

### Gửi đến Nhóm

```python
client.send_to_group(
    group="my-group",
    message="Hello group!",
    content_type="text/plain"
)
```

### Gửi đến Kết nối

```python
client.send_to_connection(
    connection_id="abc123",
    message="Hello connection!",
    content_type="text/plain"
)
```

### Quản lý Nhóm

```python
# Thêm user vào nhóm
client.add_user_to_group(group="my-group", user_id="user123")

# Xóa user khỏi nhóm
client.remove_user_from_group(group="my-group", user_id="user123")

# Thêm kết nối vào nhóm
client.add_connection_to_group(group="my-group", connection_id="abc123")

# Xóa kết nối khỏi nhóm
client.remove_connection_from_group(group="my-group", connection_id="abc123")
```

### Quản lý Kết nối

```python
# Kiểm tra xem kết nối có tồn tại không
exists = client.connection_exists(connection_id="abc123")

# Kiểm tra xem user có kết nối không
exists = client.user_exists(user_id="user123")

# Kiểm tra xem nhóm có kết nối không
exists = client.group_exists(group="my-group")

# Đóng kết nối
client.close_connection(connection_id="abc123", reason="Session ended")

# Đóng tất cả kết nối của user
client.close_all_connections(user_id="user123")
```

### Cấp/Thu hồi Quyền

```python
from azure.messaging.webpubsubservice import WebPubSubServiceClient

# Cấp quyền
client.grant_permission(
    permission="joinLeaveGroup",
    connection_id="abc123",
    target_name="my-group"
)

# Thu hồi quyền
client.revoke_permission(
    permission="joinLeaveGroup",
    connection_id="abc123",
    target_name="my-group"
)

# Kiểm tra quyền
has_permission = client.check_permission(
    permission="joinLeaveGroup",
    connection_id="abc123",
    target_name="my-group"
)
```

## Client SDK (Python WebSocket Client)

```python
from azure.messaging.webpubsubclient import WebPubSubClient

client = WebPubSubClient(credential=token["url"])

# Event handlers
@client.on("connected")
def on_connected(e):
    print(f"Connected: {e.connection_id}")

@client.on("server-message")
def on_message(e):
    print(f"Message: {e.data}")

@client.on("group-message")
def on_group_message(e):
    print(f"Group {e.group}: {e.data}")

# Kết nối và gửi
client.open()
client.send_to_group("my-group", "Hello from Python!")
```

## Async Service Client

```python
from azure.messaging.webpubsubservice.aio import WebPubSubServiceClient
from azure.identity.aio import DefaultAzureCredential

async def broadcast():
    credential = DefaultAzureCredential()
    client = WebPubSubServiceClient(
        endpoint="https://<name>.webpubsub.azure.com",
        hub="my-hub",
        credential=credential
    )

    await client.send_to_all("Hello async!", content_type="text/plain")

    await client.close()
    await credential.close()
```

## Các Thao tác Client

| Thao tác                  | Mô tả                        |
| ------------------------- | ---------------------------- |
| `get_client_access_token` | Tạo URL kết nối WebSocket    |
| `send_to_all`             | Phát sóng đến tất cả kết nối |
| `send_to_user`            | Gửi đến user cụ thể          |
| `send_to_group`           | Gửi đến các thành viên nhóm  |
| `send_to_connection`      | Gửi đến kết nối cụ thể       |
| `add_user_to_group`       | Thêm user vào nhóm           |
| `remove_user_from_group`  | Xóa user khỏi nhóm           |
| `close_connection`        | Ngắt kết nối client          |
| `connection_exists`       | Kiểm tra trạng thái kết nối  |

## Thực hành Tốt nhất

1. **Sử dụng roles** để giới hạn quyền của client
2. **Sử dụng nhóm** để nhắn tin mục tiêu
3. **Tạo token ngắn hạn** để bảo mật
4. **Sử dụng user IDs** để gửi tới người dùng qua các kết nối
5. **Xử lý kết nối lại** trong các ứng dụng client
6. **Sử dụng JSON** content type cho dữ liệu có cấu trúc
7. **Đóng kết nối** một cách duyên dáng với lý do
