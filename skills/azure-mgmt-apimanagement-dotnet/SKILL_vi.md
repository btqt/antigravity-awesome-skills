---
name: azure-mgmt-apimanagement-dotnet
description: |
  Azure Resource Manager SDK cho API Management trong .NET. Sử dụng cho các thao tác MANAGEMENT PLANE: tạo/quản lý dịch vụ APIM, APIs, products, subscriptions, policies, users, groups, gateways, và backends thông qua Azure Resource Manager. Triggers: "API Management", "APIM service", "create APIM", "manage APIs", "ApiManagementServiceResource", "API policies", "APIM products", "APIM subscriptions".
package: Azure.ResourceManager.ApiManagement
---

# Azure.ResourceManager.ApiManagement (.NET)

SDK management plane để cung cấp và quản lý tài nguyên Azure API Management thông qua Azure Resource Manager.

> **⚠️ Management vs Data Plane**
>
> - **SDK này (Azure.ResourceManager.ApiManagement)**: Tạo services, APIs, products, subscriptions, policies, users, groups
> - **Data Plane**: Gọi API trực tiếp đến các endpoint gateway APIM của bạn

## Cài đặt

```bash
dotnet add package Azure.ResourceManager.ApiManagement
dotnet add package Azure.Identity
```

**Phiên bản Hiện tại**: v1.3.0

## Biến Môi trường

```bash
AZURE_SUBSCRIPTION_ID=<your-subscription-id>
# Cho xác thực service principal (tùy chọn)
AZURE_TENANT_ID=<tenant-id>
AZURE_CLIENT_ID=<client-id>
AZURE_CLIENT_SECRET=<client-secret>
```

## Xác thực

```csharp
using Azure.Identity;
using Azure.ResourceManager;
using Azure.ResourceManager.ApiManagement;

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
        └── ApiManagementServiceResource
            ├── ApiResource
            │   ├── ApiOperationResource
            │   │   └── ApiOperationPolicyResource
            │   ├── ApiPolicyResource
            │   ├── ApiSchemaResource
            │   └── ApiDiagnosticResource
            ├── ApiManagementProductResource
            │   ├── ProductApiResource
            │   ├── ProductGroupResource
            │   └── ProductPolicyResource
            ├── ApiManagementSubscriptionResource
            ├── ApiManagementPolicyResource
            ├── ApiManagementUserResource
            ├── ApiManagementGroupResource
            ├── ApiManagementBackendResource
            ├── ApiManagementGatewayResource
            ├── ApiManagementCertificateResource
            ├── ApiManagementNamedValueResource
            └── ApiManagementLoggerResource
```

## Quy trình Làm việc Cốt lõi

### 1. Tạo Dịch vụ API Management

```csharp
using Azure.ResourceManager.ApiManagement;
using Azure.ResourceManager.ApiManagement.Models;

// Lấy resource group
var resourceGroup = await subscription
    .GetResourceGroupAsync("my-resource-group");

// Định nghĩa dịch vụ
var serviceData = new ApiManagementServiceData(
    location: AzureLocation.EastUS,
    sku: new ApiManagementServiceSkuProperties(
        ApiManagementServiceSkuType.Developer,
        capacity: 1),
    publisherEmail: "admin@contoso.com",
    publisherName: "Contoso");

// Tạo dịch vụ (thao tác chạy lâu - có thể mất hơn 30 phút)
var serviceCollection = resourceGroup.Value.GetApiManagementServices();
var operation = await serviceCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "my-apim-service",
    serviceData);

ApiManagementServiceResource service = operation.Value;
```

### 2. Tạo một API

```csharp
var apiData = new ApiCreateOrUpdateContent
{
    DisplayName = "My API",
    Path = "myapi",
    Protocols = { ApiOperationInvokableProtocol.Https },
    ServiceUri = new Uri("https://backend.contoso.com/api")
};

var apiCollection = service.GetApis();
var apiOperation = await apiCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "my-api",
    apiData);

ApiResource api = apiOperation.Value;
```

### 3. Tạo một Product

```csharp
var productData = new ApiManagementProductData
{
    DisplayName = "Starter",
    Description = "Starter tier with limited access",
    IsSubscriptionRequired = true,
    IsApprovalRequired = false,
    SubscriptionsLimit = 1,
    State = ApiManagementProductState.Published
};

var productCollection = service.GetApiManagementProducts();
var productOperation = await productCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "starter",
    productData);

ApiManagementProductResource product = productOperation.Value;

// Thêm API vào product
await product.GetProductApis().CreateOrUpdateAsync(
    WaitUntil.Completed,
    "my-api");
```

### 4. Tạo Subscription

```csharp
var subscriptionData = new ApiManagementSubscriptionCreateOrUpdateContent
{
    DisplayName = "My Subscription",
    Scope = $"/products/{product.Data.Name}",
    State = ApiManagementSubscriptionState.Active
};

var subscriptionCollection = service.GetApiManagementSubscriptions();
var subOperation = await subscriptionCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "my-subscription",
    subscriptionData);

ApiManagementSubscriptionResource subscription = subOperation.Value;

// Lấy subscription keys
var keys = await subscription.GetSecretsAsync();
Console.WriteLine($"Primary Key: {keys.Value.PrimaryKey}");
```

### 5. Thiết lập Chính sách API (API Policy)

```csharp
var policyXml = @"
<policies>
    <inbound>
        <rate-limit calls=""100"" renewal-period=""60"" />
        <set-header name=""X-Custom-Header"" exists-action=""override"">
            <value>CustomValue</value>
        </set-header>
        <base />
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>";

var policyData = new PolicyContractData
{
    Value = policyXml,
    Format = PolicyContentFormat.Xml
};

await api.GetApiPolicy().CreateOrUpdateAsync(
    WaitUntil.Completed,
    policyData);
```

### 6. Sao lưu và Khôi phục

```csharp
// Sao lưu
var backupParams = new ApiManagementServiceBackupRestoreContent(
    storageAccount: "mystorageaccount",
    containerName: "apim-backups",
    backupName: "backup-2024-01-15")
{
    AccessType = StorageAccountAccessType.SystemAssignedManagedIdentity
};

await service.BackupAsync(WaitUntil.Completed, backupParams);

// Khôi phục
await service.RestoreAsync(WaitUntil.Completed, backupParams);
```

## Tham khảo Các Loại Chính

| Loại                                | Mục đích                               |
| ----------------------------------- | -------------------------------------- |
| `ArmClient`                         | Điểm vào cho tất cả các thao tác ARM   |
| `ApiManagementServiceResource`      | Đại diện cho một instance dịch vụ APIM |
| `ApiManagementServiceCollection`    | Collection cho CRUD dịch vụ            |
| `ApiResource`                       | Đại diện cho một API                   |
| `ApiManagementProductResource`      | Đại diện cho một product               |
| `ApiManagementSubscriptionResource` | Đại diện cho một subscription          |
| `ApiManagementPolicyResource`       | Policy cấp dịch vụ (Service-level)     |
| `ApiPolicyResource`                 | Policy cấp API (API-level)             |
| `ApiManagementUserResource`         | Đại diện cho một user                  |
| `ApiManagementGroupResource`        | Đại diện cho một group                 |
| `ApiManagementBackendResource`      | Đại diện cho một dịch vụ backend       |
| `ApiManagementGatewayResource`      | Đại diện cho một self-hosted gateway   |

## Các Loại SKU

| SKU           | Mục đích                                  | Công suất     |
| ------------- | ----------------------------------------- | ------------- |
| `Developer`   | Phát triển/thử nghiệm (không có SLA)      | 1             |
| `Basic`       | Sản xuất cấp nhập môn                     | 1-2           |
| `Standard`    | Khối lượng công việc trung bình           | 1-4           |
| `Premium`     | Tính sẵn sàng cao, đa vùng (multi-region) | 1-12 mỗi vùng |
| `Consumption` | Serverless, trả tiền theo lần gọi         | N/A           |

## Thực hành Tốt nhất

1. **Sử dụng `WaitUntil.Completed`** cho các thao tác phải hoàn thành trước khi tiếp tục
2. **Sử dụng `WaitUntil.Started`** cho các thao tác dài như tạo dịch vụ (30+ phút)
3. **Luôn sử dụng `DefaultAzureCredential`** — không bao giờ hardcode keys
4. **Xử lý `RequestFailedException`** cho các lỗi ARM API
5. **Sử dụng `CreateOrUpdateAsync`** cho các thao tác idempotent
6. **Điều hướng phân cấp** qua các phương thức `Get*` (ví dụ, `service.GetApis()`)
7. **Định dạng Policy** — Sử dụng định dạng XML cho policies; JSON cũng được hỗ trợ
8. **Tạo dịch vụ** — Developer SKU là nhanh nhất để thử nghiệm (~15-30 phút)

## Xử lý Lỗi

```csharp
using Azure;

try
{
    var operation = await serviceCollection.CreateOrUpdateAsync(
        WaitUntil.Completed, serviceName, serviceData);
}
catch (RequestFailedException ex) when (ex.Status == 409)
{
    Console.WriteLine("Service already exists");
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

## Các File Tham khảo

| File                                                                         | Khi nào nên đọc                                |
| ---------------------------------------------------------------------------- | ---------------------------------------------- |
| [references/service-management.md](references/service-management.md)         | Service CRUD, SKUs, networking, backup/restore |
| [references/apis-operations.md](references/apis-operations.md)               | APIs, operations, schemas, versioning          |
| [references/products-subscriptions.md](references/products-subscriptions.md) | Products, subscriptions, kiểm soát truy cập    |
| [references/policies.md](references/policies.md)                             | Mẫu Policy XML, phạm vi, các policies phổ biến |

## Tài nguyên Liên quan

| Tài nguyên                                                                                         | Mục đích                  |
| -------------------------------------------------------------------------------------------------- | ------------------------- |
| [API Management Documentation](https://learn.microsoft.com/en-us/azure/api-management/)            | Tài liệu Azure chính thức |
| [Policy Reference](https://learn.microsoft.com/en-us/azure/api-management/api-management-policies) | Tham khảo policy đầy đủ   |
| [SDK Reference](https://learn.microsoft.com/en-us/dotnet/api/azure.resourcemanager.apimanagement)  | Tham khảo API .NET        |
