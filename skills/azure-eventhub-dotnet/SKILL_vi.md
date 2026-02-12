---
name: azure-eventhub-dotnet
description: |
  Azure Event Hubs SDK cho .NET. Sử dụng cho streaming sự kiện thông lượng cao: gửi sự kiện (EventHubProducerClient, EventHubBufferedProducerClient), nhận sự kiện (EventProcessorClient với checkpointing), quản lý partition, và nhập dữ liệu thời gian thực. Triggers: "Event Hubs", "event streaming", "EventHubProducerClient", "EventProcessorClient", "send events", "receive events", "checkpointing", "partition".
package: Azure.Messaging.EventHubs
---

# Azure.Messaging.EventHubs (.NET)

SDK streaming sự kiện thông lượng cao để gửi và nhận sự kiện qua Azure Event Hubs.

## Cài đặt

```bash
# Gói cốt lõi (gửi và nhận đơn giản)
dotnet add package Azure.Messaging.EventHubs

# Gói xử lý (nhận trong sản xuất với checkpointing)
dotnet add package Azure.Messaging.EventHubs.Processor

# Xác thực
dotnet add package Azure.Identity

# Cho checkpointing (yêu cầu bởi EventProcessorClient)
dotnet add package Azure.Storage.Blobs
```

**Phiên bản Hiện tại**: Azure.Messaging.EventHubs v5.12.2, Azure.Messaging.EventHubs.Processor v5.12.2

## Biến Môi trường

```bash
EVENTHUB_FULLY_QUALIFIED_NAMESPACE=<namespace>.servicebus.windows.net
EVENTHUB_NAME=<event-hub-name>

# Cho checkpointing (EventProcessorClient)
BLOB_STORAGE_CONNECTION_STRING=<storage-connection-string>
BLOB_CONTAINER_NAME=<checkpoint-container>

# Thay thế: Connection string auth (không khuyến nghị cho sản xuất)
EVENTHUB_CONNECTION_STRING=Endpoint=sb://<namespace>.servicebus.windows.net/;SharedAccessKeyName=...
```

## Xác thực

```csharp
using Azure.Identity;
using Azure.Messaging.EventHubs;
using Azure.Messaging.EventHubs.Producer;

// Luôn sử dụng DefaultAzureCredential cho sản xuất
var credential = new DefaultAzureCredential();

var fullyQualifiedNamespace = Environment.GetEnvironmentVariable("EVENTHUB_FULLY_QUALIFIED_NAMESPACE");
var eventHubName = Environment.GetEnvironmentVariable("EVENTHUB_NAME");

var producer = new EventHubProducerClient(
    fullyQualifiedNamespace,
    eventHubName,
    credential);
```

**Các Vai trò RBAC Yêu cầu**:

- **Gửi**: `Azure Event Hubs Data Sender`
- **Nhận**: `Azure Event Hubs Data Receiver`
- **Cả hai**: `Azure Event Hubs Data Owner`

## Các Loại Client

| Client                           | Mục đích                         | Khi nào Sử dụng                                     |
| -------------------------------- | -------------------------------- | --------------------------------------------------- |
| `EventHubProducerClient`         | Gửi sự kiện ngay lập tức theo lô | Gửi thời gian thực, kiểm soát hoàn toàn việc tạo lô |
| `EventHubBufferedProducerClient` | Tự động tạo lô với gửi nền       | Khối lượng lớn, kịch bản fire-and-forget            |
| `EventHubConsumerClient`         | Đọc sự kiện đơn giản             | Chỉ dùng cho prototyping, KHÔNG dùng cho sản xuất   |
| `EventProcessorClient`           | Xử lý sự kiện sản xuất           | **Luôn sử dụng cái này để nhận trong sản xuất**     |

## Quy trình Cốt lõi

### 1. Gửi Sự kiện (Lô)

```csharp
using Azure.Identity;
using Azure.Messaging.EventHubs;
using Azure.Messaging.EventHubs.Producer;

await using var producer = new EventHubProducerClient(
    fullyQualifiedNamespace,
    eventHubName,
    new DefaultAzureCredential());

// Tạo một lô (tuân thủ giới hạn kích thước tự động)
using EventDataBatch batch = await producer.CreateBatchAsync();

// Thêm sự kiện vào lô
var events = new[]
{
    new EventData(BinaryData.FromString("{\"id\": 1, \"message\": \"Hello\"}")),
    new EventData(BinaryData.FromString("{\"id\": 2, \"message\": \"World\"}"))
};

foreach (var eventData in events)
{
    if (!batch.TryAdd(eventData))
    {
        // Lô đã đầy - gửi nó và tạo lô mới
        await producer.SendAsync(batch);
        batch = await producer.CreateBatchAsync();

        if (!batch.TryAdd(eventData))
        {
            throw new Exception("Event too large for empty batch");
        }
    }
}

// Gửi các sự kiện còn lại
if (batch.Count > 0)
{
    await producer.SendAsync(batch);
}
```

### 2. Gửi Sự kiện (Buffered - Khối lượng lớn)

```csharp
using Azure.Messaging.EventHubs.Producer;

var options = new EventHubBufferedProducerClientOptions
{
    MaximumWaitTime = TimeSpan.FromSeconds(1)
};

await using var producer = new EventHubBufferedProducerClient(
    fullyQualifiedNamespace,
    eventHubName,
    new DefaultAzureCredential(),
    options);

// Xử lý gửi thành công/thất bại
producer.SendEventBatchSucceededAsync += args =>
{
    Console.WriteLine($"Batch sent: {args.EventBatch.Count} events");
    return Task.CompletedTask;
};

producer.SendEventBatchFailedAsync += args =>
{
    Console.WriteLine($"Batch failed: {args.Exception.Message}");
    return Task.CompletedTask;
};

// Enqueue sự kiện (được gửi tự động trong nền)
for (int i = 0; i < 1000; i++)
{
    await producer.EnqueueEventAsync(new EventData($"Event {i}"));
}

// Flush các sự kiện còn lại trước khi dispose
await producer.FlushAsync();
```

### 3. Nhận Sự kiện (Sản xuất - EventProcessorClient)

```csharp
using Azure.Identity;
using Azure.Messaging.EventHubs;
using Azure.Messaging.EventHubs.Consumer;
using Azure.Messaging.EventHubs.Processor;
using Azure.Storage.Blobs;

// Blob container cho checkpointing
var blobClient = new BlobContainerClient(
    Environment.GetEnvironmentVariable("BLOB_STORAGE_CONNECTION_STRING"),
    Environment.GetEnvironmentVariable("BLOB_CONTAINER_NAME"));

await blobClient.CreateIfNotExistsAsync();

// Tạo processor
var processor = new EventProcessorClient(
    blobClient,
    EventHubConsumerClient.DefaultConsumerGroup,
    fullyQualifiedNamespace,
    eventHubName,
    new DefaultAzureCredential());

// Xử lý sự kiện
processor.ProcessEventAsync += async args =>
{
    Console.WriteLine($"Partition: {args.Partition.PartitionId}");
    Console.WriteLine($"Data: {args.Data.EventBody}");

    // Checkpoint sau khi xử lý (hoặc checkpoint theo lô)
    await args.UpdateCheckpointAsync();
};

// Xử lý lỗi
processor.ProcessErrorAsync += args =>
{
    Console.WriteLine($"Error: {args.Exception.Message}");
    Console.WriteLine($"Partition: {args.PartitionId}");
    return Task.CompletedTask;
};

// Bắt đầu xử lý
await processor.StartProcessingAsync();

// Chạy cho đến khi bị hủy
await Task.Delay(Timeout.Infinite, cancellationToken);

// Dừng một cách duyên dáng (gracefully)
await processor.StopProcessingAsync();
```

### 4. Các Thao tác Partition

```csharp
// Lấy partition IDs
string[] partitionIds = await producer.GetPartitionIdsAsync();

// Gửi đến partition cụ thể (sử dụng hạn chế)
var options = new SendEventOptions
{
    PartitionId = "0"
};
await producer.SendAsync(events, options);

// Sử dụng partition key (khuyên dùng để đảm bảo thứ tự)
var batchOptions = new CreateBatchOptions
{
    PartitionKey = "customer-123"  // Các sự kiện có cùng key sẽ vào cùng partition
};
using var batch = await producer.CreateBatchAsync(batchOptions);
```

## Tùy chọn EventPosition

Kiểm soát nơi bắt đầu đọc:

```csharp
// Bắt đầu từ đầu
EventPosition.Earliest

// Bắt đầu từ cuối (chỉ sự kiện mới)
EventPosition.Latest

// Bắt đầu từ offset cụ thể
EventPosition.FromOffset(12345)

// Bắt đầu từ số thứ tự cụ thể
EventPosition.FromSequenceNumber(100)

// Bắt đầu từ thời gian cụ thể
EventPosition.FromEnqueuedTime(DateTimeOffset.UtcNow.AddHours(-1))
```

## Tích hợp ASP.NET Core

```csharp
// Program.cs
using Azure.Identity;
using Azure.Messaging.EventHubs.Producer;
using Microsoft.Extensions.Azure;

builder.Services.AddAzureClients(clientBuilder =>
{
    clientBuilder.AddEventHubProducerClient(
        builder.Configuration["EventHub:FullyQualifiedNamespace"],
        builder.Configuration["EventHub:Name"]);

    clientBuilder.UseCredential(new DefaultAzureCredential());
});

// Inject trong controller/service
public class EventService
{
    private readonly EventHubProducerClient _producer;

    public EventService(EventHubProducerClient producer)
    {
        _producer = producer;
    }

    public async Task SendAsync(string message)
    {
        using var batch = await _producer.CreateBatchAsync();
        batch.TryAdd(new EventData(message));
        await _producer.SendAsync(batch);
    }
}
```

## Thực hành Tốt nhất

1. **Sử dụng `EventProcessorClient` để nhận** — Không bao giờ sử dụng `EventHubConsumerClient` trong sản xuất
2. **Checkpoint chiến lược** — Sau N sự kiện hoặc khoảng thời gian, không phải mỗi sự kiện
3. **Sử dụng partition keys** — Để đảm bảo thứ tự trong cùng một partition
4. **Tái sử dụng clients** — Tạo một lần, sử dụng như singleton (thread-safe)
5. **Sử dụng `await using`** — Đảm bảo disposal đúng cách
6. **Xử lý `ProcessErrorAsync`** — Luôn đăng ký xử lý lỗi
7. **Gửi sự kiện theo lô** — Sử dụng `CreateBatchAsync()` để tuân thủ giới hạn kích thước
8. **Sử dụng buffered producer** — Cho kịch bản khối lượng lớn với tự động tạo lô

## Xử lý Lỗi

```csharp
using Azure.Messaging.EventHubs;

try
{
    await producer.SendAsync(batch);
}
catch (EventHubsException ex) when (ex.Reason == EventHubsException.FailureReason.ServiceBusy)
{
    // Thử lại với backoff
    await Task.Delay(TimeSpan.FromSeconds(5));
}
catch (EventHubsException ex) when (ex.IsTransient)
{
    // Lỗi thoáng qua - an toàn để thử lại
    Console.WriteLine($"Transient error: {ex.Message}");
}
catch (EventHubsException ex)
{
    // Lỗi không thoáng qua
    Console.WriteLine($"Error: {ex.Reason} - {ex.Message}");
}
```

## Chiến lược Checkpointing

| Chiến lược         | Khi nào Sử dụng                       |
| ------------------ | ------------------------------------- |
| Mỗi sự kiện        | Khối lượng thấp, dữ liệu quan trọng   |
| Mỗi N sự kiện      | Cân bằng thông lượng/độ tin cậy       |
| Dựa trên thời gian | Khoảng thời gian checkpoint nhất quán |
| Hoàn thành lô      | Sau khi xử lý một lô logic            |

```csharp
// Checkpoint mỗi 100 sự kiện
private int _eventCount = 0;

processor.ProcessEventAsync += async args =>
{
    // Xử lý sự kiện...

    _eventCount++;
    if (_eventCount >= 100)
    {
        await args.UpdateCheckpointAsync();
        _eventCount = 0;
    }
};
```

## Các SDK Liên quan

| SDK                                            | Mục đích                    | Cài đặt                                                           |
| ---------------------------------------------- | --------------------------- | ----------------------------------------------------------------- |
| `Azure.Messaging.EventHubs`                    | Gửi/nhận cốt lõi            | `dotnet add package Azure.Messaging.EventHubs`                    |
| `Azure.Messaging.EventHubs.Processor`          | Xử lý sản xuất              | `dotnet add package Azure.Messaging.EventHubs.Processor`          |
| `Azure.ResourceManager.EventHubs`              | Management plane (tạo hubs) | `dotnet add package Azure.ResourceManager.EventHubs`              |
| `Microsoft.Azure.WebJobs.Extensions.EventHubs` | Azure Functions binding     | `dotnet add package Microsoft.Azure.WebJobs.Extensions.EventHubs` |
