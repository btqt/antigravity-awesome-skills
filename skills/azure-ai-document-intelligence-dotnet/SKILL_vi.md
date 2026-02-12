---
name: azure-ai-document-intelligence-dotnet
description: |
  Azure AI Document Intelligence SDK cho .NET. Trích xuất văn bản, bảng và dữ liệu có cấu trúc từ tài liệu sử dụng các mô hình xây dựng sẵn và tùy chỉnh. Sử dụng cho xử lý hóa đơn, trích xuất biên lai, phân tích tài liệu ID và các mô hình tài liệu tùy chỉnh. Kích hoạt: "Document Intelligence", "DocumentIntelligenceClient", "form recognizer", "trích xuất hóa đơn", "OCR biên lai", "phân tích tài liệu .NET".
package: Azure.AI.DocumentIntelligence
---

# Azure.AI.DocumentIntelligence (.NET)

Trích xuất văn bản, bảng và dữ liệu có cấu trúc từ tài liệu sử dụng các mô hình xây dựng sẵn và tùy chỉnh.

## Cài đặt

```bash
dotnet add package Azure.AI.DocumentIntelligence
dotnet add package Azure.Identity
```

**Phiên bản Hiện tại**: v1.0.0 (GA)

## Biến Môi trường

```bash
DOCUMENT_INTELLIGENCE_ENDPOINT=https://<resource-name>.cognitiveservices.azure.com/
DOCUMENT_INTELLIGENCE_API_KEY=<your-api-key>
BLOB_CONTAINER_SAS_URL=https://<storage>.blob.core.windows.net/<container>?<sas-token>
```

## Xác thực

### Microsoft Entra ID (Khuyên dùng)

```csharp
using Azure.Identity;
using Azure.AI.DocumentIntelligence;

string endpoint = Environment.GetEnvironmentVariable("DOCUMENT_INTELLIGENCE_ENDPOINT");
var credential = new DefaultAzureCredential();
var client = new DocumentIntelligenceClient(new Uri(endpoint), credential);
```

> **Lưu ý**: Entra ID yêu cầu một **tên miền phụ tùy chỉnh** (ví dụ: `https://<resource-name>.cognitiveservices.azure.com/`), không phải là endpoint khu vực.

### API Key

```csharp
string endpoint = Environment.GetEnvironmentVariable("DOCUMENT_INTELLIGENCE_ENDPOINT");
string apiKey = Environment.GetEnvironmentVariable("DOCUMENT_INTELLIGENCE_API_KEY");
var client = new DocumentIntelligenceClient(new Uri(endpoint), new AzureKeyCredential(apiKey));
```

## Các Loại Client

| Client                                     | Mục đích                                               |
| ------------------------------------------ | ------------------------------------------------------ |
| `DocumentIntelligenceClient`               | Phân tích tài liệu, phân loại tài liệu                 |
| `DocumentIntelligenceAdministrationClient` | Xây dựng/quản lý các mô hình tùy chỉnh và bộ phân loại |

## Các Mô hình Xây dựng sẵn

| ID Mô hình                        | Mô tả                                                             |
| --------------------------------- | ----------------------------------------------------------------- |
| `prebuilt-read`                   | Trích xuất văn bản, ngôn ngữ, chữ viết tay                        |
| `prebuilt-layout`                 | Trích xuất văn bản, bảng, dấu chọn, cấu trúc                      |
| `prebuilt-invoice`                | Trích xuất các trường hóa đơn (nhà cung cấp, mặt hàng, tổng cộng) |
| `prebuilt-receipt`                | Trích xuất các trường biên lai (người bán, mặt hàng, tổng cộng)   |
| `prebuilt-idDocument`             | Trích xuất các trường tài liệu ID (tên, ngày sinh, địa chỉ)       |
| `prebuilt-businessCard`           | Trích xuất các trường danh thiếp                                  |
| `prebuilt-tax.us.w2`              | Trích xuất các trường biểu mẫu thuế W-2                           |
| `prebuilt-healthInsuranceCard.us` | Trích xuất các trường thẻ bảo hiểm y tế                           |

## Quy trình Cốt lõi

### 1. Phân tích Hóa đơn

```csharp
using Azure.AI.DocumentIntelligence;

Uri invoiceUri = new Uri("https://example.com/invoice.pdf");

Operation<AnalyzeResult> operation = await client.AnalyzeDocumentAsync(
    WaitUntil.Completed,
    "prebuilt-invoice",
    invoiceUri);

AnalyzeResult result = operation.Value;

foreach (AnalyzedDocument document in result.Documents)
{
    if (document.Fields.TryGetValue("VendorName", out DocumentField vendorNameField)
        && vendorNameField.FieldType == DocumentFieldType.String)
    {
        string vendorName = vendorNameField.ValueString;
        Console.WriteLine($"Tên nhà cung cấp: '{vendorName}', độ tin cậy: {vendorNameField.Confidence}");
    }

    if (document.Fields.TryGetValue("InvoiceTotal", out DocumentField invoiceTotalField)
        && invoiceTotalField.FieldType == DocumentFieldType.Currency)
    {
        CurrencyValue invoiceTotal = invoiceTotalField.ValueCurrency;
        Console.WriteLine($"Tổng hóa đơn: '{invoiceTotal.CurrencySymbol}{invoiceTotal.Amount}'");
    }

    // Trích xuất các mục dòng (line items)
    if (document.Fields.TryGetValue("Items", out DocumentField itemsField)
        && itemsField.FieldType == DocumentFieldType.List)
    {
        foreach (DocumentField item in itemsField.ValueList)
        {
            var itemFields = item.ValueDictionary;
            if (itemFields.TryGetValue("Description", out DocumentField descField))
                Console.WriteLine($"  Mặt hàng: {descField.ValueString}");
        }
    }
}
```

### 2. Trích xuất Layout (Văn bản, Bảng, Cấu trúc)

```csharp
Uri fileUri = new Uri("https://example.com/document.pdf");

Operation<AnalyzeResult> operation = await client.AnalyzeDocumentAsync(
    WaitUntil.Completed,
    "prebuilt-layout",
    fileUri);

AnalyzeResult result = operation.Value;

// Trích xuất văn bản theo trang
foreach (DocumentPage page in result.Pages)
{
    Console.WriteLine($"Trang {page.PageNumber}: {page.Lines.Count} dòng, {page.Words.Count} từ");

    foreach (DocumentLine line in page.Lines)
    {
        Console.WriteLine($"  Dòng: '{line.Content}'");
    }
}

// Trích xuất bảng
foreach (DocumentTable table in result.Tables)
{
    Console.WriteLine($"Bảng: {table.RowCount} hàng x {table.ColumnCount} cột");
    foreach (DocumentTableCell cell in table.Cells)
    {
        Console.WriteLine($"  Ô ({cell.RowIndex}, {cell.ColumnIndex}): {cell.Content}");
    }
}
```

### 3. Phân tích Biên lai

```csharp
Operation<AnalyzeResult> operation = await client.AnalyzeDocumentAsync(
    WaitUntil.Completed,
    "prebuilt-receipt",
    receiptUri);

AnalyzeResult result = operation.Value;

foreach (AnalyzedDocument document in result.Documents)
{
    if (document.Fields.TryGetValue("MerchantName", out DocumentField merchantField))
        Console.WriteLine($"Người bán: {merchantField.ValueString}");

    if (document.Fields.TryGetValue("Total", out DocumentField totalField))
        Console.WriteLine($"Tổng cộng: {totalField.ValueCurrency.Amount}");

    if (document.Fields.TryGetValue("TransactionDate", out DocumentField dateField))
        Console.WriteLine($"Ngày: {dateField.ValueDate}");
}
```

### 4. Xây dựng Mô hình Tùy chỉnh

```csharp
var adminClient = new DocumentIntelligenceAdministrationClient(
    new Uri(endpoint),
    new AzureKeyCredential(apiKey));

string modelId = "my-custom-model";
Uri blobContainerUri = new Uri("<blob-container-sas-url>");

var blobSource = new BlobContentSource(blobContainerUri);
var options = new BuildDocumentModelOptions(modelId, DocumentBuildMode.Template, blobSource);

Operation<DocumentModelDetails> operation = await adminClient.BuildDocumentModelAsync(
    WaitUntil.Completed,
    options);

DocumentModelDetails model = operation.Value;

Console.WriteLine($"ID Mô hình: {model.ModelId}");
Console.WriteLine($"Đã tạo: {model.CreatedOn}");

foreach (var docType in model.DocumentTypes)
{
    Console.WriteLine($"Loại tài liệu: {docType.Key}");
    foreach (var field in docType.Value.FieldSchema)
    {
        Console.WriteLine($"  Trường: {field.Key}, Độ tin cậy: {docType.Value.FieldConfidence[field.Key]}");
    }
}
```

### 5. Xây dựng Bộ phân loại Tài liệu (Document Classifier)

```csharp
string classifierId = "my-classifier";
Uri blobContainerUri = new Uri("<blob-container-sas-url>");

var sourceA = new BlobContentSource(blobContainerUri) { Prefix = "TypeA/train" };
var sourceB = new BlobContentSource(blobContainerUri) { Prefix = "TypeB/train" };

var docTypes = new Dictionary<string, ClassifierDocumentTypeDetails>()
{
    { "TypeA", new ClassifierDocumentTypeDetails(sourceA) },
    { "TypeB", new ClassifierDocumentTypeDetails(sourceB) }
};

var options = new BuildClassifierOptions(classifierId, docTypes);

Operation<DocumentClassifierDetails> operation = await adminClient.BuildClassifierAsync(
    WaitUntil.Completed,
    options);

DocumentClassifierDetails classifier = operation.Value;
Console.WriteLine($"ID Bộ phân loại: {classifier.ClassifierId}");
```

### 6. Phân loại Tài liệu

```csharp
string classifierId = "my-classifier";
Uri documentUri = new Uri("https://example.com/document.pdf");

var options = new ClassifyDocumentOptions(classifierId, documentUri);

Operation<AnalyzeResult> operation = await client.ClassifyDocumentAsync(
    WaitUntil.Completed,
    options);

AnalyzeResult result = operation.Value;

foreach (AnalyzedDocument document in result.Documents)
{
    Console.WriteLine($"Loại tài liệu: {document.DocumentType}, độ tin cậy: {document.Confidence}");
}
```

### 7. Quản lý Mô hình

```csharp
// Lấy chi tiết tài nguyên
DocumentIntelligenceResourceDetails resourceDetails = await adminClient.GetResourceDetailsAsync();
Console.WriteLine($"Mô hình tùy chỉnh: {resourceDetails.CustomDocumentModels.Count}/{resourceDetails.CustomDocumentModels.Limit}");

// Lấy mô hình cụ thể
DocumentModelDetails model = await adminClient.GetModelAsync("my-model-id");
Console.WriteLine($"Mô hình: {model.ModelId}, Đã tạo: {model.CreatedOn}");

// Liệt kê các mô hình
await foreach (DocumentModelDetails modelItem in adminClient.GetModelsAsync())
{
    Console.WriteLine($"Mô hình: {modelItem.ModelId}");
}

// Xóa mô hình
await adminClient.DeleteModelAsync("my-model-id");
```

## Tài liệu tham khảo các Loại Chính

| Loại                                       | Mô tả                                          |
| ------------------------------------------ | ---------------------------------------------- |
| `DocumentIntelligenceClient`               | Client chính để phân tích                      |
| `DocumentIntelligenceAdministrationClient` | Quản lý mô hình                                |
| `AnalyzeResult`                            | Kết quả phân tích tài liệu                     |
| `AnalyzedDocument`                         | Tài liệu đơn lẻ trong kết quả                  |
| `DocumentField`                            | Trường đã trích xuất với giá trị và độ tin cậy |
| `DocumentFieldType`                        | String, Date, Number, Currency, v.v.           |
| `DocumentPage`                             | Thông tin trang (dòng, từ, dấu chọn)           |
| `DocumentTable`                            | Bảng đã trích xuất với các ô                   |
| `DocumentModelDetails`                     | Metadata mô hình tùy chỉnh                     |
| `BlobContentSource`                        | Nguồn dữ liệu huấn luyện                       |

## Chế độ Xây dựng (Build Modes)

| Chế độ                       | Trường hợp sử dụng                    |
| ---------------------------- | ------------------------------------- |
| `DocumentBuildMode.Template` | Tài liệu có bố cục cố định (biểu mẫu) |
| `DocumentBuildMode.Neural`   | Tài liệu có bố cục thay đổi           |

## Thực hành Tốt nhất

1. **Sử dụng DefaultAzureCredential** cho môi trường sản xuất
2. **Tái sử dụng các instance client** — client an toàn luồng (thread-safe)
3. **Xử lý các hoạt động chạy lâu** — Sử dụng `WaitUntil.Completed` cho đơn giản
4. **Kiểm tra độ tin cậy của trường** — Luôn xác minh thuộc tính `Confidence`
5. **Sử dụng mô hình phù hợp** — Xây dựng sẵn cho tài liệu phổ biến, tùy chỉnh cho chuyên biệt
6. **Sử dụng tên miền phụ tùy chỉnh** — Bắt buộc cho xác thực Entra ID

## Xử lý Lỗi

```csharp
using Azure;

try
{
    var operation = await client.AnalyzeDocumentAsync(
        WaitUntil.Completed,
        "prebuilt-invoice",
        documentUri);
}
catch (RequestFailedException ex)
{
    Console.WriteLine($"Lỗi: {ex.Status} - {ex.Message}");
}
```

## SDK Liên quan

| SDK                             | Mục đích                     | Cài đặt                                            |
| ------------------------------- | ---------------------------- | -------------------------------------------------- |
| `Azure.AI.DocumentIntelligence` | Phân tích tài liệu (SDK này) | `dotnet add package Azure.AI.DocumentIntelligence` |
| `Azure.AI.FormRecognizer`       | SDK cũ (đã ngừng hỗ trợ)     | Sử dụng DocumentIntelligence thay thế              |

## Liên kết Tham khảo

| Tài nguyên                   | URL                                                                                                                 |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| NuGet Package                | https://www.nuget.org/packages/Azure.AI.DocumentIntelligence                                                        |
| API Reference                | https://learn.microsoft.com/dotnet/api/azure.ai.documentintelligence                                                |
| GitHub Samples               | https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/documentintelligence/Azure.AI.DocumentIntelligence/samples |
| Document Intelligence Studio | https://documentintelligence.ai.azure.com/                                                                          |
| Prebuilt Models              | https://aka.ms/azsdk/formrecognizer/models                                                                          |
