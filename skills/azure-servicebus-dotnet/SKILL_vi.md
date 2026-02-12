---
name: azure-servicebus-dotnet
description: |
  Azure Service Bus SDK cho .NET. Nhắn tin doanh nghiệp với queues, topics, subscriptions, và sessions. Sử dụng cho việc gửi tin nhắn đáng tin cậy, các mẫu pub/sub, xử lý dead letter, và xử lý nền. Triggers: "Service Bus", "ServiceBusClient", "ServiceBusSender", "ServiceBusReceiver", "ServiceBusProcessor", "message queue", "pub/sub .NET", "dead letter queue".
package: Azure.Messaging.ServiceBus
---

# Azure.Messaging.ServiceBus (.NET)

SDK nhắn tin doanh nghiệp để gửi tin nhắn đáng tin cậy với queues, topics, subscriptions, và sessions.

## Cài đặt

```bash
dotnet add package Azure.Messaging.ServiceBus
dotnet add package Azure.Identity
```

**Phiên bản Hiện tại**: v7.20.1 (stable)

## Biến Môi trường

```bash
AZURE_SERVICEBUS_FULLY_QUALIFIED_NAMESPACE=<namespace>.servicebus.windows.net
# Hoặc chuỗi kết nối (kém bảo mật hơn)
AZURE_SERVICEBUS_CONNECTION_STRING=Endpoint=sb://...
```

## Xác thực

### Microsoft Entra ID (Khuyên dùng)

```csharp
using Azure.Identity;
using Azure.Messaging.ServiceBus;

string fullyQualifiedNamespace = "<namespace>.servicebus.windows.net";
await using ServiceBusClient client = new(fullyQualifiedNamespace, new DefaultAzureCredential());
```

### Chuỗi Kết nối

```csharp
string connectionString = "<connection_string>";
await using ServiceBusClient client = new(connectionString);
```

### ASP.NET Core Dependency Injection

```csharp
services.AddAzureClients(builder =>
{
    builder.AddServiceBusClientWithNamespace("<namespace>.servicebus.windows.net");
    builder.UseCredential(new DefaultAzureCredential());
});
```

## Phân cấp Client

```
ServiceBusClient
├── CreateSender(queueOrTopicName)      → ServiceBusSender
├── CreateReceiver(queueName)           → ServiceBusReceiver
├── CreateReceiver(topicName, subName)  → ServiceBusReceiver
├── AcceptNextSessionAsync(queueName)   → ServiceBusSessionReceiver
├── CreateProcessor(queueName)          → ServiceBusProcessor
└── CreateSessionProcessor(queueName)   → ServiceBusSessionProcessor

ServiceBusAdministrationClient (client riêng biệt cho CRUD)
```

## Quy trình Công việc Cốt lõi

### 1. Gửi Tin nhắn

```csharp
await using ServiceBusClient client = new(fullyQualifiedNamespace, new DefaultAzureCredential());
ServiceBusSender sender = client.CreateSender("my-queue");

// Tin nhắn đơn lẻ
ServiceBusMessage message = new("Hello world!");
await sender.SendMessageAsync(message);

// Gửi theo lô an toàn (khuyên dùng)
using ServiceBusMessageBatch batch = await sender.CreateMessageBatchAsync();
if (batch.TryAddMessage(new ServiceBusMessage("Message 1")))
{
    // Tin nhắn đã được thêm thành công
}
if (batch.TryAddMessage(new ServiceBusMessage("Message 2")))
{
    // Tin nhắn đã được thêm thành công
}
await sender.SendMessagesAsync(batch);
```

### 2. Nhận Tin nhắn

```csharp
ServiceBusReceiver receiver = client.CreateReceiver("my-queue");

// Tin nhắn đơn lẻ
ServiceBusReceivedMessage message = await receiver.ReceiveMessageAsync();
string body = message.Body.ToString();
Console.WriteLine(body);

// Hoàn thành tin nhắn (xóa khỏi hàng đợi)
await receiver.CompleteMessageAsync(message);

// Nhận theo lô
IReadOnlyList<ServiceBusReceivedMessage> messages = await receiver.ReceiveMessagesAsync(maxMessages: 10);
foreach (var msg in messages)
{
    Console.WriteLine(msg.Body.ToString());
    await receiver.CompleteMessageAsync(msg);
}
```

### 3. Giải quyết Tin nhắn (Message Settlement)

```csharp
// Hoàn thành - xóa tin nhắn khỏi hàng đợi
await receiver.CompleteMessageAsync(message);

// Từ bỏ - giải phóng khóa, tin nhắn có thể được nhận lại
await receiver.AbandonMessageAsync(message);

// Trì hoãn - ngăn cản việc nhận thông thường, sử dụng ReceiveDeferredMessageAsync
await receiver.DeferMessageAsync(message);

// Dead Letter - di chuyển vào hàng đợi dead letter subqueue
await receiver.DeadLetterMessageAsync(message, "InvalidFormat", "Message body was not valid JSON");
```

### 4. Xử lý Nền với Processor

```csharp
ServiceBusProcessor processor = client.CreateProcessor("my-queue", new ServiceBusProcessorOptions
{
    AutoCompleteMessages = false,
    MaxConcurrentCalls = 2
});

processor.ProcessMessageAsync += async (args) =>
{
    try
    {
        string body = args.Message.Body.ToString();
        Console.WriteLine($"Received: {body}");
        await args.CompleteMessageAsync(args.Message);
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error processing: {ex.Message}");
        await args.AbandonMessageAsync(args.Message);
    }
};

processor.ProcessErrorAsync += (args) =>
{
    Console.WriteLine($"Error source: {args.ErrorSource}");
    Console.WriteLine($"Entity: {args.EntityPath}");
    Console.WriteLine($"Exception: {args.Exception}");
    return Task.CompletedTask;
};

await processor.StartProcessingAsync();
// ... ứng dụng chạy
await processor.StopProcessingAsync();
```

### 5. Sessions (Xử lý có Tự do)

```csharp
// Gửi tin nhắn session
ServiceBusMessage message = new("Hello")
{
    SessionId = "order-123"
};
await sender.SendMessageAsync(message);

// Nhận từ session có sẵn tiếp theo
ServiceBusSessionReceiver receiver = await client.AcceptNextSessionAsync("my-queue");

// Hoặc nhận từ session cụ thể
ServiceBusSessionReceiver receiver = await client.AcceptSessionAsync("my-queue", "order-123");

// Quản lý trạng thái session
await receiver.SetSessionStateAsync(new BinaryData("processing"));
BinaryData state = await receiver.GetSessionStateAsync();

// Gia hạn khóa session
await receiver.RenewSessionLockAsync();
```

### 6. Dead Letter Queue

```csharp
// Nhận từ dead letter queue
ServiceBusReceiver dlqReceiver = client.CreateReceiver("my-queue", new ServiceBusReceiverOptions
{
    SubQueue = SubQueue.DeadLetter
});

ServiceBusReceivedMessage dlqMessage = await dlqReceiver.ReceiveMessageAsync();

// Truy cập metadata dead letter
string reason = dlqMessage.DeadLetterReason;
string description = dlqMessage.DeadLetterErrorDescription;
Console.WriteLine($"Dead letter reason: {reason} - {description}");
```

### 7. Topics và Subscriptions

```csharp
// Gửi đến topic
ServiceBusSender topicSender = client.CreateSender("my-topic");
await topicSender.SendMessageAsync(new ServiceBusMessage("Broadcast message"));

// Nhận từ subscription
ServiceBusReceiver subReceiver = client.CreateReceiver("my-topic", "my-subscription");
var message = await subReceiver.ReceiveMessageAsync();
```

### 8. Quản trị (CRUD)

```csharp
var adminClient = new ServiceBusAdministrationClient(
    fullyQualifiedNamespace,
    new DefaultAzureCredential());

// Tạo queue
var options = new CreateQueueOptions("my-queue")
{
    MaxDeliveryCount = 10,
    LockDuration = TimeSpan.FromSeconds(30),
    RequiresSession = true,
    DeadLetteringOnMessageExpiration = true
};
QueueProperties queue = await adminClient.CreateQueueAsync(options);

// Cập nhật queue
queue.LockDuration = TimeSpan.FromSeconds(60);
await adminClient.UpdateQueueAsync(queue);

// Tạo topic và subscription
await adminClient.CreateTopicAsync(new CreateTopicOptions("my-topic"));
await adminClient.CreateSubscriptionAsync(new CreateSubscriptionOptions("my-topic", "my-subscription"));

// Xóa
await adminClient.DeleteQueueAsync("my-queue");
```

### 9. Giao dịch Liên thực thể (Cross-Entity Transactions)

```csharp
var options = new ServiceBusClientOptions { EnableCrossEntityTransactions = true };
await using var client = new ServiceBusClient(connectionString, options);

ServiceBusReceiver receiverA = client.CreateReceiver("queueA");
ServiceBusSender senderB = client.CreateSender("queueB");

ServiceBusReceivedMessage receivedMessage = await receiverA.ReceiveMessageAsync();

using (var ts = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled))
{
    await receiverA.CompleteMessageAsync(receivedMessage);
    await senderB.SendMessageAsync(new ServiceBusMessage("Forwarded"));
    ts.Complete();
}
```

## Tham chiếu Các Loại Chính

| Loại                             | Mục đích                              |
| -------------------------------- | ------------------------------------- |
| `ServiceBusClient`               | Điểm vào chính, quản lý kết nối       |
| `ServiceBusSender`               | Gửi tin nhắn đến queues/topics        |
| `ServiceBusReceiver`             | Nhận tin nhắn từ queues/subscriptions |
| `ServiceBusSessionReceiver`      | Nhận tin nhắn session                 |
| `ServiceBusProcessor`            | Xử lý tin nhắn nền                    |
| `ServiceBusSessionProcessor`     | Xử lý session nền                     |
| `ServiceBusAdministrationClient` | CRUD cho queues/topics/subscriptions  |
| `ServiceBusMessage`              | Tin nhắn để gửi                       |
| `ServiceBusReceivedMessage`      | Tin nhắn đã nhận với metadata         |
| `ServiceBusMessageBatch`         | Lô tin nhắn                           |

## Thực hành Tốt nhất

1. **Sử dụng singletons** — Clients, senders, receivers, và processors là an toàn luồng (thread-safe)
2. **Luôn dispose** — Sử dụng `await using` hoặc gọi `DisposeAsync()`
3. **Thứ tự dispose** — Đóng senders/receivers/processors trước, sau đó đến client
4. **Sử dụng DefaultAzureCredential** — Ưu tiên hơn chuỗi kết nối cho production
5. **Sử dụng processors cho công việc nền** — Tự động xử lý gia hạn khóa
6. **Sử dụng gửi theo lô an toàn** — `CreateMessageBatchAsync()` và `TryAddMessage()`
7. **Xử lý lỗi tạm thời** — Sử dụng `ServiceBusException.Reason`
8. **Cấu hình transport** — Sử dụng `AmqpWebSockets` nếu cổng 5671/5672 bị chặn
9. **Thiết lập thời gian khóa phù hợp** — Mặc định là 30 giây
10. **Sử dụng sessions để sắp xếp** — FIFO trong một session

## Xử lý Lỗi

```csharp
try
{
    await sender.SendMessageAsync(message);
}
catch (ServiceBusException ex) when (ex.Reason == ServiceBusFailureReason.ServiceBusy)
{
    // Thử lại với backoff
}
catch (ServiceBusException ex)
{
    Console.WriteLine($"Service Bus Error: {ex.Reason} - {ex.Message}");
}
```

## Các SDK Liên quan

| SDK                          | Mục đích              | Cài đặt                                         |
| ---------------------------- | --------------------- | ----------------------------------------------- |
| `Azure.Messaging.ServiceBus` | Service Bus (SDK này) | `dotnet add package Azure.Messaging.ServiceBus` |
| `Azure.Messaging.EventHubs`  | Event streaming       | `dotnet add package Azure.Messaging.EventHubs`  |
| `Azure.Messaging.EventGrid`  | Event routing         | `dotnet add package Azure.Messaging.EventGrid`  |

## Liên kết Tham khảo

| Tài nguyên      | URL                                                                                                               |
| --------------- | ----------------------------------------------------------------------------------------------------------------- |
| NuGet Package   | https://www.nuget.org/packages/Azure.Messaging.ServiceBus                                                         |
| API Reference   | https://learn.microsoft.com/dotnet/api/azure.messaging.servicebus                                                 |
| GitHub Source   | https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/servicebus/Azure.Messaging.ServiceBus                    |
| Troubleshooting | https://github.com/Azure/azure-sdk-for-net/blob/main/sdk/servicebus/Azure.Messaging.ServiceBus/TROUBLESHOOTING.md |
