---
name: azure-keyvault-keys-rust
description: |
  Azure Key Vault Keys SDK cho Rust. Sử dụng để tạo, quản lý và sử dụng các khóa mật mã (cryptographic keys).
  Triggers: "keyvault keys rust", "KeyClient rust", "create key rust", "encrypt rust", "sign rust".
package: azure_security_keyvault_keys
---

# Azure Key Vault Keys SDK cho Rust

Thư viện client cho Azure Key Vault Keys — lưu trữ và quản lý an toàn các khóa mật mã.

## Cài đặt

```sh
cargo add azure_security_keyvault_keys azure_identity
```

## Biến Môi trường

```bash
AZURE_KEYVAULT_URL=https://<vault-name>.vault.azure.net/
```

## Xác thực

```rust
use azure_identity::DeveloperToolsCredential;
use azure_security_keyvault_keys::KeyClient;

let credential = DeveloperToolsCredential::new(None)?;
let client = KeyClient::new(
    "https://<vault-name>.vault.azure.net/",
    credential.clone(),
    None,
)?;
```

## Các Loại Khóa

| Loại    | Mô tả                                     |
| ------- | ----------------------------------------- |
| RSA     | Khóa RSA (2048, 3072, 4096 bits)          |
| EC      | Khóa Elliptic curve (P-256, P-384, P-521) |
| RSA-HSM | Khóa RSA được bảo vệ bởi HSM              |
| EC-HSM  | Khóa EC được bảo vệ bởi HSM               |

## Các Thao tác Cốt lõi

### Lấy Khóa (Get Key)

```rust
let key = client
    .get_key("key-name", None)
    .await?
    .into_model()?;

println!("Key ID: {:?}", key.key.as_ref().map(|k| &k.kid));
```

### Tạo Khóa (Create Key)

```rust
use azure_security_keyvault_keys::models::{CreateKeyParameters, KeyType};

let params = CreateKeyParameters {
    kty: KeyType::Rsa,
    key_size: Some(2048),
    ..Default::default()
};

let key = client
    .create_key("key-name", params.try_into()?, None)
    .await?
    .into_model()?;
```

### Tạo Khóa EC (Create EC Key)

```rust
use azure_security_keyvault_keys::models::{CreateKeyParameters, KeyType, CurveName};

let params = CreateKeyParameters {
    kty: KeyType::Ec,
    curve: Some(CurveName::P256),
    ..Default::default()
};

let key = client
    .create_key("ec-key", params.try_into()?, None)
    .await?
    .into_model()?;
```

### Xóa Khóa (Delete Key)

```rust
client.delete_key("key-name", None).await?;
```

### Liệt kê Khóa (List Keys)

```rust
use azure_security_keyvault_keys::ResourceExt;
use futures::TryStreamExt;

let mut pager = client.list_key_properties(None)?.into_stream();
while let Some(key) = pager.try_next().await? {
    let name = key.resource_id()?.name;
    println!("Key: {}", name);
}
```

### Sao lưu Khóa (Backup Key)

```rust
let backup = client.backup_key("key-name", None).await?;
// Lưu trữ backup.value một cách an toàn
```

### Khôi phục Khóa (Restore Key)

```rust
use azure_security_keyvault_keys::models::RestoreKeyParameters;

let params = RestoreKeyParameters {
    key_bundle_backup: backup_bytes,
};

client.restore_key(params.try_into()?, None).await?;
```

## Các Thao tác Mật mã (Cryptographic Operations)

Key Vault có thể thực hiện các thao tác mật mã mà không để lộ khóa riêng tư (private key):

```rust
// Cho các thao tác mật mã, sử dụng các operations của khóa
// Các thao tác khả dụng phụ thuộc vào loại khóa và quyền:
// - encrypt/decrypt (RSA)
// - sign/verify (RSA, EC)
// - wrapKey/unwrapKey (RSA)
```

## Thực hành Tốt nhất

1. **Sử dụng xác thực Entra ID** — `DeveloperToolsCredential` cho dev, `ManagedIdentityCredential` cho sản xuất
2. **Sử dụng khóa HSM cho các tải công việc nhạy cảm** — khóa được bảo vệ bằng phần cứng
3. **Sử dụng EC để ký (signing)** — hiệu quả hơn RSA
4. **Sử dụng RSA để mã hóa** — khi mã hóa dữ liệu
5. **Sao lưu khóa** — để phục hồi sau thảm họa
6. **Bật soft delete** — bắt buộc cho các vault sản xuất
7. **Sử dụng xoay vòng khóa (key rotation)** — tạo các phiên bản mới định kỳ

## Quyền RBAC

Gán các vai trò Key Vault sau:

- `Key Vault Crypto User` — sử dụng khóa cho các thao tác mật mã
- `Key Vault Crypto Officer` — toàn quyền CRUD trên khóa

## Liên kết Tham khảo

| Tài nguyên    | Liên kết                                                                                        |
| ------------- | ----------------------------------------------------------------------------------------------- |
| Tham khảo API | https://docs.rs/azure_security_keyvault_keys                                                    |
| Mã nguồn      | https://github.com/Azure/azure-sdk-for-rust/tree/main/sdk/keyvault/azure_security_keyvault_keys |
| crates.io     | https://crates.io/crates/azure_security_keyvault_keys                                           |
