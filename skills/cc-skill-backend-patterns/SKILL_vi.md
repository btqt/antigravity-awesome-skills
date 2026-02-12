---
name: backend-patterns
description: Các mẫu kiến trúc backend, thiết kế API, tối ưu hóa cơ sở dữ liệu, và các thực tiễn tốt nhất phía máy chủ cho Node.js, Express, và Next.js API routes.
author: affaan-m
version: "1.0"
---

# Các Mẫu Phát triển Backend

Các mẫu kiến trúc backend và thực tiễn tốt nhất cho các ứng dụng phía máy chủ có khả năng mở rộng.

## Mẫu Thiết kế API

### Cấu trúc RESTful API

```typescript
// ✅ URL dựa trên tài nguyên
GET    /api/markets                 # Liệt kê tài nguyên
GET    /api/markets/:id             # Lấy tài nguyên đơn lẻ
POST   /api/markets                 # Tạo tài nguyên
PUT    /api/markets/:id             # Thay thế tài nguyên
PATCH  /api/markets/:id             # Cập nhật tài nguyên
DELETE /api/markets/:id             # Xóa tài nguyên

// ✅ Tham số truy vấn để lọc, sắp xếp, phân trang
GET /api/markets?status=active&sort=volume&limit=20&offset=0
```

### Mẫu Repository

```typescript
// Logic truy cập dữ liệu trừu tượng
interface MarketRepository {
  findAll(filters?: MarketFilters): Promise<Market[]>;
  findById(id: string): Promise<Market | null>;
  create(data: CreateMarketDto): Promise<Market>;
  update(id: string, data: UpdateMarketDto): Promise<Market>;
  delete(id: string): Promise<void>;
}

class SupabaseMarketRepository implements MarketRepository {
  async findAll(filters?: MarketFilters): Promise<Market[]> {
    let query = supabase.from("markets").select("*");

    if (filters?.status) {
      query = query.eq("status", filters.status);
    }

    if (filters?.limit) {
      query = query.limit(filters.limit);
    }

    const { data, error } = await query;

    if (error) throw new Error(error.message);
    return data;
  }

  // Các phương thức khác...
}
```

### Mẫu Lớp Dịch vụ (Service Layer)

```typescript
// Logic nghiệp vụ tách biệt khỏi truy cập dữ liệu
class MarketService {
  constructor(private marketRepo: MarketRepository) {}

  async searchMarkets(query: string, limit: number = 10): Promise<Market[]> {
    // Logic nghiệp vụ
    const embedding = await generateEmbedding(query);
    const results = await this.vectorSearch(embedding, limit);

    // Lấy dữ liệu đầy đủ
    const markets = await this.marketRepo.findByIds(results.map((r) => r.id));

    // Sắp xếp theo độ tương đồng
    return markets.sort((a, b) => {
      const scoreA = results.find((r) => r.id === a.id)?.score || 0;
      const scoreB = results.find((r) => r.id === b.id)?.score || 0;
      return scoreA - scoreB;
    });
  }

  private async vectorSearch(embedding: number[], limit: number) {
    // Triển khai tìm kiếm vector
  }
}
```

### Mẫu Middleware

```typescript
// Pipeline xử lý yêu cầu/phản hồi
export function withAuth(handler: NextApiHandler): NextApiHandler {
  return async (req, res) => {
    const token = req.headers.authorization?.replace("Bearer ", "");

    if (!token) {
      return res.status(401).json({ error: "Unauthorized" });
    }

    try {
      const user = await verifyToken(token);
      req.user = user;
      return handler(req, res);
    } catch (error) {
      return res.status(401).json({ error: "Invalid token" });
    }
  };
}

// Sử dụng
export default withAuth(async (req, res) => {
  // Handler có quyền truy cập vào req.user
});
```

## Mẫu Cơ sở dữ liệu

### Tối ưu hóa Truy vấn

```typescript
// ✅ TỐT: Chỉ chọn các cột cần thiết
const { data } = await supabase
  .from("markets")
  .select("id, name, status, volume")
  .eq("status", "active")
  .order("volume", { ascending: false })
  .limit(10);

// ❌ XẤU: Chọn tất cả mọi thứ
const { data } = await supabase.from("markets").select("*");
```

### Ngăn chặn Truy vấn N+1

```typescript
// ❌ XẤU: Vấn đề truy vấn N+1
const markets = await getMarkets();
for (const market of markets) {
  market.creator = await getUser(market.creator_id); // N truy vấn
}

// ✅ TỐT: Lấy hàng loạt (Batch fetch)
const markets = await getMarkets();
const creatorIds = markets.map((m) => m.creator_id);
const creators = await getUsers(creatorIds); // 1 truy vấn
const creatorMap = new Map(creators.map((c) => [c.id, c]));

markets.forEach((market) => {
  market.creator = creatorMap.get(market.creator_id);
});
```

### Mẫu Giao dịch (Transaction)

```typescript
async function createMarketWithPosition(
  marketData: CreateMarketDto,
  positionData: CreatePositionDto
) {
  // Sử dụng giao dịch Supabase
  const { data, error } = await supabase.rpc('create_market_with_position', {
    market_data: marketData,
    position_data: positionData
  })

  if (error) throw new Error('Transaction failed')
  return data
}

// Hàm SQL trong Supabase
CREATE OR REPLACE FUNCTION create_market_with_position(
  market_data jsonb,
  position_data jsonb
)
RETURNS jsonb
LANGUAGE plpgsql
AS $$
BEGIN
  -- Tự động bắt đầu giao dịch
  INSERT INTO markets VALUES (market_data);
  INSERT INTO positions VALUES (position_data);
  RETURN jsonb_build_object('success', true);
EXCEPTION
  WHEN OTHERS THEN
    -- Rollback xảy ra tự động
    RETURN jsonb_build_object('success', false, 'error', SQLERRM);
END;
$$;
```

## Chiến lược Caching

### Lớp Caching Redis

```typescript
class CachedMarketRepository implements MarketRepository {
  constructor(
    private baseRepo: MarketRepository,
    private redis: RedisClient,
  ) {}

  async findById(id: string): Promise<Market | null> {
    // Kiểm tra cache trước
    const cached = await this.redis.get(`market:${id}`);

    if (cached) {
      return JSON.parse(cached);
    }

    // Cache miss - lấy từ cơ sở dữ liệu
    const market = await this.baseRepo.findById(id);

    if (market) {
      // Cache trong 5 phút
      await this.redis.setex(`market:${id}`, 300, JSON.stringify(market));
    }

    return market;
  }

  async invalidateCache(id: string): Promise<void> {
    await this.redis.del(`market:${id}`);
  }
}
```

### Mẫu Cache-Aside

```typescript
async function getMarketWithCache(id: string): Promise<Market> {
  const cacheKey = `market:${id}`;

  // Thử cache
  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);

  // Cache miss - lấy từ DB
  const market = await db.markets.findUnique({ where: { id } });

  if (!market) throw new Error("Market not found");

  // Cập nhật cache
  await redis.setex(cacheKey, 300, JSON.stringify(market));

  return market;
}
```

## Mẫu Xử lý Lỗi

### Trình xử lý Lỗi Tập trung

```typescript
class ApiError extends Error {
  constructor(
    public statusCode: number,
    public message: string,
    public isOperational = true,
  ) {
    super(message);
    Object.setPrototypeOf(this, ApiError.prototype);
  }
}

export function errorHandler(error: unknown, req: Request): Response {
  if (error instanceof ApiError) {
    return NextResponse.json(
      {
        success: false,
        error: error.message,
      },
      { status: error.statusCode },
    );
  }

  if (error instanceof z.ZodError) {
    return NextResponse.json(
      {
        success: false,
        error: "Validation failed",
        details: error.errors,
      },
      { status: 400 },
    );
  }

  // Ghi nhật ký lỗi không mong muốn
  console.error("Unexpected error:", error);

  return NextResponse.json(
    {
      success: false,
      error: "Internal server error",
    },
    { status: 500 },
  );
}

// Sử dụng
export async function GET(request: Request) {
  try {
    const data = await fetchData();
    return NextResponse.json({ success: true, data });
  } catch (error) {
    return errorHandler(error, request);
  }
}
```

### Thử lại với Backoff Cấp số nhân

```typescript
async function fetchWithRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
): Promise<T> {
  let lastError: Error;

  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;

      if (i < maxRetries - 1) {
        // Backoff cấp số nhân: 1s, 2s, 4s
        const delay = Math.pow(2, i) * 1000;
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }
  }

  throw lastError!;
}

// Sử dụng
const data = await fetchWithRetry(() => fetchFromAPI());
```

## Xác thực & Ủy quyền

### Xác thực JWT Token

```typescript
import jwt from "jsonwebtoken";

interface JWTPayload {
  userId: string;
  email: string;
  role: "admin" | "user";
}

export function verifyToken(token: string): JWTPayload {
  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET!) as JWTPayload;
    return payload;
  } catch (error) {
    throw new ApiError(401, "Invalid token");
  }
}

export async function requireAuth(request: Request) {
  const token = request.headers.get("authorization")?.replace("Bearer ", "");

  if (!token) {
    throw new ApiError(401, "Missing authorization token");
  }

  return verifyToken(token);
}

// Sử dụng trong API route
export async function GET(request: Request) {
  const user = await requireAuth(request);

  const data = await getDataForUser(user.userId);

  return NextResponse.json({ success: true, data });
}
```

### Kiểm soát Truy cập Dựa trên Vai trò (RBAC)

```typescript
type Permission = "read" | "write" | "delete" | "admin";

interface User {
  id: string;
  role: "admin" | "moderator" | "user";
}

const rolePermissions: Record<User["role"], Permission[]> = {
  admin: ["read", "write", "delete", "admin"],
  moderator: ["read", "write", "delete"],
  user: ["read", "write"],
};

export function hasPermission(user: User, permission: Permission): boolean {
  return rolePermissions[user.role].includes(permission);
}

export function requirePermission(permission: Permission) {
  return async (request: Request) => {
    const user = await requireAuth(request);

    if (!hasPermission(user, permission)) {
      throw new ApiError(403, "Insufficient permissions");
    }

    return user;
  };
}

// Sử dụng
export const DELETE = requirePermission("delete")(async (request: Request) => {
  // Handler với kiểm tra quyền
});
```

## Giới hạn Tốc độ (Rate Limiting)

### Bộ giới hạn Tốc độ Trong bộ nhớ Đơn giản

```typescript
class RateLimiter {
  private requests = new Map<string, number[]>();

  async checkLimit(
    identifier: string,
    maxRequests: number,
    windowMs: number,
  ): Promise<boolean> {
    const now = Date.now();
    const requests = this.requests.get(identifier) || [];

    // Loại bỏ các yêu cầu cũ ngoài cửa sổ
    const recentRequests = requests.filter((time) => now - time < windowMs);

    if (recentRequests.length >= maxRequests) {
      return false; // Vượt quá giới hạn tốc độ
    }

    // Thêm yêu cầu hiện tại
    recentRequests.push(now);
    this.requests.set(identifier, recentRequests);

    return true;
  }
}

const limiter = new RateLimiter();

export async function GET(request: Request) {
  const ip = request.headers.get("x-forwarded-for") || "unknown";

  const allowed = await limiter.checkLimit(ip, 100, 60000); // 100 req/min

  if (!allowed) {
    return NextResponse.json(
      {
        error: "Rate limit exceeded",
      },
      { status: 429 },
    );
  }

  // Tiếp tục với yêu cầu
}
```

## Công việc Nền & Hàng đợi

### Mẫu Hàng đợi Đơn giản

```typescript
class JobQueue<T> {
  private queue: T[] = [];
  private processing = false;

  async add(job: T): Promise<void> {
    this.queue.push(job);

    if (!this.processing) {
      this.process();
    }
  }

  private async process(): Promise<void> {
    this.processing = true;

    while (this.queue.length > 0) {
      const job = this.queue.shift()!;

      try {
        await this.execute(job);
      } catch (error) {
        console.error("Job failed:", error);
      }
    }

    this.processing = false;
  }

  private async execute(job: T): Promise<void> {
    // Logic thực thi công việc
  }
}

// Sử dụng để lập chỉ mục thị trường
interface IndexJob {
  marketId: string;
}

const indexQueue = new JobQueue<IndexJob>();

export async function POST(request: Request) {
  const { marketId } = await request.json();

  // Thêm vào hàng đợi thay vì chặn
  await indexQueue.add({ marketId });

  return NextResponse.json({ success: true, message: "Job queued" });
}
```

## Ghi nhật ký & Giám sát

### Ghi nhật ký Có cấu trúc (Structured Logging)

```typescript
interface LogContext {
  userId?: string;
  requestId?: string;
  method?: string;
  path?: string;
  [key: string]: unknown;
}

class Logger {
  log(level: "info" | "warn" | "error", message: string, context?: LogContext) {
    const entry = {
      timestamp: new Date().toISOString(),
      level,
      message,
      ...context,
    };

    console.log(JSON.stringify(entry));
  }

  info(message: string, context?: LogContext) {
    this.log("info", message, context);
  }

  warn(message: string, context?: LogContext) {
    this.log("warn", message, context);
  }

  error(message: string, error: Error, context?: LogContext) {
    this.log("error", message, {
      ...context,
      error: error.message,
      stack: error.stack,
    });
  }
}

const logger = new Logger();

// Sử dụng
export async function GET(request: Request) {
  const requestId = crypto.randomUUID();

  logger.info("Fetching markets", {
    requestId,
    method: "GET",
    path: "/api/markets",
  });

  try {
    const markets = await fetchMarkets();
    return NextResponse.json({ success: true, data: markets });
  } catch (error) {
    logger.error("Failed to fetch markets", error as Error, { requestId });
    return NextResponse.json({ error: "Internal error" }, { status: 500 });
  }
}
```

**Hãy nhớ**: Các mẫu backend cho phép các ứng dụng phía máy chủ có khả năng mở rộng và bảo trì. Chọn các mẫu phù hợp với mức độ phức tạp của bạn.
