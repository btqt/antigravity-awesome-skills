---
name: azure-mgmt-botservice-py
description: |
  Azure Bot Service Management SDK cho Python. Sử dụng để tạo, quản lý, và cấu hình các tài nguyên Azure Bot Service.
  Triggers: "azure-mgmt-botservice", "AzureBotService", "bot management", "conversational AI", "bot channels".
---

# Azure Bot Service Management SDK cho Python

Quản lý tài nguyên Azure Bot Service bao gồm bots, channels, và connections.

## Cài đặt

```bash
pip install azure-mgmt-botservice
pip install azure-identity
```

## Biến Môi trường

```bash
AZURE_SUBSCRIPTION_ID=<your-subscription-id>
AZURE_RESOURCE_GROUP=<your-resource-group>
```

## Xác thực

```python
from azure.identity import DefaultAzureCredential
from azure.mgmt.botservice import AzureBotService
import os

credential = DefaultAzureCredential()
client = AzureBotService(
    credential=credential,
    subscription_id=os.environ["AZURE_SUBSCRIPTION_ID"]
)
```

## Tạo Bot

```python
from azure.mgmt.botservice import AzureBotService
from azure.mgmt.botservice.models import Bot, BotProperties, Sku
from azure.identity import DefaultAzureCredential
import os

credential = DefaultAzureCredential()
client = AzureBotService(
    credential=credential,
    subscription_id=os.environ["AZURE_SUBSCRIPTION_ID"]
)

resource_group = os.environ["AZURE_RESOURCE_GROUP"]
bot_name = "my-chat-bot"

bot = client.bots.create(
    resource_group_name=resource_group,
    resource_name=bot_name,
    parameters=Bot(
        location="global",
        sku=Sku(name="F0"),  # Free tier
        kind="azurebot",
        properties=BotProperties(
            display_name="My Chat Bot",
            description="A conversational AI bot",
            endpoint="https://my-bot-app.azurewebsites.net/api/messages",
            msa_app_id="<your-app-id>",
            msa_app_type="MultiTenant"
        )
    )
)

print(f"Bot created: {bot.name}")
```

## Lấy Chi tiết Bot

```python
bot = client.bots.get(
    resource_group_name=resource_group,
    resource_name=bot_name
)

print(f"Bot: {bot.properties.display_name}")
print(f"Endpoint: {bot.properties.endpoint}")
print(f"SKU: {bot.sku.name}")
```

## Liệt kê Bots trong Resource Group

```python
bots = client.bots.list_by_resource_group(resource_group_name=resource_group)

for bot in bots:
    print(f"Bot: {bot.name} - {bot.properties.display_name}")
```

## Liệt kê Tất cả Bots trong Subscription

```python
all_bots = client.bots.list()

for bot in all_bots:
    print(f"Bot: {bot.name} in {bot.id.split('/')[4]}")
```

## Cập nhật Bot

```python
bot = client.bots.update(
    resource_group_name=resource_group,
    resource_name=bot_name,
    properties=BotProperties(
        display_name="Updated Bot Name",
        description="Updated description"
    )
)
```

## Xóa Bot

```python
client.bots.delete(
    resource_group_name=resource_group,
    resource_name=bot_name
)
```

## Cấu hình Kênh (Channels)

### Thêm Kênh Teams

```python
from azure.mgmt.botservice.models import (
    BotChannel,
    MsTeamsChannel,
    MsTeamsChannelProperties
)

channel = client.channels.create(
    resource_group_name=resource_group,
    resource_name=bot_name,
    channel_name="MsTeamsChannel",
    parameters=BotChannel(
        location="global",
        properties=MsTeamsChannel(
            properties=MsTeamsChannelProperties(
                is_enabled=True
            )
        )
    )
)
```

### Thêm Kênh Direct Line

```python
from azure.mgmt.botservice.models import (
    BotChannel,
    DirectLineChannel,
    DirectLineChannelProperties,
    DirectLineSite
)

channel = client.channels.create(
    resource_group_name=resource_group,
    resource_name=bot_name,
    channel_name="DirectLineChannel",
    parameters=BotChannel(
        location="global",
        properties=DirectLineChannel(
            properties=DirectLineChannelProperties(
                sites=[
                    DirectLineSite(
                        site_name="Default Site",
                        is_enabled=True,
                        is_v1_enabled=False,
                        is_v3_enabled=True
                    )
                ]
            )
        )
    )
)
```

### Thêm Kênh Web Chat

```python
from azure.mgmt.botservice.models import (
    BotChannel,
    WebChatChannel,
    WebChatChannelProperties,
    WebChatSite
)

channel = client.channels.create(
    resource_group_name=resource_group,
    resource_name=bot_name,
    channel_name="WebChatChannel",
    parameters=BotChannel(
        location="global",
        properties=WebChatChannel(
            properties=WebChatChannelProperties(
                sites=[
                    WebChatSite(
                        site_name="Default Site",
                        is_enabled=True
                    )
                ]
            )
        )
    )
)
```

## Lấy Chi tiết Kênh

```python
channel = client.channels.get(
    resource_group_name=resource_group,
    resource_name=bot_name,
    channel_name="DirectLineChannel"
)
```

## Liệt kê Keys của Kênh

```python
keys = client.channels.list_with_keys(
    resource_group_name=resource_group,
    resource_name=bot_name,
    channel_name="DirectLineChannel"
)

# Truy cập Direct Line keys
if hasattr(keys.properties, 'properties'):
    for site in keys.properties.properties.sites:
        print(f"Site: {site.site_name}")
        print(f"Key: {site.key}")
```

## Kết nối Bot (OAuth)

### Tạo Cài đặt Kết nối

```python
from azure.mgmt.botservice.models import (
    ConnectionSetting,
    ConnectionSettingProperties
)

connection = client.bot_connection.create(
    resource_group_name=resource_group,
    resource_name=bot_name,
    connection_name="graph-connection",
    parameters=ConnectionSetting(
        location="global",
        properties=ConnectionSettingProperties(
            client_id="<oauth-client-id>",
            client_secret="<oauth-client-secret>",
            scopes="User.Read",
            service_provider_id="<service-provider-id>"
        )
    )
)
```

### Liệt kê Kết nối

```python
connections = client.bot_connection.list_by_bot_service(
    resource_group_name=resource_group,
    resource_name=bot_name
)

for conn in connections:
    print(f"Connection: {conn.name}")
```

## Các Thao tác Client

| Thao tác                | Phương thức               |
| ----------------------- | ------------------------- |
| `client.bots`           | Thao tác CRUD Bot         |
| `client.channels`       | Cấu hình Kênh             |
| `client.bot_connection` | Cài đặt kết nối OAuth     |
| `client.direct_line`    | Thao tác kênh Direct Line |
| `client.email`          | Thao tác kênh Email       |
| `client.operations`     | Các thao tác khả dụng     |
| `client.host_settings`  | Thao tác cài đặt Host     |

## Các Tùy chọn SKU

| SKU  | Mô tả                                   |
| ---- | --------------------------------------- |
| `F0` | Free tier (giới hạn tin nhắn)           |
| `S1` | Standard tier (không giới hạn tin nhắn) |

## Các Loại Kênh

| Kênh                | Lớp             | Mục đích                           |
| ------------------- | --------------- | ---------------------------------- |
| `MsTeamsChannel`    | Microsoft Teams | Tích hợp Teams                     |
| `DirectLineChannel` | Direct Line     | Tích hợp client tùy chỉnh          |
| `WebChatChannel`    | Web Chat        | Web widget có thể nhúng            |
| `SlackChannel`      | Slack           | Tích hợp không gian làm việc Slack |
| `FacebookChannel`   | Facebook        | Tích hợp Messenger                 |
| `EmailChannel`      | Email           | Giao tiếp qua Email                |

## Thực hành Tốt nhất

1. **Sử dụng DefaultAzureCredential** cho xác thực
2. **Bắt đầu với F0 SKU** để phát triển, nâng cấp lên S1 cho sản xuất
3. **Lưu trữ MSA App ID/Secret an toàn** — sử dụng Key Vault
4. **Chỉ bật các kênh cần thiết** — giảm bề mặt tấn công
5. **Xoay vòng Direct Line keys** định kỳ
6. **Sử dụng managed identity** khi có thể cho các kết nối bot
7. **Cấu hình CORS phù hợp** cho kênh Web Chat
