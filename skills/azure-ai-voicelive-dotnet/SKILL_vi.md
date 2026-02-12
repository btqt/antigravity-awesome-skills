---
name: azure-ai-voicelive-dotnet
description: |
  Azure AI Voice Live SDK cho .NET. Xây dựng các ứng dụng giọng nói AI thời gian thực với giao tiếp WebSocket hai chiều. Sử dụng cho trợ lý giọng nói, AI đàm thoại, speech-to-speech thời gian thực, và chatbot hỗ trợ giọng nói. Triggers: "voice live", "real-time voice", "VoiceLiveClient", "VoiceLiveSession", "voice assistant .NET", "bidirectional audio", "speech-to-speech".
package: Azure.AI.VoiceLive
---

# Azure.AI.VoiceLive (.NET)

SDK giọng nói AI thời gian thực để xây dựng các trợ lý giọng nói hai chiều với Azure AI.

## Cài đặt

```bash
dotnet add package Azure.AI.VoiceLive
dotnet add package Azure.Identity
dotnet add package NAudio                    # Cho thu/phát âm thanh
```

**Phiên bản Hiện tại**: Stable v1.0.0, Preview v1.1.0-beta.1

## Biến Môi trường

```bash
AZURE_VOICELIVE_ENDPOINT=https://<resource>.services.ai.azure.com/
AZURE_VOICELIVE_MODEL=gpt-4o-realtime-preview
AZURE_VOICELIVE_VOICE=en-US-AvaNeural
# Tùy chọn: API key nếu không sử dụng Entra ID
AZURE_VOICELIVE_API_KEY=<your-api-key>
```

## Xác thực

### Microsoft Entra ID (Khuyên dùng)

```csharp
using Azure.Identity;
using Azure.AI.VoiceLive;

Uri endpoint = new Uri("https://your-resource.cognitiveservices.azure.com");
DefaultAzureCredential credential = new DefaultAzureCredential();
VoiceLiveClient client = new VoiceLiveClient(endpoint, credential);
```

**Vai trò Yêu cầu**: `Cognitive Services User` (gán trong Azure Portal → Access control)

### API Key

```csharp
Uri endpoint = new Uri("https://your-resource.cognitiveservices.azure.com");
AzureKeyCredential credential = new AzureKeyCredential("your-api-key");
VoiceLiveClient client = new VoiceLiveClient(endpoint, credential);
```

## Phân cấp Client

```
VoiceLiveClient
└── VoiceLiveSession (Kết nối WebSocket)
    ├── ConfigureSessionAsync()
    ├── GetUpdatesAsync() → Các sự kiện SessionUpdate
    ├── AddItemAsync() → UserMessageItem, FunctionCallOutputItem
    ├── SendAudioAsync()
    └── StartResponseAsync()
```

## Quy trình Cốt lõi

### 1. Bắt đầu Phiên và Cấu hình

```csharp
using Azure.Identity;
using Azure.AI.VoiceLive;

var endpoint = new Uri(Environment.GetEnvironmentVariable("AZURE_VOICELIVE_ENDPOINT"));
var client = new VoiceLiveClient(endpoint, new DefaultAzureCredential());

var model = "gpt-4o-mini-realtime-preview";

// Bắt đầu phiên
using VoiceLiveSession session = await client.StartSessionAsync(model);

// Cấu hình phiên
VoiceLiveSessionOptions sessionOptions = new()
{
    Model = model,
    Instructions = "Bạn là một trợ lý AI hữu ích. Hãy phản hồi tự nhiên.",
    Voice = new AzureStandardVoice("en-US-AvaNeural"),
    TurnDetection = new AzureSemanticVadTurnDetection()
    {
        Threshold = 0.5f,
        PrefixPadding = TimeSpan.FromMilliseconds(300),
        SilenceDuration = TimeSpan.FromMilliseconds(500)
    },
    InputAudioFormat = InputAudioFormat.Pcm16,
    OutputAudioFormat = OutputAudioFormat.Pcm16
};

// Thiết lập modality (cả văn bản và âm thanh cho trợ lý giọng nói)
sessionOptions.Modalities.Clear();
sessionOptions.Modalities.Add(InteractionModality.Text);
sessionOptions.Modalities.Add(InteractionModality.Audio);

await session.ConfigureSessionAsync(sessionOptions);
```

### 2. Xử lý Sự kiện

```csharp
await foreach (SessionUpdate serverEvent in session.GetUpdatesAsync())
{
    switch (serverEvent)
    {
        case SessionUpdateResponseAudioDelta audioDelta:
            byte[] audioData = audioDelta.Delta.ToArray();
            // Phát âm thanh qua NAudio hoặc thư viện âm thanh khác
            break;

        case SessionUpdateResponseTextDelta textDelta:
            Console.Write(textDelta.Delta);
            break;

        case SessionUpdateResponseFunctionCallArgumentsDone functionCall:
            // Xử lý gọi hàm (xem phần Function Calling)
            break;

        case SessionUpdateError error:
            Console.WriteLine($"Lỗi: {error.Error.Message}");
            break;

        case SessionUpdateResponseDone:
            Console.WriteLine("\n--- Phản hồi hoàn tất ---");
            break;
    }
}
```

### 3. Gửi Tin nhắn Người dùng

```csharp
await session.AddItemAsync(new UserMessageItem("Xin chào, bạn có thể giúp tôi không?"));
await session.StartResponseAsync();
```

### 4. Gọi Hàm (Function Calling)

```csharp
// Định nghĩa hàm
var weatherFunction = new VoiceLiveFunctionDefinition("get_current_weather")
{
    Description = "Lấy thời tiết hiện tại cho một địa điểm cụ thể",
    Parameters = BinaryData.FromString("""
        {
            "type": "object",
            "properties": {
                "location": {
                    "type": "string",
                    "description": "Thành phố và bang hoặc quốc gia"
                }
            },
            "required": ["location"]
        }
        """)
};

// Thêm vào tùy chọn phiên
sessionOptions.Tools.Add(weatherFunction);

// Xử lý gọi hàm trong vòng lặp sự kiện
if (serverEvent is SessionUpdateResponseFunctionCallArgumentsDone functionCall)
{
    if (functionCall.Name == "get_current_weather")
    {
        var parameters = JsonSerializer.Deserialize<Dictionary<string, string>>(functionCall.Arguments);
        string location = parameters?["location"] ?? "";

        // Gọi dịch vụ bên ngoài
        string weatherInfo = $"Thời tiết tại {location} là nắng, 75°F.";

        // Gửi phản hồi
        await session.AddItemAsync(new FunctionCallOutputItem(functionCall.CallId, weatherInfo));
        await session.StartResponseAsync();
    }
}
```

## Tùy chọn Giọng nói

| Loại Giọng nói | Class                | Ví dụ                              |
| -------------- | -------------------- | ---------------------------------- |
| Azure Standard | `AzureStandardVoice` | `"en-US-AvaNeural"`                |
| Azure HD       | `AzureStandardVoice` | `"en-US-Ava:DragonHDLatestNeural"` |
| Azure Custom   | `AzureCustomVoice`   | Giọng tùy chỉnh với endpoint ID    |

## Các Mô hình Hỗ trợ

| Mô hình                        | Mô tả                              |
| ------------------------------ | ---------------------------------- |
| `gpt-4o-realtime-preview`      | GPT-4o với âm thanh thời gian thực |
| `gpt-4o-mini-realtime-preview` | Tương tác nhẹ, nhanh               |
| `phi4-mm-realtime`             | Đa phương thức tiết kiệm chi phí   |

## Tài liệu tham khảo các Loại Chính

| Loại                              | Mục đích                                |
| --------------------------------- | --------------------------------------- |
| `VoiceLiveClient`                 | Client chính để tạo phiên               |
| `VoiceLiveSession`                | Phiên WebSocket đang hoạt động          |
| `VoiceLiveSessionOptions`         | Cấu hình phiên                          |
| `AzureStandardVoice`              | Nhà cung cấp giọng nói Azure tiêu chuẩn |
| `AzureSemanticVadTurnDetection`   | Phát hiện hoạt động giọng nói (VAD)     |
| `VoiceLiveFunctionDefinition`     | Định nghĩa công cụ hàm                  |
| `UserMessageItem`                 | Tin nhắn văn bản người dùng             |
| `FunctionCallOutputItem`          | Phản hồi gọi hàm                        |
| `SessionUpdateResponseAudioDelta` | Sự kiện chunk âm thanh                  |
| `SessionUpdateResponseTextDelta`  | Sự kiện chunk văn bản                   |

## Thực hành Tốt nhất

1. **Luôn đặt cả hai modality** — Bao gồm `Text` và `Audio` cho trợ lý giọng nói
2. **Sử dụng `AzureSemanticVadTurnDetection`** — Cung cấp dòng hội thoại tự nhiên
3. **Cấu hình thời gian im lặng phù hợp** — Thường là 500ms để tránh ngắt sớm
4. **Sử dụng câu lệnh `using`** — Đảm bảo giải phóng phiên đúng cách
5. **Xử lý tất cả các loại sự kiện** — Kiểm tra lỗi, âm thanh, văn bản và gọi hàm
6. **Sử dụng DefaultAzureCredential** — Không bao giờ hardcode API keys

## Xử lý Lỗi

```csharp
if (serverEvent is SessionUpdateError error)
{
    if (error.Error.Message.Contains("Cancellation failed: no active response"))
    {
        // Lỗi lành tính, có thể bỏ qua
    }
    else
    {
        Console.WriteLine($"Lỗi: {error.Error.Message}");
    }
}
```

## Cấu hình Âm thanh

- **Định dạng Đầu vào**: `InputAudioFormat.Pcm16` (16-bit PCM)
- **Định dạng Đầu ra**: `OutputAudioFormat.Pcm16`
- **Tần số Lấy mẫu**: Khuyên dùng 24kHz
- **Kênh**: Mono

## SDK Liên quan

| SDK                                  | Mục đích                           | Cài đặt                                                 |
| ------------------------------------ | ---------------------------------- | ------------------------------------------------------- |
| `Azure.AI.VoiceLive`                 | Giọng nói thời gian thực (SDK này) | `dotnet add package Azure.AI.VoiceLive`                 |
| `Microsoft.CognitiveServices.Speech` | Speech-to-text, text-to-speech     | `dotnet add package Microsoft.CognitiveServices.Speech` |
| `NAudio`                             | Thu/phát âm thanh                  | `dotnet add package NAudio`                             |

## Liên kết Tham khảo

| Tài nguyên    | URL                                                                                |
| ------------- | ---------------------------------------------------------------------------------- |
| NuGet Package | https://www.nuget.org/packages/Azure.AI.VoiceLive                                  |
| API Reference | https://learn.microsoft.com/dotnet/api/azure.ai.voicelive                          |
| GitHub Source | https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/ai/Azure.AI.VoiceLive     |
| Quickstart    | https://learn.microsoft.com/azure/ai-services/speech-service/voice-live-quickstart |
