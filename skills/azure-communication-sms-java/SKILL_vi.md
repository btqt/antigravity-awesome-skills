---
name: azure-communication-sms-java
description: Gửi tin nhắn SMS với Azure Communication Services SMS Java SDK. Sử dụng khi triển khai thông báo SMS, cảnh báo, gửi OTP, nhắn tin hàng loạt, hoặc báo cáo gửi tin.
package: com.azure:azure-communication-sms
---

# Azure Communication SMS (Java)

Gửi tin nhắn SMS đến một hoặc nhiều người nhận với báo cáo gửi tin.

## Cài đặt

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-communication-sms</artifactId>
    <version>1.2.0</version>
</dependency>
```

## Tạo Client

```java
import com.azure.communication.sms.SmsClient;
import com.azure.communication.sms.SmsClientBuilder;
import com.azure.identity.DefaultAzureCredentialBuilder;

// Với DefaultAzureCredential (khuyên dùng)
SmsClient smsClient = new SmsClientBuilder()
    .endpoint("https://<resource>.communication.azure.com")
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildClient();

// Với connection string
SmsClient smsClient = new SmsClientBuilder()
    .connectionString("<connection-string>")
    .buildClient();

// Với AzureKeyCredential
import com.azure.core.credential.AzureKeyCredential;

SmsClient smsClient = new SmsClientBuilder()
    .endpoint("https://<resource>.communication.azure.com")
    .credential(new AzureKeyCredential("<access-key>"))
    .buildClient();

// Client bất đồng bộ (Async client)
SmsAsyncClient smsAsyncClient = new SmsClientBuilder()
    .connectionString("<connection-string>")
    .buildAsyncClient();
```

## Gửi SMS đến Một Người nhận

```java
import com.azure.communication.sms.models.SmsSendResult;

// Gửi đơn giản
SmsSendResult result = smsClient.send(
    "+14255550100",      // Từ (số điện thoại ACS của bạn)
    "+14255551234",      // Đến
    "Mã xác minh của bạn là 123456");

System.out.println("Message ID: " + result.getMessageId());
System.out.println("Đến: " + result.getTo());
System.out.println("Thành công: " + result.isSuccessful());

if (!result.isSuccessful()) {
    System.out.println("Lỗi: " + result.getErrorMessage());
    System.out.println("Trạng thái: " + result.getHttpStatusCode());
}
```

## Gửi SMS đến Nhiều Người nhận

```java
import com.azure.communication.sms.models.SmsSendOptions;
import java.util.Arrays;
import java.util.List;

List<String> recipients = Arrays.asList(
    "+14255551111",
    "+14255552222",
    "+14255553333"
);

// Với tùy chọn
SmsSendOptions options = new SmsSendOptions()
    .setDeliveryReportEnabled(true)
    .setTag("marketing-campaign-001");

Iterable<SmsSendResult> results = smsClient.sendWithResponse(
    "+14255550100",      // Từ
    recipients,          // Danh sách đến
    "Flash sale! Giảm giá 50% chỉ hôm nay.",
    options,
    Context.NONE
).getValue();

for (SmsSendResult result : results) {
    if (result.isSuccessful()) {
        System.out.println("Đã gửi đến " + result.getTo() + ": " + result.getMessageId());
    } else {
        System.out.println("Thất bại gửi đến " + result.getTo() + ": " + result.getErrorMessage());
    }
}
```

## Tùy chọn Gửi

```java
SmsSendOptions options = new SmsSendOptions();

// Bật báo cáo gửi tin (gửi qua Event Grid)
options.setDeliveryReportEnabled(true);

// Thêm tag tùy chỉnh để theo dõi
options.setTag("order-confirmation-12345");
```

## Xử lý Phản hồi

```java
import com.azure.core.http.rest.Response;

Response<Iterable<SmsSendResult>> response = smsClient.sendWithResponse(
    "+14255550100",
    Arrays.asList("+14255551234"),
    "Xin chào!",
    new SmsSendOptions().setDeliveryReportEnabled(true),
    Context.NONE
);

// Kiểm tra phản hồi HTTP
System.out.println("Mã trạng thái: " + response.getStatusCode());
System.out.println("Headers: " + response.getHeaders());

// Xử lý kết quả
for (SmsSendResult result : response.getValue()) {
    System.out.println("Message ID: " + result.getMessageId());
    System.out.println("Thành công: " + result.isSuccessful());

    if (!result.isSuccessful()) {
        System.out.println("Trạng thái HTTP: " + result.getHttpStatusCode());
        System.out.println("Lỗi: " + result.getErrorMessage());
    }
}
```

## Thao tác Bất đồng bộ (Async Operations)

```java
import reactor.core.publisher.Mono;

SmsAsyncClient asyncClient = new SmsClientBuilder()
    .connectionString("<connection-string>")
    .buildAsyncClient();

// Gửi tin nhắn đơn
asyncClient.send("+14255550100", "+14255551234", "Tin nhắn Async!")
    .subscribe(
        result -> System.out.println("Đã gửi: " + result.getMessageId()),
        error -> System.out.println("Lỗi: " + error.getMessage())
    );

// Gửi đến nhiều người với tùy chọn
SmsSendOptions options = new SmsSendOptions()
    .setDeliveryReportEnabled(true);

asyncClient.sendWithResponse(
    "+14255550100",
    Arrays.asList("+14255551111", "+14255552222"),
    "Tin nhắn hàng loạt async",
    options)
    .subscribe(response -> {
        for (SmsSendResult result : response.getValue()) {
            System.out.println("Kết quả: " + result.getTo() + " - " + result.isSuccessful());
        }
    });
```

## Xử lý Lỗi

```java
import com.azure.core.exception.HttpResponseException;

try {
    SmsSendResult result = smsClient.send(
        "+14255550100",
        "+14255551234",
        "Tin nhắn thử nghiệm"
    );

    // Lỗi tin nhắn cá nhân không ném ngoại lệ
    if (!result.isSuccessful()) {
        handleMessageError(result);
    }

} catch (HttpResponseException e) {
    // Lỗi cấp yêu cầu (xác thực, mạng, v.v.)
    System.out.println("Yêu cầu thất bại: " + e.getMessage());
    System.out.println("Trạng thái: " + e.getResponse().getStatusCode());
} catch (RuntimeException e) {
    System.out.println("Lỗi không mong đợi: " + e.getMessage());
}

private void handleMessageError(SmsSendResult result) {
    int status = result.getHttpStatusCode();
    String error = result.getErrorMessage();

    if (status == 400) {
        System.out.println("Số điện thoại không hợp lệ: " + result.getTo());
    } else if (status == 429) {
        System.out.println("Giới hạn tốc độ - thử lại sau");
    } else {
        System.out.println("Lỗi " + status + ": " + error);
    }
}
```

## Báo cáo Gửi tin (Delivery Reports)

Báo cáo gửi tin được gửi qua Azure Event Grid. Cấu hình đăng ký Event Grid cho tài nguyên ACS của bạn.

```java
// Trình xử lý webhook Event Grid (trong endpoint của bạn)
public void handleDeliveryReport(String eventJson) {
    // Phân tích sự kiện Event Grid
    // Loại sự kiện: Microsoft.Communication.SMSDeliveryReportReceived

    // Dữ liệu sự kiện chứa:
    // - messageId: tương ứng với SmsSendResult.getMessageId()
    // - from: số người gửi
    // - to: số người nhận
    // - deliveryStatus: "Delivered", "Failed", v.v.
    // - deliveryStatusDetails: chi tiết trạng thái
    // - receivedTimestamp: khi nhận được trạng thái
    // - tag: tag tùy chỉnh của bạn từ SmsSendOptions
}
```

## Thuộc tính SmsSendResult

| Thuộc tính                 | Loại                | Mô tả                              |
| -------------------------- | ------------------- | ---------------------------------- |
| `getMessageId()`           | String              | Định danh tin nhắn duy nhất        |
| `getTo()`                  | String              | Số điện thoại người nhận           |
| `isSuccessful()`           | boolean             | Việc gửi có thành công hay không   |
| `getHttpStatusCode()`      | int                 | Trạng thái HTTP cho người nhận này |
| `getErrorMessage()`        | String              | Chi tiết lỗi nếu thất bại          |
| `getRepeatabilityResult()` | RepeatabilityResult | Kết quả tính idempotent            |

## Biến Môi trường

```bash
AZURE_COMMUNICATION_ENDPOINT=https://<resource>.communication.azure.com
AZURE_COMMUNICATION_CONNECTION_STRING=endpoint=https://...;accesskey=...
SMS_FROM_NUMBER=+14255550100
```

## Thực hành Tốt nhất

1. **Định dạng Số Điện thoại** - Sử dụng định dạng E.164: `+[mã quốc gia][số]`
2. **Báo cáo Gửi tin** - Bật cho các tin nhắn quan trọng (OTP, cảnh báo)
3. **Gắn thẻ (Tagging)** - Sử dụng tags để tương quan tin nhắn với ngữ cảnh kinh doanh
4. **Xử lý Lỗi** - Kiểm tra `isSuccessful()` cho từng người nhận riêng lẻ
5. **Giới hạn tốc độ** - Triển khai thử lại với backoff cho phản hồi 429
6. **Gửi hàng loạt** - Sử dụng gửi theo lô cho nhiều người nhận (hiệu quả hơn)

## Các Cụm từ Kích hoạt

- "send SMS Java", "text message Java"
- "SMS notification", "OTP SMS", "bulk SMS"
- "delivery report SMS", "Azure Communication Services SMS"
