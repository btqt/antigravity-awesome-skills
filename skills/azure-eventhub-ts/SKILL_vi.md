---
name: azure-eventhub-ts
description: Xây dựng các ứng dụng streaming sự kiện sử dụng Azure Event Hubs SDK cho JavaScript (@azure/event-hubs). Sử dụng khi triển khai nhập sự kiện thông lượng cao, analytics thời gian thực, đo từ xa IoT (IoT telemetry), hoặc các kiến trúc hướng sự kiện với partitioned consumers.
package: @azure/event-hubs
---

# Azure Event Hubs SDK cho TypeScript

Streaming sự kiện thông lượng cao và nhập dữ liệu thời gian thực.

## Cài đặt

```bash
npm install @azure/event-hubs @azure/identity
```

Cho checkpointing với consumer groups:

```bash
npm install @azure/eventhubs-checkpointstore-blob @azure/storage-blob
```

## Biến Môi trường

```bash
EVENTHUB_NAMESPACE=<namespace>.servicebus.windows.net
EVENTHUB_NAME=my-eventhub
STORAGE_ACCOUNT_NAME=<storage-account>
STORAGE_CONTAINER_NAME=checkpoints
```

## Xác thực

```typescript
import {
  EventHubProducerClient,
  EventHubConsumerClient,
} from "@azure/event-hubs";
import { DefaultAzureCredential } from "@azure/identity";

const fullyQualifiedNamespace = process.env.EVENTHUB_NAMESPACE!;
const eventHubName = process.env.EVENTHUB_NAME!;
const credential = new DefaultAzureCredential();

// Producer
const producer = new EventHubProducerClient(
  fullyQualifiedNamespace,
  eventHubName,
  credential,
);

// Consumer
const consumer = new EventHubConsumerClient(
  "$Default", // Consumer group
  fullyQualifiedNamespace,
  eventHubName,
  credential,
);
```

## Quy trình Cốt lõi

### Gửi Sự kiện

```typescript
const producer = new EventHubProducerClient(
  namespace,
  eventHubName,
  credential,
);

// Tạo lô và thêm sự kiện
const batch = await producer.createBatch();
batch.tryAdd({ body: { temperature: 72.5, deviceId: "sensor-1" } });
batch.tryAdd({ body: { temperature: 68.2, deviceId: "sensor-2" } });

await producer.sendBatch(batch);
await producer.close();
```

### Gửi đến Partition Cụ thể

```typescript
// Theo partition ID
const batch = await producer.createBatch({ partitionId: "0" });

// Theo partition key (consistent hashing)
const batch = await producer.createBatch({ partitionKey: "device-123" });
```

### Nhận Sự kiện (Đơn giản)

```typescript
const consumer = new EventHubConsumerClient(
  "$Default",
  namespace,
  eventHubName,
  credential,
);

const subscription = consumer.subscribe({
  processEvents: async (events, context) => {
    for (const event of events) {
      console.log(
        `Partition: ${context.partitionId}, Body: ${JSON.stringify(event.body)}`,
      );
    }
  },
  processError: async (err, context) => {
    console.error(`Error on partition ${context.partitionId}: ${err.message}`);
  },
});

// Dừng sau một khoảng thời gian
setTimeout(async () => {
  await subscription.close();
  await consumer.close();
}, 60000);
```

### Nhận với Checkpointing (Sản xuất)

```typescript
import { EventHubConsumerClient } from "@azure/event-hubs";
import { ContainerClient } from "@azure/storage-blob";
import { BlobCheckpointStore } from "@azure/eventhubs-checkpointstore-blob";

const containerClient = new ContainerClient(
  `https://${storageAccount}.blob.core.windows.net/${containerName}`,
  credential,
);

const checkpointStore = new BlobCheckpointStore(containerClient);

const consumer = new EventHubConsumerClient(
  "$Default",
  namespace,
  eventHubName,
  credential,
  checkpointStore,
);

const subscription = consumer.subscribe({
  processEvents: async (events, context) => {
    for (const event of events) {
      console.log(`Processing: ${JSON.stringify(event.body)}`);
    }
    // Checkpoint sau khi xử lý lô
    if (events.length > 0) {
      await context.updateCheckpoint(events[events.length - 1]);
    }
  },
  processError: async (err, context) => {
    console.error(`Error: ${err.message}`);
  },
});
```

### Nhận từ Vị trí Cụ thể

```typescript
const subscription = consumer.subscribe(
  {
    processEvents: async (events, context) => {
      /* ... */
    },
    processError: async (err, context) => {
      /* ... */
    },
  },
  {
    startPosition: {
      // Bắt đầu từ đầu
      "0": { offset: "@earliest" },
      // Bắt đầu từ cuối (chỉ sự kiện mới)
      "1": { offset: "@latest" },
      // Bắt đầu từ offset cụ thể
      "2": { offset: "12345" },
      // Bắt đầu từ thời gian cụ thể
      "3": { enqueuedOn: new Date("2024-01-01") },
    },
  },
);
```

## Thuộc tính Event Hub

```typescript
// Lấy thông tin hub
const hubProperties = await producer.getEventHubProperties();
console.log(`Partitions: ${hubProperties.partitionIds}`);

// Lấy thông tin partition
const partitionProperties = await producer.getPartitionProperties("0");
console.log(`Last sequence: ${partitionProperties.lastEnqueuedSequenceNumber}`);
```

## Tùy chọn Xử lý Lô

```typescript
const subscription = consumer.subscribe(
  {
    processEvents: async (events, context) => {
      /* ... */
    },
    processError: async (err, context) => {
      /* ... */
    },
  },
  {
    maxBatchSize: 100, // Sự kiện tối đa mỗi lô
    maxWaitTimeInSeconds: 30, // Thời gian chờ tối đa cho lô
  },
);
```

## Các Loại Chính

```typescript
import {
  EventHubProducerClient,
  EventHubConsumerClient,
  EventData,
  ReceivedEventData,
  PartitionContext,
  Subscription,
  SubscriptionEventHandlers,
  CreateBatchOptions,
  EventPosition,
} from "@azure/event-hubs";

import { BlobCheckpointStore } from "@azure/eventhubs-checkpointstore-blob";
```

## Thuộc tính Sự kiện

```typescript
// Gửi với thuộc tính
const batch = await producer.createBatch();
batch.tryAdd({
  body: { data: "payload" },
  properties: {
    eventType: "telemetry",
    deviceId: "sensor-1",
  },
  contentType: "application/json",
  correlationId: "request-123",
});

// Truy cập trong receiver
consumer.subscribe({
  processEvents: async (events, context) => {
    for (const event of events) {
      console.log(`Type: ${event.properties?.eventType}`);
      console.log(`Sequence: ${event.sequenceNumber}`);
      console.log(`Enqueued: ${event.enqueuedTimeUtc}`);
      console.log(`Offset: ${event.offset}`);
    }
  },
});
```

## Xử lý Lỗi

```typescript
consumer.subscribe({
  processEvents: async (events, context) => {
    try {
      for (const event of events) {
        await processEvent(event);
      }
      await context.updateCheckpoint(events[events.length - 1]);
    } catch (error) {
      // Đừng checkpoint khi lỗi - sự kiện sẽ được xử lý lại
      console.error("Processing failed:", error);
    }
  },
  processError: async (err, context) => {
    if (err.name === "MessagingError") {
      // Lỗi thoáng qua - SDK sẽ thử lại
      console.warn("Transient error:", err.message);
    } else {
      // Lỗi nghiêm trọng
      console.error("Fatal error:", err);
    }
  },
});
```

## Thực hành Tốt nhất

1. **Sử dụng checkpointing** - Luôn checkpoint trong sản xuất để xử lý chính xác một lần (exactly-once processing)
2. **Gửi lô** - Sử dụng `createBatch()` để gửi hiệu quả
3. **Partition keys** - Sử dụng partition keys để đảm bảo thứ tự cho các sự kiện liên quan
4. **Consumer groups** - Sử dụng các consumer groups riêng biệt cho các pipeline xử lý khác nhau
5. **Xử lý lỗi duyên dáng** - Đừng checkpoint khi xử lý thất bại
6. **Đóng clients** - Luôn đóng producer/consumer khi xong
7. **Giám sát độ trễ (lag)** - Theo dõi `lastEnqueuedSequenceNumber` so với sequence đã xử lý
