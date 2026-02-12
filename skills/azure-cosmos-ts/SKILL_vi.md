---
name: azure-cosmos-ts
description: |
  Azure Cosmos DB JavaScript/TypeScript SDK (@azure/cosmos) cho các thao tác data plane. Sử dụng cho các thao tác CRUD trên tài liệu, truy vấn, thao tác hàng loạt (bulk operations), và quản lý container. Kích hoạt: "Cosmos DB", "@azure/cosmos", "CosmosClient", "document CRUD", "NoSQL queries", "bulk operations", "partition key", "container.items".
package: @azure/cosmos
---

# @azure/cosmos (TypeScript/JavaScript)

Data plane SDK cho các thao tác Azure Cosmos DB NoSQL API — CRUD trên tài liệu, truy vấn, thao tác hàng loạt.

> **⚠️ Data vs Management Plane**
>
> - **SDK này (@azure/cosmos)**: Các thao tác CRUD trên tài liệu, truy vấn, stored procedures
> - **Management SDK (@azure/arm-cosmosdb)**: Tạo tài khoản, cơ sở dữ liệu, containers qua ARM

## Cài đặt

```bash
npm install @azure/cosmos @azure/identity
```

**Phiên bản Hiện tại**: 4.9.0  
**Node.js**: >= 20.0.0

## Biến Môi trường

```bash
COSMOS_ENDPOINT=https://<account>.documents.azure.com:443/
COSMOS_DATABASE=<database-name>
COSMOS_CONTAINER=<container-name>
# Chỉ cho xác thực dựa trên key (ưu tiên AAD)
COSMOS_KEY=<account-key>
```

## Xác thực

### AAD với DefaultAzureCredential (Khuyên dùng)

```typescript
import { CosmosClient } from "@azure/cosmos";
import { DefaultAzureCredential } from "@azure/identity";

const client = new CosmosClient({
  endpoint: process.env.COSMOS_ENDPOINT!,
  aadCredentials: new DefaultAzureCredential(),
});
```

### Xác thực Dựa trên Key

```typescript
import { CosmosClient } from "@azure/cosmos";

// Tùy chọn 1: Endpoint + Key
const client = new CosmosClient({
  endpoint: process.env.COSMOS_ENDPOINT!,
  key: process.env.COSMOS_KEY!,
});

// Tùy chọn 2: Connection String
const client = new CosmosClient(process.env.COSMOS_CONNECTION_STRING!);
```

## Phân cấp Tài nguyên

```
CosmosClient
└── Database
    └── Container
        ├── Items (documents)
        ├── Scripts (stored procedures, triggers, UDFs)
        └── Conflicts
```

## Các Thao tác Cốt lõi

### Thiết lập Database & Container

```typescript
const { database } = await client.databases.createIfNotExists({
  id: "my-database",
});

const { container } = await database.containers.createIfNotExists({
  id: "my-container",
  partitionKey: { paths: ["/partitionKey"] },
});
```

### Tạo Document

```typescript
interface Product {
  id: string;
  partitionKey: string;
  name: string;
  price: number;
}

const item: Product = {
  id: "product-1",
  partitionKey: "electronics",
  name: "Laptop",
  price: 999.99,
};

const { resource } = await container.items.create<Product>(item);
```

### Đọc Document

```typescript
const { resource } = await container
  .item("product-1", "electronics") // id, partitionKey
  .read<Product>();

if (resource) {
  console.log(resource.name);
}
```

### Cập nhật Document (Thay thế)

```typescript
const { resource: existing } = await container
  .item("product-1", "electronics")
  .read<Product>();

if (existing) {
  existing.price = 899.99;
  const { resource: updated } = await container
    .item("product-1", "electronics")
    .replace<Product>(existing);
}
```

### Upsert Document

```typescript
const item: Product = {
  id: "product-1",
  partitionKey: "electronics",
  name: "Laptop Pro",
  price: 1299.99,
};

const { resource } = await container.items.upsert<Product>(item);
```

### Xóa Document

```typescript
await container.item("product-1", "electronics").delete();
```

### Patch Document (Cập nhật Một phần)

```typescript
import { PatchOperation } from "@azure/cosmos";

const operations: PatchOperation[] = [
  { op: "replace", path: "/price", value: 799.99 },
  { op: "add", path: "/discount", value: true },
  { op: "remove", path: "/oldField" },
];

const { resource } = await container
  .item("product-1", "electronics")
  .patch<Product>(operations);
```

## Truy vấn

### Truy vấn Đơn giản

```typescript
const { resources } = await container.items
  .query<Product>("SELECT * FROM c WHERE c.price < 1000")
  .fetchAll();
```

### Truy vấn Tham số hóa (Khuyên dùng)

```typescript
import { SqlQuerySpec } from "@azure/cosmos";

const querySpec: SqlQuerySpec = {
  query:
    "SELECT * FROM c WHERE c.partitionKey = @category AND c.price < @maxPrice",
  parameters: [
    { name: "@category", value: "electronics" },
    { name: "@maxPrice", value: 1000 },
  ],
};

const { resources } = await container.items
  .query<Product>(querySpec)
  .fetchAll();
```

### Truy vấn với Phân trang

```typescript
const queryIterator = container.items.query<Product>(querySpec, {
  maxItemCount: 10, // Items mỗi trang
});

while (queryIterator.hasMoreResults()) {
  const { resources, continuationToken } = await queryIterator.fetchNext();
  console.log(`Page with ${resources?.length} items`);
  // Sử dụng continuationToken cho trang tiếp theo nếu cần
}
```

### Truy vấn Xuyên Partition (Cross-Partition Query)

```typescript
const { resources } = await container.items
  .query<Product>("SELECT * FROM c WHERE c.price > 500", {
    enableCrossPartitionQuery: true,
  })
  .fetchAll();
```

## Thao tác Hàng loạt (Bulk Operations)

### Thực thi Thao tác Hàng loạt

```typescript
import { BulkOperationType, OperationInput } from "@azure/cosmos";

const operations: OperationInput[] = [
  {
    operationType: BulkOperationType.Create,
    resourceBody: { id: "1", partitionKey: "cat-a", name: "Item 1" },
  },
  {
    operationType: BulkOperationType.Upsert,
    resourceBody: { id: "2", partitionKey: "cat-a", name: "Item 2" },
  },
  {
    operationType: BulkOperationType.Read,
    id: "3",
    partitionKey: "cat-b",
  },
  {
    operationType: BulkOperationType.Replace,
    id: "4",
    partitionKey: "cat-b",
    resourceBody: { id: "4", partitionKey: "cat-b", name: "Updated" },
  },
  {
    operationType: BulkOperationType.Delete,
    id: "5",
    partitionKey: "cat-c",
  },
  {
    operationType: BulkOperationType.Patch,
    id: "6",
    partitionKey: "cat-c",
    resourceBody: {
      operations: [{ op: "replace", path: "/name", value: "Patched" }],
    },
  },
];

const response = await container.items.executeBulkOperations(operations);

response.forEach((result, index) => {
  if (result.statusCode >= 200 && result.statusCode < 300) {
    console.log(`Operation ${index} succeeded`);
  } else {
    console.error(`Operation ${index} failed: ${result.statusCode}`);
  }
});
```

## Partition Keys

### Partition Key Đơn giản

```typescript
const { container } = await database.containers.createIfNotExists({
  id: "products",
  partitionKey: { paths: ["/category"] },
});
```

### Partition Key Phân cấp (MultiHash)

```typescript
import { PartitionKeyDefinitionVersion, PartitionKeyKind } from "@azure/cosmos";

const { container } = await database.containers.createIfNotExists({
  id: "orders",
  partitionKey: {
    paths: ["/tenantId", "/userId", "/sessionId"],
    version: PartitionKeyDefinitionVersion.V2,
    kind: PartitionKeyKind.MultiHash,
  },
});

// Các thao tác yêu cầu mảng các giá trị partition key
const { resource } = await container.items.create({
  id: "order-1",
  tenantId: "tenant-a",
  userId: "user-123",
  sessionId: "session-xyz",
  total: 99.99,
});

// Đọc với partition key phân cấp
const { resource: order } = await container
  .item("order-1", ["tenant-a", "user-123", "session-xyz"])
  .read();
```

## Xử lý Lỗi

```typescript
import { ErrorResponse } from "@azure/cosmos";

try {
  const { resource } = await container.item("missing", "pk").read();
} catch (error) {
  if (error instanceof ErrorResponse) {
    switch (error.code) {
      case 404:
        console.log("Document not found");
        break;
      case 409:
        console.log("Conflict - document already exists");
        break;
      case 412:
        console.log("Precondition failed (ETag mismatch)");
        break;
      case 429:
        console.log("Rate limited - retry after:", error.retryAfterInMs);
        break;
      default:
        console.error(`Cosmos error ${error.code}: ${error.message}`);
    }
  }
  throw error;
}
```

## Optimistic Concurrency (ETags)

```typescript
// Đọc với ETag
const { resource, etag } = await container
  .item("product-1", "electronics")
  .read<Product>();

if (resource && etag) {
  resource.price = 899.99;

  try {
    // Thay thế chỉ khi ETag khớp
    await container.item("product-1", "electronics").replace(resource, {
      accessCondition: { type: "IfMatch", condition: etag },
    });
  } catch (error) {
    if (error instanceof ErrorResponse && error.code === 412) {
      console.log("Document was modified by another process");
    }
  }
}
```

## Tham khảo Types TypeScript

```typescript
import {
  // Client & Resources
  CosmosClient,
  Database,
  Container,
  Item,
  Items,

  // Operations
  OperationInput,
  BulkOperationType,
  PatchOperation,

  // Queries
  SqlQuerySpec,
  SqlParameter,
  FeedOptions,

  // Partition Keys
  PartitionKeyDefinition,
  PartitionKeyDefinitionVersion,
  PartitionKeyKind,

  // Responses
  ItemResponse,
  FeedResponse,
  ResourceResponse,

  // Errors
  ErrorResponse,
} from "@azure/cosmos";
```

## Thực hành Tốt nhất

1. **Sử dụng xác thực AAD** — Ưu tiên `DefaultAzureCredential` hơn keys
2. **Luôn sử dụng truy vấn tham số hóa** — Ngăn chặn injection, cải thiện caching plan
3. **Chỉ định partition key** — Tránh truy vấn xuyên partition khi có thể
4. **Sử dụng thao tác hàng loạt** — Cho nhiều ghi, sử dụng `executeBulkOperations`
5. **Xử lý lỗi 429** — Triển khai logic thử lại với backoff theo cấp số nhân
6. **Sử dụng ETags cho đồng thời** — Ngăn chặn mất cập nhật trong các kịch bản đồng thời
7. **Đóng client khi tắt** — Gọi `client.dispose()` khi dọn dẹp

## Các Mẫu Chung

### Mẫu Service Layer

```typescript
export class ProductService {
  private container: Container;

  constructor(client: CosmosClient) {
    this.container = client
      .database(process.env.COSMOS_DATABASE!)
      .container(process.env.COSMOS_CONTAINER!);
  }

  async getById(id: string, category: string): Promise<Product | null> {
    try {
      const { resource } = await this.container
        .item(id, category)
        .read<Product>();
      return resource ?? null;
    } catch (error) {
      if (error instanceof ErrorResponse && error.code === 404) {
        return null;
      }
      throw error;
    }
  }

  async create(product: Omit<Product, "id">): Promise<Product> {
    const item = { ...product, id: crypto.randomUUID() };
    const { resource } = await this.container.items.create<Product>(item);
    return resource!;
  }

  async findByCategory(category: string): Promise<Product[]> {
    const querySpec: SqlQuerySpec = {
      query: "SELECT * FROM c WHERE c.partitionKey = @category",
      parameters: [{ name: "@category", value: category }],
    };
    const { resources } = await this.container.items
      .query<Product>(querySpec)
      .fetchAll();
    return resources;
  }
}
```

## Các SDK Liên quan

| SDK                   | Mục đích               | Cài đặt                           |
| --------------------- | ---------------------- | --------------------------------- |
| `@azure/cosmos`       | Data plane (SDK này)   | `npm install @azure/cosmos`       |
| `@azure/arm-cosmosdb` | Management plane (ARM) | `npm install @azure/arm-cosmosdb` |
| `@azure/identity`     | Xác thực               | `npm install @azure/identity`     |
