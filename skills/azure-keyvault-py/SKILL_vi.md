---
name: azure-keyvault-py
description: |
  Azure Key Vault SDK cho Python. Sử dụng để quản lý secrets, keys, và certificates với kho lưu trữ an toàn.
  Triggers: "key vault", "SecretClient", "KeyClient", "CertificateClient", "secrets", "encryption keys".
package: azure-keyvault-secrets, azure-keyvault-keys, azure-keyvault-certificates
---

# Azure Key Vault SDK cho Python

Lưu trữ và quản lý an toàn cho secrets, khóa mật mã (cryptographic keys), và chứng chỉ (certificates).

## Cài đặt

```bash
# Secrets
pip install azure-keyvault-secrets azure-identity

# Keys (các thao tác mật mã)
pip install azure-keyvault-keys azure-identity

# Certificates (Chứng chỉ)
pip install azure-keyvault-certificates azure-identity

# Tất cả
pip install azure-keyvault-secrets azure-keyvault-keys azure-keyvault-certificates azure-identity
```

## Biến Môi trường

```bash
AZURE_KEYVAULT_URL=https://<vault-name>.vault.azure.net/
```

## Secrets

### Thiết lập SecretClient

```python
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

credential = DefaultAzureCredential()
vault_url = "https://<vault-name>.vault.azure.net/"

client = SecretClient(vault_url=vault_url, credential=credential)
```

### Các Thao tác Secret

```python
# Đặt secret
secret = client.set_secret("database-password", "super-secret-value")
print(f"Created: {secret.name}, version: {secret.properties.version}")

# Lấy secret
secret = client.get_secret("database-password")
print(f"Value: {secret.value}")

# Lấy phiên bản cụ thể
secret = client.get_secret("database-password", version="abc123")

# Liệt kê secrets (chỉ tên, không phải giá trị)
for secret_properties in client.list_properties_of_secrets():
    print(f"Secret: {secret_properties.name}")

# Liệt kê các phiên bản
for version in client.list_properties_of_secret_versions("database-password"):
    print(f"Version: {version.version}, Created: {version.created_on}")

# Xóa secret (xóa mềm)
poller = client.begin_delete_secret("database-password")
deleted_secret = poller.result()

# Xóa vĩnh viễn (Purge, nếu soft-delete được bật)
client.purge_deleted_secret("database-password")

# Khôi phục secret đã xóa
client.begin_recover_deleted_secret("database-password").result()
```

## Keys

### Thiết lập KeyClient

```python
from azure.identity import DefaultAzureCredential
from azure.keyvault.keys import KeyClient

credential = DefaultAzureCredential()
vault_url = "https://<vault-name>.vault.azure.net/"

client = KeyClient(vault_url=vault_url, credential=credential)
```

### Các Thao tác Key

```python
from azure.keyvault.keys import KeyType

# Tạo khóa RSA
rsa_key = client.create_rsa_key("rsa-key", size=2048)

# Tạo khóa EC
ec_key = client.create_ec_key("ec-key", curve="P-256")

# Lấy khóa
key = client.get_key("rsa-key")
print(f"Key type: {key.key_type}")

# Liệt kê các khóa
for key_properties in client.list_properties_of_keys():
    print(f"Key: {key_properties.name}")

# Xóa khóa
poller = client.begin_delete_key("rsa-key")
deleted_key = poller.result()
```

### Các Thao tác Mật mã

```python
from azure.keyvault.keys.crypto import CryptographyClient, EncryptionAlgorithm

# Lấy crypto client cho một khóa cụ thể
crypto_client = CryptographyClient(key, credential=credential)
# Hoặc từ key ID
crypto_client = CryptographyClient(
    "https://<vault>.vault.azure.net/keys/<key-name>/<version>",
    credential=credential
)

# Mã hóa
plaintext = b"Hello, Key Vault!"
result = crypto_client.encrypt(EncryptionAlgorithm.rsa_oaep, plaintext)
ciphertext = result.ciphertext

# Giải mã
result = crypto_client.decrypt(EncryptionAlgorithm.rsa_oaep, ciphertext)
decrypted = result.plaintext

# Ký
from azure.keyvault.keys.crypto import SignatureAlgorithm
import hashlib

digest = hashlib.sha256(b"data to sign").digest()
result = crypto_client.sign(SignatureAlgorithm.rs256, digest)
signature = result.signature

# Xác minh
result = crypto_client.verify(SignatureAlgorithm.rs256, digest, signature)
print(f"Valid: {result.is_valid}")
```

## Certificates

### Thiết lập CertificateClient

```python
from azure.identity import DefaultAzureCredential
from azure.keyvault.certificates import CertificateClient, CertificatePolicy

credential = DefaultAzureCredential()
vault_url = "https://<vault-name>.vault.azure.net/"

client = CertificateClient(vault_url=vault_url, credential=credential)
```

### Các Thao tác Certificate

```python
# Tạo chứng chỉ tự ký (self-signed)
policy = CertificatePolicy.get_default()
poller = client.begin_create_certificate("my-cert", policy=policy)
certificate = poller.result()

# Lấy chứng chỉ
certificate = client.get_certificate("my-cert")
print(f"Thumbprint: {certificate.properties.x509_thumbprint.hex()}")

# Lấy chứng chỉ với khóa riêng tư (như là secret)
from azure.keyvault.secrets import SecretClient
secret_client = SecretClient(vault_url=vault_url, credential=credential)
cert_secret = secret_client.get_secret("my-cert")
# cert_secret.value chứa PEM hoặc PKCS12

# Liệt kê các chứng chỉ
for cert in client.list_properties_of_certificates():
    print(f"Certificate: {cert.name}")

# Xóa chứng chỉ
poller = client.begin_delete_certificate("my-cert")
deleted = poller.result()
```

## Bảng Các Loại Client

| Client               | Gói (Package)                 | Mục đích                   |
| -------------------- | ----------------------------- | -------------------------- |
| `SecretClient`       | `azure-keyvault-secrets`      | Lưu trữ/truy xuất secret   |
| `KeyClient`          | `azure-keyvault-keys`         | Quản lý khóa mật mã        |
| `CryptographyClient` | `azure-keyvault-keys`         | Mã hóa/giải mã/ký/xác minh |
| `CertificateClient`  | `azure-keyvault-certificates` | Quản lý chứng chỉ          |

## Async Clients

```python
from azure.identity.aio import DefaultAzureCredential
from azure.keyvault.secrets.aio import SecretClient

async def get_secret():
    credential = DefaultAzureCredential()
    client = SecretClient(vault_url=vault_url, credential=credential)

    async with client:
        secret = await client.get_secret("my-secret")
        print(secret.value)

import asyncio
asyncio.run(get_secret())
```

## Xử lý Lỗi

```python
from azure.core.exceptions import ResourceNotFoundError, HttpResponseError

try:
    secret = client.get_secret("nonexistent")
except ResourceNotFoundError:
    print("Secret not found")
except HttpResponseError as e:
    if e.status_code == 403:
        print("Access denied - check RBAC permissions")
    raise
```

## Thực hành Tốt nhất

1. **Sử dụng DefaultAzureCredential** cho xác thực
2. **Sử dụng managed identity** trong các ứng dụng được host trên Azure
3. **Bật soft-delete** để khôi phục (được bật theo mặc định)
4. **Sử dụng RBAC** thay vì chính sách truy cập (access policies) để kiểm soát chi tiết
5. **Xoay vòng secrets** thường xuyên sử dụng versioning
6. **Sử dụng Key Vault references** trong cấu hình App Service/Functions
7. **Cache secrets** phù hợp để giảm các cuộc gọi API
8. **Sử dụng async clients** cho các kịch bản throughput cao
