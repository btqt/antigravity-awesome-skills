# skill-creator

**Tự động hóa việc tạo CLI skill với các best practices được tích hợp sẵn.**

## Skill này làm gì (What It Does)

Skill-creator tự động hóa toàn bộ quy trình tạo các CLI skill mới cho GitHub Copilot CLI và Claude Code. Nó hướng dẫn bạn qua quá trình brainstorming, áp dụng các template tiêu chuẩn hóa, kiểm tra (validate) chất lượng nội dung và xử lý việc installation—tất cả trong khi vẫn tuân theo các best practices chính thức của Anthropic.

## Các tính năng chính (Key Features)

- **🎯 Interactive Brainstorming** - Phiên hợp tác để xác định mục đích và phạm vi của skill
- **✨ Template Automation** - Tự động tạo file mà không cần cấu hình thủ công
- **🔍 Quality Validation** - Các bước kiểm tra tích hợp cho YAML, chất lượng nội dung và phong cách viết (writing style)
- **📦 Flexible Installation** - Chọn cài đặt chỉ trong repository, global, hoặc cả hai (hybrid)
- **📊 Visual Progress Bar** - Chỉ báo tiến độ theo thời gian thực hiển thị trạng thái hoàn thành (ví dụ: `[████████████░░░░░░] 60% - Bước 3/5`)
- **🔗 Prompt Engineer Integration** - Tăng cường tùy chọn bằng cách sử dụng skill prompt-engineer

## Khi nào nên sử dụng (When to Use)

Sử dụng skill này khi bạn muốn:

- Tạo một CLI skill mới tuân theo các tiêu chuẩn chính thức
- Mở rộng chức năng CLI với các khả năng tùy chỉnh
- Đóng gói kiến thức chuyên môn (domain knowledge) vào định dạng skill có thể tái sử dụng
- Tự động hóa các tác vụ CLI lặp đi lặp lại với một skill tùy chỉnh
- Cài đặt các skill cục bộ (local) hoặc trên toàn hệ thống (global)

## Cài đặt (Installation)

### Điều kiện tiên quyết (Prerequisites)

Skill này là một phần của repository `cli-ai-skills`. Để sử dụng nó:

```bash
# Clone repository
git clone https://github.com/yourusername/cli-ai-skills.git
cd cli-ai-skills
```

### Cài đặt trên toàn hệ thống (Global) (Khuyến nghị)

Cài đặt thông qua các symlink để skill có sẵn ở mọi nơi:

```bash
# Cho GitHub Copilot CLI
ln -sf "$(pwd)/.github/skills/skill-creator" ~/.copilot/skills/skill-creator

# Cho Claude Code
ln -sf "$(pwd)/.claude/skills/skill-creator" ~/.claude/skills/skill-creator
```

**Lợi ích của global installation:**

- Hoạt động trong bất kỳ thư mục nào
- Tự động cập nhật khi bạn `git pull` repository
- Không cần các file cấu hình

### Cài đặt chỉ trong Repository (Repository-Only Installation)

Nếu bạn muốn chỉ sử dụng skill trong repository này, không cần cài đặt gì thêm. Skill sẽ có sẵn khi làm việc trong thư mục `cli-ai-skills`.

## Cách sử dụng (Usage)

### Tạo Skill cơ bản (Basic Skill Creation)

Chỉ cần yêu cầu CLI tạo một skill mới:

```bash
# GitHub Copilot CLI
gh copilot "create a new skill for debugging Python errors"

# Claude Code
claude "create a skill that helps with git workflows"
```

Skill sẽ hướng dẫn bạn qua với việc theo dõi tiến độ trực quan:

1. **Brainstorming** (20%) - Xác định mục đích, các trigger và loại skill
2. **Prompt Enhancement** (40%, tùy chọn) - Tăng cường với skill prompt-engineer
3. **File Generation** (60%) - Tạo các file từ template
4. **Validation** (80%) - Kiểm tra chất lượng và tiêu chuẩn
5. **Installation** (100%) - Chọn cài đặt local, global, hoặc cả hai

Mỗi giai đoạn đều hiển thị một thanh tiến độ:

```
[████████████░░░░░░] 60% - Bước 3/5: Tạo File (File Generation)
```

### Cách sử dụng nâng cao (Advanced Usage)

#### Tạo Skill về Code Generation

```bash
"Create a code skill that generates React components from descriptions"
```

Skill sẽ:

- Sử dụng template chuyên dụng `code-skill-template.md`
- Hỏi về các framework cụ thể (React, Vue, v.v.)
- Bao gồm các ví dụ về code trong thư mục `examples/`

#### Tạo Skill về Tài liệu (Documentation)

```bash
"Build a skill that writes API documentation from code"
```

Skill sẽ:

- Sử dụng `documentation-skill-template.md`
- Hỏi về các định dạng tài liệu
- Thiết lập các reference cho các style guide

#### Cài đặt cho Platform cụ thể

```bash
"Create a skill for Copilot only that analyzes TypeScript errors"
```

Skill sẽ:

- Chỉ tạo các file trong `.github/skills/`
- Bỏ qua cài đặt dành riêng cho Claude
- Kiểm tra (validate) so với các yêu cầu của Copilot

## Ví dụ quy trình (Example Walkthrough)

Dưới đây là một ví dụ về việc tạo một skill:

```
You: "create a skill for database schema migrations"

[████░░░░░░░░░░░░░░] 20% - Bước 1/5: Brainstorming & Lập kế hoạch

Skill này nên làm gì?
> Giúp người dùng tạo và quản lý database schema migrations một cách an toàn

Khi nào nó nên được kích hoạt? (3-5 cụm từ)
> "create migration", "generate schema change", "migrate database"

Loại skill nào?
> [×] Mục đích chung (General purpose)

Platform nào?
> [×] Cả hai (Copilot + Claude)

[... tiếp tục qua tất cả các giai đoạn ...]

🎉 Đã tạo skill thành công!

📦 Tên Skill: database-migration
📁 Vị trí: .github/skills/database-migration/
🔗 Đã cài đặt: Global (Copilot + Claude)
```

## Cấu trúc File (File Structure)

Khi bạn tạo một skill, cấu trúc này sẽ được tạo ra:

```
.github/skills/your-skill-name/
├── SKILL.md              # Các hướng dẫn chính cho skill (1.5-2k từ)
├── README.md             # Tài liệu cho người dùng (file này)
├── references/           # Hướng dẫn chi tiết (mỗi file 2k-5k từ)
│   └── (trống, sẵn sàng cho tài liệu mở rộng)
├── examples/             # Các mẫu code hoạt động được
│   └── (trống, sẵn sàng cho các ví dụ)
└── scripts/              # Các công cụ tiện ích thực thi
    └── (trống, sẵn sàng cho việc tự động hóa)
```

## Cấu hình (Configuration)

**Không cần cấu hình!** Skill này sử dụng khám phá runtime để:

- Phát hiện các platform đã cài đặt (Copilot CLI, Claude Code)
- Tự động tìm thư mục gốc (root) của repository
- Trích xuất thông tin tác giả từ git config
- Xác định vị trí file tối ưu

## Kiểm tra (Validation)

Mỗi skill được tạo ra đều tự động được kiểm tra cho:

- ✅ **YAML Frontmatter** - Các trường bắt buộc và định dạng
- ✅ **Định dạng mô tả (Description Format)** - Ngôi thứ ba, cụm từ kích hoạt (trigger phrases)
- ✅ **Số lượng từ (Word Count)** - Lý tưởng là 1,500-2,000, tối đa dưới 5,000
- ✅ **Phong cách viết (Writing Style)** - Dạng mệnh lệnh, không dùng ngôi thứ hai
- ✅ **Progressive Disclosure** - Tổ chức nội dung hợp lý

## Các Framework đã sử dụng (Frameworks Used)

Skill này tận dụng một số phương pháp đã được thiết lập:

- **Progressive Disclosure** - Phân cấp nội dung 3 cấp độ (metadata → SKILL.md → bundled resources)
- **Bundled Resources Pattern** - References, ví dụ và script được tách thành các file riêng biệt
- **Anthropic Best Practices** - Các tiêu chuẩn phát triển skill chính thức
- **Zero-Config Design** - Khám phá runtime, không có giá trị được hardcode
- **Template-Driven Generation** - Cấu trúc nhất quán trên tất cả các skill

## Xử lý sự cố (Troubleshooting)

### Lỗi "Template not found"

Đảm bảo bạn đang ở trong repository `cli-ai-skills` hoặc đã clone nó:

```bash
git clone https://github.com/yourusername/cli-ai-skills.git
cd cli-ai-skills
```

### Cảnh báo "Platform not detected"

Nếu các platform không được phát hiện:

1. Chọn cài đặt "Chỉ trong Repository" (Repository only)
2. Chỉ định platform thủ công trong quá trình thiết lập
3. Cài đặt global sau bằng các lệnh được cung cấp

### Thất bại khi Validation

Nếu validation phát hiện vấn đề:

- Xem xét các gợi ý trong kết quả đầu ra
- Chọn các bản sửa lỗi tự động cho các vấn đề phổ biến
- Chỉnh sửa file thủ công cho các vấn đề phức tạp
- Chạy lại validation: `scripts/validate-skill-yaml.sh .github/skills/your-skill`

## Các tính năng nâng cao (Advanced Features)

### Tích hợp Prompt Engineer

Nâng cao mô tả skill của bạn với AI:

1. Kích hoạt trong Giai đoạn 2 (Prompt Refinement)
2. Skill sẽ tự động gọi `prompt-engineer`
3. Xem xét kết quả được tăng cường trước khi tiếp tục

### Các tài nguyên đi kèm (Bundled Resources)

Đối với các skill phức tạp, hãy sử dụng các tài nguyên đi kèm:

- **references/** - Tài liệu chi tiết (không giới hạn số lượng từ)
- **examples/** - Các mẫu code hoạt động mà người dùng có thể chạy
- **scripts/** - Các công cụ tự động hóa được tải theo yêu cầu

### Quản lý phiên bản (Version Management)

Cập nhật các skill hiện có:

```bash
scripts/update-skill-version.sh your-skill-name 1.1.0
```

## Đóng góp (Contributing)

Đã tạo một skill hữu ích? Hãy chia sẻ nó:

1. Đảm bảo vượt qua các bước validation
2. Thêm các ví dụ sử dụng
3. Cập nhật file README.md chính
4. Gửi một pull request

## Tài nguyên (Resources)

- **Writing Style Guide:** `resources/templates/writing-style-guide.md`
- **Anthropic Official Guide:** https://github.com/anthropics/claude-plugins-official
- **Thư mục Templates:** `resources/templates/`
- **Các Validation Script:** `scripts/validate-*.sh`

## Hỗ trợ (Support)

Đối với các vấn đề hoặc câu hỏi:

- Kiểm tra các skill hiện có trong `.github/skills/` để lấy ví dụ
- Xem lại `resources/skills-development.md` để biết về phương pháp luận
- Mở một issue trong repository

---

**Phiên bản:** 1.1.0  
**Platform:** GitHub Copilot CLI, Claude Code  
**Tác giả:** Eric Andrade  
**Cập nhật lần cuối:** 2026-02-01
