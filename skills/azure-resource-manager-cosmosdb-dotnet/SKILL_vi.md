---
name: azure-resource-manager-cosmosdb-dotnet
description: |
  Azure Resource Manager SDK cho Cosmos DB trong .NET. Sử dụng cho các hoạt động MANAGEMENT PLANE: tạo/quản lý tài khoản Cosmos DB, cơ sở dữ liệu, container, cài đặt thông lượng (throughput), và RBAC thông qua Azure Resource Manager. KHÔNG dùng cho các hoạt động data plane (CRUD trên tài liệu) - hãy sử dụng Microsoft.Azure.Cosmos cho việc đó. Triggers: "Cosmos DB account", "create Cosmos account", "manage Cosmos resources", "ARM Cosmos", "CosmosDBAccountResource", "provision Cosmos DB".
package: Azure.ResourceManager.CosmosDB
---

# Azure.ResourceManager.CosmosDB (.NET)

SDK management plane để cung cấp và quản lý tài nguyên Azure Cosmos DB thông qua Azure Resource Manager.

> **⚠️ Management vs Data Plane**
>
> - **SDK này (Azure.ResourceManager.CosmosDB)**: Tạo tài khoản, cơ sở dữ liệu, container, cấu hình thông lượng, quản lý RBAC
> - **Data Plane SDK (Microsoft.Azure.Cosmos)**: Các hoạt động CRUD trên tài liệu, truy vấn, thực thi stored procedures

## Cài đặt

```bash
dotnet add package Azure.ResourceManager.CosmosDB
dotnet add package Azure.Identity
```

**Phiên bản Hiện tại**: Stable v1.4.0, Preview v1.4.0-beta.13

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
using Azure.ResourceManager.CosmosDB;

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
        └── CosmosDBAccountResource
            ├── CosmosDBSqlDatabaseResource
            │   └── CosmosDBSqlContainerResource
            │       ├── CosmosDBSqlStoredProcedureResource
            │       ├── CosmosDBSqlTriggerResource
            │       └── CosmosDBSqlUserDefinedFunctionResource
            ├── CassandraKeyspaceResource
            ├── GremlinDatabaseResource
            ├── MongoDBDatabaseResource
            └── CosmosDBTableResource
```

## Quy trình Công việc Cốt lõi

### 1. Tạo Tài khoản Cosmos DB

```csharp
using Azure.ResourceManager.CosmosDB;
using Azure.ResourceManager.CosmosDB.Models;

// Lấy resource group
var resourceGroup = await subscription
    .GetResourceGroupAsync("my-resource-group");

// Định nghĩa account
var accountData = new CosmosDBAccountCreateOrUpdateContent(
    location: AzureLocation.EastUS,
    locations: new[]
    {
        new CosmosDBAccountLocation
        {
            LocationName = AzureLocation.EastUS,
            FailoverPriority = 0,
            IsZoneRedundant = false
        }
    })
{
    Kind = CosmosDBAccountKind.GlobalDocumentDB,
    ConsistencyPolicy = new ConsistencyPolicy(DefaultConsistencyLevel.Session),
    EnableAutomaticFailover = true
};

// Tạo account (long-running operation)
var accountCollection = resourceGroup.Value.GetCosmosDBAccounts();
var operation = await accountCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "my-cosmos-account",
    accountData);

CosmosDBAccountResource account = operation.Value;
```

### 2. Tạo SQL Database

```csharp
var databaseData = new CosmosDBSqlDatabaseCreateOrUpdateContent(
    new CosmosDBSqlDatabaseResourceInfo("my-database"));

var databaseCollection = account.GetCosmosDBSqlDatabases();
var dbOperation = await databaseCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "my-database",
    databaseData);

CosmosDBSqlDatabaseResource database = dbOperation.Value;
```

### 3. Tạo SQL Container

```csharp
var containerData = new CosmosDBSqlContainerCreateOrUpdateContent(
    new CosmosDBSqlContainerResourceInfo("my-container")
    {
        PartitionKey = new CosmosDBContainerPartitionKey
        {
            Paths = { "/partitionKey" },
            Kind = CosmosDBPartitionKind.Hash
        },
        IndexingPolicy = new CosmosDBIndexingPolicy
        {
            Automatic = true,
            IndexingMode = CosmosDBIndexingMode.Consistent
        },
        DefaultTtl = 86400 // 24 giờ
    });

var containerCollection = database.GetCosmosDBSqlContainers();
var containerOperation = await containerCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "my-container",
    containerData);

CosmosDBSqlContainerResource container = containerOperation.Value;
```

### 4. Cấu hình Thông lượng (Throughput)

```csharp
// Thông lượng thủ công
var throughputData = new ThroughputSettingsUpdateData(
    new ThroughputSettingsResourceInfo
    {
        Throughput = 400
    });

// Thông lượng tự động mở rộng (Autoscale)
var autoscaleData = new ThroughputSettingsUpdateData(
    new ThroughputSettingsResourceInfo
    {
        AutoscaleSettings = new AutoscaleSettingsResourceInfo
        {
            MaxThroughput = 4000
        }
    });

// Áp dụng cho database
await database.CreateOrUpdateCosmosDBSqlDatabaseThroughputAsync(
    WaitUntil.Completed,
    throughputData);
```

### 5. Lấy Thông tin Kết nối

```csharp
// Lấy keys
var keys = await account.GetKeysAsync();
Console.WriteLine($"Primary Key: {keys.Value.PrimaryMasterKey}");

// Lấy chuỗi kết nối
var connectionStrings = await account.GetConnectionStringsAsync();
foreach (var cs in connectionStrings.Value.ConnectionStrings)
{
    Console.WriteLine($"{cs.Description}: {cs.ConnectionString}");
}
```

## Tham chiếu Các Loại Chính

| Loại                                        | Mục đích                              |
| ------------------------------------------- | ------------------------------------- |
| `ArmClient`                                 | Điểm vào cho tất cả các hoạt động ARM |
| `CosmosDBAccountResource`                   | Đại diện cho một tài khoản Cosmos DB  |
| `CosmosDBAccountCollection`                 | Collection cho account CRUD           |
| `CosmosDBSqlDatabaseResource`               | SQL API database                      |
| `CosmosDBSqlContainerResource`              | SQL API container                     |
| `CosmosDBAccountCreateOrUpdateContent`      | Payload tạo tài khoản                 |
| `CosmosDBSqlDatabaseCreateOrUpdateContent`  | Payload tạo database                  |
| `CosmosDBSqlContainerCreateOrUpdateContent` | Payload tạo container                 |
| `ThroughputSettingsUpdateData`              | Cấu hình thông lượng                  |

## Thực hành Tốt nhất

1. **Sử dụng `WaitUntil.Completed`** cho các hoạt động phải hoàn thành trước khi tiếp tục
2. **Sử dụng `WaitUntil.Started`** khi bạn muốn thăm dò thủ công hoặc chạy các hoạt động song song
3. **Luôn sử dụng `DefaultAzureCredential`** — không bao giờ hardcode keys
4. **Xử lý `RequestFailedException`** cho các lỗi ARM API
5. **Sử dụng `CreateOrUpdateAsync`** cho các hoạt động idempotent
6. **Điều hướng phân cấp** qua các phương thức `Get*` (ví dụ: `account.GetCosmosDBSqlDatabases()`)

## Xử lý Lỗi

```csharp
using Azure;

try
{
    var operation = await accountCollection.CreateOrUpdateAsync(
        WaitUntil.Completed, accountName, accountData);
}
catch (RequestFailedException ex) when (ex.Status == 409)
{
    Console.WriteLine("Account already exists");
}
catch (RequestFailedException ex)
{
    Console.WriteLine($"ARM Error: {ex.Status} - {ex.ErrorCode}: {ex.Message}");
}
```

## Các Tệp Tham khảo

| Tệp                                                                  | Khi nào đọc                                                             |
| -------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| [references/account-management.md](references/account-management.md) | Account CRUD, chuyển đổi dự phòng (failover), keys, chuỗi kết nối, mạng |
| [references/sql-resources.md](references/sql-resources.md)           | SQL databases, containers, stored procedures, triggers, UDFs            |
| [references/throughput.md](references/throughput.md)                 | Thông lượng thủ công/autoscale, di chuyển giữa các chế độ               |

## Các SDK Liên quan

| SDK                              | Mục đích                            | Cài đặt                                             |
| -------------------------------- | ----------------------------------- | --------------------------------------------------- |
| `Microsoft.Azure.Cosmos`         | Data plane (document CRUD, queries) | `dotnet add package Microsoft.Azure.Cosmos`         |
| `Azure.ResourceManager.CosmosDB` | Management plane (SDK này)          | `dotnet add package Azure.ResourceManager.CosmosDB` |
