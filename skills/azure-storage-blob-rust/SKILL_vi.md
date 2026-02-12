---
name: azure-storage-blob-rust
description: |
  Azure Blob Storage SDK cho Rust. Sử dụng để upload, download, và quản lý blobs và containers.
  Triggers: "blob storage rust", "BlobClient rust", "upload blob rust", "download blob rust", "container rust".
package: azure_storage_blob
---

# Azure Blob Storage SDK cho Rust

Thư viện client cho Azure Blob Storage — giải pháp lưu trữ đối tượng cho đám mây của Microsoft.

## Cài đặt

```sh
cargo add azure_storage_blob azure_identity
```

## Biến Môi trường

```bash
AZURE_STORAGE_ACCOUNT_NAME=<storage-account-name>
# Endpoint: https://<account>.blob.core.windows.net/
```

## Xác thực

```rust
use azure_identity::DeveloperToolsCredential;
use azure_storage_blob::{BlobClient, BlobClientOptions};

let credential = DeveloperToolsCredential::new(None)?;
let blob_client = BlobClient::new(
    "https://<account>.blob.core.windows.net/",
    "container-name",
    "blob-name",
    Some(credential),
    Some(BlobClientOptions::default()),
)?;
```

## Các Loại Client

| Client                | Mục đích                                        |
| --------------------- | ----------------------------------------------- |
| `BlobServiceClient`   | Các hoạt động cấp tài khoản, liệt kê containers |
| `BlobContainerClient` | Các hoạt động container, liệt kê blobs          |
| `BlobClient`          | Các hoạt động blob riêng lẻ                     |

## Các Hoạt động Cốt lõi

### Tải lên Blob (Upload)

```rust
use azure_core::http::RequestContent;

let data = b"hello world";
blob_client
    .upload(
        RequestContent::from(data.to_vec()),
        false,  // ghi đè
        u64::try_from(data.len())?,
        None,
    )
    .await?;
```

### Tải xuống Blob (Download)

```rust
let response = blob_client.download(None).await?;
let content = response.into_body().collect_bytes().await?;
println!("Content: {:?}", content);
```

### Lấy Thuộc tính Blob

```rust
let properties = blob_client.get_properties(None).await?;
println!("Content-Length: {:?}", properties.content_length);
```

### Xóa Blob

```rust
blob_client.delete(None).await?;
```

## Các Hoạt động Container

```rust
use azure_storage_blob::BlobContainerClient;

let container_client = BlobContainerClient::new(
    "https://<account>.blob.core.windows.net/",
    "container-name",
    Some(credential),
    None,
)?;

// Tạo container
container_client.create(None).await?;

// Liệt kê blobs
let mut pager = container_client.list_blobs(None)?;
while let Some(blob) = pager.try_next().await? {
    println!("Blob: {}", blob.name);
}
```

## Thực hành Tốt nhất

1. **Sử dụng xác thực Entra ID** — `DeveloperToolsCredential` cho dev, `ManagedIdentityCredential` cho production
2. **Chỉ định độ dài nội dung** — bắt buộc khi upload
3. **Sử dụng `RequestContent::from()`** — để bao bọc dữ liệu upload
4. **Xử lý các hoạt động async** — sử dụng `tokio` runtime
5. **Kiểm tra quyền RBAC** — đảm bảo vai trò "Storage Blob Data Contributor"

## Quyền RBAC

Đối với xác thực Entra ID, hãy gán một trong các vai trò sau:

- `Storage Blob Data Reader` — chỉ đọc
- `Storage Blob Data Contributor` — đọc/ghi
- `Storage Blob Data Owner` — toàn quyền truy cập bao gồm RBAC

## Liên kết Tham khảo

| Tài nguyên    | Liên kết                                                                             |
| ------------- | ------------------------------------------------------------------------------------ |
| API Reference | https://docs.rs/azure_storage_blob                                                   |
| Source Code   | https://github.com/Azure/azure-sdk-for-rust/tree/main/sdk/storage/azure_storage_blob |
| crates.io     | https://crates.io/crates/azure_storage_blob                                          |
