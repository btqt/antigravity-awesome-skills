---
name: azure-monitor-opentelemetry-ts
description: Thiết lập thiết bị cho các ứng dụng với Azure Monitor và OpenTelemetry cho JavaScript (@azure/monitor-opentelemetry). Sử dụng khi thêm distributed tracing, metrics, và logs vào các ứng dụng Node.js với Application Insights.
package: @azure/monitor-opentelemetry
---

# Azure Monitor OpenTelemetry SDK cho TypeScript

Tự động thiết lập thiết bị (auto-instrument) cho các ứng dụng Node.js với distributed tracing, metrics, và logs.

## Cài đặt

```bash
# Distro (khuyến nghị - auto-instrumentation)
npm install @azure/monitor-opentelemetry

# Low-level exporters (thiết lập OpenTelemetry tùy chỉnh)
npm install @azure/monitor-opentelemetry-exporter

# Custom logs ingestion
npm install @azure/monitor-ingestion
```

## Biến Môi trường

```bash
APPLICATIONINSIGHTS_CONNECTION_STRING=InstrumentationKey=...;IngestionEndpoint=...
```

## Bắt đầu Nhanh (Auto-Instrumentation)

**QUAN TRỌNG:** Gọi `useAzureMonitor()` TRƯỚC KHI import các module khác.

```typescript
import { useAzureMonitor } from "@azure/monitor-opentelemetry";

useAzureMonitor({
  azureMonitorExporterOptions: {
    connectionString: process.env.APPLICATIONINSIGHTS_CONNECTION_STRING,
  },
});

// Bây giờ import ứng dụng của bạn
import express from "express";
const app = express();
```

## Hỗ trợ ESM (Node.js 18.19+)

```bash
node --import @azure/monitor-opentelemetry/loader ./dist/index.js
```

**package.json:**

```json
{
  "scripts": {
    "start": "node --import @azure/monitor-opentelemetry/loader ./dist/index.js"
  }
}
```

## Cấu hình Đầy đủ

```typescript
import {
  useAzureMonitor,
  AzureMonitorOpenTelemetryOptions,
} from "@azure/monitor-opentelemetry";
import { resourceFromAttributes } from "@opentelemetry/resources";

const options: AzureMonitorOpenTelemetryOptions = {
  azureMonitorExporterOptions: {
    connectionString: process.env.APPLICATIONINSIGHTS_CONNECTION_STRING,
    storageDirectory: "/path/to/offline/storage",
    disableOfflineStorage: false,
  },

  // Sampling
  samplingRatio: 1.0, // 0-1, phần trăm các traces

  // Tính năng
  enableLiveMetrics: true,
  enableStandardMetrics: true,
  enablePerformanceCounters: true,

  // Các thư viện Instrumentation
  instrumentationOptions: {
    azureSdk: { enabled: true },
    http: { enabled: true },
    mongoDb: { enabled: true },
    mySql: { enabled: true },
    postgreSql: { enabled: true },
    redis: { enabled: true },
    bunyan: { enabled: false },
    winston: { enabled: false },
  },

  // Tài nguyên tùy chỉnh
  resource: resourceFromAttributes({ "service.name": "my-service" }),
};

useAzureMonitor(options);
```

## Custom Traces

```typescript
import { trace } from "@opentelemetry/api";

const tracer = trace.getTracer("my-tracer");

const span = tracer.startSpan("doWork");
try {
  span.setAttribute("component", "worker");
  span.setAttribute("operation.id", "42");
  span.addEvent("processing started");

  // Công việc của bạn ở đây
} catch (error) {
  span.recordException(error as Error);
  span.setStatus({ code: 2, message: (error as Error).message });
} finally {
  span.end();
}
```

## Custom Metrics

```typescript
import { metrics } from "@opentelemetry/api";

const meter = metrics.getMeter("my-meter");

// Counter
const counter = meter.createCounter("requests_total");
counter.add(1, { route: "/api/users", method: "GET" });

// Histogram
const histogram = meter.createHistogram("request_duration_ms");
histogram.record(150, { route: "/api/users" });

// Observable Gauge
const gauge = meter.createObservableGauge("active_connections");
gauge.addCallback((result) => {
  result.observe(getActiveConnections(), { pool: "main" });
});
```

## Thiết lập Manual Exporter

### Trace Exporter

```typescript
import { AzureMonitorTraceExporter } from "@azure/monitor-opentelemetry-exporter";
import {
  NodeTracerProvider,
  BatchSpanProcessor,
} from "@opentelemetry/sdk-trace-node";

const exporter = new AzureMonitorTraceExporter({
  connectionString: process.env.APPLICATIONINSIGHTS_CONNECTION_STRING,
});

const provider = new NodeTracerProvider({
  spanProcessors: [new BatchSpanProcessor(exporter)],
});

provider.register();
```

### Metric Exporter

```typescript
import { AzureMonitorMetricExporter } from "@azure/monitor-opentelemetry-exporter";
import {
  PeriodicExportingMetricReader,
  MeterProvider,
} from "@opentelemetry/sdk-metrics";
import { metrics } from "@opentelemetry/api";

const exporter = new AzureMonitorMetricExporter({
  connectionString: process.env.APPLICATIONINSIGHTS_CONNECTION_STRING,
});

const meterProvider = new MeterProvider({
  readers: [new PeriodicExportingMetricReader({ exporter })],
});

metrics.setGlobalMeterProvider(meterProvider);
```

### Log Exporter

```typescript
import { AzureMonitorLogExporter } from "@azure/monitor-opentelemetry-exporter";
import {
  BatchLogRecordProcessor,
  LoggerProvider,
} from "@opentelemetry/sdk-logs";
import { logs } from "@opentelemetry/api-logs";

const exporter = new AzureMonitorLogExporter({
  connectionString: process.env.APPLICATIONINSIGHTS_CONNECTION_STRING,
});

const loggerProvider = new LoggerProvider();
loggerProvider.addLogRecordProcessor(new BatchLogRecordProcessor(exporter));

logs.setGlobalLoggerProvider(loggerProvider);
```

## Custom Logs Ingestion

```typescript
import { DefaultAzureCredential } from "@azure/identity";
import {
  LogsIngestionClient,
  isAggregateLogsUploadError,
} from "@azure/monitor-ingestion";

const endpoint = "https://<dce>.ingest.monitor.azure.com";
const ruleId = "<data-collection-rule-id>";
const streamName = "Custom-MyTable_CL";

const client = new LogsIngestionClient(endpoint, new DefaultAzureCredential());

const logs = [
  {
    Time: new Date().toISOString(),
    Computer: "Server1",
    Message: "Application started",
    Level: "Information",
  },
];

try {
  await client.upload(ruleId, streamName, logs);
} catch (error) {
  if (isAggregateLogsUploadError(error)) {
    for (const uploadError of error.errors) {
      console.error("Failed logs:", uploadError.failedLogs);
    }
  }
}
```

## Custom Span Processor

```typescript
import { SpanProcessor, ReadableSpan } from "@opentelemetry/sdk-trace-base";
import { Span, Context, SpanKind, TraceFlags } from "@opentelemetry/api";
import { useAzureMonitor } from "@azure/monitor-opentelemetry";

class FilteringSpanProcessor implements SpanProcessor {
  forceFlush(): Promise<void> {
    return Promise.resolve();
  }
  shutdown(): Promise<void> {
    return Promise.resolve();
  }
  onStart(span: Span, context: Context): void {}

  onEnd(span: ReadableSpan): void {
    // Thêm custom attributes
    span.attributes["CustomDimension"] = "value";

    // Lọc bỏ các internal spans
    if (span.kind === SpanKind.INTERNAL) {
      span.spanContext().traceFlags = TraceFlags.NONE;
    }
  }
}

useAzureMonitor({
  spanProcessors: [new FilteringSpanProcessor()],
});
```

## Sampling

```typescript
import { ApplicationInsightsSampler } from "@azure/monitor-opentelemetry-exporter";
import { NodeTracerProvider } from "@opentelemetry/sdk-trace-node";

// Lấy mẫu 75% các traces
const sampler = new ApplicationInsightsSampler(0.75);

const provider = new NodeTracerProvider({ sampler });
```

## Tắt (Shutdown)

```typescript
import {
  useAzureMonitor,
  shutdownAzureMonitor,
} from "@azure/monitor-opentelemetry";

useAzureMonitor();

// Khi ứng dụng tắt
process.on("SIGTERM", async () => {
  await shutdownAzureMonitor();
  process.exit(0);
});
```

## Các Loại Chính

```typescript
import {
  useAzureMonitor,
  shutdownAzureMonitor,
  AzureMonitorOpenTelemetryOptions,
  InstrumentationOptions,
} from "@azure/monitor-opentelemetry";

import {
  AzureMonitorTraceExporter,
  AzureMonitorMetricExporter,
  AzureMonitorLogExporter,
  ApplicationInsightsSampler,
  AzureMonitorExporterOptions,
} from "@azure/monitor-opentelemetry-exporter";

import {
  LogsIngestionClient,
  isAggregateLogsUploadError,
} from "@azure/monitor-ingestion";
```

## Thực hành Tốt nhất

1. **Gọi useAzureMonitor() đầu tiên** - Trước khi import các module khác
2. **Sử dụng ESM loader cho các dự án ESM** - `--import @azure/monitor-opentelemetry/loader`
3. **Bật lưu trữ offline** - Cho telemetry đáng tin cậy trong các tình huống mất kết nối
4. **Thiết lập tỷ lệ lấy mẫu** - Cho các ứng dụng có lưu lượng truy cập cao
5. **Thêm các khía cạnh (dimensions) tùy chỉnh** - Sử dụng span processors để làm giàu dữ liệu
6. **Graceful shutdown** - Gọi `shutdownAzureMonitor()` để đẩy hết telemetry đi
