---
name: design-orchestration
description: >
  Điều phối quy trình làm việc thiết kế bằng cách định tuyến công việc thông qua
  brainstorming, đánh giá đa tác nhân, và kiểm tra sẵn sàng thực thi
  theo thứ tự chính xác. Ngăn ngừa việc triển khai sớm,
  bỏ qua xác thực, và thiết kế rủi ro cao chưa được đánh giá.
---

# Điều Phối Thiết Kế (Meta-Skill)

## Mục đích

Đảm bảo rằng **ý tưởng trở thành thiết kế**, **thiết kế được đánh giá**, và
**chỉ các thiết kế đã xác thực mới đến bước triển khai**.

Kỹ năng này không tạo ra các thiết kế.
Nó **kiểm soát luồng giữa các kỹ năng khác**.

---

## Mô hình Hoạt động

Đây là một **kỹ năng định tuyến và thực thi**, không phải kỹ năng sáng tạo.

Nó quyết định:

- kỹ năng nào phải chạy tiếp theo
- liệu có cần leo thang hay không
- liệu việc thực thi có được phép hay không

---

## Các Kỹ năng Được kiểm soát

Meta-skill này phối hợp các kỹ năng sau:

- `brainstorming` — tạo thiết kế
- `multi-agent-brainstorming` — xác thực thiết kế
- các kỹ năng lập kế hoạch hoặc triển khai phía sau

---

## Điều kiện Đầu vào

Gọi kỹ năng này khi:

- người dùng đề xuất một tính năng, hệ thống hoặc thay đổi mới
- một quyết định thiết kế mang rủi ro đáng kể
- tính chính xác quan trọng hơn tốc độ

---

## Logic Định tuyến

### Bước 1 — Brainstorming (Bắt buộc)

Nếu không có thiết kế đã được xác thực:

- Gọi `brainstorming`
- Yêu cầu:
  - Khóa Hiểu biết (Understanding Lock)
  - Thiết kế Ban đầu
  - Nhật ký Quyết định đã bắt đầu

Bạn KHÔNG ĐƯỢC tiến hành nếu thiếu các tạo tác này.

---

### Bước 2 — Đánh giá Rủi ro

Sau khi brainstorming hoàn tất, phân loại thiết kế là:

- **Rủi ro thấp**
- **Rủi ro vừa**
- **Rủi ro cao**

Sử dụng các yếu tố như:

- tác động đến người dùng
- không thể đảo ngược
- chi phí vận hành
- độ phức tạp
- sự không chắc chắn
- tính mới lạ

---

### Bước 3 — Leo thang Có điều kiện

- **Rủi ro thấp**
  → Tiến hành lập kế hoạch triển khai

- **Rủi ro vừa**
  → Khuyến nghị `multi-agent-brainstorming`

- **Rủi ro cao**
  → YÊU CẦU `multi-agent-brainstorming`

Nghiêm cấm bỏ qua leo thang khi được yêu cầu.

---

### Bước 4 — Đánh giá Đa Tác nhân (Nếu được gọi)

Nếu `multi-agent-brainstorming` được chạy:

Yêu cầu:

- Khóa Hiểu biết đã hoàn thành
- Thiết kế hiện tại
- Nhật ký Quyết định

KHÔNG cho phép:

- lên ý tưởng mới
- mở rộng phạm vi
- mở lại định nghĩa vấn đề

Chỉ cho phép phê bình, sửa đổi và giải quyết quyết định.

---

### Bước 5 — Kiểm tra Sẵn sàng Thực thi

Trước khi cho phép triển khai:

Xác nhận:

- thiết kế đã được phê duyệt (đơn tác nhân hoặc đa tác nhân)
- Nhật ký Quyết định đã hoàn tất
- các giả định chính được tài liệu hóa
- các rủi ro đã biết được thừa nhận

Nếu bất kỳ điều kiện nào thất bại:

- chặn thực thi
- quay lại kỹ năng phù hợp

---

## Quy tắc Thực thi

- KHÔNG cho phép triển khai mà không có thiết kế đã được xác thực
- KHÔNG cho phép bỏ qua đánh giá bắt buộc
- KHÔNG cho phép leo thang hoặc giảm leo thang âm thầm
- KHÔNG hợp nhất các giai đoạn thiết kế và triển khai

---

## Điều kiện Thoát

Meta-skill này thoát CHỈ khi:

- bước tiếp theo được xác định rõ ràng, VÀ
- tất cả các bước tiên quyết bắt buộc đã hoàn tất

Các lối thoát có thể:

- "Tiến hành lập kế hoạch triển khai"
- "Chạy multi-agent-brainstorming"
- "Quay lại brainstorming để làm rõ"
- "Nếu một thiết kế đã được đánh giá báo cáo kết quả cuối cùng là ĐÃ PHÊ DUYỆT, SỬA ĐỔI, hoặc TỪ CHỐI, bạn PHẢI định tuyến quy trình làm việc tương ứng và nêu rõ bước tiếp theo đã chọn."

---

## Triết lý Thiết kế

Kỹ năng này tồn tại để:

- làm chậm các quyết định đúng đắn
- tăng tốc độ thực thi đúng đắn
- ngăn ngừa các sai lầm tốn kém

Hệ thống tốt thất bại sớm.
Hệ thống tồi thất bại trong sản xuất.

Meta-skill này tồn tại để thực thi điều trước.
