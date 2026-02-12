---
name: azure-ai-textanalytics-py
description: |
  Azure AI Text Analytics SDK cho phân tích cảm xúc, nhận dạng thực thể, cụm từ khóa, phát hiện ngôn ngữ, PII, và NLP chăm sóc sức khỏe. Sử dụng cho xử lý ngôn ngữ tự nhiên trên văn bản.
  Kích hoạt: "text analytics", "phân tích cảm xúc", "nhận dạng thực thể", "cụm từ khóa", "phát hiện PII", "TextAnalyticsClient".
package: azure-ai-textanalytics
---

# Azure AI Text Analytics SDK cho Python

Thư viện Client cho khả năng NLP của dịch vụ Azure AI Language bao gồm cảm xúc, thực thể, cụm từ khóa, và nhiều hơn nữa.

## Cài đặt

```bash
pip install azure-ai-textanalytics
```

## Biến Môi trường

```bash
AZURE_LANGUAGE_ENDPOINT=https://<resource>.cognitiveservices.azure.com
AZURE_LANGUAGE_KEY=<your-api-key>  # Nếu sử dụng API key
```

## Xác thực

### API Key

```python
import os
from azure.core.credentials import AzureKeyCredential
from azure.ai.textanalytics import TextAnalyticsClient

endpoint = os.environ["AZURE_LANGUAGE_ENDPOINT"]
key = os.environ["AZURE_LANGUAGE_KEY"]

client = TextAnalyticsClient(endpoint, AzureKeyCredential(key))
```

### Entra ID (Khuyên dùng)

```python
from azure.ai.textanalytics import TextAnalyticsClient
from azure.identity import DefaultAzureCredential

client = TextAnalyticsClient(
    endpoint=os.environ["AZURE_LANGUAGE_ENDPOINT"],
    credential=DefaultAzureCredential()
)
```

## Phân tích Cảm xúc (Sentiment Analysis)

```python
documents = [
    "Tôi đã có một chuyến đi tuyệt vời đến Seattle tuần trước!",
    "Đồ ăn thật khủng khiếp và dịch vụ thì chậm chạp."
]

result = client.analyze_sentiment(documents, show_opinion_mining=True)

for doc in result:
    if not doc.is_error:
        print(f"Cảm xúc: {doc.sentiment}")
        print(f"Điểm số: pos={doc.confidence_scores.positive:.2f}, "
              f"neg={doc.confidence_scores.negative:.2f}, "
              f"neu={doc.confidence_scores.neutral:.2f}")

        # Opinion mining (khai thác ý kiến dựa trên khía cạnh)
        for sentence in doc.sentences:
            for opinion in sentence.mined_opinions:
                target = opinion.target
                print(f"  Mục tiêu: '{target.text}' - {target.sentiment}")
                for assessment in opinion.assessments:
                    print(f"    Đánh giá: '{assessment.text}' - {assessment.sentiment}")
```

## Nhận dạng Thực thể (Entity Recognition)

```python
documents = ["Microsoft được thành lập bởi Bill Gates và Paul Allen tại Albuquerque."]

result = client.recognize_entities(documents)

for doc in result:
    if not doc.is_error:
        for entity in doc.entities:
            print(f"Thực thể: {entity.text}")
            print(f"  Danh mục: {entity.category}")
            print(f"  Danh mục con: {entity.subcategory}")
            print(f"  Độ tin cậy: {entity.confidence_score:.2f}")
```

## Phát hiện PII

```python
documents = ["SSN của tôi là 123-45-6789 và email của tôi là john@example.com"]

result = client.recognize_pii_entities(documents)

for doc in result:
    if not doc.is_error:
        print(f"Đã che: {doc.redacted_text}")
        for entity in doc.entities:
            print(f"PII: {entity.text} ({entity.category})")
```

## Trích xuất Cụm từ Khóa (Key Phrase Extraction)

```python
documents = ["Azure AI cung cấp khả năng máy học mạnh mẽ cho các nhà phát triển."]

result = client.extract_key_phrases(documents)

for doc in result:
    if not doc.is_error:
        print(f"Cụm từ khóa: {doc.key_phrases}")
```

## Phát hiện Ngôn ngữ (Language Detection)

```python
documents = ["Ce document est en francais.", "This is written in English."]

result = client.detect_language(documents)

for doc in result:
    if not doc.is_error:
        print(f"Ngôn ngữ: {doc.primary_language.name} ({doc.primary_language.iso6391_name})")
        print(f"Độ tin cậy: {doc.primary_language.confidence_score:.2f}")
```

## Phân tích Văn bản Chăm sóc Sức khỏe (Healthcare Text Analytics)

```python
documents = ["Bệnh nhân mắc bệnh tiểu đường và được kê đơn metformin 500mg hai lần mỗi ngày."]

poller = client.begin_analyze_healthcare_entities(documents)
result = poller.result()

for doc in result:
    if not doc.is_error:
        for entity in doc.entities:
            print(f"Thực thể: {entity.text}")
            print(f"  Danh mục: {entity.category}")
            print(f"  Chuẩn hóa: {entity.normalized_text}")

            # Liên kết thực thể (UMLS, v.v.)
            for link in entity.data_sources:
                print(f"  Liên kết: {link.name} - {link.entity_id}")
```

## Phân tích Nhiều (Batch)

```python
from azure.ai.textanalytics import (
    RecognizeEntitiesAction,
    ExtractKeyPhrasesAction,
    AnalyzeSentimentAction
)

documents = ["Microsoft đã công bố các tính năng Azure AI mới tại hội nghị Build."]

poller = client.begin_analyze_actions(
    documents,
    actions=[
        RecognizeEntitiesAction(),
        ExtractKeyPhrasesAction(),
        AnalyzeSentimentAction()
    ]
)

results = poller.result()
for doc_results in results:
    for result in doc_results:
        if result.kind == "EntityRecognition":
            print(f"Thực thể: {[e.text for e in result.entities]}")
        elif result.kind == "KeyPhraseExtraction":
            print(f"Cụm từ khóa: {result.key_phrases}")
        elif result.kind == "SentimentAnalysis":
            print(f"Cảm xúc: {result.sentiment}")
```

## Client Bất đồng bộ (Async Client)

```python
from azure.ai.textanalytics.aio import TextAnalyticsClient
from azure.identity.aio import DefaultAzureCredential

async def analyze():
    async with TextAnalyticsClient(
        endpoint=endpoint,
        credential=DefaultAzureCredential()
    ) as client:
        result = await client.analyze_sentiment(documents)
        # Xử lý kết quả...
```

## Các Loại Client

| Client                      | Mục đích                           |
| --------------------------- | ---------------------------------- |
| `TextAnalyticsClient`       | Tất cả hoạt động phân tích văn bản |
| `TextAnalyticsClient` (aio) | Phiên bản bất đồng bộ              |

## Các Hoạt động Có sẵn

| Phương thức                         | Mô tả                                  |
| ----------------------------------- | -------------------------------------- |
| `analyze_sentiment`                 | Phân tích cảm xúc với khai thác ý kiến |
| `recognize_entities`                | Nhận dạng thực thể được đặt tên        |
| `recognize_pii_entities`            | Phát hiện PII và biên tập lại          |
| `recognize_linked_entities`         | Liên kết thực thể đến Wikipedia        |
| `extract_key_phrases`               | Trích xuất cụm từ khóa                 |
| `detect_language`                   | Phát hiện ngôn ngữ                     |
| `begin_analyze_healthcare_entities` | NLP chăm sóc sức khỏe (chạy lâu)       |
| `begin_analyze_actions`             | Nhiều phân tích trong batch            |

## Thực hành Tốt nhất

1. **Sử dụng các hoạt động batch** cho nhiều tài liệu (lên đến 10 mỗi yêu cầu)
2. **Kích hoạt khai thác ý kiến** để có cảm xúc chi tiết dựa trên khía cạnh
3. **Sử dụng client async** cho các kịch bản lưu lượng lớn
4. **Xử lý lỗi tài liệu** — danh sách kết quả có thể chứa lỗi cho một số tài liệu
5. **Chỉ định ngôn ngữ** khi biết để cải thiện độ chính xác
6. **Sử dụng context manager** hoặc đóng client một cách rõ ràng
