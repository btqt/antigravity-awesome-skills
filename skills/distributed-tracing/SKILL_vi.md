---
name: distributed-tracing
description: Triển khai theo dõi phân tán với Jaeger và Tempo để theo dõi các yêu cầu qua các vi dịch vụ và xác định các nút thắt hiệu suất. Sử dụng khi gỡ lỗi vi dịch vụ, phân tích luồng yêu cầu, hoặc triển khai khả năng quan sát cho các hệ thống phân tán.
---

# Theo Dõi Phân Tán (Distributed Tracing)

Triển khai theo dõi phân tán với Jaeger và Tempo để hiển thị luồng yêu cầu qua các vi dịch vụ.

## Không sử dụng kỹ năng này khi

- Nhiệm vụ không liên quan đến theo dõi phân tán
- Bạn cần một miền hoặc công cụ khác ngoài phạm vi này

## Hướng dẫn

- Làm rõ mục tiêu, ràng buộc và đầu vào cần thiết.
- Áp dụng các thực tiễn tốt nhất có liên quan và xác thực kết quả.
- Cung cấp các bước có thể hành động và xác minh.
- Nếu cần các ví dụ chi tiết, hãy mở `resources/implementation-playbook.md`.

## Mục đích

Theo dõi các yêu cầu qua các hệ thống phân tán để hiểu độ trễ, sự phụ thuộc và các điểm thất bại.

## Sử dụng kỹ năng này khi

- Gỡ lỗi vấn đề độ trễ
- Hiểu sự phụ thuộc dịch vụ
- Xác định nút thắt
- Theo dõi sự lan truyền lỗi
- Phân tích đường dẫn yêu cầu

## Các Khái niệm Theo dõi Phân tán

### Cấu trúc Trace

```
Trace (Request ID: abc123)
  ↓
Span (frontend) [100ms]
  ↓
Span (api-gateway) [80ms]
  ├→ Span (auth-service) [10ms]
  └→ Span (user-service) [60ms]
      └→ Span (database) [40ms]
```

### Các Thành phần Chính

- **Trace** - Hành trình yêu cầu đầu cuối
- **Span** - Một hoạt động đơn lẻ trong một trace
- **Conex** - Siêu dữ liệu được lan truyền giữa các dịch vụ
- **Tags** - Cặp key-value để lọc
- **Logs** - Các sự kiện có dấu thời gian trong một span

## Thiết lập Jaeger

### Triển khai Kubernetes

```bash
# Deploy Jaeger Operator
kubectl create namespace observability
kubectl create -f https://github.com/jaegertracing/jaeger-operator/releases/download/v1.51.0/jaeger-operator.yaml -n observability

# Deploy Jaeger instance
kubectl apply -f - <<EOF
apiVersion: jaegertracing.io/v1
kind: Jaeger
metadata:
  name: jaeger
  namespace: observability
spec:
  strategy: production
  storage:
    type: elasticsearch
    options:
      es:
        server-urls: http://elasticsearch:9200
  ingress:
    enabled: true
EOF
```

### Docker Compose

```yaml
version: "3.8"
services:
  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "5775:5775/udp"
      - "6831:6831/udp"
      - "6832:6832/udp"
      - "5778:5778"
      - "16686:16686" # UI
      - "14268:14268" # Collector
      - "14250:14250" # gRPC
      - "9411:9411" # Zipkin
    environment:
      - COLLECTOR_ZIPKIN_HOST_PORT=:9411
```

**Tham khảo:** Xem `references/jaeger-setup.md`

## Thiết bị Ứng dụng (Application Instrumentation)

### OpenTelemetry (Khuyên dùng)

#### Python (Flask)

```python
from opentelemetry import trace
from opentelemetry.exporter.jaeger.thrift import JaegerExporter
from opentelemetry.sdk.resources import SERVICE_NAME, Resource
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from flask import Flask

# Initialize tracer
resource = Resource(attributes={SERVICE_NAME: "my-service"})
provider = TracerProvider(resource=resource)
processor = BatchSpanProcessor(JaegerExporter(
    agent_host_name="jaeger",
    agent_port=6831,
))
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)

# Instrument Flask
app = Flask(__name__)
FlaskInstrumentor().instrument_app(app)

@app.route('/api/users')
def get_users():
    tracer = trace.get_tracer(__name__)

    with tracer.start_as_current_span("get_users") as span:
        span.set_attribute("user.count", 100)
        # Business logic
        users = fetch_users_from_db()
        return {"users": users}

def fetch_users_from_db():
    tracer = trace.get_tracer(__name__)

    with tracer.start_as_current_span("database_query") as span:
        span.set_attribute("db.system", "postgresql")
        span.set_attribute("db.statement", "SELECT * FROM users")
        # Database query
        return query_database()
```

#### Node.js (Express)

```javascript
const { NodeTracerProvider } = require('@opentelemetry/sdk-trace-node');
const { JaegerExporter } = require('@opentelemetry/exporter-jaeger');
const { BatchSpanProcessor } = require('@opentelemetry/sdk-trace-base');
const { registerInstrumentations } = require('@opentelemetry/instrumentation');
const { HttpInstrumentation } = require('@opentelemetry/instrumentation-http');
const { ExpressInstrumentation } = require('@opentelemetry/instrumentation-express');

# Initialize tracer
const provider = new NodeTracerProvider({
  resource: { attributes: { 'service.name': 'my-service' } }
});

const exporter = new JaegerExporter({
  endpoint: 'http://jaeger:14268/api/traces'
});

provider.addSpanProcessor(new BatchSpanProcessor(exporter));
provider.register();

# Instrument libraries
registerInstrumentations({
  instrumentations: [
    new HttpInstrumentation(),
    new ExpressInstrumentation(),
  ],
});

const express = require('express');
const app = express();

app.get('/api/users', async (req, res) => {
  const tracer = trace.getTracer('my-service');
  const span = tracer.startSpan('get_users');

  try {
    const users = await fetchUsers();
    span.setAttributes({ 'user.count': users.length });
    res.json({ users });
  } finally {
    span.end();
  }
});
```

#### Go

```go
package main

import (
    "context"
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/exporters/jaeger"
    "go.opentelemetry.io/otel/sdk/resource"
    sdktrace "go.opentelemetry.io/otel/sdk/trace"
    semconv "go.opentelemetry.io/otel/semconv/v1.4.0"
)

func initTracer() (*sdktrace.TracerProvider, error) {
    exporter, err := jaeger.New(jaeger.WithCollectorEndpoint(
        jaeger.WithEndpoint("http://jaeger:14268/api/traces"),
    ))
    if err != nil {
        return nil, err
    }

    tp := sdktrace.NewTracerProvider(
        sdktrace.WithBatcher(exporter),
        sdktrace.WithResource(resource.NewWithAttributes(
            semconv.SchemaURL,
            semconv.ServiceNameKey.String("my-service"),
        )),
    )

    otel.SetTracerProvider(tp)
    return tp, nil
}

func getUsers(ctx context.Context) ([]User, error) {
    tracer := otel.Tracer("my-service")
    ctx, span := tracer.Start(ctx, "get_users")
    defer span.End()

    span.SetAttributes(attribute.String("user.filter", "active"))

    users, err := fetchUsersFromDB(ctx)
    if err != nil {
        span.RecordError(err)
        return nil, err
    }

    span.SetAttributes(attribute.Int("user.count", len(users)))
    return users, nil
}
```

**Tham khảo:** Xem `references/instrumentation.md`

## Lan truyền Ngữ cảnh (Context Propagation)

### HTTP Headers

```
traceparent: 00-0af7651916cd43dd8448eb211c80319c-b7ad6b7169203331-01
tracestate: congo=t61rcWkgMzE
```

### Lan truyền trong Yêu cầu HTTP

#### Python

```python
from opentelemetry.propagate import inject

headers = {}
inject(headers)  # Injects trace context

response = requests.get('http://downstream-service/api', headers=headers)
```

#### Node.js

```javascript
const { propagation } = require("@opentelemetry/api");

const headers = {};
propagation.inject(context.active(), headers);

axios.get("http://downstream-service/api", { headers });
```

## Thiết lập Tempo (Grafana)

### Triển khai Kubernetes

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: tempo-config
data:
  tempo.yaml: |
    server:
      http_listen_port: 3200

    distributor:
      receivers:
        jaeger:
          protocols:
            thrift_http:
            grpc:
        otlp:
          protocols:
            http:
            grpc:

    storage:
      trace:
        backend: s3
        s3:
          bucket: tempo-traces
          endpoint: s3.amazonaws.com

    querier:
      frontend_worker:
        frontend_address: tempo-query-frontend:9095
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tempo
spec:
  replicas: 1
  template:
    spec:
      containers:
        - name: tempo
          image: grafana/tempo:latest
          args:
            - -config.file=/etc/tempo/tempo.yaml
          volumeMounts:
            - name: config
              mountPath: /etc/tempo
      volumes:
        - name: config
          configMap:
            name: tempo-config
```

**Tham khảo:** Xem `assets/jaeger-config.yaml.template`

## Chiến lược Lấy mẫu (Sampling Strategies)

### Lấy mẫu Xác suất

```yaml
# Sample 1% of traces
sampler:
  type: probabilistic
  param: 0.01
```

### Lấy mẫu Giới hạn Tốc độ

```yaml
# Sample max 100 traces per second
sampler:
  type: ratelimiting
  param: 100
```

### Lấy mẫu Thích ứng

```python
from opentelemetry.sdk.trace.sampling import ParentBased, TraceIdRatioBased

# Sample based on trace ID (deterministic)
sampler = ParentBased(root=TraceIdRatioBased(0.01))
```

## Phân tích Trace

### Tìm Yêu cầu Chậm

**Truy vấn Jaeger:**

```
service=my-service
duration > 1s
```

### Tìm Lỗi

**Truy vấn Jaeger:**

```
service=my-service
error=true
tags.http.status_code >= 500
```

### Biểu đồ Phụ thuộc Dịch vụ

Jaeger tự động tạo biểu đồ phụ thuộc dịch vụ hiển thị:

- Mối quan hệ dịch vụ
- Tỷ lệ yêu cầu
- Tỷ lệ lỗi
- Độ trễ trung bình

## Thực tiễn Tốt nhất

1. **Lấy mẫu phù hợp** (1-10% trong sản xuất)
2. **Thêm tags có ý nghĩa** (user_id, request_id)
3. **Lan truyền ngữ cảnh** qua tất cả ranh giới dịch vụ
4. **Ghi log ngoại lệ** trong các span
5. **Sử dụng đặt tên nhất quán** cho các hoạt động
6. **Giám sát chi phí tracing** (<1% tác động CPU)
7. **Thiết lập cảnh báo** cho lỗi trace
8. **Triển khai ngữ cảnh phân tán** (baggage)
9. **Sử dụng sự kiện span** cho các mốc quan trọng
10. **Tài liệu hóa tiêu chuẩn** instrumentation

## Tích hợp với Ghi log

### Logs Tương quan

```python
import logging
from opentelemetry import trace

logger = logging.getLogger(__name__)

def process_request():
    span = trace.get_current_span()
    trace_id = span.get_span_context().trace_id

    logger.info(
        "Processing request",
        extra={"trace_id": format(trace_id, '032x')}
    )
```

## Khắc phục sự cố

**Không thấy traces:**

- Kiểm tra endpoint collector
- Xác minh kết nối mạng
- Kiểm tra cấu hình lấy mẫu
- Xem lại log ứng dụng

**Chi phí độ trễ cao:**

- Giảm tỷ lệ lấy mẫu
- Sử dụng batch span processor
- Kiểm tra cấu hình exporter

## Tệp Tham khảo

- `references/jaeger-setup.md` - Cài đặt Jaeger
- `references/instrumentation.md` - Các mẫu Instrumentation
- `assets/jaeger-config.yaml.template` - Cấu hình Jaeger

## Các Kỹ năng Liên quan

- `prometheus-configuration` - Cho chỉ số
- `grafana-dashboards` - Cho trực quan hóa
- `slo-implementation` - Cho latency SLOs
