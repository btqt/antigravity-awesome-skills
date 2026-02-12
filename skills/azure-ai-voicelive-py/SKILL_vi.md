---
name: azure-ai-voicelive-py
description: Xây dựng các ứng dụng giọng nói AI thời gian thực sử dụng Azure AI Voice Live SDK (azure-ai-voicelive). Sử dụng kỹ năng này khi tạo các ứng dụng Python cần giao tiếp âm thanh hai chiều thời gian thực với Azure AI, bao gồm trợ lý giọng nói, chatbot hỗ trợ giọng nói, dịch speech-to-speech thời gian thực, avatar điều khiển bằng giọng nói, hoặc bất kỳ streaming âm thanh dựa trên WebSocket nào với các mô hình AI. Hỗ trợ Server VAD (Voice Activity Detection), hội thoại theo lượt, gọi hàm, công cụ MCP, tích hợp avatar, và phiên âm.
package: azure-ai-voicelive
---

# Azure AI Voice Live SDK

Xây dựng các ứng dụng giọng nói AI thời gian thực với giao tiếp WebSocket hai chiều.

## Cài đặt

```bash
pip install azure-ai-voicelive aiohttp azure-identity
```

## Biến Môi trường

```bash
AZURE_COGNITIVE_SERVICES_ENDPOINT=https://<region>.api.cognitive.microsoft.com
# Cho xác thực bằng API key (không khuyến nghị cho sản xuất)
AZURE_COGNITIVE_SERVICES_KEY=<api-key>
```

## Xác thực

**DefaultAzureCredential (ưu tiên)**:

```python
from azure.ai.voicelive.aio import connect
from azure.identity.aio import DefaultAzureCredential

async with connect(
    endpoint=os.environ["AZURE_COGNITIVE_SERVICES_ENDPOINT"],
    credential=DefaultAzureCredential(),
    model="gpt-4o-realtime-preview",
    credential_scopes=["https://cognitiveservices.azure.com/.default"]
) as conn:
    ...
```

**API Key**:

```python
from azure.ai.voicelive.aio import connect
from azure.core.credentials import AzureKeyCredential

async with connect(
    endpoint=os.environ["AZURE_COGNITIVE_SERVICES_ENDPOINT"],
    credential=AzureKeyCredential(os.environ["AZURE_COGNITIVE_SERVICES_KEY"]),
    model="gpt-4o-realtime-preview"
) as conn:
    ...
```

## Bắt đầu Nhanh

```python
import asyncio
import os
from azure.ai.voicelive.aio import connect
from azure.identity.aio import DefaultAzureCredential

async def main():
    async with connect(
        endpoint=os.environ["AZURE_COGNITIVE_SERVICES_ENDPOINT"],
        credential=DefaultAzureCredential(),
        model="gpt-4o-realtime-preview",
        credential_scopes=["https://cognitiveservices.azure.com/.default"]
    ) as conn:
        # Cập nhật phiên với hướng dẫn
        await conn.session.update(session={
            "instructions": "Bạn là một trợ lý hữu ích.",
            "modalities": ["text", "audio"],
            "voice": "alloy"
        })

        # Lắng nghe sự kiện
        async for event in conn:
            print(f"Sự kiện: {event.type}")
            if event.type == "response.audio_transcript.done":
                print(f"Transcript: {event.transcript}")
            elif event.type == "response.done":
                break

asyncio.run(main())
```

## Kiến trúc Cốt lõi

### Tài nguyên Kết nối

`VoiceLiveConnection` hiển thị các tài nguyên sau:

| Tài nguyên                   | Mục đích             | Các phương thức chính                               |
| ---------------------------- | -------------------- | --------------------------------------------------- |
| `conn.session`               | Cấu hình phiên       | `update(session=...)`                               |
| `conn.response`              | Phản hồi mô hình     | `create()`, `cancel()`                              |
| `conn.input_audio_buffer`    | Đầu vào âm thanh     | `append()`, `commit()`, `clear()`                   |
| `conn.output_audio_buffer`   | Đầu ra âm thanh      | `clear()`                                           |
| `conn.conversation`          | Trạng thái hội thoại | `item.create()`, `item.delete()`, `item.truncate()` |
| `conn.transcription_session` | Cấu hình phiên âm    | `update(session=...)`                               |

## Cấu hình Phiên

```python
from azure.ai.voicelive.models import RequestSession, FunctionTool

await conn.session.update(session=RequestSession(
    instructions="Bạn là một trợ lý giọng nói hữu ích.",
    modalities=["text", "audio"],
    voice="alloy",  # hoặc "echo", "shimmer", "sage", v.v.
    input_audio_format="pcm16",
    output_audio_format="pcm16",
    turn_detection={
        "type": "server_vad",
        "threshold": 0.5,
        "prefix_padding_ms": 300,
        "silence_duration_ms": 500
    },
    tools=[
        FunctionTool(
            type="function",
            name="get_weather",
            description="Lấy thời tiết hiện tại",
            parameters={
                "type": "object",
                "properties": {
                    "location": {"type": "string"}
                },
                "required": ["location"]
            }
        )
    ]
))
```

## Streaming Âm thanh

### Gửi Âm thanh (Base64 PCM16)

```python
import base64

# Đọc chunk âm thanh (16-bit PCM, 24kHz mono)
audio_chunk = await read_audio_from_microphone()
b64_audio = base64.b64encode(audio_chunk).decode()

await conn.input_audio_buffer.append(audio=b64_audio)
```

### Nhận Âm thanh

```python
async for event in conn:
    if event.type == "response.audio.delta":
        audio_bytes = base64.b64decode(event.delta)
        await play_audio(audio_bytes)
    elif event.type == "response.audio.done":
        print("Âm thanh hoàn tất")
```

## Xử lý Sự kiện

```python
async for event in conn:
    match event.type:
        # Sự kiện phiên
        case "session.created":
            print(f"Phiên: {event.session}")
        case "session.updated":
            print("Đã cập nhật phiên")

        # Sự kiện đầu vào âm thanh
        case "input_audio_buffer.speech_started":
            print(f"Bắt đầu nói tại {event.audio_start_ms}ms")
        case "input_audio_buffer.speech_stopped":
            print(f"Dừng nói tại {event.audio_end_ms}ms")

        # Sự kiện phiên âm
        case "conversation.item.input_audio_transcription.completed":
            print(f"Người dùng nói: {event.transcript}")
        case "conversation.item.input_audio_transcription.delta":
            print(f"Một phần: {event.delta}")

        # Sự kiện phản hồi
        case "response.created":
            print(f"Phản hồi bắt đầu: {event.response.id}")
        case "response.audio_transcript.delta":
            print(event.delta, end="", flush=True)
        case "response.audio.delta":
            audio = base64.b64decode(event.delta)
        case "response.done":
            print(f"Phản hồi hoàn tất: {event.response.status}")

        # Gọi hàm
        case "response.function_call_arguments.done":
            result = handle_function(event.name, event.arguments)
            await conn.conversation.item.create(item={
                "type": "function_call_output",
                "call_id": event.call_id,
                "output": json.dumps(result)
            })
            await conn.response.create()

        # Lỗi
        case "error":
            print(f"Lỗi: {event.error.message}")
```

## Các Mẫu Phổ biến

### Chế độ Lượt Thủ công (Không có VAD)

```python
await conn.session.update(session={"turn_detection": None})

# Điều khiển lượt thủ công
await conn.input_audio_buffer.append(audio=b64_audio)
await conn.input_audio_buffer.commit()  # Kết thúc lượt người dùng
await conn.response.create()  # Kích hoạt phản hồi
```

### Xử lý Ngắt lời

```python
async for event in conn:
    if event.type == "input_audio_buffer.speech_started":
        # Người dùng ngắt lời - hủy phản hồi hiện tại
        await conn.response.cancel()
        await conn.output_audio_buffer.clear()
```

### Lịch sử Hội thoại

```python
# Thêm tin nhắn hệ thống
await conn.conversation.item.create(item={
    "type": "message",
    "role": "system",
    "content": [{"type": "input_text", "text": "Hãy ngắn gọn."}]
})

# Thêm tin nhắn người dùng
await conn.conversation.item.create(item={
    "type": "message",
    "role": "user",
    "content": [{"type": "input_text", "text": "Xin chào!"}]
})

await conn.response.create()
```

## Tùy chọn Giọng nói

| Giọng     | Mô tả                    |
| --------- | ------------------------ |
| `alloy`   | Trung tính, cân bằng     |
| `echo`    | Ấm áp, trò chuyện        |
| `shimmer` | Rõ ràng, chuyên nghiệp   |
| `sage`    | Điềm tĩnh, có thẩm quyền |
| `coral`   | Thân thiện, vui vẻ       |
| `ash`     | Trầm, chừng mực          |
| `ballad`  | Biểu cảm                 |
| `verse`   | Kể chuyện                |

Giọng Azure: Sử dụng các mô hình `AzureStandardVoice`, `AzureCustomVoice`, hoặc `AzurePersonalVoice`.

## Định dạng Âm thanh

| Định dạng       | Tần số Lấy mẫu | Trường hợp Sử dụng       |
| --------------- | -------------- | ------------------------ |
| `pcm16`         | 24kHz          | Mặc định, chất lượng cao |
| `pcm16-8000hz`  | 8kHz           | Điện thoại               |
| `pcm16-16000hz` | 16kHz          | Trợ lý giọng nói         |
| `g711_ulaw`     | 8kHz           | Điện thoại (US)          |
| `g711_alaw`     | 8kHz           | Điện thoại (EU)          |

## Tùy chọn Phát hiện Lượt nói (Turn Detection)

```python
# Server VAD (mặc định)
{"type": "server_vad", "threshold": 0.5, "silence_duration_ms": 500}

# Azure Semantic VAD (phát hiện thông minh hơn)
{"type": "azure_semantic_vad"}
{"type": "azure_semantic_vad_en"}  # Tối ưu cho tiếng Anh
{"type": "azure_semantic_vad_multilingual"}
```

## Xử lý Lỗi

```python
from azure.ai.voicelive.aio import ConnectionError, ConnectionClosed

try:
    async with connect(...) as conn:
        async for event in conn:
            if event.type == "error":
                print(f"Lỗi API: {event.error.code} - {event.error.message}")
except ConnectionClosed as e:
    print(f"Kết nối bị đóng: {e.code} - {e.reason}")
except ConnectionError as e:
    print(f"Lỗi kết nối: {e}")
```

## Tham khảo

- **API Reference Chi tiết**: Xem [references/api-reference.md](references/api-reference.md)
- **Ví dụ Hoàn chỉnh**: Xem [references/examples.md](references/examples.md)
- **Tất cả Mô hình & Loại**: Xem [references/models.md](references/models.md)
