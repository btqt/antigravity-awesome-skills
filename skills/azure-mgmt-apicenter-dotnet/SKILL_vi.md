---
name: azure-mgmt-apicenter-dotnet
description: |
  Azure API Center SDK cho .NET. Quản lý kho API tập trung với quản trị, lập phiên bản, và khám phá. Sử dụng để tạo các dịch vụ API, workspaces, APIs, versions, definitions, environments, deployments, và metadata schemas. Triggers: "API Center", "ApiCenterService", "ApiCenterWorkspace", "ApiCenterApi", "API inventory", "API governance", "API versioning", "API catalog", "API discovery".
package: Azure.ResourceManager.ApiCenter
---

# Azure.ResourceManager.ApiCenter (.NET)

SDK quản lý kho API tập trung và quản trị để quản lý các API trong tổ chức của bạn.

## Cài đặt

```bash
dotnet add package Azure.ResourceManager.ApiCenter
dotnet add package Azure.Identity
```

**Phiên bản Hiện tại**: v1.0.0 (GA)  
**Phiên bản API**: 2024-03-01

## Biến Môi trường

```bash
AZURE_SUBSCRIPTION_ID=<your-subscription-id>
AZURE_RESOURCE_GROUP=<your-resource-group>
AZURE_APICENTER_SERVICE_NAME=<your-apicenter-service>
```

## Xác thực

```csharp
using Azure.Identity;
using Azure.ResourceManager;
using Azure.ResourceManager.ApiCenter;

ArmClient client = new ArmClient(new DefaultAzureCredential());
```

## Phân cấp Tài nguyên

```
Subscription
└── ResourceGroup
    └── ApiCenterService                    # Dịch vụ kho API
        ├── Workspace                       # Nhóm logic các API
        │   ├── Api                         # Định nghĩa API
        │   │   └── ApiVersion              # Phiên bản của API
        │   │       └── ApiDefinition       # Đặc tả OpenAPI/GraphQL/etc
        │   ├── Environment                 # Mục tiêu triển khai (dev/staging/prod)
        │   └── Deployment                  # API được triển khai đến môi trường
        └── MetadataSchema                  # Định nghĩa metadata tùy chỉnh
```

## Các Quy trình Làm việc Cốt lõi

### 1. Tạo Dịch vụ API Center

```csharp
using Azure.ResourceManager.ApiCenter;
using Azure.ResourceManager.ApiCenter.Models;

ResourceGroupResource resourceGroup = await client
    .GetDefaultSubscriptionAsync()
    .Result
    .GetResourceGroupAsync("my-resource-group");

ApiCenterServiceCollection services = resourceGroup.GetApiCenterServices();

ApiCenterServiceData data = new ApiCenterServiceData(AzureLocation.EastUS)
{
    Identity = new ManagedServiceIdentity(ManagedServiceIdentityType.SystemAssigned)
};

ArmOperation<ApiCenterServiceResource> operation = await services
    .CreateOrUpdateAsync(WaitUntil.Completed, "my-api-center", data);

ApiCenterServiceResource service = operation.Value;
```

### 2. Tạo Workspace

```csharp
ApiCenterWorkspaceCollection workspaces = service.GetApiCenterWorkspaces();

ApiCenterWorkspaceData workspaceData = new ApiCenterWorkspaceData
{
    Title = "Engineering APIs",
    Description = "APIs owned by the engineering team"
};

ArmOperation<ApiCenterWorkspaceResource> operation = await workspaces
    .CreateOrUpdateAsync(WaitUntil.Completed, "engineering", workspaceData);

ApiCenterWorkspaceResource workspace = operation.Value;
```

### 3. Tạo API

```csharp
ApiCenterApiCollection apis = workspace.GetApiCenterApis();

ApiCenterApiData apiData = new ApiCenterApiData
{
    Title = "Orders API",
    Description = "API for managing customer orders",
    Kind = ApiKind.Rest,
    LifecycleStage = ApiLifecycleStage.Production,
    TermsOfService = new ApiTermsOfService
    {
        Uri = new Uri("https://example.com/terms")
    },
    ExternalDocumentation =
    {
        new ApiExternalDocumentation
        {
            Title = "Documentation",
            Uri = new Uri("https://docs.example.com/orders")
        }
    },
    Contacts =
    {
        new ApiContact
        {
            Name = "API Support",
            Email = "api-support@example.com"
        }
    }
};

// Thêm metadata tùy chỉnh
apiData.CustomProperties = BinaryData.FromObjectAsJson(new
{
    team = "orders-team",
    costCenter = "CC-1234"
});

ArmOperation<ApiCenterApiResource> operation = await apis
    .CreateOrUpdateAsync(WaitUntil.Completed, "orders-api", apiData);

ApiCenterApiResource api = operation.Value;
```

### 4. Tạo Phiên bản API

```csharp
ApiCenterApiVersionCollection versions = api.GetApiCenterApiVersions();

ApiCenterApiVersionData versionData = new ApiCenterApiVersionData
{
    Title = "v1.0.0",
    LifecycleStage = ApiLifecycleStage.Production
};

ArmOperation<ApiCenterApiVersionResource> operation = await versions
    .CreateOrUpdateAsync(WaitUntil.Completed, "v1-0-0", versionData);

ApiCenterApiVersionResource version = operation.Value;
```

### 5. Tạo Định nghĩa API (Tải lên Đặc tả OpenAPI)

```csharp
ApiCenterApiDefinitionCollection definitions = version.GetApiCenterApiDefinitions();

ApiCenterApiDefinitionData definitionData = new ApiCenterApiDefinitionData
{
    Title = "OpenAPI Specification",
    Description = "Orders API OpenAPI 3.0 definition"
};

ArmOperation<ApiCenterApiDefinitionResource> operation = await definitions
    .CreateOrUpdateAsync(WaitUntil.Completed, "openapi", definitionData);

ApiCenterApiDefinitionResource definition = operation.Value;

// Import đặc tả
string openApiSpec = await File.ReadAllTextAsync("orders-api.yaml");

ApiSpecImportContent importContent = new ApiSpecImportContent
{
    Format = ApiSpecImportSourceFormat.Inline,
    Value = openApiSpec,
    Specification = new ApiSpecImportSpecification
    {
        Name = "openapi",
        Version = "3.0.1"
    }
};

await definition.ImportSpecificationAsync(WaitUntil.Completed, importContent);
```

### 6. Export Đặc tả API

```csharp
ApiCenterApiDefinitionResource definition = await client
    .GetApiCenterApiDefinitionResource(definitionResourceId)
    .GetAsync();

ArmOperation<ApiSpecExportResult> operation = await definition
    .ExportSpecificationAsync(WaitUntil.Completed);

ApiSpecExportResult result = operation.Value;

// result.Format - ví dụ, "inline"
// result.Value - nội dung đặc tả
```

### 7. Tạo Môi trường

```csharp
ApiCenterEnvironmentCollection environments = workspace.GetApiCenterEnvironments();

ApiCenterEnvironmentData envData = new ApiCenterEnvironmentData
{
    Title = "Production",
    Description = "Production environment",
    Kind = ApiCenterEnvironmentKind.Production,
    Server = new ApiCenterEnvironmentServer
    {
        ManagementPortalUris = { new Uri("https://portal.azure.com") }
    },
    Onboarding = new EnvironmentOnboardingModel
    {
        Instructions = "Contact platform team for access",
        DeveloperPortalUris = { new Uri("https://developer.example.com") }
    }
};

ArmOperation<ApiCenterEnvironmentResource> operation = await environments
    .CreateOrUpdateAsync(WaitUntil.Completed, "production", envData);
```

### 8. Tạo Deployment

```csharp
ApiCenterDeploymentCollection deployments = workspace.GetApiCenterDeployments();

// Lấy environment resource ID
ResourceIdentifier envResourceId = ApiCenterEnvironmentResource.CreateResourceIdentifier(
    subscriptionId, resourceGroupName, serviceName, workspaceName, "production");

// Lấy API definition resource ID
ResourceIdentifier definitionResourceId = ApiCenterApiDefinitionResource.CreateResourceIdentifier(
    subscriptionId, resourceGroupName, serviceName, workspaceName,
    "orders-api", "v1-0-0", "openapi");

ApiCenterDeploymentData deploymentData = new ApiCenterDeploymentData
{
    Title = "Orders API - Production",
    Description = "Production deployment of Orders API v1.0.0",
    EnvironmentId = envResourceId,
    DefinitionId = definitionResourceId,
    State = ApiCenterDeploymentState.Active,
    Server = new ApiCenterDeploymentServer
    {
        RuntimeUris = { new Uri("https://api.example.com/orders") }
    }
};

ArmOperation<ApiCenterDeploymentResource> operation = await deployments
    .CreateOrUpdateAsync(WaitUntil.Completed, "orders-api-prod", deploymentData);
```

### 9. Tạo Metadata Schema

```csharp
ApiCenterMetadataSchemaCollection schemas = service.GetApiCenterMetadataSchemas();

string jsonSchema = """
{
    "type": "object",
    "properties": {
        "team": {
            "type": "string",
            "title": "Owning Team"
        },
        "costCenter": {
            "type": "string",
            "title": "Cost Center"
        },
        "dataClassification": {
            "type": "string",
            "enum": ["public", "internal", "confidential"],
            "title": "Data Classification"
        }
    },
    "required": ["team"]
}
""";

ApiCenterMetadataSchemaData schemaData = new ApiCenterMetadataSchemaData
{
    Schema = jsonSchema,
    AssignedTo =
    {
        new MetadataAssignment
        {
            Entity = MetadataAssignmentEntity.Api,
            Required = true
        }
    }
};

ArmOperation<ApiCenterMetadataSchemaResource> operation = await schemas
    .CreateOrUpdateAsync(WaitUntil.Completed, "api-metadata", schemaData);
```

### 10. Liệt kê và Tìm kiếm APIs

```csharp
// Liệt kê tất cả APIs trong một workspace
ApiCenterWorkspaceResource workspace = await client
    .GetApiCenterWorkspaceResource(workspaceResourceId)
    .GetAsync();

await foreach (ApiCenterApiResource api in workspace.GetApiCenterApis())
{
    Console.WriteLine($"API: {api.Data.Title}");
    Console.WriteLine($"  Kind: {api.Data.Kind}");
    Console.WriteLine($"  Stage: {api.Data.LifecycleStage}");

    // Liệt kê versions
    await foreach (ApiCenterApiVersionResource version in api.GetApiCenterApiVersions())
    {
        Console.WriteLine($"  Version: {version.Data.Title}");
    }
}

// Liệt kê environments
await foreach (ApiCenterEnvironmentResource env in workspace.GetApiCenterEnvironments())
{
    Console.WriteLine($"Environment: {env.Data.Title} ({env.Data.Kind})");
}

// Liệt kê deployments
await foreach (ApiCenterDeploymentResource deployment in workspace.GetApiCenterDeployments())
{
    Console.WriteLine($"Deployment: {deployment.Data.Title}");
    Console.WriteLine($"  State: {deployment.Data.State}");
}
```

## Tham khảo Các Loại Chính

| Loại                              | Mục đích                                                               |
| --------------------------------- | ---------------------------------------------------------------------- |
| `ApiCenterServiceResource`        | Instance dịch vụ API Center                                            |
| `ApiCenterWorkspaceResource`      | Nhóm logic các API                                                     |
| `ApiCenterApiResource`            | API riêng lẻ                                                           |
| `ApiCenterApiVersionResource`     | Phiên bản của một API                                                  |
| `ApiCenterApiDefinitionResource`  | Đặc tả API (OpenAPI, etc.)                                             |
| `ApiCenterEnvironmentResource`    | Môi trường triển khai                                                  |
| `ApiCenterDeploymentResource`     | API triển khai đến môi trường                                          |
| `ApiCenterMetadataSchemaResource` | Metadata schema tùy chỉnh                                              |
| `ApiKind`                         | rest, graphql, grpc, soap, webhook, websocket, mcp                     |
| `ApiLifecycleStage`               | design, development, testing, preview, production, deprecated, retired |
| `ApiCenterEnvironmentKind`        | development, testing, staging, production                              |
| `ApiCenterDeploymentState`        | active, inactive                                                       |

## Thực hành Tốt nhất

1. **Tổ chức với workspaces** — Nhóm APIs theo team, domain, hoặc sản phẩm
2. **Sử dụng metadata schemas** — Định nghĩa các thuộc tính tùy chỉnh cho quản trị
3. **Theo dõi các giai đoạn vòng đời** — Giữ trạng thái API cập nhật (design → production → deprecated)
4. **Tài liệu hóa môi trường** — Bao gồm hướng dẫn onboarding và portal URIs
5. **Lập phiên bản nhất quán** — Sử dụng semantic versioning cho các phiên bản API
6. **Import đặc tả** — Tải lên các specs OpenAPI/GraphQL để khám phá
7. **Liên kết các triển khai** — Kết nối APIs với môi trường runtime của chúng
8. **Sử dụng managed identity** — Bật SystemAssigned identity cho tích hợp an toàn

## Xử lý Lỗi

```csharp
using Azure;

try
{
    ArmOperation<ApiCenterApiResource> operation = await apis
        .CreateOrUpdateAsync(WaitUntil.Completed, "my-api", apiData);
}
catch (RequestFailedException ex) when (ex.Status == 409)
{
    Console.WriteLine("API already exists with conflicting configuration");
}
catch (RequestFailedException ex) when (ex.Status == 400)
{
    Console.WriteLine($"Invalid request: {ex.Message}");
}
catch (RequestFailedException ex)
{
    Console.WriteLine($"Azure error: {ex.Status} - {ex.Message}");
}
```

## Các SDK Liên quan

| SDK                                   | Mục đích                     | Cài đặt                                                  |
| ------------------------------------- | ---------------------------- | -------------------------------------------------------- |
| `Azure.ResourceManager.ApiCenter`     | Quản lý API Center (SDK này) | `dotnet add package Azure.ResourceManager.ApiCenter`     |
| `Azure.ResourceManager.ApiManagement` | API gateway và policies      | `dotnet add package Azure.ResourceManager.ApiManagement` |

## Liên kết Tham khảo

| Tài nguyên        | URL                                                                                                |
| ----------------- | -------------------------------------------------------------------------------------------------- |
| Gói NuGet         | https://www.nuget.org/packages/Azure.ResourceManager.ApiCenter                                     |
| Tham khảo API     | https://learn.microsoft.com/dotnet/api/azure.resourcemanager.apicenter                             |
| Tài liệu Sản phẩm | https://learn.microsoft.com/azure/api-center/                                                      |
| Mã nguồn GitHub   | https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/apicenter/Azure.ResourceManager.ApiCenter |
