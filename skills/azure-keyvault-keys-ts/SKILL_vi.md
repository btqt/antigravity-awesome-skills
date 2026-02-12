---
name: azure-keyvault-keys-ts
description: Quản lý các khóa mật mã sử dụng Azure Key Vault Keys SDK cho JavaScript (@azure/keyvault-keys). Sử dụng khi tạo, mã hóa/giải mã, ký, hoặc xoay vòng khóa.
package: @azure/keyvault-keys
---

# Azure Key Vault Keys SDK cho TypeScript

Quản lý các khóa mật mã (cryptographic keys) với Azure Key Vault.

## Cài đặt

```bash
# Keys SDK
npm install @azure/keyvault-keys @azure/identity
```

## Biến Môi trường

```bash
KEY_VAULT_URL=https://<vault-name>.vault.azure.net
# Hoặc
AZURE_KEYVAULT_NAME=<vault-name>
```

## Xác thực

```typescript
import { DefaultAzureCredential } from "@azure/identity";
import { KeyClient, CryptographyClient } from "@azure/keyvault-keys";

const credential = new DefaultAzureCredential();
const vaultUrl = `https://${process.env.AZURE_KEYVAULT_NAME}.vault.azure.net`;

const keyClient = new KeyClient(vaultUrl, credential);
const secretClient = new SecretClient(vaultUrl, credential);
```

## Các Thao tác Secrets

### Tạo/Đặt Secret

```typescript
const secret = await secretClient.setSecret("MySecret", "secret-value");

// Với các thuộc tính
const secretWithAttrs = await secretClient.setSecret("MySecret", "value", {
  enabled: true,
  expiresOn: new Date("2025-12-31"),
  contentType: "application/json",
  tags: { environment: "production" },
});
```

### Lấy Secret

```typescript
// Lấy phiên bản mới nhất
const secret = await secretClient.getSecret("MySecret");
console.log(secret.value);

// Lấy phiên bản cụ thể
const specificSecret = await secretClient.getSecret("MySecret", {
  version: secret.properties.version,
});
```

### Liệt kê Secrets

```typescript
for await (const secretProperties of secretClient.listPropertiesOfSecrets()) {
  console.log(secretProperties.name);
}

// Liệt kê các phiên bản
for await (const version of secretClient.listPropertiesOfSecretVersions(
  "MySecret",
)) {
  console.log(version.version);
}
```

### Xóa Secret

```typescript
// Xóa mềm (Soft delete)
const deletePoller = await secretClient.beginDeleteSecret("MySecret");
await deletePoller.pollUntilDone();

// Xóa vĩnh viễn (Purge)
await secretClient.purgeDeletedSecret("MySecret");

// Khôi phục
const recoverPoller = await secretClient.beginRecoverDeletedSecret("MySecret");
await recoverPoller.pollUntilDone();
```

## Các Thao tác Khóa (Keys Operations)

### Tạo Khóa

```typescript
// Khóa chung
const key = await keyClient.createKey("MyKey", "RSA");

// Khóa RSA với kích thước
const rsaKey = await keyClient.createRsaKey("MyRsaKey", { keySize: 2048 });

// Khóa Elliptic Curve
const ecKey = await keyClient.createEcKey("MyEcKey", { curve: "P-256" });

// Với các thuộc tính
const keyWithAttrs = await keyClient.createKey("MyKey", "RSA", {
  enabled: true,
  expiresOn: new Date("2025-12-31"),
  tags: { purpose: "encryption" },
  keyOps: ["encrypt", "decrypt", "sign", "verify"],
});
```

### Lấy Khóa

```typescript
const key = await keyClient.getKey("MyKey");
console.log(key.name, key.keyType);
```

### Liệt kê Khóa

```typescript
for await (const keyProperties of keyClient.listPropertiesOfKeys()) {
  console.log(keyProperties.name);
}
```

### Xoay vòng Khóa (Rotate Key)

```typescript
// Xoay vòng thủ công
const rotatedKey = await keyClient.rotateKey("MyKey");

// Đặt chính sách xoay vòng
await keyClient.updateKeyRotationPolicy("MyKey", {
  lifetimeActions: [{ action: "Rotate", timeBeforeExpiry: "P30D" }],
  expiresIn: "P90D",
});
```

### Xóa Khóa

```typescript
const deletePoller = await keyClient.beginDeleteKey("MyKey");
await deletePoller.pollUntilDone();

// Xóa vĩnh viễn (Purge)
await keyClient.purgeDeletedKey("MyKey");
```

## Các Thao tác Mật mã (Cryptographic Operations)

### Tạo CryptographyClient

```typescript
import { CryptographyClient } from "@azure/keyvault-keys";

// Từ đối tượng key
const cryptoClient = new CryptographyClient(key, credential);

// Từ key ID
const cryptoClient = new CryptographyClient(key.id!, credential);
```

### Mã hóa/Giải mã (Encrypt/Decrypt)

```typescript
// Mã hóa
const encryptResult = await cryptoClient.encrypt({
  algorithm: "RSA-OAEP",
  plaintext: Buffer.from("My secret message"),
});

// Giải mã
const decryptResult = await cryptoClient.decrypt({
  algorithm: "RSA-OAEP",
  ciphertext: encryptResult.result,
});

console.log(decryptResult.result.toString());
```

### Ký/Xác minh (Sign/Verify)

```typescript
import { createHash } from "node:crypto";

// Tạo bản tóm tắt (digest)
const hash = createHash("sha256").update("My message").digest();

// Ký
const signResult = await cryptoClient.sign("RS256", hash);

// Xác minh
const verifyResult = await cryptoClient.verify(
  "RS256",
  hash,
  signResult.result,
);
console.log("Valid:", verifyResult.result);
```

### Wrap/Unwrap Keys

```typescript
// Wrap một khóa (mã hóa nó để lưu trữ)
const wrapResult = await cryptoClient.wrapKey(
  "RSA-OAEP",
  Buffer.from("key-material"),
);

// Unwrap
const unwrapResult = await cryptoClient.unwrapKey(
  "RSA-OAEP",
  wrapResult.result,
);
```

## Sao lưu và Khôi phục

```typescript
// Sao lưu
const keyBackup = await keyClient.backupKey("MyKey");
const secretBackup = await secretClient.backupSecret("MySecret");

// Khôi phục (có thể khôi phục sang vault khác)
const restoredKey = await keyClient.restoreKeyBackup(keyBackup!);
const restoredSecret = await secretClient.restoreSecretBackup(secretBackup!);
```

## Các Loại Key

```typescript
import {
  KeyClient,
  KeyVaultKey,
  KeyProperties,
  DeletedKey,
  CryptographyClient,
  KnownEncryptionAlgorithms,
  KnownSignatureAlgorithms,
} from "@azure/keyvault-keys";

import {
  SecretClient,
  KeyVaultSecret,
  SecretProperties,
  DeletedSecret,
} from "@azure/keyvault-secrets";
```

## Xử lý Lỗi

```typescript
try {
  const secret = await secretClient.getSecret("NonExistent");
} catch (error: any) {
  if (error.code === "SecretNotFound") {
    console.log("Secret does not exist");
  } else {
    throw error;
  }
}
```

## Thực hành Tốt nhất

1. **Sử dụng DefaultAzureCredential** - Hoạt động xuyên suốt dev và sản xuất
2. **Bật soft-delete** - Bắt buộc cho các vault sản xuất
3. **Đặt ngày hết hạn** - Trên cả khóa và secret
4. **Sử dụng chính sách xoay vòng khóa** - Tự động hóa việc xoay vòng khóa
5. **Giới hạn các thao tác khóa** - Chỉ cấp các quyền cần thiết (encrypt, sign, v.v.)
6. **Không hỗ trợ trình duyệt** - Các SDK này chỉ dành cho Node.js
