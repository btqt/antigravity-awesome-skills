# ❓ Câu hỏi thường gặp (FAQ)

**Bạn có câu hỏi?** Bạn không hề đơn độc! Dưới đây là câu trả lời cho những câu hỏi thường gặp nhất về Antigravity Awesome Skills.

---

## 🎯 Câu hỏi chung (General Questions)

### Chính xác thì "skills" là gì?

Skills là các tệp chỉ dẫn chuyên biệt giúp hướng dẫn các AI assistants cách thức xử lý các tác vụ cụ thể. Hãy xem chúng như là các module kiến thức chuyên môn mà AI của bạn có thể tải theo yêu cầu (load on-demand).
**Một ví dụ so sánh đơn giản:** Giống như việc bạn có thể tham vấn các chuyên gia khác nhau (luật sư, bác sĩ, thợ máy), những skills này giúp AI của bạn trở thành chuyên gia trong các lĩnh vực khác nhau khi bạn cần đến chúng.

### Tôi có cần cài đặt tất cả hơn 700 skills không?

**Không!** Khi bạn clone repository, tất cả các skills đều có sẵn, nhưng AI của bạn chỉ tải chúng khi bạn gọi chúng một cách rõ ràng với cú pháp `@skill-name`.
Nó giống như việc sở hữu một thư viện - tất cả sách đều ở đó, nhưng bạn chỉ đọc những cuốn nào bạn cần.
**Pro Tip:** Sử dụng [Starter Packs](BUNDLES.md) để cài đặt chỉ những thứ phù hợp với vai trò của bạn.

### Sự khác biệt giữa Bundles và Workflows là gì?

- **Bundles** là những đề xuất được tuyển chọn, nhóm theo vai trò hoặc domain.
- **Workflows** là các sổ tay (playbooks) hướng dẫn thực thi có thứ tự nhằm đem lại kết quả cụ thể.

Sử dụng bundles khi bạn đang quyết định xem _những skills nào_ cần bao gồm. Sử dụng workflows khi bạn cần sự _thực thi từng bước_.

Bắt đầu từ:

- [BUNDLES.md](BUNDLES.md)
- [WORKFLOWS.md](WORKFLOWS.md)

### Những AI tools nào tương thích với các skills này?

- ✅ **Claude Code** (Anthropic CLI)
- ✅ **Gemini CLI** (Google)
- ✅ **Codex CLI** (OpenAI)
- ✅ **Cursor** (AI IDE)
- ✅ **Antigravity IDE**
- ✅ **OpenCode**
- ⚠️ **GitHub Copilot** (hỗ trợ một phần thông qua thao tác copy-paste)

### Các skills này có được sử dụng miễn phí không?

**Có!** Repository này được cấp phép theo MIT License.

- ✅ Miễn phí cho sử dụng cá nhân
- ✅ Miễn phí cho mục đích thương mại
- ✅ Bạn có thể sửa đổi chúng

### Các skills có hoạt động offline không?

Bản thân các file skill được lưu trữ trực tiếp trên máy tính của bạn (locally), nhưng AI assistant của bạn thì lại cần kết nối internet để hoạt động.

---

## 🔒 Security & Trust (V4 Update)

### Các nhãn Rủi ro (Risk Labels) có nghĩa là gì?

Chúng tôi phân loại các skills để bạn biết mình đang khởi chạy điều gì:

- ⚪ **Safe (White/Blue)**: Các skills chỉ-đọc (Read-only), lập kế hoạch (planning), hoặc các skills vô hại.
- 🔴 **Risk (Red)**: Các skills thực hiện sửa đổi các files (như xóa), sử dụng trình dò quét mạng (network scanners) hoặc thực hiện các hành động phá hủy. **Sử dụng thật cẩn trọng.**
- 🟣 **Official (Purple)**: Được duy trì bởi các đơn vị cung cấp đáng tin cậy (Anthropic, DeepMind, v.v.).

### Những skills này có thể hack máy tính của tôi không?

**Không.** Skills chỉ là các tệp văn bản. Tuy nhiên, chúng _chỉ thị_ cho AI thực thi các câu lệnh (commands). Nếu một skill ra lệnh "xóa tất cả các file", một AI tận tụy có thể cố gắng thực hiện điều đó.
_Luôn kiểm tra nhãn Risk và review lại code._

---

## 📦 Cài đặt và Thiết lập (Installation & Setup)

### Tôi nên cài đặt skills ở đâu?

Đường dẫn chung (universal path) hoạt động tốt với hầu hết các tools là `.agent/skills/`.

**Sử dụng npx:** `npx antigravity-awesome-skills` (hoặc `npx github:sickn33/antigravity-awesome-skills` nếu bạn gặp lỗi 404).

**Sử dụng git clone:**

```bash
git clone https://github.com/sickn33/antigravity-awesome-skills.git .agent/skills
```

**Các đường dẫn cụ thể theo Tool:**

- Claude Code: `.claude/skills/`
- Gemini CLI: `.gemini/skills/`
- Codex CLI: `.codex/skills/`
- Cursor: `.cursor/skills/` hoặc nằm ở thư mục gốc của project (project root)

### Điều này có hoạt động với Windows không?

**Có**, nhưng một số skills loại "Official" sử dụng **symlinks**, thứ mà Windows thường xử lý rất kém theo thiết lập mặc định.
Chạy git với lệnh sau:

```bash
git clone -c core.symlinks=true https://github.com/sickn33/antigravity-awesome-skills.git .agent/skills
```

Hoặc bật chế độ "Developer Mode" trong cài đặt Windows Settings.

### Làm thế nào để cập nhật skills?

Điều hướng tới thư mục chứa skills của bạn và tải mới (pull) những thay đổi (changes) mới nhất:

```bash
cd .agent/skills
git pull origin main
```

---

## 🛠️ Sử dụng Skills (Using Skills)

### Làm thế nào để gọi (invoke) một skill?

Sử dụng ký hiệu `@` theo sau là tên skill:

```bash
@brainstorming help me design a todo app
```

### Tôi có thể dùng nhiều skills cùng một lúc không?

**Có!** Bạn có thể gọi (invoke) một lúc nhiều skills:

```bash
@brainstorming help me design this, then use @writing-plans to create a task list.
```

### Làm sao tôi biết nên sử dụng skill nào?

1. **Khám phá danh mục (catalog)**: Hãy kiểm tra [Skill Catalog](../CATALOG.md).
2. **Tìm kiếm (Search)**: `ls skills/ | grep "keyword"`
3. **Hỏi AI của bạn**: "Bạn có những skills nào dành cho việc testing?"

---

## 🏗️ Gỡ rối (Troubleshooting)

### AI assistant của tôi không nhận diện được skills

**Nguyên nhân có thể:**

1. **Sai đường dẫn cài đặt**: Kiểm tra tài liệu về tool của bạn. Hãy thử dùng `.agent/skills/`.
2. **Cần Khởi động lại**: Khởi động lại AI/IDE của bạn sau khi cài đặt.
3. **Lỗi chính tả (Typos)**: Bạn có đã gõ `@brain-storming` thay cho `@brainstorming`?

### Một skill đưa ra lời khuyên sai hoặc lỗi thời

Vui lòng [Mở một issue](https://github.com/sickn33/antigravity-awesome-skills/issues)!
Cung cấp thông tin:

- Bị ở skill nào
- Đã xảy ra lỗi gì
- Thay vì vậy thì điều gì đáng lẽ nên diễn ra

---

## 🤝 Contribution

### Tôi mới biết đến open source. Tôi có thể đóng góp không?

**Chắc chắn rồi!** Chúng tôi luôn chào đón các beginners.

- Sửa lỗi typos
- Thêm examples
- Cải thiện tài liệu (docs)
  Hãy xem qua file [CONTRIBUTING.md](../CONTRIBUTING.md) để biết thêm các hướng dẫn chi tiết.

### Yêu cầu PR của tôi bị trượt kiểm tra chất lượng "Quality Bar" check. Tại sao?

Phiên bản V4 giới thiệu tính năng kiểm soát chất lượng tự động. Skill của bạn có thể đã bị thiếu mất:

1. Một lời mô tả `description` hợp lệ.
2. Các ví dụ cách sử dụng (Usage examples).
   Chạy lệnh `python3 scripts/validate_skills.py` locally để kiểm tra trước khi bạn thực hiện push.

### Tôi có thể cập nhật một skill loại "Official" không?

**Không.** Các skills loại Official (trong thư mục `skills/official/`) được sao lưu (mirrored) từ các vendors. Thay vì việc cập nhật, hãy mở một issue.

---

## 💡 Pro Tips

- Bắt đầu với `@brainstorming` trước khi xây dựng thêm bất kỳ thứ gì mới
- Dùng `@systematic-debugging` khi bị mắc kẹt với các bugs
- Hãy dùng thử `@test-driven-development` nhằm có được chất lượng code tốt hơn
- Khám phá `@skill-creator` để tự tạo cho bản thân những skills riêng

**Vẫn còn bối rối?** [Hãy mở một cuộc thảo luận (Open a discussion)](https://github.com/sickn33/antigravity-awesome-skills/discussions) và chúng tôi sẽ trợ giúp cho bạn nhé! 🙌
