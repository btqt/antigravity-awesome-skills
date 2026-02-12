---
name: azure-mgmt-apimanagement-py
description: |
  Azure API Management SDK cho Python. Sử dụng để quản lý các dịch vụ APIM, APIs, products, subscriptions, và policies.
  Triggers: "azure-mgmt-apimanagement", "ApiManagementClient", "APIM", "API gateway", "API Management".
package: azure-mgmt-apimanagement
---

# Azure API Management SDK cho Python

Quản lý các dịch vụ Azure API Management, APIs, products, và policies.

## Cài đặt

```bash
pip install azure-mgmt-apimanagement
pip install azure-identity
```

## Biến Môi trường

```bash
AZURE_SUBSCRIPTION_ID=your-subscription-id
```

## Xác thực

```python
from azure.identity import DefaultAzureCredential
from azure.mgmt.apimanagement import ApiManagementClient
import os

client = ApiManagementClient(
    credential=DefaultAzureCredential(),
    subscription_id=os.environ["AZURE_SUBSCRIPTION_ID"]
)
```

## Tạo Dịch vụ APIM

```python
from azure.mgmt.apimanagement.models import (
    ApiManagementServiceResource,
    ApiManagementServiceSkuProperties,
    SkuType
)

service = client.api_management_service.begin_create_or_update(
    resource_group_name="my-resource-group",
    service_name="my-apim",
    parameters=ApiManagementServiceResource(
        location="eastus",
        publisher_email="admin@example.com",
        publisher_name="My Organization",
        sku=ApiManagementServiceSkuProperties(
            name=SkuType.DEVELOPER,
            capacity=1
        )
    )
).result()

print(f"Created APIM: {service.name}")
```

## Import API từ OpenAPI

```python
from azure.mgmt.apimanagement.models import (
    ApiCreateOrUpdateParameter,
    ContentFormat,
    Protocol
)

api = client.api.begin_create_or_update(
    resource_group_name="my-resource-group",
    service_name="my-apim",
    api_id="my-api",
    parameters=ApiCreateOrUpdateParameter(
        display_name="My API",
        path="myapi",
        protocols=[Protocol.HTTPS],
        format=ContentFormat.OPENAPI_JSON,
        value='{"openapi": "3.0.0", "info": {"title": "My API", "version": "1.0"}, "paths": {"/health": {"get": {"responses": {"200": {"description": "OK"}}}}}}'
    )
).result()

print(f"Imported API: {api.display_name}")
```

## Import API từ URL

```python
api = client.api.begin_create_or_update(
    resource_group_name="my-resource-group",
    service_name="my-apim",
    api_id="petstore",
    parameters=ApiCreateOrUpdateParameter(
        display_name="Petstore API",
        path="petstore",
        protocols=[Protocol.HTTPS],
        format=ContentFormat.OPENAPI_LINK,
        value="https://petstore.swagger.io/v2/swagger.json"
    )
).result()
```

## Liệt kê APIs

```python
apis = client.api.list_by_service(
    resource_group_name="my-resource-group",
    service_name="my-apim"
)

for api in apis:
    print(f"{api.name}: {api.display_name} - {api.path}")
```

## Tạo Product

```python
from azure.mgmt.apimanagement.models import ProductContract

product = client.product.create_or_update(
    resource_group_name="my-resource-group",
    service_name="my-apim",
    product_id="premium",
    parameters=ProductContract(
        display_name="Premium",
        description="Premium tier with unlimited access",
        subscription_required=True,
        approval_required=False,
        state="published"
    )
)

print(f"Created product: {product.display_name}")
```

## Thêm API vào Product

```python
client.product_api.create_or_update(
    resource_group_name="my-resource-group",
    service_name="my-apim",
    product_id="premium",
    api_id="my-api"
)
```

## Tạo Subscription

```python
from azure.mgmt.apimanagement.models import SubscriptionCreateParameters

subscription = client.subscription.create_or_update(
    resource_group_name="my-resource-group",
    service_name="my-apim",
    sid="my-subscription",
    parameters=SubscriptionCreateParameters(
        display_name="My Subscription",
        scope=f"/products/premium",
        state="active"
    )
)

print(f"Subscription key: {subscription.primary_key}")
```

## Thiết lập Chính sách API (API Policy)

```python
from azure.mgmt.apimanagement.models import PolicyContract

policy_xml = """
<policies>
    <inbound>
        <rate-limit calls="100" renewal-period="60" />
        <set-header name="X-Custom-Header" exists-action="override">
            <value>CustomValue</value>
        </set-header>
    </inbound>
    <backend>
        <forward-request />
    </backend>
    <outbound />
    <on-error />
</policies>
"""

client.api_policy.create_or_update(
    resource_group_name="my-resource-group",
    service_name="my-apim",
    api_id="my-api",
    policy_id="policy",
    parameters=PolicyContract(
        value=policy_xml,
        format="xml"
    )
)
```

## Tạo Named Value (Secret)

```python
from azure.mgmt.apimanagement.models import NamedValueCreateContract

named_value = client.named_value.begin_create_or_update(
    resource_group_name="my-resource-group",
    service_name="my-apim",
    named_value_id="backend-api-key",
    parameters=NamedValueCreateContract(
        display_name="Backend API Key",
        value="secret-key-value",
        secret=True
    )
).result()
```

## Tạo Backend

```python
from azure.mgmt.apimanagement.models import BackendContract

backend = client.backend.create_or_update(
    resource_group_name="my-resource-group",
    service_name="my-apim",
    backend_id="my-backend",
    parameters=BackendContract(
        url="https://api.backend.example.com",
        protocol="http",
        description="My backend service"
    )
)
```

## Tạo User

```python
from azure.mgmt.apimanagement.models import UserCreateParameters

user = client.user.create_or_update(
    resource_group_name="my-resource-group",
    service_name="my-apim",
    user_id="newuser",
    parameters=UserCreateParameters(
        email="user@example.com",
        first_name="John",
        last_name="Doe"
    )
)
```

## Các Nhóm Thao tác (Operation Groups)

| Nhóm                     | Mục đích              |
| ------------------------ | --------------------- |
| `api_management_service` | Quản lý instance APIM |
| `api`                    | Thao tác API          |
| `api_operation`          | Chi tiết thao tác API |
| `api_policy`             | Policy cấp API        |
| `product`                | Quản lý Product       |
| `product_api`            | Liên kết Product-API  |
| `subscription`           | Quản lý Subscription  |
| `user`                   | Quản lý User          |
| `named_value`            | Named values/secrets  |
| `backend`                | Dịch vụ Backend       |
| `certificate`            | Certificates          |
| `gateway`                | Self-hosted gateways  |

## Thực hành Tốt nhất

1. **Sử dụng named values** cho secrets và cấu hình
2. **Áp dụng policies** ở các phạm vi phù hợp (toàn cục, product, API, thao tác)
3. **Sử dụng products** để đóng gói APIs và quản lý truy cập
4. **Bật Application Insights** để giám sát
5. **Sử dụng backends** để trừu tượng hóa các dịch vụ backend
6. **Phiên bản hóa APIs** sử dụng các tính năng versioning của APIM
