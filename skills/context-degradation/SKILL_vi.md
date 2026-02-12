---
name: context-degradation
description: "Nhận biết các mô hình suy giảm bối cảnh: mất ở giữa, nhiễm độc, mất tập trung và xung đột"
source: "https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering/tree/main/skills/context-degradation"
risk: safe
---

## Khi nào Sử dụng Kỹ năng Này

Nhận biết các mô hình suy giảm bối cảnh: mất ở giữa, nhiễm độc, mất tập trung và xung đột

Sử dụng kỹ năng này khi làm việc với nhận biết các mô hình suy giảm bối cảnh: mất ở giữa, nhiễm độc, mất tập trung và xung đột.

# Các Mô hình Suy giảm Bối cảnh

Các mô hình ngôn ngữ thể hiện các mô hình suy giảm có thể dự đoán được khi độ dài bối cảnh tăng lên. Hiểu các mô hình này là điều cần thiết để chẩn đoán các thất bại và thiết kế các hệ thống linh hoạt. Suy giảm bối cảnh không phải là một trạng thái nhị phân mà là một sự liên tục của suy giảm hiệu suất biểu hiện theo một số cách riêng biệt.

## Khi nào Kích hoạt

Kích hoạt kỹ năng này khi:

- Hiệu suất tác nhân suy giảm bất ngờ trong các cuộc trò chuyện dài
- Gỡ lỗi các trường hợp tác nhân tạo ra kết quả không chính xác hoặc không liên quan
- Thiết kế các hệ thống phải xử lý bối cảnh lớn một cách đáng tin cậy
- Đánh giá các lựa chọn kỹ thuật bối cảnh cho các hệ thống sản xuất
- Điều tra hiện tượng "mất ở giữa" trong đầu ra của tác nhân
- Phân tích các thất bại liên quan đến bối cảnh trong hành vi của tác nhân

## Các Khái niệm Cốt lõi

Suy giảm bối cảnh biểu hiện qua một số mô hình riêng biệt. Hiện tượng mất ở giữa (lost-in-middle) khiến thông tin ở trung tâm bối cảnh nhận được ít sự chú ý hơn. Nhiễm độc bối cảnh (context poisoning) xảy ra khi các lỗi cộng gộp thông qua tham chiếu lặp đi lặp lại. Mất tập trung bối cảnh (context distraction) xảy ra khi thông tin không liên quan lấn át nội dung liên quan. Nhầm lẫn bối cảnh (context confusion) nảy sinh khi mô hình không thể xác định bối cảnh nào áp dụng. Xung đột bối cảnh (context clash) phát triển khi thông tin tích lũy mâu thuẫn trực tiếp.

Các mô hình này có thể dự đoán được và có thể được giảm thiểu thông qua các mô hình kiến trúc như nén chặt, che giấu, phân vùng và cô lập.

## Chủ đề Chi tiết

### Hiện tượng Mất ở Giữa (Lost-in-Middle)

Mô hình suy giảm được ghi nhận rõ nhất là hiệu ứng "mất ở giữa", nơi các mô hình thể hiện các đường cong chú ý hình chữ U. Thông tin ở đầu và cuối bối cảnh nhận được sự chú ý đáng tin cậy, trong khi thông tin bị chôn vùi ở giữa chịu độ chính xác gợi nhớ giảm đáng kể.

**Bằng chứng Thực nghiệm**
Nghiên cứu chứng minh rằng thông tin liên quan được đặt ở giữa bối cảnh trải qua độ chính xác gợi nhớ thấp hơn 10-40% so với thông tin tương tự ở đầu hoặc cuối. Đây không phải là thất bại của mô hình mà là hậu quả của cơ chế chú ý và phân phối dữ liệu đào tạo.

Các mô hình phân bổ sự chú ý lớn cho token đầu tiên (thường là token BOS) để ổn định các trạng thái bên trong. Điều này tạo ra một "bồn chứa chú ý" (attention sink) hấp thụ ngân sách chú ý. Khi bối cảnh phát triển, ngân sách hạn chế bị kéo căng hơn, và các token ở giữa không thu được đủ trọng lượng chú ý để truy xuất đáng tin cậy.

**Ý nghĩa Thực tế**
Thiết kế vị trí bối cảnh với các mô hình chú ý trong tâm trí. Đặt thông tin quan trọng ở đầu hoặc cuối bối cảnh. Xem xét liệu thông tin sẽ được truy vấn trực tiếp hay cần hỗ trợ lý luận—nếu là trường hợp sau, vị trí ít quan trọng hơn nhưng chất lượng tín hiệu tổng thể quan trọng hơn.

Đối với các tài liệu hoặc cuộc trò chuyện dài, sử dụng các cấu trúc tóm tắt hiển thị thông tin chính tại các vị trí được ưu tiên chú ý. Sử dụng các tiêu đề phần và chuyển tiếp rõ ràng để giúp các mô hình điều hướng cấu trúc.

### Nhiễm độc Bối cảnh (Context Poisoning)

Nhiễm độc bối cảnh xảy ra khi ảo giác, lỗi hoặc thông tin không chính xác đi vào bối cảnh và cộng gộp thông qua tham chiếu lặp đi lặp lại. Sau khi bị nhiễm độc, bối cảnh tạo ra các vòng phản hồi củng cố niềm tin sai lầm.

**Cách Nhiễm độc Xảy ra**
Nhiễm độc thường đi vào qua ba con đường. Thứ nhất, đầu ra công cụ có thể chứa lỗi hoặc định dạng bất ngờ mà mô hình chấp nhận là sự thật cơ sở. Thứ hai, các tài liệu được truy xuất có thể chứa thông tin không chính xác hoặc lỗi thời mà mô hình kết hợp vào lý luận. Thứ ba, các tóm tắt do mô hình tạo ra hoặc đầu ra trung gian có thể giới thiệu các ảo giác tồn tại trong bối cảnh.

Hiệu ứng cộng gộp là nghiêm trọng. Nếu phần mục tiêu của một tác nhân bị nhiễm độc, nó phát triển các chiến lược tốn nhiều nỗ lực để hoàn tác. Mỗi quyết định tiếp theo tham chiếu đến nội dung bị nhiễm độc, củng cố các giả định sai lầm.

**Phát hiện và Phục hồi**
Theo dõi các triệu chứng bao gồm chất lượng đầu ra bị suy giảm trên các nhiệm vụ đã thành công trước đó, sự sai lệch công cụ nơi các tác nhân gọi sai công cụ hoặc tham số, và các ảo giác tồn tại bất chấp nỗ lực sửa chữa. Khi các triệu chứng này xuất hiện, hãy xem xét nhiễm độc bối cảnh.

Phục hồi yêu cầu loại bỏ hoặc thay thế nội dung bị nhiễm độc. Điều này có thể liên quan đến việc cắt bối cảnh về trước điểm nhiễm độc, ghi chú rõ ràng việc nhiễm độc trong bối cảnh và yêu cầu đánh giá lại, hoặc khởi động lại với bối cảnh sạch và chỉ bảo tồn thông tin đã được xác minh.

### Mất tập trung Bối cảnh (Context Distraction)

Mất tập trung bối cảnh xuất hiện khi bối cảnh phát triển quá dài khiến các mô hình tập trung quá mức vào thông tin được cung cấp thay vì kiến thức đào tạo của chúng. Mô hình chú ý đến mọi thứ trong bối cảnh bất kể mức độ liên quan, và điều này tạo ra áp lực sử dụng thông tin được cung cấp ngay cả khi kiến thức bên trong chính xác hơn.

**Hiệu ứng Gây xao nhãng**
Nghiên cứu cho thấy ngay cả một tài liệu không liên quan duy nhất trong bối cảnh cũng làm giảm hiệu suất trên các nhiệm vụ liên quan đến tài liệu liên quan. Nhiều yếu tố gây xao nhãng làm tăng thêm sự suy giảm. Hiệu ứng không phải là về tiếng ồn theo nghĩa tuyệt đối mà là về phân bổ sự chú ý—thông tin không liên quan cạnh tranh với thông tin liên quan cho ngân sách chú ý hạn chế.

Các mô hình không có cơ chế để "bỏ qua" bối cảnh không liên quan. Chúng phải chú ý đến mọi thứ được cung cấp, và nghĩa vụ này tạo ra sự mất tập trung ngay cả khi thông tin không liên quan rõ ràng không hữu ích.

**Chiến lược Giảm thiểu**
Giảm thiểu sự mất tập trung thông qua việc quản lý cẩn thận những gì đi vào bối cảnh. Áp dụng lọc mức độ liên quan trước khi tải các tài liệu được truy xuất. Sử dụng không gian tên và tổ chức để làm cho các phần không liên quan dễ dàng bị bỏ qua về mặt cấu trúc. Xem xét liệu thông tin có thực sự cần phải ở trong bối cảnh hay có thể được truy cập thông qua các cuộc gọi công cụ thay thế.

### Nhầm lẫn Bối cảnh (Context Confusion)

Nhầm lẫn bối cảnh nảy sinh khi thông tin không liên quan ảnh hưởng đến phản hồi theo cách làm giảm chất lượng. Điều này liên quan đến sự mất tập trung nhưng khác biệt—nhầm lẫn liên quan đến ảnh hưởng của bối cảnh đối với hành vi mô hình thay vì phân bổ sự chú ý.

Nếu bạn đặt cái gì đó vào bối cảnh, mô hình phải chú ý đến nó. Mô hình có thể kết hợp thông tin không liên quan, sử dụng định nghĩa công cụ không phù hợp hoặc áp dụng các ràng buộc đến từ các bối cảnh khác nhau. Nhầm lẫn đặc biệt có vấn đề khi bối cảnh chứa nhiều loại nhiệm vụ hoặc khi chuyển đổi giữa các nhiệm vụ trong một phiên duy nhất.

**Dấu hiệu Nhầm lẫn**
Theo dõi các phản hồi giải quyết sai khía cạnh của truy vấn, các cuộc gọi công cụ có vẻ phù hợp cho một nhiệm vụ khác, hoặc đầu ra trộn lẫn các yêu cầu từ nhiều nguồn. Những điều này cho thấy sự nhầm lẫn về bối cảnh nào áp dụng cho tình huống hiện tại.

**Giải pháp Kiến trúc**
Các giải pháp kiến trúc bao gồm phân đoạn nhiệm vụ rõ ràng nơi các nhiệm vụ khác nhau nhận được các cửa sổ bối cảnh khác nhau, chuyển tiếp rõ ràng giữa các bối cảnh nhiệm vụ, và quản lý trạng thái cô lập bối cảnh cho các mục tiêu khác nhau.

### Xung đột Bối cảnh (Context Clash)

Xung đột bối cảnh phát triển khi thông tin tích lũy xung đột trực tiếp, tạo ra hướng dẫn mâu thuẫn làm trật bánh lý luận. Điều này khác với nhiễm độc nơi một phần thông tin không chính xác—trong xung đột, nhiều phần thông tin chính xác mâu thuẫn với nhau.

**Nguồn gốc của Xung đột**
Xung đột thường nảy sinh từ việc truy xuất đa nguồn nơi các nguồn khác nhau có thông tin mâu thuẫn, xung đột phiên bản nơi thông tin lỗi thời và hiện tại đều xuất hiện trong bối cảnh, và xung đột quan điểm nơi các quan điểm khác nhau đều hợp lệ nhưng không tương thích.

**Cách tiếp cận Giải quyết**
Các cách tiếp cận giải quyết bao gồm đánh dấu xung đột rõ ràng xác định các mâu thuẫn và yêu cầu làm rõ, các quy tắc ưu tiên thiết lập nguồn nào được ưu tiên, và lọc phiên bản loại trừ thông tin lỗi thời khỏi bối cảnh.

### Điểm chuẩn Thực nghiệm và Ngưỡng

Nghiên cứu cung cấp dữ liệu cụ thể về các mô hình suy giảm thông báo cho các quyết định thiết kế.

**Phát hiện Điểm chuẩn RULER**
Điểm chuẩn RULER đưa ra những phát hiện nghiêm túc: chỉ 50% các mô hình tuyên bố bối cảnh 32K+ duy trì hiệu suất thỏa đáng ở 32K token. GPT-5.2 cho thấy ít suy giảm nhất trong số các mô hình hiện tại, trong khi nhiều mô hình vẫn giảm 30+ điểm ở các bối cảnh mở rộng. Điểm gần như hoàn hảo trên các bài kiểm tra kim trong đống cỏ khô đơn giản không chuyển thành sự hiểu biết bối cảnh dài thực sự.

**Ngưỡng Suy giảm Cụ thể của Mô hình**
| Mô hình | Khởi phát Suy giảm | Suy giảm Nghiêm trọng | Ghi chú |
|---------|--------------------|-----------------------|---------|
| GPT-5.2 | ~64K tokens | ~200K tokens | Kháng suy giảm tổng thể tốt nhất với chế độ tư duy |
| Claude Opus 4.5 | ~100K tokens | ~180K tokens | Cửa sổ bối cảnh 200K, quản lý chú ý mạnh mẽ |
| Claude Sonnet 4.5 | ~80K tokens | ~150K tokens | Tối ưu hóa cho các tác nhân và nhiệm vụ mã hóa |
| Gemini 3 Pro | ~500K tokens | ~800K tokens | Cửa sổ bối cảnh 1M, đa phương thức gốc |
| Gemini 3 Flash | ~300K tokens | ~600K tokens | Tốc độ gấp 3 lần Gemini 2.5, 81.2% MMMU-Pro |

**Mô hình Hành vi Cụ thể của Mô hình**
Các mô hình khác nhau thể hiện các chế độ thất bại riêng biệt dưới áp lực bối cảnh:

- **Dòng Claude 4.5**: Tỷ lệ ảo giác thấp nhất với độ không chắc chắn được hiệu chỉnh. Claude Opus 4.5 đạt 80.9% trên SWE-bench Verified. Có xu hướng từ chối hoặc yêu cầu làm rõ thay vì bịa đặt.
- **GPT-5.2**: Có sẵn hai chế độ - tức thì (nhanh) và tư duy (lý luận). Chế độ tư duy giảm ảo giác thông qua xác minh từng bước nhưng tăng độ trễ.
- **Gemini 3 Pro/Flash**: Đa phương thức gốc với cửa sổ bối cảnh 1M. Gemini 3 Flash cung cấp cải thiện tốc độ gấp 3 lần so với thế hệ trước. Mạnh về lý luận đa phương thức trên văn bản, mã, hình ảnh, âm thanh và video.

Những mô hình này thông báo cho việc lựa chọn mô hình cho các trường hợp sử dụng khác nhau. Các nhiệm vụ rủi ro cao được hưởng lợi từ cách tiếp cận thận trọng của Claude 4.5 hoặc chế độ tư duy của GPT-5.2; các nhiệm vụ quan trọng về tốc độ có thể sử dụng các chế độ tức thì.

### Những Phát hiện Phản trực giác

Nghiên cứu tiết lộ một số mô hình phản trực giác thách thức các giả định về quản lý bối cảnh.

**Đống Cỏ Khô Xáo trộn Vượt trội hơn Đống Liên kết**
Các nghiên cứu cho thấy các đống cỏ khô xáo trộn (không liên kết) tạo ra hiệu suất tốt hơn so với các đống cỏ khô liên kết logic. Điều này cho thấy bối cảnh liên kết có thể tạo ra các liên kết sai gây nhầm lẫn cho việc truy xuất, trong khi bối cảnh không liên kết buộc các mô hình phải dựa vào khớp chính xác.

**Các Yếu tố Gây xao nhãng Đơn lẻ Có Tác động Quá lớn**
Ngay cả một tài liệu không liên quan duy nhất cũng làm giảm hiệu suất đáng kể. Hiệu ứng không tỷ lệ thuận với lượng nhiễu mà tuân theo hàm bước, nơi sự hiện diện của bất kỳ yếu tố gây xao nhãng nào kích hoạt sự suy giảm.

**Tương quan Tương tự Kim-Câu hỏi**
Độ tương tự thấp hơn giữa các cặp kim và câu hỏi cho thấy sự suy giảm nhanh hơn với độ dài bối cảnh. Các nhiệm vụ yêu cầu suy luận qua nội dung không giống nhau đặc biệt dễ bị tổn thương.

### Khi Bối cảnh Lớn hơn Gây hại

Cửa sổ bối cảnh lớn hơn không cải thiện hiệu suất một cách đồng nhất. Trong nhiều trường hợp, bối cảnh lớn hơn tạo ra các vấn đề mới vượt quá lợi ích.

**Đường cong Suy giảm Hiệu suất**
Các mô hình thể hiện sự suy giảm phi tuyến tính với độ dài bối cảnh. Hiệu suất vẫn ổn định đến một ngưỡng, sau đó suy giảm nhanh chóng. Ngưỡng này thay đổi theo độ phức tạp của mô hình và nhiệm vụ. Đối với nhiều mô hình, sự suy giảm có ý nghĩa bắt đầu khoảng 8,000-16,000 token ngay cả khi cửa sổ bối cảnh hỗ trợ kích thước lớn hơn nhiều.

**Ý nghĩa Chi phí**
Chi phí xử lý tăng không cân xứng với độ dài bối cảnh. Chi phí để xử lý bối cảnh 400K token không phải là gấp đôi chi phí của 200K—nó tăng theo cấp số nhân cả về thời gian và tài nguyên tính toán. Đối với nhiều ứng dụng, điều này làm cho việc xử lý bối cảnh lớn trở nên không thực tế về mặt kinh tế.

**Ẩn dụ Tải Nhận thức**
Ngay cả với bối cảnh vô hạn, việc yêu cầu một mô hình duy nhất duy trì chất lượng nhất quán qua hàng chục nhiệm vụ độc lập tạo ra một nút thắt cổ chai nhận thức. Mô hình phải liên tục chuyển đổi bối cảnh giữa các mục, duy trì khuôn khổ so sánh và đảm bảo tính nhất quán về phong cách. Đây không phải là vấn đề mà nhiều bối cảnh hơn giải quyết được.

## Hướng dẫn Thực tế

### Cách tiếp cận Bốn Nhóm

Bốn chiến lược giải quyết các khía cạnh khác nhau của sự suy giảm bối cảnh:

**Viết (Write)**: Lưu bối cảnh bên ngoài cửa sổ sử dụng scratchpad, hệ thống tệp hoặc lưu trữ bên ngoài. Điều này giữ cho bối cảnh hoạt động gọn gàng trong khi vẫn bảo tồn quyền truy cập thông tin.

**Chọn (Select)**: Đưa bối cảnh liên quan vào cửa sổ thông qua truy xuất, lọc và ưu tiên. Điều này giải quyết sự mất tập trung bằng cách loại trừ thông tin không liên quan.

**Nén (Compress)**: Giảm token trong khi bảo tồn thông tin thông qua tóm tắt, trừu tượng hóa và che giấu quan sát. Điều này mở rộng khả năng bối cảnh hiệu quả.

**Cô lập (Isolate)**: Tách bối cảnh qua các phân hệ tác nhân hoặc phiên để ngăn chặn bất kỳ bối cảnh đơn lẻ nào phát triển đủ lớn để gây suy giảm. Đây là chiến lược tích cực nhất nhưng thường hiệu quả nhất.

### Các Mô hình Kiến trúc

Triển khai các chiến lược này thông qua các mô hình kiến trúc cụ thể. Sử dụng tải bối cảnh đúng lúc (just-in-time) để truy xuất thông tin chỉ khi cần thiết. Sử dụng che giấu quan sát để thay thế đầu ra công cụ dài dòng bằng các tham chiếu nhỏ gọn. Sử dụng kiến trúc phân hệ tác nhân để cô lập bối cảnh cho các nhiệm vụ khác nhau. Sử dụng nén chặt để tóm tắt bối cảnh đang phát triển trước khi nó vượt quá giới hạn.

## Ví dụ

**Ví dụ 1: Phát hiện Suy giảm**

```yaml
# Bối cảnh phát triển trong cuộc trò chuyện dài
turn_1: 1000 tokens
turn_5: 8000 tokens
turn_10: 25000 tokens
turn_20: 60000 tokens (suy giảm bắt đầu)
turn_30: 90000 tokens (suy giảm đáng kể)
```

**Ví dụ 2: Giảm thiểu Mất ở Giữa**

```markdown
# Tổ chức bối cảnh với thông tin quan trọng ở các cạnh

[CURRENT TASK] # Ở đầu

- Goal: Generate quarterly report
- Deadline: End of week

[DETAILED CONTEXT] # Ở giữa (ít chú ý hơn)

- 50 pages of data
- Multiple analysis sections
- Supporting evidence

[KEY FINDINGS] # Ở cuối

- Revenue up 15%
- Costs down 8%
- Growth in Region A
```

## Hướng dẫn

1. Giám sát độ dài bối cảnh và tương quan hiệu suất trong quá trình phát triển
2. Đặt thông tin quan trọng ở đầu hoặc cuối bối cảnh
3. Triển khai các trình kích hoạt nén chặt trước khi sự suy giảm trở nên nghiêm trọng
4. Xác thực các tài liệu được truy xuất về độ chính xác trước khi thêm vào bối cảnh
5. Sử dụng lập phiên bản để ngăn chặn thông tin lỗi thời gây ra xung đột
6. Phân đoạn các nhiệm vụ để ngăn chặn sự nhầm lẫn bối cảnh giữa các mục tiêu khác nhau
7. Thiết kế cho sự suy giảm duyên dáng thay vì giả định các điều kiện hoàn hảo
8. Thử nghiệm với các bối cảnh lớn hơn dần dần để tìm ngưỡng suy giảm

## Tích hợp

Kỹ năng này xây dựng dựa trên context-fundamentals và nên được nghiên cứu sau khi hiểu các khái niệm bối cảnh cơ bản. Nó kết nối với:

- context-optimization - Các kỹ thuật giảm thiểu sự suy giảm
- multi-agent-patterns - Sử dụng cô lập để ngăn chặn sự suy giảm
- evaluation - Đo lường và phát hiện sự suy giảm trong sản xuất

## Tham khảo

Tham khảo nội bộ:

- [Degradation Patterns Reference](./references/patterns.md) - Tham khảo kỹ thuật chi tiết

Các kỹ năng liên quan trong bộ sưu tập này:

- context-fundamentals - Cơ bản về bối cảnh
- context-optimization - Các kỹ thuật giảm thiểu
- evaluation - Phát hiện và đo lường

Tài nguyên bên ngoài:

- Research on attention mechanisms and context window limitations
- Studies on the "lost-in-middle" phenomenon
- Production engineering guides from AI labs
