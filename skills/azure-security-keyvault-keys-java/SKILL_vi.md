---
name: azure-security-keyvault-keys-java
description: Azure Key Vault Keys Java SDK để quản lý khóa mật mã. Sử dụng khi tạo, quản lý, hoặc sử dụng các khóa RSA/EC, thực hiện các hoạt động mã hóa/giải mã/ký/xác minh, hoặc làm việc với các khóa được hỗ trợ bởi HSM.
package: com.azure:azure-security-keyvault-keys
---

# Azure Key Vault Keys (Java)

Quản lý các khóa mật mã và thực hiện các hoạt động mật mã trong Azure Key Vault và Managed HSM.

## Cài đặt

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-security-keyvault-keys</artifactId>
    <version>4.9.0</version>
</dependency>
```

## Tạo Client

```java
import com.azure.security.keyvault.keys.KeyClient;
import com.azure.security.keyvault.keys.KeyClientBuilder;
import com.azure.security.keyvault.keys.cryptography.CryptographyClient;
import com.azure.security.keyvault.keys.cryptography.CryptographyClientBuilder;
import com.azure.identity.DefaultAzureCredentialBuilder;

// Key management client
KeyClient keyClient = new KeyClientBuilder()
    .vaultUrl("https://<vault-name>.vault.azure.net")
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildClient();

// Async client
KeyAsyncClient keyAsyncClient = new KeyClientBuilder()
    .vaultUrl("https://<vault-name>.vault.azure.net")
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildAsyncClient();

// Cryptography client (cho mã hóa/giải mã/ký/xác minh)
CryptographyClient cryptoClient = new CryptographyClientBuilder()
    .keyIdentifier("https://<vault-name>.vault.azure.net/keys/<key-name>/<key-version>")
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildClient();
```

## Các Loại Khóa

| Loại      | Mô tả                            |
| --------- | -------------------------------- |
| `RSA`     | Khóa RSA (2048, 3072, 4096 bits) |
| `RSA_HSM` | Khóa RSA trong HSM               |
| `EC`      | Khóa Elliptic Curve              |
| `EC_HSM`  | Khóa Elliptic Curve trong HSM    |
| `OCT`     | Khóa đối xứng (Chỉ Managed HSM)  |
| `OCT_HSM` | Khóa đối xứng trong HSM          |

## Tạo Khóa

### Tạo Khóa RSA

```java
import com.azure.security.keyvault.keys.models.*;

// Khóa RSA đơn giản
KeyVaultKey rsaKey = keyClient.createRsaKey(new CreateRsaKeyOptions("my-rsa-key")
    .setKeySize(2048));

System.out.println("Key name: " + rsaKey.getName());
System.out.println("Key ID: " + rsaKey.getId());
System.out.println("Key type: " + rsaKey.getKeyType());

// Khóa RSA với các tùy chọn
KeyVaultKey rsaKeyWithOptions = keyClient.createRsaKey(new CreateRsaKeyOptions("my-rsa-key-2")
    .setKeySize(4096)
    .setExpiresOn(OffsetDateTime.now().plusYears(1))
    .setNotBefore(OffsetDateTime.now())
    .setEnabled(true)
    .setKeyOperations(KeyOperation.ENCRYPT, KeyOperation.DECRYPT,
                       KeyOperation.WRAP_KEY, KeyOperation.UNWRAP_KEY)
    .setTags(Map.of("environment", "production")));

// HSM-backed RSA key
KeyVaultKey hsmKey = keyClient.createRsaKey(new CreateRsaKeyOptions("my-hsm-key")
    .setKeySize(2048)
    .setHardwareProtected(true));
```

### Tạo Khóa EC

```java
// Khóa EC với P-256 curve
KeyVaultKey ecKey = keyClient.createEcKey(new CreateEcKeyOptions("my-ec-key")
    .setCurveName(KeyCurveName.P_256));

// Khóa EC với các curves khác
KeyVaultKey ecKey384 = keyClient.createEcKey(new CreateEcKeyOptions("my-ec-key-384")
    .setCurveName(KeyCurveName.P_384));

KeyVaultKey ecKey521 = keyClient.createEcKey(new CreateEcKeyOptions("my-ec-key-521")
    .setCurveName(KeyCurveName.P_521));

// HSM-backed EC key
KeyVaultKey ecHsmKey = keyClient.createEcKey(new CreateEcKeyOptions("my-ec-hsm-key")
    .setCurveName(KeyCurveName.P_256)
    .setHardwareProtected(true));
```

### Tạo Khóa Đối xứng (Chỉ Managed HSM)

```java
KeyVaultKey octKey = keyClient.createOctKey(new CreateOctKeyOptions("my-symmetric-key")
    .setKeySize(256)
    .setHardwareProtected(true));
```

## Lấy Khóa

```java
// Lấy phiên bản mới nhất
KeyVaultKey key = keyClient.getKey("my-key");

// Lấy phiên bản cụ thể
KeyVaultKey keyVersion = keyClient.getKey("my-key", "<version-id>");

// Chỉ lấy thuộc tính khóa (không có tài liệu khóa)
KeyProperties keyProps = keyClient.getKey("my-key").getProperties();
```

## Cập nhật Thuộc tính Khóa

```java
KeyVaultKey key = keyClient.getKey("my-key");

// Cập nhật thuộc tính
key.getProperties()
    .setEnabled(false)
    .setExpiresOn(OffsetDateTime.now().plusMonths(6))
    .setTags(Map.of("status", "archived"));

KeyVaultKey updatedKey = keyClient.updateKeyProperties(key.getProperties(),
    KeyOperation.ENCRYPT, KeyOperation.DECRYPT);
```

## Liệt kê Khóa

```java
import com.azure.core.util.paging.PagedIterable;

// Liệt kê tất cả các khóa
for (KeyProperties keyProps : keyClient.listPropertiesOfKeys()) {
    System.out.println("Key: " + keyProps.getName());
    System.out.println("  Enabled: " + keyProps.isEnabled());
    System.out.println("  Created: " + keyProps.getCreatedOn());
}

// Liệt kê các phiên bản của khóa
for (KeyProperties version : keyClient.listPropertiesOfKeyVersions("my-key")) {
    System.out.println("Version: " + version.getVersion());
    System.out.println("Created: " + version.getCreatedOn());
}
```

## Xóa Khóa

```java
import com.azure.core.util.polling.SyncPoller;

// Bắt đầu xóa (đối với vaults đã bật soft-delete)
SyncPoller<DeletedKey, Void> deletePoller = keyClient.beginDeleteKey("my-key");

// Đợi việc xóa
DeletedKey deletedKey = deletePoller.poll().getValue();
System.out.println("Deleted: " + deletedKey.getDeletedOn());

deletePoller.waitForCompletion();

// Purge khóa đã xóa (xóa vĩnh viễn)
keyClient.purgeDeletedKey("my-key");

// Khôi phục khóa đã xóa
SyncPoller<KeyVaultKey, Void> recoverPoller = keyClient.beginRecoverDeletedKey("my-key");
recoverPoller.waitForCompletion();
```

## Các Hoạt động Mật mã

### Mã hóa/Giải mã

```java
import com.azure.security.keyvault.keys.cryptography.models.*;

CryptographyClient cryptoClient = new CryptographyClientBuilder()
    .keyIdentifier("https://<vault>.vault.azure.net/keys/<key-name>")
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildClient();

byte[] plaintext = "Hello, World!".getBytes(StandardCharsets.UTF_8);

// Mã hóa
EncryptResult encryptResult = cryptoClient.encrypt(EncryptionAlgorithm.RSA_OAEP, plaintext);
byte[] ciphertext = encryptResult.getCipherText();
System.out.println("Ciphertext length: " + ciphertext.length);

// Giải mã
DecryptResult decryptResult = cryptoClient.decrypt(EncryptionAlgorithm.RSA_OAEP, ciphertext);
String decrypted = new String(decryptResult.getPlainText(), StandardCharsets.UTF_8);
System.out.println("Decrypted: " + decrypted);
```

### Ký/Xác minh

```java
import java.security.MessageDigest;

// Tạo digest của dữ liệu
byte[] data = "Data to sign".getBytes(StandardCharsets.UTF_8);
MessageDigest md = MessageDigest.getInstance("SHA-256");
byte[] digest = md.digest(data);

// Ký
SignResult signResult = cryptoClient.sign(SignatureAlgorithm.RS256, digest);
byte[] signature = signResult.getSignature();

// Xác minh
VerifyResult verifyResult = cryptoClient.verify(SignatureAlgorithm.RS256, digest, signature);
System.out.println("Valid signature: " + verifyResult.isValid());
```

### Wrap/Unwrap Khóa

```java
// Khóa để wrap (ví dụ: AES key)
byte[] keyToWrap = new byte[32];  // 256-bit key
new SecureRandom().nextBytes(keyToWrap);

// Wrap
WrapResult wrapResult = cryptoClient.wrapKey(KeyWrapAlgorithm.RSA_OAEP, keyToWrap);
byte[] wrappedKey = wrapResult.getEncryptedKey();

// Unwrap
UnwrapResult unwrapResult = cryptoClient.unwrapKey(KeyWrapAlgorithm.RSA_OAEP, wrappedKey);
byte[] unwrappedKey = unwrapResult.getKey();
```

## Sao lưu và Khôi phục

```java
// Sao lưu
byte[] backup = keyClient.backupKey("my-key");

// Lưu bản sao lưu vào tệp
Files.write(Paths.get("key-backup.blob"), backup);

// Khôi phục
byte[] backupData = Files.readAllBytes(Paths.get("key-backup.blob"));
KeyVaultKey restoredKey = keyClient.restoreKeyBackup(backupData);
```

## Xoay vòng Khóa (Key Rotation)

```java
// Xoay vòng sang phiên bản mới
KeyVaultKey rotatedKey = keyClient.rotateKey("my-key");
System.out.println("New version: " + rotatedKey.getProperties().getVersion());

// Thiết lập chính sách xoay vòng
KeyRotationPolicy policy = new KeyRotationPolicy()
    .setExpiresIn("P90D")  // Hết hạn sau 90 ngày
    .setLifetimeActions(Arrays.asList(
        new KeyRotationLifetimeAction(KeyRotationPolicyAction.ROTATE)
            .setTimeBeforeExpiry("P30D")));  // Xoay vòng 30 ngày trước khi hết hạn

keyClient.updateKeyRotationPolicy("my-key", policy);

// Lấy chính sách xoay vòng
KeyRotationPolicy currentPolicy = keyClient.getKeyRotationPolicy("my-key");
```

## Import Khóa

```java
import com.azure.security.keyvault.keys.models.ImportKeyOptions;
import com.azure.security.keyvault.keys.models.JsonWebKey;

// Import tài liệu khóa hiện có
JsonWebKey jsonWebKey = new JsonWebKey()
    .setKeyType(KeyType.RSA)
    .setN(modulus)
    .setE(exponent)
    .setD(privateExponent)
    // ... các thành phần RSA khác
    ;

ImportKeyOptions importOptions = new ImportKeyOptions("imported-key", jsonWebKey)
    .setHardwareProtected(false);

KeyVaultKey importedKey = keyClient.importKey(importOptions);
```

## Các Thuật toán Mã hóa

| Thuật toán     | Loại Khóa | Mô tả                         |
| -------------- | --------- | ----------------------------- |
| `RSA1_5`       | RSA       | RSAES-PKCS1-v1_5              |
| `RSA_OAEP`     | RSA       | RSAES with OAEP (khuyên dùng) |
| `RSA_OAEP_256` | RSA       | RSAES with OAEP using SHA-256 |
| `A128Gcm`      | OCT       | AES-GCM 128-bit               |
| `A256Gcm`      | OCT       | AES-GCM 256-bit               |
| `A128CBC`      | OCT       | AES-CBC 128-bit               |
| `A256CBC`      | OCT       | AES-CBC 256-bit               |

## Các Thuật toán Chữ ký

| Thuật toán | Loại Khóa | Hash          |
| ---------- | --------- | ------------- |
| `RS256`    | RSA       | SHA-256       |
| `RS384`    | RSA       | SHA-384       |
| `RS512`    | RSA       | SHA-512       |
| `PS256`    | RSA       | SHA-256 (PSS) |
| `ES256`    | EC P-256  | SHA-256       |
| `ES384`    | EC P-384  | SHA-384       |
| `ES512`    | EC P-521  | SHA-512       |

## Xử lý Lỗi

```java
import com.azure.core.exception.HttpResponseException;
import com.azure.core.exception.ResourceNotFoundException;

try {
    KeyVaultKey key = keyClient.getKey("non-existent-key");
} catch (ResourceNotFoundException e) {
    System.out.println("Key not found: " + e.getMessage());
} catch (HttpResponseException e) {
    System.out.println("HTTP error " + e.getResponse().getStatusCode());
    System.out.println("Message: " + e.getMessage());
}
```

## Biến Môi trường

```bash
AZURE_KEYVAULT_URL=https://<vault-name>.vault.azure.net
```

## Thực hành Tốt nhất

1. **Sử dụng HSM Keys cho Production** - Đặt `setHardwareProtected(true)` cho các khóa nhạy cảm
2. **Bật Soft Delete** - Bảo vệ chống lại việc xóa ngẫu nhiên
3. **Xoay vòng Khóa** - Thiết lập các chính sách xoay vòng tự động
4. **Quyền hạn Tối thiểu** - Sử dụng các khóa riêng biệt cho các hoạt động khác nhau
5. **Local Crypto khi có thể** - Sử dụng `CryptographyClient` với tài liệu khóa cục bộ để giảm round-trips

## Trigger Phrases

- "Key Vault keys Java", "cryptographic keys Java"
- "encrypt decrypt Java", "sign verify Java"
- "RSA key", "EC key", "HSM key"
- "key rotation", "wrap unwrap key"
