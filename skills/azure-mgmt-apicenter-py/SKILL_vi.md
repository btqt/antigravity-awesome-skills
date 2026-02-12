---
name: azure-mgmt-apicenter-py
description: |
  Azure API Center Management SDK cho Python. Sử dụng để quản lý kho API, metadata, và quản trị trên toàn tổ chức của bạn.
  Triggers: "azure-mgmt-apicenter", "ApiCenterMgmtClient", "API Center", "API inventory", "API governance".
package: azure-mgmt-apicenter
---

# Azure API Center Management SDK cho Python

Quản lý kho API, metadata, và quản trị trong Azure API Center.

## Cài đặt

```bash
pip install azure-mgmt-apicenter
pip install azure-identity
```

## Biến Môi trường

```bash
AZURE_SUBSCRIPTION_ID=your-subscription-id
```

## Xác thực

```python
from azure.identity import DefaultAzureCredential
from azure.mgmt.apicenter import ApiCenterMgmtClient
import os

client = ApiCenterMgmtClient(
    credential=DefaultAzureCredential(),
    subscription_id=os.environ["AZURE_SUBSCRIPTION_ID"]
)
```

## Tạo API Center

```python
from azure.mgmt.apicenter.models import Service

api_center = client.services.create_or_update(
    resource_group_name="my-resource-group",
    service_name="my-api-center",
    resource=Service(
        location="eastus",
        tags={"environment": "production"}
    )
)

print(f"Created API Center: {api_center.name}")
```

## Liệt kê API Centers

```python
api_centers = client.services.list_by_subscription()

for api_center in api_centers:
    print(f"{api_center.name} - {api_center.location}")
```

## Đăng ký một API

```python
from azure.mgmt.apicenter.models import Api, ApiKind, LifecycleStage

api = client.apis.create_or_update(
    resource_group_name="my-resource-group",
    service_name="my-api-center",
    workspace_name="default",
    api_name="my-api",
    resource=Api(
        title="My API",
        description="A sample API for demonstration",
        kind=ApiKind.REST,
        lifecycle_stage=LifecycleStage.PRODUCTION,
        terms_of_service={"url": "https://example.com/terms"},
        contacts=[{"name": "API Team", "email": "api-team@example.com"}]
    )
)

print(f"Registered API: {api.title}")
```

## Tạo Phiên bản API

```python
from azure.mgmt.apicenter.models import ApiVersion, LifecycleStage

version = client.api_versions.create_or_update(
    resource_group_name="my-resource-group",
    service_name="my-api-center",
    workspace_name="default",
    api_name="my-api",
    version_name="v1",
    resource=ApiVersion(
        title="Version 1.0",
        lifecycle_stage=LifecycleStage.PRODUCTION
    )
)

print(f"Created version: {version.title}")
```

## Thêm Định nghĩa API

```python
from azure.mgmt.apicenter.models import ApiDefinition

definition = client.api_definitions.create_or_update(
    resource_group_name="my-resource-group",
    service_name="my-api-center",
    workspace_name="default",
    api_name="my-api",
    version_name="v1",
    definition_name="openapi",
    resource=ApiDefinition(
        title="OpenAPI Definition",
        description="OpenAPI 3.0 specification"
    )
)
```

## Import Đặc tả API

```python
from azure.mgmt.apicenter.models import ApiSpecImportRequest, ApiSpecImportSourceFormat

# Import từ nội dung inline
client.api_definitions.import_specification(
    resource_group_name="my-resource-group",
    service_name="my-api-center",
    workspace_name="default",
    api_name="my-api",
    version_name="v1",
    definition_name="openapi",
    body=ApiSpecImportRequest(
        format=ApiSpecImportSourceFormat.INLINE,
        value='{"openapi": "3.0.0", "info": {"title": "My API", "version": "1.0"}, "paths": {}}'
    )
)
```

## Liệt kê APIs

```python
apis = client.apis.list(
    resource_group_name="my-resource-group",
    service_name="my-api-center",
    workspace_name="default"
)

for api in apis:
    print(f"{api.name}: {api.title} ({api.kind})")
```

## Tạo Môi trường

```python
from azure.mgmt.apicenter.models import Environment, EnvironmentKind

environment = client.environments.create_or_update(
    resource_group_name="my-resource-group",
    service_name="my-api-center",
    workspace_name="default",
    environment_name="production",
    resource=Environment(
        title="Production",
        description="Production environment",
        kind=EnvironmentKind.PRODUCTION,
        server={"type": "Azure API Management", "management_portal_uri": ["https://portal.azure.com"]}
    )
)
```

## Tạo Deployment

```python
from azure.mgmt.apicenter.models import Deployment, DeploymentState

deployment = client.deployments.create_or_update(
    resource_group_name="my-resource-group",
    service_name="my-api-center",
    workspace_name="default",
    api_name="my-api",
    deployment_name="prod-deployment",
    resource=Deployment(
        title="Production Deployment",
        description="Deployed to production APIM",
        environment_id="/workspaces/default/environments/production",
        definition_id="/workspaces/default/apis/my-api/versions/v1/definitions/openapi",
        state=DeploymentState.ACTIVE,
        server={"runtime_uri": ["https://api.example.com"]}
    )
)
```

## Định nghĩa Metadata Tùy chỉnh

```python
from azure.mgmt.apicenter.models import MetadataSchema

metadata = client.metadata_schemas.create_or_update(
    resource_group_name="my-resource-group",
    service_name="my-api-center",
    metadata_schema_name="data-classification",
    resource=MetadataSchema(
        schema='{"type": "string", "title": "Data Classification", "enum": ["public", "internal", "confidential"]}'
    )
)
```

## Các Loại Client

| Client                | Mục đích                             |
| --------------------- | ------------------------------------ |
| `ApiCenterMgmtClient` | Client chính cho tất cả các thao tác |

## Các Thao tác

| Nhóm Thao tác      | Mục đích                      |
| ------------------ | ----------------------------- |
| `services`         | Quản lý dịch vụ API Center    |
| `workspaces`       | Quản lý Workspace             |
| `apis`             | Đăng ký và quản lý API        |
| `api_versions`     | Quản lý phiên bản API         |
| `api_definitions`  | Quản lý định nghĩa API        |
| `deployments`      | Theo dõi triển khai           |
| `environments`     | Quản lý môi trường            |
| `metadata_schemas` | Định nghĩa metadata tùy chỉnh |

## Thực hành Tốt nhất

1. **Sử dụng workspaces** để tổ chức các API theo nhóm hoặc domain
2. **Định nghĩa metadata schemas** để quản trị nhất quán
3. **Theo dõi triển khai** để hiểu APIs đang chạy ở đâu
4. **Import đặc tả** để kích hoạt phân tích và linting API
5. **Sử dụng các giai đoạn vòng đời** để theo dõi độ trưởng thành của API
6. **Thêm liên hệ (contacts)** để xác định quyền sở hữu và hỗ trợ API
