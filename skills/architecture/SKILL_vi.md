---
name: architecture
description: Khung ra quyết định kiến trúc. Phân tích yêu cầu, đánh giá đánh đổi, tài liệu hóa ADR. Sử dụng khi đưa ra các quyết định kiến trúc hoặc phân tích thiết kế hệ thống.
allowed-tools: Read, Glob, Grep
---

# Khung Quyết định Kiến trúc

> "Yêu cầu thúc đẩy kiến trúc. Sự đánh đổi thông báo cho quyết định. ADR nắm bắt cơ sở lý luận."

## 🎯 Quy tắc Đọc Chọn lọc

**CHỈ đọc các tệp liên quan đến yêu cầu!** Kiểm tra bản đồ nội dung, tìm những gì bạn cần.

| Tệp                     | Mô tả                                   | Khi nào Đọc                |
| ----------------------- | --------------------------------------- | -------------------------- |
| `context-discovery.md`  | Các câu hỏi cần đặt ra, phân loại dự án | Bắt đầu thiết kế kiến trúc |
| `trade-off-analysis.md` | Mẫu ADR, khung đánh đổi                 | Ghi lại quyết định         |
| `pattern-selection.md`  | Cây quyết định, anti-patterns           | Chọn mẫu                   |
| `examples.md`           | Dụ án MVP, SaaS, Doanh nghiệp           | Triển khai tham khảo       |
| `patterns-reference.md` | Tra cứu nhanh các mẫu                   | So sánh mẫu                |

---

## 🔗 Các Kỹ năng Liên quan

| Kỹ năng                           | Sử dụng Cho                    |
| --------------------------------- | ------------------------------ |
| `@[skills/database-design]`       | Thiết kế lược đồ cơ sở dữ liệu |
| `@[skills/api-patterns]`          | Mẫu thiết kế API               |
| `@[skills/deployment-procedures]` | Kiến trúc triển khai           |

---

## Nguyên tắc Cốt lõi

**"Đơn giản là đỉnh cao của sự tinh tế."**

- Bắt đầu đơn giản
- Thêm sự phức tạp CHỈ KHI được chứng minh là cần thiết
- Bạn luôn có thể thêm các mẫu sau
- Loại bỏ sự phức tạp KHÓ HƠN NHIỀU so với việc thêm nó

---

## Danh sách Kiểm tra Xác thực

Trước khi hoàn thiện kiến trúc:

- [ ] Yêu cầu được hiểu rõ ràng
- [ ] Các ràng buộc được xác định
- [ ] Mỗi quyết định đều có phân tích đánh đổi
- [ ] Các lựa chọn thay thế đơn giản hơn đã được xem xét
- [ ] ADR được viết cho các quyết định quan trọng
- [ ] Chuyên môn của nhóm phù hợp với các mẫu đã chọn
