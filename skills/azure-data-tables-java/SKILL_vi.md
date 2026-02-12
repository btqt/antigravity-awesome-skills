---
name: azure-data-tables-java
description: Xây dựng các ứng dụng lưu trữ bảng với Azure Tables SDK cho Java. Sử dụng khi làm việc với Azure Table Storage hoặc Cosmos DB Table API cho dữ liệu key-value NoSQL, lưu trữ phi cấu trúc (schemaless), hoặc dữ liệu có cấu trúc ở quy mô lớn.
package: com.azure:azure-data-tables
---

# Azure Tables SDK cho Java

Xây dựng các ứng dụng lưu trữ bảng sử dụng Azure Tables SDK cho Java. Hoạt động với cả Azure Table Storage và Cosmos DB Table API.

## Cài đặt

```xml
<dependency>
  <groupId>com.azure</groupId>
  <artifactId>azure-data-tables</artifactId>
  <version>12.6.0-beta.1</version>
</dependency>
```

## Tạo Client

### Với Connection String

```java
import com.azure.data.tables.TableServiceClient;
import com.azure.data.tables.TableServiceClientBuilder;
import com.azure.data.tables.TableClient;

TableServiceClient serviceClient = new TableServiceClientBuilder()
    .connectionString("<your-connection-string>")
    .buildClient();
```

### Với Shared Key

```java
import com.azure.core.credential.AzureNamedKeyCredential;

AzureNamedKeyCredential credential = new AzureNamedKeyCredential(
    "<account-name>",
    "<account-key>");

TableServiceClient serviceClient = new TableServiceClientBuilder()
    .endpoint("<your-table-account-url>")
    .credential(credential)
    .buildClient();
```

### Với SAS Token

```java
TableServiceClient serviceClient = new TableServiceClientBuilder()
    .endpoint("<your-table-account-url>")
    .sasToken("<sas-token>")
    .buildClient();
```

### Với DefaultAzureCredential (Chỉ Storage)

```java
import com.azure.identity.DefaultAzureCredentialBuilder;

TableServiceClient serviceClient = new TableServiceClientBuilder()
    .endpoint("<your-table-account-url>")
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildClient();
```

## Các Khái niệm Chính

- **TableServiceClient**: Quản lý các bảng (tạo, liệt kê, xóa)
- **TableClient**: Quản lý các thực thể (entities) trong một bảng (CRUD)
- **Partition Key**: Nhóm các thực thể để truy vấn hiệu quả
- **Row Key**: Định danh duy nhất trong một partition
- **Entity**: Một hàng với tối đa 252 thuộc tính (1MB Storage, 2MB Cosmos)

## Các Mẫu Cốt lõi

### Tạo Bảng

```java
// Tạo bảng (ném ngoại lệ nếu đã tồn tại)
TableClient tableClient = serviceClient.createTable("mytable");

// Tạo nếu chưa tồn tại (không ngoại lệ)
TableClient tableClient = serviceClient.createTableIfNotExists("mytable");
```

### Lấy Table Client

```java
// Từ service client
TableClient tableClient = serviceClient.getTableClient("mytable");

// Khởi tạo trực tiếp
TableClient tableClient = new TableClientBuilder()
    .connectionString("<connection-string>")
    .tableName("mytable")
    .buildClient();
```

### Tạo Entity

```java
import com.azure.data.tables.models.TableEntity;

TableEntity entity = new TableEntity("partitionKey", "rowKey")
    .addProperty("Name", "Product A")
    .addProperty("Price", 29.99)
    .addProperty("Quantity", 100)
    .addProperty("IsAvailable", true);

tableClient.createEntity(entity);
```

### Lấy Entity

```java
TableEntity entity = tableClient.getEntity("partitionKey", "rowKey");

String name = (String) entity.getProperty("Name");
Double price = (Double) entity.getProperty("Price");
System.out.printf("Product: %s, Price: %.2f%n", name, price);
```

### Cập nhật Entity

```java
import com.azure.data.tables.models.TableEntityUpdateMode;

// Merge (chỉ cập nhật các thuộc tính được chỉ định)
TableEntity updateEntity = new TableEntity("partitionKey", "rowKey")
    .addProperty("Price", 24.99);
tableClient.updateEntity(updateEntity, TableEntityUpdateMode.MERGE);

// Replace (thay thế toàn bộ entity)
TableEntity replaceEntity = new TableEntity("partitionKey", "rowKey")
    .addProperty("Name", "Product A Updated")
    .addProperty("Price", 24.99)
    .addProperty("Quantity", 150);
tableClient.updateEntity(replaceEntity, TableEntityUpdateMode.REPLACE);
```

### Upsert Entity

```java
// Insert hoặc update (chế độ merge)
tableClient.upsertEntity(entity, TableEntityUpdateMode.MERGE);

// Insert hoặc replace
tableClient.upsertEntity(entity, TableEntityUpdateMode.REPLACE);
```

### Xóa Entity

```java
tableClient.deleteEntity("partitionKey", "rowKey");
```

### Liệt kê Entities

```java
import com.azure.data.tables.models.ListEntitiesOptions;

// Liệt kê tất cả entities
for (TableEntity entity : tableClient.listEntities()) {
    System.out.printf("%s - %s%n",
        entity.getPartitionKey(),
        entity.getRowKey());
}

// Với lọc và chọn
ListEntitiesOptions options = new ListEntitiesOptions()
    .setFilter("PartitionKey eq 'sales'")
    .setSelect("Name", "Price");

for (TableEntity entity : tableClient.listEntities(options, null, null)) {
    System.out.printf("%s: %.2f%n",
        entity.getProperty("Name"),
        entity.getProperty("Price"));
}
```

### Truy vấn với OData Filter

```java
// Lọc theo partition key
ListEntitiesOptions options = new ListEntitiesOptions()
    .setFilter("PartitionKey eq 'electronics'");

// Lọc với nhiều điều kiện
options.setFilter("PartitionKey eq 'electronics' and Price gt 100");

// Lọc với các toán tử so sánh
options.setFilter("Quantity ge 10 and Quantity le 100");

// Top N kết quả
options.setTop(10);

for (TableEntity entity : tableClient.listEntities(options, null, null)) {
    System.out.println(entity.getRowKey());
}
```

### Thao tác Hàng loạt (Transactions)

```java
import com.azure.data.tables.models.TableTransactionAction;
import com.azure.data.tables.models.TableTransactionActionType;
import java.util.Arrays;

// Tất cả entities phải có cùng partition key
List<TableTransactionAction> actions = Arrays.asList(
    new TableTransactionAction(
        TableTransactionActionType.CREATE,
        new TableEntity("batch", "row1").addProperty("Name", "Item 1")),
    new TableTransactionAction(
        TableTransactionActionType.CREATE,
        new TableEntity("batch", "row2").addProperty("Name", "Item 2")),
    new TableTransactionAction(
        TableTransactionActionType.UPSERT_MERGE,
        new TableEntity("batch", "row3").addProperty("Name", "Item 3"))
);

tableClient.submitTransaction(actions);
```

### Liệt kê Bảng (Tables)

```java
import com.azure.data.tables.models.TableItem;
import com.azure.data.tables.models.ListTablesOptions;

// Liệt kê tất cả các bảng
for (TableItem table : serviceClient.listTables()) {
    System.out.println(table.getName());
}

// Lọc bảng
ListTablesOptions options = new ListTablesOptions()
    .setFilter("TableName eq 'mytable'");

for (TableItem table : serviceClient.listTables(options, null, null)) {
    System.out.println(table.getName());
}
```

### Xóa Bảng

```java
serviceClient.deleteTable("mytable");
```

## Typed Entities

```java
public class Product implements TableEntity {
    private String partitionKey;
    private String rowKey;
    private OffsetDateTime timestamp;
    private String eTag;
    private String name;
    private double price;

    // Getters và setters cho tất cả các trường
    @Override
    public String getPartitionKey() { return partitionKey; }
    @Override
    public void setPartitionKey(String partitionKey) { this.partitionKey = partitionKey; }
    @Override
    public String getRowKey() { return rowKey; }
    @Override
    public void setRowKey(String rowKey) { this.rowKey = rowKey; }
    // ... các getters/setters khác

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public double getPrice() { return price; }
    public void setPrice(double price) { this.price = price; }
}

// Sử dụng
Product product = new Product();
product.setPartitionKey("electronics");
product.setRowKey("laptop-001");
product.setName("Laptop");
product.setPrice(999.99);

tableClient.createEntity(product);
```

## Xử lý Lỗi

```java
import com.azure.data.tables.models.TableServiceException;

try {
    tableClient.createEntity(entity);
} catch (TableServiceException e) {
    System.out.println("Status: " + e.getResponse().getStatusCode());
    System.out.println("Error: " + e.getMessage());
    // 409 = Conflict (entity đã tồn tại)
    // 404 = Not Found
}
```

## Biến Môi trường

```bash
# Storage Account
AZURE_TABLES_CONNECTION_STRING=DefaultEndpointsProtocol=https;AccountName=...
AZURE_TABLES_ENDPOINT=https://<account>.table.core.windows.net

# Cosmos DB Table API
COSMOS_TABLE_ENDPOINT=https://<account>.table.cosmosdb.azure.com
```

## Thực hành Tốt nhất

1. **Thiết kế Partition Key**: Chọn key phân phối tải đều
2. **Thao tác Hàng loạt**: Sử dụng transaction cho cập nhật đa thực thể nguyên tử (atomic)
3. **Tối ưu hóa Truy vấn**: Luôn lọc theo PartitionKey khi có thể
4. **Select Projection**: Chỉ chọn các thuộc tính cần thiết để tăng hiệu năng
5. **Kích thước Entity**: Giữ entities dưới 1MB (Storage) hoặc 2MB (Cosmos)

## Các Cụm từ Kích hoạt

- "Azure Tables Java"
- "table storage SDK"
- "Cosmos DB Table API"
- "NoSQL key-value storage"
- "partition key row key"
- "table entity CRUD"
