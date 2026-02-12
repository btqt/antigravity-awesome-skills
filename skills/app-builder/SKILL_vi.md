---
name: app-builder
description: Điều phối viên xây dựng ứng dụng chính. Tạo ứng dụng full-stack từ các yêu cầu ngôn ngữ tự nhiên. Xác định loại dự án, chọn tech stack, điều phối các agent.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Agent
---

# App Builder - Điều phối viên Xây dựng Ứng dụng

> Phân tích yêu cầu của người dùng, xác định tech stack, lập kế hoạch cấu trúc và điều phối các agent.

## 🎯 Quy tắc Đọc Chọn lọc

**CHỈ đọc các tệp liên quan đến yêu cầu!** Kiểm tra bản đồ nội dung, tìm những gì bạn cần.

| Tệp                     | Mô tả                                      | Khi nào cần Đọc                  |
| ----------------------- | ------------------------------------------ | -------------------------------- |
| `project-detection.md`  | Ma trận từ khóa, phát hiện loại dự án      | Bắt đầu dự án mới                |
| `tech-stack.md`         | Stack mặc định 2025, các lựa chọn thay thế | Chọn công nghệ                   |
| `agent-coordination.md` | Pipeline của agent, thứ tự thực thi        | Điều phối công việc đa agent     |
| `scaffolding.md`        | Cấu trúc thư mục, các tệp cốt lõi          | Tạo cấu trúc dự án               |
| `feature-building.md`   | Phân tích tính năng, xử lý lỗi             | Thêm tính năng vào dự án hiện có |
| `templates/SKILL.md`    | **Các mẫu dự án**                          | Scaffolding dự án mới            |

---

## 📦 Các mẫu (Templates) (13)

Scaffolding khởi động nhanh cho các dự án mới. **Chỉ đọc mẫu phù hợp!**

| Mẫu                                                            | Tech Stack          | Khi nào sử dụng              |
| -------------------------------------------------------------- | ------------------- | ---------------------------- |
| [nextjs-fullstack](templates/nextjs-fullstack/TEMPLATE.md)     | Next.js + Prisma    | Ứng dụng web Full-stack      |
| [nextjs-saas](templates/nextjs-saas/TEMPLATE.md)               | Next.js + Stripe    | Sản phẩm SaaS                |
| [nextjs-static](templates/nextjs-static/TEMPLATE.md)           | Next.js + Framer    | Trang Landing page           |
| [nuxt-app](templates/nuxt-app/TEMPLATE.md)                     | Nuxt 3 + Pinia      | Ứng dụng Vue full-stack      |
| [express-api](templates/express-api/TEMPLATE.md)               | Express + JWT       | REST API                     |
| [python-fastapi](templates/python-fastapi/TEMPLATE.md)         | FastAPI             | Python API                   |
| [react-native-app](templates/react-native-app/TEMPLATE.md)     | Expo + Zustand      | Ứng dụng di động             |
| [flutter-app](templates/flutter-app/TEMPLATE.md)               | Flutter + Riverpod  | Di động đa nền tảng          |
| [electron-desktop](templates/electron-desktop/TEMPLATE.md)     | Electron + React    | Ứng dụng máy tính để bàn     |
| [chrome-extension](templates/chrome-extension/TEMPLATE.md)     | Chrome MV3          | Tiện ích mở rộng trình duyệt |
| [cli-tool](templates/cli-tool/TEMPLATE.md)                     | Node.js + Commander | Ứng dụng CLI                 |
| [monorepo-turborepo](templates/monorepo-turborepo/TEMPLATE.md) | Turborepo + pnpm    | Monorepo                     |

---

## 🔗 Các Agent Liên quan

| Agent                 | Vai trò                              |
| --------------------- | ------------------------------------ |
| `project-planner`     | Phân chia nhiệm vụ, đồ thị phụ thuộc |
| `frontend-specialist` | Thành phần UI, các trang             |
| `backend-specialist`  | API, logic nghiệp vụ                 |
| `database-architect`  | Lược đồ (Schema), migrations         |
| `devops-engineer`     | Triển khai, xem trước (preview)      |

---

## Ví dụ Sử dụng

```
User: "Make an Instagram clone with photo sharing and likes"

Quy trình App Builder:
1. Loại dự án: Ứng dụng Mạng xã hội
2. Tech stack: Next.js + Prisma + Cloudinary + Clerk
3. Tạo kế hoạch:
   ├─ Lược đồ cơ sở dữ liệu (users, posts, likes, follows)
   ├─ Các tuyến API (12 endpoints)
   ├─ Các trang (feed, profile, upload)
   └─ Các thành phần (PostCard, Feed, LikeButton)
4. Điều phối các agent
5. Báo cáo tiến độ
6. Bắt đầu xem trước
```
