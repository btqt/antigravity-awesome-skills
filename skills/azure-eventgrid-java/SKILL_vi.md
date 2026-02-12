---
name: azure-eventgrid-java
description: Xây dựng các ứng dụng hướng sự kiện với Azure Event Grid SDK cho Java. Sử dụng khi xuất bản (publishing) sự kiện, triển khai các mẫu pub/sub, hoặc tích hợp với các dịch vụ Azure thông qua sự kiện.
package: com.azure:azure-messaging-eventgrid
---

# Azure Event Grid SDK cho Java

Xây dựng các ứng dụng hướng sự kiện sử dụng Azure Event Grid SDK cho Java.

## Cài đặt

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-messaging-eventgrid</artifactId>
    <version>4.27.0</version>
</dependency>
```

## Tạo Client

### EventGridPublisherClient

```java
import com.azure.messaging.eventgrid.EventGridPublisherClient;
import com.azure.messaging.eventgrid.EventGridPublisherClientBuilder;
import com.azure.core.credential.AzureKeyCredential;

// Với API Key
EventGridPublisherClient<EventGridEvent> client = new EventGridPublisherClientBuilder()
    .endpoint("<topic-endpoint>")
    .credential(new AzureKeyCredential("<access-key>"))
    .buildEventGridEventPublisherClient();

// Cho CloudEvents
EventGridPublisherClient<CloudEvent> cloudClient = new EventGridPublisherClientBuilder()
    .endpoint("<topic-endpoint>")
    .credential(new AzureKeyCredential("<access-key>"))
    .buildCloudEventPublisherClient();
```

### Với DefaultAzureCredential

```java
import com.azure.identity.DefaultAzureCredentialBuilder;

EventGridPublisherClient<EventGridEvent> client = new EventGridPublisherClientBuilder()
    .endpoint("<topic-endpoint>")
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildEventGridEventPublisherClient();
```

### Async Client

```java
import com.azure.messaging.eventgrid.EventGridPublisherAsyncClient;

EventGridPublisherAsyncClient<EventGridEvent> asyncClient = new EventGridPublisherClientBuilder()
    .endpoint("<topic-endpoint>")
    .credential(new AzureKeyCredential("<access-key>"))
    .buildEventGridEventPublisherAsyncClient();
```

## Các Loại Sự kiện

| Loại             | Mô tả                              |
| ---------------- | ---------------------------------- |
| `EventGridEvent` | Azure Event Grid native schema     |
| `CloudEvent`     | CNCF CloudEvents 1.0 specification |
| `BinaryData`     | Sự kiện schema tùy chỉnh           |

## Các Mẫu Cốt lõi

### Xuất bản EventGridEvent

```java
import com.azure.messaging.eventgrid.EventGridEvent;
import com.azure.core.util.BinaryData;

EventGridEvent event = new EventGridEvent(
    "resource/path",           // subject
    "MyApp.Events.OrderCreated", // eventType
    BinaryData.fromObject(new OrderData("order-123", 99.99)), // data
    "1.0"                      // dataVersion
);

client.sendEvent(event);
```

### Xuất bản Nhiều Sự kiện

```java
List<EventGridEvent> events = Arrays.asList(
    new EventGridEvent("orders/1", "Order.Created",
        BinaryData.fromObject(order1), "1.0"),
    new EventGridEvent("orders/2", "Order.Created",
        BinaryData.fromObject(order2), "1.0")
);

client.sendEvents(events);
```

### Xuất bản CloudEvent

```java
import com.azure.core.models.CloudEvent;
import com.azure.core.models.CloudEventDataFormat;

CloudEvent cloudEvent = new CloudEvent(
    "/myapp/orders",           // source
    "order.created",           // type
    BinaryData.fromObject(orderData), // data
    CloudEventDataFormat.JSON  // dataFormat
);
cloudEvent.setSubject("orders/12345");
cloudEvent.setId(UUID.randomUUID().toString());

cloudClient.sendEvent(cloudEvent);
```

### Xuất bản CloudEvents Batch

```java
List<CloudEvent> cloudEvents = Arrays.asList(
    new CloudEvent("/app", "event.type1", BinaryData.fromString("data1"), CloudEventDataFormat.JSON),
    new CloudEvent("/app", "event.type2", BinaryData.fromString("data2"), CloudEventDataFormat.JSON)
);

cloudClient.sendEvents(cloudEvents);
```

### Xuất bản Async

```java
asyncClient.sendEvent(event)
    .subscribe(
        unused -> System.out.println("Event sent successfully"),
        error -> System.err.println("Error: " + error.getMessage())
    );

// Với nhiều sự kiện
asyncClient.sendEvents(events)
    .doOnSuccess(unused -> System.out.println("All events sent"))
    .doOnError(error -> System.err.println("Failed: " + error))
    .block(); // Block nếu cần
```

### Lớp Dữ liệu Sự kiện Tùy chỉnh

```java
public class OrderData {
    private String orderId;
    private double amount;
    private String customerId;

    public OrderData(String orderId, double amount) {
        this.orderId = orderId;
        this.amount = amount;
    }

    // Getters and setters
}

// Sử dụng
OrderData order = new OrderData("ORD-123", 150.00);
EventGridEvent event = new EventGridEvent(
    "orders/" + order.getOrderId(),
    "MyApp.Order.Created",
    BinaryData.fromObject(order),
    "1.0"
);
```

## Nhận Sự kiện (Receiving Events)

### Parse EventGridEvent

```java
import com.azure.messaging.eventgrid.EventGridEvent;

// Từ chuỗi JSON (ví dụ: webhook payload)
String jsonPayload = "[{\"id\": \"...\", ...}]";
List<EventGridEvent> events = EventGridEvent.fromString(jsonPayload);

for (EventGridEvent event : events) {
    System.out.println("Event Type: " + event.getEventType());
    System.out.println("Subject: " + event.getSubject());
    System.out.println("Event Time: " + event.getEventTime());

    // Lấy dữ liệu
    BinaryData data = event.getData();
    OrderData orderData = data.toObject(OrderData.class);
}
```

### Parse CloudEvent

```java
import com.azure.core.models.CloudEvent;

String cloudEventJson = "[{\"specversion\": \"1.0\", ...}]";
List<CloudEvent> cloudEvents = CloudEvent.fromString(cloudEventJson);

for (CloudEvent event : cloudEvents) {
    System.out.println("Type: " + event.getType());
    System.out.println("Source: " + event.getSource());
    System.out.println("ID: " + event.getId());

    MyEventData data = event.getData().toObject(MyEventData.class);
}
```

### Xử lý Sự kiện Hệ thống

```java
import com.azure.messaging.eventgrid.systemevents.*;

for (EventGridEvent event : events) {
    if (event.getEventType().equals("Microsoft.Storage.BlobCreated")) {
        StorageBlobCreatedEventData blobData =
            event.getData().toObject(StorageBlobCreatedEventData.class);
        System.out.println("Blob URL: " + blobData.getUrl());
    }
}
```

## Event Grid Namespaces (MQTT/Pull)

### Nhận từ Namespace Topic

```java
import com.azure.messaging.eventgrid.namespaces.EventGridReceiverClient;
import com.azure.messaging.eventgrid.namespaces.EventGridReceiverClientBuilder;
import com.azure.messaging.eventgrid.namespaces.models.*;

EventGridReceiverClient receiverClient = new EventGridReceiverClientBuilder()
    .endpoint("<namespace-endpoint>")
    .credential(new AzureKeyCredential("<key>"))
    .topicName("my-topic")
    .subscriptionName("my-subscription")
    .buildClient();

// Nhận sự kiện
ReceiveResult result = receiverClient.receive(10, Duration.ofSeconds(30));

for (ReceiveDetails detail : result.getValue()) {
    CloudEvent event = detail.getEvent();
    System.out.println("Event: " + event.getType());

    // Acknowledge sự kiện
    receiverClient.acknowledge(Arrays.asList(detail.getBrokerProperties().getLockToken()));
}
```

### Từ chối (Reject) hoặc Release Sự kiện

```java
// Reject (không thử lại)
receiverClient.reject(Arrays.asList(lockToken));

// Release (thử lại sau)
receiverClient.release(Arrays.asList(lockToken));

// Release với độ trễ
receiverClient.release(Arrays.asList(lockToken),
    new ReleaseOptions().setDelay(ReleaseDelay.BY_60_SECONDS));
```

## Xử lý Lỗi

```java
import com.azure.core.exception.HttpResponseException;

try {
    client.sendEvent(event);
} catch (HttpResponseException e) {
    System.out.println("Status: " + e.getResponse().getStatusCode());
    System.out.println("Error: " + e.getMessage());
}
```

## Biến Môi trường

```bash
EVENT_GRID_TOPIC_ENDPOINT=https://<topic-name>.<region>.eventgrid.azure.net/api/events
EVENT_GRID_ACCESS_KEY=<your-access-key>
```

## Thực hành Tốt nhất

1. **Batch Events**: Gửi nhiều sự kiện trong một cuộc gọi khi có thể
2. **Idempotency**: Bao gồm ID sự kiện duy nhất để loại bỏ trùng lặp
3. **Xác thực Schema**: Sử dụng các lớp dữ liệu sự kiện có kiểu mạnh (strongly-typed)
4. **Retry Logic**: Tích hợp sẵn, nhưng cân nhắc dead-letter cho các lỗi
5. **Kích thước Sự kiện**: Giữ sự kiện dưới 1MB (64KB cho basic tier)

## Các Cụm từ Kích hoạt

- "Event Grid Java"
- "publish events Azure"
- "CloudEvent SDK"
- "event-driven messaging"
- "pub/sub Azure"
- "webhook events"
