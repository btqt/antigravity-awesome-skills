---
name: azure-storage-blob-ts
description: |
  Azure Blob Storage JavaScript/TypeScript SDK (@azure/storage-blob) cho các hoạt động blob. Sử dụng để upload, download, liệt kê, và quản lý blobs và containers. Hỗ trợ block blobs, append blobs, page blobs, SAS tokens, và streaming. Triggers: "blob storage", "@azure/storage-blob", "BlobServiceClient", "ContainerClient", "upload blob", "download blob", "SAS token", "block blob".
package: @azure/storage-blob
---

# @azure/storage-blob (TypeScript/JavaScript)

SDK cho các hoạt động Azure Blob Storage — upload, download, liệt kê, và quản lý blobs và containers.

## Cài đặt

```bash
npm install @azure/storage-blob @azure/identity
```

**Phiên bản Hiện tại**: 12.x
**Node.js**: >= 18.0.0

## Biến Môi trường

```bash
AZURE_STORAGE_ACCOUNT_NAME=<account-name>
AZURE_STORAGE_ACCOUNT_KEY=<account-key>
# HOẶC chuỗi kết nối
AZURE_STORAGE_CONNECTION_STRING=DefaultEndpointsProtocol=https;AccountName=...
```

## Xác thực

### DefaultAzureCredential (Khuyên dùng)

```typescript
import { BlobServiceClient } from "@azure/storage-blob";
import { DefaultAzureCredential } from "@azure/identity";

const accountName = process.env.AZURE_STORAGE_ACCOUNT_NAME!;
const client = new BlobServiceClient(
  `https://${accountName}.blob.core.windows.net`,
  new DefaultAzureCredential(),
);
```

### Chuỗi Kết nối

```typescript
import { BlobServiceClient } from "@azure/storage-blob";

const client = BlobServiceClient.fromConnectionString(
  process.env.AZURE_STORAGE_CONNECTION_STRING!,
);
```

### StorageSharedKeyCredential (Chỉ Node.js)

```typescript
import {
  BlobServiceClient,
  StorageSharedKeyCredential,
} from "@azure/storage-blob";

const accountName = process.env.AZURE_STORAGE_ACCOUNT_NAME!;
const accountKey = process.env.AZURE_STORAGE_ACCOUNT_KEY!;

const sharedKeyCredential = new StorageSharedKeyCredential(
  accountName,
  accountKey,
);
const client = new BlobServiceClient(
  `https://${accountName}.blob.core.windows.net`,
  sharedKeyCredential,
);
```

### SAS Token

```typescript
import { BlobServiceClient } from "@azure/storage-blob";

const accountName = process.env.AZURE_STORAGE_ACCOUNT_NAME!;
const sasToken = process.env.AZURE_STORAGE_SAS_TOKEN!; // bắt đầu bằng "?"

const client = new BlobServiceClient(
  `https://${accountName}.blob.core.windows.net${sasToken}`,
);
```

## Phân cấp Client

```
BlobServiceClient (cấp tài khoản)
└── ContainerClient (cấp container)
    └── BlobClient (cấp blob)
        ├── BlockBlobClient (block blobs - phổ biến nhất)
        ├── AppendBlobClient (append-only blobs)
        └── PageBlobClient (page blobs - VHDs)
```

## Các Hoạt động Container

### Tạo Container

```typescript
const containerClient = client.getContainerClient("my-container");
await containerClient.create();

// Hoặc tạo nếu không tồn tại
await containerClient.createIfNotExists();
```

### Liệt kê Containers

```typescript
for await (const container of client.listContainers()) {
  console.log(container.name);
}

// Với bộ lọc prefix
for await (const container of client.listContainers({ prefix: "logs-" })) {
  console.log(container.name);
}
```

### Xóa Container

```typescript
await containerClient.delete();
// Hoặc xóa nếu tồn tại
await containerClient.deleteIfExists();
```

## Các Hoạt động Blob

### Tải lên Blob (Đơn giản)

```typescript
const containerClient = client.getContainerClient("my-container");
const blockBlobClient = containerClient.getBlockBlobClient("my-file.txt");

// Upload string
await blockBlobClient.upload("Hello, World!", 13);

// Upload Buffer
const buffer = Buffer.from("Hello, World!");
await blockBlobClient.upload(buffer, buffer.length);
```

### Tải lên từ File (Chỉ Node.js)

```typescript
const blockBlobClient = containerClient.getBlockBlobClient("uploaded-file.txt");
await blockBlobClient.uploadFile("/path/to/local/file.txt");
```

### Tải lên từ Stream (Chỉ Node.js)

```typescript
import * as fs from "fs";

const blockBlobClient = containerClient.getBlockBlobClient("streamed-file.txt");
const readStream = fs.createReadStream("/path/to/local/file.txt");

await blockBlobClient.uploadStream(readStream, 4 * 1024 * 1024, 5, {
  // bufferSize: 4MB, maxConcurrency: 5
  onProgress: (progress) =>
    console.log(`Uploaded ${progress.loadedBytes} bytes`),
});
```

### Tải lên từ Trình duyệt

```typescript
const blockBlobClient =
  containerClient.getBlockBlobClient("browser-upload.txt");

// Từ File input
const fileInput = document.getElementById("fileInput") as HTMLInputElement;
const file = fileInput.files![0];
await blockBlobClient.uploadData(file);

// Từ Blob/ArrayBuffer
const arrayBuffer = new ArrayBuffer(1024);
await blockBlobClient.uploadData(arrayBuffer);
```

### Tải xuống Blob

```typescript
const blobClient = containerClient.getBlobClient("my-file.txt");
const downloadResponse = await blobClient.download();

// Đọc dưới dạng string (browser & Node.js)
const downloaded = await streamToText(downloadResponse.readableStreamBody!);

async function streamToText(readable: NodeJS.ReadableStream): Promise<string> {
  const chunks: Buffer[] = [];
  for await (const chunk of readable) {
    chunks.push(Buffer.from(chunk));
  }
  return Buffer.concat(chunks).toString("utf-8");
}
```

### Tải xuống vào File (Chỉ Node.js)

```typescript
const blockBlobClient = containerClient.getBlockBlobClient("my-file.txt");
await blockBlobClient.downloadToFile("/path/to/local/destination.txt");
```

### Tải xuống vào Buffer (Chỉ Node.js)

```typescript
const blockBlobClient = containerClient.getBlockBlobClient("my-file.txt");
const buffer = await blockBlobClient.downloadToBuffer();
console.log(buffer.toString());
```

### Liệt kê Blobs

```typescript
// Liệt kê tất cả blobs
for await (const blob of containerClient.listBlobsFlat()) {
  console.log(blob.name, blob.properties.contentLength);
}

// Liệt kê với prefix
for await (const blob of containerClient.listBlobsFlat({ prefix: "logs/" })) {
  console.log(blob.name);
}

// Liệt kê theo phân cấp (thư mục ảo)
for await (const item of containerClient.listBlobsByHierarchy("/")) {
  if (item.kind === "prefix") {
    console.log(`Directory: ${item.name}`);
  } else {
    console.log(`Blob: ${item.name}`);
  }
}
```

### Xóa Blob

```typescript
const blobClient = containerClient.getBlobClient("my-file.txt");
await blobClient.delete();

// Xóa nếu tồn tại
await blobClient.deleteIfExists();

// Xóa cùng snapshots
await blobClient.delete({ deleteSnapshots: "include" });
```

### Copy Blob

```typescript
const sourceBlobClient = containerClient.getBlobClient("source.txt");
const destBlobClient = containerClient.getBlobClient("destination.txt");

// Bắt đầu copy
const copyPoller = await destBlobClient.beginCopyFromURL(sourceBlobClient.url);
await copyPoller.pollUntilDone();
```

## Thuộc tính Blob & Metadata

### Lấy Thuộc tính

```typescript
const blobClient = containerClient.getBlobClient("my-file.txt");
const properties = await blobClient.getProperties();

console.log("Content-Type:", properties.contentType);
console.log("Content-Length:", properties.contentLength);
console.log("Last Modified:", properties.lastModified);
console.log("ETag:", properties.etag);
```

### Đặt Metadata

```typescript
await blobClient.setMetadata({
  author: "John Doe",
  category: "documents",
});
```

### Đặt HTTP Headers

```typescript
await blobClient.setHTTPHeaders({
  blobContentType: "text/plain",
  blobCacheControl: "max-age=3600",
  blobContentDisposition: "attachment; filename=download.txt",
});
```

## Tạo SAS Token (Chỉ Node.js)

### Tạo Blob SAS

```typescript
import {
  BlobSASPermissions,
  generateBlobSASQueryParameters,
  StorageSharedKeyCredential,
} from "@azure/storage-blob";

const sharedKeyCredential = new StorageSharedKeyCredential(
  accountName,
  accountKey,
);

const sasToken = generateBlobSASQueryParameters(
  {
    containerName: "my-container",
    blobName: "my-file.txt",
    permissions: BlobSASPermissions.parse("r"), // chỉ đọc
    startsOn: new Date(),
    expiresOn: new Date(Date.now() + 3600 * 1000), // 1 giờ
  },
  sharedKeyCredential,
).toString();

const sasUrl = `https://${accountName}.blob.core.windows.net/my-container/my-file.txt?${sasToken}`;
```

### Tạo Container SAS

```typescript
import {
  ContainerSASPermissions,
  generateBlobSASQueryParameters,
} from "@azure/storage-blob";

const sasToken = generateBlobSASQueryParameters(
  {
    containerName: "my-container",
    permissions: ContainerSASPermissions.parse("racwdl"), // read, add, create, write, delete, list
    expiresOn: new Date(Date.now() + 24 * 3600 * 1000), // 24 giờ
  },
  sharedKeyCredential,
).toString();
```

### Tạo Account SAS

```typescript
import {
  AccountSASPermissions,
  AccountSASResourceTypes,
  AccountSASServices,
  generateAccountSASQueryParameters,
} from "@azure/storage-blob";

const sasToken = generateAccountSASQueryParameters(
  {
    services: AccountSASServices.parse("b").toString(), // blob
    resourceTypes: AccountSASResourceTypes.parse("sco").toString(), // service, container, object
    permissions: AccountSASPermissions.parse("rwdlacupi"), // tất cả quyền
    expiresOn: new Date(Date.now() + 24 * 3600 * 1000),
  },
  sharedKeyCredential,
).toString();
```

## Các Loại Blob

### Block Blob (Mặc định)

Loại phổ biến nhất cho văn bản và tệp nhị phân.

```typescript
const blockBlobClient = containerClient.getBlockBlobClient("document.pdf");
await blockBlobClient.uploadFile("/path/to/document.pdf");
```

### Append Blob

Được tối ưu hóa cho các thao tác chèn thêm (logs, audit trails).

```typescript
const appendBlobClient = containerClient.getAppendBlobClient("app.log");

// Tạo append blob
await appendBlobClient.create();

// Thêm dữ liệu
await appendBlobClient.appendBlock("Log entry 1\n", 12);
await appendBlobClient.appendBlock("Log entry 2\n", 12);
```

### Page Blob

Blobs kích thước cố định cho đọc/ghi ngẫu nhiên (VHDs).

```typescript
const pageBlobClient = containerClient.getPageBlobClient("disk.vhd");

// Tạo page blob căn chỉnh 512-byte
await pageBlobClient.create(1024 * 1024); // 1MB

// Ghi trang (phải căn chỉnh 512-byte)
const buffer = Buffer.alloc(512);
await pageBlobClient.uploadPages(buffer, 0, 512);
```

## Xử lý Lỗi

```typescript
import { RestError } from "@azure/storage-blob";

try {
  await containerClient.create();
} catch (error) {
  if (error instanceof RestError) {
    switch (error.statusCode) {
      case 404:
        console.log("Container not found");
        break;
      case 409:
        console.log("Container already exists");
        break;
      case 403:
        console.log("Access denied");
        break;
      default:
        console.error(`Storage error ${error.statusCode}: ${error.message}`);
    }
  }
  throw error;
}
```

## Tham chiếu Types trong TypeScript

```typescript
import {
  // Clients
  BlobServiceClient,
  ContainerClient,
  BlobClient,
  BlockBlobClient,
  AppendBlobClient,
  PageBlobClient,

  // Authentication
  StorageSharedKeyCredential,
  AnonymousCredential,

  // SAS
  BlobSASPermissions,
  ContainerSASPermissions,
  AccountSASPermissions,
  AccountSASServices,
  AccountSASResourceTypes,
  generateBlobSASQueryParameters,
  generateAccountSASQueryParameters,

  // Options & Responses
  BlobDownloadResponseParsed,
  BlobUploadCommonResponse,
  ContainerCreateResponse,
  BlobItem,
  ContainerItem,

  // Errors
  RestError,
} from "@azure/storage-blob";
```

## Thực hành Tốt nhất

1. **Sử dụng DefaultAzureCredential** — Ưu tiên AAD hơn chuỗi kết nối/keys
2. **Sử dụng streaming cho files lớn** — `uploadStream`/`downloadToFile` cho files > 256MB
3. **Đặt content types phù hợp** — Sử dụng `setHTTPHeaders` cho đúng MIME types
4. **Sử dụng SAS tokens cho truy cập client** — Tạo tokens ngắn hạn cho browser uploads
5. **Xử lý lỗi khéo léo** — Kiểm tra `RestError.statusCode` để xử lý cụ thể
6. **Sử dụng các phương thức `*IfNotExists`** — Cho việc tạo container/blob idempotent
7. **Đóng clients** — Không bắt buộc nhưng là thực hành tốt trong các ứng dụng chạy lâu dài

## Khác biệt Nền tảng

| Tính năng                    | Node.js | Browser |
| ---------------------------- | ------- | ------- |
| `StorageSharedKeyCredential` | ✅      | ❌      |
| `uploadFile()`               | ✅      | ❌      |
| `uploadStream()`             | ✅      | ❌      |
| `downloadToFile()`           | ✅      | ❌      |
| `downloadToBuffer()`         | ✅      | ❌      |
| `uploadData()`               | ✅      | ✅      |
| SAS generation               | ✅      | ❌      |
| DefaultAzureCredential       | ✅      | ❌      |
| Anonymous/SAS access         | ✅      | ✅      |
