---
name: context-compression
description: "Thiết kế và đánh giá các chiến lược nén cho các phiên chạy dài"
source: "https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering/tree/main/skills/context-compression"
risk: safe
---

# Chiến lược Nén Bối cảnh (Context Compression Strategies)

Khi các phiên tác nhân tạo ra hàng triệu token lịch sử cuộc trò chuyện, việc nén trở thành bắt buộc. Cách tiếp cận ngây thơ là nén tích cực để giảm thiểu token trên mỗi yêu cầu. Mục tiêu tối ưu hóa chính xác là token trên mỗi nhiệm vụ: tổng số token tiêu thụ để hoàn thành một nhiệm vụ, bao gồm cả chi phí tìm nạp lại khi nén làm mất thông tin quan trọng.

## Khi nào Kích hoạt

Kích hoạt kỹ năng này khi:

- Các phiên tác nhân vượt quá giới hạn cửa sổ bối cảnh
- Các cơ sở mã vượt quá cửa sổ bối cảnh (hệ thống 5M+ token)
- Thiết kế chiến lược tóm tắt cuộc trò chuyện
- Gỡ lỗi các trường hợp tác nhân "quên" những tệp nào họ đã sửa đổi
- Xây dựng khuôn khổ đánh giá cho chất lượng nén

## Các Khái niệm Cốt lõi

Nén bối cảnh đánh đổi việc tiết kiệm token với việc mất thông tin. Có ba cách tiếp cận sẵn sàng cho sản xuất:

1. **Tóm tắt Lặp lại Neo (Anchored Iterative Summarization)**: Duy trì các bản tóm tắt có cấu trúc, bền vững với các phần rõ ràng cho ý định phiên, sửa đổi tệp, quyết định và các bước tiếp theo. Khi nén kích hoạt, chỉ tóm tắt khoảng thời gian mới bị cắt ngắn và hợp nhất với bản tóm tắt hiện có. Cấu trúc buộc phải bảo tồn bằng cách dành riêng các phần cho các loại thông tin cụ thể.

2. **Nén Mờ (Opaque Compression)**: Tạo ra các biểu diễn nén được tối ưu hóa cho độ trung thực tái tạo. Đạt được tỷ lệ nén cao nhất (99%+) nhưng hy sinh khả năng diễn giải. Không thể xác minh những gì đã được bảo tồn.

3. **Tóm tắt Đầy đủ Tái tạo (Regenerative Full Summary)**: Tạo tóm tắt có cấu trúc chi tiết trên mỗi lần nén. Tạo ra đầu ra dễ đọc nhưng có thể mất chi tiết qua các chu kỳ nén lặp lại do tái tạo đầy đủ thay vì hợp nhất gia tăng.

Thông tin chi tiết quan trọng: cấu trúc buộc phải bảo tồn. Các phần dành riêng hoạt động như danh sách kiểm tra mà trình tóm tắt phải điền vào, ngăn chặn sự trôi dạt thông tin âm thầm.

## Chủ đề Chi tiết

### Tại sao Token-Trên-Nhiệm vụ Quan trọng

Các chỉ số nén truyền thống nhắm mục tiêu token-trên-yêu cầu. Đây là sự tối ưu hóa sai lầm. Khi nén làm mất các chi tiết quan trọng như đường dẫn tệp hoặc thông báo lỗi, tác nhân phải tìm nạp lại thông tin, khám phá lại các cách tiếp cận và lãng phí token để khôi phục bối cảnh.

Chỉ số đúng là token-trên-nhiệm vụ: tổng số token tiêu thụ từ khi bắt đầu nhiệm vụ đến khi hoàn thành. Một chiến lược nén tiết kiệm thêm 0.5% token nhưng gây ra chi phí tìm nạp lại nhiều hơn 20% sẽ tốn kém hơn về tổng thể.

### Vấn đề Dấu vết Tạo tác (Artifact Trail Problem)

Tính toàn vẹn của dấu vết tạo tác là khía cạnh yếu nhất trên tất cả các phương pháp nén, đạt điểm 2.2-2.5 trên 5.0 trong các đánh giá. Ngay cả tóm tắt có cấu trúc với các phần tệp rõ ràng cũng gặp khó khăn trong việc duy trì theo dõi tệp hoàn chỉnh qua các phiên dài.

Các tác nhân mã hóa cần biết:

- Những tệp nào đã được tạo
- Những tệp nào đã được sửa đổi và những gì đã thay đổi
- Những tệp nào đã được đọc nhưng không thay đổi
- Tên hàm, tên biến, thông báo lỗi

Vấn đề này có thể yêu cầu xử lý chuyên biệt ngoài việc tóm tắt chung: một chỉ mục tạo tác riêng biệt hoặc theo dõi trạng thái tệp rõ ràng trong giàn giáo tác nhân.

### Các Phần Tóm tắt Có Cấu trúc

Các bản tóm tắt có cấu trúc hiệu quả bao gồm các phần rõ ràng:

```markdown
## Session Intent

[What the user is trying to accomplish]

## Files Modified

- auth.controller.ts: Fixed JWT token generation
- config/redis.ts: Updated connection pooling
- tests/auth.test.ts: Added mock setup for new config

## Decisions Made

- Using Redis connection pool instead of per-request connections
- Retry logic with exponential backoff for transient failures

## Current State

- 14 tests passing, 2 failing
- Remaining: mock setup for session service tests

## Next Steps

1. Fix remaining test failures
2. Run full test suite
3. Update documentation
```

Cấu trúc này ngăn chặn việc mất đường dẫn tệp hoặc quyết định một cách âm thầm vì mỗi phần phải được giải quyết một cách rõ ràng.

### Chiến lược Kích hoạt Nén

Khi nào kích hoạt nén cũng quan trọng như cách nén:

| Chiến lược              | Điểm Kích hoạt                    | Đánh đổi                                       |
| ----------------------- | --------------------------------- | ---------------------------------------------- |
| Ngưỡng cố định          | 70-80% sử dụng bối cảnh           | Đơn giản nhưng có thể nén quá sớm              |
| Cửa sổ trượt            | Giữ N lượt cuối + tóm tắt         | Kích thước bối cảnh có thể dự đoán             |
| Dựa trên tầm quan trọng | Nén các phần ít liên quan trước   | Phức tạp nhưng bảo tồn tín hiệu                |
| Ranh giới nhiệm vụ      | Nén khi hoàn thành nhiệm vụ logic | Tóm tắt sạch nhưng thời gian không thể dự đoán |

Cách tiếp cận cửa sổ trượt với tóm tắt có cấu trúc cung cấp sự cân bằng tốt nhất giữa khả năng dự đoán và chất lượng cho hầu hết các trường hợp sử dụng tác nhân mã hóa.

### Đánh giá Dựa trên Thăm dò (Probe-Based Evaluation)

Các chỉ số truyền thống như ROUGE hoặc độ tương tự nhúng không nắm bắt được chất lượng nén chức năng. Một bản tóm tắt có thể đạt điểm cao về sự chồng chéo từ vựng trong khi thiếu một đường dẫn tệp mà tác nhân cần.

Đánh giá dựa trên thăm dò đo lường trực tiếp chất lượng chức năng bằng cách đặt câu hỏi sau khi nén:

| Loại Thăm dò            | Nó Kiểm tra Gì        | Ví dụ Câu hỏi                                |
| ----------------------- | --------------------- | -------------------------------------------- |
| Gợi nhớ (Recall)        | Giữ lại sự thật       | "Thông báo lỗi ban đầu là gì?"               |
| Tạo tác (Artifact)      | Theo dõi tệp          | "Chúng ta đã sửa đổi những tệp nào?"         |
| Tiếp tục (Continuation) | Lập kế hoạch nhiệm vụ | "Chúng ta nên làm gì tiếp theo?"             |
| Quyết định (Decision)   | Chuỗi lý luận         | "Chúng ta đã quyết định gì về vấn đề Redis?" |

Nếu nén bảo tồn đúng thông tin, tác nhân sẽ trả lời chính xác. Nếu không, nó sẽ đoán hoặc ảo giác.

### Các Khía cạnh Đánh giá

Sáu khía cạnh nắm bắt chất lượng nén cho các tác nhân mã hóa:

1. **Độ chính xác**: Các chi tiết kỹ thuật có chính xác không? Đường dẫn tệp, tên hàm, mã lỗi.
2. **Nhận thức Bối cảnh**: Phản hồi có phản ánh trạng thái cuộc trò chuyện hiện tại không?
3. **Dấu vết Tạo tác**: Tác nhân có biết những tệp nào đã được đọc hoặc sửa đổi không?
4. **Tính đầy đủ**: Phản hồi có giải quyết tất cả các phần của câu hỏi không?
5. **Tính liên tục**: Công việc có thể tiếp tục mà không cần tìm nạp lại thông tin không?
6. **Tuân thủ Hướng dẫn**: Phản hồi có tôn trọng các ràng buộc đã nêu không?

Độ chính xác cho thấy sự thay đổi lớn nhất giữa các phương pháp nén (khoảng cách 0.6 điểm). Dấu vết tạo tác yếu trên toàn cầu (phạm vi 2.2-2.5).

## Hướng dẫn Thực tế

### Quy trình Nén Ba Giai đoạn

Đối với các cơ sở mã lớn hoặc hệ thống tác nhân vượt quá cửa sổ bối cảnh, áp dụng nén qua ba giai đoạn:

1. **Giai đoạn Nghiên cứu**: Tạo tài liệu nghiên cứu từ sơ đồ kiến trúc, tài liệu và các giao diện chính. Nén quá trình khám phá thành một phân tích có cấu trúc về các thành phần và phụ thuộc. Đầu ra: tài liệu nghiên cứu đơn lẻ.

2. **Giai đoạn Lập kế hoạch**: Chuyển đổi nghiên cứu thành đặc tả triển khai với chữ ký hàm, định nghĩa kiểu và luồng dữ liệu. Một cơ sở mã 5M token nén thành khoảng 2.000 từ đặc tả.

3. **Giai đoạn Triển khai**: Thực thi dựa trên đặc tả. Bối cảnh vẫn tập trung vào đặc tả thay vì khám phá cơ sở mã thô.

### Sử dụng Tạo tác Ví dụ làm Hạt giống

Khi được cung cấp một ví dụ di chuyển thủ công hoặc PR tham khảo, hãy sử dụng nó như một mẫu để hiểu mẫu đích. Ví dụ tiết lộ các ràng buộc mà phân tích tĩnh không thể hiển thị: những bất biến nào phải giữ, những dịch vụ nào bị phá vỡ khi thay đổi, và một cuộc di chuyển sạch sẽ trông như thế nào.

Điều này đặc biệt quan trọng khi tác nhân không thể phân biệt độ phức tạp thiết yếu (yêu cầu kinh doanh) với độ phức tạp ngẫu nhiên (các giải pháp thay thế cũ). Tạo tác ví dụ mã hóa sự phân biệt đó.

### Triển khai Tóm tắt Lặp lại Neo

1. Xác định các phần tóm tắt rõ ràng phù hợp với nhu cầu của tác nhân
2. Khi kích hoạt nén lần đầu, tóm tắt lịch sử bị cắt ngắn thành các phần
3. Trong các lần nén tiếp theo, chỉ tóm tắt nội dung bị cắt ngắn mới
4. Hợp nhất tóm tắt mới vào các phần hiện có thay vì tạo lại
5. Theo dõi thông tin nào đến từ chu kỳ nén nào để gỡ lỗi

### Khi nào Sử dụng Mỗi Cách tiếp cận

**Sử dụng tóm tắt lặp lại neo khi:**

- Các phiên chạy dài (100+ tin nhắn)
- Theo dõi tệp quan trọng (mã hóa, gỡ lỗi)
- Bạn cần xác minh những gì đã được bảo tồn

**Sử dụng nén mờ khi:**

- Yêu cầu tiết kiệm token tối đa
- Các phiên tương đối ngắn
- Chi phí tìm nạp lại thấp

**Sử dụng tóm tắt tái tạo khi:**

- Khả năng diễn giải tóm tắt là quan trọng
- Các phiên có ranh giới giai đoạn rõ ràng
- Đánh giá bối cảnh đầy đủ được chấp nhận trên mỗi lần nén

### Cân nhắc Tỷ lệ Nén

| Phương pháp | Tỷ lệ Nén | Điểm Chất lượng | Đánh đổi                                 |
| ----------- | --------- | --------------- | ---------------------------------------- |
| Lặp lại Neo | 98.6%     | 3.70            | Chất lượng tốt nhất, nén ít hơn một chút |
| Tái tạo     | 98.7%     | 3.44            | Chất lượng tốt, nén vừa phải             |
| Mờ          | 99.3%     | 3.35            | Nén tốt nhất, mất chất lượng             |

0.7% token bổ sung được giữ lại bởi tóm tắt có cấu trúc mua được 0.35 điểm chất lượng. Đối với bất kỳ nhiệm vụ nào mà chi phí tìm nạp lại quan trọng, sự đánh đổi này ủng hộ các cách tiếp cận có cấu trúc.

## Ví dụ

**Ví dụ 1: Nén Phiên Gỡ lỗi**

Bối cảnh gốc (89,000 token, 178 tin nhắn):

- Lỗi 401 trên điểm cuối /api/auth/login
- Truy tìm qua auth controller, middleware, session store
- Tìm thấy kết nối Redis cũ
- Đã sửa kết nối pooling, thêm logic thử lại
- 14 bài kiểm tra đạt, 2 thất bại

Tóm tắt có cấu trúc sau khi nén:

```markdown
## Session Intent

Debug 401 Unauthorized error on /api/auth/login despite valid credentials.

## Root Cause

Stale Redis connection in session store. JWT generated correctly but session could not be persisted.

## Files Modified

- auth.controller.ts: No changes (read only)
- middleware/cors.ts: No changes (examined)
- config/redis.ts: Fixed connection pooling configuration
- services/session.service.ts: Added retry logic for transient failures
- tests/auth.test.ts: Updated mock setup

## Test Status

14 passing, 2 failing (mock setup issues)

## Next Steps

1. Fix remaining test failures (mock session service)
2. Run full test suite
3. Deploy to staging
```

**Ví dụ 2: Chất lượng Phản hồi Thăm dò**

Sau khi nén, hỏi "Lỗi ban đầu là gì?":

Phản hồi tốt (tóm tắt có cấu trúc):

> "Lỗi ban đầu là phản hồi 401 Unauthorized từ điểm cuối /api/auth/login. Người dùng nhận được lỗi này với thông tin đăng nhập hợp lệ. Nguyên nhân gốc rễ là kết nối Redis cũ trong kho lưu trữ phiên."

Phản hồi kém (nén tích cực):

> "Chúng tôi đang gỡ lỗi một vấn đề xác thực. Đăng nhập bị lỗi. Chúng tôi đã sửa một số vấn đề cấu hình."

Phản hồi có cấu trúc bảo tồn điểm cuối, mã lỗi và nguyên nhân gốc rễ. Phản hồi tích cực làm mất tất cả các chi tiết kỹ thuật.

## Hướng dẫn

1. Tối ưu hóa cho token-trên-nhiệm vụ, không phải token-trên-yêu cầu
2. Sử dụng tóm tắt có cấu trúc với các phần rõ ràng cho việc theo dõi tệp
3. Kích hoạt nén ở mức sử dụng bối cảnh 70-80%
4. Triển khai hợp nhất gia tăng thay vì tái tạo đầy đủ
5. Kiểm tra chất lượng nén với đánh giá dựa trên thăm dò
6. Theo dõi dấu vết tạo tác riêng biệt nếu việc theo dõi tệp là quan trọng
7. Chấp nhận tỷ lệ nén thấp hơn một chút để giữ lại chất lượng tốt hơn
8. Giám sát tần suất tìm nạp lại như một tín hiệu chất lượng nén

## Tích hợp

Kỹ năng này kết nối với một số kỹ năng khác trong bộ sưu tập:

- context-degradation - Nén là một chiến lược giảm thiểu cho sự suy giảm
- context-optimization - Nén là một kỹ thuật tối ưu hóa trong số nhiều kỹ thuật
- evaluation - Đánh giá dựa trên thăm dò áp dụng cho thử nghiệm nén
- memory-systems - Nén liên quan đến các mẫu bộ nhớ scratchpad và tóm tắt

## Tham khảo

Tham khảo nội bộ:

- [Evaluation Framework Reference](./references/evaluation-framework.md) - Các loại thăm dò chi tiết và thang điểm

Các kỹ năng liên quan trong bộ sưu tập này:

- context-degradation - Hiểu những gì nén ngăn chặn
- context-optimization - Các chiến lược tối ưu hóa rộng lớn hơn
- evaluation - Xây dựng khuôn khổ đánh giá

Tài nguyên bên ngoài:

- Factory Research: Evaluating Context Compression for AI Agents (December 2025)
- Research on LLM-as-judge evaluation methodology (Zheng et al., 2023)
- Netflix Engineering: "The Infinite Software Crisis" - Three-phase workflow and context compression at scale (AI Summit 2025)
