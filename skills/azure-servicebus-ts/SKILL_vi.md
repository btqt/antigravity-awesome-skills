---
name: azure-servicebus-ts
description: Xây dựng các ứng dụng nhắn tin sử dụng Azure Service Bus SDK cho JavaScript (@azure/service-bus). Sử dụng khi triển khai queues, topics/subscriptions, message sessions, xử lý dead-letter, hoặc các mẫu nhắn tin doanh nghiệp.
package: @azure/service-bus
---

# Azure Service Bus SDK cho TypeScript

Nhắn tin doanh nghiệp với queues, topics, và subscriptions.

## Cài đặt

```bash
npm install @azure/service-bus @azure/identity
```

## Biến Môi trường

```bash
SERVICEBUS_NAMESPACE=<namespace>.servicebus.windows.net
SERVICEBUS_QUEUE_NAME=my-queue
SERVICEBUS_TOPIC_NAME=my-topic
SERVICEBUS_SUBSCRIPTION_NAME=my-subscription
```

## Xác thực

```typescript
import { ServiceBusClient } from "@azure/service-bus";
import { DefaultAzureCredential } from "@azure/identity";

const fullyQualifiedNamespace = process.env.SERVICEBUS_NAMESPACE!;
const client = new ServiceBusClient(
  fullyQualifiedNamespace,
  new DefaultAzureCredential(),
);
```

## Quy trình Công việc Cốt lõi

### Gửi Tin nhắn đến Queue

```typescript
const sender = client.createSender("my-queue");

// Tin nhắn đơn lẻ
await sender.sendMessages({
  body: { orderId: "12345", amount: 99.99 },
  contentType: "application/json",
});

// Tin nhắn theo lô
const batch = await sender.createMessageBatch();
batch.tryAddMessage({ body: "Message 1" });
batch.tryAddMessage({ body: "Message 2" });
await sender.sendMessages(batch);

await sender.close();
```

### Nhận Tin nhắn từ Queue

```typescript
const receiver = client.createReceiver("my-queue");

// Nhận theo lô
const messages = await receiver.receiveMessages(10, { maxWaitTimeInMs: 5000 });
for (const message of messages) {
  console.log(`Received: ${message.body}`);
  await receiver.completeMessage(message);
}

await receiver.close();
```

### Đăng ký nhận Tin nhắn (Hướng sự kiện)

```typescript
const receiver = client.createReceiver("my-queue");

const subscription = receiver.subscribe({
  processMessage: async (message) => {
    console.log(`Processing: ${message.body}`);
    // Tự động hoàn thành tin nhắn khi thành công
  },
  processError: async (args) => {
    console.error(`Error: ${args.error}`);
  },
});

// Dừng sau một khoảng thời gian
setTimeout(async () => {
  await subscription.close();
  await receiver.close();
}, 60000);
```

### Topics và Subscriptions

```typescript
// Gửi đến topic
const topicSender = client.createSender("my-topic");
await topicSender.sendMessages({
  body: { event: "order.created", data: { orderId: "123" } },
  applicationProperties: { eventType: "order.created" },
});

// Nhận từ subscription
const subscriptionReceiver = client.createReceiver(
  "my-topic",
  "my-subscription",
);
const messages = await subscriptionReceiver.receiveMessages(10);
```

## Message Sessions

```typescript
// Gửi tin nhắn session
const sender = client.createSender("session-queue");
await sender.sendMessages({
  body: { step: 1, data: "First step" },
  sessionId: "workflow-123",
});

// Nhận tin nhắn session
const sessionReceiver = await client.acceptSession(
  "session-queue",
  "workflow-123",
);
const messages = await sessionReceiver.receiveMessages(10);

// Lấy/đặt trạng thái session
const state = await sessionReceiver.getSessionState();
await sessionReceiver.setSessionState(
  Buffer.from(JSON.stringify({ progress: 50 })),
);

await sessionReceiver.close();
```

## Xử lý Dead-Letter

```typescript
// Di chuyển đến dead-letter
await receiver.deadLetterMessage(message, {
  deadLetterReason: "Validation failed",
  deadLetterErrorDescription: "Missing required field: orderId",
});

// Xử lý dead-letter queue
const dlqReceiver = client.createReceiver("my-queue", {
  subQueueType: "deadLetter",
});
const dlqMessages = await dlqReceiver.receiveMessages(10);
for (const msg of dlqMessages) {
  console.log(`DLQ Reason: ${msg.deadLetterReason}`);
  // Xử lý lại hoặc log
  await dlqReceiver.completeMessage(msg);
}
```

## Tin nhắn được Lên lịch

```typescript
const sender = client.createSender("my-queue");

// Lên lịch để gửi trong tương lai
const scheduledTime = new Date(Date.now() + 60000); // 1 phút kể từ bây giờ
const sequenceNumber = await sender.scheduleMessages(
  { body: "Delayed message" },
  scheduledTime,
);

// Hủy tin nhắn đã lên lịch
await sender.cancelScheduledMessages(sequenceNumber);
```

## Trì hoãn Tin nhắn (Message Deferral)

```typescript
// Trì hoãn tin nhắn để xử lý sau
await receiver.deferMessage(message);

// Nhận tin nhắn bị trì hoãn bằng số thứ tự (sequence number)
const deferredMessage = await receiver.receiveDeferredMessages(
  message.sequenceNumber!,
);
await receiver.completeMessage(deferredMessage[0]);
```

## Peek Tin nhắn (Không phá hủy)

```typescript
const receiver = client.createReceiver("my-queue");

// Peek mà không xóa
const peekedMessages = await receiver.peekMessages(10);
for (const msg of peekedMessages) {
  console.log(`Peeked: ${msg.body}`);
}
```

## Các Loại Chính

```typescript
import {
  ServiceBusClient,
  ServiceBusSender,
  ServiceBusReceiver,
  ServiceBusSessionReceiver,
  ServiceBusMessage,
  ServiceBusReceivedMessage,
  ProcessMessageCallback,
  ProcessErrorCallback,
} from "@azure/service-bus";
```

## Các Chế độ Nhận

```typescript
// Peek-Lock (mặc định) - tin nhắn bị khóa cho đến khi hoàn thành/từ bỏ
const receiver = client.createReceiver("my-queue", { receiveMode: "peekLock" });
await receiver.completeMessage(message); // Xóa khỏi queue
await receiver.abandonMessage(message); // Trả về queue
await receiver.deferMessage(message); // Trì hoãn
await receiver.deadLetterMessage(message); // Di chuyển đến DLQ

// Receive-and-Delete - tin nhắn bị xóa ngay lập tức
const receiver = client.createReceiver("my-queue", {
  receiveMode: "receiveAndDelete",
});
```

## Thực hành Tốt nhất

1. **Sử dụng xác thực Entra ID** - Tránh dùng chuỗi kết nối trong production
2. **Tái sử dụng clients** - Tạo `ServiceBusClient` một lần, chia sẻ giữa các senders/receivers
3. **Đóng tài nguyên** - Luôn đóng senders/receivers khi xong việc
4. **Xử lý lỗi** - Triển khai `processError` callback cho subscription receivers
5. **Sử dụng sessions để sắp xếp** - Khi thứ tự tin nhắn quan trọng trong một nhóm
6. **Cấu hình dead-letter** - Luôn xử lý các tin nhắn DLQ
7. **Gửi theo lô** - Sử dụng `createMessageBatch()` cho nhiều tin nhắn

## Tài liệu Tham khảo

Để biết các mẫu chi tiết, xem:

- [Queues vs Topics Patterns](references/queues-topics.md) - Các mẫu queue/topic, sessions, chế độ nhận, giải quyết tin nhắn
- [Error Handling and Reliability](references/error-handling.md) - Mã lỗi ServiceBusError, xử lý DLQ, gia hạn khóa, tắt máy an toàn
