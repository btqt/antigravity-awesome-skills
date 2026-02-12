---
name: azure-communication-callingserver-java
description: Azure Communication Services CallingServer (cũ) Java SDK. Lưu ý - SDK này đã bị phản đối (deprecated). Sử dụng azure-communication-callautomation thay thế cho các dự án mới. Chỉ sử dụng kỹ năng này khi bảo trì mã cũ.
package: com.azure:azure-communication-callingserver
---

# Azure Communication CallingServer (Java) - ĐÃ BỊ PHẢN ĐỐI (DEPRECATED)

> **⚠️ ĐÃ BỊ PHẢN ĐỐI**: SDK này đã được đổi tên thành **Call Automation**. Đối với các dự án mới, hãy sử dụng `azure-communication-callautomation`. Kỹ năng này chỉ dành cho việc bảo trì mã cũ.

## Di chuyển sang Call Automation

```xml
<!-- CŨ (đã bị phản đối) -->
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-communication-callingserver</artifactId>
    <version>1.0.0-beta.5</version>
</dependency>

<!-- MỚI (sử dụng cái này thay thế) -->
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-communication-callautomation</artifactId>
    <version>1.6.0</version>
</dependency>
```

## Thay đổi Tên Lớp

| CallingServer (Cũ)           | Call Automation (Mới)             |
| ---------------------------- | --------------------------------- |
| `CallingServerClient`        | `CallAutomationClient`            |
| `CallingServerClientBuilder` | `CallAutomationClientBuilder`     |
| `CallConnection`             | `CallConnection` (như cũ)         |
| `ServerCall`                 | Đã xóa - sử dụng `CallConnection` |

## Tạo Client Cũ

```java
// CÁCH CŨ (đã bị phản đối)
import com.azure.communication.callingserver.CallingServerClient;
import com.azure.communication.callingserver.CallingServerClientBuilder;

CallingServerClient client = new CallingServerClientBuilder()
    .connectionString("<connection-string>")
    .buildClient();

// CÁCH MỚI
import com.azure.communication.callautomation.CallAutomationClient;
import com.azure.communication.callautomation.CallAutomationClientBuilder;

CallAutomationClient client = new CallAutomationClientBuilder()
    .connectionString("<connection-string>")
    .buildClient();
```

## Ghi âm Cũ

```java
// CÁCH CŨ
StartRecordingOptions options = new StartRecordingOptions(serverCallId)
    .setRecordingStateCallbackUri(callbackUri);

StartCallRecordingResult result = client.startRecording(options);
String recordingId = result.getRecordingId();

client.pauseRecording(recordingId);
client.resumeRecording(recordingId);
client.stopRecording(recordingId);

// CÁCH MỚI - xem kỹ năng azure-communication-callautomation
```

## Cho Phát triển Mới

**Không sử dụng SDK này cho các dự án mới.**

Xem kỹ năng `azure-communication-callautomation-java` để biết về:

- Thực hiện cuộc gọi đi
- Trả lời cuộc gọi đến
- Ghi âm cuộc gọi
- Nhận dạng DTMF
- Chuyển văn bản thành giọng nói / chuyển giọng nói thành văn bản
- Thêm/xóa người tham gia
- Chuyển cuộc gọi

## Các Cụm từ Kích hoạt

- "callingserver legacy", "deprecated calling SDK"
- "migrate callingserver to callautomation"
