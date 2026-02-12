---
name: azure-ai-voicelive-java
description: |
  Azure AI VoiceLive SDK cho Java. Hội thoại giọng nói thời gian thực, hai chiều với trợ lý AI sử dụng WebSocket.
  Kích hoạt: "VoiceLiveClient java", "voice assistant java", "giọng nói thời gian thực java", "audio streaming java", "voice activity detection java".
package: com.azure:azure-ai-voicelive
---

# Azure AI VoiceLive SDK cho Java

Hội thoại giọng nói thời gian thực, hai chiều với trợ lý AI sử dụng công nghệ WebSocket.

## Cài đặt

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-ai-voicelive</artifactId>
    <version>1.0.0-beta.2</version>
</dependency>
```

## Biến Môi trường

```bash
AZURE_VOICELIVE_ENDPOINT=https://<resource>.openai.azure.com/
AZURE_VOICELIVE_API_KEY=<your-api-key>
```

## Xác thực

### API Key

```java
import com.azure.ai.voicelive.VoiceLiveAsyncClient;
import com.azure.ai.voicelive.VoiceLiveClientBuilder;
import com.azure.core.credential.AzureKeyCredential;

VoiceLiveAsyncClient client = new VoiceLiveClientBuilder()
    .endpoint(System.getenv("AZURE_VOICELIVE_ENDPOINT"))
    .credential(new AzureKeyCredential(System.getenv("AZURE_VOICELIVE_API_KEY")))
    .buildAsyncClient();
```

### DefaultAzureCredential (Khuyên dùng)

```java
import com.azure.identity.DefaultAzureCredentialBuilder;

VoiceLiveAsyncClient client = new VoiceLiveClientBuilder()
    .endpoint(System.getenv("AZURE_VOICELIVE_ENDPOINT"))
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildAsyncClient();
```

## Các Khái niệm Chính

| Khái niệm                     | Mô tả                                         |
| ----------------------------- | --------------------------------------------- |
| `VoiceLiveAsyncClient`        | Điểm nhập chính cho các phiên giọng nói       |
| `VoiceLiveSessionAsyncClient` | Kết nối WebSocket đang hoạt động để streaming |
| `VoiceLiveSessionOptions`     | Cấu hình cho hành vi phiên                    |

### Yêu cầu Âm thanh

- **Tần số Lấy mẫu**: 24kHz (24000 Hz)
- **Độ sâu Bit**: 16-bit PCM
- **Kênh**: Mono (1 kênh)
- **Định dạng**: Signed PCM, little-endian

## Quy trình Cốt lõi

### 1. Bắt đầu Phiên

```java
import reactor.core.publisher.Mono;

client.startSession("gpt-4o-realtime-preview")
    .flatMap(session -> {
        System.out.println("Session started");

        // Đăng ký nhận sự kiện
        session.receiveEvents()
            .subscribe(
                event -> System.out.println("Event: " + event.getType()),
                error -> System.err.println("Error: " + error.getMessage())
            );

        return Mono.just(session);
    })
    .block();
```

### 2. Cấu hình Tùy chọn Phiên

```java
import com.azure.ai.voicelive.models.*;
import java.util.Arrays;

ServerVadTurnDetection turnDetection = new ServerVadTurnDetection()
    .setThreshold(0.5)                    // Độ nhạy (0.0-1.0)
    .setPrefixPaddingMs(300)              // Âm thanh trước khi nói
    .setSilenceDurationMs(500)            // Khoảng lặng để kết thúc lượt nói
    .setInterruptResponse(true)           // Cho phép ngắt lời
    .setAutoTruncate(true)
    .setCreateResponse(true);

AudioInputTranscriptionOptions transcription = new AudioInputTranscriptionOptions(
    AudioInputTranscriptionOptionsModel.WHISPER_1);

VoiceLiveSessionOptions options = new VoiceLiveSessionOptions()
    .setInstructions("Bạn là một trợ lý giọng nói AI hữu ích.")
    .setVoice(BinaryData.fromObject(new OpenAIVoice(OpenAIVoiceName.ALLOY)))
    .setModalities(Arrays.asList(InteractionModality.TEXT, InteractionModality.AUDIO))
    .setInputAudioFormat(InputAudioFormat.PCM16)
    .setOutputAudioFormat(OutputAudioFormat.PCM16)
    .setInputAudioSamplingRate(24000)
    .setInputAudioNoiseReduction(new AudioNoiseReduction(AudioNoiseReductionType.NEAR_FIELD))
    .setInputAudioEchoCancellation(new AudioEchoCancellation())
    .setInputAudioTranscription(transcription)
    .setTurnDetection(turnDetection);

// Gửi cấu hình
ClientEventSessionUpdate updateEvent = new ClientEventSessionUpdate(options);
session.sendEvent(updateEvent).subscribe();
```

### 3. Gửi Đầu vào Âm thanh

```java
byte[] audioData = readAudioChunk(); // Dữ liệu âm thanh PCM16 của bạn
session.sendInputAudio(BinaryData.fromBytes(audioData)).subscribe();
```

### 4. Xử lý Sự kiện

```java
session.receiveEvents().subscribe(event -> {
    ServerEventType eventType = event.getType();

    if (ServerEventType.SESSION_CREATED.equals(eventType)) {
        System.out.println("Phiên đã tạo");
    } else if (ServerEventType.INPUT_AUDIO_BUFFER_SPEECH_STARTED.equals(eventType)) {
        System.out.println("Người dùng bắt đầu nói");
    } else if (ServerEventType.INPUT_AUDIO_BUFFER_SPEECH_STOPPED.equals(eventType)) {
        System.out.println("Người dùng dừng nói");
    } else if (ServerEventType.RESPONSE_AUDIO_DELTA.equals(eventType)) {
        if (event instanceof SessionUpdateResponseAudioDelta) {
            SessionUpdateResponseAudioDelta audioEvent = (SessionUpdateResponseAudioDelta) event;
            playAudioChunk(audioEvent.getDelta());
        }
    } else if (ServerEventType.RESPONSE_DONE.equals(eventType)) {
        System.out.println("Phản hồi hoàn tất");
    } else if (ServerEventType.ERROR.equals(eventType)) {
        if (event instanceof SessionUpdateError) {
            SessionUpdateError errorEvent = (SessionUpdateError) event;
            System.err.println("Lỗi: " + errorEvent.getError().getMessage());
        }
    }
});
```

## Cấu hình Giọng nói

### OpenAI Voices

```java
// Có sẵn: ALLOY, ASH, BALLAD, CORAL, ECHO, SAGE, SHIMMER, VERSE
VoiceLiveSessionOptions options = new VoiceLiveSessionOptions()
    .setVoice(BinaryData.fromObject(new OpenAIVoice(OpenAIVoiceName.ALLOY)));
```

### Azure Voices

```java
// Azure Standard Voice
options.setVoice(BinaryData.fromObject(new AzureStandardVoice("en-US-JennyNeural")));

// Azure Custom Voice
options.setVoice(BinaryData.fromObject(new AzureCustomVoice("myVoice", "endpointId")));

// Azure Personal Voice
options.setVoice(BinaryData.fromObject(
    new AzurePersonalVoice("speakerProfileId", PersonalVoiceModels.PHOENIX_LATEST_NEURAL)));
```

## Gọi Hàm (Function Calling)

```java
VoiceLiveFunctionDefinition weatherFunction = new VoiceLiveFunctionDefinition("get_weather")
    .setDescription("Lấy thời tiết hiện tại cho một địa điểm")
    .setParameters(BinaryData.fromObject(parametersSchema));

VoiceLiveSessionOptions options = new VoiceLiveSessionOptions()
    .setTools(Arrays.asList(weatherFunction))
    .setInstructions("Bạn có quyền truy cập thông tin thời tiết.");
```

## Thực hành Tốt nhất

1. **Sử dụng client async** — VoiceLive yêu cầu các pattern reactive
2. **Cấu hình phát hiện lượt nói** để có dòng hội thoại tự nhiên
3. **Bật giảm nhiễu** để nhận dạng giọng nói tốt hơn
4. **Xử lý ngắt lời** một cách uyển chuyển với `setInterruptResponse(true)`
5. **Sử dụng phiên âm Whisper** cho phiên âm âm thanh đầu vào
6. **Đóng phiên** đúng cách khi hội thoại kết thúc

## Xử lý Lỗi

```java
session.receiveEvents()
    .doOnError(error -> System.err.println("Lỗi kết nối: " + error.getMessage()))
    .onErrorResume(error -> {
        // Thử kết nối lại hoặc dọn dẹp
        return Flux.empty();
    })
    .subscribe();
```

## Liên kết Tham khảo

| Tài nguyên    | URL                                                                                         |
| ------------- | ------------------------------------------------------------------------------------------- |
| GitHub Source | https://github.com/Azure/azure-sdk-for-java/tree/main/sdk/ai/azure-ai-voicelive             |
| Samples       | https://github.com/Azure/azure-sdk-for-java/tree/main/sdk/ai/azure-ai-voicelive/src/samples |
