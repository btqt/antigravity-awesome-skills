---
name: conductor-implement
description: Thực hiện các nhiệm vụ từ kế hoạch triển khai của một theo dõi (track) theo quy trình làm việc TDD
metadata:
  argument-hint: "[track-id] [--task X.Y] [--phase N]"
---

# Triển khai Theo dõi (Implement Track)

Thực hiện các nhiệm vụ từ kế hoạch triển khai của một theo dõi, tuân theo các quy tắc quy trình làm việc được xác định trong `conductor/workflow.md`.

## Sử dụng kỹ năng này khi

- Làm việc trên các nhiệm vụ hoặc quy trình triển khai theo dõi
- Cần hướng dẫn, thực tiễn tốt nhất hoặc danh sách kiểm tra cho triển khai theo dõi

## Không sử dụng kỹ năng này khi

- Nhiệm vụ không liên quan đến triển khai theo dõi
- Bạn cần một miền hoặc công cụ khác ngoài phạm vi này

## Hướng dẫn

- Làm rõ mục tiêu, ràng buộc và đầu vào cần thiết.
- Áp dụng các thực tiễn tốt nhất có liên quan và xác thực kết quả.
- Cung cấp các bước có thể hành động và xác minh.
- Nếu cần các ví dụ chi tiết, hãy mở `resources/implementation-playbook.md`.

## Kiểm tra Trước chuyến bay (Pre-flight Checks)

1. Xác minh Conductor đã được khởi tạo:
   - Kiểm tra `conductor/product.md` tồn tại
   - Kiểm tra `conductor/workflow.md` tồn tại
   - Kiểm tra `conductor/tracks.md` tồn tại
   - Nếu thiếu: Hiển thị lỗi và đề xuất chạy `/conductor:setup` trước

2. Tải cấu hình quy trình làm việc:
   - Đọc `conductor/workflow.md`
   - Phân tích mức độ nghiêm ngặt TDD
   - Phân tích chiến lược cam kết
   - Phân tích các quy tắc điểm kiểm tra xác minh

## Lựa chọn Theo dõi

### Nếu đối số được cung cấp:

- Xác thực theo dõi tồn tại: `conductor/tracks/{argument}/plan.md`
- Nếu không tìm thấy: Tìm kiếm các kết quả khớp một phần, đề xuất sửa chữa

### Nếu không có đối số:

1. Đọc `conductor/tracks.md`
2. Phân tích các theo dõi chưa hoàn thành (trạng thái `[ ]` hoặc `[~]`)
3. Hiển thị menu lựa chọn:

   ```
   Chọn một theo dõi để triển khai:

   Đang tiến hành:
   1. [~] auth_20250115 - User Authentication (Phase 2, Task 3)

   Đang chờ xử lý:
   2. [ ] nav-fix_20250114 - Navigation Bug Fix
   3. [ ] dashboard_20250113 - Dashboard Feature

   Nhập số hoặc ID theo dõi:
   ```

## Tải Bối cảnh

Tải tất cả bối cảnh liên quan để triển khai:

1. Tài liệu theo dõi:
   - `conductor/tracks/{trackId}/spec.md` - Yêu cầu
   - `conductor/tracks/{trackId}/plan.md` - Danh sách nhiệm vụ
   - `conductor/tracks/{trackId}/metadata.json` - Trạng thái tiến độ

2. Bối cảnh dự án:
   - `conductor/product.md` - Hiểu biết sản phẩm
   - `conductor/tech-stack.md` - Ràng buộc kỹ thuật
   - `conductor/workflow.md` - Quy tắc quy trình

3. Kiểu mã (nếu tồn tại):
   - `conductor/code_styleguides/{language}.md`

## Cập nhật Trạng thái Theo dõi

Cập nhật theo dõi thành đang tiến hành:

1. Trong `conductor/tracks.md`:
   - Thay đổi `[ ]` thành `[~]` cho theo dõi này

2. Trong `conductor/tracks/{trackId}/metadata.json`:
   - Đặt `status: "in_progress"`
   - Cập nhật dấu thời gian `updated`

## Vòng lặp Thực hiện Nhiệm vụ

Đối với mỗi nhiệm vụ chưa hoàn thành trong plan.md (được đánh dấu bằng `[ ]`):

### 1. Xác định Nhiệm vụ

Phân tích plan.md để tìm nhiệm vụ chưa hoàn thành tiếp theo:

- Tìm các dòng khớp với `- [ ] Task X.Y: {description}`
- Theo dõi giai đoạn hiện tại từ cấu trúc

### 2. Bắt đầu Nhiệm vụ

Đánh dấu nhiệm vụ là đang tiến hành:

- Cập nhật plan.md: Thay đổi `[ ]` thành `[~]` cho nhiệm vụ hiện tại
- Thông báo: "Starting Task X.Y: {description}"

### 3. Quy trình làm việc TDD (nếu TDD được bật trong workflow.md)

**Giai đoạn Đỏ - Viết Kiểm thử Thất bại:**

```
Following TDD workflow for Task X.Y...

Step 1: Writing failing test
```

- Tạo tệp kiểm thử nếu cần
- Viết (các) kiểm thử cho chức năng nhiệm vụ
- Chạy các kiểm thử để xác nhận chúng thất bại
- Nếu các kiểm thử vượt qua bất ngờ: DỪNG, điều tra

**Giai đoạn Xanh - Triển khai:**

```
Step 2: Implementing minimal code to pass test
```

- Viết mã tối thiểu để làm cho kiểm thử vượt qua
- Chạy các kiểm thử để xác nhận chúng vượt qua
- Nếu các kiểm thử thất bại: Gỡ lỗi và sửa

**Giai đoạn Tái cấu trúc:**

```
Step 3: Refactoring while keeping tests green
```

- Làm sạch mã
- Chạy các kiểm thử để đảm bảo vẫn đang vượt qua

### 4. Quy trình làm việc Không TDD (nếu TDD không nghiêm ngặt)

- Triển khai nhiệm vụ trực tiếp
- Chạy bất kỳ kiểm thử hiện có nào
- Xác minh thủ công khi cần thiết

### 5. Hoàn thành Nhiệm vụ

**Cam kết các thay đổi** (tuân theo chiến lược cam kết từ workflow.md):

```bash
git add -A
git commit -m "{commit_prefix}: {task description} ({trackId})"
```

**Cập nhật plan.md:**

- Thay đổi `[~]` thành `[x]` cho nhiệm vụ đã hoàn thành
- Cam kết cập nhật kế hoạch:

```bash
git add conductor/tracks/{trackId}/plan.md
git commit -m "chore: mark task X.Y complete ({trackId})"
```

**Cập nhật metadata.json:**

- Tăng `tasks.completed`
- Cập nhật dấu thời gian `updated`

### 6. Kiểm tra Hoàn thành Giai đoạn

Sau mỗi nhiệm vụ, kiểm tra xem giai đoạn đã hoàn thành chưa:

- Phân tích plan.md cho cấu trúc giai đoạn
- Nếu tất cả các nhiệm vụ trong giai đoạn hiện tại là `[x]`:

**Chạy xác minh giai đoạn:**

```
Phase {N} complete. Running verification...
```

- Thực hiện các nhiệm vụ xác minh được liệt kê cho giai đoạn
- Chạy bộ kiểm thử đầy đủ: `npm test` / `pytest` / v.v.

**Báo cáo và chờ phê duyệt:**

```
Phase {N} Verification Results:
- All phase tasks: Complete
- Tests: {passing/failing}
- Verification: {pass/fail}

Approve to continue to Phase {N+1}?
1. Yes, continue
2. No, there are issues to fix
3. Pause implementation
```

**QUAN TRỌNG: Đợi sự phê duyệt rõ ràng của người dùng trước khi tiến hành giai đoạn tiếp theo.**

## Xử lý Lỗi Trong quá trình Triển khai

### Khi Công cụ Thất bại

```
ERROR: {tool} failed with: {error message}

Options:
1. Retry the operation
2. Skip this task and continue
3. Pause implementation
4. Revert current task changes
```

- DỪNG và trình bày các tùy chọn
- KHÔNG tự động tiếp tục

### Khi Kiểm thử Thất bại

```
TESTS FAILING after Task X.Y

Failed tests:
- {test name}: {failure reason}

Options:
1. Attempt to fix
2. Rollback task changes
3. Pause for manual intervention
```

### Khi Git Thất bại

```
GIT ERROR: {error message}

This may indicate:
- Uncommitted changes from outside Conductor
- Merge conflicts
- Permission issues

Options:
1. Show git status
2. Attempt to resolve
3. Pause for manual intervention
```

## Hoàn thành Theo dõi

Khi tất cả các giai đoạn và nhiệm vụ hoàn tất:

### 1. Xác minh Cuối cùng

```
All tasks complete. Running final verification...
```

- Chạy bộ kiểm thử đầy đủ
- Kiểm tra tất cả các tiêu chí chấp nhận từ spec.md
- Tạo báo cáo xác minh

### 2. Cập nhật Trạng thái Theo dõi

Trong `conductor/tracks.md`:

- Thay đổi `[~]` thành `[x]` cho theo dõi này
- Cập nhật cột "Updated"

Trong `conductor/tracks/{trackId}/metadata.json`:

- Đặt `status: "complete"`
- Đặt `phases.completed` thành tổng số
- Đặt `tasks.completed` thành tổng số
- Cập nhật dấu thời gian `updated`

Trong `conductor/tracks/{trackId}/plan.md`:

- Cập nhật trạng thái đầu trang thành `[x] Complete`

### 3. Đề nghị Đồng bộ Tài liệu

```
Track complete! Would you like to sync documentation?

This will update:
- conductor/product.md (if new features added)
- conductor/tech-stack.md (if new dependencies added)
- README.md (if applicable)

1. Yes, sync documentation
2. No, skip
```

### 4. Đề nghị Dọn dẹp

```
Track {trackId} is complete.

Cleanup options:
1. Archive - Move to conductor/tracks/_archive/
2. Delete - Remove track directory
3. Keep - Leave as-is
```

### 5. Tóm tắt Hoàn thành

```
Track Complete: {track title}

Summary:
- Track ID: {trackId}
- Phases completed: {N}/{N}
- Tasks completed: {M}/{M}
- Commits created: {count}
- Tests: All passing

Next steps:
- Run /conductor:status to see project progress
- Run /conductor:new-track for next feature
```

## Theo dõi Tiến độ

Duy trì tiến độ trong `metadata.json` xuyên suốt:

```json
{
  "id": "auth_20250115",
  "title": "User Authentication",
  "type": "feature",
  "status": "in_progress",
  "created": "2025-01-15T10:00:00Z",
  "updated": "2025-01-15T14:30:00Z",
  "current_phase": 2,
  "current_task": "2.3",
  "phases": {
    "total": 3,
    "completed": 1
  },
  "tasks": {
    "total": 12,
    "completed": 7
  },
  "commits": [
    "abc1234: feat: add login form (auth_20250115)",
    "def5678: feat: add password validation (auth_20250115)"
  ]
}
```

## Tiếp tục lại

Nếu việc triển khai bị tạm dừng và tiếp tục lại:

1. Tải `metadata.json` cho trạng thái hiện tại
2. Tìm nhiệm vụ hiện tại từ trường `current_task`
3. Kiểm tra xem nhiệm vụ có phải là `[~]` trong plan.md không
4. Hỏi người dùng:

   ```
   Resuming track: {title}

   Last task in progress: Task {X.Y}: {description}

   Options:
   1. Continue from where we left off
   2. Restart current task
   3. Show progress summary first
   ```

## Các Quy tắc Quan trọng

1. **KHÔNG BAO GIỜ bỏ qua các điểm kiểm tra xác minh** - Luôn đợi người dùng phê duyệt giữa các giai đoạn
2. **DỪNG lại khi có bất kỳ thất bại nào** - Không cố gắng tiếp tục qua các lỗi
3. **Tuân thủ quy trình làm việc.md nghiêm ngặt** - TDD, chiến lược cam kết và các quy tắc xác minh là bắt buộc
4. **Giữ plan.md được cập nhật** - Trạng thái nhiệm vụ phải phản ánh tiến độ thực tế
5. **Cam kết thường xuyên** - Mỗi khi hoàn thành nhiệm vụ nên được cam kết
6. **Theo dõi tất cả các cam kết** - Ghi lại băm cam kết trong metadata.json để có thể quay lại
