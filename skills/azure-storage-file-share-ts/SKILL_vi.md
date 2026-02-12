---
name: azure-storage-file-share-ts
description: |
  Azure File Share JavaScript/TypeScript SDK (@azure/storage-file-share) cho các hoạt động SMB file share. Sử dụng để tạo shares, quản lý thư mục, upload/download files, và xử lý metadata của file. Hỗ trợ các kịch bản giao thức Azure Files SMB. Triggers: "file share", "@azure/storage-file-share", "ShareServiceClient", "ShareClient", "SMB", "Azure Files".
package: @azure/storage-file-share
---

# @azure/storage-file-share (TypeScript/JavaScript)

SDK cho các hoạt động Azure File Share — SMB file shares, thư mục, và các hoạt động tệp.

## Cài đặt

```bash
npm install @azure/storage-file-share @azure/identity
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

### Chuỗi Kết nối (Đơn giản nhất)

```typescript
import { ShareServiceClient } from "@azure/storage-file-share";

const client = ShareServiceClient.fromConnectionString(
  process.env.AZURE_STORAGE_CONNECTION_STRING!,
);
```

### StorageSharedKeyCredential (Chỉ Node.js)

```typescript
import {
  ShareServiceClient,
  StorageSharedKeyCredential,
} from "@azure/storage-file-share";

const accountName = process.env.AZURE_STORAGE_ACCOUNT_NAME!;
const accountKey = process.env.AZURE_STORAGE_ACCOUNT_KEY!;

const sharedKeyCredential = new StorageSharedKeyCredential(
  accountName,
  accountKey,
);
const client = new ShareServiceClient(
  `https://${accountName}.file.core.windows.net`,
  sharedKeyCredential,
);
```

### DefaultAzureCredential

```typescript
import { ShareServiceClient } from "@azure/storage-file-share";
import { DefaultAzureCredential } from "@azure/identity";

const accountName = process.env.AZURE_STORAGE_ACCOUNT_NAME!;
const client = new ShareServiceClient(
  `https://${accountName}.file.core.windows.net`,
  new DefaultAzureCredential(),
);
```

### SAS Token

```typescript
import { ShareServiceClient } from "@azure/storage-file-share";

const accountName = process.env.AZURE_STORAGE_ACCOUNT_NAME!;
const sasToken = process.env.AZURE_STORAGE_SAS_TOKEN!;

const client = new ShareServiceClient(
  `https://${accountName}.file.core.windows.net${sasToken}`,
);
```

## Phân cấp Client

```
ShareServiceClient (cấp tài khoản)
└── ShareClient (cấp share)
    └── ShareDirectoryClient (cấp directory)
        └── ShareFileClient (cấp file)
```

## Các Hoạt động Share

### Tạo Share

```typescript
const shareClient = client.getShareClient("my-share");
await shareClient.create();

// Tạo với hạn ngạch (tính bằng GB)
await shareClient.create({ quota: 100 });
```

### Liệt kê Shares

```typescript
for await (const share of client.listShares()) {
  console.log(share.name, share.properties.quota);
}

// Với bộ lọc prefix
for await (const share of client.listShares({ prefix: "logs-" })) {
  console.log(share.name);
}
```

### Xóa Share

```typescript
await shareClient.delete();

// Xóa nếu tồn tại
await shareClient.deleteIfExists();
```

### Lấy Thuộc tính Share

```typescript
const properties = await shareClient.getProperties();
console.log("Quota:", properties.quota, "GB");
console.log("Last Modified:", properties.lastModified);
```

### Đặt Hạn ngạch Share

```typescript
await shareClient.setQuota(200); // 200 GB
```

## Các Hoạt động Thư mục

### Tạo Thư mục

```typescript
const directoryClient = shareClient.getDirectoryClient("my-directory");
await directoryClient.create();

// Tạo thư mục lồng nhau
const nestedDir = shareClient.getDirectoryClient("parent/child/grandchild");
await nestedDir.create();
```

### Liệt kê Thư mục và Tệp

```typescript
const directoryClient = shareClient.getDirectoryClient("my-directory");

for await (const item of directoryClient.listFilesAndDirectories()) {
  if (item.kind === "directory") {
    console.log(`[DIR] ${item.name}`);
  } else {
    console.log(`[FILE] ${item.name} (${item.properties.contentLength} bytes)`);
  }
}
```

### Xóa Thư mục

```typescript
await directoryClient.delete();

// Xóa nếu tồn tại
await directoryClient.deleteIfExists();
```

### Kiểm tra xem Thư mục có tồn tại không

```typescript
const exists = await directoryClient.exists();
if (!exists) {
  await directoryClient.create();
}
```

## Các Hoạt động Tệp

### Tải lên Tệp (Đơn giản)

```typescript
const fileClient = shareClient
  .getDirectoryClient("my-directory")
  .getFileClient("my-file.txt");

// Upload string
const content = "Hello, World!";
await fileClient.create(content.length);
await fileClient.uploadRange(content, 0, content.length);
```

### Tải lên Tệp (Node.js - từ file cục bộ)

```typescript
import * as fs from "fs";
import * as path from "path";

const fileClient =
  shareClient.rootDirectoryClient.getFileClient("uploaded.txt");
const localFilePath = "/path/to/local/file.txt";
const fileSize = fs.statSync(localFilePath).size;

await fileClient.create(fileSize);
await fileClient.uploadFile(localFilePath);
```

### Tải lên Tệp (Buffer)

```typescript
const buffer = Buffer.from("Hello, Azure Files!");
const fileClient =
  shareClient.rootDirectoryClient.getFileClient("buffer-file.txt");

await fileClient.create(buffer.length);
await fileClient.uploadRange(buffer, 0, buffer.length);
```

### Tải lên Tệp (Stream)

```typescript
import * as fs from "fs";

const fileClient =
  shareClient.rootDirectoryClient.getFileClient("streamed.txt");
const readStream = fs.createReadStream("/path/to/local/file.txt");
const fileSize = fs.statSync("/path/to/local/file.txt").size;

await fileClient.create(fileSize);
await fileClient.uploadStream(readStream, fileSize, 4 * 1024 * 1024, 4); // 4MB buffer, 4 concurrency
```

### Tải xuống Tệp

```typescript
const fileClient = shareClient
  .getDirectoryClient("my-directory")
  .getFileClient("my-file.txt");

const downloadResponse = await fileClient.download();

// Đọc dưới dạng string
const chunks: Buffer[] = [];
for await (const chunk of downloadResponse.readableStreamBody!) {
  chunks.push(Buffer.from(chunk));
}
const content = Buffer.concat(chunks).toString("utf-8");
```

### Tải xuống vào File (Node.js)

```typescript
const fileClient = shareClient.rootDirectoryClient.getFileClient("my-file.txt");
await fileClient.downloadToFile("/path/to/local/destination.txt");
```

### Tải xuống vào Buffer (Node.js)

```typescript
const fileClient = shareClient.rootDirectoryClient.getFileClient("my-file.txt");
const buffer = await fileClient.downloadToBuffer();
console.log(buffer.toString());
```

### Xóa Tệp

```typescript
const fileClient = shareClient.rootDirectoryClient.getFileClient("my-file.txt");
await fileClient.delete();

// Xóa nếu tồn tại
await fileClient.deleteIfExists();
```

### Copy Tệp

```typescript
const sourceUrl = "https://account.file.core.windows.net/share/source.txt";
const destFileClient =
  shareClient.rootDirectoryClient.getFileClient("destination.txt");

// Bắt đầu copy
const copyPoller = await destFileClient.startCopyFromURL(sourceUrl);
await copyPoller.pollUntilDone();
```

## Thuộc tính Tệp & Metadata

### Lấy Thuộc tính Tệp

```typescript
const fileClient = shareClient.rootDirectoryClient.getFileClient("my-file.txt");
const properties = await fileClient.getProperties();

console.log("Content-Length:", properties.contentLength);
console.log("Content-Type:", properties.contentType);
console.log("Last Modified:", properties.lastModified);
console.log("ETag:", properties.etag);
```

### Đặt Metadata

```typescript
await fileClient.setMetadata({
  author: "John Doe",
  category: "documents",
});
```

### Đặt HTTP Headers

```typescript
await fileClient.setHttpHeaders({
  fileContentType: "text/plain",
  fileCacheControl: "max-age=3600",
  fileContentDisposition: "attachment; filename=download.txt",
});
```

## Các Hoạt động theo Phạm vi (Range Operations)

### Upload Range

```typescript
const data = Buffer.from("partial content");
await fileClient.uploadRange(data, 100, data.length); // Ghi tại offset 100
```

### Download Range

```typescript
const downloadResponse = await fileClient.download(100, 50); // offset 100, length 50
```

### Clear Range

```typescript
await fileClient.clearRange(0, 100); // Xóa 100 bytes đầu tiên
```

## Các Hoạt động Snapshot

### Tạo Snapshot

```typescript
const snapshotResponse = await shareClient.createSnapshot();
console.log("Snapshot:", snapshotResponse.snapshot);
```

### Truy cập Snapshot

```typescript
const snapshotShareClient = shareClient.withSnapshot(
  snapshotResponse.snapshot!,
);
const snapshotFileClient =
  snapshotShareClient.rootDirectoryClient.getFileClient("file.txt");
const content = await snapshotFileClient.downloadToBuffer();
```

### Xóa Snapshot

```typescript
await shareClient.delete({ deleteSnapshots: "include" });
```

## Tạo SAS Token (Chỉ Node.js)

### Tạo File SAS

```typescript
import {
  generateFileSASQueryParameters,
  FileSASPermissions,
  StorageSharedKeyCredential,
} from "@azure/storage-file-share";

const sharedKeyCredential = new StorageSharedKeyCredential(
  accountName,
  accountKey,
);

const sasToken = generateFileSASQueryParameters(
  {
    shareName: "my-share",
    filePath: "my-directory/my-file.txt",
    permissions: FileSASPermissions.parse("r"), // chỉ đọc
    expiresOn: new Date(Date.now() + 3600 * 1000), // 1 giờ
  },
  sharedKeyCredential,
).toString();

const sasUrl = `https://${accountName}.file.core.windows.net/my-share/my-directory/my-file.txt?${sasToken}`;
```

### Tạo Share SAS

```typescript
import {
  ShareSASPermissions,
  generateFileSASQueryParameters,
} from "@azure/storage-file-share";

const sasToken = generateFileSASQueryParameters(
  {
    shareName: "my-share",
    permissions: ShareSASPermissions.parse("rcwdl"), // read, create, write, delete, list
    expiresOn: new Date(Date.now() + 24 * 3600 * 1000), // 24 giờ
  },
  sharedKeyCredential,
).toString();
```

## Xử lý Lỗi

```typescript
import { RestError } from "@azure/storage-file-share";

try {
  await shareClient.create();
} catch (error) {
  if (error instanceof RestError) {
    switch (error.statusCode) {
      case 404:
        console.log("Share not found");
        break;
      case 409:
        console.log("Share already exists");
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
  ShareServiceClient,
  ShareClient,
  ShareDirectoryClient,
  ShareFileClient,

  // Authentication
  StorageSharedKeyCredential,
  AnonymousCredential,

  // SAS
  FileSASPermissions,
  ShareSASPermissions,
  AccountSASPermissions,
  AccountSASServices,
  AccountSASResourceTypes,
  generateFileSASQueryParameters,
  generateAccountSASQueryParameters,

  // Options & Responses
  ShareCreateResponse,
  FileDownloadResponseModel,
  DirectoryItem,
  FileItem,
  ShareProperties,
  FileProperties,

  // Errors
  RestError,
} from "@azure/storage-file-share";
```

## Thực hành Tốt nhất

1. **Sử dụng chuỗi kết nối cho sự đơn giản** — Thiết lập dễ nhất cho phát triển
2. **Sử dụng DefaultAzureCredential cho production** — Bật managed identity trong Azure
3. **Đặt hạn ngạch trên shares** — Ngăn chi phí lưu trữ bất ngờ
4. **Sử dụng streaming cho files lớn** — `uploadStream`/`downloadToFile` cho files > 256MB
5. **Sử dụng ranges cho cập nhật một phần** — Hiệu quả hơn so với thay thế toàn bộ file
6. **Tạo snapshots trước những thay đổi lớn** — Phục hồi điểm thời gian (Point-in-time recovery)
7. **Xử lý lỗi khéo léo** — Kiểm tra `RestError.statusCode` để xử lý cụ thể
8. **Sử dụng các phương thức `*IfExists`** — Cho các hoạt động idempotent

## Khác biệt Nền tảng

| Tính năng                    | Node.js | Browser |
| ---------------------------- | ------- | ------- |
| `StorageSharedKeyCredential` | ✅      | ❌      |
| `uploadFile()`               | ✅      | ❌      |
| `uploadStream()`             | ✅      | ❌      |
| `downloadToFile()`           | ✅      | ❌      |
| `downloadToBuffer()`         | ✅      | ❌      |
| SAS generation               | ✅      | ❌      |
| DefaultAzureCredential       | ✅      | ❌      |
| Anonymous/SAS access         | ✅      | ✅      |
