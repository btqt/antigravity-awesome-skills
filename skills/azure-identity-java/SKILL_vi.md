---
name: azure-identity-java
description: Azure Identity Java SDK cho xác thực với các dịch vụ Azure. Sử dụng khi triển khai DefaultAzureCredential, managed identity, service principal, hoặc bất kỳ mẫu xác thực Azure nào trong ứng dụng Java.
package: com.azure:azure-identity
---

# Azure Identity (Java)

Xác thực ứng dụng Java với các dịch vụ Azure sử dụng Microsoft Entra ID (Azure AD).

## Cài đặt

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-identity</artifactId>
    <version>1.15.0</version>
</dependency>
```

## Các Khái niệm Chính

| Credential                     | Trường hợp Sử dụng                                  |
| ------------------------------ | --------------------------------------------------- |
| `DefaultAzureCredential`       | **Khuyên dùng** - Hoạt động trong dev và sản xuất   |
| `ManagedIdentityCredential`    | Ứng dụng Azure-hosted (App Service, Functions, VMs) |
| `EnvironmentCredential`        | CI/CD pipelines với biến môi trường                 |
| `ClientSecretCredential`       | Service principals với secret                       |
| `ClientCertificateCredential`  | Service principals với certificate                  |
| `AzureCliCredential`           | Dev cục bộ sử dụng `az login`                       |
| `InteractiveBrowserCredential` | Luồng đăng nhập tương tác                           |
| `DeviceCodeCredential`         | Xác thực thiết bị không đầu (headless)              |

## DefaultAzureCredential (Khuyên dùng)

`DefaultAzureCredential` thử nhiều phương thức xác thực theo thứ tự:

1. Biến môi trường
2. Workload Identity
3. Managed Identity
4. Azure CLI
5. Azure PowerShell
6. Azure Developer CLI

```java
import com.azure.identity.DefaultAzureCredential;
import com.azure.identity.DefaultAzureCredentialBuilder;

// Sử dụng đơn giản
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();

// Sử dụng với bất kỳ Azure client nào
BlobServiceClient blobClient = new BlobServiceClientBuilder()
    .endpoint("https://<storage-account>.blob.core.windows.net")
    .credential(credential)
    .buildClient();

KeyClient keyClient = new KeyClientBuilder()
    .vaultUrl("https://<vault-name>.vault.azure.net")
    .credential(credential)
    .buildClient();
```

### Cấu hình DefaultAzureCredential

```java
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder()
    .managedIdentityClientId("<user-assigned-identity-client-id>")  // Cho user-assigned MI
    .tenantId("<tenant-id>")                                        // Giới hạn tenant cụ thể
    .excludeEnvironmentCredential()                                 // Bỏ qua biến môi trường
    .excludeAzureCliCredential()                                    // Bỏ qua Azure CLI
    .build();
```

## Managed Identity

Cho các ứng dụng được host trên Azure (App Service, Functions, AKS, VMs).

```java
import com.azure.identity.ManagedIdentityCredential;
import com.azure.identity.ManagedIdentityCredentialBuilder;

// System-assigned managed identity
ManagedIdentityCredential credential = new ManagedIdentityCredentialBuilder()
    .build();

// User-assigned managed identity (bởi client ID)
ManagedIdentityCredential credential = new ManagedIdentityCredentialBuilder()
    .clientId("<user-assigned-client-id>")
    .build();

// User-assigned managed identity (bởi resource ID)
ManagedIdentityCredential credential = new ManagedIdentityCredentialBuilder()
    .resourceId("/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<name>")
    .build();
```

## Service Principal với Secret

```java
import com.azure.identity.ClientSecretCredential;
import com.azure.identity.ClientSecretCredentialBuilder;

ClientSecretCredential credential = new ClientSecretCredentialBuilder()
    .tenantId("<tenant-id>")
    .clientId("<client-id>")
    .clientSecret("<client-secret>")
    .build();
```

## Service Principal với Certificate

```java
import com.azure.identity.ClientCertificateCredential;
import com.azure.identity.ClientCertificateCredentialBuilder;

// Từ tệp PEM
ClientCertificateCredential credential = new ClientCertificateCredentialBuilder()
    .tenantId("<tenant-id>")
    .clientId("<client-id>")
    .pemCertificate("<path-to-cert.pem>")
    .build();

// Từ tệp PFX với mật khẩu
ClientCertificateCredential credential = new ClientCertificateCredentialBuilder()
    .tenantId("<tenant-id>")
    .clientId("<client-id>")
    .pfxCertificate("<path-to-cert.pfx>", "<pfx-password>")
    .build();

// Gửi chuỗi chứng chỉ cho SNI
ClientCertificateCredential credential = new ClientCertificateCredentialBuilder()
    .tenantId("<tenant-id>")
    .clientId("<client-id>")
    .pemCertificate("<path-to-cert.pem>")
    .sendCertificateChain(true)
    .build();
```

## Environment Credential

Đọc thông tin xác thực từ biến môi trường.

```java
import com.azure.identity.EnvironmentCredential;
import com.azure.identity.EnvironmentCredentialBuilder;

EnvironmentCredential credential = new EnvironmentCredentialBuilder().build();
```

### Các Biến Môi trường Yêu cầu

**Cho service principal với secret:**

```bash
AZURE_TENANT_ID=<tenant-id>
AZURE_CLIENT_ID=<client-id>
AZURE_CLIENT_SECRET=<client-secret>
```

**Cho service principal với certificate:**

```bash
AZURE_TENANT_ID=<tenant-id>
AZURE_CLIENT_ID=<client-id>
AZURE_CLIENT_CERTIFICATE_PATH=/path/to/cert.pem
AZURE_CLIENT_CERTIFICATE_PASSWORD=<optional-password>
```

**Cho username/password:**

```bash
AZURE_TENANT_ID=<tenant-id>
AZURE_CLIENT_ID=<client-id>
AZURE_USERNAME=<username>
AZURE_PASSWORD=<password>
```

## Azure CLI Credential

Cho phát triển cục bộ sử dụng `az login`.

```java
import com.azure.identity.AzureCliCredential;
import com.azure.identity.AzureCliCredentialBuilder;

AzureCliCredential credential = new AzureCliCredentialBuilder()
    .tenantId("<tenant-id>")  // Tùy chọn: tenant cụ thể
    .build();
```

## Interactive Browser

Cho các ứng dụng desktop yêu cầu người dùng đăng nhập.

```java
import com.azure.identity.InteractiveBrowserCredential;
import com.azure.identity.InteractiveBrowserCredentialBuilder;

InteractiveBrowserCredential credential = new InteractiveBrowserCredentialBuilder()
    .clientId("<client-id>")
    .redirectUrl("http://localhost:8080")  // Phải khớp với app registration
    .build();
```

## Device Code

Cho các thiết bị không đầu (IoT, công cụ CLI).

```java
import com.azure.identity.DeviceCodeCredential;
import com.azure.identity.DeviceCodeCredentialBuilder;

DeviceCodeCredential credential = new DeviceCodeCredentialBuilder()
    .clientId("<client-id>")
    .challengeConsumer(challenge -> {
        // Hiển thị cho người dùng
        System.out.println(challenge.getMessage());
    })
    .build();
```

## Chained Credential

Tạo các chuỗi xác thực tùy chỉnh.

```java
import com.azure.identity.ChainedTokenCredential;
import com.azure.identity.ChainedTokenCredentialBuilder;

ChainedTokenCredential credential = new ChainedTokenCredentialBuilder()
    .addFirst(new ManagedIdentityCredentialBuilder().build())
    .addLast(new AzureCliCredentialBuilder().build())
    .build();
```

## Workload Identity (AKS)

Cho Azure Kubernetes Service với workload identity.

```java
import com.azure.identity.WorkloadIdentityCredential;
import com.azure.identity.WorkloadIdentityCredentialBuilder;

// Đọc từ AZURE_TENANT_ID, AZURE_CLIENT_ID, AZURE_FEDERATED_TOKEN_FILE
WorkloadIdentityCredential credential = new WorkloadIdentityCredentialBuilder().build();

// Hoặc cấu hình tường minh
WorkloadIdentityCredential credential = new WorkloadIdentityCredentialBuilder()
    .tenantId("<tenant-id>")
    .clientId("<client-id>")
    .tokenFilePath("/var/run/secrets/azure/tokens/azure-identity-token")
    .build();
```

## Token Caching

Bật caching token bền vững để có hiệu năng tốt hơn.

```java
// Bật token caching (trong bộ nhớ theo mặc định)
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder()
    .enableAccountIdentifierLogging()
    .build();

// Với shared token cache (cho kịch bản đa credential)
SharedTokenCacheCredential credential = new SharedTokenCacheCredentialBuilder()
    .clientId("<client-id>")
    .build();
```

## Sovereign Clouds (Các Đám mây Chủ quyền)

```java
import com.azure.identity.AzureAuthorityHosts;

// Azure Government
DefaultAzureCredential govCredential = new DefaultAzureCredentialBuilder()
    .authorityHost(AzureAuthorityHosts.AZURE_GOVERNMENT)
    .build();

// Azure China
DefaultAzureCredential chinaCredential = new DefaultAzureCredentialBuilder()
    .authorityHost(AzureAuthorityHosts.AZURE_CHINA)
    .build();
```

## Xử lý Lỗi

```java
import com.azure.identity.CredentialUnavailableException;
import com.azure.core.exception.ClientAuthenticationException;

try {
    DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
    AccessToken token = credential.getToken(new TokenRequestContext()
        .addScopes("https://management.azure.com/.default"));
} catch (CredentialUnavailableException e) {
    // Không credential nào có thể xác thực
    System.out.println("Authentication failed: " + e.getMessage());
} catch (ClientAuthenticationException e) {
    // Lỗi xác thực (sai credentials, hết hạn, v.v.)
    System.out.println("Auth error: " + e.getMessage());
}
```

## Logging

Bật logging xác thực để gỡ lỗi.

```java
// Qua biến môi trường
// AZURE_LOG_LEVEL=verbose

// Hoặc bằng lập trình
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder()
    .enableAccountIdentifierLogging()  // Log thông tin tài khoản
    .build();
```

## Biến Môi trường

```bash
# Cấu hình DefaultAzureCredential
AZURE_TENANT_ID=<tenant-id>
AZURE_CLIENT_ID=<client-id>
AZURE_CLIENT_SECRET=<client-secret>

# Managed Identity
AZURE_CLIENT_ID=<user-assigned-mi-client-id>

# Workload Identity (AKS)
AZURE_FEDERATED_TOKEN_FILE=/var/run/secrets/azure/tokens/azure-identity-token

# Logging
AZURE_LOG_LEVEL=verbose

# Authority host
AZURE_AUTHORITY_HOST=https://login.microsoftonline.com/
```

## Thực hành Tốt nhất

1. **Sử dụng DefaultAzureCredential** - Hoạt động mượt mà từ dev đến sản xuất
2. **Managed Identity trong Sản xuất** - Không cần quản lý secret, tự động xoay vòng
3. **Azure CLI cho Dev Cục bộ** - Chạy `az login` trước khi chạy ứng dụng
4. **Đặc quyền Tối thiểu** - Chỉ cấp quyền cần thiết cho service principals
5. **Token Caching** - Bật theo mặc định, giảm số lần auth round-trip
6. **Biến Môi trường** - Sử dụng cho CI/CD, không hardcode secrets

## Ma trận Lựa chọn Credential

| Môi trường               | Credential Khuyên dùng                              |
| ------------------------ | --------------------------------------------------- |
| Local Development        | `DefaultAzureCredential` (sử dụng Azure CLI)        |
| Azure App Service        | `DefaultAzureCredential` (sử dụng Managed Identity) |
| Azure Functions          | `DefaultAzureCredential` (sử dụng Managed Identity) |
| Azure Kubernetes Service | `WorkloadIdentityCredential`                        |
| Azure VMs                | `DefaultAzureCredential` (sử dụng Managed Identity) |
| CI/CD Pipeline           | `EnvironmentCredential`                             |
| Desktop App              | `InteractiveBrowserCredential`                      |
| CLI Tool                 | `DeviceCodeCredential`                              |

## Các Cụm từ Kích hoạt

- "Azure authentication Java", "DefaultAzureCredential Java"
- "managed identity Java", "service principal Java"
- "Azure login Java", "Azure credentials Java"
- "AZURE_CLIENT_ID", "AZURE_TENANT_ID"
