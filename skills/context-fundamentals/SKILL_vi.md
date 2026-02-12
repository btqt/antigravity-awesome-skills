---
name: context-fundamentals
description: "Hiểu bối cảnh là gì, tại sao nó quan trọng và giải phẫu của bối cảnh trong các hệ thống tác nhân"
source: "https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering/tree/main/skills/context-fundamentals"
risk: safe
---

## Khi nào Sử dụng Kỹ năng Này

Hiểu bối cảnh là gì, tại sao nó quan trọng và giải phẫu của bối cảnh trong các hệ thống tác nhân

Sử dụng kỹ năng này khi làm việc với hiểu bối cảnh là gì, tại sao nó quan trọng và giải phẫu của bối cảnh trong các hệ thống tác nhân.

# Các Nguyên tắc Cơ bản về Kỹ thuật Bối cảnh

Bối cảnh là trạng thái hoàn chỉnh có sẵn cho một mô hình ngôn ngữ tại thời điểm suy luận. Nó bao gồm tất cả những gì mô hình có thể chú ý đến khi tạo ra phản hồi: hướng dẫn hệ thống, định nghĩa công cụ, tài liệu được truy xuất, lịch sử tin nhắn và đầu ra công cụ. Hiểu các nguyên tắc cơ bản về bối cảnh là điều kiện tiên quyết để kỹ thuật bối cảnh hiệu quả.

## Khi nào Kích hoạt

Kích hoạt kỹ năng này khi:

- Thiết kế hệ thống tác nhân mới hoặc sửa đổi kiến trúc hiện có
- Gỡ lỗi hành vi tác nhân bất ngờ có thể liên quan đến bối cảnh
- Tối ưu hóa việc sử dụng bối cảnh để giảm chi phí token hoặc cải thiện hiệu suất
- Giới thiệu thành viên nhóm mới với các khái niệm kỹ thuật bối cảnh
- Xem xét các quyết định thiết kế liên quan đến bối cảnh

## Các Khái niệm Cốt lõi

Bối cảnh bao gồm một số thành phần riêng biệt, mỗi thành phần có các đặc điểm và ràng buộc khác nhau. Cơ chế chú ý tạo ra một ngân sách hữu hạn hạn chế việc sử dụng bối cảnh hiệu quả. Tiết lộ lũy tiến (progressive disclosure) quản lý ràng buộc này bằng cách chỉ tải thông tin khi cần thiết. Kỷ luật kỹ thuật là quản lý tập hợp token tín hiệu cao nhỏ nhất đạt được kết quả mong muốn.

## Chủ đề Chi tiết

### Giải phẫu của Bối cảnh

**Lời nhắc Hệ thống (System Prompts)**
Lời nhắc hệ thống thiết lập danh tính cốt lõi, ràng buộc và hướng dẫn hành vi của tác nhân. Chúng được tải một lần khi bắt đầu phiên và thường tồn tại trong suốt cuộc trò chuyện. Lời nhắc hệ thống phải cực kỳ rõ ràng và sử dụng ngôn ngữ đơn giản, trực tiếp ở độ cao phù hợp cho tác nhân.

Độ cao phù hợp cân bằng hai chế độ thất bại. Ở một cực đoan, các kỹ sư mã hóa cứng logic phức tạp dễ vỡ tạo ra gánh nặng bảo trì và sự mong manh. Ở cực đoan khác, các kỹ sư cung cấp hướng dẫn cấp cao mơ hồ không đưa ra tín hiệu cụ thể cho kết quả mong muốn hoặc giả định sai lầm về bối cảnh chung. Độ cao tối ưu đạt được sự cân bằng: đủ cụ thể để hướng dẫn hành vi hiệu quả, nhưng đủ linh hoạt để cung cấp các heuristic mạnh mẽ.

Tổ chức các lời nhắc thành các phần riêng biệt sử dụng thẻ XML hoặc tiêu đề Markdown để phân định thông tin nền, hướng dẫn, hướng dẫn công cụ và mô tả đầu ra. Định dạng chính xác ít quan trọng hơn khi các mô hình trở nên có năng lực hơn, nhưng sự rõ ràng về cấu trúc vẫn có giá trị.

**Định nghĩa Công cụ**
Định nghĩa công cụ chỉ định các hành động mà một tác nhân có thể thực hiện. Mỗi công cụ bao gồm tên, mô tả, tham số và định dạng trả về. Định nghĩa công cụ nằm gần đầu bối cảnh sau khi tuần tự hóa, thường là trước hoặc sau lời nhắc hệ thống.

Mô tả công cụ tập thể định hướng hành vi của tác nhân. Mô tả kém buộc các tác nhân phải đoán; mô tả được tối ưu hóa bao gồm bối cảnh sử dụng, ví dụ và mặc định. Nguyên tắc hợp nhất nói rằng nếu một kỹ sư con người không thể nói dứt khoát công cụ nào nên được sử dụng trong một tình huống nhất định, thì một tác nhân không thể được mong đợi làm tốt hơn.

**Tài liệu được Truy xuất**
Tài liệu được truy xuất cung cấp kiến thức cụ thể về miền, tài liệu tham khảo hoặc thông tin liên quan đến nhiệm vụ. Các tác nhân sử dụng thế hệ tăng cường truy xuất (RAG) để kéo các tài liệu liên quan vào bối cảnh khi chạy thay vì tải trước tất cả thông tin có thể.

Cách tiếp cận đúng lúc (just-in-time) duy trì các định danh nhẹ (đường dẫn tệp, truy vấn được lưu trữ, liên kết web) và sử dụng các tham chiếu này để tải dữ liệu vào bối cảnh một cách linh hoạt. Điều này phản ánh nhận thức của con người: chúng ta thường không ghi nhớ toàn bộ kho thông tin mà sử dụng các hệ thống tổ chức và lập chỉ mục bên ngoài để truy xuất thông tin liên quan theo yêu cầu.

**Lịch sử Tin nhắn**
Lịch sử tin nhắn chứa cuộc trò chuyện giữa người dùng và tác nhân, bao gồm các truy vấn trước đó, phản hồi và lý luận. Đối với các nhiệm vụ chạy dài, lịch sử tin nhắn có thể phát triển để chiếm ưu thế trong việc sử dụng bối cảnh.

Lịch sử tin nhắn đóng vai trò là bộ nhớ nháp nơi các tác nhân theo dõi tiến độ, duy trì trạng thái nhiệm vụ và bảo tồn lý luận qua các lượt. Quản lý hiệu quả lịch sử tin nhắn là rất quan trọng để hoàn thành nhiệm vụ tầm nhìn dài.

**Đầu ra Công cụ**
Đầu ra công cụ là kết quả của các hành động tác nhân: nội dung tệp, kết quả tìm kiếm, đầu ra thực thi lệnh, phản hồi API và dữ liệu tương tự. Đầu ra công cụ bao gồm phần lớn các token trong quỹ đạo tác nhân điển hình, với nghiên cứu cho thấy các quan sát (đầu ra công cụ) có thể đạt tới 83,9% tổng mức sử dụng bối cảnh.

Đầu ra công cụ tiêu thụ bối cảnh cho dù chúng có liên quan đến các quyết định hiện tại hay không. Điều này tạo ra áp lực cho các chiến lược như che giấu quan sát, nén chặt và giữ lại kết quả công cụ có chọn lọc.

### Cửa sổ Bối cảnh và Cơ chế Chú ý

**Ràng buộc Ngân sách Chú ý**
Các mô hình ngôn ngữ xử lý các token thông qua các cơ chế chú ý tạo ra các mối quan hệ đôi một giữa tất cả các token trong bối cảnh. Đối với n token, điều này tạo ra n² mối quan hệ phải được tính toán và lưu trữ. Khi độ dài bối cảnh tăng lên, khả năng của mô hình để nắm bắt các mối quan hệ này bị kéo căng.

Các mô hình phát triển các mô hình chú ý từ phân phối dữ liệu đào tạo nơi các chuỗi ngắn hơn chiếm ưu thế. Điều này có nghĩa là các mô hình có ít kinh nghiệm hơn và ít tham số chuyên biệt hơn cho các phụ thuộc rộng bối cảnh. Kết quả là một "ngân sách chú ý" cạn kiệt khi bối cảnh phát triển.

**Mã hóa Vị trí và Mở rộng Bối cảnh**
Nội suy mã hóa vị trí cho phép các mô hình xử lý các chuỗi dài hơn bằng cách thích ứng chúng với các bối cảnh nhỏ hơn được đào tạo ban đầu. Tuy nhiên, sự thích ứng này giới thiệu sự suy giảm trong hiểu biết vị trí token. Các mô hình vẫn có khả năng cao ở các bối cảnh dài hơn nhưng cho thấy độ chính xác giảm đối với việc truy xuất thông tin và lý luận tầm xa so với hiệu suất trên các bối cảnh ngắn hơn.

**Nguyên tắc Tiết lộ Lũy tiến**
Tiết lộ lũy tiến quản lý bối cảnh hiệu quả bằng cách chỉ tải thông tin khi cần thiết. Khi khởi động, các tác nhân chỉ tải tên kỹ năng và mô tả—đủ để biết khi nào một kỹ năng có thể liên quan. Nội dung đầy đủ chỉ tải khi một kỹ năng được kích hoạt cho các nhiệm vụ cụ thể.

Cách tiếp cận này giữ cho các tác nhân nhanh trong khi cung cấp cho chúng quyền truy cập vào nhiều bối cảnh hơn theo yêu cầu. Nguyên tắc áp dụng ở nhiều cấp độ: lựa chọn kỹ năng, tải tài liệu và thậm chí truy xuất kết quả công cụ.

### Chất lượng Bối cảnh so với Số lượng Bối cảnh

Giả định rằng cửa sổ bối cảnh lớn hơn giải quyết các vấn đề bộ nhớ đã bị bác bỏ về mặt thực nghiệm. Kỹ thuật bối cảnh có nghĩa là tìm tập hợp nhỏ nhất có thể các token tín hiệu cao tối đa hóa khả năng của kết quả mong muốn.

Một số yếu tố tạo ra áp lực cho hiệu quả bối cảnh. Chi phí xử lý tăng không cân xứng với độ dài bối cảnh—không chỉ gấp đôi chi phí cho gấp đôi token, mà còn nhiều hơn theo cấp số nhân về thời gian và tài nguyên tính toán. Hiệu suất mô hình suy giảm vượt quá độ dài bối cảnh nhất định ngay cả khi cửa sổ hỗ trợ kỹ thuật nhiều token hơn. Đầu vào dài vẫn đắt đỏ ngay cả với bộ nhớ đệm tiền tố.

Nguyên tắc hướng dẫn là thông tin hơn sự đầy đủ. Bao gồm những gì quan trọng cho quyết định hiện tại, loại trừ những gì không, và thiết kế các hệ thống có thể truy cập thông tin bổ sung theo yêu cầu.

### Bối cảnh như Tài nguyên Hữu hạn

Bối cảnh phải được coi là một tài nguyên hữu hạn với lợi nhuận biên giảm dần. Giống như con người với bộ nhớ làm việc hạn chế, các mô hình ngôn ngữ có một ngân sách chú ý được rút ra khi phân tích khối lượng lớn bối cảnh.

Mỗi token mới được giới thiệu làm cạn kiệt ngân sách này một lượng nào đó. Điều này tạo ra nhu cầu quản lý cẩn thận các token có sẵn. Vấn đề kỹ thuật là tối ưu hóa tiện ích so với các ràng buộc vốn có.

Kỹ thuật bối cảnh là lặp lại và giai đoạn quản lý xảy ra mỗi khi bạn quyết định chuyển gì cho mô hình. Nó không phải là một bài tập viết lời nhắc một lần mà là một kỷ luật liên tục của quản lý bối cảnh.

## Hướng dẫn Thực tế

### Truy cập Dựa trên Hệ thống Tệp

Các tác nhân có quyền truy cập hệ thống tệp có thể sử dụng tiết lộ lũy tiến một cách tự nhiên. Lưu trữ tài liệu tham khảo, tài liệu và dữ liệu bên ngoài. Chỉ tải tệp khi cần thiết sử dụng các hoạt động hệ thống tệp tiêu chuẩn. Mô hình này tránh nhồi nhét bối cảnh với thông tin có thể không liên quan.

Hệ thống tệp tự nó cung cấp cấu trúc mà các tác nhân có thể điều hướng. Kích thước tệp gợi ý độ phức tạp; quy ước đặt tên gợi ý mục đích; dấu thời gian đóng vai trò là đại diện cho mức độ liên quan. Siêu dữ liệu của các tham chiếu tệp cung cấp cơ chế để tinh chỉnh hành vi hiệu quả.

### Chiến lược Lai

Các tác nhân hiệu quả nhất sử dụng các chiến lược lai. Tải trước một số bối cảnh cho tốc độ (như các tệp CLAUDE.md hoặc quy tắc dự án), nhưng cho phép khám phá tự chủ cho bối cảnh bổ sung khi cần thiết. Ranh giới quyết định phụ thuộc vào đặc điểm nhiệm vụ và động lực bối cảnh.

Đối với các bối cảnh có nội dung ít động hơn, tải trước nhiều hơn ngay từ đầu là hợp lý. Đối với thông tin thay đổi nhanh chóng hoặc rất cụ thể, tải đúng lúc tránh bối cảnh cũ.

### Lập ngân sách Bối cảnh

Thiết kế với ngân sách bối cảnh rõ ràng trong tâm trí. Biết giới hạn bối cảnh hiệu quả cho mô hình và nhiệm vụ của bạn. Giám sát việc sử dụng bối cảnh trong quá trình phát triển. Triển khai các trình kích hoạt nén chặt ở các ngưỡng thích hợp. Thiết kế hệ thống giả định bối cảnh sẽ suy giảm thay vì hy vọng nó sẽ không.

Lập ngân sách bối cảnh hiệu quả đòi hỏi phải hiểu không chỉ số lượng token thô mà còn cả các mô hình phân phối sự chú ý. Phần giữa của bối cảnh nhận được ít sự chú ý hơn phần đầu và phần cuối. Đặt thông tin quan trọng tại các vị trí được ưu tiên chú ý.

## Ví dụ

**Ví dụ 1: Tổ chức Lời nhắc Hệ thống**

```markdown
<BACKGROUND_INFORMATION>
You are a Python expert helping a development team.
Current project: Data processing pipeline in Python 3.9+
</BACKGROUND_INFORMATION>

<INSTRUCTIONS>
- Write clean, idiomatic Python code
- Include type hints for function signatures
- Add docstrings for public functions
- Follow PEP 8 style guidelines
</INSTRUCTIONS>

<TOOL_GUIDANCE>
Use bash for shell operations, python for code tasks.
File operations should use pathlib for cross-platform compatibility.
</TOOL_GUIDANCE>

<OUTPUT_DESCRIPTION>
Provide code blocks with syntax highlighting.
Explain non-obvious decisions in comments.
</OUTPUT_DESCRIPTION>
```

**Ví dụ 2: Tải Tài liệu Lũy tiến**

```markdown
# Instead of loading all documentation at once:

# Step 1: Load summary

docs/api_summary.md # Lightweight overview

# Step 2: Load specific section as needed

docs/api/endpoints.md # Only when API calls needed
docs/api/authentication.md # Only when auth context needed
```

## Hướng dẫn

1. Coi bối cảnh là tài nguyên hữu hạn với lợi nhuận giảm dần
2. Đặt thông tin quan trọng tại các vị trí được ưu tiên chú ý (đầu và cuối)
3. Sử dụng tiết lộ lũy tiến để hoãn tải cho đến khi cần thiết
4. Tổ chức lời nhắc hệ thống với ranh giới phần rõ ràng
5. Giám sát việc sử dụng bối cảnh trong quá trình phát triển
6. Triển khai các trình kích hoạt nén chặt ở mức sử dụng 70-80%
7. Thiết kế cho sự suy giảm bối cảnh thay vì hy vọng tránh được nó
8. Ưu tiên bối cảnh tín hiệu cao nhỏ hơn so với bối cảnh tín hiệu thấp lớn hơn

## Tích hợp

Kỹ năng này cung cấp bối cảnh nền tảng mà tất cả các kỹ năng khác xây dựng dựa trên. Nó nên được nghiên cứu trước tiên trước khi khám phá:

- context-degradation - Hiểu cách bối cảnh thất bại
- context-optimization - Các kỹ thuật mở rộng khả năng bối cảnh
- multi-agent-patterns - Cách cô lập bối cảnh cho phép hệ thống đa tác nhân
- tool-design - Cách định nghĩa công cụ tương tác với bối cảnh

## Tham khảo

Tham khảo nội bộ:

- [Context Components Reference](./references/context-components.md) - Tham khảo kỹ thuật chi tiết

Các kỹ năng liên quan trong bộ sưu tập này:

- context-degradation - Hiểu các mô hình thất bại bối cảnh
- context-optimization - Các kỹ thuật sử dụng bối cảnh hiệu quả

Tài nguyên bên ngoài:

- Research on transformer attention mechanisms
- Production engineering guides from leading AI labs
- Framework documentation on context window management
