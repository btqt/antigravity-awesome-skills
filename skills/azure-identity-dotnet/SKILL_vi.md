---
name: azure-identity-dotnet
description: |
  Azure Identity SDK cho .NET. Thư viện xác thực cho các client Azure SDK sử dụng Microsoft Entra ID. Sử dụng cho DefaultAzureCredential, managed identity, service principals, và thông tin xác thực nhà phát triển. Triggers: "Azure Identity", "DefaultAzureCredential", "ManagedIdentityCredential", "ClientSecretCredential", "authentication .NET", "Azure auth", "credential chain".
package: Azure.Identity
---

# Azure.Identity (.NET)

Thư viện xác thực cho các client Azure SDK sử dụng Microsoft Entra ID (trước đây là Azure AD).

## Cài đặt

```bash
dotnet add package Azure.Identity

# Cho ASP.NET Core
dotnet add package Microsoft.Extensions.Azure

# Cho xác thực qua trung gian (Windows)
dotnet add package Azure.Identity.Broker
```

**Phiên bản Hiện tại**: Stable v1.17.1, Preview v1.18.0-beta.2

## Biến Môi trường

### Service Principal với Secret

```bash
AZURE_CLIENT_ID=<application-client-id>
AZURE_TENANT_ID=<directory-tenant-id>
AZURE_CLIENT_SECRET=<client-secret-value>
```

### Service Principal với Certificate

```bash
AZURE_CLIENT_ID=<application-client-id>
AZURE_TENANT_ID=<directory-tenant-id>
AZURE_CLIENT_CERTIFICATE_PATH=<path-to-pfx-or-pem>
AZURE_CLIENT_CERTIFICATE_PASSWORD=<certificate-password>  # Tùy chọn
```

### Managed Identity

```bash
AZURE_CLIENT_ID=<user-assigned-managed-identity-client-id>  # Chỉ cho user-assigned
```

## DefaultAzureCredential

Thông tin xác thực được khuyến nghị cho hầu hết các kịch bản. Thử nhiều phương thức xác thực theo thứ tự:

| Thứ tự | Credential                   | Bật Mặc định |
| ------ | ---------------------------- | ------------ |
| 1      | EnvironmentCredential        | Có           |
| 2      | WorkloadIdentityCredential   | Có           |
| 3      | ManagedIdentityCredential    | Có           |
| 4      | VisualStudioCredential       | Có           |
| 5      | VisualStudioCodeCredential   | Có           |
| 6      | AzureCliCredential           | Có           |
| 7      | AzurePowerShellCredential    | Có           |
| 8      | AzureDeveloperCliCredential  | Có           |
| 9      | InteractiveBrowserCredential | **Không**    |

### Cách dùng Cơ bản

```csharp
using Azure.Identity;
using Azure.Storage.Blobs;

var credential = new DefaultAzureCredential();
var blobClient = new BlobServiceClient(
    new Uri("https://myaccount.blob.core.windows.net"),
    credential);
```

### ASP.NET Core với Dependency Injection

```csharp
using Azure.Identity;
using Microsoft.Extensions.Azure;

builder.Services.AddAzureClients(clientBuilder =>
{
    clientBuilder.AddBlobServiceClient(
        new Uri("https://myaccount.blob.core.windows.net"));
    clientBuilder.AddSecretClient(
        new Uri("https://myvault.vault.azure.net"));

    // Sử dụng DefaultAzureCredential mặc định
    clientBuilder.UseCredential(new DefaultAzureCredential());
});
```

### Tùy chỉnh DefaultAzureCredential

```csharp
var credential = new DefaultAzureCredential(
    new DefaultAzureCredentialOptions
    {
        ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = false,
        ExcludeVisualStudioCredential = false,
        ExcludeAzureCliCredential = false,
        ExcludeInteractiveBrowserCredential = false, // Bật tương tác
        TenantId = "<tenant-id>",
        ManagedIdentityClientId = "<user-assigned-mi-client-id>"
    });
```

## Các Loại Credential

### ManagedIdentityCredential (Sản xuất)

```csharp
// System-assigned managed identity
var credential = new ManagedIdentityCredential(ManagedIdentityId.SystemAssigned);

// User-assigned bởi client ID
var credential = new ManagedIdentityCredential(
    ManagedIdentityId.FromUserAssignedClientId("<client-id>"));

// User-assigned bởi resource ID
var credential = new ManagedIdentityCredential(
    ManagedIdentityId.FromUserAssignedResourceId("<resource-id>"));
```

### ClientSecretCredential

```csharp
var credential = new ClientSecretCredential(
    tenantId: "<tenant-id>",
    clientId: "<client-id>",
    clientSecret: "<client-secret>");

var client = new SecretClient(
    new Uri("https://myvault.vault.azure.net"),
    credential);
```

### ClientCertificateCredential

```csharp
var certificate = X509CertificateLoader.LoadCertificateFromFile("MyCertificate.pfx");
var credential = new ClientCertificateCredential(
    tenantId: "<tenant-id>",
    clientId: "<client-id>",
    certificate);
```

### ChainedTokenCredential (Chuỗi Tùy chỉnh)

```csharp
var credential = new ChainedTokenCredential(
    new ManagedIdentityCredential(),
    new AzureCliCredential());

var client = new SecretClient(
    new Uri("https://myvault.vault.azure.net"),
    credential);
```

### Thông tin Xác thực Nhà phát triển

```csharp
// Azure CLI
var credential = new AzureCliCredential();

// Azure PowerShell
var credential = new AzurePowerShellCredential();

// Azure Developer CLI (azd)
var credential = new AzureDeveloperCliCredential();

// Visual Studio
var credential = new VisualStudioCredential();

// Interactive Browser
var credential = new InteractiveBrowserCredential();
```

## Cấu hình Dựa trên Môi trường

```csharp
// Sản xuất vs Phát triển
TokenCredential credential = builder.Environment.IsProduction()
    ? new ManagedIdentityCredential("<client-id>")
    : new DefaultAzureCredential();
```

## Sovereign Clouds (Các Đám mây Chủ quyền)

```csharp
var credential = new DefaultAzureCredential(
    new DefaultAzureCredentialOptions
    {
        AuthorityHost = AzureAuthorityHosts.AzureGovernment
    });

// Các authority host có sẵn:
// AzureAuthorityHosts.AzurePublicCloud (mặc định)
// AzureAuthorityHosts.AzureGovernment
// AzureAuthorityHosts.AzureChina
// AzureAuthorityHosts.AzureGermany
```

## Tham khảo Các Loại Credential

| Danh mục              | Credential                     | Mục đích                             |
| --------------------- | ------------------------------ | ------------------------------------ |
| **Chains**            | `DefaultAzureCredential`       | Chuỗi cấu hình sẵn cho dev-to-prod   |
|                       | `ChainedTokenCredential`       | Chuỗi credential tùy chỉnh           |
| **Azure-Hosted**      | `ManagedIdentityCredential`    | Azure managed identity               |
|                       | `WorkloadIdentityCredential`   | Kubernetes workload identity         |
|                       | `EnvironmentCredential`        | Biến môi trường                      |
| **Service Principal** | `ClientSecretCredential`       | Client ID + secret                   |
|                       | `ClientCertificateCredential`  | Client ID + certificate              |
|                       | `ClientAssertionCredential`    | Signed client assertion              |
| **User**              | `InteractiveBrowserCredential` | Xác thực dựa trên trình duyệt        |
|                       | `DeviceCodeCredential`         | Luồng mã thiết bị (Device code flow) |
|                       | `OnBehalfOfCredential`         | Danh tính được ủy quyền              |
| **Developer**         | `AzureCliCredential`           | Azure CLI                            |
|                       | `AzurePowerShellCredential`    | Azure PowerShell                     |
|                       | `AzureDeveloperCliCredential`  | Azure Developer CLI                  |
|                       | `VisualStudioCredential`       | Visual Studio                        |

## Thực hành Tốt nhất

### 1. Sử dụng Credential Xác định trong Sản xuất

```csharp
// Phát triển
var devCredential = new DefaultAzureCredential();

// Sản xuất - sử dụng credential cụ thể
var prodCredential = new ManagedIdentityCredential("<client-id>");
```

### 2. Tái sử dụng Các Instance Credential

```csharp
// Tốt: Một instance credential duy nhất được chia sẻ giữa các client
var credential = new DefaultAzureCredential();
var blobClient = new BlobServiceClient(blobUri, credential);
var secretClient = new SecretClient(vaultUri, credential);
```

### 3. Cấu hình Chính sách Thử lại (Retry Policies)

```csharp
var options = new ManagedIdentityCredentialOptions(
    ManagedIdentityId.FromUserAssignedClientId(clientId))
{
    Retry =
    {
        MaxRetries = 3,
        Delay = TimeSpan.FromSeconds(0.5),
    }
};
var credential = new ManagedIdentityCredential(options);
```

### 4. Bật Logging để Gỡ lỗi

```csharp
using Azure.Core.Diagnostics;

using AzureEventSourceListener listener = new((args, message) =>
{
    if (args is { EventSource.Name: "Azure-Identity" })
    {
        Console.WriteLine(message);
    }
}, EventLevel.LogAlways);
```

## Xử lý Lỗi

```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

var client = new SecretClient(
    new Uri("https://myvault.vault.azure.net"),
    new DefaultAzureCredential());

try
{
    KeyVaultSecret secret = await client.GetSecretAsync("secret1");
}
catch (AuthenticationFailedException e)
{
    Console.WriteLine($"Authentication Failed: {e.Message}");
}
catch (CredentialUnavailableException e)
{
    Console.WriteLine($"Credential Unavailable: {e.Message}");
}
```

## Các Ngoại lệ Chính

| Ngoại lệ                          | Mô tả                                                   |
| --------------------------------- | ------------------------------------------------------- |
| `AuthenticationFailedException`   | Ngoại lệ cơ sở cho các lỗi xác thực                     |
| `CredentialUnavailableException`  | Credential không thể xác thực trong môi trường hiện tại |
| `AuthenticationRequiredException` | Yêu cầu xác thực tương tác                              |

## Hỗ trợ Managed Identity

Các dịch vụ Azure được hỗ trợ:

- Azure App Service và Azure Functions
- Azure Arc
- Azure Cloud Shell
- Azure Kubernetes Service (AKS)
- Azure Service Fabric
- Azure Virtual Machines
- Azure Virtual Machine Scale Sets

## An toàn Luồng (Thread Safety)

Tất cả các triển khai credential đều an toàn luồng. Một instance credential duy nhất có thể được chia sẻ an toàn giữa nhiều client và luồng.

## Các SDK Liên quan

| SDK                          | Mục đích                          | Cài đặt                                         |
| ---------------------------- | --------------------------------- | ----------------------------------------------- |
| `Azure.Identity`             | Xác thực (SDK này)                | `dotnet add package Azure.Identity`             |
| `Microsoft.Extensions.Azure` | Tích hợp DI                       | `dotnet add package Microsoft.Extensions.Azure` |
| `Azure.Identity.Broker`      | Xác thực qua trung gian (Windows) | `dotnet add package Azure.Identity.Broker`      |

## Liên kết Tham khảo

| Tài nguyên         | URL                                                                              |
| ------------------ | -------------------------------------------------------------------------------- |
| Gói NuGet          | https://www.nuget.org/packages/Azure.Identity                                    |
| Tham khảo API      | https://learn.microsoft.com/dotnet/api/azure.identity                            |
| Chuỗi Credential   | https://learn.microsoft.com/dotnet/azure/sdk/authentication/credential-chains    |
| Thực hành Tốt nhất | https://learn.microsoft.com/dotnet/azure/sdk/authentication/best-practices       |
| Mã nguồn GitHub    | https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/identity/Azure.Identity |
