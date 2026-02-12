---
name: azure-ai-transcription-py
description: |
  Azure AI Transcription SDK cho Python. Sử dụng để phiên âm giọng nói thành văn bản thời gian thực và hàng loạt với dấu thời gian và phân loại người nói (diarization).
  Kích hoạt: "transcription", "speech to text", "Azure AI Transcription", "TranscriptionClient".
package: azure-ai-transcription
---

# Azure AI Transcription SDK cho Python

Thư viện Client cho Azure AI Transcription (biến đổi giọng nói thành văn bản) với khả năng phiên âm thời gian thực và hàng loạt.

## Cài đặt

```bash
pip install azure-ai-transcription
```

## Biến Môi trường

```bash
TRANSCRIPTION_ENDPOINT=https://<resource>.cognitiveservices.azure.com
TRANSCRIPTION_KEY=<your-key>
```

## Xác thực

Sử dụng xác thực bằng khóa đăng ký (DefaultAzureCredential không được hỗ trợ cho client này):

```python
import os
from azure.ai.transcription import TranscriptionClient

client = TranscriptionClient(
    endpoint=os.environ["TRANSCRIPTION_ENDPOINT"],
    credential=os.environ["TRANSCRIPTION_KEY"]
)
```

## Phiên âm (Hàng loạt - Batch)

```python
job = client.begin_transcription(
    name="meeting-transcription",
    locale="en-US",
    content_urls=["https://<storage>/audio.wav"],
    diarization_enabled=True
)
result = job.result()
print(result.status)
```

## Phiên âm (Thời gian thực - Real-time)

```python
stream = client.begin_stream_transcription(locale="en-US")
stream.send_audio_file("audio.wav")
for event in stream:
    print(event.text)
```

## Thực hành Tốt nhất

1. **Bật diarization** khi có nhiều người nói
2. **Sử dụng phiên âm hàng loạt** cho các tệp dài được lưu trữ trong blob storage
3. **Thu thập dấu thời gian** để tạo phụ đề
4. **Chỉ định ngôn ngữ** để cải thiện độ chính xác nhận dạng
5. **Xử lý streaming backpressure** cho phiên âm thời gian thực
6. **Đóng các phiên phiên âm** khi hoàn thành
