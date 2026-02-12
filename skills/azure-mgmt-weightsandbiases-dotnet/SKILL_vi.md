---
name: azure-mgmt-weightsandbiases-dotnet
description: |
  Azure Weights & Biases SDK cho .NET. Theo dõi thử nghiệm ML và quản lý mô hình thông qua Azure Marketplace. Sử dụng để tạo các instance W&B, quản lý SSO, tích hợp marketplace, và khả năng quan sát ML. Triggers: "Weights and Biases", "W&B", "WeightsAndBiases", "ML experiment tracking", "model registry", "experiment management", "wandb".
package: Azure.ResourceManager.WeightsAndBiases
---

# Azure.ResourceManager.WeightsAndBiases (.NET)

SDK Azure Resource Manager để triển khai và quản lý các instance theo dõi thử nghiệm ML Weights & Biases thông qua Azure Marketplace.

## Cài đặt

```bash
dotnet add package Azure.ResourceManager.WeightsAndBiases --prerelease
dotnet add package Azure.Identity
```

**Phiên bản Hiện tại**: v1.0.0-beta.1 (preview)  
**Phiên bản API**: 2024-09-18-preview

## Biến Môi trường

```bash
AZURE_SUBSCRIPTION_ID=<your-subscription-id>
AZURE_RESOURCE_GROUP=<your-resource-group>
AZURE_WANDB_INSTANCE_NAME=<your-wandb-instance>
```

## Xác thực

```csharp
using Azure.Identity;
using Azure.ResourceManager;
using Azure.ResourceManager.WeightsAndBiases;

ArmClient client = new ArmClient(new DefaultAzureCredential());
```

## Phân cấp Tài nguyên

```
Subscription
└── ResourceGroup
    └── WeightsAndBiasesInstance    # Triển khai W&B từ Azure Marketplace
        ├── Properties
        │   ├── Marketplace          # Chi tiết offer, plan, publisher
        │   ├── User                 # Thông tin người dùng Admin
        │   ├── PartnerProperties    # Cấu hình cụ thể cho W&B (vùng, subdomain)
        │   └── SingleSignOnPropertiesV2  # Cấu hình SSO Entra ID
        └── Identity                 # Managed identity (tùy chọn)
```

## Quy trình Làm việc Cốt lõi

### 1. Tạo Instance Weights & Biases

```csharp
using Azure.ResourceManager.WeightsAndBiases;
using Azure.ResourceManager.WeightsAndBiases.Models;

ResourceGroupResource resourceGroup = await client
    .GetDefaultSubscriptionAsync()
    .Result
    .GetResourceGroupAsync("my-resource-group");

WeightsAndBiasesInstanceCollection instances = resourceGroup.GetWeightsAndBiasesInstances();

WeightsAndBiasesInstanceData data = new WeightsAndBiasesInstanceData(AzureLocation.EastUS)
{
    Properties = new WeightsAndBiasesInstanceProperties
    {
        // Cấu hình Marketplace
        Marketplace = new WeightsAndBiasesMarketplaceDetails
        {
            SubscriptionId = "<marketplace-subscription-id>",
            OfferDetails = new WeightsAndBiasesOfferDetails
            {
                PublisherId = "wandb",
                OfferId = "wandb-pay-as-you-go",
                PlanId = "wandb-payg",
                PlanName = "Pay As You Go",
                TermId = "monthly",
                TermUnit = "P1M"
            }
        },
        // Người dùng Admin
        User = new WeightsAndBiasesUserDetails
        {
            FirstName = "Admin",
            LastName = "User",
            EmailAddress = "admin@example.com",
            Upn = "admin@example.com"
        },
        // Cấu hình cụ thể cho W&B
        PartnerProperties = new WeightsAndBiasesPartnerProperties
        {
            Region = WeightsAndBiasesRegion.EastUS,
            Subdomain = "my-company-wandb"
        }
    },
    // Tùy chọn: Bật managed identity
    Identity = new ManagedServiceIdentity(ManagedServiceIdentityType.SystemAssigned)
};

ArmOperation<WeightsAndBiasesInstanceResource> operation = await instances
    .CreateOrUpdateAsync(WaitUntil.Completed, "my-wandb-instance", data);

WeightsAndBiasesInstanceResource instance = operation.Value;

Console.WriteLine($"W&B Instance created: {instance.Data.Name}");
Console.WriteLine($"Provisioning state: {instance.Data.Properties.ProvisioningState}");
```

### 2. Lấy Instance Hiện có

```csharp
WeightsAndBiasesInstanceResource instance = await resourceGroup
    .GetWeightsAndBiasesInstanceAsync("my-wandb-instance");

Console.WriteLine($"Instance: {instance.Data.Name}");
Console.WriteLine($"Location: {instance.Data.Location}");
Console.WriteLine($"State: {instance.Data.Properties.ProvisioningState}");

if (instance.Data.Properties.PartnerProperties != null)
{
    Console.WriteLine($"Region: {instance.Data.Properties.PartnerProperties.Region}");
    Console.WriteLine($"Subdomain: {instance.Data.Properties.PartnerProperties.Subdomain}");
}
```

### 3. Liệt kê Tất cả Instances

```csharp
// Liệt kê trong resource group
await foreach (WeightsAndBiasesInstanceResource instance in
    resourceGroup.GetWeightsAndBiasesInstances())
{
    Console.WriteLine($"Instance: {instance.Data.Name}");
    Console.WriteLine($"  Location: {instance.Data.Location}");
    Console.WriteLine($"  State: {instance.Data.Properties.ProvisioningState}");
}

// Liệt kê trong subscription
SubscriptionResource subscription = await client.GetDefaultSubscriptionAsync();
await foreach (WeightsAndBiasesInstanceResource instance in
    subscription.GetWeightsAndBiasesInstancesAsync())
{
    Console.WriteLine($"{instance.Data.Name} in {instance.Id.ResourceGroupName}");
}
```

### 4. Cấu hình Single Sign-On (SSO)

```csharp
WeightsAndBiasesInstanceResource instance = await resourceGroup
    .GetWeightsAndBiasesInstanceAsync("my-wandb-instance");

// Cập nhật với cấu hình SSO
WeightsAndBiasesInstanceData updateData = instance.Data;

updateData.Properties.SingleSignOnPropertiesV2 = new WeightsAndBiasSingleSignOnPropertiesV2
{
    Type = WeightsAndBiasSingleSignOnType.Saml,
    State = WeightsAndBiasSingleSignOnState.Enable,
    EnterpriseAppId = "<entra-app-id>",
    AadDomains = { "example.com", "contoso.com" }
};

ArmOperation<WeightsAndBiasesInstanceResource> operation = await resourceGroup
    .GetWeightsAndBiasesInstances()
    .CreateOrUpdateAsync(WaitUntil.Completed, "my-wandb-instance", updateData);
```

### 5. Cập nhật Instance

```csharp
WeightsAndBiasesInstanceResource instance = await resourceGroup
    .GetWeightsAndBiasesInstanceAsync("my-wandb-instance");

// Cập nhật tags
WeightsAndBiasesInstancePatch patch = new WeightsAndBiasesInstancePatch
{
    Tags =
    {
        { "environment", "production" },
        { "team", "ml-platform" },
        { "costCenter", "CC-ML-001" }
    }
};

instance = await instance.UpdateAsync(patch);
Console.WriteLine($"Updated instance: {instance.Data.Name}");
```

### 6. Xóa Instance

```csharp
WeightsAndBiasesInstanceResource instance = await resourceGroup
    .GetWeightsAndBiasesInstanceAsync("my-wandb-instance");

await instance.DeleteAsync(WaitUntil.Completed);
Console.WriteLine("Instance deleted");
```

### 7. Kiểm tra Tính khả dụng của Tên Tài nguyên

```csharp
// Kiểm tra xem tên có khả dụng không trước khi tạo
// (Thực hiện thông qua lệnh gọi trực tiếp ARM nếu SDK không cung cấp chức năng này)
try
{
    await resourceGroup.GetWeightsAndBiasesInstanceAsync("desired-name");
    Console.WriteLine("Name is already taken");
}
catch (RequestFailedException ex) when (ex.Status == 404)
{
    Console.WriteLine("Name is available");
}
```

## Tham khảo Các Loại Chính

| Loại                                     | Mục đích                      |
| ---------------------------------------- | ----------------------------- |
| `WeightsAndBiasesInstanceResource`       | Tài nguyên instance W&B       |
| `WeightsAndBiasesInstanceData`           | Dữ liệu cấu hình instance     |
| `WeightsAndBiasesInstanceCollection`     | Collection các instance       |
| `WeightsAndBiasesInstanceProperties`     | Các thuộc tính instance       |
| `WeightsAndBiasesMarketplaceDetails`     | Thông tin đăng ký Marketplace |
| `WeightsAndBiasesOfferDetails`           | Chi tiết offer Marketplace    |
| `WeightsAndBiasesUserDetails`            | Thông tin người dùng Admin    |
| `WeightsAndBiasesPartnerProperties`      | Cấu hình cụ thể cho W&B       |
| `WeightsAndBiasSingleSignOnPropertiesV2` | Cấu hình SSO                  |
| `WeightsAndBiasesInstancePatch`          | Patch cho các cập nhật        |
| `WeightsAndBiasesRegion`                 | Enum các vùng được hỗ trợ     |

## Các Vùng Khả dụng

| Region Enum                           | Vùng Azure    |
| ------------------------------------- | ------------- |
| `WeightsAndBiasesRegion.EastUS`       | East US       |
| `WeightsAndBiasesRegion.CentralUS`    | Central US    |
| `WeightsAndBiasesRegion.WestUS`       | West US       |
| `WeightsAndBiasesRegion.WestEurope`   | West Europe   |
| `WeightsAndBiasesRegion.JapanEast`    | Japan East    |
| `WeightsAndBiasesRegion.KoreaCentral` | Korea Central |

## Chi tiết Offer Marketplace

Cho tích hợp Azure Marketplace:

| Thuộc tính   | Giá trị                      |
| ------------ | ---------------------------- |
| Publisher ID | `wandb`                      |
| Offer ID     | `wandb-pay-as-you-go`        |
| Plan ID      | `wandb-payg` (Pay As You Go) |

## Thực hành Tốt nhất

1. **Sử dụng DefaultAzureCredential** — Tự động hỗ trợ nhiều phương thức xác thực
2. **Bật managed identity** — Để truy cập an toàn vào các tài nguyên Azure khác
3. **Cấu hình SSO** — Bật Entra ID SSO cho bảo mật doanh nghiệp
4. **Gắn thẻ tài nguyên** — Sử dụng tags để theo dõi chi phí và tổ chức
5. **Kiểm tra trạng thái cung cấp** — Chờ `Succeeded` trước khi sử dụng instance
6. **Sử dụng vùng phù hợp** — Chọn vùng gần nhất với compute của bạn
7. **Giám sát với Azure** — Sử dụng Azure Monitor cho sức khỏe tài nguyên

## Xử lý Lỗi

```csharp
using Azure;

try
{
    ArmOperation<WeightsAndBiasesInstanceResource> operation = await instances
        .CreateOrUpdateAsync(WaitUntil.Completed, "my-wandb", data);
}
catch (RequestFailedException ex) when (ex.Status == 409)
{
    Console.WriteLine("Instance already exists or name conflict");
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

## Tích hợp với W&B SDK

Sau khi tạo tài nguyên Azure, sử dụng W&B Python SDK để theo dõi thử nghiệm:

```python
# Cài đặt: pip install wandb
import wandb

# Đăng nhập với API key W&B của bạn từ instance đã triển khai trên Azure
wandb.login(host="https://my-company-wandb.wandb.ai")

# Khởi tạo một run
run = wandb.init(project="my-ml-project")

# Log số liệu
wandb.log({"accuracy": 0.95, "loss": 0.05})

# Kết thúc run
run.finish()
```

## Các SDK Liên quan

| SDK                                      | Mục đích                       | Cài đặt                                                                  |
| ---------------------------------------- | ------------------------------ | ------------------------------------------------------------------------ |
| `Azure.ResourceManager.WeightsAndBiases` | Quản lý instance W&B (SDK này) | `dotnet add package Azure.ResourceManager.WeightsAndBiases --prerelease` |
| `Azure.ResourceManager.MachineLearning`  | Azure ML workspaces            | `dotnet add package Azure.ResourceManager.MachineLearning`               |

## Liên kết Tham khảo

| Tài nguyên        | URL                                                                               |
| ----------------- | --------------------------------------------------------------------------------- |
| Gói NuGet         | https://www.nuget.org/packages/Azure.ResourceManager.WeightsAndBiases             |
| W&B Documentation | https://docs.wandb.ai/                                                            |
| Azure Marketplace | https://azuremarketplace.microsoft.com/marketplace/apps/wandb.wandb-pay-as-you-go |
| GitHub Source     | https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/weightsandbiases         |
