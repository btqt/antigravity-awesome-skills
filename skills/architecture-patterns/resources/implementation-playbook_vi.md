# Sách Hướng dẫn Triển khai Các Mẫu Kiến trúc

Tệp này chứa các mẫu chi tiết, danh sách kiểm tra, và mã mẫu được tham chiếu bởi kỹ năng.

## Các Khái niệm Cốt lõi

### 1. Kiến trúc Sạch (Clean Architecture - Uncle Bob)

**Các lớp (luồng phụ thuộc hướng vào trong):**

- **Entities**: Mô hình nghiệp vụ cốt lõi
- **Use Cases**: Quy tắc nghiệp vụ ứng dụng
- **Interface Adapters**: Controllers, presenters, gateways
- **Frameworks & Drivers**: UI, cơ sở dữ liệu, dịch vụ bên ngoài

**Các Nguyên tắc Chính:**

- Sự phụ thuộc hướng vào trong
- Các lớp bên trong không biết gì về các lớp bên ngoài
- Logic nghiệp vụ độc lập với frameworks
- Có thể kiểm thử mà không cần UI, cơ sở dữ liệu, hoặc dịch vụ bên ngoài

### 2. Kiến trúc Lục giác (Hexagonal Architecture - Ports and Adapters)

**Các Thành phần:**

- **Domain Core**: Logic nghiệp vụ
- **Ports**: Giao diện xác định các tương tác
- **Adapters**: Triển khai của các ports (cơ sở dữ liệu, REST, hàng đợi tin nhắn)

**Lợi ích:**

- Hoán đổi triển khai dễ dàng (mock để kiểm thử)
- Cốt lõi bất khả tri công nghệ (Technology-agnostic)
- Phân tách mối quan tâm rõ ràng

### 3. Thiết kế Hướng Tên miền (Domain-Driven Design - DDD)

**Các Mẫu Chiến lược:**

- **Bounded Contexts**: Các mô hình riêng biệt cho các tên miền khác nhau
- **Context Mapping**: Cách các ngữ cảnh liên hệ với nhau
- **Ubiquitous Language**: Thuật ngữ chung

**Các Mẫu Chiến thuật:**

- **Entities**: Các đối tượng có danh tính
- **Value Objects**: Các đối tượng bất biến được xác định bởi thuộc tính
- **Aggregates**: Ranh giới nhất quán
- **Repositories**: Trừu tượng hóa truy cập dữ liệu
- **Domain Events**: Những điều đã xảy ra

## Mẫu Kiến trúc Sạch

### Cấu trúc Thư mục

```
app/
├── domain/           # Entities & quy tắc nghiệp vụ
│   ├── entities/
│   │   ├── user.py
│   │   └── order.py
│   ├── value_objects/
│   │   ├── email.py
│   │   └── money.py
│   └── interfaces/   # Giao diện trừu tượng
│       ├── user_repository.py
│       └── payment_gateway.py
├── use_cases/        # Quy tắc nghiệp vụ ứng dụng
│   ├── create_user.py
│   ├── process_order.py
│   └── send_notification.py
├── adapters/         # Triển khai giao diện
│   ├── repositories/
│   │   ├── postgres_user_repository.py
│   │   └── redis_cache_repository.py
│   ├── controllers/
│   │   └── user_controller.py
│   └── gateways/
│       ├── stripe_payment_gateway.py
│       └── sendgrid_email_gateway.py
└── infrastructure/   # Framework & mối quan tâm bên ngoài
    ├── database.py
    ├── config.py
    └── logging.py
```

### Ví dụ Triển khai

```python
# domain/entities/user.py
from dataclasses import dataclass
from datetime import datetime
from typing import Optional

@dataclass
class User:
    """Core user entity - no framework dependencies."""
    id: str
    email: str
    name: str
    created_at: datetime
    is_active: bool = True

    def deactivate(self):
        """Business rule: deactivating user."""
        self.is_active = False

    def can_place_order(self) -> bool:
        """Business rule: active users can order."""
        return self.is_active

# domain/interfaces/user_repository.py
from abc import ABC, abstractmethod
from typing import Optional, List
from domain.entities.user import User

class IUserRepository(ABC):
    """Port: defines contract, no implementation."""

    @abstractmethod
    async def find_by_id(self, user_id: str) -> Optional[User]:
        pass

    @abstractmethod
    async def find_by_email(self, email: str) -> Optional[User]:
        pass

    @abstractmethod
    async def save(self, user: User) -> User:
        pass

    @abstractmethod
    async def delete(self, user_id: str) -> bool:
        pass

# use_cases/create_user.py
from domain.entities.user import User
from domain.interfaces.user_repository import IUserRepository
from dataclasses import dataclass
from datetime import datetime
import uuid

@dataclass
class CreateUserRequest:
    email: str
    name: str

@dataclass
class CreateUserResponse:
    user: User
    success: bool
    error: Optional[str] = None

class CreateUserUseCase:
    """Use case: orchestrates business logic."""

    def __init__(self, user_repository: IUserRepository):
        self.user_repository = user_repository

    async def execute(self, request: CreateUserRequest) -> CreateUserResponse:
        # Business validation
        existing = await self.user_repository.find_by_email(request.email)
        if existing:
            return CreateUserResponse(
                user=None,
                success=False,
                error="Email already exists"
            )

        # Create entity
        user = User(
            id=str(uuid.uuid4()),
            email=request.email,
            name=request.name,
            created_at=datetime.now(),
            is_active=True
        )

        # Persist
        saved_user = await self.user_repository.save(user)

        return CreateUserResponse(
            user=saved_user,
            success=True
        )

# adapters/repositories/postgres_user_repository.py
from domain.interfaces.user_repository import IUserRepository
from domain.entities.user import User
from typing import Optional
import asyncpg

class PostgresUserRepository(IUserRepository):
    """Adapter: PostgreSQL implementation."""

    def __init__(self, pool: asyncpg.Pool):
        self.pool = pool

    async def find_by_id(self, user_id: str) -> Optional[User]:
        async with self.pool.acquire() as conn:
            row = await conn.fetchrow(
                "SELECT * FROM users WHERE id = $1", user_id
            )
            return self._to_entity(row) if row else None

    async def find_by_email(self, email: str) -> Optional[User]:
        async with self.pool.acquire() as conn:
            row = await conn.fetchrow(
                "SELECT * FROM users WHERE email = $1", email
            )
            return self._to_entity(row) if row else None

    async def save(self, user: User) -> User:
        async with self.pool.acquire() as conn:
            await conn.execute(
                """
                INSERT INTO users (id, email, name, created_at, is_active)
                VALUES ($1, $2, $3, $4, $5)
                ON CONFLICT (id) DO UPDATE
                SET email = $2, name = $3, is_active = $5
                """,
                user.id, user.email, user.name, user.created_at, user.is_active
            )
            return user

    async def delete(self, user_id: str) -> bool:
        async with self.pool.acquire() as conn:
            result = await conn.execute(
                "DELETE FROM users WHERE id = $1", user_id
            )
            return result == "DELETE 1"

    def _to_entity(self, row) -> User:
        """Map database row to entity."""
        return User(
            id=row["id"],
            email=row["email"],
            name=row["name"],
            created_at=row["created_at"],
            is_active=row["is_active"]
        )

# adapters/controllers/user_controller.py
from fastapi import APIRouter, Depends, HTTPException
from use_cases.create_user import CreateUserUseCase, CreateUserRequest
from pydantic import BaseModel

router = APIRouter()

class CreateUserDTO(BaseModel):
    email: str
    name: str

@router.post("/users")
async def create_user(
    dto: CreateUserDTO,
    use_case: CreateUserUseCase = Depends(get_create_user_use_case)
):
    """Controller: handles HTTP concerns only."""
    request = CreateUserRequest(email=dto.email, name=dto.name)
    response = await use_case.execute(request)

    if not response.success:
        raise HTTPException(status_code=400, detail=response.error)

    return {"user": response.user}
```

## Mẫu Kiến trúc Lục giác

```python
# Core domain (hexagon center)
class OrderService:
    """Domain service - no infrastructure dependencies."""

    def __init__(
        self,
        order_repository: OrderRepositoryPort,
        payment_gateway: PaymentGatewayPort,
        notification_service: NotificationPort
    ):
        self.orders = order_repository
        self.payments = payment_gateway
        self.notifications = notification_service

    async def place_order(self, order: Order) -> OrderResult:
        # Business logic
        if not order.is_valid():
            return OrderResult(success=False, error="Invalid order")

        # Use ports (interfaces)
        payment = await self.payments.charge(
            amount=order.total,
            customer=order.customer_id
        )

        if not payment.success:
            return OrderResult(success=False, error="Payment failed")

        order.mark_as_paid()
        saved_order = await self.orders.save(order)

        await self.notifications.send(
            to=order.customer_email,
            subject="Order confirmed",
            body=f"Order {order.id} confirmed"
        )

        return OrderResult(success=True, order=saved_order)

# Ports (interfaces)
class OrderRepositoryPort(ABC):
    @abstractmethod
    async def save(self, order: Order) -> Order:
        pass

class PaymentGatewayPort(ABC):
    @abstractmethod
    async def charge(self, amount: Money, customer: str) -> PaymentResult:
        pass

class NotificationPort(ABC):
    @abstractmethod
    async def send(self, to: str, subject: str, body: str):
        pass

# Adapters (implementations)
class StripePaymentAdapter(PaymentGatewayPort):
    """Primary adapter: connects to Stripe API."""

    def __init__(self, api_key: str):
        self.stripe = stripe
        self.stripe.api_key = api_key

    async def charge(self, amount: Money, customer: str) -> PaymentResult:
        try:
            charge = self.stripe.Charge.create(
                amount=amount.cents,
                currency=amount.currency,
                customer=customer
            )
            return PaymentResult(success=True, transaction_id=charge.id)
        except stripe.error.CardError as e:
            return PaymentResult(success=False, error=str(e))

class MockPaymentAdapter(PaymentGatewayPort):
    """Test adapter: no external dependencies."""

    async def charge(self, amount: Money, customer: str) -> PaymentResult:
        return PaymentResult(success=True, transaction_id="mock-123")
```

## Mẫu Thiết kế Hướng Tên miền (DDD)

```python
# Value Objects (immutable)
from dataclasses import dataclass
from typing import Optional

@dataclass(frozen=True)
class Email:
    """Value object: validated email."""
    value: str

    def __post_init__(self):
        if "@" not in self.value:
            raise ValueError("Invalid email")

@dataclass(frozen=True)
class Money:
    """Value object: amount with currency."""
    amount: int  # cents
    currency: str

    def add(self, other: "Money") -> "Money":
        if self.currency != other.currency:
            raise ValueError("Currency mismatch")
        return Money(self.amount + other.amount, self.currency)

# Entities (with identity)
class Order:
    """Entity: has identity, mutable state."""

    def __init__(self, id: str, customer: Customer):
        self.id = id
        self.customer = customer
        self.items: List[OrderItem] = []
        self.status = OrderStatus.PENDING
        self._events: List[DomainEvent] = []

    def add_item(self, product: Product, quantity: int):
        """Business logic in entity."""
        item = OrderItem(product, quantity)
        self.items.append(item)
        self._events.append(ItemAddedEvent(self.id, item))

    def total(self) -> Money:
        """Calculated property."""
        return sum(item.subtotal() for item in self.items)

    def submit(self):
        """State transition with business rules."""
        if not self.items:
            raise ValueError("Cannot submit empty order")
        if self.status != OrderStatus.PENDING:
            raise ValueError("Order already submitted")

        self.status = OrderStatus.SUBMITTED
        self._events.append(OrderSubmittedEvent(self.id))

# Aggregates (consistency boundary)
class Customer:
    """Aggregate root: controls access to entities."""

    def __init__(self, id: str, email: Email):
        self.id = id
        self.email = email
        self._addresses: List[Address] = []
        self._orders: List[str] = []  # Order IDs, not full objects

    def add_address(self, address: Address):
        """Aggregate enforces invariants."""
        if len(self._addresses) >= 5:
            raise ValueError("Maximum 5 addresses allowed")
        self._addresses.append(address)

    @property
    def primary_address(self) -> Optional[Address]:
        return next((a for a in self._addresses if a.is_primary), None)

# Domain Events
@dataclass
class OrderSubmittedEvent:
    order_id: str
    occurred_at: datetime = field(default_factory=datetime.now)

# Repository (aggregate persistence)
class OrderRepository:
    """Repository: persist/retrieve aggregates."""

    async def find_by_id(self, order_id: str) -> Optional[Order]:
        """Reconstitute aggregate from storage."""
        pass

    async def save(self, order: Order):
        """Persist aggregate and publish events."""
        await self._persist(order)
        await self._publish_events(order._events)
        order._events.clear()
```

## Tài nguyên

- **references/clean-architecture-guide.md**: Phân tích chi tiết các lớp
- **references/hexagonal-architecture-guide.md**: Các mẫu ports và adapters
- **references/ddd-tactical-patterns.md**: Entities, value objects, aggregates
- **assets/clean-architecture-template/**: Cấu trúc dự án hoàn chỉnh
- **assets/ddd-examples/**: Các ví dụ mô hình hóa tên miền

## Thực hành Tốt nhất

1. **Quy tắc Phụ thuộc**: Sự phụ thuộc luôn hướng vào trong
2. **Phân tách Giao diện**: Giao diện nhỏ, tập trung
3. **Logic Nghiệp vụ trong Tên miền**: Giữ frameworks bên ngoài cốt lõi
4. **Kiểm thử Độc lập**: Cốt lõi có thể kiểm thử mà không cần cơ sở hạ tầng
5. **Bounded Contexts**: Ranh giới tên miền rõ ràng
6. **Ubiquitous Language**: Thuật ngữ nhất quán
7. **Thin Controllers**: Ủy quyền cho use cases
8. **Rich Domain Models**: Hành vi đi kèm dữ liệu

## Các Cạm bẫy Phổ biến

- **Tên miền Thiếu máu (Anemic Domain)**: Entities chỉ có dữ liệu, không có hành vi
- **Ghép nối Framework**: Logic nghiệp vụ phụ thuộc vào frameworks
- **Fat Controllers**: Logic nghiệp vụ trong controllers
- **Rò rỉ Repository**: Để lộ các đối tượng ORM
- **Thiếu Trừu tượng hóa**: Sự phụ thuộc cụ thể trong cốt lõi
- **Kỹ thuật Quá mức (Over-Engineering)**: Kiến trúc sạch cho CRUD đơn giản
