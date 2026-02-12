---
name: azure-speech-to-text-rest-py
description: |
  Azure Speech to Text REST API cho âm thanh ngắn (Python). Sử dụng để nhận dạng giọng nói đơn giản cho các tệp âm thanh lên đến 60 giây mà không cần Speech SDK.
  Triggers: "speech to text REST", "short audio transcription", "speech recognition REST API", "STT REST", "recognize speech REST".
  KHÔNG SỬ DỤNG CHO: Âm thanh dài (>60 giây), real-time streaming, batch transcription, custom speech models, speech translation. Hãy sử dụng Speech SDK hoặc Batch Transcription API thay thế.
---

# Azure Speech to Text REST API cho Âm thanh Ngắn

REST API đơn giản để chuyển đổi giọng nói thành văn bản cho các tệp âm thanh ngắn (lên đến 60 giây). Không yêu cầu SDK - chỉ cần các yêu cầu HTTP.

## Điều kiện Tiên quyết

1. **Azure subscription** - [Tạo miễn phí](https://azure.microsoft.com/free/)
2. **Speech resource** - Tạo trong [Azure Portal](https://portal.azure.com/#create/Microsoft.CognitiveServicesSpeechServices)
3. **Lấy thông tin xác thực** - Sau khi triển khai, đi đến resource > Keys and Endpoint

## Biến Môi trường

```bash
# Bắt buộc
AZURE_SPEECH_KEY=<your-speech-resource-key>
AZURE_SPEECH_REGION=<region>  # ví dụ: eastus, westus2, westeurope

# Thay thế: Sử dụng endpoint trực tiếp
AZURE_SPEECH_ENDPOINT=https://<region>.stt.speech.microsoft.com
```

## Cài đặt

```bash
pip install requests
```

## Bắt đầu Nhanh

```python
import os
import requests

def transcribe_audio(audio_file_path: str, language: str = "en-US") -> dict:
    """Transcribe tệp âm thanh ngắn (tối đa 60 giây) sử dụng REST API."""
    region = os.environ["AZURE_SPEECH_REGION"]
    api_key = os.environ["AZURE_SPEECH_KEY"]

    url = f"https://{region}.stt.speech.microsoft.com/speech/recognition/conversation/cognitiveservices/v1"

    headers = {
        "Ocp-Apim-Subscription-Key": api_key,
        "Content-Type": "audio/wav; codecs=audio/pcm; samplerate=16000",
        "Accept": "application/json"
    }

    params = {
        "language": language,
        "format": "detailed"  # hoặc "simple"
    }

    with open(audio_file_path, "rb") as audio_file:
        response = requests.post(url, headers=headers, params=params, data=audio_file)

    response.raise_for_status()
    return response.json()

# Sử dụng
result = transcribe_audio("audio.wav", "en-US")
print(result["DisplayText"])
```

## Yêu cầu về Âm thanh

| Định dạng | Codec | Sample Rate  | Ghi chú                 |
| --------- | ----- | ------------ | ----------------------- |
| WAV       | PCM   | 16 kHz, mono | **Khuyên dùng**         |
| OGG       | OPUS  | 16 kHz, mono | Kích thước file nhỏ hơn |

**Hạn chế:**

- Tối đa 60 giây âm thanh
- Đối với đánh giá phát âm: tối đa 30 giây
- Không có kết quả một phần/tạm thời (chỉ có kết quả cuối cùng)

## Content-Type Headers

```python
# WAV PCM 16kHz
"Content-Type": "audio/wav; codecs=audio/pcm; samplerate=16000"

# OGG OPUS
"Content-Type": "audio/ogg; codecs=opus"
```

## Định dạng Phản hồi

### Simple Format (mặc định)

```python
params = {"language": "en-US", "format": "simple"}
```

```json
{
  "RecognitionStatus": "Success",
  "DisplayText": "Remind me to buy 5 pencils.",
  "Offset": "1236645672289",
  "Duration": "1236645672289"
}
```

### Detailed Format

```python
params = {"language": "en-US", "format": "detailed"}
```

```json
{
  "RecognitionStatus": "Success",
  "Offset": "1236645672289",
  "Duration": "1236645672289",
  "NBest": [
    {
      "Confidence": 0.9052885,
      "Display": "What's the weather like?",
      "ITN": "what's the weather like",
      "Lexical": "what's the weather like",
      "MaskedITN": "what's the weather like"
    }
  ]
}
```

## Chunked Transfer (Khuyên dùng)

Để có độ trễ thấp hơn, stream âm thanh theo từng đoạn (chunk):

```python
import os
import requests

def transcribe_chunked(audio_file_path: str, language: str = "en-US") -> dict:
    """Stream âm thanh theo từng đoạn để có độ trễ thấp hơn."""
    region = os.environ["AZURE_SPEECH_REGION"]
    api_key = os.environ["AZURE_SPEECH_KEY"]

    url = f"https://{region}.stt.speech.microsoft.com/speech/recognition/conversation/cognitiveservices/v1"

    headers = {
        "Ocp-Apim-Subscription-Key": api_key,
        "Content-Type": "audio/wav; codecs=audio/pcm; samplerate=16000",
        "Accept": "application/json",
        "Transfer-Encoding": "chunked",
        "Expect": "100-continue"
    }

    params = {"language": language, "format": "detailed"}

    def generate_chunks(file_path: str, chunk_size: int = 1024):
        with open(file_path, "rb") as f:
            while chunk := f.read(chunk_size):
                yield chunk

    response = requests.post(
        url,
        headers=headers,
        params=params,
        data=generate_chunks(audio_file_path)
    )

    response.raise_for_status()
    return response.json()
```

## Các Tùy chọn Xác thực

### Tùy chọn 1: Subscription Key (Đơn giản)

```python
headers = {
    "Ocp-Apim-Subscription-Key": os.environ["AZURE_SPEECH_KEY"]
}
```

### Tùy chọn 2: Bearer Token

```python
import requests
import os

def get_access_token() -> str:
    """Lấy access token từ token endpoint."""
    region = os.environ["AZURE_SPEECH_REGION"]
    api_key = os.environ["AZURE_SPEECH_KEY"]

    token_url = f"https://{region}.api.cognitive.microsoft.com/sts/v1.0/issueToken"

    response = requests.post(
        token_url,
        headers={
            "Ocp-Apim-Subscription-Key": api_key,
            "Content-Type": "application/x-www-form-urlencoded",
            "Content-Length": "0"
        }
    )
    response.raise_for_status()
    return response.text

# Sử dụng token trong các request (có hiệu lực trong 10 phút)
token = get_access_token()
headers = {
    "Authorization": f"Bearer {token}",
    "Content-Type": "audio/wav; codecs=audio/pcm; samplerate=16000",
    "Accept": "application/json"
}
```

## Các Tham số Truy vấn

| Tham số     | Bắt buộc | Giá trị                    | Mô tả                                   |
| ----------- | -------- | -------------------------- | --------------------------------------- |
| `language`  | **Có**   | `en-US`, `de-DE`, v.v.     | Ngôn ngữ của giọng nói                  |
| `format`    | Không    | `simple`, `detailed`       | Định dạng kết quả (mặc định: simple)    |
| `profanity` | Không    | `masked`, `removed`, `raw` | Xử lý từ ngữ thô tục (mặc định: masked) |

## Các Giá trị Trạng thái Nhận dạng

| Trạng thái              | Mô tả                                       |
| ----------------------- | ------------------------------------------- |
| `Success`               | Nhận dạng thành công                        |
| `NoMatch`               | Phát hiện giọng nói nhưng không khớp từ nào |
| `InitialSilenceTimeout` | Chỉ phát hiện sự im lặng                    |
| `BabbleTimeout`         | Chỉ phát hiện tiếng ồn                      |
| `Error`                 | Lỗi dịch vụ nội bộ                          |

## Xử lý Từ ngữ Thô tục

```python
# Che từ ngữ thô tục bằng dấu sao (mặc định)
params = {"language": "en-US", "profanity": "masked"}

# Xóa hoàn toàn từ ngữ thô tục
params = {"language": "en-US", "profanity": "removed"}

# Bao gồm từ ngữ thô tục nguyên văn
params = {"language": "en-US", "profanity": "raw"}
```

## Xử lý Lỗi

```python
import requests

def transcribe_with_error_handling(audio_path: str, language: str = "en-US") -> dict | None:
    """Transcribe với xử lý lỗi thích hợp."""
    region = os.environ["AZURE_SPEECH_REGION"]
    api_key = os.environ["AZURE_SPEECH_KEY"]

    url = f"https://{region}.stt.speech.microsoft.com/speech/recognition/conversation/cognitiveservices/v1"

    try:
        with open(audio_path, "rb") as audio_file:
            response = requests.post(
                url,
                headers={
                    "Ocp-Apim-Subscription-Key": api_key,
                    "Content-Type": "audio/wav; codecs=audio/pcm; samplerate=16000",
                    "Accept": "application/json"
                },
                params={"language": language, "format": "detailed"},
                data=audio_file
            )

        if response.status_code == 200:
            result = response.json()
            if result.get("RecognitionStatus") == "Success":
                return result
            else:
                print(f"Recognition failed: {result.get('RecognitionStatus')}")
                return None
        elif response.status_code == 400:
            print(f"Bad request: Check language code or audio format")
        elif response.status_code == 401:
            print(f"Unauthorized: Check API key or token")
        elif response.status_code == 403:
            print(f"Forbidden: Missing authorization header")
        else:
            print(f"Error {response.status_code}: {response.text}")

        return None

    except requests.exceptions.RequestException as e:
        print(f"Request failed: {e}")
        return None
```

## Phiên bản Async

```python
import os
import aiohttp
import asyncio

async def transcribe_async(audio_file_path: str, language: str = "en-US") -> dict:
    """Async version using aiohttp."""
    region = os.environ["AZURE_SPEECH_REGION"]
    api_key = os.environ["AZURE_SPEECH_KEY"]

    url = f"https://{region}.stt.speech.microsoft.com/speech/recognition/conversation/cognitiveservices/v1"

    headers = {
        "Ocp-Apim-Subscription-Key": api_key,
        "Content-Type": "audio/wav; codecs=audio/pcm; samplerate=16000",
        "Accept": "application/json"
    }

    params = {"language": language, "format": "detailed"}

    async with aiohttp.ClientSession() as session:
        with open(audio_file_path, "rb") as f:
            audio_data = f.read()

        async with session.post(url, headers=headers, params=params, data=audio_data) as response:
            response.raise_for_status()
            return await response.json()

# Sử dụng
result = asyncio.run(transcribe_async("audio.wav", "en-US"))
print(result["DisplayText"])
```

## Các Ngôn ngữ Được Hỗ trợ

Mã ngôn ngữ phổ biến (xem [danh sách đầy đủ](https://learn.microsoft.com/azure/ai-services/speech-service/language-support)):

| Mã      | Ngôn ngữ                        |
| ------- | ------------------------------- |
| `en-US` | Tiếng Anh (Mỹ)                  |
| `en-GB` | Tiếng Anh (Anh)                 |
| `de-DE` | Tiếng Đức                       |
| `fr-FR` | Tiếng Pháp                      |
| `es-ES` | Tiếng Tây Ban Nha (Tây Ban Nha) |
| `es-MX` | Tiếng Tây Ban Nha (Mexico)      |
| `zh-CN` | Tiếng Trung (Quan thoại)        |
| `ja-JP` | Tiếng Nhật                      |
| `ko-KR` | Tiếng Hàn                       |
| `pt-BR` | Tiếng Bồ Đào Nha (Brazil)       |

## Thực hành Tốt nhất

1. **Sử dụng WAV PCM 16kHz mono** để có độ tương thích tốt nhất
2. **Bật chunked transfer** để có độ trễ thấp hơn
3. **Cache access tokens** trong 9 phút (có hiệu lực trong 10 phút)
4. **Chỉ định ngôn ngữ chính xác** để nhận dạng chính xác
5. **Sử dụng detailed format** khi bạn cần điểm tin cậy
6. **Xử lý tất cả các giá trị RecognitionStatus** trong mã production

## Khi nào KHÔNG sử dụng API này

Hãy sử dụng Speech SDK hoặc Batch Transcription API thay thế khi bạn cần:

- Âm thanh dài hơn 60 giây
- Real-time streaming transcription (phiên âm thời gian thực)
- Kết quả một phần/tạm thời
- Dịch giọng nói
- Custom speech models
- Phiên âm hàng loạt nhiều tệp

## Các File Tham khảo

| File                                                                             | Nội dung                              |
| -------------------------------------------------------------------------------- | ------------------------------------- |
| [references/pronunciation-assessment.md](references/pronunciation-assessment.md) | Tham số và chấm điểm đánh giá phát âm |
