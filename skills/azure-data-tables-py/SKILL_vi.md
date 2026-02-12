---
name: azure-data-tables-py
description: |
  Azure Tables SDK cho Python (Storage và Cosmos DB). Sử dụng cho lưu trữ key-value NoSQL, CRUD thực thể (entity), và các thao tác hàng loạt.
  Triggers: "table storage", "TableServiceClient", "TableClient", "entities", "PartitionKey", "RowKey".
package: azure-data-tables
---

# Azure Tables SDK cho Python

Lưu trữ key-value NoSQL cho dữ liệu có cấu trúc (Azure Storage Tables hoặc Cosmos DB Table API).

## Cài đặt

```bash
pip install azure-data-tables azure-identity
```

## Biến Môi trường

```bash
# Azure Storage Tables
AZURE_STORAGE_ACCOUNT_URL=https://<account>.table.core.windows.net

# Cosmos DB Table API
COSMOS_TABLE_ENDPOINT=https://<account>.table.cosmos.azure.com
```

## Xác thực

```python
from azure.identity import DefaultAzureCredential
from azure.data.tables import TableServiceClient, TableClient

credential = DefaultAzureCredential()
endpoint = "https://<account>.table.core.windows.net"

# Service client (quản lý bảng)
service_client = TableServiceClient(endpoint=endpoint, credential=credential)

# Table client (làm việc với entities)
table_client = TableClient(endpoint=endpoint, table_name="mytable", credential=credential)
```

## Các Loại Client

| Client               | Mục đích                   |
| -------------------- | -------------------------- |
| `TableServiceClient` | Tạo/xóa bảng, liệt kê bảng |
| `TableClient`        | Entity CRUD, truy vấn      |

## Các Thao tác Bảng

```python
# Tạo bảng
service_client.create_table("mytable")

# Tạo nếu chưa tồn tại
service_client.create_table_if_not_exists("mytable")

# Xóa bảng
service_client.delete_table("mytable")

# Liệt kê bảng
for table in service_client.list_tables():
    print(table.name)

# Lấy table client
table_client = service_client.get_table_client("mytable")
```

## Các Thao tác Entity

**Quan trọng**: Mọi entity yêu cầu `PartitionKey` và `RowKey` (cùng nhau tạo thành ID duy nhất).

### Tạo Entity

```python
entity = {
    "PartitionKey": "sales",
    "RowKey": "order-001",
    "product": "Widget",
    "quantity": 5,
    "price": 9.99,
    "shipped": False
}

# Tạo (thất bại nếu đã tồn tại)
table_client.create_entity(entity=entity)

# Upsert (tạo hoặc thay thế)
table_client.upsert_entity(entity=entity)
```

### Lấy Entity

```python
# Lấy theo key (nhanh nhất)
entity = table_client.get_entity(
    partition_key="sales",
    row_key="order-001"
)
print(f"Product: {entity['product']}")
```

### Cập nhật Entity

```python
# Thay thế toàn bộ entity
entity["quantity"] = 10
table_client.update_entity(entity=entity, mode="replace")

# Merge (chỉ cập nhật các trường cụ thể)
update = {
    "PartitionKey": "sales",
    "RowKey": "order-001",
    "shipped": True
}
table_client.update_entity(entity=update, mode="merge")
```

### Xóa Entity

```python
table_client.delete_entity(
    partition_key="sales",
    row_key="order-001"
)
```

## Truy vấn Entities

### Truy vấn Trong Partition

```python
# Truy vấn theo partition (hiệu quả)
entities = table_client.query_entities(
    query_filter="PartitionKey eq 'sales'"
)
for entity in entities:
    print(entity)
```

### Truy vấn với Bộ lọc

```python
# Lọc theo thuộc tính
entities = table_client.query_entities(
    query_filter="PartitionKey eq 'sales' and quantity gt 3"
)

# Với tham số (an toàn hơn)
entities = table_client.query_entities(
    query_filter="PartitionKey eq @pk and price lt @max_price",
    parameters={"pk": "sales", "max_price": 50.0}
)
```

### Chọn Thuộc tính Cụ thể

```python
entities = table_client.query_entities(
    query_filter="PartitionKey eq 'sales'",
    select=["RowKey", "product", "price"]
)
```

### Liệt kê Tất cả Entities

```python
# Liệt kê tất cả (xuyên partition - sử dụng hạn chế)
for entity in table_client.list_entities():
    print(entity)
```

## Thao tác Hàng loạt

```python
from azure.data.tables import TableTransactionError

# Thao tác hàng loạt (chỉ trong cùng một partition!)
operations = [
    ("create", {"PartitionKey": "batch", "RowKey": "1", "data": "first"}),
    ("create", {"PartitionKey": "batch", "RowKey": "2", "data": "second"}),
    ("upsert", {"PartitionKey": "batch", "RowKey": "3", "data": "third"}),
]

try:
    table_client.submit_transaction(operations)
except TableTransactionError as e:
    print(f"Transaction failed: {e}")
```

## Async Client

```python
from azure.data.tables.aio import TableServiceClient, TableClient
from azure.identity.aio import DefaultAzureCredential

async def table_operations():
    credential = DefaultAzureCredential()

    async with TableClient(
        endpoint="https://<account>.table.core.windows.net",
        table_name="mytable",
        credential=credential
    ) as client:
        # Tạo
        await client.create_entity(entity={
            "PartitionKey": "async",
            "RowKey": "1",
            "data": "test"
        })

        # Truy vấn
        async for entity in client.query_entities("PartitionKey eq 'async'"):
            print(entity)

import asyncio
asyncio.run(table_operations())
```

## Các Kiểu Dữ liệu

| Kiểu Python | Kiểu Table Storage |
| ----------- | ------------------ |
| `str`       | String             |
| `int`       | Int64              |
| `float`     | Double             |
| `bool`      | Boolean            |
| `datetime`  | DateTime           |
| `bytes`     | Binary             |
| `UUID`      | Guid               |

## Thực hành Tốt nhất

1. **Thiết kế partition key** cho các mẫu truy vấn và phân phối đều
2. **Truy vấn trong partitions** bất cứ khi nào có thể (xuyên partition rất tốn kém)
3. **Sử dụng thao tác hàng loạt** cho nhiều entities trong cùng một partition
4. **Sử dụng `upsert_entity`** cho ghi idempotent
5. **Sử dụng truy vấn tham số hóa** để ngăn chặn injection
6. **Giữ entities nhỏ** — tối đa 1MB mỗi entity
7. **Sử dụng async client** cho các kịch bản thông lượng cao
