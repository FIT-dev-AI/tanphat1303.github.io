---
title: "Worklog Tuần 8"
date: "2026-06-02"
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

## Tuần 8: CI/CD Pipeline với GitHub Actions

> **Thời gian:** 02/06 - 08/06/2026  
> **Chủ đề:** Automation & DevOps Best Practices

### 📚 Mục tiêu học tập

- Xây dựng complete CI/CD pipeline với GitHub Actions
- Tự động hóa quá trình build, test, và deployment
- Cấu hình IAM roles với nguyên tắc least privilege
- Thiết lập GitHub Secrets cho quản lý credentials bảo mật

---

### 🎯 Kiến thức AWS đã học

#### GitHub Actions Fundamentals

| Thành phần | Mô tả |
|-----------|-------------|
| **Workflow** | Quy trình tự động được định nghĩa trong YAML |
| **Job** | Tập hợp các steps thực thi trên runner |
| **Step** | Task riêng lẻ (action hoặc shell command) |
| **Action** | Đơn vị code có thể tái sử dụng |
| **Runner** | Server chạy workflows |

#### CI/CD Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         GitHub Repository                        │
└─────────────────────────────────────────────────────────────────┘
                                │
                                │ Push Code
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                      GitHub Actions CI/CD                        │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    │
│  │   Lint/Test  │───▶│  Build/Push │───▶│   Deploy     │    │
│  └──────────────┘    └──────────────┘    └──────────────┘    │
└─────────────────────────────────────────────────────────────────┘
         │                      │                      │
         ▼                      ▼                      ▼
   Code Quality           Docker Image            AWS EC2/ECS
```

#### Backend Workflow (GitHub Actions)

```yaml
name: Backend CI/CD

on:
  push:
    branches: [main]
    paths:
      - 'backend/**'

jobs:
  backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-southeast-1
      
      - name: Login to ECR
        uses: aws-actions/amazon-ecr-login@v2
        
      - name: Build and push Docker image
        run: |
          docker build -t ocr-api:${{ github.sha }} backend/
          docker tag ocr-api:${{ github.sha }} ${{ env.ECR_REGISTRY }}/ocr-api:latest
          docker push ${{ env.ECR_REGISTRY }}/ocr-api:latest
      
      - name: Deploy to EC2
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ubuntu
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            docker pull ${{ env.ECR_REGISTRY }}/ocr-api:latest
            docker stop ocr-api || true
            docker run -d --name ocr-api \
              --restart unless-stopped \
              -p 8000:8000 \
              ${{ env.ECR_REGISTRY }}/ocr-api:latest
```

#### Frontend Workflow

```yaml
name: Frontend CI/CD

on:
  push:
    branches: [main]
    paths:
      - 'frontend/**'

jobs:
  frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Build frontend
        run: |
          cd frontend
          npm ci
          npm run build
        env:
          REACT_APP_API_URL: ${{ secrets.API_URL }}
      
      - name: Deploy to S3
        uses: jakejarvis/s3-sync-action@master
        with:
          args: '--delete'
        env:
          AWS_S3_BUCKET: ${{ secrets.AWS_S3_BUCKET }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      
      - name: Invalidate CloudFront
        uses: churtz/cloudfront-invalidate-action@v1
        with:
          distribution: ${{ secrets.CF_DISTRIBUTION_ID }}
          pattern: '/*'
```

#### IAM Policy cho GitHub Actions (Least Privilege)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ecr:PutImage"
      ],
      "Resource": "arn:aws:ecr:ap-southeast-1:123456789:repository/ocr-api"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::my-frontend-bucket/*"
    }
  ]
}
```

---

### 📋 Công việc trong tuần

| Thứ | Công việc | Bắt đầu | Kết thúc | Tài liệu |
|-----|-----------|---------|---------|-----------|
| 2 (T3) | GitHub Actions fundamentals | 02/06 | 02/06 | GitHub Actions |
| 3 (T4) | Tạo backend workflow | 03/06 | 03/06 | CI/CD |
| 4 (T5) | Tạo frontend workflow | 04/06 | 04/06 | CI/CD |
| 5 (T6) | Cấu hình IAM cho GitHub Actions | 05/06 | 05/06 | AWS IAM |
| 6 (T7) | Thiết lập GitHub Secrets | 06/06 | 06/06 | GitHub Secrets |
| 7 (CN) | Test complete pipeline | 07/06 | 08/06 | CI/CD Testing |

---

### ✅ Kết quả đạt được

- Fully automated CI/CD pipeline hoạt động end-to-end
- Thời gian deployment giảm từ ~30 phút manual xuống ~8 phút automated
- Nắm vững nguyên tắc IAM least privilege
- Hiểu GitHub Actions integration với AWS

### ⚡ Cải thiện hiệu suất

| Metric | Trước (Manual) | Sau (CI/CD) |
|--------|-----------------|---------------|
| Deployment Time | ~30 phút | ~8 phút |
| Human Errors | Cao | Tối thiểu |
| Rollback Speed | Chậm | Nhanh |

---

### 🚧 Khó khăn & Giải pháp

| Khó khăn | Giải pháp |
|------------|----------|
| YAML syntax errors | Dùng marketplace actions thay vì viết từ đầu |
| IAM permissions không đủ | Kiểm tra CloudTrail để biết permissions cần thiết |
| CodeDeploy không nhận EC2 | Thêm đúng tags cho EC2 instance |

### 💡 Best Practices

- Sử dụng **GitHub Actions Marketplace** actions
- Luôn dùng **secrets** cho dữ liệu nhạy cảm
- Áp dụng **least privilege** cho IAM roles
- Sử dụng **environment-specific** configurations

---

### 📅 Kế hoạch tuần tới

- Thiết lập **Amazon CloudWatch** để monitoring
- Tạo **CloudWatch Alarms** cho notifications
- Cấu hình **CloudWatch Logs** cho application logging

---

### 📖 Tài liệu tham khảo

- [GitHub Actions Documentation](https://docs.github.com/actions)
- [AWS Actions for GitHub](https://github.com/aws-actions)
- [CI/CD Best Practices](https://aws.amazon.com/solutions/devops-tools/cicd)
