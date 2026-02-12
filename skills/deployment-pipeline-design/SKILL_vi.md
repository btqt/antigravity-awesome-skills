---
name: deployment-pipeline-design
description: Thiết kế các đường ống CI/CD đa giai đoạn với các cổng phê duyệt, kiểm tra bảo mật và điều phối triển khai. Sử dụng khi kiến trúc quy trình triển khai, thiết lập phân phối liên tục hoặc thực hiện các thực tiễn GitOps.
---

# Thiết Kế Đường Ống Triển Khai (Deployment Pipeline Design)

Các mẫu kiến trúc cho các đường ống CI/CD đa giai đoạn với các cổng phê duyệt và chiến lược triển khai.

## Không sử dụng kỹ năng này khi

- Nhiệm vụ không liên quan đến thiết kế đường ống triển khai
- Bạn cần một miền hoặc công cụ khác ngoài phạm vi này

## Hướng dẫn

- Làm rõ mục tiêu, ràng buộc và đầu vào cần thiết.
- Áp dụng các thực tiễn tốt nhất có liên quan và xác thực kết quả.
- Cung cấp các bước có thể hành động và xác minh.
- Nếu cần các ví dụ chi tiết, hãy mở `resources/implementation-playbook.md`.

## Sử dụng kỹ năng này khi

- Thiết kế kiến trúc CI/CD
- Triển khai các cổng triển khai
- Cấu hình đường ống đa môi trường
- Thiết lập các thực tiễn tốt nhất về triển khai
- Thực hiện phân phối lũy tiến

## Các Giai đoạn Đường ống

### Luồng Đường ống Tiêu chuẩn

```
┌─────────┐   ┌──────┐   ┌─────────┐   ┌────────┐   ┌──────────┐
│  Build  │ → │ Test │ → │ Staging │ → │ Approve│ → │Production│
└─────────┘   └──────┘   └─────────┘   └────────┘   └──────────┘
```

### Phân tích Chi tiết Giai đoạn

1. **Source** - Kéo mã về (Code checkout)
2. **Build** - Biên dịch, đóng gói, đóng container
3. **Test** - Unit, integration, quét bảo mật
4. **Staging Deploy** - Triển khai đến môi trường staging
5. **Integration Tests** - E2E, smoke tests
6. **Approval Gate** - Yêu cầu phê duyệt thủ công
7. **Production Deploy** - Canary, blue-green, rolling
8. **Verification** - Kiểm tra sức khỏe, giám sát
9. **Rollback** - Rollback tự động khi thất bại

## Các Mẫu Cổng Phê duyệt

### Mẫu 1: Phê duyệt Thủ công

```yaml
# GitHub Actions
production-deploy:
  needs: staging-deploy
  environment:
    name: production
    url: https://app.example.com
  runs-on: ubuntu-latest
  steps:
    - name: Deploy to production
      run: |
        # Deployment commands
```

### Mẫu 2: Phê duyệt Dựa trên Thời gian

```yaml
# GitLab CI
deploy:production:
  stage: deploy
  script:
    - deploy.sh production
  environment:
    name: production
  when: delayed
  start_in: 30 minutes
  only:
    - main
```

### Mẫu 3: Đa Người phê duyệt

```yaml
# Azure Pipelines
stages:
  - stage: Production
    dependsOn: Staging
    jobs:
      - deployment: Deploy
        environment:
          name: production
          resourceType: Kubernetes
        strategy:
          runOnce:
            preDeploy:
              steps:
                - task: ManualValidation@0
                  inputs:
                    notifyUsers: "team-leads@example.com"
                    instructions: "Review staging metrics before approving"
```

**Tham khảo:** Xem `assets/approval-gate-template.yml`

## Chiến lược Triển khai

### 1. Triển khai Cuốn chiếu (Rolling Deployment)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
```

**Đặc điểm:**

- Triển khai dần dần
- Không thời gian chết
- Dễ dàng rollback
- Tốt nhất cho hầu hết các ứng dụng

### 2. Triển khai Xanh-Lục (Blue-Green Deployment)

```yaml
# Blue (current)
kubectl apply -f blue-deployment.yaml
kubectl label service my-app version=blue

# Green (new)
kubectl apply -f green-deployment.yaml
# Test green environment
kubectl label service my-app version=green

# Rollback if needed
kubectl label service my-app version=blue
```

**Đặc điểm:**

- Chuyển đổi tức thì
- Dễ dàng rollback
- Tạm thời gấp đôi chi phí cơ sở hạ tầng
- Tốt cho các triển khai rủi ro cao

### 3. Triển khai Canary

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: my-app
spec:
  replicas: 10
  strategy:
    canary:
      steps:
        - setWeight: 10
        - pause: { duration: 5m }
        - setWeight: 25
        - pause: { duration: 5m }
        - setWeight: 50
        - pause: { duration: 5m }
        - setWeight: 100
```

**Đặc điểm:**

- Chuyển hướng lưu lượng dần dần
- Giảm thiểu rủi ro
- Kiểm thử người dùng thực
- Yêu cầu service mesh hoặc tương tự

### 4. Cờ Tính năng (Feature Flags)

```python
from flagsmith import Flagsmith

flagsmith = Flagsmith(environment_key="API_KEY")

if flagsmith.has_feature("new_checkout_flow"):
    # New code path
    process_checkout_v2()
else:
    # Existing code path
    process_checkout_v1()
```

**Đặc điểm:**

- Triển khai mà không cần phát hành
- Kiểm thử A/B
- Rollback tức thì
- Kiểm soát chi tiết

## Thực tiễn Tốt nhất cho Đường ống

1. **Fail fast** - Chạy các kiểm thử nhanh trước
2. **Parallel execution** - Chạy các công việc độc lập đồng thời
3. **Caching** - Cache các phụ thuộc giữa các lần chạy
4. **Artifact management** - Lưu trữ các tạo tác xây dựng
5. **Environment parity** - Giữ các môi trường nhất quán
6. **Secrets management** - Sử dụng kho lưu trữ bí mật (Vault, v.v.)
7. **Deployment windows** - Lên lịch triển khai phù hợp
8. **Monitoring integration** - Theo dõi các chỉ số triển khai
9. **Rollback automation** - Tự động rollback khi thất bại
10. **Documentation** - Tài liệu hóa các giai đoạn đường ống

## Chiến lược Rollback

### Rollback Tự động

```yaml
deploy-and-verify:
  steps:
    - name: Deploy new version
      run: kubectl apply -f k8s/

    - name: Wait for rollout
      run: kubectl rollout status deployment/my-app

    - name: Health check
      id: health
      run: |
        for i in {1..10}; do
          if curl -sf https://app.example.com/health; then
            exit 0
          fi
          sleep 10
        done
        exit 1

    - name: Rollback on failure
      if: failure()
      run: kubectl rollout undo deployment/my-app
```

## Giám sát và Chỉ số

### Các Chỉ số Đường ống Chính

- **Tần suất Triển khai** - Mức độ thường xuyên triển khai xảy ra
- **Thời gian Dẫn** - Thời gian từ commit đến sản xuất
- **Tỷ lệ Thất bại Thay đổi** - Tỷ lệ phần trăm các triển khai thất bại
- **Thời gian Phục hồi Trung bình (MTTR)** - Thời gian để phục hồi sau thất bại
- **Tỷ lệ Thành công Đường ống** - Tỷ lệ phần trăm các lần chạy thành công
- **Thời gian Đường ống Trung bình** - Thời gian để hoàn thành đường ống

## Tệp Tham khảo

- `references/pipeline-orchestration.md` - Các mẫu đường ống phức tạp
- `assets/approval-gate-template.yml` - Mẫu quy trình phê duyệt

## Các Kỹ năng Liên quan

- `github-actions-templates` - Cho triển khai GitHub Actions
- `gitlab-ci-patterns` - Cho triển khai GitLab CI
- `secrets-management` - Cho xử lý bí mật
