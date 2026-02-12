---
name: azure-ai-contentsafety-ts
description: Phân tích văn bản và hình ảnh để tìm nội dung độc hại bằng Azure AI Content Safety (@azure-rest/ai-content-safety). Sử dụng khi kiểm duyệt nội dung do người dùng tạo, phát hiện ngôn từ thù ghét, bạo lực, nội dung tình dục hoặc tự làm hại, hoặc quản lý danh sách chặn tùy chỉnh.
package: @azure-rest/ai-content-safety
---

# Azure AI Content Safety REST SDK cho TypeScript

Phân tích văn bản và hình ảnh tìm nội dung độc hại với danh sách chặn có thể tùy chỉnh.

## Cài đặt

```bash
npm install @azure-rest/ai-content-safety @azure/identity @azure/core-auth
```

## Biến Môi trường

```bash
CONTENT_SAFETY_ENDPOINT=https://<resource>.cognitiveservices.azure.com
CONTENT_SAFETY_KEY=<api-key>
```

## Xác thực

**Quan trọng**: Đây là một REST client. `ContentSafetyClient` là một **hàm**, không phải là một lớp (class).

### API Key

```typescript
import ContentSafetyClient from "@azure-rest/ai-content-safety";
import { AzureKeyCredential } from "@azure/core-auth";

const client = ContentSafetyClient(
  process.env.CONTENT_SAFETY_ENDPOINT!,
  new AzureKeyCredential(process.env.CONTENT_SAFETY_KEY!),
);
```

### DefaultAzureCredential

```typescript
import ContentSafetyClient from "@azure-rest/ai-content-safety";
import { DefaultAzureCredential } from "@azure/identity";

const client = ContentSafetyClient(
  process.env.CONTENT_SAFETY_ENDPOINT!,
  new DefaultAzureCredential(),
);
```

## Phân tích Văn bản

```typescript
import ContentSafetyClient, {
  isUnexpected,
} from "@azure-rest/ai-content-safety";

const result = await client.path("/text:analyze").post({
  body: {
    text: "Nội dung văn bản để phân tích",
    categories: ["Hate", "Sexual", "Violence", "SelfHarm"],
    outputType: "FourSeverityLevels", // hoặc "EightSeverityLevels"
  },
});

if (isUnexpected(result)) {
  throw result.body;
}

for (const analysis of result.body.categoriesAnalysis) {
  console.log(`${analysis.category}: mức độ nghiêm trọng ${analysis.severity}`);
}
```

## Phân tích Hình ảnh

### Nội dung Base64

```typescript
import { readFileSync } from "node:fs";

const imageBuffer = readFileSync("./image.png");
const base64Image = imageBuffer.toString("base64");

const result = await client.path("/image:analyze").post({
  body: {
    image: { content: base64Image },
  },
});

if (isUnexpected(result)) {
  throw result.body;
}

for (const analysis of result.body.categoriesAnalysis) {
  console.log(`${analysis.category}: mức độ nghiêm trọng ${analysis.severity}`);
}
```

### Blob URL

```typescript
const result = await client.path("/image:analyze").post({
  body: {
    image: {
      blobUrl: "https://storage.blob.core.windows.net/container/image.png",
    },
  },
});
```

## Quản lý Danh sách chặn (Blocklist)

### Tạo Danh sách chặn

```typescript
const result = await client
  .path("/text/blocklists/{blocklistName}", "my-blocklist")
  .patch({
    contentType: "application/merge-patch+json",
    body: {
      description: "Danh sách chặn tùy chỉnh cho các thuật ngữ bị cấm",
    },
  });

if (isUnexpected(result)) {
  throw result.body;
}

console.log(`Đã tạo: ${result.body.blocklistName}`);
```

### Thêm Mục vào Danh sách chặn

```typescript
const result = await client
  .path(
    "/text/blocklists/{blocklistName}:addOrUpdateBlocklistItems",
    "my-blocklist",
  )
  .post({
    body: {
      blocklistItems: [
        {
          text: "prohibited-term-1",
          description: "Thuật ngữ bị chặn đầu tiên",
        },
        { text: "prohibited-term-2", description: "Thuật ngữ bị chặn thứ hai" },
      ],
    },
  });

if (isUnexpected(result)) {
  throw result.body;
}

for (const item of result.body.blocklistItems ?? []) {
  console.log(`Đã thêm: ${item.blocklistItemId}`);
}
```

### Phân tích với Danh sách chặn

```typescript
const result = await client.path("/text:analyze").post({
  body: {
    text: "Văn bản có thể chứa các thuật ngữ bị chặn",
    blocklistNames: ["my-blocklist"],
    haltOnBlocklistHit: false,
  },
});

if (isUnexpected(result)) {
  throw result.body;
}

// Kiểm tra khớp danh sách chặn
if (result.body.blocklistsMatch) {
  for (const match of result.body.blocklistsMatch) {
    console.log(
      `Bị chặn: "${match.blocklistItemText}" từ ${match.blocklistName}`,
    );
  }
}
```

### Liệt kê Danh sách chặn

```typescript
const result = await client.path("/text/blocklists").get();

if (isUnexpected(result)) {
  throw result.body;
}

for (const blocklist of result.body.value ?? []) {
  console.log(`${blocklist.blocklistName}: ${blocklist.description}`);
}
```

### Xóa Danh sách chặn

```typescript
await client.path("/text/blocklists/{blocklistName}", "my-blocklist").delete();
```

## Các Danh mục Gây hại

| Danh mục          | Thuật ngữ API | Mô tả                                               |
| ----------------- | ------------- | --------------------------------------------------- |
| Hate and Fairness | `Hate`        | Ngôn ngữ phân biệt đối xử nhắm vào các nhóm bản sắc |
| Sexual            | `Sexual`      | Nội dung tình dục, khỏa thân, khiêu dâm             |
| Violence          | `Violence`    | Tác hại vật lý, vũ khí, khủng bố                    |
| Self-Harm         | `SelfHarm`    | Tự gây thương tích, tự sát, rối loạn ăn uống        |

## Các Mức độ Nghiêm trọng

| Mức độ | Rủi ro     | Hành động Khuyến nghị               |
| ------ | ---------- | ----------------------------------- |
| 0      | An toàn    | Cho phép                            |
| 2      | Thấp       | Xem xét hoặc cho phép với cảnh báo  |
| 4      | Trung bình | Chặn hoặc yêu cầu con người xem xét |
| 6      | Cao        | Chặn ngay lập tức                   |

**Các Loại Đầu ra**:

- `FourSeverityLevels` (mặc định): Trả về 0, 2, 4, 6
- `EightSeverityLevels`: Trả về 0-7

## Trợ giúp Kiểm duyệt Nội dung

```typescript
import ContentSafetyClient, {
  isUnexpected,
  TextCategoriesAnalysisOutput,
} from "@azure-rest/ai-content-safety";

interface ModerationResult {
  isAllowed: boolean;
  flaggedCategories: string[];
  maxSeverity: number;
  blocklistMatches: string[];
}

async function moderateContent(
  client: ReturnType<typeof ContentSafetyClient>,
  text: string,
  maxAllowedSeverity = 2,
  blocklistNames: string[] = [],
): Promise<ModerationResult> {
  const result = await client.path("/text:analyze").post({
    body: { text, blocklistNames, haltOnBlocklistHit: false },
  });

  if (isUnexpected(result)) {
    throw result.body;
  }

  const flaggedCategories = result.body.categoriesAnalysis
    .filter((c) => (c.severity ?? 0) > maxAllowedSeverity)
    .map((c) => c.category!);

  const maxSeverity = Math.max(
    ...result.body.categoriesAnalysis.map((c) => c.severity ?? 0),
  );

  const blocklistMatches = (result.body.blocklistsMatch ?? []).map(
    (m) => m.blocklistItemText!,
  );

  return {
    isAllowed: flaggedCategories.length === 0 && blocklistMatches.length === 0,
    flaggedCategories,
    maxSeverity,
    blocklistMatches,
  };
}
```

## Các API Endpoints

| Thao tác                    | Phương thức | Đường dẫn                                                    |
| --------------------------- | ----------- | ------------------------------------------------------------ |
| Phân tích Văn bản           | POST        | `/text:analyze`                                              |
| Phân tích Hình ảnh          | POST        | `/image:analyze`                                             |
| Tạo/Cập nhật Danh sách chặn | PATCH       | `/text/blocklists/{blocklistName}`                           |
| Liệt kê Danh sách chặn      | GET         | `/text/blocklists`                                           |
| Xóa Danh sách chặn          | DELETE      | `/text/blocklists/{blocklistName}`                           |
| Thêm Mục Danh sách chặn     | POST        | `/text/blocklists/{blocklistName}:addOrUpdateBlocklistItems` |
| Liệt kê Mục Danh sách chặn  | GET         | `/text/blocklists/{blocklistName}/blocklistItems`            |
| Xóa Mục Danh sách chặn      | POST        | `/text/blocklists/{blocklistName}:removeBlocklistItems`      |

## Các Types Chính

```typescript
import ContentSafetyClient, {
  isUnexpected,
  AnalyzeTextParameters,
  AnalyzeImageParameters,
  TextCategoriesAnalysisOutput,
  ImageCategoriesAnalysisOutput,
  TextBlocklist,
  TextBlocklistItem,
} from "@azure-rest/ai-content-safety";
```

## Thực hành Tốt nhất

1. **Luôn sử dụng isUnexpected()** - Type guard để xử lý lỗi
2. **Đặt các ngưỡng phù hợp** - Các danh mục khác nhau có thể cần các ngưỡng mức độ nghiêm trọng khác nhau
3. **Sử dụng danh sách chặn cho các thuật ngữ cụ thể theo miền** - Bổ sung cho phát hiện AI với các quy tắc tùy chỉnh
4. **Ghi log quyết định kiểm duyệt** - Giữ dấu vết kiểm toán để tuân thủ
5. **Xử lý các trường hợp biên** - Văn bản trống, văn bản rất dài, định dạng hình ảnh không được hỗ trợ
