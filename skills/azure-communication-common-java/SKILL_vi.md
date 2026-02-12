---
name: azure-communication-common-java
description: Các tiện ích chung của Azure Communication Services cho Java. Sử dụng khi làm việc với CommunicationTokenCredential, định danh người dùng (user identifiers), làm mới token, hoặc xác thực chia sẻ trên các dịch vụ ACS.
package: com.azure:azure-communication-common
---

# Azure Communication Common (Java)

Các tiện ích xác thực chia sẻ và cấu trúc dữ liệu cho Azure Communication Services.

## Cài đặt

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-communication-common</artifactId>
    <version>1.4.0</version>
</dependency>
```

## Các Khái niệm Chính

| Lớp                                | Mục đích                                    |
| ---------------------------------- | ------------------------------------------- |
| `CommunicationTokenCredential`     | Xác thực người dùng với các dịch vụ ACS     |
| `CommunicationTokenRefreshOptions` | Cấu hình tự động làm mới token              |
| `CommunicationUserIdentifier`      | Xác định người dùng ACS                     |
| `PhoneNumberIdentifier`            | Xác định số điện thoại PSTN                 |
| `MicrosoftTeamsUserIdentifier`     | Xác định người dùng Teams                   |
| `UnknownIdentifier`                | Định danh chung cho các loại không xác định |

## CommunicationTokenCredential

### Token Tĩnh (Client Ngắn hạn)

```java
import com.azure.communication.common.CommunicationTokenCredential;

// Token tĩnh đơn giản - không làm mới
String userToken = "<user-access-token>";
CommunicationTokenCredential credential = new CommunicationTokenCredential(userToken);

// Sử dụng với Chat, Calling, v.v.
ChatClient chatClient = new ChatClientBuilder()
    .endpoint("https://<resource>.communication.azure.com")
    .credential(credential)
    .buildClient();
```

### Chủ động Làm mới Token (Client Dài hạn)

```java
import com.azure.communication.common.CommunicationTokenRefreshOptions;
import java.util.concurrent.Callable;

// Callback làm mới token - được gọi khi token sắp hết hạn
Callable<String> tokenRefresher = () -> {
    // Gọi server của bạn để lấy token mới
    return fetchNewTokenFromServer();
};

// Với làm mới chủ động
CommunicationTokenRefreshOptions refreshOptions = new CommunicationTokenRefreshOptions(tokenRefresher)
    .setRefreshProactively(true)      // Làm mới trước khi hết hạn
    .setInitialToken(currentToken);    // Token ban đầu tùy chọn

CommunicationTokenCredential credential = new CommunicationTokenCredential(refreshOptions);
```

### Làm mới Token Bất đồng bộ (Async)

```java
import java.util.concurrent.CompletableFuture;

// Trình lấy token bất đồng bộ
Callable<String> asyncRefresher = () -> {
    CompletableFuture<String> future = fetchTokenAsync();
    return future.get();  // Chặn cho đến khi có token
};

CommunicationTokenRefreshOptions options = new CommunicationTokenRefreshOptions(asyncRefresher)
    .setRefreshProactively(true);

CommunicationTokenCredential credential = new CommunicationTokenCredential(options);
```

## Xác thực Entra ID (Azure AD)

```java
import com.azure.identity.InteractiveBrowserCredentialBuilder;
import com.azure.communication.common.EntraCommunicationTokenCredentialOptions;
import java.util.Arrays;
import java.util.List;

// Cho Teams Phone Extensibility
InteractiveBrowserCredential entraCredential = new InteractiveBrowserCredentialBuilder()
    .clientId("<your-client-id>")
    .tenantId("<your-tenant-id>")
    .redirectUrl("<your-redirect-uri>")
    .build();

String resourceEndpoint = "https://<resource>.communication.azure.com";
List<String> scopes = Arrays.asList(
    "https://auth.msft.communication.azure.com/TeamsExtension.ManageCalls"
);

EntraCommunicationTokenCredentialOptions entraOptions =
    new EntraCommunicationTokenCredentialOptions(entraCredential, resourceEndpoint)
        .setScopes(scopes);

CommunicationTokenCredential credential = new CommunicationTokenCredential(entraOptions);
```

## Định danh Giao tiếp (Communication Identifiers)

### CommunicationUserIdentifier

```java
import com.azure.communication.common.CommunicationUserIdentifier;

// Tạo định danh cho người dùng ACS
CommunicationUserIdentifier user = new CommunicationUserIdentifier("8:acs:resource-id_user-id");

// Lấy ID thô
String rawId = user.getId();
```

### PhoneNumberIdentifier

```java
import com.azure.communication.common.PhoneNumberIdentifier;

// Số điện thoại định dạng E.164
PhoneNumberIdentifier phone = new PhoneNumberIdentifier("+14255551234");

String phoneNumber = phone.getPhoneNumber();  // "+14255551234"
String rawId = phone.getRawId();              // "4:+14255551234"
```

### MicrosoftTeamsUserIdentifier

```java
import com.azure.communication.common.MicrosoftTeamsUserIdentifier;

// Định danh người dùng Teams
MicrosoftTeamsUserIdentifier teamsUser = new MicrosoftTeamsUserIdentifier("<teams-user-id>")
    .setCloudEnvironment(CommunicationCloudEnvironment.PUBLIC);

// Cho người dùng Teams ẩn danh
MicrosoftTeamsUserIdentifier anonymousTeamsUser = new MicrosoftTeamsUserIdentifier("<teams-user-id>")
    .setAnonymous(true);
```

### UnknownIdentifier

```java
import com.azure.communication.common.UnknownIdentifier;

// Cho các định danh loại không xác định
UnknownIdentifier unknown = new UnknownIdentifier("some-raw-id");
```

## Phân tích Định danh

```java
import com.azure.communication.common.CommunicationIdentifier;
import com.azure.communication.common.CommunicationIdentifierModel;

// Phân tích ID thô thành loại thích hợp
public CommunicationIdentifier parseIdentifier(String rawId) {
    if (rawId.startsWith("8:acs:")) {
        return new CommunicationUserIdentifier(rawId);
    } else if (rawId.startsWith("4:")) {
        String phone = rawId.substring(2);
        return new PhoneNumberIdentifier(phone);
    } else if (rawId.startsWith("8:orgid:")) {
        String teamsId = rawId.substring(8);
        return new MicrosoftTeamsUserIdentifier(teamsId);
    } else {
        return new UnknownIdentifier(rawId);
    }
}
```

## Kiểm tra Loại Định danh

```java
import com.azure.communication.common.CommunicationIdentifier;

public void processIdentifier(CommunicationIdentifier identifier) {
    if (identifier instanceof CommunicationUserIdentifier) {
        CommunicationUserIdentifier user = (CommunicationUserIdentifier) identifier;
        System.out.println("ACS User: " + user.getId());

    } else if (identifier instanceof PhoneNumberIdentifier) {
        PhoneNumberIdentifier phone = (PhoneNumberIdentifier) identifier;
        System.out.println("Phone: " + phone.getPhoneNumber());

    } else if (identifier instanceof MicrosoftTeamsUserIdentifier) {
        MicrosoftTeamsUserIdentifier teams = (MicrosoftTeamsUserIdentifier) identifier;
        System.out.println("Teams User: " + teams.getUserId());
        System.out.println("Ẩn danh: " + teams.isAnonymous());

    } else if (identifier instanceof UnknownIdentifier) {
        UnknownIdentifier unknown = (UnknownIdentifier) identifier;
        System.out.println("Unknown: " + unknown.getId());
    }
}
```

## Truy cập Token

```java
import com.azure.core.credential.AccessToken;

// Lấy token hiện tại (để debug/logging - không được để lộ!)
CommunicationTokenCredential credential = new CommunicationTokenCredential(token);

// Truy cập đồng bộ
AccessToken accessToken = credential.getToken();
System.out.println("Token hết hạn: " + accessToken.getExpiresAt());

// Truy cập bất đồng bộ
credential.getTokenAsync()
    .subscribe(token -> {
        System.out.println("Token: " + token.getToken().substring(0, 20) + "...");
        System.out.println("Hết hạn: " + token.getExpiresAt());
    });
```

## Hủy Credential

```java
// Dọn dẹp khi xong
credential.close();

// Hoặc sử dụng try-with-resources
try (CommunicationTokenCredential cred = new CommunicationTokenCredential(options)) {
    // Sử dụng credential
    chatClient.doSomething();
}
```

## Môi trường Đám mây

```java
import com.azure.communication.common.CommunicationCloudEnvironment;

// Các môi trường có sẵn
CommunicationCloudEnvironment publicCloud = CommunicationCloudEnvironment.PUBLIC;
CommunicationCloudEnvironment govCloud = CommunicationCloudEnvironment.GCCH;
CommunicationCloudEnvironment dodCloud = CommunicationCloudEnvironment.DOD;

// Đặt trên định danh Teams
MicrosoftTeamsUserIdentifier teamsUser = new MicrosoftTeamsUserIdentifier("<user-id>")
    .setCloudEnvironment(CommunicationCloudEnvironment.GCCH);
```

## Biến Môi trường

```bash
AZURE_COMMUNICATION_ENDPOINT=https://<resource>.communication.azure.com
AZURE_COMMUNICATION_USER_TOKEN=<user-access-token>
```

## Thực hành Tốt nhất

1. **Chủ động Làm mới** - Luôn sử dụng `setRefreshProactively(true)` cho client dài hạn
2. **Bảo mật Token** - Không bao giờ log hoặc để lộ token đầy đủ
3. **Đóng Credentials** - Hủy các credential khi không còn cần thiết
4. **Xử lý Lỗi** - Xử lý lỗi làm mới token một cách duyên dáng
5. **Loại Định danh** - Sử dụng các loại định danh cụ thể, không sử dụng chuỗi thô

## Mẫu Sử dụng Chung

```java
// Mẫu: Tạo credential cho Chat/Calling client
public ChatClient createChatClient(String token, String endpoint) {
    CommunicationTokenRefreshOptions refreshOptions =
        new CommunicationTokenRefreshOptions(this::refreshToken)
            .setRefreshProactively(true)
            .setInitialToken(token);

    CommunicationTokenCredential credential =
        new CommunicationTokenCredential(refreshOptions);

    return new ChatClientBuilder()
        .endpoint(endpoint)
        .credential(credential)
        .buildClient();
}

private String refreshToken() {
    // Gọi endpoint token của bạn
    return tokenService.getNewToken();
}
```

## Các Cụm từ Kích hoạt

- "ACS authentication", "communication token credential"
- "user access token", "token refresh"
- "CommunicationUserIdentifier", "PhoneNumberIdentifier"
- "Azure Communication Services authentication"
