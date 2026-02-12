---
name: azure-storage-queue-ts
description: |
  Azure Queue Storage JavaScript/TypeScript SDK (@azure/storage-queue) cho các hoạt động hàng đợi tin nhắn. Sử dụng để gửi, nhận, peek, và xóa tin nhắn trong hàng đợi. Hỗ trợ visibility timeout, mã hóa tin nhắn, và các hoạt động theo lô. Triggers: "queue storage", "@azure/storage-queue", "QueueServiceClient", "QueueClient", "send message", "receive message", "dequeue", "visibility timeout".
package: @azure/storage-queue
---

# @azure/storage-queue (TypeScript/JavaScript)

SDK cho các hoạt động Azure Queue Storage — gửi, nhận, peek, và quản lý tin nhắn trong hàng đợi.

## Cài đặt

```bash
npm install @azure/storage-queue @azure/identity
```

**Phiên bản Hiện tại**: 12.x
**Node.js**: >= 18.0.0

## Biến Môi trường

```bash
AZURE_STORAGE_ACCOUNT_NAME=<account-name>
AZURE_STORAGE_ACCOUNT_KEY=<account-key>
# HOẶC chuỗi kết nối
AZURE_STORAGE_CONNECTION_STRING=DefaultEndpointsProtocol=https;AccountName=...
```

## Xác thực

### DefaultAzureCredential (Khuyên dùng)

```typescript
import { QueueServiceClient } from "@azure/storage-queue";
import { DefaultAzureCredential } from "@azure/identity";

const accountName = process.env.AZURE_STORAGE_ACCOUNT_NAME!;
const client = new QueueServiceClient(
  `https://${accountName}.queue.core.windows.net`,
  new DefaultAzureCredential(),
);
```

### Chuỗi Kết nối

```typescript
import { QueueServiceClient } from "@azure/storage-queue";

const client = QueueServiceClient.fromConnectionString(
  process.env.AZURE_STORAGE_CONNECTION_STRING!,
);
```

### StorageSharedKeyCredential (Chỉ Node.js)

```typescript
import {
  QueueServiceClient,
  StorageSharedKeyCredential,
} from "@azure/storage-queue";

const accountName = process.env.AZURE_STORAGE_ACCOUNT_NAME!;
const accountKey = process.env.AZURE_STORAGE_ACCOUNT_KEY!;

const sharedKeyCredential = new StorageSharedKeyCredential(
  accountName,
  accountKey,
);
const client = new QueueServiceClient(
  `https://${accountName}.queue.core.windows.net`,
  sharedKeyCredential,
);
```

### SAS Token

```typescript
import { QueueServiceClient } from "@azure/storage-queue";

const accountName = process.env.AZURE_STORAGE_ACCOUNT_NAME!;
const sasToken = process.env.AZURE_STORAGE_SAS_TOKEN!;

const client = new QueueServiceClient(
  `https://${accountName}.queue.core.windows.net${sasToken}`,
);
```

## Phân cấp Client

```
QueueServiceClient (cấp tài khoản)
└── QueueClient (cấp hàng đợi)
    └── Messages (gửi, nhận, peek, xóa)
```

## Các Hoạt động Queue

### Tạo Queue

```typescript
const queueClient = client.getQueueClient("my-queue");
await queueClient.create();

// Hoặc tạo nếu không tồn tại
await queueClient.createIfNotExists();
```

### Liệt kê Queues

```typescript
for await (const queue of client.listQueues()) {
  console.log(queue.name);
}

// Với bộ lọc prefix
for await (const queue of client.listQueues({ prefix: "task-" })) {
  console.log(queue.name);
}
```

### Xóa Queue

```typescript
await queueClient.delete();

// Hoặc xóa nếu tồn tại
await queueClient.deleteIfExists();
```

### Lấy Thuộc tính Queue

```typescript
const properties = await queueClient.getProperties();
console.log("Approximate message count:", properties.approximateMessagesCount);
console.log("Metadata:", properties.metadata);
```

### Đặt Queue Metadata

```typescript
await queueClient.setMetadata({
  department: "engineering",
  priority: "high",
});
```

## Các Hoạt động Tin nhắn

### Gửi Tin nhắn

```typescript
const queueClient = client.getQueueClient("my-queue");

// Tin nhắn đơn giản
await queueClient.sendMessage("Hello, World!");

// Với các tùy chọn
await queueClient.sendMessage("Delayed message", {
  visibilityTimeout: 60, // Ẩn trong 60 giây
  messageTimeToLive: 3600, // Hết hạn trong 1 giờ
});

// Tin nhắn JSON (phải là chuỗi)
const task = { type: "process", data: { id: 123 } };
await queueClient.sendMessage(JSON.stringify(task));
```

### Nhận Tin nhắn

```typescript
// Nhận tối đa 32 tin nhắn (mặc định: 1)
const response = await queueClient.receiveMessages({
  numberOfMessages: 10,
  visibilityTimeout: 30, // 30 giây để xử lý
});

for (const message of response.receivedMessageItems) {
  console.log("Message ID:", message.messageId);
  console.log("Content:", message.messageText);
  console.log("Dequeue Count:", message.dequeueCount);
  console.log("Pop Receipt:", message.popReceipt);

  // Xử lý tin nhắn...

  // Xóa sau khi xử lý
  await queueClient.deleteMessage(message.messageId, message.popReceipt);
}
```

### Peek Tin nhắn

Peek mà không xóa khỏi hàng đợi (không có visibility timeout).

```typescript
const response = await queueClient.peekMessages({
  numberOfMessages: 5,
});

for (const message of response.peekedMessageItems) {
  console.log("Message ID:", message.messageId);
  console.log("Content:", message.messageText);
  // Lưu ý: Không có popReceipt - không thể xóa tin nhắn đã peek
}
```

### Cập nhật Tin nhắn

Gia hạn visibility timeout hoặc cập nhật nội dung.

```typescript
// Nhận một tin nhắn
const response = await queueClient.receiveMessages();
const message = response.receivedMessageItems[0];

if (message) {
  // Cập nhật nội dung và gia hạn visibility
  const updateResponse = await queueClient.updateMessage(
    message.messageId,
    message.popReceipt,
    "Updated content",
    60, // Visibility timeout mới tính bằng giây
  );

  // Sử dụng popReceipt mới cho các hoạt động tiếp theo
  console.log("New pop receipt:", updateResponse.popReceipt);
}
```

### Xóa Tin nhắn

```typescript
// Sau khi nhận
const response = await queueClient.receiveMessages();
const message = response.receivedMessageItems[0];

if (message) {
  await queueClient.deleteMessage(message.messageId, message.popReceipt);
}
```

### Xóa Tất cả Tin nhắn (Clear)

```typescript
await queueClient.clearMessages();
```

## Các Mẫu Xử lý Tin nhắn

### Mẫu Worker Cơ bản

```typescript
async function processQueue(queueClient: QueueClient): Promise<void> {
  while (true) {
    const response = await queueClient.receiveMessages({
      numberOfMessages: 10,
      visibilityTimeout: 30,
    });

    if (response.receivedMessageItems.length === 0) {
      // Không có tin nhắn, đợi trước khi poll lại
      await sleep(5000);
      continue;
    }

    for (const message of response.receivedMessageItems) {
      try {
        await processMessage(message.messageText);
        await queueClient.deleteMessage(message.messageId, message.popReceipt);
      } catch (error) {
        console.error(`Failed to process message ${message.messageId}:`, error);
        // Tin nhắn sẽ hiển thị lại sau khi timeout
      }
    }
  }
}

async function processMessage(content: string): Promise<void> {
  const task = JSON.parse(content);
  // Xử lý tác vụ...
}

function sleep(ms: number): Promise<void> {
  return new Promise((resolve) => setTimeout(resolve, ms));
}
```

### Xử lý Tin nhắn Lỗi (Poison Message)

```typescript
const MAX_DEQUEUE_COUNT = 5;

async function processWithPoisonHandling(
  queueClient: QueueClient,
  poisonQueueClient: QueueClient,
): Promise<void> {
  const response = await queueClient.receiveMessages({
    numberOfMessages: 10,
    visibilityTimeout: 30,
  });

  for (const message of response.receivedMessageItems) {
    if (message.dequeueCount > MAX_DEQUEUE_COUNT) {
      // Di chuyển đến poison queue
      await poisonQueueClient.sendMessage(message.messageText);
      await queueClient.deleteMessage(message.messageId, message.popReceipt);
      console.log(`Moved message ${message.messageId} to poison queue`);
      continue;
    }

    try {
      await processMessage(message.messageText);
      await queueClient.deleteMessage(message.messageId, message.popReceipt);
    } catch (error) {
      console.error(
        `Processing failed (attempt ${message.dequeueCount}):`,
        error,
      );
    }
  }
}
```

### Xử lý Lô với Gia hạn Visibility

```typescript
async function processBatchWithExtension(
  queueClient: QueueClient,
): Promise<void> {
  const response = await queueClient.receiveMessages({
    numberOfMessages: 1,
    visibilityTimeout: 60,
  });

  const message = response.receivedMessageItems[0];
  if (!message) return;

  let popReceipt = message.popReceipt;

  // Bắt đầu timer gia hạn visibility
  const extensionInterval = setInterval(async () => {
    try {
      const updateResponse = await queueClient.updateMessage(
        message.messageId,
        popReceipt,
        message.messageText,
        60, // Gia hạn thêm 60 giây
      );
      popReceipt = updateResponse.popReceipt;
    } catch (error) {
      console.error("Failed to extend visibility:", error);
    }
  }, 45000); // Gia hạn mỗi 45 giây

  try {
    await longRunningProcess(message.messageText);
    await queueClient.deleteMessage(message.messageId, popReceipt);
  } finally {
    clearInterval(extensionInterval);
  }
}
```

## Mã hóa Tin nhắn

Theo mặc định, tin nhắn được mã hóa Base64. Bạn có thể tùy chỉnh điều này:

```typescript
import { QueueClient } from "@azure/storage-queue";

// Encoder/decoder tùy chỉnh cho văn bản thuần túy
const queueClient = new QueueClient(
  `https://${accountName}.queue.core.windows.net/my-queue`,
  credential,
  {
    messageEncoding: "text", // "base64" (mặc định) hoặc "text"
  },
);

// Hoặc với encoder tùy chỉnh
const customQueueClient = new QueueClient(
  `https://${accountName}.queue.core.windows.net/my-queue`,
  credential,
  {
    messageEncoding: {
      encode: (message: string) => Buffer.from(message).toString("base64"),
      decode: (message: string) => Buffer.from(message, "base64").toString(),
    },
  },
);
```

## Tạo SAS Token (Chỉ Node.js)

### Tạo Queue SAS

```typescript
import {
  QueueSASPermissions,
  generateQueueSASQueryParameters,
  StorageSharedKeyCredential,
} from "@azure/storage-queue";

const sharedKeyCredential = new StorageSharedKeyCredential(
  accountName,
  accountKey,
);

const sasToken = generateQueueSASQueryParameters(
  {
    queueName: "my-queue",
    permissions: QueueSASPermissions.parse("raup"), // read, add, update, process
    startsOn: new Date(),
    expiresOn: new Date(Date.now() + 3600 * 1000), // 1 giờ
  },
  sharedKeyCredential,
).toString();

const sasUrl = `https://${accountName}.queue.core.windows.net/my-queue?${sasToken}`;
```

### Tạo Account SAS

```typescript
import {
  AccountSASPermissions,
  AccountSASResourceTypes,
  AccountSASServices,
  generateAccountSASQueryParameters,
} from "@azure/storage-queue";

const sasToken = generateAccountSASQueryParameters(
  {
    services: AccountSASServices.parse("q").toString(), // queue
    resourceTypes: AccountSASResourceTypes.parse("sco").toString(),
    permissions: AccountSASPermissions.parse("rwdlacupi"),
    expiresOn: new Date(Date.now() + 24 * 3600 * 1000),
  },
  sharedKeyCredential,
).toString();
```

## Xử lý Lỗi

```typescript
import { RestError } from "@azure/storage-queue";

try {
  await queueClient.sendMessage("test");
} catch (error) {
  if (error instanceof RestError) {
    switch (error.statusCode) {
      case 404:
        console.log("Queue not found");
        break;
      case 400:
        console.log("Bad request - message too large or invalid");
        break;
      case 403:
        console.log("Access denied");
        break;
      case 409:
        console.log("Queue already exists or being deleted");
        break;
      default:
        console.error(`Storage error ${error.statusCode}: ${error.message}`);
    }
  }
  throw error;
}
```

## Tham chiếu Types trong TypeScript

```typescript
import {
  // Clients
  QueueServiceClient,
  QueueClient,

  // Authentication
  StorageSharedKeyCredential,
  AnonymousCredential,

  // SAS
  QueueSASPermissions,
  AccountSASPermissions,
  AccountSASServices,
  AccountSASResourceTypes,
  generateQueueSASQueryParameters,
  generateAccountSASQueryParameters,

  // Messages
  DequeuedMessageItem,
  PeekedMessageItem,
  QueueSendMessageResponse,
  QueueReceiveMessageResponse,
  QueueUpdateMessageResponse,

  // Queue
  QueueItem,
  QueueGetPropertiesResponse,

  // Errors
  RestError,
} from "@azure/storage-queue";
```

## Giới hạn Tin nhắn

| Giới hạn                     | Giá trị                     |
| ---------------------------- | --------------------------- |
| Kích thước tin nhắn tối đa   | 64 KB                       |
| Visibility timeout tối đa    | 7 ngày                      |
| Time-to-live tối đa          | 7 ngày (hoặc -1 cho vô hạn) |
| Tin nhắn tối đa mỗi lần nhận | 32                          |
| Visibility timeout mặc định  | 30 giây                     |

## Thực hành Tốt nhất

1. **Sử dụng DefaultAzureCredential** — Ưu tiên AAD hơn chuỗi kết nối/keys
2. **Luôn xóa sau khi xử lý** — Ngăn chặn xử lý trùng lặp
3. **Xử lý tin nhắn lỗi (poison messages)** — Di chuyển tin nhắn bị lỗi đến dead-letter queue
4. **Sử dụng visibility timeout phù hợp** — Đặt dựa trên thời gian xử lý dự kiến
5. **Gia hạn visibility cho các tác vụ dài** — Cập nhật tin nhắn để ngăn chặn timeout
6. **Sử dụng JSON cho dữ liệu có cấu trúc** — Serialize đối tượng thành chuỗi JSON
7. **Kiểm tra dequeueCount** — Phát hiện các tin nhắn bị lỗi lặp lại
8. **Sử dụng nhận theo lô** — Nhận nhiều tin nhắn để tăng hiệu quả

## Khác biệt Nền tảng

| Tính năng                    | Node.js | Browser |
| ---------------------------- | ------- | ------- |
| `StorageSharedKeyCredential` | ✅      | ❌      |
| SAS generation               | ✅      | ❌      |
| DefaultAzureCredential       | ✅      | ❌      |
| Anonymous/SAS access         | ✅      | ✅      |
| All message operations       | ✅      | ✅      |
