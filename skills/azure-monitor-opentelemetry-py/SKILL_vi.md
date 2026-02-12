---
name: azure-monitor-opentelemetry-py
description: |
  Azure Monitor OpenTelemetry Distro cho Python. Sử dụng để thiết lập Application Insights một dòng với tự động hóa thiết bị (auto-instrumentation).
  Triggers: "azure-monitor-opentelemetry", "configure_azure_monitor", "Application Insights", "OpenTelemetry distro", "auto-instrumentation".
package: azure-monitor-opentelemetry
---

# Azure Monitor OpenTelemetry Distro cho Python

Thiết lập một dòng cho Application Insights với OpenTelemetry auto-instrumentation.

## Cài đặt

```bash
pip install azure-monitor-opentelemetry
```

## Biến Môi trường

```bash
APPLICATIONINSIGHTS_CONNECTION_STRING=InstrumentationKey=xxx;IngestionEndpoint=https://xxx.in.applicationinsights.azure.com/
```

## Bắt đầu Nhanh

```python
from azure.monitor.opentelemetry import configure_azure_monitor

# Thiết lập một dòng - đọc connection string từ môi trường
configure_azure_monitor()

# Mã ứng dụng của bạn...
```

## Cấu hình Tường minh

```python
from azure.monitor.opentelemetry import configure_azure_monitor

configure_azure_monitor(
    connection_string="InstrumentationKey=xxx;IngestionEndpoint=https://xxx.in.applicationinsights.azure.com/"
)
```

## Với Flask

```python
from flask import Flask
from azure.monitor.opentelemetry import configure_azure_monitor

configure_azure_monitor()

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello, World!"

if __name__ == "__main__":
    app.run()
```

## Với Django

```python
# settings.py
from azure.monitor.opentelemetry import configure_azure_monitor

configure_azure_monitor()

# Django settings...
```

## Với FastAPI

```python
from fastapi import FastAPI
from azure.monitor.opentelemetry import configure_azure_monitor

configure_azure_monitor()

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}
```

## Custom Traces

```python
from opentelemetry import trace
from azure.monitor.opentelemetry import configure_azure_monitor

configure_azure_monitor()

tracer = trace.get_tracer(__name__)

with tracer.start_as_current_span("my-operation") as span:
    span.set_attribute("custom.attribute", "value")
    # Thực hiện công việc...
```

## Custom Metrics

```python
from opentelemetry import metrics
from azure.monitor.opentelemetry import configure_azure_monitor

configure_azure_monitor()

meter = metrics.get_meter(__name__)
counter = meter.create_counter("my_counter")

counter.add(1, {"dimension": "value"})
```

## Custom Logs

```python
import logging
from azure.monitor.opentelemetry import configure_azure_monitor

configure_azure_monitor()

logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)

logger.info("This will appear in Application Insights")
logger.error("Errors are captured too", exc_info=True)
```

## Sampling

```python
from azure.monitor.opentelemetry import configure_azure_monitor

# Lấy mẫu 10% các requests
configure_azure_monitor(
    sampling_ratio=0.1
)
```

## Tên Cloud Role

Thiết lập tên cloud role cho Application Map:

```python
from azure.monitor.opentelemetry import configure_azure_monitor
from opentelemetry.sdk.resources import Resource, SERVICE_NAME

configure_azure_monitor(
    resource=Resource.create({SERVICE_NAME: "my-service-name"})
)
```

## Vô hiệu hóa Các Instrumentation Cụ thể

```python
from azure.monitor.opentelemetry import configure_azure_monitor

configure_azure_monitor(
    instrumentations=["flask", "requests"]  # Chỉ bật những cái này
)
```

## Bật Live Metrics

```python
from azure.monitor.opentelemetry import configure_azure_monitor

configure_azure_monitor(
    enable_live_metrics=True
)
```

## Xác thực Azure AD

```python
from azure.monitor.opentelemetry import configure_azure_monitor
from azure.identity import DefaultAzureCredential

configure_azure_monitor(
    credential=DefaultAzureCredential()
)
```

## Auto-Instrumentations Được Bao gồm

| Thư viện | Loại Telemetry |
| -------- | -------------- |
| Flask    | Traces         |
| Django   | Traces         |
| FastAPI  | Traces         |
| Requests | Traces         |
| urllib3  | Traces         |
| httpx    | Traces         |
| aiohttp  | Traces         |
| psycopg2 | Traces         |
| pymysql  | Traces         |
| pymongo  | Traces         |
| redis    | Traces         |

## Các Tùy chọn Cấu hình

| Tham số               | Mô tả                                  | Mặc định           |
| --------------------- | -------------------------------------- | ------------------ |
| `connection_string`   | Connection string Application Insights | Từ biến môi trường |
| `credential`          | Azure credential cho xác thực AAD      | None               |
| `sampling_ratio`      | Tỷ lệ lấy mẫu (0.0 đến 1.0)            | 1.0                |
| `resource`            | OpenTelemetry Resource                 | Tự động phát hiện  |
| `instrumentations`    | Danh sách các instrumentations để bật  | Tất cả             |
| `enable_live_metrics` | Bật Live Metrics stream                | False              |

## Thực hành Tốt nhất

1. **Gọi configure_azure_monitor() sớm** — Trước khi import các thư viện được đo lường
2. **Sử dụng biến môi trường** cho connection string trong môi trường production
3. **Thiết lập tên cloud role** cho các ứng dụng đa dịch vụ
4. **Bật sampling** trong các ứng dụng có lưu lượng truy cập cao
5. **Sử dụng structured logging** để truy vấn log analytics tốt hơn
6. **Thêm custom attributes** vào spans để gỡ lỗi tốt hơn
7. **Sử dụng xác thực AAD** cho tải công việc production
