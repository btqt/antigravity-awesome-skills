---
name: azure-eventhub-java
description: Xây dựng các ứng dụng streaming thời gian thực với Azure Event Hubs SDK cho Java. Sử dụng khi triển khai streaming sự kiện, nhập dữ liệu thông lượng cao, hoặc xây dựng các kiến trúc hướng sự kiện.
package: com.azure:azure-messaging-eventhubs
---

# Azure Event Hubs SDK cho Java

Xây dựng các ứng dụng streaming thời gian thực sử dụng Azure Event Hubs SDK cho Java.

## Cài đặt

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-messaging-eventhubs</artifactId>
    <version>5.19.0</version>
</dependency>

<!-- Cho checkpoint store (sản xuất) -->
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-messaging-eventhubs-checkpointstore-blob</artifactId>
    <version>1.20.0</version>
</dependency>
```

## Tạo Client

### EventHubProducerClient

```java
import com.azure.messaging.eventhubs.EventHubProducerClient;
import com.azure.messaging.eventhubs.EventHubClientBuilder;

// Với connection string
EventHubProducerClient producer = new EventHubClientBuilder()
    .connectionString("<connection-string>", "<event-hub-name>")
    .buildProducerClient();

// Connection string đầy đủ với EntityPath
EventHubProducerClient producer = new EventHubClientBuilder()
    .connectionString("<connection-string-with-entity-path>")
    .buildProducerClient();
```

### Với DefaultAzureCredential

```java
import com.azure.identity.DefaultAzureCredentialBuilder;

EventHubProducerClient producer = new EventHubClientBuilder()
    .fullyQualifiedNamespace("<namespace>.servicebus.windows.net")
    .eventHubName("<event-hub-name>")
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildProducerClient();
```

### EventHubConsumerClient

```java
import com.azure.messaging.eventhubs.EventHubConsumerClient;

EventHubConsumerClient consumer = new EventHubClientBuilder()
    .connectionString("<connection-string>", "<event-hub-name>")
    .consumerGroup(EventHubClientBuilder.DEFAULT_CONSUMER_GROUP_NAME)
    .buildConsumerClient();
```

### Async Clients

```java
import com.azure.messaging.eventhubs.EventHubProducerAsyncClient;
import com.azure.messaging.eventhubs.EventHubConsumerAsyncClient;

EventHubProducerAsyncClient asyncProducer = new EventHubClientBuilder()
    .connectionString("<connection-string>", "<event-hub-name>")
    .buildAsyncProducerClient();

EventHubConsumerAsyncClient asyncConsumer = new EventHubClientBuilder()
    .connectionString("<connection-string>", "<event-hub-name>")
    .consumerGroup("$Default")
    .buildAsyncConsumerClient();
```

## Các Mẫu Cốt lõi

### Gửi Sự kiện Đơn

```java
import com.azure.messaging.eventhubs.EventData;

EventData eventData = new EventData("Hello, Event Hubs!");
producer.send(Collections.singletonList(eventData));
```

### Gửi Lô Sự kiện

```java
import com.azure.messaging.eventhubs.EventDataBatch;
import com.azure.messaging.eventhubs.models.CreateBatchOptions;

// Tạo lô
EventDataBatch batch = producer.createBatch();

// Thêm sự kiện (trả về false nếu lô đầy)
for (int i = 0; i < 100; i++) {
    EventData event = new EventData("Event " + i);
    if (!batch.tryAdd(event)) {
        // Lô đầy, gửi và tạo lô mới
        producer.send(batch);
        batch = producer.createBatch();
        batch.tryAdd(event);
    }
}

// Gửi các sự kiện còn lại
if (batch.getCount() > 0) {
    producer.send(batch);
}
```

### Gửi đến Partition Cụ thể

```java
CreateBatchOptions options = new CreateBatchOptions()
    .setPartitionId("0");

EventDataBatch batch = producer.createBatch(options);
batch.tryAdd(new EventData("Partition 0 event"));
producer.send(batch);
```

### Gửi với Partition Key

```java
CreateBatchOptions options = new CreateBatchOptions()
    .setPartitionKey("customer-123");

EventDataBatch batch = producer.createBatch(options);
batch.tryAdd(new EventData("Customer event"));
producer.send(batch);
```

### Sự kiện với Thuộc tính

```java
EventData event = new EventData("Order created");
event.getProperties().put("orderId", "ORD-123");
event.getProperties().put("customerId", "CUST-456");
event.getProperties().put("priority", 1);

producer.send(Collections.singletonList(event));
```

### Nhận Sự kiện (Đơn giản)

```java
import com.azure.messaging.eventhubs.models.EventPosition;
import com.azure.messaging.eventhubs.models.PartitionEvent;

// Nhận từ partition cụ thể
Iterable<PartitionEvent> events = consumer.receiveFromPartition(
    "0",                           // partitionId
    10,                            // maxEvents
    EventPosition.earliest(),      // startingPosition
    Duration.ofSeconds(30)         // timeout
);

for (PartitionEvent partitionEvent : events) {
    EventData event = partitionEvent.getData();
    System.out.println("Body: " + event.getBodyAsString());
    System.out.println("Sequence: " + event.getSequenceNumber());
    System.out.println("Offset: " + event.getOffset());
}
```

### EventProcessorClient (Sản xuất)

```java
import com.azure.messaging.eventhubs.EventProcessorClient;
import com.azure.messaging.eventhubs.EventProcessorClientBuilder;
import com.azure.messaging.eventhubs.checkpointstore.blob.BlobCheckpointStore;
import com.azure.storage.blob.BlobContainerAsyncClient;
import com.azure.storage.blob.BlobContainerClientBuilder;

// Tạo checkpoint store
BlobContainerAsyncClient blobClient = new BlobContainerClientBuilder()
    .connectionString("<storage-connection-string>")
    .containerName("checkpoints")
    .buildAsyncClient();

// Tạo processor
EventProcessorClient processor = new EventProcessorClientBuilder()
    .connectionString("<eventhub-connection-string>", "<event-hub-name>")
    .consumerGroup("$Default")
    .checkpointStore(new BlobCheckpointStore(blobClient))
    .processEvent(eventContext -> {
        EventData event = eventContext.getEventData();
        System.out.println("Processing: " + event.getBodyAsString());

        // Checkpoint sau khi xử lý
        eventContext.updateCheckpoint();
    })
    .processError(errorContext -> {
        System.err.println("Error: " + errorContext.getThrowable().getMessage());
        System.err.println("Partition: " + errorContext.getPartitionContext().getPartitionId());
    })
    .buildEventProcessorClient();

// Bắt đầu xử lý
processor.start();

// Tiếp tục chạy...
Thread.sleep(Duration.ofMinutes(5).toMillis());

// Dừng gracefully
processor.stop();
```

### Xử lý Lô

```java
EventProcessorClient processor = new EventProcessorClientBuilder()
    .connectionString("<connection-string>", "<event-hub-name>")
    .consumerGroup("$Default")
    .checkpointStore(new BlobCheckpointStore(blobClient))
    .processEventBatch(eventBatchContext -> {
        List<EventData> events = eventBatchContext.getEvents();
        System.out.printf("Received %d events%n", events.size());

        for (EventData event : events) {
            // Xử lý mỗi sự kiện
            System.out.println(event.getBodyAsString());
        }

        // Checkpoint sau khi xử lý lô
        eventBatchContext.updateCheckpoint();
    }, 50) // maxBatchSize
    .processError(errorContext -> {
        System.err.println("Error: " + errorContext.getThrowable());
    })
    .buildEventProcessorClient();
```

### Nhận Async

```java
asyncConsumer.receiveFromPartition("0", EventPosition.latest())
    .subscribe(
        partitionEvent -> {
            EventData event = partitionEvent.getData();
            System.out.println("Received: " + event.getBodyAsString());
        },
        error -> System.err.println("Error: " + error),
        () -> System.out.println("Complete")
    );
```

### Lấy Thuộc tính Event Hub

```java
// Lấy thông tin hub
EventHubProperties hubProps = producer.getEventHubProperties();
System.out.println("Hub: " + hubProps.getName());
System.out.println("Partitions: " + hubProps.getPartitionIds());

// Lấy thông tin partition
PartitionProperties partitionProps = producer.getPartitionProperties("0");
System.out.println("Begin sequence: " + partitionProps.getBeginningSequenceNumber());
System.out.println("Last sequence: " + partitionProps.getLastEnqueuedSequenceNumber());
System.out.println("Last offset: " + partitionProps.getLastEnqueuedOffset());
```

## Các Vị trí Sự kiện (Event Positions)

```java
// Bắt đầu từ đầu
EventPosition.earliest()

// Bắt đầu từ cuối (chỉ sự kiện mới)
EventPosition.latest()

// Từ offset cụ thể
EventPosition.fromOffset(12345L)

// Từ số thứ tự cụ thể
EventPosition.fromSequenceNumber(100L)

// Từ thời gian cụ thể
EventPosition.fromEnqueuedTime(Instant.now().minus(Duration.ofHours(1)))
```

## Xử lý Lỗi

```java
import com.azure.messaging.eventhubs.models.ErrorContext;

.processError(errorContext -> {
    Throwable error = errorContext.getThrowable();
    String partitionId = errorContext.getPartitionContext().getPartitionId();

    if (error instanceof AmqpException) {
        AmqpException amqpError = (AmqpException) error;
        if (amqpError.isTransient()) {
            System.out.println("Transient error, will retry");
        }
    }

    System.err.printf("Error on partition %s: %s%n", partitionId, error.getMessage());
})
```

## Dọn dẹp Tài nguyên

```java
// Luôn đóng clients
try {
    producer.send(batch);
} finally {
    producer.close();
}

// Hoặc sử dụng try-with-resources
try (EventHubProducerClient producer = new EventHubClientBuilder()
        .connectionString(connectionString, eventHubName)
        .buildProducerClient()) {
    producer.send(events);
}
```

## Biến Môi trường

```bash
EVENT_HUBS_CONNECTION_STRING=Endpoint=sb://<namespace>.servicebus.windows.net/;SharedAccessKeyName=...
EVENT_HUBS_NAME=<event-hub-name>
STORAGE_CONNECTION_STRING=<for-checkpointing>
```

## Thực hành Tốt nhất

1. **Sử dụng EventProcessorClient**: Cho sản xuất, cung cấp cân bằng tải và checkpointing
2. **Gửi Lô Sự kiện**: Sử dụng `EventDataBatch` để gửi hiệu quả
3. **Partition Keys**: Sử dụng để đảm bảo thứ tự trong cùng một partition
4. **Checkpointing**: Checkpoint sau khi xử lý để tránh xử lý lại
5. **Xử lý Lỗi**: Xử lý các lỗi thoáng qua với retry
6. **Đóng Clients**: Luôn đóng producer/consumer khi xong

## Các Cụm từ Kích hoạt

- "Event Hubs Java"
- "event streaming Azure"
- "real-time data ingestion"
- "EventProcessorClient"
- "event hub producer consumer"
- "partition processing"
