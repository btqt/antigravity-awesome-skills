---
name: attack-tree-construction
description: Xây dựng các cây tấn công (attack trees) toàn diện để trực quan hóa các đường dẫn đe dọa. Sử dụng khi lập bản đồ các kịch bản tấn công, xác định các lỗ hổng phòng thủ, hoặc truyền đạt các rủi ro bảo mật cho các bên liên quan.
---

# Xây dựng Cây Tấn công (Attack Tree Construction)

Trực quan hóa và phân tích đường dẫn tấn công có hệ thống.

## Sử dụng kỹ năng này khi

- Trực quan hóa các kịch bản tấn công phức tạp
- Xác định các lỗ hổng phòng thủ và các ưu tiên
- Truyền đạt rủi ro cho các bên liên quan
- Lập kế hoạch đầu tư phòng thủ hoặc phạm vi kiểm thử

## Không sử dụng kỹ năng này khi

- Bạn thiếu sự cho phép hoặc phạm vi xác định để mô hình hóa hệ thống
- Nhiệm vụ là đánh giá rủi ro chung mà không có mô hình hóa đường dẫn tấn công
- Yêu cầu không liên quan đến đánh giá hoặc thiết kế bảo mật

## Hướng dẫn

- Xác nhận phạm vi, tài sản và mục tiêu của kẻ tấn công cho nút gốc (root node).
- Phân tách thành các mục tiêu phụ với cấu trúc AND/OR.
- Chú thích các nút lá (leaves) với chi phí, kỹ năng, thời gian và khả năng bị phát hiện.
- Lập bản đồ các biện pháp giảm thiểu theo từng nhánh và ưu tiên các đường dẫn có tác động cao.
- Nếu cần các mẫu chi tiết, hãy mở `resources/implementation-playbook.md`.

## An toàn

- Chỉ chia sẻ cây tấn công với các bên liên quan được ủy quyền.
- Tránh bao gồm các chi tiết khai thác nhạy cảm trừ khi bắt buộc.

## Tài nguyên

- `resources/implementation-playbook.md` để biết các mẫu, khuôn mẫu (templates) và ví dụ chi tiết.
