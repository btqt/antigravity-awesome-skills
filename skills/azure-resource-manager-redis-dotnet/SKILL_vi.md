---
name: azure-resource-manager-redis-dotnet
description: |
  Azure Resource Manager SDK cho Redis trong .NET. Sử dụng cho các hoạt động MANAGEMENT PLANE: tạo/quản lý các instance Azure Cache for Redis, quy tắc tường lửa, access keys, lịch vá lỗi, linked servers (geo-replication), và các private endpoints thông qua Azure Resource Manager. KHÔNG dùng cho các hoạt động data plane (get/set keys, pub/sub) - hãy sử dụng StackExchange.Redis cho việc đó. Triggers: "Redis cache", "create Redis", "manage Redis", "ARM Redis", "RedisResource", "provision Redis", "Azure Cache for Redis".
package: Azure.ResourceManager.Redis
---

# Azure.ResourceManager.Redis (.NET)

SDK management plane để cung cấp và quản lý tài nguyên Azure Cache for Redis thông qua Azure Resource Manager.

> **⚠️ Management vs Data Plane**
>
> - **SDK này (Azure.ResourceManager.Redis)**: Tạo caches, cấu hình tường lửa, quản lý access keys, thiết lập geo-replication
> - **Data Plane SDK (StackExchange.Redis)**: Get/set keys, pub/sub, streams, Lua scripts

## Cài đặt

```bash
dotnet add package Azure.ResourceManager.Redis
dotnet add package Azure.Identity
```

**Phiên bản Hiện tại**: 1.5.1 (Stable)  
**Phiên bản API**: 2024-11-01  
**Framework Mục tiêu**: .NET 8.0, .NET Standard 2.0

## Biến Môi trường

```bash
AZURE_SUBSCRIPTION_ID=<your-subscription-id>
# Cho service principal auth (tùy chọn)
AZURE_TENANT_ID=<tenant-id>
AZURE_CLIENT_ID=<client-id>
AZURE_CLIENT_SECRET=<client-secret>
```

## Xác thực

```csharp
using Azure.Identity;
using Azure.ResourceManager;
using Azure.ResourceManager.Redis;

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
        └── RedisResource
            ├── RedisFirewallRuleResource
            ├── RedisPatchScheduleResource
            ├── RedisLinkedServerWithPropertyResource
            ├── RedisPrivateEndpointConnectionResource
            └── RedisCacheAccessPolicyResource
```

## Quy trình Công việc Cốt lõi

### 1. Tạo Redis Cache

```csharp
using Azure.ResourceManager.Redis;
using Azure.ResourceManager.Redis.Models;

// Lấy resource group
var resourceGroup = await subscription
    .GetResourceGroupAsync("my-resource-group");

// Định nghĩa cấu hình cache
var cacheData = new RedisCreateOrUpdateContent(
    location: AzureLocation.EastUS,
    sku: new RedisSku(RedisSkuName.Standard, RedisSkuFamily.BasicOrStandard, 1))
{
    EnableNonSslPort = false,
    MinimumTlsVersion = RedisTlsVersion.Tls1_2,
    RedisConfiguration = new RedisCommonConfiguration
    {
        MaxMemoryPolicy = "volatile-lru"
    },
    Tags =
    {
        ["environment"] = "production"
    }
};

// Tạo cache (long-running operation)
var cacheCollection = resourceGroup.Value.GetAllRedis();
var operation = await cacheCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "my-redis-cache",
    cacheData);

RedisResource cache = operation.Value;
Console.WriteLine($"Cache created: {cache.Data.HostName}");
```

### 2. Lấy Redis Cache

```csharp
// Lấy cache hiện có
var cache = await resourceGroup.Value
    .GetRedisAsync("my-redis-cache");

Console.WriteLine($"Host: {cache.Value.Data.HostName}");
Console.WriteLine($"Port: {cache.Value.Data.Port}");
Console.WriteLine($"SSL Port: {cache.Value.Data.SslPort}");
Console.WriteLine($"Provisioning State: {cache.Value.Data.ProvisioningState}");
```

### 3. Cập nhật Redis Cache

```csharp
var patchData = new RedisPatch
{
    Sku = new RedisSku(RedisSkuName.Standard, RedisSkuFamily.BasicOrStandard, 2),
    RedisConfiguration = new RedisCommonConfiguration
    {
        MaxMemoryPolicy = "allkeys-lru"
    }
};

var updateOperation = await cache.Value.UpdateAsync(
    WaitUntil.Completed,
    patchData);
```

### 4. Xóa Redis Cache

```csharp
await cache.Value.DeleteAsync(WaitUntil.Completed);
```

### 5. Lấy Access Keys

```csharp
var keys = await cache.Value.GetKeysAsync();
Console.WriteLine($"Primary Key: {keys.Value.PrimaryKey}");
Console.WriteLine($"Secondary Key: {keys.Value.SecondaryKey}");
```

### 6. Tạo lại Access Keys

```csharp
var regenerateContent = new RedisRegenerateKeyContent(RedisRegenerateKeyType.Primary);
var newKeys = await cache.Value.RegenerateKeyAsync(regenerateContent);
Console.WriteLine($"New Primary Key: {newKeys.Value.PrimaryKey}");
```

### 7. Quản lý Quy tắc Tường lửa

```csharp
// Tạo quy tắc tường lửa
var firewallData = new RedisFirewallRuleData(
    startIP: System.Net.IPAddress.Parse("10.0.0.1"),
    endIP: System.Net.IPAddress.Parse("10.0.0.255"));

var firewallCollection = cache.Value.GetRedisFirewallRules();
var firewallOperation = await firewallCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "allow-internal-network",
    firewallData);

// Liệt kê tất cả quy tắc tường lửa
await foreach (var rule in firewallCollection.GetAllAsync())
{
    Console.WriteLine($"Rule: {rule.Data.Name} ({rule.Data.StartIP} - {rule.Data.EndIP})");
}

// Xóa quy tắc tường lửa
var ruleToDelete = await firewallCollection.GetAsync("allow-internal-network");
await ruleToDelete.Value.DeleteAsync(WaitUntil.Completed);
```

### 8. Cấu hình Lịch vá lỗi (Premium SKU)

```csharp
// Lịch vá lỗi yêu cầu Premium SKU
var scheduleData = new RedisPatchScheduleData(
    new[]
    {
        new RedisPatchScheduleSetting(RedisDayOfWeek.Saturday, 2) // 2 AM Saturday
        {
            MaintenanceWindow = TimeSpan.FromHours(5)
        },
        new RedisPatchScheduleSetting(RedisDayOfWeek.Sunday, 2) // 2 AM Sunday
        {
            MaintenanceWindow = TimeSpan.FromHours(5)
        }
    });

var scheduleCollection = cache.Value.GetRedisPatchSchedules();
await scheduleCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    RedisPatchScheduleDefaultName.Default,
    scheduleData);
```

### 9. Import/Export Dữ liệu (Premium SKU)

```csharp
// Import dữ liệu từ blob storage
var importContent = new ImportRdbContent(
    files: new[] { "https://mystorageaccount.blob.core.windows.net/container/dump.rdb" },
    format: "RDB");

await cache.Value.ImportDataAsync(WaitUntil.Completed, importContent);

// Export dữ liệu ra blob storage
var exportContent = new ExportRdbContent(
    prefix: "backup",
    container: "https://mystorageaccount.blob.core.windows.net/container?sastoken",
    format: "RDB");

await cache.Value.ExportDataAsync(WaitUntil.Completed, exportContent);
```

### 10. Buộc Khởi động lại (Force Reboot)

```csharp
var rebootContent = new RedisRebootContent
{
    RebootType = RedisRebootType.AllNodes,
    ShardId = 0 // Cho clustered caches
};

await cache.Value.ForceRebootAsync(rebootContent);
```

## Tham chiếu SKU

| SKU      | Family | Dung lượng | Tính năng                                      |
| -------- | ------ | ---------- | ---------------------------------------------- |
| Basic    | C      | 0-6        | Một node, không có SLA, chỉ dev/test           |
| Standard | C      | 0-6        | Hai nodes (chính/bản sao), SLA                 |
| Premium  | P      | 1-5        | Clustering, geo-replication, VNet, persistence |

**Các kích thước dung lượng (Family C - Basic/Standard)**:

- C0: 250 MB
- C1: 1 GB
- C2: 2.5 GB
- C3: 6 GB
- C4: 13 GB
- C5: 26 GB
- C6: 53 GB

**Các kích thước dung lượng (Family P - Premium)**:

- P1: 6 GB mỗi shard
- P2: 13 GB mỗi shard
- P3: 26 GB mỗi shard
- P4: 53 GB mỗi shard
- P5: 120 GB mỗi shard

## Tham chiếu Các Loại Chính

| Loại                                     | Mục đích                                |
| ---------------------------------------- | --------------------------------------- |
| `ArmClient`                              | Điểm vào cho tất cả các hoạt động ARM   |
| `RedisResource`                          | Đại diện cho một instance Redis cache   |
| `RedisCollection`                        | Collection cho các hoạt động CRUD cache |
| `RedisFirewallRuleResource`              | Quy tắc tường lửa lọc IP                |
| `RedisPatchScheduleResource`             | Cấu hình cửa sổ bảo trì                 |
| `RedisLinkedServerWithPropertyResource`  | Geo-replication linked server           |
| `RedisPrivateEndpointConnectionResource` | Kết nối private endpoint                |
| `RedisCacheAccessPolicyResource`         | Chính sách truy cập RBAC                |
| `RedisCreateOrUpdateContent`             | Payload tạo cache                       |
| `RedisPatch`                             | Payload cập nhật cache                  |
| `RedisSku`                               | Cấu hình SKU (name, family, capacity)   |
| `RedisAccessKeys`                        | Access keys chính và phụ                |
| `RedisRegenerateKeyContent`              | Yêu cầu tạo lại key                     |

## Thực hành Tốt nhất

1. **Sử dụng `WaitUntil.Completed`** cho các hoạt động phải hoàn thành trước khi tiếp tục
2. **Sử dụng `WaitUntil.Started`** khi bạn muốn thăm dò thủ công hoặc chạy các hoạt động song song
3. **Luôn sử dụng `DefaultAzureCredential`** — không bao giờ hardcode keys
4. **Xử lý `RequestFailedException`** cho các lỗi ARM API
5. **Sử dụng `CreateOrUpdateAsync`** cho các hoạt động idempotent
6. **Điều hướng phân cấp** qua các phương thức `Get*` (ví dụ: `cache.GetRedisFirewallRules()`)
7. **Sử dụng Premium SKU** cho khối lượng công việc production yêu cầu geo-replication, clustering, hoặc persistence
8. **Bật TLS 1.2 minimum** — đặt `MinimumTlsVersion = RedisTlsVersion.Tls1_2`
9. **Vô hiệu hóa non-SSL port** — đặt `EnableNonSslPort = false` để bảo mật
10. **Xoay vòng keys thường xuyên** — sử dụng `RegenerateKeyAsync` và cập nhật chuỗi kết nối

## Xử lý Lỗi

```csharp
using Azure;

try
{
    var operation = await cacheCollection.CreateOrUpdateAsync(
        WaitUntil.Completed, cacheName, cacheData);
}
catch (RequestFailedException ex) when (ex.Status == 409)
{
    Console.WriteLine("Cache already exists");
}
catch (RequestFailedException ex) when (ex.Status == 400)
{
    Console.WriteLine($"Invalid configuration: {ex.Message}");
}
catch (RequestFailedException ex)
{
    Console.WriteLine($"ARM Error: {ex.Status} - {ex.ErrorCode}: {ex.Message}");
}
```

## Các Vấn đề Phổ biến

1. **Không cho phép hạ cấp SKU** — Bạn không thể hạ cấp từ Premium xuống Standard/Basic
2. **Clustering yêu cầu Premium** — Cấu hình Shard chỉ khả dụng trên Premium SKU
3. **Geo-replication yêu cầu Premium** — Linked servers chỉ hoạt động với Premium caches
4. **VNet injection yêu cầu Premium** — Hỗ trợ mạng ảo là Premium-only
5. **Lịch vá lỗi yêu cầu Premium** — Cửa sổ bảo trì chỉ cấu hình được trên Premium
6. **Tên Cache là duy nhất toàn cầu** — Tên Redis cache phải là duy nhất trên tất cả các subscriptions Azure
7. **Thời gian cung cấp lâu** — Tạo cache có thể mất 15-20 phút; sử dụng `WaitUntil.Started` cho các mẫu async

## Kết nối với StackExchange.Redis (Data Plane)

Sau khi tạo cache với SDK quản lý này, sử dụng StackExchange.Redis cho các hoạt động dữ liệu:

```csharp
using StackExchange.Redis;

// Lấy thông tin kết nối từ management SDK
var cache = await resourceGroup.Value.GetRedisAsync("my-redis-cache");
var keys = await cache.Value.GetKeysAsync();

// Kết nối với StackExchange.Redis
var connectionString = $"{cache.Value.Data.HostName}:{cache.Value.Data.SslPort},password={keys.Value.PrimaryKey},ssl=True,abortConnect=False";
var connection = ConnectionMultiplexer.Connect(connectionString);
var db = connection.GetDatabase();

// Các hoạt động dữ liệu
await db.StringSetAsync("key", "value");
var value = await db.StringGetAsync("key");
```

## Các SDK Liên quan

| SDK                                  | Mục đích                               | Cài đặt                                                 |
| ------------------------------------ | -------------------------------------- | ------------------------------------------------------- |
| `StackExchange.Redis`                | Data plane (get/set, pub/sub, streams) | `dotnet add package StackExchange.Redis`                |
| `Azure.ResourceManager.Redis`        | Management plane (SDK này)             | `dotnet add package Azure.ResourceManager.Redis`        |
| `Microsoft.Azure.StackExchangeRedis` | Azure-specific Redis extensions        | `dotnet add package Microsoft.Azure.StackExchangeRedis` |
