---
name: azure-security-keyvault-keys-dotnet
description: |
  Azure Key Vault Keys SDK cho .NET. Thư viện client để quản lý các khóa mật mã trong Azure Key Vault và Managed HSM. Sử dụng để tạo khóa, xoay vòng (rotation), mã hóa, giải mã, ký, và xác minh. Triggers: "Key Vault keys", "KeyClient", "CryptographyClient", "RSA key", "EC key", "encrypt decrypt .NET", "key rotation", "HSM".
package: Azure.Security.KeyVault.Keys
---

# Azure.Security.KeyVault.Keys (.NET)

Thư viện client để quản lý các khóa mật mã trong Azure Key Vault và Managed HSM.

## Cài đặt

```bash
dotnet add package Azure.Security.KeyVault.Keys
dotnet add package Azure.Identity
```

**Phiên bản Hiện tại**: 4.7.0 (stable)

## Biến Môi trường

```bash
KEY_VAULT_NAME=<your-key-vault-name>
# Hoặc URI đầy đủ
AZURE_KEYVAULT_URL=https://<vault-name>.vault.azure.net
```

## Phân cấp Client

```
KeyClient (quản lý khóa)
├── CreateKey / CreateRsaKey / CreateEcKey
├── GetKey / GetKeys
├── UpdateKeyProperties
├── DeleteKey / PurgeDeletedKey
├── BackupKey / RestoreKey
└── GetCryptographyClient() → CryptographyClient

CryptographyClient (các hoạt động mật mã)
├── Encrypt / Decrypt
├── WrapKey / UnwrapKey
├── Sign / Verify
└── SignData / VerifyData

KeyResolver (phân giải khóa)
└── Resolve(keyId) → CryptographyClient
```

## Xác thực

### DefaultAzureCredential (Khuyên dùng)

```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Keys;

var keyVaultName = Environment.GetEnvironmentVariable("KEY_VAULT_NAME");
var kvUri = $"https://{keyVaultName}.vault.azure.net";

var client = new KeyClient(new Uri(kvUri), new DefaultAzureCredential());
```

### Service Principal

```csharp
var credential = new ClientSecretCredential(
    tenantId: "<tenant-id>",
    clientId: "<client-id>",
    clientSecret: "<client-secret>");

var client = new KeyClient(new Uri(kvUri), credential);
```

## Quản lý Khóa

### Tạo Khóa

```csharp
// Tạo RSA key
KeyVaultKey rsaKey = await client.CreateKeyAsync("my-rsa-key", KeyType.Rsa);
Console.WriteLine($"Created key: {rsaKey.Name}, Type: {rsaKey.KeyType}");

// Tạo RSA key với các tùy chọn
var rsaOptions = new CreateRsaKeyOptions("my-rsa-key-2048")
{
    KeySize = 2048,
    HardwareProtected = false, // true cho HSM-backed
    ExpiresOn = DateTimeOffset.UtcNow.AddYears(1),
    NotBefore = DateTimeOffset.UtcNow,
    Enabled = true
};
rsaOptions.KeyOperations.Add(KeyOperation.Encrypt);
rsaOptions.KeyOperations.Add(KeyOperation.Decrypt);

KeyVaultKey rsaKey2 = await client.CreateRsaKeyAsync(rsaOptions);

// Tạo EC key
var ecOptions = new CreateEcKeyOptions("my-ec-key")
{
    CurveName = KeyCurveName.P256,
    HardwareProtected = true // HSM-backed
};
KeyVaultKey ecKey = await client.CreateEcKeyAsync(ecOptions);

// Tạo Oct (symmetric) key để wrap/unwrap
var octOptions = new CreateOctKeyOptions("my-oct-key")
{
    KeySize = 256,
    HardwareProtected = true
};
KeyVaultKey octKey = await client.CreateOctKeyAsync(octOptions);
```

### Truy xuất Khóa

```csharp
// Lấy khóa cụ thể (phiên bản mới nhất)
KeyVaultKey key = await client.GetKeyAsync("my-rsa-key");
Console.WriteLine($"Key ID: {key.Id}");
Console.WriteLine($"Key Type: {key.KeyType}");
Console.WriteLine($"Version: {key.Properties.Version}");

// Lấy phiên bản cụ thể
KeyVaultKey keyVersion = await client.GetKeyAsync("my-rsa-key", "version-id");

// Liệt kê tất cả các khóa
await foreach (KeyProperties keyProps in client.GetPropertiesOfKeysAsync())
{
    Console.WriteLine($"Key: {keyProps.Name}, Enabled: {keyProps.Enabled}");
}

// Liệt kê các phiên bản của khóa
await foreach (KeyProperties version in client.GetPropertiesOfKeyVersionsAsync("my-rsa-key"))
{
    Console.WriteLine($"Version: {version.Version}, Created: {version.CreatedOn}");
}
```

### Cập nhật Thuộc tính Khóa

```csharp
KeyVaultKey key = await client.GetKeyAsync("my-rsa-key");

key.Properties.ExpiresOn = DateTimeOffset.UtcNow.AddYears(2);
key.Properties.Tags["environment"] = "production";

KeyVaultKey updatedKey = await client.UpdateKeyPropertiesAsync(key.Properties);
```

### Xóa và Purge Khóa

```csharp
// Bắt đầu hoạt động xóa
DeleteKeyOperation operation = await client.StartDeleteKeyAsync("my-rsa-key");

// Đợi việc xóa hoàn tất (bắt buộc trước khi purge)
await operation.WaitForCompletionAsync();
Console.WriteLine($"Deleted key scheduled purge date: {operation.Value.ScheduledPurgeDate}");

// Purge ngay lập tức (nếu soft-delete được bật)
await client.PurgeDeletedKeyAsync("my-rsa-key");

// Hoặc khôi phục khóa đã xóa
KeyVaultKey recoveredKey = await client.StartRecoverDeletedKeyAsync("my-rsa-key");
```

### Sao lưu và Khôi phục

```csharp
// Sao lưu khóa
byte[] backup = await client.BackupKeyAsync("my-rsa-key");
await File.WriteAllBytesAsync("key-backup.bin", backup);

// Khôi phục khóa
byte[] backupData = await File.ReadAllBytesAsync("key-backup.bin");
KeyVaultKey restoredKey = await client.RestoreKeyBackupAsync(backupData);
```

## Các Hoạt động Mật mã

### Lấy CryptographyClient

```csharp
// Từ KeyClient
KeyVaultKey key = await client.GetKeyAsync("my-rsa-key");
CryptographyClient cryptoClient = client.GetCryptographyClient(
    key.Name,
    key.Properties.Version);

// Hoặc tạo trực tiếp với key ID
CryptographyClient cryptoClient = new CryptographyClient(
    new Uri("https://myvault.vault.azure.net/keys/my-rsa-key/version"),
    new DefaultAzureCredential());
```

### Mã hóa và Giải mã

```csharp
byte[] plaintext = Encoding.UTF8.GetBytes("Secret message to encrypt");

// Mã hóa
EncryptResult encryptResult = await cryptoClient.EncryptAsync(
    EncryptionAlgorithm.RsaOaep256,
    plaintext);
Console.WriteLine($"Encrypted: {Convert.ToBase64String(encryptResult.Ciphertext)}");

// Giải mã
DecryptResult decryptResult = await cryptoClient.DecryptAsync(
    EncryptionAlgorithm.RsaOaep256,
    encryptResult.Ciphertext);
string decrypted = Encoding.UTF8.GetString(decryptResult.Plaintext);
Console.WriteLine($"Decrypted: {decrypted}");
```

### Wrap và Unwrap Khóa

```csharp
// Khóa để wrap (ví dụ: AES key)
byte[] keyToWrap = new byte[32]; // 256-bit key
RandomNumberGenerator.Fill(keyToWrap);

// Wrap khóa
WrapResult wrapResult = await cryptoClient.WrapKeyAsync(
    KeyWrapAlgorithm.RsaOaep256,
    keyToWrap);

// Unwrap khóa
UnwrapResult unwrapResult = await cryptoClient.UnwrapKeyAsync(
    KeyWrapAlgorithm.RsaOaep256,
    wrapResult.EncryptedKey);
```

### Ký và Xác minh

```csharp
// Dữ liệu để ký
byte[] data = Encoding.UTF8.GetBytes("Data to sign");

// Ký dữ liệu (tính toán hash bên trong)
SignResult signResult = await cryptoClient.SignDataAsync(
    SignatureAlgorithm.RS256,
    data);

// Xác minh chữ ký
VerifyResult verifyResult = await cryptoClient.VerifyDataAsync(
    SignatureAlgorithm.RS256,
    data,
    signResult.Signature);
Console.WriteLine($"Signature valid: {verifyResult.IsValid}");

// Hoặc ký hash đã tính toán trước
using var sha256 = SHA256.Create();
byte[] hash = sha256.ComputeHash(data);

SignResult signHashResult = await cryptoClient.SignAsync(
    SignatureAlgorithm.RS256,
    hash);
```

## Key Resolver

```csharp
using Azure.Security.KeyVault.Keys.Cryptography;

var resolver = new KeyResolver(new DefaultAzureCredential());

// Phân giải key bằng ID để lấy CryptographyClient
CryptographyClient cryptoClient = await resolver.ResolveAsync(
    new Uri("https://myvault.vault.azure.net/keys/my-key/version"));

// Sử dụng để mã hóa
EncryptResult result = await cryptoClient.EncryptAsync(
    EncryptionAlgorithm.RsaOaep256,
    plaintext);
```

## Xoay vòng Khóa (Key Rotation)

```csharp
// Xoay vòng khóa (tạo phiên bản mới)
KeyVaultKey rotatedKey = await client.RotateKeyAsync("my-rsa-key");
Console.WriteLine($"New version: {rotatedKey.Properties.Version}");

// Lấy chính sách xoay vòng
KeyRotationPolicy policy = await client.GetKeyRotationPolicyAsync("my-rsa-key");

// Cập nhật chính sách xoay vòng
policy.ExpiresIn = "P90D"; // 90 ngày
policy.LifetimeActions.Add(new KeyRotationLifetimeAction
{
    Action = KeyRotationPolicyAction.Rotate,
    TimeBeforeExpiry = "P30D" // Xoay vòng 30 ngày trước khi hết hạn
});

await client.UpdateKeyRotationPolicyAsync("my-rsa-key", policy);
```

## Tham chiếu Các Loại Chính

| Loại                  | Mục đích                                     |
| --------------------- | -------------------------------------------- |
| `KeyClient`           | Các hoạt động quản lý khóa                   |
| `CryptographyClient`  | Các hoạt động mật mã                         |
| `KeyResolver`         | Phân giải key ID thành CryptographyClient    |
| `KeyVaultKey`         | Khóa với tài liệu mật mã                     |
| `KeyProperties`       | Metadata của khóa (không có tài liệu mật mã) |
| `CreateRsaKeyOptions` | Tùy chọn tạo khóa RSA                        |
| `CreateEcKeyOptions`  | Tùy chọn tạo khóa EC                         |
| `CreateOctKeyOptions` | Tùy chọn khóa đối xứng                       |
| `EncryptResult`       | Kết quả mã hóa                               |
| `DecryptResult`       | Kết quả giải mã                              |
| `SignResult`          | Kết quả ký                                   |
| `VerifyResult`        | Kết quả xác minh                             |
| `WrapResult`          | Kết quả wrap khóa                            |
| `UnwrapResult`        | Kết quả unwrap khóa                          |

## Tham chiếu Thuật toán

### Thuật toán Mã hóa

| Thuật toán   | Loại Khóa | Mô tả            |
| ------------ | --------- | ---------------- |
| `RsaOaep`    | RSA       | RSA-OAEP         |
| `RsaOaep256` | RSA       | RSA-OAEP-256     |
| `Rsa15`      | RSA       | RSA 1.5 (legacy) |
| `A128Gcm`    | Oct       | AES-128-GCM      |
| `A256Gcm`    | Oct       | AES-256-GCM      |

### Thuật toán Chữ ký

| Thuật toán | Loại Khóa | Mô tả                     |
| ---------- | --------- | ------------------------- |
| `RS256`    | RSA       | RSASSA-PKCS1-v1_5 SHA-256 |
| `RS384`    | RSA       | RSASSA-PKCS1-v1_5 SHA-384 |
| `RS512`    | RSA       | RSASSA-PKCS1-v1_5 SHA-512 |
| `PS256`    | RSA       | RSASSA-PSS SHA-256        |
| `ES256`    | EC        | ECDSA P-256 SHA-256       |
| `ES384`    | EC        | ECDSA P-384 SHA-384       |
| `ES512`    | EC        | ECDSA P-521 SHA-512       |

### Thuật toán Wrap Khóa

| Thuật toán   | Loại Khóa | Mô tả            |
| ------------ | --------- | ---------------- |
| `RsaOaep`    | RSA       | RSA-OAEP         |
| `RsaOaep256` | RSA       | RSA-OAEP-256     |
| `A128KW`     | Oct       | AES-128 Key Wrap |
| `A256KW`     | Oct       | AES-256 Key Wrap |

## Thực hành Tốt nhất

1. **Sử dụng Managed Identity** — Ưu tiên `DefaultAzureCredential` hơn là secrets
2. **Bật soft-delete** — Bảo vệ chống lại việc vô tình xóa
3. **Sử dụng HSM-backed keys** — Thiết lập `HardwareProtected = true` cho các khóa nhạy cảm
4. **Triển khai xoay vòng khóa** — Sử dụng các chính sách xoay vòng tự động
5. **Hạn chế các hoạt động khóa** — Chỉ bật các `KeyOperations` cần thiết
6. **Thiết lập ngày hết hạn** — Luôn thiết lập `ExpiresOn` cho các khóa
7. **Sử dụng các phiên bản cụ thể** — Pin vào các phiên bản trong production
8. **Cache CryptographyClient** — Tái sử dụng cho nhiều hoạt động

## Xử lý Lỗi

```csharp
using Azure;

try
{
    KeyVaultKey key = await client.GetKeyAsync("my-key");
}
catch (RequestFailedException ex) when (ex.Status == 404)
{
    Console.WriteLine("Key not found");
}
catch (RequestFailedException ex) when (ex.Status == 403)
{
    Console.WriteLine("Access denied - check RBAC permissions");
}
catch (RequestFailedException ex)
{
    Console.WriteLine($"Key Vault error: {ex.Status} - {ex.Message}");
}
```

## Các Vai trò RBAC Yêu cầu

| Vai trò                  | Quyền hạn                             |
| ------------------------ | ------------------------------------- |
| Key Vault Crypto Officer | Quản lý khóa đầy đủ                   |
| Key Vault Crypto User    | Sử dụng khóa cho các hoạt động mật mã |
| Key Vault Reader         | Đọc metadata của khóa                 |

## Các SDK Liên quan

| SDK                                    | Mục đích       | Cài đặt                                                   |
| -------------------------------------- | -------------- | --------------------------------------------------------- |
| `Azure.Security.KeyVault.Keys`         | Keys (SDK này) | `dotnet add package Azure.Security.KeyVault.Keys`         |
| `Azure.Security.KeyVault.Secrets`      | Secrets        | `dotnet add package Azure.Security.KeyVault.Secrets`      |
| `Azure.Security.KeyVault.Certificates` | Certificates   | `dotnet add package Azure.Security.KeyVault.Certificates` |
| `Azure.Identity`                       | Xác thực       | `dotnet add package Azure.Identity`                       |
