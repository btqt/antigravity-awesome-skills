---
name: ab-test-setup
description: Hướng dẫn có cấu trúc để thiết lập thử nghiệm A/B với các quy trình bắt buộc cho giả thuyết, chỉ số và mức độ sẵn sàng thực thi.
---

# Thiết lập Thử nghiệm A/B

## 1️⃣ Mục đích & Phạm vi

Đảm bảo mọi thử nghiệm A/B đều **hợp lệ, nghiêm ngặt và an toàn** trước khi viết bất kỳ dòng mã nào.

- Ngăn chặn việc "nhìn trộm" kết quả sớm (peeking)
- Đảm bảo lực thống kê (statistical power)
- Loại bỏ các giả thuyết không hợp lệ

---

## 2️⃣ Điều kiện Tiên quyết

Bạn phải có:

- Một vấn đề người dùng rõ ràng
- Quyền truy cập vào nguồn dữ liệu phân tích
- Ước tính sơ bộ về lượng lưu lượng truy cập (traffic)

### Danh sách Kiểm tra Chất lượng Giả thuyết

Một giả thuyết hợp lệ bao gồm:

- Quan sát hoặc bằng chứng
- Một thay đổi cụ thể, duy nhất
- Kỳ vọng về hướng thay đổi
- Đối tượng xác định
- Tiêu chí thành công có thể đo lường được

---

### 3️⃣ Chốt Giả thuyết (Cổng kiểm soát cứng)

Trước khi thiết kế các biến thể hoặc chỉ số, bạn PHẢI:

- Trình bày **giả thuyết cuối cùng**
- Chỉ định rõ:
  - Đối tượng mục tiêu
  - Chỉ số chính (Primary metric)
  - Hướng tác động kỳ vọng
  - Hiệu ứng tối thiểu có thể phát hiện (MDE)

Hỏi rõ ràng:

> “Đây có phải là giả thuyết cuối cùng mà chúng ta cam kết cho thử nghiệm này không?”

**KHÔNG tiếp tục cho đến khi được xác nhận.**

---

### 4️⃣ Kiểm tra Giả định & Tính hợp lệ (Bắt buộc)

Liệt kê rõ ràng các giả định về:

- Sự ổn định của lưu lượng truy cập
- Tính độc lập của người dùng
- Độ tin cậy của chỉ số
- Chất lượng của việc ngẫu nhiên hóa
- Các yếu tố bên ngoài (tính mùa vụ, chiến dịch, các lần phát hành khác)

Nếu các giả định yếu hoặc bị vi phạm:

- Cảnh báo người dùng
- Khuyến nghị trì hoãn hoặc thiết kế lại thử nghiệm

---

### 5️⃣ Lựa chọn Loại Thử nghiệm

Chọn loại thử nghiệm đơn giản nhất mà vẫn hợp lệ:

- **A/B Test** – một thay đổi duy nhất, hai biến thể
- **A/B/n Test** – nhiều biến thể, yêu cầu lưu lượng truy cập cao hơn
- **Multivariate Test (MVT)** – hiệu ứng tương tác, yêu cầu lưu lượng truy cập rất cao
- **Split URL Test** – các thay đổi lớn về cấu trúc

Mặc định chọn **A/B** trừ khi có lý do rõ ràng khác.

---

### 6️⃣ Định nghĩa Chỉ số

#### Chỉ số Chính (Bắt buộc)

- Chỉ số duy nhất được sử dụng để đánh giá thành công
- Gắn liền trực tiếp với giả thuyết
- Được xác định trước và "đóng băng" trước khi bắt đầu

#### Chỉ số Phụ

- Cung cấp ngữ cảnh
- Giải thích _tại sao_ kết quả đó xảy ra
- Không được ghi đè lên chỉ số chính

#### Chỉ số Rào chắn (Guardrail Metrics)

- Các chỉ số không được phép suy giảm
- Được sử dụng để ngăn chặn các "chiến thắng" gây hại cho hệ thống
- Kích hoạt dừng thử nghiệm nếu có kết quả tiêu cực đáng kể

---

### 7️⃣ Quy mô Mẫu & Thời lượng

Xác định trước:

- Tỷ lệ cơ sở (Baseline rate)
- MDE
- Mức ý nghĩa (Significance level - thường là 95%)
- Lực thống kê (Statistical power - thường là 80%)

Ước tính:

- Quy mô mẫu cần thiết cho mỗi biến thể
- Thời lượng thử nghiệm dự kiến

**KHÔNG tiếp tục nếu không có ước tính quy mô mẫu thực tế.**

---

### 8️⃣ Cổng Sẵn sàng Thực thi (Dừng khẩn cấp)

Bạn chỉ có thể tiến hành triển khai **nếu tất cả các điều sau đều đúng**:

- Giả thuyết đã được chốt
- Chỉ số chính đã được đóng băng
- Quy mô mẫu đã được tính toán
- Thời lượng thử nghiệm đã được xác định
- Các chỉ số rào chắn đã được thiết lập
- Việc theo dõi (tracking) đã được xác minh

Nếu thiếu bất kỳ mục nào, hãy dừng lại và giải quyết nó.

---

## Chạy Thử nghiệm

### Trong khi Thử nghiệm

**NÊN:**

- Theo dõi sức khỏe kỹ thuật
- Ghi lại các yếu tố bên ngoài

**KHÔNG ĐƯỢC:**

- Dừng sớm vì kết quả "trông có vẻ tốt"
- Thay đổi các biến thể giữa chừng
- Thêm các nguồn lưu lượng truy cập mới
- Định nghĩa lại tiêu chí thành công

---

## Phân tích Kết quả

### Kỷ luật Phân tích

Khi diễn giải kết quả:

- KHÔNG tổng quát hóa ngoài phạm vi đối tượng được thử nghiệm
- KHÔNG khẳng định quan hệ nhân quả ngoài thay đổi được thử nghiệm
- KHÔNG bỏ qua các lỗi vi phạm chỉ số rào chắn
- Tách biệt ý nghĩa thống kê khỏi đánh giá kinh doanh

### Kết quả Diễn giải

| Kết quả                 | Hành động                                          |
| ----------------------- | -------------------------------------------------- |
| Tích cực đáng kể        | Cân nhắc triển khai rộng rãi (rollout)             |
| Tiêu cực đáng kể        | Từ bỏ biến thể, ghi lại bài học rút ra             |
| Không kết luận được     | Cân nhắc thêm lưu lượng hoặc thay đổi mạnh dạn hơn |
| Vi phạm chỉ số rào chắn | Không triển khai, ngay cả khi chỉ số chính thắng   |

---

## Tài liệu & Học hỏi

### Hồ sơ Thử nghiệm (Bắt buộc)

Ghi chép lại:

- Giả thuyết
- Các biến thể
- Chỉ số
- Quy mô mẫu dự kiến và thực tế đạt được
- Kết quả
- Quyết định
- Các bài học rút ra
- Ý tưởng tiếp theo

Lưu trữ hồ sơ ở một vị trí chung, có thể tìm kiếm được để tránh lặp lại các thất bại cũ.

---

## Các Điều kiện Từ chối (An toàn)

Từ chối thực hiện nếu:

- Tỷ lệ cơ sở không xác định và không thể ước tính
- Lưu lượng truy cập không đủ để phát hiện MDE
- Chỉ số chính chưa được định nghĩa
- Nhiều biến số bị thay đổi mà không có thiết kế phù hợp
- Giả thuyết không được trình bày rõ ràng

Giải thích lý do và khuyến nghị các bước tiếp theo.

---

## Các Nguyên tắc Cố định (Không thỏa hiệp)

- Một giả thuyết cho mỗi thử nghiệm
- Một chỉ số chính
- Cam kết trước khi bắt đầu
- Không "nhìn trộm" kết quả
- Học hỏi quan trọng hơn chiến thắng
- Sự nghiêm ngặt về thống kê là trên hết

---

## Nhắc nhở Cuối cùng

Thử nghiệm A/B không phải là để chứng minh các ý tưởng là đúng.
Mà là để **tìm ra sự thật với sự tự tin**.

Nếu bạn cảm thấy muốn vội vàng, đơn giản hóa hoặc "cứ thử xem" —
đó là dấu hiệu cho thấy bạn cần **chậm lại và kiểm tra lại thiết kế của mình**.
