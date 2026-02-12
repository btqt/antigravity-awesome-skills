---
name: azure-appconfiguration-py
description: |
  Azure App Configuration SDK cho Python. Sử dụng để quản lý cấu hình tập trung, feature flags, và các cài đặt động.
  Kích hoạt: "azure-appconfiguration", "AzureAppConfigurationClient", "feature flags", "configuration", "key-value settings".
package: azure-appconfiguration
---

# Azure App Configuration SDK cho Python

Quản lý cấu hình tập trung với feature flags và các cài đặt động.

## Cài đặt

```bash
pip install azure-appconfiguration
```

## Biến Môi trường

```bash
AZURE_APPCONFIGURATION_CONNECTION_STRING=Endpoint=https://<name>.azconfig.io;Id=...;Secret=...
# Hoặc cho Entra ID:
AZURE_APPCONFIGURATION_ENDPOINT=https://<name>.azconfig.io
```

## Xác thực

### Connection String

```python
from azure.appconfiguration import AzureAppConfigurationClient

client = AzureAppConfigurationClient.from_connection_string(
    os.environ["AZURE_APPCONFIGURATION_CONNECTION_STRING"]
)
```

### Entra ID

```python
from azure.appconfiguration import AzureAppConfigurationClient
from azure.identity import DefaultAzureCredential

client = AzureAppConfigurationClient(
    base_url=os.environ["AZURE_APPCONFIGURATION_ENDPOINT"],
    credential=DefaultAzureCredential()
)
```

## Cài đặt Cấu hình

### Lấy Cài đặt

```python
setting = client.get_configuration_setting(key="app:settings:message")
print(f"{setting.key} = {setting.value}")
```

### Lấy với Nhãn (Label)

```python
# Nhãn cho phép các giá trị cụ thể cho môi trường
setting = client.get_configuration_setting(
    key="app:settings:message",
    label="production"
)
```

### Đặt Cài đặt

```python
from azure.appconfiguration import ConfigurationSetting

setting = ConfigurationSetting(
    key="app:settings:message",
    value="Hello, World!",
    label="development",
    content_type="text/plain",
    tags={"environment": "dev"}
)

client.set_configuration_setting(setting)
```

### Xóa Cài đặt

```python
client.delete_configuration_setting(
    key="app:settings:message",
    label="development"
)
```

## Liệt kê Cài đặt

### Tất cả Cài đặt

```python
settings = client.list_configuration_settings()
for setting in settings:
    print(f"{setting.key} [{setting.label}] = {setting.value}")
```

### Lọc theo Tiền tố Key

```python
settings = client.list_configuration_settings(
    key_filter="app:settings:*"
)
```

### Lọc theo Nhãn

```python
settings = client.list_configuration_settings(
    label_filter="production"
)
```

## Feature Flags

### Đặt Feature Flag

```python
from azure.appconfiguration import ConfigurationSetting
import json

feature_flag = ConfigurationSetting(
    key=".appconfig.featureflag/beta-feature",
    value=json.dumps({
        "id": "beta-feature",
        "enabled": True,
        "conditions": {
            "client_filters": []
        }
    }),
    content_type="application/vnd.microsoft.appconfig.ff+json;charset=utf-8"
)

client.set_configuration_setting(feature_flag)
```

### Lấy Feature Flag

```python
setting = client.get_configuration_setting(
    key=".appconfig.featureflag/beta-feature"
)
flag_data = json.loads(setting.value)
print(f"Tính năng được bật: {flag_data['enabled']}")
```

### Liệt kê Feature Flags

```python
flags = client.list_configuration_settings(
    key_filter=".appconfig.featureflag/*"
)
for flag in flags:
    data = json.loads(flag.value)
    print(f"{data['id']}: {'enabled' if data['enabled'] else 'disabled'}")
```

## Cài đặt Chỉ đọc (Read-Only)

```python
# Đặt cài đặt là chỉ đọc
client.set_read_only(
    configuration_setting=setting,
    read_only=True
)

# Xóa chỉ đọc
client.set_read_only(
    configuration_setting=setting,
    read_only=False
)
```

## Snapshots

### Tạo Snapshot

```python
from azure.appconfiguration import ConfigurationSnapshot, ConfigurationSettingFilter

snapshot = ConfigurationSnapshot(
    name="v1-snapshot",
    filters=[
        ConfigurationSettingFilter(key="app:*", label="production")
    ]
)

created = client.begin_create_snapshot(
    name="v1-snapshot",
    snapshot=snapshot
).result()
```

### Liệt kê Cài đặt Snapshot

```python
settings = client.list_configuration_settings(
    snapshot_name="v1-snapshot"
)
```

## Client Bất đồng bộ (Async Client)

```python
from azure.appconfiguration.aio import AzureAppConfigurationClient
from azure.identity.aio import DefaultAzureCredential

async def main():
    credential = DefaultAzureCredential()
    client = AzureAppConfigurationClient(
        base_url=endpoint,
        credential=credential
    )

    setting = await client.get_configuration_setting(key="app:message")
    print(setting.value)

    await client.close()
    await credential.close()
```

## Các Thao tác Client

| Thao tác                       | Mô tả                          |
| ------------------------------ | ------------------------------ |
| `get_configuration_setting`    | Lấy một cài đặt                |
| `set_configuration_setting`    | Tạo hoặc cập nhật cài đặt      |
| `delete_configuration_setting` | Xóa cài đặt                    |
| `list_configuration_settings`  | Liệt kê với bộ lọc             |
| `set_read_only`                | Khóa/Mở khóa cài đặt           |
| `begin_create_snapshot`        | Tạo snapshot tại một thời điểm |
| `list_snapshots`               | Liệt kê tất cả snapshots       |

## Thực hành Tốt nhất

1. **Sử dụng nhãn** để phân tách môi trường (dev, staging, prod)
2. **Sử dụng tiền tố key** để nhóm logic (app:database:_, app:cache:_)
3. **Đặt cài đặt sản xuất là chỉ đọc** để ngăn chặn thay đổi vô tình
4. **Tạo snapshots** trước khi triển khai để có khả năng rollback
5. **Sử dụng Entra ID** thay vì connection strings trong môi trường sản xuất
6. **Làm mới cài đặt định kỳ** trong các ứng dụng chạy lâu dài
7. **Sử dụng feature flags** cho triển khai dần dần và A/B testing
