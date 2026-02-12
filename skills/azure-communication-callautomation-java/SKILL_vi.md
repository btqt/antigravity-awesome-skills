---
name: azure-communication-callautomation-java
description: Xây dựng các quy trình tự động hóa cuộc gọi với Azure Communication Services Call Automation Java SDK. Sử dụng khi triển khai các hệ thống IVR, định tuyến cuộc gọi, ghi âm cuộc gọi, nhận dạng DTMF, chuyển văn bản thành giọng nói (text-to-speech), hoặc các luồng cuộc gọi được hỗ trợ bởi AI.
package: com.azure:azure-communication-callautomation
---

# Azure Communication Call Automation (Java)

Xây dựng các quy trình tự động hóa cuộc gọi phía server bao gồm hệ thống IVR, định tuyến cuộc gọi, ghi âm, và các tương tác được hỗ trợ bởi AI.

## Cài đặt

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-communication-callautomation</artifactId>
    <version>1.6.0</version>
</dependency>
```

## Tạo Client

```java
import com.azure.communication.callautomation.CallAutomationClient;
import com.azure.communication.callautomation.CallAutomationClientBuilder;
import com.azure.identity.DefaultAzureCredentialBuilder;

// Với DefaultAzureCredential
CallAutomationClient client = new CallAutomationClientBuilder()
    .endpoint("https://<resource>.communication.azure.com")
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildClient();

// Với connection string
CallAutomationClient client = new CallAutomationClientBuilder()
    .connectionString("<connection-string>")
    .buildClient();
```

## Các Khái niệm Chính

| Lớp                         | Mục đích                                                                  |
| --------------------------- | ------------------------------------------------------------------------- |
| `CallAutomationClient`      | Thực hiện cuộc gọi, trả lời/từ chối cuộc gọi đến, chuyển hướng cuộc gọi   |
| `CallConnection`            | Các hành động trong cuộc gọi đã thiết lập (thêm người tham gia, kết thúc) |
| `CallMedia`                 | Các thao tác media (phát âm thanh, nhận dạng DTMF/giọng nói)              |
| `CallRecording`             | Bắt đầu/dừng/tạm dừng ghi âm                                              |
| `CallAutomationEventParser` | Phân tích các sự kiện webhook từ ACS                                      |

## Tạo Cuộc gọi Đi (Outbound Call)

```java
import com.azure.communication.callautomation.models.*;
import com.azure.communication.common.CommunicationUserIdentifier;
import com.azure.communication.common.PhoneNumberIdentifier;

// Gọi đến số PSTN
PhoneNumberIdentifier target = new PhoneNumberIdentifier("+14255551234");
PhoneNumberIdentifier caller = new PhoneNumberIdentifier("+14255550100");

CreateCallOptions options = new CreateCallOptions(
    new CommunicationUserIdentifier("<user-id>"),  // Nguồn
    List.of(target))                                // Đích
    .setSourceCallerId(caller)
    .setCallbackUrl("https://your-app.com/api/callbacks");

CreateCallResult result = client.createCall(options);
String callConnectionId = result.getCallConnectionProperties().getCallConnectionId();
```

## Trả lời Cuộc gọi Đến (Answer Incoming Call)

```java
// Từ webhook Event Grid - sự kiện IncomingCall
String incomingCallContext = "<incoming-call-context-from-event>";

AnswerCallOptions options = new AnswerCallOptions(
    incomingCallContext,
    "https://your-app.com/api/callbacks");

AnswerCallResult result = client.answerCall(options);
CallConnection callConnection = result.getCallConnection();
```

## Phát Âm thanh (Text-to-Speech)

```java
CallConnection callConnection = client.getCallConnection(callConnectionId);
CallMedia callMedia = callConnection.getCallMedia();

// Phát text-to-speech
TextSource textSource = new TextSource()
    .setText("Chào mừng đến với Contoso. Nhấn phím 1 để gặp bộ phận bán hàng, phím 2 để gặp hỗ trợ.")
    .setVoiceName("vi-VN-HoaiMyNeural"); // Ví dụ giọng tiếng Việt

PlayOptions playOptions = new PlayOptions(
    List.of(textSource),
    List.of(new CommunicationUserIdentifier("<target-user>")));

callMedia.play(playOptions);

// Phát tệp âm thanh
FileSource fileSource = new FileSource()
    .setUrl("https://storage.blob.core.windows.net/audio/greeting.wav");

callMedia.play(new PlayOptions(List.of(fileSource), List.of(target)));
```

## Nhận dạng Đầu vào DTMF

```java
// Nhận dạng các âm DTMF
DtmfTone stopTones = DtmfTone.POUND;

CallMediaRecognizeDtmfOptions recognizeOptions = new CallMediaRecognizeDtmfOptions(
    new CommunicationUserIdentifier("<target-user>"),
    5)  // Tối đa 5 âm
    .setInterToneTimeout(Duration.ofSeconds(5))
    .setStopTones(List.of(stopTones))
    .setInitialSilenceTimeout(Duration.ofSeconds(15))
    .setPlayPrompt(new TextSource().setText("Nhập số tài khoản của bạn theo sau là phím thăng."));

callMedia.startRecognizing(recognizeOptions);
```

## Nhận dạng Giọng nói (Speech Recognition)

```java
// Nhận dạng giọng nói với AI
CallMediaRecognizeSpeechOptions speechOptions = new CallMediaRecognizeSpeechOptions(
    new CommunicationUserIdentifier("<target-user>"))
    .setEndSilenceTimeout(Duration.ofSeconds(2))
    .setSpeechLanguage("en-US")
    .setPlayPrompt(new TextSource().setText("Tôi có thể giúp gì cho bạn hôm nay?"));

callMedia.startRecognizing(speechOptions);
```

## Ghi âm Cuộc gọi

```java
CallRecording callRecording = client.getCallRecording();

// Bắt đầu ghi âm
StartRecordingOptions recordingOptions = new StartRecordingOptions(
    new ServerCallLocator("<server-call-id>"))
    .setRecordingChannel(RecordingChannel.MIXED)
    .setRecordingContent(RecordingContent.AUDIO_VIDEO)
    .setRecordingFormat(RecordingFormat.MP4);

RecordingStateResult recordingResult = callRecording.start(recordingOptions);
String recordingId = recordingResult.getRecordingId();

// Tạm dừng/tiếp tục/dừng
callRecording.pause(recordingId);
callRecording.resume(recordingId);
callRecording.stop(recordingId);

// Tải xuống bản ghi (sau sự kiện RecordingFileStatusUpdated)
callRecording.downloadTo(recordingUrl, Paths.get("recording.mp4"));
```

## Thêm Người tham gia vào Cuộc gọi

```java
CallConnection callConnection = client.getCallConnection(callConnectionId);

CommunicationUserIdentifier participant = new CommunicationUserIdentifier("<user-id>");
AddParticipantOptions addOptions = new AddParticipantOptions(participant)
    .setInvitationTimeout(Duration.ofSeconds(30));

AddParticipantResult result = callConnection.addParticipant(addOptions);
```

## Chuyển Cuộc gọi (Transfer Call)

```java
// Chuyển cuộc gọi mù (Blind transfer)
PhoneNumberIdentifier transferTarget = new PhoneNumberIdentifier("+14255559999");
TransferCallToParticipantResult result = callConnection.transferCallToParticipant(transferTarget);
```

## Xử lý Sự kiện (Webhook)

```java
import com.azure.communication.callautomation.CallAutomationEventParser;
import com.azure.communication.callautomation.models.events.*;

// Trong endpoint webhook của bạn
public void handleCallback(String requestBody) {
    List<CallAutomationEventBase> events = CallAutomationEventParser.parseEvents(requestBody);

    for (CallAutomationEventBase event : events) {
        if (event instanceof CallConnected) {
            CallConnected connected = (CallConnected) event;
            System.out.println("Cuộc gọi đã kết nối: " + connected.getCallConnectionId());
        } else if (event instanceof RecognizeCompleted) {
            RecognizeCompleted recognized = (RecognizeCompleted) event;
            // Xử lý kết quả nhận dạng DTMF hoặc giọng nói
            DtmfResult dtmfResult = (DtmfResult) recognized.getRecognizeResult();
            String tones = dtmfResult.getTones().stream()
                .map(DtmfTone::toString)
                .collect(Collectors.joining());
            System.out.println("DTMF đã nhận: " + tones);
        } else if (event instanceof PlayCompleted) {
            System.out.println("Phát lại âm thanh hoàn tất");
        } else if (event instanceof CallDisconnected) {
            System.out.println("Cuộc gọi đã kết thúc");
        }
    }
}
```

## Ngắt Cuộc gọi (Hang Up)

```java
// Ngắt cuộc gọi cho tất cả người tham gia
callConnection.hangUp(true);

// Chỉ ngắt nhánh cuộc gọi này
callConnection.hangUp(false);
```

## Xử lý Lỗi

```java
import com.azure.core.exception.HttpResponseException;

try {
    client.answerCall(options);
} catch (HttpResponseException e) {
    if (e.getResponse().getStatusCode() == 404) {
        System.out.println("Cuộc gọi không tìm thấy hoặc đã kết thúc");
    } else if (e.getResponse().getStatusCode() == 400) {
        System.out.println("Yêu cầu không hợp lệ: " + e.getMessage());
    }
}
```

## Biến Môi trường

```bash
AZURE_COMMUNICATION_ENDPOINT=https://<resource>.communication.azure.com
AZURE_COMMUNICATION_CONNECTION_STRING=endpoint=https://...;accesskey=...
CALLBACK_BASE_URL=https://your-app.com/api/callbacks
```

## Các Cụm từ Kích hoạt

- "call automation Java", "IVR Java", "tương tác giọng nói tương tác"
- "ghi âm cuộc gọi Java", "nhận dạng DTMF Java"
- "text to speech call", "nhận dạng giọng nói call"
- "trả lời cuộc gọi đến", "chuyển cuộc gọi Java"
- "Azure Communication Services call automation"
