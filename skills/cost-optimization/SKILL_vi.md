---
name: cost-optimization
description: Tối ưu hóa chi phí đám mây thông qua định cỡ tài nguyên, chiến lược gắn thẻ, các phiên bản dành riêng và phân tích chi tiêu. Sử dụng khi giảm chi phí đám mây, phân tích chi phí cơ sở hạ tầng hoặc thực hiện các chính sách quản trị chi phí.
---

# Tối Ưu Hóa Chi Phí Đám Mây (Cloud Cost Optimization)

Các chiến lược và mẫu để tối ưu hóa chi phí đám mây trên AWS, Azure và GCP.

## Không sử dụng kỹ năng này khi

- Nhiệm vụ không liên quan đến tối ưu hóa chi phí đám mây
- Bạn cần một miền hoặc công cụ khác ngoài phạm vi này

## Hướng dẫn

- Làm rõ mục tiêu, ràng buộc và đầu vào cần thiết.
- Áp dụng các thực tiễn tốt nhất có liên quan và xác thực kết quả.
- Cung cấp các bước có thể hành động và xác minh.
- Nếu cần các ví dụ chi tiết, hãy mở `resources/implementation-playbook.md`.

## Mục đích

Triển khai các chiến lược tối ưu hóa chi phí có hệ thống để giảm chi tiêu đám mây trong khi vẫn duy trì hiệu suất và độ tin cậy.

## Sử dụng kỹ năng này khi

- Giảm chi tiêu đám mây
- Định cỡ tài nguyên phù hợp (Right-size)
- Thực hiện quản trị chi phí
- Tối ưu hóa chi phí đa đám mây
- Đáp ứng các ràng buộc ngân sách

## Khung Tối ưu hóa Chi phí

### 1. Tầm nhìn (Visibility)

- Triển khai gắn thẻ phân bổ chi phí
- Sử dụng các công cụ quản lý chi phí đám mây
- Thiết lập cảnh báo ngân sách
- Tạo bảng điều khiển chi phí

### 2. Định cỡ phù hợp (Right-Sizing)

- Phân tích mức sử dụng tài nguyên
- Giảm kích thước tài nguyên được cung cấp quá mức
- Sử dụng tự động mở rộng (auto-scaling)
- Loại bỏ tài nguyên nhàn rỗi

### 3. Mô hình Định giá

- Sử dụng dung lượng dành riêng (reserved capacity)
- Tận dụng các phiên bản spot/preemptible
- Triển khai các kế hoạch tiết kiệm (savings plans)
- Sử dụng giảm giá cam kết sử dụng

### 4. Tối ưu hóa Kiến trúc

- Sử dụng các dịch vụ được quản lý
- Triển khai bộ nhớ đệm
- Tối ưu hóa chuyển dữ liệu
- Sử dụng các chính sách vòng đời

## Tối ưu hóa Chi phí AWS

### Các Phiên bản Dành riêng (Reserved Instances)

```
Tiết kiệm: 30-72% so với Theo nhu cầu (On-Demand)
Thời hạn: 1 hoặc 3 năm
Thanh toán: Tất cả/Một phần/Không trả trước
Tính linh hoạt: Tiêu chuẩn hoặc Có thể chuyển đổi
```

### Kế hoạch Tiết kiệm (Savings Plans)

```
Compute Savings Plans: Tiết kiệm 66%
EC2 Instance Savings Plans: Tiết kiệm 72%
Áp dụng cho: EC2, Fargate, Lambda
Linh hoạt trên: Các họ phiên bản, khu vực, hệ điều hành
```

### Các Phiên bản Spot (Spot Instances)

```
Tiết kiệm: Lên đến 90% so với Theo nhu cầu
Tốt nhất cho: Các công việc hàng loạt, CI/CD, khối lượng công việc phi trạng thái
Rủi ro: Thông báo gián đoạn 2 phút
Chiến lược: Kết hợp với Theo nhu cầu để tăng khả năng phục hồi
```

### Tối ưu hóa Chi phí S3

```hcl
resource "aws_s3_bucket_lifecycle_configuration" "example" {
  bucket = aws_s3_bucket.example.id

  rule {
    id     = "transition-to-ia"
    status = "Enabled"

    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }

    transition {
      days          = 90
      storage_class = "GLACIER"
    }

    expiration {
      days = 365
    }
  }
}
```

## Tối ưu hóa Chi phí Azure

### Các Phiên bản VM Dành riêng

- Thời hạn 1 hoặc 3 năm
- Tiết kiệm lên đến 72%
- Định cỡ linh hoạt
- Có thể trao đổi

### Lợi ích Lai Azure (Azure Hybrid Benefit)

- Sử dụng giấy phép Windows Server hiện có
- Tiết kiệm lên đến 80% với RI
- Có sẵn cho Windows và SQL Server

### Các Đề xuất của Azure Advisor

- Định cỡ VM phù hợp
- Xóa các tài nguyên không sử dụng
- Sử dụng dung lượng dành riêng
- Tối ưu hóa lưu trữ

## Tối ưu hóa Chi phí GCP

### Giảm giá Cam kết Sử dụng (Committed Use Discounts)

- Cam kết 1 hoặc 3 năm
- Tiết kiệm lên đến 57%
- Áp dụng cho vCPUs và bộ nhớ
- Dựa trên tài nguyên hoặc dựa trên chi tiêu

### Giảm giá Sử dụng Bền vững (Sustained Use Discounts)

- Giảm giá tự động
- Lên đến 30% cho các phiên bản đang chạy
- Không yêu cầu cam kết
- Áp dụng cho Compute Engine, GKE

### Preemptible VMs

- Tiết kiệm lên đến 80%
- Thời gian chạy tối đa 24 giờ
- Tốt nhất cho khối lượng công việc hàng loạt

## Chiến lược Gắn thẻ

### Gắn thẻ AWS

```hcl
locals {
  common_tags = {
    Environment = "production"
    Project     = "my-project"
    CostCenter  = "engineering"
    Owner       = "team@example.com"
    ManagedBy   = "terraform"
  }
}

resource "aws_instance" "example" {
  ami           = "ami-12345678"
  instance_type = "t3.medium"

  tags = merge(
    local.common_tags,
    {
      Name = "web-server"
    }
  )
}
```

**Tham khảo:** Xem `references/tagging-standards.md`

## Giám sát Chi phí

### Cảnh báo Ngân sách

```hcl
# AWS Budget
resource "aws_budgets_budget" "monthly" {
  name              = "monthly-budget"
  budget_type       = "COST"
  limit_amount      = "1000"
  limit_unit        = "USD"
  time_period_start = "2024-01-01_00:00"
  time_unit         = "MONTHLY"

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 80
    threshold_type            = "PERCENTAGE"
    notification_type         = "ACTUAL"
    subscriber_email_addresses = ["team@example.com"]
  }
}
```

### Phát hiện Bất thường Chi phí

- AWS Cost Anomaly Detection
- Azure Cost Management alerts
- GCP Budget alerts

## Các Mẫu Kiến trúc

### Mẫu 1: Serverless Trước tiên

- Sử dụng Lambda/Functions cho sự kiện
- Chỉ trả tiền cho thời gian thực thi
- Tự động mở rộng được bao gồm
- Không có chi phí nhàn rỗi

### Mẫu 2: Cơ sở dữ liệu Định cỡ Phù hợp

```
Development: t3.small RDS
Staging: t3.large RDS
Production: r6g.2xlarge RDS with read replicas
```

### Mẫu 3: Lưu trữ Đa tầng

```
Hot data: S3 Standard
Warm data: S3 Standard-IA (30 days)
Cold data: S3 Glacier (90 days)
Archive: S3 Deep Archive (365 days)
```

### Mẫu 4: Tự động Mở rộng

```hcl
resource "aws_autoscaling_policy" "scale_up" {
  name                   = "scale-up"
  scaling_adjustment     = 2
  adjustment_type        = "ChangeInCapacity"
  cooldown              = 300
  autoscaling_group_name = aws_autoscaling_group.main.name
}

resource "aws_cloudwatch_metric_alarm" "cpu_high" {
  alarm_name          = "cpu-high"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = "60"
  statistic           = "Average"
  threshold           = "80"
  alarm_actions       = [aws_autoscaling_policy.scale_up.arn]
}
```

## Danh sách Kiểm tra Tối ưu hóa Chi phí

- [ ] Triển khai gắn thẻ phân bổ chi phí
- [ ] Xóa các tài nguyên không sử dụng (EBS, EIPs, snapshots)
- [ ] Định cỡ các phiên bản dựa trên mức sử dụng
- [ ] Sử dụng dung lượng dành riêng cho khối lượng công việc ổn định
- [ ] Triển khai tự động mở rộng
- [ ] Tối ưu hóa các lớp lưu trữ
- [ ] Sử dụng các chính sách vòng đời
- [ ] Bật phát hiện bất thường chi phí
- [ ] Thiết lập cảnh báo ngân sách
- [ ] Xem xét chi phí hàng tuần
- [ ] Sử dụng các phiên bản spot/preemptible
- [ ] Tối ưu hóa chi phí chuyển dữ liệu
- [ ] Triển khai các lớp bộ nhớ đệm
- [ ] Sử dụng các dịch vụ được quản lý
- [ ] Giám sát và tối ưu hóa liên tục

## Công cụ

- **AWS:** Cost Explorer, Cost Anomaly Detection, Compute Optimizer
- **Azure:** Cost Management, Advisor
- **GCP:** Cost Management, Recommender
- **Multi-cloud:** CloudHealth, Cloudability, Kubecost

## Tệp Tham khảo

- `references/tagging-standards.md` - Quy ước gắn thẻ
- `assets/cost-analysis-template.xlsx` - Bảng tính phân tích chi phí

## Các Kỹ năng Liên quan

- `terraform-module-library` - Để cung cấp tài nguyên
- `multi-cloud-architecture` - Để lựa chọn đám mây
