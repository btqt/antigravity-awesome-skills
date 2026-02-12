---
name: azure-identity-py
description: |
  Azure Identity SDK cho xác thực Python. Sử dụng cho DefaultAzureCredential, managed identity, service principals, và token caching.
  Triggers: "azure-identity", "DefaultAzureCredential", "authentication", "managed identity", "service principal", "credential".
package: azure-identity
---

# Azure Identity SDK cho Python

Thư viện xác thực cho các client Azure SDK sử dụng Microsoft Entra ID (trước đây là Azure AD).

## Cài đặt

```bash
pip install azure-identity
```

## Biến Môi trường

```bash
# Service Principal (cho sản xuất/CI)
AZURE_TENANT_ID=<your-tenant-id>
AZURE_CLIENT_ID=<your-client-id>
AZURE_CLIENT_SECRET=<your-client-secret>

# User-assigned Managed Identity (tùy chọn)
AZURE_CLIENT_ID=<managed-identity-client-id>
```

## DefaultAzureCredential

Thông tin xác thực được khuyến nghị cho hầu hết các kịch bản. Thử nhiều phương thức xác thực theo thứ tự:

```python
from azure.identity import DefaultAzureCredential
from azure.storage.blob import BlobServiceClient

# Hoạt động trong local dev VÀ sản xuất mà không cần thay đổi mã
credential = DefaultAzureCredential()

client = BlobServiceClient(
    account_url="https://<account>.blob.core.windows.net",
    credential=credential
)
```

### Thứ tự Chuỗi Credential

| Thứ tự | Credential                  | Môi trường                        |
| ------ | --------------------------- | --------------------------------- |
| 1      | EnvironmentCredential       | CI/CD, containers                 |
| 2      | WorkloadIdentityCredential  | Kubernetes                        |
| 3      | ManagedIdentityCredential   | Azure VMs, App Service, Functions |
| 4      | SharedTokenCacheCredential  | Chỉ Windows                       |
| 5      | VisualStudioCodeCredential  | VS Code với Azure extension       |
| 6      | AzureCliCredential          | `az login`                        |
| 7      | AzurePowerShellCredential   | `Connect-AzAccount`               |
| 8      | AzureDeveloperCliCredential | `azd auth login`                  |

### Tùy chỉnh DefaultAzureCredential

```python
# Loại trừ các credential bạn không cần
credential = DefaultAzureCredential(
    exclude_environment_credential=True,
    exclude_shared_token_cache_credential=True,
    managed_identity_client_id="<user-assigned-mi-client-id>"  # Cho user-assigned MI
)

# Bật trình duyệt tương tác (tắt theo mặc định)
credential = DefaultAzureCredential(
    exclude_interactive_browser_credential=False
)
```

## Các Loại Credential Cụ thể

### ManagedIdentityCredential

Cho các tài nguyên được host trên Azure (VMs, App Service, Functions, AKS):

```python
from azure.identity import ManagedIdentityCredential

# System-assigned managed identity
credential = ManagedIdentityCredential()

# User-assigned managed identity
credential = ManagedIdentityCredential(
    client_id="<user-assigned-mi-client-id>"
)
```

### ClientSecretCredential

Cho service principal với secret:

```python
from azure.identity import ClientSecretCredential

credential = ClientSecretCredential(
    tenant_id=os.environ["AZURE_TENANT_ID"],
    client_id=os.environ["AZURE_CLIENT_ID"],
    client_secret=os.environ["AZURE_CLIENT_SECRET"]
)
```

### AzureCliCredential

Sử dụng tài khoản từ `az login`:

```python
from azure.identity import AzureCliCredential

credential = AzureCliCredential()
```

### ChainedTokenCredential

Chuỗi credential tùy chỉnh:

```python
from azure.identity import (
    ChainedTokenCredential,
    ManagedIdentityCredential,
    AzureCliCredential
)

# Thử managed identity trước, nếu không được thì dùng CLI
credential = ChainedTokenCredential(
    ManagedIdentityCredential(client_id="<user-assigned-mi-client-id>"),
    AzureCliCredential()
)
```

## Bảng Các Loại Credential

| Credential                     | Trường hợp Sử dụng    | Phương thức Auth    |
| ------------------------------ | --------------------- | ------------------- |
| `DefaultAzureCredential`       | Hầu hết các kịch bản  | Tự động phát hiện   |
| `ManagedIdentityCredential`    | Ứng dụng Azure-hosted | Managed Identity    |
| `ClientSecretCredential`       | Service principal     | Client secret       |
| `ClientCertificateCredential`  | Service principal     | Certificate         |
| `AzureCliCredential`           | Phát triển cục bộ     | Azure CLI           |
| `AzureDeveloperCliCredential`  | Phát triển cục bộ     | Azure Developer CLI |
| `InteractiveBrowserCredential` | Người dùng đăng nhập  | Browser OAuth       |
| `DeviceCodeCredential`         | Headless/SSH          | Device code flow    |

## Lấy Token Trực tiếp

```python
from azure.identity import DefaultAzureCredential

credential = DefaultAzureCredential()

# Lấy token cho một scope cụ thể
token = credential.get_token("https://management.azure.com/.default")
print(f"Token expires: {token.expires_on}")

# Cho Azure Database for PostgreSQL
token = credential.get_token("https://ossrdbms-aad.database.windows.net/.default")
```

## Async Client

```python
from azure.identity.aio import DefaultAzureCredential
from azure.storage.blob.aio import BlobServiceClient

async def main():
    credential = DefaultAzureCredential()

    async with BlobServiceClient(
        account_url="https://<account>.blob.core.windows.net",
        credential=credential
    ) as client:
        # ... các thao tác async
        pass

    await credential.close()
```

## Thực hành Tốt nhất

1. **Sử dụng DefaultAzureCredential** cho mã chạy cả cục bộ và trên Azure
2. **Không bao giờ hardcode credentials** — sử dụng biến môi trường hoặc managed identity
3. **Ưu tiên managed identity** trong các triển khai Azure sản xuất
4. **Sử dụng ChainedTokenCredential** khi bạn cần thứ tự credential tùy chỉnh
5. **Đóng các async credentials** một cách rõ ràng hoặc sử dụng context managers
6. **Thiết lập AZURE_CLIENT_ID** cho user-assigned managed identities
7. **Loại trừ các credentials không sử dụng** để tăng tốc độ xác thực
