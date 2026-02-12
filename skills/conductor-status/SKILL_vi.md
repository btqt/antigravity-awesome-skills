---
name: conductor-status
description: Hiển thị trạng thái dự án, các theo dõi đang hoạt động và các hành động tiếp theo
metadata:
  argument-hint: "[track-id] [--detailed]"
---

# Trạng thái Conductor (Conductor Status)

Hiển thị trạng thái hiện tại của dự án Conductor, bao gồm tiến độ tổng thể, các theo dõi đang hoạt động và các hành động tiếp theo.

## Sử dụng kỹ năng này khi

- Làm việc trên các nhiệm vụ hoặc quy trình trạng thái conductor
- Cần hướng dẫn, thực tiễn tốt nhất hoặc danh sách kiểm tra cho trạng thái conductor

## Không sử dụng kỹ năng này khi

- Nhiệm vụ không liên quan đến trạng thái conductor
- Bạn cần một miền hoặc công cụ khác ngoài phạm vi này

## Hướng dẫn

- Làm rõ mục tiêu, ràng buộc và đầu vào cần thiết.
- Áp dụng các thực tiễn tốt nhất có liên quan và xác thực kết quả.
- Cung cấp các bước có thể hành động và xác minh.
- Nếu cần các ví dụ chi tiết, hãy mở `resources/implementation-playbook.md`.

## Kiểm tra Trước chuyến bay (Pre-flight Checks)

1. Xác minh Conductor đã được khởi tạo:
   - Kiểm tra `conductor/product.md` tồn tại
   - Kiểm tra `conductor/tracks.md` tồn tại
   - Nếu thiếu: Hiển thị lỗi và đề xuất chạy `/conductor:setup` trước

2. Kiểm tra bất kỳ theo dõi nào:
   - Đọc `conductor/tracks.md`
   - Nếu không có theo dõi nào được đăng ký: Hiển thị thông báo hoàn thành thiết lập với đề xuất tạo theo dõi đầu tiên

## Thu thập Dữ liệu

### 1. Thông tin Dự án

Đọc `conductor/product.md` và trích xuất:

- Tên dự án
- Mô tả dự án

### 2. Tổng quan Theo dõi

Đọc `conductor/tracks.md` và phân tích:

- Tổng số theo dõi
- Các theo dõi hoàn thành (đánh dấu `[x]`)
- Các theo dõi đang tiến hành (đánh dấu `[~]`)
- Các theo dõi đang chờ xử lý (đánh dấu `[ ]`)

### 3. Phân tích Theo dõi Chi tiết

Đối với mỗi theo dõi trong `conductor/tracks/`:

Đọc `conductor/tracks/{trackId}/plan.md`:

- Đếm tổng số nhiệm vụ (các dòng khớp với `- [x]`, `- [~]`, `- [ ]` với tiền tố Task)
- Đếm các nhiệm vụ hoàn thành (`[x]`)
- Đếm các nhiệm vụ đang tiến hành (`[~]`)
- Đếm các nhiệm vụ đang chờ xử lý (`[ ]`)
- Xác định giai đoạn hiện tại (giai đoạn đầu tiên với các nhiệm vụ chưa hoàn thành)
- Xác định nhiệm vụ chờ xử lý tiếp theo

Đọc `conductor/tracks/{trackId}/metadata.json`:

- Loại theo dõi (feature, bug, chore, refactor)
- Ngày tạo
- Ngày cập nhật lần cuối
- Trạng thái

Đọc `conductor/tracks/{trackId}/spec.md`:

- Kiểm tra bất kỳ chặn (blockers) hoặc phụ thuộc nào được ghi chú

### 4. Phát hiện Chặn

Quét các chặn tiềm năng:

- Các nhiệm vụ được đánh dấu bằng tiền tố `BLOCKED:`
- Phụ thuộc vào các theo dõi chưa hoàn thành
- Các nhiệm vụ xác minh thất bại

## Định dạng Đầu ra

### Trạng thái Dự án Đầy đủ (không có đối số)

```
================================================================================
                        PROJECT STATUS: {Project Name}
================================================================================
Cập nhật lần cuối: {current timestamp}

--------------------------------------------------------------------------------
                              TIẾN ĐỘ TỔNG THỂ
--------------------------------------------------------------------------------

Theo dõi:   {completed}/{total} completed ({percentage}%)
Nhiệm vụ:   {completed}/{total} completed ({percentage}%)

Tiến độ:    [##########..........] {percentage}%

--------------------------------------------------------------------------------
                              TÓM TẮT THEO DÕI
--------------------------------------------------------------------------------

| Status | Track ID          | Type    | Tasks      | Last Updated |
|--------|-------------------|---------|------------|--------------|
| [x]    | auth_20250110     | feature | 12/12 (100%)| 2025-01-12  |
| [~]    | dashboard_20250112| feature | 7/15 (47%) | 2025-01-15  |
| [ ]    | nav-fix_20250114  | bug     | 0/4 (0%)   | 2025-01-14  |

--------------------------------------------------------------------------------
                              TRỌNG TÂM HIỆN TẠI
--------------------------------------------------------------------------------

Theo dõi Đang hoạt động: dashboard_20250112 - Dashboard Feature
Giai đoạn Hiện tại:      Phase 2: Core Components
Nhiệm vụ Hiện tại:       [~] Task 2.3: Implement chart rendering

Tiến độ trong Giai đoạn:
  - [x] Task 2.1: Create dashboard layout
  - [x] Task 2.2: Add data fetching hooks
  - [~] Task 2.3: Implement chart rendering
  - [ ] Task 2.4: Add filter controls

--------------------------------------------------------------------------------
                              HÀNH ĐỘNG TIẾP THEO
--------------------------------------------------------------------------------

1. Hoàn thành: Task 2.3 - Implement chart rendering (dashboard_20250112)
2. Sau đó: Task 2.4 - Add filter controls (dashboard_20250112)
3. Sau Giai đoạn 2: Phase verification checkpoint

--------------------------------------------------------------------------------
                               CÁC CHẶN (BLOCKERS)
--------------------------------------------------------------------------------

{If blockers found:}
! BLOCKED: Task 3.1 in dashboard_20250112 depends on api_20250111 (incomplete)

{If no blockers:}
Không có chặn nào được xác định.

================================================================================
Lệnh: /conductor:implement {trackId} | /conductor:new-track | /conductor:revert
================================================================================
```

### Trạng thái Theo dõi Đơn (với đối số track-id)

```
================================================================================
                    TRACK STATUS: {Track Title}
================================================================================
Track ID:    {trackId}
Type:        {feature|bug|chore|refactor}
Status:      {Pending|In Progress|Complete}
Created:     {date}
Updated:     {date}

--------------------------------------------------------------------------------
                              ĐẶC TẢ
--------------------------------------------------------------------------------

Tóm tắt: {brief summary from spec.md}

Tiêu chí Chấp nhận:
  - [x] {Criterion 1}
  - [ ] {Criterion 2}
  - [ ] {Criterion 3}

--------------------------------------------------------------------------------
                              TRIỂN KHAI
--------------------------------------------------------------------------------

Tổng thể:   {completed}/{total} tasks ({percentage}%)
Tiến độ:    [##########..........] {percentage}%

## Phase 1: {Phase Name} [COMPLETE]
  - [x] Task 1.1: {description}
  - [x] Task 1.2: {description}
  - [x] Verification: {description}

## Phase 2: {Phase Name} [IN PROGRESS]
  - [x] Task 2.1: {description}
  - [~] Task 2.2: {description}  <-- CURRENT
  - [ ] Task 2.3: {description}
  - [ ] Verification: {description}

## Phase 3: {Phase Name} [PENDING]
  - [ ] Task 3.1: {description}
  - [ ] Task 3.2: {description}
  - [ ] Verification: {description}

--------------------------------------------------------------------------------
                              LỊCH SỬ GIT
--------------------------------------------------------------------------------

Các Cam kết Liên quan:
  abc1234 - feat: add login form ({trackId})
  def5678 - feat: add password validation ({trackId})
  ghi9012 - chore: mark task 1.2 complete ({trackId})

--------------------------------------------------------------------------------
                              CÁC BƯỚC TIẾP THEO
--------------------------------------------------------------------------------

1. Hiện tại: Task 2.2 - {description}
2. Tiếp theo: Task 2.3 - {description}
3. Kiểm tra xác minh Giai đoạn 2 đang chờ xử lý

================================================================================
Lệnh: /conductor:implement {trackId} | /conductor:revert {trackId}
================================================================================
```

## Chú giải Đánh dấu Trạng thái

Hiển thị ở dưới cùng nếu hữu ích:

```
Chú giải:
  [x] = Hoàn thành
  [~] = Đang tiến hành
  [ ] = Chờ xử lý
  [!] = Bị chặn
```

## Các Trạng thái Lỗi

### Không Tìm thấy Theo dõi

```
================================================================================
                        PROJECT STATUS: {Project Name}
================================================================================

Conductor đã được thiết lập nhưng chưa có theo dõi nào được tạo.

Để bắt đầu:
  /conductor:new-track "your feature description"

================================================================================
```

### Conductor Chưa được Khởi tạo

```
LỖI: Conductor chưa được khởi tạo

Không thể tìm thấy conductor/product.md

Chạy /conductor:setup để khởi tạo Conductor cho dự án này.
```

### Theo dõi Không Tìm thấy (với đối số)

```
LỖI: Không tìm thấy theo dõi: {argument}

Các theo dõi có sẵn:
  - auth_20250115
  - dashboard_20250112
  - nav-fix_20250114

Cách dùng: /conductor:status [track-id]
```

## Logic Tính toán

### Đếm Nhiệm vụ

```
Cho mỗi plan.md:
  - Hoàn thành: đếm dòng khớp /^- \[x\] Task/
  - Đang tiến hành: đếm dòng khớp /^- \[~\] Task/
  - Chờ xử lý: đếm dòng khớp /^- \[ \] Task/
  - Tổng số: Hoàn thành + Đang tiến hành + Chờ xử lý
```

### Phát hiện Giai đoạn

```
Giai đoạn hiện tại = tiêu đề giai đoạn đầu tiên theo sau bởi bất kỳ nhiệm vụ chưa hoàn thành nào ([ ] hoặc [~])
```

### Thanh Tiến độ

```
filled = floor((completed / total) * 20)
empty = 20 - filled
bar = "[" + "#".repeat(filled) + ".".repeat(empty) + "]"
```

## Chế độ Nhanh (Quick Mode)

Nếu gọi với `--quick` hoặc `-q`:

```
{Project Name}: {completed}/{total} tasks ({percentage}%)
Active: {trackId} - Task {X.Y}
```

## Đầu ra JSON

Nếu gọi với `--json`:

```json
{
  "project": "{name}",
  "timestamp": "ISO_TIMESTAMP",
  "tracks": {
    "total": N,
    "completed": X,
    "in_progress": Y,
    "pending": Z
  },
  "tasks": {
    "total": M,
    "completed": A,
    "in_progress": B,
    "pending": C
  },
  "current": {
    "track": "{trackId}",
    "phase": N,
    "task": "{X.Y}"
  },
  "blockers": []
}
```
