---
name: azure-resource-manager-durabletask-dotnet
description: |
  Azure Resource Manager SDK cho Durable Task Scheduler trong .NET. Sử dụng cho các hoạt động MANAGEMENT PLANE: tạo/quản lý Durable Task Schedulers, Task Hubs, và các chính sách lưu giữ (retention policies) thông qua Azure Resource Manager. Triggers: "Durable Task Scheduler", "create scheduler", "task hub", "DurableTaskSchedulerResource", "provision Durable Task", "orchestration scheduler".
package: Azure.ResourceManager.DurableTask
---

# Azure.ResourceManager.DurableTask (.NET)

SDK management plane để cung cấp và quản lý tài nguyên Azure Durable Task Scheduler thông qua Azure Resource Manager.

> **⚠️ Management vs Data Plane**
>
> - **SDK này (Azure.ResourceManager.DurableTask)**: Tạo schedulers, task hubs, cấu hình các chính sách lưu giữ
> - **Data Plane SDK (Microsoft.DurableTask.Client.AzureManaged)**: Bắt đầu orchestrations, truy vấn instances, gửi events

## Cài đặt

```bash
dotnet add package Azure.ResourceManager.DurableTask
dotnet add package Azure.Identity
```

**Phiên bản Hiện tại**: Stable v1.0.0 (2025-11-03), Preview v1.0.0-beta.1 (2025-04-24)
**Phiên bản API**: 2025-11-01

## Biến Môi trường

```bash
AZURE_SUBSCRIPTION_ID=<your-subscription-id>
AZURE_RESOURCE_GROUP=<your-resource-group>
# Cho service principal auth (tùy chọn)
AZURE_TENANT_ID=<tenant-id>
AZURE_CLIENT_ID=<client-id>
AZURE_CLIENT_SECRET=<client-secret>
```

## Xác thực

```csharp
using Azure.Identity;
using Azure.ResourceManager;
using Azure.ResourceManager.DurableTask;

// Luôn sử dụng DefaultAzureCredential
var credential = new DefaultAzureCredential();
var armClient = new ArmClient(credential);

// Lấy subscription
var subscriptionId = Environment.GetEnvironmentVariable("AZURE_SUBSCRIPTION_ID");
var subscription = armClient.GetSubscriptionResource(
    new ResourceIdentifier($"/subscriptions/{subscriptionId}"));
```

## Phân cấp Tài nguyên

```
ArmClient
└── SubscriptionResource
    └── ResourceGroupResource
        └── DurableTaskSchedulerResource
            ├── DurableTaskHubResource
            └── DurableTaskRetentionPolicyResource
```

## Quy trình Công việc Cốt lõi

### 1. Tạo Durable Task Scheduler

```csharp
using Azure.ResourceManager.DurableTask;
using Azure.ResourceManager.DurableTask.Models;

// Lấy resource group
var resourceGroup = await subscription
    .GetResourceGroupAsync("my-resource-group");

// Định nghĩa scheduler với Dedicated SKU
var schedulerData = new DurableTaskSchedulerData(AzureLocation.EastUS)
{
    Properties = new DurableTaskSchedulerProperties
    {
        Sku = new DurableTaskSchedulerSku(DurableTaskSchedulerSkuName.Dedicated)
        {
            Capacity = 1  // Số lượng instances
        },
        // Tùy chọn: IP allowlist cho bảo mật mạng
        IPAllowlist = { "10.0.0.0/24", "192.168.1.0/24" }
    }
};

// Tạo scheduler (long-running operation)
var schedulerCollection = resourceGroup.Value.GetDurableTaskSchedulers();
var operation = await schedulerCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "my-scheduler",
    schedulerData);

DurableTaskSchedulerResource scheduler = operation.Value;
Console.WriteLine($"Scheduler created: {scheduler.Data.Name}");
Console.WriteLine($"Endpoint: {scheduler.Data.Properties.Endpoint}");
```

### 2. Tạo Scheduler với Consumption SKU

```csharp
// Consumption SKU (serverless)
var consumptionSchedulerData = new DurableTaskSchedulerData(AzureLocation.EastUS)
{
    Properties = new DurableTaskSchedulerProperties
    {
        Sku = new DurableTaskSchedulerSku(DurableTaskSchedulerSkuName.Consumption)
        // Không cần capacity cho consumption
    }
};

var operation = await schedulerCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "my-serverless-scheduler",
    consumptionSchedulerData);
```

### 3. Tạo Task Hub

```csharp
// Task hubs được tạo dưới một scheduler
var taskHubData = new DurableTaskHubData
{
    // Các thuộc tính là tùy chọn cho task hub cơ bản
};

var taskHubCollection = scheduler.GetDurableTaskHubs();
var hubOperation = await taskHubCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "my-taskhub",
    taskHubData);

DurableTaskHubResource taskHub = hubOperation.Value;
Console.WriteLine($"Task Hub created: {taskHub.Data.Name}");
```

### 4. Liệt kê Schedulers

```csharp
// Liệt kê tất cả schedulers trong subscription
await foreach (var sched in subscription.GetDurableTaskSchedulersAsync())
{
    Console.WriteLine($"Scheduler: {sched.Data.Name}");
    Console.WriteLine($"  Location: {sched.Data.Location}");
    Console.WriteLine($"  SKU: {sched.Data.Properties.Sku?.Name}");
    Console.WriteLine($"  Endpoint: {sched.Data.Properties.Endpoint}");
}

// Liệt kê schedulers trong resource group
var schedulers = resourceGroup.Value.GetDurableTaskSchedulers();
await foreach (var sched in schedulers.GetAllAsync())
{
    Console.WriteLine($"Scheduler: {sched.Data.Name}");
}
```

### 5. Lấy Scheduler theo Tên

```csharp
// Lấy scheduler hiện có
var existingScheduler = await schedulerCollection.GetAsync("my-scheduler");
Console.WriteLine($"Found: {existingScheduler.Value.Data.Name}");

// Hoặc sử dụng phương thức mở rộng
var schedulerResource = armClient.GetDurableTaskSchedulerResource(
    DurableTaskSchedulerResource.CreateResourceIdentifier(
        subscriptionId,
        "my-resource-group",
        "my-scheduler"));
var scheduler = await schedulerResource.GetAsync();
```

### 6. Cập nhật Scheduler

```csharp
// Lấy scheduler hiện tại
var scheduler = await schedulerCollection.GetAsync("my-scheduler");

// Cập nhật với cấu hình mới
var updateData = new DurableTaskSchedulerData(scheduler.Value.Data.Location)
{
    Properties = new DurableTaskSchedulerProperties
    {
        Sku = new DurableTaskSchedulerSku(DurableTaskSchedulerSkuName.Dedicated)
        {
            Capacity = 2  // Scale up
        },
        IPAllowlist = { "10.0.0.0/16" }  // Cập nhật IP allowlist
    }
};

var updateOperation = await schedulerCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "my-scheduler",
    updateData);
```

### 7. Xóa Tài nguyên

```csharp
// Xóa task hub trước
var taskHub = await scheduler.GetDurableTaskHubs().GetAsync("my-taskhub");
await taskHub.Value.DeleteAsync(WaitUntil.Completed);

// Sau đó xóa scheduler
await scheduler.DeleteAsync(WaitUntil.Completed);
```

### 8. Quản lý Các Chính sách Lưu giữ

```csharp
// Lấy bộ sưu tập chính sách lưu giữ
var retentionPolicies = scheduler.GetDurableTaskRetentionPolicies();

// Tạo hoặc cập nhật chính sách lưu giữ
var retentionData = new DurableTaskRetentionPolicyData
{
    Properties = new DurableTaskRetentionPolicyProperties
    {
        // Cấu hình cài đặt lưu giữ
    }
};

var retentionOperation = await retentionPolicies.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "default",  // Tên chính sách
    retentionData);
```

## Tham chiếu Các Loại Chính

| Loại                                 | Mục đích                                      |
| ------------------------------------ | --------------------------------------------- |
| `ArmClient`                          | Điểm vào cho tất cả các hoạt động ARM         |
| `DurableTaskSchedulerResource`       | Đại diện cho một Durable Task Scheduler       |
| `DurableTaskSchedulerCollection`     | Collection cho scheduler CRUD                 |
| `DurableTaskSchedulerData`           | Payload tạo/cập nhật scheduler                |
| `DurableTaskSchedulerProperties`     | Cấu hình scheduler (SKU, IPAllowlist)         |
| `DurableTaskSchedulerSku`            | Cấu hình SKU (Tên, Capacity, RedundancyState) |
| `DurableTaskSchedulerSkuName`        | Các tùy chọn SKU: `Dedicated`, `Consumption`  |
| `DurableTaskHubResource`             | Đại diện cho một Task Hub                     |
| `DurableTaskHubCollection`           | Collection cho task hub CRUD                  |
| `DurableTaskHubData`                 | Payload tạo task hub                          |
| `DurableTaskRetentionPolicyResource` | Quản lý chính sách lưu giữ                    |
| `DurableTaskRetentionPolicyData`     | Cấu hình chính sách lưu giữ                   |
| `DurableTaskExtensions`              | Các phương thức mở rộng cho ARM client        |

## Các Tùy chọn SKU

| SKU           | Mô tả                                                   | Trường hợp Sử dụng                                      |
| ------------- | ------------------------------------------------------- | ------------------------------------------------------- |
| `Dedicated`   | Capacity cố định với số lượng instances có thể cấu hình | Khối lượng công việc production, hiệu suất dự đoán được |
| `Consumption` | Serverless, tự động mở rộng (auto-scaling)              | Phát triển, khối lượng công việc thay đổi               |

## Các Phương thức Mở rộng

SDK cung cấp các phương thức mở rộng trên `SubscriptionResource` và `ResourceGroupResource`:

```csharp
// Trên SubscriptionResource
subscription.GetDurableTaskSchedulers();           // Liệt kê tất cả trong subscription
subscription.GetDurableTaskSchedulersAsync();      // Async enumerable

// Trên ResourceGroupResource
resourceGroup.GetDurableTaskSchedulers();          // Lấy collection
resourceGroup.GetDurableTaskSchedulerAsync(name);  // Lấy theo tên

// Trên ArmClient
armClient.GetDurableTaskSchedulerResource(id);     // Lấy theo resource ID
armClient.GetDurableTaskHubResource(id);           // Lấy task hub theo ID
```

## Thực hành Tốt nhất

1. **Sử dụng `WaitUntil.Completed`** cho các hoạt động phải hoàn thành trước khi tiếp tục
2. **Sử dụng `WaitUntil.Started`** khi bạn muốn thăm dò thủ công hoặc chạy các hoạt động song song
3. **Luôn sử dụng `DefaultAzureCredential`** — không bao giờ hardcode keys
4. **Xử lý `RequestFailedException`** cho các lỗi ARM API
5. **Sử dụng `CreateOrUpdateAsync`** cho các hoạt động idempotent
6. **Xóa task hubs trước schedulers** — schedulers có task hubs không thể bị xóa
7. **Sử dụng IP allowlists** cho bảo mật mạng trong production

## Xử lý Lỗi

```csharp
using Azure;

try
{
    var operation = await schedulerCollection.CreateOrUpdateAsync(
        WaitUntil.Completed, schedulerName, schedulerData);
}
catch (RequestFailedException ex) when (ex.Status == 409)
{
    Console.WriteLine("Scheduler already exists");
}
catch (RequestFailedException ex) when (ex.Status == 404)
{
    Console.WriteLine("Resource group not found");
}
catch (RequestFailedException ex)
{
    Console.WriteLine($"ARM Error: {ex.Status} - {ex.ErrorCode}: {ex.Message}");
}
```

## Ví dụ Hoàn chỉnh

```csharp
using Azure;
using Azure.Identity;
using Azure.ResourceManager;
using Azure.ResourceManager.DurableTask;
using Azure.ResourceManager.DurableTask.Models;
using Azure.ResourceManager.Resources;

// Setup
var credential = new DefaultAzureCredential();
var armClient = new ArmClient(credential);

var subscriptionId = Environment.GetEnvironmentVariable("AZURE_SUBSCRIPTION_ID")!;
var resourceGroupName = Environment.GetEnvironmentVariable("AZURE_RESOURCE_GROUP")!;

var subscription = armClient.GetSubscriptionResource(
    new ResourceIdentifier($"/subscriptions/{subscriptionId}"));
var resourceGroup = await subscription.GetResourceGroupAsync(resourceGroupName);

// Tạo scheduler
var schedulerData = new DurableTaskSchedulerData(AzureLocation.EastUS)
{
    Properties = new DurableTaskSchedulerProperties
    {
        Sku = new DurableTaskSchedulerSku(DurableTaskSchedulerSkuName.Dedicated)
        {
            Capacity = 1
        }
    }
};

var schedulerCollection = resourceGroup.Value.GetDurableTaskSchedulers();
var schedulerOp = await schedulerCollection.CreateOrUpdateAsync(
    WaitUntil.Completed, "my-scheduler", schedulerData);
var scheduler = schedulerOp.Value;

Console.WriteLine($"Scheduler endpoint: {scheduler.Data.Properties.Endpoint}");

// Tạo task hub
var taskHubData = new DurableTaskHubData();
var taskHubOp = await scheduler.GetDurableTaskHubs().CreateOrUpdateAsync(
    WaitUntil.Completed, "my-taskhub", taskHubData);
var taskHub = taskHubOp.Value;

Console.WriteLine($"Task Hub: {taskHub.Data.Name}");

// Dọn dẹp
await taskHub.DeleteAsync(WaitUntil.Completed);
await scheduler.DeleteAsync(WaitUntil.Completed);
```

## Các SDK Liên quan

| SDK                                         | Mục đích                                | Cài đặt                                                        |
| ------------------------------------------- | --------------------------------------- | -------------------------------------------------------------- |
| `Azure.ResourceManager.DurableTask`         | Management plane (SDK này)              | `dotnet add package Azure.ResourceManager.DurableTask`         |
| `Microsoft.DurableTask.Client.AzureManaged` | Data plane (orchestrations, activities) | `dotnet add package Microsoft.DurableTask.Client.AzureManaged` |
| `Microsoft.DurableTask.Worker.AzureManaged` | Worker để chạy orchestrations           | `dotnet add package Microsoft.DurableTask.Worker.AzureManaged` |
| `Azure.Identity`                            | Xác thực                                | `dotnet add package Azure.Identity`                            |
| `Azure.ResourceManager`                     | Base ARM SDK                            | `dotnet add package Azure.ResourceManager`                     |

## Tham khảo Nguồn

- [GitHub: Azure.ResourceManager.DurableTask](https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/durabletask/Azure.ResourceManager.DurableTask)
- [NuGet: Azure.ResourceManager.DurableTask](https://www.nuget.org/packages/Azure.ResourceManager.DurableTask)
