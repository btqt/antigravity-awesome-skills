---
name: azure-web-pubsub-ts
description: Xây dựng các ứng dụng nhắn tin thời gian thực sử dụng Azure Web PubSub SDKs cho JavaScript (@azure/web-pubsub, @azure/web-pubsub-client). Sử dụng khi triển khai các tính năng thời gian thực dựa trên WebSocket, nhắn tin pub/sub, chat nhóm, hoặc thông báo trực tiếp.
package: @azure/web-pubsub, @azure/web-pubsub-client
---

# Azure Web PubSub SDKs cho TypeScript

Nhắn tin thời gian thực với các kết nối WebSocket và các mẫu pub/sub.

## Cài đặt

```bash
# Quản lý phía Server
npm install @azure/web-pubsub @azure/identity

# Nhắn tin thời gian thực phía Client
npm install @azure/web-pubsub-client

# Express middleware cho event handlers
npm install @azure/web-pubsub-express
```

## Biến Môi trường

```bash
WEBPUBSUB_CONNECTION_STRING=Endpoint=https://<resource>.webpubsub.azure.com;AccessKey=<key>;Version=1.0;
WEBPUBSUB_ENDPOINT=https://<resource>.webpubsub.azure.com
```

## Phía Server: WebPubSubServiceClient

### Xác thực

```typescript
import { WebPubSubServiceClient, AzureKeyCredential } from "@azure/web-pubsub";
import { DefaultAzureCredential } from "@azure/identity";

// Chuỗi kết nối
const client = new WebPubSubServiceClient(
  process.env.WEBPUBSUB_CONNECTION_STRING!,
  "chat", // tên hub
);

// DefaultAzureCredential (khuyên dùng)
const client2 = new WebPubSubServiceClient(
  process.env.WEBPUBSUB_ENDPOINT!,
  new DefaultAzureCredential(),
  "chat",
);

// AzureKeyCredential
const client3 = new WebPubSubServiceClient(
  process.env.WEBPUBSUB_ENDPOINT!,
  new AzureKeyCredential("<access-key>"),
  "chat",
);
```

### Tạo Client Access Token

```typescript
// Token cơ bản
const token = await client.getClientAccessToken();
console.log(token.url); // wss://...?access_token=...

// Token với user ID
const userToken = await client.getClientAccessToken({
  userId: "user123",
});

// Token với các quyền
const permToken = await client.getClientAccessToken({
  userId: "user123",
  roles: [
    "webpubsub.joinLeaveGroup",
    "webpubsub.sendToGroup",
    "webpubsub.sendToGroup.chat-room", // nhóm cụ thể
  ],
  groups: ["chat-room"], // tự động tham gia khi kết nối
  expirationTimeInMinutes: 60,
});
```

### Gửi Tin nhắn

```typescript
// Phát sóng tới tất cả kết nối trong hub
await client.sendToAll({ message: "Hello everyone!" });
await client.sendToAll("Plain text", { contentType: "text/plain" });

// Gửi tới user cụ thể (tất cả kết nối của họ)
await client.sendToUser("user123", { message: "Hello!" });

// Gửi tới kết nối cụ thể
await client.sendToConnection("connectionId", { data: "Direct message" });

// Gửi với bộ lọc (cú pháp OData)
await client.sendToAll(
  { message: "Filtered" },
  {
    filter: "userId ne 'admin'",
  },
);
```

### Quản lý Nhóm

```typescript
const group = client.group("chat-room");

// Thêm user/kết nối vào nhóm
await group.addUser("user123");
await group.addConnection("connectionId");

// Xóa khỏi nhóm
await group.removeUser("user123");

// Gửi tới nhóm
await group.sendToAll({ message: "Group message" });

// Đóng tất cả kết nối trong nhóm
await group.closeAllConnections({ reason: "Maintenance" });
```

### Quản lý Kết nối

```typescript
// Kiểm tra sự tồn tại
const userExists = await client.userExists("user123");
const connExists = await client.connectionExists("connectionId");

// Đóng kết nối
await client.closeConnection("connectionId", { reason: "Kicked" });
await client.closeUserConnections("user123");
await client.closeAllConnections();

// Cấp quyền
await client.grantPermission("connectionId", "sendToGroup", {
  targetName: "chat",
});
await client.revokePermission("connectionId", "sendToGroup", {
  targetName: "chat",
});
```

## Phía Client: WebPubSubClient

### Kết nối

```typescript
import { WebPubSubClient } from "@azure/web-pubsub-client";

// URL trực tiếp
const client = new WebPubSubClient("<client-access-url>");

// URL động từ negotiate endpoint
const client2 = new WebPubSubClient({
  getClientAccessUrl: async () => {
    const response = await fetch("/negotiate");
    const { url } = await response.json();
    return url;
  },
});

// Đăng ký handlers TRƯỚC KHI bắt đầu
client.on("connected", (e) => {
  console.log(`Connected: ${e.connectionId}`);
});

client.on("group-message", (e) => {
  console.log(`${e.message.group}: ${e.message.data}`);
});

await client.start();
```

### Gửi Tin nhắn

```typescript
// Tham gia nhóm trước
await client.joinGroup("chat-room");

// Gửi tới nhóm
await client.sendToGroup("chat-room", "Hello!", "text");
await client.sendToGroup(
  "chat-room",
  { type: "message", content: "Hi" },
  "json",
);

// Tùy chọn gửi
await client.sendToGroup("chat-room", "Hello", "text", {
  noEcho: true, // Không vọng lại người gửi
  fireAndForget: true, // Không đợi xác nhận (ack)
});

// Gửi sự kiện tới server
await client.sendEvent("userAction", { action: "typing" }, "json");
```

### Event Handlers

```typescript
// Vòng đời kết nối
client.on("connected", (e) => {
  console.log(`Connected: ${e.connectionId}, User: ${e.userId}`);
});

client.on("disconnected", (e) => {
  console.log(`Disconnected: ${e.message}`);
});

client.on("stopped", () => {
  console.log("Client stopped");
});

// Tin nhắn
client.on("group-message", (e) => {
  console.log(
    `[${e.message.group}] ${e.message.fromUserId}: ${e.message.data}`,
  );
});

client.on("server-message", (e) => {
  console.log(`Server: ${e.message.data}`);
});

// Lỗi khi tham gia lại
client.on("rejoin-group-failed", (e) => {
  console.log(`Failed to rejoin ${e.group}: ${e.error}`);
});
```

## Express Event Handler

```typescript
import express from "express";
import { WebPubSubEventHandler } from "@azure/web-pubsub-express";

const app = express();

const handler = new WebPubSubEventHandler("chat", {
  path: "/api/webpubsub/hubs/chat/",

  // Blocking: chấp nhận/từ chối kết nối
  handleConnect: (req, res) => {
    if (!req.claims?.sub) {
      res.fail(401, "Authentication required");
      return;
    }
    res.success({
      userId: req.claims.sub[0],
      groups: ["general"],
      roles: ["webpubsub.sendToGroup"],
    });
  },

  // Blocking: xử lý sự kiện tùy chỉnh
  handleUserEvent: (req, res) => {
    console.log(`Event from ${req.context.userId}:`, req.data);
    res.success(`Received: ${req.data}`, "text");
  },

  // Non-blocking
  onConnected: (req) => {
    console.log(`Client connected: ${req.context.connectionId}`);
  },

  onDisconnected: (req) => {
    console.log(`Client disconnected: ${req.context.connectionId}`);
  },
});

app.use(handler.getMiddleware());

// Negotiate endpoint
app.get("/negotiate", async (req, res) => {
  const token = await serviceClient.getClientAccessToken({
    userId: req.user?.id,
  });
  res.json({ url: token.url });
});

app.listen(8080);
```

## Các Loại Chính (Key Types)

```typescript
// Server
import {
  WebPubSubServiceClient,
  WebPubSubGroup,
  GenerateClientTokenOptions,
  HubSendToAllOptions,
} from "@azure/web-pubsub";

// Client
import {
  WebPubSubClient,
  WebPubSubClientOptions,
  OnConnectedArgs,
  OnGroupDataMessageArgs,
} from "@azure/web-pubsub-client";

// Express
import {
  WebPubSubEventHandler,
  ConnectRequest,
  UserEventRequest,
  ConnectResponseHandler,
} from "@azure/web-pubsub-express";
```

## Thực hành Tốt nhất

1. **Sử dụng xác thực Entra ID** - `DefaultAzureCredential` cho production
2. **Đăng ký handlers trước khi bắt đầu** - Đừng bỏ lỡ các sự kiện ban đầu
3. **Sử dụng groups cho channels** - Tổ chức tin nhắn theo chủ đề/phòng
4. **Xử lý kết nối lại** - Client tự động kết nối lại theo mặc định
5. **Validate trong handleConnect** - Từ chối kết nối không hợp lệ sớm
6. **Sử dụng noEcho** - Ngăn tin nhắn vọng lại người gửi khi cần thiết
