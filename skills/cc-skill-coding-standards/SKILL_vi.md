---
name: coding-standards
description: Các tiêu chuẩn mã hóa phổ quát, thực tiễn tốt nhất, và mẫu cho phát triển TypeScript, JavaScript, React, và Node.js.
author: affaan-m
version: "1.0"
---

# Tiêu chuẩn Mã hóa & Thực tiễn Tốt nhất

Các tiêu chuẩn mã hóa phổ quát áp dụng cho tất cả các dự án.

## Nguyên tắc Chất lượng Mã

### 1. Khả năng đọc là trên hết

- Mã được đọc nhiều hơn viết
- Tên biến và tên hàm rõ ràng
- Mã tự tài liệu hóa được ưu tiên hơn bình luận
- Định dạng nhất quán

### 2. KISS (Giữ nó đơn giản, ngốc nghếch)

- Giải pháp đơn giản nhất hoạt động
- Tránh kỹ thuật quá mức
- Không tối ưu hóa sớm
- Dễ hiểu > mã thông minh

### 3. DRY (Đừng lặp lại chính mình)

- Trích xuất logic chung vào các hàm
- Tạo các thành phần có thể tái sử dụng
- Chia sẻ các tiện ích giữa các mô-đun
- Tránh lập trình sao chép-dán

### 4. YAGNI (Bạn sẽ không cần nó)

- Đừng xây dựng các tính năng trước khi chúng cần thiết
- Tránh tính tổng quát đầu cơ
- Chỉ thêm độ phức tạp khi cần thiết
- Bắt đầu đơn giản, tái cấu trúc khi cần

## Tiêu chuẩn TypeScript/JavaScript

### Đặt tên Biến

```typescript
// ✅ TỐT: Tên mô tả
const marketSearchQuery = "election";
const isUserAuthenticated = true;
const totalRevenue = 1000;

// ❌ XẤU: Tên không rõ ràng
const q = "election";
const flag = true;
const x = 1000;
```

### Đặt tên Hàm

```typescript
// ✅ TỐT: Mẫu động từ-danh từ
async function fetchMarketData(marketId: string) {}
function calculateSimilarity(a: number[], b: number[]) {}
function isValidEmail(email: string): boolean {}

// ❌ XẤU: Không rõ ràng hoặc chỉ danh từ
async function market(id: string) {}
function similarity(a, b) {}
function email(e) {}
```

### Mẫu Bất biến (QUAN TRỌNG)

```typescript
// ✅ LUÔN sử dụng toán tử spread
const updatedUser = {
  ...user,
  name: "New Name",
};

const updatedArray = [...items, newItem];

// ❌ KHÔNG BAO GIỜ thay đổi trực tiếp
user.name = "New Name"; // XẤU
items.push(newItem); // XẤU
```

### Xử lý Lỗi

```typescript
// ✅ TỐT: Xử lý lỗi toàn diện
async function fetchData(url: string) {
  try {
    const response = await fetch(url);

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }

    return await response.json();
  } catch (error) {
    console.error("Fetch failed:", error);
    throw new Error("Failed to fetch data");
  }
}

// ❌ XẤU: Không xử lý lỗi
async function fetchData(url) {
  const response = await fetch(url);
  return response.json();
}
```

### Thực tiễn Tốt nhất Async/Await

```typescript
// ✅ TỐT: Thực thi song song khi có thể
const [users, markets, stats] = await Promise.all([
  fetchUsers(),
  fetchMarkets(),
  fetchStats(),
]);

// ❌ XẤU: Tuần tự khi không cần thiết
const users = await fetchUsers();
const markets = await fetchMarkets();
const stats = await fetchStats();
```

### An toàn Kiểu

```typescript
// ✅ TỐT: Các kiểu thích hợp
interface Market {
  id: string;
  name: string;
  status: "active" | "resolved" | "closed";
  created_at: Date;
}

function getMarket(id: string): Promise<Market> {
  // Triển khai
}

// ❌ XẤU: Sử dụng 'any'
function getMarket(id: any): Promise<any> {
  // Triển khai
}
```

## Thực tiễn Tốt nhất React

### Cấu trúc Thành phần

```typescript
// ✅ TỐT: Thành phần chức năng với các kiểu
interface ButtonProps {
  children: React.ReactNode
  onClick: () => void
  disabled?: boolean
  variant?: 'primary' | 'secondary'
}

export function Button({
  children,
  onClick,
  disabled = false,
  variant = 'primary'
}: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {children}
    </button>
  )
}

// ❌ XẤU: Không có kiểu, cấu trúc không rõ ràng
export function Button(props) {
  return <button onClick={props.onClick}>{props.children}</button>
}
```

### Custom Hooks

```typescript
// ✅ TỐT: Custom hook có thể tái sử dụng
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(handler);
  }, [value, delay]);

  return debouncedValue;
}

// Sử dụng
const debouncedQuery = useDebounce(searchQuery, 500);
```

### Quản lý Trạng thái

```typescript
// ✅ TỐT: Cập nhật trạng thái thích hợp
const [count, setCount] = useState(0);

// Cập nhật chức năng cho trạng thái dựa trên trạng thái trước đó
setCount((prev) => prev + 1);

// ❌ XẤU: Tham chiếu trạng thái trực tiếp
setCount(count + 1); // Có thể bị cũ trong các kịch bản không đồng bộ
```

### Kết xuất Có điều kiện

```typescript
// ✅ TỐT: Kết xuất có điều kiện rõ ràng
{isLoading && <Spinner />}
{error && <ErrorMessage error={error} />}
{data && <DataDisplay data={data} />}

// ❌ XẤU: Địa ngục toán tử ba ngôi
{isLoading ? <Spinner /> : error ? <ErrorMessage error={error} /> : data ? <DataDisplay data={data} /> : null}
```

## Tiêu chuẩn Thiết kế API

### Quy ước REST API

```
GET    /api/markets              # Liệt kê tất cả các thị trường
GET    /api/markets/:id          # Lấy thị trường cụ thể
POST   /api/markets              # Tạo thị trường mới
PUT    /api/markets/:id          # Cập nhật thị trường (đầy đủ)
PATCH  /api/markets/:id          # Cập nhật thị trường (một phần)
DELETE /api/markets/:id          # Xóa thị trường

# Tham số truy vấn để lọc
GET /api/markets?status=active&limit=10&offset=0
```

### Định dạng Phản hồi

```typescript
// ✅ TỐT: Cấu trúc phản hồi nhất quán
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
  meta?: {
    total: number;
    page: number;
    limit: number;
  };
}

// Phản hồi thành công
return NextResponse.json({
  success: true,
  data: markets,
  meta: { total: 100, page: 1, limit: 10 },
});

// Phản hồi lỗi
return NextResponse.json(
  {
    success: false,
    error: "Invalid request",
  },
  { status: 400 },
);
```

### Xác thực Đầu vào

```typescript
import { z } from "zod";

// ✅ TỐT: Xác thực lược đồ
const CreateMarketSchema = z.object({
  name: z.string().min(1).max(200),
  description: z.string().min(1).max(2000),
  endDate: z.string().datetime(),
  categories: z.array(z.string()).min(1),
});

export async function POST(request: Request) {
  const body = await request.json();

  try {
    const validated = CreateMarketSchema.parse(body);
    // Tiếp tục với dữ liệu đã xác thực
  } catch (error) {
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
  }
}
```

## Tổ chức Tệp

### Cấu trúc Dự án

```
src/
├── app/                    # Next.js App Router
│   ├── api/               # Các route API
│   ├── markets/           # Các trang thị trường
│   └── (auth)/           # Các trang xác thực (nhóm route)
├── components/            # Các thành phần React
│   ├── ui/               # Các thành phần UI chung
│   ├── forms/            # Các thành phần Form
│   └── layouts/          # Các thành phần Layout
├── hooks/                # Các React hooks tùy chỉnh
├── lib/                  # Các tiện ích và cấu hình
│   ├── api/             # Các client API
│   ├── utils/           # Các hàm hỗ trợ
│   └── constants/       # Các hằng số
├── types/                # Các kiểu TypeScript
└── styles/              # Các kiểu toàn cục
```

### Đặt tên Tệp

```
components/Button.tsx          # PascalCase cho các thành phần
hooks/useAuth.ts              # camelCase với tiền tố 'use'
lib/formatDate.ts             # camelCase cho các tiện ích
types/market.types.ts         # camelCase với hậu tố .types
```

## Bình luận & Tài liệu

### Khi nào Bình luận

```typescript
// ✅ TỐT: Giải thích TẠI SAO, không phải CÁI GÌ
// Sử dụng backoff cấp số nhân để tránh làm quá tải API trong khi ngừng hoạt động
const delay = Math.min(1000 * Math.pow(2, retryCount), 30000);

// Cố tình sử dụng đột biến ở đây để có hiệu suất với các mảng lớn
items.push(newItem);

// ❌ XẤU: Nói điều hiển nhiên
// Tăng bộ đếm lên 1
count++;

// Đặt tên thành tên của người dùng
name = user.name;
```

### JSDoc cho các API Công khai

````typescript
/**
 * Tìm kiếm các thị trường sử dụng độ tương đồng ngữ nghĩa.
 *
 * @param query - Truy vấn tìm kiếm ngôn ngữ tự nhiên
 * @param limit - Số lượng kết quả tối đa (mặc định: 10)
 * @returns Mảng các thị trường được sắp xếp theo điểm tương đồng
 * @throws {Error} Nếu OpenAI API thất bại hoặc Redis không khả dụng
 *
 * @example
 * ```typescript
 * const results = await searchMarkets('election', 5)
 * console.log(results[0].name) // "Trump vs Biden"
 * ```
 */
export async function searchMarkets(
  query: string,
  limit: number = 10,
): Promise<Market[]> {
  // Triển khai
}
````

## Thực tiễn Tốt nhất về Hiệu suất

### Ghi nhớ (Memoization)

```typescript
import { useMemo, useCallback } from "react";

// ✅ TỐT: Ghi nhớ các tính toán đắt tiền
const sortedMarkets = useMemo(() => {
  return markets.sort((a, b) => b.volume - a.volume);
}, [markets]);

// ✅ TỐT: Ghi nhớ các callback
const handleSearch = useCallback((query: string) => {
  setSearchQuery(query);
}, []);
```

### Tải lười (Lazy Loading)

```typescript
import { lazy, Suspense } from 'react'

// ✅ TỐT: Tải lười các thành phần nặng
const HeavyChart = lazy(() => import('./HeavyChart'))

export function Dashboard() {
  return (
    <Suspense fallback={<Spinner />}>
      <HeavyChart />
    </Suspense>
  )
}
```

### Truy vấn Cơ sở dữ liệu

```typescript
// ✅ TỐT: Chỉ chọn các cột cần thiết
const { data } = await supabase
  .from("markets")
  .select("id, name, status")
  .limit(10);

// ❌ XẤU: Chọn tất cả mọi thứ
const { data } = await supabase.from("markets").select("*");
```

## Tiêu chuẩn Kiểm thử

### Cấu trúc Kiểm thử (Mẫu AAA)

```typescript
test("calculates similarity correctly", () => {
  // Arrange (Sắp xếp)
  const vector1 = [1, 0, 0];
  const vector2 = [0, 1, 0];

  // Act (Hành động)
  const similarity = calculateCosineSimilarity(vector1, vector2);

  // Assert (Khẳng định)
  expect(similarity).toBe(0);
});
```

### Đặt tên Kiểm thử

```typescript
// ✅ TỐT: Tên kiểm thử mô tả
test("returns empty array when no markets match query", () => {});
test("throws error when OpenAI API key is missing", () => {});
test("falls back to substring search when Redis unavailable", () => {});

// ❌ XẤU: Tên kiểm thử mơ hồ
test("works", () => {});
test("test search", () => {});
```

## Phát hiện Code Smell

Hãy coi chừng các mẫu chống đối (anti-patterns) này:

### 1. Hàm Dài

```typescript
// ❌ XẤU: Hàm > 50 dòng
function processMarketData() {
  // 100 dòng mã
}

// ✅ TỐT: Chia thành các hàm nhỏ hơn
function processMarketData() {
  const validated = validateData();
  const transformed = transformData(validated);
  return saveData(transformed);
}
```

### 2. Lồng nhau Sâu

```typescript
// ❌ XẤU: 5+ cấp độ lồng nhau
if (user) {
  if (user.isAdmin) {
    if (market) {
      if (market.isActive) {
        if (hasPermission) {
          // Làm gì đó
        }
      }
    }
  }
}

// ✅ TỐT: Trả về sớm
if (!user) return;
if (!user.isAdmin) return;
if (!market) return;
if (!market.isActive) return;
if (!hasPermission) return;

// Làm gì đó
```

### 3. Số Ma thuật

```typescript
// ❌ XẤU: Số không được giải thích
if (retryCount > 3) {
}
setTimeout(callback, 500);

// ✅ TỐT: Hằng số được đặt tên
const MAX_RETRIES = 3;
const DEBOUNCE_DELAY_MS = 500;

if (retryCount > MAX_RETRIES) {
}
setTimeout(callback, DEBOUNCE_DELAY_MS);
```

**Hãy nhớ**: Chất lượng mã là không thể thương lượng. Mã rõ ràng, có thể bảo trì cho phép phát triển nhanh chóng và tái cấu trúc tự tin.
