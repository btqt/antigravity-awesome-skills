# Khám phá Ngữ cảnh

> Trước khi đề xuất bất kỳ kiến trúc nào, hãy thu thập ngữ cảnh.

## Phân cấp Câu hỏi (Hỏi Người dùng ĐẦU TIÊN)

1. **Quy mô (Scale)**
   - Bao nhiêu người dùng? (10, 1K, 100K, 1M+)
   - Khối lượng dữ liệu? (MB, GB, TB)
   - Tỷ lệ giao dịch? (mỗi giây/phút)

2. **Nhóm (Team)**
   - Nhà phát triển độc lập hay nhóm?
   - Quy mô và chuyên môn của nhóm?
   - Phân tán hay cùng địa điểm (co-located)?

3. **Dòng thời gian (Timeline)**
   - MVP/Nguyên mẫu hay sản phẩm dài hạn?
   - Áp lực thời gian ra thị trường?

4. **Tên miền (Domain)**
   - Nặng về CRUD hay logic nghiệp vụ phức tạp?
   - Yêu cầu thời gian thực?
   - Tuân thủ/quy định?

5. **Ràng buộc (Constraints)**
   - Hạn chế ngân sách?
   - Hệ thống cũ cần tích hợp?
   - Sở thích ngăn xếp công nghệ?

## Ma trận Phân loại Dự án

```
                    MVP              SaaS           Doanh nghiệp (Enterprise)
┌─────────────────────────────────────────────────────────────┐
│ Quy mô       │ <1K           │ 1K-100K      │ 100K+        │
│ Nhóm         │ Độc lập       │ 2-10         │ 10+          │
│ Dòng thời gian│ Nhanh (tuần)  │ Trung bình (tháng)│ Dài (năm)   │
│ Kiến trúc    │ Đơn giản      │ Modular      │ Phân tán     │
│ Mẫu          │ Tối thiểu     │ Chọn lọc     │ Toàn diện    │
│ Ví dụ        │ Next.js API   │ NestJS       │ Microservices│
└─────────────────────────────────────────────────────────────┘
```
