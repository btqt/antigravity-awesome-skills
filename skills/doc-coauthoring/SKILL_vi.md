---
name: doc-coauthoring
description: Hướng dẫn người dùng qua một quy trình làm việc có cấu trúc để đồng tác giả tài liệu. Sử dụng khi người dùng muốn viết tài liệu, đề xuất, thông số kỹ thuật, tài liệu quyết định, hoặc nội dung có cấu trúc tương tự. Quy trình này giúp người dùng chuyển giao bối cảnh hiệu quả, tinh chỉnh nội dung qua các lần lặp, và xác minh tài liệu hoạt động tốt cho người đọc. Kích hoạt khi người dùng đề cập đến việc viết tài liệu, tạo đề xuất, soạn thảo thông số kỹ thuật, hoặc các nhiệm vụ tài liệu tương tự.
---

# Quy Trình Đồng Tác Giả Tài Liệu (Doc Co-Authoring Workflow)

Kỹ năng này cung cấp một quy trình làm việc có cấu trúc để hướng dẫn người dùng qua việc tạo tài liệu cộng tác. Đóng vai trò là người hướng dẫn tích cực, dẫn dắt người dùng qua ba giai đoạn: Thu thập Bối cảnh, Tinh chỉnh & Cấu trúc, và Kiểm thử Người đọc.

## Khi nào nên Đề xuất Quy trình này

**Điều kiện kích hoạt:**

- Người dùng đề cập đến việc viết tài liệu: "viết một tài liệu", "soạn thảo một đề xuất", "tạo một thông số kỹ thuật", "write up"
- Người dùng đề cập đến các loại tài liệu cụ thể: "PRD", "design doc", "decision doc", "RFC"
- Người dùng có vẻ đang bắt đầu một nhiệm vụ viết đáng kể

**Đề xuất ban đầu:**
Đề xuất cho người dùng một quy trình làm việc có cấu trúc để đồng tác giả tài liệu. Giải thích ba giai đoạn:

1. **Thu thập Bối cảnh**: Người dùng cung cấp tất cả bối cảnh liên quan trong khi Claude đặt các câu hỏi làm rõ
2. **Tinh chỉnh & Cấu trúc**: Xây dựng lặp lại từng phần thông qua brainstorming và chỉnh sửa
3. **Kiểm thử Người đọc**: Kiểm thử tài liệu với một Claude mới (không có bối cảnh) để bắt các điểm mù trước khi người khác đọc nó

Giải thích rằng cách tiếp cận này giúp đảm bảo tài liệu hoạt động tốt khi người khác đọc nó (bao gồm cả khi họ dán nó vào Claude). Hỏi xem họ có muốn thử quy trình này hay thích làm việc tự do.

Nếu người dùng từ chối, hãy làm việc tự do. Nếu người dùng chấp nhận, tiến hành Giai đoạn 1.

## Giai đoạn 1: Thu thập Bối cảnh

**Mục tiêu:** Thu hẹp khoảng cách giữa những gì người dùng biết và những gì Claude biết, cho phép hướng dẫn thông minh sau này.

### Các Câu hỏi Ban đầu

Bắt đầu bằng cách hỏi người dùng về bối cảnh meta về tài liệu:

1. Loại tài liệu này là gì? (ví dụ: thông số kỹ thuật, tài liệu quyết định, đề xuất)
2. Đối tượng chính là ai?
3. Tác động mong muốn khi ai đó đọc tài liệu này là gì?
4. Có mẫu hoặc định dạng cụ thể nào để tuân theo không?
5. Bất kỳ ràng buộc hoặc bối cảnh nào khác cần biết không?

Thông báo cho họ rằng họ có thể trả lời vắn tắt hoặc đổ thông tin theo bất kỳ cách nào phù hợp nhất với họ.

**Nếu người dùng cung cấp một mẫu hoặc đề cập đến một loại tài liệu:**

- Hỏi xem họ có tài liệu mẫu để chia sẻ không
- Nếu họ cung cấp liên kết đến tài liệu được chia sẻ, hãy sử dụng tích hợp thích hợp để lấy nó
- Nếu họ cung cấp một tệp, hãy đọc nó

**Nếu người dùng đề cập đến việc chỉnh sửa một tài liệu được chia sẻ hiện có:**

- Sử dụng tích hợp thích hợp để đọc trạng thái hiện tại
- Kiểm tra các hình ảnh không có văn bản thay thế (alt-text)
- Nếu tồn tại hình ảnh không có alt-text, hãy giải thích rằng khi người khác sử dụng Claude để hiểu tài liệu, Claude sẽ không thể nhìn thấy chúng. Hỏi xem họ có muốn tạo alt-text không. Nếu có, yêu cầu họ dán từng hình ảnh vào cuộc trò chuyện để tạo alt-text mô tả.

### Đổ Thông tin (Info Dumping)

Khi các câu hỏi ban đầu được trả lời, hãy khuyến khích người dùng đổ tất cả bối cảnh họ có. Yêu cầu thông tin như:

- Bối cảnh về dự án/vấn đề
- Các thảo luận nhóm liên quan hoặc tài liệu được chia sẻ
- Tại sao các giải pháp thay thế không được sử dụng
- Bối cảnh tổ chức (động lực nhóm, sự cố trong quá khứ, chính trị)
- Áp lực thời gian hoặc ràng buộc
- Kiến trúc kỹ thuật hoặc sự phụ thuộc
- Lo ngại của các bên liên quan

Khuyên họ đừng lo lắng về việc tổ chức nó - chỉ cần đưa tất cả ra ngoài. Cung cấp nhiều cách để cung cấp bối cảnh:

- Đổ thông tin theo dòng suy nghĩ (stream-of-consciousness)
- Chỉ vào các kênh nhóm hoặc chủ đề để đọc
- Liên kết đến các tài liệu được chia sẻ

**Nếu các tích hợp khả dụng** (ví dụ: Slack, Teams, Google Drive, SharePoint, hoặc các máy chủ MCP khác), hãy đề cập rằng chúng có thể được sử dụng để kéo bối cảnh trực tiếp.

**Nếu không phát hiện tích hợp nào và đang ở trong Claude.ai hoặc ứng dụng Claude:** Đề xuất họ có thể bật các trình kết nối trong cài đặt Claude của họ để cho phép kéo bối cảnh từ các ứng dụng nhắn tin và lưu trữ tài liệu trực tiếp.

Thông báo cho họ rằng các câu hỏi làm rõ sẽ được đặt ra khi họ đã thực hiện việc đổ thông tin ban đầu.

**Trong quá trình thu thập bối cảnh:**

- Nếu người dùng đề cập đến các kênh nhóm hoặc tài liệu được chia sẻ:
  - Nếu tích hợp khả dụng: Thông báo cho họ rằng nội dung sẽ được đọc ngay bây giờ, sau đó sử dụng tích hợp thích hợp
  - Nếu tích hợp không khả dụng: Giải thích việc thiếu quyền truy cập. Đề xuất họ bật trình kết nối trong cài đặt Claude, hoặc dán trực tiếp nội dung liên quan.

- Nếu người dùng đề cập đến các thực thể/dự án chưa biết:
  - Hỏi xem các công cụ được kết nối có nên được tìm kiếm để tìm hiểu thêm không
  - Chờ người dùng xác nhận trước khi tìm kiếm

- Khi người dùng cung cấp bối cảnh, hãy theo dõi những gì đang được học và những gì vẫn chưa rõ ràng

**Đặt câu hỏi làm rõ:**

Khi người dùng ra hiệu họ đã thực hiện việc đổ thông tin ban đầu (hoặc sau khi cung cấp bối cảnh đáng kể), hãy đặt các câu hỏi làm rõ để đảm bảo sự hiểu biết:

Tạo 5-10 câu hỏi được đánh số dựa trên các khoảng trống trong bối cảnh.

Thông báo cho họ rằng họ có thể sử dụng cách viết tắt để trả lời (ví dụ: "1: có, 2: xem #channel, 3: không vì tương thích ngược"), liên kết đến nhiều tài liệu hơn, chỉ vào các kênh để đọc, hoặc chỉ tiếp tục đổ thông tin. Bất cứ điều gì hiệu quả nhất cho họ.

**Điều kiện thoát:**
Bối cảnh đủ đã được thu thập khi các câu hỏi cho thấy sự hiểu biết - khi các trường hợp biên và sự đánh đổi có thể được hỏi mà không cần giải thích những điều cơ bản.

**Chuyển tiếp:**
Hỏi xem họ có muốn cung cấp thêm bối cảnh nào ở giai đoạn này không, hay đã đến lúc chuyển sang soạn thảo tài liệu.

Nếu người dùng muốn thêm, hãy để họ làm. Khi đã sẵn sàng, tiến hành Giai đoạn 2.

## Giai đoạn 2: Tinh chỉnh & Cấu trúc

**Mục tiêu:** Xây dựng tài liệu từng phần thông qua brainstorming, chọn lọc và tinh chỉnh lặp lại.

**Hướng dẫn cho người dùng:**
Giải thích rằng tài liệu sẽ được xây dựng từng phần. Đối với mỗi phần:

1. Các câu hỏi làm rõ sẽ được hỏi về những gì cần bao gồm
2. 5-20 tùy chọn sẽ được brainstorming
3. Người dùng sẽ chỉ ra những gì cần giữ/xóa/kết hợp
4. Phần đó sẽ được soạn thảo
5. Nó sẽ được tinh chỉnh thông qua các chỉnh sửa phẫu thuật

Bắt đầu với bất kỳ phần nào có nhiều ẩn số nhất (thường là quyết định/đề xuất cốt lõi), sau đó làm việc qua phần còn lại.

**Thứ tự phần:**

Nếu cấu trúc tài liệu rõ ràng:
Hỏi họ muốn bắt đầu với phần nào.

Đề xuất bắt đầu với bất kỳ phần nào có nhiều ẩn số nhất. Đối với tài liệu quyết định, đó thường là đề xuất cốt lõi. Đối với thông số kỹ thuật, thường là cách tiếp cận kỹ thuật. Các phần tóm tắt tốt nhất nên để lại cuối cùng.

Nếu người dùng không biết họ cần những phần nào:
Dựa trên loại tài liệu và mẫu, đề xuất 3-5 phần phù hợp cho loại tài liệu.

Hỏi xem cấu trúc này có hoạt động không, hoặc họ có muốn điều chỉnh nó không.

**Khi cấu trúc được thống nhất:**

Tạo cấu trúc tài liệu ban đầu với văn bản giữ chỗ cho tất cả các phần.

**Nếu có quyền truy cập vào tạo tác (artifacts):**
Sử dụng `create_file` để tạo một tạo tác. Điều này cung cấp cho cả Claude và người dùng một giàn giáo để làm việc.

Thông báo cho họ rằng cấu trúc ban đầu với các chỗ giữ chỗ cho tất cả các phần sẽ được tạo.

Tạo tạo tác với tất cả các tiêu đề phần và văn bản giữ chỗ ngắn gọn như "[Sẽ được viết]" hoặc "[Nội dung ở đây]".

Cung cấp liên kết giàn giáo và chỉ ra rằng đã đến lúc điền vào từng phần.

**Nếu không có quyền truy cập vào tạo tác:**
Tạo một tệp markdown trong thư mục làm việc. Đặt tên phù hợp (ví dụ: `decision-doc.md`, `technical-spec.md`).

Thông báo cho họ rằng cấu trúc ban đầu với các chỗ giữ chỗ cho tất cả các phần sẽ được tạo.

Tạo tệp với tất cả các tiêu đề phần và văn bản giữ chỗ.

Xác nhận tên tệp đã được tạo và chỉ ra rằng đã đến lúc điền vào từng phần.

**Đối với từng phần:**

### Bước 1: Câu hỏi Làm rõ

Thông báo công việc sẽ bắt đầu trên phần [TÊN PHẦN]. Đặt 5-10 câu hỏi làm rõ về những gì nên được bao gồm:

Tạo 5-10 câu hỏi cụ thể dựa trên bối cảnh và mục đích phần.

Thông báo cho họ rằng họ có thể trả lời vắn tắt hoặc chỉ ra những gì quan trọng cần đề cập.

### Bước 2: Brainstorming

Đối với phần [TÊN PHẦN], brainstorm [5-20] điều có thể được bao gồm, tùy thuộc vào độ phức tạp của phần. Tìm kiếm:

- Bối cảnh đã chia sẻ có thể đã bị quên
- Các góc độ hoặc cân nhắc chưa được đề cập

Tạo 5-20 tùy chọn được đánh số dựa trên độ phức tạp của phần. Cuối cùng, đề nghị brainstorm thêm nếu họ muốn có thêm tùy chọn.

### Bước 3: Chọn lọc (Curation)

Hỏi điểm nào nên được giữ, xóa hoặc kết hợp. Yêu cầu lý do ngắn gọn để giúp tìm hiểu ưu tiên cho các phần tiếp theo.

Cung cấp ví dụ:

- "Giữ 1,4,7,9"
- "Xóa 3 (trùng lặp 1)"
- "Xóa 6 (đối tượng đã biết điều này)"
- "Kết hợp 11 và 12"

**Nếu người dùng đưa ra phản hồi tự do** (ví dụ: "trông ổn" hoặc "tôi thích hầu hết nhưng...") thay vì các lựa chọn được đánh số, hãy trích xuất sở thích của họ và tiếp tục. Phân tích những gì họ muốn giữ/xóa/thay đổi và áp dụng nó.

### Bước 4: Kiểm tra Khoảng trống (Gap Check)

Dựa trên những gì họ đã chọn, hỏi xem có gì quan trọng bị thiếu cho phần [TÊN PHẦN] không.

### Bước 5: Soạn thảo

Sử dụng `str_replace` để thay thế văn bản giữ chỗ cho phần này bằng nội dung đã soạn thảo thực tế.

Thông báo phần [TÊN PHẦN] sẽ được soạn thảo ngay bây giờ dựa trên những gì họ đã chọn.

**Nếu sử dụng tạo tác:**
Sau khi soạn thảo, cung cấp liên kết đến tạo tác.

Yêu cầu họ đọc qua và chỉ ra những gì cần thay đổi. Lưu ý rằng việc cụ thể giúp học hỏi cho các phần tiếp theo.

**Nếu sử dụng tệp (không có tạo tác):**
Sau khi soạn thảo, xác nhận hoàn thành.

Thông báo cho họ rằng phần [TÊN PHẦN] đã được soạn thảo trong [tên tệp]. Yêu cầu họ đọc qua và chỉ ra những gì cần thay đổi. Lưu ý rằng việc cụ thể giúp học hỏi cho các phần tiếp theo.

**Hướng dẫn chính cho người dùng (bao gồm khi soạn thảo phần đầu tiên):**
Cung cấp một ghi chú: Thay vì chỉnh sửa tài liệu trực tiếp, hãy yêu cầu họ chỉ ra những gì cần thay đổi. Điều này giúp tìm hiểu phong cách của họ cho các phần trong tương lai. Ví dụ: "Xóa gạch đầu dòng X - đã được bao gồm bởi Y" hoặc "Làm cho đoạn thứ ba ngắn gọn hơn".

### Bước 6: Tinh chỉnh Lặp lại

Khi người dùng cung cấp phản hồi:

- Sử dụng `str_replace` để thực hiện chỉnh sửa (không bao giờ in lại toàn bộ tài liệu)
- **Nếu sử dụng tạo tác:** Cung cấp liên kết đến tạo tác sau mỗi lần chỉnh sửa
- **Nếu sử dụng tệp:** Chỉ cần xác nhận chỉnh sửa đã hoàn tất
- Nếu người dùng chỉnh sửa tài liệu trực tiếp và yêu cầu đọc nó: ghi nhớ các thay đổi họ đã thực hiện và ghi nhớ chúng cho các phần trong tương lai (điều này cho thấy sở thích của họ)

**Tiếp tục lặp lại** cho đến khi người dùng hài lòng với phần đó.

### Kiểm tra Chất lượng

Sau 3 lần lặp liên tiếp không có thay đổi đáng kể, hỏi xem có gì có thể loại bỏ mà không làm mất thông tin quan trọng không.

Khi phần đã xong, xác nhận [TÊN PHẦN] đã hoàn thành. Hỏi xem có sẵn sàng chuyển sang phần tiếp theo không.

**Lặp lại cho tất cả các phần.**

### Gần Hoàn thành

Khi gần hoàn thành (80%+ các phần đã xong), thông báo ý định đọc lại toàn bộ tài liệu và kiểm tra:

- Luồng và tính nhất quán giữa các phần
- Sự dư thừa hoặc mâu thuẫn
- Bất cứ điều gì cảm thấy giống như "rác" hoặc chất độn chung chung
- Liệu mỗi câu có mang trọng lượng không

Đọc toàn bộ tài liệu và cung cấp phản hồi.

**Khi tất cả các phần đã được soạn thảo và tinh chỉnh:**
Thông báo tất cả các phần đã được soạn thảo. Chỉ ra ý định xem xét tài liệu hoàn chỉnh một lần nữa.

Xem xét sự mạch lạc tổng thể, luồng, tính đầy đủ.

Cung cấp bất kỳ đề xuất cuối cùng nào.

Hỏi xem có sẵn sàng chuyển sang Kiểm thử Người đọc không, hay họ muốn tinh chỉnh bất cứ điều gì khác.

## Giai đoạn 3: Kiểm thử Người đọc

**Mục tiêu:** Kiểm thử tài liệu với một Claude mới (không có bối cảnh bị rò rỉ) để xác minh nó hoạt động cho người đọc.

**Hướng dẫn cho người dùng:**
Giải thích rằng việc kiểm thử bây giờ sẽ diễn ra để xem liệu tài liệu có thực sự hoạt động cho người đọc hay không. Điều này bắt được các điểm mù - những điều có ý nghĩa với tác giả nhưng có thể gây nhầm lẫn cho người khác.

### Cách tiếp cận Kiểm thử

**Nếu quyền truy cập vào sub-agents khả dụng (ví dụ: trong Claude Code):**

Thực hiện kiểm thử trực tiếp mà không cần sự tham gia của người dùng.

### Bước 1: Dự đoán Câu hỏi Người đọc

Thông báo ý định dự đoán những câu hỏi mà người đọc có thể hỏi khi cố gắng khám phá tài liệu này.

Tạo 5-10 câu hỏi mà người đọc thực tế sẽ hỏi.

### Bước 2: Kiểm thử với Sub-Agent

Thông báo rằng các câu hỏi này sẽ được kiểm thử với một phiên bản Claude mới (không có bối cảnh từ cuộc trò chuyện này).

Đối với mỗi câu hỏi, gọi một sub-agent chỉ với nội dung tài liệu và câu hỏi.

Tóm tắt những gì Reader Claude đã làm đúng/sai cho mỗi câu hỏi.

### Bước 3: Chạy Các Kiểm tra Bổ sung

Thông báo các kiểm tra bổ sung sẽ được thực hiện.

Gọi sub-agent để kiểm tra sự mơ hồ, giả định sai, mâu thuẫn.

Tóm tắt bất kỳ vấn đề nào được tìm thấy.

### Bước 4: Báo cáo và Sửa

Nếu tìm thấy vấn đề:
Báo cáo rằng Reader Claude đã gặp khó khăn với các vấn đề cụ thể.

Liệt kê các vấn đề cụ thể.

Chỉ ra ý định sửa chữa những khoảng trống này.

Quay lại tinh chỉnh cho các phần có vấn đề.

---

**Nếu không có quyền truy cập vào sub-agents (ví dụ: giao diện web claude.ai):**

Người dùng sẽ cần thực hiện kiểm thử thủ công.

### Bước 1: Dự đoán Câu hỏi Người đọc

Hỏi những câu hỏi nào mọi người có thể hỏi khi cố gắng khám phá tài liệu này. Họ sẽ nhập gì vào Claude.ai?

Tạo 5-10 câu hỏi mà người đọc thực tế sẽ hỏi.

### Bước 2: Thiết lập Kiểm thử

Cung cấp hướng dẫn kiểm thử:

1. Mở một cuộc trò chuyện Claude mới: https://claude.ai
2. Dán hoặc chia sẻ nội dung tài liệu (nếu sử dụng nền tảng tài liệu được chia sẻ có bật trình kết nối, hãy cung cấp liên kết)
3. Hỏi Reader Claude các câu hỏi đã tạo

Đối với mỗi câu hỏi, hướng dẫn Reader Claude cung cấp:

- Câu trả lời
- Liệu có bất cứ điều gì mơ hồ hoặc không rõ ràng không
- Kiến thức/bối cảnh nào tài liệu giả định đã được biết

Kiểm tra xem Reader Claude có đưa ra câu trả lời đúng hay hiểu sai bất cứ điều gì không.

### Bước 3: Các Kiểm tra Bổ sung

Cũng hỏi Reader Claude:

- "Điều gì trong tài liệu này có thể mơ hồ hoặc không rõ ràng cho người đọc?"
- "Tài liệu này giả định người đọc đã có kiến thức hoặc bối cảnh nào?"
- "Có bất kỳ mâu thuẫn hoặc sự không nhất quán nội bộ nào không?"

### Bước 4: Lặp lại Dựa trên Kết quả

Hỏi Reader Claude đã sai hoặc gặp khó khăn với điều gì. Chỉ ra ý định sửa chữa những khoảng trống đó.

Quay lại tinh chỉnh cho bất kỳ phần nào có vấn đề.

---

### Điều kiện Thoát (Cả hai Cách tiếp cận)

Khi Reader Claude liên tục trả lời đúng các câu hỏi và không đưa ra các khoảng trống hoặc sự mơ hồ mới, tài liệu đã sẵn sàng.

## Đánh giá Cuối cùng

Khi Kiểm thử Người đọc vượt qua:
Thông báo tài liệu đã vượt qua kiểm thử Reader Claude. Trước khi hoàn thành:

1. Khuyên họ tự đọc lại lần cuối - họ sở hữu tài liệu này và chịu trách nhiệm về chất lượng của nó
2. Đề xuất kiểm tra kỹ bất kỳ sự thật, liên kết, hoặc chi tiết kỹ thuật nào
3. Yêu cầu họ xác minh nó đạt được tác động họ muốn

Hỏi xem họ có muốn đánh giá thêm một lần nữa không, hay công việc đã hoàn tất.

**Nếu người dùng muốn đánh giá cuối cùng, hãy cung cấp nó. Nếu không:**
Thông báo hoàn thành tài liệu. Cung cấp một vài mẹo cuối cùng:

- Cân nhắc liên kết cuộc trò chuyện này trong phụ lục để người đọc có thể xem cách tài liệu được phát triển
- Sử dụng phụ lục để cung cấp chiều sâu mà không làm phình to tài liệu chính
- Cập nhật tài liệu khi nhận được phản hồi từ người đọc thực

## Mẹo để Hướng dẫn Hiệu quả

**Giọng điệu:**

- Trực tiếp và theo thủ tục
- Giải thích lý do ngắn gọn khi nó ảnh hưởng đến hành vi của người dùng
- Đừng cố "bán" cách tiếp cận - chỉ cần thực hiện nó

**Xử lý Sai lệch:**

- Nếu người dùng muốn bỏ qua một giai đoạn: Hỏi xem họ có muốn bỏ qua điều này và viết tự do không
- Nếu người dùng có vẻ thất vọng: Thừa nhận điều này đang mất nhiều thời gian hơn dự kiến. Đề xuất các cách để di chuyển nhanh hơn
- Luôn trao quyền cho người dùng điều chỉnh quy trình

**Quản lý Bối cảnh:**

- Trong suốt quá trình, nếu thiếu bối cảnh về điều gì đó được đề cập, hãy chủ động hỏi
- Đừng để các khoảng trống tích tụ - giải quyết chúng khi chúng xuất hiện

**Quản lý Tạo tác:**

- Sử dụng `create_file` để soạn thảo các phần đầy đủ
- Sử dụng `str_replace` cho tất cả các chỉnh sửa
- Cung cấp liên kết tạo tác sau mỗi thay đổi
- Không bao giờ sử dụng tạo tác cho danh sách brainstorming - đó chỉ là cuộc trò chuyện

**Chất lượng hơn Tốc độ:**

- Đừng vội vàng qua các giai đoạn
- Mỗi lần lặp lại nên tạo ra những cải tiến có ý nghĩa
- Mục tiêu là một tài liệu thực sự hoạt động cho người đọc
