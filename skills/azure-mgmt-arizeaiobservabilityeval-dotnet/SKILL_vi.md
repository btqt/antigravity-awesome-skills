---
name: azure-mgmt-arizeaiobservabilityeval-dotnet
description: |
  Azure Resource Manager SDK cho Arize AI Observability và Evaluation (.NET). Sử dụng khi quản lý các tổ chức Arize AI 
  trên Azure thông qua Azure Marketplace, tạo/cập nhật/xóa tài nguyên Arize, hoặc tích hợp Arize ML observability 
  vào các ứng dụng .NET. Triggers: "Arize AI", "ML observability", "ArizeAIObservabilityEval", "Arize organization".
package: Azure.ResourceManager.ArizeAIObservabilityEval
---

# Azure.ResourceManager.ArizeAIObservabilityEval

SDK .NET để quản lý tài nguyên Arize AI Observability và Evaluation trên Azure.

## Cài đặt

```bash
dotnet add package Azure.ResourceManager.ArizeAIObservabilityEval --version 1.0.0
```

## Thông tin Gói

| Thuộc tính    | Giá trị                                                   |
| ------------- | --------------------------------------------------------- |
| Gói           | `Azure.ResourceManager.ArizeAIObservabilityEval`          |
| Phiên bản     | `1.0.0` (GA)                                              |
| Phiên bản API | `2024-10-01`                                              |
| Loại ARM      | `ArizeAi.ObservabilityEval/organizations`                 |
| Phụ thuộc     | `Azure.Core` >= 1.46.2, `Azure.ResourceManager` >= 1.13.1 |

## Biến Môi trường

```bash
AZURE_SUBSCRIPTION_ID=<your-subscription-id>
AZURE_TENANT_ID=<your-tenant-id>
AZURE_CLIENT_ID=<your-client-id>
AZURE_CLIENT_SECRET=<your-client-secret>
```

## Xác thực

```csharp
using Azure.Identity;
using Azure.ResourceManager;
using Azure.ResourceManager.ArizeAIObservabilityEval;

// Luôn sử dụng DefaultAzureCredential
var credential = new DefaultAzureCredential();
var armClient = new ArmClient(credential);
```

## Quy trình Làm việc Cốt lõi

### Tạo một Tổ chức Arize AI

```csharp
using Azure.Core;
using Azure.ResourceManager.Resources;
using Azure.ResourceManager.ArizeAIObservabilityEval;
using Azure.ResourceManager.ArizeAIObservabilityEval.Models;

// Lấy subscription và resource group
var subscriptionId = Environment.GetEnvironmentVariable("AZURE_SUBSCRIPTION_ID");
var subscription = await armClient.GetSubscriptionResource(
    SubscriptionResource.CreateResourceIdentifier(subscriptionId)).GetAsync();
var resourceGroup = await subscription.Value.GetResourceGroupAsync("my-resource-group");

// Lấy collection organization
var collection = resourceGroup.Value.GetArizeAIObservabilityEvalOrganizations();

// Tạo dữ liệu organization
var data = new ArizeAIObservabilityEvalOrganizationData(AzureLocation.EastUS)
{
    Properties = new ArizeAIObservabilityEvalOrganizationProperties
    {
        Marketplace = new ArizeAIObservabilityEvalMarketplaceDetails
        {
            SubscriptionId = "marketplace-subscription-id",
            OfferDetails = new ArizeAIObservabilityEvalOfferDetails
            {
                PublisherId = "arikimlabs1649082416596",
                OfferId = "arize-liftr-1",
                PlanId = "arize-liftr-1-plan",
                PlanName = "Arize AI Plan",
                TermUnit = "P1M",
                TermId = "term-id"
            }
        },
        User = new ArizeAIObservabilityEvalUserDetails
        {
            FirstName = "John",
            LastName = "Doe",
            EmailAddress = "john.doe@example.com"
        }
    },
    Tags = { ["environment"] = "production" }
};

// Tạo (thao tác chạy lâu)
var operation = await collection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "my-arize-org",
    data);

var organization = operation.Value;
Console.WriteLine($"Created: {organization.Data.Name}");
```

### Lấy một Tổ chức

```csharp
// Tùy chọn 1: Từ collection
var org = await collection.GetAsync("my-arize-org");

// Tùy chọn 2: Kiểm tra xem có tồn tại trước không
var exists = await collection.ExistsAsync("my-arize-org");
if (exists.Value)
{
    var org = await collection.GetAsync("my-arize-org");
}

// Tùy chọn 3: GetIfExists (trả về null nếu không tìm thấy)
var response = await collection.GetIfExistsAsync("my-arize-org");
if (response.HasValue)
{
    var org = response.Value;
}
```

### Liệt kê các Tổ chức

```csharp
// Liệt kê trong resource group
await foreach (var org in collection.GetAllAsync())
{
    Console.WriteLine($"Org: {org.Data.Name}, State: {org.Data.Properties?.ProvisioningState}");
}

// Liệt kê trong subscription
await foreach (var org in subscription.Value.GetArizeAIObservabilityEvalOrganizationsAsync())
{
    Console.WriteLine($"Org: {org.Data.Name}");
}
```

### Cập nhật một Tổ chức

```csharp
// Cập nhật tags
var org = await collection.GetAsync("my-arize-org");
var updateData = new ArizeAIObservabilityEvalOrganizationPatch
{
    Tags = { ["environment"] = "staging", ["team"] = "ml-ops" }
};
var updated = await org.Value.UpdateAsync(updateData);
```

### Xóa một Tổ chức

```csharp
var org = await collection.GetAsync("my-arize-org");
await org.Value.DeleteAsync(WaitUntil.Completed);
```

## Các Loại Chính

| Loại                                               | Mục đích                                   |
| -------------------------------------------------- | ------------------------------------------ |
| `ArizeAIObservabilityEvalOrganizationResource`     | Tài nguyên ARM chính cho các tổ chức Arize |
| `ArizeAIObservabilityEvalOrganizationCollection`   | Collection cho các thao tác CRUD           |
| `ArizeAIObservabilityEvalOrganizationData`         | Mô hình dữ liệu tài nguyên                 |
| `ArizeAIObservabilityEvalOrganizationProperties`   | Các thuộc tính tổ chức                     |
| `ArizeAIObservabilityEvalMarketplaceDetails`       | Thông tin đăng ký Azure Marketplace        |
| `ArizeAIObservabilityEvalOfferDetails`             | Cấu hình offer Marketplace                 |
| `ArizeAIObservabilityEvalUserDetails`              | Thông tin liên hệ người dùng               |
| `ArizeAIObservabilityEvalOrganizationPatch`        | Mô hình Patch cho các bản cập nhật         |
| `ArizeAIObservabilityEvalSingleSignOnPropertiesV2` | Cấu hình SSO                               |

## Enums

| Enum                                                    | Giá trị                                                                               |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| `ArizeAIObservabilityEvalOfferProvisioningState`        | `Succeeded`, `Failed`, `Canceled`, `Provisioning`, `Updating`, `Deleting`, `Accepted` |
| `ArizeAIObservabilityEvalMarketplaceSubscriptionStatus` | `PendingFulfillmentStart`, `Subscribed`, `Suspended`, `Unsubscribed`                  |
| `ArizeAIObservabilityEvalSingleSignOnState`             | `Initial`, `Enable`, `Disable`                                                        |
| `ArizeAIObservabilityEvalSingleSignOnType`              | `Saml`, `OpenId`                                                                      |

## Thực hành Tốt nhất

1. **Sử dụng các phương thức async** — Tất cả các thao tác đều hỗ trợ async/await
2. **Xử lý các thao tác chạy lâu** — Sử dụng `WaitUntil.Completed` hoặc thăm dò thủ công
3. **Sử dụng GetIfExistsAsync** — Tránh các ngoại lệ cho logic có điều kiện
4. **Triển khai các chính sách thử lại (retry policies)** — Cấu hình thông qua `ArmClientOptions`
5. **Sử dụng định danh tài nguyên (resource identifiers)** — Để truy cập tài nguyên trực tiếp mà không cần liệt kê
6. **Đóng client đúng cách** — Sử dụng câu lệnh `using` hoặc dispose một cách rõ ràng

## Xử lý Lỗi

```csharp
try
{
    var org = await collection.GetAsync("my-arize-org");
}
catch (Azure.RequestFailedException ex) when (ex.Status == 404)
{
    Console.WriteLine("Organization not found");
}
catch (Azure.RequestFailedException ex)
{
    Console.WriteLine($"Azure error: {ex.Message}");
}
```

## Truy cập Tài nguyên Trực tiếp

```csharp
// Truy cập tài nguyên trực tiếp bằng ID (không cần liệt kê)
var resourceId = ArizeAIObservabilityEvalOrganizationResource.CreateResourceIdentifier(
    subscriptionId,
    "my-resource-group",
    "my-arize-org");

var org = armClient.GetArizeAIObservabilityEvalOrganizationResource(resourceId);
var data = await org.GetAsync();
```

## Liên kết

- [Gói NuGet](https://www.nuget.org/packages/Azure.ResourceManager.ArizeAIObservabilityEval)
- [Azure SDK for .NET](https://github.com/Azure/azure-sdk-for-net)
- [Arize AI](https://arize.com/)
