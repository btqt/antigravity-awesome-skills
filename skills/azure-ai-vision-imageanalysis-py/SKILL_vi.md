---
name: azure-ai-vision-imageanalysis-py
description: |
  Azure AI Vision Image Analysis SDK cho chú thích, thẻ, đối tượng, OCR, phát hiện người, và cắt thông minh. Sử dụng cho các tác vụ thị giác máy tính và hiểu hình ảnh.
  Kích hoạt: "image analysis", "computer vision", "OCR", "object detection", "ImageAnalysisClient", "image caption".
package: azure-ai-vision-imageanalysis
---

# Azure AI Vision Image Analysis SDK cho Python

Thư viện Client cho Azure AI Vision 4.0 phân tích hình ảnh (image analysis) bao gồm chú thích, thẻ, đối tượng, OCR, và nhiều hơn nữa.

## Cài đặt

```bash
pip install azure-ai-vision-imageanalysis
```

## Biến Môi trường

```bash
VISION_ENDPOINT=https://<resource>.cognitiveservices.azure.com
VISION_KEY=<your-api-key>  # Nếu sử dụng API key
```

## Xác thực

### API Key

```python
import os
from azure.ai.vision.imageanalysis import ImageAnalysisClient
from azure.core.credentials import AzureKeyCredential

endpoint = os.environ["VISION_ENDPOINT"]
key = os.environ["VISION_KEY"]

client = ImageAnalysisClient(
    endpoint=endpoint,
    credential=AzureKeyCredential(key)
)
```

### Entra ID (Khuyên dùng)

```python
from azure.ai.vision.imageanalysis import ImageAnalysisClient
from azure.identity import DefaultAzureCredential

client = ImageAnalysisClient(
    endpoint=os.environ["VISION_ENDPOINT"],
    credential=DefaultAzureCredential()
)
```

## Phân tích Hình ảnh từ URL

```python
from azure.ai.vision.imageanalysis.models import VisualFeatures

image_url = "https://example.com/image.jpg"

result = client.analyze_from_url(
    image_url=image_url,
    visual_features=[
        VisualFeatures.CAPTION,
        VisualFeatures.TAGS,
        VisualFeatures.OBJECTS,
        VisualFeatures.READ,
        VisualFeatures.PEOPLE,
        VisualFeatures.SMART_CROPS,
        VisualFeatures.DENSE_CAPTIONS
    ],
    gender_neutral_caption=True,
    language="en"
)
```

## Phân tích Hình ảnh từ Tệp

```python
with open("image.jpg", "rb") as f:
    image_data = f.read()

result = client.analyze(
    image_data=image_data,
    visual_features=[VisualFeatures.CAPTION, VisualFeatures.TAGS]
)
```

## Chú thích Hình ảnh (Image Caption)

```python
result = client.analyze_from_url(
    image_url=image_url,
    visual_features=[VisualFeatures.CAPTION],
    gender_neutral_caption=True
)

if result.caption:
    print(f"Chú thích: {result.caption.text}")
    print(f"Độ tin cậy: {result.caption.confidence:.2f}")
```

## Chú thích Dày đặc (Dense Captions)

```python
result = client.analyze_from_url(
    image_url=image_url,
    visual_features=[VisualFeatures.DENSE_CAPTIONS]
)

if result.dense_captions:
    for caption in result.dense_captions.list:
        print(f"Chú thích: {caption.text}")
        print(f"  Độ tin cậy: {caption.confidence:.2f}")
        print(f"  Hộp giới hạn: {caption.bounding_box}")
```

## Thẻ (Tags)

```python
result = client.analyze_from_url(
    image_url=image_url,
    visual_features=[VisualFeatures.TAGS]
)

if result.tags:
    for tag in result.tags.list:
        print(f"Thẻ: {tag.name} (độ tin cậy: {tag.confidence:.2f})")
```

## Phát hiện Đối tượng (Object Detection)

```python
result = client.analyze_from_url(
    image_url=image_url,
    visual_features=[VisualFeatures.OBJECTS]
)

if result.objects:
    for obj in result.objects.list:
        print(f"Đối tượng: {obj.tags[0].name}")
        print(f"  Độ tin cậy: {obj.tags[0].confidence:.2f}")
        box = obj.bounding_box
        print(f"  Hộp giới hạn: x={box.x}, y={box.y}, w={box.width}, h={box.height}")
```

## OCR (Trích xuất Văn bản)

```python
result = client.analyze_from_url(
    image_url=image_url,
    visual_features=[VisualFeatures.READ]
)

if result.read:
    for block in result.read.blocks:
        for line in block.lines:
            print(f"Dòng: {line.text}")
            print(f"  Đa giác giới hạn: {line.bounding_polygon}")

            # Chi tiết cấp từ
            for word in line.words:
                print(f"  Từ: {word.text} (độ tin cậy: {word.confidence:.2f})")
```

## Phát hiện Người (People Detection)

```python
result = client.analyze_from_url(
    image_url=image_url,
    visual_features=[VisualFeatures.PEOPLE]
)

if result.people:
    for person in result.people.list:
        print(f"Phát hiện người:")
        print(f"  Độ tin cậy: {person.confidence:.2f}")
        box = person.bounding_box
        print(f"  Hộp giới hạn: x={box.x}, y={box.y}, w={box.width}, h={box.height}")
```

## Cắt Thông minh (Smart Cropping)

```python
result = client.analyze_from_url(
    image_url=image_url,
    visual_features=[VisualFeatures.SMART_CROPS],
    smart_crops_aspect_ratios=[0.9, 1.33, 1.78]  # Chân dung, 4:3, 16:9
)

if result.smart_crops:
    for crop in result.smart_crops.list:
        print(f"Tỷ lệ khung hình: {crop.aspect_ratio}")
        box = crop.bounding_box
        print(f"  Vùng cắt: x={box.x}, y={box.y}, w={box.width}, h={box.height}")
```

## Client Bất đồng bộ (Async Client)

```python
from azure.ai.vision.imageanalysis.aio import ImageAnalysisClient
from azure.identity.aio import DefaultAzureCredential

async def analyze_image():
    async with ImageAnalysisClient(
        endpoint=endpoint,
        credential=DefaultAzureCredential()
    ) as client:
        result = await client.analyze_from_url(
            image_url=image_url,
            visual_features=[VisualFeatures.CAPTION]
        )
        print(result.caption.text)
```

## Các Tính năng Hình ảnh

| Tính năng        | Mô tả                                         |
| ---------------- | --------------------------------------------- |
| `CAPTION`        | Câu đơn mô tả hình ảnh                        |
| `DENSE_CAPTIONS` | Chú thích cho nhiều vùng                      |
| `TAGS`           | Thẻ nội dung (như đối tượng, cảnh, hành động) |
| `OBJECTS`        | Phát hiện đối tượng với hộp giới hạn          |
| `READ`           | Trích xuất văn bản OCR                        |
| `PEOPLE`         | Phát hiện người với hộp giới hạn              |
| `SMART_CROPS`    | Đề xuất vùng cắt cho hình thu nhỏ             |

## Xử lý Lỗi

```python
from azure.core.exceptions import HttpResponseError

try:
    result = client.analyze_from_url(
        image_url=image_url,
        visual_features=[VisualFeatures.CAPTION]
    )
except HttpResponseError as e:
    print(f"Mã trạng thái: {e.status_code}")
    print(f"Lý do: {e.reason}")
    print(f"Thông báo: {e.error.message}")
```

## Yêu cầu Hình ảnh

- Định dạng: JPEG, PNG, GIF, BMP, WEBP, ICO, TIFF, MPO
- Kích thước tối đa: 20 MB
- Kích thước pixel: 50x50 đến 16000x16000 pixels

## Thực hành Tốt nhất

1. **Chỉ chọn các tính năng cần thiết** để tối ưu hóa độ trễ và chi phí
2. **Sử dụng client async** cho các kịch bản lưu lượng lớn
3. **Xử lý HttpResponseError** cho hình ảnh không hợp lệ hoặc vấn đề xác thực
4. **Bật gender_neutral_caption** cho các mô tả bao quát
5. **Chỉ định ngôn ngữ** cho các chú thích được địa phương hóa
6. **Sử dụng smart_crops_aspect_ratios** phù hợp với yêu cầu hình thu nhỏ của bạn
7. **Cache kết quả** khi phân tích cùng một hình ảnh nhiều lần
