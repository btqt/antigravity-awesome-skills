---
name: azure-keyvault-certificates-rust
description: |
  Azure Key Vault Certificates SDK cho Rust. Sử dụng để tạo, import, và quản lý các chứng chỉ.
  Triggers: "keyvault certificates rust", "CertificateClient rust", "create certificate rust", "import certificate rust".
package: azure_security_keyvault_certificates
---

# Azure Key Vault Certificates SDK cho Rust

Thư viện client cho Azure Key Vault Certificates — lưu trữ và quản lý chứng chỉ (certificates) an toàn.

## Cài đặt

```sh
cargo add azure_security_keyvault_certificates azure_identity
```

## Biến Môi trường

```bash
AZURE_KEYVAULT_URL=https://<vault-name>.vault.azure.net/
```

## Xác thực

```rust
use azure_identity::DeveloperToolsCredential;
use azure_security_keyvault_certificates::CertificateClient;

let credential = DeveloperToolsCredential::new(None)?;
let client = CertificateClient::new(
    "https://<vault-name>.vault.azure.net/",
    credential.clone(),
    None,
)?;
```

## Các Thao tác Cốt lõi

### Lấy Chứng chỉ (Get Certificate)

```rust
use azure_core::base64;

let certificate = client
    .get_certificate("certificate-name", None)
    .await?
    .into_model()?;

println!(
    "Thumbprint: {:?}",
    certificate.x509_thumbprint.map(base64::encode_url_safe)
);
```

### Tạo Chứng chỉ (Create Certificate)

```rust
use azure_security_keyvault_certificates::models::{
    CreateCertificateParameters, CertificatePolicy,
    IssuerParameters, X509CertificateProperties,
};

let policy = CertificatePolicy {
    issuer_parameters: Some(IssuerParameters {
        name: Some("Self".into()),
        ..Default::default()
    }),
    x509_certificate_properties: Some(X509CertificateProperties {
        subject: Some("CN=example.com".into()),
        ..Default::default()
    }),
    ..Default::default()
};

let params = CreateCertificateParameters {
    certificate_policy: Some(policy),
    ..Default::default()
};

let operation = client
    .create_certificate("cert-name", params.try_into()?, None)
    .await?;
```

### Import Chứng chỉ

```rust
use azure_security_keyvault_certificates::models::ImportCertificateParameters;

let params = ImportCertificateParameters {
    base64_encoded_certificate: Some(base64_cert_data),
    password: Some("optional-password".into()),
    ..Default::default()
};

let certificate = client
    .import_certificate("cert-name", params.try_into()?, None)
    .await?
    .into_model()?;
```

### Xóa Chứng chỉ (Delete Certificate)

```rust
client.delete_certificate("certificate-name", None).await?;
```

### Liệt kê Chứng chỉ (List Certificates)

```rust
use azure_security_keyvault_certificates::ResourceExt;
use futures::TryStreamExt;

let mut pager = client.list_certificate_properties(None)?.into_stream();
while let Some(cert) = pager.try_next().await? {
    let name = cert.resource_id()?.name;
    println!("Certificate: {}", name);
}
```

### Lấy Chính sách Chứng chỉ (Get Certificate Policy)

```rust
let policy = client
    .get_certificate_policy("certificate-name", None)
    .await?
    .into_model()?;
```

### Cập nhật Chính sách Chứng chỉ (Update Certificate Policy)

```rust
use azure_security_keyvault_certificates::models::UpdateCertificatePolicyParameters;

let params = UpdateCertificatePolicyParameters {
    // Cập nhật các thuộc tính policy
    ..Default::default()
};

client
    .update_certificate_policy("cert-name", params.try_into()?, None)
    .await?;
```

## Vòng đời Chứng chỉ (Certificate Lifecycle)

1. **Create** — tạo chứng chỉ mới với chính sách (policy)
2. **Import** — import chứng chỉ PFX/PEM hiện có
3. **Get** — lấy chứng chỉ (chỉ public key)
4. **Update** — sửa đổi các thuộc tính chứng chỉ
5. **Delete** — xóa mềm (soft delete, có thể khôi phục)
6. **Purge** — xóa vĩnh viễn

## Thực hành Tốt nhất

1. **Sử dụng xác thực Entra ID** — `DeveloperToolsCredential` cho dev
2. **Sử dụng managed certificates** — tự động gia hạn với các nhà phát hành hỗ trợ
3. **Đặt thời hạn hiệu lực hợp lý** — cân bằng giữa bảo mật và bảo trì
4. **Sử dụng chính sách chứng chỉ** — xác định các thuộc tính gia hạn và khóa
5. **Giám sát hết hạn** — thiết lập cảnh báo cho các chứng chỉ sắp hết hạn
6. **Bật soft delete** — bắt buộc cho các vault sản xuất

## Quyền RBAC

Gán các vai trò Key Vault sau:

- `Key Vault Certificates Officer` — toàn quyền CRUD trên chứng chỉ
- `Key Vault Reader` — đọc metadata chứng chỉ

## Liên kết Tham khảo

| Tài nguyên    | Liên kết                                                                                                |
| ------------- | ------------------------------------------------------------------------------------------------------- |
| Tham khảo API | https://docs.rs/azure_security_keyvault_certificates                                                    |
| Mã nguồn      | https://github.com/Azure/azure-sdk-for-rust/tree/main/sdk/keyvault/azure_security_keyvault_certificates |
| crates.io     | https://crates.io/crates/azure_security_keyvault_certificates                                           |
