---
name: agent-orchestration-improve-agent
description: "Cải thiện có hệ thống các đại lý hiện có thông qua phân tích hiệu suất, prompt engineering và lặp lại liên tục."
---

# Quy trình Tối ưu hóa Hiệu suất Đại lý

Cải thiện có hệ thống các đại lý hiện có thông qua phân tích hiệu suất, prompt engineering và lặp lại liên tục.

[Tư duy mở rộng: Việc tối ưu hóa đại lý đòi hỏi một phương pháp dựa trên dữ liệu, kết hợp các chỉ số hiệu suất, phân tích phản hồi của người dùng và các kỹ thuật prompt engineering nâng cao. Thành công phụ thuộc vào việc đánh giá có hệ thống, cải thiện có mục tiêu và kiểm thử nghiêm ngặt với khả năng khôi phục (rollback) để đảm bảo an toàn khi vận hành thực tế.]

## Sử dụng kỹ năng này khi

- Cải thiện hiệu suất hoặc độ tin cậy của một đại lý hiện có
- Phân tích các lỗi (failure modes), chất lượng prompt hoặc việc sử dụng công cụ
- Chạy các bài kiểm thử A/B có cấu trúc hoặc các bộ đánh giá
- Thiết kế các quy trình tối ưu hóa lặp lại cho các đại lý

## Không sử dụng kỹ năng này khi

- Bạn đang xây dựng một đại lý hoàn toàn mới từ đầu
- Không có bất kỳ chỉ số, phản hồi hoặc trường hợp kiểm thử nào có sẵn
- Nhiệm vụ không liên quan đến hiệu suất đại lý hoặc chất lượng prompt

## Hướng dẫn

1. Thiết lập các chỉ số cơ sở (baseline) và thu thập các ví dụ đại diện.
2. Xác định các lỗi và ưu tiên các giải pháp có tác động cao.
3. Áp dụng các cải tiến cho prompt và quy trình công việc với các mục tiêu có thể đo lường được.
4. Xác thực bằng các bài kiểm thử và triển khai các thay đổi theo các giai đoạn được kiểm soát.

## An toàn

- Tránh triển khai các thay đổi prompt mà không kiểm thử hồi quy (regression testing).
- Khôi phục (roll back) nhanh chóng nếu các chỉ số chất lượng hoặc an toàn bị suy giảm.

## Giai đoạn 1: Phân tích Hiệu suất và Chỉ số Cơ sở

Phân tích toàn diện hiệu suất của đại lý bằng cách sử dụng công cụ quản lý ngữ cảnh (context-manager) để thu thập dữ liệu lịch sử.

### 1.1 Thu thập Dữ liệu Hiệu suất

```
Sử dụng: context-manager
Lệnh: analyze-agent-performance $ARGUMENTS --days 30
```

Thu thập các chỉ số bao gồm:

- Tỷ lệ hoàn thành nhiệm vụ (nhiệm vụ thành công so với thất bại)
- Độ chính xác của câu trả lời và tính đúng đắn của sự thật
- Hiệu quả sử dụng công cụ (đúng công cụ, tần suất gọi)
- Thời gian phản hồi trung bình và tiêu thụ token
- Các chỉ số hài lòng của người dùng (số lần sửa lỗi, số lần thử lại)
- Các sự cố ảo tưởng (hallucination) và các mẫu lỗi

### 1.2 Phân tích Mẫu Phản hồi của Người dùng

Xác định các mẫu lặp lại trong tương tác của người dùng:

- **Mẫu sửa lỗi**: Nơi người dùng liên tục sửa đổi đầu ra của đại lý
- **Yêu cầu làm rõ**: Các khu vực thường gây mơ hồ
- **Từ bỏ nhiệm vụ**: Các điểm mà người dùng bỏ cuộc
- **Câu hỏi tiếp theo**: Các dấu hiệu cho thấy câu trả lời chưa đầy đủ
- **Phản hồi tích cực**: Các mẫu thành công cần giữ lại

### 1.3 Phân loại Lỗi (Failure Mode Classification)

Phân loại các lỗi theo nguyên nhân gốc rễ:

- **Hiểu sai hướng dẫn**: Nhầm lẫn về vai trò hoặc nhiệm vụ
- **Lỗi định dạng đầu ra**: Các vấn đề về cấu trúc hoặc định dạng
- **Mất ngữ cảnh**: Suy giảm chất lượng trong các cuộc hội thoại dài
- **Lạm dụng công cụ**: Lựa chọn công cụ sai hoặc không hiệu quả
- **Vi phạm ràng buộc**: Vi phạm các quy tắc an toàn hoặc kinh doanh
- **Xử lý trường hợp biên**: Các kịch bản đầu vào bất thường

### 1.4 Báo cáo Hiệu suất Cơ sở

Tạo các chỉ số cơ sở định lượng:

```
Cơ sở Hiệu suất (Performance Baseline):
- Tỷ lệ Thành công Nhiệm vụ: [X%]
- Số lần Sửa lỗi Trung bình mỗi Nhiệm vụ: [Y]
- Hiệu quả Gọi Công cụ: [Z%]
- Điểm Hài lòng của Người dùng: [1-10]
- Độ trễ Phản hồi Trung bình: [Xms]
- Tỷ lệ Hiệu quả Token: [X:Y]
```

## Giai đoạn 2: Cải tiến Prompt Engineering

Áp dụng các kỹ thuật tối ưu hóa prompt nâng cao bằng cách sử dụng đại lý prompt-engineer.

### 2.1 Tăng cường Chuỗi Tư duy (Chain-of-Thought)

Triển khai các mẫu lập luận có cấu trúc:

```
Sử dụng: prompt-engineer
Kỹ thuật: chain-of-thought-optimization
```

- Thêm các bước lập luận rõ ràng: "Hãy tiếp cận vấn đề này theo từng bước..."
- Bao gồm các điểm kiểm tra tự xác minh: "Trước khi tiếp tục, hãy xác minh rằng..."
- Triển khai phân rã đệ quy cho các nhiệm vụ phức tạp
- Hiển thị dấu vết lập luận (reasoning trace) để phục vụ gỡ lỗi

### 2.2 Tối ưu hóa Ví dụ Few-Shot

Chọn lọc các ví dụ chất lượng cao từ các tương tác thành công:

- **Chọn các ví dụ đa dạng** bao gồm các trường hợp sử dụng phổ biến
- **Bao gồm các trường hợp biên** mà trước đây đã từng thất bại
- **Hiển thị cả ví dụ tích cực và tiêu cực** kèm theo giải thích
- **Sắp xếp các ví dụ** từ đơn giản đến phức tạp
- **Chú thích các ví dụ** với các điểm quyết định quan trọng

Cấu trúc ví dụ:

```
Ví dụ Tốt:
Đầu vào: [Yêu cầu của người dùng]
Lập luận: [Quy trình tư duy từng bước]
Đầu ra: [Câu trả lời thành công]
Tại sao cách này hiệu quả: [Các yếu tố thành công chính]

Ví dụ Xấu:
Đầu vào: [Yêu cầu tương tự]
Đầu ra: [Câu trả lời thất bại]
Tại sao cách này thất bại: [Các vấn đề cụ thể]
Cách tiếp cận đúng: [Phiên bản đã sửa]
```

### 2.3 Tinh chỉnh Định nghĩa Vai trò

Tăng cường danh tính và năng lực của đại lý:

- **Mục đích cốt lõi**: Nhiệm vụ rõ ràng, gói gọn trong một câu
- **Các lĩnh vực chuyên môn**: Các mảng kiến thức cụ thể
- **Đặc điểm hành vi**: Tính cách và phong cách tương tác
- **Sự thành thạo công cụ**: Các công cụ hiện có và khi nào nên sử dụng chúng
- **Ràng buộc**: Những gì đại lý KHÔNG được làm
- **Tiêu chí thành công**: Cách đo lường việc hoàn thành nhiệm vụ

### 2.4 Tích hợp AI Hiến pháp (Constitutional AI)

Triển khai các cơ chế tự sửa lỗi:

```
Nguyên tắc Hiến pháp (Constitutional Principles):
1. Xác minh tính chính xác của sự thật trước khi phản hồi
2. Tự kiểm tra các định kiến tiềm ẩn hoặc nội dung có hại
3. Xác thực định dạng đầu ra khớp với yêu cầu
4. Đảm bảo tính đầy đủ của câu trả lời
5. Duy trì sự nhất quán với các câu trả lời trước đó
```

Thêm các vòng lặp phê bình-và-sửa-đổi (critique-and-revise):

- Tạo câu trả lời ban đầu
- Tự phê bình dựa trên các nguyên tắc
- Tự động sửa đổi nếu phát hiện vấn đề
- Xác thực cuối cùng trước khi đưa ra kết quả

### 2.5 Điều chỉnh Định dạng Đầu ra

Tối ưu hóa cấu trúc phản hồi:

- **Các mẫu có cấu trúc** cho các nhiệm vụ phổ biến
- **Định dạng động** dựa trên độ phức tạp
- **Tiết lộ dần dần** cho các thông tin chi tiết
- **Tối ưu hóa Markdown** để dễ đọc
- **Định dạng khối mã** với tô sáng cú pháp
- **Tạo bảng và danh sách** để trình bày dữ liệu

## Giai đoạn 3: Kiểm thử và Xác thực

Khung kiểm thử toàn diện với so sánh A/B.

### 3.1 Phát triển Bộ Kiểm thử (Test Suite)

Tạo các kịch bản kiểm thử đại diện:

```
Các loại Kiểm thử:
1. Kịch bản "đường vàng" (các trường hợp thành công phổ biến)
2. Các nhiệm vụ thất bại trước đó (kiểm thử hồi quy)
3. Các trường hợp biên và kịch bản góc
4. Kiểm thử áp lực (các nhiệm vụ phức tạp, nhiều bước)
5. Đầu vào đối kháng (các điểm có khả năng gây lỗi)
6. Các nhiệm vụ chéo lĩnh vực (kết hợp các năng lực)
```

### 3.2 Khung Kiểm thử A/B

So sánh đại lý ban đầu và đại lý đã cải tiến:

```
Sử dụng: parallel-test-runner
Cấu hình:
  - Đại lý A: Phiên bản gốc
  - Đại lý B: Phiên bản cải tiến
  - Bộ kiểm thử: 100 nhiệm vụ đại diện
  - Chỉ số: Tỷ lệ thành công, tốc độ, sử dụng token
  - Đánh giá: Đánh giá mù bởi con người + chấm điểm tự động
```

Kiểm định ý nghĩa thống kê:

- Kích thước mẫu tối thiểu: 100 nhiệm vụ cho mỗi biến thể
- Mức độ tin cậy: 95% (p < 0.05)
- Tính toán cỡ tác động (Cohen's d)
- Phân tích lực lượng (power analysis) cho các bài kiểm thử tương lai

### 3.3 Chỉ số Đánh giá

Khung chấm điểm toàn diện:

**Chỉ số cấp độ Nhiệm vụ:**

- Tỷ lệ hoàn thành (thành công/thất bại theo hệ nhị phân)
- Điểm đúng đắn (độ chính xác 0-100%)
- Điểm hiệu quả (số bước thực hiện so với tối ưu)
- Tính phù hợp của việc sử dụng công cụ
- Tính liên quan và đầy đủ của câu trả lời

**Chỉ số Chất lượng:**

- Tỷ lệ ảo tưởng (lỗi sự thật trên mỗi câu trả lời)
- Điểm nhất quán (sự phù hợp với các câu trả lời trước đó)
- Tuân thủ định dạng (khớp với cấu trúc đã chỉ định)
- Điểm an toàn (tuân thủ ràng buộc)
- Dự đoán sự hài lòng của người dùng

**Chỉ số Hiệu suất:**

- Độ trễ phản hồi (thời gian đến token đầu tiên)
- Tổng thời gian tạo
- Tiêu thụ token (đầu vào + đầu ra)
- Chi phí cho mỗi nhiệm vụ (phí sử dụng API)
- Hiệu quả bộ nhớ/ngữ cảnh

### 3.4 Quy trình Đánh giá bởi Con người

Quá trình xem xét có cấu trúc bởi con người:

- Đánh giá mù (người đánh giá không biết phiên bản nào)
- Tiêu chí chấm điểm chuẩn hóa với các quy tắc rõ ràng
- Nhiều người đánh giá cho mỗi mẫu (đảm bảo độ tin cậy giữa các người chấm)
- Thu thập phản hồi định tính
- Xếp hạng ưu tiên (so sánh A và B)

## Giai đoạn 4: Quản lý Phiên bản và Triển khai

Triển khai an toàn với khả năng giám sát và khôi phục.

### 4.1 Quản lý Phiên bản

Chiến lược đánh số phiên bản có hệ thống:

```
Định dạng Phiên bản: agent-name-v[MAJOR].[MINOR].[PATCH]
Ví dụ: customer-support-v2.3.1

MAJOR: Thay đổi năng lực đáng kể
MINOR: Cải tiến prompt, thêm ví dụ mới
PATCH: Sửa lỗi, điều chỉnh nhỏ
```

Duy trì lịch sử phiên bản:

- Lưu trữ prompt dựa trên Git
- Nhật ký thay đổi (changelog) với chi tiết cải tiến
- Chỉ số hiệu suất cho từng phiên bản
- Quy trình khôi phục được ghi chép đầy đủ

### 4.2 Triển khai theo Giai đoạn

Chiến lược triển khai dần dần:

1. **Kiểm thử Alpha**: Xác thực bởi đội ngũ nội bộ (5% lưu lượng)
2. **Kiểm thử Beta**: Người dùng được chọn lọc (20% lưu lượng)
3. **Phát hành kiểu Canary**: Tăng dần dần (20% → 50% → 100%)
4. **Triển khai toàn bộ**: Sau khi đạt các tiêu chí thành công
5. **Giai đoạn giám sát**: Cửa sổ quan sát trong 7 ngày

### 4.3 Quy trình Khôi phục (Rollback)

Cơ chế phục hồi nhanh chóng:

```
Kích hoạt Khôi phục khi:
- Tỷ lệ thành công giảm >10% so với cơ sở
- Các lỗi nghiêm trọng tăng >5%
- Khiếu nại của người dùng tăng vọt
- Chi phí mỗi nhiệm vụ tăng >20%
- Phát hiện vi phạm an toàn

Quy trình Khôi phục:
1. Phát hiện vấn đề qua giám sát
2. Cảnh báo đội ngũ ngay lập tức
3. Chuyển sang phiên bản ổn định trước đó
4. Phân tích nguyên nhân gốc rễ
5. Sửa lỗi và kiểm thử lại trước khi thử lại
```

### 4.4 Giám sát Liên tục

Theo dõi hiệu suất trong thời gian thực:

- Bảng điều khiển với các chỉ số chính
- Cảnh báo phát hiện bất thường
- Thu thập phản hồi của người dùng
- Kiểm thử hồi quy tự động
- Báo cáo hiệu suất hàng tuần

## Tiêu chí Thành công

Việc cải thiện đại lý được coi là thành công khi:

- Tỷ lệ thành công nhiệm vụ cải thiện ≥15%
- Số lần sửa lỗi của người dùng giảm ≥25%
- Không có sự gia tăng vi phạm an toàn
- Thời gian phản hồi nằm trong khoảng 10% so với cơ sở
- Chi phí mỗi nhiệm vụ không tăng >5%
- Phản hồi tích cực của người dùng tăng lên

## Đánh giá Sau triển khai

Sau 30 ngày sử dụng thực tế:

1. Phân tích dữ liệu hiệu suất tích lũy
2. So sánh với cơ sở và mục tiêu
3. Xác định các cơ hội cải thiện mới
4. Ghi lại các bài học kinh nghiệm
5. Lập kế hoạch cho chu kỳ tối ưu hóa tiếp theo

## Chu kỳ Cải tiến Liên tục

Thiết lập nhịp độ cải tiến định kỳ:

- **Hàng tuần**: Giám sát các chỉ số và thu thập phản hồi
- **Hàng tháng**: Phân tích các mẫu và lập kế hoạch cải tiến
- **Hàng quý**: Cập nhật phiên bản chính với các năng lực mới
- **Hàng năm**: Xem xét chiến lược và cập nhật kiến trúc

Ghi nhớ: Tối ưu hóa đại lý là một quá trình lặp đi lặp lại. Mỗi chu kỳ được xây dựng dựa trên những bài học trước đó, dần dần cải thiện hiệu suất trong khi vẫn duy trì tính ổn định và an toàn.
