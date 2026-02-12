---
name: conductor-revert
description: Hoàn tác nhận biết Git theo đơn vị công việc logic (theo dõi, giai đoạn hoặc nhiệm vụ)
metadata:
  argument-hint: "[track-id | track-id:phase | track-id:task]"
---

# Hoàn tác Theo dõi (Revert Track)

Hoàn tác các thay đổi theo đơn vị công việc logic với nhận thức Git đầy đủ. Hỗ trợ hoàn tác toàn bộ theo dõi, các giai đoạn cụ thể hoặc các nhiệm vụ riêng lẻ.

## Sử dụng kỹ năng này khi

- Làm việc trên các nhiệm vụ hoặc quy trình làm việc hoàn tác theo dõi
- Cần hướng dẫn, thực tiễn tốt nhất hoặc danh sách kiểm tra cho hoàn tác theo dõi

## Không sử dụng kỹ năng này khi

- Nhiệm vụ không liên quan đến hoàn tác theo dõi
- Bạn cần một miền hoặc công cụ khác ngoài phạm vi này

## Hướng dẫn

- Làm rõ mục tiêu, ràng buộc và đầu vào cần thiết.
- Áp dụng các thực tiễn tốt nhất có liên quan và xác thực kết quả.
- Cung cấp các bước có thể hành động và xác minh.
- Nếu cần các ví dụ chi tiết, hãy mở `resources/implementation-playbook.md`.

## Kiểm tra Trước chuyến bay (Pre-flight Checks)

1. Xác minh Conductor đã được khởi tạo:
   - Kiểm tra `conductor/tracks.md` tồn tại
   - Nếu thiếu: Hiển thị lỗi và đề xuất chạy `/conductor:setup` trước

2. Xác minh kho lưu trữ git:
   - Chạy `git status` để xác nhận kho lưu trữ git
   - Kiểm tra các thay đổi chưa được cam kết
   - Nếu tồn tại các thay đổi chưa được cam kết:

     ```
     WARNING: Uncommitted changes detected

     Files with changes:
     {list of files}

     Options:
     1. Stash changes and continue
     2. Commit changes first
     3. Cancel revert
     ```

3. Xác minh git đủ sạch để hoàn tác:
   - Không có hợp nhất (merge) đang diễn ra
   - Không có rebase đang diễn ra
   - Nếu tìm thấy sự cố: Dừng và giải thích các bước giải quyết

## Lựa chọn Mục tiêu

### Nếu đối số được cung cấp:

Phân tích định dạng đối số:

**Toàn bộ theo dõi:** `{trackId}`

- Ví dụ: `auth_20250115`
- Hoàn tác tất cả các cam kết cho toàn bộ theo dõi

**Giai đoạn cụ thể:** `{trackId}:phase{N}`

- Ví dụ: `auth_20250115:phase2`
- Hoàn tác các cam kết cho giai đoạn N và tất cả các giai đoạn tiếp theo

**Nhiệm vụ cụ thể:** `{trackId}:task{X.Y}`

- Ví dụ: `auth_20250115:task2.3`
- Hoàn tác các cam kết cho chỉ nhiệm vụ X.Y

### Nếu không có đối số:

Hiển thị menu lựa chọn có hướng dẫn:

```
Bạn muốn hoàn tác cái gì?

Hiện đang tiến hành:
1. [~] Task 2.3 in dashboard_20250112 (gần đây nhất)

Gần đây đã hoàn thành:
2. [x] Task 2.2 in dashboard_20250112 (1 giờ trước)
3. [x] Phase 1 in dashboard_20250112 (3 giờ trước)
4. [x] Full track: auth_20250115 (hôm qua)

Tùy chọn:
5. Nhập tham chiếu cụ thể (track:phase hoặc track:task)
6. Hủy bỏ

Chọn tùy chọn:
```

## Khám phá Cam kết (Commit Discovery)

### Để Hoàn tác Nhiệm vụ

1. Tìm kiếm git log cho các cam kết cụ thể theo nhiệm vụ:

   ```bash
   git log --oneline --grep="{trackId}" --grep="Task {X.Y}" --all-match
   ```

2. Cũng tìm cam kết cập nhật plan.md:

   ```bash
   git log --oneline --grep="mark task {X.Y} complete" --grep="{trackId}" --all-match
   ```

3. Thu thập tất cả các SHA cam kết khớp

### Để Hoàn tác Giai đoạn

1. Xác định phạm vi nhiệm vụ cho giai đoạn bằng cách đọc plan.md
2. Tìm kiếm tất cả các cam kết nhiệm vụ trong giai đoạn đó:

   ```bash
   git log --oneline --grep="{trackId}" | grep -E "Task {N}\.[0-9]"
   ```

3. Tìm cam kết xác minh giai đoạn nếu tồn tại
4. Tìm tất cả các cam kết cập nhật plan.md cho các nhiệm vụ giai đoạn
5. Thu thập tất cả các SHA cam kết khớp theo thứ tự thời gian

### Để Hoàn tác Toàn bộ Theo dõi

1. Tìm TẤT CẢ các cam kết đề cập đến theo dõi:

   ```bash
   git log --oneline --grep="{trackId}"
   ```

2. Tìm các cam kết tạo theo dõi:

   ```bash
   git log --oneline -- "conductor/tracks/{trackId}/"
   ```

3. Thu thập tất cả các SHA cam kết khớp theo thứ tự thời gian

## Hiển thị Kế hoạch Thực thi

Trước bất kỳ hoạt động hoàn tác nào, hiển thị kế hoạch đầy đủ:

```
================================================================================
                           REVERT EXECUTION PLAN
================================================================================

Mục tiêu: {description of what's being reverted}

Các cam kết cần hoàn tác (theo thứ tự thời gian ngược):
  1. abc1234 - feat: add chart rendering (dashboard_20250112)
  2. def5678 - chore: mark task 2.3 complete (dashboard_20250112)
  3. ghi9012 - feat: add data hooks (dashboard_20250112)
  4. jkl3456 - chore: mark task 2.2 complete (dashboard_20250112)

Các tệp sẽ bị ảnh hưởng:
  - src/components/Dashboard.tsx (modified)
  - src/hooks/useData.ts (will be deleted - was created in these commits)
  - conductor/tracks/dashboard_20250112/plan.md (modified)

Cập nhật kế hoạch:
  - Task 2.2: [x] -> [ ]
  - Task 2.3: [~] -> [ ]

================================================================================
                              !! CẢNH BÁO !!
================================================================================

Hoạt động này sẽ:
- Tạo {N} cam kết hoàn tác
- Sửa đổi {M} tệp
- Đặt lại {P} nhiệm vụ về trạng thái chờ xử lý

Điều này KHÔNG THỂ dễ dàng hoàn tác mà không có sự can thiệp thủ công.

================================================================================

Gõ 'YES' để tiếp tục, hoặc bất cứ điều gì khác để hủy bỏ:
```

**QUAN TRỌNG: Yêu cầu xác nhận 'YES' rõ ràng. Không tiến hành khi 'y', 'yes', hoặc enter.**

## Thực thi Hoàn tác

Thực thi hoàn tác theo thứ tự thời gian ngược (mới nhất trước):

```
Đang thực thi kế hoạch hoàn tác...

[1/4] Reverting abc1234...
      git revert --no-edit abc1234
      ✓ Success

[2/4] Reverting def5678...
      git revert --no-edit def5678
      ✓ Success

[3/4] Reverting ghi9012...
      git revert --no-edit ghi9012
      ✓ Success

[4/4] Reverting jkl3456...
      git revert --no-edit jkl3456
      ✓ Success
```

### Khi Xung đột Hợp nhất

Nếu bất kỳ hoàn tác nào tạo ra xung đột hợp nhất:

```
================================================================================
                           MERGE CONFLICT DETECTED
================================================================================

Xung đột xảy ra trong khi hoàn tác: {sha} - {message}

Các tệp bị xung đột:
  - src/components/Dashboard.tsx

Tùy chọn:
1. Hiển thị chi tiết xung đột
2. Hủy bỏ trình tự hoàn tác (giữ các hoàn tác đã hoàn thành)
3. Mở hướng dẫn giải quyết thủ công

QUAN TRỌNG: Các hoàn tác 1-{N} đã được hoàn thành. Bạn có thể cần phải giải quyết
thủ công xung đột này trước khi tiếp tục hoặc hoàn tác hoàn toàn trình tự hoàn tác.

Chọn tùy chọn:
```

**DỪNG ngay lập tức khi có bất kỳ xung đột nào. Không cố gắng giải quyết tự động.**

## Cập nhật Plan.md

Sau khi git revert thành công, cập nhật plan.md:

1. Đọc plan.md hiện tại
2. Đối với mỗi nhiệm vụ đã hoàn tác, thay đổi đánh dấu:
   - `[x]` -> `[ ]`
   - `[~]` -> `[ ]`
3. Ghi plan.md đã cập nhật
4. Cập nhật metadata.json:
   - Giảm `tasks.completed`
   - Cập nhật `status` nếu cần
   - Cập nhật dấu thời gian `updated`

**KHÔNG cam kết các thay đổi plan.md** - chúng là một phần của hoạt động hoàn tác

## Cập nhật Trạng thái Theo dõi

### Nếu hoàn tác toàn bộ theo dõi:

- Trong tracks.md: Thay đổi `[x]` hoặc `[~]` thành `[ ]`
- Xem xét đề nghị xóa hoàn toàn thư mục theo dõi

### Nếu hoàn tác về trạng thái chưa hoàn thành:

- Trong tracks.md: Đảm bảo được đánh dấu là `[~]` nếu hoàn thành một phần, `[ ]` nếu hoàn tác hoàn toàn

## Xác minh

Sau khi hoàn thành hoàn tác:

```
================================================================================
                           REVERT COMPLETE
================================================================================

Tóm tắt:
  - Reverted {N} commits
  - Reset {P} tasks to pending
  - {M} files affected

Git log bây giờ hiển thị:
  {recent commit history}

Trạng thái Plan.md:
  - Task 2.2: [ ] Pending
  - Task 2.3: [ ] Pending

================================================================================

Xác minh việc hoàn tác đã thành công:
  1. Chạy kiểm thử: {test command}
  2. Kiểm tra ứng dụng: {relevant check}

Nếu tìm thấy sự cố, bạn có thể cần phải:
  - Sửa xung đột thủ công
  - Triển khai lại các nhiệm vụ đã hoàn tác
  - Sử dụng 'git revert HEAD~{N}..HEAD' để hoàn tác các hoàn tác

================================================================================
```

## Quy tắc An toàn

1. **KHÔNG BAO GIỜ sử dụng `git reset --hard`** - Chỉ sử dụng `git revert`
2. **KHÔNG BAO GIỜ sử dụng `git push --force`** - Chỉ các hoạt động đẩy an toàn
3. **KHÔNG BAO GIỜ tự động giải quyết xung đột** - Luôn dừng lại để có sự can thiệp của con người
4. **LUÔN hiển thị kế hoạch đầy đủ** - Người dùng phải thấy chính xác những gì sẽ xảy ra
5. **YÊU CẦU 'YES' rõ ràng** - Không phải 'y', không phải enter, chỉ 'YES'
6. **DỪNG lại khi có BẤT KỲ lỗi nào** - Không cố gắng tiếp tục qua các thất bại
7. **BẢO TỒN lịch sử** - Cam kết hoàn tác được ưu tiên hơn viết lại lịch sử

## Các Trường hợp Biên

### Theo dõi Chưa bao giờ được Cam kết

```
Không tìm thấy cam kết nào cho theo dõi: {trackId}

Theo dõi tồn tại nhưng không có cam kết liên quan. Điều này có thể có nghĩa là:
- Việc triển khai chưa bao giờ bắt đầu
- Các cam kết sử dụng định dạng khác

Tùy chọn:
1. Chỉ xóa thư mục theo dõi
2. Hủy bỏ
```

### Các Cam kết Đã được Hoàn tác

```
Một số cam kết dường như đã được hoàn tác:
  - abc1234 đã được hoàn tác bởi xyz9876

Tùy chọn:
1. Bỏ qua các cam kết đã hoàn tác
2. Hủy bỏ và điều tra
```

### Remote Đã được Đẩy (Pushed)

```
CẢNH BÁO: Một số cam kết đã được đẩy lên remote

Cam kết trên remote:
  - abc1234 (origin/main)
  - def5678 (origin/main)

Việc hoàn tác sẽ tạo ra các cam kết hoàn tác mới mà bạn sẽ cần phải đẩy.
Đây là cách tiếp cận an toàn (không yêu cầu buộc đẩy).

Tiếp tục với hoàn tác? (YES/no):
```

## Hoàn tác việc Hoàn tác

Nếu người dùng cần hoàn tác chính hoạt động hoàn tác:

```
Để hoàn tác hoạt động hoàn tác này:

  git revert HEAD~{N}..HEAD

Điều này sẽ tạo ra các cam kết mới khôi phục các thay đổi đã hoàn tác.

Ngoài ra, nếu chưa đẩy:
  git reset --soft HEAD~{N}
  git checkout -- .

(Sử dụng cẩn thận - điều này loại bỏ các cam kết hoàn tác)
```
