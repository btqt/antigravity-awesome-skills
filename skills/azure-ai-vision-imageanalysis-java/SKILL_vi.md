---
name: azure-ai-vision-imageanalysis-java
description: Xây dựng các ứng dụng phân tích hình ảnh với Azure AI Vision SDK cho Java. Sử dụng khi triển khai tạo chú thích hình ảnh, trích xuất văn bản OCR, phát hiện đối tượng, gắn thẻ, hoặc cắt thông minh (smart cropping).
package: com.azure:azure-ai-vision-imageanalysis
---

# Azure AI Vision Image Analysis SDK cho Java

Xây dựng các ứng dụng phân tích hình ảnh sử dụng Azure AI Vision Image Analysis SDK cho Java.

## Cài đặt

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-ai-vision-imageanalysis</artifactId>
    <version>1.1.0-beta.1</version>
</dependency>
```

## Tạo Client

### Với API Key

```java
import com.azure.ai.vision.imageanalysis.ImageAnalysisClient;
import com.azure.ai.vision.imageanalysis.ImageAnalysisClientBuilder;
import com.azure.core.credential.KeyCredential;

String endpoint = System.getenv("VISION_ENDPOINT");
String key = System.getenv("VISION_KEY");

ImageAnalysisClient client = new ImageAnalysisClientBuilder()
    .endpoint(endpoint)
    .credential(new KeyCredential(key))
    .buildClient();
```

### Client Bất đồng bộ (Async Client)

```java
import com.azure.ai.vision.imageanalysis.ImageAnalysisAsyncClient;

ImageAnalysisAsyncClient asyncClient = new ImageAnalysisClientBuilder()
    .endpoint(endpoint)
    .credential(new KeyCredential(key))
    .buildAsyncClient();
```

### Với DefaultAzureCredential

```java
import com.azure.identity.DefaultAzureCredentialBuilder;

ImageAnalysisClient client = new ImageAnalysisClientBuilder()
    .endpoint(endpoint)
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildClient();
```

## Các Tính năng Hình ảnh

| Tính năng        | Mô tả                                                 |
| ---------------- | ----------------------------------------------------- |
| `CAPTION`        | Tạo mô tả hình ảnh dễ đọc cho con người               |
| `DENSE_CAPTIONS` | Chú thích cho tối đa 10 vùng                          |
| `READ`           | OCR - Trích xuất văn bản từ hình ảnh                  |
| `TAGS`           | Gắn thẻ nội dung cho đối tượng, cảnh, hành động       |
| `OBJECTS`        | Phát hiện đối tượng với hộp giới hạn (bounding boxes) |
| `SMART_CROPS`    | Các vùng hình thu nhỏ thông minh                      |
| `PEOPLE`         | Phát hiện người với vị trí                            |

## Các Mẫu Cốt lõi

### Tạo Chú thích (Caption)

```java
import com.azure.ai.vision.imageanalysis.models.*;
import com.azure.core.util.BinaryData;
import java.io.File;
import java.util.Arrays;

// Từ tệp
BinaryData imageData = BinaryData.fromFile(new File("image.jpg").toPath());

ImageAnalysisResult result = client.analyze(
    imageData,
    Arrays.asList(VisualFeatures.CAPTION),
    new ImageAnalysisOptions().setGenderNeutralCaption(true));

System.out.printf("Chú thích: \"%s\" (độ tin cậy: %.4f)%n",
    result.getCaption().getText(),
    result.getCaption().getConfidence());
```

### Tạo Chú thích từ URL

```java
ImageAnalysisResult result = client.analyzeFromUrl(
    "https://example.com/image.jpg",
    Arrays.asList(VisualFeatures.CAPTION),
    new ImageAnalysisOptions().setGenderNeutralCaption(true));

System.out.printf("Chú thích: \"%s\"%n", result.getCaption().getText());
```

### Trích xuất Văn bản (OCR)

```java
ImageAnalysisResult result = client.analyze(
    BinaryData.fromFile(new File("document.jpg").toPath()),
    Arrays.asList(VisualFeatures.READ),
    null);

for (DetectedTextBlock block : result.getRead().getBlocks()) {
    for (DetectedTextLine line : block.getLines()) {
        System.out.printf("Dòng: '%s'%n", line.getText());
        System.out.printf("  Đa giác giới hạn: %s%n", line.getBoundingPolygon());

        for (DetectedTextWord word : line.getWords()) {
            System.out.printf("  Từ: '%s' (độ tin cậy: %.4f)%n",
                word.getText(),
                word.getConfidence());
        }
    }
}
```

### Phát hiện Đối tượng

```java
ImageAnalysisResult result = client.analyzeFromUrl(
    imageUrl,
    Arrays.asList(VisualFeatures.OBJECTS),
    null);

for (DetectedObject obj : result.getObjects()) {
    System.out.printf("Đối tượng: %s (độ tin cậy: %.4f)%n",
        obj.getTags().get(0).getName(),
        obj.getTags().get(0).getConfidence());

    ImageBoundingBox box = obj.getBoundingBox();
    System.out.printf("  Vị trí: x=%d, y=%d, w=%d, h=%d%n",
        box.getX(), box.getY(), box.getWidth(), box.getHeight());
}
```

### Lấy Thẻ (Tags)

```java
ImageAnalysisResult result = client.analyzeFromUrl(
    imageUrl,
    Arrays.asList(VisualFeatures.TAGS),
    null);

for (DetectedTag tag : result.getTags()) {
    System.out.printf("Thẻ: %s (độ tin cậy: %.4f)%n",
        tag.getName(),
        tag.getConfidence());
}
```

### Phát hiện Người

```java
ImageAnalysisResult result = client.analyzeFromUrl(
    imageUrl,
    Arrays.asList(VisualFeatures.PEOPLE),
    null);

for (DetectedPerson person : result.getPeople()) {
    ImageBoundingBox box = person.getBoundingBox();
    System.out.printf("Người tại x=%d, y=%d (độ tin cậy: %.4f)%n",
        box.getX(), box.getY(), person.getConfidence());
}
```

### Cắt Thông minh (Smart Cropping)

```java
ImageAnalysisResult result = client.analyzeFromUrl(
    imageUrl,
    Arrays.asList(VisualFeatures.SMART_CROPS),
    new ImageAnalysisOptions().setSmartCropsAspectRatios(Arrays.asList(1.0, 1.5)));

for (CropRegion crop : result.getSmartCrops()) {
    System.out.printf("Vùng cắt: tỷ lệ=%.2f, x=%d, y=%d, w=%d, h=%d%n",
        crop.getAspectRatio(),
        crop.getBoundingBox().getX(),
        crop.getBoundingBox().getY(),
        crop.getBoundingBox().getWidth(),
        crop.getBoundingBox().getHeight());
}
```

### Chú thích Dày đặc (Dense Captions)

```java
ImageAnalysisResult result = client.analyzeFromUrl(
    imageUrl,
    Arrays.asList(VisualFeatures.DENSE_CAPTIONS),
    new ImageAnalysisOptions().setGenderNeutralCaption(true));

for (DenseCaption caption : result.getDenseCaptions()) {
    System.out.printf("Chú thích: \"%s\" (độ tin cậy: %.4f)%n",
        caption.getText(),
        caption.getConfidence());
    System.out.printf("  Vùng: x=%d, y=%d, w=%d, h=%d%n",
        caption.getBoundingBox().getX(),
        caption.getBoundingBox().getY(),
        caption.getBoundingBox().getWidth(),
        caption.getBoundingBox().getHeight());
}
```

### Nhiều Tính năng

```java
ImageAnalysisResult result = client.analyzeFromUrl(
    imageUrl,
    Arrays.asList(
        VisualFeatures.CAPTION,
        VisualFeatures.TAGS,
        VisualFeatures.OBJECTS,
        VisualFeatures.READ),
    new ImageAnalysisOptions()
        .setGenderNeutralCaption(true)
        .setLanguage("en"));

// Truy cập tất cả kết quả
System.out.println("Chú thích: " + result.getCaption().getText());
System.out.println("Thẻ: " + result.getTags().size());
System.out.println("Đối tượng: " + result.getObjects().size());
System.out.println("Khối văn bản: " + result.getRead().getBlocks().size());
```

### Phân tích Bất đồng bộ

```java
asyncClient.analyzeFromUrl(
    imageUrl,
    Arrays.asList(VisualFeatures.CAPTION),
    null)
    .subscribe(
        result -> System.out.println("Chú thích: " + result.getCaption().getText()),
        error -> System.err.println("Lỗi: " + error.getMessage()),
        () -> System.out.println("Hoàn thành")
    );
```

## Xử lý Lỗi

```java
import com.azure.core.exception.HttpResponseException;

try {
    client.analyzeFromUrl(imageUrl, Arrays.asList(VisualFeatures.CAPTION), null);
} catch (HttpResponseException e) {
    System.out.println("Trạng thái: " + e.getResponse().getStatusCode());
    System.out.println("Lỗi: " + e.getMessage());
}
```

## Biến Môi trường

```bash
VISION_ENDPOINT=https://<resource>.cognitiveservices.azure.com/
VISION_KEY=<your-api-key>
```

## Yêu cầu Hình ảnh

- Định dạng: JPEG, PNG, GIF, BMP, WEBP, ICO, TIFF, MPO
- Kích thước: < 20 MB
- Kích thước pixel: 50x50 đến 16000x16000 pixels

## Tính khả dụng theo Khu vực (Regional Availability)

Caption và Dense Captions yều cầu các khu vực hỗ trợ GPU. Kiểm tra [các khu vực được hỗ trợ](https://learn.microsoft.com/azure/ai-services/computer-vision/concept-describe-images-40) trước khi triển khai.

## Các Cụm từ Kích hoạt

- "image analysis Java"
- "Azure Vision SDK"
- "tạo chú thích hình ảnh"
- "trích xuất văn bản hình ảnh OCR"
- "phát hiện đối tượng hình ảnh"
- "hình thu nhỏ smart crop"
- "phát hiện người trong hình ảnh"
