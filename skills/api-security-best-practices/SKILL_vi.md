---
name: api-security-best-practices
description: "Triển khai các mẫu thiết kế API an toàn bao gồm xác thực, phân quyền, xác thực đầu vào, giới hạn tốc độ và bảo vệ chống lại các lỗ hổng API phổ biến"
---

# Thực hành Tốt nhất về Bảo mật API

## Tổng quan

Hướng dẫn các nhà phát triển xây dựng các API an toàn bằng cách triển khai xác thực, phân quyền, xác thực đầu vào, giới hạn tốc độ và bảo vệ chống lại các lỗ hổng phổ biến. Kỹ năng này bao gồm các mẫu bảo mật cho REST, GraphQL và WebSocket API.

## Khi nào Sử dụng Kỹ năng này

- Sử dụng khi thiết kế các điểm cuối API mới
- Sử dụng khi bảo mật các API hiện có
- Sử dụng khi triển khai xác thực và phân quyền
- Sử dụng khi bảo vệ chống lại các cuộc tấn công API (injection, DDoS, v.v.)
- Sử dụng khi thực hiện đánh giá bảo mật API
- Sử dụng khi chuẩn bị cho kiểm toán bảo mật
- Sử dụng khi triển khai giới hạn tốc độ và điều tiết (throttling)
- Sử dụng khi xử lý dữ liệu nhạy cảm trong API

## Cách thức Hoạt động

### Bước 1: Xác thực & Phân quyền

Tôi sẽ giúp bạn triển khai xác thực an toàn:

- Chọn phương pháp xác thực (JWT, OAuth 2.0, API keys)
- Triển khai xác thực dựa trên token
- Thiết lập kiểm soát truy cập dựa trên vai trò (RBAC)
- Quản lý phiên an toàn
- Triển khai xác thực đa yếu tố (MFA)

### Bước 2: Xác thực & Làm sạch Đầu vào

Bảo vệ chống lại các cuộc tấn công tiêm nhiễm:

- Xác thực tất cả dữ liệu đầu vào
- Làm sạch đầu vào của người dùng
- Sử dụng các truy vấn tham số hóa
- Triển khai xác thực lược đồ yêu cầu
- Ngăn chặn SQL injection, XSS và command injection

### Bước 3: Giới hạn Tốc độ & Điều tiết

Ngăn chặn lạm dụng và tấn công DDoS:

- Triển khai giới hạn tốc độ theo người dùng/IP
- Thiết lập điều tiết API
- Cấu hình hạn mức yêu cầu
- Xử lý lỗi giới hạn tốc độ một cách khéo léo
- Giám sát hoạt động đáng ngờ

### Bước 4: Bảo vệ Dữ liệu

Bảo mật dữ liệu nhạy cảm:

- Mã hóa dữ liệu khi truyền (HTTPS/TLS)
- Mã hóa dữ liệu nhạy cảm khi nghỉ (at rest)
- Triển khai xử lý lỗi đúng cách (không rò rỉ dữ liệu)
- Làm sạch thông báo lỗi
- Sử dụng các tiêu đề bảo mật

### Bước 5: Kiểm thử Bảo mật API

Xác minh triển khai bảo mật:

- Kiểm thử xác thực và phân quyền
- Thực hiện kiểm thử thâm nhập
- Kiểm tra các lỗ hổng phổ biến (OWASP API Top 10)
- Xác thực xử lý đầu vào
- Kiểm thử giới hạn tốc độ

## Các ví dụ

### Ví dụ 1: Triển khai Xác thực JWT An toàn

```markdown
## Triển khai Xác thực JWT An toàn

### Luồng Xác thực

1. Người dùng đăng nhập với thông tin đăng nhập
2. Máy chủ xác thực thông tin đăng nhập
3. Máy chủ tạo mã thông báo JWT
4. Máy khách lưu trữ mã thông báo an toàn
5. Máy khách gửi mã thông báo với mỗi yêu cầu
6. Máy chủ xác thực mã thông báo

### Triển khai

#### 1. Tạo Mã thông báo JWT An toàn

\`\`\`javascript
// auth.js
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

// Login endpoint
app.post('/api/auth/login', async (req, res) => {
try {
const { email, password } = req.body;

    // Validate input
    if (!email || !password) {
      return res.status(400).json({
        error: 'Email and password are required'
      });
    }

    // Find user
    const user = await db.user.findUnique({
      where: { email }
    });

    if (!user) {
      // Don't reveal if user exists
      return res.status(401).json({
        error: 'Invalid credentials'
      });
    }

    // Verify password
    const validPassword = await bcrypt.compare(
      password,
      user.passwordHash
    );

    if (!validPassword) {
      return res.status(401).json({
        error: 'Invalid credentials'
      });
    }

    // Generate JWT token
    const token = jwt.sign(
      {
        userId: user.id,
        email: user.email,
        role: user.role
      },
      process.env.JWT_SECRET,
      {
        expiresIn: '1h',
        issuer: 'your-app',
        audience: 'your-app-users'
      }
    );

    // Generate refresh token
    const refreshToken = jwt.sign(
      { userId: user.id },
      process.env.JWT_REFRESH_SECRET,
      { expiresIn: '7d' }
    );

    // Store refresh token in database
    await db.refreshToken.create({
      data: {
        token: refreshToken,
        userId: user.id,
        expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)
      }
    });

    res.json({
      token,
      refreshToken,
      expiresIn: 3600
    });

} catch (error) {
console.error('Login error:', error);
res.status(500).json({
error: 'An error occurred during login'
});
}
});
\`\`\`

#### 2. Xác thực Mã thông báo JWT (Middleware)

\`\`\`javascript
// middleware/auth.js
const jwt = require('jsonwebtoken');

function authenticateToken(req, res, next) {
// Get token from header
const authHeader = req.headers['authorization'];
const token = authHeader && authHeader.split(' ')[1]; // Bearer TOKEN

if (!token) {
return res.status(401).json({
error: 'Access token required'
});
}

// Verify token
jwt.verify(
token,
process.env.JWT_SECRET,
{
issuer: 'your-app',
audience: 'your-app-users'
},
(err, user) => {
if (err) {
if (err.name === 'TokenExpiredError') {
return res.status(401).json({
error: 'Token expired'
});
}
return res.status(403).json({
error: 'Invalid token'
});
}

      // Attach user to request
      req.user = user;
      next();
    }

);
}

module.exports = { authenticateToken };
\`\`\`

#### 3. Bảo vệ Tuyến đường (Routes)

\`\`\`javascript
const { authenticateToken } = require('./middleware/auth');

// Protected route
app.get('/api/user/profile', authenticateToken, async (req, res) => {
try {
const user = await db.user.findUnique({
where: { id: req.user.userId },
select: {
id: true,
email: true,
name: true,
// Don't return passwordHash
}
});

    res.json(user);

} catch (error) {
res.status(500).json({ error: 'Server error' });
}
});
\`\`\`

#### 4. Triển khai Làm mới Token

\`\`\`javascript
app.post('/api/auth/refresh', async (req, res) => {
const { refreshToken } = req.body;

if (!refreshToken) {
return res.status(401).json({
error: 'Refresh token required'
});
}

try {
// Verify refresh token
const decoded = jwt.verify(
refreshToken,
process.env.JWT_REFRESH_SECRET
);

    // Check if refresh token exists in database
    const storedToken = await db.refreshToken.findFirst({
      where: {
        token: refreshToken,
        userId: decoded.userId,
        expiresAt: { gt: new Date() }
      }
    });

    if (!storedToken) {
      return res.status(403).json({
        error: 'Invalid refresh token'
      });
    }

    // Generate new access token
    const user = await db.user.findUnique({
      where: { id: decoded.userId }
    });

    const newToken = jwt.sign(
      {
        userId: user.id,
        email: user.email,
        role: user.role
      },
      process.env.JWT_SECRET,
      { expiresIn: '1h' }
    );

    res.json({
      token: newToken,
      expiresIn: 3600
    });

} catch (error) {
res.status(403).json({
error: 'Invalid refresh token'
});
}
});
\`\`\`

### Thực hành Tốt nhất về Bảo mật

- ✅ Sử dụng bí mật JWT mạnh (tối thiểu 256-bit)
- ✅ Đặt thời gian hết hạn ngắn (1 giờ cho mã thông báo truy cập)
- ✅ Triển khai mã thông báo làm mới cho các phiên dài hạn
- ✅ Lưu trữ mã thông báo làm mới trong cơ sở dữ liệu (có thể bị thu hồi)
- ✅ Chỉ sử dụng HTTPS
- ✅ Không lưu trữ dữ liệu nhạy cảm trong payload JWT
- ✅ Xác thực người phát hành và đối tượng của mã thông báo
- ✅ Triển khai danh sách đen mã thông báo để đăng xuất
```

### Ví dụ 2: Xác thực Đầu vào và Ngăn chặn SQL Injection

```markdown
## Ngăn chặn SQL Injection và Xác thực Đầu vào

### Vấn đề

**❌ Mã Dễ bị tổn thương:**
\`\`\`javascript
// NEVER DO THIS - SQL Injection vulnerability
app.get('/api/users/:id', async (req, res) => {
const userId = req.params.id;

// Dangerous: User input directly in query
const query = \`SELECT \* FROM users WHERE id = '\${userId}'\`;
const user = await db.query(query);

res.json(user);
});

// Attack example:
// GET /api/users/1' OR '1'='1
// Returns all users!
\`\`\`

### Giải pháp

#### 1. Sử dụng Truy vấn Tham số hóa

\`\`\`javascript
// ✅ Safe: Parameterized query
app.get('/api/users/:id', async (req, res) => {
const userId = req.params.id;

// Validate input first
if (!userId || !/^\d+$/.test(userId)) {
return res.status(400).json({
error: 'Invalid user ID'
});
}

// Use parameterized query
const user = await db.query(
'SELECT id, email, name FROM users WHERE id = $1',
[userId]
);

if (!user) {
return res.status(404).json({
error: 'User not found'
});
}

res.json(user);
});
\`\`\`

#### 2. Sử dụng ORM với Escaping Đúng cách

\`\`\`javascript
// ✅ Safe: Using Prisma ORM
app.get('/api/users/:id', async (req, res) => {
const userId = parseInt(req.params.id);

if (isNaN(userId)) {
return res.status(400).json({
error: 'Invalid user ID'
});
}

const user = await prisma.user.findUnique({
where: { id: userId },
select: {
id: true,
email: true,
name: true,
// Don't select sensitive fields
}
});

if (!user) {
return res.status(404).json({
error: 'User not found'
});
}

res.json(user);
});
\`\`\`

#### 3. Triển khai Xác thực Yêu cầu với Zod

\`\`\`javascript
const { z } = require('zod');

// Define validation schema
const createUserSchema = z.object({
email: z.string().email('Invalid email format'),
password: z.string()
.min(8, 'Password must be at least 8 characters')
.regex(/[A-Z]/, 'Password must contain uppercase letter')
.regex(/[a-z]/, 'Password must contain lowercase letter')
.regex(/[0-9]/, 'Password must contain number'),
name: z.string()
.min(2, 'Name must be at least 2 characters')
.max(100, 'Name too long'),
age: z.number()
.int('Age must be an integer')
.min(18, 'Must be 18 or older')
.max(120, 'Invalid age')
.optional()
});

// Validation middleware
function validateRequest(schema) {
return (req, res, next) => {
try {
schema.parse(req.body);
next();
} catch (error) {
res.status(400).json({
error: 'Validation failed',
details: error.errors
});
}
};
}

// Use validation
app.post('/api/users',
validateRequest(createUserSchema),
async (req, res) => {
// Input is validated at this point
const { email, password, name, age } = req.body;

    // Hash password
    const passwordHash = await bcrypt.hash(password, 10);

    // Create user
    const user = await prisma.user.create({
      data: {
        email,
        passwordHash,
        name,
        age
      }
    });

    // Don't return password hash
    const { passwordHash: _, ...userWithoutPassword } = user;
    res.status(201).json(userWithoutPassword);

}
);
\`\`\`

#### 4. Làm sạch Đầu ra để Ngăn chặn XSS

\`\`\`javascript
const DOMPurify = require('isomorphic-dompurify');

app.post('/api/comments', authenticateToken, async (req, res) => {
const { content } = req.body;

// Validate
if (!content || content.length > 1000) {
return res.status(400).json({
error: 'Invalid comment content'
});
}

// Sanitize HTML to prevent XSS
const sanitizedContent = DOMPurify.sanitize(content, {
ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a'],
ALLOWED_ATTR: ['href']
});

const comment = await prisma.comment.create({
data: {
content: sanitizedContent,
userId: req.user.userId
}
});

res.status(201).json(comment);
});
\`\`\`

### Danh sách kiểm tra Xác thực

- [ ] Xác thực tất cả đầu vào của người dùng
- [ ] Sử dụng truy vấn tham số hóa hoặc ORM
- [ ] Xác thực kiểu dữ liệu (chuỗi, số, email, v.v.)
- [ ] Xác thực phạm vi dữ liệu (độ dài tối thiểu/tối đa, phạm vi giá trị)
- [ ] Làm sạch nội dung HTML
- [ ] Thoát các ký tự đặc biệt
- [ ] Xác thực tải lên tệp (loại, kích thước, nội dung)
- [ ] Sử dụng danh sách cho phép (allowlists), không phải danh sách chặn (blocklists)
```

### Ví dụ 3: Giới hạn Tốc độ và Bảo vệ DDoS

```markdown
## Triển khai Giới hạn Tốc độ

### Tại sao Cần Giới hạn Tốc độ?

- Ngăn chặn các cuộc tấn công brute force
- Bảo vệ chống lại DDoS
- Ngăn chặn lạm dụng API
- Đảm bảo sử dụng công bằng
- Giảm chi phí máy chủ

### Triển khai với Express Rate Limit

\`\`\`javascript
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');
const Redis = require('ioredis');

// Create Redis client
const redis = new Redis({
host: process.env.REDIS_HOST,
port: process.env.REDIS_PORT
});

// General API rate limit
const apiLimiter = rateLimit({
store: new RedisStore({
client: redis,
prefix: 'rl:api:'
}),
windowMs: 15 _ 60 _ 1000, // 15 minutes
max: 100, // 100 requests per window
message: {
error: 'Too many requests, please try again later',
retryAfter: 900 // seconds
},
standardHeaders: true, // Return rate limit info in headers
legacyHeaders: false,
// Custom key generator (by user ID or IP)
keyGenerator: (req) => {
return req.user?.userId || req.ip;
}
});

// Strict rate limit for authentication endpoints
const authLimiter = rateLimit({
store: new RedisStore({
client: redis,
prefix: 'rl:auth:'
}),
windowMs: 15 _ 60 _ 1000, // 15 minutes
max: 5, // Only 5 login attempts per 15 minutes
skipSuccessfulRequests: true, // Don't count successful logins
message: {
error: 'Too many login attempts, please try again later',
retryAfter: 900
}
});

// Apply rate limiters
app.use('/api/', apiLimiter);
app.use('/api/auth/login', authLimiter);
app.use('/api/auth/register', authLimiter);

// Custom rate limiter for expensive operations
const expensiveLimiter = rateLimit({
windowMs: 60 _ 60 _ 1000, // 1 hour
max: 10, // 10 requests per hour
message: {
error: 'Rate limit exceeded for this operation'
}
});

app.post('/api/reports/generate',
authenticateToken,
expensiveLimiter,
async (req, res) => {
// Expensive operation
}
);
\`\`\`

### Nâng cao: Giới hạn Tốc độ Theo Người dùng

\`\`\`javascript
// Different limits based on user tier
function createTieredRateLimiter() {
const limits = {
free: { windowMs: 60 _ 60 _ 1000, max: 100 },
pro: { windowMs: 60 _ 60 _ 1000, max: 1000 },
enterprise: { windowMs: 60 _ 60 _ 1000, max: 10000 }
};

return async (req, res, next) => {
const user = req.user;
const tier = user?.tier || 'free';
const limit = limits[tier];

    const key = \`rl:user:\${user.userId}\`;
    const current = await redis.incr(key);

    if (current === 1) {
      await redis.expire(key, limit.windowMs / 1000);
    }

    if (current > limit.max) {
      return res.status(429).json({
        error: 'Rate limit exceeded',
        limit: limit.max,
        remaining: 0,
        reset: await redis.ttl(key)
      });
    }

    // Set rate limit headers
    res.set({
      'X-RateLimit-Limit': limit.max,
      'X-RateLimit-Remaining': limit.max - current,
      'X-RateLimit-Reset': await redis.ttl(key)
    });

    next();

};
}

app.use('/api/', authenticateToken, createTieredRateLimiter());
\`\`\`

### Bảo vệ DDoS với Helmet

\`\`\`javascript
const helmet = require('helmet');

app.use(helmet({
// Content Security Policy
contentSecurityPolicy: {
directives: {
defaultSrc: ["'self'"],
styleSrc: ["'self'", "'unsafe-inline'"],
scriptSrc: ["'self'"],
imgSrc: ["'self'", 'data:', 'https:']
}
},
// Prevent clickjacking
frameguard: { action: 'deny' },
// Hide X-Powered-By header
hidePoweredBy: true,
// Prevent MIME type sniffing
noSniff: true,
// Enable HSTS
hsts: {
maxAge: 31536000,
includeSubDomains: true,
preload: true
}
}));
\`\`\`

### Tiêu đề Phản hồi Giới hạn Tốc độ

\`\`\`
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1640000000
Retry-After: 900
\`\`\`
```

## Thực hành Tốt nhất

### ✅ Nên làm

- **Sử dụng HTTPS mọi nơi** - Không bao giờ gửi dữ liệu nhạy cảm qua HTTP
- **Triển khai Xác thực** - Yêu cầu xác thực cho các điểm cuối được bảo vệ
- **Xác thực Tất cả Đầu vào** - Không bao giờ tin tưởng đầu vào của người dùng
- **Sử dụng Truy vấn Tham số hóa** - Ngăn chặn SQL injection
- **Triển khai Giới hạn Tốc độ** - Bảo vệ chống lại brute force và DDoS
- **Băm Mật khẩu** - Sử dụng bcrypt với salt rounds >= 10
- **Sử dụng Token Ngắn hạn** - Mã thông báo truy cập JWT nên hết hạn nhanh chóng
- **Triển khai CORS Đúng cách** - Chỉ cho phép các nguồn đáng tin cậy
- **Ghi nhật ký Sự kiện Bảo mật** - Giám sát hoạt động đáng ngờ
- **Luôn Cập nhật Phụ thuộc** - Cập nhật thường xuyên các gói
- **Sử dụng Tiêu đề Bảo mật** - Triển khai Helmet.js
- **Làm sạch Thông báo Lỗi** - Không để lộ thông tin nhạy cảm

### ❌ Không nên làm

- **Không Lưu trữ Mật khẩu ở Dạng Văn bản Thuần** - Luôn băm mật khẩu
- **Không Sử dụng Bí mật Yếu** - Sử dụng bí mật JWT mạnh, ngẫu nhiên
- **Không Tin tưởng Đầu vào của Người dùng** - Luôn xác thực và làm sạch
- **Không Để lộ Stack Traces** - Ẩn chi tiết lỗi trong sản xuất
- **Không Sử dụng Nối Chuỗi cho SQL** - Sử dụng truy vấn tham số hóa
- **Không Lưu trữ Dữ liệu Nhạy cảm trong JWT** - JWT không được mã hóa
- **Không Bỏ qua Cập nhật Bảo mật** - Cập nhật phụ thuộc thường xuyên
- **Không Sử dụng Thông tin Đăng nhập Mặc định** - Thay đổi tất cả mật khẩu mặc định
- **Không Tắt CORS Hoàn toàn** - Cấu hình nó đúng cách thay vào đó
- **Không Ghi nhật ký Dữ liệu Nhạy cảm** - Làm sạch nhật ký

## Các Cạm bẫy Phổ biến

### Vấn đề: Bí mật JWT Bị lộ trong Mã

**Triệu chứng:** Bí mật JWT được hardcode hoặc cam kết vào Git
**Giải pháp:**
\`\`\`javascript
// ❌ Bad
const JWT_SECRET = 'my-secret-key';

// ✅ Good
const JWT_SECRET = process.env.JWT_SECRET;
if (!JWT_SECRET) {
throw new Error('JWT_SECRET environment variable is required');
}

// Generate strong secret
// node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
\`\`\`

### Vấn đề: Yêu cầu Mật khẩu Yếu

**Triệu chứng:** Người dùng có thể đặt mật khẩu yếu như "password123"
**Giải pháp:**
\`\`\`javascript
const passwordSchema = z.string()
.min(12, 'Password must be at least 12 characters')
.regex(/[A-Z]/, 'Must contain uppercase letter')
.regex(/[a-z]/, 'Must contain lowercase letter')
.regex(/[0-9]/, 'Must contain number')
.regex(/[^A-Za-z0-9]/, 'Must contain special character');

// Or use a password strength library
const zxcvbn = require('zxcvbn');
const result = zxcvbn(password);
if (result.score < 3) {
return res.status(400).json({
error: 'Password too weak',
suggestions: result.feedback.suggestions
});
}
\`\`\`

### Vấn đề: Thiếu Kiểm tra Phân quyền

**Triệu chứng:** Người dùng có thể truy cập tài nguyên mà họ không nên
**Giải pháp:**
\`\`\`javascript
// ❌ Bad: Only checks authentication
app.delete('/api/posts/:id', authenticateToken, async (req, res) => {
await prisma.post.delete({ where: { id: req.params.id } });
res.json({ success: true });
});

// ✅ Good: Checks both authentication and authorization
app.delete('/api/posts/:id', authenticateToken, async (req, res) => {
const post = await prisma.post.findUnique({
where: { id: req.params.id }
});

if (!post) {
return res.status(404).json({ error: 'Post not found' });
}

// Check if user owns the post or is admin
if (post.userId !== req.user.userId && req.user.role !== 'admin') {
return res.status(403).json({
error: 'Not authorized to delete this post'
});
}

await prisma.post.delete({ where: { id: req.params.id } });
res.json({ success: true });
});
\`\`\`

### Vấn đề: Thông báo Lỗi Dông dài

**Triệu chứng:** Thông báo lỗi tiết lộ chi tiết hệ thống
**Giải pháp:**
\`\`\`javascript
// ❌ Bad: Exposes database details
app.post('/api/users', async (req, res) => {
try {
const user = await prisma.user.create({ data: req.body });
res.json(user);
} catch (error) {
res.status(500).json({ error: error.message });
// Error: "Unique constraint failed on the fields: (`email`)"
}
});

// ✅ Good: Generic error message
app.post('/api/users', async (req, res) => {
try {
const user = await prisma.user.create({ data: req.body });
res.json(user);
} catch (error) {
console.error('User creation error:', error); // Log full error

    if (error.code === 'P2002') {
      return res.status(400).json({
        error: 'Email already exists'
      });
    }

    res.status(500).json({
      error: 'An error occurred while creating user'
    });

}
});
\`\`\`

## Danh sách kiểm tra Bảo mật

### Xác thực & Phân quyền

- [ ] Triển khai xác thực mạnh (JWT, OAuth 2.0)
- [ ] Sử dụng HTTPS cho tất cả các điểm cuối
- [ ] Băm mật khẩu với bcrypt (salt rounds >= 10)
- [ ] Triển khai hết hạn token
- [ ] Thêm cơ chế refresh token
- [ ] Xác minh phân quyền người dùng cho mỗi yêu cầu
- [ ] Triển khai kiểm soát truy cập dựa trên vai trò (RBAC)

### Xác thực Đầu vào

- [ ] Xác thực tất cả đầu vào của người dùng
- [ ] Sử dụng truy vấn tham số hóa hoặc ORM
- [ ] Làm sạch nội dung HTML
- [ ] Xác thực tải lên tệp
- [ ] Triển khai xác thực lược đồ yêu cầu
- [ ] Sử dụng danh sách cho phép (allowlists), không phải danh sách chặn (blocklists)

### Giới hạn Tốc độ & Bảo vệ DDoS

- [ ] Triển khai giới hạn tốc độ theo người dùng/IP
- [ ] Thêm giới hạn nghiêm ngặt hơn cho các điểm cuối xác thực
- [ ] Sử dụng Redis để giới hạn tốc độ phân tán
- [ ] Trả về tiêu đề giới hạn tốc độ hợp lý
- [ ] Triển khai điều tiết (throttling) yêu cầu

### Bảo vệ Dữ liệu

- [ ] Sử dụng HTTPS/TLS cho tất cả lưu lượng
- [ ] Mã hóa dữ liệu nhạy cảm khi nghỉ (at rest)
- [ ] Không lưu trữ dữ liệu nhạy cảm trong JWT
- [ ] Làm sạch thông báo lỗi
- [ ] Triển khai cấu hình CORS đúng cách
- [ ] Sử dụng các tiêu đề bảo mật (Helmet.js)

### Giám sát & Ghi nhật ký

- [ ] Ghi nhật ký các sự kiện bảo mật
- [ ] Giám sát hoạt động đáng ngờ
- [ ] Thiết lập cảnh báo cho các nỗ lực xác thực thất bại
- [ ] Theo dõi các mẫu sử dụng API
- [ ] Không ghi nhật ký dữ liệu nhạy cảm

## Top 10 Bảo mật API của OWASP

1. **Broken Object Level Authorization** - Luôn xác minh người dùng có thể truy cập tài nguyên
2. **Broken Authentication** - Triển khai các cơ chế xác thực mạnh
3. **Broken Object Property Level Authorization** - Xác thực thuộc tính nào người dùng có thể truy cập
4. **Unrestricted Resource Consumption** - Triển khai giới hạn tốc độ và hạn mức
5. **Broken Function Level Authorization** - Xác minh vai trò người dùng cho mỗi chức năng
6. **Unrestricted Access to Sensitive Business Flows** - Bảo vệ các quy trình công việc quan trọng
7. **Server Side Request Forgery (SSRF)** - Xác thực và làm sạch URL
8. **Security Misconfiguration** - Sử dụng các thực hành tốt nhất về bảo mật và tiêu đề
9. **Improper Inventory Management** - Lập tài liệu và bảo mật tất cả các điểm cuối API
10. **Unsafe Consumption of APIs** - Xác thực dữ liệu từ các API bên thứ ba

## Các Kỹ năng Liên quan

- `@ethical-hacking-methodology` - Góc nhìn kiểm thử bảo mật
- `@sql-injection-testing` - Kiểm thử SQL injection
- `@xss-html-injection` - Kiểm thử lỗ hổng XSS
- `@broken-authentication` - Lỗ hổng xác thực
- `@backend-dev-guidelines` - Tiêu chuẩn phát triển Backend
- `@systematic-debugging` - Gỡ lỗi các vấn đề bảo mật

## Tài nguyên Bổ sung

- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [JWT Best Practices](https://tools.ietf.org/html/rfc8725)
- [Express Security Best Practices](https://expressjs.com/en/advanced/best-practice-security.html)
- [Node.js Security Checklist](https://blog.risingstack.com/node-js-security-checklist/)
- [API Security Checklist](https://github.com/shieldfy/API-Security-Checklist)

---

**Mẹo Chuyên nghiệp:** Bảo mật không phải là nhiệm vụ một lần - hãy kiểm toán thường xuyên các API của bạn, giữ các phụ thuộc được cập nhật và cập nhật thông tin về các lỗ hổng mới!
