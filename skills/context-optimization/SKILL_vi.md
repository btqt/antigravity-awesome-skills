---
name: context-optimization
description: "Áp dụng các chiến lược nén chặt, che giấu và lưu trữ bộ nhớ đệm"
source: "https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering/tree/main/skills/context-optimization"
risk: safe
---

## Khi nào Sử dụng Kỹ năng Này

Áp dụng các chiến lược nén chặt, che giấu và lưu trữ bộ nhớ đệm

Sử dụng kỹ năng này khi làm việc với áp dụng các chiến lược nén chặt, che giấu và lưu trữ bộ nhớ đệm.

# Các Kỹ thuật Tối ưu hóa Bối cảnh

Tối ưu hóa bối cảnh mở rộng khả năng hiệu quả của các cửa sổ bối cảnh hạn chế thông qua nén chiến lược, che giấu, lưu trữ bộ nhớ đệm và phân vùng. Mục tiêu không phải là tăng cửa sổ bối cảnh một cách kỳ diệu mà là tận dụng tốt hơn khả năng sẵn có. Tối ưu hóa hiệu quả có thể tăng gấp đôi hoặc gấp ba khả năng bối cảnh hiệu quả mà không cần các mô hình lớn hơn hoặc bối cảnh dài hơn.

## Khi nào Kích hoạt

Kích hoạt kỹ năng này khi:

- Giới hạn bối cảnh hạn chế độ phức tạp của nhiệm vụ
- Tối ưu hóa để giảm chi phí (ít token hơn = chi phí thấp hơn)
- Giảm độ trễ cho các cuộc trò chuyện dài
- Triển khai các hệ thống tác nhân chạy dài
- Cần xử lý các tài liệu hoặc cuộc trò chuyện lớn hơn
- Xây dựng các hệ thống sản xuất ở quy mô lớn

## Các Khái niệm Cốt lõi

Tối ưu hóa bối cảnh mở rộng khả năng hiệu quả thông qua bốn chiến lược chính: nén chặt (tóm tắt bối cảnh gần giới hạn), che giấu quan sát (thay thế đầu ra dài dòng bằng các tham chiếu), tối ưu hóa bộ nhớ đệm KV (tái sử dụng các tính toán được lưu trong bộ nhớ đệm) và phân vùng bối cảnh (chia công việc qua các bối cảnh cô lập).

Thông tin chi tiết chính là chất lượng bối cảnh quan trọng hơn số lượng. Tối ưu hóa bảo tồn tín hiệu trong khi giảm nhiễu. Nghệ thuật nằm ở việc chọn những gì cần giữ so với những gì cần loại bỏ, và khi nào áp dụng từng kỹ thuật.

## Chủ đề Chi tiết

### Chiến lược Nén chặt (Compaction Strategies)

**Nén chặt là gì**
Nén chặt là thực hành tóm tắt nội dung bối cảnh khi tiếp cận giới hạn, sau đó khởi tạo lại một cửa sổ bối cảnh mới với bản tóm tắt. Điều này chưng cất nội dung của một cửa sổ bối cảnh theo cách có độ trung thực cao, cho phép tác nhân tiếp tục với sự suy giảm hiệu suất tối thiểu.

Nén chặt thường đóng vai trò là đòn bẩy đầu tiên trong tối ưu hóa bối cảnh. Nghệ thuật nằm ở việc chọn những gì cần giữ so với những gì cần loại bỏ.

**Triển khai Nén chặt**
Nén chặt hoạt động bằng cách xác định các phần có thể được nén, tạo các bản tóm tắt nắm bắt các điểm thiết yếu và thay thế nội dung đầy đủ bằng các bản tóm tắt. Ưu tiên nén cho đầu ra công cụ (thay thế bằng bản tóm tắt), các lượt cũ (tóm tắt cuộc trò chuyện sớm), tài liệu được truy xuất (tóm tắt nếu có phiên bản gần đây) và không bao giờ nén lời nhắc hệ thống.

**Tạo Tóm tắt**
Các bản tóm tắt hiệu quả bảo tồn các yếu tố khác nhau tùy thuộc vào loại tin nhắn:

Đầu ra công cụ: Bảo tồn các phát hiện chính, số liệu và kết luận. Loại bỏ đầu ra thô dài dòng.

Các lượt trò chuyện: Bảo tồn các quyết định chính, cam kết và thay đổi bối cảnh. Loại bỏ các phần đệm và qua lại.

Tài liệu được truy xuất: Bảo tồn các sự kiện và tuyên bố chính. Loại bỏ bằng chứng hỗ trợ và sự công phu.

### Che giấu Quan sát (Observation Masking)

**Vấn đề Quan sát**
Đầu ra công cụ có thể bao gồm 80%+ mức sử dụng token trong các quỹ đạo tác nhân. Phần lớn trong số này là đầu ra dài dòng đã phục vụ mục đích của nó. Khi một tác nhân đã sử dụng đầu ra công cụ để đưa ra quyết định, việc giữ đầu ra đầy đủ cung cấp giá trị giảm dần trong khi tiêu thụ bối cảnh đáng kể.

Che giấu quan sát thay thế đầu ra công cụ dài dòng bằng các tham chiếu nhỏ gọn. Thông tin vẫn có thể truy cập nếu cần nhưng không tiêu thụ bối cảnh liên tục.

**Lựa chọn Chiến lược Che giấu**
Không phải tất cả các quan sát đều nên được che giấu như nhau:

Không bao giờ che giấu: Các quan sát quan trọng đối với nhiệm vụ hiện tại, các quan sát từ lượt gần đây nhất, các quan sát được sử dụng trong lý luận tích cực.

Cân nhắc che giấu: Các quan sát từ 3+ lượt trước, đầu ra dài dòng với các điểm chính có thể trích xuất, các quan sát có mục đích đã được phục vụ.

Luôn che giấu: Đầu ra lặp đi lặp lại, tiêu đề/chân trang soạn sẵn, đầu ra đã được tóm tắt trong cuộc trò chuyện.

### Tối ưu hóa Bộ nhớ đệm KV (KV-Cache Optimization)

**Hiểu Bộ nhớ đệm KV**
Bộ nhớ đệm KV lưu trữ các tensor Khóa và Giá trị được tính toán trong quá trình suy luận, tăng tuyến tính với độ dài chuỗi. Việc lưu trữ bộ nhớ đệm KV qua các yêu cầu chia sẻ các tiền tố giống hệt nhau tránh việc tính toán lại.

Lưu trữ bộ nhớ đệm tiền tố tái sử dụng các khối KV qua các yêu cầu với các tiền tố giống hệt nhau sử dụng khớp khối dựa trên băm. Điều này làm giảm đáng kể chi phí và độ trễ cho các yêu cầu có tiền tố chung như lời nhắc hệ thống.

**Các Mẫu Tối ưu hóa Bộ nhớ đệm**
Tối ưu hóa cho việc lưu trữ bộ nhớ đệm bằng cách sắp xếp lại các yếu tố bối cảnh để tối đa hóa số lần truy cập bộ nhớ đệm. Đặt các yếu tố ổn định trước (lời nhắc hệ thống, định nghĩa công cụ), sau đó là các yếu tố được sử dụng lại thường xuyên, sau đó là các yếu tố duy nhất cuối cùng.

Thiết kế lời nhắc để tối đa hóa sự ổn định của bộ nhớ đệm: tránh nội dung động như dấu thời gian, sử dụng định dạng nhất quán, giữ cấu trúc ổn định qua các phiên.

### Phân vùng Bối cảnh (Context Partitioning)

**Phân vùng Phân hệ Tác nhân**
Hình thức tối ưu hóa bối cảnh tích cực nhất là phân vùng công việc qua các phân hệ tác nhân với bối cảnh cô lập. Mỗi phân hệ tác nhân hoạt động trong một bối cảnh sạch tập trung vào nhiệm vụ phụ của nó mà không mang theo bối cảnh tích lũy từ các nhiệm vụ phụ khác.

Cách tiếp cận này đạt được sự phân tách các mối quan tâm—bối cảnh tìm kiếm chi tiết vẫn bị cô lập trong các phân hệ tác nhân trong khi người điều phối tập trung vào tổng hợp và phân tích.

**Tổng hợp Kết quả**
Tổng hợp kết quả từ các nhiệm vụ phụ được phân vùng bằng cách xác thực tất cả các phân vùng đã hoàn thành, hợp nhất các kết quả tương thích và tóm tắt nếu vẫn còn quá lớn.

### Quản lý Ngân sách

**Phân bổ Ngân sách Bối cảnh**
Thiết kế ngân sách bối cảnh rõ ràng. Phân bổ token cho các danh mục: lời nhắc hệ thống, định nghĩa công cụ, tài liệu được truy xuất, lịch sử tin nhắn và bộ đệm dự trữ. Giám sát việc sử dụng so với ngân sách và kích hoạt tối ưu hóa khi tiếp cận giới hạn.

**Tối ưu hóa Dựa trên Trình kích hoạt**
Giám sát các tín hiệu cho các trình kích hoạt tối ưu hóa: sử dụng token trên 80%, chỉ báo suy giảm và giảm hiệu suất. Áp dụng các kỹ thuật tối ưu hóa phù hợp dựa trên thành phần bối cảnh.

## Hướng dẫn Thực tế

### Khung Quyết định Tối ưu hóa

Khi nào tối ưu hóa:

- Sử dụng bối cảnh vượt quá 70%
- Chất lượng phản hồi suy giảm khi các cuộc trò chuyện kéo dài
- Chi phí tăng do bối cảnh dài
- Độ trễ tăng theo độ dài cuộc trò chuyện

Áp dụng cái gì:

- Đầu ra công cụ chiếm ưu thế: che giấu quan sát
- Tài liệu được truy xuất chiếm ưu thế: tóm tắt hoặc phân vùng
- Lịch sử tin nhắn chiếm ưu thế: nén chặt với tóm tắt
- Nhiều thành phần: kết hợp các chiến lược

### Cân nhắc Hiệu suất

Nén chặt nên đạt được giảm token 50-70% với suy giảm chất lượng dưới 5%. Che giấu nên đạt được giảm 60-80% trong các quan sát bị che giấu. Tối ưu hóa bộ nhớ đệm nên đạt được tỷ lệ truy cập 70%+ cho khối lượng công việc ổn định.

Giám sát và lặp lại các chiến lược tối ưu hóa dựa trên hiệu quả đo được.

## Ví dụ

**Ví dụ 1: Trình kích hoạt Nén chặt**

```python
if context_tokens / context_limit > 0.8:
    context = compact_context(context)
```

**Ví dụ 2: Che giấu Quan sát**

```python
if len(observation) > max_length:
    ref_id = store_observation(observation)
    return f"[Obs:{ref_id} elided. Key: {extract_key(observation)}]"
```

**Ví dụ 3: Sắp xếp Thân thiện với Bộ nhớ đệm**

```python
# Stable content first
context = [system_prompt, tool_definitions]  # Cacheable
context += [reused_templates]  # Reusable
context += [unique_content]  # Unique
```

## Hướng dẫn

1. Đo lường trước khi tối ưu hóa—biết trạng thái hiện tại của bạn
2. Áp dụng nén chặt trước khi che giấu khi có thể
3. Thiết kế cho sự ổn định của bộ nhớ đệm với các lời nhắc nhất quán
4. Phân vùng trước khi bối cảnh trở nên có vấn đề
5. Giám sát hiệu quả tối ưu hóa theo thời gian
6. Cân bằng việc tiết kiệm token với việc bảo tồn chất lượng
7. Thử nghiệm tối ưu hóa ở quy mô sản xuất
8. Triển khai sự suy giảm duyên dáng cho các trường hợp biên

## Tích hợp

Kỹ năng này xây dựng dựa trên context-fundamentals và context-degradation. Nó kết nối với:

- multi-agent-patterns - Phân vùng như sự cô lập
- evaluation - Đo lường hiệu quả tối ưu hóa
- memory-systems - Giảm tải bối cảnh vào bộ nhớ

## Tham khảo

Tham khảo nội bộ:

- [Optimization Techniques Reference](./references/optimization_techniques.md) - Tham khảo kỹ thuật chi tiết

Các kỹ năng liên quan trong bộ sưu tập này:

- context-fundamentals - Cơ bản về bối cảnh
- context-degradation - Hiểu khi nào cần tối ưu hóa
- evaluation - Đo lường tối ưu hóa

Tài nguyên bên ngoài:

- Research on context window limitations
- KV-cache optimization techniques
- Production engineering guides
