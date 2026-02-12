# Các Nguyên tắc REST

> Thiết kế API dựa trên tài nguyên - danh từ không phải động từ.

## Quy tắc Đặt tên Tài nguyên

```
Nguyên tắc:
├── Sử dụng DANH TỪ, không phải động từ (tài nguyên, không phải hành động)
├── Sử dụng dạng SỐ NHIỀU (/users không phải /user)
├── Sử dụng chữ thường với dấu gạch ngang (/user-profiles)
├── Lồng nhau cho các mối quan hệ (/users/123/posts)
└── Giữ nông (tối đa 3 cấp)
```

## Lựa chọn Phương thức HTTP

| Phương thức | Mục đích                    | Lũy đẳng? | Body? |
| ----------- | --------------------------- | --------- | ----- |
| **GET**     | Đọc tài nguyên              | Có        | Không |
| **POST**    | Tạo tài nguyên mới          | Không     | Có    |
| **PUT**     | Thay thế toàn bộ tài nguyên | Có        | Có    |
| **PATCH**   | Cập nhật một phần           | Không     | Có    |
| **DELETE**  | Xóa tài nguyên              | Có        | Không |

## Lựa chọn Mã Trạng thái

| Tình huống         | Mã  | Tại sao                              |
| ------------------ | --- | ------------------------------------ |
| Thành công (đọc)   | 200 | Thành công tiêu chuẩn                |
| Đã tạo             | 201 | Tài nguyên mới được tạo              |
| Không có nội dung  | 204 | Thành công, không có gì để trả về    |
| Yêu cầu xấu        | 400 | Yêu cầu không đúng định dạng         |
| Chưa xác thực      | 401 | Thiếu/xác thực không hợp lệ          |
| Bị cấm             | 403 | Xác thực hợp lệ, không có quyền      |
| Không tìm thấy     | 404 | Tài nguyên không tồn tại             |
| Xung đột           | 409 | Xung đột trạng thái (trùng lặp)      |
| Lỗi xác thực       | 422 | Cú pháp hợp lệ, dữ liệu không hợp lệ |
| Bị giới hạn tốc độ | 429 | Quá nhiều yêu cầu                    |
| Lỗi máy chủ        | 500 | Lỗi của chúng tôi                    |
