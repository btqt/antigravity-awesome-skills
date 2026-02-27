# 🏆 Quality Bar & Validation Standards

Để biến đổi **Antigravity Awesome Skills** từ một bộ sưu tập các scripts thành một platform đáng tin cậy, mỗi skill đều phải đáp ứng một tiêu chuẩn cụ thể về chất lượng và độ an toàn.

## Huy hiệu "Validated" ✅

Một skill chỉ giành được huy hiệu "Validated" nếu nó vượt qua được **5 automated checks** sau đây:

### 1. Tính toàn vẹn của Metadata (Metadata Integrity)

Phần frontmatter của file `SKILL.md` phải là một YAML hợp lệ và chứa:

- `name`: Kebab-case, khớp với tên folder.
- `description`: Dưới 200 ký tự, thể hiện rõ giá trị mang lại (value prop).
- `risk`: Là một trong các giá trị `[none, safe, critical, offensive]`.
- `source`: URL dẫn đến source gốc (hoặc "self" nếu đây là bản gốc).

### 2. Các Triggers rõ ràng ("When to use")

Skill BẮT BUỘC (MUST) có một phần trình bày rõ ràng về việc khi nào cần trigger nó.

- **Tốt**: "Sử dụng khi user yêu cầu debug một React component."
- **Kém**: "Skill này giúp bạn với code."
  Các headings được chấp nhận: `## When to Use`, `## Use this skill when`, `## When to Use This Skill`.

### 3. Phân loại An toàn & Rủi ro (Safety & Risk Classification)

Mỗi một skill phải khai báo risk level của nó:

- 🟢 **none**: Hoàn toàn là văn bản/suy luận (text/reasoning) (ví dụ: Brainstorming).
- 🔵 **safe**: Đọc file, chạy các commands an toàn (ví dụ: Linter).
- 🟠 **critical**: Sửa đổi state, xóa files, đẩy code (pushes) lên prod (ví dụ: Git Push).
- 🔴 **offensive**: Các tools thuần Pentesting/Red Team. **BẮT BUỘC (MUST)** có cảnh báo "Authorized Use Only" (Chỉ sử dụng khi được cấp phép).

### 4. Các Examples có thể Copy-Paste (Copy-Pasteable Examples)

Phải có ít nhất một code block hoặc ví dụ tương tác (interaction example) mà user (hoặc agent) có thể sử dụng được ngay lập tức.

### 5. Những Hạn chế Rõ ràng (Explicit Limitations)

Một danh sách liệt kê các edge cases (trường hợp ngoại lệ) đã biết hoặc những thứ mà skill _không thể_ làm được.

- _Ví dụ_: "Không hoạt động trên Windows nếu không có WSL."

---

## Support Levels (Các cấp độ hỗ trợ)

Chúng tôi cũng phân loại các skills dựa trên đối tượng duy trì (maintains) chúng:

| Level         | Badge | Ý nghĩa                                                                                       |
| :------------ | :---- | :-------------------------------------------------------------------------------------------- |
| **Official**  | 🟣    | Được duy trì bởi core team. Độ tin cậy cao.                                                   |
| **Community** | ⚪    | Được đóng góp từ ecosystem. Cung cấp sự hỗ trợ trong khả năng tốt nhất (Best effort support). |
| **Verified**  | ✨    | Skill từ cộng đồng (community skill) mà đã vượt qua quá trình review thủ công chuyên sâu.     |

---

## Làm thế nào để Validate Skill của bạn

Validator tiêu chuẩn là `scripts/validate_skills.py`. Hãy chạy lệnh `npm run validate` (hoặc `npm run validate:strict`) trước khi gửi lên một yêu cầu thay đổi (submitting a PR):

```bash
npm run validate       # soft mode (chỉ cảnh báo - warnings only)
npm run validate:strict  # strict mode (hệ thống CI sử dụng quyền này)
```
