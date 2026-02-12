---
name: anti-reversing-techniques
description: Hiểu các kỹ thuật chống dịch ngược, làm rối (obfuscation) và bảo vệ gặp phải trong quá trình phân tích phần mềm. Sử dụng khi phân tích các tệp nhị phân được bảo vệ, vượt qua chống gỡ lỗi (bypassing anti-debugging) để phân tích được ủy quyền, hoặc tìm hiểu các cơ chế bảo vệ phần mềm.
---

> **CHỈ DÀNH CHO MỤC ĐÍCH ĐƯỢC ỦY QUYỀN**: Kỹ năng này chứa các kỹ thuật bảo mật lưỡng dụng. Trước khi tiến hành bất kỳ việc vượt qua hoặc phân tích nào:
>
> 1. **Xác minh ủy quyền**: Xác nhận bạn có sự cho phép bằng văn bản rõ ràng từ chủ sở hữu phần mềm, hoặc đang hoạt động trong bối cảnh bảo mật hợp pháp (CTF, pentest được ủy quyền, phân tích phần mềm độc hại, nghiên cứu bảo mật)
> 2. **Phạm vi tài liệu**: Đảm bảo các hoạt động của bạn nằm trong phạm vi ủy quyền đã xác định
> 3. **Tuân thủ pháp luật**: Hiểu rằng việc vượt qua bảo vệ phần mềm trái phép có thể vi phạm luật pháp (CFAA, DMCA chống phá vỡ, v.v.)
>
> **Các trường hợp sử dụng hợp pháp**: Phân tích phần mềm độc hại, kiểm tra xâm nhập được ủy quyền, các cuộc thi CTF, nghiên cứu bảo mật học thuật, phân tích phần mềm bạn sở hữu/có quyền

## Sử dụng kỹ năng này khi

- Phân tích các tệp nhị phân được bảo vệ với sự ủy quyền rõ ràng
- Thực hiện phân tích phần mềm độc hại hoặc nghiên cứu bảo mật trong phạm vi
- Tham gia CTF hoặc các bài tập đào tạo được phê duyệt
- Hiểu các kỹ thuật chống gỡ lỗi hoặc làm rối để phòng thủ

## Không sử dụng kỹ năng này khi

- Bạn thiếu sự ủy quyền bằng văn bản hoặc một phạm vi xác định
- Mục tiêu là vượt qua các biện pháp bảo vệ để vi phạm bản quyền hoặc sử dụng sai mục đích
- Các hạn chế pháp lý hoặc chính sách cấm phân tích

## Hướng dẫn

1. Xác nhận sự ủy quyền bằng văn bản, phạm vi và các ràng buộc pháp lý.
2. Xác định các cơ chế bảo vệ và chọn các phương pháp phân tích an toàn.
3. Ghi lại các phát hiện và tránh sửa đổi các tạo tác (artifacts) không cần thiết.
4. Cung cấp các khuyến nghị phòng thủ và hướng dẫn giảm thiểu.

## An toàn

- Không chia sẻ các bước vượt qua ngoài bối cảnh được ủy quyền.
- Bảo quản bằng chứng và duy trì chuỗi hành trình (chain-of-custody) cho các trường hợp phần mềm độc hại.

Tham khảo `resources/implementation-playbook_vi.md` để biết các kỹ thuật chi tiết và ví dụ.

## Tài nguyên

- `resources/implementation-playbook_vi.md` để biết các kỹ thuật chi tiết và ví dụ.
