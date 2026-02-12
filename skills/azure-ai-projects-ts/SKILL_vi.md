---
name: azure-ai-projects-ts
description: Xây dựng các ứng dụng AI sử dụng Azure AI Projects SDK cho JavaScript (@azure/ai-projects). Sử dụng khi làm việc với các client dự án Foundry, agents, connections, deployments, datasets, indexes, evaluations, hoặc lấy OpenAI clients.
package: @azure/ai-projects
---

# Azure AI Projects SDK cho TypeScript

SDK cấp cao cho các dự án Azure AI Foundry với agents, connections, deployments, và evaluations.

## Cài đặt

```bash
npm install @azure/ai-projects @azure/identity
```

Để tracing:

```bash
npm install @azure/monitor-opentelemetry @opentelemetry/api
```

## Biến Môi trường

```bash
AZURE_AI_PROJECT_ENDPOINT=https://<resource>.services.ai.azure.com/api/projects/<project>
MODEL_DEPLOYMENT_NAME=gpt-4o
```

## Xác thực

```typescript
import { AIProjectClient } from "@azure/ai-projects";
import { DefaultAzureCredential } from "@azure/identity";

const client = new AIProjectClient(
  process.env.AZURE_AI_PROJECT_ENDPOINT!,
  new DefaultAzureCredential(),
);
```

## Các Nhóm Hoạt động

| Nhóm                  | Mục đích                                |
| --------------------- | --------------------------------------- |
| `client.agents`       | Tạo và quản lý AI agents                |
| `client.connections`  | Liệt kê các tài nguyên Azure đã kết nối |
| `client.deployments`  | Liệt kê các triển khai mô hình          |
| `client.datasets`     | Tải lên và quản lý bộ dữ liệu           |
| `client.indexes`      | Tạo và quản lý chỉ mục tìm kiếm         |
| `client.evaluators`   | Quản lý các chỉ số đánh giá             |
| `client.memoryStores` | Quản lý bộ nhớ agent                    |

## Lấy OpenAI Client

```typescript
const openAIClient = await client.getOpenAIClient();

// Sử dụng cho phản hồi (responses)
const response = await openAIClient.responses.create({
  model: "gpt-4o",
  input: "Thủ đô của Pháp là gì?",
});

// Sử dụng cho hội thoại (conversations)
const conversation = await openAIClient.conversations.create({
  items: [{ type: "message", role: "user", content: "Xin chào!" }],
});
```

## Agents

### Tạo Agent

```typescript
const agent = await client.agents.createVersion("my-agent", {
  kind: "prompt",
  model: "gpt-4o",
  instructions: "Bạn là một trợ lý hữu ích.",
});
```

### Agent với Công cụ

```typescript
// Code Interpreter
const agent = await client.agents.createVersion("code-agent", {
  kind: "prompt",
  model: "gpt-4o",
  instructions: "Bạn có thể thực thi mã.",
  tools: [{ type: "code_interpreter", container: { type: "auto" } }],
});

// File Search
const agent = await client.agents.createVersion("search-agent", {
  kind: "prompt",
  model: "gpt-4o",
  tools: [{ type: "file_search", vector_store_ids: [vectorStoreId] }],
});

// Web Search
const agent = await client.agents.createVersion("web-agent", {
  kind: "prompt",
  model: "gpt-4o",
  tools: [
    {
      type: "web_search_preview",
      user_location: { type: "approximate", country: "US", city: "Seattle" },
    },
  ],
});

// Azure AI Search
const agent = await client.agents.createVersion("aisearch-agent", {
  kind: "prompt",
  model: "gpt-4o",
  tools: [
    {
      type: "azure_ai_search",
      azure_ai_search: {
        indexes: [
          {
            project_connection_id: connectionId,
            index_name: "my-index",
            query_type: "simple",
          },
        ],
      },
    },
  ],
});

// Function Tool
const agent = await client.agents.createVersion("func-agent", {
  kind: "prompt",
  model: "gpt-4o",
  tools: [
    {
      type: "function",
      function: {
        name: "get_weather",
        description: "Lấy thời tiết cho một địa điểm",
        strict: true,
        parameters: {
          type: "object",
          properties: { location: { type: "string" } },
          required: ["location"],
        },
      },
    },
  ],
});

// MCP Tool
const agent = await client.agents.createVersion("mcp-agent", {
  kind: "prompt",
  model: "gpt-4o",
  tools: [
    {
      type: "mcp",
      server_label: "my-mcp",
      server_url: "https://mcp-server.example.com",
      require_approval: "always",
    },
  ],
});
```

### Chạy Agent

```typescript
const openAIClient = await client.getOpenAIClient();

// Tạo hội thoại
const conversation = await openAIClient.conversations.create({
  items: [{ type: "message", role: "user", content: "Xin chào!" }],
});

// Tạo phản hồi sử dụng agent
const response = await openAIClient.responses.create(
  { conversation: conversation.id },
  { body: { agent: { name: agent.name, type: "agent_reference" } } },
);

// Dọn dẹp
await openAIClient.conversations.delete(conversation.id);
await client.agents.deleteVersion(agent.name, agent.version);
```

## Kết nối (Connections)

```typescript
// Liệt kê tất cả kết nối
for await (const conn of client.connections.list()) {
  console.log(conn.name, conn.type);
}

// Lấy kết nối theo tên
const conn = await client.connections.get("my-connection");

// Lấy kết nối với thông tin xác thực
const connWithCreds =
  await client.connections.getWithCredentials("my-connection");

// Lấy kết nối mặc định theo loại
const defaultAzureOpenAI = await client.connections.getDefault(
  "AzureOpenAI",
  true,
);
```

## Triển khai (Deployments)

```typescript
// Liệt kê tất cả triển khai
for await (const deployment of client.deployments.list()) {
  if (deployment.type === "ModelDeployment") {
    console.log(deployment.name, deployment.modelName);
  }
}

// Lọc theo nhà xuất bản
for await (const d of client.deployments.list({ modelPublisher: "OpenAI" })) {
  console.log(d.name);
}

// Lấy triển khai cụ thể
const deployment = await client.deployments.get("gpt-4o");
```

## Bộ dữ liệu (Datasets)

```typescript
// Tải lên một tệp
const dataset = await client.datasets.uploadFile(
  "my-dataset",
  "1.0",
  "./data/training.jsonl",
);

// Tải lên thư mục
const dataset = await client.datasets.uploadFolder(
  "my-dataset",
  "2.0",
  "./data/documents/",
);

// Lấy bộ dữ liệu
const ds = await client.datasets.get("my-dataset", "1.0");

// Liệt kê các phiên bản
for await (const version of client.datasets.listVersions("my-dataset")) {
  console.log(version);
}

// Xóa
await client.datasets.delete("my-dataset", "1.0");
```

## Chỉ mục (Indexes)

```typescript
import { AzureAISearchIndex } from "@azure/ai-projects";

const indexConfig: AzureAISearchIndex = {
  name: "my-index",
  type: "AzureSearch",
  version: "1",
  indexName: "my-index",
  connectionName: "search-connection",
};

// Tạo chỉ mục
const index = await client.indexes.createOrUpdate("my-index", "1", indexConfig);

// Liệt kê chỉ mục
for await (const idx of client.indexes.list()) {
  console.log(idx.name);
}

// Xóa
await client.indexes.delete("my-index", "1");
```

## Các Loại Chính (Key Types)

```typescript
import {
  AIProjectClient,
  AIProjectClientOptionalParams,
  Connection,
  ModelDeployment,
  DatasetVersionUnion,
  AzureAISearchIndex,
} from "@azure/ai-projects";
```

## Thực hành Tốt nhất

1. **Sử dụng getOpenAIClient()** - Cho phản hồi, hội thoại, tệp và kho vector
2. **Phiên bản hóa agent của bạn** - Sử dụng `createVersion` cho các định nghĩa agent có thể tái tạo
3. **Dọn dẹp tài nguyên** - Xóa agent, hội thoại khi xong
4. **Sử dụng kết nối** - Lấy thông tin xác thực từ kết nối dự án, đừng hardcode
5. **Lọc triển khai** - Sử dụng bộ lọc `modelPublisher` để tìm các mô hình cụ thể
