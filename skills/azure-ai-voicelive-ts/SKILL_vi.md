---
name: azure-ai-voicelive-ts
description: |
  Azure AI Voice Live SDK cho JavaScript/TypeScript. Xây dựng các ứng dụng giọng nói AI thời gian thực với giao tiếp WebSocket hai chiều. Sử dụng cho trợ lý giọng nói, AI đàm thoại, speech-to-speech thời gian thực, và chatbot hỗ trợ giọng nói trong môi trường Node.js hoặc trình duyệt. Triggers: "voice live", "real-time voice", "VoiceLiveClient", "VoiceLiveSession", "voice assistant TypeScript", "bidirectional audio", "speech-to-speech JavaScript".
package: @azure/ai-voicelive
---

# @azure/ai-voicelive (JavaScript/TypeScript)

SDK giọng nói AI thời gian thực để xây dựng các trợ lý giọng nói hai chiều với Azure AI trong môi trường Node.js và trình duyệt.

## Cài đặt

```bash
npm install @azure/ai-voicelive @azure/identity
# TypeScript users
npm install @types/node
```

**Phiên bản Hiện tại**: 1.0.0-beta.3

**Môi trường Hỗ trợ**:

- Node.js phiên bản LTS (20+)
- Các trình duyệt hiện đại (Chrome, Firefox, Safari, Edge)

## Biến Môi trường

```bash
AZURE_VOICELIVE_ENDPOINT=https://<resource>.cognitiveservices.azure.com
# Tùy chọn: API key nếu không sử dụng Entra ID
AZURE_VOICELIVE_API_KEY=<your-api-key>
# Tùy chọn: Logging
AZURE_LOG_LEVEL=info
```

## Xác thực

### Microsoft Entra ID (Khuyên dùng)

```typescript
import { DefaultAzureCredential } from "@azure/identity";
import { VoiceLiveClient } from "@azure/ai-voicelive";

const credential = new DefaultAzureCredential();
const endpoint = "https://your-resource.cognitiveservices.azure.com";

const client = new VoiceLiveClient(endpoint, credential);
```

### API Key

```typescript
import { AzureKeyCredential } from "@azure/core-auth";
import { VoiceLiveClient } from "@azure/ai-voicelive";

const endpoint = "https://your-resource.cognitiveservices.azure.com";
const credential = new AzureKeyCredential("your-api-key");

const client = new VoiceLiveClient(endpoint, credential);
```

## Phân cấp Client

```
VoiceLiveClient
└── VoiceLiveSession (Kết nối WebSocket)
    ├── updateSession()      → Cấu hình tùy chọn phiên
    ├── subscribe()          → Các trình xử lý sự kiện (Azure SDK pattern)
    ├── sendAudio()          → Stream đầu vào âm thanh
    ├── addConversationItem() → Thêm tin nhắn/đầu ra hàm
    └── sendEvent()          → Gửi sự kiện giao thức thô
```

## Bắt đầu Nhanh

```typescript
import { DefaultAzureCredential } from "@azure/identity";
import { VoiceLiveClient } from "@azure/ai-voicelive";

const credential = new DefaultAzureCredential();
const endpoint = process.env.AZURE_VOICELIVE_ENDPOINT!;

// Tạo client và bắt đầu phiên
const client = new VoiceLiveClient(endpoint, credential);
const session = await client.startSession("gpt-4o-mini-realtime-preview");

// Cấu hình phiên
await session.updateSession({
  modalities: ["text", "audio"],
  instructions: "Bạn là một trợ lý AI hữu ích. Hãy phản hồi tự nhiên.",
  voice: {
    type: "azure-standard",
    name: "en-US-AvaNeural",
  },
  turnDetection: {
    type: "server_vad",
    threshold: 0.5,
    prefixPaddingMs: 300,
    silenceDurationMs: 500,
  },
  inputAudioFormat: "pcm16",
  outputAudioFormat: "pcm16",
});

// Đăng ký nhận sự kiện
const subscription = session.subscribe({
  onResponseAudioDelta: async (event, context) => {
    // Xử lý đầu ra âm thanh streaming
    const audioData = event.delta;
    playAudioChunk(audioData);
  },
  onResponseTextDelta: async (event, context) => {
    // Xử lý văn bản streaming
    process.stdout.write(event.delta);
  },
  onInputAudioTranscriptionCompleted: async (event, context) => {
    console.log("Người dùng nói:", event.transcript);
  },
});

// Gửi âm thanh từ microphone
function sendAudioChunk(audioBuffer: ArrayBuffer) {
  session.sendAudio(audioBuffer);
}
```

## Cấu hình Phiên

```typescript
await session.updateSession({
  // Modalities
  modalities: ["audio", "text"],

  // Hướng dẫn hệ thống
  instructions: "Bạn là một đại diện dịch vụ khách hàng.",

  // Chọn giọng nói
  voice: {
    type: "azure-standard", // hoặc "azure-custom", "openai"
    name: "en-US-AvaNeural",
  },

  // Phát hiện lượt nói (VAD)
  turnDetection: {
    type: "server_vad", // hoặc "azure_semantic_vad"
    threshold: 0.5,
    prefixPaddingMs: 300,
    silenceDurationMs: 500,
  },

  // Định dạng âm thanh
  inputAudioFormat: "pcm16",
  outputAudioFormat: "pcm16",

  // Công cụ (gọi hàm)
  tools: [
    {
      type: "function",
      name: "get_weather",
      description: "Lấy thời tiết hiện tại",
      parameters: {
        type: "object",
        properties: {
          location: { type: "string" },
        },
        required: ["location"],
      },
    },
  ],
  toolChoice: "auto",
});
```

## Xử lý Sự kiện (Azure SDK Pattern)

SDK sử dụng mô hình xử lý sự kiện dựa trên đăng ký (subscription-based):

```typescript
const subscription = session.subscribe({
  // Vòng đời kết nối
  onConnected: async (args, context) => {
    console.log("Đã kết nối:", args.connectionId);
  },
  onDisconnected: async (args, context) => {
    console.log("Đã ngắt kết nối:", args.code, args.reason);
  },
  onError: async (args, context) => {
    console.error("Lỗi:", args.error.message);
  },

  // Sự kiện phiên
  onSessionCreated: async (event, context) => {
    console.log("Phiên đã tạo:", context.sessionId);
  },
  onSessionUpdated: async (event, context) => {
    console.log("Phiên đã cập nhật");
  },

  // Sự kiện đầu vào âm thanh (VAD)
  onInputAudioBufferSpeechStarted: async (event, context) => {
    console.log("Bắt đầu nói tại:", event.audioStartMs);
  },
  onInputAudioBufferSpeechStopped: async (event, context) => {
    console.log("Dừng nói tại:", event.audioEndMs);
  },

  // Sự kiện phiên âm
  onConversationItemInputAudioTranscriptionCompleted: async (
    event,
    context,
  ) => {
    console.log("Người dùng nói:", event.transcript);
  },
  onConversationItemInputAudioTranscriptionDelta: async (event, context) => {
    process.stdout.write(event.delta);
  },

  // Sự kiện phản hồi
  onResponseCreated: async (event, context) => {
    console.log("Phản hồi bắt đầu");
  },
  onResponseDone: async (event, context) => {
    console.log("Phản hồi hoàn tất");
  },

  // Văn bản streaming
  onResponseTextDelta: async (event, context) => {
    process.stdout.write(event.delta);
  },
  onResponseTextDone: async (event, context) => {
    console.log("\n--- Văn bản hoàn tất ---");
  },

  // Âm thanh streaming
  onResponseAudioDelta: async (event, context) => {
    const audioData = event.delta;
    playAudioChunk(audioData);
  },
  onResponseAudioDone: async (event, context) => {
    console.log("Âm thanh hoàn tất");
  },

  // Transcript âm thanh (trợ lý đã nói gì)
  onResponseAudioTranscriptDelta: async (event, context) => {
    process.stdout.write(event.delta);
  },

  // Gọi hàm
  onResponseFunctionCallArgumentsDone: async (event, context) => {
    if (event.name === "get_weather") {
      const args = JSON.parse(event.arguments);
      const result = await getWeather(args.location);

      await session.addConversationItem({
        type: "function_call_output",
        callId: event.callId,
        output: JSON.stringify(result),
      });

      await session.sendEvent({ type: "response.create" });
    }
  },

  // Catch-all để debug
  onServerEvent: async (event, context) => {
    console.log("Sự kiện:", event.type);
  },
});

// Dọn dẹp khi xong
await subscription.close();
```

## Gọi Hàm (Function Calling)

```typescript
// Định nghĩa công cụ trong cấu hình phiên
await session.updateSession({
  modalities: ["audio", "text"],
  instructions: "Giúp người dùng về thông tin thời tiết.",
  tools: [
    {
      type: "function",
      name: "get_weather",
      description: "Lấy thời tiết hiện tại cho một địa điểm",
      parameters: {
        type: "object",
        properties: {
          location: {
            type: "string",
            description: "Thành phố và bang hoặc quốc gia",
          },
        },
        required: ["location"],
      },
    },
  ],
  toolChoice: "auto",
});

// Xử lý gọi hàm
const subscription = session.subscribe({
  onResponseFunctionCallArgumentsDone: async (event, context) => {
    if (event.name === "get_weather") {
      const args = JSON.parse(event.arguments);
      const weatherData = await fetchWeather(args.location);

      // Gửi kết quả hàm
      await session.addConversationItem({
        type: "function_call_output",
        callId: event.callId,
        output: JSON.stringify(weatherData),
      });

      // Kích hoạt tạo phản hồi
      await session.sendEvent({ type: "response.create" });
    }
  },
});
```

## Tùy chọn Giọng nói

| Loại Giọng nói | Cấu hình                                                   | Ví dụ                            |
| -------------- | ---------------------------------------------------------- | -------------------------------- |
| Azure Standard | `{ type: "azure-standard", name: "..." }`                  | `"en-US-AvaNeural"`              |
| Azure Custom   | `{ type: "azure-custom", name: "...", endpointId: "..." }` | Custom voice endpoint            |
| Azure Personal | `{ type: "azure-personal", speakerProfileId: "..." }`      | Personal voice clone             |
| OpenAI         | `{ type: "openai", name: "..." }`                          | `"alloy"`, `"echo"`, `"shimmer"` |

## Các Mô hình Hỗ trợ

| Mô hình                        | Mô tả                              | Trường hợp Sử dụng          |
| ------------------------------ | ---------------------------------- | --------------------------- |
| `gpt-4o-realtime-preview`      | GPT-4o với âm thanh thời gian thực | AI hội thoại chất lượng cao |
| `gpt-4o-mini-realtime-preview` | Lightweight GPT-4o                 | Tương tác nhanh, hiệu quả   |
| `phi4-mm-realtime`             | Phi đa phương thức                 | Ứng dụng tiết kiệm chi phí  |

## Tùy chọn Phát hiện Lượt nói (Turn Detection)

```typescript
// Server VAD (mặc định)
turnDetection: {
  type: "server_vad",
  threshold: 0.5,
  prefixPaddingMs: 300,
  silenceDurationMs: 500,
}

// Azure Semantic VAD (phát hiện thông minh hơn)
turnDetection: {
  type: "azure_semantic_vad",
}

// Azure Semantic VAD (Tối ưu cho tiếng Anh)
turnDetection: {
  type: "azure_semantic_vad_en",
}

// Azure Semantic VAD (Đa ngôn ngữ)
turnDetection: {
  type: "azure_semantic_vad_multilingual",
}
```

## Định dạng Âm thanh

| Định dạng       | Tần số Lấy mẫu | Trường hợp Sử dụng       |
| --------------- | -------------- | ------------------------ |
| `pcm16`         | 24kHz          | Mặc định, chất lượng cao |
| `pcm16-8000hz`  | 8kHz           | Điện thoại               |
| `pcm16-16000hz` | 16kHz          | Trợ lý giọng nói         |
| `g711_ulaw`     | 8kHz           | Điện thoại (US)          |
| `g711_alaw`     | 8kHz           | Điện thoại (EU)          |

## Tài liệu tham khảo các Loại Chính

| Loại                       | Mục đích                         |
| -------------------------- | -------------------------------- |
| `VoiceLiveClient`          | Client chính để tạo phiên        |
| `VoiceLiveSession`         | Phiên WebSocket đang hoạt động   |
| `VoiceLiveSessionHandlers` | Giao diện trình xử lý sự kiện    |
| `VoiceLiveSubscription`    | Đăng ký sự kiện đang hoạt động   |
| `ConnectionContext`        | Ngữ cảnh cho sự kiện kết nối     |
| `SessionContext`           | Ngữ cảnh cho sự kiện phiên       |
| `ServerEventUnion`         | Union của tất cả sự kiện máy chủ |

## Xử lý Lỗi

```typescript
import {
  VoiceLiveError,
  VoiceLiveConnectionError,
  VoiceLiveAuthenticationError,
  VoiceLiveProtocolError,
} from "@azure/ai-voicelive";

const subscription = session.subscribe({
  onError: async (args, context) => {
    const { error } = args;

    if (error instanceof VoiceLiveConnectionError) {
      console.error("Lỗi kết nối:", error.message);
    } else if (error instanceof VoiceLiveAuthenticationError) {
      console.error("Lỗi xác thực:", error.message);
    } else if (error instanceof VoiceLiveProtocolError) {
      console.error("Lỗi giao thức:", error.message);
    }
  },

  onServerError: async (event, context) => {
    console.error("Lỗi máy chủ:", event.error?.message);
  },
});
```

## Logging

```typescript
import { setLogLevel } from "@azure/logger";

// Bật verbose logging
setLogLevel("info");

// Hoặc qua biến môi trường
// AZURE_LOG_LEVEL=info
```

## Sử dụng trên Trình duyệt

```typescript
// Trình duyệt yêu cầu bundler (Vite, webpack, v.v.)
import { VoiceLiveClient } from "@azure/ai-voicelive";
import { InteractiveBrowserCredential } from "@azure/identity";

// Sử dụng credential tương thích với trình duyệt
const credential = new InteractiveBrowserCredential({
  clientId: "your-client-id",
  tenantId: "your-tenant-id",
});

const client = new VoiceLiveClient(endpoint, credential);

// Yêu cầu truy cập microphone
const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
const audioContext = new AudioContext({ sampleRate: 24000 });

// Xử lý âm thanh và gửi đến phiên
// ... (xem samples để biết chi tiết triển khai)
```

## Thực hành Tốt nhất

1. **Luôn sử dụng `DefaultAzureCredential`** — Không bao giờ hardcode API keys
2. **Đặt cả hai modality** — Bao gồm `["text", "audio"]` cho trợ lý giọng nói
3. **Sử dụng Azure Semantic VAD** — Phát hiện lượt nói tốt hơn VAD máy chủ cơ bản
4. **Xử lý tất cả các loại lỗi** — Lỗi kết nối, xác thực và giao thức
5. **Dọn dẹp đăng ký** — Gọi `subscription.close()` khi xong
6. **Sử dụng định dạng âm thanh phù hợp** — PCM16 tại 24kHz để có chất lượng tốt nhất

## Liên kết Tham khảo

| Tài nguyên    | URL                                                                             |
| ------------- | ------------------------------------------------------------------------------- |
| npm Package   | https://www.npmjs.com/package/@azure/ai-voicelive                               |
| GitHub Source | https://github.com/Azure/azure-sdk-for-js/tree/main/sdk/ai/ai-voicelive         |
| Samples       | https://github.com/Azure/azure-sdk-for-js/tree/main/sdk/ai/ai-voicelive/samples |
| API Reference | https://learn.microsoft.com/javascript/api/@azure/ai-voicelive                  |
