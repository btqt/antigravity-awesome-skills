---
name: azure-ai-contentsafety-java
description: Xây dựng các ứng dụng kiểm duyệt nội dung với Azure AI Content Safety SDK cho Java. Sử dụng khi triển khai phân tích văn bản/hình ảnh, quản lý danh sách chặn (blocklist), hoặc phát hiện tác hại như thù ghét, bạo lực, nội dung tình dục và tự làm hại bản thân.
package: com.azure:azure-ai-contentsafety
---

# Azure AI Content Safety SDK cho Java

Xây dựng các ứng dụng kiểm duyệt nội dung sử dụng Azure AI Content Safety SDK cho Java.

## Cài đặt

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-ai-contentsafety</artifactId>
    <version>1.1.0-beta.1</version>
</dependency>
```

## Tạo Client

### Với API Key

```java
import com.azure.ai.contentsafety.ContentSafetyClient;
import com.azure.ai.contentsafety.ContentSafetyClientBuilder;
import com.azure.ai.contentsafety.BlocklistClient;
import com.azure.ai.contentsafety.BlocklistClientBuilder;
import com.azure.core.credential.KeyCredential;

String endpoint = System.getenv("CONTENT_SAFETY_ENDPOINT");
String key = System.getenv("CONTENT_SAFETY_KEY");

ContentSafetyClient contentSafetyClient = new ContentSafetyClientBuilder()
    .credential(new KeyCredential(key))
    .endpoint(endpoint)
    .buildClient();

BlocklistClient blocklistClient = new BlocklistClientBuilder()
    .credential(new KeyCredential(key))
    .endpoint(endpoint)
    .buildClient();
```

### Với DefaultAzureCredential

```java
import com.azure.identity.DefaultAzureCredentialBuilder;

ContentSafetyClient client = new ContentSafetyClientBuilder()
    .credential(new DefaultAzureCredentialBuilder().build())
    .endpoint(endpoint)
    .buildClient();
```

## Các Khái niệm Chính

### Các Danh mục Gây hại

| Danh mục               | Mô tả                                               |
| ---------------------- | --------------------------------------------------- |
| Hate (Thù ghét)        | Ngôn ngữ phân biệt đối xử dựa trên các nhóm bản sắc |
| Sexual (Tình dục)      | Nội dung tình dục, các mối quan hệ, hành vi         |
| Violence (Bạo lực)     | Tác hại vật lý, vũ khí, thương tích                 |
| Self-harm (Tự làm hại) | Tự gây thương tích, nội dung liên quan đến tự sát   |

### Các Mức độ Nghiêm trọng

- Văn bản: Thang đo 0-7 (mặc định đầu ra 0, 2, 4, 6)
- Hình ảnh: 0, 2, 4, 6 (thang đo rút gọn)

## Các Mẫu Cốt lõi

### Phân tích Văn bản

```java
import com.azure.ai.contentsafety.models.*;

AnalyzeTextResult result = contentSafetyClient.analyzeText(
    new AnalyzeTextOptions("Đây là văn bản để phân tích"));

for (TextCategoriesAnalysis category : result.getCategoriesAnalysis()) {
    System.out.printf("Danh mục: %s, Mức độ nghiêm trọng: %d%n",
        category.getCategory(),
        category.getSeverity());
}
```

### Phân tích Văn bản với Tùy chọn

```java
AnalyzeTextOptions options = new AnalyzeTextOptions("Văn bản để phân tích")
    .setCategories(Arrays.asList(
        TextCategory.HATE,
        TextCategory.VIOLENCE))
    .setOutputType(AnalyzeTextOutputType.EIGHT_SEVERITY_LEVELS);

AnalyzeTextResult result = contentSafetyClient.analyzeText(options);
```

### Phân tích Văn bản với Danh sách chặn

```java
AnalyzeTextOptions options = new AnalyzeTextOptions("Tôi g*ét bạn và muốn g*ết bạn")
    .setBlocklistNames(Arrays.asList("my-blocklist"))
    .setHaltOnBlocklistHit(true);

AnalyzeTextResult result = contentSafetyClient.analyzeText(options);

if (result.getBlocklistsMatch() != null) {
    for (TextBlocklistMatch match : result.getBlocklistsMatch()) {
        System.out.printf("Danh sách chặn: %s, Mục: %s, Văn bản: %s%n",
            match.getBlocklistName(),
            match.getBlocklistItemId(),
            match.getBlocklistItemText());
    }
}
```

### Phân tích Hình ảnh

```java
import com.azure.ai.contentsafety.models.*;
import com.azure.core.util.BinaryData;
import java.nio.file.Files;
import java.nio.file.Paths;

// Từ tệp
byte[] imageBytes = Files.readAllBytes(Paths.get("image.png"));
ContentSafetyImageData imageData = new ContentSafetyImageData()
    .setContent(BinaryData.fromBytes(imageBytes));

AnalyzeImageResult result = contentSafetyClient.analyzeImage(
    new AnalyzeImageOptions(imageData));

for (ImageCategoriesAnalysis category : result.getCategoriesAnalysis()) {
    System.out.printf("Danh mục: %s, Mức độ nghiêm trọng: %d%n",
        category.getCategory(),
        category.getSeverity());
}
```

### Phân tích Hình ảnh từ URL

```java
ContentSafetyImageData imageData = new ContentSafetyImageData()
    .setBlobUrl("https://example.com/image.jpg");

AnalyzeImageResult result = contentSafetyClient.analyzeImage(
    new AnalyzeImageOptions(imageData));
```

## Quản lý Danh sách chặn (Blocklist)

### Tạo hoặc Cập nhật Danh sách chặn

```java
import com.azure.core.http.rest.RequestOptions;
import com.azure.core.http.rest.Response;
import com.azure.core.util.BinaryData;
import java.util.Map;

Map<String, String> description = Map.of("description", "Danh sách chặn tùy chỉnh");
BinaryData resource = BinaryData.fromObject(description);

Response<BinaryData> response = blocklistClient.createOrUpdateTextBlocklistWithResponse(
    "my-blocklist", resource, new RequestOptions());

if (response.getStatusCode() == 201) {
    System.out.println("Đã tạo danh sách chặn");
} else if (response.getStatusCode() == 200) {
    System.out.println("Đã cập nhật danh sách chặn");
}
```

### Thêm Mục Chặn

```java
import com.azure.ai.contentsafety.models.*;
import java.util.Arrays;

List<TextBlocklistItem> items = Arrays.asList(
    new TextBlocklistItem("badword1").setDescription("Thuật ngữ xúc phạm"),
    new TextBlocklistItem("badword2").setDescription("Thuật ngữ khác")
);

AddOrUpdateTextBlocklistItemsResult result = blocklistClient.addOrUpdateBlocklistItems(
    "my-blocklist",
    new AddOrUpdateTextBlocklistItemsOptions(items));

for (TextBlocklistItem item : result.getBlocklistItems()) {
    System.out.printf("Đã thêm: %s (ID: %s)%n",
        item.getText(),
        item.getBlocklistItemId());
}
```

### Liệt kê Danh sách chặn

```java
PagedIterable<TextBlocklist> blocklists = blocklistClient.listTextBlocklists();

for (TextBlocklist blocklist : blocklists) {
    System.out.printf("Danh sách chặn: %s, Mô tả: %s%n",
        blocklist.getName(),
        blocklist.getDescription());
}
```

### Lấy Danh sách chặn

```java
TextBlocklist blocklist = blocklistClient.getTextBlocklist("my-blocklist");
System.out.println("Tên: " + blocklist.getName());
```

### Liệt kê Mục Chặn

```java
PagedIterable<TextBlocklistItem> items =
    blocklistClient.listTextBlocklistItems("my-blocklist");

for (TextBlocklistItem item : items) {
    System.out.printf("ID: %s, Văn bản: %s%n",
        item.getBlocklistItemId(),
        item.getText());
}
```

### Xóa Mục Chặn

```java
List<String> itemIds = Arrays.asList("item-id-1", "item-id-2");

blocklistClient.removeBlocklistItems(
    "my-blocklist",
    new RemoveTextBlocklistItemsOptions(itemIds));
```

### Xóa Danh sách chặn

```java
blocklistClient.deleteTextBlocklist("my-blocklist");
```

## Xử lý Lỗi

```java
import com.azure.core.exception.HttpResponseException;

try {
    contentSafetyClient.analyzeText(new AnalyzeTextOptions("test"));
} catch (HttpResponseException e) {
    System.out.println("Trạng thái: " + e.getResponse().getStatusCode());
    System.out.println("Lỗi: " + e.getMessage());
    // Các mã phổ biến: InvalidRequestBody, ResourceNotFound, TooManyRequests
}
```

## Biến Môi trường

```bash
CONTENT_SAFETY_ENDPOINT=https://<resource>.cognitiveservices.azure.com/
CONTENT_SAFETY_KEY=<your-api-key>
```

## Thực hành Tốt nhất

1. **Độ trễ Danh sách chặn**: Thay đổi mất ~5 phút để có hiệu lực
2. **Lựa chọn Danh mục**: Chỉ yêu cầu các danh mục cần thiết để giảm độ trễ
3. **Ngưỡng Mức độ Nghiêm trọng**: Thường chặn mức độ nghiêm trọng >= 4 để kiểm duyệt nghiêm ngặt
4. **Xử lý theo Lô**: Xử lý nhiều mục song song để tăng thông lượng
5. **Caching**: Cache kết quả danh sách chặn khi thích hợp

## Các Cụm từ Kích hoạt

- "an toàn nội dung Java"
- "kiểm duyệt nội dung Azure"
- "phân tích an toàn văn bản"
- "kiểm duyệt hình ảnh Java"
- "quản lý danh sách chặn"
- "phát hiện ngôn từ thù ghét"
- "bộ lọc nội dung độc hại"
