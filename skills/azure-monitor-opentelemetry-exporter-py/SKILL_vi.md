---
name: azure-monitor-opentelemetry-exporter-py
description: |
  Azure Monitor OpenTelemetry Exporter cho Python. Sử dụng để xuất dữ liệu OpenTelemetry mức thấp đến Application Insights.
  Triggers: "azure-monitor-opentelemetry-exporter", "AzureMonitorTraceExporter", "AzureMonitorMetricExporter", "AzureMonitorLogExporter".
package: azure-monitor-opentelemetry-exporter
---

# Azure Monitor OpenTelemetry Exporter cho Python

Exporter mức thấp để gửi các traces, metrics, và logs OpenTelemetry đến Application Insights.

## Cài đặt

```bash
pip install azure-monitor-opentelemetry-exporter
```

## Biến Môi trường

```bash
APPLICATIONINSIGHTS_CONNECTION_STRING=InstrumentationKey=xxx;IngestionEndpoint=https://xxx.in.applicationinsights.azure.com/
```

## Khi nào Sử dụng

| Tình huống                                                   | Sử dụng                                          |
| ------------------------------------------------------------ | ------------------------------------------------ |
| Thiết lập nhanh, tự động hóa thiết bị (auto-instrumentation) | `azure-monitor-opentelemetry` (distro)           |
| Pipeline OpenTelemetry tùy chỉnh                             | `azure-monitor-opentelemetry-exporter` (cái này) |
| Kiểm soát chi tiết dữ liệu đo từ xa (telemetry)              | `azure-monitor-opentelemetry-exporter` (cái này) |

## Trace Exporter

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from azure.monitor.opentelemetry.exporter import AzureMonitorTraceExporter

# Tạo exporter
exporter = AzureMonitorTraceExporter(
    connection_string="InstrumentationKey=xxx;..."
)

# Cấu hình tracer provider
trace.set_tracer_provider(TracerProvider())
trace.get_tracer_provider().add_span_processor(
    BatchSpanProcessor(exporter)
)

# Sử dụng tracer
tracer = trace.get_tracer(__name__)
with tracer.start_as_current_span("my-span"):
    print("Hello, World!")
```

## Metric Exporter

```python
from opentelemetry import metrics
from opentelemetry.sdk.metrics import MeterProvider
from opentelemetry.sdk.metrics.export import PeriodicExportingMetricReader
from azure.monitor.opentelemetry.exporter import AzureMonitorMetricExporter

# Tạo exporter
exporter = AzureMonitorMetricExporter(
    connection_string="InstrumentationKey=xxx;..."
)

# Cấu hình meter provider
reader = PeriodicExportingMetricReader(exporter, export_interval_millis=60000)
metrics.set_meter_provider(MeterProvider(metric_readers=[reader]))

# Sử dụng meter
meter = metrics.get_meter(__name__)
counter = meter.create_counter("requests_total")
counter.add(1, {"route": "/api/users"})
```

## Log Exporter

```python
import logging
from opentelemetry._logs import set_logger_provider
from opentelemetry.sdk._logs import LoggerProvider, LoggingHandler
from opentelemetry.sdk._logs.export import BatchLogRecordProcessor
from azure.monitor.opentelemetry.exporter import AzureMonitorLogExporter

# Tạo exporter
exporter = AzureMonitorLogExporter(
    connection_string="InstrumentationKey=xxx;..."
)

# Cấu hình logger provider
logger_provider = LoggerProvider()
logger_provider.add_log_record_processor(BatchLogRecordProcessor(exporter))
set_logger_provider(logger_provider)

# Thêm handler vào Python logging
handler = LoggingHandler(level=logging.INFO, logger_provider=logger_provider)
logging.getLogger().addHandler(handler)

# Sử dụng logging
logger = logging.getLogger(__name__)
logger.info("This will be sent to Application Insights")
```

## Từ Biến Môi trường

Các exporter tự động đọc `APPLICATIONINSIGHTS_CONNECTION_STRING`:

```python
from azure.monitor.opentelemetry.exporter import AzureMonitorTraceExporter

# Connection string từ môi trường
exporter = AzureMonitorTraceExporter()
```

## Xác thực Azure AD

```python
from azure.identity import DefaultAzureCredential
from azure.monitor.opentelemetry.exporter import AzureMonitorTraceExporter

exporter = AzureMonitorTraceExporter(
    credential=DefaultAzureCredential()
)
```

## Sampling

Sử dụng `ApplicationInsightsSampler` để lấy mẫu nhất quán:

```python
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.sampling import ParentBasedTraceIdRatio
from azure.monitor.opentelemetry.exporter import ApplicationInsightsSampler

# Lấy mẫu 10% các traces
sampler = ApplicationInsightsSampler(sampling_ratio=0.1)

trace.set_tracer_provider(TracerProvider(sampler=sampler))
```

## Lưu trữ Offline

Cấu hình lưu trữ offline để thử lại:

```python
from azure.monitor.opentelemetry.exporter import AzureMonitorTraceExporter

exporter = AzureMonitorTraceExporter(
    connection_string="...",
    storage_directory="/path/to/storage",  # Đường dẫn lưu trữ tùy chỉnh
    disable_offline_storage=False  # Bật thử lại (mặc định)
)
```

## Vô hiệu hóa Lưu trữ Offline

```python
exporter = AzureMonitorTraceExporter(
    connection_string="...",
    disable_offline_storage=True  # Không thử lại khi thất bại
)
```

## Sovereign Clouds

```python
from azure.identity import AzureAuthorityHosts, DefaultAzureCredential
from azure.monitor.opentelemetry.exporter import AzureMonitorTraceExporter

# Azure Government
credential = DefaultAzureCredential(authority=AzureAuthorityHosts.AZURE_GOVERNMENT)
exporter = AzureMonitorTraceExporter(
    connection_string="InstrumentationKey=xxx;IngestionEndpoint=https://xxx.in.applicationinsights.azure.us/",
    credential=credential
)
```

## Các Loại Exporter

| Exporter                     | Loại Telemetry | Bảng Application Insights          |
| ---------------------------- | -------------- | ---------------------------------- |
| `AzureMonitorTraceExporter`  | Traces/Spans   | requests, dependencies, exceptions |
| `AzureMonitorMetricExporter` | Metrics        | customMetrics, performanceCounters |
| `AzureMonitorLogExporter`    | Logs           | traces, customEvents               |

## Các Tùy chọn Cấu hình

| Tham số                   | Mô tả                                  | Mặc định           |
| ------------------------- | -------------------------------------- | ------------------ |
| `connection_string`       | Connection string Application Insights | Từ biến môi trường |
| `credential`              | Azure credential cho xác thực AAD      | None               |
| `disable_offline_storage` | Vô hiệu hóa lưu trữ để thử lại         | False              |
| `storage_directory`       | Đường dẫn lưu trữ tùy chỉnh            | Thư mục Temp       |

## Thực hành Tốt nhất

1. **Sử dụng BatchSpanProcessor** cho môi trường production (không phải SimpleSpanProcessor)
2. **Sử dụng ApplicationInsightsSampler** để lấy mẫu nhất quán giữa các dịch vụ
3. **Bật lưu trữ offline** để đảm bảo độ tin cậy trong môi trường production
4. **Sử dụng xác thực AAD** thay vì instrumentation keys
5. **Thiết lập khoảng thời gian xuất** phù hợp với khối lượng công việc của bạn
6. **Sử dụng distro** (`azure-monitor-opentelemetry`) trừ khi bạn cần các custom pipeline
