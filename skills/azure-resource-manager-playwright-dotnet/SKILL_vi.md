---
name: azure-resource-manager-playwright-dotnet
description: |
  Azure Resource Manager SDK cho Microsoft Playwright Testing trong .NET. Sử dụng cho các hoạt động MANAGEMENT PLANE: tạo/quản lý workspace Playwright Testing, kiểm tra tính khả dụng của tên, và quản lý hạn ngạch (quota) workspace thông qua Azure Resource Manager. KHÔNG dùng để chạy các bài kiểm tra Playwright - hãy sử dụng Azure.Developer.MicrosoftPlaywrightTesting.NUnit cho việc đó. Triggers: "Playwright workspace", "create Playwright Testing workspace", "manage Playwright resources", "ARM Playwright", "PlaywrightWorkspaceResource", "provision Playwright Testing".
package: Azure.ResourceManager.Playwright
---

# Azure.ResourceManager.Playwright (.NET)

SDK management plane để cung cấp và quản lý workspace Microsoft Playwright Testing thông qua Azure Resource Manager.

> **⚠️ Management vs Kiểm thử**
>
> - **SDK này (Azure.ResourceManager.Playwright)**: Tạo workspace, quản lý quota, kiểm tra tính khả dụng của tên
> - **SDK Thực thi Kiểm thử (Azure.Developer.MicrosoftPlaywrightTesting.NUnit)**: Chạy các bài kiểm tra Playwright quy mô lớn trên các trình duyệt đám mây

## Cài đặt

```bash
dotnet add package Azure.ResourceManager.Playwright
dotnet add package Azure.Identity
```

**Phiên bản Hiện tại**: Stable v1.0.0, Preview v1.0.0-beta.1

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
using Azure.ResourceManager.Playwright;

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
    ├── PlaywrightQuotaResource (hạn ngạch cấp subscription)
    └── ResourceGroupResource
        └── PlaywrightWorkspaceResource
            └── PlaywrightWorkspaceQuotaResource (hạn ngạch cấp workspace)
```

## Quy trình Công việc Cốt lõi

### 1. Tạo Playwright Workspace

```csharp
using Azure.ResourceManager.Playwright;
using Azure.ResourceManager.Playwright.Models;

// Lấy resource group
var resourceGroup = await subscription
    .GetResourceGroupAsync("my-resource-group");

// Định nghĩa workspace
var workspaceData = new PlaywrightWorkspaceData(AzureLocation.WestUS3)
{
    // Tùy chọn: Cấu hình regional affinity và local auth
    RegionalAffinity = PlaywrightRegionalAffinity.Enabled,
    LocalAuth = PlaywrightLocalAuth.Enabled,
    Tags =
    {
        ["Team"] = "Dev Exp",
        ["Environment"] = "Production"
    }
};

// Tạo workspace (long-running operation)
var workspaceCollection = resourceGroup.Value.GetPlaywrightWorkspaces();
var operation = await workspaceCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "my-playwright-workspace",
    workspaceData);

PlaywrightWorkspaceResource workspace = operation.Value;

// Lấy data plane URI để chạy tests
Console.WriteLine($"Data Plane URI: {workspace.Data.DataplaneUri}");
Console.WriteLine($"Workspace ID: {workspace.Data.WorkspaceId}");
```

### 2. Lấy Workspace Hiện có

```csharp
// Lấy theo tên
var workspace = await workspaceCollection.GetAsync("my-playwright-workspace");

// Hoặc kiểm tra xem có tồn tại không trước
bool exists = await workspaceCollection.ExistsAsync("my-playwright-workspace");
if (exists)
{
    var existingWorkspace = await workspaceCollection.GetAsync("my-playwright-workspace");
    Console.WriteLine($"Workspace found: {existingWorkspace.Value.Data.Name}");
}
```

### 3. Liệt kê Workspaces

```csharp
// Liệt kê trong resource group
await foreach (var workspace in workspaceCollection.GetAllAsync())
{
    Console.WriteLine($"Workspace: {workspace.Data.Name}");
    Console.WriteLine($"  Location: {workspace.Data.Location}");
    Console.WriteLine($"  State: {workspace.Data.ProvisioningState}");
    Console.WriteLine($"  Data Plane URI: {workspace.Data.DataplaneUri}");
}

// Liệt kê trên toàn bộ subscription
await foreach (var workspace in subscription.GetPlaywrightWorkspacesAsync())
{
    Console.WriteLine($"Workspace: {workspace.Data.Name}");
}
```

### 4. Cập nhật Workspace

```csharp
var patch = new PlaywrightWorkspacePatch
{
    Tags =
    {
        ["Team"] = "Dev Exp",
        ["Environment"] = "Staging",
        ["UpdatedAt"] = DateTime.UtcNow.ToString("o")
    }
};

var updatedWorkspace = await workspace.Value.UpdateAsync(patch);
```

### 5. Kiểm tra Tính khả dụng của Tên

```csharp
using Azure.ResourceManager.Playwright.Models;

var checkRequest = new PlaywrightCheckNameAvailabilityContent
{
    Name = "my-new-workspace",
    ResourceType = "Microsoft.LoadTestService/playwrightWorkspaces"
};

var result = await subscription.CheckPlaywrightNameAvailabilityAsync(checkRequest);

if (result.Value.IsNameAvailable == true)
{
    Console.WriteLine("Name is available!");
}
else
{
    Console.WriteLine($"Name unavailable: {result.Value.Message}");
    Console.WriteLine($"Reason: {result.Value.Reason}");
}
```

### 6. Lấy Thông tin Hạn ngạch (Quota)

```csharp
// Hạn ngạch cấp Subscription
await foreach (var quota in subscription.GetPlaywrightQuotasAsync(AzureLocation.WestUS3))
{
    Console.WriteLine($"Quota: {quota.Data.Name}");
    Console.WriteLine($"  Limit: {quota.Data.Limit}");
    Console.WriteLine($"  Used: {quota.Data.Used}");
}

// Hạn ngạch cấp Workspace
var workspaceQuotas = workspace.Value.GetAllPlaywrightWorkspaceQuota();
await foreach (var quota in workspaceQuotas.GetAllAsync())
{
    Console.WriteLine($"Workspace Quota: {quota.Data.Name}");
}
```

### 7. Xóa Workspace

```csharp
// Xóa (long-running operation)
await workspace.Value.DeleteAsync(WaitUntil.Completed);
```

## Tham chiếu Các Loại Chính

| Loại                                     | Mục đích                                      |
| ---------------------------------------- | --------------------------------------------- |
| `ArmClient`                              | Điểm vào cho tất cả các hoạt động ARM         |
| `PlaywrightWorkspaceResource`            | Đại diện cho một workspace Playwright Testing |
| `PlaywrightWorkspaceCollection`          | Collection cho workspace CRUD                 |
| `PlaywrightWorkspaceData`                | Payload tạo/phản hồi workspace                |
| `PlaywrightWorkspacePatch`               | Payload cập nhật workspace                    |
| `PlaywrightQuotaResource`                | Thông tin hạn ngạch cấp subscription          |
| `PlaywrightWorkspaceQuotaResource`       | Thông tin hạn ngạch cấp workspace             |
| `PlaywrightExtensions`                   | Các phương thức mở rộng cho tài nguyên ARM    |
| `PlaywrightCheckNameAvailabilityContent` | Yêu cầu kiểm tra tính khả dụng của tên        |

## Các Thuộc tính Workspace

| Thuộc tính          | Mô tả                                                                   |
| ------------------- | ----------------------------------------------------------------------- |
| `DataplaneUri`      | URI để chạy các test (ví dụ: `https://api.dataplane.{guid}.domain.com`) |
| `WorkspaceId`       | Định danh workspace duy nhất (GUID)                                     |
| `RegionalAffinity`  | Bật/tắt regional affinity cho thực thi test                             |
| `LocalAuth`         | Bật/tắt xác thực cục bộ (access tokens)                                 |
| `ProvisioningState` | Trạng thái provisioning hiện tại (Succeeded, Failed, v.v.)              |

## Thực hành Tốt nhất

1. **Sử dụng `WaitUntil.Completed`** cho các hoạt động phải hoàn thành trước khi tiếp tục
2. **Sử dụng `WaitUntil.Started`** khi bạn muốn thăm dò thủ công hoặc chạy các hoạt động song song
3. **Luôn sử dụng `DefaultAzureCredential`** — không bao giờ hardcode keys
4. **Xử lý `RequestFailedException`** cho các lỗi ARM API
5. **Sử dụng `CreateOrUpdateAsync`** cho các hoạt động idempotent
6. **Điều hướng phân cấp** qua các phương thức `Get*` (ví dụ: `resourceGroup.GetPlaywrightWorkspaces()`)
7. **Lưu trữ DataplaneUri** sau khi tạo workspace để cấu hình thực thi test

## Xử lý Lỗi

```csharp
using Azure;

try
{
    var operation = await workspaceCollection.CreateOrUpdateAsync(
        WaitUntil.Completed, workspaceName, workspaceData);
}
catch (RequestFailedException ex) when (ex.Status == 409)
{
    Console.WriteLine("Workspace already exists");
}
catch (RequestFailedException ex) when (ex.Status == 400)
{
    Console.WriteLine($"Bad request: {ex.Message}");
}
catch (RequestFailedException ex)
{
    Console.WriteLine($"ARM Error: {ex.Status} - {ex.ErrorCode}: {ex.Message}");
}
```

## Tích hợp với Thực thi Kiểm thử

Sau khi tạo một workspace, sử dụng `DataplaneUri` để cấu hình các test Playwright của bạn:

```csharp
// 1. Tạo workspace (SDK này)
var workspace = await workspaceCollection.CreateOrUpdateAsync(
    WaitUntil.Completed, "my-workspace", workspaceData);

// 2. Lấy service URL
var serviceUrl = workspace.Value.Data.DataplaneUri;

// 3. Thiết lập biến môi trường cho thực thi test
Environment.SetEnvironmentVariable("PLAYWRIGHT_SERVICE_URL", serviceUrl.ToString());

// 4. Chạy tests bằng cách sử dụng Azure.Developer.MicrosoftPlaywrightTesting.NUnit
// (gói riêng biệt cho thực thi test)
```

## Các SDK Liên quan

| SDK                                                | Mục đích                                 | Cài đặt                                                                            |
| -------------------------------------------------- | ---------------------------------------- | ---------------------------------------------------------------------------------- |
| `Azure.ResourceManager.Playwright`                 | Management plane (SDK này)               | `dotnet add package Azure.ResourceManager.Playwright`                              |
| `Azure.Developer.MicrosoftPlaywrightTesting.NUnit` | Chạy NUnit Playwright tests ở quy mô lớn | `dotnet add package Azure.Developer.MicrosoftPlaywrightTesting.NUnit --prerelease` |
| `Azure.Developer.Playwright`                       | Thư viện client Playwright               | `dotnet add package Azure.Developer.Playwright`                                    |

## Thông tin API

- **Nhà cung cấp Tài nguyên**: `Microsoft.LoadTestService`
- **Phiên bản API Mặc định**: `2025-09-01`
- **Loại Tài nguyên**: `Microsoft.LoadTestService/playwrightWorkspaces`

## Liên kết Tài liệu

- [Azure.ResourceManager.Playwright API Reference](https://learn.microsoft.com/en-us/dotnet/api/azure.resourcemanager.playwright)
- [Microsoft Playwright Testing Overview](https://learn.microsoft.com/en-us/azure/playwright-testing/overview-what-is-microsoft-playwright-testing)
