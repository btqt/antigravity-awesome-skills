---
name: azure-storage-file-datalake-py
description: |
  Azure Data Lake Storage Gen2 SDK cho Python. Sử dụng cho hệ thống tệp phân cấp, phân tích dữ liệu lớn, và các hoạt động file/directory.
  Triggers: "data lake", "DataLakeServiceClient", "FileSystemClient", "ADLS Gen2", "hierarchical namespace".
package: azure-storage-file-datalake
---

# Azure Data Lake Storage Gen2 SDK cho Python

Hệ thống tệp phân cấp cho khối lượng công việc phân tích dữ liệu lớn.

## Cài đặt

```bash
pip install azure-storage-file-datalake azure-identity
```

## Biến Môi trường

```bash
AZURE_STORAGE_ACCOUNT_URL=https://<account>.dfs.core.windows.net
```

## Xác thực

```python
from azure.identity import DefaultAzureCredential
from azure.storage.filedatalake import DataLakeServiceClient

credential = DefaultAzureCredential()
account_url = "https://<account>.dfs.core.windows.net"

service_client = DataLakeServiceClient(account_url=account_url, credential=credential)
```

## Phân cấp Client

| Client                    | Mục đích                               |
| ------------------------- | -------------------------------------- |
| `DataLakeServiceClient`   | Các hoạt động cấp tài khoản            |
| `FileSystemClient`        | Các hoạt động Container (hệ thống tệp) |
| `DataLakeDirectoryClient` | Các hoạt động thư mục                  |
| `DataLakeFileClient`      | Các hoạt động tệp                      |

## Các Hoạt động Hệ thống Tệp

```python
# Tạo file system (container)
file_system_client = service_client.create_file_system("myfilesystem")

# Lấy file system hiện có
file_system_client = service_client.get_file_system_client("myfilesystem")

# Xóa
service_client.delete_file_system("myfilesystem")

# Liệt kê file systems
for fs in service_client.list_file_systems():
    print(fs.name)
```

## Các Hoạt động Thư mục

```python
file_system_client = service_client.get_file_system_client("myfilesystem")

# Tạo thư mục
directory_client = file_system_client.create_directory("mydir")

# Tạo các thư mục lồng nhau
directory_client = file_system_client.create_directory("path/to/nested/dir")

# Lấy directory client
directory_client = file_system_client.get_directory_client("mydir")

# Xóa thư mục
directory_client.delete_directory()

# Đổi tên/di chuyển thư mục
directory_client.rename_directory(new_name="myfilesystem/newname")
```

## Các Hoạt động Tệp

### Tải lên Tệp (Upload)

```python
# Lấy file client
file_client = file_system_client.get_file_client("path/to/file.txt")

# Upload từ file cục bộ
with open("local-file.txt", "rb") as data:
    file_client.upload_data(data, overwrite=True)

# Upload bytes
file_client.upload_data(b"Hello, Data Lake!", overwrite=True)

# Nối dữ liệu (cho tệp lớn)
file_client.append_data(data=b"chunk1", offset=0, length=6)
file_client.append_data(data=b"chunk2", offset=6, length=6)
file_client.flush_data(12)  # Commit dữ liệu
```

### Tải xuống Tệp (Download)

```python
file_client = file_system_client.get_file_client("path/to/file.txt")

# Download toàn bộ nội dung
download = file_client.download_file()
content = download.readall()

# Download xuống file
with open("downloaded.txt", "wb") as f:
    download = file_client.download_file()
    download.readinto(f)

# Download theo phạm vi
download = file_client.download_file(offset=0, length=100)
```

### Xóa Tệp

```python
file_client.delete_file()
```

## Liệt kê Nội dung

```python
# Liệt kê đường dẫn (files và directories)
for path in file_system_client.get_paths():
    print(f"{'DIR' if path.is_directory else 'FILE'}: {path.name}")

# Liệt kê đường dẫn trong thư mục
for path in file_system_client.get_paths(path="mydir"):
    print(path.name)

# Liệt kê đệ quy
for path in file_system_client.get_paths(path="mydir", recursive=True):
    print(path.name)
```

## Thuộc tính Tệp/Thư mục

```python
# Lấy thuộc tính
properties = file_client.get_file_properties()
print(f"Size: {properties.size}")
print(f"Last modified: {properties.last_modified}")

# Đặt metadata
file_client.set_metadata(metadata={"processed": "true"})
```

## Kiểm soát Truy cập (ACL)

```python
# Lấy ACL
acl = directory_client.get_access_control()
print(f"Owner: {acl['owner']}")
print(f"Permissions: {acl['permissions']}")

# Đặt ACL
directory_client.set_access_control(
    owner="user-id",
    permissions="rwxr-x---"
)

# Cập nhật các mục ACL
from azure.storage.filedatalake import AccessControlChangeResult
directory_client.update_access_control_recursive(
    acl="user:user-id:rwx"
)
```

## Async Client

```python
from azure.storage.filedatalake.aio import DataLakeServiceClient
from azure.identity.aio import DefaultAzureCredential

async def datalake_operations():
    credential = DefaultAzureCredential()

    async with DataLakeServiceClient(
        account_url="https://<account>.dfs.core.windows.net",
        credential=credential
    ) as service_client:
        file_system_client = service_client.get_file_system_client("myfilesystem")
        file_client = file_system_client.get_file_client("test.txt")

        await file_client.upload_data(b"async content", overwrite=True)

        download = await file_client.download_file()
        content = await download.readall()

import asyncio
asyncio.run(datalake_operations())
```

## Thực hành Tốt nhất

1. **Sử dụng không gian tên phân cấp** cho ngữ nghĩa hệ thống tệp
2. **Sử dụng `append_data` + `flush_data`** cho upload tệp lớn
3. **Đặt ACLs ở cấp thư mục** và kế thừa cho con
4. **Sử dụng async client** cho các kịch bản thông lượng cao
5. **Sử dụng `get_paths` với `recursive=True`** để liệt kê thư mục đầy đủ
6. **Đặt metadata** cho các thuộc tính tệp tùy chỉnh
7. **Cân nhắc Blob API** cho các trường hợp sử dụng lưu trữ đối tượng đơn giản
