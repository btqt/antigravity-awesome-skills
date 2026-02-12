---
name: bun-development
description: "Phát triển JavaScript/TypeScript hiện đại với runtime Bun. Bao gồm quản lý gói, đóng gói (bundling), kiểm thử, và di chuyển từ Node.js. Sử dụng khi làm việc với Bun, tối ưu hóa tốc độ phát triển JS/TS, hoặc di chuyển từ Node.js sang Bun."
---

# ⚡ Phát triển Bun

> Phát triển JavaScript/TypeScript nhanh, hiện đại với runtime Bun, lấy cảm hứng từ [oven-sh/bun](https://github.com/oven-sh/bun).

## Khi nào Sử dụng Kỹ năng Này

Sử dụng kỹ năng này khi:

- Bắt đầu các dự án JS/TS mới với Bun
- Di chuyển từ Node.js sang Bun
- Tối ưu hóa tốc độ phát triển
- Sử dụng các công cụ tích hợp của Bun (bundler, test runner)
- Khắc phục các sự cố cụ thể của Bun

---

## 1. Bắt đầu

### 1.1 Cài đặt

```bash
# macOS / Linux
curl -fsSL https://bun.sh/install | bash

# Windows
powershell -c "irm bun.sh/install.ps1 | iex"

# Homebrew
brew tap oven-sh/bun
brew install bun

# npm (nếu cần)
npm install -g bun

# Nâng cấp
bun upgrade
```

### 1.2 Tại sao là Bun?

| Tính năng                | Bun               | Node.js                               |
| :----------------------- | :---------------- | :------------------------------------ |
| Thời gian khởi động      | ~25ms             | ~100ms+                               |
| Cài đặt gói              | nhanh hơn 10-100x | Cơ bản                                |
| TypeScript               | Gốc (Native)      | Yêu cầu trình chuyển đổi (transpiler) |
| JSX                      | Gốc (Native)      | Yêu cầu trình chuyển đổi (transpiler) |
| Trình chạy kiểm thử      | Tích hợp sẵn      | Bên ngoài (Jest, Vitest)              |
| Trình đóng gói (Bundler) | Tích hợp sẵn      | Bên ngoài (Webpack, esbuild)          |

---

## 2. Thiết lập Dự án

### 2.1 Tạo Dự án Mới

```bash
# Khởi tạo dự án
bun init

# Tạo ra:
# ├── package.json
# ├── tsconfig.json
# ├── index.ts
# └── README.md

# Với template cụ thể
bun create <template> <project-name>

# Ví dụ
bun create react my-app        # Ứng dụng React
bun create next my-app         # Ứng dụng Next.js
bun create vite my-app         # Ứng dụng Vite
bun create elysia my-api       # Elysia API
```

### 2.2 package.json

```json
{
  "name": "my-bun-project",
  "version": "1.0.0",
  "module": "index.ts",
  "type": "module",
  "scripts": {
    "dev": "bun run --watch index.ts",
    "start": "bun run index.ts",
    "test": "bun test",
    "build": "bun build ./index.ts --outdir ./dist",
    "lint": "bunx eslint ."
  },
  "devDependencies": {
    "@types/bun": "latest"
  },
  "peerDependencies": {
    "typescript": "^5.0.0"
  }
}
```

### 2.3 tsconfig.json (Tối ưu hóa cho Bun)

```json
{
  "compilerOptions": {
    "lib": ["ESNext"],
    "module": "esnext",
    "target": "esnext",
    "moduleResolution": "bundler",
    "moduleDetection": "force",
    "allowImportingTsExtensions": true,
    "noEmit": true,
    "composite": true,
    "strict": true,
    "downlevelIteration": true,
    "skipLibCheck": true,
    "jsx": "react-jsx",
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    "allowJs": true,
    "types": ["bun-types"]
  }
}
```

---

## 3. Quản lý Gói

### 3.1 Cài đặt Gói

```bash
# Cài đặt từ package.json
bun install              # hoặc 'bun i'

# Thêm phụ thuộc
bun add express          # Phụ thuộc thông thường
bun add -d typescript    # Phụ thuộc phát triển (Dev dependency)
bun add -D @types/node   # Phụ thuộc phát triển (alias)
bun add --optional pkg   # Phụ thuộc tùy chọn

# Từ registry cụ thể
bun add lodash --registry https://registry.npmmirror.com

# Cài đặt phiên bản cụ thể
bun add react@18.2.0
bun add react@latest
bun add react@next

# Từ git
bun add github:user/repo
bun add git+https://github.com/user/repo.git
```

### 3.2 Xóa & Cập nhật

```bash
# Xóa gói
bun remove lodash

# Cập nhật gói
bun update              # Cập nhật tất cả
bun update lodash       # Cập nhật cụ thể
bun update --latest     # Cập nhật lên mới nhất (bỏ qua phạm vi)

# Kiểm tra lỗi thời
bun outdated
```

### 3.3 bunx (tương đương npx)

```bash
# Thực thi các tệp nhị phân của gói
bunx prettier --write .
bunx tsc --init
bunx create-react-app my-app

# Với phiên bản cụ thể
bunx -p typescript@4.9 tsc --version

# Chạy mà không cần cài đặt
bunx cowsay "Hello from Bun!"
```

### 3.4 Lockfile

```bash
# bun.lockb là một lockfile nhị phân (phân tích nhanh hơn)
# Để tạo lockfile văn bản để gỡ lỗi:
bun install --yarn    # Tạo yarn.lock

# Tin tưởng lockfile hiện có
bun install --frozen-lockfile
```

---

## 4. Chạy Mã

### 4.1 Thực thi Cơ bản

```bash
# Chạy TypeScript trực tiếp (không cần bước build!)
bun run index.ts

# Chạy JavaScript
bun run index.js

# Chạy với tham số
bun run server.ts --port 3000

# Chạy script package.json
bun run dev
bun run build

# Dạng ngắn (cho scripts)
bun dev
bun build
```

### 4.2 Chế độ Watch

```bash
# Tự động khởi động lại khi tệp thay đổi
bun --watch run index.ts

# Với hot reloading
bun --hot run server.ts
```

### 4.3 Biến Môi trường

```typescript
// Tệp .env được tải tự động!

// Truy cập biến môi trường
const apiKey = Bun.env.API_KEY;
const port = Bun.env.PORT ?? "3000";

// Hoặc sử dụng process.env (tương thích Node.js)
const dbUrl = process.env.DATABASE_URL;
```

```bash
# Chạy với tệp env cụ thể
bun --env-file=.env.production run index.ts
```

---

## 5. API Tích hợp sẵn

### 5.1 Hệ thống Tệp (Bun.file)

```typescript
// Đọc tệp
const file = Bun.file("./data.json");
const text = await file.text();
const json = await file.json();
const buffer = await file.arrayBuffer();

// Thông tin tệp
console.log(file.size); // bytes
console.log(file.type); // MIME type

// Ghi tệp
await Bun.write("./output.txt", "Hello, Bun!");
await Bun.write("./data.json", JSON.stringify({ foo: "bar" }));

// Stream các tệp lớn
const reader = file.stream();
for await (const chunk of reader) {
  console.log(chunk);
}
```

### 5.2 Máy chủ HTTP (Bun.serve)

```typescript
const server = Bun.serve({
  port: 3000,

  fetch(request) {
    const url = new URL(request.url);

    if (url.pathname === "/") {
      return new Response("Hello World!");
    }

    if (url.pathname === "/api/users") {
      return Response.json([
        { id: 1, name: "Alice" },
        { id: 2, name: "Bob" },
      ]);
    }

    return new Response("Not Found", { status: 404 });
  },

  error(error) {
    return new Response(`Error: ${error.message}`, { status: 500 });
  },
});

console.log(`Server running at http://localhost:${server.port}`);
```

### 5.3 Máy chủ WebSocket

```typescript
const server = Bun.serve({
  port: 3000,

  fetch(req, server) {
    // Nâng cấp lên WebSocket
    if (server.upgrade(req)) {
      return; // Đã nâng cấp
    }
    return new Response("Upgrade failed", { status: 500 });
  },

  websocket: {
    open(ws) {
      console.log("Client connected");
      ws.send("Welcome!");
    },

    message(ws, message) {
      console.log(`Received: ${message}`);
      ws.send(`Echo: ${message}`);
    },

    close(ws) {
      console.log("Client disconnected");
    },
  },
});
```

### 5.4 SQLite (Bun.sql)

```typescript
import { Database } from "bun:sqlite";

const db = new Database("mydb.sqlite");

// Tạo bảng
db.run(`
  CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    email TEXT UNIQUE
  )
`);

// Chèn
const insert = db.prepare("INSERT INTO users (name, email) VALUES (?, ?)");
insert.run("Alice", "alice@example.com");

// Truy vấn
const query = db.prepare("SELECT * FROM users WHERE name = ?");
const user = query.get("Alice");
console.log(user); // { id: 1, name: "Alice", email: "alice@example.com" }

// Truy vấn tất cả
const allUsers = db.query("SELECT * FROM users").all();
```

### 5.5 Băm Mật khẩu

```typescript
// Băm mật khẩu
const password = "super-secret";
const hash = await Bun.password.hash(password);

// Xác minh mật khẩu
const isValid = await Bun.password.verify(password, hash);
console.log(isValid); // true

// Với các tùy chọn thuật toán
const bcryptHash = await Bun.password.hash(password, {
  algorithm: "bcrypt",
  cost: 12,
});
```

---

## 6. Kiểm thử

### 6.1 Kiểm thử Cơ bản

```typescript
// math.test.ts
import { describe, it, expect, beforeAll, afterAll } from "bun:test";

describe("Math operations", () => {
  it("adds two numbers", () => {
    expect(1 + 1).toBe(2);
  });

  it("subtracts two numbers", () => {
    expect(5 - 3).toBe(2);
  });
});
```

### 6.2 Chạy Kiểm thử

```bash
# Chạy tất cả các bài kiểm tra
bun test

# Chạy tệp cụ thể
bun test math.test.ts

# Chạy mẫu khớp
bun test --grep "adds"

# Chế độ Watch
bun test --watch

# Với coverage
bun test --coverage

# Timeout
bun test --timeout 5000
```

### 6.3 Matchers

```typescript
import { expect, test } from "bun:test";

test("matchers", () => {
  // Equality
  expect(1).toBe(1);
  expect({ a: 1 }).toEqual({ a: 1 });
  expect([1, 2]).toContain(1);

  // Comparisons
  expect(10).toBeGreaterThan(5);
  expect(5).toBeLessThanOrEqual(5);

  // Truthiness
  expect(true).toBeTruthy();
  expect(null).toBeNull();
  expect(undefined).toBeUndefined();

  // Strings
  expect("hello").toMatch(/ell/);
  expect("hello").toContain("ell");

  // Arrays
  expect([1, 2, 3]).toHaveLength(3);

  // Exceptions
  expect(() => {
    throw new Error("fail");
  }).toThrow("fail");

  // Async
  await expect(Promise.resolve(1)).resolves.toBe(1);
  await expect(Promise.reject("err")).rejects.toBe("err");
});
```

### 6.4 Mocking

```typescript
import { mock, spyOn } from "bun:test";

// Mock function
const mockFn = mock((x: number) => x * 2);
mockFn(5);
expect(mockFn).toHaveBeenCalled();
expect(mockFn).toHaveBeenCalledWith(5);
expect(mockFn.mock.results[0].value).toBe(10);

// Spy on method
const obj = {
  method: () => "original",
};
const spy = spyOn(obj, "method").mockReturnValue("mocked");
expect(obj.method()).toBe("mocked");
expect(spy).toHaveBeenCalled();
```

---

## 7. Đóng gói (Bundling)

### 7.1 Build Cơ bản

```bash
# Bundle cho production
bun build ./src/index.ts --outdir ./dist

# Với các tùy chọn
bun build ./src/index.ts \
  --outdir ./dist \
  --target browser \
  --minify \
  --sourcemap
```

### 7.2 Build API

```typescript
const result = await Bun.build({
  entrypoints: ["./src/index.ts"],
  outdir: "./dist",
  target: "browser", // hoặc "bun", "node"
  minify: true,
  sourcemap: "external",
  splitting: true,
  format: "esm",

  // Các gói bên ngoài (không được đóng gói cùng)
  external: ["react", "react-dom"],

  // Định nghĩa globals
  define: {
    "process.env.NODE_ENV": JSON.stringify("production"),
  },

  // Đặt tên
  naming: {
    entry: "[name].[hash].js",
    chunk: "chunks/[name].[hash].js",
    asset: "assets/[name].[hash][ext]",
  },
});

if (!result.success) {
  console.error(result.logs);
}
```

### 7.3 Biên dịch thành Tệp thực thi

```bash
# Tạo tệp thực thi độc lập
bun build ./src/cli.ts --compile --outfile myapp

# Cross-compile
bun build ./src/cli.ts --compile --target=bun-linux-x64 --outfile myapp-linux
bun build ./src/cli.ts --compile --target=bun-darwin-arm64 --outfile myapp-mac

# Với assets nhúng
bun build ./src/cli.ts --compile --outfile myapp --embed ./assets
```

---

## 8. Di chuyển từ Node.js

### 8.1 Khả năng Tương thích

```typescript
// Hầu hết các API Node.js hoạt động ngay lập tức
import fs from "fs";
import path from "path";
import crypto from "crypto";

// process là toàn cục
console.log(process.cwd());
console.log(process.env.HOME);

// Buffer là toàn cục
const buf = Buffer.from("hello");

// __dirname và __filename hoạt động
console.log(__dirname);
console.log(__filename);
```

### 8.2 Các Bước Di chuyển Phổ biến

```bash
# 1. Cài đặt Bun
curl -fsSL https://bun.sh/install | bash

# 2. Thay thế trình quản lý gói
rm -rf node_modules package-lock.json
bun install

# 3. Cập nhật scripts trong package.json
# "start": "node index.js" → "start": "bun run index.ts"
# "test": "jest" → "test": "bun test"

# 4. Thêm Bun types
bun add -d @types/bun
```

### 8.3 Khác biệt so với Node.js

```typescript
// ❌ Cụ thể của Node.js (có thể không hoạt động)
require("module")             // Sử dụng import thay thế
require.resolve("pkg")        // Sử dụng import.meta.resolve
__non_webpack_require__       // Không được hỗ trợ

// ✅ Tương đương trong Bun
import pkg from "pkg";
const resolved = import.meta.resolve("pkg");
Bun.resolveSync("pkg", process.cwd());

// ❌ Các biến toàn cục này khác biệt
process.hrtime()              // Sử dụng Bun.nanoseconds()
setImmediate()                // Sử dụng queueMicrotask()

// ✅ Các tính năng cụ thể của Bun
const file = Bun.file("./data.txt");  // API tệp nhanh
Bun.serve({ port: 3000, fetch: ... }); // Máy chủ HTTP nhanh
Bun.password.hash(password);           // Băm tích hợp sẵn
```

---

## 9. Mẹo Hiệu suất

### 9.1 Sử dụng API gốc của Bun

```typescript
// Chậm (Tương thích Node.js)
import fs from "fs/promises";
const content = await fs.readFile("./data.txt", "utf-8");

// Nhanh (Gốc Bun)
const file = Bun.file("./data.txt");
const content = await file.text();
```

### 9.2 Sử dụng Bun.serve cho HTTP

```typescript
// Không nên: Express/Fastify (overhead)
import express from "express";
const app = express();

// Nên: Bun.serve (gốc, nhanh hơn 4-10x)
Bun.serve({
  fetch(req) {
    return new Response("Hello!");
  },
});

// Hoặc sử dụng Elysia (Framework tối ưu cho Bun)
import { Elysia } from "elysia";
new Elysia().get("/", () => "Hello!").listen(3000);
```

### 9.3 Đóng gói cho Production

```bash
# Luôn đóng gói và minify cho production
bun build ./src/index.ts --outdir ./dist --minify --target node

# Sau đó chạy gói
bun run ./dist/index.js
```

---

## Tham khảo Nhanh

| Tác vụ         | Lệnh                                       |
| :------------- | :----------------------------------------- |
| Khởi tạo dự án | `bun init`                                 |
| Cài đặt deps   | `bun install`                              |
| Thêm gói       | `bun add <pkg>`                            |
| Chạy script    | `bun run <script>`                         |
| Chạy tệp       | `bun run file.ts`                          |
| Chế độ Watch   | `bun --watch run file.ts`                  |
| Chạy tests     | `bun test`                                 |
| Build          | `bun build ./src/index.ts --outdir ./dist` |
| Thực thi pkg   | `bunx <pkg>`                               |

---

## Tài nguyên

- [Tài liệu Bun](https://bun.sh/docs)
- [Bun GitHub](https://github.com/oven-sh/bun)
- [Elysia Framework](https://elysiajs.com/)
- [Bun Discord](https://bun.sh/discord)
