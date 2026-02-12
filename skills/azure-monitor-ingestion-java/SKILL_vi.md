---
name: azure-monitor-ingestion-java
description: |
  Azure Monitor Ingestion SDK cho Java. Gửi các nhật ký tùy chỉnh (custom logs) đến Azure Monitor thông qua Data Collection Rules (DCR) và Data Collection Endpoints (DCE).
  Triggers: "LogsIngestionClient java", "azure monitor ingestion java", "custom logs java", "DCR java", "data collection rule java".
package: com.azure:azure-monitor-ingestion
---

# Azure Monitor Ingestion SDK cho Java

Thư viện client để gửi các nhật ký tùy chỉnh đến Azure Monitor sử dụng Logs Ingestion API thông qua Data Collection Rules.

## Cài đặt

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-monitor-ingestion</artifactId>
    <version>1.2.11</version>
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
        <artifactId>azure-monitor-ingestion</artifactId>
    </dependency>
</dependencies>
```

## Điều kiện Tiên quyết

- Data Collection Endpoint (DCE)
- Data Collection Rule (DCR)
- Log Analytics workspace
- Bảng đích (custom hoặc built-in: CommonSecurityLog, SecurityEvents, Syslog, WindowsEvents)

## Biến Môi trường

```bash
DATA_COLLECTION_ENDPOINT=https://<dce-name>.<region>.ingest.monitor.azure.com
DATA_COLLECTION_RULE_ID=dcr-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
STREAM_NAME=Custom-MyTable_CL
```

## Tạo Client

### Synchronous Client

```java
import com.azure.identity.DefaultAzureCredential;
import com.azure.identity.DefaultAzureCredentialBuilder;
import com.azure.monitor.ingestion.LogsIngestionClient;
import com.azure.monitor.ingestion.LogsIngestionClientBuilder;

DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();

LogsIngestionClient client = new LogsIngestionClientBuilder()
    .endpoint("<data-collection-endpoint>")
    .credential(credential)
    .buildClient();
```

### Asynchronous Client

```java
import com.azure.monitor.ingestion.LogsIngestionAsyncClient;

LogsIngestionAsyncClient asyncClient = new LogsIngestionClientBuilder()
    .endpoint("<data-collection-endpoint>")
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildAsyncClient();
```

## Các Khái niệm Chính

| Khái niệm                      | Mô tả                                                    |
| ------------------------------ | -------------------------------------------------------- |
| Data Collection Endpoint (DCE) | URL endpoint ingestion cho vùng của bạn                  |
| Data Collection Rule (DCR)     | Định nghĩa chuyển đổi dữ liệu và định tuyến đến các bảng |
| Stream Name                    | Luồng đích trong DCR (ví dụ: `Custom-MyTable_CL`)        |
| Log Analytics Workspace        | Điểm đến cho các logs được nhập vào                      |

## Các Thao tác Cốt lõi

### Upload Custom Logs

```java
import java.util.List;
import java.util.ArrayList;

List<Object> logs = new ArrayList<>();
logs.add(new MyLogEntry("2024-01-15T10:30:00Z", "INFO", "Application started"));
logs.add(new MyLogEntry("2024-01-15T10:30:05Z", "DEBUG", "Processing request"));

client.upload("<data-collection-rule-id>", "<stream-name>", logs);
System.out.println("Logs uploaded successfully");
```

### Upload với Concurrency

Đối với các bộ sưu tập logs lớn, kích hoạt concurrent uploads:

```java
import com.azure.monitor.ingestion.models.LogsUploadOptions;
import com.azure.core.util.Context;

List<Object> logs = getLargeLogs(); // Bộ sưu tập lớn

LogsUploadOptions options = new LogsUploadOptions()
    .setMaxConcurrency(3);

client.upload("<data-collection-rule-id>", "<stream-name>", logs, options, Context.NONE);
```

### Upload với Xử lý Lỗi

Xử lý các lỗi upload một phần một cách nhẹ nhàng:

```java
LogsUploadOptions options = new LogsUploadOptions()
    .setLogsUploadErrorConsumer(uploadError -> {
        System.err.println("Upload error: " + uploadError.getResponseException().getMessage());
        System.err.println("Failed logs count: " + uploadError.getFailedLogs().size());

        // Option 1: Log và tiếp tục
        // Option 2: Throw để hủy các upload còn lại
        // throw uploadError.getResponseException();
    });

client.upload("<data-collection-rule-id>", "<stream-name>", logs, options, Context.NONE);
```

### Async Upload với Reactor

```java
import reactor.core.publisher.Mono;

List<Object> logs = getLogs();

asyncClient.upload("<data-collection-rule-id>", "<stream-name>", logs)
    .doOnSuccess(v -> System.out.println("Upload completed"))
    .doOnError(e -> System.err.println("Upload failed: " + e.getMessage()))
    .subscribe();
```

## Ví dụ Model Log Entry

```java
public class MyLogEntry {
    private String timeGenerated;
    private String level;
    private String message;

    public MyLogEntry(String timeGenerated, String level, String message) {
        this.timeGenerated = timeGenerated;
        this.level = level;
        this.message = message;
    }

    // Getters bắt buộc cho JSON serialization
    public String getTimeGenerated() { return timeGenerated; }
    public String getLevel() { return level; }
    public String getMessage() { return message; }
}
```

## Xử lý Lỗi

```java
import com.azure.core.exception.HttpResponseException;

try {
    client.upload(ruleId, streamName, logs);
} catch (HttpResponseException e) {
    System.err.println("HTTP Status: " + e.getResponse().getStatusCode());
    System.err.println("Error: " + e.getMessage());

    if (e.getResponse().getStatusCode() == 403) {
        System.err.println("Check DCR permissions and managed identity");
    } else if (e.getResponse().getStatusCode() == 404) {
        System.err.println("Verify DCE endpoint and DCR ID");
    }
}
```

## Thực hành Tốt nhất

1. **Batch logs** — Upload theo lô thay vì từng cái một
2. **Sử dụng concurrency** — Thiết lập `maxConcurrency` cho các upload lớn
3. **Xử lý lỗi một phần** — Sử dụng error consumer để log các mục nhập thất bại
4. **Khớp schema DCR** — Các trường log entry phải khớp với kỳ vọng chuyển đổi của DCR
5. **Bao gồm TimeGenerated** — Hầu hết các bảng yêu cầu một trường timestamp
6. **Tái sử dụng client** — Tạo một lần, tái sử dụng trong suốt ứng dụng
7. **Sử dụng async cho thông lượng cao** — `LogsIngestionAsyncClient` cho các mẫu reactive

## Truy vấn Logs Đã Upload

Sử dụng [azure-monitor-query](../query/SKILL.md) để truy vấn logs đã nhập:

```java
// Xem azure-monitor-query skill để biết cách sử dụng LogsQueryClient
String query = "MyTable_CL | where TimeGenerated > ago(1h) | limit 10";
```

## Liên kết Tham khảo

| Tài nguyên        | URL                                                                                                          |
| ----------------- | ------------------------------------------------------------------------------------------------------------ |
| Maven Package     | https://central.sonatype.com/artifact/com.azure/azure-monitor-ingestion                                      |
| GitHub            | https://github.com/Azure/azure-sdk-for-java/tree/main/sdk/monitor/azure-monitor-ingestion                    |
| Tài liệu Sản phẩm | https://learn.microsoft.com/azure/azure-monitor/logs/logs-ingestion-api-overview                             |
| Tổng quan DCE     | https://learn.microsoft.com/azure/azure-monitor/essentials/data-collection-endpoint-overview                 |
| Tổng quan DCR     | https://learn.microsoft.com/azure/azure-monitor/essentials/data-collection-rule-overview                     |
| Khắc phục sự cố   | https://github.com/Azure/azure-sdk-for-java/blob/main/sdk/monitor/azure-monitor-ingestion/TROUBLESHOOTING.md |
