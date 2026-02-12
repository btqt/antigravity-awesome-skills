# Thư mục Kỹ năng (Skills)

**Chào mừng bạn đến với thư mục kỹ năng!** Đây là nơi chứa hơn 179 kỹ năng AI chuyên biệt.

## 🤔 Kỹ năng (Skills) là gì?

Kỹ năng là các bộ hướng dẫn chuyên biệt giúp hướng dẫn các trợ lý AI cách xử lý các tác vụ cụ thể. Hãy coi chúng là các mô-đun kiến thức chuyên gia mà AI của bạn có thể tải khi cần.

**Phép so sánh đơn giản:** Giống như việc bạn có thể tham vấn các chuyên gia khác nhau (nhà thiết kế, chuyên gia bảo mật, nhà tiếp thị), kỹ năng cho phép AI của bạn trở thành chuyên gia trong các lĩnh vực khác nhau khi bạn cần.

---

## 📂 Cấu trúc Thư mục

Mỗi kỹ năng nằm trong thư mục riêng với cấu trúc như sau:

```
skills/
├── skill-name/              # Thư mục kỹ năng riêng lẻ
│   ├── SKILL.md             # Định nghĩa kỹ năng chính (bắt buộc)
│   ├── scripts/             # Các tập lệnh bổ trợ (tùy chọn)
│   ├── examples/            # Ví dụ sử dụng (tùy chọn)
│   └── resources/           # Biểu mẫu & tài nguyên (tùy chọn)
```

**Điểm mấu chốt:** Chỉ có `SKILL.md` là bắt buộc. Tất cả những thứ khác đều là tùy chọn!

---

## Cách Sử dụng Kỹ năng

### Bước 1: Đảm bảo các kỹ năng đã được cài đặt

Các kỹ năng nên nằm trong thư mục `.agent/skills/` của bạn (hoặc `.claude/skills/`, `.gemini/skills/`, v.v.)

### Bước 2: Gọi một kỹ năng trong cuộc trò chuyện AI

Sử dụng biểu tượng `@` theo sau là tên kỹ năng:

```
@brainstorming giúp tôi thiết kế một ứng dụng todo
```

hoặc

```
@stripe-integration thêm tính năng xử lý thanh toán vào ứng dụng của tôi
```

### Bước 3: AI trở thành chuyên gia

AI sẽ tải kiến thức của kỹ năng đó và giúp bạn với chuyên môn chuyên sâu!

---

## Danh mục Kỹ năng

### Sáng tạo & Thiết kế

Các kỹ năng về thiết kế hình ảnh, UI/UX và sáng tạo nghệ thuật:

- `@algorithmic-art` - Tạo nghệ thuật thuật toán với p5.js
- `@canvas-design` - Thiết kế poster và tác phẩm nghệ thuật (xuất PNG/PDF)
- `@frontend-design` - Xây dựng giao diện frontend chất lượng cao
- `@ui-ux-pro-max` - Thiết kế UI/UX chuyên nghiệp với màu sắc, phông chữ, bố cục
- `@web-artifacts-builder` - Xây dựng ứng dụng web hiện đại (React, Tailwind, Shadcn/ui)
- `@theme-factory` - Tạo chủ đề (theme) cho tài liệu và bài thuyết trình
- `@brand-guidelines` - Áp dụng các tiêu chuẩn thiết kế thương hiệu của Anthropic
- `@slack-gif-creator` - Tạo GIF chất lượng cao cho Slack

### Phát triển & Kỹ thuật

Các kỹ năng lập trình, kiểm thử, gỡ lỗi và đánh giá mã nguồn:

- `@test-driven-development` - Viết kiểm thử trước khi triển khai (TDD)
- `@systematic-debugging` - Gỡ lỗi theo hệ thống, không ngẫu nhiên
- `@webapp-testing` - Kiểm thử ứng dụng web với Playwright
- `@receiving-code-review` - Xử lý phản hồi đánh giá mã nguồn đúng cách
- `@requesting-code-review` - Yêu cầu đánh giá mã nguồn trước khi hợp nhất
- `@finishing-a-development-branch` - Hoàn tất các nhánh phát triển (hợp nhất, PR, dọn dẹp)
- `@subagent-driven-development` - Phối hợp nhiều tác nhân AI cho các tác vụ song song

### Tài liệu & Văn phòng

Các kỹ năng làm việc với tài liệu và tệp tin văn phòng:

- `@doc-coauthoring` - Cộng tác trên các tài liệu có cấu trúc
- `@docx` - Tạo, chỉnh sửa và phân tích tài liệu Word
- `@xlsx` - Làm việc với bảng tính Excel (công thức, biểu đồ)
- `@pptx` - Tạo và chỉnh sửa bài thuyết trình PowerPoint
- `@pdf` - Xử lý tệp PDF (trích xuất văn bản, hợp nhất, chia nhỏ, điền biểu mẫu)
- `@internal-comms` - Soạn thảo thông tin nội bộ (báo cáo, thông báo)
- `@notebooklm` - Truy vấn sổ tay Google NotebookLM

### Lập kế hoạch & Quy trình làm việc

Các kỹ năng lập kế hoạch tác vụ và tối ưu hóa quy trình làm việc:

- `@brainstorming` - Động não và thiết kế trước khi lập trình
- `@writing-plans` - Viết kế hoạch triển khai chi tiết
- `@planning-with-files` - Hệ thống lập kế hoạch dựa trên tệp (kiểu Manus)
- `@executing-plans` - Thực thi kế hoạch với các điểm kiểm tra và đánh giá
- `@using-git-worktrees` - Tạo các worktree Git độc lập để làm việc song song
- `@verification-before-completion` - Xác minh công việc trước khi xác nhận hoàn thành
- `@using-superpowers` - Khám phá và sử dụng các kỹ năng nâng cao

### Mở rộng Hệ thống

Các kỹ năng mở rộng khả năng của AI:

- `@mcp-builder` - Xây dựng máy chủ MCP (Model Context Protocol)
- `@skill-creator` - Tạo kỹ năng mới hoặc cập nhật kỹ năng hiện có
- `@writing-skills` - Các công cụ để viết và xác thực tệp kỹ năng
- `@dispatching-parallel-agents` - Phân phối tác vụ cho nhiều tác nhân

---

## Tìm kiếm Kỹ năng

### Cách 1: Duyệt thư mục này

```bash
ls skills/
```

### Cách 2: Tìm kiếm theo từ khóa

```bash
ls skills/ | grep "từ-khóa"
```

### Cách 3: Kiểm tra tệp README chính

Xem [README chính](../README_vi.md) để biết danh sách đầy đủ của hơn 179 kỹ năng được sắp xếp theo danh mục.

---

## 💡 Các Kỹ năng Phổ biến nên Thử

**Cho người mới bắt đầu:**

- `@brainstorming` - Thiết kế trước khi lập trình
- `@systematic-debugging` - Sửa lỗi một cách có phương pháp
- `@git-pushing` - Commit với thông báo rõ ràng

**Cho nhà phát triển:**

- `@test-driven-development` - Viết kiểm thử trước
- `@react-best-practices` - Các mẫu React hiện đại
- `@senior-fullstack` - Phát triển Full-stack

**Cho bảo mật:**

- `@ethical-hacking-methodology` - Kiến thức bảo mật cơ bản
- `@burp-suite-testing` - Kiểm thử bảo mật ứng dụng web

---

## Tự Tạo Kỹ năng của Riêng Bạn

Bạn muốn tạo một kỹ năng mới? Hãy xem:

1. [CONTRIBUTING.md](../CONTRIBUTING_vi.md) - Cách đóng góp
2. [docs/SKILL_ANATOMY.md](../docs/vietnamese/SKILL_ANATOMY.vi.md) - Hướng dẫn cấu trúc kỹ năng
3. `@skill-creator` - Sử dụng kỹ năng này để tạo các kỹ năng mới!

**Cấu trúc cơ bản:**

```markdown
---
name: tên-kỹ-năng-của-tôi
description: "Kỹ năng này làm gì"
---

# Tiêu đề Kỹ năng

## Tổng quan

[Kỹ năng này làm gì]

## Khi nào nên Sử dụng

- Sử dụng khi [kịch bản]

## Hướng dẫn

[Hướng dẫn từng bước]

## Ví dụ

[Ví dụ về mã nguồn]
```

---

## Tài liệu

- **[Bắt đầu](../docs/vietnamese/GETTING_STARTED.vi.md)** - Hướng dẫn bắt đầu nhanh
- **[Ví dụ](../docs/vietnamese/EXAMPLES.vi.md)** - Các ví dụ sử dụng thực tế
- **[FAQ](../docs/vietnamese/FAQ.vi.md)** - Các câu hỏi thường gặp
- **[Hướng dẫn Trực quan](../docs/vietnamese/VISUAL_GUIDE.vi.md)** - Sơ đồ và biểu đồ luồng

---

## 🌟 Đóng góp

Bạn tìm thấy một kỹ năng cần cải thiện? Bạn muốn thêm một kỹ năng mới?

1. Đọc [CONTRIBUTING.md](../CONTRIBUTING_vi.md)
2. Nghiên cứu các kỹ năng hiện có trong thư mục này
3. Tạo kỹ năng của bạn theo cấu trúc
4. Gửi Pull Request

---

## Tham chiếu

- [Anthropic Skills](https://github.com/anthropic/skills) - Các kỹ năng chính thức của Anthropic
- [UI/UX Pro Max Skills](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) - Các kỹ năng thiết kế
- [Superpowers](https://github.com/obra/superpowers) - Bộ sưu tập siêu năng lực gốc
- [Planning with Files](https://github.com/OthmanAdi/planning-with-files) - Các mẫu lập kế hoạch
- [NotebookLM](https://github.com/PleasePrompto/notebooklm-skill) - Tích hợp NotebookLM

---

**Cần trợ giúp?** Kiểm tra [FAQ](../docs/vietnamese/FAQ.vi.md) hoặc mở một issue trên GitHub!
