---
name: azure-ai-projects-dotnet
description: |
  Azure AI Projects SDK cho .NET. Client cấp cao cho các dự án Azure AI Foundry bao gồm agents, connections, datasets, deployments, evaluations, và indexes. Sử dụng cho quản lý dự án AI Foundry, các agent có phiên bản, và điều phối. Kích hoạt: "AI Projects", "AIProjectClient", "dự án Foundry", "agent có phiên bản", "đánh giá", "bộ dữ liệu", "kết nối", "deployments .NET".
package: Azure.AI.Projects
---

# Azure.AI.Projects (.NET)

SDK cấp cao cho các hoạt động dự án Azure AI Foundry bao gồm agents, connections, datasets, deployments, evaluations, và indexes.

## Cài đặt

```bash
dotnet add package Azure.AI.Projects
dotnet add package Azure.Identity

# Tùy chọn: Cho các agent có phiên bản với tiện ích mở rộng OpenAI
dotnet add package Azure.AI.Projects.OpenAI --prerelease

# Tùy chọn: Cho các hoạt động agent cấp thấp
dotnet add package Azure.AI.Agents.Persistent --prerelease
```

**Phiên bản Hiện tại**: GA v1.1.0, Preview v1.2.0-beta.5

## Biến Môi trường

```bash
PROJECT_ENDPOINT=https://<resource>.services.ai.azure.com/api/projects/<project>
MODEL_DEPLOYMENT_NAME=gpt-4o-mini
CONNECTION_NAME=<your-connection-name>
AI_SEARCH_CONNECTION_NAME=<ai-search-connection>
```

## Xác thực

```csharp
using Azure.Identity;
using Azure.AI.Projects;

var endpoint = Environment.GetEnvironmentVariable("PROJECT_ENDPOINT");
AIProjectClient projectClient = new AIProjectClient(
    new Uri(endpoint),
    new DefaultAzureCredential());
```

## Phân cấp Client

```
AIProjectClient
├── Agents          → AIProjectAgentsOperations (agent có phiên bản)
├── Connections     → ConnectionsClient
├── Datasets        → DatasetsClient
├── Deployments     → DeploymentsClient
├── Evaluations     → EvaluationsClient
├── Evaluators      → EvaluatorsClient
├── Indexes         → IndexesClient
├── Telemetry       → AIProjectTelemetry
├── OpenAI          → ProjectOpenAIClient (preview)
└── GetPersistentAgentsClient() → PersistentAgentsClient
```

## Quy trình Cốt lõi

### 1. Lấy Client Agents Bền vững (Low-level)

```csharp
// Lấy client agents cấp thấp từ project client
PersistentAgentsClient agentsClient = projectClient.GetPersistentAgentsClient();

// Tạo agent
PersistentAgent agent = await agentsClient.Administration.CreateAgentAsync(
    model: "gpt-4o-mini",
    name: "Math Tutor",
    instructions: "Bạn là một gia sư toán cá nhân.");

// Tạo luồng (thread) và chạy (run)
PersistentAgentThread thread = await agentsClient.Threads.CreateThreadAsync();
await agentsClient.Messages.CreateMessageAsync(thread.Id, MessageRole.User, "Giải phương trình 3x + 11 = 14");
ThreadRun run = await agentsClient.Runs.CreateRunAsync(thread.Id, agent.Id);

// Poll để đợi hoàn thành
do
{
    await Task.Delay(500);
    run = await agentsClient.Runs.GetRunAsync(thread.Id, run.Id);
}
while (run.Status == RunStatus.Queued || run.Status == RunStatus.InProgress);

// Lấy tin nhắn
await foreach (var msg in agentsClient.Messages.GetMessagesAsync(thread.Id))
{
    foreach (var content in msg.ContentItems)
    {
        if (content is MessageTextContent textContent)
            Console.WriteLine(textContent.Text);
    }
}

// Dọn dẹp
await agentsClient.Threads.DeleteThreadAsync(thread.Id);
await agentsClient.Administration.DeleteAgentAsync(agent.Id);
```

### 2. Các Agent có Phiên bản với Công cụ (Preview)

```csharp
using Azure.AI.Projects.OpenAI;

// Tạo agent với công cụ tìm kiếm web
PromptAgentDefinition agentDefinition = new(model: "gpt-4o-mini")
{
    Instructions = "Bạn là một trợ lý hữu ích có thể tìm kiếm trên web",
    Tools = {
        ResponseTool.CreateWebSearchTool(
            userLocation: WebSearchToolLocation.CreateApproximateLocation(
                country: "US",
                city: "Seattle",
                region: "Washington"
            )
        ),
    }
};

AgentVersion agentVersion = await projectClient.Agents.CreateAgentVersionAsync(
    agentName: "myAgent",
    options: new(agentDefinition));

// Lấy response client
ProjectResponsesClient responseClient = projectClient.OpenAI.GetProjectResponsesClientForAgent(agentVersion.Name);

// Tạo phản hồi
ResponseResult response = responseClient.CreateResponse("Thời tiết ở Seattle thế nào?");
Console.WriteLine(response.GetOutputText());

// Dọn dẹp
projectClient.Agents.DeleteAgentVersion(agentName: agentVersion.Name, agentVersion: agentVersion.Version);
```

### 3. Kết nối (Connections)

```csharp
// Liệt kê tất cả kết nối
foreach (AIProjectConnection connection in projectClient.Connections.GetConnections())
{
    Console.WriteLine($"{connection.Name}: {connection.ConnectionType}");
}

// Lấy kết nối cụ thể
AIProjectConnection conn = projectClient.Connections.GetConnection(
    connectionName,
    includeCredentials: true);

// Lấy kết nối mặc định
AIProjectConnection defaultConn = projectClient.Connections.GetDefaultConnection(
    includeCredentials: false);
```

### 4. Triển khai (Deployments)

```csharp
// Liệt kê tất cả triển khai
foreach (AIProjectDeployment deployment in projectClient.Deployments.GetDeployments())
{
    Console.WriteLine($"{deployment.Name}: {deployment.ModelName}");
}

// Lọc theo nhà xuất bản
foreach (var deployment in projectClient.Deployments.GetDeployments(modelPublisher: "Microsoft"))
{
    Console.WriteLine(deployment.Name);
}

// Lấy triển khai cụ thể
ModelDeployment details = (ModelDeployment)projectClient.Deployments.GetDeployment("gpt-4o-mini");
```

### 5. Bộ dữ liệu (Datasets)

```csharp
// Tải lên một tệp
FileDataset fileDataset = projectClient.Datasets.UploadFile(
    name: "my-dataset",
    version: "1.0",
    filePath: "data/training.txt",
    connectionName: connectionName);

// Tải lên thư mục
FolderDataset folderDataset = projectClient.Datasets.UploadFolder(
    name: "my-dataset",
    version: "2.0",
    folderPath: "data/training",
    connectionName: connectionName,
    filePattern: new Regex(".*\\.txt"));

// Lấy bộ dữ liệu
AIProjectDataset dataset = projectClient.Datasets.GetDataset("my-dataset", "1.0");

// Xóa bộ dữ liệu
projectClient.Datasets.Delete("my-dataset", "1.0");
```

### 6. Chỉ mục (Indexes)

```csharp
// Tạo chỉ mục Azure AI Search
AzureAISearchIndex searchIndex = new(aiSearchConnectionName, aiSearchIndexName)
{
    Description = "Index Mẫu"
};

searchIndex = (AzureAISearchIndex)projectClient.Indexes.CreateOrUpdate(
    name: "my-index",
    version: "1.0",
    index: searchIndex);

// Liệt kê chỉ mục
foreach (AIProjectIndex index in projectClient.Indexes.GetIndexes())
{
    Console.WriteLine(index.Name);
}

// Xóa chỉ mục
projectClient.Indexes.Delete(name: "my-index", version: "1.0");
```

### 7. Đánh giá (Evaluations)

```csharp
// Tạo cấu hình đánh giá
var evaluatorConfig = new EvaluatorConfiguration(id: EvaluatorIDs.Relevance);
evaluatorConfig.InitParams.Add("deployment_name", BinaryData.FromObjectAsJson("gpt-4o"));

// Tạo đánh giá
Evaluation evaluation = new Evaluation(
    data: new InputDataset("<dataset_id>"),
    evaluators: new Dictionary<string, EvaluatorConfiguration>
    {
        { "relevance", evaluatorConfig }
    }
)
{
    DisplayName = "Đánh giá Mẫu"
};

// Chạy đánh giá
Evaluation result = projectClient.Evaluations.Create(evaluation: evaluation);

// Lấy đánh giá
Evaluation getResult = projectClient.Evaluations.Get(result.Name);

// Liệt kê đánh giá
foreach (var eval in projectClient.Evaluations.GetAll())
{
    Console.WriteLine($"{eval.DisplayName}: {eval.Status}");
}
```

### 8. Lấy Azure OpenAI Chat Client

```csharp
using Azure.AI.OpenAI;
using OpenAI.Chat;

ClientConnection connection = projectClient.GetConnection(typeof(AzureOpenAIClient).FullName!);

if (!connection.TryGetLocatorAsUri(out Uri uri) || uri is null)
    throw new InvalidOperationException("URI không hợp lệ.");

uri = new Uri($"https://{uri.Host}");

AzureOpenAIClient azureOpenAIClient = new AzureOpenAIClient(uri, new DefaultAzureCredential());
ChatClient chatClient = azureOpenAIClient.GetChatClient("gpt-4o-mini");

ChatCompletion result = chatClient.CompleteChat("Liệt kê tất cả màu cầu vồng");
Console.WriteLine(result.Content[0].Text);
```

## Các Công cụ Agent Có sẵn

| Công cụ          | Lớp                             | Mục đích                       |
| ---------------- | ------------------------------- | ------------------------------ |
| Code Interpreter | `CodeInterpreterToolDefinition` | Thực thi mã Python             |
| File Search      | `FileSearchToolDefinition`      | Tìm kiếm tệp đã tải lên        |
| Function Calling | `FunctionToolDefinition`        | Gọi hàm tùy chỉnh              |
| Bing Grounding   | `BingGroundingToolDefinition`   | Tìm kiếm web qua Bing          |
| Azure AI Search  | `AzureAISearchToolDefinition`   | Tìm kiếm chỉ mục Azure AI      |
| OpenAPI          | `OpenApiToolDefinition`         | Gọi API bên ngoài              |
| Azure Functions  | `AzureFunctionToolDefinition`   | Gọi Azure Functions            |
| MCP              | `MCPToolDefinition`             | Công cụ Model Context Protocol |

## Tài liệu tham khảo các Loại Chính

| Loại                     | Mục đích                      |
| ------------------------ | ----------------------------- |
| `AIProjectClient`        | Điểm nhập chính               |
| `PersistentAgentsClient` | Hoạt động agent cấp thấp      |
| `PromptAgentDefinition`  | Định nghĩa agent có phiên bản |
| `AgentVersion`           | Instance agent có phiên bản   |
| `AIProjectConnection`    | Kết nối đến tài nguyên Azure  |
| `AIProjectDeployment`    | Thông tin triển khai mô hình  |
| `AIProjectDataset`       | Metadata bộ dữ liệu           |
| `AIProjectIndex`         | Metadata chỉ mục tìm kiếm     |
| `Evaluation`             | Cấu hình và kết quả đánh giá  |

## Thực hành Tốt nhất

1. **Sử dụng `DefaultAzureCredential`** cho xác thực trong sản xuất
2. **Sử dụng các phương thức async** (`*Async`) cho tất cả hoạt động I/O
3. **Poll với độ trễ phù hợp** (khuyến nghị 500ms) khi chờ các run
4. **Dọn dẹp tài nguyên** — xóa luồng, agent và tệp khi xong
5. **Sử dụng agent có phiên bản** (thông qua `Azure.AI.Projects.OpenAI`) cho các kịch bản sản xuất
6. **Lưu trữ ID kết nối** thay vì tên cho cấu hình công cụ
7. **Sử dụng `includeCredentials: true`** chỉ khi cần thông tin xác thực
8. **Xử lý phân trang** — sử dụng `AsyncPageable<T>` cho các hoạt động liệt kê

## Xử lý Lỗi

```csharp
using Azure;

try
{
    var result = await projectClient.Evaluations.CreateAsync(evaluation);
}
catch (RequestFailedException ex)
{
    Console.WriteLine($"Lỗi: {ex.Status} - {ex.ErrorCode}: {ex.Message}");
}
```

## SDK Liên quan

| SDK                          | Mục đích                       | Cài đặt                                         |
| ---------------------------- | ------------------------------ | ----------------------------------------------- |
| `Azure.AI.Projects`          | Client dự án cấp cao (SDK này) | `dotnet add package Azure.AI.Projects`          |
| `Azure.AI.Agents.Persistent` | Hoạt động agent cấp thấp       | `dotnet add package Azure.AI.Agents.Persistent` |
| `Azure.AI.Projects.OpenAI`   | Agent có phiên bản với OpenAI  | `dotnet add package Azure.AI.Projects.OpenAI`   |

## Liên kết Tham khảo

| Tài nguyên    | URL                                                                                   |
| ------------- | ------------------------------------------------------------------------------------- |
| NuGet Package | https://www.nuget.org/packages/Azure.AI.Projects                                      |
| API Reference | https://learn.microsoft.com/dotnet/api/azure.ai.projects                              |
| GitHub Source | https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/ai/Azure.AI.Projects         |
| Samples       | https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/ai/Azure.AI.Projects/samples |
