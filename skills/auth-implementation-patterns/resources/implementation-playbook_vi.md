# Sổ tay Triển khai các Mẫu Xác thực và Ủy quyền (Authentication and Authorization Implementation Patterns Implementation Playbook)

Tệp này chứa các mẫu, danh sách kiểm tra và các mẫu mã chi tiết được tham chiếu bởi kỹ năng.

## Các Khái niệm Cốt lõi

### 1. Xác thực so với Ủy quyền

**Xác thực (Authentication - AuthN)**: Bạn là ai?

- Xác minh danh tính (tên người dùng/mật khẩu, OAuth, sinh trắc học)
- Cấp thông tin xác thực (phiên làm việc, token)
- Quản lý đăng nhập/đăng xuất

**Ủy quyền (Authorization - AuthZ)**: Bạn có thể làm gì?

- Kiểm tra quyền hạn
- Kiểm soát truy cập dựa trên vai trò (RBAC)
- Xác thực quyền sở hữu tài nguyên
- Thực thi chính sách

### 2. Chiến lược Xác thực

**Dựa trên Phiên làm việc (Session-Based):**

- Máy chủ lưu trữ trạng thái phiên
- ID phiên nằm trong cookie
- Truyền thống, đơn giản, có trạng thái (stateful)

**Dựa trên Token (JWT):**

- Không trạng thái (stateless), khép kín
- Có thể mở rộng theo chiều ngang
- Có thể lưu trữ các yêu cầu (claims)

**OAuth2/OpenID Connect:**

- Ủy quyền xác thực
- Đăng nhập mạng xã hội (Google, GitHub)
- Đăng nhập một lần (SSO) doanh nghiệp

## Xác thực JWT

### Mẫu 1: Triển khai JWT

```typescript
// Cấu trúc JWT: header.payload.signature
import jwt from "jsonwebtoken";
import { Request, Response, NextFunction } from "express";

interface JWTPayload {
  userId: string;
  email: string;
  role: string;
  iat: number;
  exp: number;
}

// Tạo JWT
function generateTokens(userId: string, email: string, role: string) {
  const accessToken = jwt.sign(
    { userId, email, role },
    process.env.JWT_SECRET!,
    { expiresIn: "15m" }, // Thời gian sống ngắn
  );

  const refreshToken = jwt.sign(
    { userId },
    process.env.JWT_REFRESH_SECRET!,
    { expiresIn: "7d" }, // Thời gian sống dài
  );

  return { accessToken, refreshToken };
}

// Xác minh JWT
function verifyToken(token: string): JWTPayload {
  try {
    return jwt.verify(token, process.env.JWT_SECRET!) as JWTPayload;
  } catch (error) {
    if (error instanceof jwt.TokenExpiredError) {
      throw new Error("Token đã hết hạn");
    }
    if (error instanceof jwt.JsonWebTokenError) {
      throw new Error("Token không hợp lệ");
    }
    throw error;
  }
}

// Middleware
function authenticate(req: Request, res: Response, next: NextFunction) {
  const authHeader = req.headers.authorization;
  if (!authHeader?.startsWith("Bearer ")) {
    return res.status(401).json({ error: "Không có token được cung cấp" });
  }

  const token = authHeader.substring(7);
  try {
    const payload = verifyToken(token);
    req.user = payload; // Đính kèm người dùng vào yêu cầu
    next();
  } catch (error) {
    return res.status(401).json({ error: "Token không hợp lệ" });
  }
}

// Sử dụng
app.get("/api/profile", authenticate, (req, res) => {
  res.json({ user: req.user });
});
```

### Mẫu 2: Luồng Refresh Token

```typescript
interface StoredRefreshToken {
  token: string;
  userId: string;
  expiresAt: Date;
  createdAt: Date;
}

class RefreshTokenService {
  // Lưu trữ refresh token trong cơ sở dữ liệu
  async storeRefreshToken(userId: string, refreshToken: string) {
    const expiresAt = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000);
    await db.refreshTokens.create({
      token: await hash(refreshToken), // Hash trước khi lưu
      userId,
      expiresAt,
    });
  }

  // Làm mới access token
  async refreshAccessToken(refreshToken: string) {
    // Xác minh refresh token
    let payload;
    try {
      payload = jwt.verify(refreshToken, process.env.JWT_REFRESH_SECRET!) as {
        userId: string;
      };
    } catch {
      throw new Error("Refresh token không hợp lệ");
    }

    // Kiểm tra xem token có tồn tại trong cơ sở dữ liệu không
    const storedToken = await db.refreshTokens.findOne({
      where: {
        token: await hash(refreshToken),
        userId: payload.userId,
        expiresAt: { $gt: new Date() },
      },
    });

    if (!storedToken) {
      throw new Error("Refresh token không tìm thấy hoặc đã hết hạn");
    }

    // Lấy người dùng
    const user = await db.users.findById(payload.userId);
    if (!user) {
      throw new Error("Người dùng không tìm thấy");
    }

    // Tạo access token mới
    const accessToken = jwt.sign(
      { userId: user.id, email: user.email, role: user.role },
      process.env.JWT_SECRET!,
      { expiresIn: "15m" },
    );

    return { accessToken };
  }

  // Thu hồi refresh token (đăng xuất)
  async revokeRefreshToken(refreshToken: string) {
    await db.refreshTokens.deleteOne({
      token: await hash(refreshToken),
    });
  }

  // Thu hồi tất cả token của người dùng (đăng xuất khỏi tất cả thiết bị)
  async revokeAllUserTokens(userId: string) {
    await db.refreshTokens.deleteMany({ userId });
  }
}

// Các endpoint API
app.post("/api/auth/refresh", async (req, res) => {
  const { refreshToken } = req.body;
  try {
    const { accessToken } =
      await refreshTokenService.refreshAccessToken(refreshToken);
    res.json({ accessToken });
  } catch (error) {
    res.status(401).json({ error: "Refresh token không hợp lệ" });
  }
});

app.post("/api/auth/logout", authenticate, async (req, res) => {
  const { refreshToken } = req.body;
  await refreshTokenService.revokeRefreshToken(refreshToken);
  res.json({ message: "Đăng xuất thành công" });
});
```

## Xác thực Dựa trên Phiên làm việc (Session-Based)

### Mẫu 1: Express Session

```typescript
import session from "express-session";
import RedisStore from "connect-redis";
import { createClient } from "redis";

// Thiết lập Redis để lưu trữ phiên
const redisClient = createClient({
  url: process.env.REDIS_URL,
});
await redisClient.connect();

app.use(
  session({
    store: new RedisStore({ client: redisClient }),
    secret: process.env.SESSION_SECRET!,
    resave: false,
    saveUninitialized: false,
    cookie: {
      secure: process.env.NODE_ENV === "production", // Chỉ HTTPS
      httpOnly: true, // Không cho phép truy cập JavaScript
      maxAge: 24 * 60 * 60 * 1000, // 24 giờ
      sameSite: "strict", // Bảo vệ CSRF
    },
  }),
);

// Đăng nhập
app.post("/api/auth/login", async (req, res) => {
  const { email, password } = req.body;

  const user = await db.users.findOne({ email });
  if (!user || !(await verifyPassword(password, user.passwordHash))) {
    return res.status(401).json({ error: "Thông tin đăng nhập không hợp lệ" });
  }

  // Lưu trữ người dùng trong phiên
  req.session.userId = user.id;
  req.session.role = user.role;

  res.json({ user: { id: user.id, email: user.email, role: user.role } });
});

// Middleware phiên
function requireAuth(req: Request, res: Response, next: NextFunction) {
  if (!req.session.userId) {
    return res.status(401).json({ error: "Chưa xác thực" });
  }
  next();
}

// Tuyến bảo vệ
app.get("/api/profile", requireAuth, async (req, res) => {
  const user = await db.users.findById(req.session.userId);
  res.json({ user });
});

// Đăng xuất
app.post("/api/auth/logout", (req, res) => {
  req.session.destroy((err) => {
    if (err) {
      return res.status(500).json({ error: "Đăng xuất thất bại" });
    }
    res.clearCookie("connect.sid");
    res.json({ message: "Đăng xuất thành công" });
  });
});
```

## OAuth2 / Đăng nhập Mạng xã hội

### Mẫu 1: OAuth2 với Passport.js

```typescript
import passport from "passport";
import { Strategy as GoogleStrategy } from "passport-google-oauth20";
import { Strategy as GitHubStrategy } from "passport-github2";

// Google OAuth
passport.use(
  new GoogleStrategy(
    {
      clientID: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
      callbackURL: "/api/auth/google/callback",
    },
    async (accessToken, refreshToken, profile, done) => {
      try {
        // Tìm hoặc tạo người dùng
        let user = await db.users.findOne({
          googleId: profile.id,
        });

        if (!user) {
          user = await db.users.create({
            googleId: profile.id,
            email: profile.emails?.[0]?.value,
            name: profile.displayName,
            avatar: profile.photos?.[0]?.value,
          });
        }

        return done(null, user);
      } catch (error) {
        return done(error, undefined);
      }
    },
  ),
);

// Các tuyến đường
app.get(
  "/api/auth/google",
  passport.authenticate("google", {
    scope: ["profile", "email"],
  }),
);

app.get(
  "/api/auth/google/callback",
  passport.authenticate("google", { session: false }),
  (req, res) => {
    // Tạo JWT
    const tokens = generateTokens(req.user.id, req.user.email, req.user.role);
    // Chuyển hướng về frontend với token
    res.redirect(
      `${process.env.FRONTEND_URL}/auth/callback?token=${tokens.accessToken}`,
    );
  },
);
```

## Các Mẫu Ủy quyền (Authorization)

### Mẫu 1: Kiểm soát truy cập dựa trên vai trò (RBAC)

```typescript
enum Role {
  USER = "user",
  MODERATOR = "moderator",
  ADMIN = "admin",
}

const roleHierarchy: Record<Role, Role[]> = {
  [Role.ADMIN]: [Role.ADMIN, Role.MODERATOR, Role.USER],
  [Role.MODERATOR]: [Role.MODERATOR, Role.USER],
  [Role.USER]: [Role.USER],
};

function hasRole(userRole: Role, requiredRole: Role): boolean {
  return roleHierarchy[userRole].includes(requiredRole);
}

// Middleware
function requireRole(...roles: Role[]) {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({ error: "Chưa xác thực" });
    }

    if (!roles.some((role) => hasRole(req.user.role, role))) {
      return res.status(403).json({ error: "Không đủ quyền hạn" });
    }

    next();
  };
}

// Sử dụng
app.delete(
  "/api/users/:id",
  authenticate,
  requireRole(Role.ADMIN),
  async (req, res) => {
    // Chỉ admin mới có thể xóa người dùng
    await db.users.delete(req.params.id);
    res.json({ message: "Người dùng đã bị xóa" });
  },
);
```

### Mẫu 2: Kiểm soát truy cập dựa trên Quyền (Permission-Based Access Control)

```typescript
enum Permission {
  READ_USERS = "read:users",
  WRITE_USERS = "write:users",
  DELETE_USERS = "delete:users",
  READ_POSTS = "read:posts",
  WRITE_POSTS = "write:posts",
}

const rolePermissions: Record<Role, Permission[]> = {
  [Role.USER]: [Permission.READ_POSTS, Permission.WRITE_POSTS],
  [Role.MODERATOR]: [
    Permission.READ_POSTS,
    Permission.WRITE_POSTS,
    Permission.READ_USERS,
  ],
  [Role.ADMIN]: Object.values(Permission),
};

function hasPermission(userRole: Role, permission: Permission): boolean {
  return rolePermissions[userRole]?.includes(permission) ?? false;
}

function requirePermission(...permissions: Permission[]) {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({ error: "Chưa xác thực" });
    }

    const hasAllPermissions = permissions.every((permission) =>
      hasPermission(req.user.role, permission),
    );

    if (!hasAllPermissions) {
      return res.status(403).json({ error: "Không đủ quyền hạn" });
    }

    next();
  };
}

// Sử dụng
app.get(
  "/api/users",
  authenticate,
  requirePermission(Permission.READ_USERS),
  async (req, res) => {
    const users = await db.users.findAll();
    res.json({ users });
  },
);
```

### Mẫu 3: Quyền sở hữu Tài nguyên (Resource Ownership)

```typescript
// Kiểm tra xem người dùng có sở hữu tài nguyên không
async function requireOwnership(
  resourceType: "post" | "comment",
  resourceIdParam: string = "id",
) {
  return async (req: Request, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({ error: "Chưa xác thực" });
    }

    const resourceId = req.params[resourceIdParam];

    // Admin có thể truy cập bất cứ thứ gì
    if (req.user.role === Role.ADMIN) {
      return next();
    }

    // Kiểm tra quyền sở hữu
    let resource;
    if (resourceType === "post") {
      resource = await db.posts.findById(resourceId);
    } else if (resourceType === "comment") {
      resource = await db.comments.findById(resourceId);
    }

    if (!resource) {
      return res.status(404).json({ error: "Không tìm thấy tài nguyên" });
    }

    if (resource.userId !== req.user.userId) {
      return res.status(403).json({ error: "Không được phép" });
    }

    next();
  };
}

// Sử dụng
app.put(
  "/api/posts/:id",
  authenticate,
  requireOwnership("post"),
  async (req, res) => {
    // Người dùng chỉ có thể cập nhật bài viết của chính họ
    const post = await db.posts.update(req.params.id, req.body);
    res.json({ post });
  },
);
```

## Các Thực hành Tốt nhất về Bảo mật

### Mẫu 1: Bảo mật Mật khẩu

```typescript
import bcrypt from "bcrypt";
import { z } from "zod";

// Sơ đồ xác thực mật khẩu
const passwordSchema = z
  .string()
  .min(12, "Mật khẩu phải có ít nhất 12 ký tự")
  .regex(/[A-Z]/, "Mật khẩu phải chứa chữ hoa")
  .regex(/[a-z]/, "Mật khẩu phải chứa chữ thường")
  .regex(/[0-9]/, "Mật khẩu phải chứa con số")
  .regex(/[^A-Za-z0-9]/, "Mật khẩu phải chứa ký tự đặc biệt");

// Hash mật khẩu
async function hashPassword(password: string): Promise<string> {
  const saltRounds = 12; // 2^12 vòng lặp
  return bcrypt.hash(password, saltRounds);
}

// Xác minh mật khẩu
async function verifyPassword(
  password: string,
  hash: string,
): Promise<boolean> {
  return bcrypt.compare(password, hash);
}

// Đăng ký với xác thực mật khẩu
app.post("/api/auth/register", async (req, res) => {
  try {
    const { email, password } = req.body;

    // Xác thực mật khẩu
    passwordSchema.parse(password);

    // Kiểm tra xem người dùng đã tồn tại chưa
    const existingUser = await db.users.findOne({ email });
    if (existingUser) {
      return res.status(400).json({ error: "Email đã được đăng ký" });
    }

    // Hash mật khẩu
    const passwordHash = await hashPassword(password);

    // Tạo người dùng
    const user = await db.users.create({
      email,
      passwordHash,
    });

    // Tạo token
    const tokens = generateTokens(user.id, user.email, user.role);

    res.status(201).json({
      user: { id: user.id, email: user.email },
      ...tokens,
    });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return res.status(400).json({ error: error.errors[0].message });
    }
    res.status(500).json({ error: "Đăng ký thất bại" });
  }
});
```

### Mẫu 2: Giới hạn Tốc độ (Rate Limiting)

```typescript
import rateLimit from "express-rate-limit";
import RedisStore from "rate-limit-redis";

// Trình giới hạn tốc độ đăng nhập
const loginLimiter = rateLimit({
  store: new RedisStore({ client: redisClient }),
  windowMs: 15 * 60 * 1000, // 15 phút
  max: 5, // 5 lần thử
  message: "Quá nhiều lần thử đăng nhập, vui lòng thử lại sau",
  standardHeaders: true,
  legacyHeaders: false,
});

// Trình giới hạn tốc độ API
const apiLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 phút
  max: 100, // 100 yêu cầu mỗi phút
  standardHeaders: true,
});

// Áp dụng cho các tuyến đường
app.post("/api/auth/login", loginLimiter, async (req, res) => {
  // Logic đăng nhập
});

app.use("/api/", apiLimiter);
```

## Các Thực hành Tốt nhất

1. **Không bao giờ Lưu trữ Mật khẩu Dưới dạng Văn bản Thuần**: Luôn hash bằng bcrypt/argon2.
2. **Sử dụng HTTPS**: Mã hóa dữ liệu khi đang truyền tải.
3. **Access Tokens có Thời hạn Ngắn**: Tối đa 15-30 phút.
4. **Cookie An toàn**: Sử dụng các cờ httpOnly, secure, sameSite.
5. **Xác thực Tất cả Đầu vào**: Định dạng email, độ mạnh mật khẩu.
6. **Giới hạn Tốc độ các Endpoint Xác thực**: Ngăn chặn các cuộc tấn công brute force.
7. **Triển khai Bảo vệ CSRF**: Đối với xác thực dựa trên phiên.
8. **Xoay vòng các Bí mật Thường xuyên**: Các bí mật JWT, bí mật phiên.
9. **Ghi nhật ký các Sự kiện Bảo mật**: Các lần thử đăng nhập, xác thực thất bại.
10. **Sử dụng MFA Khi có thể**: Thêm một lớp bảo mật bổ sung.

## Các Bẫy phổ biến (Common Pitfalls)

- **Mật khẩu Yếu**: Thực thi các chính sách mật khẩu mạnh.
- **Lưu JWT trong localStorage**: Dễ bị tấn công XSS, nên sử dụng cookie httpOnly.
- **Token Không có Thời hạn**: Các token nên hết hạn.
- **Chỉ Kiểm tra Xác thực ở Phía Client**: Luôn luôn xác thực ở phía máy chủ.
- **Khôi phục Mật khẩu Không An toàn**: Sử dụng các token an toàn có thời hạn.
- **Không có Giới hạn Tốc độ**: Dễ bị tấn công brute force.
- **Tin tưởng Dữ liệu từ Client**: Luôn xác thực trên máy chủ.

## Tài nguyên

- **references/jwt-best-practices.md**: Hướng dẫn triển khai JWT
- **references/oauth2-flows.md**: Sơ đồ luồng OAuth2 và các ví dụ
- **references/session-security.md**: Quản lý phiên an toàn
- **assets/auth-security-checklist.md**: Danh sách kiểm tra review bảo mật
- **assets/password-policy-template.md**: Mẫu yêu cầu mật khẩu
- **scripts/token-validator.ts**: Tiện ích xác thực JWT
