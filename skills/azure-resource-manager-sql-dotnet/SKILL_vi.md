---
name: azure-resource-manager-sql-dotnet
description: |
  Azure Resource Manager SDK cho Azure SQL trong .NET. Sử dụng cho các hoạt động MANAGEMENT PLANE: tạo/quản lý SQL servers, databases, elastic pools, quy tắc tường lửa, và các nhóm chuyển đổi dự phòng (failover groups) thông qua Azure Resource Manager. KHÔNG dùng cho các hoạt động data plane (thực thi truy vấn) - hãy sử dụng Microsoft.Data.SqlClient cho việc đó. Triggers: "SQL server", "create SQL database", "manage SQL resources", "ARM SQL", "SqlServerResource", "provision Azure SQL", "elastic pool", "firewall rule".
package: Azure.ResourceManager.Sql
---

# Azure.ResourceManager.Sql (.NET)

SDK management plane để cung cấp và quản lý tài nguyên Azure SQL thông qua Azure Resource Manager.

> **⚠️ Management vs Data Plane**
>
> - **SDK này (Azure.ResourceManager.Sql)**: Tạo servers, databases, elastic pools, cấu hình quy tắc tường lửa, quản lý failover groups
> - **Data Plane SDK (Microsoft.Data.SqlClient)**: Thực thi truy vấn, stored procedures, quản lý kết nối

## Cài đặt

```bash
dotnet add package Azure.ResourceManager.Sql
dotnet add package Azure.Identity
```

**Phiên bản Hiện tại**: Stable v1.3.0, Preview v1.4.0-beta.3

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
using Azure.ResourceManager.Sql;

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
        └── SqlServerResource
            ├── SqlDatabaseResource
            ├── ElasticPoolResource
            │   └── ElasticPoolDatabaseResource
            ├── SqlFirewallRuleResource
            ├── FailoverGroupResource
            ├── ServerBlobAuditingPolicyResource
            ├── EncryptionProtectorResource
            └── VirtualNetworkRuleResource
```

## Quy trình Công việc Cốt lõi

### 1. Tạo SQL Server

```csharp
using Azure.ResourceManager.Sql;
using Azure.ResourceManager.Sql.Models;

// Lấy resource group
var resourceGroup = await subscription
    .GetResourceGroupAsync("my-resource-group");

// Định nghĩa server
var serverData = new SqlServerData(AzureLocation.EastUS)
{
    AdministratorLogin = "sqladmin",
    AdministratorLoginPassword = "YourSecurePassword123!",
    Version = "12.0",
    MinimalTlsVersion = SqlMinimalTlsVersion.Tls1_2,
    PublicNetworkAccess = ServerNetworkAccessFlag.Enabled
};

// Tạo server (long-running operation)
var serverCollection = resourceGroup.Value.GetSqlServers();
var operation = await serverCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "my-sql-server",
    serverData);

SqlServerResource server = operation.Value;
```

### 2. Tạo SQL Database

```csharp
var databaseData = new SqlDatabaseData(AzureLocation.EastUS)
{
    Sku = new SqlSku("S0") { Tier = "Standard" },
    MaxSizeBytes = 2L * 1024 * 1024 * 1024, // 2 GB
    Collation = "SQL_Latin1_General_CP1_CI_AS",
    RequestedBackupStorageRedundancy = SqlBackupStorageRedundancy.Local
};

var databaseCollection = server.GetSqlDatabases();
var dbOperation = await databaseCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "my-database",
    databaseData);

SqlDatabaseResource database = dbOperation.Value;
```

### 3. Tạo Elastic Pool

```csharp
var poolData = new ElasticPoolData(AzureLocation.EastUS)
{
    Sku = new SqlSku("StandardPool")
    {
        Tier = "Standard",
        Capacity = 100 // 100 eDTUs
    },
    PerDatabaseSettings = new ElasticPoolPerDatabaseSettings
    {
        MinCapacity = 0,
        MaxCapacity = 100
    }
};

var poolCollection = server.GetElasticPools();
var poolOperation = await poolCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "my-elastic-pool",
    poolData);

ElasticPoolResource pool = poolOperation.Value;
```

### 4. Thêm Database vào Elastic Pool

```csharp
var databaseData = new SqlDatabaseData(AzureLocation.EastUS)
{
    ElasticPoolId = pool.Id
};

await databaseCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "pooled-database",
    databaseData);
```

### 5. Cấu hình Quy tắc Tường lửa

```csharp
// Cho phép các dịch vụ Azure
var azureServicesRule = new SqlFirewallRuleData
{
    StartIPAddress = "0.0.0.0",
    EndIPAddress = "0.0.0.0"
};

var firewallCollection = server.GetSqlFirewallRules();
await firewallCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "AllowAzureServices",
    azureServicesRule);

// Cho phép dải IP cụ thể
var clientRule = new SqlFirewallRuleData
{
    StartIPAddress = "203.0.113.0",
    EndIPAddress = "203.0.113.255"
};

await firewallCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "AllowClientIPs",
    clientRule);
```

### 6. Liệt kê Tài nguyên

```csharp
// Liệt kê tất cả servers trong subscription
await foreach (var srv in subscription.GetSqlServersAsync())
{
    Console.WriteLine($"Server: {srv.Data.Name} in {srv.Data.Location}");
}

// Liệt kê databases trong một server
await foreach (var db in server.GetSqlDatabases())
{
    Console.WriteLine($"Database: {db.Data.Name}, SKU: {db.Data.Sku?.Name}");
}

// Liệt kê elastic pools
await foreach (var ep in server.GetElasticPools())
{
    Console.WriteLine($"Pool: {ep.Data.Name}, DTU: {ep.Data.Sku?.Capacity}");
}
```

### 7. Lấy Chuỗi Kết nối

```csharp
// Xây dựng chuỗi kết nối (server FQDN có thể đoán được)
var serverFqdn = $"{server.Data.Name}.database.windows.net";
var connectionString = $"Server=tcp:{serverFqdn},1433;" +
    $"Initial Catalog={database.Data.Name};" +
    "Persist Security Info=False;" +
    $"User ID={server.Data.AdministratorLogin};" +
    "Password=<your-password>;" +
    "MultipleActiveResultSets=False;" +
    "Encrypt=True;" +
    "TrustServerCertificate=False;" +
    "Connection Timeout=30;";
```

## Tham chiếu Các Loại Chính

| Loại                        | Mục đích                              |
| --------------------------- | ------------------------------------- |
| `ArmClient`                 | Điểm vào cho tất cả các hoạt động ARM |
| `SqlServerResource`         | Đại diện cho một Azure SQL server     |
| `SqlServerCollection`       | Collection cho server CRUD            |
| `SqlDatabaseResource`       | Đại diện cho một SQL database         |
| `SqlDatabaseCollection`     | Collection cho database CRUD          |
| `ElasticPoolResource`       | Đại diện cho một elastic pool         |
| `ElasticPoolCollection`     | Collection cho elastic pool CRUD      |
| `SqlFirewallRuleResource`   | Đại diện cho một quy tắc tường lửa    |
| `SqlFirewallRuleCollection` | Collection cho firewall rule CRUD     |
| `SqlServerData`             | Payload tạo/cập nhật server           |
| `SqlDatabaseData`           | Payload tạo/cập nhật database         |
| `ElasticPoolData`           | Payload tạo/cập nhật elastic pool     |
| `SqlFirewallRuleData`       | Payload tạo/cập nhật firewall rule    |
| `SqlSku`                    | Cấu hình SKU (tier, capacity)         |

## Các SKU Phổ biến

### Database SKUs

| Tên SKU     | Tier             | Mô tả                 |
| ----------- | ---------------- | --------------------- |
| `Basic`     | Basic            | 5 DTUs, tối đa 2 GB   |
| `S0`-`S12`  | Standard         | 10-3000 DTUs          |
| `P1`-`P15`  | Premium          | 125-4000 DTUs         |
| `GP_Gen5_2` | GeneralPurpose   | vCore-based, 2 vCores |
| `BC_Gen5_2` | BusinessCritical | vCore-based, 2 vCores |
| `HS_Gen5_2` | Hyperscale       | vCore-based, 2 vCores |

### Elastic Pool SKUs

| Tên SKU        | Tier             | Mô tả          |
| -------------- | ---------------- | -------------- |
| `BasicPool`    | Basic            | 50-1600 eDTUs  |
| `StandardPool` | Standard         | 50-3000 eDTUs  |
| `PremiumPool`  | Premium          | 125-4000 eDTUs |
| `GP_Gen5_2`    | GeneralPurpose   | vCore-based    |
| `BC_Gen5_2`    | BusinessCritical | vCore-based    |

## Thực hành Tốt nhất

1. **Sử dụng `WaitUntil.Completed`** cho các hoạt động phải hoàn thành trước khi tiếp tục
2. **Sử dụng `WaitUntil.Started`** khi bạn muốn thăm dò thủ công hoặc chạy các hoạt động song song
3. **Luôn sử dụng `DefaultAzureCredential`** — không bao giờ hardcode mật khẩu trong production
4. **Xử lý `RequestFailedException`** cho các lỗi ARM API
5. **Sử dụng `CreateOrUpdateAsync`** cho các hoạt động idempotent
6. **Điều hướng phân cấp** qua các phương thức `Get*` (ví dụ: `server.GetSqlDatabases()`)
7. **Sử dụng elastic pools** để tối ưu hóa chi phí khi quản lý nhiều cơ sở dữ liệu
8. **Cấu hình quy tắc tường lửa** trước khi cố gắng kết nối

## Xử lý Lỗi

```csharp
using Azure;

try
{
    var operation = await serverCollection.CreateOrUpdateAsync(
        WaitUntil.Completed, serverName, serverData);
}
catch (RequestFailedException ex) when (ex.Status == 409)
{
    Console.WriteLine("Server already exists");
}
catch (RequestFailedException ex) when (ex.Status == 400)
{
    Console.WriteLine($"Invalid request: {ex.Message}");
}
catch (RequestFailedException ex)
{
    Console.WriteLine($"ARM Error: {ex.Status} - {ex.ErrorCode}: {ex.Message}");
}
```

## Các Tệp Tham khảo

| Tệp                                                                    | Khi nào đọc                                                |
| ---------------------------------------------------------------------- | ---------------------------------------------------------- |
| [references/server-management.md](references/server-management.md)     | Server CRUD, thông tin xác thực admin, Azure AD auth, mạng |
| [references/database-operations.md](references/database-operations.md) | Database CRUD, scaling, sao lưu, khôi phục, sao chép       |
| [references/elastic-pools.md](references/elastic-pools.md)             | Quản lý Pool, thêm/xóa databases, scaling                  |

## Các SDK Liên quan

| SDK                                       | Mục đích                                        | Cài đặt                                                      |
| ----------------------------------------- | ----------------------------------------------- | ------------------------------------------------------------ |
| `Microsoft.Data.SqlClient`                | Data plane (thực thi truy vấn, thủ tục lưu trữ) | `dotnet add package Microsoft.Data.SqlClient`                |
| `Azure.ResourceManager.Sql`               | Management plane (SDK này)                      | `dotnet add package Azure.ResourceManager.Sql`               |
| `Microsoft.EntityFrameworkCore.SqlServer` | ORM cho SQL Server                              | `dotnet add package Microsoft.EntityFrameworkCore.SqlServer` |
