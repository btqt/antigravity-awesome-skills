---
name: azure-mgmt-fabric-py
description: |
  Azure Fabric Management SDK cho Python. Sử dụng để quản lý Microsoft Fabric capacities và tài nguyên.
  Triggers: "azure-mgmt-fabric", "FabricMgmtClient", "Fabric capacity", "Microsoft Fabric", "Power BI capacity".
---

# Azure Fabric Management SDK cho Python

Quản lý Microsoft Fabric capacities và tài nguyên bằng lập trình.

## Cài đặt

```bash
pip install azure-mgmt-fabric
pip install azure-identity
```

## Biến Môi trường

```bash
AZURE_SUBSCRIPTION_ID=<your-subscription-id>
AZURE_RESOURCE_GROUP=<your-resource-group>
```

## Xác thực

```python
from azure.identity import DefaultAzureCredential
from azure.mgmt.fabric import FabricMgmtClient
import os

credential = DefaultAzureCredential()
client = FabricMgmtClient(
    credential=credential,
    subscription_id=os.environ["AZURE_SUBSCRIPTION_ID"]
)
```

## Tạo Fabric Capacity

```python
from azure.mgmt.fabric import FabricMgmtClient
from azure.mgmt.fabric.models import FabricCapacity, FabricCapacityProperties, CapacitySku
from azure.identity import DefaultAzureCredential
import os

credential = DefaultAzureCredential()
client = FabricMgmtClient(
    credential=credential,
    subscription_id=os.environ["AZURE_SUBSCRIPTION_ID"]
)

resource_group = os.environ["AZURE_RESOURCE_GROUP"]
capacity_name = "myfabriccapacity"

capacity = client.fabric_capacities.begin_create_or_update(
    resource_group_name=resource_group,
    capacity_name=capacity_name,
    resource=FabricCapacity(
        location="eastus",
        sku=CapacitySku(
            name="F2",  # Fabric SKU
            tier="Fabric"
        ),
        properties=FabricCapacityProperties(
            administration=FabricCapacityAdministration(
                members=["user@contoso.com"]
            )
        )
    )
).result()

print(f"Capacity created: {capacity.name}")
```

## Lấy Chi tiết Capacity

```python
capacity = client.fabric_capacities.get(
    resource_group_name=resource_group,
    capacity_name=capacity_name
)

print(f"Capacity: {capacity.name}")
print(f"SKU: {capacity.sku.name}")
print(f"State: {capacity.properties.state}")
print(f"Location: {capacity.location}")
```

## Liệt kê Capacities trong Resource Group

```python
capacities = client.fabric_capacities.list_by_resource_group(
    resource_group_name=resource_group
)

for capacity in capacities:
    print(f"Capacity: {capacity.name} - SKU: {capacity.sku.name}")
```

## Liệt kê Tất cả Capacities trong Subscription

```python
all_capacities = client.fabric_capacities.list_by_subscription()

for capacity in all_capacities:
    print(f"Capacity: {capacity.name} in {capacity.location}")
```

## Cập nhật Capacity

```python
from azure.mgmt.fabric.models import FabricCapacityUpdate, CapacitySku

updated = client.fabric_capacities.begin_update(
    resource_group_name=resource_group,
    capacity_name=capacity_name,
    properties=FabricCapacityUpdate(
        sku=CapacitySku(
            name="F4",  # Scale up
            tier="Fabric"
        ),
        tags={"environment": "production"}
    )
).result()

print(f"Updated SKU: {updated.sku.name}")
```

## Tạm dừng Capacity

Tạm dừng capacity để ngừng tính phí:

```python
client.fabric_capacities.begin_suspend(
    resource_group_name=resource_group,
    capacity_name=capacity_name
).result()

print("Capacity suspended")
```

## Tiếp tục Capacity

Tiếp tục một capacity đang tạm dừng:

```python
client.fabric_capacities.begin_resume(
    resource_group_name=resource_group,
    capacity_name=capacity_name
).result()

print("Capacity resumed")
```

## Xóa Capacity

```python
client.fabric_capacities.begin_delete(
    resource_group_name=resource_group,
    capacity_name=capacity_name
).result()

print("Capacity deleted")
```

## Kiểm tra Tính khả dụng của Tên

```python
from azure.mgmt.fabric.models import CheckNameAvailabilityRequest

result = client.fabric_capacities.check_name_availability(
    location="eastus",
    body=CheckNameAvailabilityRequest(
        name="my-new-capacity",
        type="Microsoft.Fabric/capacities"
    )
)

if result.name_available:
    print("Name is available")
else:
    print(f"Name not available: {result.reason}")
```

## Liệt kê SKUs Có sẵn

```python
skus = client.fabric_capacities.list_skus(
    resource_group_name=resource_group,
    capacity_name=capacity_name
)

for sku in skus:
    print(f"SKU: {sku.name} - Tier: {sku.tier}")
```

## Các Thao tác Client

| Thao tác                   | Phương thức                   |
| -------------------------- | ----------------------------- |
| `client.fabric_capacities` | Thao tác CRUD Capacity        |
| `client.operations`        | Liệt kê các thao tác khả dụng |

## Fabric SKUs

| SKU     | Mô tả       | CUs                 |
| ------- | ----------- | ------------------- |
| `F2`    | Entry level | 2 Capacity Units    |
| `F4`    | Small       | 4 Capacity Units    |
| `F8`    | Medium      | 8 Capacity Units    |
| `F16`   | Large       | 16 Capacity Units   |
| `F32`   | X-Large     | 32 Capacity Units   |
| `F64`   | 2X-Large    | 64 Capacity Units   |
| `F128`  | 4X-Large    | 128 Capacity Units  |
| `F256`  | 8X-Large    | 256 Capacity Units  |
| `F512`  | 16X-Large   | 512 Capacity Units  |
| `F1024` | 32X-Large   | 1024 Capacity Units |
| `F2048` | 64X-Large   | 2048 Capacity Units |

## Trạng thái Capacity

| Trạng thái     | Mô tả                                 |
| -------------- | ------------------------------------- |
| `Active`       | Capacity đang chạy                    |
| `Paused`       | Capacity bị tạm dừng (không tính phí) |
| `Provisioning` | Đang được tạo                         |
| `Updating`     | Đang được chỉnh sửa                   |
| `Deleting`     | Đang bị xóa                           |
| `Failed`       | Thao tác thất bại                     |

## Thao tác Chạy lâu (Long-Running Operations)

Tất cả các thao tác thay đổi trạng thái đều chạy lâu (LRO). Sử dụng `.result()` để chờ:

```python
# Chờ đồng bộ
capacity = client.fabric_capacities.begin_create_or_update(...).result()

# Hoặc thăm dò thủ công
poller = client.fabric_capacities.begin_create_or_update(...)
while not poller.done():
    print(f"Status: {poller.status()}")
    time.sleep(5)
capacity = poller.result()
```

## Thực hành Tốt nhất

1. **Sử dụng DefaultAzureCredential** cho xác thực
2. **Tạm dừng các capacity không sử dụng** để giảm chi phí
3. **Bắt đầu với SKU nhỏ hơn** và mở rộng khi cần thiết
4. **Sử dụng tags** để theo dõi chi phí và tổ chức
5. **Kiểm tra tính khả dụng của tên** trước khi tạo capacities
6. **Xử lý LRO đúng cách** — đừng giả định hoàn thành ngay lập tức
7. **Thiết lập admin cho capacity** — chỉ định người dùng có thể quản lý workspaces
8. **Giám sát sử dụng capacity** thông qua số liệu Azure Monitor
