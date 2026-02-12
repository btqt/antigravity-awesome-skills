---
name: azure-appconfiguration-java
description: |
  Azure App Configuration SDK cho Java. Quản lý cấu hình ứng dụng tập trung với các cài đặt key-value, feature flags, và snapshots.
  Kích hoạt: "ConfigurationClient java", "app configuration java", "feature flag java", "configuration setting java", "azure config java".
package: com.azure:azure-data-appconfiguration
---

# Azure App Configuration SDK cho Java

Thư viện Client cho Azure App Configuration, một dịch vụ được quản lý để tập trung hóa các cấu hình ứng dụng.

## Cài đặt

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-data-appconfiguration</artifactId>
    <version>1.8.0</version>
</dependency>
```

Hoặc sử dụng Azure SDK BOM:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.azure</groupId>
            <artifactId>azure-sdk-bom</artifactId>
            <version>{bom_version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-data-appconfiguration</artifactId>
    </dependency>
</dependencies>
```

## Điều kiện Tiên quyết

- Azure App Configuration store
- Connection string hoặc thông tin xác thực Entra ID

## Biến Môi trường

```bash
AZURE_APPCONFIG_CONNECTION_STRING=Endpoint=https://<store>.azconfig.io;Id=<id>;Secret=<secret>
AZURE_APPCONFIG_ENDPOINT=https://<store>.azconfig.io
```

## Tạo Client

### Với Connection String

```java
import com.azure.data.appconfiguration.ConfigurationClient;
import com.azure.data.appconfiguration.ConfigurationClientBuilder;

ConfigurationClient configClient = new ConfigurationClientBuilder()
    .connectionString(System.getenv("AZURE_APPCONFIG_CONNECTION_STRING"))
    .buildClient();
```

### Client Bất đồng bộ (Async Client)

```java
import com.azure.data.appconfiguration.ConfigurationAsyncClient;

ConfigurationAsyncClient asyncClient = new ConfigurationClientBuilder()
    .connectionString(connectionString)
    .buildAsyncClient();
```

### Với Entra ID (Khuyên dùng)

```java
import com.azure.identity.DefaultAzureCredentialBuilder;

ConfigurationClient configClient = new ConfigurationClientBuilder()
    .credential(new DefaultAzureCredentialBuilder().build())
    .endpoint(System.getenv("AZURE_APPCONFIG_ENDPOINT"))
    .buildClient();
```

## Các Khái niệm Chính

| Khái niệm             | Mô tả                                                        |
| --------------------- | ------------------------------------------------------------ |
| Configuration Setting | Cặp key-value với nhãn (label) tùy chọn                      |
| Label                 | Chiều để phân tách các cài đặt (ví dụ: môi trường, versions) |
| Feature Flag          | Cài đặt đặc biệt cho quản lý tính năng                       |
| Secret Reference      | Cài đặt trỏ đến bí mật trong Key Vault                       |
| Snapshot              | Chế độ xem bất biến tại một thời điểm của các cài đặt        |

## Các Thao tác Cài đặt Cấu hình

### Tạo Cài đặt (Add)

Chỉ tạo nếu cài đặt chưa tồn tại:

```java
import com.azure.data.appconfiguration.models.ConfigurationSetting;

ConfigurationSetting setting = configClient.addConfigurationSetting(
    "app/database/connection",
    "Production",
    "Server=prod.db.com;Database=myapp"
);
```

### Tạo hoặc Cập nhật Cài đặt (Set)

Tạo hoặc ghi đè:

```java
ConfigurationSetting setting = configClient.setConfigurationSetting(
    "app/cache/enabled",
    "Production",
    "true"
);
```

### Lấy Cài đặt (Get)

```java
ConfigurationSetting setting = configClient.getConfigurationSetting(
    "app/database/connection",
    "Production"
);
System.out.println("Giá trị: " + setting.getValue());
System.out.println("Content-Type: " + setting.getContentType());
System.out.println("Sửa đổi lần cuối: " + setting.getLastModified());
```

### Lấy có Điều kiện (Nếu Thay đổi)

```java
import com.azure.core.http.rest.Response;
import com.azure.core.util.Context;

Response<ConfigurationSetting> response = configClient.getConfigurationSettingWithResponse(
    setting,      // Setting với ETag
    null,         // Accept datetime
    true,         // ifChanged - chỉ lấy nếu đã sửa đổi
    Context.NONE
);

if (response.getStatusCode() == 304) {
    System.out.println("Cài đặt không thay đổi");
} else {
    ConfigurationSetting updated = response.getValue();
}
```

### Cập nhật Cài đặt

```java
ConfigurationSetting updated = configClient.setConfigurationSetting(
    "app/cache/enabled",
    "Production",
    "false"
);
```

### Cập nhật có Điều kiện (Nếu Không thay đổi)

```java
// Chỉ cập nhật nếu ETag khớp (không có sửa đổi đồng thời)
Response<ConfigurationSetting> response = configClient.setConfigurationSettingWithResponse(
    setting,     // Setting với ETag hiện tại
    true,        // ifUnchanged
    Context.NONE
);
```

### Xóa Cài đặt

```java
ConfigurationSetting deleted = configClient.deleteConfigurationSetting(
    "app/cache/enabled",
    "Production"
);
```

### Xóa có Điều kiện

```java
Response<ConfigurationSetting> response = configClient.deleteConfigurationSettingWithResponse(
    setting,     // Setting với ETag
    true,        // ifUnchanged
    Context.NONE
);
```

## Liệt kê và Lọc Cài đặt

### Liệt kê theo Mẫu Key

```java
import com.azure.data.appconfiguration.models.SettingSelector;
import com.azure.core.http.rest.PagedIterable;

SettingSelector selector = new SettingSelector()
    .setKeyFilter("app/*");

PagedIterable<ConfigurationSetting> settings = configClient.listConfigurationSettings(selector);
for (ConfigurationSetting s : settings) {
    System.out.println(s.getKey() + " = " + s.getValue());
}
```

### Liệt kê theo Label

```java
SettingSelector selector = new SettingSelector()
    .setKeyFilter("*")
    .setLabelFilter("Production");

PagedIterable<ConfigurationSetting> settings = configClient.listConfigurationSettings(selector);
```

### Liệt kê theo Nhiều Key

```java
SettingSelector selector = new SettingSelector()
    .setKeyFilter("app/database/*,app/cache/*");

PagedIterable<ConfigurationSetting> settings = configClient.listConfigurationSettings(selector);
```

### Liệt kê các Phiên bản (Revisions)

```java
SettingSelector selector = new SettingSelector()
    .setKeyFilter("app/database/connection");

PagedIterable<ConfigurationSetting> revisions = configClient.listRevisions(selector);
for (ConfigurationSetting revision : revisions) {
    System.out.println("Giá trị: " + revision.getValue() + ", Modified: " + revision.getLastModified());
}
```

## Feature Flags

### Tạo Feature Flag

```java
import com.azure.data.appconfiguration.models.FeatureFlagConfigurationSetting;
import com.azure.data.appconfiguration.models.FeatureFlagFilter;
import java.util.Arrays;

FeatureFlagFilter percentageFilter = new FeatureFlagFilter("Microsoft.Percentage")
    .addParameter("Value", 50);

FeatureFlagConfigurationSetting featureFlag = new FeatureFlagConfigurationSetting("beta-feature", true)
    .setDescription("Triển khai tính năng beta")
    .setClientFilters(Arrays.asList(percentageFilter));

FeatureFlagConfigurationSetting created = (FeatureFlagConfigurationSetting)
    configClient.addConfigurationSetting(featureFlag);
```

### Lấy Feature Flag

```java
FeatureFlagConfigurationSetting flag = (FeatureFlagConfigurationSetting)
    configClient.getConfigurationSetting(featureFlag);

System.out.println("Feature: " + flag.getFeatureId());
System.out.println("Enabled: " + flag.isEnabled());
System.out.println("Filters: " + flag.getClientFilters());
```

### Cập nhật Feature Flag

```java
featureFlag.setEnabled(false);
FeatureFlagConfigurationSetting updated = (FeatureFlagConfigurationSetting)
    configClient.setConfigurationSetting(featureFlag);
```

## Tham chiếu Bí mật (Secret References)

### Tạo Tham chiếu Bí mật

```java
import com.azure.data.appconfiguration.models.SecretReferenceConfigurationSetting;

SecretReferenceConfigurationSetting secretRef = new SecretReferenceConfigurationSetting(
    "app/secrets/api-key",
    "https://myvault.vault.azure.net/secrets/api-key"
);

SecretReferenceConfigurationSetting created = (SecretReferenceConfigurationSetting)
    configClient.addConfigurationSetting(secretRef);
```

### Lấy Tham chiếu Bí mật

```java
SecretReferenceConfigurationSetting ref = (SecretReferenceConfigurationSetting)
    configClient.getConfigurationSetting(secretRef);

System.out.println("URI Bí mật: " + ref.getSecretId());
```

## Cài đặt Chỉ đọc (Read-Only)

### Đặt Chỉ đọc

```java
ConfigurationSetting readOnly = configClient.setReadOnly(
    "app/critical/setting",
    "Production",
    true
);
```

### Xóa Chỉ đọc

```java
ConfigurationSetting writable = configClient.setReadOnly(
    "app/critical/setting",
    "Production",
    false
);
```

## Snapshots

### Tạo Snapshot

```java
import com.azure.data.appconfiguration.models.ConfigurationSnapshot;
import com.azure.data.appconfiguration.models.ConfigurationSettingsFilter;
import com.azure.core.util.polling.SyncPoller;
import com.azure.core.util.polling.PollOperationDetails;

List<ConfigurationSettingsFilter> filters = new ArrayList<>();
filters.add(new ConfigurationSettingsFilter("app/*"));

SyncPoller<PollOperationDetails, ConfigurationSnapshot> poller = configClient.beginCreateSnapshot(
    "release-v1.0",
    new ConfigurationSnapshot(filters),
    Context.NONE
);
poller.setPollInterval(Duration.ofSeconds(10));
poller.waitForCompletion();

ConfigurationSnapshot snapshot = poller.getFinalResult();
System.out.println("Snapshot: " + snapshot.getName() + ", Trạng thái: " + snapshot.getStatus());
```

### Lấy Snapshot

```java
ConfigurationSnapshot snapshot = configClient.getSnapshot("release-v1.0");
System.out.println("Đã tạo: " + snapshot.getCreatedAt());
System.out.println("Số mục: " + snapshot.getItemCount());
```

### Liệt kê Cài đặt trong Snapshot

```java
PagedIterable<ConfigurationSetting> settings =
    configClient.listConfigurationSettingsForSnapshot("release-v1.0");

for (ConfigurationSetting setting : settings) {
    System.out.println(setting.getKey() + " = " + setting.getValue());
}
```

### Lưu trữ (Archive) Snapshot

```java
ConfigurationSnapshot archived = configClient.archiveSnapshot("release-v1.0");
System.out.println("Trạng thái: " + archived.getStatus()); // archived
```

### Khôi phục (Recover) Snapshot

```java
ConfigurationSnapshot recovered = configClient.recoverSnapshot("release-v1.0");
System.out.println("Trạng thái: " + recovered.getStatus()); // ready
```

### Liệt kê Tất cả Snapshots

```java
import com.azure.data.appconfiguration.models.SnapshotSelector;

SnapshotSelector selector = new SnapshotSelector().setNameFilter("release-*");
PagedIterable<ConfigurationSnapshot> snapshots = configClient.listSnapshots(selector);

for (ConfigurationSnapshot snap : snapshots) {
    System.out.println(snap.getName() + " - " + snap.getStatus());
}
```

## Nhãn (Labels)

### Liệt kê Nhãn

```java
import com.azure.data.appconfiguration.models.SettingLabelSelector;

configClient.listLabels(new SettingLabelSelector().setNameFilter("*"))
    .forEach(label -> System.out.println("Label: " + label.getName()));
```

## Thao tác Bất đồng bộ (Async Operations)

```java
ConfigurationAsyncClient asyncClient = new ConfigurationClientBuilder()
    .connectionString(connectionString)
    .buildAsyncClient();

// List async với reactive streams
asyncClient.listConfigurationSettings(new SettingSelector().setLabelFilter("Production"))
    .subscribe(
        setting -> System.out.println(setting.getKey() + " = " + setting.getValue()),
        error -> System.err.println("Lỗi: " + error.getMessage()),
        () -> System.out.println("Hoàn thành")
    );
```

## Xử lý Lỗi

```java
import com.azure.core.exception.HttpResponseException;

try {
    configClient.getConfigurationSetting("nonexistent", null);
} catch (HttpResponseException e) {
    if (e.getResponse().getStatusCode() == 404) {
        System.err.println("Không tìm thấy cài đặt");
    } else {
        System.err.println("Lỗi: " + e.getMessage());
    }
}
```

## Thực hành Tốt nhất

1. **Sử dụng labels** — Phân tách cấu hình theo môi trường (Dev, Staging, Production)
2. **Sử dụng snapshots** — Tạo snapshots bất biến cho các bản phát hành
3. **Feature flags** — Sử dụng cho triển khai dần dần và A/B testing
4. **Tham chiếu bí mật** — Lưu trữ các giá trị nhạy cảm trong Key Vault
5. **Yêu cầu có điều kiện** — Sử dụng ETags cho optimistic concurrency
6. **Bảo vệ chỉ đọc** — Khóa các cài đặt sản xuất quan trọng
7. **Sử dụng Entra ID** — Ưu tiên hơn connection strings
8. **Client async** — Sử dụng cho các kịch bản thông lượng cao
