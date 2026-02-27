# Bắt đầu cùng Antigravity Awesome Skills (V4)

**Bạn là người mới? Hướng dẫn này sẽ giúp bạn gia tăng sức mạnh một cách vượt trội cho AI Agent của mình chỉ trong 5 phút.**

---

## 🤔 "Skills" là gì?

Các AI Agents (như **Claude Code**, **Gemini**, **Cursor**) rất thông minh, nhưng chúng thiếu đi kiến thức cụ thể về các tools của bạn.
**Skills** là những bản hướng dẫn chuyên biệt (dưới dạng các file markdown) để chỉ dạy AI của bạn cách thực hiện các tác vụ cụ thể một cách hoàn hảo trong mọi lúc.

**Ví dụ so sánh:** AI của bạn như một thực tập sinh xuất chúng. **Skills** là những bản quy trình chuẩn (SOPs - Standard Operating Procedures) giúp biến họ thành một Senior Engineer thực thụ.

---

## ⚡️ Quick Start: "Starter Packs"

Đừng hoảng sợ trước hơn 700 skills. Bạn không cần dùng tất cả chúng cùng một lúc.
Chúng tôi đã tuyển chọn sẵn các **Starter Packs** để bạn có thể bắt đầu sử dụng ngay lập tức.

Bạn **chỉ cần cài đặt toàn bộ repo một lần duy nhất** (qua npx hoặc clone); Các Starter Packs là những danh sách đã được tuyển chọn để giúp bạn **chọn đúng những skills nào cần dùng** tùy theo vai trò (ví dụ: Web Wizard, Hacker Pack)—đó không phải là một cách cài đặt khác.

### 1. Cài đặt Repo

**Lựa chọn A — npx (dễ nhất):**

```bash
npx antigravity-awesome-skills
```

Lệnh này sẽ clone repo vào thư mục `~/.agent/skills` theo mặc định. Sử dụng các tùy chọn `--cursor`, `--claude`, `--gemini`, hoặc `--codex` để cài đặt cho một tool cụ thể, hoặc dùng `--path <dir>` cho một đường dẫn tùy chỉnh. Chạy lệnh `npx antigravity-awesome-skills --help` để xem thêm chi tiết.

Nếu bạn gặp lỗi 404, hãy sử dụng: `npx github:sickn33/antigravity-awesome-skills`

**Lựa chọn B — git clone:**

```bash
# Phổ quát (hoạt động cho hầu hết các agents)
git clone https://github.com/sickn33/antigravity-awesome-skills.git .agent/skills
```

### 2. Chọn Persona của bạn

Tìm một bundle phù hợp với vai trò của bạn (xem [BUNDLES.md](BUNDLES.md)):

| Persona               | Tên Bundle     | Có những gì bên trong?                                             |
| :-------------------- | :------------- | :----------------------------------------------------------------- |
| **Web Developer**     | `Web Wizard`   | Các React Patterns, Làm chủ Tailwind, Frontend Design              |
| **Security Engineer** | `Hacker Pack`  | Chuẩn OWASP, Metasploit, Pentest Methodology                       |
| **Manager / PM**      | `Product Pack` | Brainstorming, Lên kế hoạch (Planning), SEO, Strategy (Chiến lược) |
| **Bao quát tất cả**   | `Essentials`   | Clean Code, Planning, Validation (Những thứ nền tảng nhất)         |

---

## 🧭 Bundles vs Workflows

Bundles và workflows giải quyết những vấn đề khác nhau:

- **Bundles** = những tập hợp được tinh tuyển theo vai trò chức năng (chỉ ra những gì cần dùng).
- **Workflows** = những bản cẩm nang hướng dẫn theo từng bước một (cách thức thực thi).

Hãy bắt đàu với các bundles trong file [BUNDLES.md](BUNDLES.md), sau đó chạy một workflow từ file [WORKFLOWS.md](WORKFLOWS.md) khi bạn cần một hướng dẫn thực thi cụ thể.

Ví dụ:

> "Hãy sử dụng **@antigravity-workflows** và chạy `ship-saas-mvp` cho ý tưởng project của tôi."

---

## 🚀 Cách sử dụng một Skill

Một khi đã cài đặt, bạn chỉ việc trò chuyện với AI của mình một cách tự nhiên.

### Example 1: Lên kế hoạch (Planning) cho một tính năng (Feature) (**Essentials**)

> "Sử dụng **@brainstorming** để giúp tôi thiết kế một luồng login mới."

**Điều gì diễn ra:** AI sẽ tải skill brainstorming, hỏi bạn những cấu trúc câu hỏi có sẵn, và sản xuất ra một bản đặc tả kỹ thuật chuyên nghiệp (spec).

### Example 2: Checking Code của bạn (**Web Wizard**)

> "Chạy **@lint-and-validate** trên file này và sửa các lỗi."

**Điều gì diễn ra:** AI tuân thủ các strict linting rules được định nghĩa cẩn thận trong một skill để làm sạch code của bạn.

### Example 3: Security Audit (**Hacker Pack**)

> "Sử dụng **@api-security-best-practices** để đánh giá lại các API endpoints của tôi."

**Điều gì diễn ra:** AI sẽ đối chiếu và kiểm tra source code của bạn tuân theo chuẩn quy chuẩn bảo mật OWASP.

---

## 🔌 Supported Tools

| Tool            | Trạng thái       | Đường dẫn (Path)    |
| :-------------- | :--------------- | :------------------ |
| **Claude Code** | ✅ Hỗ trợ đầy đủ | `.claude/skills/`   |
| **Gemini CLI**  | ✅ Hỗ trợ đầy đủ | `.gemini/skills/`   |
| **Codex CLI**   | ✅ Hỗ trợ đầy đủ | `.codex/skills/`    |
| **Antigravity** | ✅ Native        | `.agent/skills/`    |
| **Cursor**      | ✅ Native        | `.cursor/skills/`   |
| **Copilot**     | ⚠️ Chỉ Văn bản   | Copy-paste thủ công |

---

## 🛡️ Trust & Safety (Được thêm ở bản V4)

Chúng tôi phân loại các skills để bạn biết mình đang khởi chạy thứ gì:

- 🟣 **Official**: Được duy trì bởi Anthropic/Google/Vendors (Độ tin cậy cao).
- 🔵 **Safe**: Các skills từ cộng đồng đảm bảo tính bảo toàn không gây phá hoại (Chỉ-đọc (Read-only)/Lên kế hoạch (Planning)).
- 🔴 **Risk**: Các skills sẽ có khả năng sửa đổi hệ thống hay tiến hành các đợt rà quét an ninh (Chỉ sử dụng khi được cấp phép).

_Kiểm tra [Skill Catalog](../CATALOG.md) để xem trọn danh sách đầy đủ._

---

## ❓ FAQ

**Q: Tôi có cần cài đặt tất cả hơn 700 skills hay không?**
A: Bạn chỉ phải thực hiện lệnh clone nguyên chiếc repo lại 1 lần; AI của bạn chỉ _đọc (reads)_ các skills khi bạn có nhu cầu triệu gọi (hoặc liên quan), bởi việc đó nhờ vậy tổng thể vẫn giữ được sự nhẹ nhàng (lightweight) cần thiết. Những **Starter Packs** theo danh mục trong file [BUNDLES.md](BUNDLES.md) là những gợi ý có giá trị lớn dành cho bạn nhằm mục tiêu khai phá xem những skills gì hoàn hảo với mục tiêu phát triển đang diễn ra—nhấn mạnh rằng tính năng đấy không làm quy trình tải về ban đầu rườm rà thêm.

**Q: Tôi có thể tạo skills của riêng mình được không?**
A: Chắc chắn rồi! Dùng skill **@skill-creator** theo để xây dựng cho mình tính năng cá nhân nào đó đó đi nha.

**Q: Có mất phí không thế?**
A: Có đấy, trả cho Giấy phép MIT. Mãi mãi Open Source.

---

## ⏭️ Các bước tiếp theo (Next Steps)

1. [Duyệt qua các Bundles](BUNDLES.md)
2. [Tham khảo các Ví dụ Thực tế](EXAMPLES.md)
3. [Hãy đóng góp thêm một Skill mới](../CONTRIBUTING.md)
