---
name: azure-storage-file-share-py
description: |
  Azure Storage File Share SDK cho Python. Sử dụng cho SMB file shares, thư mục, và các hoạt động tệp trên đám mây.
  Triggers: "azure-storage-file-share", "ShareServiceClient", "ShareClient", "file share", "SMB".
---

# Azure Storage File Share SDK cho Python

Quản lý SMB file shares cho các kịch bản cloud-native và lift-and-shift.

## Cài đặt

```bash
pip install azure-storage-file-share
```

## Biến Môi trường

```bash
AZURE_STORAGE_CONNECTION_STRING=DefaultEndpointsProtocol=https;AccountName=...;AccountKey=...
# Hoặc
AZURE_STORAGE_ACCOUNT_URL=https://<account>.file.core.windows.net
```

## Xác thực

### Chuỗi Kết nối

```python
from azure.storage.fileshare import ShareServiceClient

service = ShareServiceClient.from_connection_string(
    os.environ["AZURE_STORAGE_CONNECTION_STRING"]
)
```

### Entra ID

```python
from azure.storage.fileshare import ShareServiceClient
from azure.identity import DefaultAzureCredential

service = ShareServiceClient(
    account_url=os.environ["AZURE_STORAGE_ACCOUNT_URL"],
    credential=DefaultAzureCredential()
)
```

## Các Hoạt động Share

### Tạo Share

```python
share = service.create_share("my-share")
```

### Liệt kê Shares

```python
for share in service.list_shares():
    print(f"{share.name}: {share.quota} GB")
```

### Lấy Share Client

```python
share_client = service.get_share_client("my-share")
```

### Xóa Share

```python
service.delete_share("my-share")
```

## Các Hoạt động Thư mục

### Tạo Thư mục

```python
share_client = service.get_share_client("my-share")
share_client.create_directory("my-directory")

# Thư mục lồng nhau
share_client.create_directory("my-directory/sub-directory")
```

### Liệt kê Thư mục và Tệp

```python
directory_client = share_client.get_directory_client("my-directory")

for item in directory_client.list_directories_and_files():
    if item["is_directory"]:
        print(f"[DIR] {item['name']}")
    else:
        print(f"[FILE] {item['name']} ({item['size']} bytes)")
```

### Xóa Thư mục

```python
share_client.delete_directory("my-directory")
```

## Các Hoạt động Tệp

### Tải lên Tệp (Upload)

```python
file_client = share_client.get_file_client("my-directory/file.txt")

# Từ chuỗi
file_client.upload_file("Hello, World!")

# Từ file
with open("local-file.txt", "rb") as f:
    file_client.upload_file(f)

# Từ bytes
file_client.upload_file(b"Binary content")
```

### Tải xuống Tệp (Download)

```python
file_client = share_client.get_file_client("my-directory/file.txt")

# Thành bytes
data = file_client.download_file().readall()

# Vào file
with open("downloaded.txt", "wb") as f:
    data = file_client.download_file()
    data.readinto(f)

# Stream chunks
download = file_client.download_file()
for chunk in download.chunks():
    process(chunk)
```

### Lấy Thuộc tính Tệp

```python
properties = file_client.get_file_properties()
print(f"Size: {properties.size}")
print(f"Content type: {properties.content_settings.content_type}")
print(f"Last modified: {properties.last_modified}")
```

### Xóa Tệp

```python
file_client.delete_file()
```

### Copy Tệp

```python
source_url = "https://account.file.core.windows.net/share/source.txt"
dest_client = share_client.get_file_client("destination.txt")
dest_client.start_copy_from_url(source_url)
```

## Các Hoạt động theo Phạm vi (Range Operations)

### Upload Range

```python
# Upload vào phạm vi cụ thể
file_client.upload_range(data=b"content", offset=0, length=7)
```

### Download Range

```python
# Download phạm vi cụ thể
download = file_client.download_file(offset=0, length=100)
data = download.readall()
```

## Các Hoạt động Snapshot

### Tạo Snapshot

```python
snapshot = share_client.create_snapshot()
print(f"Snapshot: {snapshot['snapshot']}")
```

### Truy cập Snapshot

```python
snapshot_client = service.get_share_client(
    "my-share",
    snapshot=snapshot["snapshot"]
)
```

## Async Client

```python
from azure.storage.fileshare.aio import ShareServiceClient
from azure.identity.aio import DefaultAzureCredential

async def upload_file():
    credential = DefaultAzureCredential()
    service = ShareServiceClient(account_url, credential=credential)

    share = service.get_share_client("my-share")
    file_client = share.get_file_client("test.txt")

    await file_client.upload_file("Hello!")

    await service.close()
    await credential.close()
```

## Các Loại Client

| Client                 | Mục đích                    |
| ---------------------- | --------------------------- |
| `ShareServiceClient`   | Các hoạt động cấp tài khoản |
| `ShareClient`          | Các hoạt động Share         |
| `ShareDirectoryClient` | Các hoạt động thư mục       |
| `ShareFileClient`      | Các hoạt động tệp           |

## Thực hành Tốt nhất

1. **Sử dụng chuỗi kết nối** cho thiết lập đơn giản nhất
2. **Sử dụng Entra ID** cho production với RBAC
3. **Stream các tệp lớn** sử dụng chunks() để tránh vấn đề bộ nhớ
4. **Tạo snapshots** trước những thay đổi lớn
5. **Đặt hạn ngạch (quotas)** để ngăn chi phí lưu trữ bất ngờ
6. **Sử dụng ranges** cho cập nhật tệp một phần
7. **Đóng các async clients** một cách rõ ràng
