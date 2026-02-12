---
name: azure-ai-contentunderstanding-py
description: |
  Azure AI Content Understanding SDK cho Python. Sử dụng để trích xuất nội dung đa phương thức từ tài liệu, hình ảnh, âm thanh và video.
  Kích hoạt: "azure-ai-contentunderstanding", "ContentUnderstandingClient", "phân tích đa phương thức", "trích xuất tài liệu", "phân tích video", "phiên âm âm thanh".
package: azure-ai-contentunderstanding
---

# Azure AI Content Understanding SDK cho Python

Dịch vụ AI đa phương thức trích xuất nội dung ngữ nghĩa từ các tệp tài liệu, video, âm thanh và hình ảnh cho RAG và quy trình công việc tự động.

## Cài đặt

```bash
pip install azure-ai-contentunderstanding
```

## Biến Môi trường

```bash
CONTENTUNDERSTANDING_ENDPOINT=https://<resource>.cognitiveservices.azure.com/
```

## Xác thực

```python
import os
from azure.ai.contentunderstanding import ContentUnderstandingClient
from azure.identity import DefaultAzureCredential

endpoint = os.environ["CONTENTUNDERSTANDING_ENDPOINT"]
credential = DefaultAzureCredential()
client = ContentUnderstandingClient(endpoint=endpoint, credential=credential)
```

## Quy trình Cốt lõi

Các hoạt động Content Understanding là các hoạt động không đồng bộ chạy lâu:

1. **Bắt đầu Phân tích** — Bắt đầu hoạt động phân tích với `begin_analyze()` (trả về một poller)
2. **Poll kết quả** — Poll cho đến khi phân tích hoàn tất (SDK xử lý việc này với `.result()`)
3. **Xử lý Kết quả** — Trích xuất kết quả có cấu trúc từ `AnalyzeResult.contents`

## Các Analyzer Xây dựng sẵn

| Analyzer                  | Loại nội dung | Mục đích                                 |
| ------------------------- | ------------- | ---------------------------------------- |
| `prebuilt-documentSearch` | Tài liệu      | Trích xuất markdown cho các ứng dụng RAG |
| `prebuilt-imageSearch`    | Hình ảnh      | Trích xuất nội dung từ hình ảnh          |
| `prebuilt-audioSearch`    | Âm thanh      | Phiên âm âm thanh với thời gian          |
| `prebuilt-videoSearch`    | Video         | Trích xuất khung hình, bản ghi, tóm tắt  |
| `prebuilt-invoice`        | Tài liệu      | Trích xuất các trường hóa đơn            |

## Phân tích Tài liệu

```python
import os
from azure.ai.contentunderstanding import ContentUnderstandingClient
from azure.ai.contentunderstanding.models import AnalyzeInput
from azure.identity import DefaultAzureCredential

endpoint = os.environ["CONTENTUNDERSTANDING_ENDPOINT"]
client = ContentUnderstandingClient(
    endpoint=endpoint,
    credential=DefaultAzureCredential()
)

# Phân tích tài liệu từ URL
poller = client.begin_analyze(
    analyzer_id="prebuilt-documentSearch",
    inputs=[AnalyzeInput(url="https://example.com/document.pdf")]
)

result = poller.result()

# Truy cập nội dung markdown (contents là một danh sách)
content = result.contents[0]
print(content.markdown)
```

## Truy cập Chi tiết Nội dung Tài liệu

```python
from azure.ai.contentunderstanding.models import MediaContentKind, DocumentContent

content = result.contents[0]
if content.kind == MediaContentKind.DOCUMENT:
    document_content: DocumentContent = content  # type: ignore
    print(document_content.start_page_number)
```

## Phân tích Hình ảnh

```python
from azure.ai.contentunderstanding.models import AnalyzeInput

poller = client.begin_analyze(
    analyzer_id="prebuilt-imageSearch",
    inputs=[AnalyzeInput(url="https://example.com/image.jpg")]
)
result = poller.result()
content = result.contents[0]
print(content.markdown)
```

## Phân tích Video

```python
from azure.ai.contentunderstanding.models import AnalyzeInput

poller = client.begin_analyze(
    analyzer_id="prebuilt-videoSearch",
    inputs=[AnalyzeInput(url="https://example.com/video.mp4")]
)

result = poller.result()

# Truy cập nội dung video (AudioVisualContent)
content = result.contents[0]

# Lấy các cụm từ phiên âm với thời gian
for phrase in content.transcript_phrases:
    print(f"[{phrase.start_time} - {phrase.end_time}]: {phrase.text}")

# Lấy các khung hình chính (cho video)
for frame in content.key_frames:
    print(f"Khung hình tại {frame.time}: {frame.description}")
```

## Phân tích Âm thanh

```python
from azure.ai.contentunderstanding.models import AnalyzeInput

poller = client.begin_analyze(
    analyzer_id="prebuilt-audioSearch",
    inputs=[AnalyzeInput(url="https://example.com/audio.mp3")]
)

result = poller.result()

# Truy cập bản ghi âm thanh
content = result.contents[0]
for phrase in content.transcript_phrases:
    print(f"[{phrase.start_time}] {phrase.text}")
```

## Analyzer Tùy chỉnh

Tạo các analyzer tùy chỉnh với lược đồ trường cho trích xuất chuyên biệt:

```python
# Tạo analyzer tùy chỉnh
analyzer = client.create_analyzer(
    analyzer_id="my-invoice-analyzer",
    analyzer={
        "description": "Analyzer hóa đơn tùy chỉnh",
        "base_analyzer_id": "prebuilt-documentSearch",
        "field_schema": {
            "fields": {
                "vendor_name": {"type": "string"},
                "invoice_total": {"type": "number"},
                "line_items": {
                    "type": "array",
                    "items": {
                        "type": "object",
                        "properties": {
                            "description": {"type": "string"},
                            "amount": {"type": "number"}
                        }
                    }
                }
            }
        }
    }
)

# Sử dụng analyzer tùy chỉnh
from azure.ai.contentunderstanding.models import AnalyzeInput

poller = client.begin_analyze(
    analyzer_id="my-invoice-analyzer",
    inputs=[AnalyzeInput(url="https://example.com/invoice.pdf")]
)

result = poller.result()

# Truy cập các trường đã trích xuất
print(result.fields["vendor_name"])
print(result.fields["invoice_total"])
```

## Quản lý Analyzer

```python
# Liệt kê tất cả analyzer
analyzers = client.list_analyzers()
for analyzer in analyzers:
    print(f"{analyzer.analyzer_id}: {analyzer.description}")

# Lấy analyzer cụ thể
analyzer = client.get_analyzer("prebuilt-documentSearch")

# Xóa analyzer tùy chỉnh
client.delete_analyzer("my-custom-analyzer")
```

## Async Client

```python
import asyncio
import os
from azure.ai.contentunderstanding.aio import ContentUnderstandingClient
from azure.ai.contentunderstanding.models import AnalyzeInput
from azure.identity.aio import DefaultAzureCredential

async def analyze_document():
    endpoint = os.environ["CONTENTUNDERSTANDING_ENDPOINT"]
    credential = DefaultAzureCredential()

    async with ContentUnderstandingClient(
        endpoint=endpoint,
        credential=credential
    ) as client:
        poller = await client.begin_analyze(
            analyzer_id="prebuilt-documentSearch",
            inputs=[AnalyzeInput(url="https://example.com/doc.pdf")]
        )
        result = await poller.result()
        content = result.contents[0]
        return content.markdown

asyncio.run(analyze_document())
```

## Các Loại Nội dung

| Lớp                  | Dành cho                       | Cung cấp                                     |
| -------------------- | ------------------------------ | -------------------------------------------- |
| `DocumentContent`    | PDF, hình ảnh, tài liệu Office | Trang, bảng, hình vẽ, đoạn văn               |
| `AudioVisualContent` | Tệp âm thanh, video            | Cụm từ phiên âm, thời gian, khung hình chính |

Cả hai đều kế thừa từ `MediaContent` cung cấp thông tin cơ bản và biểu diễn markdown.

## Import Mô hình

```python
from azure.ai.contentunderstanding.models import (
    AnalyzeInput,
    AnalyzeResult,
    MediaContentKind,
    DocumentContent,
    AudioVisualContent,
)
```

## Các Loại Client

| Client                             | Mục đích                                |
| ---------------------------------- | --------------------------------------- |
| `ContentUnderstandingClient`       | Client đồng bộ cho tất cả hoạt động     |
| `ContentUnderstandingClient` (aio) | Client bất đồng bộ cho tất cả hoạt động |

## Thực hành Tốt nhất

1. **Sử dụng `begin_analyze` với `AnalyzeInput`** — đây là chữ ký phương thức chính xác
2. **Truy cập kết quả qua `result.contents[0]`** — kết quả được trả về dưới dạng danh sách
3. **Sử dụng các analyzer xây dựng sẵn** cho các kịch bản phổ biến (tìm kiếm tài liệu/hình ảnh/âm thanh/video)
4. **Tạo các analyzer tùy chỉnh** chỉ cho trích xuất trường cụ thể theo miền
5. **Sử dụng async client** cho các kịch bản thông lượng cao với thông tin xác thực `azure.identity.aio`
6. **Xử lý các hoạt động chạy lâu** — phân tích video/âm thanh có thể mất vài phút
7. **Sử dụng nguồn URL** khi có thể để tránh chi phí tải lên
