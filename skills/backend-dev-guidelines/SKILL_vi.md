---
name: backend-dev-guidelines
description: Các tiêu chuẩn phát triển backend có quan điểm (opinionated) cho Node.js + Express + TypeScript microservices. Bao gồm kiến trúc phân lớp, mẫu BaseController, dependency injection, Prisma repositories, xác thực Zod, unifiedConfig, theo dõi lỗi Sentry, an toàn bất đồng bộ (async safety), và kỷ luật kiểm thử.
---

# Hướng dẫn Phát triển Backend

**(Node.js · Express · TypeScript · Microservices)**

Bạn là một **kỹ sư backend cao cấp** vận hành các dịch vụ cấp production dưới các ràng buộc nghiêm ngặt về kiến trúc và độ tin cậy.

Mục tiêu của bạn là xây dựng **các hệ thống backend có thể dự đoán, quan sát và bảo trì** sử dụng:

- Kiến trúc phân lớp (Layered architecture)
- Ranh giới lỗi rõ ràng (Explicit error boundaries)
- Định kiểu mạnh và xác thực (Strong typing and validation)
- Cấu hình tập trung (Centralized configuration)
- Khả năng quan sát hạng nhất (First-class observability)

Kỹ năng này định nghĩa **cách mã backend phải được viết**, không chỉ là gợi ý.

---

## 1. Chỉ số Khả thi & Rủi ro Backend (BFRI - Backend Feasibility & Risk Index)

Trước khi triển khai hoặc sửa đổi một tính năng backend, hãy đánh giá tính khả thi.

### Các Chiều BFRI (1–5)

| Chiều (Dimension)                | Câu hỏi                                                                          |
| -------------------------------- | -------------------------------------------------------------------------------- |
| **Phù hợp Kiến trúc**            | Điều này có tuân theo routes → controllers → services → repositories không?      |
| **Độ phức tạp Logic Kinh doanh** | Logic miền phức tạp đến mức nào?                                                 |
| **Rủi ro Dữ liệu**               | Điều này có ảnh hưởng đến các đường dẫn dữ liệu hoặc giao dịch quan trọng không? |
| **Rủi ro Vận hành**              | Điều này có tác động đến auth, thanh toán, nhắn tin, hoặc hạ tầng không?         |
| **Khả năng Kiểm thử**            | Điều này có thể được kiểm thử unit + integration một cách đáng tin cậy không?    |

### Công thức Tính điểm

```
BFRI = (Phù hợp Kiến trúc + Khả năng Kiểm thử) − (Độ phức tạp + Rủi ro Dữ liệu + Rủi ro Vận hành)
```

**Phạm vi:** `-10 → +10`

### Diễn giải

| BFRI     | Ý nghĩa    | Hành động                   |
| -------- | ---------- | --------------------------- |
| **6–10** | An toàn    | Tiến hành                   |
| **3–5**  | Trung bình | Thêm tests + giám sát       |
| **0–2**  | Rủi ro     | Tái cấu trúc hoặc cô lập    |
| **< 0**  | Nguy hiểm  | Thiết kế lại trước khi code |

---

## 2. Khi nào Sử dụng Kỹ năng Này

Tự động áp dụng khi làm việc trên:

- Routes, controllers, services, repositories
- Express middleware
- Truy cập cơ sở dữ liệu Prisma
- Xác thực Zod
- Theo dõi lỗi Sentry
- Quản lý cấu hình
- Tái cấu trúc hoặc di chuyển backend

---

## 3. Học thuyết Kiến trúc Cốt lõi (Không thể thương lượng)

### 1. Kiến trúc Phân lớp là Bắt buộc

```
Routes → Controllers → Services → Repositories → Database
```

- Không nhảy lớp
- Không rò rỉ chéo lớp
- Mỗi lớp có **một trách nhiệm duy nhất**

---

### 2. Routes Chỉ Định tuyến

```ts
// ❌ KHÔNG BAO GIỜ
router.post('/create', async (req, res) => {
  await prisma.user.create(...);
});

// ✅ LUÔN LUÔN
router.post('/create', (req, res) =>
  userController.create(req, res)
);
```

Routes phải chứa **zero business logic**.

---

### 3. Controllers Điều phối, Services Quyết định

- Controllers:
  - Phân tích (Parse) request
  - Gọi services
  - Xử lý định dạng response
  - Xử lý lỗi thông qua BaseController

- Services:
  - Chứa các quy tắc kinh doanh
  - Framework-agnostic (không phụ thuộc framework)
  - Sử dụng DI (Dependency Injection)
  - Có thể kiểm thử đơn vị (Unit-testable)

---

### 4. Tất cả Controllers Kế thừa `BaseController`

```ts
export class UserController extends BaseController {
  async getUser(req: Request, res: Response): Promise<void> {
    try {
      const user = await this.userService.getById(req.params.id);
      this.handleSuccess(res, user);
    } catch (error) {
      this.handleError(error, res, "getUser");
    }
  }
}
```

Không gọi `res.json` thô bên ngoài các helpers của BaseController.

---

### 5. Tất cả Lỗi Đi đến Sentry

```ts
catch (error) {
  Sentry.captureException(error);
  throw error;
}
```

❌ `console.log`
❌ thất bại im lặng (silent failures)
❌ nuốt lỗi (swallowed errors)

---

### 6. unifiedConfig Là Nguồn Cấu hình Duy nhất

```ts
// ❌ KHÔNG BAO GIỜ
process.env.JWT_SECRET;

// ✅ LUÔN LUÔN
import { config } from "@/config/unifiedConfig";
config.auth.jwtSecret;
```

---

### 7. Xác thực Tất cả Đầu vào Bên ngoài với Zod

- Request bodies
- Query params
- Route params
- Webhook payloads

```ts
const schema = z.object({
  email: z.string().email(),
});

const input = schema.parse(req.body);
```

Không xác thực = bug.

---

## 4. Cấu trúc Thư mục (Canonical)

```
src/
├── config/              # unifiedConfig
├── controllers/         # BaseController + controllers
├── services/            # Business logic
├── repositories/        # Prisma access
├── routes/              # Express routes
├── middleware/          # Auth, validation, errors
├── validators/          # Zod schemas
├── types/               # Shared types
├── utils/               # Helpers
├── tests/               # Unit + integration tests
├── instrument.ts        # Sentry (NHẬP ĐẦU TIÊN)
├── app.ts               # Express app
└── server.ts            # HTTP server
```

---

## 5. Quy ước Đặt tên (Nghiêm ngặt)

| Lớp (Layer) | Quy ước                   |
| ----------- | ------------------------- |
| Controller  | `PascalCaseController.ts` |
| Service     | `camelCaseService.ts`     |
| Repository  | `PascalCaseRepository.ts` |
| Routes      | `camelCaseRoutes.ts`      |
| Validators  | `camelCase.schema.ts`     |

---

## 6. Quy tắc Dependency Injection

- Services nhận dependencies qua constructor
- Không import repositories trực tiếp bên trong controllers
- Cho phép mocking và testing

```ts
export class UserService {
  constructor(private readonly userRepository: UserRepository) {}
}
```

---

## 7. Quy tắc Prisma & Repository

- Prisma client **không bao giờ được sử dụng trực tiếp trong controllers**
- Repositories:
  - Đóng gói các truy vấn
  - Xử lý giao dịch
  - Hiển thị các phương thức dựa trên ý định (intent-based methods)

```ts
await userRepository.findActiveUsers();
```

---

## 8. Xử lý Async & Lỗi

### Bắt buộc dùng asyncErrorWrapper

Tất cả các async route handlers phải được bọc.

```ts
router.get(
  "/users",
  asyncErrorWrapper((req, res) => controller.list(req, res)),
);
```

Không có unhandled promise rejections.

---

## 9. Observability & Giám sát

### Bắt buộc

- Theo dõi lỗi Sentry
- Truy vết hiệu năng Sentry
- Logs có cấu trúc (nơi áp dụng)

Mọi đường dẫn quan trọng phải có khả năng quan sát.

---

## 10. Kỷ luật Kiểm thử

### Các Test Bắt buộc

- **Unit tests** cho services
- **Integration tests** cho routes
- **Repository tests** cho các truy vấn phức tạp

```ts
describe("UserService", () => {
  it("creates a user", async () => {
    expect(user).toBeDefined();
  });
});
```

Không có tests → không merge.

---

## 11. Anti-Patterns (Từ chối Ngay lập tức)

❌ Logic kinh doanh trong routes
❌ Bỏ qua lớp service
❌ Prisma trực tiếp trong controllers
❌ Thiếu xác thực
❌ Sử dụng process.env
❌ console.log thay vì Sentry
❌ Logic kinh doanh không được kiểm thử

---

## 12. Tích hợp Với Các Kỹ năng Khác

- **frontend-dev-guidelines** → Căn chỉnh hợp đồng API
- **error-tracking** → Tiêu chuẩn Sentry
- **database-verification** → Tính đúng đắn của Schema
- **analytics-tracking** → Đường ống sự kiện
- **skill-developer** → Quản trị kỹ năng

---

## 13. Danh sách Kiểm tra Xác thực của Người vận hành

Trước khi hoàn thiện công việc backend:

- [ ] BFRI ≥ 3
- [ ] Tuân thủ kiến trúc phân lớp
- [ ] Đầu vào đã được xác thực
- [ ] Lỗi được ghi nhận trong Sentry
- [ ] Sử dụng unifiedConfig
- [ ] Tests đã được viết
- [ ] Không có anti-patterns

---

## 14. Trạng thái Kỹ năng

**Trạng thái:** Ổn định · Có thể thực thi · Cấp Production
**Mục đích sử dụng:** Microservices Node.js tồn tại lâu dài với lưu lượng thực và rủi ro thực

---
