---
name: agent-memory-mcp
author: Amit Rathiesh
description: Một hệ thống bộ nhớ lai cung cấp khả năng quản lý kiến thức bền vững, có thể tìm kiếm cho các đại lý AI (Kiến trúc, Mẫu, Quyết định).
---

# Kỹ năng Bộ nhớ Đại lý (Agent Memory Skill)

Kỹ năng này cung cấp một ngân hàng bộ nhớ bền vững, có thể tìm kiếm, tự động đồng bộ hóa với tài liệu dự án. Nó chạy như một máy chủ MCP để cho phép đọc/ghi/tìm kiếm các ký ức dài hạn.

## Điều kiện tiên quyết

- Node.js (v18+)

## Thiết lập

1. **Clone kho lưu trữ**:
   Clone dự án `agentMemory` vào không gian làm việc của đại lý hoặc một thư mục song song:

   ```bash
   git clone https://github.com/webzler/agentMemory.git .agent/skills/agent-memory
   ```

2. **Cài đặt các phụ thuộc**:

   ```bash
   cd .agent/skills/agent-memory
   npm install
   npm run compile
   ```

3. **Bắt đầu máy chủ MCP**:
   Sử dụng tập lệnh bổ trợ để kích hoạt ngân hàng bộ nhớ cho dự án hiện tại của bạn:

   ```bash
   npm run start-server <project_id> <absolute_path_to_target_workspace>
   ```

   _Ví dụ cho thư mục hiện tại:_

   ```bash
   npm run start-server my-project $(pwd)
   ```

## Năng lực (Các công cụ MCP)

### `memory_search`

Tìm kiếm ký ức theo câu truy vấn, loại hoặc thẻ.

- **Đối số**: `query` (chuỗi), `type?` (chuỗi), `tags?` (mảng chuỗi)
- **Cách dùng**: "Tìm tất cả các mẫu xác thực" -> `memory_search({ query: "authentication", type: "pattern" })`

### `memory_write`

Ghi lại kiến thức hoặc quyết định mới.

- **Đối số**: `key` (chuỗi), `type` (chuỗi), `content` (chuỗi), `tags?` (mảng chuỗi)
- **Cách dùng**: "Lưu quyết định kiến trúc này" -> `memory_write({ key: "auth-v1", type: "decision", content: "..." })`

### `memory_read`

Lấy nội dung ký ức cụ thể theo khóa (key).

- **Đối số**: `key` (chuỗi)
- **Cách dùng**: "Lấy thiết kế xác thực" -> `memory_read({ key: "auth-v1" })`

### `memory_stats`

Xem phân tích về việc sử dụng bộ nhớ.

- **Cách dùng**: "Hiển thị thống kê bộ nhớ" -> `memory_stats({})`

## Bảng điều khiển (Dashboard)

Kỹ năng này bao gồm một bảng điều khiển độc lập để trực quan hóa việc sử dụng bộ nhớ.

```bash
npm run start-dashboard <absolute_path_to_target_workspace>
```

Truy cập tại: `http://localhost:3333`
