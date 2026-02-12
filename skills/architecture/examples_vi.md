# Các Ví dụ Kiến trúc

> Các quyết định kiến trúc trong thế giới thực theo loại dự án.

---

## Ví dụ 1: MVP E-commerce (Nhà phát triển Độc lập)

```yaml
Yêu cầu:
  - <1000 người dùng ban đầu
  - Nhà phát triển độc lập
  - Thời gian ra thị trường nhanh (8 tuần)
  - Cân nhắc ngân sách

Các Quyết định Kiến trúc:
  Cấu trúc Ứng dụng: Monolith (đơn giản hơn cho độc lập)
  Framework: Next.js (full-stack, nhanh)
  Lớp Dữ liệu: Prisma trực tiếp (không trừu tượng hóa quá mức)
  Xác thực: JWT (đơn giản hơn OAuth)
  Thanh toán: Stripe (giải pháp được lưu trữ)
  Cơ sở dữ liệu: PostgreSQL (ACID cho đơn hàng)

Các Đánh đổi Được chấp nhận:
  - Monolith → Không thể mở rộng độc lập (nhóm không biện minh cho nó)
  - Không Repository → Ít khả năng kiểm thử hơn (CRUD đơn giản không cần nó)
  - JWT → Không đăng nhập xã hội ban đầu (có thể thêm sau)

Lộ trình Di chuyển Tương lai:
  - Người dùng > 10K → Trích xuất dịch vụ thanh toán
  - Nhóm > 3 → Thêm mẫu Repository
  - Yêu cầu đăng nhập xã hội → Thêm OAuth
```

---

## Ví dụ 2: Sản phẩm SaaS (5-10 Nhà phát triển)

```yaml
Yêu cầu:
  - 1K-100K người dùng
  - 5-10 nhà phát triển
  - Dài hạn (12+ tháng)
  - Nhiều tên miền (thanh toán, người dùng, cốt lõi)

Các Quyết định Kiến trúc:
  Cấu trúc Ứng dụng: Modular Monolith (quy mô nhóm tối ưu)
  Framework: NestJS (thiết kế theo module)
  Lớp Dữ liệu: Mẫu Repository (kiểm thử, linh hoạt)
  Mô hình Tên miền: Partial DDD (các thực thể phong phú)
  Xác thực: OAuth + JWT
  Caching: Redis
  Cơ sở dữ liệu: PostgreSQL

Các Đánh đổi Được chấp nhận:
  - Modular Monolith → Một số khớp nối module (microservices chưa được biện minh)
  - Partial DDD → Không có aggregates đầy đủ (không có chuyên gia tên miền)
  - RabbitMQ sau này → Ban đầu đồng bộ (thêm khi được chứng minh là cần thiết)

Lộ trình Di chuyển:
  - Nhóm > 10 → Cân nhắc microservices
  - Xung đột tên miền → Trích xuất bounded contexts
  - Vấn đề hiệu suất đọc → Thêm CQRS
```

---

## Ví dụ 3: Doanh nghiệp (100K+ Người dùng)

```yaml
Yêu cầu:
  - 100K+ người dùng
  - 10+ nhà phát triển
  - Nhiều tên miền kinh doanh
  - Nhu cầu mở rộng khác nhau
  - Tính sẵn sàng 24/7

Các Quyết định Kiến trúc:
  Cấu trúc Ứng dụng: Microservices (mở rộng độc lập)
  API Gateway: Kong/AWS API GW
  Mô hình Tên miền: Full DDD
  Tính nhất quán: Hướng sự kiện (eventual OK)
  Message Bus: Kafka
  Xác thực: OAuth + SAML (SSO doanh nghiệp)
  Cơ sở dữ liệu: Đa ngôn ngữ/Polyglot (công cụ phù hợp cho từng công việc)
  CQRS: Các dịch vụ được chọn

Các Yêu cầu Vận hành:
  - Service mesh (Istio/Linkerd)
  - Distributed tracing (Jaeger/Tempo)
  - Ghi nhật ký tập trung (ELK/Loki)
  - Circuit breakers (Resilience4j)
  - Kubernetes/Helm
```
