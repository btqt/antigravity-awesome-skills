---
name: azure-resource-manager-postgresql-dotnet
description: |
  Azure PostgreSQL Flexible Server SDK cho .NET. Quản lý cơ sở dữ liệu cho các triển khai PostgreSQL Flexible Server. Sử dụng để tạo server, cơ sở dữ liệu, quy tắc tường lửa, cấu hình, sao lưu và tính sẵn sàng cao. Triggers: "PostgreSQL", "PostgreSqlFlexibleServer", "PostgreSQL Flexible Server", "Azure Database for PostgreSQL", "PostgreSQL database management", "PostgreSQL firewall", "PostgreSQL backup", "Postgres".
package: Azure.ResourceManager.PostgreSql
---

# Azure.ResourceManager.PostgreSql (.NET)

Azure Resource Manager SDK để quản lý các triển khai PostgreSQL Flexible Server.

## Cài đặt

```bash
dotnet add package Azure.ResourceManager.PostgreSql
dotnet add package Azure.Identity
```

**Phiên bản Hiện tại**: v1.2.0 (GA)  
**Phiên bản API**: 2023-12-01-preview

> **Lưu ý**: Skill này tập trung vào PostgreSQL Flexible Server. Single Server đã lỗi thời và được lên kế hoạch ngừng hoạt động.

## Biến Môi trường

```bash
AZURE_SUBSCRIPTION_ID=<your-subscription-id>
AZURE_RESOURCE_GROUP=<your-resource-group>
AZURE_POSTGRESQL_SERVER_NAME=<your-postgresql-server>
```

## Xác thực

```csharp
using Azure.Identity;
using Azure.ResourceManager;
using Azure.ResourceManager.PostgreSql;
using Azure.ResourceManager.PostgreSql.FlexibleServers;

ArmClient client = new ArmClient(new DefaultAzureCredential());
```

## Phân cấp Tài nguyên

```
Subscription
└── ResourceGroup
    └── PostgreSqlFlexibleServer              # Phiên bản PostgreSQL Flexible Server
        ├── PostgreSqlFlexibleServerDatabase  # Cơ sở dữ liệu bên trong server
        ├── PostgreSqlFlexibleServerFirewallRule # Quy tắc tường lửa IP
        ├── PostgreSqlFlexibleServerConfiguration # Các tham số Server
        ├── PostgreSqlFlexibleServerBackup    # Thông tin sao lưu
        ├── PostgreSqlFlexibleServerActiveDirectoryAdministrator # Quản trị viên Entra ID
        └── PostgreSqlFlexibleServerVirtualEndpoint # Các endpoint bản sao đọc (Read replica)
```

## Quy trình Công việc Cốt lõi

### 1. Tạo PostgreSQL Flexible Server

```csharp
using Azure.ResourceManager.PostgreSql.FlexibleServers;
using Azure.ResourceManager.PostgreSql.FlexibleServers.Models;

ResourceGroupResource resourceGroup = await client
    .GetDefaultSubscriptionAsync()
    .Result
    .GetResourceGroupAsync("my-resource-group");

PostgreSqlFlexibleServerCollection servers = resourceGroup.GetPostgreSqlFlexibleServers();

PostgreSqlFlexibleServerData data = new PostgreSqlFlexibleServerData(AzureLocation.EastUS)
{
    Sku = new PostgreSqlFlexibleServerSku("Standard_D2ds_v4", PostgreSqlFlexibleServerSkuTier.GeneralPurpose),
    AdministratorLogin = "pgadmin",
    AdministratorLoginPassword = "YourSecurePassword123!",
    Version = PostgreSqlFlexibleServerVersion.Ver16,
    Storage = new PostgreSqlFlexibleServerStorage
    {
        StorageSizeInGB = 128,
        AutoGrow = StorageAutoGrow.Enabled,
        Tier = PostgreSqlStorageTierName.P30
    },
    Backup = new PostgreSqlFlexibleServerBackupProperties
    {
        BackupRetentionDays = 7,
        GeoRedundantBackup = PostgreSqlFlexibleServerGeoRedundantBackupEnum.Disabled
    },
    HighAvailability = new PostgreSqlFlexibleServerHighAvailability
    {
        Mode = PostgreSqlFlexibleServerHighAvailabilityMode.ZoneRedundant,
        StandbyAvailabilityZone = "2"
    },
    AvailabilityZone = "1",
    AuthConfig = new PostgreSqlFlexibleServerAuthConfig
    {
        ActiveDirectoryAuth = PostgreSqlFlexibleServerActiveDirectoryAuthEnum.Enabled,
        PasswordAuth = PostgreSqlFlexibleServerPasswordAuthEnum.Enabled
    }
};

ArmOperation<PostgreSqlFlexibleServerResource> operation = await servers
    .CreateOrUpdateAsync(WaitUntil.Completed, "my-postgresql-server", data);

PostgreSqlFlexibleServerResource server = operation.Value;
Console.WriteLine($"Server created: {server.Data.FullyQualifiedDomainName}");
```

### 2. Tạo Database

```csharp
PostgreSqlFlexibleServerResource server = await resourceGroup
    .GetPostgreSqlFlexibleServerAsync("my-postgresql-server");

PostgreSqlFlexibleServerDatabaseCollection databases = server.GetPostgreSqlFlexibleServerDatabases();

PostgreSqlFlexibleServerDatabaseData dbData = new PostgreSqlFlexibleServerDatabaseData
{
    Charset = "UTF8",
    Collation = "en_US.utf8"
};

ArmOperation<PostgreSqlFlexibleServerDatabaseResource> operation = await databases
    .CreateOrUpdateAsync(WaitUntil.Completed, "myappdb", dbData);

PostgreSqlFlexibleServerDatabaseResource database = operation.Value;
Console.WriteLine($"Database created: {database.Data.Name}");
```

### 3. Cấu hình Quy tắc Tường lửa

```csharp
PostgreSqlFlexibleServerFirewallRuleCollection firewallRules = server.GetPostgreSqlFlexibleServerFirewallRules();

// Cho phép dải IP cụ thể
PostgreSqlFlexibleServerFirewallRuleData ruleData = new PostgreSqlFlexibleServerFirewallRuleData
{
    StartIPAddress = System.Net.IPAddress.Parse("10.0.0.1"),
    EndIPAddress = System.Net.IPAddress.Parse("10.0.0.255")
};

ArmOperation<PostgreSqlFlexibleServerFirewallRuleResource> operation = await firewallRules
    .CreateOrUpdateAsync(WaitUntil.Completed, "allow-internal", ruleData);

// Cho phép các dịch vụ Azure
PostgreSqlFlexibleServerFirewallRuleData azureServicesRule = new PostgreSqlFlexibleServerFirewallRuleData
{
    StartIPAddress = System.Net.IPAddress.Parse("0.0.0.0"),
    EndIPAddress = System.Net.IPAddress.Parse("0.0.0.0")
};

await firewallRules.CreateOrUpdateAsync(WaitUntil.Completed, "AllowAllAzureServicesAndResourcesWithinAzureIps", azureServicesRule);
```

### 4. Cập nhật Cấu hình Server

```csharp
PostgreSqlFlexibleServerConfigurationCollection configurations = server.GetPostgreSqlFlexibleServerConfigurations();

// Lấy cấu hình hiện tại
PostgreSqlFlexibleServerConfigurationResource config = await configurations
    .GetAsync("max_connections");

// Cập nhật cấu hình
PostgreSqlFlexibleServerConfigurationData configData = new PostgreSqlFlexibleServerConfigurationData
{
    Value = "500",
    Source = "user-override"
};

ArmOperation<PostgreSqlFlexibleServerConfigurationResource> operation = await configurations
    .CreateOrUpdateAsync(WaitUntil.Completed, "max_connections", configData);

// Các cấu hình PostgreSQL phổ biến cần điều chỉnh
string[] commonParams = {
    "max_connections",
    "shared_buffers",
    "work_mem",
    "maintenance_work_mem",
    "effective_cache_size",
    "log_min_duration_statement"
};
```

### 5. Cấu hình Quản trị viên Entra ID

```csharp
PostgreSqlFlexibleServerActiveDirectoryAdministratorCollection admins =
    server.GetPostgreSqlFlexibleServerActiveDirectoryAdministrators();

PostgreSqlFlexibleServerActiveDirectoryAdministratorData adminData =
    new PostgreSqlFlexibleServerActiveDirectoryAdministratorData
{
    PrincipalType = PostgreSqlFlexibleServerPrincipalType.User,
    PrincipalName = "aad-admin@contoso.com",
    TenantId = Guid.Parse("<tenant-id>")
};

ArmOperation<PostgreSqlFlexibleServerActiveDirectoryAdministratorResource> operation = await admins
    .CreateOrUpdateAsync(WaitUntil.Completed, "<entra-object-id>", adminData);
```

### 6. Liệt kê và Quản lý Servers

```csharp
// Liệt kê các servers trong resource group
await foreach (PostgreSqlFlexibleServerResource server in resourceGroup.GetPostgreSqlFlexibleServers())
{
    Console.WriteLine($"Server: {server.Data.Name}");
    Console.WriteLine($"  FQDN: {server.Data.FullyQualifiedDomainName}");
    Console.WriteLine($"  Version: {server.Data.Version}");
    Console.WriteLine($"  State: {server.Data.State}");
    Console.WriteLine($"  SKU: {server.Data.Sku.Name} ({server.Data.Sku.Tier})");
    Console.WriteLine($"  HA: {server.Data.HighAvailability?.Mode}");
}

// Liệt kê các databases trong server
await foreach (PostgreSqlFlexibleServerDatabaseResource db in server.GetPostgreSqlFlexibleServerDatabases())
{
    Console.WriteLine($"Database: {db.Data.Name}");
}
```

### 7. Sao lưu và Khôi phục Tại một thời điểm (PITR)

```csharp
// Liệt kê các bản sao lưu có sẵn
await foreach (PostgreSqlFlexibleServerBackupResource backup in server.GetPostgreSqlFlexibleServerBackups())
{
    Console.WriteLine($"Backup: {backup.Data.Name}");
    Console.WriteLine($"  Type: {backup.Data.BackupType}");
    Console.WriteLine($"  Completed: {backup.Data.CompletedOn}");
}

// Khôi phục tại một thời điểm
PostgreSqlFlexibleServerData restoreData = new PostgreSqlFlexibleServerData(AzureLocation.EastUS)
{
    CreateMode = PostgreSqlFlexibleServerCreateMode.PointInTimeRestore,
    SourceServerResourceId = server.Id,
    PointInTimeUtc = DateTimeOffset.UtcNow.AddHours(-2)
};

ArmOperation<PostgreSqlFlexibleServerResource> operation = await servers
    .CreateOrUpdateAsync(WaitUntil.Completed, "my-postgresql-restored", restoreData);
```

### 8. Tạo Read Replica

```csharp
PostgreSqlFlexibleServerData replicaData = new PostgreSqlFlexibleServerData(AzureLocation.WestUS)
{
    CreateMode = PostgreSqlFlexibleServerCreateMode.Replica,
    SourceServerResourceId = server.Id,
    Sku = new PostgreSqlFlexibleServerSku("Standard_D2ds_v4", PostgreSqlFlexibleServerSkuTier.GeneralPurpose)
};

ArmOperation<PostgreSqlFlexibleServerResource> operation = await servers
    .CreateOrUpdateAsync(WaitUntil.Completed, "my-postgresql-replica", replicaData);
```

### 9. Dừng và Bắt đầu Server

```csharp
PostgreSqlFlexibleServerResource server = await resourceGroup
    .GetPostgreSqlFlexibleServerAsync("my-postgresql-server");

// Dừng server (tiết kiệm chi phí khi không sử dụng)
await server.StopAsync(WaitUntil.Completed);

// Bắt đầu server
await server.StartAsync(WaitUntil.Completed);

// Khởi động lại server
await server.RestartAsync(WaitUntil.Completed, new PostgreSqlFlexibleServerRestartParameter
{
    RestartWithFailover = true,
    FailoverMode = PostgreSqlFlexibleServerFailoverMode.PlannedFailover
});
```

### 10. Cập nhật Server (Mở rộng quy mô)

```csharp
PostgreSqlFlexibleServerResource server = await resourceGroup
    .GetPostgreSqlFlexibleServerAsync("my-postgresql-server");

PostgreSqlFlexibleServerPatch patch = new PostgreSqlFlexibleServerPatch
{
    Sku = new PostgreSqlFlexibleServerSku("Standard_D4ds_v4", PostgreSqlFlexibleServerSkuTier.GeneralPurpose),
    Storage = new PostgreSqlFlexibleServerStorage
    {
        StorageSizeInGB = 256,
        Tier = PostgreSqlStorageTierName.P40
    }
};

ArmOperation<PostgreSqlFlexibleServerResource> operation = await server
    .UpdateAsync(WaitUntil.Completed, patch);
```

### 11. Xóa Server

```csharp
PostgreSqlFlexibleServerResource server = await resourceGroup
    .GetPostgreSqlFlexibleServerAsync("my-postgresql-server");

await server.DeleteAsync(WaitUntil.Completed);
```

## Tham chiếu Các Loại Chính

| Loại                                                           | Mục đích                   |
| -------------------------------------------------------------- | -------------------------- |
| `PostgreSqlFlexibleServerResource`                             | Phiên bản Flexible Server  |
| `PostgreSqlFlexibleServerData`                                 | Dữ liệu cấu hình server    |
| `PostgreSqlFlexibleServerCollection`                           | Collection của các servers |
| `PostgreSqlFlexibleServerDatabaseResource`                     | Database bên trong server  |
| `PostgreSqlFlexibleServerFirewallRuleResource`                 | Quy tắc tường lửa IP       |
| `PostgreSqlFlexibleServerConfigurationResource`                | Tham số server             |
| `PostgreSqlFlexibleServerBackupResource`                       | Metadata sao lưu           |
| `PostgreSqlFlexibleServerActiveDirectoryAdministratorResource` | Quản trị viên Entra ID     |
| `PostgreSqlFlexibleServerSku`                                  | SKU (compute tier + size)  |
| `PostgreSqlFlexibleServerStorage`                              | Cấu hình lưu trữ           |
| `PostgreSqlFlexibleServerHighAvailability`                     | Cấu hình HA                |
| `PostgreSqlFlexibleServerBackupProperties`                     | Cài đặt sao lưu            |
| `PostgreSqlFlexibleServerAuthConfig`                           | Cài đặt xác thực           |

## Các Tầng SKU

| Tầng              | Trường hợp Sử dụng                 | Ví dụ SKU                          |
| ----------------- | ---------------------------------- | ---------------------------------- |
| `Burstable`       | Dev/test, khối lượng công việc nhẹ | Standard_B1ms, Standard_B2s        |
| `GeneralPurpose`  | Khối lượng công việc production    | Standard_D2ds_v4, Standard_D4ds_v4 |
| `MemoryOptimized` | Yêu cầu bộ nhớ cao                 | Standard_E2ds_v4, Standard_E4ds_v4 |

## Phiên bản PostgreSQL

| Phiên bản     | Giá trị Enum |
| ------------- | ------------ |
| PostgreSQL 11 | `Ver11`      |
| PostgreSQL 12 | `Ver12`      |
| PostgreSQL 13 | `Ver13`      |
| PostgreSQL 14 | `Ver14`      |
| PostgreSQL 15 | `Ver15`      |
| PostgreSQL 16 | `Ver16`      |

## Các Chế độ Tính sẵn sàng Cao (HA)

| Chế độ          | Mô tả                                    |
| --------------- | ---------------------------------------- |
| `Disabled`      | Không có HA (single server)              |
| `SameZone`      | HA trong cùng một availability zone      |
| `ZoneRedundant` | HA trên các availability zones khác nhau |

## Thực hành Tốt nhất

1. **Sử dụng Flexible Server** — Single Server đã lỗi thời
2. **Bật zone-redundant HA** — Cho khối lượng công việc production
3. **Sử dụng DefaultAzureCredential** — Ưu tiên hơn connection strings
4. **Cấu hình xác thực Entra ID** — An toàn hơn xác thực SQL thuần túy
5. **Bật cả hai phương thức xác thực** — Entra ID + mật khẩu để linh hoạt
6. **Thiết lập thời gian lưu giữ sao lưu phù hợp** — 7-35 ngày dựa trên tuân thủ
7. **Sử dụng private endpoints** — Để truy cập mạng an toàn
8. **Điều chỉnh các tham số server** — Dựa trên đặc điểm khối lượng công việc
9. **Sử dụng read replicas** — Cho khối lượng công việc đọc nhiều
10. **Dừng các máy chủ dev/test** — Tiết kiệm chi phí khi không sử dụng

## Xử lý Lỗi

```csharp
using Azure;

try
{
    ArmOperation<PostgreSqlFlexibleServerResource> operation = await servers
        .CreateOrUpdateAsync(WaitUntil.Completed, "my-postgresql", data);
}
catch (RequestFailedException ex) when (ex.Status == 409)
{
    Console.WriteLine("Server already exists");
}
catch (RequestFailedException ex) when (ex.Status == 400)
{
    Console.WriteLine($"Invalid configuration: {ex.Message}");
}
catch (RequestFailedException ex)
{
    Console.WriteLine($"Azure error: {ex.Status} - {ex.Message}");
}
```

## Chuỗi Kết nối

Sau khi tạo server, kết nối bằng cách sử dụng:

```csharp
// Chuỗi kết nối Npgsql
string connectionString = $"Host={server.Data.FullyQualifiedDomainName};" +
    "Database=myappdb;" +
    "Username=pgadmin;" +
    "Password=YourSecurePassword123!;" +
    "SSL Mode=Require;Trust Server Certificate=true;";

// Với Entra ID token (khuyên dùng)
var credential = new DefaultAzureCredential();
var token = await credential.GetTokenAsync(
    new TokenRequestContext(new[] { "https://ossrdbms-aad.database.windows.net/.default" }));

string connectionString = $"Host={server.Data.FullyQualifiedDomainName};" +
    "Database=myappdb;" +
    $"Username=aad-admin@contoso.com;" +
    $"Password={token.Token};" +
    "SSL Mode=Require;";
```

## Các SDK Liên quan

| SDK                                     | Mục đích                     | Cài đặt                                                    |
| --------------------------------------- | ---------------------------- | ---------------------------------------------------------- |
| `Azure.ResourceManager.PostgreSql`      | Quản lý PostgreSQL (SDK này) | `dotnet add package Azure.ResourceManager.PostgreSql`      |
| `Azure.ResourceManager.MySql`           | Quản lý MySQL                | `dotnet add package Azure.ResourceManager.MySql`           |
| `Npgsql`                                | Truy cập dữ liệu PostgreSQL  | `dotnet add package Npgsql`                                |
| `Npgsql.EntityFrameworkCore.PostgreSQL` | Provider cho EF Core         | `dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL` |

## Liên kết Tham khảo

| Tài nguyên            | URL                                                                                                  |
| --------------------- | ---------------------------------------------------------------------------------------------------- |
| NuGet Package         | https://www.nuget.org/packages/Azure.ResourceManager.PostgreSql                                      |
| API Reference         | https://learn.microsoft.com/dotnet/api/azure.resourcemanager.postgresql                              |
| Product Documentation | https://learn.microsoft.com/azure/postgresql/flexible-server/                                        |
| GitHub Source         | https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/postgresql/Azure.ResourceManager.PostgreSql |
