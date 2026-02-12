---
name: azure-appconfiguration-ts
description: Xây dựng các ứng dụng sử dụng Azure App Configuration SDK cho JavaScript (@azure/app-configuration). Sử dụng khi làm việc với các cài đặt cấu hình, feature flags, tham chiếu Key Vault, làm mới động (dynamic refresh), hoặc quản lý cấu hình tập trung.
package: @azure/app-configuration
---

# Azure App Configuration SDK cho TypeScript

Quản lý cấu hình tập trung với feature flags và làm mới động.

## Cài đặt

```bash
# SDK CRUD cấp thấp
npm install @azure/app-configuration @azure/identity

# Provider cấp cao (khuyên dùng cho ứng dụng)
npm install @azure/app-configuration-provider @azure/identity

# Quản lý feature flag
npm install @microsoft/feature-management
```

## Biến Môi trường

```bash
AZURE_APPCONFIG_ENDPOINT=https://<your-resource>.azconfig.io
# HOẶC
AZURE_APPCONFIG_CONNECTION_STRING=Endpoint=https://...;Id=...;Secret=...
```

## Xác thực

```typescript
import { AppConfigurationClient } from "@azure/app-configuration";
import { DefaultAzureCredential } from "@azure/identity";

// DefaultAzureCredential (khuyên dùng)
const client = new AppConfigurationClient(
  process.env.AZURE_APPCONFIG_ENDPOINT!,
  new DefaultAzureCredential(),
);

// Connection string
const client2 = new AppConfigurationClient(
  process.env.AZURE_APPCONFIG_CONNECTION_STRING!,
);
```

## Thao tác CRUD

### Tạo/Cập nhật Cài đặt

```typescript
// Thêm mới (thất bại nếu đã tồn tại)
await client.addConfigurationSetting({
  key: "app:settings:message",
  value: "Hello World",
  label: "production",
  contentType: "text/plain",
  tags: { environment: "prod" },
});

// Set (tạo hoặc cập nhật)
await client.setConfigurationSetting({
  key: "app:settings:message",
  value: "Updated value",
  label: "production",
});

// Cập nhật với optimistic concurrency
const existing = await client.getConfigurationSetting({ key: "myKey" });
existing.value = "new value";
await client.setConfigurationSetting(existing, { onlyIfUnchanged: true });
```

### Đọc Cài đặt

```typescript
// Lấy một cài đặt đơn lẻ
const setting = await client.getConfigurationSetting({
  key: "app:settings:message",
  label: "production", // tùy chọn
});
console.log(setting.value);

// Liệt kê với bộ lọc
const settings = client.listConfigurationSettings({
  keyFilter: "app:*",
  labelFilter: "production",
});

for await (const setting of settings) {
  console.log(`${setting.key}: ${setting.value}`);
}
```

### Xóa Cài đặt

```typescript
await client.deleteConfigurationSetting({
  key: "app:settings:message",
  label: "production",
});
```

### Khóa/Mở khóa (Chỉ đọc)

```typescript
// Khóa
await client.setReadOnly({ key: "myKey", label: "prod" }, true);

// Mở khóa
await client.setReadOnly({ key: "myKey", label: "prod" }, false);
```

## App Configuration Provider

### Tải Cấu hình

```typescript
import { load } from "@azure/app-configuration-provider";
import { DefaultAzureCredential } from "@azure/identity";

const appConfig = await load(
  process.env.AZURE_APPCONFIG_ENDPOINT!,
  new DefaultAzureCredential(),
  {
    selectors: [{ keyFilter: "app:*", labelFilter: "production" }],
    trimKeyPrefixes: ["app:"],
  },
);

// Truy cập kiểu Map
const value = appConfig.get("settings:message");

// Truy cập kiểu Object
const config = appConfig.constructConfigurationObject({ separator: ":" });
console.log(config.settings.message);
```

### Làm mới Động (Dynamic Refresh)

```typescript
const appConfig = await load(endpoint, credential, {
  selectors: [{ keyFilter: "app:*" }],
  refreshOptions: {
    enabled: true,
    refreshIntervalInMs: 30_000, // 30 giây
  },
});

// Kích hoạt làm mới (không chặn - non-blocking)
appConfig.refresh();

// Lắng nghe sự kiện làm mới
const disposer = appConfig.onRefresh(() => {
  console.log("Configuration refreshed!");
});

// Mẫu Express middleware
app.use((req, res, next) => {
  appConfig.refresh();
  next();
});
```

### Tham chiếu Key Vault

```typescript
const appConfig = await load(endpoint, credential, {
  selectors: [{ keyFilter: "app:*" }],
  keyVaultOptions: {
    credential: new DefaultAzureCredential(),
    secretRefreshIntervalInMs: 7200_000, // 2 giờ
  },
});

// Bí mật được tự động phân giải
const dbPassword = appConfig.get("database:password");
```

## Feature Flags

### Tạo Feature Flag (Cấp thấp)

```typescript
import {
  featureFlagPrefix,
  featureFlagContentType,
  FeatureFlagValue,
  ConfigurationSetting,
} from "@azure/app-configuration";

const flag: ConfigurationSetting<FeatureFlagValue> = {
  key: `${featureFlagPrefix}Beta`,
  contentType: featureFlagContentType,
  value: {
    id: "Beta",
    enabled: true,
    description: "Beta feature",
    conditions: {
      clientFilters: [
        {
          name: "Microsoft.Targeting",
          parameters: {
            Audience: {
              Users: ["user@example.com"],
              Groups: [{ Name: "beta-testers", RolloutPercentage: 50 }],
              DefaultRolloutPercentage: 0,
            },
          },
        },
      ],
    },
  },
};

await client.addConfigurationSetting(flag);
```

### Tải và Đánh giá Feature Flags

```typescript
import { load } from "@azure/app-configuration-provider";
import {
  ConfigurationMapFeatureFlagProvider,
  FeatureManager,
} from "@microsoft/feature-management";

const appConfig = await load(endpoint, credential, {
  featureFlagOptions: {
    enabled: true,
    selectors: [{ keyFilter: "*" }],
    refresh: {
      enabled: true,
      refreshIntervalInMs: 30_000,
    },
  },
});

const featureProvider = new ConfigurationMapFeatureFlagProvider(appConfig);
const featureManager = new FeatureManager(featureProvider);

// Kiểm tra đơn giản
const isEnabled = await featureManager.isEnabled("Beta");

// Với ngữ cảnh targeting
const isEnabledForUser = await featureManager.isEnabled("Beta", {
  userId: "user@example.com",
  groups: ["beta-testers"],
});
```

## Snapshots

```typescript
// Tạo snapshot
const snapshot = await client.beginCreateSnapshotAndWait({
  name: "release-v1.0",
  retentionPeriod: 2592000, // 30 ngày
  filters: [{ keyFilter: "app:*", labelFilter: "production" }],
});

// Lấy snapshot
const snap = await client.getSnapshot("release-v1.0");

// Liệt kê cài đặt trong snapshot
const settings = client.listConfigurationSettingsForSnapshot("release-v1.0");
for await (const setting of settings) {
  console.log(`${setting.key}: ${setting.value}`);
}

// Lưu trữ/Khôi phục
await client.archiveSnapshot("release-v1.0");
await client.recoverSnapshot("release-v1.0");

// Tải từ snapshot (provider)
const config = await load(endpoint, credential, {
  selectors: [{ snapshotName: "release-v1.0" }],
});
```

## Nhãn (Labels)

```typescript
// Tạo cài đặt với nhãn
await client.setConfigurationSetting({
  key: "database:host",
  value: "dev-db.example.com",
  label: "development",
});

await client.setConfigurationSetting({
  key: "database:host",
  value: "prod-db.example.com",
  label: "production",
});

// Lọc theo nhãn
const prodSettings = client.listConfigurationSettings({
  keyFilter: "*",
  labelFilter: "production",
});

// Không có nhãn (nhãn null)
const noLabelSettings = client.listConfigurationSettings({
  labelFilter: "\0",
});

// Liệt kê các nhãn có sẵn
for await (const label of client.listLabels()) {
  console.log(label.name);
}
```

## Các Loại Chính (Key Types)

```typescript
import {
  AppConfigurationClient,
  ConfigurationSetting,
  FeatureFlagValue,
  SecretReferenceValue,
  featureFlagPrefix,
  featureFlagContentType,
  secretReferenceContentType,
  ListConfigurationSettingsOptions,
} from "@azure/app-configuration";

import { load } from "@azure/app-configuration-provider";

import {
  FeatureManager,
  ConfigurationMapFeatureFlagProvider,
} from "@microsoft/feature-management";
```

## Thực hành Tốt nhất

1. **Sử dụng provider cho ứng dụng** - `@azure/app-configuration-provider` cho cấu hình runtime
2. **Sử dụng cấp thấp để quản lý** - `@azure/app-configuration` cho các thao tác CRUD
3. **Bật làm mới** - Để cập nhật cấu hình động
4. **Sử dụng nhãn** - Phân tách cấu hình theo môi trường
5. **Sử dụng snapshots** - Cho các cấu hình phát hành bất biến
6. **Mẫu Sentinel** - Sử dụng một khóa sentinel để kích hoạt làm mới toàn bộ
7. **Vai trò RBAC** - `App Configuration Data Reader` cho truy cập chỉ đọc
