---
name: azure-identity-rust
description: |
  Azure Identity SDK cho xác thực Rust. Sử dụng cho DeveloperToolsCredential, ManagedIdentityCredential, ClientSecretCredential, và xác thực dựa trên token.
  Triggers: "azure-identity", "DeveloperToolsCredential", "authentication rust", "managed identity rust", "credential rust".
package: azure_identity
---

# Azure Identity SDK cho Rust

Thư viện xác thực cho các client Azure SDK sử dụng Microsoft Entra ID (trước đây là Azure AD).

## Cài đặt

```sh
cargo add azure_identity
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

## DeveloperToolsCredential

Thông tin xác thực được khuyến nghị cho phát triển cục bộ. Thử các công cụ nhà phát triển theo thứ tự (Azure CLI, Azure Developer CLI):

```rust
use azure_identity::DeveloperToolsCredential;
use azure_security_keyvault_secrets::SecretClient;

let credential = DeveloperToolsCredential::new(None)?;
let client = SecretClient::new(
    "https://my-vault.vault.azure.net/",
    credential.clone(),
    None,
)?;
```

### Thứ tự Chuỗi Credential

| Thứ tự | Credential                  | Môi trường       |
| ------ | --------------------------- | ---------------- |
| 1      | AzureCliCredential          | `az login`       |
| 2      | AzureDeveloperCliCredential | `azd auth login` |

## Các Loại Credential

| Credential                    | Sử dụng                                   |
| ----------------------------- | ----------------------------------------- |
| `DeveloperToolsCredential`    | Phát triển cục bộ - thử các công cụ CLI   |
| `ManagedIdentityCredential`   | Azure VMs, App Service, Functions, AKS    |
| `WorkloadIdentityCredential`  | Kubernetes workload identity              |
| `ClientSecretCredential`      | Service principal với secret              |
| `ClientCertificateCredential` | Service principal với certificate         |
| `AzureCliCredential`          | Xác thực trực tiếp Azure CLI              |
| `AzureDeveloperCliCredential` | Xác thực trực tiếp azd CLI                |
| `AzurePipelinesCredential`    | Kết nối dịch vụ Azure Pipelines           |
| `ClientAssertionCredential`   | Assertions tùy chỉnh (danh tính liên kết) |

## ManagedIdentityCredential

Cho các tài nguyên được host trên Azure:

```rust
use azure_identity::ManagedIdentityCredential;

// System-assigned managed identity
let credential = ManagedIdentityCredential::new(None)?;

// User-assigned managed identity
let options = ManagedIdentityCredentialOptions {
    client_id: Some("<user-assigned-mi-client-id>".into()),
    ..Default::default()
};
let credential = ManagedIdentityCredential::new(Some(options))?;
```

## ClientSecretCredential

Cho service principal với secret:

```rust
use azure_identity::ClientSecretCredential;

let credential = ClientSecretCredential::new(
    "<tenant-id>".into(),
    "<client-id>".into(),
    "<client-secret>".into(),
    None,
)?;
```

## Thực hành Tốt nhất

1. **Sử dụng `DeveloperToolsCredential` cho dev cục bộ** — tự động nhận diện Azure CLI
2. **Sử dụng `ManagedIdentityCredential` trong sản xuất** — không cần quản lý secret
3. **Clone credentials** — credentials được bao bọc bởi `Arc` và clone rất rẻ
4. **Tái sử dụng các instance credential** — cùng một credential có thể dùng với nhiều client
5. **Sử dụng tính năng `tokio`** — `cargo add azure_identity --features tokio`

## Liên kết Tham khảo

| Tài nguyên    | Liên kết                                                                          |
| ------------- | --------------------------------------------------------------------------------- |
| Tham khảo API | https://docs.rs/azure_identity                                                    |
| Mã nguồn      | https://github.com/Azure/azure-sdk-for-rust/tree/main/sdk/identity/azure_identity |
| crates.io     | https://crates.io/crates/azure_identity                                           |
