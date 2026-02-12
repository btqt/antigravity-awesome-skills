---
name: azure-search-documents-ts
description: Xây dựng các ứng dụng tìm kiếm sử dụng Azure AI Search SDK cho JavaScript (@azure/search-documents). Sử dụng khi tạo/quản lý indexes, triển khai vector/hybrid search, semantic ranking, hoặc xây dựng agentic retrieval với cơ sở tri thức.
package: @azure/search-documents
---

# Azure AI Search SDK cho TypeScript

Xây dựng các ứng dụng tìm kiếm với khả năng vector, hybrid, và semantic search.

## Cài đặt

```bash
npm install @azure/search-documents @azure/identity
```

## Biến Môi trường

```bash
AZURE_SEARCH_ENDPOINT=https://<service-name>.search.windows.net
AZURE_SEARCH_INDEX_NAME=my-index
AZURE_SEARCH_ADMIN_KEY=<admin-key>  # Tùy chọn nếu sử dụng Entra ID
```

## Xác thực

```typescript
import { SearchClient, SearchIndexClient } from "@azure/search-documents";
import { DefaultAzureCredential } from "@azure/identity";

const endpoint = process.env.AZURE_SEARCH_ENDPOINT!;
const indexName = process.env.AZURE_SEARCH_INDEX_NAME!;
const credential = new DefaultAzureCredential();

// Cho tìm kiếm
const searchClient = new SearchClient(endpoint, indexName, credential);

// Cho quản lý index
const indexClient = new SearchIndexClient(endpoint, credential);
```

## Quy trình Công việc Cốt lõi

### Tạo Index với Trường Vector

```typescript
import {
  SearchIndex,
  SearchField,
  VectorSearch,
} from "@azure/search-documents";

const index: SearchIndex = {
  name: "products",
  fields: [
    { name: "id", type: "Edm.String", key: true },
    { name: "title", type: "Edm.String", searchable: true },
    { name: "description", type: "Edm.String", searchable: true },
    { name: "category", type: "Edm.String", filterable: true, facetable: true },
    {
      name: "embedding",
      type: "Collection(Edm.Single)",
      searchable: true,
      vectorSearchDimensions: 1536,
      vectorSearchProfileName: "vector-profile",
    },
  ],
  vectorSearch: {
    algorithms: [{ name: "hnsw-algorithm", kind: "hnsw" }],
    profiles: [
      { name: "vector-profile", algorithmConfigurationName: "hnsw-algorithm" },
    ],
  },
};

await indexClient.createOrUpdateIndex(index);
```

### Index Tài liệu

```typescript
const documents = [
  { id: "1", title: "Widget", description: "A useful widget", category: "Tools", embedding: [...] },
  { id: "2", title: "Gadget", description: "A cool gadget", category: "Electronics", embedding: [...] },
];

const result = await searchClient.uploadDocuments(documents);
console.log(`Indexed ${result.results.length} documents`);
```

### Full-Text Search

```typescript
const results = await searchClient.search("widget", {
  select: ["id", "title", "description"],
  filter: "category eq 'Tools'",
  orderBy: ["title asc"],
  top: 10,
});

for await (const result of results.results) {
  console.log(`${result.document.title}: ${result.score}`);
}
```

### Vector Search

```typescript
const queryVector = await getEmbedding("useful tool"); // Hàm embedding của bạn

const results = await searchClient.search("*", {
  vectorSearchOptions: {
    queries: [
      {
        kind: "vector",
        vector: queryVector,
        fields: ["embedding"],
        kNearestNeighborsCount: 10,
      },
    ],
  },
  select: ["id", "title", "description"],
});

for await (const result of results.results) {
  console.log(`${result.document.title}: ${result.score}`);
}
```

### Hybrid Search (Text + Vector)

```typescript
const queryVector = await getEmbedding("useful tool");

const results = await searchClient.search("tool", {
  vectorSearchOptions: {
    queries: [
      {
        kind: "vector",
        vector: queryVector,
        fields: ["embedding"],
        kNearestNeighborsCount: 50,
      },
    ],
  },
  select: ["id", "title", "description"],
  top: 10,
});
```

### Semantic Search

```typescript
// Index phải có cấu hình semantic
const index: SearchIndex = {
  name: "products",
  fields: [...],
  semanticSearch: {
    configurations: [
      {
        name: "semantic-config",
        prioritizedFields: {
          titleField: { name: "title" },
          contentFields: [{ name: "description" }],
        },
      },
    ],
  },
};

// Tìm kiếm với xếp hạng ngữ nghĩa
const results = await searchClient.search("best tool for the job", {
  queryType: "semantic",
  semanticSearchOptions: {
    configurationName: "semantic-config",
    captions: { captionType: "extractive" },
    answers: { answerType: "extractive", count: 3 },
  },
  select: ["id", "title", "description"],
});

for await (const result of results.results) {
  console.log(`${result.document.title}`);
  console.log(`  Caption: ${result.captions?.[0]?.text}`);
  console.log(`  Reranker Score: ${result.rerankerScore}`);
}
```

## Lọc và Phân loại (Facets)

```typescript
// Cú pháp bộ lọc
const results = await searchClient.search("*", {
  filter: "category eq 'Electronics' and price lt 100",
  facets: ["category,count:10", "brand"],
});

// Truy cập facets
for (const [facetName, facetResults] of Object.entries(results.facets || {})) {
  console.log(`${facetName}:`);
  for (const facet of facetResults) {
    console.log(`  ${facet.value}: ${facet.count}`);
  }
}
```

## Autocomplete và Suggestions

```typescript
// Tạo suggester trong index
const index: SearchIndex = {
  name: "products",
  fields: [...],
  suggesters: [
    { name: "sg", sourceFields: ["title", "description"] },
  ],
};

// Autocomplete
const autocomplete = await searchClient.autocomplete("wid", "sg", {
  mode: "twoTerms",
  top: 5,
});

// Suggestions
const suggestions = await searchClient.suggest("wid", "sg", {
  select: ["title"],
  top: 5,
});
```

## Các Hoạt động Theo Lô (Batch Operations)

```typescript
// Batch upload, merge, delete
const batch = [
  { upload: { id: "1", title: "New Item" } },
  { merge: { id: "2", title: "Updated Title" } },
  { delete: { id: "3" } },
];

const result = await searchClient.indexDocuments({ actions: batch });
```

## Các Loại Chính

```typescript
import {
  SearchClient,
  SearchIndexClient,
  SearchIndexerClient,
  SearchIndex,
  SearchField,
  SearchOptions,
  VectorSearch,
  SemanticSearch,
  SearchIterator,
} from "@azure/search-documents";
```

## Thực hành Tốt nhất

1. **Sử dụng hybrid search** - Kết hợp vector + text để có kết quả tốt nhất
2. **Bật semantic ranking** - Cải thiện độ phù hợp cho các truy vấn ngôn ngữ tự nhiên
3. **Tải lên tài liệu theo lô** - Sử dụng `uploadDocuments` với mảng, không phải từng tài liệu đơn lẻ
4. **Sử dụng bộ lọc để bảo mật** - Triển khai bảo mật cấp tài liệu với bộ lọc
5. **Index tăng dần** - Sử dụng `mergeOrUploadDocuments` cho các cập nhật
6. **Giám sát hiệu suất truy vấn** - Sử dụng `includeTotalCount: true` một cách tiết kiệm trong production
