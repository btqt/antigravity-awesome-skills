---
name: azure-mgmt-botservice-dotnet
description: |
  Azure Resource Manager SDK cho Bot Service trong .NET. Các thao tác management plane để tạo và quản lý tài nguyên Azure Bot, các kênh (Teams, DirectLine, Slack), và cài đặt kết nối. Triggers: "Bot Service", "BotResource", "Azure Bot", "DirectLine channel", "Teams channel", "bot management .NET", "create bot".
package: Azure.ResourceManager.BotService
---

# Azure.ResourceManager.BotService (.NET)

SDK management plane để cung cấp và quản lý tài nguyên Azure Bot Service thông qua Azure Resource Manager.

## Cài đặt

```bash
dotnet add package Azure.ResourceManager.BotService
dotnet add package Azure.Identity
```

**Phiên bản Hiện tại**: Stable v1.1.1, Preview v1.1.0-beta.1

## Biến Môi trường

```bash
AZURE_SUBSCRIPTION_ID=<your-subscription-id>
# Cho xác thực service principal (tùy chọn)
AZURE_TENANT_ID=<tenant-id>
AZURE_CLIENT_ID=<client-id>
AZURE_CLIENT_SECRET=<client-secret>
```

## Xác thực

```csharp
using Azure.Identity;
using Azure.ResourceManager;
using Azure.ResourceManager.BotService;

// Xác thực sử dụng DefaultAzureCredential
var credential = new DefaultAzureCredential();
ArmClient armClient = new ArmClient(credential);

// Lấy subscription và resource group
SubscriptionResource subscription = await armClient.GetDefaultSubscriptionAsync();
ResourceGroupResource resourceGroup = await subscription.GetResourceGroups().GetAsync("myResourceGroup");

// Truy cập bot collection
BotCollection botCollection = resourceGroup.GetBots();
```

## Phân cấp Tài nguyên

```
ArmClient
└── SubscriptionResource
    └── ResourceGroupResource
        └── BotResource
            ├── BotChannelResource (DirectLine, Teams, Slack, etc.)
            ├── BotConnectionSettingResource (OAuth connections)
            └── BotServicePrivateEndpointConnectionResource
```

## Quy trình Làm việc Cốt lõi

### 1. Tạo Tài nguyên Bot

```csharp
using Azure.ResourceManager.BotService;
using Azure.ResourceManager.BotService.Models;

// Tạo bot data
var botData = new BotData(AzureLocation.WestUS2)
{
    Kind = BotServiceKind.Azurebot,
    Sku = new BotServiceSku(BotServiceSkuName.F0),
    Properties = new BotProperties(
        displayName: "MyBot",
        endpoint: new Uri("https://mybot.azurewebsites.net/api/messages"),
        msaAppId: "<your-msa-app-id>")
    {
        Description = "My Azure Bot",
        MsaAppType = BotMsaAppType.MultiTenant
    }
};

// Tạo hoặc cập nhật bot
ArmOperation<BotResource> operation = await botCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "myBotName",
    botData);

BotResource bot = operation.Value;
Console.WriteLine($"Bot created: {bot.Data.Name}");
```

### 2. Cấu hình Kênh DirectLine

```csharp
// Lấy bot
BotResource bot = await resourceGroup.GetBots().GetAsync("myBotName");

// Lấy channel collection
BotChannelCollection channels = bot.GetBotChannels();

// Tạo cấu hình kênh DirectLine
var channelData = new BotChannelData(AzureLocation.WestUS2)
{
    Properties = new DirectLineChannel()
    {
        Properties = new DirectLineChannelProperties()
        {
            Sites =
            {
                new DirectLineSite("Default Site")
                {
                    IsEnabled = true,
                    IsV1Enabled = false,
                    IsV3Enabled = true,
                    IsSecureSiteEnabled = true
                }
            }
        }
    }
};

// Tạo hoặc cập nhật kênh
ArmOperation<BotChannelResource> channelOp = await channels.CreateOrUpdateAsync(
    WaitUntil.Completed,
    BotChannelName.DirectLineChannel,
    channelData);

Console.WriteLine("DirectLine channel configured");
```

### 3. Cấu hình Kênh Microsoft Teams

```csharp
var teamsChannelData = new BotChannelData(AzureLocation.WestUS2)
{
    Properties = new MsTeamsChannel()
    {
        Properties = new MsTeamsChannelProperties()
        {
            IsEnabled = true,
            EnableCalling = false
        }
    }
};

await channels.CreateOrUpdateAsync(
    WaitUntil.Completed,
    BotChannelName.MsTeamsChannel,
    teamsChannelData);
```

### 4. Cấu hình Kênh Web Chat

```csharp
var webChatChannelData = new BotChannelData(AzureLocation.WestUS2)
{
    Properties = new WebChatChannel()
    {
        Properties = new WebChatChannelProperties()
        {
            Sites =
            {
                new WebChatSite("Default Site")
                {
                    IsEnabled = true
                }
            }
        }
    }
};

await channels.CreateOrUpdateAsync(
    WaitUntil.Completed,
    BotChannelName.WebChatChannel,
    webChatChannelData);
```

### 5. Lấy Bot và Liệt kê Kênh

```csharp
// Lấy bot
BotResource bot = await botCollection.GetAsync("myBotName");
Console.WriteLine($"Bot: {bot.Data.Properties.DisplayName}");
Console.WriteLine($"Endpoint: {bot.Data.Properties.Endpoint}");

// Liệt kê kênh
await foreach (BotChannelResource channel in bot.GetBotChannels().GetAllAsync())
{
    Console.WriteLine($"Channel: {channel.Data.Name}");
}
```

### 6. Tạo lại Keys DirectLine

```csharp
var regenerateRequest = new BotChannelRegenerateKeysContent(BotChannelName.DirectLineChannel)
{
    SiteName = "Default Site"
};

BotChannelResource channelWithKeys = await bot.GetBotChannelWithRegenerateKeysAsync(regenerateRequest);
```

### 7. Cập nhật Bot

```csharp
BotResource bot = await botCollection.GetAsync("myBotName");

// Cập nhật sử dụng patch
var updateData = new BotData(bot.Data.Location)
{
    Properties = new BotProperties(
        displayName: "Updated Bot Name",
        endpoint: bot.Data.Properties.Endpoint,
        msaAppId: bot.Data.Properties.MsaAppId)
    {
        Description = "Updated description"
    }
};

await bot.UpdateAsync(updateData);
```

### 8. Xóa Bot

```csharp
BotResource bot = await botCollection.GetAsync("myBotName");
await bot.DeleteAsync(WaitUntil.Completed);
```

## Các Loại Kênh Được Hỗ trợ

| Kênh               | Hằng số                                  | Class                     |
| ------------------ | ---------------------------------------- | ------------------------- |
| Direct Line        | `BotChannelName.DirectLineChannel`       | `DirectLineChannel`       |
| Direct Line Speech | `BotChannelName.DirectLineSpeechChannel` | `DirectLineSpeechChannel` |
| Microsoft Teams    | `BotChannelName.MsTeamsChannel`          | `MsTeamsChannel`          |
| Web Chat           | `BotChannelName.WebChatChannel`          | `WebChatChannel`          |
| Slack              | `BotChannelName.SlackChannel`            | `SlackChannel`            |
| Facebook           | `BotChannelName.FacebookChannel`         | `FacebookChannel`         |
| Email              | `BotChannelName.EmailChannel`            | `EmailChannel`            |
| Telegram           | `BotChannelName.TelegramChannel`         | `TelegramChannel`         |
| Telephony          | `BotChannelName.TelephonyChannel`        | `TelephonyChannel`        |

## Tham khảo Các Loại Chính

| Loại                           | Mục đích                              |
| ------------------------------ | ------------------------------------- |
| `ArmClient`                    | Điểm vào cho tất cả các thao tác ARM  |
| `BotResource`                  | Đại diện cho một tài nguyên Azure Bot |
| `BotCollection`                | Collection cho CRUD bot               |
| `BotData`                      | Định nghĩa tài nguyên Bot             |
| `BotProperties`                | Các thuộc tính cấu hình Bot           |
| `BotChannelResource`           | Cấu hình kênh                         |
| `BotChannelCollection`         | Collection các kênh                   |
| `BotChannelData`               | Dữ liệu cấu hình kênh                 |
| `BotConnectionSettingResource` | Các cài đặt kết nối OAuth             |

## Giá trị BotServiceKind

| Giá trị                   | Mô tả                        |
| ------------------------- | ---------------------------- |
| `BotServiceKind.Azurebot` | Azure Bot (được khuyến nghị) |
| `BotServiceKind.Bot`      | Legacy Bot Framework bot     |
| `BotServiceKind.Designer` | Composer bot                 |
| `BotServiceKind.Function` | Function bot                 |
| `BotServiceKind.Sdk`      | SDK bot                      |

## Giá trị BotServiceSkuName

| Giá trị                | Mô tả         |
| ---------------------- | ------------- |
| `BotServiceSkuName.F0` | Free tier     |
| `BotServiceSkuName.S1` | Standard tier |

## Giá trị BotMsaAppType

| Giá trị                         | Mô tả                              |
| ------------------------------- | ---------------------------------- |
| `BotMsaAppType.MultiTenant`     | Ứng dụng đa thuê bao               |
| `BotMsaAppType.SingleTenant`    | Ứng dụng đơn thuê bao              |
| `BotMsaAppType.UserAssignedMSI` | Managed identity do người dùng gán |

## Thực hành Tốt nhất

1. **Luôn sử dụng `DefaultAzureCredential`** — hỗ trợ nhiều phương thức xác thực
2. **Sử dụng `WaitUntil.Completed`** cho các thao tác đồng bộ
3. **Xử lý `RequestFailedException`** cho các lỗi API
4. **Sử dụng các phương thức async** (`*Async`) cho tất cả các thao tác
5. **Lưu trữ thông tin xác thực MSA App an toàn** — sử dụng Key Vault cho secrets
6. **Sử dụng managed identity** (`BotMsaAppType.UserAssignedMSI`) cho các bot sản xuất
7. **Bật secure sites** cho các kênh DirectLine trong môi trường sản xuất

## Xử lý Lỗi

```csharp
using Azure;

try
{
    var operation = await botCollection.CreateOrUpdateAsync(
        WaitUntil.Completed,
        botName,
        botData);
}
catch (RequestFailedException ex) when (ex.Status == 409)
{
    Console.WriteLine("Bot already exists");
}
catch (RequestFailedException ex)
{
    Console.WriteLine($"ARM Error: {ex.Status} - {ex.ErrorCode}: {ex.Message}");
}
```

## Các SDK Liên quan

| SDK                                             | Mục đích              | Cài đặt                                                            |
| ----------------------------------------------- | --------------------- | ------------------------------------------------------------------ |
| `Azure.ResourceManager.BotService`              | Quản lý Bot (SDK này) | `dotnet add package Azure.ResourceManager.BotService`              |
| `Microsoft.Bot.Builder`                         | Bot Framework SDK     | `dotnet add package Microsoft.Bot.Builder`                         |
| `Microsoft.Bot.Builder.Integration.AspNet.Core` | Tích hợp ASP.NET Core | `dotnet add package Microsoft.Bot.Builder.Integration.AspNet.Core` |

## Liên kết Tham khảo

| Tài nguyên                 | URL                                                                                                  |
| -------------------------- | ---------------------------------------------------------------------------------------------------- |
| Gói NuGet                  | https://www.nuget.org/packages/Azure.ResourceManager.BotService                                      |
| Tham khảo API              | https://learn.microsoft.com/dotnet/api/azure.resourcemanager.botservice                              |
| Mã nguồn GitHub            | https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/botservice/Azure.ResourceManager.BotService |
| Tài liệu Azure Bot Service | https://learn.microsoft.com/azure/bot-service/                                                       |
