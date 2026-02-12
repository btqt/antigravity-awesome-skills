---
name: clean-code
description: "Áp dụng các nguyên tắc từ 'Clean Code' của Robert C. Martin. Sử dụng kỹ năng này khi viết, đánh giá hoặc tái cấu trúc mã để đảm bảo chất lượng cao, khả năng đọc và khả năng bảo trì. Bao gồm đặt tên, hàm, bình luận, xử lý lỗi và thiết kế lớp."
user-invocable: true
risk: safe
source: "ClawForge (https://github.com/jackjin1997/ClawForge)"
---

# Kỹ năng Clean Code

Kỹ năng này thể hiện các nguyên tắc của "Clean Code" của Robert C. Martin (Uncle Bob). Sử dụng nó để chuyển đổi "mã hoạt động" thành "mã sạch".

## 🧠 Triết lý Cốt lõi

> "Mã là sạch nếu nó có thể được đọc và nâng cao bởi một nhà phát triển khác ngoài tác giả ban đầu của nó." — Grady Booch

## Khi nào Sử dụng

Sử dụng kỹ năng này khi:

- **Viết mã mới**: Để đảm bảo chất lượng cao ngay từ đầu.
- **Đánh giá Pull Requests**: Để cung cấp phản hồi mang tính xây dựng, dựa trên nguyên tắc.
- **Tái cấu trúc mã cũ**: Để xác định và loại bỏ các mùi mã (code smells).
- **Cải thiện tiêu chuẩn nhóm**: Để phù hợp với các thực tiễn tốt nhất theo tiêu chuẩn ngành.

## 1. Tên có Ý nghĩa

- **Sử dụng Tên Tiết lộ Ý định**: `elapsedTimeInDays` thay vì `d`.
- **Tránh Thông tin Sai lệch**: Đừng sử dụng `accountList` nếu nó thực sự là một `Map`.
- **Tạo sự Phân biệt Có ý nghĩa**: Tránh `ProductData` so với `ProductInfo`.
- **Sử dụng Tên Có thể Phát âm/Tìm kiếm**: Tránh `genymdhms`.
- **Tên Lớp**: Sử dụng danh từ (`Customer`, `WikiPage`). Tránh `Manager`, `Data`.
- **Tên Phương thức**: Sử dụng động từ (`postPayment`, `deletePage`).

## 2. Hàm

- **Nhỏ!**: Các hàm nên ngắn hơn bạn nghĩ.
- **Làm Một Việc**: Một hàm chỉ nên làm một việc, và làm tốt việc đó.
- **Một Mức Trừu tượng**: Đừng trộn lẫn logic nghiệp vụ cấp cao với các chi tiết cấp thấp (như regex).
- **Tên Mô tả**: `isPasswordValid` tốt hơn là `check`.
- **Đối số**: 0 là lý tưởng, 1-2 là ổn, 3+ yêu cầu một lý do rất mạnh mẽ.
- **Không Tác dụng Phụ**: Các hàm không nên thay đổi trạng thái toàn cục một cách bí mật.

## 3. Bình luận

- **Đừng Bình luận Mã Xấu—Hãy Viết lại Nó**: Hầu hết các bình luận là dấu hiệu của việc không thể diễn đạt bản thân bằng mã.
- **Giải thích Bản thân bằng Mã**:
  ```python
  # Kiểm tra xem nhân viên có đủ điều kiện nhận đầy đủ phúc lợi không
  if employee.flags & HOURLY and employee.age > 65:
  ```
  so với
  ```python
  if employee.isEligibleForFullBenefits():
  ```
- **Bình luận Tốt**: Pháp lý, Thông tin (ý định regex), Làm rõ (thư viện bên ngoài), TODOs.
- **Bình luận Xấu**: Lẩm bẩm, Dư thừa, Gây hiểu lầm, Bắt buộc, Nhiễu, Đánh dấu Vị trí.

## 4. Định dạng

- **Phép ẩn dụ Báo chí**: Các khái niệm cấp cao ở trên cùng, chi tiết ở dưới cùng.
- **Mật độ Dọc**: Các dòng liên quan nên ở gần nhau.
- **Khoảng cách**: Các biến nên được khai báo gần nơi sử dụng của chúng.
- **Thụt lề**: Cần thiết cho khả năng đọc cấu trúc.

## 5. Đối tượng và Cấu trúc Dữ liệu

- **Trừu tượng hóa Dữ liệu**: Ẩn việc triển khai đằng sau các giao diện.
- **Định luật Demeter**: Một mô-đun không nên biết về nội tại của các đối tượng mà nó thao tác. Tránh `a.getB().getC().doSomething()`.
- **Đối tượng Chuyển Dữ liệu (DTO)**: Các lớp có biến công khai và không có hàm.

## 6. Xử lý Lỗi

- **Sử dụng Ngoại lệ thay vì Mã Trả về**: Giữ cho logic sạch sẽ.
- **Viết Try-Catch-Finally Đầu tiên**: Xác định phạm vi của hoạt động.
- **Đừng Trả về Null**: Nó buộc người gọi phải kiểm tra null mỗi lần.
- **Đừng Truyền Null**: Dẫn đến `NullPointerException`.

## 7. Kiểm thử Đơn vị (Unit Tests)

- **Ba Định luật của TDD**:
  1. Đừng viết mã sản xuất cho đến khi bạn có một bài kiểm tra đơn vị thất bại.
  2. Đừng viết nhiều bài kiểm tra đơn vị hơn mức đủ để thất bại.
  3. Đừng viết nhiều mã sản xuất hơn mức đủ để vượt qua bài kiểm tra thất bại.
- **Nguyên tắc F.I.R.S.T.**: Nhanh (Fast), Độc lập (Independent), Có thể lặp lại (Repeatable), Tự xác thực (Self-Validating), Kịp thời (Timely).

## 8. Lớp (Classes)

- **Nhỏ!**: Các lớp nên có một trách nhiệm duy nhất (SRP).
- **Quy tắc Stepdown**: Chúng ta muốn mã đọc như một câu chuyện từ trên xuống dưới.

## 9. Mùi và Heuristics

- **Cứng nhắc**: Khó thay đổi.
- **Dễ vỡ**: Vỡ ở nhiều nơi.
- **Bất động**: Khó tái sử dụng.
- **Độ nhớt**: Khó làm điều đúng đắn.
- **Phức tạp/Lặp lại không cần thiết**.

## 🛠️ Danh sách kiểm tra Triển khai

- [ ] Hàm này có nhỏ hơn 20 dòng không?
- [ ] Hàm này có làm chính xác một việc không?
- [ ] Tất cả các tên có thể tìm kiếm và tiết lộ ý định không?
- [ ] Tôi có tránh được các bình luận bằng cách làm cho mã rõ ràng hơn không?
- [ ] Tôi có đang truyền quá nhiều đối số không?
- [ ] Có bài kiểm tra thất bại cho thay đổi này không?
