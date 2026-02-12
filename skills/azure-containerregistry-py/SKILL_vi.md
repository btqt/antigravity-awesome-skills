---
name: azure-containerregistry-py
description: |
  Azure Container Registry SDK cho Python. Sử dụng để quản lý hình ảnh container (container images), artifacts, và repositories.
  Kích hoạt: "azure-containerregistry", "ContainerRegistryClient", "container images", "docker registry", "ACR".
package: azure-containerregistry
---

# Azure Container Registry SDK cho Python

Quản lý hình ảnh container, artifacts, và repositories trong Azure Container Registry.

## Cài đặt

```bash
pip install azure-containerregistry
```

## Biến Môi trường

```bash
AZURE_CONTAINERREGISTRY_ENDPOINT=https://<registry-name>.azurecr.io
```

## Xác thực

### Entra ID (Khuyên dùng)

```python
from azure.containerregistry import ContainerRegistryClient
from azure.identity import DefaultAzureCredential

client = ContainerRegistryClient(
    endpoint=os.environ["AZURE_CONTAINERREGISTRY_ENDPOINT"],
    credential=DefaultAzureCredential()
)
```

### Truy cập Ẩn danh (Public Registry)

```python
from azure.containerregistry import ContainerRegistryClient

client = ContainerRegistryClient(
    endpoint="https://mcr.microsoft.com",
    credential=None,
    audience="https://mcr.microsoft.com"
)
```

## Liệt kê Repositories

```python
client = ContainerRegistryClient(endpoint, DefaultAzureCredential())

for repository in client.list_repository_names():
    print(repository)
```

## Các Thao tác Repository

### Lấy Thuộc tính Repository

```python
properties = client.get_repository_properties("my-image")
print(f"Đã tạo: {properties.created_on}")
print(f"Đã sửa đổi: {properties.last_updated_on}")
print(f"Manifests: {properties.manifest_count}")
print(f"Tags: {properties.tag_count}")
```

### Cập nhật Thuộc tính Repository

```python
from azure.containerregistry import RepositoryProperties

client.update_repository_properties(
    "my-image",
    properties=RepositoryProperties(
        can_delete=False,
        can_write=False
    )
)
```

### Xóa Repository

```python
client.delete_repository("my-image")
```

## Liệt kê Tags

```python
for tag in client.list_tag_properties("my-image"):
    print(f"{tag.name}: {tag.created_on}")
```

### Lọc theo Thứ tự

```python
from azure.containerregistry import ArtifactTagOrder

# Mới nhất trước
for tag in client.list_tag_properties(
    "my-image",
    order_by=ArtifactTagOrder.LAST_UPDATED_ON_DESCENDING
):
    print(f"{tag.name}: {tag.last_updated_on}")
```

## Các Thao tác Manifest

### Liệt kê Manifests

```python
from azure.containerregistry import ArtifactManifestOrder

for manifest in client.list_manifest_properties(
    "my-image",
    order_by=ArtifactManifestOrder.LAST_UPDATED_ON_DESCENDING
):
    print(f"Digest: {manifest.digest}")
    print(f"Tags: {manifest.tags}")
    print(f"Kích thước: {manifest.size_in_bytes}")
```

### Lấy Thuộc tính Manifest

```python
manifest = client.get_manifest_properties("my-image", "latest")
print(f"Digest: {manifest.digest}")
print(f"Kiến trúc: {manifest.architecture}")
print(f"Hệ điều hành: {manifest.operating_system}")
```

### Cập nhật Thuộc tính Manifest

```python
from azure.containerregistry import ArtifactManifestProperties

client.update_manifest_properties(
    "my-image",
    "latest",
    properties=ArtifactManifestProperties(
        can_delete=False,
        can_write=False
    )
)
```

### Xóa Manifest

```python
# Xóa theo digest
client.delete_manifest("my-image", "sha256:abc123...")

# Xóa theo tag
manifest = client.get_manifest_properties("my-image", "old-tag")
client.delete_manifest("my-image", manifest.digest)
```

## Các Thao tác Tag

### Lấy Thuộc tính Tag

```python
tag = client.get_tag_properties("my-image", "latest")
print(f"Digest: {tag.digest}")
print(f"Đã tạo: {tag.created_on}")
```

### Xóa Tag

```python
client.delete_tag("my-image", "old-tag")
```

## Tải lên và Tải xuống Artifacts

```python
from azure.containerregistry import ContainerRegistryClient

client = ContainerRegistryClient(endpoint, DefaultAzureCredential())

# Tải xuống manifest
manifest = client.download_manifest("my-image", "latest")
print(f"Media type: {manifest.media_type}")
print(f"Digest: {manifest.digest}")

# Tải xuống blob
blob = client.download_blob("my-image", "sha256:abc123...")
with open("layer.tar.gz", "wb") as f:
    for chunk in blob:
        f.write(chunk)
```

## Client Bất đồng bộ (Async Client)

```python
from azure.containerregistry.aio import ContainerRegistryClient
from azure.identity.aio import DefaultAzureCredential

async def list_repos():
    credential = DefaultAzureCredential()
    client = ContainerRegistryClient(endpoint, credential)

    async for repo in client.list_repository_names():
        print(repo)

    await client.close()
    await credential.close()
```

## Dọn dẹp Hình ảnh Cũ

```python
from datetime import datetime, timedelta, timezone

cutoff = datetime.now(timezone.utc) - timedelta(days=30)

for manifest in client.list_manifest_properties("my-image"):
    if manifest.last_updated_on < cutoff and not manifest.tags:
        print(f"Đang xóa {manifest.digest}")
        client.delete_manifest("my-image", manifest.digest)
```

## Các Thao tác Client

| Thao tác                    | Mô tả                              |
| --------------------------- | ---------------------------------- |
| `list_repository_names`     | Liệt kê tất cả repositories        |
| `get_repository_properties` | Lấy metadata của repository        |
| `delete_repository`         | Xóa repository và tất cả hình ảnh  |
| `list_tag_properties`       | Liệt kê tags trong repository      |
| `get_tag_properties`        | Lấy metadata của tag               |
| `delete_tag`                | Xóa tag cụ thể                     |
| `list_manifest_properties`  | Liệt kê manifests trong repository |
| `get_manifest_properties`   | Lấy metadata của manifest          |
| `delete_manifest`           | Xóa manifest theo digest           |
| `download_manifest`         | Tải xuống nội dung manifest        |
| `download_blob`             | Tải xuống layer blob               |

## Thực hành Tốt nhất

1. **Sử dụng Entra ID** để xác thực trong môi trường sản xuất
2. **Xóa theo digest** thay vì tag để tránh hình ảnh mồ côi (orphaned images)
3. **Khóa hình ảnh sản xuất** với can_delete=False
4. **Dọn dẹp manifests không có tag** thường xuyên
5. **Sử dụng async client** cho các thao tác thông lượng cao
6. **Sắp xếp theo last_updated** để tìm hình ảnh mới nhất/cũ nhất
7. **Kiểm tra manifest.tags** trước khi xóa để tránh xóa hình ảnh được gắn thẻ
