---
name: azure-ai-document-intelligence-ts
description: Trích xuất văn bản, bảng và dữ liệu có cấu trúc từ tài liệu sử dụng Azure Document Intelligence (@azure-rest/ai-document-intelligence). Sử dụng khi xử lý hóa đơn, biên lai, ID, biểu mẫu hoặc xây dựng các mô hình tài liệu tùy chỉnh.
package: @azure-rest/ai-document-intelligence
---

# Azure Document Intelligence REST SDK cho TypeScript

Trích xuất văn bản, bảng và dữ liệu có cấu trúc từ tài liệu sử dụng các mô hình xây dựng sẵn và tùy chỉnh.

## Cài đặt

```bash
npm install @azure-rest/ai-document-intelligence @azure/identity
```

## Biến Môi trường

```bash
DOCUMENT_INTELLIGENCE_ENDPOINT=https://<resource>.cognitiveservices.azure.com
DOCUMENT_INTELLIGENCE_API_KEY=<api-key>
```

## Xác thực

**Quan trọng**: Đây là một REST client. `DocumentIntelligence` là một **hàm**, không phải là một lớp (class).

### DefaultAzureCredential

```typescript
import DocumentIntelligence from "@azure-rest/ai-document-intelligence";
import { DefaultAzureCredential } from "@azure/identity";

const client = DocumentIntelligence(
  process.env.DOCUMENT_INTELLIGENCE_ENDPOINT!,
  new DefaultAzureCredential(),
);
```

### API Key

```typescript
import DocumentIntelligence from "@azure-rest/ai-document-intelligence";

const client = DocumentIntelligence(
  process.env.DOCUMENT_INTELLIGENCE_ENDPOINT!,
  { key: process.env.DOCUMENT_INTELLIGENCE_API_KEY! },
);
```

## Phân tích Tài liệu (URL)

```typescript
import DocumentIntelligence, {
  isUnexpected,
  getLongRunningPoller,
  AnalyzeOperationOutput,
} from "@azure-rest/ai-document-intelligence";

const initialResponse = await client
  .path("/documentModels/{modelId}:analyze", "prebuilt-layout")
  .post({
    contentType: "application/json",
    body: {
      urlSource: "https://example.com/document.pdf",
    },
    queryParameters: { locale: "en-US" },
  });

if (isUnexpected(initialResponse)) {
  throw initialResponse.body.error;
}

const poller = getLongRunningPoller(client, initialResponse);
const result = (await poller.pollUntilDone()).body as AnalyzeOperationOutput;

console.log("Trang:", result.analyzeResult?.pages?.length);
console.log("Bảng:", result.analyzeResult?.tables?.length);
```

## Phân tích Tài liệu (Tệp Cục bộ)

```typescript
import { readFile } from "node:fs/promises";

const fileBuffer = await readFile("./document.pdf");
const base64Source = fileBuffer.toString("base64");

const initialResponse = await client
  .path("/documentModels/{modelId}:analyze", "prebuilt-invoice")
  .post({
    contentType: "application/json",
    body: { base64Source },
  });

if (isUnexpected(initialResponse)) {
  throw initialResponse.body.error;
}

const poller = getLongRunningPoller(client, initialResponse);
const result = (await poller.pollUntilDone()).body as AnalyzeOperationOutput;
```

## Các Mô hình Xây dựng sẵn

| ID Mô hình                        | Mô tả                                |
| --------------------------------- | ------------------------------------ |
| `prebuilt-read`                   | OCR - trích xuất văn bản và ngôn ngữ |
| `prebuilt-layout`                 | Văn bản, bảng, dấu chọn, cấu trúc    |
| `prebuilt-invoice`                | Các trường hóa đơn                   |
| `prebuilt-receipt`                | Các trường biên lai                  |
| `prebuilt-idDocument`             | Các trường tài liệu ID               |
| `prebuilt-tax.us.w2`              | Các trường biểu mẫu thuế W-2         |
| `prebuilt-healthInsuranceCard.us` | Các trường thẻ bảo hiểm y tế         |
| `prebuilt-contract`               | Các trường hợp đồng                  |
| `prebuilt-bankStatement.us`       | Các trường sao kê ngân hàng          |

## Trích xuất Các trường Hóa đơn

```typescript
const initialResponse = await client
  .path("/documentModels/{modelId}:analyze", "prebuilt-invoice")
  .post({
    contentType: "application/json",
    body: { urlSource: invoiceUrl },
  });

if (isUnexpected(initialResponse)) {
  throw initialResponse.body.error;
}

const poller = getLongRunningPoller(client, initialResponse);
const result = (await poller.pollUntilDone()).body as AnalyzeOperationOutput;

const invoice = result.analyzeResult?.documents?.[0];
if (invoice) {
  console.log("Nhà cung cấp:", invoice.fields?.VendorName?.content);
  console.log("Tổng cộng:", invoice.fields?.InvoiceTotal?.content);
  console.log("Hạn thanh toán:", invoice.fields?.DueDate?.content);
}
```

## Trích xuất Các trường Biên lai

```typescript
const initialResponse = await client
  .path("/documentModels/{modelId}:analyze", "prebuilt-receipt")
  .post({
    contentType: "application/json",
    body: { urlSource: receiptUrl },
  });

const poller = getLongRunningPoller(client, initialResponse);
const result = (await poller.pollUntilDone()).body as AnalyzeOperationOutput;

const receipt = result.analyzeResult?.documents?.[0];
if (receipt) {
  console.log("Người bán:", receipt.fields?.MerchantName?.content);
  console.log("Tổng cộng:", receipt.fields?.Total?.content);

  for (const item of receipt.fields?.Items?.values || []) {
    console.log("Mặt hàng:", item.properties?.Description?.content);
    console.log("Giá:", item.properties?.TotalPrice?.content);
  }
}
```

## Liệt kê Các Mô hình Tài liệu

```typescript
import DocumentIntelligence, {
  isUnexpected,
  paginate,
} from "@azure-rest/ai-document-intelligence";

const response = await client.path("/documentModels").get();

if (isUnexpected(response)) {
  throw response.body.error;
}

for await (const model of paginate(client, response)) {
  console.log(model.modelId);
}
```

## Xây dựng Mô hình Tùy chỉnh

```typescript
const initialResponse = await client.path("/documentModels:build").post({
  body: {
    modelId: "my-custom-model",
    description: "Mô hình tùy chỉnh cho đơn đặt hàng",
    buildMode: "template", // hoặc "neural"
    azureBlobSource: {
      containerUrl: process.env.TRAINING_CONTAINER_SAS_URL!,
      prefix: "training-data/",
    },
  },
});

if (isUnexpected(initialResponse)) {
  throw initialResponse.body.error;
}

const poller = getLongRunningPoller(client, initialResponse);
const result = await poller.pollUntilDone();
console.log("Mô hình đã xây dựng:", result.body);
```

## Xây dựng Bộ phân loại Tài liệu

```typescript
import { DocumentClassifierBuildOperationDetailsOutput } from "@azure-rest/ai-document-intelligence";

const containerSasUrl = process.env.TRAINING_CONTAINER_SAS_URL!;

const initialResponse = await client.path("/documentClassifiers:build").post({
  body: {
    classifierId: "my-classifier",
    description: "Bộ phân loại Hóa đơn vs Biên lai",
    docTypes: {
      invoices: {
        azureBlobSource: { containerUrl: containerSasUrl, prefix: "invoices/" },
      },
      receipts: {
        azureBlobSource: { containerUrl: containerSasUrl, prefix: "receipts/" },
      },
    },
  },
});

if (isUnexpected(initialResponse)) {
  throw initialResponse.body.error;
}

const poller = getLongRunningPoller(client, initialResponse);
const result = (await poller.pollUntilDone())
  .body as DocumentClassifierBuildOperationDetailsOutput;
console.log("Bộ phân loại:", result.result?.classifierId);
```

## Phân loại Tài liệu

```typescript
const initialResponse = await client
  .path("/documentClassifiers/{classifierId}:analyze", "my-classifier")
  .post({
    contentType: "application/json",
    body: { urlSource: documentUrl },
    queryParameters: { split: "auto" },
  });

if (isUnexpected(initialResponse)) {
  throw initialResponse.body.error;
}

const poller = getLongRunningPoller(client, initialResponse);
const result = await poller.pollUntilDone();
console.log("Phân loại:", result.body.analyzeResult?.documents);
```

## Lấy Thông tin Dịch vụ

```typescript
const response = await client.path("/info").get();

if (isUnexpected(response)) {
  throw response.body.error;
}

console.log(
  "Giới hạn mô hình tùy chỉnh:",
  response.body.customDocumentModels.limit,
);
console.log(
  "Số lượng mô hình tùy chỉnh:",
  response.body.customDocumentModels.count,
);
```

## Mẫu Polling

```typescript
import DocumentIntelligence, {
  isUnexpected,
  getLongRunningPoller,
  AnalyzeOperationOutput,
} from "@azure-rest/ai-document-intelligence";

// 1. Bắt đầu hoạt động
const initialResponse = await client
  .path("/documentModels/{modelId}:analyze", "prebuilt-layout")
  .post({ contentType: "application/json", body: { urlSource } });

// 2. Kiểm tra lỗi
if (isUnexpected(initialResponse)) {
  throw initialResponse.body.error;
}

// 3. Tạo poller
const poller = getLongRunningPoller(client, initialResponse);

// 4. Tùy chọn: Theo dõi tiến độ
poller.onProgress((state) => {
  console.log("Trạng thái:", state.status);
});

// 5. Đợi hoàn thành
const result = (await poller.pollUntilDone()).body as AnalyzeOperationOutput;
```

## Các Types Chính

```typescript
import DocumentIntelligence, {
  isUnexpected,
  getLongRunningPoller,
  paginate,
  parseResultIdFromResponse,
  AnalyzeOperationOutput,
  DocumentClassifierBuildOperationDetailsOutput,
} from "@azure-rest/ai-document-intelligence";
```

## Thực hành Tốt nhất

1. **Sử dụng getLongRunningPoller()** - Phân tích tài liệu là bất đồng bộ, luôn poll để lấy kết quả
2. **Kiểm tra isUnexpected()** - Type guard để xử lý lỗi đúng cách
3. **Chọn đúng mô hình** - Sử dụng mô hình xây dựng sẵn khi có thể, tùy chỉnh cho tài liệu chuyên biệt
4. **Xử lý điểm tin cậy** - Các trường có giá trị độ tin cậy, hãy đặt ngưỡng cho trường hợp sử dụng của bạn
5. **Sử dụng phân trang** - Sử dụng helper `paginate()` để liệt kê các mô hình
6. **Ưu tiên chế độ neural** - Đối với mô hình tùy chỉnh, neural xử lý nhiều biến thể hơn template
