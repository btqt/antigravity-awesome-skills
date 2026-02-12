---
name: azure-storage-blob-py
description: |
  Azure Blob Storage SDK cho Python. Sử dụng để upload, download, liệt kê blobs, quản lý containers, và vòng đời blob.
  Triggers: "blob storage", "BlobServiceClient", "ContainerClient", "BlobClient", "upload blob", "download blob".
package: azure-storage-blob
---

# Azure Blob Storage SDK cho Python

Thư viện client cho Azure Blob Storage — lưu trữ đối tượng cho dữ liệu phi cấu trúc.

## Cài đặt

```bash
pip install azure-storage-blob azure-identity
```

## Biến Môi trường

```bash
AZURE_STORAGE_ACCOUNT_NAME=<your-storage-account>
# Hoặc sử dụng URL đầy đủ
AZURE_STORAGE_ACCOUNT_URL=https://<account>.blob.core.windows.net
```

## Xác thực

```python
from azure.identity import DefaultAzureCredential
from azure.storage.blob import BlobServiceClient

credential = DefaultAzureCredential()
account_url = "https://<account>.blob.core.windows.net"

blob_service_client = BlobServiceClient(account_url, credential=credential)
```

## Phân cấp Client

| Client              | Mục đích                    | Lấy Từ                                       |
| ------------------- | --------------------------- | -------------------------------------------- |
| `BlobServiceClient` | Các hoạt động cấp tài khoản | Khởi tạo trực tiếp                           |
| `ContainerClient`   | Các hoạt động Container     | `blob_service_client.get_container_client()` |
| `BlobClient`        | Các hoạt động Blob đơn lẻ   | `container_client.get_blob_client()`         |

## Quy trình Công việc Cốt lõi

### Tạo Container

```python
container_client = blob_service_client.get_container_client("mycontainer")
container_client.create_container()
```

### Tải lên Blob (Upload)

```python
# Từ đường dẫn file
blob_client = blob_service_client.get_blob_client(
    container="mycontainer",
    blob="sample.txt"
)

with open("./local-file.txt", "rb") as data:
    blob_client.upload_blob(data, overwrite=True)

# Từ bytes/string
blob_client.upload_blob(b"Hello, World!", overwrite=True)

# Từ stream
import io
stream = io.BytesIO(b"Stream content")
blob_client.upload_blob(stream, overwrite=True)
```

### Tải xuống Blob (Download)

```python
blob_client = blob_service_client.get_blob_client(
    container="mycontainer",
    blob="sample.txt"
)

# Xuống file
with open("./downloaded.txt", "wb") as file:
    download_stream = blob_client.download_blob()
    file.write(download_stream.readall())

# Vào bô nhớ
download_stream = blob_client.download_blob()
content = download_stream.readall()  # bytes

# Đọc vào buffer hiện có
stream = io.BytesIO()
num_bytes = blob_client.download_blob().readinto(stream)
```

### Liệt kê Blobs

```python
container_client = blob_service_client.get_container_client("mycontainer")

# Liệt kê tất cả blobs
for blob in container_client.list_blobs():
    print(f"{blob.name} - {blob.size} bytes")

# Liệt kê với prefix (giống thư mục)
for blob in container_client.list_blobs(name_starts_with="logs/"):
    print(blob.name)

# Duyệt phân cấp blob (thư mục ảo)
for item in container_client.walk_blobs(delimiter="/"):
    if item.get("prefix"):
        print(f"Directory: {item['prefix']}")
    else:
        print(f"Blob: {item.name}")
```

### Xóa Blob

```python
blob_client.delete_blob()

# Xóa cùng snapshots
blob_client.delete_blob(delete_snapshots="include")
```

## Tinh chỉnh Hiệu suất (Performance Tuning)

```python
# Cấu hình kích thước chunk cho uploads/downloads lớn
blob_client = BlobClient(
    account_url=account_url,
    container_name="mycontainer",
    blob_name="large-file.zip",
    credential=credential,
    max_block_size=4 * 1024 * 1024,  # 4 MiB blocks
    max_single_put_size=64 * 1024 * 1024  # 64 MiB giới hạn upload đơn
)

# Upload song song
blob_client.upload_blob(data, max_concurrency=4)

# Download song song
download_stream = blob_client.download_blob(max_concurrency=4)
```

## SAS Tokens

```python
from datetime import datetime, timedelta, timezone
from azure.storage.blob import generate_blob_sas, BlobSasPermissions

sas_token = generate_blob_sas(
    account_name="<account>",
    container_name="mycontainer",
    blob_name="sample.txt",
    account_key="<account-key>",  # Hoặc sử dụng user delegation key
    permission=BlobSasPermissions(read=True),
    expiry=datetime.now(timezone.utc) + timedelta(hours=1)
)

# Sử dụng SAS token
blob_url = f"https://<account>.blob.core.windows.net/mycontainer/sample.txt?{sas_token}"
```

## Thuộc tính Blob và Metadata

```python
# Lấy thuộc tính
properties = blob_client.get_blob_properties()
print(f"Size: {properties.size}")
print(f"Content-Type: {properties.content_settings.content_type}")
print(f"Last modified: {properties.last_modified}")

# Đặt metadata
blob_client.set_blob_metadata(metadata={"category": "logs", "year": "2024"})

# Đặt content type
from azure.storage.blob import ContentSettings
blob_client.set_http_headers(
    content_settings=ContentSettings(content_type="application/json")
)
```

## Async Client

```python
from azure.identity.aio import DefaultAzureCredential
from azure.storage.blob.aio import BlobServiceClient

async def upload_async():
    credential = DefaultAzureCredential()

    async with BlobServiceClient(account_url, credential=credential) as client:
        blob_client = client.get_blob_client("mycontainer", "sample.txt")

        with open("./file.txt", "rb") as data:
            await blob_client.upload_blob(data, overwrite=True)

# Download async
async def download_async():
    async with BlobServiceClient(account_url, credential=credential) as client:
        blob_client = client.get_blob_client("mycontainer", "sample.txt")

        stream = await blob_client.download_blob()
        data = await stream.readall()
```

## Thực hành Tốt nhất

1. **Sử dụng DefaultAzureCredential** thay vì chuỗi kết nối
2. **Sử dụng context managers** cho async clients
3. **Đặt `overwrite=True`** rõ ràng khi upload lại
4. **Sử dụng `max_concurrency`** cho việc truyền file lớn
5. **Ưu tiên `readinto()`** hơn `readall()` để tiết kiệm bộ nhớ
6. **Sử dụng `walk_blobs()`** cho việc liệt kê phân cấp
7. **Đặt content types phù hợp** cho blobs phục vụ web
