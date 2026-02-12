---
name: create-pr
description: "Tạo các yêu cầu kéo tuân theo các quy ước của Sentry. Sử dụng khi mở PR, viết mô tả PR hoặc chuẩn bị thay đổi để xem xét. Tuân theo hướng dẫn xem xét mã của Sentry."
source: "https://github.com/getsentry/skills/tree/main/plugins/sentry-skills/skills/create-pr"
risk: safe
---

# Tạo Yêu Cầu Kéo (Create Pull Request)

Tạo các yêu cầu kéo tuân theo các thực tiễn kỹ thuật của Sentry.

## Khi nào Sử dụng Kỹ năng Này

Sử dụng kỹ năng này khi:

- Mở các yêu cầu kéo
- Viết mô tả PR
- Chuẩn bị thay đổi để xem xét
- Tuân theo hướng dẫn xem xét mã của Sentry
- Tạo các PR tuân theo thực tiễn tốt nhất

**Yêu cầu**: GitHub CLI (`gh`) đã được xác thực và có sẵn.

## Điều kiện tiên quyết

Trước khi tạo PR, hãy đảm bảo tất cả các thay đổi đã được cam kết. Nếu có các thay đổi chưa được cam kết, hãy chạy kỹ năng `sentry-skills:commit` trước để cam kết chúng đúng cách.

```bash
# Check for uncommitted changes
git status --porcelain
```

Nếu đầu ra hiển thị bất kỳ thay đổi nào chưa được cam kết (các tệp đã sửa đổi, đã thêm hoặc chưa được theo dõi nên được bao gồm), hãy gọi kỹ năng `sentry-skills:commit` trước khi tiếp tục.

## Quy trình

### Bước 1: Xác minh Trạng thái Nhánh

```bash
# Detect the default branch
BASE=$(gh repo view --json defaultBranchRef --jq '.defaultBranchRef.name')

# Check current branch and status
git status
git log $BASE..HEAD --oneline
```

Đảm bảo:

- Tất cả các thay đổi đã được cam kết
- Nhánh đã cập nhật với remote
- Các thay đổi được rebase trên nhánh cơ sở nếu cần

### Bước 2: Phân tích Thay đổi

Xem xét những gì sẽ được bao gồm trong PR:

```bash
# See all commits that will be in the PR
git log $BASE..HEAD

# See the full diff
git diff $BASE...HEAD
```

Hiểu phạm vi và mục đích của tất cả các thay đổi trước khi viết mô tả.

### Bước 3: Viết Mô tả PR

Sử dụng cấu trúc này cho các mô tả PR (bỏ qua bất kỳ mẫu PR kho lưu trữ nào):

```markdown
<mô tả ngắn gọn về những gì PR làm>

<tại sao những thay đổi này đang được thực hiện - động lực>

<các cách tiếp cận thay thế đã được xem xét, nếu có>

<bất kỳ bối cảnh bổ sung nào người đánh giá cần>
```

**KHÔNG bao gồm:**

- Các phần "Kế hoạch kiểm tra"
- Danh sách hộp kiểm các bước kiểm tra
- Tóm tắt thừa thãi về sự khác biệt

**Nên bao gồm:**

- Giải thích rõ ràng về cái gì và tại sao
- Liên kết đến các vấn đề hoặc vé liên quan
- Bối cảnh không rõ ràng từ mã
- Ghi chú về các khu vực cụ thể cần xem xét kỹ lưỡng

### Bước 4: Tạo PR

```bash
gh pr create --draft --title "<type>(<scope>): <description>" --body "$(cat <<'EOF'
<description body here>
EOF
)"
```

**Định dạng tiêu đề** tuân theo các quy ước cam kết:

- `feat(scope): Add new feature`
- `fix(scope): Fix the bug`
- `ref: Refactor something`

## Ví dụ Mô tả PR

### PR Tính năng

```markdown
Add Slack thread replies for alert notifications

When an alert is updated or resolved, we now post a reply to the original
Slack thread instead of creating a new message. This keeps related
notifications grouped and reduces channel noise.

Previously considered posting edits to the original message, but threading
better preserves the timeline of events and works when the original message
is older than Slack's edit window.

Refs SENTRY-1234
```

### PR Sửa lỗi

```markdown
Handle null response in user API endpoint

The user endpoint could return null for soft-deleted accounts, causing
dashboard crashes when accessing user properties. This adds a null check
and returns a proper 404 response.

Found while investigating SENTRY-5678.

Fixes SENTRY-5678
```

### PR Tái cấu trúc

```markdown
Extract validation logic to shared module

Moves duplicate validation code from the alerts, issues, and projects
endpoints into a shared validator class. No behavior change.

This prepares for adding new validation rules in SENTRY-9999 without
duplicating logic across endpoints.
```

## Tham chiếu Vấn đề

Tham chiếu các vấn đề trong phần thân PR:

| Cú pháp               | Hiệu ứng                        |
| --------------------- | ------------------------------- |
| `Fixes #1234`         | Đóng vấn đề GitHub khi hợp nhất |
| `Fixes SENTRY-1234`   | Đóng vấn đề Sentry              |
| `Refs GH-1234`        | Liên kết mà không đóng          |
| `Refs LINEAR-ABC-123` | Liên kết vấn đề Linear          |

## Hướng dẫn

- **Một PR cho mỗi tính năng/sửa lỗi** - Đừng gộp các thay đổi không liên quan
- **Giữ cho PR có thể xem xét được** - Các PR nhỏ hơn nhận được đánh giá nhanh hơn, tốt hơn
- **Giải thích tại sao** - Mã hiển thị cái gì; mô tả giải thích tại sao
- **Đánh dấu WIP sớm** - Sử dụng PR nháp để nhận phản hồi sớm

## Chỉnh sửa PR Hiện có

Nếu bạn cần cập nhật PR sau khi tạo, hãy sử dụng `gh api` thay vì `gh pr edit`:

```bash
# Update PR description
gh api -X PATCH repos/{owner}/{repo}/pulls/PR_NUMBER -f body="$(cat <<'EOF'
Updated description here
EOF
)"

# Update PR title
gh api -X PATCH repos/{owner}/{repo}/pulls/PR_NUMBER -f title='new: Title here'

# Update both
gh api -X PATCH repos/{owner}/{repo}/pulls/PR_NUMBER \
  -f title='new: Title' \
  -f body='New description'
```

Lưu ý: `gh pr edit` hiện đang bị hỏng do việc ngừng sử dụng Projects (classic) của GitHub.

## Tài liệu tham khảo

- [Sentry Code Review Guidelines](https://develop.sentry.dev/engineering-practices/code-review/)
- [Sentry Commit Messages](https://develop.sentry.dev/engineering-practices/commit-messages/)
