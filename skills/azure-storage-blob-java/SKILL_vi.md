---
name: azure-storage-blob-java
description: Xây dựng các ứng dụng blob storage với Azure Storage Blob SDK cho Java. Sử dụng khi upload, download, hoặc quản lý files trong Azure Blob Storage, làm việc với containers, hoặc triển khai các hoạt động dữ liệu streaming.
package: com.azure:azure-storage-blob
---

# Azure Storage Blob SDK cho Java

Xây dựng các ứng dụng blob storage sử dụng Azure Storage Blob SDK cho Java.

## Cài đặt

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-storage-blob</artifactId>
    <version>12.33.0</version>
</dependency>
```

## Tạo Client

### BlobServiceClient

```java
import com.azure.storage.blob.BlobServiceClient;
import com.azure.storage.blob.BlobServiceClientBuilder;

// Với mã SAS token
BlobServiceClient serviceClient = new BlobServiceClientBuilder()
    .endpoint("<storage-account-url>")
    .sasToken("<sas-token>")
    .buildClient();

// Với chuỗi kết nối
BlobServiceClient serviceClient = new BlobServiceClientBuilder()
    .connectionString("<connection-string>")
    .buildClient();
```

### Với DefaultAzureCredential

```java
import com.azure.identity.DefaultAzureCredentialBuilder;

BlobServiceClient serviceClient = new BlobServiceClientBuilder()
    .endpoint("<storage-account-url>")
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildClient();
```

### BlobContainerClient

```java
import com.azure.storage.blob.BlobContainerClient;

// Từ service client
BlobContainerClient containerClient = serviceClient.getBlobContainerClient("mycontainer");

// Khởi tạo trực tiếp
BlobContainerClient containerClient = new BlobContainerClientBuilder()
    .connectionString("<connection-string>")
    .containerName("mycontainer")
    .buildClient();
```

### BlobClient

```java
import com.azure.storage.blob.BlobClient;

// Từ container client
BlobClient blobClient = containerClient.getBlobClient("myblob.txt");

// Với cấu trúc thư mục
BlobClient blobClient = containerClient.getBlobClient("folder/subfolder/myblob.txt");

// Khởi tạo trực tiếp
BlobClient blobClient = new BlobClientBuilder()
    .connectionString("<connection-string>")
    .containerName("mycontainer")
    .blobName("myblob.txt")
    .buildClient();
```

## Các Mẫu Cốt lõi

### Tạo Container

```java
// Tạo container
serviceClient.createBlobContainer("mycontainer");

// Tạo nếu không tồn tại
BlobContainerClient container = serviceClient.createBlobContainerIfNotExists("mycontainer");

// Từ container client
containerClient.create();
containerClient.createIfNotExists();
```

### Tải lên Dữ liệu (Upload)

```java
import com.azure.core.util.BinaryData;

// Upload chuỗi
String data = "Hello, Azure Blob Storage!";
blobClient.upload(BinaryData.fromString(data));

// Upload có ghi đè
blobClient.upload(BinaryData.fromString(data), true);
```

### Tải lên từ File

```java
blobClient.uploadFromFile("local-file.txt");

// Có ghi đè
blobClient.uploadFromFile("local-file.txt", true);
```

### Tải lên từ Stream

```java
import com.azure.storage.blob.specialized.BlockBlobClient;

BlockBlobClient blockBlobClient = blobClient.getBlockBlobClient();

try (ByteArrayInputStream dataStream = new ByteArrayInputStream(data.getBytes())) {
    blockBlobClient.upload(dataStream, data.length());
}
```

### Tải lên với Các tùy chọn

```java
import com.azure.storage.blob.models.BlobHttpHeaders;
import com.azure.storage.blob.options.BlobParallelUploadOptions;

BlobHttpHeaders headers = new BlobHttpHeaders()
    .setContentType("text/plain")
    .setCacheControl("max-age=3600");

Map<String, String> metadata = Map.of("author", "john", "version", "1.0");

try (InputStream stream = new FileInputStream("large-file.bin")) {
    BlobParallelUploadOptions options = new BlobParallelUploadOptions(stream)
        .setHeaders(headers)
        .setMetadata(metadata);

    blobClient.uploadWithResponse(options, null, Context.NONE);
}
```

### Tải lên nếu Không Tồn tại

```java
import com.azure.storage.blob.models.BlobRequestConditions;

BlobParallelUploadOptions options = new BlobParallelUploadOptions(inputStream, length)
    .setRequestConditions(new BlobRequestConditions().setIfNoneMatch("*"));

blobClient.uploadWithResponse(options, null, Context.NONE);
```

### Tải xuống Dữ liệu (Download)

```java
// Download vào BinaryData
BinaryData content = blobClient.downloadContent();
String text = content.toString();

// Download xuống file
blobClient.downloadToFile("downloaded-file.txt");
```

### Tải xuống vào Stream

```java
try (ByteArrayOutputStream outputStream = new ByteArrayOutputStream()) {
    blobClient.downloadStream(outputStream);
    byte[] data = outputStream.toByteArray();
}
```

### Tải xuống với InputStream

```java
import com.azure.storage.blob.specialized.BlobInputStream;

try (BlobInputStream blobIS = blobClient.openInputStream()) {
    byte[] buffer = new byte[1024];
    int bytesRead;
    while ((bytesRead = blobIS.read(buffer)) != -1) {
        // Xử lý buffer
    }
}
```

### Tải lên qua OutputStream

```java
import com.azure.storage.blob.specialized.BlobOutputStream;

try (BlobOutputStream blobOS = blobClient.getBlockBlobClient().getBlobOutputStream()) {
    blobOS.write("Data to upload".getBytes());
}
```

### Liệt kê Blobs

```java
import com.azure.storage.blob.models.BlobItem;

// Liệt kê tất cả blobs
for (BlobItem blobItem : containerClient.listBlobs()) {
    System.out.println("Blob: " + blobItem.getName());
}

// Liệt kê với prefix (thư mục ảo)
import com.azure.storage.blob.models.ListBlobsOptions;

ListBlobsOptions options = new ListBlobsOptions().setPrefix("folder/");
for (BlobItem blobItem : containerClient.listBlobs(options, null)) {
    System.out.println("Blob: " + blobItem.getName());
}
```

### Liệt kê Blobs theo Hệ thống phân cấp

```java
import com.azure.storage.blob.models.BlobListDetails;

String delimiter = "/";
ListBlobsOptions options = new ListBlobsOptions()
    .setPrefix("data/")
    .setDetails(new BlobListDetails().setRetrieveMetadata(true));

for (BlobItem item : containerClient.listBlobsByHierarchy(delimiter, options, null)) {
    if (item.isPrefix()) {
        System.out.println("Directory: " + item.getName());
    } else {
        System.out.println("Blob: " + item.getName());
    }
}
```

### Xóa Blob

```java
blobClient.delete();

// Xóa nếu tồn tại
blobClient.deleteIfExists();

// Xóa cùng snapshots
import com.azure.storage.blob.models.DeleteSnapshotsOptionType;
blobClient.deleteWithResponse(DeleteSnapshotsOptionType.INCLUDE, null, null, Context.NONE);
```

### Copy Blob

```java
import com.azure.storage.blob.models.BlobCopyInfo;
import com.azure.core.util.polling.SyncPoller;

// Async copy (cho blobs lớn hoặc khác tài khoản)
SyncPoller<BlobCopyInfo, Void> poller = blobClient.beginCopy("<source-blob-url>", Duration.ofSeconds(1));
poller.waitForCompletion();

// Sync copy từ URL (cùng tài khoản)
blobClient.copyFromUrl("<source-blob-url>");
```

### Tạo SAS Token

```java
import com.azure.storage.blob.sas.*;
import java.time.OffsetDateTime;

// Blob-level SAS
BlobSasPermission permissions = new BlobSasPermission().setReadPermission(true);
OffsetDateTime expiry = OffsetDateTime.now().plusDays(1);

BlobServiceSasSignatureValues sasValues = new BlobServiceSasSignatureValues(expiry, permissions);
String sasToken = blobClient.generateSas(sasValues);

// Container-level SAS
BlobContainerSasPermission containerPermissions = new BlobContainerSasPermission()
    .setReadPermission(true)
    .setListPermission(true);

BlobServiceSasSignatureValues containerSasValues = new BlobServiceSasSignatureValues(expiry, containerPermissions);
String containerSas = containerClient.generateSas(containerSasValues);
```

### Thuộc tính Blob và Metadata

```java
import com.azure.storage.blob.models.BlobProperties;

// Get properties
BlobProperties properties = blobClient.getProperties();
System.out.println("Size: " + properties.getBlobSize());
System.out.println("Content-Type: " + properties.getContentType());
System.out.println("Last Modified: " + properties.getLastModified());

// Set metadata
Map<String, String> metadata = Map.of("key1", "value1", "key2", "value2");
blobClient.setMetadata(metadata);

// Set HTTP headers
BlobHttpHeaders headers = new BlobHttpHeaders()
    .setContentType("application/json")
    .setCacheControl("max-age=86400");
blobClient.setHttpHeaders(headers);
```

### Lease Blob

```java
import com.azure.storage.blob.specialized.BlobLeaseClient;
import com.azure.storage.blob.specialized.BlobLeaseClientBuilder;

BlobLeaseClient leaseClient = new BlobLeaseClientBuilder()
    .blobClient(blobClient)
    .buildClient();

// Acquire lease (-1 cho vô hạn)
String leaseId = leaseClient.acquireLease(60);

// Renew lease
leaseClient.renewLease();

// Release lease
leaseClient.releaseLease();
```

## Xử lý Lỗi

```java
import com.azure.storage.blob.models.BlobStorageException;

try {
    blobClient.download(outputStream);
} catch (BlobStorageException e) {
    System.out.println("Status: " + e.getStatusCode());
    System.out.println("Error code: " + e.getErrorCode());
    // 404 = Blob not found
    // 409 = Conflict (lease, etc.)
}
```

## Cấu hình Proxy

```java
import com.azure.core.http.ProxyOptions;
import com.azure.core.http.netty.NettyAsyncHttpClientBuilder;
import java.net.InetSocketAddress;

ProxyOptions proxyOptions = new ProxyOptions(
    ProxyOptions.Type.HTTP,
    new InetSocketAddress("localhost", 8888));

BlobServiceClient client = new BlobServiceClientBuilder()
    .endpoint("<endpoint>")
    .sasToken("<sas-token>")
    .httpClient(new NettyAsyncHttpClientBuilder().proxy(proxyOptions).build())
    .buildClient();
```

## Biến Môi trường

```bash
AZURE_STORAGE_CONNECTION_STRING=DefaultEndpointsProtocol=https;AccountName=...
AZURE_STORAGE_ACCOUNT_URL=https://<account>.blob.core.windows.net
```

## Trigger Phrases

- "Azure Blob Storage Java"
- "upload download blob"
- "blob container SDK"
- "storage streaming"
- "SAS token generation"
- "blob metadata properties"
