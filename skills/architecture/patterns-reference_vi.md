# Tham chiếu Các Mẫu Kiến trúc

> Tham chiếu nhanh cho các mẫu phổ biến với hướng dẫn sử dụng.

## Các Mẫu Truy cập Dữ liệu

| Mẫu               | Khi nào Sử dụng              | Khi nào KHÔNG Sử dụng               | Độ phức tạp |
| ----------------- | ---------------------------- | ----------------------------------- | ----------- |
| **Active Record** | CRUD đơn giản, tạo mẫu nhanh | Truy vấn phức tạp, nhiều nguồn      | Thấp        |
| **Repository**    | Cần kiểm thử, nhiều nguồn    | CRUD đơn giản, cơ sở dữ liệu đơn lẻ | Trung bình  |
| **Unit of Work**  | Giao dịch phức tạp           | Hoạt động đơn giản                  | Cao         |
| **Data Mapper**   | Tên miền phức tạp, hiệu suất | CRUD đơn giản, phát triển nhanh     | Cao         |

## Các Mẫu Logic Tên miền

| Mẫu                    | Khi nào Sử dụng                        | Khi nào KHÔNG Sử dụng                  | Độ phức tạp |
| ---------------------- | -------------------------------------- | -------------------------------------- | ----------- |
| **Transaction Script** | CRUD đơn giản, thủ tục                 | Quy tắc nghiệp vụ phức tạp             | Thấp        |
| **Table Module**       | Logic dựa trên bản ghi                 | Cần hành vi phong phú                  | Thấp        |
| **Domain Model**       | Logic nghiệp vụ phức tạp               | CRUD đơn giản                          | Trung bình  |
| **DDD (Full)**         | Tên miền phức tạp, chuyên gia tên miền | Tên miền đơn giản, không có chuyên gia | Cao         |

## Các Mẫu Hệ thống Phân tán

| Mẫu                  | Khi nào Sử dụng                   | Khi nào KHÔNG Sử dụng                             | Độ phức tạp |
| -------------------- | --------------------------------- | ------------------------------------------------- | ----------- |
| **Modular Monolith** | Nhóm nhỏ, ranh giới không rõ ràng | Ngữ cảnh rõ ràng, quy mô khác nhau                | Trung bình  |
| **Microservices**    | Quy mô khác nhau, nhóm lớn        | Nhóm nhỏ, tên miền đơn giản                       | Rất Cao     |
| **Event-Driven**     | Thời gian thực, ghép nối lỏng lẻo | Quy trình công việc đơn giản, tính nhất quán mạnh | Cao         |
| **CQRS**             | Hiệu suất đọc/ghi phân kỳ         | CRUD đơn giản, cùng mô hình                       | Cao         |
| **Saga**             | Giao dịch phân tán                | Cơ sở dữ liệu đơn lẻ, ACID đơn giản               | Cao         |

## Các Mẫu API

| Mẫu           | Khi nào Sử dụng                  | Khi nào KHÔNG Sử dụng             | Độ phức tạp |
| ------------- | -------------------------------- | --------------------------------- | ----------- |
| **REST**      | CRUD chuẩn, tài nguyên           | Thời gian thực, truy vấn phức tạp | Thấp        |
| **GraphQL**   | Truy vấn linh hoạt, nhiều client | CRUD đơn giản, nhu cầu caching    | Trung bình  |
| **gRPC**      | Dịch vụ nội bộ, hiệu suất        | API công khai, trình duyệt client | Trung bình  |
| **WebSocket** | Cập nhật thời gian thực          | Request/response đơn giản         | Trung bình  |

---

## Nguyên tắc Đơn giản

**"Bắt đầu đơn giản, thêm sự phức tạp chỉ khi được chứng minh là cần thiết."**

- Bạn luôn có thể thêm các mẫu sau
- Loại bỏ sự phức tạp KHÓ HƠN NHIỀU so với việc thêm nó
- Khi nghi ngờ, hãy chọn phương án đơn giản hơn
