---
name: azure-monitor-opentelemetry-exporter-java
description: |
  Azure Monitor OpenTelemetry Exporter cho Java. Xuất OpenTelemetry traces, metrics, và logs đến Azure Monitor/Application Insights.
  Triggers: "AzureMonitorExporter java", "opentelemetry azure java", "application insights java otel", "azure monitor tracing java".
  Lưu ý: Gói này ĐÃ NGỪNG HỖ TRỢ (DEPRECATED). Hãy di chuyển sang azure-monitor-opentelemetry-autoconfigure.
package: com.azure:azure-monitor-opentelemetry-exporter
---

# Azure Monitor OpenTelemetry Exporter cho Java

> **⚠️ THÔNG BÁO NGỪNG HỖ TRỢ**: Gói này đã bị ngừng hỗ trợ. Hãy di chuyển sang `azure-monitor-opentelemetry-autoconfigure`.
>
> Xem [Hướng dẫn Di chuyển](https://github.com/Azure/azure-sdk-for-java/blob/main/sdk/monitor/azure-monitor-opentelemetry-exporter/MIGRATION.md) để biết hướng dẫn chi tiết.

Xuất dữ liệu đo từ xa OpenTelemetry đến Azure Monitor / Application Insights.

## Cài đặt (Đã ngừng hỗ trợ)

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-monitor-opentelemetry-exporter</artifactId>
    <version>1.0.0-beta.x</version>
</dependency>
```

## Khuyến nghị: Sử dụng Autoconfigure Thay thế

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-monitor-opentelemetry-autoconfigure</artifactId>
    <version>LATEST</version>
</dependency>
```

## Biến Môi trường

```bash
APPLICATIONINSIGHTS_CONNECTION_STRING=InstrumentationKey=xxx;IngestionEndpoint=https://xxx.in.applicationinsights.azure.com/
```

## Thiết lập Cơ bản với Autoconfigure (Khuyến nghị)

### Sử dụng Biến Môi trường

```java
import io.opentelemetry.sdk.autoconfigure.AutoConfiguredOpenTelemetrySdk;
import io.opentelemetry.sdk.autoconfigure.AutoConfiguredOpenTelemetrySdkBuilder;
import io.opentelemetry.api.OpenTelemetry;
import com.azure.monitor.opentelemetry.exporter.AzureMonitorExporter;

// Connection string từ biến môi trường APPLICATIONINSIGHTS_CONNECTION_STRING
AutoConfiguredOpenTelemetrySdkBuilder sdkBuilder = AutoConfiguredOpenTelemetrySdk.builder();
AzureMonitorExporter.customize(sdkBuilder);
OpenTelemetry openTelemetry = sdkBuilder.build().getOpenTelemetrySdk();
```

### Với Connection String Tường minh

```java
AutoConfiguredOpenTelemetrySdkBuilder sdkBuilder = AutoConfiguredOpenTelemetrySdk.builder();
AzureMonitorExporter.customize(sdkBuilder, "{connection-string}");
OpenTelemetry openTelemetry = sdkBuilder.build().getOpenTelemetrySdk();
```

## Tạo Spans

```java
import io.opentelemetry.api.trace.Tracer;
import io.opentelemetry.api.trace.Span;
import io.opentelemetry.context.Scope;

// Lấy tracer
Tracer tracer = openTelemetry.getTracer("com.example.myapp");

// Tạo span
Span span = tracer.spanBuilder("myOperation").startSpan();

try (Scope scope = span.makeCurrent()) {
    // Logic ứng dụng của bạn
    doWork();
} catch (Throwable t) {
    span.recordException(t);
    throw t;
} finally {
    span.end();
}
```

## Thêm Thuộc tính Span

```java
import io.opentelemetry.api.common.AttributeKey;
import io.opentelemetry.api.common.Attributes;

Span span = tracer.spanBuilder("processOrder")
    .setAttribute("order.id", "12345")
    .setAttribute("customer.tier", "premium")
    .startSpan();

try (Scope scope = span.makeCurrent()) {
    // Thêm thuộc tính trong quá trình thực thi
    span.setAttribute("items.count", 3);
    span.setAttribute("total.amount", 99.99);

    processOrder();
} finally {
    span.end();
}
```

## Custom Span Processor

```java
import io.opentelemetry.sdk.trace.SpanProcessor;
import io.opentelemetry.sdk.trace.ReadWriteSpan;
import io.opentelemetry.sdk.trace.ReadableSpan;
import io.opentelemetry.context.Context;

private static final AttributeKey<String> CUSTOM_ATTR = AttributeKey.stringKey("custom.attribute");

SpanProcessor customProcessor = new SpanProcessor() {
    @Override
    public void onStart(Context context, ReadWriteSpan span) {
        // Thêm custom attribute vào mọi span
        span.setAttribute(CUSTOM_ATTR, "customValue");
    }

    @Override
    public boolean isStartRequired() {
        return true;
    }

    @Override
    public void onEnd(ReadableSpan span) {
        // Xử lý hậu kỳ nếu cần
    }

    @Override
    public boolean isEndRequired() {
        return false;
    }
};

// Đăng ký processor
AutoConfiguredOpenTelemetrySdkBuilder sdkBuilder = AutoConfiguredOpenTelemetrySdk.builder();
AzureMonitorExporter.customize(sdkBuilder);

sdkBuilder.addTracerProviderCustomizer(
    (sdkTracerProviderBuilder, configProperties) ->
        sdkTracerProviderBuilder.addSpanProcessor(customProcessor)
);

OpenTelemetry openTelemetry = sdkBuilder.build().getOpenTelemetrySdk();
```

## Nested Spans

```java
public void parentOperation() {
    Span parentSpan = tracer.spanBuilder("parentOperation").startSpan();
    try (Scope scope = parentSpan.makeCurrent()) {
        childOperation();
    } finally {
        parentSpan.end();
    }
}

public void childOperation() {
    // Tự động liên kết với parent thông qua Context
    Span childSpan = tracer.spanBuilder("childOperation").startSpan();
    try (Scope scope = childSpan.makeCurrent()) {
        // Công việc con
    } finally {
        childSpan.end();
    }
}
```

## Ghi lại Exceptions

```java
Span span = tracer.spanBuilder("riskyOperation").startSpan();
try (Scope scope = span.makeCurrent()) {
    performRiskyWork();
} catch (Exception e) {
    span.recordException(e);
    span.setStatus(StatusCode.ERROR, e.getMessage());
    throw e;
} finally {
    span.end();
}
```

## Metrics (thông qua OpenTelemetry)

```java
import io.opentelemetry.api.metrics.Meter;
import io.opentelemetry.api.metrics.LongCounter;
import io.opentelemetry.api.metrics.LongHistogram;

Meter meter = openTelemetry.getMeter("com.example.myapp");

// Counter
LongCounter requestCounter = meter.counterBuilder("http.requests")
    .setDescription("Total HTTP requests")
    .setUnit("requests")
    .build();

requestCounter.add(1, Attributes.of(
    AttributeKey.stringKey("http.method"), "GET",
    AttributeKey.longKey("http.status_code"), 200L
));

// Histogram
LongHistogram latencyHistogram = meter.histogramBuilder("http.latency")
    .setDescription("Request latency")
    .setUnit("ms")
    .ofLongs()
    .build();

latencyHistogram.record(150, Attributes.of(
    AttributeKey.stringKey("http.route"), "/api/users"
));
```

## Các Khái niệm Chính

| Khái niệm         | Mô tả                                                         |
| ----------------- | ------------------------------------------------------------- |
| Connection String | Chuỗi kết nối Application Insights với instrumentation key    |
| Tracer            | Tạo spans cho distributed tracing                             |
| Span              | Đại diện cho một đơn vị công việc với thời gian và thuộc tính |
| SpanProcessor     | Can thiệp vào vòng đời span để tùy chỉnh                      |
| Exporter          | Gửi dữ liệu đo từ xa đến Azure Monitor                        |

## Di chuyển sang Autoconfigure

Gói `azure-monitor-opentelemetry-autoconfigure` cung cấp:

- Tự động thiết lập thiết bị (instrumentation) cho các thư viện phổ biến
- Cấu hình đơn giản hóa
- Tích hợp tốt hơn với OpenTelemetry SDK

### Các Bước Di chuyển

1. Thay thế dependency:

   ```xml
   <!-- Xóa -->
   <dependency>
       <groupId>com.azure</groupId>
       <artifactId>azure-monitor-opentelemetry-exporter</artifactId>
   </dependency>

   <!-- Thêm -->
   <dependency>
       <groupId>com.azure</groupId>
       <artifactId>azure-monitor-opentelemetry-autoconfigure</artifactId>
   </dependency>
   ```

2. Cập nhật mã khởi tạo theo [Hướng dẫn Di chuyển](https://github.com/Azure/azure-sdk-for-java/blob/main/sdk/monitor/azure-monitor-opentelemetry-exporter/MIGRATION.md)

## Thực hành Tốt nhất

1. **Sử dụng autoconfigure** — Di chuyển sang `azure-monitor-opentelemetry-autoconfigure`
2. **Đặt tên span có ý nghĩa** — Sử dụng tên thao tác mô tả rõ ràng
3. **Thêm các thuộc tính liên quan** — Bao gồm dữ liệu ngữ cảnh để gỡ lỗi
4. **Xử lý exceptions** — Luôn ghi lại exceptions vào spans
5. **Sử dụng các quy ước ngữ nghĩa** — Tuân theo các semantic conventions của OpenTelemetry
6. **Kết thúc spans trong finally** — Đảm bảo spans luôn được kết thúc
7. **Sử dụng try-with-resources** — Quản lý phạm vi với mẫu try-with-resources

## Liên kết Tham khảo

| Tài nguyên            | URL                                                                                                                 |
| --------------------- | ------------------------------------------------------------------------------------------------------------------- |
| Maven Package         | https://central.sonatype.com/artifact/com.azure/azure-monitor-opentelemetry-exporter                                |
| GitHub                | https://github.com/Azure/azure-sdk-for-java/tree/main/sdk/monitor/azure-monitor-opentelemetry-exporter              |
| Hướng dẫn Di chuyển   | https://github.com/Azure/azure-sdk-for-java/blob/main/sdk/monitor/azure-monitor-opentelemetry-exporter/MIGRATION.md |
| Autoconfigure Package | https://central.sonatype.com/artifact/com.azure/azure-monitor-opentelemetry-autoconfigure                           |
| OpenTelemetry Java    | https://opentelemetry.io/docs/languages/java/                                                                       |
| Application Insights  | https://learn.microsoft.com/azure/azure-monitor/app/app-insights-overview                                           |
