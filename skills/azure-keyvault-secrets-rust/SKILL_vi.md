---
name: azure-keyvault-secrets-rust
description: |
  Azure Key Vault Secrets SDK cho Rust. Sử dụng để lưu trữ và truy xuất các bí mật, mật khẩu và khóa API.
  Triggers: "keyvault secrets rust", "SecretClient rust", "get secret rust", "set secret rust".
package: azure_security_keyvault_secrets
---

# Azure Key Vault Secrets SDK cho Rust

Thư viện client cho Azure Key Vault Secrets — lưu trữ an toàn cho mật khẩu, khóa API và các secrets khác.

## Cài đặt

```sh
cargo add azure_security_keyvault_secrets azure_identity
```

## Biến Môi trường

```bash
AZURE_KEYVAULT_URL=https://<vault-name>.vault.azure.net/
```

## Xác thực

```rust
use azure_identity::DeveloperToolsCredential;
use azure_security_keyvault_secrets::SecretClient;

let credential = DeveloperToolsCredential::new(None)?;
let client = SecretClient::new(
    "https://<vault-name>.vault.azure.net/",
    credential.clone(),
    None,
)?;
```

## Các Thao tác Cốt lõi

### Lấy Secret

```rust
let secret = client
    .get_secret("secret-name", None)
    .await?
    .into_model()?;

println!("Secret value: {:?}", secret.value);
```

### Đặt Secret

```rust
use azure_security_keyvault_secrets::models::SetSecretParameters;

let params = SetSecretParameters {
    value: Some("secret-value".into()),
    ..Default::default()
};

let secret = client
    .set_secret("secret-name", params.try_into()?, None)
    .await?
    .into_model()?;
```

### Cập nhật Thuộc tính Secret

```rust
use azure_security_keyvault_secrets::models::UpdateSecretPropertiesParameters;
use std::collections::HashMap;

let params = UpdateSecretPropertiesParameters {
    content_type: Some("text/plain".into()),
    tags: Some(HashMap::from([("env".into(), "prod".into())])),
    ..Default::default()
};

client
    .update_secret_properties("secret-name", params.try_into()?, None)
    .await?;
```

### Xóa Secret

```rust
client.delete_secret("secret-name", None).await?;
```

### Liệt kê Secrets

```rust
use azure_security_keyvault_secrets::ResourceExt;
use futures::TryStreamExt;

let mut pager = client.list_secret_properties(None)?.into_stream();
while let Some(secret) = pager.try_next().await? {
    let name = secret.resource_id()?.name;
    println!("Secret: {}", name);
}
```

### Lấy Phiên bản Cụ thể

```rust
use azure_security_keyvault_secrets::models::SecretClientGetSecretOptions;

let options = SecretClientGetSecretOptions {
    secret_version: Some("version-id".into()),
    ..Default::default()
};

let secret = client
    .get_secret("secret-name", Some(options))
    .await?
    .into_model()?;
```

## Thực hành Tốt nhất

1. **Sử dụng xác thực Entra ID** — `DeveloperToolsCredential` cho dev, `ManagedIdentityCredential` cho sản xuất
2. **Sử dụng `into_model()?`** — để deserialize phản hồi
3. **Sử dụng `ResourceExt` trait** — để trích xuất tên từ ID
4. **Xử lý soft delete** — các secret đã xóa có thể được khôi phục trong thời gian lưu giữ
5. **Đặt content type** — giúp xác định định dạng secret
6. **Sử dụng tags** — để tổ chức và lọc secrets
7. **Phiên bản hóa secrets** — các giá trị mới tạo ra phiên bản mới tự động

## Quyền RBAC

Gán các vai trò Key Vault sau:

- `Key Vault Secrets User` — get và list
- `Key Vault Secrets Officer` — toàn quyền CRUD

## Liên kết Tham khảo

| Tài nguyên    | Liên kết                                                                                           |
| ------------- | -------------------------------------------------------------------------------------------------- |
| Tham khảo API | https://docs.rs/azure_security_keyvault_secrets                                                    |
| Mã nguồn      | https://github.com/Azure/azure-sdk-for-rust/tree/main/sdk/keyvault/azure_security_keyvault_secrets |
| crates.io     | https://crates.io/crates/azure_security_keyvault_secrets                                           |
