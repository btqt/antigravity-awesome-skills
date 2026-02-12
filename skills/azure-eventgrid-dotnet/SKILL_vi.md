---
name: azure-eventgrid-dotnet
description: |
  Azure Event Grid SDK cho .NET. Thư viện client để xuất bản (publishing) và tiêu thụ (consuming) sự kiện với Azure Event Grid. Sử dụng cho kiến trúc hướng sự kiện, nhắn tin pub/sub, CloudEvents, và EventGridEvents. Triggers: "Event Grid", "EventGridPublisherClient", "CloudEvent", "EventGridEvent", "publish events .NET", "event-driven", "pub/sub".
package: Azure.Messaging.EventGrid
---

# Azure.Messaging.EventGrid (.NET)

Thư viện client để xuất bản sự kiện đến Azure Event Grid topics, domains, và namespaces.

## Cài đặt

```bash
# Cho topics và domains (push delivery)
dotnet add package Azure.Messaging.EventGrid

# Cho namespaces (pull delivery)
dotnet add package Azure.Messaging.EventGrid.Namespaces

# Cho CloudNative CloudEvents interop
dotnet add package Microsoft.Azure.Messaging.EventGrid.CloudNativeCloudEvents
```

**Phiên bản Hiện tại**: 4.28.0 (ổn định)

## Biến Môi trường

```bash
# Topic/Domain endpoint
EVENT_GRID_TOPIC_ENDPOINT=https://<topic-name>.<region>.eventgrid.azure.net/api/events
EVENT_GRID_TOPIC_KEY=<access-key>

# Namespace endpoint (cho pull delivery)
EVENT_GRID_NAMESPACE_ENDPOINT=https://<namespace>.<region>.eventgrid.azure.net
EVENT_GRID_TOPIC_NAME=<topic-name>
EVENT_GRID_SUBSCRIPTION_NAME=<subscription-name>
```

## Phân cấp Client

```
Push Delivery (Topics/Domains)
└── EventGridPublisherClient
    ├── SendEventAsync(EventGridEvent)
    ├── SendEventsAsync(IEnumerable<EventGridEvent>)
    ├── SendEventAsync(CloudEvent)
    └── SendEventsAsync(IEnumerable<CloudEvent>)

Pull Delivery (Namespaces)
├── EventGridSenderClient
│   └── SendAsync(CloudEvent)
└── EventGridReceiverClient
    ├── ReceiveAsync()
    ├── AcknowledgeAsync()
    ├── ReleaseAsync()
    └── RejectAsync()
```

## Xác thực

### Xác thực API Key

```csharp
using Azure;
using Azure.Messaging.EventGrid;

EventGridPublisherClient client = new(
    new Uri("https://mytopic.eastus-1.eventgrid.azure.net/api/events"),
    new AzureKeyCredential("<access-key>"));
```

### Microsoft Entra ID (Khuyên dùng)

```csharp
using Azure.Identity;
using Azure.Messaging.EventGrid;

EventGridPublisherClient client = new(
    new Uri("https://mytopic.eastus-1.eventgrid.azure.net/api/events"),
    new DefaultAzureCredential());
```

### Xác thực SAS Token

```csharp
string sasToken = EventGridPublisherClient.BuildSharedAccessSignature(
    new Uri(topicEndpoint),
    DateTimeOffset.UtcNow.AddHours(1),
    new AzureKeyCredential(topicKey));

var sasCredential = new AzureSasCredential(sasToken);
EventGridPublisherClient client = new(
    new Uri(topicEndpoint),
    sasCredential);
```

## Xuất bản Sự kiện (Publishing Events)

### EventGridEvent Schema

```csharp
EventGridPublisherClient client = new(
    new Uri(topicEndpoint),
    new AzureKeyCredential(topicKey));

// Sự kiện đơn
EventGridEvent egEvent = new(
    subject: "orders/12345",
    eventType: "Order.Created",
    dataVersion: "1.0",
    data: new { OrderId = "12345", Amount = 99.99 });

await client.SendEventAsync(egEvent);

// Lô sự kiện (Batch of events)
List<EventGridEvent> events = new()
{
    new EventGridEvent(
        subject: "orders/12345",
        eventType: "Order.Created",
        dataVersion: "1.0",
        data: new OrderData { OrderId = "12345", Amount = 99.99 }),
    new EventGridEvent(
        subject: "orders/12346",
        eventType: "Order.Created",
        dataVersion: "1.0",
        data: new OrderData { OrderId = "12346", Amount = 149.99 })
};

await client.SendEventsAsync(events);
```

### CloudEvent Schema

```csharp
CloudEvent cloudEvent = new(
    source: "/orders/system",
    type: "Order.Created",
    data: new { OrderId = "12345", Amount = 99.99 });

cloudEvent.Subject = "orders/12345";
cloudEvent.Id = Guid.NewGuid().ToString();
cloudEvent.Time = DateTimeOffset.UtcNow;

await client.SendEventAsync(cloudEvent);

// Lô CloudEvents
List<CloudEvent> cloudEvents = new()
{
    new CloudEvent("/orders", "Order.Created", new { OrderId = "1" }),
    new CloudEvent("/orders", "Order.Updated", new { OrderId = "2" })
};

await client.SendEventsAsync(cloudEvents);
```

### Xuất bản đến Event Grid Domain

```csharp
// Sự kiện phải chỉ định thuộc tính Topic cho domain routing
List<EventGridEvent> events = new()
{
    new EventGridEvent(
        subject: "orders/12345",
        eventType: "Order.Created",
        dataVersion: "1.0",
        data: new { OrderId = "12345" })
    {
        Topic = "orders-topic"  // Tên domain topic
    },
    new EventGridEvent(
        subject: "inventory/item-1",
        eventType: "Inventory.Updated",
        dataVersion: "1.0",
        data: new { ItemId = "item-1" })
    {
        Topic = "inventory-topic"
    }
};

await client.SendEventsAsync(events);
```

### Custom Serialization

```csharp
using System.Text.Json;

var serializerOptions = new JsonSerializerOptions
{
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase
};

var customSerializer = new JsonObjectSerializer(serializerOptions);

EventGridEvent egEvent = new(
    subject: "orders/12345",
    eventType: "Order.Created",
    dataVersion: "1.0",
    data: customSerializer.Serialize(new OrderData { OrderId = "12345" }));

await client.SendEventAsync(egEvent);
```

## Pull Delivery (Namespaces)

### Gửi Sự kiện đến Namespace Topic

```csharp
using Azure;
using Azure.Messaging;
using Azure.Messaging.EventGrid.Namespaces;

var senderClient = new EventGridSenderClient(
    new Uri(namespaceEndpoint),
    topicName,
    new AzureKeyCredential(topicKey));

// Gửi sự kiện đơn
CloudEvent cloudEvent = new("employee_source", "Employee.Created",
    new { Name = "John", Age = 30 });
await senderClient.SendAsync(cloudEvent);

// Gửi lô (batch)
await senderClient.SendAsync(new[]
{
    new CloudEvent("source", "type", new { Name = "Alice" }),
    new CloudEvent("source", "type", new { Name = "Bob" })
});
```

### Nhận và Xử lý Sự kiện

```csharp
var receiverClient = new EventGridReceiverClient(
    new Uri(namespaceEndpoint),
    topicName,
    subscriptionName,
    new AzureKeyCredential(topicKey));

// Nhận sự kiện
ReceiveResult result = await receiverClient.ReceiveAsync(maxEvents: 10);

List<string> lockTokensToAck = new();
List<string> lockTokensToRelease = new();

foreach (ReceiveDetails detail in result.Details)
{
    CloudEvent cloudEvent = detail.Event;
    string lockToken = detail.BrokerProperties.LockToken;

    try
    {
        // Xử lý sự kiện
        Console.WriteLine($"Event: {cloudEvent.Type}, Data: {cloudEvent.Data}");
        lockTokensToAck.Add(lockToken);
    }
    catch (Exception)
    {
        // Release để thử lại
        lockTokensToRelease.Add(lockToken);
    }
}

// Acknowledge các sự kiện đã xử lý thành công
if (lockTokensToAck.Any())
{
    await receiverClient.AcknowledgeAsync(lockTokensToAck);
}

// Release các sự kiện để thử lại
if (lockTokensToRelease.Any())
{
    await receiverClient.ReleaseAsync(lockTokensToRelease);
}
```

### Từ chối Sự kiện (Dead Letter)

```csharp
// Từ chối các sự kiện không thể xử lý
await receiverClient.RejectAsync(new[] { lockToken });
```

## Tiêu thụ Sự kiện (Azure Functions)

### EventGridEvent Trigger

```csharp
using Azure.Messaging.EventGrid;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.EventGrid;

public static class EventGridFunction
{
    [FunctionName("ProcessEventGridEvent")]
    public static void Run(
        [EventGridTrigger] EventGridEvent eventGridEvent,
        ILogger log)
    {
        log.LogInformation($"Event Type: {eventGridEvent.EventType}");
        log.LogInformation($"Subject: {eventGridEvent.Subject}");
        log.LogInformation($"Data: {eventGridEvent.Data}");
    }
}
```

### CloudEvent Trigger

```csharp
using Azure.Messaging;
using Microsoft.Azure.Functions.Worker;

public class CloudEventFunction
{
    [Function("ProcessCloudEvent")]
    public void Run(
        [EventGridTrigger] CloudEvent cloudEvent,
        FunctionContext context)
    {
        var logger = context.GetLogger("ProcessCloudEvent");
        logger.LogInformation($"Event Type: {cloudEvent.Type}");
        logger.LogInformation($"Source: {cloudEvent.Source}");
        logger.LogInformation($"Data: {cloudEvent.Data}");
    }
}
```

## Parsing Events

### Parse EventGridEvent

```csharp
// Từ chuỗi JSON
string json = "..."; // Payload webhook Event Grid
EventGridEvent[] events = EventGridEvent.ParseMany(BinaryData.FromString(json));

foreach (EventGridEvent egEvent in events)
{
    if (egEvent.TryGetSystemEventData(out object systemEvent))
    {
        // Xử lý sự kiện hệ thống
        switch (systemEvent)
        {
            case StorageBlobCreatedEventData blobCreated:
                Console.WriteLine($"Blob created: {blobCreated.Url}");
                break;
        }
    }
    else
    {
        // Xử lý sự kiện tùy chỉnh
        var customData = egEvent.Data.ToObjectFromJson<MyCustomData>();
    }
}
```

### Parse CloudEvent

```csharp
CloudEvent[] cloudEvents = CloudEvent.ParseMany(BinaryData.FromString(json));

foreach (CloudEvent cloudEvent in cloudEvents)
{
    var data = cloudEvent.Data.ToObjectFromJson<MyEventData>();
    Console.WriteLine($"Type: {cloudEvent.Type}, Data: {data}");
}
```

## Các Sự kiện Hệ thống

```csharp
// Các loại sự kiện hệ thống phổ biến
using Azure.Messaging.EventGrid.SystemEvents;

// Storage events
StorageBlobCreatedEventData blobCreated;
StorageBlobDeletedEventData blobDeleted;

// Resource events
ResourceWriteSuccessEventData resourceCreated;
ResourceDeleteSuccessEventData resourceDeleted;

// App Service events
WebAppUpdatedEventData webAppUpdated;

// Container Registry events
ContainerRegistryImagePushedEventData imagePushed;

// IoT Hub events
IotHubDeviceCreatedEventData deviceCreated;
```

## Tham khảo Các Loại Chính

| Loại                       | Mục đích                          |
| -------------------------- | --------------------------------- |
| `EventGridPublisherClient` | Xuất bản đến topics/domains       |
| `EventGridSenderClient`    | Gửi đến namespace topics          |
| `EventGridReceiverClient`  | Nhận từ namespace subscriptions   |
| `EventGridEvent`           | Event Grid native schema          |
| `CloudEvent`               | CloudEvents 1.0 schema            |
| `ReceiveResult`            | Phản hồi Pull delivery            |
| `ReceiveDetails`           | Sự kiện với các thuộc tính broker |
| `BrokerProperties`         | Lock token, số lần giao hàng      |

## So sánh Schemas Sự kiện

| Tính năng          | EventGridEvent                        | CloudEvent                                |
| ------------------ | ------------------------------------- | ----------------------------------------- |
| Tiêu chuẩn         | Azure-specific                        | CNCF standard                             |
| Trường bắt buộc    | subject, eventType, dataVersion, data | source, type                              |
| Khả năng mở rộng   | Giới hạn                              | Thuộc tính mở rộng (Extension attributes) |
| Khả năng tương tác | Chỉ Azure                             | Đa nền tảng                               |

## Thực hành Tốt nhất

1. **Sử dụng CloudEvents** — Ưu tiên CloudEvents cho triển khai mới (tiêu chuẩn ngành)
2. **Gửi lô sự kiện** — Gửi nhiều sự kiện trong một cuộc gọi để tăng hiệu quả
3. **Sử dụng Entra ID** — Ưu tiên managed identity hơn access keys
4. **Xử lý idempotent** — Sự kiện có thể được giao nhiều lần
5. **Thiết lập event TTL** — Cấu hình time-to-live cho namespace events
6. **Xử lý lỗi một phần** — Acknowledge/release sự kiện riêng lẻ
7. **Sử dụng dead-letter** — Cấu hình dead-letter cho các sự kiện thất bại
8. **Xác thực schemas** — Xác thực dữ liệu sự kiện trước khi xử lý

## Xử lý Lỗi

```csharp
using Azure;

try
{
    await client.SendEventAsync(cloudEvent);
}
catch (RequestFailedException ex) when (ex.Status == 401)
{
    Console.WriteLine("Authentication failed - check credentials");
}
catch (RequestFailedException ex) when (ex.Status == 403)
{
    Console.WriteLine("Authorization failed - check RBAC permissions");
}
catch (RequestFailedException ex) when (ex.Status == 413)
{
    Console.WriteLine("Payload too large - max 1MB per event, 1MB total batch");
}
catch (RequestFailedException ex)
{
    Console.WriteLine($"Event Grid error: {ex.Status} - {ex.Message}");
}
```

## Mẫu Failover

```csharp
try
{
    var primaryClient = new EventGridPublisherClient(primaryUri, primaryKey);
    await primaryClient.SendEventsAsync(events);
}
catch (RequestFailedException)
{
    // Failover sang vùng phụ (secondary region)
    var secondaryClient = new EventGridPublisherClient(secondaryUri, secondaryKey);
    await secondaryClient.SendEventsAsync(events);
}
```

## Các SDK Liên quan

| SDK                                            | Mục đích                 | Cài đặt                                                           |
| ---------------------------------------------- | ------------------------ | ----------------------------------------------------------------- |
| `Azure.Messaging.EventGrid`                    | Topics/Domains (SDK này) | `dotnet add package Azure.Messaging.EventGrid`                    |
| `Azure.Messaging.EventGrid.Namespaces`         | Pull delivery            | `dotnet add package Azure.Messaging.EventGrid.Namespaces`         |
| `Azure.Identity`                               | Xác thực                 | `dotnet add package Azure.Identity`                               |
| `Microsoft.Azure.WebJobs.Extensions.EventGrid` | Azure Functions trigger  | `dotnet add package Microsoft.Azure.WebJobs.Extensions.EventGrid` |

## Liên kết Tham khảo

| Tài nguyên      | URL                                                                                          |
| --------------- | -------------------------------------------------------------------------------------------- |
| Gói NuGet       | https://www.nuget.org/packages/Azure.Messaging.EventGrid                                     |
| Tham khảo API   | https://learn.microsoft.com/dotnet/api/azure.messaging.eventgrid                             |
| Bắt đầu nhanh   | https://learn.microsoft.com/azure/event-grid/custom-event-quickstart                         |
| Pull Delivery   | https://learn.microsoft.com/azure/event-grid/pull-delivery-overview                          |
| Mã nguồn GitHub | https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/eventgrid/Azure.Messaging.EventGrid |
