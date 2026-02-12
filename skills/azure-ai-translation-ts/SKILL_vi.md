---
name: azure-ai-translation-ts
description: Xây dựng các ứng dụng dịch thuật sử dụng Azure Translation SDKs cho JavaScript (@azure-rest/ai-translation-text, @azure-rest/ai-translation-document). Sử dụng khi triển khai dịch văn bản, chuyển tự, phát hiện ngôn ngữ, hoặc dịch tài liệu hàng loạt.
package: @azure-rest/ai-translation-text, @azure-rest/ai-translation-document
---

# Azure Translation SDKs cho TypeScript

Dịch văn bản và tài liệu với các client kiểu REST.

## Cài đặt

```bash
# Dịch văn bản
npm install @azure-rest/ai-translation-text @azure/identity

# Dịch tài liệu
npm install @azure-rest/ai-translation-document @azure/identity
```

## Biến Môi trường

```bash
TRANSLATOR_ENDPOINT=https://api.cognitive.microsofttranslator.com
TRANSLATOR_SUBSCRIPTION_KEY=<your-api-key>
TRANSLATOR_REGION=<your-region>  # ví dụ: westus, eastus
```

## Client Dịch Văn bản (Text Translation Client)

### Xác thực

```typescript
import TextTranslationClient, {
  TranslatorCredential,
} from "@azure-rest/ai-translation-text";

// API Key + Region
const credential: TranslatorCredential = {
  key: process.env.TRANSLATOR_SUBSCRIPTION_KEY!,
  region: process.env.TRANSLATOR_REGION!,
};
const client = TextTranslationClient(
  process.env.TRANSLATOR_ENDPOINT!,
  credential,
);

// Hoặc chỉ credential (sử dụng endpoint toàn cầu)
const client2 = TextTranslationClient(credential);
```

### Dịch Văn bản

```typescript
import TextTranslationClient, {
  isUnexpected,
} from "@azure-rest/ai-translation-text";

const response = await client.path("/translate").post({
  body: {
    inputs: [
      {
        text: "Hello, how are you?",
        language: "en", // nguồn (tùy chọn, tự động phát hiện)
        targets: [{ language: "es" }, { language: "fr" }],
      },
    ],
  },
});

if (isUnexpected(response)) {
  throw response.body.error;
}

for (const result of response.body.value) {
  for (const translation of result.translations) {
    console.log(`${translation.language}: ${translation.text}`);
  }
}
```

### Dịch với Tùy chọn

```typescript
const response = await client.path("/translate").post({
  body: {
    inputs: [
      {
        text: "Hello world",
        language: "en",
        textType: "Plain", // hoặc "Html"
        targets: [
          {
            language: "de",
            profanityAction: "NoAction", // "Marked" | "Deleted"
            tone: "formal", // Dành riêng cho LLM
          },
        ],
      },
    ],
  },
});
```

### Lấy Ngôn ngữ Hỗ trợ

```typescript
const response = await client.path("/languages").get();

if (isUnexpected(response)) {
  throw response.body.error;
}

// Ngôn ngữ dịch
for (const [code, lang] of Object.entries(response.body.translation || {})) {
  console.log(`${code}: ${lang.name} (${lang.nativeName})`);
}
```

### Chuyển tự (Transliterate)

```typescript
const response = await client.path("/transliterate").post({
  body: { inputs: [{ text: "这是个测试" }] },
  queryParameters: {
    language: "zh-Hans",
    fromScript: "Hans",
    toScript: "Latn",
  },
});

if (!isUnexpected(response)) {
  for (const t of response.body.value) {
    console.log(`${t.script}: ${t.text}`); // Latn: zhè shì gè cè shì
  }
}
```

### Phát hiện Ngôn ngữ

```typescript
const response = await client.path("/detect").post({
  body: { inputs: [{ text: "Bonjour le monde" }] },
});

if (!isUnexpected(response)) {
  for (const result of response.body.value) {
    console.log(`Language: ${result.language}, Score: ${result.score}`);
  }
}
```

## Client Dịch Tài liệu (Document Translation Client)

### Xác thực

```typescript
import DocumentTranslationClient from "@azure-rest/ai-translation-document";
import { DefaultAzureCredential } from "@azure/identity";

const endpoint = "https://<translator>.cognitiveservices.azure.com";

// TokenCredential
const client = DocumentTranslationClient(
  endpoint,
  new DefaultAzureCredential(),
);

// API Key
const client2 = DocumentTranslationClient(endpoint, { key: "<api-key>" });
```

### Dịch Một Tài liệu

```typescript
import DocumentTranslationClient from "@azure-rest/ai-translation-document";
import { writeFile } from "node:fs/promises";

const response = await client
  .path("/document:translate")
  .post({
    queryParameters: {
      targetLanguage: "es",
      sourceLanguage: "en", // tùy chọn
    },
    contentType: "multipart/form-data",
    body: [
      {
        name: "document",
        body: "Hello, this is a test document.",
        filename: "test.txt",
        contentType: "text/plain",
      },
    ],
  })
  .asNodeStream();

if (response.status === "200") {
  await writeFile("translated.txt", response.body);
}
```

### Dịch Tài liệu Hàng loạt

```typescript
import {
  ContainerSASPermissions,
  BlobServiceClient,
} from "@azure/storage-blob";

// Tạo SAS URLs cho container nguồn và đích
const sourceSas = await sourceContainer.generateSasUrl({
  permissions: ContainerSASPermissions.parse("rl"),
  expiresOn: new Date(Date.now() + 24 * 60 * 60 * 1000),
});

const targetSas = await targetContainer.generateSasUrl({
  permissions: ContainerSASPermissions.parse("rwl"),
  expiresOn: new Date(Date.now() + 24 * 60 * 60 * 1000),
});

// Bắt đầu dịch hàng loạt
const response = await client.path("/document/batches").post({
  body: {
    inputs: [
      {
        source: { sourceUrl: sourceSas },
        targets: [{ targetUrl: targetSas, language: "fr" }],
      },
    ],
  },
});

// Lấy ID hoạt động từ header
const operationId = new URL(response.headers["operation-location"]).pathname
  .split("/")
  .pop();
```

### Lấy Trạng thái Dịch

```typescript
import { isUnexpected, paginate } from "@azure-rest/ai-translation-document";

const statusResponse = await client
  .path("/document/batches/{id}", operationId)
  .get();

if (!isUnexpected(statusResponse)) {
  const status = statusResponse.body;
  console.log(`Status: ${status.status}`);
  console.log(`Total: ${status.summary.total}`);
  console.log(`Success: ${status.summary.success}`);
}

// Liệt kê tài liệu với phân trang
const docsResponse = await client
  .path("/document/batches/{id}/documents", operationId)
  .get();
const documents = paginate(client, docsResponse);

for await (const doc of documents) {
  console.log(`${doc.id}: ${doc.status}`);
}
```

### Lấy Định dạng Hỗ trợ

```typescript
const response = await client.path("/document/formats").get();

if (!isUnexpected(response)) {
  for (const format of response.body.value) {
    console.log(`${format.format}: ${format.fileExtensions.join(", ")}`);
  }
}
```

## Các Loại Chính (Key Types)

```typescript
// Text Translation
import type {
  TranslatorCredential,
  TranslatorTokenCredential,
} from "@azure-rest/ai-translation-text";

// Document Translation
import type {
  DocumentTranslateParameters,
  StartTranslationDetails,
  TranslationStatus,
} from "@azure-rest/ai-translation-document";
```

## Thực hành Tốt nhất

1. **Tự động phát hiện nguồn** - Bỏ qua tham số `language` để tự động phát hiện
2. **Yêu cầu hàng loạt** - Dịch nhiều văn bản trong một lần gọi để hiệu quả
3. **Sử dụng SAS tokens** - Cho dịch tài liệu, sử dụng SAS URLs giới hạn thời gian
4. **Xử lý lỗi** - Luôn kiểm tra `isUnexpected(response)` trước khi truy cập body
5. **Endpoint khu vực** - Sử dụng endpoint khu vực để có độ trễ thấp hơn
