---
name: azure-messaging-webpubsub-java
description: Xây dựng các ứng dụng web thời gian thực với Azure Web PubSub SDK cho Java. Sử dụng khi triển khai nhắn tin dựa trên WebSocket, cập nhật trực tiếp, ứng dụng chat, hoặc thông báo đẩy từ server đến client.
package: com.azure:azure-messaging-webpubsub
---

# Azure Web PubSub SDK cho Java

Xây dựng các ứng dụng web thời gian thực sử dụng Azure Web PubSub SDK cho Java.

## Cài đặt

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-messaging-webpubsub</artifactId>
    <version>1.5.0</version>
</dependency>
```

## Tạo Client

### Với Chuỗi Kết nối (Connection String)

```java
import com.azure.messaging.webpubsub.WebPubSubServiceClient;
import com.azure.messaging.webpubsub.WebPubSubServiceClientBuilder;

WebPubSubServiceClient client = new WebPubSubServiceClientBuilder()
    .connectionString("<connection-string>")
    .hub("chat")
    .buildClient();
```

### Với Access Key

```java
import com.azure.core.credential.AzureKeyCredential;

WebPubSubServiceClient client = new WebPubSubServiceClientBuilder()
    .credential(new AzureKeyCredential("<access-key>"))
    .endpoint("<endpoint>")
    .hub("chat")
    .buildClient();
```

### Với DefaultAzureCredential

```java
import com.azure.identity.DefaultAzureCredentialBuilder;

WebPubSubServiceClient client = new WebPubSubServiceClientBuilder()
    .credential(new DefaultAzureCredentialBuilder().build())
    .endpoint("<endpoint>")
    .hub("chat")
    .buildClient();
```

### Async Client

```java
import com.azure.messaging.webpubsub.WebPubSubServiceAsyncClient;

WebPubSubServiceAsyncClient asyncClient = new WebPubSubServiceClientBuilder()
    .connectionString("<connection-string>")
    .hub("chat")
    .buildAsyncClient();
```

## Các Khái niệm Chính

- **Hub**: Đơn vị cách ly logic cho các kết nối
- **Group**: Tập hợp con các kết nối trong một hub
- **Connection**: Kết nối WebSocket của client riêng lẻ
- **User**: Thực thể có thể có nhiều kết nối

## Các Mẫu Cốt lõi

### Gửi đến Tất cả Kết nối

```java
import com.azure.messaging.webpubsub.models.WebPubSubContentType;

// Gửi tin nhắn văn bản
client.sendToAll("Hello everyone!", WebPubSubContentType.TEXT_PLAIN);

// Gửi JSON
String jsonMessage = "{\"type\": \"notification\", \"message\": \"New update!\"}";
client.sendToAll(jsonMessage, WebPubSubContentType.APPLICATION_JSON);
```

### Gửi đến Tất cả với Bộ lọc (Filter)

```java
import com.azure.core.http.rest.RequestOptions;
import com.azure.core.util.BinaryData;

BinaryData message = BinaryData.fromString("Hello filtered users!");

// Lọc theo userId
client.sendToAllWithResponse(
    message,
    WebPubSubContentType.TEXT_PLAIN,
    message.getLength(),
    new RequestOptions().addQueryParam("filter", "userId ne 'user1'"));

// Lọc theo nhóm (groups)
client.sendToAllWithResponse(
    message,
    WebPubSubContentType.TEXT_PLAIN,
    message.getLength(),
    new RequestOptions().addQueryParam("filter", "'GroupA' in groups and not('GroupB' in groups)"));
```

### Gửi đến Nhóm (Group)

```java
// Gửi đến tất cả kết nối trong một nhóm
client.sendToGroup("java-developers", "Hello Java devs!", WebPubSubContentType.TEXT_PLAIN);

// Gửi JSON đến nhóm
String json = "{\"event\": \"update\", \"data\": {\"version\": \"2.0\"}}";
client.sendToGroup("subscribers", json, WebPubSubContentType.APPLICATION_JSON);
```

### Gửi đến Kết nối Cụ thể

```java
// Gửi đến một kết nối cụ thể theo ID
client.sendToConnection("connectionId123", "Private message", WebPubSubContentType.TEXT_PLAIN);
```

### Gửi đến User

```java
// Gửi đến tất cả kết nối của một user cụ thể
client.sendToUser("andy", "Hello Andy!", WebPubSubContentType.TEXT_PLAIN);
```

### Quản lý Nhóm

```java
// Thêm kết nối vào nhóm
client.addConnectionToGroup("premium-users", "connectionId123");

// Xóa kết nối khỏi nhóm
client.removeConnectionFromGroup("premium-users", "connectionId123");

// Thêm user vào nhóm (tất cả kết nối của họ)
client.addUserToGroup("admin-group", "userId456");

// Xóa user khỏi nhóm
client.removeUserFromGroup("admin-group", "userId456");

// Kiểm tra xem user có trong nhóm không
boolean exists = client.userExistsInGroup("admin-group", "userId456");
```

### Quản lý Kết nối

```java
// Kiểm tra xem kết nối có tồn tại không
boolean connected = client.connectionExists("connectionId123");

// Đóng một kết nối
client.closeConnection("connectionId123");

// Đóng với lý do
client.closeConnection("connectionId123", "Session expired");

// Kiểm tra xem user có tồn tại không (có bất kỳ kết nối nào)
boolean userOnline = client.userExists("userId456");

// Đóng tất cả kết nối của một user
client.closeUserConnections("userId456");

// Đóng tất cả kết nối trong một nhóm
client.closeGroupConnections("inactive-group");
```

### Tạo Client Access Token

```java
import com.azure.messaging.webpubsub.models.GetClientAccessTokenOptions;
import com.azure.messaging.webpubsub.models.WebPubSubClientAccessToken;

// Token cơ bản
WebPubSubClientAccessToken token = client.getClientAccessToken(
    new GetClientAccessTokenOptions());
System.out.println("URL: " + token.getUrl());

// Với user ID
WebPubSubClientAccessToken userToken = client.getClientAccessToken(
    new GetClientAccessTokenOptions().setUserId("user123"));

// Với roles (quyền)
WebPubSubClientAccessToken roleToken = client.getClientAccessToken(
    new GetClientAccessTokenOptions()
        .setUserId("user123")
        .addRole("webpubsub.joinLeaveGroup")
        .addRole("webpubsub.sendToGroup"));

// Với các nhóm để tham gia khi kết nối
WebPubSubClientAccessToken groupToken = client.getClientAccessToken(
    new GetClientAccessTokenOptions()
        .setUserId("user123")
        .addGroup("announcements")
        .addGroup("updates"));

// Với thời gian hết hạn tùy chỉnh
WebPubSubClientAccessToken expToken = client.getClientAccessToken(
    new GetClientAccessTokenOptions()
        .setUserId("user123")
        .setExpiresAfter(Duration.ofHours(2)));
```

### Cấp/Thu hồi Quyền

```java
import com.azure.messaging.webpubsub.models.WebPubSubPermission;

// Cấp quyền gửi đến một nhóm
client.grantPermission(
    WebPubSubPermission.SEND_TO_GROUP,
    "connectionId123",
    new RequestOptions().addQueryParam("targetName", "chat-room"));

// Thu hồi quyền
client.revokePermission(
    WebPubSubPermission.SEND_TO_GROUP,
    "connectionId123",
    new RequestOptions().addQueryParam("targetName", "chat-room"));

// Kiểm tra quyền
boolean hasPermission = client.checkPermission(
    WebPubSubPermission.SEND_TO_GROUP,
    "connectionId123",
    new RequestOptions().addQueryParam("targetName", "chat-room"));
```

### Các Thao tác Async

```java
asyncClient.sendToAll("Async message!", WebPubSubContentType.TEXT_PLAIN)
    .subscribe(
        unused -> System.out.println("Message sent"),
        error -> System.err.println("Error: " + error.getMessage())
    );

asyncClient.sendToGroup("developers", "Group message", WebPubSubContentType.TEXT_PLAIN)
    .doOnSuccess(v -> System.out.println("Sent to group"))
    .doOnError(e -> System.err.println("Failed: " + e))
    .subscribe();
```

## Xử lý Lỗi

```java
import com.azure.core.exception.HttpResponseException;

try {
    client.sendToConnection("invalid-id", "test", WebPubSubContentType.TEXT_PLAIN);
} catch (HttpResponseException e) {
    System.out.println("Status: " + e.getResponse().getStatusCode());
    System.out.println("Error: " + e.getMessage());
}
```

## Biến Môi trường

```bash
WEB_PUBSUB_CONNECTION_STRING=Endpoint=https://<resource>.webpubsub.azure.com;AccessKey=...
WEB_PUBSUB_ENDPOINT=https://<resource>.webpubsub.azure.com
WEB_PUBSUB_ACCESS_KEY=<your-access-key>
```

## Vai trò Client (Client Roles)

| Role                               | Quyền                             |
| ---------------------------------- | --------------------------------- |
| `webpubsub.joinLeaveGroup`         | Tham gia/rời khỏi bất kỳ nhóm nào |
| `webpubsub.sendToGroup`            | Gửi đến bất kỳ nhóm nào           |
| `webpubsub.joinLeaveGroup.<group>` | Tham gia/rời khỏi nhóm cụ thể     |
| `webpubsub.sendToGroup.<group>`    | Gửi đến nhóm cụ thể               |

## Thực hành Tốt nhất

1. **Sử dụng Nhóm**: Tổ chức các kết nối thành các nhóm để nhắn tin mục tiêu
2. **User IDs**: Liên kết kết nối với user IDs để nhắn tin cấp người dùng
3. **Hết hạn Token**: Thiết lập thời gian hết hạn token phù hợp để bảo mật
4. **Roles**: Cấp quyền tối thiểu cần thiết thông qua roles
5. **Cách ly Hub**: Sử dụng các hub riêng biệt cho các tính năng ứng dụng khác nhau
6. **Quản lý Kết nối**: Dọn dẹp các kết nối không hoạt động

## Các Cụm từ Kích hoạt

- "Web PubSub Java"
- "WebSocket messaging Azure"
- "real-time push notifications"
- "server-sent events"
- "chat application backend"
- "live updates broadcasting"
