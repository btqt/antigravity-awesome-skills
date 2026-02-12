# Thực hành Tốt nhất cho REST API

## Cấu trúc URL

### Đặt tên Tài nguyên

```
# Tốt - Danh từ số nhiều
GET /api/users
GET /api/orders
GET /api/products

# Xấu - Động từ hoặc quy ước hỗn hợp
GET /api/getUser
GET /api/user  (số ít không nhất quán)
POST /api/createOrder
```

### Tài nguyên Lồng nhau

```
# Lồng nhau nông (được ưa thích)
GET /api/users/{id}/orders
GET /api/orders/{id}

# Lồng nhau sâu (tránh dùng)
GET /api/users/{id}/orders/{orderId}/items/{itemId}/reviews
# Tốt hơn:
GET /api/order-items/{id}/reviews
```

## Phương thức HTTP và Mã Trạng thái

### GET - Truy xuất Tài nguyên

```
GET /api/users              → 200 OK (với danh sách)
GET /api/users/{id}         → 200 OK hoặc 404 Not Found
GET /api/users?page=2       → 200 OK (có phân trang)
```

### POST - Tạo Tài nguyên

```
POST /api/users
  Body: {"name": "John", "email": "john@example.com"}
  → 201 Created
  Location: /api/users/123
  Body: {"id": "123", "name": "John", ...}

POST /api/users (lỗi xác thực)
  → 422 Unprocessable Entity
  Body: {"errors": [...]}
```

### PUT - Thay thế Tài nguyên

```
PUT /api/users/{id}
  Body: {đối tượng người dùng đầy đủ}
  → 200 OK (đã cập nhật)
  → 404 Not Found (không tồn tại)

# Phải bao gồm TẤT CẢ các trường
```

### PATCH - Cập nhật Một phần

```
PATCH /api/users/{id}
  Body: {"name": "Jane"}  (chỉ các trường thay đổi)
  → 200 OK
  → 404 Not Found
```

### DELETE - Xóa Tài nguyên

```
DELETE /api/users/{id}
  → 204 No Content (đã xóa)
  → 404 Not Found
  → 409 Conflict (không thể xóa do tham chiếu)
```

## Lọc, Sắp xếp và Tìm kiếm

### Tham số Truy vấn

```
# Lọc
GET /api/users?status=active
GET /api/users?role=admin&status=active

# Sắp xếp
GET /api/users?sort=created_at
GET /api/users?sort=-created_at  (giảm dần)
GET /api/users?sort=name,created_at

# Tìm kiếm
GET /api/users?search=john
GET /api/users?q=john

# Lựa chọn trường (sparse fieldsets)
GET /api/users?fields=id,name,email
```

## Các Mẫu Phân trang

### Phân trang dựa trên Offset

```python
GET /api/users?page=2&page_size=20

Phản hồi:
{
  "items": [...],
  "page": 2,
  "page_size": 20,
  "total": 150,
  "pages": 8
}
```

### Phân trang dựa trên Con trỏ (cho tập dữ liệu lớn)

```python
GET /api/users?limit=20&cursor=eyJpZCI6MTIzfQ

Phản hồi:
{
  "items": [...],
  "next_cursor": "eyJpZCI6MTQzfQ",
  "has_more": true
}
```

### Phân trang Header Liên kết (RESTful)

```
GET /api/users?page=2

Tiêu đề Phản hồi:
Link: <https://api.example.com/users?page=3>; rel="next",
      <https://api.example.com/users?page=1>; rel="prev",
      <https://api.example.com/users?page=1>; rel="first",
      <https://api.example.com/users?page=8>; rel="last"
```

## Các Chiến lược Đánh phiên bản

### Đánh phiên bản URL (Được khuyến nghị)

```
/api/v1/users
/api/v2/users

Ưu điểm: Rõ ràng, dễ định tuyến
Nhược điểm: Nhiều URL cho cùng một tài nguyên
```

### Đánh phiên bản Header

```
GET /api/users
Accept: application/vnd.api+json; version=2

Ưu điểm: URL sạch
Nhược điểm: Ít hiển thị hơn, khó kiểm thử hơn
```

### Tham số Truy vấn

```
GET /api/users?version=2

Ưu điểm: Dễ dàng kiểm thử
Nhược điểm: Tham số tùy chọn có thể bị quên
```

## Giới hạn Tốc độ

### Headers

```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 742
X-RateLimit-Reset: 1640000000

Phản hồi khi bị giới hạn:
429 Too Many Requests
Retry-After: 3600
```

### Mẫu Triển khai

```python
from fastapi import HTTPException, Request
from datetime import datetime, timedelta

class RateLimiter:
    def __init__(self, calls: int, period: int):
        self.calls = calls
        self.period = period
        self.cache = {}

    def check(self, key: str) -> bool:
        now = datetime.now()
        if key not in self.cache:
            self.cache[key] = []

        # Xóa các yêu cầu cũ
        self.cache[key] = [
            ts for ts in self.cache[key]
            if now - ts < timedelta(seconds=self.period)
        ]

        if len(self.cache[key]) >= self.calls:
            return False

        self.cache[key].append(now)
        return True

limiter = RateLimiter(calls=100, period=60)

@app.get("/api/users")
async def get_users(request: Request):
    if not limiter.check(request.client.host):
        raise HTTPException(
            status_code=429,
            headers={"Retry-After": "60"}
        )
    return {"users": [...]}
```

## Xác thực và Phân quyền

### Bearer Token

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

401 Unauthorized - Thiếu/token không hợp lệ
403 Forbidden - Token hợp lệ, không đủ quyền
```

### API Keys

```
X-API-Key: your-api-key-here
```

## Định dạng Phản hồi Lỗi

### Cấu trúc Nhất quán

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format",
        "value": "not-an-email"
      }
    ],
    "timestamp": "2025-10-16T12:00:00Z",
    "path": "/api/users"
  }
}
```

### Hướng dẫn Mã Trạng thái

- `200 OK`: Thành công GET, PATCH, PUT
- `201 Created`: Thành công POST
- `204 No Content`: Thành công DELETE
- `400 Bad Request`: Yêu cầu không đúng định dạng
- `401 Unauthorized`: Yêu cầu xác thực
- `403 Forbidden`: Đã xác thực nhưng không được ủy quyền
- `404 Not Found`: Tài nguyên không tồn tại
- `409 Conflict`: Xung đột trạng thái (email trùng lặp, v.v.)
- `422 Unprocessable Entity`: Lỗi xác thực
- `429 Too Many Requests`: Bị giới hạn tốc độ
- `500 Internal Server Error`: Lỗi máy chủ
- `503 Service Unavailable`: Thời gian chết tạm thời

## Lưu trữ đệm (Caching)

### Cache Headers

```
# Caching phía máy khách
Cache-Control: public, max-age=3600

# Không caching
Cache-Control: no-cache, no-store, must-revalidate

# Yêu cầu có điều kiện
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
If-None-Match: "33a64df551425fcc55e4d42a148795d9f25f89d4"
→ 304 Not Modified
```

## Hoạt động Hàng loạt

### Các điểm cuối Batch

```python
POST /api/users/batch
{
  "items": [
    {"name": "User1", "email": "user1@example.com"},
    {"name": "User2", "email": "user2@example.com"}
  ]
}

Phản hồi:
{
  "results": [
    {"id": "1", "status": "created"},
    {"id": null, "status": "failed", "error": "Email already exists"}
  ]
}
```

## Tính Lũy đẳng (Idempotency)

### Khóa Lũy đẳng (Idempotency Keys)

```
POST /api/orders
Idempotency-Key: unique-key-123

Nếu yêu cầu trùng lặp:
→ 200 OK (trả về phản hồi đã lưu trong cache)
```

## Cấu hình CORS

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://example.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

## Tài liệu với OpenAPI

```python
from fastapi import FastAPI

app = FastAPI(
    title="My API",
    description="API for managing users",
    version="1.0.0",
    docs_url="/docs",
    redoc_url="/redoc"
)

@app.get(
    "/api/users/{user_id}",
    summary="Get user by ID",
    response_description="User details",
    tags=["Users"]
)
async def get_user(
    user_id: str = Path(..., description="The user ID")
):
    """
    Retrieve user by ID.

    Returns full user profile including:
    - Basic information
    - Contact details
    - Account status
    """
    pass
```

## Các điểm cuối Sức khỏe và Giám sát

```python
@app.get("/health")
async def health_check():
    return {
        "status": "healthy",
        "version": "1.0.0",
        "timestamp": datetime.now().isoformat()
    }

@app.get("/health/detailed")
async def detailed_health():
    return {
        "status": "healthy",
        "checks": {
            "database": await check_database(),
            "redis": await check_redis(),
            "external_api": await check_external_api()
        }
    }
```
