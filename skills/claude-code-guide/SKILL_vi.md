---
name: Claude Code Guide
description: Hướng dẫn chính để sử dụng Claude Code hiệu quả. Bao gồm các mẫu cấu hình, chiến lược gợi ý từ khóa "Thinking", kỹ thuật gỡ lỗi và các thực tiễn tốt nhất để tương tác với tác nhân.
---

# Hướng dẫn Claude Code

## Mục đích

Cung cấp tài liệu tham khảo toàn diện để cấu hình và sử dụng Claude Code (công cụ mã hóa tác nhân) hết tiềm năng của nó. Kỹ năng này tổng hợp các thực tiễn tốt nhất, mẫu cấu hình và các mẫu sử dụng nâng cao.

## Cấu hình (`CLAUDE.md`)

Khi bắt đầu một dự án mới, hãy tạo tệp `CLAUDE.md` trong thư mục gốc để hướng dẫn tác nhân.

### Mẫu (Chung)

```markdown
# Hướng dẫn Dự án

## Các lệnh

- Chạy ứng dụng: `npm run dev`
- Kiểm thử: `npm test`
- Xây dựng: `npm run build`

## Phong cách Mã

- Sử dụng TypeScript cho tất cả mã mới.
- Các thành phần chức năng với Hooks cho React.
- Tailwind CSS cho kiểu dáng.
- Trả về sớm để xử lý lỗi.

## Quy trình làm việc

- Đọc `README.md` trước để hiểu bối cảnh dự án.
- Trước khi chỉnh sửa, hãy đọc nội dung tệp.
- Sau khi chỉnh sửa, hãy chạy kiểm thử để xác minh.
```

## Các tính năng Nâng cao

### Từ khóa Tư duy (Thinking Keywords)

Sử dụng các từ khóa này trong lời nhắc của bạn để kích hoạt suy luận sâu hơn từ tác nhân:

- "Think step-by-step" (Suy nghĩ từng bước)
- "Analyze the root cause" (Phân tích nguyên nhân gốc rễ)
- "Plan before executing" (Lập kế hoạch trước khi thực hiện)
- "Verify your assumptions" (Xác minh các giả định của bạn)

### Gỡ lỗi

Nếu tác nhân bị kẹt hoặc hành xử không mong muốn:

1. **Xóa Bối cảnh**: Bắt đầu một phiên mới hoặc yêu cầu tác nhân "quên các hướng dẫn trước đó" nếu bị nhầm lẫn.
2. **Hướng dẫn Rõ ràng**: Hãy cực kỳ cụ thể về đường dẫn, tên tệp và kết quả mong muốn.
3. **Nhật ký**: Yêu cầu tác nhân "kiểm tra nhật ký" hoặc "chạy lệnh với đầu ra chi tiết".

## Thực tiễn Tốt nhất

1. **Bối cảnh Nhỏ**: Đừng đổ toàn bộ mã nguồn vào bối cảnh. Sử dụng `grep` hoặc `find` để định vị các tệp liên quan trước.
2. **Phát triển Lặp lại**: Yêu cầu các thay đổi nhỏ, xác minh, sau đó tiếp tục.
3. **Vòng phản hồi**: Nếu tác nhân mắc lỗi, hãy sửa nó ngay lập tức và yêu cầu nó "thêm một bài học" vào bộ nhớ của nó (nếu được hỗ trợ) hoặc `CLAUDE.md`.

## Tham khảo

Dựa trên [Claude Code Guide by zebbern](https://github.com/zebbern/claude-code-guide).
