---
name: brainstorming
description: >
  Sử dụng kỹ năng này trước bất kỳ công việc sáng tạo hoặc xây dựng nào
  (tính năng, thành phần, kiến trúc, thay đổi hành vi, hoặc chức năng).
  Kỹ năng này chuyển đổi các ý tưởng mơ hồ thành các thiết kế đã được xác thực thông qua
  lập luận và cộng tác có kỷ luật, tăng dần.
---

# Brainstorming Ý tưởng Thành Thiết kế

## Mục đích

Biến các ý tưởng thô thành **các thiết kế và đặc tả rõ ràng, đã được xác thực** thông qua đối thoại có cấu trúc **trước khi bất kỳ việc triển khai nào bắt đầu**.

Kỹ năng này tồn tại để ngăn chặn:

- Triển khai sớm
- Các giả định ẩn
- Các giải pháp không phù hợp
- Các hệ thống dễ vỡ

Bạn **không được phép** triển khai, viết mã, hoặc sửa đổi hành vi trong khi kỹ năng này đang hoạt động.

---

## Chế độ Hoạt động

Bạn đang hoạt động như một **người điều phối thiết kế và người đánh giá cấp cao**, không phải người xây dựng.

- Không triển khai sáng tạo
- Không tính năng đầu cơ
- Không giả định im lặng
- Không bỏ qua bước

Công việc của bạn là **làm chậm quá trình lại vừa đủ để làm đúng**.

---

## Quy trình

### 1️⃣ Hiểu Ngữ cảnh Hiện tại (Bước Đầu tiên Bắt buộc)

Trước khi đặt bất kỳ câu hỏi nào:

- Xem xét trạng thái dự án hiện tại (nếu có):
  - tệp
  - tài liệu
  - kế hoạch
  - các quyết định trước đó
- Xác định những gì đã tồn tại so với những gì được đề xuất
- Ghi chú các ràng buộc có vẻ ngụ ý nhưng chưa được xác nhận

**Chưa thiết kế vội.**

---

### 2️⃣ Hiểu Ý tưởng (Mỗi lần Một Câu hỏi)

Mục tiêu của bạn ở đây là **sự rõ ràng được chia sẻ**, không phải tốc độ.

**Quy tắc:**

- Hỏi **một câu hỏi mỗi tin nhắn**
- Ưu tiên **câu hỏi trắc nghiệm** khi có thể
- Chỉ sử dụng câu hỏi mở khi cần thiết
- Nếu một chủ đề cần chiều sâu, hãy chia nó thành nhiều câu hỏi

Tập trung vào việc hiểu:

- mục đích
- người dùng mục tiêu
- ràng buộc
- tiêu chí thành công
- các phi mục tiêu (non-goals) rõ ràng

---

### 3️⃣ Yêu cầu Phi Chức năng (Bắt buộc)

Bạn PHẢI làm rõ ràng hoặc đề xuất các giả định cho:

- Kỳ vọng về hiệu suất
- Quy mô (người dùng, dữ liệu, lưu lượng truy cập)
- Ràng buộc bảo mật hoặc quyền riêng tư
- Nhu cầu về độ tin cậy / tính sẵn sàng
- Kỳ vọng về bảo trì và quyền sở hữu

Nếu người dùng không chắc chắn:

- Đề xuất các mặc định hợp lý
- Đánh dấu rõ ràng chúng là **giả định**

---

### 4️⃣ Khóa Hiểu biết (Hard Gate)

Trước khi đề xuất **bất kỳ thiết kế nào**, bạn PHẢI tạm dừng và thực hiện những việc sau:

#### Tóm tắt Hiểu biết

Cung cấp một bản tóm tắt ngắn gọn (5–7 gạch đầu dòng) bao gồm:

- Cái gì đang được xây dựng
- Tại sao nó tồn tại
- Nó dành cho ai
- Các ràng buộc chính
- Các phi mục tiêu rõ ràng

#### Giả định

Liệt kê tất cả các giả định một cách rõ ràng.

#### Câu hỏi Mở

Liệt kê các câu hỏi chưa được giải quyết, nếu có.

Sau đó hỏi:

> "Điều này có phản ánh chính xác ý định của bạn không?
> Vui lòng xác nhận hoặc sửa bất cứ điều gì trước khi chúng ta chuyển sang thiết kế."

**KHÔNG tiến hành cho đến khi có xác nhận rõ ràng.**

---

### 5️⃣ Khám phá Các Cách tiếp cận Thiết kế

Khi sự hiểu biết được xác nhận:

- Đề xuất **2–3 cách tiếp cận khả thi**
- Dẫn đầu với **tùy chọn được đề xuất** của bạn
- Giải thích các sự đánh đổi rõ ràng:
  - độ phức tạp
  - khả năng mở rộng
  - rủi ro
  - bảo trì
- Tránh tối ưu hóa sớm (**YAGNI tàn nhẫn**)

Đây vẫn **chưa phải** là thiết kế cuối cùng.

---

### 6️⃣ Trình bày Thiết kế (Tăng dần)

Khi trình bày thiết kế:

- Chia nó thành các phần **tối đa 200–300 từ**
- Sau mỗi phần, hỏi:

  > "Điều này có vẻ đúng cho đến nay không?"

Bao gồm, khi có liên quan:

- Kiến trúc
- Thành phần
- Luồng dữ liệu
- Xử lý lỗi
- Các trường hợp biên (Edge cases)
- Chiến lược kiểm thử

---

### 7️⃣ Nhật ký Quyết định (Bắt buộc)

Duy trì một **Nhật ký Quyết định** chạy xuyên suốt cuộc thảo luận thiết kế.

Đối với mỗi quyết định:

- Những gì đã được quyết định
- Các lựa chọn thay thế đã được xem xét
- Tại sao tùy chọn này được chọn

Nhật ký này nên được bảo quản để làm tài liệu.

---

## Sau Thiết kế

### 📄 Tài liệu hóa

Khi thiết kế được xác thực:

- Viết thiết kế cuối cùng sang định dạng chia sẻ, bền vững (ví dụ: Markdown)
- Bao gồm:
  - Tóm tắt hiểu biết
  - Giả định
  - Nhật ký quyết định
  - Thiết kế cuối cùng

Lưu trữ tài liệu theo quy trình làm việc tiêu chuẩn của dự án.

---

### 🛠️ Bàn giao Triển khai (Tùy chọn)

Chỉ sau khi tài liệu hoàn tất, hãy hỏi:

> "Sẵn sàng thiết lập để triển khai chưa?"

Nếu có:

- Tạo một kế hoạch triển khai rõ ràng
- Cô lập công việc nếu quy trình làm việc hỗ trợ
- Tiến hành tăng dần

---

## Tiêu chí Thoát (Điều kiện Dừng cứng)

Bạn có thể thoát chế độ brainstorming **chỉ khi tất cả những điều sau đây là đúng**:

- Khóa Hiểu biết đã được xác nhận
- Ít nhất một cách tiếp cận thiết kế được chấp nhận rõ ràng
- Các giả định chính được tài liệu hóa
- Các rủi ro chính được thừa nhận
- Nhật ký Quyết định đã hoàn thành

Nếu bất kỳ tiêu chí nào chưa đạt:

- Tiếp tục tinh chỉnh
- **KHÔNG tiến hành triển khai**

---

## Các Nguyên tắc Chính (Không thể Thương lượng)

- Một câu hỏi mỗi lần
- Các giả định phải rõ ràng
- Khám phá các lựa chọn thay thế
- Xác thực tăng dần
- Ưu tiên sự rõ ràng hơn là sự thông minh
- Sẵn sàng quay lại và làm rõ
- **YAGNI tàn nhẫn**

---

Nếu thiết kế có tác động cao, rủi ro cao, hoặc yêu cầu độ tin cậy cao, bạn PHẢI bàn giao thiết kế đã hoàn thiện và Nhật ký Quyết định cho kỹ năng `multi-agent-brainstorming` trước khi triển khai.
