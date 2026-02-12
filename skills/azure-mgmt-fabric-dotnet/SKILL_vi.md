---
name: azure-mgmt-fabric-dotnet
description: |
  Azure Resource Manager SDK cho Fabric trong .NET. Sử dụng cho các thao tác MANAGEMENT PLANE: cung cấp (provisioning), mở rộng (scaling), tạm dừng/tiếp tục (suspending/resuming) các capacity của Microsoft Fabric, kiểm tra tính khả dụng của tên, và liệt kê SKUs thông qua Azure Resource Manager. Triggers: "Fabric capacity", "create capacity", "suspend capacity", "resume capacity", "Fabric SKU", "provision Fabric", "ARM Fabric", "FabricCapacityResource".
package: Azure.ResourceManager.Fabric
---

# Azure.ResourceManager.Fabric (.NET)

SDK management plane để cung cấp và quản lý tài nguyên Microsoft Fabric capacity thông qua Azure Resource Manager.

> **Management Plane Only**
> SDK này quản lý Fabric _capacities_ (tài nguyên tính toán/compute). Để làm việc với Fabric workspaces, lakehouses, warehouses, và các mục dữ liệu (data items), hãy sử dụng Microsoft Fabric REST API hoặc data plane SDKs.

## Cài đặt

```bash
dotnet add package Azure.ResourceManager.Fabric
dotnet add package Azure.Identity
```

**Phiên bản Hiện tại**: 1.0.0 (GA - Tháng 9/2025)  
**Phiên bản API**: 2023-11-01  
**Target Frameworks**: .NET 8.0, .NET Standard 2.0

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
using Azure.ResourceManager.Fabric;

// Luôn sử dụng DefaultAzureCredential
var credential = new DefaultAzureCredential();
var armClient = new ArmClient(credential);

// Lấy subscription
var subscription = await armClient.GetDefaultSubscriptionAsync();
```

## Phân cấp Tài nguyên

```
ArmClient
└── SubscriptionResource
    └── ResourceGroupResource
        └── FabricCapacityResource
```

## Quy trình Làm việc Cốt lõi

### 1. Tạo Fabric Capacity

```csharp
using Azure.ResourceManager.Fabric;
using Azure.ResourceManager.Fabric.Models;
using Azure.Core;

// Lấy resource group
var resourceGroup = await subscription.GetResourceGroupAsync("my-resource-group");

// Định nghĩa cấu hình capacity
var administration = new FabricCapacityAdministration(
    new[] { "admin@contoso.com" }  // Capacity administrators (UPNs hoặc object IDs)
);

var properties = new FabricCapacityProperties(administration);

var sku = new FabricSku("F64", FabricSkuTier.Fabric);

var capacityData = new FabricCapacityData(
    AzureLocation.WestUS2,
    properties,
    sku)
{
    Tags = { ["Environment"] = "Production" }
};

// Tạo capacity (thao tác chạy lâu)
var capacityCollection = resourceGroup.Value.GetFabricCapacities();
var operation = await capacityCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "my-fabric-capacity",
    capacityData);

FabricCapacityResource capacity = operation.Value;
Console.WriteLine($"Created capacity: {capacity.Data.Name}");
Console.WriteLine($"State: {capacity.Data.Properties.State}");
```

### 2. Lấy Fabric Capacity

```csharp
// Lấy capacity hiện có
var capacity = await resourceGroup.Value
    .GetFabricCapacityAsync("my-fabric-capacity");

Console.WriteLine($"Name: {capacity.Value.Data.Name}");
Console.WriteLine($"Location: {capacity.Value.Data.Location}");
Console.WriteLine($"SKU: {capacity.Value.Data.Sku.Name}");
Console.WriteLine($"State: {capacity.Value.Data.Properties.State}");
Console.WriteLine($"Provisioning State: {capacity.Value.Data.Properties.ProvisioningState}");
```

### 3. Cập nhật Capacity (Mở rộng SKU hoặc Thay đổi Admin)

```csharp
var capacity = await resourceGroup.Value
    .GetFabricCapacityAsync("my-fabric-capacity");

var patch = new FabricCapacityPatch
{
    Sku = new FabricSku("F128", FabricSkuTier.Fabric),  // Scale up
    Properties = new FabricCapacityUpdateProperties
    {
        Administration = new FabricCapacityAdministration(
            new[] { "admin@contoso.com", "newadmin@contoso.com" }
        )
    }
};

var updateOperation = await capacity.Value.UpdateAsync(
    WaitUntil.Completed,
    patch);

Console.WriteLine($"Updated SKU: {updateOperation.Value.Data.Sku.Name}");
```

### 4. Tạm dừng và Tiếp tục Capacity

```csharp
// Tạm dừng capacity (ngừng tính phí compute)
await capacity.Value.SuspendAsync(WaitUntil.Completed);
Console.WriteLine("Capacity suspended");

// Tiếp tục capacity
var resumeOperation = await capacity.Value.ResumeAsync(WaitUntil.Completed);
Console.WriteLine($"Capacity resumed. State: {resumeOperation.Value.Data.Properties.State}");
```

### 5. Xóa Capacity

```csharp
await capacity.Value.DeleteAsync(WaitUntil.Completed);
Console.WriteLine("Capacity deleted");
```

### 6. Liệt kê Tất cả Capacities

```csharp
// Trong một resource group
await foreach (var cap in resourceGroup.Value.GetFabricCapacities())
{
    Console.WriteLine($"- {cap.Data.Name} ({cap.Data.Sku.Name})");
}

// Trong một subscription
await foreach (var cap in subscription.GetFabricCapacitiesAsync())
{
    Console.WriteLine($"- {cap.Data.Name} in {cap.Data.Location}");
}
```

### 7. Kiểm tra Tính khả dụng của Tên

```csharp
var checkContent = new FabricNameAvailabilityContent
{
    Name = "my-new-capacity",
    ResourceType = "Microsoft.Fabric/capacities"
};

var result = await subscription.CheckFabricCapacityNameAvailabilityAsync(
    AzureLocation.WestUS2,
    checkContent);

if (result.Value.IsNameAvailable == true)
{
    Console.WriteLine("Name is available!");
}
else
{
    Console.WriteLine($"Name unavailable: {result.Value.Reason} - {result.Value.Message}");
}
```

### 8. Liệt kê SKUs Có sẵn

```csharp
// Liệt kê tất cả SKUs có sẵn trong subscription
await foreach (var skuDetails in subscription.GetSkusFabricCapacitiesAsync())
{
    Console.WriteLine($"SKU: {skuDetails.Name}");
    Console.WriteLine($"  Resource Type: {skuDetails.ResourceType}");
    foreach (var location in skuDetails.Locations)
    {
        Console.WriteLine($"  Location: {location}");
    }
}

// Liệt kê SKUs có sẵn cho một capacity hiện có (để scaling)
await foreach (var skuDetails in capacity.Value.GetSkusForCapacityAsync())
{
    Console.WriteLine($"Can scale to: {skuDetails.Sku.Name}");
}
```

## Tham khảo SKU

| Tên SKU | Capacity Units (CU) | Tương đương Power BI |
| ------- | ------------------- | -------------------- |
| F2      | 2                   | -                    |
| F4      | 4                   | -                    |
| F8      | 8                   | EM1/A1               |
| F16     | 16                  | EM2/A2               |
| F32     | 32                  | EM3/A3               |
| F64     | 64                  | P1/A4                |
| F128    | 128                 | P2/A5                |
| F256    | 256                 | P3/A6                |
| F512    | 512                 | P4/A7                |
| F1024   | 1024                | P5/A8                |
| F2048   | 2048                | -                    |

## Tham khảo Các Loại Chính

| Loại                            | Mục đích                                        |
| ------------------------------- | ----------------------------------------------- |
| `ArmClient`                     | Điểm vào cho tất cả các thao tác ARM            |
| `FabricCapacityResource`        | Đại diện cho một instance Fabric capacity       |
| `FabricCapacityCollection`      | Collection cho các thao tác CRUD capacity       |
| `FabricCapacityData`            | Mô hình dữ liệu tạo/đọc capacity                |
| `FabricCapacityPatch`           | Payload cập nhật capacity                       |
| `FabricCapacityProperties`      | Các thuộc tính capacity (quản trị, trạng thái)  |
| `FabricCapacityAdministration`  | Cấu hình thành viên quản trị                    |
| `FabricSku`                     | Cấu hình SKU (tên và tier)                      |
| `FabricSkuTier`                 | Pricing tier (hiện tại chỉ có "Fabric")         |
| `FabricProvisioningState`       | Trạng thái cung cấp (Succeeded, Failed, etc.)   |
| `FabricResourceState`           | Trạng thái tài nguyên (Active, Suspended, etc.) |
| `FabricNameAvailabilityContent` | Yêu cầu kiểm tra tính khả dụng của tên          |
| `FabricNameAvailabilityResult`  | Phản hồi kiểm tra tính khả dụng của tên         |

## Trạng thái Cung cấp và Tài nguyên

### Trạng thái Cung cấp (`FabricProvisioningState`)

- `Succeeded` - Thao tác hoàn thành thành công
- `Failed` - Thao tác thất bại
- `Canceled` - Thao tác bị hủy
- `Deleting` - Capacity đang bị xóa
- `Provisioning` - Đang cung cấp ban đầu
- `Updating` - Thao tác cập nhật đang diễn ra

### Trạng thái Tài nguyên (`FabricResourceState`)

- `Active` - Capacity đang chạy và khả dụng
- `Provisioning` - Đang được cung cấp
- `Failed` - Ở trạng thái lỗi
- `Updating` - Đang được cập nhật
- `Deleting` - Đang bị xóa
- `Suspending` - Đang chuyển sang tạm dừng
- `Suspended` - Đã tạm dừng (không tính phí compute)
- `Pausing` - Đang chuyển sang tạm nghỉ (paused)
- `Paused` - Đã tạm nghỉ
- `Resuming` - Đang tiếp tục từ trạng thái suspended/paused
- `Scaling` - Đang mở rộng sang SKU khác
- `Preparing` - Đang chuẩn bị tài nguyên

## Thực hành Tốt nhất

1. **Sử dụng `WaitUntil.Completed`** cho các thao tác phải hoàn thành trước khi tiếp tục
2. **Sử dụng `WaitUntil.Started`** khi bạn muốn thăm dò thủ công hoặc chạy các thao tác song song
3. **Luôn sử dụng `DefaultAzureCredential`** — không bao giờ hardcode thông tin xác thực
4. **Xử lý `RequestFailedException`** cho các lỗi ARM API
5. **Sử dụng `CreateOrUpdateAsync`** cho các thao tác idempotent
6. **Tạm dừng khi không sử dụng** — Fabric capacities tính phí compute ngay cả khi nhàn rỗi
7. **Kiểm tra trạng thái cung cấp** trước khi thực hiện các thao tác trên một capacity
8. **Sử dụng SKU phù hợp** — Bắt đầu nhỏ (F2/F4) cho dev/test, mở rộng cho production

## Xử lý Lỗi

```csharp
using Azure;

try
{
    var operation = await capacityCollection.CreateOrUpdateAsync(
        WaitUntil.Completed, capacityName, capacityData);
}
catch (RequestFailedException ex) when (ex.Status == 409)
{
    Console.WriteLine("Capacity already exists or conflict");
}
catch (RequestFailedException ex) when (ex.Status == 400)
{
    Console.WriteLine($"Invalid configuration: {ex.Message}");
}
catch (RequestFailedException ex) when (ex.Status == 403)
{
    Console.WriteLine("Insufficient permissions or quota exceeded");
}
catch (RequestFailedException ex)
{
    Console.WriteLine($"ARM Error: {ex.Status} - {ex.ErrorCode}: {ex.Message}");
}
```

## Các Cạm bẫy Thường gặp

1. **Tên Capacity phải là duy nhất toàn cầu** — Tên Fabric capacity phải duy nhất trên tất cả các subscription Azure
2. **Tạm dừng không xóa** — Các capacity bị tạm dừng vẫn tồn tại nhưng không tính phí compute
3. **Thay đổi SKU có thể yêu cầu thời gian chết** — Các thao tác mở rộng có thể mất vài phút
4. **Admin UPNs phải hợp lệ** — Quản trị viên Capacity phải là người dùng Azure AD hợp lệ
5. **Ràng buộc vị trí** — Không phải tất cả SKU đều có sẵn ở mọi vùng; sử dụng `GetSkusFabricCapacitiesAsync` để kiểm tra
6. **Thời gian cung cấp dài** — Việc tạo capacity có thể mất 5-15 phút

## Các SDK Liên quan

| SDK                            | Mục đích                     | Cài đặt                                                |
| ------------------------------ | ---------------------------- | ------------------------------------------------------ |
| `Azure.ResourceManager.Fabric` | Management plane (SDK này)   | `dotnet add package Azure.ResourceManager.Fabric`      |
| `Microsoft.Fabric.Api`         | Data plane operations (beta) | `dotnet add package Microsoft.Fabric.Api --prerelease` |
| `Azure.ResourceManager`        | Core ARM SDK                 | `dotnet add package Azure.ResourceManager`             |
| `Azure.Identity`               | Authentication               | `dotnet add package Azure.Identity`                    |

## Tham khảo

- [Azure.ResourceManager.Fabric NuGet](https://www.nuget.org/packages/Azure.ResourceManager.Fabric)
- [Mã nguồn GitHub](https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/fabric/Azure.ResourceManager.Fabric)
- [Tài liệu Microsoft Fabric](https://learn.microsoft.com/fabric/)
- [Quản lý Fabric Capacity](https://learn.microsoft.com/fabric/admin/service-admin-portal-capacity-settings)
