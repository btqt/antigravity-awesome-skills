---
name: azure-mgmt-applicationinsights-dotnet
description: |
  Azure Application Insights SDK cho .NET. Quản lý tài nguyên giám sát hiệu suất ứng dụng và khả năng quan sát (observability). Sử dụng để tạo các thành phần Application Insights, kiểm tra web (web tests), workbooks, analytics items, và API keys. Triggers: "Application Insights", "ApplicationInsights", "App Insights", "APM", "application monitoring", "web tests", "availability tests", "workbooks".
package: Azure.ResourceManager.ApplicationInsights
---

# Azure.ResourceManager.ApplicationInsights (.NET)

SDK Azure Resource Manager để quản lý tài nguyên Application Insights cho giám sát hiệu suất ứng dụng.

## Cài đặt

```bash
dotnet add package Azure.ResourceManager.ApplicationInsights
dotnet add package Azure.Identity
```

**Phiên bản Hiện tại**: v1.0.0 (GA)  
**Phiên bản API**: 2022-06-15

## Biến Môi trường

```bash
AZURE_SUBSCRIPTION_ID=<your-subscription-id>
AZURE_RESOURCE_GROUP=<your-resource-group>
AZURE_APPINSIGHTS_NAME=<your-appinsights-component>
```

## Xác thực

```csharp
using Azure.Identity;
using Azure.ResourceManager;
using Azure.ResourceManager.ApplicationInsights;

ArmClient client = new ArmClient(new DefaultAzureCredential());
```

## Phân cấp Tài nguyên

```
Subscription
└── ResourceGroup
    └── ApplicationInsightsComponent          # Tài nguyên App Insights
        ├── ApplicationInsightsComponentApiKey  # API keys cho truy cập lập trình
        ├── ComponentLinkedStorageAccount      # Linked storage cho xuất dữ liệu
        └── (thông qua component ID)
            ├── WebTest                        # Kiểm tra tính sẵn sàng (Availability tests)
            ├── Workbook                       # Workbooks để phân tích
            ├── WorkbookTemplate               # Workbook templates
            └── MyWorkbook                     # Private workbooks
```

## Các Quy trình Làm việc Cốt lõi

### 1. Tạo Thành phần Application Insights (Workspace-based)

```csharp
using Azure.ResourceManager.ApplicationInsights;
using Azure.ResourceManager.ApplicationInsights.Models;

ResourceGroupResource resourceGroup = await client
    .GetDefaultSubscriptionAsync()
    .Result
    .GetResourceGroupAsync("my-resource-group");

ApplicationInsightsComponentCollection components = resourceGroup.GetApplicationInsightsComponents();

// Workspace-based Application Insights (được khuyến nghị)
ApplicationInsightsComponentData data = new ApplicationInsightsComponentData(
    AzureLocation.EastUS,
    ApplicationInsightsApplicationType.Web)
{
    Kind = "web",
    WorkspaceResourceId = new ResourceIdentifier(
        "/subscriptions/<sub-id>/resourceGroups/<rg>/providers/Microsoft.OperationalInsights/workspaces/<workspace-name>"),
    IngestionMode = IngestionMode.LogAnalytics,
    PublicNetworkAccessForIngestion = PublicNetworkAccessType.Enabled,
    PublicNetworkAccessForQuery = PublicNetworkAccessType.Enabled,
    RetentionInDays = 90,
    SamplingPercentage = 100,
    DisableIPMasking = false,
    ImmediatePurgeDataOn30Days = false,
    Tags =
    {
        { "environment", "production" },
        { "application", "mywebapp" }
    }
};

ArmOperation<ApplicationInsightsComponentResource> operation = await components
    .CreateOrUpdateAsync(WaitUntil.Completed, "my-appinsights", data);

ApplicationInsightsComponentResource component = operation.Value;

Console.WriteLine($"Component created: {component.Data.Name}");
Console.WriteLine($"Instrumentation Key: {component.Data.InstrumentationKey}");
Console.WriteLine($"Connection String: {component.Data.ConnectionString}");
```

### 2. Lấy Connection String và Keys

```csharp
ApplicationInsightsComponentResource component = await resourceGroup
    .GetApplicationInsightsComponentAsync("my-appinsights");

// Lấy connection string để cấu hình SDK
string connectionString = component.Data.ConnectionString;
string instrumentationKey = component.Data.InstrumentationKey;
string appId = component.Data.AppId;

Console.WriteLine($"Connection String: {connectionString}");
Console.WriteLine($"Instrumentation Key: {instrumentationKey}");
Console.WriteLine($"App ID: {appId}");
```

### 3. Tạo API Key

```csharp
ApplicationInsightsComponentResource component = await resourceGroup
    .GetApplicationInsightsComponentAsync("my-appinsights");

ApplicationInsightsComponentApiKeyCollection apiKeys = component.GetApplicationInsightsComponentApiKeys();

// API key để đọc telemetry
ApplicationInsightsApiKeyContent keyContent = new ApplicationInsightsApiKeyContent
{
    Name = "ReadTelemetryKey",
    LinkedReadProperties =
    {
        $"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/microsoft.insights/components/{component.Data.Name}/api",
        $"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/microsoft.insights/components/{component.Data.Name}/agentconfig"
    }
};

ApplicationInsightsComponentApiKeyResource apiKey = await apiKeys
    .CreateOrUpdateAsync(WaitUntil.Completed, keyContent);

Console.WriteLine($"API Key Name: {apiKey.Data.Name}");
Console.WriteLine($"API Key: {apiKey.Data.ApiKey}"); // Chỉ hiển thị một lần!
```

### 4. Tạo Web Test (Kiểm tra Tính sẵn sàng)

```csharp
WebTestCollection webTests = resourceGroup.GetWebTests();

// URL Ping Test
WebTestData urlPingTest = new WebTestData(AzureLocation.EastUS)
{
    Kind = WebTestKind.Ping,
    SyntheticMonitorId = "webtest-ping-myapp",
    WebTestName = "Homepage Availability",
    Description = "Checks if homepage is available",
    IsEnabled = true,
    Frequency = 300, // 5 phút
    Timeout = 120,   // 2 phút
    WebTestKind = WebTestKind.Ping,
    IsRetryEnabled = true,
    Locations =
    {
        new WebTestGeolocation { WebTestLocationId = "us-ca-sjc-azr" },  // West US
        new WebTestGeolocation { WebTestLocationId = "us-tx-sn1-azr" },  // South Central US
        new WebTestGeolocation { WebTestLocationId = "us-il-ch1-azr" },  // North Central US
        new WebTestGeolocation { WebTestLocationId = "emea-gb-db3-azr" }, // UK South
        new WebTestGeolocation { WebTestLocationId = "apac-sg-sin-azr" }  // Southeast Asia
    },
    Configuration = new WebTestConfiguration
    {
        WebTest = """
            <WebTest Name="Homepage" Enabled="True" Timeout="120"
                     xmlns="http://microsoft.com/schemas/VisualStudio/TeamTest/2010">
                <Items>
                    <Request Method="GET" Version="1.1" Url="https://myapp.example.com"
                             ThinkTime="0" Timeout="120" ParseDependentRequests="False"
                             FollowRedirects="True" RecordResult="True" Cache="False"
                             ResponseTimeGoal="0" Encoding="utf-8" ExpectedHttpStatusCode="200" />
                </Items>
            </WebTest>
        """
    },
    Tags =
    {
        { $"hidden-link:/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/microsoft.insights/components/my-appinsights", "Resource" }
    }
};

ArmOperation<WebTestResource> operation = await webTests
    .CreateOrUpdateAsync(WaitUntil.Completed, "webtest-homepage", urlPingTest);

WebTestResource webTest = operation.Value;
Console.WriteLine($"Web test created: {webTest.Data.Name}");
```

### 5. Tạo Multi-Step Web Test

```csharp
WebTestData multiStepTest = new WebTestData(AzureLocation.EastUS)
{
    Kind = WebTestKind.MultiStep,
    SyntheticMonitorId = "webtest-multistep-login",
    WebTestName = "Login Flow Test",
    Description = "Tests login functionality",
    IsEnabled = true,
    Frequency = 900, // 15 phút
    Timeout = 300,   // 5 phút
    WebTestKind = WebTestKind.MultiStep,
    IsRetryEnabled = true,
    Locations =
    {
        new WebTestGeolocation { WebTestLocationId = "us-ca-sjc-azr" }
    },
    Configuration = new WebTestConfiguration
    {
        WebTest = """
            <WebTest Name="LoginFlow" Enabled="True" Timeout="300"
                     xmlns="http://microsoft.com/schemas/VisualStudio/TeamTest/2010">
                <Items>
                    <Request Method="GET" Version="1.1" Url="https://myapp.example.com/login"
                             ThinkTime="0" Timeout="60" />
                    <Request Method="POST" Version="1.1" Url="https://myapp.example.com/api/auth"
                             ThinkTime="0" Timeout="60">
                        <Headers>
                            <Header Name="Content-Type" Value="application/json" />
                        </Headers>
                        <Body>{"username":"testuser","password":"{{TestPassword}}"}</Body>
                    </Request>
                </Items>
            </WebTest>
        """
    },
    Tags =
    {
        { $"hidden-link:/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/microsoft.insights/components/my-appinsights", "Resource" }
    }
};

await webTests.CreateOrUpdateAsync(WaitUntil.Completed, "webtest-login-flow", multiStepTest);
```

### 6. Tạo Workbook

```csharp
WorkbookCollection workbooks = resourceGroup.GetWorkbooks();

WorkbookData workbookData = new WorkbookData(AzureLocation.EastUS)
{
    DisplayName = "Application Performance Dashboard",
    Category = "workbook",
    Kind = WorkbookSharedTypeKind.Shared,
    SerializedData = """
    {
        "version": "Notebook/1.0",
        "items": [
            {
                "type": 1,
                "content": {
                    "json": "# Application Performance\n\nThis workbook shows application performance metrics."
                },
                "name": "header"
            },
            {
                "type": 3,
                "content": {
                    "version": "KqlItem/1.0",
                    "query": "requests\n| summarize count() by bin(timestamp, 1h)\n| render timechart",
                    "size": 0,
                    "title": "Requests per Hour",
                    "timeContext": {
                        "durationMs": 86400000
                    },
                    "queryType": 0,
                    "resourceType": "microsoft.insights/components"
                },
                "name": "requestsChart"
            }
        ],
        "isLocked": false
    }
    """,
    SourceId = component.Id,
    Tags =
    {
        { "environment", "production" }
    }
};

// Lưu ý: Workbook ID phải là một GUID mới
string workbookId = Guid.NewGuid().ToString();

ArmOperation<WorkbookResource> operation = await workbooks
    .CreateOrUpdateAsync(WaitUntil.Completed, workbookId, workbookData);

WorkbookResource workbook = operation.Value;
Console.WriteLine($"Workbook created: {workbook.Data.DisplayName}");
```

### 7. Liên kết Storage Account

```csharp
ApplicationInsightsComponentResource component = await resourceGroup
    .GetApplicationInsightsComponentAsync("my-appinsights");

ComponentLinkedStorageAccountCollection linkedStorage = component.GetComponentLinkedStorageAccounts();

ComponentLinkedStorageAccountData storageData = new ComponentLinkedStorageAccountData
{
    LinkedStorageAccount = new ResourceIdentifier(
        "/subscriptions/<sub-id>/resourceGroups/<rg>/providers/Microsoft.Storage/storageAccounts/<storage-account>")
};

ArmOperation<ComponentLinkedStorageAccountResource> operation = await linkedStorage
    .CreateOrUpdateAsync(WaitUntil.Completed, StorageType.ServiceProfiler, storageData);
```

### 8. Liệt kê và Quản lý Thành phần

```csharp
// Liệt kê tất cả các thành phần Application Insights trong resource group
await foreach (ApplicationInsightsComponentResource component in
    resourceGroup.GetApplicationInsightsComponents())
{
    Console.WriteLine($"Component: {component.Data.Name}");
    Console.WriteLine($"  App ID: {component.Data.AppId}");
    Console.WriteLine($"  Type: {component.Data.ApplicationType}");
    Console.WriteLine($"  Ingestion Mode: {component.Data.IngestionMode}");
    Console.WriteLine($"  Retention: {component.Data.RetentionInDays} days");
}

// Liệt kê web tests
await foreach (WebTestResource webTest in resourceGroup.GetWebTests())
{
    Console.WriteLine($"Web Test: {webTest.Data.WebTestName}");
    Console.WriteLine($"  Enabled: {webTest.Data.IsEnabled}");
    Console.WriteLine($"  Frequency: {webTest.Data.Frequency}s");
}

// Liệt kê workbooks
await foreach (WorkbookResource workbook in resourceGroup.GetWorkbooks())
{
    Console.WriteLine($"Workbook: {workbook.Data.DisplayName}");
}
```

### 9. Cập nhật Thành phần

```csharp
ApplicationInsightsComponentResource component = await resourceGroup
    .GetApplicationInsightsComponentAsync("my-appinsights");

// Cập nhật sử dụng dữ liệu đầy đủ (thao tác PUT)
ApplicationInsightsComponentData updateData = component.Data;
updateData.RetentionInDays = 180;
updateData.SamplingPercentage = 50;
updateData.Tags["updated"] = "true";

ArmOperation<ApplicationInsightsComponentResource> operation = await resourceGroup
    .GetApplicationInsightsComponents()
    .CreateOrUpdateAsync(WaitUntil.Completed, "my-appinsights", updateData);
```

### 10. Xóa Tài nguyên

```csharp
// Xóa thành phần Application Insights
ApplicationInsightsComponentResource component = await resourceGroup
    .GetApplicationInsightsComponentAsync("my-appinsights");
await component.DeleteAsync(WaitUntil.Completed);

// Xóa web test
WebTestResource webTest = await resourceGroup.GetWebTestAsync("webtest-homepage");
await webTest.DeleteAsync(WaitUntil.Completed);
```

## Tham khảo Các Loại Chính

| Loại                                         | Mục đích                        |
| -------------------------------------------- | ------------------------------- |
| `ApplicationInsightsComponentResource`       | Thành phần App Insights         |
| `ApplicationInsightsComponentData`           | Cấu hình thành phần             |
| `ApplicationInsightsComponentCollection`     | Collection các thành phần       |
| `ApplicationInsightsComponentApiKeyResource` | API key cho truy cập lập trình  |
| `WebTestResource`                            | Availability/web test           |
| `WebTestData`                                | Cấu hình Web test               |
| `WorkbookResource`                           | Workbook phân tích              |
| `WorkbookData`                               | Cấu hình Workbook               |
| `ComponentLinkedStorageAccountResource`      | Linked storage cho xuất dữ liệu |

## Các Loại Ứng dụng

| Loại                | Giá trị Enum |
| ------------------- | ------------ |
| Web Application     | `Web`        |
| iOS Application     | `iOS`        |
| Java Application    | `Java`       |
| Node.js Application | `NodeJS`     |
| .NET Application    | `MRT`        |
| Other               | `Other`      |

## Web Test Locations

| Location ID         | Vùng             |
| ------------------- | ---------------- |
| `us-ca-sjc-azr`     | West US          |
| `us-tx-sn1-azr`     | South Central US |
| `us-il-ch1-azr`     | North Central US |
| `us-va-ash-azr`     | East US          |
| `emea-gb-db3-azr`   | UK South         |
| `emea-nl-ams-azr`   | West Europe      |
| `emea-fr-pra-edge`  | France Central   |
| `apac-sg-sin-azr`   | Southeast Asia   |
| `apac-hk-hkn-azr`   | East Asia        |
| `apac-jp-kaw-edge`  | Japan East       |
| `latam-br-gru-edge` | Brazil South     |
| `emea-au-syd-edge`  | Australia East   |

## Thực hành Tốt nhất

1. **Sử dụng workspace-based** — App Insights dựa trên Workspace là tiêu chuẩn hiện tại
2. **Liên kết với Log Analytics** — Lưu trữ dữ liệu trong Log Analytics để truy vấn tốt hơn
3. **Đặt thời gian lưu giữ phù hợp** — Cân bằng giữa chi phí và khả năng sẵn có của dữ liệu
4. **Sử dụng lấy mẫu (sampling)** — Giảm chi phí cho các ứng dụng có khối lượng lớn
5. **Lưu trữ connection string an toàn** — Sử dụng Key Vault hoặc managed identity
6. **Bật nhiều vị trí kiểm tra** — Để giám sát tính sẵn sàng chính xác
7. **Sử dụng workbooks** — Cho các bảng điều khiển tùy chỉnh và phân tích
8. **Thiết lập cảnh báo** — Dựa trên các bài kiểm tra tính sẵn sàng và số liệu
9. **Gắn thẻ tài nguyên** — Để phân bổ chi phí và tổ chức
10. **Sử dụng private endpoints** — Để nhập dữ liệu an toàn

## Xử lý Lỗi

```csharp
using Azure;

try
{
    ArmOperation<ApplicationInsightsComponentResource> operation = await components
        .CreateOrUpdateAsync(WaitUntil.Completed, "my-appinsights", data);
}
catch (RequestFailedException ex) when (ex.Status == 409)
{
    Console.WriteLine("Component already exists");
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

## Tích hợp SDK

Sử dụng connection string với Application Insights SDK:

```csharp
// Program.cs trong ASP.NET Core
builder.Services.AddApplicationInsightsTelemetry(options =>
{
    options.ConnectionString = configuration["ApplicationInsights:ConnectionString"];
});

// Hoặc thiết lập qua biến môi trường
// APPLICATIONINSIGHTS_CONNECTION_STRING=InstrumentationKey=...;IngestionEndpoint=...
```

## Các SDK Liên quan

| SDK                                         | Mục đích                     | Cài đặt                                                        |
| ------------------------------------------- | ---------------------------- | -------------------------------------------------------------- |
| `Azure.ResourceManager.ApplicationInsights` | Quản lý tài nguyên (SDK này) | `dotnet add package Azure.ResourceManager.ApplicationInsights` |
| `Microsoft.ApplicationInsights`             | Telemetry SDK                | `dotnet add package Microsoft.ApplicationInsights`             |
| `Microsoft.ApplicationInsights.AspNetCore`  | Tích hợp ASP.NET Core        | `dotnet add package Microsoft.ApplicationInsights.AspNetCore`  |
| `Azure.Monitor.OpenTelemetry.Exporter`      | Xuất OpenTelemetry           | `dotnet add package Azure.Monitor.OpenTelemetry.Exporter`      |

## Liên kết Tham khảo

| Tài nguyên        | URL                                                                                                                    |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Gói NuGet         | https://www.nuget.org/packages/Azure.ResourceManager.ApplicationInsights                                               |
| Tham khảo API     | https://learn.microsoft.com/dotnet/api/azure.resourcemanager.applicationinsights                                       |
| Tài liệu Sản phẩm | https://learn.microsoft.com/azure/azure-monitor/app/app-insights-overview                                              |
| Mã nguồn GitHub   | https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/applicationinsights/Azure.ResourceManager.ApplicationInsights |
