# Sổ tay Triển khai Quy trình làm việc Antigravity

Tài liệu này giải thích cách một tác nhân nên thực hiện điều phối dựa trên quy trình làm việc.

## Hợp đồng Thực thi

Đối với mỗi quy trình làm việc:

1. Xác nhận mục tiêu và phạm vi.
2. Chọn quy trình làm việc phù hợp nhất.
3. Thực hiện các bước quy trình làm việc theo thứ tự.
4. Tạo ra một tạo tác cụ thể cho mỗi bước.
5. Xác thực trước khi tiếp tục.

## Ví dụ về Tạo tác Bước

- Bước lập kế hoạch -> tài liệu phạm vi hoặc danh sách kiểm tra cột mốc.
- Bước xây dựng -> thay đổi mã và ghi chú triển khai.
- Bước kiểm thử -> kết quả kiểm thử và phân loại lỗi.
- Bước phát hành -> danh sách kiểm tra triển khai và nhật ký rủi ro.

## Các Biện pháp An toàn

- Không bao giờ thực hiện các hành động phá hoại mà không có sự chấp thuận rõ ràng của người dùng.
- Nếu thiếu một kỹ năng cần thiết, hãy nêu rõ khoảng trống và quay lại kỹ năng có sẵn gần nhất.
- Khi liên quan đến kiểm thử bảo mật, hãy đảm bảo sự ủy quyền là rõ ràng.

## Định dạng Hoàn thành Được đề xuất

Khi hoàn thành quy trình làm việc, trả về:

1. Các bước đã hoàn thành
2. Các tạo tác đã tạo ra
3. Bằng chứng xác thực
4. Các rủi ro mở
5. Hành động tiếp theo được đề xuất
