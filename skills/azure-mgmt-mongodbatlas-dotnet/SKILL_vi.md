---
name: azure-mgmt-mongodbatlas-dotnet
description: Quản lý MongoDB Atlas Organizations như là tài nguyên Azure ARM bằng Azure.ResourceManager.MongoDBAtlas SDK. Sử dụng khi tạo, cập nhật, liệt kê, hoặc xóa các tổ chức MongoDB Atlas thông qua tích hợp Azure Marketplace. SDK này quản lý tài nguyên tổ chức phía Azure, không quản lý clusters/databases Atlas trực tiếp.
package: Azure.ResourceManager.MongoDBAtlas
---

# Azure.ResourceManager.MongoDBAtlas SDK

Quản lý MongoDB Atlas Organizations như là tài nguyên Azure ARM với hóa đơn thống nhất thông qua Azure Marketplace.

## Thông tin Gói

| Thuộc tính      | Giá trị                                                                                                 |
| --------------- | ------------------------------------------------------------------------------------------------------- |
| Gói             | `Azure.ResourceManager.MongoDBAtlas`                                                                    |
| Phiên bản       | 1.0.0 (GA)                                                                                              |
| Phiên bản API   | 2025-06-01                                                                                              |
| Loại Tài nguyên | `MongoDB.Atlas/organizations`                                                                           |
| NuGet           | [Azure.ResourceManager.MongoDBAtlas](https://www.nuget.org/packages/Azure.ResourceManager.MongoDBAtlas) |

## Cài đặt

```bash
dotnet add package Azure.ResourceManager.MongoDBAtlas
dotnet add package Azure.Identity
dotnet add package Azure.ResourceManager
```

## Giới hạn Phạm vi Quan trọng

SDK này quản lý **MongoDB Atlas Organizations như là tài nguyên Azure ARM** để tích hợp marketplace. Nó KHÔNG trực tiếp quản lý:

- Atlas clusters
- Databases
- Collections
- Users/roles

Để quản lý cluster, hãy sử dụng MongoDB Atlas API trực tiếp sau khi tạo tổ chức.

## Authentication

```csharp
using Azure.Identity;
using Azure.ResourceManager;
using Azure.ResourceManager.MongoDBAtlas;
using Azure.ResourceManager.MongoDBAtlas.Models;

// Tạo ARM client với DefaultAzureCredential
var credential = new DefaultAzureCredential();
var armClient = new ArmClient(credential);
```

## Các Loại Cốt lõi

| Loại                                 | Mục đích                                        |
| ------------------------------------ | ----------------------------------------------- |
| `MongoDBAtlasOrganizationResource`   | Tài nguyên ARM đại diện cho một tổ chức Atlas   |
| `MongoDBAtlasOrganizationCollection` | Collection các tổ chức trong một resource group |
| `MongoDBAtlasOrganizationData`       | Mô hình dữ liệu cho tài nguyên tổ chức          |
| `MongoDBAtlasOrganizationProperties` | Các thuộc tính đặc thù của tổ chức              |
| `MongoDBAtlasMarketplaceDetails`     | Chi tiết đăng ký Azure Marketplace              |
| `MongoDBAtlasOfferDetails`           | Cấu hình offer Marketplace                      |
| `MongoDBAtlasUserDetails`            | Thông tin người dùng cho tổ chức                |
| `MongoDBAtlasPartnerProperties`      | Các thuộc tính MongoDB cụ thể (tên org, ID)     |

## Quy trình Làm việc

### Lấy Organization Collection

```csharp
// Lấy resource group
var subscription = await armClient.GetDefaultSubscriptionAsync();
var resourceGroup = await subscription.GetResourceGroupAsync("my-resource-group");

// Lấy organizations collection
MongoDBAtlasOrganizationCollection organizations =
    resourceGroup.Value.GetMongoDBAtlasOrganizations();
```

### Tạo Organization

```csharp
var organizationName = "my-atlas-org";
var location = AzureLocation.EastUS2;

// Xây dựng dữ liệu tổ chức
var organizationData = new MongoDBAtlasOrganizationData(location)
{
    Properties = new MongoDBAtlasOrganizationProperties(
        marketplace: new MongoDBAtlasMarketplaceDetails(
            subscriptionId: "your-azure-subscription-id",
            offerDetails: new MongoDBAtlasOfferDetails(
                publisherId: "mongodb",
                offerId: "mongodb_atlas_azure_native_prod",
                planId: "private_plan",
                planName: "Pay as You Go (Free) (Private)",
                termUnit: "P1M",
                termId: "gmz7xq9ge3py"
            )
        ),
        user: new MongoDBAtlasUserDetails(
            emailAddress: "admin@example.com",
            upn: "admin@example.com"
        )
        {
            FirstName = "Admin",
            LastName = "User"
        }
    )
    {
        PartnerProperties = new MongoDBAtlasPartnerProperties
        {
            OrganizationName = organizationName
        }
    },
    Tags = { ["Environment"] = "Production" }
};

// Tạo tổ chức (thao tác chạy lâu)
var operation = await organizations.CreateOrUpdateAsync(
    WaitUntil.Completed,
    organizationName,
    organizationData
);

MongoDBAtlasOrganizationResource organization = operation.Value;
Console.WriteLine($"Created: {organization.Id}");
```

### Lấy Organization Hiện có

```csharp
// Tùy chọn 1: Từ collection
MongoDBAtlasOrganizationResource org =
    await organizations.GetAsync("my-atlas-org");

// Tùy chọn 2: Từ định danh tài nguyên
var resourceId = MongoDBAtlasOrganizationResource.CreateResourceIdentifier(
    subscriptionId: "subscription-id",
    resourceGroupName: "my-resource-group",
    organizationName: "my-atlas-org"
);
MongoDBAtlasOrganizationResource org2 =
    armClient.GetMongoDBAtlasOrganizationResource(resourceId);
await org2.GetAsync(); // Lấy dữ liệu
```

### Liệt kê Organizations

```csharp
// Liệt kê trong resource group
await foreach (var org in organizations.GetAllAsync())
{
    Console.WriteLine($"Org: {org.Data.Name}");
    Console.WriteLine($"  Location: {org.Data.Location}");
    Console.WriteLine($"  State: {org.Data.Properties?.ProvisioningState}");
}

// Liệt kê trên toàn subscription
await foreach (var org in subscription.GetMongoDBAtlasOrganizationsAsync())
{
    Console.WriteLine($"Org: {org.Data.Name} in {org.Data.Id}");
}
```

### Cập nhật Tags

```csharp
// Thêm một tag đơn lẻ
await organization.AddTagAsync("CostCenter", "12345");

// Thay thế tất cả tags
await organization.SetTagsAsync(new Dictionary<string, string>
{
    ["Environment"] = "Production",
    ["Team"] = "Platform"
});

// Xóa một tag
await organization.RemoveTagAsync("OldTag");
```

### Cập nhật Thuộc tính Organization

```csharp
var patch = new MongoDBAtlasOrganizationPatch
{
    Tags = { ["UpdatedAt"] = DateTime.UtcNow.ToString("o") },
    Properties = new MongoDBAtlasOrganizationUpdateProperties
    {
        // Cập nhật chi tiết người dùng nếu cần
        User = new MongoDBAtlasUserDetails(
            emailAddress: "newadmin@example.com",
            upn: "newadmin@example.com"
        )
    }
};

var updateOperation = await organization.UpdateAsync(
    WaitUntil.Completed,
    patch
);
```

### Xóa Organization

```csharp
// Xóa (thao tác chạy lâu)
await organization.DeleteAsync(WaitUntil.Completed);
```

## Tham khảo Thuộc tính Model

### MongoDBAtlasOrganizationProperties

| Thuộc tính          | Loại                                    | Mô tả                                  |
| ------------------- | --------------------------------------- | -------------------------------------- |
| `Marketplace`       | `MongoDBAtlasMarketplaceDetails`        | Bắt buộc. Chi tiết đăng ký Marketplace |
| `User`              | `MongoDBAtlasUserDetails`               | Bắt buộc. Người dùng quản trị tổ chức  |
| `PartnerProperties` | `MongoDBAtlasPartnerProperties`         | Các thuộc tính cụ thể của MongoDB      |
| `ProvisioningState` | `MongoDBAtlasResourceProvisioningState` | Chỉ đọc. Trạng thái cung cấp hiện tại  |

### MongoDBAtlasMarketplaceDetails

| Thuộc tính           | Loại                            | Mô tả                                         |
| -------------------- | ------------------------------- | --------------------------------------------- |
| `SubscriptionId`     | `string`                        | Bắt buộc. Azure subscription ID để thanh toán |
| `OfferDetails`       | `MongoDBAtlasOfferDetails`      | Bắt buộc. Cấu hình offer Marketplace          |
| `SubscriptionStatus` | `MarketplaceSubscriptionStatus` | Chỉ đọc. Trạng thái đăng ký                   |

### MongoDBAtlasOfferDetails

| Thuộc tính    | Loại     | Mô tả                                             |
| ------------- | -------- | ------------------------------------------------- |
| `PublisherId` | `string` | Bắt buộc. Publisher ID (thường là "mongodb")      |
| `OfferId`     | `string` | Bắt buộc. Offer ID                                |
| `PlanId`      | `string` | Bắt buộc. Plan ID                                 |
| `PlanName`    | `string` | Bắt buộc. Tên hiển thị của gói                    |
| `TermUnit`    | `string` | Bắt buộc. Đơn vị kỳ hạn thanh toán (ví dụ: "P1M") |
| `TermId`      | `string` | Bắt buộc. Định danh kỳ hạn                        |

### MongoDBAtlasUserDetails

| Thuộc tính     | Loại     | Mô tả                              |
| -------------- | -------- | ---------------------------------- |
| `EmailAddress` | `string` | Bắt buộc. Địa chỉ email người dùng |
| `Upn`          | `string` | Bắt buộc. User principal name      |
| `FirstName`    | `string` | Tùy chọn. Tên người dùng           |
| `LastName`     | `string` | Tùy chọn. Họ người dùng            |

### MongoDBAtlasPartnerProperties

| Thuộc tính         | Loại     | Mô tả                                  |
| ------------------ | -------- | -------------------------------------- |
| `OrganizationName` | `string` | Tên của MongoDB Atlas organization     |
| `OrganizationId`   | `string` | Chỉ đọc. MongoDB Atlas organization ID |

## Trạng thái Cung cấp

| Trạng thái     | Mô tả                                    |
| -------------- | ---------------------------------------- |
| `Succeeded`    | Tài nguyên được cung cấp thành công      |
| `Failed`       | Cung cấp thất bại                        |
| `Canceled`     | Cung cấp bị hủy                          |
| `Provisioning` | Tài nguyên đang được cung cấp            |
| `Updating`     | Tài nguyên đang được cập nhật            |
| `Deleting`     | Tài nguyên đang bị xóa                   |
| `Accepted`     | Yêu cầu được chấp nhận, bắt đầu cung cấp |

## Trạng thái Đăng ký Marketplace

| Trạng thái                | Mô tả                      |
| ------------------------- | -------------------------- |
| `PendingFulfillmentStart` | Đăng ký đang chờ kích hoạt |
| `Subscribed`              | Đăng ký đang hoạt động     |
| `Suspended`               | Đăng ký bị tạm ngưng       |
| `Unsubscribed`            | Đăng ký bị hủy             |

## Thực hành Tốt nhất

### Sử dụng Phương thức Async

```csharp
// Ưu tiên async cho tất cả các thao tác
var org = await organizations.GetAsync("my-org");
await org.Value.AddTagAsync("key", "value");
```

### Xử lý Thao tác Chạy lâu

```csharp
// Chờ hoàn thành
var operation = await organizations.CreateOrUpdateAsync(
    WaitUntil.Completed,  // Chặn cho đến khi xong
    name,
    data
);

// Hoặc bắt đầu và thăm dò sau
var operation = await organizations.CreateOrUpdateAsync(
    WaitUntil.Started,  // Trả về ngay lập tức
    name,
    data
);

// Thăm dò để hoàn thành
while (!operation.HasCompleted)
{
    await Task.Delay(TimeSpan.FromSeconds(5));
    await operation.UpdateStatusAsync();
}
```

### Kiểm tra Trạng thái Cung cấp

```csharp
var org = await organizations.GetAsync("my-org");
if (org.Value.Data.Properties?.ProvisioningState ==
    MongoDBAtlasResourceProvisioningState.Succeeded)
{
    Console.WriteLine("Organization is ready");
}
```

### Sử dụng Định danh Tài nguyên

```csharp
// Tạo định danh mà không cần gọi API
var resourceId = MongoDBAtlasOrganizationResource.CreateResourceIdentifier(
    subscriptionId,
    resourceGroupName,
    organizationName
);

// Lấy handle tài nguyên (chưa có dữ liệu)
var orgResource = armClient.GetMongoDBAtlasOrganizationResource(resourceId);

// Lấy dữ liệu khi cần
var response = await orgResource.GetAsync();
```

## Các Lỗi Thường gặp

| Lỗi                   | Nguyên nhân                   | Giải pháp                                            |
| --------------------- | ----------------------------- | ---------------------------------------------------- |
| `ResourceNotFound`    | Organization không tồn tại    | Xác minh tên và resource group                       |
| `AuthorizationFailed` | Không đủ quyền                | Kiểm tra các vai trò RBAC trên resource group        |
| `InvalidParameter`    | Thiếu các thuộc tính bắt buộc | Đảm bảo tất cả các trường bắt buộc đã được thiết lập |
| `MarketplaceError`    | Vấn đề đăng ký Marketplace    | Xác minh chi tiết offer và subscription              |

## Tài nguyên Liên quan

- [Microsoft Learn: MongoDB Atlas on Azure](https://learn.microsoft.com/en-us/azure/partner-solutions/mongodb-atlas/)
- [Tham khảo API](https://learn.microsoft.com/en-us/dotnet/api/azure.resourcemanager.mongodbatlas)
- [Azure SDK for .NET](https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/mongodbatlas)
