---
name: azure-ai-projects-java
description: |
  Azure AI Projects SDK cho Java. SDK cấp cao để quản lý dự án Azure AI Foundry bao gồm kết nối, bộ dữ liệu, chỉ mục và đánh giá.
  Kích hoạt: "AIProjectClient java", "azure ai projects java", "dự án Foundry java", "ConnectionsClient", "DatasetsClient", "IndexesClient".
package: com.azure:azure-ai-projects
---

# Azure AI Projects SDK cho Java

SDK cấp cao để quản lý dự án Azure AI Foundry với quyền truy cập vào các kết nối, bộ dữ liệu, chỉ mục và đánh giá.

## Cài đặt

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-ai-projects</artifactId>
    <version>1.0.0-beta.1</version>
</dependency>
```

## Biến Môi trường

```bash
PROJECT_ENDPOINT=https://<resource>.services.ai.azure.com/api/projects/<project>
```

## Xác thực

```java
import com.azure.ai.projects.AIProjectClientBuilder;
import com.azure.identity.DefaultAzureCredentialBuilder;

AIProjectClientBuilder builder = new AIProjectClientBuilder()
    .endpoint(System.getenv("PROJECT_ENDPOINT"))
    .credential(new DefaultAzureCredentialBuilder().build());
```

## Phân cấp Client

SDK cung cấp nhiều sub-client cho các hoạt động khác nhau:

| Client              | Mục đích                                |
| ------------------- | --------------------------------------- |
| `ConnectionsClient` | Liệt kê các tài nguyên Azure đã kết nối |
| `DatasetsClient`    | Tải lên tài liệu và quản lý bộ dữ liệu  |
| `DeploymentsClient` | Liệt kê các triển khai mô hình AI       |
| `IndexesClient`     | Tạo và quản lý chỉ mục tìm kiếm         |
| `EvaluationsClient` | Chạy đánh giá mô hình AI                |
| `EvaluatorsClient`  | Quản lý cấu hình bộ đánh giá            |
| `SchedulesClient`   | Quản lý các hoạt động đã lên lịch       |

```java
// Xây dựng sub-client từ builder
ConnectionsClient connectionsClient = builder.buildConnectionsClient();
DatasetsClient datasetsClient = builder.buildDatasetsClient();
DeploymentsClient deploymentsClient = builder.buildDeploymentsClient();
IndexesClient indexesClient = builder.buildIndexesClient();
EvaluationsClient evaluationsClient = builder.buildEvaluationsClient();
```

## Các Hoạt động Cốt lõi

### Liệt kê Kết nối

```java
import com.azure.ai.projects.models.Connection;
import com.azure.core.http.rest.PagedIterable;

PagedIterable<Connection> connections = connectionsClient.listConnections();
for (Connection connection : connections) {
    System.out.println("Tên: " + connection.getName());
    System.out.println("Loại: " + connection.getType());
    System.out.println("Loại Credential: " + connection.getCredentials().getType());
}
```

### Liệt kê Chỉ mục

```java
indexesClient.listLatest().forEach(index -> {
    System.out.println("Tên Index: " + index.getName());
    System.out.println("Phiên bản: " + index.getVersion());
    System.out.println("Mô tả: " + index.getDescription());
});
```

### Tạo hoặc Cập nhật Chỉ mục

```java
import com.azure.ai.projects.models.AzureAISearchIndex;
import com.azure.ai.projects.models.Index;

String indexName = "my-index";
String indexVersion = "1.0";
String searchConnectionName = System.getenv("AI_SEARCH_CONNECTION_NAME");
String searchIndexName = System.getenv("AI_SEARCH_INDEX_NAME");

Index index = indexesClient.createOrUpdate(
    indexName,
    indexVersion,
    new AzureAISearchIndex()
        .setConnectionName(searchConnectionName)
        .setIndexName(searchIndexName)
);

System.out.println("Đã tạo index: " + index.getName());
```

### Truy cập Đánh giá OpenAI

SDK hiển thị SDK chính thức của OpenAI cho các đánh giá:

```java
import com.openai.services.EvalService;

EvalService evalService = evaluationsClient.getOpenAIClient();
// Sử dụng trực tiếp API đánh giá của OpenAI
```

## Thực hành Tốt nhất

1. **Sử dụng DefaultAzureCredential** cho xác thực trong sản xuất
2. **Tái sử dụng client builder** để tạo nhiều sub-client một cách hiệu quả
3. **Xử lý phân trang** khi liệt kê tài nguyên với `PagedIterable`
4. **Sử dụng biến môi trường** cho tên kết nối và cấu hình
5. **Kiểm tra loại kết nối** trước khi truy cập credential

## Xử lý Lỗi

```java
import com.azure.core.exception.HttpResponseException;
import com.azure.core.exception.ResourceNotFoundException;

try {
    Index index = indexesClient.get(indexName, version);
} catch (ResourceNotFoundException e) {
    System.err.println("Không tìm thấy Index: " + indexName);
} catch (HttpResponseException e) {
    System.err.println("Lỗi: " + e.getResponse().getStatusCode());
}
```

## Liên kết Tham khảo

| Tài nguyên             | URL                                                                                        |
| ---------------------- | ------------------------------------------------------------------------------------------ |
| Tài liệu Sản phẩm      | https://learn.microsoft.com/azure/ai-studio/                                               |
| Tài liệu tham khảo API | https://learn.microsoft.com/rest/api/aifoundry/aiprojects/                                 |
| GitHub Source          | https://github.com/Azure/azure-sdk-for-java/tree/main/sdk/ai/azure-ai-projects             |
| Samples                | https://github.com/Azure/azure-sdk-for-java/tree/main/sdk/ai/azure-ai-projects/src/samples |
