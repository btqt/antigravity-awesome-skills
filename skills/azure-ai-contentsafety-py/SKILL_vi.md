---
name: azure-ai-contentsafety-py
description: |
  Azure AI Content Safety SDK cho Python. Sử dụng để phát hiện nội dung độc hại trong văn bản và hình ảnh với phân loại đa mức độ nghiêm trọng.
  Kích hoạt: "azure-ai-contentsafety", "ContentSafetyClient", "kiểm duyệt nội dung", "nội dung độc hại", "phân tích văn bản", "phân tích hình ảnh".
package: azure-ai-contentsafety
---

# Azure AI Content Safety SDK cho Python

Phát hiện nội dung độc hại do người dùng và AI tạo ra trong các ứng dụng.

## Cài đặt

```bash
pip install azure-ai-contentsafety
```

## Biến Môi trường

```bash
CONTENT_SAFETY_ENDPOINT=https://<resource>.cognitiveservices.azure.com
CONTENT_SAFETY_KEY=<your-api-key>
```

## Xác thực

### API Key

```python
from azure.ai.contentsafety import ContentSafetyClient
from azure.core.credentials import AzureKeyCredential
import os

client = ContentSafetyClient(
    endpoint=os.environ["CONTENT_SAFETY_ENDPOINT"],
    credential=AzureKeyCredential(os.environ["CONTENT_SAFETY_KEY"])
)
```

### Entra ID

```python
from azure.ai.contentsafety import ContentSafetyClient
from azure.identity import DefaultAzureCredential

client = ContentSafetyClient(
    endpoint=os.environ["CONTENT_SAFETY_ENDPOINT"],
    credential=DefaultAzureCredential()
)
```

## Phân tích Văn bản

```python
from azure.ai.contentsafety import ContentSafetyClient
from azure.ai.contentsafety.models import AnalyzeTextOptions, TextCategory
from azure.core.credentials import AzureKeyCredential

client = ContentSafetyClient(endpoint, AzureKeyCredential(key))

request = AnalyzeTextOptions(text="Nội dung văn bản của bạn để phân tích")
response = client.analyze_text(request)

# Kiểm tra từng danh mục
for category in [TextCategory.HATE, TextCategory.SELF_HARM,
                 TextCategory.SEXUAL, TextCategory.VIOLENCE]:
    result = next((r for r in response.categories_analysis
                   if r.category == category), None)
    if result:
        print(f"{category}: mức độ nghiêm trọng {result.severity}")
```

## Phân tích Hình ảnh

```python
from azure.ai.contentsafety import ContentSafetyClient
from azure.ai.contentsafety.models import AnalyzeImageOptions, ImageData
from azure.core.credentials import AzureKeyCredential
import base64

client = ContentSafetyClient(endpoint, AzureKeyCredential(key))

# Từ tệp
with open("image.jpg", "rb") as f:
    image_data = base64.b64encode(f.read()).decode("utf-8")

request = AnalyzeImageOptions(
    image=ImageData(content=image_data)
)

response = client.analyze_image(request)

for result in response.categories_analysis:
    print(f"{result.category}: mức độ nghiêm trọng {result.severity}")
```

### Hình ảnh từ URL

```python
from azure.ai.contentsafety.models import AnalyzeImageOptions, ImageData

request = AnalyzeImageOptions(
    image=ImageData(blob_url="https://example.com/image.jpg")
)

response = client.analyze_image(request)
```

## Quản lý Danh sách chặn Văn bản (Text Blocklist Management)

### Tạo Danh sách chặn

```python
from azure.ai.contentsafety import BlocklistClient
from azure.ai.contentsafety.models import TextBlocklist
from azure.core.credentials import AzureKeyCredential

blocklist_client = BlocklistClient(endpoint, AzureKeyCredential(key))

blocklist = TextBlocklist(
    blocklist_name="my-blocklist",
    description="Các thuật ngữ tùy chỉnh để chặn"
)

result = blocklist_client.create_or_update_text_blocklist(
    blocklist_name="my-blocklist",
    options=blocklist
)
```

### Thêm Mục Chặn

```python
from azure.ai.contentsafety.models import AddOrUpdateTextBlocklistItemsOptions, TextBlocklistItem

items = AddOrUpdateTextBlocklistItemsOptions(
    blocklist_items=[
        TextBlocklistItem(text="term-bị-chặn-1"),
        TextBlocklistItem(text="term-bị-chặn-2")
    ]
)

result = blocklist_client.add_or_update_blocklist_items(
    blocklist_name="my-blocklist",
    options=items
)
```

### Phân tích với Danh sách chặn

```python
from azure.ai.contentsafety.models import AnalyzeTextOptions

request = AnalyzeTextOptions(
    text="Văn bản chứa term-bị-chặn-1",
    blocklist_names=["my-blocklist"],
    halt_on_blocklist_hit=True
)

response = client.analyze_text(request)

if response.blocklists_match:
    for match in response.blocklists_match:
        print(f"Đã chặn: {match.blocklist_item_text}")
```

## Các Mức độ Nghiêm trọng

Phân tích văn bản trả về 4 mức độ nghiêm trọng (0, 2, 4, 6) theo mặc định. Đối với 8 mức độ (0-7):

```python
from azure.ai.contentsafety.models import AnalyzeTextOptions, AnalyzeTextOutputType

request = AnalyzeTextOptions(
    text="Văn bản của bạn",
    output_type=AnalyzeTextOutputType.EIGHT_SEVERITY_LEVELS
)
```

## Các Danh mục Gây hại

| Danh mục   | Mô tả                                                            |
| ---------- | ---------------------------------------------------------------- |
| `Hate`     | Tấn công dựa trên bản sắc (chủng tộc, tôn giáo, giới tính, v.v.) |
| `Sexual`   | Nội dung tình dục, các mối quan hệ, giải phẫu học                |
| `Violence` | Tác hại vật lý, vũ khí, thương tích                              |
| `SelfHarm` | Tự gây thương tích, tự sát, rối loạn ăn uống                     |

## Thang đo Mức độ Nghiêm trọng

| Mức độ | Phạm vi Văn bản | Phạm vi Hình ảnh | Ý nghĩa                   |
| ------ | --------------- | ---------------- | ------------------------- |
| 0      | An toàn         | An toàn          | Không có nội dung độc hại |
| 2      | Thấp            | Thấp             | Tham chiếu nhẹ            |
| 4      | Trung bình      | Trung bình       | Nội dung vừa phải         |
| 6      | Cao             | Cao              | Nội dung nghiêm trọng     |

## Các Loại Client

| Client                | Mục đích                         |
| --------------------- | -------------------------------- |
| `ContentSafetyClient` | Phân tích văn bản và hình ảnh    |
| `BlocklistClient`     | Quản lý danh sách chặn tùy chỉnh |

## Thực hành Tốt nhất

1. **Sử dụng danh sách chặn** cho các thuật ngữ cụ thể theo miền
2. **Đặt ngưỡng mức độ nghiêm trọng** phù hợp cho trường hợp sử dụng của bạn
3. **Xử lý nhiều danh mục** — nội dung có thể gây hại theo nhiều cách
4. **Sử dụng halt_on_blocklist_hit** để từ chối ngay lập tức
5. **Ghi log kết quả phân tích** để kiểm toán và cải thiện
6. **Cân nhắc chế độ 8-mức độ** để kiểm soát chi tiết hơn
7. **Kiểm duyệt trước đầu ra AI** trước khi hiển thị cho người dùng
