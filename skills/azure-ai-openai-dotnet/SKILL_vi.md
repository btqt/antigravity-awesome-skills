---
name: azure-ai-openai-dotnet
description: |
  Azure OpenAI SDK cho .NET. Thư viện client cho các dịch vụ Azure OpenAI và OpenAI. Sử dụng cho chat completions, embeddings, tạo hình ảnh, phiên âm âm thanh và assistants. Kích hoạt: "Azure OpenAI", "AzureOpenAIClient", "ChatClient", "chat completions .NET", "GPT-4", "embeddings", "DALL-E", "Whisper", "OpenAI .NET".
package: Azure.AI.OpenAI
---

# Azure.AI.OpenAI (.NET)

Thư viện client cho Dịch vụ Azure OpenAI cung cấp quyền truy cập vào các mô hình OpenAI bao gồm GPT-4, GPT-4o, embeddings, DALL-E và Whisper.

## Cài đặt

```bash
dotnet add package Azure.AI.OpenAI

# Để tương thích với OpenAI (không phải Azure)
dotnet add package OpenAI
```

**Phiên bản Hiện tại**: 2.1.0 (ổn định)

## Biến Môi trường

```bash
AZURE_OPENAI_ENDPOINT=https://<resource-name>.openai.azure.com
AZURE_OPENAI_API_KEY=<api-key>                    # Cho xác thực dựa trên key
AZURE_OPENAI_DEPLOYMENT_NAME=gpt-4o-mini          # Tên deployment của bạn
```

## Phân cấp Client

```
AzureOpenAIClient (top-level)
├── GetChatClient(deploymentName)      → ChatClient
├── GetEmbeddingClient(deploymentName) → EmbeddingClient
├── GetImageClient(deploymentName)     → ImageClient
├── GetAudioClient(deploymentName)     → AudioClient
└── GetAssistantClient()               → AssistantClient
```

## Xác thực

### Xác thực API Key

```csharp
using Azure;
using Azure.AI.OpenAI;

AzureOpenAIClient client = new(
    new Uri(Environment.GetEnvironmentVariable("AZURE_OPENAI_ENDPOINT")!),
    new AzureKeyCredential(Environment.GetEnvironmentVariable("AZURE_OPENAI_API_KEY")!));
```

### Microsoft Entra ID (Khuyên dùng cho Sản xuất)

```csharp
using Azure.Identity;
using Azure.AI.OpenAI;

AzureOpenAIClient client = new(
    new Uri(Environment.GetEnvironmentVariable("AZURE_OPENAI_ENDPOINT")!),
    new DefaultAzureCredential());
```

### Sử dụng trực tiếp OpenAI SDK với Azure

```csharp
using Azure.Identity;
using OpenAI;
using OpenAI.Chat;
using System.ClientModel.Primitives;

#pragma warning disable OPENAI001

BearerTokenPolicy tokenPolicy = new(
    new DefaultAzureCredential(),
    "https://cognitiveservices.azure.com/.default");

ChatClient client = new(
    model: "gpt-4o-mini",
    authenticationPolicy: tokenPolicy,
    options: new OpenAIClientOptions()
    {
        Endpoint = new Uri("https://YOUR-RESOURCE.openai.azure.com/openai/v1")
    });
```

## Chat Completions

### Chat Cơ bản

```csharp
using Azure.AI.OpenAI;
using OpenAI.Chat;

AzureOpenAIClient azureClient = new(
    new Uri(endpoint),
    new DefaultAzureCredential());

ChatClient chatClient = azureClient.GetChatClient("gpt-4o-mini");

ChatCompletion completion = chatClient.CompleteChat(
[
    new SystemChatMessage("Bạn là một trợ lý hữu ích."),
    new UserChatMessage("Azure OpenAI là gì?")
]);

Console.WriteLine(completion.Content[0].Text);
```

### Chat Bất đồng bộ (Async Chat)

```csharp
ChatCompletion completion = await chatClient.CompleteChatAsync(
[
    new SystemChatMessage("Bạn là một trợ lý hữu ích."),
    new UserChatMessage("Giải thích về điện toán đám mây bằng từ ngữ đơn giản.")
]);

Console.WriteLine($"Phản hồi: {completion.Content[0].Text}");
Console.WriteLine($"Token đã sử dụng: {completion.Usage.TotalTokenCount}");
```

### Chat Streaming

```csharp
await foreach (StreamingChatCompletionUpdate update
    in chatClient.CompleteChatStreamingAsync(messages))
{
    if (update.ContentUpdate.Count > 0)
    {
        Console.Write(update.ContentUpdate[0].Text);
    }
}
```

### Chat với Tùy chọn

```csharp
ChatCompletionOptions options = new()
{
    MaxOutputTokenCount = 1000,
    Temperature = 0.7f,
    TopP = 0.95f,
    FrequencyPenalty = 0,
    PresencePenalty = 0
};

ChatCompletion completion = await chatClient.CompleteChatAsync(messages, options);
```

### Hội thoại Nhiều lượt (Multi-turn Conversation)

```csharp
List<ChatMessage> messages = new()
{
    new SystemChatMessage("Bạn là một trợ lý hữu ích."),
    new UserChatMessage("Chào, bạn có thể giúp tôi không?"),
    new AssistantChatMessage("Tất nhiên! Bạn cần giúp gì?"),
    new UserChatMessage("Thủ đô của Pháp là gì?")
};

ChatCompletion completion = await chatClient.CompleteChatAsync(messages);
messages.Add(new AssistantChatMessage(completion.Content[0].Text));
```

## Đầu ra Cấu trúc (JSON Schema)

```csharp
using System.Text.Json;

ChatCompletionOptions options = new()
{
    ResponseFormat = ChatResponseFormat.CreateJsonSchemaFormat(
        jsonSchemaFormatName: "math_reasoning",
        jsonSchema: BinaryData.FromBytes("""
            {
                "type": "object",
                "properties": {
                    "steps": {
                        "type": "array",
                        "items": {
                            "type": "object",
                            "properties": {
                                "explanation": { "type": "string" },
                                "output": { "type": "string" }
                            },
                            "required": ["explanation", "output"],
                            "additionalProperties": false
                        }
                    },
                    "final_answer": { "type": "string" }
                },
                "required": ["steps", "final_answer"],
                "additionalProperties": false
            }
            """u8.ToArray()),
        jsonSchemaIsStrict: true)
};

ChatCompletion completion = await chatClient.CompleteChatAsync(
    [new UserChatMessage("Làm thế nào để giải 8x + 7 = -23?")],
    options);

using JsonDocument json = JsonDocument.Parse(completion.Content[0].Text);
Console.WriteLine($"Câu trả lời: {json.RootElement.GetProperty("final_answer")}");
```

## Các Mô hình Suy luận (Reasoning Models) (o1, o4-mini)

```csharp
ChatCompletionOptions options = new()
{
    ReasoningEffortLevel = ChatReasoningEffortLevel.Low,
    MaxOutputTokenCount = 100000
};

ChatCompletion completion = await chatClient.CompleteChatAsync(
[
    new DeveloperChatMessage("Bạn là một trợ lý hữu ích"),
    new UserChatMessage("Giải thích thuyết tương đối")
], options);
```

## Tích hợp Azure AI Search (RAG)

```csharp
using Azure.AI.OpenAI.Chat;

#pragma warning disable AOAI001

ChatCompletionOptions options = new();
options.AddDataSource(new AzureSearchChatDataSource()
{
    Endpoint = new Uri(searchEndpoint),
    IndexName = searchIndex,
    Authentication = DataSourceAuthentication.FromApiKey(searchKey)
});

ChatCompletion completion = await chatClient.CompleteChatAsync(
    [new UserChatMessage("Có những chương trình sức khỏe nào?")],
    options);

ChatMessageContext context = completion.GetMessageContext();
if (context?.Intent is not null)
{
    Console.WriteLine($"Ý định: {context.Intent}");
}
foreach (ChatCitation citation in context?.Citations ?? [])
{
    Console.WriteLine($"Trích dẫn: {citation.Content}");
}
```

## Embeddings

```csharp
using OpenAI.Embeddings;

EmbeddingClient embeddingClient = azureClient.GetEmbeddingClient("text-embedding-ada-002");

OpenAIEmbedding embedding = await embeddingClient.GenerateEmbeddingAsync("Xin chào thế giới!");
ReadOnlyMemory<float> vector = embedding.ToFloats();

Console.WriteLine($"Kích thước embedding: {vector.Length}");
```

### Batch Embeddings

```csharp
List<string> inputs = new()
{
    "Văn bản tài liệu thứ nhất",
    "Văn bản tài liệu thứ hai",
    "Văn bản tài liệu thứ ba"
};

OpenAIEmbeddingCollection embeddings = await embeddingClient.GenerateEmbeddingsAsync(inputs);

foreach (OpenAIEmbedding emb in embeddings)
{
    Console.WriteLine($"Chỉ số {emb.Index}: {emb.ToFloats().Length} chiều");
}
```

## Tạo Hình ảnh (DALL-E)

```csharp
using OpenAI.Images;

ImageClient imageClient = azureClient.GetImageClient("dall-e-3");

GeneratedImage image = await imageClient.GenerateImageAsync(
    "Đường chân trời thành phố tương lai lúc hoàng hôn",
    new ImageGenerationOptions
    {
        Size = GeneratedImageSize.W1024xH1024,
        Quality = GeneratedImageQuality.High,
        Style = GeneratedImageStyle.Vivid
    });

Console.WriteLine($"URL Hình ảnh: {image.ImageUri}");
```

## Âm thanh (Whisper)

### Phiên âm (Transcription)

```csharp
using OpenAI.Audio;

AudioClient audioClient = azureClient.GetAudioClient("whisper");

AudioTranscription transcription = await audioClient.TranscribeAudioAsync(
    "audio.mp3",
    new AudioTranscriptionOptions
    {
        ResponseFormat = AudioTranscriptionFormat.Verbose,
        Language = "en"
    });

Console.WriteLine(transcription.Text);
```

### Chuyển văn bản thành giọng nói (Text-to-Speech)

```csharp
BinaryData speech = await audioClient.GenerateSpeechAsync(
    "Xin chào, chào mừng đến với Azure OpenAI!",
    GeneratedSpeechVoice.Alloy,
    new SpeechGenerationOptions
    {
        SpeedRatio = 1.0f,
        ResponseFormat = GeneratedSpeechFormat.Mp3
    });

await File.WriteAllBytesAsync("output.mp3", speech.ToArray());
```

## Gọi Hàm (Function Calling)

```csharp
ChatTool getCurrentWeatherTool = ChatTool.CreateFunctionTool(
    functionName: "get_current_weather",
    functionDescription: "Lấy thời tiết hiện tại ở một địa điểm nhất định",
    functionParameters: BinaryData.FromString("""
        {
            "type": "object",
            "properties": {
                "location": {
                    "type": "string",
                    "description": "Thành phố và bang, ví dụ: San Francisco, CA"
                },
                "unit": {
                    "type": "string",
                    "enum": ["celsius", "fahrenheit"]
                }
            },
            "required": ["location"]
        }
        """));

ChatCompletionOptions options = new()
{
    Tools = { getCurrentWeatherTool }
};

ChatCompletion completion = await chatClient.CompleteChatAsync(
    [new UserChatMessage("Thời tiết ở Seattle thế nào?")],
    options);

if (completion.FinishReason == ChatFinishReason.ToolCalls)
{
    foreach (ChatToolCall toolCall in completion.ToolCalls)
    {
        Console.WriteLine($"Hàm: {toolCall.FunctionName}");
        Console.WriteLine($"Tham số: {toolCall.FunctionArguments}");
    }
}
```

## Tài liệu tham khảo các Loại Chính

| Loại                            | Mục đích                                   |
| ------------------------------- | ------------------------------------------ |
| `AzureOpenAIClient`             | Client cấp cao nhất cho Azure OpenAI       |
| `ChatClient`                    | Chat completions                           |
| `EmbeddingClient`               | Text embeddings                            |
| `ImageClient`                   | Tạo hình ảnh (DALL-E)                      |
| `AudioClient`                   | Phiên âm âm thanh/TTS                      |
| `ChatCompletion`                | Phản hồi chat                              |
| `ChatCompletionOptions`         | Cấu hình yêu cầu                           |
| `StreamingChatCompletionUpdate` | Phần phản hồi streaming                    |
| `ChatMessage`                   | Loại tin nhắn cơ bản                       |
| `SystemChatMessage`             | System prompt                              |
| `UserChatMessage`               | Đầu vào người dùng                         |
| `AssistantChatMessage`          | Phản hồi trợ lý                            |
| `DeveloperChatMessage`          | Tin nhắn nhà phát triển (mô hình suy luận) |
| `ChatTool`                      | Định nghĩa hàm/công cụ                     |
| `ChatToolCall`                  | Yêu cầu gọi công cụ                        |

## Thực hành Tốt nhất

1. **Sử dụng Entra ID trong sản xuất** — Tránh API keys; sử dụng `DefaultAzureCredential`
2. **Tái sử dụng các instance client** — Tạo một lần, chia sẻ qua các yêu cầu
3. **Xử lý giới hạn tốc độ** — Triển khai exponential backoff cho lỗi 429
4. **Stream cho các phản hồi dài** — Sử dụng `CompleteChatStreamingAsync` cho trải nghiệm người dùng tốt hơn
5. **Đặt timeout phù hợp** — Các completion dài có thể cần timeout mở rộng
6. **Sử dụng đầu ra cấu trúc** — JSON schema đảm bảo định dạng phản hồi nhất quán
7. **Giám sát sử dụng token** — Theo dõi `completion.Usage` để quản lý chi phí
8. **Xác thực gọi công cụ** — Luôn xác thực tham số hàm trước khi thực thi

## Xử lý Lỗi

```csharp
using Azure;

try
{
    ChatCompletion completion = await chatClient.CompleteChatAsync(messages);
}
catch (RequestFailedException ex) when (ex.Status == 429)
{
    Console.WriteLine("Bị giới hạn tốc độ. Thử lại sau một khoảng thời gian.");
    await Task.Delay(TimeSpan.FromSeconds(10));
}
catch (RequestFailedException ex) when (ex.Status == 400)
{
    Console.WriteLine($"Yêu cầu không hợp lệ: {ex.Message}");
}
catch (RequestFailedException ex)
{
    Console.WriteLine($"Lỗi Azure OpenAI: {ex.Status} - {ex.Message}");
}
```
