---
name: azure-ai-translation-document-py
description: |
  Azure AI Document Translation SDK để dịch hàng loạt tài liệu với khả năng bảo toàn định dạng. Sử dụng để dịch Word, PDF, Excel, PowerPoint, và các định dạng tài liệu khác ở quy mô lớn.
  Kích hoạt: "document translation", "batch translation", "dịch tài liệu", "DocumentTranslationClient".
package: azure-ai-translation-document
---

# Azure AI Document Translation SDK cho Python

Thư viện Client cho dịch vụ dịch tài liệu Azure AI Translator để dịch tài liệu hàng loạt với khả năng bảo toàn định dạng.

## Cài đặt

```bash
pip install azure-ai-translation-document
```

## Biến Môi trường

```bash
AZURE_DOCUMENT_TRANSLATION_ENDPOINT=https://<resource>.cognitiveservices.azure.com
AZURE_DOCUMENT_TRANSLATION_KEY=<your-api-key>  # Nếu sử dụng API key

# Storage cho tài liệu nguồn và đích
AZURE_SOURCE_CONTAINER_URL=https://<storage>.blob.core.windows.net/<container>?<sas>
AZURE_TARGET_CONTAINER_URL=https://<storage>.blob.core.windows.net/<container>?<sas>
```

## Xác thực

### API Key

```python
import os
from azure.ai.translation.document import DocumentTranslationClient
from azure.core.credentials import AzureKeyCredential

endpoint = os.environ["AZURE_DOCUMENT_TRANSLATION_ENDPOINT"]
key = os.environ["AZURE_DOCUMENT_TRANSLATION_KEY"]

client = DocumentTranslationClient(endpoint, AzureKeyCredential(key))
```

### Entra ID (Khuyên dùng)

```python
from azure.ai.translation.document import DocumentTranslationClient
from azure.identity import DefaultAzureCredential

client = DocumentTranslationClient(
    endpoint=os.environ["AZURE_DOCUMENT_TRANSLATION_ENDPOINT"],
    credential=DefaultAzureCredential()
)
```

## Dịch Tài liệu Cơ bản

```python
from azure.ai.translation.document import DocumentTranslationInput, TranslationTarget

source_url = os.environ["AZURE_SOURCE_CONTAINER_URL"]
target_url = os.environ["AZURE_TARGET_CONTAINER_URL"]

# Bắt đầu công việc dịch
poller = client.begin_translation(
    inputs=[
        DocumentTranslationInput(
            source_url=source_url,
            targets=[
                TranslationTarget(
                    target_url=target_url,
                    language="es"  # Dịch sang tiếng Tây Ban Nha
                )
            ]
        )
    ]
)

# Đợi hoàn thành
result = poller.result()

print(f"Trạng thái: {poller.status()}")
print(f"Tài liệu thành công: {poller.details.documents_succeeded_count}")
print(f"Tài liệu thất bại: {poller.details.documents_failed_count}")
```

## Nhiều Ngôn ngữ Đích

```python
poller = client.begin_translation(
    inputs=[
        DocumentTranslationInput(
            source_url=source_url,
            targets=[
                TranslationTarget(target_url=target_url_es, language="es"),
                TranslationTarget(target_url=target_url_fr, language="fr"),
                TranslationTarget(target_url=target_url_de, language="de")
            ]
        )
    ]
)
```

## Dịch Một Tài liệu

```python
from azure.ai.translation.document import SingleDocumentTranslationClient

single_client = SingleDocumentTranslationClient(endpoint, AzureKeyCredential(key))

with open("document.docx", "rb") as f:
    document_content = f.read()

result = single_client.translate(
    body=document_content,
    target_language="es",
    content_type="application/vnd.openxmlformats-officedocument.wordprocessingml.document"
)

# Lưu tài liệu đã dịch
with open("document_es.docx", "wb") as f:
    f.write(result)
```

## Kiểm tra Trạng thái Dịch

```python
# Lấy tất cả các hoạt động dịch
operations = client.list_translation_statuses()

for op in operations:
    print(f"ID Hoạt động: {op.id}")
    print(f"Trạng thái: {op.status}")
    print(f"Đã tạo: {op.created_on}")
    print(f"Tổng tài liệu: {op.documents_total_count}")
    print(f"Thành công: {op.documents_succeeded_count}")
    print(f"Thất bại: {op.documents_failed_count}")
```

## Liệt kê Trạng thái Tài liệu

```python
# Lấy trạng thái của từng tài liệu trong một công việc
operation_id = poller.id
document_statuses = client.list_document_statuses(operation_id)

for doc in document_statuses:
    print(f"Tài liệu: {doc.source_document_url}")
    print(f"  Trạng thái: {doc.status}")
    print(f"  Dịch sang: {doc.translated_to}")
    if doc.error:
        print(f"  Lỗi: {doc.error.message}")
```

## Hủy Dịch

```python
# Hủy một bản dịch đang chạy
client.cancel_translation(operation_id)
```

## Sử dụng Bảng thuật ngữ (Glossary)

```python
from azure.ai.translation.document import TranslationGlossary

poller = client.begin_translation(
    inputs=[
        DocumentTranslationInput(
            source_url=source_url,
            targets=[
                TranslationTarget(
                    target_url=target_url,
                    language="es",
                    glossaries=[
                        TranslationGlossary(
                            glossary_url="https://<storage>.blob.core.windows.net/glossary/terms.csv?<sas>",
                            file_format="csv"
                        )
                    ]
                )
            ]
        )
    ]
)
```

## Các Định dạng Tài liệu Hỗ trợ

```python
# Lấy các định dạng hỗ trợ
formats = client.get_supported_document_formats()

for fmt in formats:
    print(f"Định dạng: {fmt.format}")
    print(f"  Phần mở rộng: {fmt.file_extensions}")
    print(f"  Loại nội dung: {fmt.content_types}")
```

## Các Ngôn ngữ Hỗ trợ

```python
# Lấy các ngôn ngữ hỗ trợ
languages = client.get_supported_languages()

for lang in languages:
    print(f"Ngôn ngữ: {lang.name} ({lang.code})")
```

## Client Bất đồng bộ (Async Client)

```python
from azure.ai.translation.document.aio import DocumentTranslationClient
from azure.identity.aio import DefaultAzureCredential

async def translate_documents():
    async with DocumentTranslationClient(
        endpoint=endpoint,
        credential=DefaultAzureCredential()
    ) as client:
        poller = await client.begin_translation(inputs=[...])
        result = await poller.result()
```

## Các Định dạng Hỗ trợ

| Danh mục       | Định dạng                             |
| -------------- | ------------------------------------- |
| Tài liệu       | DOCX, PDF, PPTX, XLSX, HTML, TXT, RTF |
| Cấu trúc       | CSV, TSV, JSON, XML                   |
| Địa phương hóa | XLIFF, XLF, MHTML                     |

## Yêu cầu Lưu trữ

- Container nguồn và đích phải là Azure Blob Storage
- Sử dụng SAS tokens với quyền thích hợp:
  - Nguồn: Read, List
  - Đích: Write, List

## Thực hành Tốt nhất

1. **Sử dụng SAS tokens** với quyền hạn tối thiểu cần thiết
2. **Theo dõi các hoạt động chạy lâu** với `poller.status()`
3. **Xử lý lỗi cấp tài liệu** bằng cách lặp qua trạng thái tài liệu
4. **Sử dụng bảng thuật ngữ** cho thuật ngữ chuyên ngành
5. **Tách biệt container đích** cho mỗi ngôn ngữ
6. **Sử dụng client async** cho nhiều công việc đồng thời
7. **Kiểm tra định dạng hỗ trợ** trước khi gửi tài liệu
