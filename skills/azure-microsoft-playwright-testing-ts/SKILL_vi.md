---
name: azure-microsoft-playwright-testing-ts
description: Chạy các bài kiểm tra Playwright ở quy mô lớn sử dụng Azure Playwright Workspaces (trước đây là Microsoft Playwright Testing). Sử dụng khi mở rộng quy mô kiểm tra trình duyệt trên các trình duyệt được lưu trữ trên đám mây, tích hợp với quy trình CI/CD, hoặc xuất bản kết quả kiểm tra lên cổng thông tin Azure.
package: "@azure/playwright"
---

# Azure Playwright Workspaces SDK cho TypeScript

Chạy các bài kiểm tra Playwright ở quy mô lớn với các trình duyệt được lưu trữ trên đám mây và báo cáo tích hợp trên cổng thông tin Azure.

> **Thông báo Di chuyển:** `@azure/microsoft-playwright-testing` sẽ ngừng hoạt động vào **ngày 8 tháng 3 năm 2026**. Hãy sử dụng `@azure/playwright` thay thế. Xem [hướng dẫn di chuyển](https://aka.ms/mpt/migration-guidance).

## Cài đặt

```bash
# Khuyến nghị: Tự động tạo cấu hình
npm init @azure/playwright@latest

# Cài đặt thủ công
npm install @azure/playwright --save-dev
npm install @playwright/test@^1.47 --save-dev
npm install @azure/identity --save-dev
```

**Yêu cầu:**

- Playwright phiên bản 1.47+ (sử dụng cơ bản)
- Playwright phiên bản 1.57+ (các tính năng Azure reporter)

## Biến Môi trường

```bash
PLAYWRIGHT_SERVICE_URL=wss://eastus.api.playwright.microsoft.com/playwrightworkspaces/{workspace-id}/browsers
```

## Xác thực

### Microsoft Entra ID (Khuyến nghị)

```bash
# Đăng nhập bằng Azure CLI
az login
```

```typescript
// playwright.service.config.ts
import { defineConfig } from "@playwright/test";
import { createAzurePlaywrightConfig, ServiceOS } from "@azure/playwright";
import { DefaultAzureCredential } from "@azure/identity";
import config from "./playwright.config";

export default defineConfig(
  config,
  createAzurePlaywrightConfig(config, {
    os: ServiceOS.LINUX,
    credential: new DefaultAzureCredential(),
  }),
);
```

### Custom Credential

```typescript
import { ManagedIdentityCredential } from "@azure/identity";
import { createAzurePlaywrightConfig } from "@azure/playwright";

export default defineConfig(
  config,
  createAzurePlaywrightConfig(config, {
    credential: new ManagedIdentityCredential(),
  }),
);
```

## Quy trình Làm việc Cốt lõi

### Cấu hình Dịch vụ

```typescript
// playwright.service.config.ts
import { defineConfig } from "@playwright/test";
import { createAzurePlaywrightConfig, ServiceOS } from "@azure/playwright";
import { DefaultAzureCredential } from "@azure/identity";
import config from "./playwright.config";

export default defineConfig(
  config,
  createAzurePlaywrightConfig(config, {
    os: ServiceOS.LINUX,
    connectTimeout: 30000,
    exposeNetwork: "<loopback>",
    credential: new DefaultAzureCredential(),
  }),
);
```

### Chạy Kiểm tra

```bash
npx playwright test --config=playwright.service.config.ts --workers=20
```

### Với Azure Reporter

```typescript
import { defineConfig } from "@playwright/test";
import { createAzurePlaywrightConfig, ServiceOS } from "@azure/playwright";
import { DefaultAzureCredential } from "@azure/identity";
import config from "./playwright.config";

export default defineConfig(
  config,
  createAzurePlaywrightConfig(config, {
    os: ServiceOS.LINUX,
    credential: new DefaultAzureCredential(),
  }),
  {
    reporter: [["html", { open: "never" }], ["@azure/playwright/reporter"]],
  },
);
```

### Kết nối Trình duyệt Thủ công

```typescript
import playwright, { test, expect, BrowserType } from "@playwright/test";
import { getConnectOptions } from "@azure/playwright";

test("manual connection", async ({ browserName }) => {
  const { wsEndpoint, options } = await getConnectOptions();
  const browser = await (playwright[browserName] as BrowserType).connect(
    wsEndpoint,
    options,
  );
  const context = await browser.newContext();
  const page = await context.newPage();

  await page.goto("https://example.com");
  await expect(page).toHaveTitle(/Example/);

  await browser.close();
});
```

## Tùy chọn Cấu hình

```typescript
type PlaywrightServiceAdditionalOptions = {
  serviceAuthType?: "ENTRA_ID" | "ACCESS_TOKEN"; // Mặc định: ENTRA_ID
  os?: "linux" | "windows"; // Mặc định: linux
  runName?: string; // Tên chạy tùy chỉnh cho portal
  connectTimeout?: number; // Mặc định: 30000ms
  exposeNetwork?: string; // Mặc định: <loopback>
  credential?: TokenCredential; // BẮT BUỘC cho Entra ID
};
```

### ServiceOS Enum

```typescript
import { ServiceOS } from "@azure/playwright";

// Các giá trị khả dụng
ServiceOS.LINUX; // "linux" - mặc định
ServiceOS.WINDOWS; // "windows"
```

### ServiceAuth Enum

```typescript
import { ServiceAuth } from "@azure/playwright";

// Các giá trị khả dụng
ServiceAuth.ENTRA_ID; // Khuyến nghị - sử dụng credential
ServiceAuth.ACCESS_TOKEN; // Sử dụng biến môi trường PLAYWRIGHT_SERVICE_ACCESS_TOKEN
```

## Tích hợp CI/CD

### GitHub Actions

```yaml
name: playwright-ts
on: [push, pull_request]

permissions:
  id-token: write
  contents: read

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Azure Login
        uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - run: npm ci

      - name: Run Tests
        env:
          PLAYWRIGHT_SERVICE_URL: ${{ secrets.PLAYWRIGHT_SERVICE_URL }}
        run: npx playwright test -c playwright.service.config.ts --workers=20
```

### Azure Pipelines

```yaml
- task: AzureCLI@2
  displayName: Run Playwright Tests
  env:
    PLAYWRIGHT_SERVICE_URL: $(PLAYWRIGHT_SERVICE_URL)
  inputs:
    azureSubscription: My_Service_Connection
    scriptType: pscore
    inlineScript: |
      npx playwright test -c playwright.service.config.ts --workers=20
    addSpnToEnvironment: true
```

## Các Loại Chính

```typescript
import {
  createAzurePlaywrightConfig,
  getConnectOptions,
  ServiceOS,
  ServiceAuth,
  ServiceEnvironmentVariable,
} from "@azure/playwright";

import type {
  OsType,
  AuthenticationType,
  BrowserConnectOptions,
  PlaywrightServiceAdditionalOptions,
} from "@azure/playwright";
```

## Di chuyển từ Gói Cũ

| Cũ (`@azure/microsoft-playwright-testing`)     | Mới (`@azure/playwright`)       |
| ---------------------------------------------- | ------------------------------- |
| `getServiceConfig()`                           | `createAzurePlaywrightConfig()` |
| `timeout` option                               | `connectTimeout` option         |
| `runId` option                                 | `runName` option                |
| `useCloudHostedBrowsers` option                | Removed (luôn được bật)         |
| `@azure/microsoft-playwright-testing/reporter` | `@azure/playwright/reporter`    |
| Implicit credential                            | Tham số `credential` tường minh |

### Trước (Cũ)

```typescript
import {
  getServiceConfig,
  ServiceOS,
} from "@azure/microsoft-playwright-testing";

export default defineConfig(
  config,
  getServiceConfig(config, {
    os: ServiceOS.LINUX,
    timeout: 30000,
    useCloudHostedBrowsers: true,
  }),
  {
    reporter: [["@azure/microsoft-playwright-testing/reporter"]],
  },
);
```

### Sau (Mới)

```typescript
import { createAzurePlaywrightConfig, ServiceOS } from "@azure/playwright";
import { DefaultAzureCredential } from "@azure/identity";

export default defineConfig(
  config,
  createAzurePlaywrightConfig(config, {
    os: ServiceOS.LINUX,
    connectTimeout: 30000,
    credential: new DefaultAzureCredential(),
  }),
  {
    reporter: [["html", { open: "never" }], ["@azure/playwright/reporter"]],
  },
);
```

## Thực hành Tốt nhất

1. **Sử dụng Entra ID auth** — Bảo mật hơn so với access tokens
2. **Cung cấp credential tường minh** — Luôn truyền `credential: new DefaultAzureCredential()`
3. **Bật artifacts** — Thiết lập `trace: "on-first-retry"`, `video: "retain-on-failure"` trong config
4. **Mở rộng workers** — Sử dụng `--workers=20` hoặc cao hơn để thực thi song song
5. **Lựa chọn vùng** — Chọn vùng gần nhất với các mục tiêu kiểm tra của bạn
6. **HTML reporter trước** — Khi sử dụng Azure reporter, liệt kê HTML reporter trước Azure reporter
