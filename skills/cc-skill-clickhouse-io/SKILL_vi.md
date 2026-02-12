---
name: clickhouse-io
description: Các mẫu ClickHouse database, tối ưu hóa truy vấn, phân tích, và các thực tiễn tốt nhất về kỹ thuật dữ liệu cho khối lượng công việc phân tích hiệu suất cao.
author: affaan-m
version: "1.0"
---

# Các Mẫu Phân tích ClickHouse

Các mẫu cụ thể của ClickHouse cho phân tích hiệu suất cao và kỹ thuật dữ liệu.

## Tổng quan

ClickHouse là một hệ quản trị cơ sở dữ liệu hướng cột (DBMS) cho xử lý phân tích trực tuyến (OLAP). Nó được tối ưu hóa cho các truy vấn phân tích nhanh trên các tập dữ liệu lớn.

**Các Tính năng Chính:**

- Lưu trữ hướng cột
- Nén dữ liệu
- Thực thi truy vấn song song
- Truy vấn phân tán
- Phân tích thời gian thực

## Mẫu Thiết kế Bảng

### MergeTree Engine (Phổ biến nhất)

```sql
CREATE TABLE markets_analytics (
    date Date,
    market_id String,
    market_name String,
    volume UInt64,
    trades UInt32,
    unique_traders UInt32,
    avg_trade_size Float64,
    created_at DateTime
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(date)
ORDER BY (date, market_id)
SETTINGS index_granularity = 8192;
```

### ReplacingMergeTree (Khử trùng lặp)

```sql
-- Đối với dữ liệu có thể có các bản sao (ví dụ: từ nhiều nguồn)
CREATE TABLE user_events (
    event_id String,
    user_id String,
    event_type String,
    timestamp DateTime,
    properties String
) ENGINE = ReplacingMergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (user_id, event_id, timestamp)
PRIMARY KEY (user_id, event_id);
```

### AggregatingMergeTree (Tiền tổng hợp)

```sql
-- Để duy trì các chỉ số được tổng hợp
CREATE TABLE market_stats_hourly (
    hour DateTime,
    market_id String,
    total_volume AggregateFunction(sum, UInt64),
    total_trades AggregateFunction(count, UInt32),
    unique_users AggregateFunction(uniq, String)
) ENGINE = AggregatingMergeTree()
PARTITION BY toYYYYMM(hour)
ORDER BY (hour, market_id);

-- Truy vấn dữ liệu đã tổng hợp
SELECT
    hour,
    market_id,
    sumMerge(total_volume) AS volume,
    countMerge(total_trades) AS trades,
    uniqMerge(unique_users) AS users
FROM market_stats_hourly
WHERE hour >= toStartOfHour(now() - INTERVAL 24 HOUR)
GROUP BY hour, market_id
ORDER BY hour DESC;
```

## Mẫu Tối ưu hóa Truy vấn

### Lọc Hiệu quả

```sql
-- ✅ TỐT: Sử dụng các cột được lập chỉ mục trước
SELECT *
FROM markets_analytics
WHERE date >= '2025-01-01'
  AND market_id = 'market-123'
  AND volume > 1000
ORDER BY date DESC
LIMIT 100;

-- ❌ XẤU: Lọc trên các cột không được lập chỉ mục trước
SELECT *
FROM markets_analytics
WHERE volume > 1000
  AND market_name LIKE '%election%'
  AND date >= '2025-01-01';
```

### Tổng hợp (Aggregations)

```sql
-- ✅ TỐT: Sử dụng các hàm tổng hợp cụ thể của ClickHouse
SELECT
    toStartOfDay(created_at) AS day,
    market_id,
    sum(volume) AS total_volume,
    count() AS total_trades,
    uniq(trader_id) AS unique_traders,
    avg(trade_size) AS avg_size
FROM trades
WHERE created_at >= today() - INTERVAL 7 DAY
GROUP BY day, market_id
ORDER BY day DESC, total_volume DESC;

-- ✅ Sử dụng quantile cho phân vị (hiệu quả hơn percentile)
SELECT
    quantile(0.50)(trade_size) AS median,
    quantile(0.95)(trade_size) AS p95,
    quantile(0.99)(trade_size) AS p99
FROM trades
WHERE created_at >= now() - INTERVAL 1 HOUR;
```

### Các hàm Cửa sổ (Window Functions)

```sql
-- Tính tổng chạy (running totals)
SELECT
    date,
    market_id,
    volume,
    sum(volume) OVER (
        PARTITION BY market_id
        ORDER BY date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_volume
FROM markets_analytics
WHERE date >= today() - INTERVAL 30 DAY
ORDER BY market_id, date;
```

## Mẫu Chèn Dữ liệu

### Chèn Hàng loạt (Được khuyến nghị)

```typescript
import { ClickHouse } from "clickhouse";

const clickhouse = new ClickHouse({
  url: process.env.CLICKHOUSE_URL,
  port: 8123,
  basicAuth: {
    username: process.env.CLICKHOUSE_USER,
    password: process.env.CLICKHOUSE_PASSWORD,
  },
});

// ✅ Chèn hàng loạt (hiệu quả)
async function bulkInsertTrades(trades: Trade[]) {
  const values = trades
    .map(
      (trade) => `(
    '${trade.id}',
    '${trade.market_id}',
    '${trade.user_id}',
    ${trade.amount},
    '${trade.timestamp.toISOString()}'
  )`,
    )
    .join(",");

  await clickhouse
    .query(
      `
    INSERT INTO trades (id, market_id, user_id, amount, timestamp)
    VALUES ${values}
  `,
    )
    .toPromise();
}

// ❌ Chèn từng cá nhân (chậm)
async function insertTrade(trade: Trade) {
  // Đừng làm điều này trong một vòng lặp!
  await clickhouse
    .query(
      `
    INSERT INTO trades VALUES ('${trade.id}', ...)
  `,
    )
    .toPromise();
}
```

### Chèn Dòng (Streaming Insert)

```typescript
// Đối với việc nhập dữ liệu liên tục
import { createWriteStream } from "fs";
import { pipeline } from "stream/promises";

async function streamInserts() {
  const stream = clickhouse.insert("trades").stream();

  for await (const batch of dataSource) {
    stream.write(batch);
  }

  await stream.end();
}
```

## Materialized Views

### Tổng hợp Thời gian thực

```sql
-- Tạo materialized view cho thống kê hàng giờ
CREATE MATERIALIZED VIEW market_stats_hourly_mv
TO market_stats_hourly
AS SELECT
    toStartOfHour(timestamp) AS hour,
    market_id,
    sumState(amount) AS total_volume,
    countState() AS total_trades,
    uniqState(user_id) AS unique_users
FROM trades
GROUP BY hour, market_id;

-- Truy vấn materialized view
SELECT
    hour,
    market_id,
    sumMerge(total_volume) AS volume,
    countMerge(total_trades) AS trades,
    uniqMerge(unique_users) AS users
FROM market_stats_hourly
WHERE hour >= now() - INTERVAL 24 HOUR
GROUP BY hour, market_id;
```

## Giám sát Hiệu suất

### Hiệu suất Truy vấn

```sql
-- Kiểm tra các truy vấn chậm
SELECT
    query_id,
    user,
    query,
    query_duration_ms,
    read_rows,
    read_bytes,
    memory_usage
FROM system.query_log
WHERE type = 'QueryFinish'
  AND query_duration_ms > 1000
  AND event_time >= now() - INTERVAL 1 HOUR
ORDER BY query_duration_ms DESC
LIMIT 10;
```

### Thống kê Bảng

```sql
-- Kiểm tra kích thước bảng
SELECT
    database,
    table,
    formatReadableSize(sum(bytes)) AS size,
    sum(rows) AS rows,
    max(modification_time) AS latest_modification
FROM system.parts
WHERE active
GROUP BY database, table
ORDER BY sum(bytes) DESC;
```

## Các Truy vấn Phân tích Phổ biến

### Phân tích Chuỗi Thời gian

```sql
-- Người dùng hoạt động hàng ngày
SELECT
    toDate(timestamp) AS date,
    uniq(user_id) AS daily_active_users
FROM events
WHERE timestamp >= today() - INTERVAL 30 DAY
GROUP BY date
ORDER BY date;

-- Phân tích duy trì (Retention analysis)
SELECT
    signup_date,
    countIf(days_since_signup = 0) AS day_0,
    countIf(days_since_signup = 1) AS day_1,
    countIf(days_since_signup = 7) AS day_7,
    countIf(days_since_signup = 30) AS day_30
FROM (
    SELECT
        user_id,
        min(toDate(timestamp)) AS signup_date,
        toDate(timestamp) AS activity_date,
        dateDiff('day', signup_date, activity_date) AS days_since_signup
    FROM events
    GROUP BY user_id, activity_date
)
GROUP BY signup_date
ORDER BY signup_date DESC;
```

### Phân tích Phễu

```sql
-- Phễu chuyển đổi
SELECT
    countIf(step = 'viewed_market') AS viewed,
    countIf(step = 'clicked_trade') AS clicked,
    countIf(step = 'completed_trade') AS completed,
    round(clicked / viewed * 100, 2) AS view_to_click_rate,
    round(completed / clicked * 100, 2) AS click_to_completion_rate
FROM (
    SELECT
        user_id,
        session_id,
        event_type AS step
    FROM events
    WHERE event_date = today()
)
GROUP BY session_id;
```

### Phân tích Đoàn hệ (Cohort Analysis)

```sql
-- Đoàn hệ người dùng theo tháng đăng ký
SELECT
    toStartOfMonth(signup_date) AS cohort,
    toStartOfMonth(activity_date) AS month,
    dateDiff('month', cohort, month) AS months_since_signup,
    count(DISTINCT user_id) AS active_users
FROM (
    SELECT
        user_id,
        min(toDate(timestamp)) OVER (PARTITION BY user_id) AS signup_date,
        toDate(timestamp) AS activity_date
    FROM events
)
GROUP BY cohort, month, months_since_signup
ORDER BY cohort, months_since_signup;
```

## Mẫu Đường ống Dữ liệu (Data Pipeline)

### Mẫu ETL

```typescript
// Extract, Transform, Load
async function etlPipeline() {
  // 1. Trích xuất từ nguồn
  const rawData = await extractFromPostgres();

  // 2. Chuyển đổi
  const transformed = rawData.map((row) => ({
    date: new Date(row.created_at).toISOString().split("T")[0],
    market_id: row.market_slug,
    volume: parseFloat(row.total_volume),
    trades: parseInt(row.trade_count),
  }));

  // 3. Tải vào ClickHouse
  await bulkInsertToClickHouse(transformed);
}

// Chạy định kỳ
setInterval(etlPipeline, 60 * 60 * 1000); // Mỗi giờ
```

### Change Data Capture (CDC)

```typescript
// Nghe các thay đổi PostgreSQL và đồng bộ hóa với ClickHouse
import { Client } from "pg";

const pgClient = new Client({ connectionString: process.env.DATABASE_URL });

pgClient.query("LISTEN market_updates");

pgClient.on("notification", async (msg) => {
  const update = JSON.parse(msg.payload);

  await clickhouse.insert("market_updates", [
    {
      market_id: update.id,
      event_type: update.operation, // INSERT, UPDATE, DELETE
      timestamp: new Date(),
      data: JSON.stringify(update.new_data),
    },
  ]);
});
```

## Thực tiễn Tốt nhất

### 1. Chiến lược Phân vùng

- Phân vùng theo thời gian (thường là tháng hoặc ngày)
- Tránh quá nhiều phân vùng (ảnh hưởng hiệu suất)
- Sử dụng loại DATE cho khóa phân vùng

### 2. Khóa Sắp xếp

- Đặt các cột được lọc thường xuyên nhất lên đầu
- Xem xét cardinality (cardinality cao trước)
- Thứ tự ảnh hưởng đến nén

### 3. Loại Dữ liệu

- Sử dụng loại nhỏ nhất thích hợp (UInt32 vs UInt64)
- Sử dụng LowCardinality cho các chuỗi lặp lại
- Sử dụng Enum cho dữ liệu phân loại

### 4. Tránh

- SELECT \* (chỉ định cột)
- FINAL (hợp nhất dữ liệu trước khi truy vấn thay thế)
- Quá nhiều JOINs (phi chuẩn hóa cho phân tích)
- Chèn thường xuyên nhỏ (batch thay thế)

### 5. Giám sát

- Theo dõi hiệu suất truy vấn
- Giám sát sử dụng đĩa
- Kiểm tra các hoạt động hợp nhất
- Xem nhật ký truy vấn chậm

**Hãy nhớ**: ClickHouse vượt trội ở khối lượng công việc phân tích. Thiết kế các bảng cho các mẫu truy vấn của bạn, chèn hàng loạt, và tận dụng materialized views cho các tổng hợp thời gian thực.
