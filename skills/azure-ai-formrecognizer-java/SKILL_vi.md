---
name: azure-ai-formrecognizer-java
description: Xây dựng các ứng dụng phân tích tài liệu với Azure Document Intelligence (Form Recognizer) SDK cho Java. Sử dụng khi trích xuất văn bản, bảng, cặp khóa-giá trị từ tài liệu, biên lai, hóa đơn, hoặc xây dựng các mô hình tài liệu tùy chỉnh.
package: com.azure:azure-ai-formrecognizer
---

# Azure Document Intelligence (Form Recognizer) SDK cho Java

Xây dựng các ứng dụng phân tích tài liệu sử dụng Azure AI Document Intelligence SDK cho Java.

## Cài đặt

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-ai-formrecognizer</artifactId>
    <version>4.2.0-beta.1</version>
</dependency>
```

## Tạo Client

### DocumentAnalysisClient

```java
import com.azure.ai.formrecognizer.documentanalysis.DocumentAnalysisClient;
import com.azure.ai.formrecognizer.documentanalysis.DocumentAnalysisClientBuilder;
import com.azure.core.credential.AzureKeyCredential;

DocumentAnalysisClient client = new DocumentAnalysisClientBuilder()
    .credential(new AzureKeyCredential("{key}"))
    .endpoint("{endpoint}")
    .buildClient();
```

### DocumentModelAdministrationClient

```java
import com.azure.ai.formrecognizer.documentanalysis.administration.DocumentModelAdministrationClient;
import com.azure.ai.formrecognizer.documentanalysis.administration.DocumentModelAdministrationClientBuilder;

DocumentModelAdministrationClient adminClient = new DocumentModelAdministrationClientBuilder()
    .credential(new AzureKeyCredential("{key}"))
    .endpoint("{endpoint}")
    .buildClient();
```

### Với DefaultAzureCredential

```java
import com.azure.identity.DefaultAzureCredentialBuilder;

DocumentAnalysisClient client = new DocumentAnalysisClientBuilder()
    .endpoint("{endpoint}")
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildClient();
```

## Các Mô hình Xây dựng sẵn

| ID Mô hình              | Mục đích                                |
| ----------------------- | --------------------------------------- |
| `prebuilt-layout`       | Trích xuất văn bản, bảng, dấu chọn      |
| `prebuilt-document`     | Tài liệu chung với các cặp khóa-giá trị |
| `prebuilt-receipt`      | Trích xuất dữ liệu biên lai             |
| `prebuilt-invoice`      | Trích xuất trường hóa đơn               |
| `prebuilt-businessCard` | Phân tích danh thiếp                    |
| `prebuilt-idDocument`   | Tài liệu ID (hộ chiếu, bằng lái)        |
| `prebuilt-tax.us.w2`    | Biểu mẫu thuế W2 của Mỹ                 |

## Các Mẫu Cốt lõi

### Trích xuất Layout

```java
import com.azure.ai.formrecognizer.documentanalysis.models.*;
import com.azure.core.util.BinaryData;
import com.azure.core.util.polling.SyncPoller;
import java.io.File;

File document = new File("document.pdf");
BinaryData documentData = BinaryData.fromFile(document.toPath());

SyncPoller<OperationResult, AnalyzeResult> poller =
    client.beginAnalyzeDocument("prebuilt-layout", documentData);

AnalyzeResult result = poller.getFinalResult();

// Xử lý các trang
for (DocumentPage page : result.getPages()) {
    System.out.printf("Trang %d: %.2f x %.2f %s%n",
        page.getPageNumber(),
        page.getWidth(),
        page.getHeight(),
        page.getUnit());

    // Các dòng
    for (DocumentLine line : page.getLines()) {
        System.out.println("Dòng: " + line.getContent());
    }

    // Dấu chọn (checkboxes)
    for (DocumentSelectionMark mark : page.getSelectionMarks()) {
        System.out.printf("Checkbox: %s (độ tin cậy: %.2f)%n",
            mark.getSelectionMarkState(),
            mark.getConfidence());
    }
}

// Các bảng
for (DocumentTable table : result.getTables()) {
    System.out.printf("Bảng: %d hàng x %d cột%n",
        table.getRowCount(),
        table.getColumnCount());

    for (DocumentTableCell cell : table.getCells()) {
        System.out.printf("Ô[%d,%d]: %s%n",
            cell.getRowIndex(),
            cell.getColumnIndex(),
            cell.getContent());
    }
}
```

### Phân tích từ URL

```java
String documentUrl = "https://example.com/invoice.pdf";

SyncPoller<OperationResult, AnalyzeResult> poller =
    client.beginAnalyzeDocumentFromUrl("prebuilt-invoice", documentUrl);

AnalyzeResult result = poller.getFinalResult();
```

### Phân tích Biên lai

```java
SyncPoller<OperationResult, AnalyzeResult> poller =
    client.beginAnalyzeDocumentFromUrl("prebuilt-receipt", receiptUrl);

AnalyzeResult result = poller.getFinalResult();

for (AnalyzedDocument doc : result.getDocuments()) {
    Map<String, DocumentField> fields = doc.getFields();

    DocumentField merchantName = fields.get("MerchantName");
    if (merchantName != null && merchantName.getType() == DocumentFieldType.STRING) {
        System.out.printf("Người bán: %s (độ tin cậy: %.2f)%n",
            merchantName.getValueAsString(),
            merchantName.getConfidence());
    }

    DocumentField transactionDate = fields.get("TransactionDate");
    if (transactionDate != null && transactionDate.getType() == DocumentFieldType.DATE) {
        System.out.printf("Ngày: %s%n", transactionDate.getValueAsDate());
    }

    DocumentField items = fields.get("Items");
    if (items != null && items.getType() == DocumentFieldType.LIST) {
        for (DocumentField item : items.getValueAsList()) {
            Map<String, DocumentField> itemFields = item.getValueAsMap();
            System.out.printf("Mặt hàng: %s, Giá: %.2f%n",
                itemFields.get("Name").getValueAsString(),
                itemFields.get("Price").getValueAsDouble());
        }
    }
}
```

### Phân tích Tài liệu Chung

```java
SyncPoller<OperationResult, AnalyzeResult> poller =
    client.beginAnalyzeDocumentFromUrl("prebuilt-document", documentUrl);

AnalyzeResult result = poller.getFinalResult();

// Cặp khóa-giá trị
for (DocumentKeyValuePair kvp : result.getKeyValuePairs()) {
    System.out.printf("Khóa: %s => Giá trị: %s%n",
        kvp.getKey().getContent(),
        kvp.getValue() != null ? kvp.getValue().getContent() : "null");
}
```

## Các Mô hình Tùy chỉnh

### Xây dựng Mô hình Tùy chỉnh

```java
import com.azure.ai.formrecognizer.documentanalysis.administration.models.*;

String blobContainerUrl = "{SAS_URL_of_training_data}";
String prefix = "training-docs/";

SyncPoller<OperationResult, DocumentModelDetails> poller = adminClient.beginBuildDocumentModel(
    blobContainerUrl,
    DocumentModelBuildMode.TEMPLATE,
    prefix,
    new BuildDocumentModelOptions()
        .setModelId("my-custom-model")
        .setDescription("Mô hình hóa đơn tùy chỉnh"),
    Context.NONE);

DocumentModelDetails model = poller.getFinalResult();

System.out.println("ID Mô hình: " + model.getModelId());
System.out.println("Đã tạo: " + model.getCreatedOn());

model.getDocumentTypes().forEach((docType, details) -> {
    System.out.println("Loại tài liệu: " + docType);
    details.getFieldSchema().forEach((field, schema) -> {
        System.out.printf("  Trường: %s (%s)%n", field, schema.getType());
    });
});
```

### Phân tích với Mô hình Tùy chỉnh

```java
SyncPoller<OperationResult, AnalyzeResult> poller =
    client.beginAnalyzeDocumentFromUrl("my-custom-model", documentUrl);

AnalyzeResult result = poller.getFinalResult();

for (AnalyzedDocument doc : result.getDocuments()) {
    System.out.printf("Loại tài liệu: %s (độ tin cậy: %.2f)%n",
        doc.getDocType(),
        doc.getConfidence());

    doc.getFields().forEach((name, field) -> {
        System.out.printf("Trường '%s': %s (độ tin cậy: %.2f)%n",
            name,
            field.getContent(),
            field.getConfidence());
    });
}
```

### Soạn thảo Mô hình (Compose Models)

```java
List<String> modelIds = Arrays.asList("model-1", "model-2", "model-3");

SyncPoller<OperationResult, DocumentModelDetails> poller =
    adminClient.beginComposeDocumentModel(
        modelIds,
        new ComposeDocumentModelOptions()
            .setModelId("composed-model")
            .setDescription("Được soạn từ nhiều mô hình"));

DocumentModelDetails composedModel = poller.getFinalResult();
```

### Quản lý Mô hình

```java
// Liệt kê các mô hình
PagedIterable<DocumentModelSummary> models = adminClient.listDocumentModels();
for (DocumentModelSummary summary : models) {
    System.out.printf("Mô hình: %s, Đã tạo: %s%n",
        summary.getModelId(),
        summary.getCreatedOn());
}

// Lấy chi tiết mô hình
DocumentModelDetails model = adminClient.getDocumentModel("model-id");

// Xóa mô hình
adminClient.deleteDocumentModel("model-id");

// Kiểm tra giới hạn tài nguyên
ResourceDetails resources = adminClient.getResourceDetails();
System.out.printf("Mô hình: %d / %d%n",
    resources.getCustomDocumentModelCount(),
    resources.getCustomDocumentModelLimit());
```

## Phân loại Tài liệu

### Xây dựng Bộ phân loại

```java
Map<String, ClassifierDocumentTypeDetails> docTypes = new HashMap<>();
docTypes.put("invoice", new ClassifierDocumentTypeDetails()
    .setAzureBlobSource(new AzureBlobContentSource(containerUrl).setPrefix("invoices/")));
docTypes.put("receipt", new ClassifierDocumentTypeDetails()
    .setAzureBlobSource(new AzureBlobContentSource(containerUrl).setPrefix("receipts/")));

SyncPoller<OperationResult, DocumentClassifierDetails> poller =
    adminClient.beginBuildDocumentClassifier(docTypes,
        new BuildDocumentClassifierOptions().setClassifierId("my-classifier"));

DocumentClassifierDetails classifier = poller.getFinalResult();
```

### Phân loại Tài liệu

```java
SyncPoller<OperationResult, AnalyzeResult> poller =
    client.beginClassifyDocumentFromUrl("my-classifier", documentUrl, Context.NONE);

AnalyzeResult result = poller.getFinalResult();

for (AnalyzedDocument doc : result.getDocuments()) {
    System.out.printf("Được phân loại là: %s (độ tin cậy: %.2f)%n",
        doc.getDocType(),
        doc.getConfidence());
}
```

## Xử lý Lỗi

```java
import com.azure.core.exception.HttpResponseException;

try {
    client.beginAnalyzeDocumentFromUrl("prebuilt-receipt", "invalid-url");
} catch (HttpResponseException e) {
    System.out.println("Trạng thái: " + e.getResponse().getStatusCode());
    System.out.println("Lỗi: " + e.getMessage());
}
```

## Biến Môi trường

```bash
FORM_RECOGNIZER_ENDPOINT=https://<resource>.cognitiveservices.azure.com/
FORM_RECOGNIZER_KEY=<your-api-key>
```

## Các Cụm từ Kích hoạt

- "document intelligence Java"
- "form recognizer SDK"
- "trích xuất văn bản từ PDF"
- "OCR tài liệu Java"
- "phân tích hóa đơn biên lai"
- "mô hình tài liệu tùy chỉnh"
- "phân loại tài liệu"
