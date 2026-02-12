---
name: azure-identity-ts
description: Xác thực với các dịch vụ Azure sử dụng Azure Identity SDK cho JavaScript (@azure/identity). Sử dụng khi cấu hình xác thực với DefaultAzureCredential, managed identity, service principals, hoặc đăng nhập trình duyệt tương tác.
package: @azure/identity
---

# Azure Identity SDK cho TypeScript

Xác thực với các dịch vụ Azure bằng nhiều loại thông tin xác thực (credential) khác nhau.

## Cài đặt

```bash
npm install @azure/identity
```

## Biến Môi trường

### Service Principal (Secret)

```bash
AZURE_TENANT_ID=<tenant-id>
AZURE_CLIENT_ID=<client-id>
AZURE_CLIENT_SECRET=<client-secret>
```

### Service Principal (Certificate)

```bash
AZURE_TENANT_ID=<tenant-id>
AZURE_CLIENT_ID=<client-id>
AZURE_CLIENT_CERTIFICATE_PATH=/path/to/cert.pem
AZURE_CLIENT_CERTIFICATE_PASSWORD=<optional-password>
```

### Workload Identity (Kubernetes)

```bash
AZURE_TENANT_ID=<tenant-id>
AZURE_CLIENT_ID=<client-id>
AZURE_FEDERATED_TOKEN_FILE=/var/run/secrets/tokens/azure-identity
```

## DefaultAzureCredential (Khuyên dùng)

```typescript
import { DefaultAzureCredential } from "@azure/identity";

const credential = new DefaultAzureCredential();

// Sử dụng với bất kỳ Azure SDK client nào
import { BlobServiceClient } from "@azure/storage-blob";
const blobClient = new BlobServiceClient(
  "https://<account>.blob.core.windows.net",
  credential,
);
```

**Thứ tự Chuỗi Credential:**

1. EnvironmentCredential
2. WorkloadIdentityCredential
3. ManagedIdentityCredential
4. VisualStudioCodeCredential
5. AzureCliCredential
6. AzurePowerShellCredential
7. AzureDeveloperCliCredential

## Managed Identity

### System-Assigned

```typescript
import { ManagedIdentityCredential } from "@azure/identity";

const credential = new ManagedIdentityCredential();
```

### User-Assigned (theo Client ID)

```typescript
const credential = new ManagedIdentityCredential({
  clientId: "<user-assigned-client-id>",
});
```

### User-Assigned (theo Resource ID)

```typescript
const credential = new ManagedIdentityCredential({
  resourceId:
    "/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<name>",
});
```

## Service Principal

### Client Secret

```typescript
import { ClientSecretCredential } from "@azure/identity";

const credential = new ClientSecretCredential(
  "<tenant-id>",
  "<client-id>",
  "<client-secret>",
);
```

### Client Certificate

```typescript
import { ClientCertificateCredential } from "@azure/identity";

const credential = new ClientCertificateCredential(
  "<tenant-id>",
  "<client-id>",
  { certificatePath: "/path/to/cert.pem" },
);

// Với mật khẩu
const credentialWithPwd = new ClientCertificateCredential(
  "<tenant-id>",
  "<client-id>",
  {
    certificatePath: "/path/to/cert.pem",
    certificatePassword: "<password>",
  },
);
```

## Xác thực Tương tác

### Đăng nhập Dựa trên Trình duyệt

```typescript
import { InteractiveBrowserCredential } from "@azure/identity";

const credential = new InteractiveBrowserCredential({
  clientId: "<client-id>",
  tenantId: "<tenant-id>",
  loginHint: "user@example.com",
});
```

### Luồng Mã Thiết bị (Device Code Flow)

```typescript
import { DeviceCodeCredential } from "@azure/identity";

const credential = new DeviceCodeCredential({
  clientId: "<client-id>",
  tenantId: "<tenant-id>",
  userPromptCallback: (info) => {
    console.log(info.message);
    // "To sign in, use a web browser to open..."
  },
});
```

## Chuỗi Credential Tùy chỉnh

```typescript
import {
  ChainedTokenCredential,
  ManagedIdentityCredential,
  AzureCliCredential,
} from "@azure/identity";

// Thử managed identity trước, nếu không được thì dùng CLI
const credential = new ChainedTokenCredential(
  new ManagedIdentityCredential(),
  new AzureCliCredential(),
);
```

## Thông tin Xác thực Nhà phát triển

### Azure CLI

```typescript
import { AzureCliCredential } from "@azure/identity";

const credential = new AzureCliCredential();
// Sử dụng: az login
```

### Azure Developer CLI

```typescript
import { AzureDeveloperCliCredential } from "@azure/identity";

const credential = new AzureDeveloperCliCredential();
// Sử dụng: azd auth login
```

### Azure PowerShell

```typescript
import { AzurePowerShellCredential } from "@azure/identity";

const credential = new AzurePowerShellCredential();
// Sử dụng: Connect-AzAccount
```

## Sovereign Clouds (Các Đám mây Chủ quyền)

```typescript
import { ClientSecretCredential, AzureAuthorityHosts } from "@azure/identity";

// Azure Government
const credential = new ClientSecretCredential(
  "<tenant>",
  "<client>",
  "<secret>",
  { authorityHost: AzureAuthorityHosts.AzureGovernment },
);

// Azure China
const credentialChina = new ClientSecretCredential(
  "<tenant>",
  "<client>",
  "<secret>",
  { authorityHost: AzureAuthorityHosts.AzureChina },
);
```

## Bearer Token Provider

```typescript
import {
  DefaultAzureCredential,
  getBearerTokenProvider,
} from "@azure/identity";

const credential = new DefaultAzureCredential();

// Tạo một function trả về token
const getAccessToken = getBearerTokenProvider(
  credential,
  "https://cognitiveservices.azure.com/.default",
);

// Sử dụng với các API cần bearer tokens
const token = await getAccessToken();
```

## Các Loại Chính

```typescript
import type {
  TokenCredential,
  AccessToken,
  GetTokenOptions,
} from "@azure/core-auth";

import {
  DefaultAzureCredential,
  DefaultAzureCredentialOptions,
  ManagedIdentityCredential,
  ClientSecretCredential,
  ClientCertificateCredential,
  InteractiveBrowserCredential,
  ChainedTokenCredential,
  AzureCliCredential,
  AzurePowerShellCredential,
  AzureDeveloperCliCredential,
  DeviceCodeCredential,
  AzureAuthorityHosts,
} from "@azure/identity";
```

## Triển khai Credential Tùy chỉnh

```typescript
import type {
  TokenCredential,
  AccessToken,
  GetTokenOptions,
} from "@azure/core-auth";

class CustomCredential implements TokenCredential {
  async getToken(
    scopes: string | string[],
    options?: GetTokenOptions,
  ): Promise<AccessToken | null> {
    // Logic lấy token tùy chỉnh
    return {
      token: "<access-token>",
      expiresOnTimestamp: Date.now() + 3600000,
    };
  }
}
```

## Gỡ lỗi

```typescript
import { setLogLevel, AzureLogger } from "@azure/logger";

setLogLevel("verbose");

// Custom log handler
AzureLogger.log = (...args) => {
  console.log("[Azure]", ...args);
};
```

## Thực hành Tốt nhất

1. **Sử dụng DefaultAzureCredential** - Hoạt động trong phát triển (CLI) và sản xuất (managed identity)
2. **Không bao giờ hardcode credentials** - Sử dụng biến môi trường hoặc managed identity
3. **Ưu tiên managed identity** - Không cần quản lý secret trong sản xuất
4. **Phạm vi (Scope) credentials phù hợp** - Sử dụng user-assigned identity cho kịch bản đa tenant
5. **Xử lý refresh token** - Azure SDK xử lý việc này tự động
6. **Sử dụng ChainedTokenCredential** - Cho các kịch bản dự phòng tùy chỉnh
