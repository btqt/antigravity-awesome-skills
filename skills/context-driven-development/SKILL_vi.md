---
name: context-driven-development
description: Sử dụng kỹ năng này khi làm việc với phương pháp phát triển dựa trên bối cảnh của Conductor, quản lý các tạo tác bối cảnh dự án, hoặc hiểu mối quan hệ giữa các tệp product.md, tech-stack.md, và workflow.md.
metadata:
  version: 1.0.0
---

# Phát triển Dựa trên Bối cảnh (Context-Driven Development)

Hướng dẫn triển khai và duy trì bối cảnh như một tạo tác được quản lý cùng với mã, cho phép các tương tác AI nhất quán và sự liên kết nhóm thông qua tài liệu dự án có cấu trúc.

## Không sử dụng kỹ năng này khi

- Nhiệm vụ không liên quan đến phát triển dựa trên bối cảnh
- Bạn cần một miền hoặc công cụ khác ngoài phạm vi này

## Hướng dẫn

- Làm rõ mục tiêu, ràng buộc và đầu vào cần thiết.
- Áp dụng các thực tiễn tốt nhất có liên quan và xác thực kết quả.
- Cung cấp các bước có thể hành động và xác minh.
- Nếu cần các ví dụ chi tiết, hãy mở `resources/implementation-playbook.md`.

## Sử dụng kỹ năng này khi

- Thiết lập dự án mới với Conductor
- Hiểu mối quan hệ giữa các tạo tác bối cảnh
- Duy trì tính nhất quán qua các phiên phát triển được hỗ trợ bởi AI
- Giới thiệu thành viên nhóm vào dự án Conductor hiện có
- Quyết định khi nào cập nhật tài liệu bối cảnh
- Quản lý bối cảnh dự án greenfield so với brownfield

## Triết lý Cốt lõi

Phát triển Dựa trên Bối cảnh coi bối cảnh dự án là một tạo tác hạng nhất được quản lý cùng với mã. Thay vì dựa vào các lời nhắc đặc biệt hoặc tài liệu nằm rải rác, hãy thiết lập một nền tảng có cấu trúc, bền vững thông báo cho tất cả các tương tác AI.

Các nguyên tắc chính:

1. **Bối cảnh đi trước mã**: Xác định những gì bạn đang xây dựng và làm thế nào trước khi triển khai
2. **Tài liệu sống**: Các tạo tác bối cảnh phát triển cùng với dự án
3. **Nguồn sự thật duy nhất**: Một vị trí chính tắc cho từng loại thông tin
4. **Căn chỉnh AI**: Bối cảnh nhất quán tạo ra hành vi AI nhất quán

## Quy trình làm việc

Tuân theo quy trình **Bối cảnh → Đặc tả & Kế hoạch → Triển khai**:

1. **Giai đoạn Bối cảnh**: Thiết lập hoặc xác minh các tạo tác bối cảnh dự án tồn tại và là hiện tại
2. **Giai đoạn Đặc tả**: Xác định yêu cầu và tiêu chí chấp nhận cho các đơn vị công việc
3. **Giai đoạn Lập kế hoạch**: Chia đặc tả thành các nhiệm vụ theo giai đoạn, có thể hành động
4. **Giai đoạn Triển khai**: Thực thi các nhiệm vụ tuân theo các mẫu quy trình đã thiết lập

## Mối quan hệ Tạo tác

### product.md - Xác định CÁI GÌ và TẠI SAO

Mục đích: Nắm bắt tầm nhìn sản phẩm, mục tiêu, người dùng mục tiêu và bối cảnh kinh doanh.

Nội dung:

- Tên sản phẩm và mô tả một dòng
- Tuyên bố vấn đề và cách tiếp cận giải pháp
- Chân dung người dùng mục tiêu
- Các tính năng và khả năng cốt lõi
- Các chỉ số thành công và KPI
- Lộ trình sản phẩm (cấp cao)

Cập nhật khi:

- Tầm nhìn hoặc mục tiêu sản phẩm thay đổi
- Các tính năng chính mới được lên kế hoạch
- Đối tượng mục tiêu thay đổi
- Ưu tiên kinh doanh phát triển

### product-guidelines.md - Xác định CÁCH giao tiếp

Mục đích: Thiết lập giọng nói thương hiệu, tiêu chuẩn thông điệp và các mẫu giao tiếp.

Nội dung:

- Hướng dẫn giọng nói và tông màu thương hiệu
- Thuật ngữ và bảng thuật ngữ
- Quy ước thông báo lỗi
- Tiêu chuẩn văn bản hướng tới người dùng
- Phong cách tài liệu

Cập nhật khi:

- Hướng dẫn thương hiệu thay đổi
- Thuật ngữ mới được giới thiệu
- Các mẫu giao tiếp cần tinh chỉnh

### tech-stack.md - Xác định VỚI CÁI GÌ

Mục đích: Tài liệu hóa các lựa chọn công nghệ, phụ thuộc và các quyết định kiến trúc.

Nội dung:

- Ngôn ngữ và khuôn khổ chính
- Các phụ thuộc chính với phiên bản
- Cơ sở hạ tầng và mục tiêu triển khai
- Công cụ phát triển và môi trường
- Khuôn khổ kiểm thử
- Công cụ chất lượng mã

Cập nhật khi:

- Thêm các phụ thuộc mới
- Nâng cấp các phiên bản chính
- Thay đổi cơ sở hạ tầng
- Áp dụng các công cụ hoặc mẫu mới

### workflow.md - Xác định CÁCH làm việc

Mục đích: Thiết lập các thực tiễn phát triển, cổng chất lượng và quy trình làm việc nhóm.

Nội dung:

- Phương pháp phát triển (TDD, v.v.)
- Quy trình Git và quy ước cam kết
- Yêu cầu đánh giá mã
- Yêu cầu kiểm thử và mục tiêu bao phủ
- Cổng đảm bảo chất lượng
- Quy trình triển khai

Cập nhật khi:

- Thực tiễn nhóm phát triển
- Tiêu chuẩn chất lượng thay đổi
- Các mẫu quy trình làm việc mới được áp dụng

### tracks.md - Theo dõi ĐIỀU GÌ ĐANG XẢY RA

Mục đích: Sổ đăng ký tất cả các đơn vị công việc với trạng thái và siêu dữ liệu.

Nội dung:

- Các theo dõi đang hoạt động với trạng thái hiện tại
- Các theo dõi đã hoàn thành với ngày hoàn thành
- Siêu dữ liệu theo dõi (loại, mức độ ưu tiên, người được giao)
- Liên kết đến các thư mục theo dõi riêng lẻ

Cập nhật khi:

- Các theo dõi mới được tạo
- Trạng thái theo dõi thay đổi
- Các theo dõi được hoàn thành hoặc lưu trữ

## Nguyên tắc Duy trì Bối cảnh

### Giữ Các Tạo tác Đồng bộ

Đảm bảo các thay đổi trong một tạo tác phản ánh trong các tài liệu liên quan:

- Tính năng mới trong product.md → Cập nhật tech-stack.md nếu cần phụ thuộc mới
- Theo dõi hoàn thành → Cập nhật product.md để phản ánh khả năng mới
- Thay đổi quy trình làm việc → Cập nhật tất cả các kế hoạch theo dõi bị ảnh hưởng

### Cập nhật tech-stack.md Khi Thêm Phụ thuộc

Trước khi thêm bất kỳ phụ thuộc mới nào:

1. Kiểm tra xem các phụ thuộc hiện có có giải quyết nhu cầu không
2. Tài liệu hóa lý do cho các phụ thuộc mới
3. Thêm các ràng buộc phiên bản
4. Ghi chú bất kỳ yêu cầu cấu hình nào

### Cập nhật product.md Khi Các Tính năng Hoàn thành

Sau khi hoàn thành một theo dõi tính năng:

1. Di chuyển tính năng từ "đã lên kế hoạch" sang "đã triển khai" trong product.md
2. Cập nhật bất kỳ chỉ số thành công nào bị ảnh hưởng
3. Tài liệu hóa bất kỳ thay đổi phạm vi nào so với kế hoạch ban đầu

### Xác minh Bối cảnh Trước khi Triển khai

Trước khi bắt đầu bất kỳ theo dõi nào:

1. Đọc tất cả các tạo tác bối cảnh
2. Đánh dấu bất kỳ thông tin lỗi thời nào
3. Đề xuất cập nhật trước khi tiếp tục
4. Xác nhận độ chính xác của bối cảnh với các bên liên quan

## Xử lý Greenfield so với Brownfield

### Dự án Greenfield (Mới)

Đối với các dự án mới:

1. Chạy `/conductor:setup` để tạo tất cả các tạo tác một cách tương tác
2. Trả lời các câu hỏi về tầm nhìn sản phẩm, sở thích công nghệ và quy trình làm việc
3. Tạo hướng dẫn phong cách ban đầu cho các ngôn ngữ đã chọn
4. Tạo sổ đăng ký theo dõi trống

Đặc điểm:

- Kiểm soát hoàn toàn cấu trúc bối cảnh
- Xác định tiêu chuẩn trước khi mã tồn tại
- Thiết lập các mẫu sớm

### Dự án Brownfield (Hiện có)

Đối với các cơ sở mã hiện có:

1. Chạy `/conductor:setup` với phát hiện cơ sở mã hiện có
2. Hệ thống phân tích mã, cấu hình và tài liệu hiện có
3. Điền trước các tạo tác dựa trên các mẫu được phát hiện
4. Xem xét và tinh chỉnh bối cảnh đã tạo

Đặc điểm:

- Trích xuất bối cảnh ngầm từ mã hiện có
- Hòa giải các mẫu hiện có với các mẫu mong muốn
- Tài liệu hóa nợ kỹ thuật và kế hoạch hiện đại hóa
- Bảo tồn các mẫu làm việc trong khi thiết lập các tiêu chuẩn

## Lợi ích

### Căn chỉnh Nhóm

- Thành viên nhóm mới tham gia nhanh hơn với bối cảnh rõ ràng
- Thuật ngữ và quy ước nhất quán trong toàn nhóm
- Hiểu biết chung về mục tiêu sản phẩm và quyết định kỹ thuật

### Tính Nhất quán của AI

- Các trợ lý AI tạo ra đầu ra được căn chỉnh qua các phiên
- Giảm nhu cầu giải thích lại bối cảnh trong mỗi tương tác
- Hành vi có thể dự đoán dựa trên các tiêu chuẩn đã được tài liệu hóa

### Bộ nhớ Tổ chức

- Các quyết định và lý do được bảo tồn
- Bối cảnh tồn tại qua các thay đổi nhóm
- Bối cảnh lịch sử thông báo cho các quyết định trong tương lai

### Đảm bảo Chất lượng

- Các tiêu chuẩn là rõ ràng và có thể kiểm chứng
- Các sai lệch so với bối cảnh có thể phát hiện được
- Các cổng chất lượng được tài liệu hóa và có thể thực thi

## Cấu trúc Thư mục

```
conductor/
├── index.md              # Trung tâm điều hướng liên kết tất cả các tạo tác
├── product.md            # Tầm nhìn và mục tiêu sản phẩm
├── product-guidelines.md # Tiêu chuẩn giao tiếp
├── tech-stack.md         # Sở thích công nghệ
├── workflow.md           # Thực tiễn phát triển
├── tracks.md             # Sổ đăng ký đơn vị công việc
├── setup_state.json      # Trạng thái thiết lập có thể tiếp tục
├── code_styleguides/     # Quy ước dành riêng cho ngôn ngữ
│   ├── python.md
│   ├── typescript.md
│   └── ...
└── tracks/
    └── <track-id>/
        ├── spec.md
        ├── plan.md
        ├── metadata.json
        └── index.md
```

## Vòng đời Bối cảnh

1. **Tạo**: Thiết lập ban đầu qua `/conductor:setup`
2. **Xác thực**: Xác minh trước mỗi theo dõi
3. **Phát triển**: Cập nhật khi dự án phát triển
4. **Đồng bộ hóa**: Giữ các tạo tác được căn chỉnh
5. **Lưu trữ**: Tài liệu hóa các quyết định lịch sử

## Danh sách Kiểm tra Xác thực Bối cảnh

Trước khi bắt đầu triển khai trên bất kỳ theo dõi nào, hãy xác thực bối cảnh:

### Bối cảnh Sản phẩm

- [ ] product.md phản ánh tầm nhìn sản phẩm hiện tại
- [ ] Người dùng mục tiêu được mô tả chính xác
- [ ] Danh sách tính năng được cập nhật
- [ ] Các chỉ số thành công được xác định

### Bối cảnh Kỹ thuật

- [ ] tech-stack.md liệt kê tất cả các phụ thuộc hiện tại
- [ ] Số phiên bản là chính xác
- [ ] Các mục tiêu cơ sở hạ tầng là chính xác
- [ ] Các công cụ phát triển được tài liệu hóa

### Bối cảnh Quy trình làm việc

- [ ] workflow.md mô tả các thực tiễn hiện tại
- [ ] Các cổng chất lượng được xác định
- [ ] Mục tiêu bao phủ được chỉ định
- [ ] Quy ước cam kết được tài liệu hóa

### Bối cảnh Theo dõi

- [ ] tracks.md hiển thị tất cả công việc đang hoạt động
- [ ] Không có theo dõi cũ hoặc bị bỏ rơi
- [ ] Các phụ thuộc giữa các theo dõi được ghi chú

## Các Chống Mẫu (Anti-Patterns) Phổ biến

Tránh các sai lầm quản lý bối cảnh này:

### Bối cảnh Cũ

Vấn đề: Tài liệu bối cảnh trở nên lỗi thời và gây hiểu lầm.
Giải pháp: Cập nhật bối cảnh như một phần của quy trình hoàn thành mỗi theo dõi.

### Bối cảnh Lan man

Vấn đề: Thông tin nằm rải rác trên nhiều vị trí.
Giải pháp: Sử dụng cấu trúc tạo tác đã xác định; chống lại việc tạo các loại tài liệu mới.

### Bối cảnh Ngầm

Vấn đề: Dựa vào kiến thức không được nắm bắt trong các tạo tác.
Giải pháp: Nếu bạn tham chiếu điều gì đó lặp đi lặp lại, hãy thêm nó vào tạo tác thích hợp.

### Tích trữ Bối cảnh

Vấn đề: Một người duy trì bối cảnh mà không có đầu vào của nhóm.
Giải pháp: Xem xét các tạo tác bối cảnh trong các yêu cầu kéo; thực hiện cập nhật cộng tác.

### Đặc tả Quá mức

Vấn đề: Bối cảnh trở nên quá chi tiết đến mức không thể duy trì.
Giải pháp: Giữ các tạo tác tập trung vào các quyết định ảnh hưởng đến hành vi AI và sự liên kết nhóm.

## Tích hợp với Công cụ Phát triển

### Tích hợp IDE

Cấu hình IDE của bạn để hiển thị các tệp bối cảnh nổi bật:

- Ghim conductor/product.md để tham khảo nhanh
- Thêm tech-stack.md vào ghi chú dự án
- Tạo các đoạn mã cho các mẫu phổ biến từ hướng dẫn phong cách

### Git Hooks

Xem xét các pre-commit hooks mà:

- Cảnh báo khi các phụ thuộc thay đổi mà không cần cập nhật tech-stack.md
- Nhắc nhở cập nhật product.md khi các nhánh tính năng hợp nhất
- Xác thực cú pháp tạo tác bối cảnh

### Tích hợp CI/CD

Bao gồm xác thực bối cảnh trong các đường ống:

- Kiểm tra tech-stack.md khớp với các phụ thuộc thực tế
- Xác minh các liên kết trong tài liệu bối cảnh phân giải được
- Đảm bảo trạng thái tracks.md khớp với trạng thái nhánh git

## Tính liên tục của Phiên

Conductor hỗ trợ phát triển đa phiên thông qua sự kiên trì của bối cảnh:

### Bắt đầu một Phiên Mới

1. Đọc index.md để định hướng bản thân
2. Kiểm tra tracks.md cho công việc đang hoạt động
3. Xem xét plan.md của theo dõi liên quan cho nhiệm vụ hiện tại
4. Xác minh các tạo tác bối cảnh là hiện tại

### Kết thúc một Phiên

1. Cập nhật plan.md với tiến độ hiện tại
2. Ghi chú bất kỳ chặn hoặc quyết định nào được đưa ra
3. Cam kết công việc đang tiến hành với trạng thái rõ ràng
4. Cập nhật tracks.md nếu trạng thái thay đổi

### Xử lý Gián đoạn

Nếu bị gián đoạn giữa nhiệm vụ:

1. Đánh dấu nhiệm vụ là `[~]` với ghi chú về điểm dừng
2. Cam kết công việc đang tiến hành vào nhánh tính năng
3. Tài liệu hóa bất kỳ quyết định chưa cam kết nào trong plan.md

## Thực tiễn Tốt nhất

1. **Đọc bối cảnh trước**: Luôn đọc các tạo tác liên quan trước khi bắt đầu công việc
2. **Cập nhật nhỏ**: Thực hiện các thay đổi bối cảnh gia tăng, không phải viết lại lớn
3. **Liên kết quyết định**: Tham chiếu bối cảnh khi đưa ra lựa chọn triển khai
4. **Lập phiên bản bối cảnh**: Cam kết thay đổi bối cảnh cùng với thay đổi mã
5. **Xem xét bối cảnh**: Bao gồm xem xét tạo tác bối cảnh trong đánh giá mã
6. **Xác thực thường xuyên**: Chạy danh sách kiểm tra xác thực bối cảnh trước khi làm việc lớn
7. **Thông báo thay đổi**: Thông báo cho nhóm khi các tạo tác bối cảnh thay đổi đáng kể
8. **Bảo tồn lịch sử**: Sử dụng git để theo dõi sự phát triển của bối cảnh theo thời gian
9. **Đặt câu hỏi về sự cũ kỹ**: Nếu bối cảnh cảm thấy sai, hãy điều tra và cập nhật
10. **Giữ cho nó có thể hành động**: Mọi mục bối cảnh nên thông báo cho một quyết định hoặc hành vi
