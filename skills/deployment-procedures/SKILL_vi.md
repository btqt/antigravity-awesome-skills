---
name: deployment-procedures
description: Các nguyên tắc triển khai sản xuất và ra quyết định. Quy trình triển khai an toàn, chiến lược rollback và xác minh. Dạy tư duy, không phải kịch bản.
allowed-tools: Read, Glob, Grep, Bash
---

# Quy Trình Triển Khai (Deployment Procedures)

> Các nguyên tắc triển khai và ra quyết định cho các bản phát hành sản xuất an toàn.
> **Học cách TƯ DUY, không sao chép kịch bản.**

---

## ⚠️ Cách Sử dụng Kỹ năng này

Kỹ năng này dạy **các nguyên tắc triển khai**, không phải sao chép các tập lệnh bash.

- Mọi triển khai đều là duy nhất
- Hiểu LÝ DO đằng sau mỗi bước
- Thích ứng các quy trình với nền tảng của bạn

---

## 1. Lựa chọn Nền tảng

### Cây Quyết định

```
Bạn đang triển khai cái gì?
│
├── Static site / JAMstack
│   └── Vercel, Netlify, Cloudflare Pages
│
├── Simple web app
│   ├── Managed → Railway, Render, Fly.io
│   └── Control → VPS + PM2/Docker
│
├── Microservices
│   └── Điều phối container
│
└── Serverless
    └── Edge functions, Lambda
```

### Mỗi Nền tảng Có Quy trình Khác nhau

| Nền tảng           | Phương pháp Triển khai       |
| ------------------ | ---------------------------- |
| **Vercel/Netlify** | Git push, triển khai tự động |
| **Railway/Render** | Git push hoặc CLI            |
| **VPS + PM2**      | SSH + các bước thủ công      |
| **Docker**         | Image push + điều phối       |
| **Kubernetes**     | kubectl apply                |

---

## 2. Các Nguyên tắc Trước Triển khai

### 4 Danh mục Xác minh

| Danh mục          | Kiểm tra cái gì                               |
| ----------------- | --------------------------------------------- |
| **Chất lượng Mã** | Tests thông qua, linting sạch, đã review      |
| **Build**         | Build sản xuất hoạt động, không có cảnh báo   |
| **Môi trường**    | Biến môi trường đã thiết lập, bí mật hiện tại |
| **An toàn**       | Sao lưu hoàn tất, kế hoạch rollback sẵn sàng  |

### Danh sách Kiểm tra Trước Triển khai

- [ ] Tất cả tests thông qua
- [ ] Mã đã được review và phê duyệt
- [ ] Build sản xuất thành công
- [ ] Biến môi trường đã xác minh
- [ ] Di chuyển cơ sở dữ liệu sẵn sàng (nếu có)
- [ ] Kế hoạch rollback được tài liệu hóa
- [ ] Nhóm đã được thông báo
- [ ] Giám sát sẵn sàng

---

## 3. Các Nguyên tắc Quy trình Triển khai

### Quy trình 5 Giai đoạn

```
1. PREPARE (CHUẨN BỊ)
   └── Xác minh mã, build, biến môi trường

2. BACKUP (SAO LƯU)
   └── Lưu trạng thái hiện tại trước khi thay đổi

3. DEPLOY (TRIỂN KHAI)
   └── Thực thi với giám sát mở

4. VERIFY (XÁC MINH)
   └── Kiểm tra sức khỏe, nhật ký, luồng chính

5. CONFIRM or ROLLBACK (XÁC NHẬN hoặc HOÀN TÁC)
   └── Mọi thứ ổn? Xác nhận. Có vấn đề? Rollback.
```

### Các Nguyên tắc Giai đoạn

| Giai đoạn   | Nguyên tắc                                |
| ----------- | ----------------------------------------- |
| **Prepare** | Không bao giờ triển khai mã chưa kiểm thử |
| **Backup**  | Không thể rollback nếu không có sao lưu   |
| **Deploy**  | Theo dõi quá trình, đừng rời đi           |
| **Verify**  | Tin tưởng nhưng hãy xác minh              |
| **Confirm** | Có sẵn kích hoạt rollback                 |

---

## 4. Xác minh Sau Triển khai

### Những gì cần Xác minh

| Kiểm tra                   | Tại sao                                  |
| -------------------------- | ---------------------------------------- |
| **Điểm cuối sức khỏe**     | Dịch vụ đang chạy                        |
| **Nhật ký lỗi**            | Không có lỗi mới                         |
| **Luồng người dùng chính** | Các tính năng quan trọng hoạt động       |
| **Hiệu suất**              | Thời gian phản hồi có thể chấp nhận được |

### Cửa sổ Xác minh

- **5 phút đầu**: Giám sát tích cực
- **15 phút**: Xác nhận ổn định
- **1 giờ**: Xác minh cuối cùng
- **Ngày tiếp theo**: Xem lại các chỉ số

---

## 5. Các Nguyên tắc Rollback

### Khi nào nên Rollback

| Triệu chứng             | Hành động                       |
| ----------------------- | ------------------------------- |
| Dịch vụ ngừng hoạt động | Rollback ngay lập tức           |
| Lỗi nghiêm trọng        | Rollback                        |
| Hiệu suất giảm >50%     | Cân nhắc rollback               |
| Vấn đề nhỏ              | Khắc phục chuyển tiếp nếu nhanh |

### Chiến lược Rollback theo Nền tảng

| Nền tảng           | Phương pháp Rollback             |
| ------------------ | -------------------------------- |
| **Vercel/Netlify** | Triển khai lại commit trước đó   |
| **Railway/Render** | Rollback trong bảng điều khiển   |
| **VPS + PM2**      | Khôi phục sao lưu, khởi động lại |
| **Docker**         | Tag image trước đó               |
| **K8s**            | kubectl rollout undo             |

### Các Nguyên tắc Rollback

1. **Tốc độ hơn hoàn hảo**: Rollback trước, gỡ lỗi sau
2. **Đừng làm phức tạp lỗi**: Một rollback, không phải nhiều thay đổi
3. **Giao tiếp**: Nói cho nhóm biết chuyện gì đã xảy ra
4. **Hậu kiểm (Post-mortem)**: Hiểu tại sao sau khi ổn định

---

## 6. Triển khai Không Thời gian chết

### Chiến lược

| Chiến lược     | Cách thức Hoạt động                      |
| -------------- | ---------------------------------------- |
| **Rolling**    | Thay thế từng phiên bản một              |
| **Blue-Green** | Chuyển đổi lưu lượng giữa các môi trường |
| **Canary**     | Chuyển hướng lưu lượng dần dần           |

### Các Nguyên tắc Lựa chọn

| Tình huống           | Chiến lược                             |
| -------------------- | -------------------------------------- |
| Phát hành tiêu chuẩn | Rolling                                |
| Thay đổi rủi ro cao  | Blue-green (dễ rollback)               |
| Cần xác thực         | Canary (thử nghiệm với lưu lượng thực) |

---

## 7. Quy trình Khẩn cấp

### Ưu tiên Dịch vụ Ngừng hoạt động

1. **Đánh giá**: Triệu chứng là gì?
2. **Sửa nhanh**: Khởi động lại nếu không rõ ràng
3. **Rollback**: Nếu khởi động lại không giúp ích
4. **Điều tra**: Sau khi ổn định

### Thứ tự Điều tra

| Kiểm tra         | Vấn đề Phổ biến     |
| ---------------- | ------------------- |
| **Logs**         | Lỗi, ngoại lệ       |
| **Resources**    | Đĩa đầy, bộ nhớ     |
| **Network**      | DNS, tường lửa      |
| **Dependencies** | Cơ sở dữ liệu, APIs |

---

## 8. Các Mẫu Chống (Anti-Patterns)

| ❌ Đừng                   | ✅ Nên                       |
| ------------------------- | ---------------------------- |
| Triển khai vào thứ Sáu    | Triển khai sớm trong tuần    |
| Vội vàng triển khai       | Tuân thủ quy trình           |
| Bỏ qua staging            | Luôn kiểm thử trước          |
| Triển khai không sao lưu  | Sao lưu trước khi triển khai |
| Rời đi sau khi triển khai | Giám sát trong 15+ phút      |
| Nhiều thay đổi cùng lúc   | Mỗi lần một thay đổi         |

---

## 9. Danh sách Kiểm tra Quyết định

Trước khi triển khai:

- [ ] **Quy trình phù hợp với nền tảng?**
- [ ] **Chiến lược sao lưu sẵn sàng?**
- [ ] **Kế hoạch rollback được tài liệu hóa?**
- [ ] **Giám sát đã được cấu hình?**
- [ ] **Nhóm đã được thông báo?**
- [ ] **Thời gian để giám sát sau đó?**

---

## 10. Thực tiễn Tốt nhất

1. **Triển khai nhỏ, thường xuyên** hơn là phát hành lớn
2. **Cờ tính năng** cho các thay đổi rủi ro
3. **Tự động hóa** các bước lặp lại
4. **Tài liệu hóa** mọi lần triển khai
5. **Xem xét** những gì đã sai sau khi có vấn đề
6. **Kiểm thử rollback** trước khi bạn cần nó

---

> **Ghi nhớ:** Mọi triển khai đều là rủi ro. Giảm thiểu rủi ro thông qua sự chuẩn bị, không phải tốc độ.
