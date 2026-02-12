---
name: security-review
description: Sử dụng kỹ năng này khi thêm xác thực, xử lý đầu vào của người dùng, làm việc với các bí mật, tạo các endpoint API, hoặc thực hiện tính năng thanh toán/nhạy cảm. Cung cấp danh sách kiểm tra bảo mật toàn diện và các mẫu.
author: affaan-m
version: "1.0"
---

# Kỹ năng Đánh giá Bảo mật

Kỹ năng này đảm bảo tất cả mã tuân theo các thực tiễn tốt nhất về bảo mật và xác định các lỗ hổng tiềm ẩn.

## Khi nào Kích hoạt

- Thực hiện xác thực hoặc ủy quyền
- Xử lý đầu vào của người dùng hoặc tải lên tệp
- Tạo các endpoint API mới
- Làm việc với các bí mật hoặc thông tin đăng nhập
- Thực hiện các tính năng thanh toán
- Lưu trữ hoặc truyền dữ liệu nhạy cảm
- Tích hợp các API của bên thứ ba

## Danh sách kiểm tra Bảo mật

### 1. Quản lý Bí mật

#### ❌ KHÔNG BAO GIỜ Làm điều này

```typescript
const apiKey = "sk-proj-xxxxx"; // Bí mật được mã hóa cứng
const dbPassword = "password123"; // Trong mã nguồn
```

#### ✅ LUÔN Làm điều này

```typescript
const apiKey = process.env.OPENAI_API_KEY;
const dbUrl = process.env.DATABASE_URL;

// Xác minh bí mật tồn tại
if (!apiKey) {
  throw new Error("OPENAI_API_KEY not configured");
}
```

#### Các bước Xác minh

- [ ] Không có khóa API, mã thông báo hoặc mật khẩu được mã hóa cứng
- [ ] Tất cả các bí mật trong các biến môi trường
- [ ] `.env.local` trong .gitignore
- [ ] Không có bí mật trong lịch sử git
- [ ] Bí mật sản xuất trong nền tảng lưu trữ (Vercel, Railway)

### 2. Xác thực Đầu vào

#### Luôn Xác thực Đầu vào Của người dùng

```typescript
import { z } from "zod";

// Xác định lược đồ xác thực
const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150),
});

// Xác thực trước khi xử lý
export async function createUser(input: unknown) {
  try {
    const validated = CreateUserSchema.parse(input);
    return await db.users.create(validated);
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { success: false, errors: error.errors };
    }
    throw error;
  }
}
```

#### Xác thực Tải lên Tệp

```typescript
function validateFileUpload(file: File) {
  // Kiểm tra kích thước (tối đa 5MB)
  const maxSize = 5 * 1024 * 1024;
  if (file.size > maxSize) {
    throw new Error("File too large (max 5MB)");
  }

  // Kiểm tra loại
  const allowedTypes = ["image/jpeg", "image/png", "image/gif"];
  if (!allowedTypes.includes(file.type)) {
    throw new Error("Invalid file type");
  }

  // Kiểm tra phần mở rộng
  const allowedExtensions = [".jpg", ".jpeg", ".png", ".gif"];
  const extension = file.name.toLowerCase().match(/\.[^.]+$/)?.[0];
  if (!extension || !allowedExtensions.includes(extension)) {
    throw new Error("Invalid file extension");
  }

  return true;
}
```

#### Các bước Xác minh

- [ ] Tất cả các đầu vào của người dùng được xác thực với các lược đồ
- [ ] Tải lên tệp bị hạn chế (kích thước, loại, phần mở rộng)
- [ ] Không sử dụng trực tiếp đầu vào của người dùng trong các truy vấn
- [ ] Xác thực danh sách trắng (không phải danh sách đen)
- [ ] Thông báo lỗi không rò rỉ thông tin nhạy cảm

### 3. Ngăn chặn SQL Injection

#### ❌ KHÔNG BAO GIỜ Nối SQL

```typescript
// NGUY HIỂM - Lỗ hổng SQL Injection
const query = `SELECT * FROM users WHERE email = '${userEmail}'`;
await db.query(query);
```

#### ✅ LUÔN Sử dụng Truy vấn Tham số hóa

```typescript
// An toàn - truy vấn tham số hóa
const { data } = await supabase
  .from("users")
  .select("*")
  .eq("email", userEmail);

// Hoặc với SQL thô
await db.query("SELECT * FROM users WHERE email = $1", [userEmail]);
```

#### Các bước Xác minh

- [ ] Tất cả các truy vấn cơ sở dữ liệu sử dụng truy vấn tham số hóa
- [ ] Không nối chuỗi trong SQL
- [ ] ORM/trình xây dựng truy vấn được sử dụng đúng cách
- [ ] Các truy vấn Supabase được làm sạch đúng cách

### 4. Xác thực & Ủy quyền

#### Xử lý JWT Token

```typescript
// ❌ SAI: localStorage (dễ bị XSS)
localStorage.setItem("token", token);

// ✅ ĐÚNG: httpOnly cookies
res.setHeader(
  "Set-Cookie",
  `token=${token}; HttpOnly; Secure; SameSite=Strict; Max-Age=3600`,
);
```

#### Kiểm tra Ủy quyền

```typescript
export async function deleteUser(userId: string, requesterId: string) {
  // LUÔN xác minh ủy quyền trước
  const requester = await db.users.findUnique({
    where: { id: requesterId },
  });

  if (requester.role !== "admin") {
    return NextResponse.json({ error: "Unauthorized" }, { status: 403 });
  }

  // Tiếp tục với việc xóa
  await db.users.delete({ where: { id: userId } });
}
```

#### Row Level Security (Supabase)

```sql
-- Kích hoạt RLS trên tất cả các bảng
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- Người dùng chỉ có thể xem dữ liệu của riêng họ
CREATE POLICY "Users view own data"
  ON users FOR SELECT
  USING (auth.uid() = id);

-- Người dùng chỉ có thể cập nhật dữ liệu của riêng họ
CREATE POLICY "Users update own data"
  ON users FOR UPDATE
  USING (auth.uid() = id);
```

#### Các bước Xác minh

- [ ] Mã thông báo được lưu trữ trong httpOnly cookies (không phải localStorage)
- [ ] Kiểm tra ủy quyền trước các hoạt động nhạy cảm
- [ ] Row Level Security được kích hoạt trong Supabase
- [ ] Kiểm soát truy cập dựa trên vai trò được thực hiện
- [ ] Quản lý phiên an toàn

### 5. Ngăn chặn XSS

#### Làm sạch HTML

```typescript
import DOMPurify from 'isomorphic-dompurify'

// LUÔN làm sạch HTML do người dùng cung cấp
function renderUserContent(html: string) {
  const clean = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p'],
    ALLOWED_ATTR: []
  })
  return <div dangerouslySetInnerHTML={{ __html: clean }} />
}
```

#### Chính sách Bảo mật Nội dung (Content Security Policy)

```typescript
// next.config.js
const securityHeaders = [
  {
    key: "Content-Security-Policy",
    value: `
      default-src 'self';
      script-src 'self' 'unsafe-eval' 'unsafe-inline';
      style-src 'self' 'unsafe-inline';
      img-src 'self' data: https:;
      font-src 'self';
      connect-src 'self' https://api.example.com;
    `
      .replace(/\s{2,}/g, " ")
      .trim(),
  },
];
```

#### Các bước Xác minh

- [ ] HTML do người dùng cung cấp được làm sạch
- [ ] Các tiêu đề CSP được cấu hình
- [ ] Không kết xuất nội dung động không được xác thực
- [ ] Bảo vệ XSS tích hợp của React được sử dụng

### 6. Bảo vệ CSRF

#### CSRF Tokens

```typescript
import { csrf } from "@/lib/csrf";

export async function POST(request: Request) {
  const token = request.headers.get("X-CSRF-Token");

  if (!csrf.verify(token)) {
    return NextResponse.json({ error: "Invalid CSRF token" }, { status: 403 });
  }

  // Xử lý yêu cầu
}
```

#### SameSite Cookies

```typescript
res.setHeader(
  "Set-Cookie",
  `session=${sessionId}; HttpOnly; Secure; SameSite=Strict`,
);
```

#### Các bước Xác minh

- [ ] CSRF tokens trên các hoạt động thay đổi trạng thái
- [ ] SameSite=Strict trên tất cả các cookie
- [ ] Mẫu double-submit cookie được thực hiện

### 7. Giới hạn Tốc độ (Rate Limiting)

#### Giới hạn Tốc độ API

```typescript
import rateLimit from "express-rate-limit";

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 phút
  max: 100, // 100 yêu cầu mỗi cửa sổ
  message: "Too many requests",
});

// Áp dụng cho các route
app.use("/api/", limiter);
```

#### Các hoạt động Đắt tiền

```typescript
// Giới hạn tốc độ tích cực cho tìm kiếm
const searchLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 phút
  max: 10, // 10 yêu cầu mỗi phút
  message: "Too many search requests",
});

app.use("/api/search", searchLimiter);
```

#### Các bước Xác minh

- [ ] Giới hạn tốc độ trên tất cả các endpoint API
- [ ] Giới hạn nghiêm ngặt hơn đối với các hoạt động đắt tiền
- [ ] Giới hạn tốc độ dựa trên IP
- [ ] Giới hạn tốc độ dựa trên người dùng (đã xác thực)

### 8. Phơi nhiễm Dữ liệu Nhạy cảm

#### Ghi nhật ký (Logging)

```typescript
// ❌ SAI: Ghi nhật ký dữ liệu nhạy cảm
console.log("User login:", { email, password });
console.log("Payment:", { cardNumber, cvv });

// ✅ ĐÚNG: Chỉnh sửa dữ liệu nhạy cảm
console.log("User login:", { email, userId });
console.log("Payment:", { last4: card.last4, userId });
```

#### Thông báo Lỗi

```typescript
// ❌ SAI: Phơi bày chi tiết nội bộ
catch (error) {
  return NextResponse.json(
    { error: error.message, stack: error.stack },
    { status: 500 }
  )
}

// ✅ ĐÚNG: Thông báo lỗi chung
catch (error) {
  console.error('Internal error:', error)
  return NextResponse.json(
    { error: 'An error occurred. Please try again.' },
    { status: 500 }
  )
}
```

#### Các bước Xác minh

- [ ] Không có mật khẩu, mã thông báo hoặc bí mật trong nhật ký
- [ ] Thông báo lỗi chung cho người dùng
- [ ] Lỗi chi tiết chỉ trong nhật ký máy chủ
- [ ] Không có dấu vết ngăn xếp (stack traces) được phơi bày cho người dùng

### 9. Bảo mật Blockchain (Solana)

#### Xác minh Ví

```typescript
import { verify } from "@solana/web3.js";

async function verifyWalletOwnership(
  publicKey: string,
  signature: string,
  message: string,
) {
  try {
    const isValid = verify(
      Buffer.from(message),
      Buffer.from(signature, "base64"),
      Buffer.from(publicKey, "base64"),
    );
    return isValid;
  } catch (error) {
    return false;
  }
}
```

#### Xác minh Giao dịch

```typescript
async function verifyTransaction(transaction: Transaction) {
  // Xác minh người nhận
  if (transaction.to !== expectedRecipient) {
    throw new Error("Invalid recipient");
  }

  // Xác minh số tiền
  if (transaction.amount > maxAmount) {
    throw new Error("Amount exceeds limit");
  }

  // Xác minh người dùng có đủ số dư không
  const balance = await getBalance(transaction.from);
  if (balance < transaction.amount) {
    throw new Error("Insufficient balance");
  }

  return true;
}
```

#### Các bước Xác minh

- [ ] Chữ ký ví được xác minh
- [ ] Chi tiết giao dịch được xác thực
- [ ] Kiểm tra số dư trước các giao dịch
- [ ] Không ký mù giao dịch

### 10. Bảo mật Phụ thuộc

#### Cập nhật Thường xuyên

```bash
# Kiểm tra các lỗ hổng
npm audit

# Sửa các vấn đề có thể sửa tự động
npm audit fix

# Cập nhật các phụ thuộc
npm update

# Kiểm tra các gói lỗi thời
npm outdated
```

#### Tệp Khóa (Lock Files)

```bash
# LUÔN cam kết các tệp khóa
git add package-lock.json

# Sử dụng trong CI/CD cho các bản dựng có thể tái tạo
npm ci  # Thay vì npm install
```

#### Các bước Xác minh

- [ ] Các phụ thuộc được cập nhật
- [ ] Không có lỗ hổng đã biết (npm audit clean)
- [ ] Các tệp khóa được cam kết
- [ ] Dependabot được kích hoạt trên GitHub
- [ ] Cập nhật bảo mật thường xuyên

## Kiểm thử Bảo mật

### Kiểm thử Bảo mật Tự động

```typescript
// Kiểm tra xác thực
test("requires authentication", async () => {
  const response = await fetch("/api/protected");
  expect(response.status).toBe(401);
});

// Kiểm tra ủy quyền
test("requires admin role", async () => {
  const response = await fetch("/api/admin", {
    headers: { Authorization: `Bearer ${userToken}` },
  });
  expect(response.status).toBe(403);
});

// Kiểm tra xác thực đầu vào
test("rejects invalid input", async () => {
  const response = await fetch("/api/users", {
    method: "POST",
    body: JSON.stringify({ email: "not-an-email" }),
  });
  expect(response.status).toBe(400);
});

// Kiểm tra giới hạn tốc độ
test("enforces rate limits", async () => {
  const requests = Array(101)
    .fill(null)
    .map(() => fetch("/api/endpoint"));

  const responses = await Promise.all(requests);
  const tooManyRequests = responses.filter((r) => r.status === 429);

  expect(tooManyRequests.length).toBeGreaterThan(0);
});
```

## Danh sách kiểm tra Bảo mật Trước khi Triển khai

Trước BẤT KỲ triển khai sản xuất nào:

- [ ] **Bí mật**: Không có bí mật được mã hóa cứng, tất cả trong env vars
- [ ] **Xác thực Đầu vào**: Tất cả các đầu vào của người dùng được xác thực
- [ ] **SQL Injection**: Tất cả các truy vấn được tham số hóa
- [ ] **XSS**: Nội dung người dùng được làm sạch
- [ ] **CSRF**: Bảo vệ được kích hoạt
- [ ] **Xác thực**: Xử lý mã thông báo đúng cách
- [ ] **Ủy quyền**: Kiểm tra vai trò tại chỗ
- [ ] **Giới hạn Tốc độ**: Được kích hoạt trên tất cả các endpoint
- [ ] **HTTPS**: Bắt buộc trong sản xuất
- [ ] **Tiêu đề Bảo mật**: CSP, X-Frame-Options được cấu hình
- [ ] **Xử lý Lỗi**: Không có dữ liệu nhạy cảm trong lỗi
- [ ] **Ghi nhật ký**: Không có dữ liệu nhạy cảm được ghi lại
- [ ] **Phụ thuộc**: Được cập nhật, không có lỗ hổng
- [ ] **Row Level Security**: Được kích hoạt trong Supabase
- [ ] **CORS**: Được cấu hình đúng cách
- [ ] **Tải lên Tệp**: Được xác thực (kích thước, loại)
- [ ] **Chữ ký Ví**: Được xác minh (nếu blockchain)

## Tài nguyên

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Next.js Security](https://nextjs.org/docs/security)
- [Supabase Security](https://supabase.com/docs/guides/auth)
- [Web Security Academy](https://portswigger.net/web-security)

---

**Hãy nhớ**: Bảo mật không phải là tùy chọn. Một lỗ hổng có thể làm tổn hại toàn bộ nền tảng. Khi nghi ngờ, hãy thận trọng.
