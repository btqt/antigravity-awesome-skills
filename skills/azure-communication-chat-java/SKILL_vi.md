---
name: azure-communication-chat-java
description: Xây dựng các ứng dụng chat thời gian thực với Azure Communication Services Chat Java SDK. Sử dụng khi triển khai các luồng chat, nhắn tin, người tham gia, thông báo đã đọc, thông báo đang nhập, hoặc các tính năng chat thời gian thực.
package: com.azure:azure-communication-chat
---

# Azure Communication Chat (Java)

Xây dựng các ứng dụng chat thời gian thực với quản lý luồng (thread management), nhắn tin, người tham gia, và thông báo đã đọc.

## Cài đặt

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-communication-chat</artifactId>
    <version>1.6.0</version>
</dependency>
```

## Tạo Client

```java
import com.azure.communication.chat.ChatClient;
import com.azure.communication.chat.ChatClientBuilder;
import com.azure.communication.chat.ChatThreadClient;
import com.azure.communication.common.CommunicationTokenCredential;

// ChatClient yêu cầu CommunicationTokenCredential (user access token)
String endpoint = "https://<resource>.communication.azure.com";
String userAccessToken = "<user-access-token>";

CommunicationTokenCredential credential = new CommunicationTokenCredential(userAccessToken);

ChatClient chatClient = new ChatClientBuilder()
    .endpoint(endpoint)
    .credential(credential)
    .buildClient();

// Client bất đồng bộ (Async client)
ChatAsyncClient chatAsyncClient = new ChatClientBuilder()
    .endpoint(endpoint)
    .credential(credential)
    .buildAsyncClient();
```

## Các Khái niệm Chính

| Lớp                      | Mục đích                                                          |
| ------------------------ | ----------------------------------------------------------------- |
| `ChatClient`             | Tạo/xóa luồng chat, lấy thread clients                            |
| `ChatThreadClient`       | Các thao tác trong một luồng (tin nhắn, người tham gia, biên lai) |
| `ChatParticipant`        | Người dùng trong một luồng chat với tên hiển thị                  |
| `ChatMessage`            | Nội dung tin nhắn, loại, thông tin người gửi, dấu thời gian       |
| `ChatMessageReadReceipt` | Theo dõi thông báo đã đọc cho từng người tham gia                 |

## Tạo Luồng Chat (Chat Thread)

```java
import com.azure.communication.chat.models.*;
import com.azure.communication.common.CommunicationUserIdentifier;
import java.util.ArrayList;
import java.util.List;

// Định nghĩa người tham gia
List<ChatParticipant> participants = new ArrayList<>();

ChatParticipant participant1 = new ChatParticipant()
    .setCommunicationIdentifier(new CommunicationUserIdentifier("<user-id-1>"))
    .setDisplayName("Alice");

ChatParticipant participant2 = new ChatParticipant()
    .setCommunicationIdentifier(new CommunicationUserIdentifier("<user-id-2>"))
    .setDisplayName("Bob");

participants.add(participant1);
participants.add(participant2);

// Tạo luồng
CreateChatThreadOptions options = new CreateChatThreadOptions("Thảo luận Dự án")
    .setParticipants(participants);

CreateChatThreadResult result = chatClient.createChatThread(options);
String threadId = result.getChatThread().getId();

// Lấy thread client cho các thao tác
ChatThreadClient threadClient = chatClient.getChatThreadClient(threadId);
```

## Gửi Tin nhắn

```java
// Gửi tin nhắn văn bản
SendChatMessageOptions messageOptions = new SendChatMessageOptions()
    .setContent("Xin chào nhóm!")
    .setSenderDisplayName("Alice")
    .setType(ChatMessageType.TEXT);

SendChatMessageResult sendResult = threadClient.sendMessage(messageOptions);
String messageId = sendResult.getId();

// Gửi tin nhắn HTML
SendChatMessageOptions htmlOptions = new SendChatMessageOptions()
    .setContent("<strong>Quan trọng:</strong> Họp lúc 3 giờ chiều")
    .setType(ChatMessageType.HTML);

threadClient.sendMessage(htmlOptions);
```

## Lấy Tin nhắn

```java
import com.azure.core.util.paging.PagedIterable;

// Liệt kê tất cả tin nhắn
PagedIterable<ChatMessage> messages = threadClient.listMessages();

for (ChatMessage message : messages) {
    System.out.println("ID: " + message.getId());
    System.out.println("Loại: " + message.getType());
    System.out.println("Nội dung: " + message.getContent().getMessage());
    System.out.println("Người gửi: " + message.getSenderDisplayName());
    System.out.println("Đã tạo: " + message.getCreatedOn());

    // Kiểm tra nếu đã chỉnh sửa hoặc bị xóa
    if (message.getEditedOn() != null) {
        System.out.println("Đã chỉnh sửa: " + message.getEditedOn());
    }
    if (message.getDeletedOn() != null) {
        System.out.println("Đã xóa: " + message.getDeletedOn());
    }
}

// Lấy tin nhắn cụ thể
ChatMessage message = threadClient.getMessage(messageId);
```

## Cập nhật và Xóa Tin nhắn

```java
// Cập nhật tin nhắn
UpdateChatMessageOptions updateOptions = new UpdateChatMessageOptions()
    .setContent("Nội dung tin nhắn đã cập nhật");

threadClient.updateMessage(messageId, updateOptions);

// Xóa tin nhắn
threadClient.deleteMessage(messageId);
```

## Quản lý Người tham gia

```java
// Liệt kê người tham gia
PagedIterable<ChatParticipant> participants = threadClient.listParticipants();

for (ChatParticipant participant : participants) {
    CommunicationUserIdentifier user =
        (CommunicationUserIdentifier) participant.getCommunicationIdentifier();
    System.out.println("User: " + user.getId());
    System.out.println("Tên hiển thị: " + participant.getDisplayName());
}

// Thêm người tham gia
List<ChatParticipant> newParticipants = new ArrayList<>();
newParticipants.add(new ChatParticipant()
    .setCommunicationIdentifier(new CommunicationUserIdentifier("<new-user-id>"))
    .setDisplayName("Charlie")
    .setShareHistoryTime(OffsetDateTime.now().minusDays(7))); // Chia sẻ lịch sử 7 ngày qua

threadClient.addParticipants(newParticipants);

// Xóa người tham gia
CommunicationUserIdentifier userToRemove = new CommunicationUserIdentifier("<user-id>");
threadClient.removeParticipant(userToRemove);
```

## Thông báo Đã đọc (Read Receipts)

```java
// Gửi thông báo đã đọc
threadClient.sendReadReceipt(messageId);

// Lấy thông báo đã đọc
PagedIterable<ChatMessageReadReceipt> receipts = threadClient.listReadReceipts();

for (ChatMessageReadReceipt receipt : receipts) {
    System.out.println("Message ID: " + receipt.getChatMessageId());
    System.out.println("Đọc bởi: " + receipt.getSenderCommunicationIdentifier());
    System.out.println("Đọc lúc: " + receipt.getReadOn());
}
```

## Thông báo Đang nhập (Typing Notifications)

```java
import com.azure.communication.chat.models.TypingNotificationOptions;

// Gửi thông báo đang nhập
TypingNotificationOptions typingOptions = new TypingNotificationOptions()
    .setSenderDisplayName("Alice");

threadClient.sendTypingNotificationWithResponse(typingOptions, Context.NONE);

// Thông báo đang nhập đơn giản
threadClient.sendTypingNotification();
```

## Các Thao tác Luồng (Thread Operations)

```java
// Lấy thuộc tính luồng
ChatThreadProperties properties = threadClient.getProperties();
System.out.println("Chủ đề: " + properties.getTopic());
System.out.println("Đã tạo: " + properties.getCreatedOn());

// Cập nhật chủ đề
threadClient.updateTopic("Chủ đề Thảo luận Dự án Mới");

// Xóa luồng
chatClient.deleteChatThread(threadId);
```

## Liệt kê Luồng

```java
// Liệt kê tất cả luồng chat cho người dùng
PagedIterable<ChatThreadItem> threads = chatClient.listChatThreads();

for (ChatThreadItem thread : threads) {
    System.out.println("Thread ID: " + thread.getId());
    System.out.println("Chủ đề: " + thread.getTopic());
    System.out.println("Tin nhắn cuối: " + thread.getLastMessageReceivedOn());
}
```

## Phân trang (Pagination)

```java
import com.azure.core.http.rest.PagedResponse;

// Phân trang qua các tin nhắn
int maxPageSize = 10;
ListChatMessagesOptions listOptions = new ListChatMessagesOptions()
    .setMaxPageSize(maxPageSize);

PagedIterable<ChatMessage> pagedMessages = threadClient.listMessages(listOptions);

pagedMessages.iterableByPage().forEach(page -> {
    System.out.println("Mã trạng thái trang: " + page.getStatusCode());
    page.getElements().forEach(msg ->
        System.out.println("Tin nhắn: " + msg.getContent().getMessage()));
});
```

## Xử lý Lỗi

```java
import com.azure.core.exception.HttpResponseException;

try {
    threadClient.sendMessage(messageOptions);
} catch (HttpResponseException e) {
    switch (e.getResponse().getStatusCode()) {
        case 401:
            System.out.println("Không được phép - kiểm tra token");
            break;
        case 403:
            System.out.println("Bị cấm - người dùng không trong luồng");
            break;
        case 404:
            System.out.println("Không tìm thấy luồng");
            break;
        default:
            System.out.println("Lỗi: " + e.getMessage());
    }
}
```

## Loại Tin nhắn

| Loại                  | Mô tả                                          |
| --------------------- | ---------------------------------------------- |
| `TEXT`                | Tin nhắn chat thông thường                     |
| `HTML`                | Tin nhắn định dạng HTML                        |
| `TOPIC_UPDATED`       | Tin nhắn hệ thống - chủ đề đã thay đổi         |
| `PARTICIPANT_ADDED`   | Tin nhắn hệ thống - người tham gia đã tham gia |
| `PARTICIPANT_REMOVED` | Tin nhắn hệ thống - người tham gia đã rời đi   |

## Biến Môi trường

```bash
AZURE_COMMUNICATION_ENDPOINT=https://<resource>.communication.azure.com
AZURE_COMMUNICATION_USER_TOKEN=<user-access-token>
```

## Thực hành Tốt nhất

1. **Quản lý Token** - Token người dùng hết hạn; triển khai logic làm mới với `CommunicationTokenRefreshOptions`
2. **Phân trang** - Sử dụng `listMessages(options)` với `maxPageSize` cho các luồng lớn
3. **Chia sẻ Lịch sử** - Đặt `shareHistoryTime` khi thêm người tham gia để kiểm soát khả năng hiển thị tin nhắn
4. **Loại Tin nhắn** - Lọc tin nhắn hệ thống (`PARTICIPANT_ADDED`, v.v.) khỏi tin nhắn người dùng
5. **Thông báo Đã đọc** - Chỉ gửi thông báo khi tin nhắn thực sự được người dùng xem

## Các Cụm từ Kích hoạt

- "chat application Java", "real-time messaging Java"
- "chat thread", "chat participants", "chat messages"
- "read receipts", "typing notifications"
- "Azure Communication Services chat"
