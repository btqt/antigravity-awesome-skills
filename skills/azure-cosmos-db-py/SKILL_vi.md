---
name: azure-cosmos-db-py
description: Xây dựng các dịch vụ Azure Cosmos DB NoSQL với Python/FastAPI theo các mẫu production-grade. Sử dụng khi triển khai thiết lập database client với xác thực kép (DefaultAzureCredential + emulator), các lớp service layer với các thao tác CRUD, chiến lược partition key, truy vấn tham số hóa, hoặc các mẫu TDD cho Cosmos. Kích hoạt với các cụm từ như "Cosmos DB", "NoSQL database", "document store", "add persistence", "database service layer", hoặc "Python Cosmos SDK".
package: azure-cosmos
---

# Triển khai Dịch vụ Cosmos DB

Xây dựng các dịch vụ Azure Cosmos DB NoSQL cấp sản xuất tuân theo clean code, các thực hành bảo mật tốt nhất, và nguyên tắc TDD.

## Cài đặt

```bash
pip install azure-cosmos azure-identity
```

## Biến Môi trường

```bash
COSMOS_ENDPOINT=https://<account>.documents.azure.com:443/
COSMOS_DATABASE_NAME=<database-name>
COSMOS_CONTAINER_ID=<container-id>
# Chỉ cho emulator (không dùng cho sản xuất)
COSMOS_KEY=<emulator-key>
```

## Xác thực

**DefaultAzureCredential (ưu tiên)**:

```python
from azure.cosmos import CosmosClient
from azure.identity import DefaultAzureCredential

client = CosmosClient(
    url=os.environ["COSMOS_ENDPOINT"],
    credential=DefaultAzureCredential()
)
```

**Emulator (phát triển cục bộ)**:

```python
from azure.cosmos import CosmosClient

client = CosmosClient(
    url="https://localhost:8081",
    credential=os.environ["COSMOS_KEY"],
    connection_verify=False
)
```

## Tổng quan Kiến trúc

```
┌─────────────────────────────────────────────────────────────────┐
│                         FastAPI Router                          │
│  - Auth dependencies (get_current_user, get_current_user_required)
│  - HTTP error responses (HTTPException)                         │
└──────────────────────────────┬──────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                        Service Layer                            │
│  - Business logic và validation                                 │
│  - Chuyển đổi Document ↔ Model                                  │
│  - Graceful degradation khi Cosmos không khả dụng               │
└──────────────────────────────┬──────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                     Cosmos DB Client Module                     │
│  - Khởi tạo singleton container                                 │
│  - Dual auth: DefaultAzureCredential (Azure) / Key (emulator)   │
│  - Wrapper Async qua run_in_threadpool                          │
└─────────────────────────────────────────────────────────────────┘
```

## Bắt đầu Nhanh

### 1. Thiết lập Client Module

Tạo một singleton Cosmos client với xác thực kép:

```python
# db/cosmos.py
from azure.cosmos import CosmosClient
from azure.identity import DefaultAzureCredential
from starlette.concurrency import run_in_threadpool

_cosmos_container = None

def _is_emulator_endpoint(endpoint: str) -> bool:
    return "localhost" in endpoint or "127.0.0.1" in endpoint

async def get_container():
    global _cosmos_container
    if _cosmos_container is None:
        if _is_emulator_endpoint(settings.cosmos_endpoint):
            client = CosmosClient(
                url=settings.cosmos_endpoint,
                credential=settings.cosmos_key,
                connection_verify=False
            )
        else:
            client = CosmosClient(
                url=settings.cosmos_endpoint,
                credential=DefaultAzureCredential()
            )
        db = client.get_database_client(settings.cosmos_database_name)
        _cosmos_container = db.get_container_client(settings.cosmos_container_id)
    return _cosmos_container
```

**Triển khai đầy đủ**: Xem [references/client-setup.md](references/client-setup.md)

### 2. Phân cấp Pydantic Model

Sử dụng mẫu model năm tầng để phân tách rõ ràng:

```python
class ProjectBase(BaseModel):           # Các trường chia sẻ
    name: str = Field(..., min_length=1, max_length=200)

class ProjectCreate(ProjectBase):       # Yêu cầu tạo
    workspace_id: str = Field(..., alias="workspaceId")

class ProjectUpdate(BaseModel):         # Cập nhật một phần (tất cả tùy chọn)
    name: Optional[str] = Field(None, min_length=1)

class Project(ProjectBase):             # Phản hồi API
    id: str
    created_at: datetime = Field(..., alias="createdAt")

class ProjectInDB(Project):             # Nội bộ với docType
    doc_type: str = "project"
```

### 3. Mẫu Service Layer

```python
class ProjectService:
    def _use_cosmos(self) -> bool:
        return get_container() is not None

    async def get_by_id(self, project_id: str, workspace_id: str) -> Project | None:
        if not self._use_cosmos():
            return None
        doc = await get_document(project_id, partition_key=workspace_id)
        if doc is None:
            return None
        return self._doc_to_model(doc)
```

**Các mẫu đầy đủ**: Xem [references/service-layer.md](references/service-layer.md)

## Các Nguyên tắc Cốt lõi

### Yêu cầu Bảo mật

1. **Xác thực RBAC**: Sử dụng `DefaultAzureCredential` trong Azure — không bao giờ lưu trữ key trong code
2. **Key chỉ cho Emulator**: Chỉ hardcode key emulator phổ biến cho phát triển cục bộ
3. **Truy vấn Tham số hóa**: Luôn sử dụng cú pháp `@parameter` — không bao giờ nối chuỗi
4. **Validation Partition Key**: Xác thực truy cập partition key khớp với ủy quyền người dùng

### Quy ước Clean Code

1. **Trách nhiệm Đơn nhất**: Client module xử lý kết nối; services xử lý logic nghiệp vụ
2. **Graceful Degradation**: Services trả về `None`/`[]` khi Cosmos không khả dụng
3. **Đặt tên Nhất quán**: `_doc_to_model()`, `_model_to_doc()`, `_use_cosmos()`
4. **Type Hints**: Đầy đủ typing trên tất cả các phương thức public
5. **CamelCase Aliases**: Sử dụng `Field(alias="camelCase")` cho JSON serialization

### Yêu cầu TDD

Viết test TRƯỚC KHI triển khai sử dụng các mẫu này:

```python
@pytest.fixture
def mock_cosmos_container(mocker):
    container = mocker.MagicMock()
    mocker.patch("app.db.cosmos.get_container", return_value=container)
    return container

@pytest.mark.asyncio
async def test_get_project_by_id_returns_project(mock_cosmos_container):
    # Arrange
    mock_cosmos_container.read_item.return_value = {"id": "123", "name": "Test"}

    # Act
    result = await project_service.get_by_id("123", "workspace-1")

    # Assert
    assert result.id == "123"
    assert result.name == "Test"
```

**Hướng dẫn test đầy đủ**: Xem [references/testing.md](references/testing.md)

## Tệp Tham khảo

| Tệp                                                          | Khi nào Đọc                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------------------ |
| [references/client-setup.md](references/client-setup.md)     | Thiết lập Cosmos client với dual auth, cấu hình SSL, mẫu singleton       |
| [references/service-layer.md](references/service-layer.md)   | Triển khai lớp service đầy đủ với CRUD, chuyển đổi, graceful degradation |
| [references/testing.md](references/testing.md)               | Viết pytest tests, mocking Cosmos, thiết lập integration test            |
| [references/partitioning.md](references/partitioning.md)     | Chọn partition keys, truy vấn xuyên partition, thao tác di chuyển        |
| [references/error-handling.md](references/error-handling.md) | Xử lý CosmosResourceNotFoundError, logging, ánh xạ lỗi HTTP              |

## Tệp Mẫu

| Tệp                                                                  | Mục đích                           |
| -------------------------------------------------------------------- | ---------------------------------- |
| [assets/cosmos_client_template.py](assets/cosmos_client_template.py) | Client module sẵn sàng sử dụng     |
| [assets/service_template.py](assets/service_template.py)             | Khung lớp Service                  |
| [assets/conftest_template.py](assets/conftest_template.py)           | pytest fixtures cho Cosmos mocking |

## Thuộc tính Chất lượng (NFRs)

### Độ tin cậy (Reliability)

- Graceful degradation khi Cosmos không khả dụng
- Logic thử lại với backoff theo cấp số nhân cho các lỗi thoáng qua
- Connection pooling qua mẫu singleton

### Bảo mật (Security)

- Không có bí mật trong code (RBAC qua DefaultAzureCredential)
- Truy vấn tham số hóa ngăn chặn injection
- Cô lập partition key thực thi ranh giới dữ liệu

### Khả năng Bảo trì (Maintainability)

- Mẫu model năm tầng cho phép phát triển schema
- Lớp service tách biệt logic nghiệp vụ khỏi lưu trữ
- Các mẫu nhất quán trên tất cả các entity services

### Khả năng Kiểm thử (Testability)

- Dependency injection qua `get_container()`
- Mocking dễ dàng với module-level globals
- Phân tách rõ ràng cho phép unit testing không cần Cosmos

### Hiệu năng (Performance)

- Truy vấn partition key tránh quét xuyên partition
- Wrapper Async ngăn chặn chặn vòng lặp sự kiện FastAPI
- Chi phí chuyển đổi document tối thiểu
