---
name: azure-maps-search-dotnet
description: |
  Azure Maps SDK cho .NET. Các dịch vụ dựa trên vị trí bao gồm geocoding, định tuyến, rendering, geolocation, và thời tiết. Sử dụng cho tìm kiếm địa chỉ, chỉ đường, bản đồ gạch (map tiles), IP geolocation, và dữ liệu thời tiết. Triggers: "Azure Maps", "MapsSearchClient", "MapsRoutingClient", "MapsRenderingClient", "geocoding .NET", "route directions", "map tiles", "geolocation".
package: Azure.Maps.Search
---

# Azure Maps (.NET)

Azure Maps SDK cho .NET cung cấp các dịch vụ dựa trên vị trí: geocoding, định tuyến (routing), rendering, geolocation, và thời tiết.

## Cài đặt

```bash
# Search (geocoding, reverse geocoding)
dotnet add package Azure.Maps.Search --prerelease

# Routing (directions, route matrix)
dotnet add package Azure.Maps.Routing --prerelease

# Rendering (map tiles, static images)
dotnet add package Azure.Maps.Rendering --prerelease

# Geolocation (IP to location)
dotnet add package Azure.Maps.Geolocation --prerelease

# Weather (Thời tiết)
dotnet add package Azure.Maps.Weather --prerelease

# Resource Management (quản lý tài khoản, SAS tokens)
dotnet add package Azure.ResourceManager.Maps --prerelease

# Yêu cầu cho xác thực
dotnet add package Azure.Identity
```

**Phiên bản Hiện tại**:

- `Azure.Maps.Search`: v2.0.0-beta.5
- `Azure.Maps.Routing`: v1.0.0-beta.4
- `Azure.Maps.Rendering`: v2.0.0-beta.1
- `Azure.Maps.Geolocation`: v1.0.0-beta.3
- `Azure.ResourceManager.Maps`: v1.1.0-beta.2

## Biến Môi trường

```bash
AZURE_MAPS_SUBSCRIPTION_KEY=<your-subscription-key>
AZURE_MAPS_CLIENT_ID=<your-client-id>  # Cho xác thực Entra ID
```

## Xác thực

### Subscription Key (Shared Key)

```csharp
using Azure;
using Azure.Maps.Search;

var subscriptionKey = Environment.GetEnvironmentVariable("AZURE_MAPS_SUBSCRIPTION_KEY");
var credential = new AzureKeyCredential(subscriptionKey);

var client = new MapsSearchClient(credential);
```

### Microsoft Entra ID (Khuyên dùng cho Sản xuất)

```csharp
using Azure.Identity;
using Azure.Maps.Search;

var credential = new DefaultAzureCredential();
var clientId = Environment.GetEnvironmentVariable("AZURE_MAPS_CLIENT_ID");

var client = new MapsSearchClient(credential, clientId);
```

### Shared Access Signature (SAS)

```csharp
using Azure;
using Azure.Core;
using Azure.Identity;
using Azure.ResourceManager;
using Azure.ResourceManager.Maps;
using Azure.ResourceManager.Maps.Models;
using Azure.Maps.Search;

// Authenticate với Azure Resource Manager
ArmClient armClient = new ArmClient(new DefaultAzureCredential());

// Lấy Maps account resource
ResourceIdentifier mapsAccountResourceId = MapsAccountResource.CreateResourceIdentifier(
    subscriptionId, resourceGroupName, accountName);
MapsAccountResource mapsAccount = armClient.GetMapsAccountResource(mapsAccountResourceId);

// Tạo SAS token
MapsAccountSasContent sasContent = new MapsAccountSasContent(
    MapsSigningKey.PrimaryKey,
    principalId,
    maxRatePerSecond: 500,
    start: DateTime.UtcNow.ToString("O"),
    expiry: DateTime.UtcNow.AddDays(1).ToString("O"));

Response<MapsAccountSasToken> sas = mapsAccount.GetSas(sasContent);

// Tạo client với SAS token
var sasCredential = new AzureSasCredential(sas.Value.AccountSasToken);
var client = new MapsSearchClient(sasCredential);
```

## Phân cấp Client

```
Azure.Maps.Search
└── MapsSearchClient
    ├── GetGeocoding()                    → Geocode addresses
    ├── GetGeocodingBatch()               → Batch geocoding
    ├── GetReverseGeocoding()             → Coordinates to address
    ├── GetReverseGeocodingBatch()        → Batch reverse geocoding
    └── GetPolygon()                      → Get boundary polygons

Azure.Maps.Routing
└── MapsRoutingClient
    ├── GetDirections()                   → Route directions
    ├── GetImmediateRouteMatrix()         → Route matrix (sync, ≤100)
    ├── GetRouteMatrix()                  → Route matrix (async, ≤700)
    └── GetRouteRange()                   → Isochrone/reachable range

Azure.Maps.Rendering
└── MapsRenderingClient
    ├── GetMapTile()                      → Map tiles
    ├── GetMapStaticImage()               → Static map images
    └── GetCopyrightCaption()             → Copyright info

Azure.Maps.Geolocation
└── MapsGeolocationClient
    └── GetCountryCode()                  → IP to country/region

Azure.Maps.Weather
└── MapsWeatherClient
    ├── GetCurrentWeatherConditions()     → Current weather
    ├── GetDailyForecast()                → Daily forecast
    ├── GetHourlyForecast()               → Hourly forecast
    └── GetSevereWeatherAlerts()          → Weather alerts
```

## Các Quy trình Làm việc Cốt lõi

### 1. Geocoding (Địa chỉ sang Tọa độ)

```csharp
using Azure;
using Azure.Maps.Search;

var credential = new AzureKeyCredential(subscriptionKey);
var client = new MapsSearchClient(credential);

Response<GeocodingResponse> result = client.GetGeocoding("1 Microsoft Way, Redmond, WA 98052");

foreach (var feature in result.Value.Features)
{
    Console.WriteLine($"Coordinates: {string.Join(",", feature.Geometry.Coordinates)}");
    Console.WriteLine($"Address: {feature.Properties.Address.FormattedAddress}");
    Console.WriteLine($"Confidence: {feature.Properties.Confidence}");
}
```

### 2. Batch Geocoding

```csharp
using Azure.Maps.Search.Models.Queries;

List<GeocodingQuery> queries = new List<GeocodingQuery>
{
    new GeocodingQuery() { Query = "400 Broad St, Seattle, WA" },
    new GeocodingQuery() { Query = "1 Microsoft Way, Redmond, WA" },
    new GeocodingQuery() { AddressLine = "Space Needle", Top = 1 },
};

Response<GeocodingBatchResponse> results = client.GetGeocodingBatch(queries);

foreach (var batchItem in results.Value.BatchItems)
{
    foreach (var feature in batchItem.Features)
    {
        Console.WriteLine($"Coordinates: {string.Join(",", feature.Geometry.Coordinates)}");
    }
}
```

### 3. Reverse Geocoding (Tọa độ sang Địa chỉ)

```csharp
using Azure.Core.GeoJson;

GeoPosition coordinates = new GeoPosition(-122.138685, 47.6305637);
Response<GeocodingResponse> result = client.GetReverseGeocoding(coordinates);

foreach (var feature in result.Value.Features)
{
    Console.WriteLine($"Address: {feature.Properties.Address.FormattedAddress}");
    Console.WriteLine($"Locality: {feature.Properties.Address.Locality}");
}
```

### 4. Lấy Polygon Ranh giới (Boundary Polygon)

```csharp
using Azure.Maps.Search.Models;

GetPolygonOptions options = new GetPolygonOptions()
{
    Coordinates = new GeoPosition(-122.204141, 47.61256),
    ResultType = BoundaryResultTypeEnum.Locality,
    Resolution = ResolutionEnum.Small,
};

Response<Boundary> result = client.GetPolygon(options);

Console.WriteLine($"Boundary copyright: {result.Value.Properties?.Copyright}");
Console.WriteLine($"Polygon count: {result.Value.Geometry.Count}");
```

### 5. Chỉ đường (Route Directions)

```csharp
using Azure;
using Azure.Core.GeoJson;
using Azure.Maps.Routing;
using Azure.Maps.Routing.Models;

var client = new MapsRoutingClient(new AzureKeyCredential(subscriptionKey));

List<GeoPosition> routePoints = new List<GeoPosition>()
{
    new GeoPosition(-122.34, 47.61),  // Seattle
    new GeoPosition(-122.13, 47.64)   // Redmond
};

RouteDirectionQuery query = new RouteDirectionQuery(routePoints);
Response<RouteDirections> result = client.GetDirections(query);

foreach (var route in result.Value.Routes)
{
    Console.WriteLine($"Distance: {route.Summary.LengthInMeters} meters");
    Console.WriteLine($"Duration: {route.Summary.TravelTimeDuration}");

    foreach (RouteLeg leg in route.Legs)
    {
        Console.WriteLine($"Leg points: {leg.Points.Count}");
    }
}
```

### 6. Chỉ đường với Tùy chọn

```csharp
RouteDirectionOptions options = new RouteDirectionOptions()
{
    RouteType = RouteType.Fastest,
    UseTrafficData = true,
    TravelMode = TravelMode.Bicycle,
    Language = RoutingLanguage.EnglishUsa,
    InstructionsType = RouteInstructionsType.Text,
};

RouteDirectionQuery query = new RouteDirectionQuery(routePoints)
{
    RouteDirectionOptions = options
};

Response<RouteDirections> result = client.GetDirections(query);
```

### 7. Ma trận Lộ trình (Route Matrix)

```csharp
RouteMatrixQuery routeMatrixQuery = new RouteMatrixQuery
{
    Origins = new List<GeoPosition>()
    {
        new GeoPosition(-122.34, 47.61),
        new GeoPosition(-122.13, 47.64)
    },
    Destinations = new List<GeoPosition>()
    {
        new GeoPosition(-122.20, 47.62),
        new GeoPosition(-122.40, 47.65)
    },
};

// Đồng bộ (lên đến 100 tổ hợp lộ trình)
Response<RouteMatrixResult> result = client.GetImmediateRouteMatrix(routeMatrixQuery);

foreach (var cell in result.Value.Matrix.SelectMany(row => row))
{
    Console.WriteLine($"Distance: {cell.Response?.RouteSummary?.LengthInMeters}");
    Console.WriteLine($"Duration: {cell.Response?.RouteSummary?.TravelTimeDuration}");
}

// Bất đồng bộ (lên đến 700 tổ hợp lộ trình)
RouteMatrixOptions routeMatrixOptions = new RouteMatrixOptions(routeMatrixQuery)
{
    TravelTimeType = TravelTimeType.All,
};
GetRouteMatrixOperation asyncResult = client.GetRouteMatrix(WaitUntil.Completed, routeMatrixOptions);
```

### 8. Phạm vi Lộ trình (Isochrone)

```csharp
RouteRangeOptions options = new RouteRangeOptions(-122.34, 47.61)
{
    TimeBudget = new TimeSpan(0, 20, 0)  // 20 phút
};

Response<RouteRangeResult> result = client.GetRouteRange(options);

// result.Value.ReachableRange chứa polygon
Console.WriteLine($"Boundary points: {result.Value.ReachableRange.Boundary.Count}");
```

### 9. Lấy Bản đồ Gạch (Map Tiles)

```csharp
using Azure;
using Azure.Maps.Rendering;

var client = new MapsRenderingClient(new AzureKeyCredential(subscriptionKey));

int zoom = 10;
int tileSize = 256;

// Chuyển đổi tọa độ sang chỉ số tile (tile index)
MapTileIndex tileIndex = MapsRenderingClient.PositionToTileXY(
    new GeoPosition(13.3854, 52.517), zoom, tileSize);

// Lấy map tile
GetMapTileOptions options = new GetMapTileOptions(
    MapTileSetId.MicrosoftImagery,
    new MapTileIndex(tileIndex.X, tileIndex.Y, zoom)
);

Response<Stream> mapTile = client.GetMapTile(options);

// Lưu vào file
using (FileStream fileStream = File.Create("./MapTile.png"))
{
    mapTile.Value.CopyTo(fileStream);
}
```

### 10. IP Geolocation

```csharp
using System.Net;
using Azure;
using Azure.Maps.Geolocation;

var client = new MapsGeolocationClient(new AzureKeyCredential(subscriptionKey));

IPAddress ipAddress = IPAddress.Parse("2001:4898:80e8:b::189");
Response<CountryRegionResult> result = client.GetCountryCode(ipAddress);

Console.WriteLine($"Country ISO Code: {result.Value.IsoCode}");
```

### 11. Thời tiết Hiện tại

```csharp
using Azure;
using Azure.Core.GeoJson;
using Azure.Maps.Weather;

var client = new MapsWeatherClient(new AzureKeyCredential(subscriptionKey));

var position = new GeoPosition(-122.13071, 47.64011);
var options = new GetCurrentWeatherConditionsOptions(position);

Response<CurrentConditionsResult> result = client.GetCurrentWeatherConditions(options);

foreach (var condition in result.Value.Results)
{
    Console.WriteLine($"Temperature: {condition.Temperature.Value} {condition.Temperature.Unit}");
    Console.WriteLine($"Weather: {condition.Phrase}");
    Console.WriteLine($"Humidity: {condition.RelativeHumidity}%");
}
```

## Tham khảo Các Loại Chính

### Gói Search

| Loại                     | Mục đích                                       |
| ------------------------ | ---------------------------------------------- |
| `MapsSearchClient`       | Client chính cho các thao tác tìm kiếm         |
| `GeocodingResponse`      | Kết quả Geocoding                              |
| `GeocodingBatchResponse` | Kết quả Batch geocoding                        |
| `GeocodingQuery`         | Truy vấn cho batch geocoding                   |
| `ReverseGeocodingQuery`  | Truy vấn cho batch reverse geocoding           |
| `GetPolygonOptions`      | Tùy chọn để lấy polygon                        |
| `Boundary`               | Kết quả polygon ranh giới                      |
| `BoundaryResultTypeEnum` | Loại ranh giới (Locality, AdminDistrict, v.v.) |
| `ResolutionEnum`         | Độ phân giải polygon (Small, Medium, Large)    |

### Gói Routing

| Loại                    | Mục đích                                           |
| ----------------------- | -------------------------------------------------- |
| `MapsRoutingClient`     | Client chính cho các thao tác định tuyến           |
| `RouteDirectionQuery`   | Truy vấn chỉ đường                                 |
| `RouteDirectionOptions` | Tùy chọn tính toán lộ trình                        |
| `RouteDirections`       | Kết quả chỉ đường                                  |
| `RouteLeg`              | Đoạn của một lộ trình                              |
| `RouteMatrixQuery`      | Truy vấn ma trận lộ trình                          |
| `RouteMatrixResult`     | Kết quả ma trận lộ trình                           |
| `RouteRangeOptions`     | Tùy chọn cho isochrone                             |
| `RouteRangeResult`      | Kết quả isochrone                                  |
| `RouteType`             | Loại lộ trình (Fastest, Shortest, Eco, Thrilling)  |
| `TravelMode`            | Chế độ di chuyển (Car, Truck, Bicycle, Pedestrian) |

### Gói Rendering

| Loại                  | Mục đích                   |
| --------------------- | -------------------------- |
| `MapsRenderingClient` | Client chính cho rendering |
| `GetMapTileOptions`   | Tùy chọn bản đồ gạch       |
| `MapTileIndex`        | Tọa độ tile (X, Y, Zoom)   |
| `MapTileSetId`        | Định danh bộ tile          |

### Các Loại Chung

| Loại             | Mục đích                        |
| ---------------- | ------------------------------- |
| `GeoPosition`    | Vị trí địa lý (kinh độ, vĩ độ)  |
| `GeoBoundingBox` | Hộp giới hạn cho khu vực địa lý |

## Thực hành Tốt nhất

1. **Sử dụng Entra ID cho sản xuất** — Ưu tiên hơn subscription keys
2. **Batch operations** — Sử dụng batch geocoding cho nhiều địa chỉ
3. **Cache kết quả** — Kết quả Geocoding không thay đổi thường xuyên
4. **Sử dụng kích thước tile phù hợp** — 256 hoặc 512 pixels tùy theo hiển thị
5. **Xử lý giới hạn tốc độ (rate limits)** — Triển khai exponential backoff
6. **Sử dụng async route matrix** — Cho các tính toán ma trận lớn (>100)
7. **Cân nhắc dữ liệu giao thông** — Đặt `UseTrafficData = true` để có ETA chính xác

## Xử lý Lỗi

```csharp
try
{
    Response<GeocodingResponse> result = client.GetGeocoding(address);
}
catch (RequestFailedException ex)
{
    Console.WriteLine($"Status: {ex.Status}");
    Console.WriteLine($"Error: {ex.Message}");

    switch (ex.Status)
    {
        case 400:
            // Tham số yêu cầu không hợp lệ
            break;
        case 401:
            // Xác thực thất bại
            break;
        case 429:
            // Bị giới hạn tốc độ - triển khai backoff
            break;
    }
}
```

## Các SDK Liên quan

| SDK                          | Mục đích              | Cài đặt                                                      |
| ---------------------------- | --------------------- | ------------------------------------------------------------ |
| `Azure.Maps.Search`          | Geocoding, tìm kiếm   | `dotnet add package Azure.Maps.Search --prerelease`          |
| `Azure.Maps.Routing`         | Chỉ đường, ma trận    | `dotnet add package Azure.Maps.Routing --prerelease`         |
| `Azure.Maps.Rendering`       | Bản đồ gạch, hình ảnh | `dotnet add package Azure.Maps.Rendering --prerelease`       |
| `Azure.Maps.Geolocation`     | IP geolocation        | `dotnet add package Azure.Maps.Geolocation --prerelease`     |
| `Azure.Maps.Weather`         | Dữ liệu thời tiết     | `dotnet add package Azure.Maps.Weather --prerelease`         |
| `Azure.ResourceManager.Maps` | Quản lý tài khoản     | `dotnet add package Azure.ResourceManager.Maps --prerelease` |

## Liên kết Tham khảo

| Tài nguyên            | URL                                                           |
| --------------------- | ------------------------------------------------------------- |
| Tài liệu Azure Maps   | https://learn.microsoft.com/azure/azure-maps/                 |
| Tham khảo API Search  | https://learn.microsoft.com/dotnet/api/azure.maps.search      |
| Tham khảo API Routing | https://learn.microsoft.com/dotnet/api/azure.maps.routing     |
| Mã nguồn GitHub       | https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/maps |
| Giá cả                | https://azure.microsoft.com/pricing/details/azure-maps/       |
