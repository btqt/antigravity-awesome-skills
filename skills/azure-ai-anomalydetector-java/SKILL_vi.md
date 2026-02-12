---
name: azure-ai-anomalydetector-java
description: Xây dựng các ứng dụng phát hiện bất thường với Azure AI Anomaly Detector SDK cho Java. Sử dụng khi triển khai phát hiện bất thường đơn biến/đa biến, phân tích chuỗi thời gian, hoặc giám sát được hỗ trợ bởi AI.
package: com.azure:azure-ai-anomalydetector
---

# Azure AI Anomaly Detector SDK cho Java

Xây dựng các ứng dụng phát hiện bất thường sử dụng Azure AI Anomaly Detector SDK cho Java.

## Cài đặt

```xml
<dependency>
  <groupId>com.azure</groupId>
  <artifactId>azure-ai-anomalydetector</artifactId>
  <version>3.0.0-beta.6</version>
</dependency>
```

## Tạo Client

### Client Đồng bộ và Bất đồng bộ

```java
import com.azure.ai.anomalydetector.AnomalyDetectorClientBuilder;
import com.azure.ai.anomalydetector.MultivariateClient;
import com.azure.ai.anomalydetector.UnivariateClient;
import com.azure.core.credential.AzureKeyCredential;

String endpoint = System.getenv("AZURE_ANOMALY_DETECTOR_ENDPOINT");
String key = System.getenv("AZURE_ANOMALY_DETECTOR_API_KEY");

// Client đa biến cho nhiều tín hiệu tương quan
MultivariateClient multivariateClient = new AnomalyDetectorClientBuilder()
    .credential(new AzureKeyCredential(key))
    .endpoint(endpoint)
    .buildMultivariateClient();

// Client đơn biến cho phân tích biến đơn lẻ
UnivariateClient univariateClient = new AnomalyDetectorClientBuilder()
    .credential(new AzureKeyCredential(key))
    .endpoint(endpoint)
    .buildUnivariateClient();
```

### Với DefaultAzureCredential

```java
import com.azure.identity.DefaultAzureCredentialBuilder;

MultivariateClient client = new AnomalyDetectorClientBuilder()
    .credential(new DefaultAzureCredentialBuilder().build())
    .endpoint(endpoint)
    .buildMultivariateClient();
```

## Các Khái niệm Chính

### Phát hiện Bất thường Đơn biến (Univariate)

- **Phát hiện theo Lô (Batch Detection)**: Phân tích toàn bộ chuỗi thời gian cùng một lúc
- **Phát hiện theo Luồng (Streaming Detection)**: Phát hiện thời gian thực trên điểm dữ liệu mới nhất
- **Phát hiện Điểm thay đổi (Change Point Detection)**: Phát hiện thay đổi xu hướng trong chuỗi thời gian

### Phát hiện Bất thường Đa biến (Multivariate)

- Phát hiện bất thường trên hơn 300 tín hiệu tương quan
- Sử dụng Mạng Graph Attention cho các mối tương quan lẫn nhau
- Quy trình ba bước: Huấn luyện → Suy luận → Kết quả

## Các Mẫu Cốt lõi

### Phát hiện theo Lô Đơn biến

```java
import com.azure.ai.anomalydetector.models.*;
import java.time.OffsetDateTime;
import java.util.List;

List<TimeSeriesPoint> series = List.of(
    new TimeSeriesPoint(OffsetDateTime.parse("2023-01-01T00:00:00Z"), 1.0),
    new TimeSeriesPoint(OffsetDateTime.parse("2023-01-02T00:00:00Z"), 2.5),
    // ... thêm các điểm dữ liệu (yêu cầu tối thiểu 12 điểm)
);

UnivariateDetectionOptions options = new UnivariateDetectionOptions(series)
    .setGranularity(TimeGranularity.DAILY)
    .setSensitivity(95);

UnivariateEntireDetectionResult result = univariateClient.detectUnivariateEntireSeries(options);

// Kiểm tra bất thường
for (int i = 0; i < result.getIsAnomaly().size(); i++) {
    if (result.getIsAnomaly().get(i)) {
        System.out.printf("Bất thường được phát hiện tại chỉ số %d với giá trị %.2f%n",
            i, series.get(i).getValue());
    }
}
```

### Phát hiện Điểm cuối cùng Đơn biến (Streaming)

```java
UnivariateLastDetectionResult lastResult = univariateClient.detectUnivariateLastPoint(options);

if (lastResult.isAnomaly()) {
    System.out.println("Điểm mới nhất là bất thường!");
    System.out.printf("Kỳ vọng: %.2f, Cận trên: %.2f, Cận dưới: %.2f%n",
        lastResult.getExpectedValue(),
        lastResult.getUpperMargin(),
        lastResult.getLowerMargin());
}
```

### Phát hiện Điểm thay đổi

```java
UnivariateChangePointDetectionOptions changeOptions =
    new UnivariateChangePointDetectionOptions(series, TimeGranularity.DAILY);

UnivariateChangePointDetectionResult changeResult =
    univariateClient.detectUnivariateChangePoint(changeOptions);

for (int i = 0; i < changeResult.getIsChangePoint().size(); i++) {
    if (changeResult.getIsChangePoint().get(i)) {
        System.out.printf("Điểm thay đổi tại chỉ số %d với độ tin cậy %.2f%n",
            i, changeResult.getConfidenceScores().get(i));
    }
}
```

### Huấn luyện Mô hình Đa biến

```java
import com.azure.ai.anomalydetector.models.*;
import com.azure.core.util.polling.SyncPoller;

// Chuẩn bị yêu cầu huấn luyện với dữ liệu blob storage
ModelInfo modelInfo = new ModelInfo()
    .setDataSource("https://storage.blob.core.windows.net/container/data.zip?sasToken")
    .setStartTime(OffsetDateTime.parse("2023-01-01T00:00:00Z"))
    .setEndTime(OffsetDateTime.parse("2023-06-01T00:00:00Z"))
    .setSlidingWindow(200)
    .setDisplayName("MyMultivariateModel");

// Huấn luyện mô hình (hoạt động chạy lâu)
AnomalyDetectionModel trainedModel = multivariateClient.trainMultivariateModel(modelInfo);

String modelId = trainedModel.getModelId();
System.out.println("Model ID: " + modelId);

// Kiểm tra trạng thái huấn luyện
AnomalyDetectionModel model = multivariateClient.getMultivariateModel(modelId);
System.out.println("Trạng thái: " + model.getModelInfo().getStatus());
```

### Suy luận theo Lô Đa biến (Multivariate Batch Inference)

```java
MultivariateBatchDetectionOptions detectionOptions = new MultivariateBatchDetectionOptions()
    .setDataSource("https://storage.blob.core.windows.net/container/inference-data.zip?sasToken")
    .setStartTime(OffsetDateTime.parse("2023-07-01T00:00:00Z"))
    .setEndTime(OffsetDateTime.parse("2023-07-31T00:00:00Z"))
    .setTopContributorCount(10);

MultivariateDetectionResult detectionResult =
    multivariateClient.detectMultivariateBatchAnomaly(modelId, detectionOptions);

String resultId = detectionResult.getResultId();

// Poll kết quả
MultivariateDetectionResult result = multivariateClient.getBatchDetectionResult(resultId);
for (AnomalyState state : result.getResults()) {
    if (state.getValue().isAnomaly()) {
        System.out.printf("Bất thường tại %s, mức độ nghiêm trọng: %.2f%n",
            state.getTimestamp(),
            state.getValue().getSeverity());
    }
}
```

### Phát hiện Điểm cuối cùng Đa biến

```java
MultivariateLastDetectionOptions lastOptions = new MultivariateLastDetectionOptions()
    .setVariables(List.of(
        new VariableValues("variable1", List.of("timestamp1"), List.of(1.0f)),
        new VariableValues("variable2", List.of("timestamp1"), List.of(2.5f))
    ))
    .setTopContributorCount(5);

MultivariateLastDetectionResult lastResult =
    multivariateClient.detectMultivariateLastAnomaly(modelId, lastOptions);

if (lastResult.getValue().isAnomaly()) {
    System.out.println("Phát hiện bất thường!");
    // Kiểm tra các biến đóng góp
    for (AnomalyContributor contributor : lastResult.getValue().getInterpretation()) {
        System.out.printf("Biến: %s, Đóng góp: %.2f%n",
            contributor.getVariable(),
            contributor.getContributionScore());
    }
}
```

### Quản lý Mô hình

```java
// Liệt kê tất cả mô hình
PagedIterable<AnomalyDetectionModel> models = multivariateClient.listMultivariateModels();
for (AnomalyDetectionModel m : models) {
    System.out.printf("Mô hình: %s, Trạng thái: %s%n",
        m.getModelId(),
        m.getModelInfo().getStatus());
}

// Xóa một mô hình
multivariateClient.deleteMultivariateModel(modelId);
```

## Xử lý Lỗi

```java
import com.azure.core.exception.HttpResponseException;

try {
    univariateClient.detectUnivariateEntireSeries(options);
} catch (HttpResponseException e) {
    System.out.println("Mã trạng thái: " + e.getResponse().getStatusCode());
    System.out.println("Lỗi: " + e.getMessage());
}
```

## Biến Môi trường

```bash
AZURE_ANOMALY_DETECTOR_ENDPOINT=https://<resource>.cognitiveservices.azure.com/
AZURE_ANOMALY_DETECTOR_API_KEY=<your-api-key>
```

## Thực hành Tốt nhất

1. **Điểm dữ liệu tối thiểu**: Đơn biến yêu cầu ít nhất 12 điểm; nhiều dữ liệu hơn cải thiện độ chính xác
2. **Căn chỉnh độ chi tiết (Granularity)**: Khớp `TimeGranularity` với tần suất dữ liệu thực tế của bạn
3. **Điều chỉnh độ nhạy**: Giá trị cao hơn (0-99) phát hiện nhiều bất thường hơn
4. **Huấn luyện Đa biến**: Sử dụng cửa sổ trượt 200-1000 dựa trên độ phức tạp của mẫu
5. **Xử lý Lỗi**: Luôn xử lý `HttpResponseException` cho các lỗi API

## Các Cụm từ Kích hoạt

- "phát hiện bất thường Java"
- "phát hiện bất thường chuỗi thời gian"
- "bất thường đa biến Java"
- "phát hiện bất thường đơn biến"
- "phát hiện bất thường theo luồng"
- "phát hiện điểm thay đổi"
- "Azure AI Anomaly Detector"
