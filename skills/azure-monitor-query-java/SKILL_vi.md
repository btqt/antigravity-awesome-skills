---
name: azure-monitor-query-java
description: |
  Azure Monitor Query SDK cho Java. Thực thi các truy vấn Kusto đối với Log Analytics workspaces và truy vấn số liệu (metrics) từ các tài nguyên Azure.
  Triggers: "LogsQueryClient java", "MetricsQueryClient java", "kusto query java", "log analytics java", "azure monitor query java".
  Lưu ý: Gói này đã ngừng hỗ trợ (deprecated). Hãy di chuyển sang azure-monitor-query-logs và azure-monitor-query-metrics.
package: com.azure:azure-monitor-query
---

# Azure Monitor Query SDK cho Java

> **THÔNG BÁO NGỪNG HỖ TRỢ**: Gói này đã bị ngừng hỗ trợ để chuyển sang:
>
> - `azure-monitor-query-logs` — Cho các truy vấn Log Analytics
> - `azure-monitor-query-metrics` — Cho các truy vấn metrics
>
> Xem các hướng dẫn di chuyển: [Logs Migration](https://github.com/Azure/azure-sdk-for-java/blob/main/sdk/monitor/azure-monitor-query-logs/migration-guide.md) | [Metrics Migration](https://github.com/Azure/azure-sdk-for-java/blob/main/sdk/monitor/azure-monitor-query-metrics/migration-guide.md)

Thư viện client để truy vấn Azure Monitor Logs và Metrics.

## Cài đặt

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-monitor-query</artifactId>
    <version>1.5.9</version>
</dependency>
```

Hoặc sử dụng Azure SDK BOM:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.azure</groupId>
            <artifactId>azure-sdk-bom</artifactId>
            <version>{bom_version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-monitor-query</artifactId>
    </dependency>
</dependencies>
```

## Điều kiện Tiên quyết

- Log Analytics workspace (cho các truy vấn logs)
- Tài nguyên Azure (cho các truy vấn metrics)
- TokenCredential với các quyền thích hợp

## Biến Môi trường

```bash
LOG_ANALYTICS_WORKSPACE_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
AZURE_RESOURCE_ID=/subscriptions/{sub}/resourceGroups/{rg}/providers/{provider}/{resource}
```

## Tạo Client

### LogsQueryClient (Sync)

```java
import com.azure.identity.DefaultAzureCredentialBuilder;
import com.azure.monitor.query.LogsQueryClient;
import com.azure.monitor.query.LogsQueryClientBuilder;

LogsQueryClient logsClient = new LogsQueryClientBuilder()
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildClient();
```

### LogsQueryAsyncClient

```java
import com.azure.monitor.query.LogsQueryAsyncClient;

LogsQueryAsyncClient logsAsyncClient = new LogsQueryClientBuilder()
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildAsyncClient();
```

### MetricsQueryClient (Sync)

```java
import com.azure.monitor.query.MetricsQueryClient;
import com.azure.monitor.query.MetricsQueryClientBuilder;

MetricsQueryClient metricsClient = new MetricsQueryClientBuilder()
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildClient();
```

### MetricsQueryAsyncClient

```java
import com.azure.monitor.query.MetricsQueryAsyncClient;

MetricsQueryAsyncClient metricsAsyncClient = new MetricsQueryClientBuilder()
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildAsyncClient();
```

### Cấu hình Sovereign Cloud

```java
// Azure China Cloud - Logs
LogsQueryClient logsClient = new LogsQueryClientBuilder()
    .credential(new DefaultAzureCredentialBuilder().build())
    .endpoint("https://api.loganalytics.azure.cn/v1")
    .buildClient();

// Azure China Cloud - Metrics
MetricsQueryClient metricsClient = new MetricsQueryClientBuilder()
    .credential(new DefaultAzureCredentialBuilder().build())
    .endpoint("https://management.chinacloudapi.cn")
    .buildClient();
```

## Các Khái niệm Chính

| Khái niệm         | Mô tả                                                                                  |
| ----------------- | -------------------------------------------------------------------------------------- |
| Logs              | Dữ liệu nhật ký và hiệu suất từ các tài nguyên Azure thông qua Ngôn ngữ Truy vấn Kusto |
| Metrics           | Dữ liệu chuỗi thời gian số được thu thập theo các khoảng thời gian đều đặn             |
| Workspace ID      | Định danh Log Analytics workspace                                                      |
| Resource ID       | URI tài nguyên Azure cho các truy vấn metrics                                          |
| QueryTimeInterval | Khoảng thời gian cho truy vấn                                                          |

## Các Thao tác Truy vấn Logs

### Truy vấn Cơ bản

```java
import com.azure.monitor.query.models.LogsQueryResult;
import com.azure.monitor.query.models.LogsTableRow;
import com.azure.monitor.query.models.QueryTimeInterval;
import java.time.Duration;

LogsQueryResult result = logsClient.queryWorkspace(
    "{workspace-id}",
    "AzureActivity | summarize count() by ResourceGroup | top 10 by count_",
    new QueryTimeInterval(Duration.ofDays(7))
);

for (LogsTableRow row : result.getTable().getRows()) {
    System.out.println(row.getColumnValue("ResourceGroup") + ": " + row.getColumnValue("count_"));
}
```

### Truy vấn theo Resource ID

```java
LogsQueryResult result = logsClient.queryResource(
    "{resource-id}",
    "AzureMetrics | where TimeGenerated > ago(1h)",
    new QueryTimeInterval(Duration.ofDays(1))
);

for (LogsTableRow row : result.getTable().getRows()) {
    System.out.println(row.getColumnValue("MetricName") + " " + row.getColumnValue("Average"));
}
```

### Ánh xạ Kết quả sang Model Tùy chỉnh

```java
// Định nghĩa class model
public class ActivityLog {
    private String resourceGroup;
    private String operationName;

    public String getResourceGroup() { return resourceGroup; }
    public String getOperationName() { return operationName; }
}

// Truy vấn với mapping model
List<ActivityLog> logs = logsClient.queryWorkspace(
    "{workspace-id}",
    "AzureActivity | project ResourceGroup, OperationName | take 100",
    new QueryTimeInterval(Duration.ofDays(2)),
    ActivityLog.class
);

for (ActivityLog log : logs) {
    System.out.println(log.getOperationName() + " - " + log.getResourceGroup());
}
```

### Truy vấn Theo Lô (Batch Query)

```java
import com.azure.monitor.query.models.LogsBatchQuery;
import com.azure.monitor.query.models.LogsBatchQueryResult;
import com.azure.monitor.query.models.LogsBatchQueryResultCollection;
import com.azure.core.util.Context;

LogsBatchQuery batchQuery = new LogsBatchQuery();
String q1 = batchQuery.addWorkspaceQuery("{workspace-id}", "AzureActivity | count", new QueryTimeInterval(Duration.ofDays(1)));
String q2 = batchQuery.addWorkspaceQuery("{workspace-id}", "Heartbeat | count", new QueryTimeInterval(Duration.ofDays(1)));
String q3 = batchQuery.addWorkspaceQuery("{workspace-id}", "Perf | count", new QueryTimeInterval(Duration.ofDays(1)));

LogsBatchQueryResultCollection results = logsClient
    .queryBatchWithResponse(batchQuery, Context.NONE)
    .getValue();

LogsBatchQueryResult result1 = results.getResult(q1);
LogsBatchQueryResult result2 = results.getResult(q2);
LogsBatchQueryResult result3 = results.getResult(q3);

// Kiểm tra thất bại
if (result3.getQueryResultStatus() == LogsQueryResultStatus.FAILURE) {
    System.err.println("Query failed: " + result3.getError().getMessage());
}
```

### Truy vấn với Tùy chọn

```java
import com.azure.monitor.query.models.LogsQueryOptions;
import com.azure.core.http.rest.Response;

LogsQueryOptions options = new LogsQueryOptions()
    .setServerTimeout(Duration.ofMinutes(10))
    .setIncludeStatistics(true)
    .setIncludeVisualization(true);

Response<LogsQueryResult> response = logsClient.queryWorkspaceWithResponse(
    "{workspace-id}",
    "AzureActivity | summarize count() by bin(TimeGenerated, 1h)",
    new QueryTimeInterval(Duration.ofDays(7)),
    options,
    Context.NONE
);

LogsQueryResult result = response.getValue();

// Truy cập thống kê
BinaryData statistics = result.getStatistics();
// Truy cập dữ liệu trực quan hóa
BinaryData visualization = result.getVisualization();
```

### Truy vấn Nhiều Workspaces

```java
import java.util.Arrays;

LogsQueryOptions options = new LogsQueryOptions()
    .setAdditionalWorkspaces(Arrays.asList("{workspace-id-2}", "{workspace-id-3}"));

Response<LogsQueryResult> response = logsClient.queryWorkspaceWithResponse(
    "{workspace-id-1}",
    "AzureActivity | summarize count() by TenantId",
    new QueryTimeInterval(Duration.ofDays(1)),
    options,
    Context.NONE
);
```

## Các Thao tác Truy vấn Metrics

### Truy vấn Metrics Cơ bản

```java
import com.azure.monitor.query.models.MetricsQueryResult;
import com.azure.monitor.query.models.MetricResult;
import com.azure.monitor.query.models.TimeSeriesElement;
import com.azure.monitor.query.models.MetricValue;
import java.util.Arrays;

MetricsQueryResult result = metricsClient.queryResource(
    "{resource-uri}",
    Arrays.asList("SuccessfulCalls", "TotalCalls")
);

for (MetricResult metric : result.getMetrics()) {
    System.out.println("Metric: " + metric.getMetricName());
    for (TimeSeriesElement ts : metric.getTimeSeries()) {
        System.out.println("  Dimensions: " + ts.getMetadata());
        for (MetricValue value : ts.getValues()) {
            System.out.println("    " + value.getTimeStamp() + ": " + value.getTotal());
        }
    }
}
```

### Metrics với Aggregations

```java
import com.azure.monitor.query.models.MetricsQueryOptions;
import com.azure.monitor.query.models.AggregationType;

Response<MetricsQueryResult> response = metricsClient.queryResourceWithResponse(
    "{resource-id}",
    Arrays.asList("SuccessfulCalls", "TotalCalls"),
    new MetricsQueryOptions()
        .setGranularity(Duration.ofHours(1))
        .setAggregations(Arrays.asList(AggregationType.AVERAGE, AggregationType.COUNT)),
    Context.NONE
);

MetricsQueryResult result = response.getValue();
```

### Truy vấn Nhiều Tài nguyên (MetricsClient)

```java
import com.azure.monitor.query.MetricsClient;
import com.azure.monitor.query.MetricsClientBuilder;
import com.azure.monitor.query.models.MetricsQueryResourcesResult;

MetricsClient metricsClient = new MetricsClientBuilder()
    .credential(new DefaultAzureCredentialBuilder().build())
    .endpoint("{endpoint}")
    .buildClient();

MetricsQueryResourcesResult result = metricsClient.queryResources(
    Arrays.asList("{resourceId1}", "{resourceId2}"),
    Arrays.asList("{metric1}", "{metric2}"),
    "{metricNamespace}"
);

for (MetricsQueryResult queryResult : result.getMetricsQueryResults()) {
    for (MetricResult metric : queryResult.getMetrics()) {
        System.out.println(metric.getMetricName());
        metric.getTimeSeries().stream()
            .flatMap(ts -> ts.getValues().stream())
            .forEach(mv -> System.out.println(
                mv.getTimeStamp() + " Count=" + mv.getCount() + " Avg=" + mv.getAverage()));
    }
}
```

## Cấu trúc Phản hồi

### Phân cấp Phản hồi Logs

```
LogsQueryResult
├── statistics (BinaryData)
├── visualization (BinaryData)
├── error
└── tables (List<LogsTable>)
    ├── name
    ├── columns (List<LogsTableColumn>)
    │   ├── name
    │   └── type
    └── rows (List<LogsTableRow>)
        ├── rowIndex
        └── rowCells (List<LogsTableCell>)
```

### Phân cấp Phản hồi Metrics

```
MetricsQueryResult
├── granularity
├── timeInterval
├── namespace
├── resourceRegion
└── metrics (List<MetricResult>)
    ├── id, name, type, unit
    └── timeSeries (List<TimeSeriesElement>)
        ├── metadata (dimensions)
        └── values (List<MetricValue>)
            ├── timeStamp
            ├── count, average, total
            ├── maximum, minimum
```

## Xử lý Lỗi

```java
import com.azure.core.exception.HttpResponseException;
import com.azure.monitor.query.models.LogsQueryResultStatus;

try {
    LogsQueryResult result = logsClient.queryWorkspace(workspaceId, query, timeInterval);

    // Kiểm tra thất bại một phần
    if (result.getStatus() == LogsQueryResultStatus.PARTIAL_FAILURE) {
        System.err.println("Partial failure: " + result.getError().getMessage());
    }
} catch (HttpResponseException e) {
    System.err.println("Query failed: " + e.getMessage());
    System.err.println("Status: " + e.getResponse().getStatusCode());
}
```

## Thực hành Tốt nhất

1. **Sử dụng truy vấn lô** — Kết hợp nhiều truy vấn thành một yêu cầu duy nhất
2. **Thiết lập timeouts phù hợp** — Các truy vấn dài có thể cần thời gian chờ máy chủ kéo dài
3. **Giới hạn kích thước kết quả** — Sử dụng `top` hoặc `take` trong các truy vấn Kusto
4. **Sử dụng projections** — Chọn chỉ các cột cần thiết với `project`
5. **Kiểm tra trạng thái truy vấn** — Xử lý kết quả PARTIAL_FAILURE một cách nhẹ nhàng
6. **Cache kết quả** — Metrics không thay đổi thường xuyên; cache khi thích hợp
7. **Di chuyển sang các gói mới** — Lên kế hoạch di chuyển sang `azure-monitor-query-logs` và `azure-monitor-query-metrics`

## Liên kết Tham khảo

| Tài nguyên           | URL                                                                                                      |
| -------------------- | -------------------------------------------------------------------------------------------------------- |
| Maven Package        | https://central.sonatype.com/artifact/com.azure/azure-monitor-query                                      |
| GitHub               | https://github.com/Azure/azure-sdk-for-java/tree/main/sdk/monitor/azure-monitor-query                    |
| API Reference        | https://learn.microsoft.com/java/api/com.azure.monitor.query                                             |
| Kusto Query Language | https://learn.microsoft.com/azure/data-explorer/kusto/query/                                             |
| Log Analytics Limits | https://learn.microsoft.com/azure/azure-monitor/service-limits#la-query-api                              |
| Khắc phục sự cố      | https://github.com/Azure/azure-sdk-for-java/blob/main/sdk/monitor/azure-monitor-query/TROUBLESHOOTING.md |
