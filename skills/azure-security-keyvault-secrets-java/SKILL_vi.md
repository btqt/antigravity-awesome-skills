---
name: azure-security-keyvault-secrets-java
description: Azure Key Vault Secrets Java SDK để quản lý bí mật. Sử dụng khi lưu trữ, truy xuất, hoặc quản lý mật khẩu, khóa API, chuỗi kết nối, hoặc dữ liệu cấu hình nhạy cảm khác.
package: com.azure:azure-security-keyvault-secrets
---

# Azure Key Vault Secrets (Java)

Lưu trữ và quản lý an toàn các bí mật như mật khẩu, khóa API, và chuỗi kết nối.

## Cài đặt

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-security-keyvault-secrets</artifactId>
    <version>4.9.0</version>
</dependency>
```

## Tạo Client

```java
import com.azure.security.keyvault.secrets.SecretClient;
import com.azure.security.keyvault.secrets.SecretClientBuilder;
import com.azure.identity.DefaultAzureCredentialBuilder;

// Sync client
SecretClient secretClient = new SecretClientBuilder()
    .vaultUrl("https://<vault-name>.vault.azure.net")
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildClient();

// Async client
SecretAsyncClient secretAsyncClient = new SecretClientBuilder()
    .vaultUrl("https://<vault-name>.vault.azure.net")
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildAsyncClient();
```

## Tạo/Đặt Bí mật

```java
import com.azure.security.keyvault.secrets.models.KeyVaultSecret;

// Bí mật đơn giản
KeyVaultSecret secret = secretClient.setSecret("database-password", "P@ssw0rd123!");
System.out.println("Secret name: " + secret.getName());
System.out.println("Secret ID: " + secret.getId());

// Bí mật với các tùy chọn
KeyVaultSecret secretWithOptions = secretClient.setSecret(
    new KeyVaultSecret("api-key", "sk_live_abc123xyz")
        .setProperties(new SecretProperties()
            .setContentType("application/json")
            .setExpiresOn(OffsetDateTime.now().plusYears(1))
            .setNotBefore(OffsetDateTime.now())
            .setEnabled(true)
            .setTags(Map.of(
                "environment", "production",
                "service", "payment-api"
            ))
        )
);
```

## Lấy Bí mật

```java
// Lấy phiên bản mới nhất
KeyVaultSecret secret = secretClient.getSecret("database-password");
String value = secret.getValue();
System.out.println("Secret value: " + value);

// Lấy phiên bản cụ thể
KeyVaultSecret specificVersion = secretClient.getSecret("database-password", "<version-id>");

// Chỉ lấy các thuộc tính (không có giá trị)
SecretProperties props = secretClient.getSecret("database-password").getProperties();
System.out.println("Enabled: " + props.isEnabled());
System.out.println("Created: " + props.getCreatedOn());
```

## Cập nhật Thuộc tính Bí mật

```java
// Lấy bí mật
KeyVaultSecret secret = secretClient.getSecret("api-key");

// Cập nhật thuộc tính (không thể cập nhật giá trị - hãy tạo phiên bản mới thay thế)
secret.getProperties()
    .setEnabled(false)
    .setExpiresOn(OffsetDateTime.now().plusMonths(6))
    .setTags(Map.of("status", "rotating"));

SecretProperties updated = secretClient.updateSecretProperties(secret.getProperties());
System.out.println("Updated: " + updated.getUpdatedOn());
```

## Liệt kê Bí mật

```java
import com.azure.core.util.paging.PagedIterable;
import com.azure.security.keyvault.secrets.models.SecretProperties;

// Liệt kê tất cả các bí mật (chỉ thuộc tính, không có giá trị)
for (SecretProperties secretProps : secretClient.listPropertiesOfSecrets()) {
    System.out.println("Secret: " + secretProps.getName());
    System.out.println("  Enabled: " + secretProps.isEnabled());
    System.out.println("  Created: " + secretProps.getCreatedOn());
    System.out.println("  Content-Type: " + secretProps.getContentType());

    // Lấy giá trị nếu cần
    if (secretProps.isEnabled()) {
        KeyVaultSecret fullSecret = secretClient.getSecret(secretProps.getName());
        System.out.println("  Value: " + fullSecret.getValue().substring(0, 5) + "...");
    }
}

// Liệt kê các phiên bản của một bí mật
for (SecretProperties version : secretClient.listPropertiesOfSecretVersions("database-password")) {
    System.out.println("Version: " + version.getVersion());
    System.out.println("Created: " + version.getCreatedOn());
    System.out.println("Enabled: " + version.isEnabled());
}
```

## Xóa Bí mật

```java
import com.azure.core.util.polling.SyncPoller;
import com.azure.security.keyvault.secrets.models.DeletedSecret;

// Bắt đầu xóa (trả về poller cho các vaults đã bật soft-delete)
SyncPoller<DeletedSecret, Void> deletePoller = secretClient.beginDeleteSecret("old-secret");

// Đợi việc xóa
DeletedSecret deletedSecret = deletePoller.poll().getValue();
System.out.println("Deleted on: " + deletedSecret.getDeletedOn());
System.out.println("Scheduled purge: " + deletedSecret.getScheduledPurgeDate());

deletePoller.waitForCompletion();
```

## Khôi phục Bí mật đã Xóa

```java
// Liệt kê các bí mật đã xóa
for (DeletedSecret deleted : secretClient.listDeletedSecrets()) {
    System.out.println("Deleted: " + deleted.getName());
    System.out.println("Deletion date: " + deleted.getDeletedOn());
}

// Khôi phục bí mật đã xóa
SyncPoller<KeyVaultSecret, Void> recoverPoller = secretClient.beginRecoverDeletedSecret("old-secret");
recoverPoller.waitForCompletion();

KeyVaultSecret recovered = recoverPoller.getFinalResult();
System.out.println("Recovered: " + recovered.getName());
```

## Purge Bí mật đã Xóa

```java
// Xóa vĩnh viễn (không thể khôi phục)
secretClient.purgeDeletedSecret("old-secret");

// Lấy thông tin bí mật đã xóa trước
DeletedSecret deleted = secretClient.getDeletedSecret("old-secret");
System.out.println("Will purge: " + deleted.getName());
secretClient.purgeDeletedSecret("old-secret");
```

## Sao lưu và Khôi phục

```java
// Sao lưu bí mật (tất cả các phiên bản)
byte[] backup = secretClient.backupSecret("important-secret");

// Lưu vào tệp
Files.write(Paths.get("secret-backup.blob"), backup);

// Khôi phục từ bản sao lưu
byte[] backupData = Files.readAllBytes(Paths.get("secret-backup.blob"));
KeyVaultSecret restored = secretClient.restoreSecretBackup(backupData);
System.out.println("Restored: " + restored.getName());
```

## Các Hoạt động Bất đồng bộ (Async Operations)

```java
SecretAsyncClient asyncClient = new SecretClientBuilder()
    .vaultUrl("https://<vault>.vault.azure.net")
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildAsyncClient();

// Thiết lập bí mật async
asyncClient.setSecret("async-secret", "async-value")
    .subscribe(
        secret -> System.out.println("Created: " + secret.getName()),
        error -> System.out.println("Error: " + error.getMessage())
    );

// Lấy bí mật async
asyncClient.getSecret("async-secret")
    .subscribe(secret -> System.out.println("Value: " + secret.getValue()));

// Liệt kê bí mật async
asyncClient.listPropertiesOfSecrets()
    .doOnNext(props -> System.out.println("Found: " + props.getName()))
    .subscribe();
```

## Các Mẫu Cấu hình

### Tải Nhiều Bí mật

```java
public class ConfigLoader {
    private final SecretClient client;

    public ConfigLoader(String vaultUrl) {
        this.client = new SecretClientBuilder()
            .vaultUrl(vaultUrl)
            .credential(new DefaultAzureCredentialBuilder().build())
            .buildClient();
    }

    public Map<String, String> loadSecrets(List<String> secretNames) {
        Map<String, String> secrets = new HashMap<>();
        for (String name : secretNames) {
            try {
                KeyVaultSecret secret = client.getSecret(name);
                secrets.put(name, secret.getValue());
            } catch (ResourceNotFoundException e) {
                System.out.println("Secret not found: " + name);
            }
        }
        return secrets;
    }
}

// Sử dụng
ConfigLoader loader = new ConfigLoader("https://my-vault.vault.azure.net");
Map<String, String> config = loader.loadSecrets(
    Arrays.asList("db-connection-string", "api-key", "jwt-secret")
);
```

### Mẫu Xoay vòng Bí mật

```java
public void rotateSecret(String secretName, String newValue) {
    // Lấy bí mật hiện tại
    KeyVaultSecret current = secretClient.getSecret(secretName);

    // Vô hiệu hóa phiên bản cũ
    current.getProperties().setEnabled(false);
    secretClient.updateSecretProperties(current.getProperties());

    // Tạo phiên bản mới với giá trị mới
    KeyVaultSecret newSecret = secretClient.setSecret(secretName, newValue);
    System.out.println("Rotated to version: " + newSecret.getProperties().getVersion());
}
```

## Xử lý Lỗi

```java
import com.azure.core.exception.HttpResponseException;
import com.azure.core.exception.ResourceNotFoundException;

try {
    KeyVaultSecret secret = secretClient.getSecret("my-secret");
    System.out.println("Value: " + secret.getValue());
} catch (ResourceNotFoundException e) {
    System.out.println("Secret not found");
} catch (HttpResponseException e) {
    int status = e.getResponse().getStatusCode();
    if (status == 403) {
        System.out.println("Access denied - check permissions");
    } else if (status == 429) {
        System.out.println("Rate limited - retry later");
    } else {
        System.out.println("HTTP error: " + status);
    }
}
```

## Các Thuộc tính Bí mật

| Thuộc tính      | Mô tả                                       |
| --------------- | ------------------------------------------- |
| `name`          | Tên bí mật                                  |
| `value`         | Giá trị bí mật (chuỗi)                      |
| `id`            | URL định danh đầy đủ                        |
| `contentType`   | Gợi ý loại MIME                             |
| `enabled`       | Liệu bí mật có thể được truy xuất hay không |
| `notBefore`     | Thời gian kích hoạt                         |
| `expiresOn`     | Thời gian hết hạn                           |
| `createdOn`     | Dấu thời gian tạo                           |
| `updatedOn`     | Dấu thời gian cập nhật lần cuối             |
| `recoveryLevel` | Cấp độ khôi phục soft-delete                |
| `tags`          | Metadata do người dùng định nghĩa           |

## Biến Môi trường

```bash
AZURE_KEYVAULT_URL=https://<vault-name>.vault.azure.net
```

## Thực hành Tốt nhất

1. **Bật Soft Delete** - Bảo vệ chống lại việc xóa ngẫu nhiên
2. **Sử dụng Tags** - Gắn thẻ bí mật với môi trường, dịch vụ, chủ sở hữu
3. **Thiết lập Hết hạn** - Sử dụng `setExpiresOn()` cho các thông tin xác thực cần xoay vòng
4. **Loại Nội dung** - Đặt `contentType` để chỉ định định dạng (ví dụ: `application/json`)
5. **Quản lý Phiên bản** - Đừng xóa các phiên bản cũ ngay lập tức trong quá trình xoay vòng
6. **Nhật ký Truy cập** - Bật nhật ký chẩn đoán trên Key Vault
7. **Quyền hạn Tối thiểu** - Sử dụng các vaults riêng biệt cho các môi trường khác nhau

## Các Loại Bí mật Phổ biến

```java
// Chuỗi kết nối cơ sở dữ liệu
secretClient.setSecret(new KeyVaultSecret("db-connection",
    "Server=myserver.database.windows.net;Database=mydb;...")
    .setProperties(new SecretProperties()
        .setContentType("text/plain")
        .setTags(Map.of("type", "connection-string"))));

// Khóa API
secretClient.setSecret(new KeyVaultSecret("stripe-api-key", "sk_live_...")
    .setProperties(new SecretProperties()
        .setContentType("text/plain")
        .setExpiresOn(OffsetDateTime.now().plusYears(1))));

// Cấu hình JSON
secretClient.setSecret(new KeyVaultSecret("app-config",
    "{\"endpoint\":\"https://...\",\"key\":\"...\"}")
    .setProperties(new SecretProperties()
        .setContentType("application/json")));

// Mật khẩu chứng chỉ
secretClient.setSecret(new KeyVaultSecret("cert-password", "CertP@ss!")
    .setProperties(new SecretProperties()
        .setContentType("text/plain")
        .setTags(Map.of("certificate", "my-cert"))));
```

## Trigger Phrases

- "Key Vault secrets Java", "secret management Java"
- "store password", "store API key", "connection string"
- "retrieve secret", "rotate secret"
- "Azure secrets", "vault secrets"
