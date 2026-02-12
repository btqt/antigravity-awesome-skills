---
name: azure-ai-translation-text-py
description: |
  Azure AI Text Translation SDK cho dịch văn bản thời gian thực, chuyển tự, phát hiện ngôn ngữ, vào tra cứu từ điển. Sử dụng để dịch nội dung văn bản trong các ứng dụng.
  Kích hoạt: "text translation", "translator", "dịch văn bản", "chuyển tự", "TextTranslationClient".
package: azure-ai-translation-text
---

# Azure AI Text Translation SDK cho Python

Thư viện Client cho dịch dụ dịch văn bản Azure AI Translator để dịch văn bản thời gian thực, chuyển tự và các hoạt động ngôn ngữ.

## Cài đặt

```bash
pip install azure-ai-translation-text
```

## Biến Môi trường

```bash
AZURE_TRANSLATOR_KEY=<your-api-key>
AZURE_TRANSLATOR_REGION=<your-region>  # ví dụ: eastus, westus2
# Hoặc sử dụng endpoint tùy chỉnh
AZURE_TRANSLATOR_ENDPOINT=https://<resource>.cognitiveservices.azure.com
```

## Xác thực

### API Key với Khu vực (Region)

```python
import os
from azure.ai.translation.text import TextTranslationClient
from azure.core.credentials import AzureKeyCredential

key = os.environ["AZURE_TRANSLATOR_KEY"]
region = os.environ["AZURE_TRANSLATOR_REGION"]

# Tạo credential với region
credential = AzureKeyCredential(key)
client = TextTranslationClient(credential=credential, region=region)
```

### API Key với Endpoint Tùy chỉnh

```python
endpoint = os.environ["AZURE_TRANSLATOR_ENDPOINT"]

client = TextTranslationClient(
    credential=AzureKeyCredential(key),
    endpoint=endpoint
)
```

### Entra ID (Khuyên dùng)

```python
from azure.ai.translation.text import TextTranslationClient
from azure.identity import DefaultAzureCredential

client = TextTranslationClient(
    credential=DefaultAzureCredential(),
    endpoint=os.environ["AZURE_TRANSLATOR_ENDPOINT"]
)
```

## Dịch Cơ bản

```python
# Dịch sang một ngôn ngữ đơn
result = client.translate(
    body=["Hello, how are you?", "Welcome to Azure!"],
    to=["es"]  # Tiếng Tây Ban Nha
)

for item in result:
    for translation in item.translations:
        print(f"Đã dịch: {translation.text}")
        print(f"Ngôn ngữ đích: {translation.to}")
```

## Dịch sang Nhiều Ngôn ngữ

```python
result = client.translate(
    body=["Hello, world!"],
    to=["es", "fr", "de", "ja"]  # Tiếng Tây Ban Nha, Pháp, Đức, Nhật
)

for item in result:
    print(f"Nguồn: {item.detected_language.language if item.detected_language else 'unknown'}")
    for translation in item.translations:
        print(f"  {translation.to}: {translation.text}")
```

## Chỉ định Ngôn ngữ Nguồn

```python
result = client.translate(
    body=["Bonjour le monde"],
    from_parameter="fr",  # Nguồn là tiếng Pháp
    to=["en", "es"]
)
```

## Phát hiện Ngôn ngữ

```python
result = client.translate(
    body=["Hola, como estas?"],
    to=["en"]
)

for item in result:
    if item.detected_language:
        print(f"Ngôn ngữ phát hiện: {item.detected_language.language}")
        print(f"Độ tin cậy: {item.detected_language.score:.2f}")
```

## Chuyển tự (Transliteration)

Chuyển đổi văn bản từ hệ thống viết này sang hệ thống viết khác:

```python
result = client.transliterate(
    body=["konnichiwa"],
    language="ja",
    from_script="Latn",  # Từ hệ Latin
    to_script="Jpan"      # Sang hệ Nhật Bản
)

for item in result:
    print(f"Chuyển tự: {item.text}")
    print(f"Hệ thống viết: {item.script}")
```

## Tra cứu Từ điển

Tìm các bản dịch thay thế và định nghĩa:

```python
result = client.lookup_dictionary_entries(
    body=["fly"],
    from_parameter="en",
    to="es"
)

for item in result:
    print(f"Nguồn: {item.normalized_source} ({item.display_source})")
    for translation in item.translations:
        print(f"  Dịch: {translation.normalized_target}")
        print(f"  Loại từ: {translation.pos_tag}")
        print(f"  Độ tin cậy: {translation.confidence:.2f}")
```

## Ví dụ Từ điển

Lấy ví dụ sử dụng cho các bản dịch:

```python
from azure.ai.translation.text.models import DictionaryExampleTextItem

result = client.lookup_dictionary_examples(
    body=[DictionaryExampleTextItem(text="fly", translation="volar")],
    from_parameter="en",
    to="es"
)

for item in result:
    for example in item.examples:
        print(f"Nguồn: {example.source_prefix}{example.source_term}{example.source_suffix}")
        print(f"Đích: {example.target_prefix}{example.target_term}{example.target_suffix}")
```

## Lấy Ngôn ngữ Hỗ trợ

```python
# Lấy tất cả ngôn ngữ hỗ trợ
languages = client.get_supported_languages()

# Ngôn ngữ dịch
print("Ngôn ngữ dịch:")
for code, lang in languages.translation.items():
    print(f"  {code}: {lang.name} ({lang.native_name})")

# Ngôn ngữ chuyển tự
print("\nNgôn ngữ chuyển tự:")
for code, lang in languages.transliteration.items():
    print(f"  {code}: {lang.name}")
    for script in lang.scripts:
        print(f"    {script.code} -> {[t.code for t in script.to_scripts]}")

# Ngôn ngữ từ điển
print("\nNgôn ngữ từ điển:")
for code, lang in languages.dictionary.items():
    print(f"  {code}: {lang.name}")
```

## Ngắt Câu

Xác định ranh giới câu:

```python
result = client.find_sentence_boundaries(
    body=["Hello! How are you? I hope you are well."],
    language="en"
)

for item in result:
    print(f"Độ dài câu: {item.sent_len}")
```

## Tùy chọn Dịch

```python
result = client.translate(
    body=["Hello, world!"],
    to=["de"],
    text_type="html",           # "plain" hoặc "html"
    profanity_action="Marked",  # "NoAction", "Deleted", "Marked"
    profanity_marker="Asterisk", # "Asterisk", "Tag"
    include_alignment=True,      # Bao gồm căn chỉnh từ
    include_sentence_length=True # Bao gồm ranh giới câu
)

for item in result:
    translation = item.translations[0]
    print(f"Đã dịch: {translation.text}")
    if translation.alignment:
        print(f"Căn chỉnh: {translation.alignment.proj}")
    if translation.sent_len:
        print(f"Độ dài câu: {translation.sent_len.src_sent_len}")
```

## Client Bất đồng bộ (Async Client)

```python
from azure.ai.translation.text.aio import TextTranslationClient
from azure.core.credentials import AzureKeyCredential

async def translate_text():
    async with TextTranslationClient(
        credential=AzureKeyCredential(key),
        region=region
    ) as client:
        result = await client.translate(
            body=["Hello, world!"],
            to=["es"]
        )
        print(result[0].translations[0].text)
```

## Các Phương thức Client

| Phương thức                  | Mô tả                                     |
| ---------------------------- | ----------------------------------------- |
| `translate`                  | Dịch văn bản sang một hoặc nhiều ngôn ngữ |
| `transliterate`              | Chuyển đổi văn bản giữa các hệ thống viết |
| `detect`                     | Phát hiện ngôn ngữ của văn bản            |
| `find_sentence_boundaries`   | Xác định ranh giới câu                    |
| `lookup_dictionary_entries`  | Tra cứu từ điển cho bản dịch              |
| `lookup_dictionary_examples` | Lấy ví dụ sử dụng                         |
| `get_supported_languages`    | Liệt kê các ngôn ngữ hỗ trợ               |

## Thực hành Tốt nhất

1. **Dịch hàng loạt** — Gửi nhiều văn bản trong một yêu cầu (lên đến 100)
2. **Chỉ định ngôn ngữ nguồn** khi biết để cải thiện độ chính xác
3. **Sử dụng client async** cho các kịch bản lưu lượng lớn
4. **Cache danh sách ngôn ngữ** — Các ngôn ngữ hỗ trợ không thay đổi thường xuyên
5. **Xử lý từ ngữ thô tục** phù hợp với ứng dụng của bạn
6. **Sử dụng `text_type="html"`** khi dịch nội dung HTML
7. **Bao gồm căn chỉnh (alignment)** cho các ứng dụng cần ánh xạ từ
