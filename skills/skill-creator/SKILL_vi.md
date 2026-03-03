---
name: skill-creator
description: "Skill này được sử dụng khi người dùng yêu cầu tạo skill mới, xây dựng một skill, tạo skill tùy chỉnh, phát triển CLI skill, hoặc muốn mở rộng CLI với các khả năng mới. Tự động hóa toàn bộ quá trình..."
category: meta
risk: safe
source: community
tags: "[automation, scaffolding, skill-creation, meta-skill]"
date_added: "2026-02-27"
---

# skill-creator

## Mục đích (Purpose)

Để tạo các CLI skill mới tuân theo các best practices chính thức của Anthropic mà không cần cấu hình thủ công. Skill này tự động hóa việc brainstorming, áp dụng template, validation và các quá trình installation trong khi vẫn duy trì các pattern progressive disclosure và các tiêu chuẩn phong cách viết (writing style).

## Khi nào nên sử dụng Skill này (When to Use This Skill)

Skill này nên được sử dụng khi:

- Người dùng muốn mở rộng chức năng CLI với các khả năng tùy chỉnh
- Người dùng cần tạo một skill tuân theo các tiêu chuẩn chính thức
- Người dùng muốn tự động hóa các tác vụ CLI lặp đi lặp lại với một skill có thể tái sử dụng
- Người dùng cần đóng gói kiến thức chuyên môn (domain knowledge) vào định dạng skill
- Người dùng muốn cả hai tùy chọn local và global skill installation

## Các khả năng cốt lõi (Core Capabilities)

1. **Interactive Brainstorming** - Phiên hợp tác để xác định mục đích và phạm vi của skill
2. **Prompt Enhancement** - Tích hợp tùy chọn với skill prompt-engineer để tinh chỉnh
3. **Template Application** - Tự động tạo file từ các template tiêu chuẩn hóa
4. **Validation** - Kiểm tra YAML, content và style so với các tiêu chuẩn của Anthropic
5. **Installation** - Local repository hoặc global installation với các symlink
6. **Progress Tracking** - Thước đo trực quan hiển thị trạng thái hoàn thành tại mỗi bước

## Bước 0: Khám phá (Step 0: Discovery)

Trước khi bắt đầu tạo skill, hãy thu thập thông tin runtime:

```bash
# Phát hiện các platform có sẵn
COPILOT_INSTALLED=false
CLAUDE_INSTALLED=false
CODEX_INSTALLED=false

if command -v gh &>/dev/null && gh copilot --version &>/dev/null 2>&1; then
    COPILOT_INSTALLED=true
fi

if [[ -d "$HOME/.claude" ]]; then
    CLAUDE_INSTALLED=true
fi

if [[ -d "$HOME/.codex" ]]; then
    CODEX_INSTALLED=true
fi

# Xác định thư mục làm việc (working directory)
REPO_ROOT=$(git rev-parse --show-toplevel 2>/dev/null || pwd)
SKILLS_REPO="$REPO_ROOT"

# Kiểm tra nếu đang ở trong cli-ai-skills repository
if [[ ! -d "$SKILLS_REPO/.github/skills" ]]; then
    echo "⚠️  Không ở trong cli-ai-skills repository. Đang tạo skill độc lập (standalone skill)."
    STANDALONE=true
fi

# Lấy thông tin người dùng từ git config
AUTHOR=$(git config user.name || echo "Unknown")
EMAIL=$(git config user.email || echo "")
```

**Các thông tin chính cần thiết:**

- Platform nào cần hướng tới (Copilot, Claude, Codex, hoặc cả ba)
- Lựa chọn installation (local, global, hoặc cả hai)
- Tên và mục đích của skill
- Loại skill (general, code, documentation, analysis)

## Quy trình chính (Main Workflow)

### Hướng dẫn theo dõi tiến độ (Progress Tracking Guidelines)

Trong suốt quy trình làm việc, hãy hiển thị một thanh tiến độ trực quan trước khi bắt đầu mỗi giai đoạn để thông báo cho người dùng. Định dạng thanh tiến độ là:

```
[████████████░░░░░░] 60% - Bước 3/5: Đang tạo SKILL.md
```

**Thông số kỹ thuật định dạng:**

- Chiều rộng 20 ký tự (sử dụng █ cho phần đã đầy, ░ cho phần trống)
- Phần trăm dựa trên bước hiện tại (Bước 1=20%, Bước 2=40%, Bước 3=60%, Bước 4=80%, Bước 5=100%)
- Bộ đếm bước hiển thị hiện tại/tổng số (ví dụ: "Bước 3/5")
- Mô tả ngắn gọn về giai đoạn hiện tại

**Hiển thị thanh tiến độ bằng cách sử dụng:**

```bash
echo "[████░░░░░░░░░░░░░░] 20% - Bước 1/5: Brainstorming & Lập kế hoạch"
```

### Giai đoạn 1: Brainstorming & Lập kế hoạch (Phase 1: Brainstorming & Planning)

**Tiến độ:** Hiển thị trước khi bắt đầu giai đoạn này:

```bash
echo "[████░░░░░░░░░░░░░░] 20% - Bước 1/5: Brainstorming & Lập kế hoạch"
```

Hiển thị tiến độ:

```
╔══════════════════════════════════════════════════════════════╗
║     🛠️  SKILL CREATOR - Đang tạo Skill mới                  ║
╠══════════════════════════════════════════════════════════════╣
║ → Giai đoạn 1: Brainstorming             [10%]               ║
║ ○ Giai đoạn 2: Tinh chỉnh Prompt                             ║
║ ○ Giai đoạn 3: Tạo File                                      ║
║ ○ Giai đoạn 4: Validation                                    ║
║ ○ Giai đoạn 5: Installation                                  ║
╠══════════════════════════════════════════════════════════════╣
║ Tiến độ: ███░░░░░░░░░░░░░░░░░░░░░░░░░░░  10%               ║
╚══════════════════════════════════════════════════════════════╝
```

**Hỏi người dùng:**

1. **Skill này nêng làm gì?** (Mô tả tự do)
   - Ví dụ: "Giúp người dùng debug code Python bằng cách phân tích stack trace"
2. **Khi nào nó nên được kích hoạt?** (Cung cấp 3-5 cụm từ kích hoạt - trigger phrases)
   - Ví dụ: "debug lỗi Python", "phân tích stack trace", "sửa exception Python"

3. **Đây là loại skill gì?**
   - [ ] Mục đích chung (template mặc định)
   - [ ] Tạo/sửa đổi code
   - [ ] Tạo/duy trì tài liệu (documentation)
   - [ ] Phân tích/điều tra

4. **Những platform nào nên hỗ trợ skill này?**
   - [ ] GitHub Copilot CLI
   - [ ] Claude Code
   - [ ] Codex
   - [ ] Cả ba (khuyến nghị)

5. **Cung cấp mô tả trong một câu** (sẽ xuất hiện trong metadata)
   - Ví dụ: "Phân tích stack trace Python và gợi ý cách sửa lỗi"

**Ghi lại các phản hồi và chuẩn bị cho giai đoạn tiếp theo.**

### Giai đoạn 2: Tăng cường Prompt (Tùy chọn) (Phase 2: Prompt Enhancement)

**Tiến độ:** Hiển thị trước khi bắt đầu giai đoạn này:

```bash
echo "[████████░░░░░░░░░░] 40% - Bước 2/5: Tăng cường Prompt"
```

Cập nhật tiến độ:

```
╔══════════════════════════════════════════════════════════════╗
║ ✓ Giai đoạn 1: Brainstorming                                 ║
║ → Giai đoạn 2: Tinh chỉnh Prompt         [30%]               ║
╠══════════════════════════════════════════════════════════════╣
║ Tiến độ: █████████░░░░░░░░░░░░░░░░░░░░░  30%               ║
╚══════════════════════════════════════════════════════════════╝
```

**Hỏi người dùng:**
"Bạn có muốn tinh chỉnh mô tả skill bằng cách sử dụng skill prompt-engineer không?"

- [ ] Có - Sử dụng prompt-engineer để tăng cường độ rõ ràng và cấu trúc
- [ ] Không - Tiếp tục với mô tả hiện tại

Nếu **Có**:

1. Kiểm tra xem skill prompt-engineer có sẵn không
2. Gọi skill với mô tả hiện tại làm đầu vào (input)
3. Xem xét kết quả đã được tăng cường với người dùng
4. Hỏi: "Chấp nhận phiên bản đã tăng cường hay giữ nguyên bản gốc?"

Nếu **Không** hoặc prompt-engineer không có sẵn:

- Tiếp tục với đầu vào ban đầu của người dùng

### Giai đoạn 3: Tạo File (Phase 3: File Generation)

**Tiến độ:** Hiển thị trước khi bắt đầu giai đoạn này:

```bash
echo "[████████████░░░░░░] 60% - Bước 3/5: Tạo File"
```

Cập nhật tiến độ:

```
╔══════════════════════════════════════════════════════════════╗
║ ✓ Giai đoạn 1: Brainstorming                                 ║
║ ✓ Giai đoạn 2: Tinh chỉnh Prompt                             ║
║ → Giai đoạn 3: Tạo File                  [50%]               ║
╠══════════════════════════════════════════════════════════════╣
║ Tiến độ: ███████████████░░░░░░░░░░░░░░░  50%               ║
╚══════════════════════════════════════════════════════════════╝
```

**Tạo cấu trúc skill:**

```bash
# Chuyển đổi tên skill sang kebab-case
SKILL_NAME=$(echo "$USER_INPUT" | tr '[:upper:]' '[:lower:]' | tr ' ' '-')

# Tạo các thư mục
if [[ "$PLATFORM" =~ "copilot" ]]; then
    mkdir -p ".github/skills/$SKILL_NAME"/{references,examples,scripts}
fi

if [[ "$PLATFORM" =~ "claude" ]]; then
    mkdir -p ".claude/skills/$SKILL_NAME"/{references,examples,scripts}
fi

if [[ "$PLATFORM" =~ "codex" ]]; then
    mkdir -p ".codex/skills/$SKILL_NAME"/{references,examples,scripts}
fi
```

**Áp dụng các template:**

1. **SKILL.md** - Sử dụng template phù hợp:
   - `skill-template-copilot.md`, `skill-template-claude.md`, hoặc `skill-template-codex.md`
   - Thay thế các placeholder:
     - `{{SKILL_NAME}}` → tên kebab-case
     - `{{DESCRIPTION}}` → mô tả một dòng
     - `{{TRIGGERS}}` → các cụm từ kích hoạt cách nhau bởi dấu phẩy
     - `{{PURPOSE}}` → mục đích chi tiết từ brainstorming
     - `{{AUTHOR}}` → từ git config
     - `{{DATE}}` → ngày hiện tại (YYYY-MM-DD)
     - `{{VERSION}}` → "1.0.0"

2. **README.md** - Sử dụng `readme-template.md`:
   - Tài liệu hướng dẫn cho người dùng (300-500 từ)
   - Bao gồm hướng dẫn installation
   - Thêm các ví dụ sử dụng (usage examples)

3. **References/** (tùy chọn nhưng được khuyến nghị):
   - Tạo `detailed-guide.md` cho tài liệu mở rộng (2k-5k từ)
   - Chuyển nội dung dài vào đây để giữ SKILL.md dưới 2k từ

**Các lệnh tạo file:**

```bash
# Áp dụng template với việc thay thế
sed "s/{{SKILL_NAME}}/$SKILL_NAME/g; \
     s/{{DESCRIPTION}}/$DESCRIPTION/g; \
     s/{{AUTHOR}}/$AUTHOR/g; \
     s/{{DATE}}/$(date +%Y-%m-%d)/g" \
    resources/templates/skill-template-copilot.md \
    > ".github/skills/$SKILL_NAME/SKILL.md"

# Tạo README
sed "s/{{SKILL_NAME}}/$SKILL_NAME/g" \
    resources/templates/readme-template.md \
    > ".github/skills/$SKILL_NAME/README.md"

# Áp dụng template cho Codex nếu được chọn
if [[ "$PLATFORM" =~ "codex" ]]; then
    sed "s/{{SKILL_NAME}}/$SKILL_NAME/g; \
         s/{{DESCRIPTION}}/$DESCRIPTION/g; \
         s/{{AUTHOR}}/$AUTHOR/g; \
         s/{{DATE}}/$(date +%Y-%m-%d)/g" \
        resources/templates/skill-template-codex.md \
        > ".codex/skills/$SKILL_NAME/SKILL.md"

    sed "s/{{SKILL_NAME}}/$SKILL_NAME/g" \
        resources/templates/readme-template.md \
        > ".codex/skills/$SKILL_NAME/README.md"
fi
```

**Hiển thị cấu trúc đã tạo:**

```
✅ Đã tạo:
   .github/skills/your-skill-name/ (nếu chọn Copilot)
   .claude/skills/your-skill-name/ (nếu chọn Claude)
   .codex/skills/your-skill-name/ (nếu chọn Codex)
   ├── SKILL.md (832 dòng)
   ├── README.md (347 dòng)
   ├── references/
   ├── examples/
   └── scripts/
```

### Giai đoạn 4: Validation (Phase 4: Validation)

**Tiến độ:** Hiển thị trước khi bắt đầu giai đoạn này:

```bash
echo "[████████████████░░] 80% - Bước 4/5: Validation"
```

Cập nhật tiến độ:

```
╔══════════════════════════════════════════════════════════════╗
║ ✓ Giai đoạn 3: Tạo File                                      ║
║ → Giai đoạn 4: Validation                [70%]               ║
╠══════════════════════════════════════════════════════════════╣
║ Tiến độ: █████████████████████░░░░░░░░░  70%               ║
╚══════════════════════════════════════════════════════════════╝
```

**Chạy các script validation:**

```bash
# Kiểm tra YAML frontmatter
scripts/validate-skill-yaml.sh ".github/skills/$SKILL_NAME"

# Kiểm tra chất lượng nội dung (content quality)
scripts/validate-skill-content.sh ".github/skills/$SKILL_NAME"
```

**Kết quả mong đợi:**

```
🔍 Đang kiểm tra YAML frontmatter...
✅ YAML frontmatter hợp lệ!

🔍 Đang kiểm tra nội dung...
✅ Số lượng từ tuyệt vời: 1847 từ
✅ Content validation hoàn tất!
```

**Nếu validation thất bại:**

- Hiển thị các lỗi cụ thể
- Đề nghị tự động sửa (cho các vấn đề phổ biến)
- Yêu cầu người dùng sửa thủ công các vấn đề phức tạp

**Các lỗi tự động sửa phổ biến:**

- Chuyển đổi ngôi thứ hai sang dạng mệnh lệnh (imperative form)
- Định dạng lại mô tả sang ngôi thứ ba (third-person)
- Thêm các trường bắt buộc còn thiếu

### Giai đoạn 5: Installation (Phase 5: Installation)

**Tiến độ:** Hiển thị trước khi bắt đầu giai đoạn này:

```bash
echo "[████████████████████] 100% - Bước 5/5: Installation"
```

Cập nhật tiến độ:

```
╔══════════════════════════════════════════════════════════════╗
║ ✓ Giai đoạn 4: Validation                                    ║
║ → Giai đoạn 5: Installation              [90%]               ║
╠══════════════════════════════════════════════════════════════╣
║ Tiến độ: ██████████████████████████░░░░░  90%               ║
╚══════════════════════════════════════════════════════════════╝
```

**Hỏi người dùng:**
"Bạn muốn cài đặt skill này như thế nào?"

- [ ] **Chỉ Repository** - Các file được tạo trong `.github/skills/` (hoạt động khi ở trong repo)
- [ ] **Global installation** - Tạo các symlink trong `~/.copilot/skills/` (hoạt động ở mọi nơi)
- [ ] **Cả hai** - Repository + global symlinks (khuyến nghị, tự động cập nhật với git pull)
- [ ] **Bỏ qua installation** - Chỉ tạo các file

**Nếu chọn global installation:**

```bash
# Phát hiện installation cho các platform nào
INSTALL_TARGETS=()

if [[ "$COPILOT_INSTALLED" == "true" ]] && [[ "$PLATFORM" =~ "copilot" ]]; then
    INSTALL_TARGETS+=("copilot")
fi

if [[ "$CLAUDE_INSTALLED" == "true" ]] && [[ "$PLATFORM" =~ "claude" ]]; then
    INSTALL_TARGETS+=("claude")
fi

if [[ "$CODEX_INSTALLED" == "true" ]] && [[ "$PLATFORM" =~ "codex" ]]; then
    INSTALL_TARGETS+=("codex")
fi

# Yêu cầu người dùng xác nhận các platform đã phát hiện
echo "Các platform đã phát hiện: ${INSTALL_TARGETS[*]}"
echo "Cài đặt cho các platform này? [Y/n]"
```

**Quá trình Installation:**

```bash
# GitHub Copilot CLI
if [[ " ${INSTALL_TARGETS[*]} " =~ " copilot " ]]; then
    ln -sf "$SKILLS_REPO/.github/skills/$SKILL_NAME" \
           "$HOME/.copilot/skills/$SKILL_NAME"
    echo "✅ Đã cài đặt cho GitHub Copilot CLI"
fi

# Claude Code
if [[ " ${INSTALL_TARGETS[*]} " =~ " claude " ]]; then
    ln -sf "$SKILLS_REPO/.claude/skills/$SKILL_NAME" \
           "$HOME/.claude/skills/$SKILL_NAME"
    echo "✅ Đã cài đặt cho Claude Code"
fi

# Codex
if [[ " ${INSTALL_TARGETS[*]} " =~ " codex " ]]; then
    ln -sf "$SKILLS_REPO/.codex/skills/$SKILL_NAME" \
           "$HOME/.codex/skills/$SKILL_NAME"
    echo "✅ Đã cài đặt cho Codex"
fi
```

**Xác minh installation:**

```bash
# Kiểm tra các symlink
ls -la ~/.copilot/skills/$SKILL_NAME 2>/dev/null
ls -la ~/.claude/skills/$SKILL_NAME 2>/dev/null
ls -la ~/.codex/skills/$SKILL_NAME 2>/dev/null
```

### Giai đoạn 6: Hoàn tất (Phase 6: Completion)

**Tiến độ:** Hiển thị thông báo hoàn tất:

```bash
echo "[████████████████████] 100% - ✓ Đã tạo skill thành công!"
```

Cập nhật tiến độ:

```
╔══════════════════════════════════════════════════════════════╗
║ ✓ Giai đoạn 5: Installation                                  ║
║ ✅ QUÁ TRÌNH TẠO SKILL HOÀN TẤT!                             ║
╠══════════════════════════════════════════════════════════════╣
║ Tiến độ: ██████████████████████████████  100%              ║
╚══════════════════════════════════════════════════════════════╝
```

**Hiển thị tóm tắt:**

```
🎉 Đã tạo skill thành công!

📦 Tên Skill: your-skill-name
📁 Vị trí: .github/skills/your-skill-name/
🔗 Đã cài đặt: Global (Copilot + Claude)

📋 Các file đã tạo:
   ✅ SKILL.md (1,847 từ)
   ✅ README.md (423 từ)
   ✅ references/ (trống, sẵn sàng cho tài liệu mở rộng)
   ✅ examples/ (trống, sẵn sàng cho các code sample)
   ✅ scripts/ (trống, sẵn sàng cho các công cụ tiện ích)

🚀 Các bước tiếp theo:
   1. Kiểm tra skill: Thử các cụm từ kích hoạt trong CLI
   2. Thêm ví dụ: Tạo các code sample hoạt động được trong thư mục examples/
   3. Mở rộng tài liệu: Thêm các hướng dẫn chi tiết vào references/
   4. Commit các thay đổi: git add .github/skills/your-skill-name && git commit
   5. Chia sẻ: Push lên repository để nhóm sử dụng

💡 Mẹo hay:
   - Giữ SKILL.md dưới 2,000 từ (hiện tại: 1,847)
   - Chuyển nội dung chi tiết vào thư mục references/
   - Thêm các executable script vào thư mục scripts/
   - Cập nhật README.md với các ví dụ sử dụng thực tế
   - Chạy validation trước khi commit: scripts/validate-skill-yaml.sh
```

## Xử lý lỗi (Error Handling)

### Các vấn đề phát hiện Platform (Platform Detection Issues)

Nếu không thể phát hiện các platform:

```
⚠️  Không thể phát hiện GitHub Copilot CLI hoặc Claude Code

Bạn muốn:
1. Chỉ cài đặt cho repository (hoạt động khi ở trong repo)
2. Chỉ định platform thủ công
3. Bỏ qua installation
```

### Không tìm thấy Template (Template Not Found)

Nếu thiếu các template:

```
❌ Lỗi: Không tìm thấy template tại resources/templates/

Skill này yêu cầu cấu trúc cli-ai-skills repository.

Các tùy chọn:
1. Clone cli-ai-skills: git clone <repo-url>
2. Tạo cấu trúc skill tối thiểu thủ công
3. Thoát và thiết lập các template trước
```

### Thất bại khi Validation (Validation Failures)

Nếu nội dung không đáp ứng các tiêu chuẩn:

```
⚠️  Tìm thấy các vấn đề khi Validation:

1. YAML: Mô tả không ở định dạng ngôi thứ ba
   Kỳ vọng: "Skill này nên được sử dụng khi..."
   Tìm thấy: "Sử dụng skill này khi..."

2. Content: Số lượng từ quá cao (5,342 từ, tối đa 5,000)
   Gợi ý: Di chuyển các phần chi tiết sang references/

Tự động sửa lỗi? [Y/n]
```

### Xung đột khi Installation (Installation Conflicts)

Nếu symlink đã tồn tại:

```
⚠️  Skill đã được cài đặt tại ~/.copilot/skills/your-skill-name

Các tùy chọn:
1. Ghi đè cài đặt hiện có
2. Đổi tên skill mới
3. Bỏ qua installation
4. Cài đặt vào vị trí khác
```

## Các tài nguyên đi kèm (Bundled Resources)

Skill này bao gồm các tài nguyên bổ sung trong các thư mục con:

### references/

Tài liệu chi tiết được tải khi cần thiết:

- `anthropic-best-practices.md` - Các hướng dẫn phát triển skill chính thức của Anthropic
- `writing-style-guide.md` - Các tiêu chuẩn viết và ví dụ
- `progressive-disclosure.md` - Các pattern tổ chức nội dung
- `validation-checklist.md` - Kiểm tra chất lượng trước khi commit

### examples/

Các ví dụ hoạt động trình diễn cách sử dụng skill:

- `basic-skill-creation.md` - Hướng dẫn tạo skill cơ bản
- `advanced-skill-bundled-resources.md` - Skill phức tạp với references/
- `global-installation.md` - Cài đặt các skill trên toàn hệ thống

### scripts/

Các tiện ích thực thi để bảo trì skill:

- `validate-all-skills.sh` - Validation hàng loạt tất cả các skill trong repository
- `update-skill-version.sh` - Tăng phiên bản và cập nhật changelog
- `generate-skill-index.sh` - Tự động tạo danh mục (catalog) skill

## Ghi chú triển khai kỹ thuật (Technical Implementation Notes)

**Thay thế Template (Template Substitution):**

- Sử dụng `sed` cho các thay thế đơn giản
- Duy trì định dạng YAML chính xác
- Xử lý các mô tả nhiều dòng với việc escape đúng cách

**Chiến lược Symlink (Symlink Strategy):**

- Luôn sử dụng đường dẫn tuyệt đối (absolute paths): `ln -sf /đường/dẫn/đến/nguồn ~/.copilot/skills/tên`
- Xác minh symlink trước khi coi installation là hoàn tất
- Lợi ích: Tự động cập nhật khi repository được pull về

**Tích hợp Validation (Validation Integration):**

- Chạy validation trước khi installation
- Chặn installation nếu tìm thấy các lỗi nghiêm trọng (critical errors)
- Các cảnh báo (warnings) chỉ mang tính chất thông tin

**Tích hợp Git (Git Integration):**

- Trích xuất tác giả (author) từ `git config user.name`
- Sử dụng phát hiện thư mục gốc repository (repository root detection): `git rev-parse --show-toplevel`
- Tuân thủ các pattern trong `.gitignore`

## Tiêu chuẩn chất lượng (Quality Standards)

**Yêu cầu đối với SKILL.md (SKILL.md Requirements):**

- 1,500-2,000 từ (lý tưởng)
- Dưới 5,000 từ (tối đa)
- Định dạng mô tả ở ngôi thứ ba
- Phong cách viết mệnh lệnh (imperative)/nguyên mẫu (infinitive)
- Pattern progressive disclosure

**Yêu cầu đối với README.md (README.md Requirements):**

- 300-500 từ
- Ngôn ngữ hướng tới người dùng
- Hướng dẫn installation rõ ràng
- Các ví dụ sử dụng thực tế

**Kiểm tra Validation (Validation Checks):**

- Tính đầy đủ của YAML frontmatter
- Định dạng mô tả (ngôi thứ ba)
- Giới hạn số lượng từ
- Phong cách viết (không dùng ngôi thứ hai)
- Các trường bắt buộc phải hiện diện

## Tài liệu tham khảo (References)

- **Anthropic Official Skill Development Guide:** https://github.com/anthropics/claude-plugins-official/blob/main/plugins/plugin-dev/skills/skill-development/SKILL.md
- **Repository:** https://github.com/yourusername/cli-ai-skills
- **Writing Style Guide:** `resources/templates/writing-style-guide.md`
- **Progress Tracker Template:** `resources/templates/progress-tracker.md`
