---
name: azure-postgres-ts
description: |
  Kết nối đến Azure Database for PostgreSQL Flexible Server từ Node.js/TypeScript sử dụng gói pg (node-postgres). Sử dụng cho các truy vấn PostgreSQL, kết nối pooling, transactions, và xác thực Microsoft Entra ID (không cần mật khẩu - passwordless). Triggers: "PostgreSQL", "postgres", "pg client", "node-postgres", "Azure PostgreSQL connection", "PostgreSQL TypeScript", "pg Pool", "passwordless postgres".
package: pg
---

# Azure PostgreSQL cho TypeScript (node-postgres)

Kết nối đến Azure Database for PostgreSQL Flexible Server sử dụng gói `pg` (node-postgres) với hỗ trợ xác thực mật khẩu và Microsoft Entra ID (passwordless).

## Cài đặt

```bash
npm install pg @azure/identity
npm install -D @types/pg
```

## Biến Môi trường

```bash
# Bắt buộc
AZURE_POSTGRESQL_HOST=<server>.postgres.database.azure.com
AZURE_POSTGRESQL_DATABASE=<database>
AZURE_POSTGRESQL_PORT=5432

# Cho xác thực mật khẩu
AZURE_POSTGRESQL_USER=<username>
AZURE_POSTGRESQL_PASSWORD=<password>

# Cho xác thực Entra ID
AZURE_POSTGRESQL_USER=<entra-user>@<server>   # ví dụ: user@contoso.com
AZURE_POSTGRESQL_CLIENTID=<managed-identity-client-id>  # Cho user-assigned identity
```

## Xác thực

### Tùy chọn 1: Xác thực Mật khẩu

```typescript
import { Client, Pool } from "pg";

const client = new Client({
  host: process.env.AZURE_POSTGRESQL_HOST,
  database: process.env.AZURE_POSTGRESQL_DATABASE,
  user: process.env.AZURE_POSTGRESQL_USER,
  password: process.env.AZURE_POSTGRESQL_PASSWORD,
  port: Number(process.env.AZURE_POSTGRESQL_PORT) || 5432,
  ssl: { rejectUnauthorized: true }, // Bắt buộc đối với Azure
});

await client.connect();
```

### Tùy chọn 2: Microsoft Entra ID (Passwordless) - Khuyên dùng

```typescript
import { Client, Pool } from "pg";
import { DefaultAzureCredential } from "@azure/identity";

// Cho system-assigned managed identity
const credential = new DefaultAzureCredential();

// Cho user-assigned managed identity
// const credential = new DefaultAzureCredential({
//   managedIdentityClientId: process.env.AZURE_POSTGRESQL_CLIENTID
// });

// Lấy access token cho Azure PostgreSQL
const tokenResponse = await credential.getToken(
  "https://ossrdbms-aad.database.windows.net/.default",
);

const client = new Client({
  host: process.env.AZURE_POSTGRESQL_HOST,
  database: process.env.AZURE_POSTGRESQL_DATABASE,
  user: process.env.AZURE_POSTGRESQL_USER, // Người dùng Entra ID
  password: tokenResponse.token, // Token làm mật khẩu
  port: Number(process.env.AZURE_POSTGRESQL_PORT) || 5432,
  ssl: { rejectUnauthorized: true },
});

await client.connect();
```

## Các Quy trình Công việc Cốt lõi

### 1. Kết nối Client Đơn lẻ

```typescript
import { Client } from "pg";

const client = new Client({
  host: process.env.AZURE_POSTGRESQL_HOST,
  database: process.env.AZURE_POSTGRESQL_DATABASE,
  user: process.env.AZURE_POSTGRESQL_USER,
  password: process.env.AZURE_POSTGRESQL_PASSWORD,
  port: 5432,
  ssl: { rejectUnauthorized: true },
});

try {
  await client.connect();

  const result = await client.query("SELECT NOW() as current_time");
  console.log(result.rows[0].current_time);
} finally {
  await client.end(); // Luôn đóng kết nối
}
```

### 2. Connection Pool (Khuyên dùng cho Production)

```typescript
import { Pool } from "pg";

const pool = new Pool({
  host: process.env.AZURE_POSTGRESQL_HOST,
  database: process.env.AZURE_POSTGRESQL_DATABASE,
  user: process.env.AZURE_POSTGRESQL_USER,
  password: process.env.AZURE_POSTGRESQL_PASSWORD,
  port: 5432,
  ssl: { rejectUnauthorized: true },

  // Cấu hình Pool
  max: 20, // Số lượng kết nối tối đa trong pool
  idleTimeoutMillis: 30000, // Đóng các kết nối nhàn rỗi sau 30s
  connectionTimeoutMillis: 10000, // Timeout cho các kết nối mới
});

// Truy vấn sử dụng pool (tự động lấy và giải phóng kết nối)
const result = await pool.query("SELECT * FROM users WHERE id = $1", [userId]);

// Checkout tường minh cho nhiều truy vấn
const client = await pool.connect();
try {
  const res1 = await client.query("SELECT * FROM users");
  const res2 = await client.query("SELECT * FROM orders");
} finally {
  client.release(); // Trả kết nối về pool
}

// Dọn dẹp khi tắt ứng dụng
await pool.end();
```

### 3. Truy vấn Tham số hóa (Ngăn chặn SQL Injection)

```typescript
// LUÔN sử dụng truy vấn tham số hóa - không bao giờ nối chuỗi đầu vào người dùng
const userId = 123;
const email = "user@example.com";

// Một tham số
const result = await pool.query("SELECT * FROM users WHERE id = $1", [userId]);

// Nhiều tham số
const result = await pool.query(
  "INSERT INTO users (email, name, created_at) VALUES ($1, $2, NOW()) RETURNING *",
  [email, "John Doe"],
);

// Tham số mảng
const ids = [1, 2, 3, 4, 5];
const result = await pool.query(
  "SELECT * FROM users WHERE id = ANY($1::int[])",
  [ids],
);
```

### 4. Transactions

```typescript
const client = await pool.connect();

try {
  await client.query("BEGIN");

  const userResult = await client.query(
    "INSERT INTO users (email) VALUES ($1) RETURNING id",
    ["user@example.com"],
  );
  const userId = userResult.rows[0].id;

  await client.query("INSERT INTO orders (user_id, total) VALUES ($1, $2)", [
    userId,
    99.99,
  ]);

  await client.query("COMMIT");
} catch (error) {
  await client.query("ROLLBACK");
  throw error;
} finally {
  client.release();
}
```

### 5. Helper Function cho Transaction

```typescript
async function withTransaction<T>(
  pool: Pool,
  fn: (client: PoolClient) => Promise<T>,
): Promise<T> {
  const client = await pool.connect();
  try {
    await client.query("BEGIN");
    const result = await fn(client);
    await client.query("COMMIT");
    return result;
  } catch (error) {
    await client.query("ROLLBACK");
    throw error;
  } finally {
    client.release();
  }
}

// Sử dụng
const order = await withTransaction(pool, async (client) => {
  const user = await client.query(
    "INSERT INTO users (email) VALUES ($1) RETURNING *",
    ["user@example.com"],
  );
  const order = await client.query(
    "INSERT INTO orders (user_id, total) VALUES ($1, $2) RETURNING *",
    [user.rows[0].id, 99.99],
  );
  return order.rows[0];
});
```

### 6. Truy vấn Có Kiểu với TypeScript

```typescript
import { Pool, QueryResult } from "pg";

interface User {
  id: number;
  email: string;
  name: string;
  created_at: Date;
}

// Định kiểu kết quả truy vấn
const result: QueryResult<User> = await pool.query<User>(
  "SELECT * FROM users WHERE id = $1",
  [userId],
);

const user: User | undefined = result.rows[0];

// Type-safe insert
async function createUser(
  pool: Pool,
  email: string,
  name: string,
): Promise<User> {
  const result = await pool.query<User>(
    "INSERT INTO users (email, name) VALUES ($1, $2) RETURNING *",
    [email, name],
  );
  return result.rows[0];
}
```

## Pool với Làm mới Token Entra ID

Đối với các ứng dụng chạy lâu dài, token sẽ hết hạn và cần làm mới:

```typescript
import { Pool, PoolConfig } from "pg";
import { DefaultAzureCredential, AccessToken } from "@azure/identity";

class AzurePostgresPool {
  private pool: Pool | null = null;
  private credential: DefaultAzureCredential;
  private tokenExpiry: Date | null = null;
  private config: Omit<PoolConfig, "password">;

  constructor(config: Omit<PoolConfig, "password">) {
    this.credential = new DefaultAzureCredential();
    this.config = config;
  }

  private async getToken(): Promise<string> {
    const tokenResponse = await this.credential.getToken(
      "https://ossrdbms-aad.database.windows.net/.default",
    );
    this.tokenExpiry = new Date(tokenResponse.expiresOnTimestamp);
    return tokenResponse.token;
  }

  private isTokenExpired(): boolean {
    if (!this.tokenExpiry) return true;
    // Làm mới trước 5 phút khi hết hạn
    return new Date() >= new Date(this.tokenExpiry.getTime() - 5 * 60 * 1000);
  }

  async getPool(): Promise<Pool> {
    if (this.pool && !this.isTokenExpired()) {
      return this.pool;
    }

    // Đóng pool hiện tại nếu token hết hạn
    if (this.pool) {
      await this.pool.end();
    }

    const token = await this.getToken();
    this.pool = new Pool({
      ...this.config,
      password: token,
    });

    return this.pool;
  }

  async query<T>(text: string, params?: any[]): Promise<QueryResult<T>> {
    const pool = await this.getPool();
    return pool.query<T>(text, params);
  }

  async end(): Promise<void> {
    if (this.pool) {
      await this.pool.end();
      this.pool = null;
    }
  }
}

// Sử dụng
const azurePool = new AzurePostgresPool({
  host: process.env.AZURE_POSTGRESQL_HOST!,
  database: process.env.AZURE_POSTGRESQL_DATABASE!,
  user: process.env.AZURE_POSTGRESQL_USER!,
  port: 5432,
  ssl: { rejectUnauthorized: true },
  max: 20,
});

const result = await azurePool.query("SELECT NOW()");
```

## Xử lý Lỗi

```typescript
import { DatabaseError } from "pg";

try {
  await pool.query("INSERT INTO users (email) VALUES ($1)", [email]);
} catch (error) {
  if (error instanceof DatabaseError) {
    switch (error.code) {
      case "23505": // unique_violation
        console.error("Duplicate entry:", error.detail);
        break;
      case "23503": // foreign_key_violation
        console.error("Foreign key constraint failed:", error.detail);
        break;
      case "42P01": // undefined_table
        console.error("Table does not exist:", error.message);
        break;
      case "28P01": // invalid_password
        console.error("Authentication failed");
        break;
      case "57P03": // cannot_connect_now (server starting)
        console.error("Server unavailable, retry later");
        break;
      default:
        console.error(`PostgreSQL error ${error.code}: ${error.message}`);
    }
  }
  throw error;
}
```

## Định dạng Connection String

```typescript
// Thay thế: Sử dụng connection string
const pool = new Pool({
  connectionString: `postgres://${user}:${password}@${host}:${port}/${database}?sslmode=require`,
});

// Với SSL bắt buộc (Azure)
const connectionString = `postgres://user:password@server.postgres.database.azure.com:5432/mydb?sslmode=require`;
```

## Các Sự kiện Pool

```typescript
const pool = new Pool({
  /* config */
});

pool.on("connect", (client) => {
  console.log("New client connected to pool");
});

pool.on("acquire", (client) => {
  console.log("Client checked out from pool");
});

pool.on("release", (err, client) => {
  console.log("Client returned to pool");
});

pool.on("remove", (client) => {
  console.log("Client removed from pool");
});

pool.on("error", (err, client) => {
  console.error("Unexpected pool error:", err);
});
```

## Cấu hình Đặc thù Azure

| Thiết lập                | Giá trị                                              | Mô tả                          |
| ------------------------ | ---------------------------------------------------- | ------------------------------ |
| `ssl.rejectUnauthorized` | `true`                                               | Luôn sử dụng SSL cho Azure     |
| Default port             | `5432`                                               | Cổng PostgreSQL tiêu chuẩn     |
| PgBouncer port           | `6432`                                               | Sử dụng khi PgBouncer được bật |
| Token scope              | `https://ossrdbms-aad.database.windows.net/.default` | Phạm vi token Entra ID         |
| Token lifetime           | ~1 giờ                                               | Làm mới trước khi hết hạn      |

## Hướng dẫn Định cỡ Pool

| Khối lượng công việc    | `max`  | `idleTimeoutMillis` |
| ----------------------- | ------ | ------------------- |
| Nhẹ (dev/test)          | 5-10   | 30000               |
| Trung bình (production) | 20-30  | 30000               |
| Nặng (concurrent cao)   | 50-100 | 10000               |

> **Lưu ý**: Azure PostgreSQL có giới hạn kết nối dựa trên SKU. Kiểm tra giới hạn kết nối của tầng của bạn.

## Thực hành Tốt nhất

1. **Luôn sử dụng connection pools** cho các ứng dụng production
2. **Sử dụng truy vấn tham số hóa** - Không bao giờ nối chuỗi đầu vào người dùng
3. **Luôn đóng các kết nối** - Sử dụng `try/finally` hoặc connection pools
4. **Bật SSL** - Bắt buộc cho Azure (`ssl: { rejectUnauthorized: true }`)
5. **Xử lý làm mới token** - Token Entra ID hết hạn sau ~1 giờ
6. **Thiết lập connection timeouts** - Tránh treo do các vấn đề mạng
7. **Sử dụng transactions** - Cho các hoạt động nhiều câu lệnh
8. **Giám sát các chỉ số pool** - Theo dõi `pool.totalCount`, `pool.idleCount`, `pool.waitingCount`
9. **Graceful shutdown** - Gọi `pool.end()` khi tắt ứng dụng
10. **Sử dụng TypeScript generics** - Định kiểu kết quả truy vấn của bạn để an toàn

## Các Loại Chính

```typescript
import {
  Client,
  Pool,
  PoolClient,
  PoolConfig,
  QueryResult,
  QueryResultRow,
  DatabaseError,
  QueryConfig,
} from "pg";
```

## Liên kết Tham khảo

| Tài nguyên              | URL                                                                                               |
| ----------------------- | ------------------------------------------------------------------------------------------------- |
| node-postgres Docs      | https://node-postgres.com                                                                         |
| npm Package             | https://www.npmjs.com/package/pg                                                                  |
| GitHub Repository       | https://github.com/brianc/node-postgres                                                           |
| Azure PostgreSQL Docs   | https://learn.microsoft.com/azure/postgresql/flexible-server/                                     |
| Passwordless Connection | https://learn.microsoft.com/azure/postgresql/flexible-server/how-to-connect-with-managed-identity |
