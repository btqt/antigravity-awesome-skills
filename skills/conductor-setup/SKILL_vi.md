---
name: conductor-setup
description: Khởi tạo dự án với các tạo tác Conductor (định nghĩa sản phẩm, ngăn xếp công nghệ, quy trình làm việc, hướng dẫn phong cách)
metadata:
  argument-hint: "[--resume]"
---

# Thiết lập Conductor (Conductor Setup)

Khởi tạo hoặc tiếp tục thiết lập dự án Conductor. Lệnh này tạo tài liệu nền tảng cho dự án thông qua Hỏi & Đáp tương tác.

## Sử dụng kỹ năng này khi

- Làm việc trên các nhiệm vụ hoặc quy trình thiết lập conductor
- Cần hướng dẫn, thực tiễn tốt nhất hoặc danh sách kiểm tra cho thiết lập conductor

## Không sử dụng kỹ năng này khi

- Nhiệm vụ không liên quan đến thiết lập conductor
- Bạn cần một miền hoặc công cụ khác ngoài phạm vi này

## Hướng dẫn

- Làm rõ mục tiêu, ràng buộc và đầu vào cần thiết.
- Áp dụng các thực tiễn tốt nhất có liên quan và xác thực kết quả.
- Cung cấp các bước có thể hành động và xác minh.
- Nếu cần các ví dụ chi tiết, hãy mở `resources/implementation-playbook.md`.

## Kiểm tra Trước chuyến bay (Pre-flight Checks)

1. Kiểm tra xem thư mục `conductor/` đã tồn tại trong thư mục gốc của dự án chưa:
   - Nếu `conductor/product.md` tồn tại: Hỏi người dùng có muốn tiếp tục thiết lập hay khởi tạo lại
   - Nếu `conductor/setup_state.json` tồn tại với trạng thái chưa hoàn thành: Đề nghị tiếp tục từ bước cuối cùng

2. Phát hiện loại dự án bằng cách kiểm tra các chỉ báo hiện có:
   - **Greenfield (dự án mới)**: Không có .git, không package.json, không requirements.txt, không go.mod, không thư mục src/
   - **Brownfield (dự án hiện có)**: Bất kỳ điều nào ở trên tồn tại

3. Tải hoặc tạo `conductor/setup_state.json`:
   ```json
   {
     "status": "in_progress",
     "project_type": "greenfield|brownfield",
     "current_section": "product|guidelines|tech_stack|workflow|styleguides",
     "current_question": 1,
     "completed_sections": [],
     "answers": {},
     "files_created": [],
     "started_at": "ISO_TIMESTAMP",
     "last_updated": "ISO_TIMESTAMP"
   }
   ```

## Giao thức Hỏi & Đáp Tương tác

**QUY TẮC QUAN TRỌNG:**

- Hỏi MỘT câu hỏi mỗi lần
- Đợi phản hồi của người dùng trước khi tiếp tục
- Cung cấp 2-3 câu trả lời gợi ý cộng với tùy chọn "Nhập câu trả lời của bạn"
- Tối đa 5 câu hỏi mỗi phần
- Cập nhật `setup_state.json` sau mỗi bước thành công
- Xác thực ghi tệp thành công trước khi tiếp tục

### Phần 1: Định nghĩa Sản phẩm (tối đa 5 câu hỏi)

**Q1: Tên Dự án**

```
Tên dự án của bạn là gì?

Gợi ý:
1. [Suy luận từ tên thư mục]
2. [Suy luận từ package.json/go.mod nếu là brownfield]
3. Nhập câu trả lời của bạn
```

**Q2: Mô tả Dự án**

```
Mô tả dự án của bạn trong một câu.

Gợi ý:
1. Một ứng dụng web [làm X]
2. Một công cụ CLI để [làm Y]
3. Nhập câu trả lời của bạn
```

**Q3: Tuyên bố Vấn đề**

```
Dự án này giải quyết vấn đề gì?

Gợi ý:
1. Người dùng gặp khó khăn khi [điểm đau]
2. Không có cách tốt để [nhu cầu]
3. Nhập câu trả lời của bạn
```

**Q4: Người dùng Mục tiêu**

```
Ai là người dùng chính?

Gợi ý:
1. Các nhà phát triển xây dựng [X]
2. Người dùng cuối cần [Y]
3. Các nhóm nội bộ quản lý [Z]
4. Nhập câu trả lời của bạn
```

**Q5: Mục tiêu Chính (tùy chọn)**

```
2-3 mục tiêu chính cho dự án này là gì? (Nhấn enter để bỏ qua)
```

### Phần 2: Hướng dẫn Sản phẩm (tối đa 3 câu hỏi)

**Q1: Giọng điệu và Tông màu**

```
Tài liệu và văn bản UI nên sử dụng giọng điệu/tông màu nào?

Gợi ý:
1. Chuyên nghiệp và kỹ thuật
2. Thân thiện và dễ tiếp cận
3. Ngắn gọn và trực tiếp
4. Nhập câu trả lời của bạn
```

**Q2: Nguyên tắc Thiết kế**

```
Nguyên tắc thiết kế nào hướng dẫn dự án này?

Gợi ý:
1. Đơn giản hơn tính năng
2. Hiệu suất là trên hết
3. Tập trung vào trải nghiệm nhà phát triển
4. An toàn và độ tin cậy của người dùng
5. Nhập câu trả lời của bạn (phân tách bằng dấu phẩy)
```

### Phần 3: Ngăn xếp Công nghệ (tối đa 5 câu hỏi)

Đối với **dự án brownfield**, trước tiên hãy phân tích mã hiện có:

- Chạy `Glob` để tìm package.json, requirements.txt, go.mod, Cargo.toml, v.v.
- Phân tích các tệp được phát hiện để điền trước ngăn xếp công nghệ
- Trình bày kết quả và yêu cầu xác nhận/bổ sung

**Q1: Ngôn ngữ Chính**

```
Dự án này sử dụng (các) ngôn ngữ chính nào?

[Đối với brownfield: "Tôi đã phát hiện: Python 3.11, JavaScript. Điều này có đúng không?"]

Gợi ý:
1. TypeScript
2. Python
3. Go
4. Rust
5. Nhập câu trả lời của bạn (phân tách bằng dấu phẩy)
```

**Q2: Khuôn khổ Frontend (nếu có)**

```
Khuôn khổ frontend nào (nếu có)?

Gợi ý:
1. React
2. Vue
3. Next.js
4. Không có / Chỉ CLI
5. Nhập câu trả lời của bạn
```

**Q3: Khuôn khổ Backend (nếu có)**

```
Khuôn khổ backend nào (nếu có)?

Gợi ý:
1. Express / Fastify
2. Django / FastAPI
3. Thư viện chuẩn Go
4. Không có / Chỉ Frontend
5. Nhập câu trả lời của bạn
```

**Q4: Cơ sở dữ liệu (nếu có)**

```
Cơ sở dữ liệu nào (nếu có)?

Gợi ý:
1. PostgreSQL
2. MongoDB
3. SQLite
4. Không có / Không trạng thái (Stateless)
5. Nhập câu trả lời của bạn
```

**Q5: Cơ sở hạ tầng**

```
Điều này sẽ được triển khai ở đâu?

Gợi ý:
1. AWS (Lambda, ECS, v.v.)
2. Vercel / Netlify
3. Tự lưu trữ / Docker
4. Chưa quyết định
5. Nhập câu trả lời của bạn
```

### Phần 4: Tùy chọn Quy trình làm việc (tối đa 4 câu hỏi)

**Q1: Mức độ Nghiêm ngặt TDD**

```
TDD nên được thực thi nghiêm ngặt như thế nào?

Gợi ý:
1. Nghiêm ngặt - yêu cầu kiểm thử trước khi triển khai
2. Vừa phải - kiểm thử được khuyến khích, không bị chặn
3. Linh hoạt - kiểm thử được khuyến nghị cho logic phức tạp
```

**Q2: Chiến lược Cam kết**

```
Chiến lược cam kết nào nên được tuân theo?

Gợi ý:
1. Conventional Commits (feat:, fix:, v.v.)
2. Thông điệp mô tả, không yêu cầu định dạng
3. Squash cam kết cho mỗi nhiệm vụ
```

**Q3: Yêu cầu Đánh giá Mã**

```
Chính sách đánh giá mã nào?

Gợi ý:
1. Bắt buộc cho tất cả các thay đổi
2. Bắt buộc cho các thay đổi không nhỏ
3. Tùy chọn / tự đánh giá OK
```

**Q4: Điểm kiểm tra Xác minh**

```
Khi nào nên yêu cầu xác minh thủ công?

Gợi ý:
1. Sau khi hoàn thành mỗi giai đoạn
2. Sau khi hoàn thành mỗi nhiệm vụ
3. Chỉ khi hoàn thành theo dõi
```

### Phần 5: Hướng dẫn Phong cách Mã (tối đa 2 câu hỏi)

**Q1: Các ngôn ngữ cần bao gồm**

```
Hướng dẫn phong cách ngôn ngữ nào nên được tạo?

[Dựa trên các ngôn ngữ được phát hiện, chọn trước]

Tùy chọn:
1. TypeScript/JavaScript
2. Python
3. Go
4. Rust
5. Tất cả các ngôn ngữ được phát hiện
6. Bỏ qua hướng dẫn phong cách
```

**Q2: Quy ước Hiện có**

```
Bạn có cấu hình linting/formatting hiện có để kết hợp không?

[Đối với brownfield: "Tôi tìm thấy .eslintrc, .prettierrc. Tôi có nên kết hợp những thứ này không?"]

Gợi ý:
1. Có, sử dụng cấu hình hiện có
2. Không, tạo hướng dẫn mới
3. Bỏ qua bước này
```

## Tạo Tạo tác (Artifact Generation)

Sau khi hoàn thành Hỏi & Đáp, tạo các tệp sau:

### 1. conductor/index.md

```markdown
# Conductor - [Tên Dự án]

Trung tâm điều hướng cho bối cảnh dự án.

## Liên kết Nhanh

- [Định nghĩa Sản phẩm](./product.md)
- [Hướng dẫn Sản phẩm](./product-guidelines.md)
- [Ngăn xếp Công nghệ](./tech-stack.md)
- [Quy trình làm việc](./workflow.md)
- [Theo dõi](./tracks.md)

## Theo dõi Đang hoạt động

<!-- Auto-populated by /conductor:new-track -->

## Bắt đầu

Chạy `/conductor:new-track` để tạo theo dõi tính năng đầu tiên của bạn.
```

### 2. conductor/product.md

Mẫu được điền với các câu trả lời Hỏi & Đáp cho:

- Tên và mô tả dự án
- Tuyên bố vấn đề
- Người dùng mục tiêu
- Mục tiêu chính

### 3. conductor/product-guidelines.md

Mẫu được điền với:

- Giọng điệu và tông màu
- Nguyên tắc thiết kế
- Bất kỳ tiêu chuẩn bổ sung nào

### 4. conductor/tech-stack.md

Mẫu được điền với:

- Ngôn ngữ (với phiên bản nếu được phát hiện)
- Khuôn khổ (frontend, backend)
- Cơ sở dữ liệu
- Cơ sở hạ tầng
- Các phụ thuộc chính (cho brownfield, từ các tệp gói)

### 5. conductor/workflow.md

Mẫu được điền với:

- Chính sách TDD và mức độ nghiêm ngặt
- Chiến lược cam kết và quy ước
- Yêu cầu đánh giá mã
- Quy tắc điểm kiểm tra xác minh
- Định nghĩa vòng đời nhiệm vụ

### 6. conductor/tracks.md

```markdown
# Sổ đăng ký Theo dõi

| Status | Track ID | Title | Created | Updated |
| ------ | -------- | ----- | ------- | ------- |

<!-- Tracks registered by /conductor:new-track -->
```

### 7. conductor/code_styleguides/

Tạo các hướng dẫn phong cách đã chọn từ `$CLAUDE_PLUGIN_ROOT/templates/code_styleguides/`

## Quản lý Trạng thái

Sau mỗi lần tạo tệp thành công:

1. Cập nhật `setup_state.json`:
   - Thêm tên tệp vào mảng `files_created`
   - Cập nhật dấu thời gian `last_updated`
   - Nếu phần hoàn thành, thêm vào `completed_sections`
2. Xác minh tệp tồn tại với công cụ `Read`

## Hoàn thành

Khi tất cả các tệp được tạo:

1. Đặt trạng thái `setup_state.json` thành "complete"
2. Hiển thị tóm tắt:

   ```
   Conductor setup complete!

   Created artifacts:
   - conductor/index.md
   - conductor/product.md
   - conductor/product-guidelines.md
   - conductor/tech-stack.md
   - conductor/workflow.md
   - conductor/tracks.md
   - conductor/code_styleguides/[languages]

   Next steps:
   1. Review generated files and customize as needed
   2. Run /conductor:new-track to create your first track
   ```

## Xử lý Tiếp tục (Resume Handling)

Nếu đối số `--resume` hoặc tiếp tục từ trạng thái:

1. Tải `setup_state.json`
2. Bỏ qua các phần đã hoàn thành
3. Tiếp tục từ `current_section` và `current_question`
4. Xác minh các tệp đã tạo trước đó vẫn tồn tại
5. Nếu tệp bị thiếu, đề nghị tạo lại

## Xử lý Lỗi

- Nếu ghi tệp thất bại: Dừng và báo cáo lỗi, không cập nhật trạng thái
- Nếu người dùng hủy: Lưu trạng thái hiện tại để tiếp tục trong tương lai
- Nếu tệp trạng thái bị hỏng: Đề nghị bắt đầu lại mới hoặc cố gắng khôi phục
