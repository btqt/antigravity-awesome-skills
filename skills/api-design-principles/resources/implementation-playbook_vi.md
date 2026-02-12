# Sổ tay Triển khai Các Nguyên tắc Thiết kế API

Tệp này chứa các mẫu chi tiết, danh sách kiểm tra và các mẫu mã được tham chiếu bởi kỹ năng.

## Các Khái niệm Cốt lõi

### 1. Các Nguyên tắc Thiết kế RESTful

**Kiến trúc Hướng Tài nguyên (Resource-Oriented Architecture)**

- Tài nguyên là danh từ (users, orders, products), không phải động từ
- Sử dụng các phương thức HTTP cho các hành động (GET, POST, PUT, PATCH, DELETE)
- URL đại diện cho phân cấp tài nguyên
- Quy ước đặt tên nhất quán

**Ngữ nghĩa Phương thức HTTP:**

- `GET`: Truy xuất tài nguyên (idempotent - lũy đẳng, an toàn)
- `POST`: Tạo tài nguyên mới
- `PUT`: Thay thế toàn bộ tài nguyên (idempotent)
- `PATCH`: Cập nhật một phần tài nguyên
- `DELETE`: Xóa tài nguyên (idempotent)

### 2. Các Nguyên tắc Thiết kế GraphQL

**Phát triển Ưu tiên Lược đồ (Schema-First Development)**

- Các loại (Types) định nghĩa mô hình miền của bạn
- Truy vấn (Queries) để đọc dữ liệu
- Đột biến (Mutations) để sửa đổi dữ liệu
- Đăng ký (Subscriptions) cho cập nhật thời gian thực

**Cấu trúc Truy vấn:**

- Khách hàng yêu cầu chính xác những gì họ cần
- Một điểm cuối duy nhất, nhiều hoạt động
- Lược đồ định kiểu mạnh (Strongly typed)
- Tích hợp sẵn khả năng tự kiểm tra (Introspection)

### 3. Các Chiến lược Đánh phiên bản API

**Đánh phiên bản qua URL:**

```
/api/v1/users
/api/v2/users
```

**Đánh phiên bản qua Header:**

```
Accept: application/vnd.api+json; version=1
```

**Đánh phiên bản qua Tham số Truy vấn:**

```
/api/users?version=1
```

## Các Mẫu Thiết kế API REST

### Mẫu 1: Thiết kế Bộ sưu tập Tài nguyên

```python
# Tốt: Các điểm cuối hướng tài nguyên
GET    /api/users              # Liệt kê người dùng (có phân trang)
POST   /api/users              # Tạo người dùng
GET    /api/users/{id}         # Lấy người dùng cụ thể
PUT    /api/users/{id}         # Thay thế người dùng
PATCH  /api/users/{id}         # Cập nhật trường người dùng
DELETE /api/users/{id}         # Xóa người dùng

# Tài nguyên lồng nhau
GET    /api/users/{id}/orders  # Lấy đơn hàng của người dùng
POST   /api/users/{id}/orders  # Tạo đơn hàng cho người dùng

# Xấu: Các điểm cuối hướng hành động (tránh dùng)
POST   /api/createUser
POST   /api/getUserById
POST   /api/deleteUser
```

### Mẫu 2: Phân trang và Lọc

```python
from typing import List, Optional
from pydantic import BaseModel, Field

class PaginationParams(BaseModel):
    page: int = Field(1, ge=1, description="Page number")
    page_size: int = Field(20, ge=1, le=100, description="Items per page")

class FilterParams(BaseModel):
    status: Optional[str] = None
    created_after: Optional[str] = None
    search: Optional[str] = None

class PaginatedResponse(BaseModel):
    items: List[dict]
    total: int
    page: int
    page_size: int
    pages: int

    @property
    def has_next(self) -> bool:
        return self.page < self.pages

    @property
    def has_prev(self) -> bool:
        return self.page > 1

# Ví dụ điểm cuối FastAPI
from fastapi import FastAPI, Query, Depends

app = FastAPI()

@app.get("/api/users", response_model=PaginatedResponse)
async def list_users(
    page: int = Query(1, ge=1),
    page_size: int = Query(20, ge=1, le=100),
    status: Optional[str] = Query(None),
    search: Optional[str] = Query(None)
):
    # Áp dụng bộ lọc
    query = build_query(status=status, search=search)

    # Đếm tổng số
    total = await count_users(query)

    # Lấy trang
    offset = (page - 1) * page_size
    users = await fetch_users(query, limit=page_size, offset=offset)

    return PaginatedResponse(
        items=users,
        total=total,
        page=page,
        page_size=page_size,
        pages=(total + page_size - 1) // page_size
    )
```

### Mẫu 3: Xử lý Lỗi và Mã Trạng thái

```python
from fastapi import HTTPException, status
from pydantic import BaseModel

class ErrorResponse(BaseModel):
    error: str
    message: str
    details: Optional[dict] = None
    timestamp: str
    path: str

class ValidationErrorDetail(BaseModel):
    field: str
    message: str
    value: Any

# Phản hồi lỗi nhất quán
STATUS_CODES = {
    "success": 200,
    "created": 201,
    "no_content": 204,
    "bad_request": 400,
    "unauthorized": 401,
    "forbidden": 403,
    "not_found": 404,
    "conflict": 409,
    "unprocessable": 422,
    "internal_error": 500
}

def raise_not_found(resource: str, id: str):
    raise HTTPException(
        status_code=status.HTTP_404_NOT_FOUND,
        detail={
            "error": "NotFound",
            "message": f"{resource} not found",
            "details": {"id": id}
        }
    )

def raise_validation_error(errors: List[ValidationErrorDetail]):
    raise HTTPException(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        detail={
            "error": "ValidationError",
            "message": "Request validation failed",
            "details": {"errors": [e.dict() for e in errors]}
        }
    )

# Ví dụ sử dụng
@app.get("/api/users/{user_id}")
async def get_user(user_id: str):
    user = await fetch_user(user_id)
    if not user:
        raise_not_found("User", user_id)
    return user
```

### Mẫu 4: HATEOAS (Hypermedia as the Engine of Application State)

```python
class UserResponse(BaseModel):
    id: str
    name: str
    email: str
    _links: dict

    @classmethod
    def from_user(cls, user: User, base_url: str):
        return cls(
            id=user.id,
            name=user.name,
            email=user.email,
            _links={
                "self": {"href": f"{base_url}/api/users/{user.id}"},
                "orders": {"href": f"{base_url}/api/users/{user.id}/orders"},
                "update": {
                    "href": f"{base_url}/api/users/{user.id}",
                    "method": "PATCH"
                },
                "delete": {
                    "href": f"{base_url}/api/users/{user.id}",
                    "method": "DELETE"
                }
            }
        )
```

## Các Mẫu Thiết kế GraphQL

### Mẫu 1: Thiết kế Lược đồ (Schema Design)

```graphql
# schema.graphql

# Định nghĩa kiểu rõ ràng
type User {
  id: ID!
  email: String!
  name: String!
  createdAt: DateTime!

  # Mối quan hệ
  orders(first: Int = 20, after: String, status: OrderStatus): OrderConnection!

  profile: UserProfile
}

type Order {
  id: ID!
  status: OrderStatus!
  total: Money!
  items: [OrderItem!]!
  createdAt: DateTime!

  # Tham chiếu ngược
  user: User!
}

# Mẫu phân trang (kiểu Relay)
type OrderConnection {
  edges: [OrderEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type OrderEdge {
  node: Order!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}

# Enums cho an toàn kiểu
enum OrderStatus {
  PENDING
  CONFIRMED
  SHIPPED
  DELIVERED
  CANCELLED
}

# Các scalar tùy chỉnh
scalar DateTime
scalar Money

# Gốc truy vấn
type Query {
  user(id: ID!): User
  users(first: Int = 20, after: String, search: String): UserConnection!

  order(id: ID!): Order
}

# Gốc đột biến
type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
  updateUser(input: UpdateUserInput!): UpdateUserPayload!
  deleteUser(id: ID!): DeleteUserPayload!

  createOrder(input: CreateOrderInput!): CreateOrderPayload!
}

# Các kiểu đầu vào cho đột biến
input CreateUserInput {
  email: String!
  name: String!
  password: String!
}

# Các kiểu payload cho đột biến
type CreateUserPayload {
  user: User
  errors: [Error!]
}

type Error {
  field: String
  message: String!
}
```

### Mẫu 2: Thiết kế Resolver

```python
from typing import Optional, List
from ariadne import QueryType, MutationType, ObjectType
from dataclasses import dataclass

query = QueryType()
mutation = MutationType()
user_type = ObjectType("User")

@query.field("user")
async def resolve_user(obj, info, id: str) -> Optional[dict]:
    """Resolve single user by ID."""
    return await fetch_user_by_id(id)

@query.field("users")
async def resolve_users(
    obj,
    info,
    first: int = 20,
    after: Optional[str] = None,
    search: Optional[str] = None
) -> dict:
    """Resolve paginated user list."""
    # Decode cursor
    offset = decode_cursor(after) if after else 0

    # Fetch users
    users = await fetch_users(
        limit=first + 1,  # Fetch one extra to check hasNextPage
        offset=offset,
        search=search
    )

    # Pagination
    has_next = len(users) > first
    if has_next:
        users = users[:first]

    edges = [
        {
            "node": user,
            "cursor": encode_cursor(offset + i)
        }
        for i, user in enumerate(users)
    ]

    return {
        "edges": edges,
        "pageInfo": {
            "hasNextPage": has_next,
            "hasPreviousPage": offset > 0,
            "startCursor": edges[0]["cursor"] if edges else None,
            "endCursor": edges[-1]["cursor"] if edges else None
        },
        "totalCount": await count_users(search=search)
    }

@user_type.field("orders")
async def resolve_user_orders(user: dict, info, first: int = 20) -> dict:
    """Resolve user's orders (N+1 prevention with DataLoader)."""
    # Use DataLoader to batch requests
    loader = info.context["loaders"]["orders_by_user"]
    orders = await loader.load(user["id"])

    return paginate_orders(orders, first)

@mutation.field("createUser")
async def resolve_create_user(obj, info, input: dict) -> dict:
    """Create new user."""
    try:
        # Validate input
        validate_user_input(input)

        # Create user
        user = await create_user(
            email=input["email"],
            name=input["name"],
            password=hash_password(input["password"])
        )

        return {
            "user": user,
            "errors": []
        }
    except ValidationError as e:
        return {
            "user": None,
            "errors": [{"field": e.field, "message": e.message}]
        }
    # ...
```

### Mẫu 3: DataLoader (Ngăn chặn vấn đề N+1)

```python
from aiodataloader import DataLoader
from typing import List, Optional

class UserLoader(DataLoader):
    """Batch load users by ID."""

    async def batch_load_fn(self, user_ids: List[str]) -> List[Optional[dict]]:
        """Load multiple users in single query."""
        users = await fetch_users_by_ids(user_ids)

        # Map results back to input order
        user_map = {user["id"]: user for user in users}
        return [user_map.get(user_id) for user_id in user_ids]

class OrdersByUserLoader(DataLoader):
    """Batch load orders by user ID."""

    async def batch_load_fn(self, user_ids: List[str]) -> List[List[dict]]:
        """Load orders for multiple users in single query."""
        orders = await fetch_orders_by_user_ids(user_ids)

        # Group orders by user_id
        orders_by_user = {}
        for order in orders:
            user_id = order["user_id"]
            if user_id not in orders_by_user:
                orders_by_user[user_id] = []
            orders_by_user[user_id].append(order)

        # Return in input order
        return [orders_by_user.get(user_id, []) for user_id in user_ids]

# Context setup
def create_context():
    return {
        "loaders": {
            "user": UserLoader(),
            "orders_by_user": OrdersByUserLoader()
        }
    }
```

## Các Thực hành Tốt nhất

### REST APIs

1. **Đặt tên Nhất quán**: Sử dụng danh từ số nhiều cho các bộ sưu tập (`/users`, không phải `/user`)
2. **Không trạng thái (Stateless)**: Mỗi yêu cầu chứa tất cả thông tin cần thiết
3. **Sử dụng Mã Trạng thái HTTP Chính xác**: 2xx thành công, 4xx lỗi máy khách, 5xx lỗi máy chủ
4. **Đánh phiên bản API của bạn**: Lên kế hoạch cho các thay đổi phá vỡ ngay từ ngày đầu tiên
5. **Phân trang**: Luôn phân trang các bộ sưu tập lớn
6. **Giới hạn tốc độ (Rate Limiting)**: Bảo vệ API của bạn bằng các giới hạn tốc độ
7. **Tài liệu**: Sử dụng OpenAPI/Swagger cho tài liệu tương tác

### GraphQL APIs

1. **Ưu tiên Lược đồ**: Thiết kế lược đồ trước khi viết resolvers
2. **Tránh N+1**: Sử dụng DataLoaders để lấy dữ liệu hiệu quả
3. **Xác thực Đầu vào**: Xác thực ở cấp độ lược đồ và resolver
4. **Xử lý Lỗi**: Trả về lỗi có cấu trúc trong payload của đột biến
5. **Phân trang**: Sử dụng phân trang dựa trên con trỏ (đặc tả Relay)
6. **Ngừng hỗ trợ (Deprecation)**: Sử dụng chỉ thị `@deprecated` cho việc di chuyển dần dần
7. **Giám sát**: Theo dõi độ phức tạp và thời gian thực thi của truy vấn

## Các Cạm bẫy Phổ biến

- **Lấy quá nhiều/Lấy thiếu (REST)**: Đã được khắc phục trong GraphQL nhưng yêu cầu DataLoaders
- **Thay đổi Phá vỡ**: Đánh phiên bản API hoặc sử dụng các chiến lược ngừng hỗ trợ
- **Định dạng Lỗi Không nhất quán**: Chuẩn hóa các phản hồi lỗi
- **Thiếu Giới hạn Tốc độ**: Các API không có giới hạn dễ bị lạm dụng
- **Tài liệu Kém**: Các API không có tài liệu làm nản lòng các nhà phát triển
- **Bỏ qua Ngữ nghĩa HTTP**: POST cho các hoạt động lũy đẳng (idempotent) phá vỡ các kỳ vọng
- **Kết nối Chặt chẽ**: Cấu trúc API không nên phản ánh lược đồ cơ sở dữ liệu

## Tài nguyên

- **references/rest-best-practices.md**: Hướng dẫn thiết kế API REST toàn diện
- **references/graphql-schema-design.md**: Các mẫu và phản mẫu lược đồ GraphQL
- **references/api-versioning-strategies.md**: Các cách tiếp cận đánh phiên bản và lộ trình di chuyển
- **assets/rest-api-template.py**: Mẫu API REST FastAPI
- **assets/graphql-schema-template.graphql**: Ví dụ lược đồ GraphQL hoàn chỉnh
- **assets/api-design-checklist.md**: Danh sách kiểm tra đánh giá trước khi triển khai
- **scripts/openapi-generator.py**: Tạo thông số kỹ thuật OpenAPI từ mã
