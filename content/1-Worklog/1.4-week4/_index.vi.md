---
title: "Worklog Tuần 4"
date: "2026-05-05"
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

## Tuần 4: Docker Containerization & Triển khai Backend

> **Thời gian:** 05/05 - 11/05/2026  
> **Chủ đề:** Containerize & Deploy Backend

### Mục tiêu học tập

- Build và push Docker images lên Amazon ECR
- Deploy containerized backend lên EC2
- Cấu hình security groups và IAM roles
- Thiết lập API endpoints và test với Postman

---

### Kiến thức AWS đã học

#### Amazon ECR Operations

```bash
# Authenticate Docker to ECR
aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.ap-southeast-1.amazonaws.com

# Tạo ECR repository
aws ecr create-repository --repository-name ocr-api --region ap-southeast-1

# Tag và push image
docker tag ocr-api:latest <account-id>.dkr.ecr.ap-southeast-1.amazonaws.com/ocr-api:latest
docker push <account-id>.dkr.ecr.ap-southeast-1.amazonaws.com/ocr-api:latest

# Pull image trên EC2
docker pull <account-id>.dkr.ecr.ap-southeast-1.amazonaws.com/ocr-api:latest
```

#### EC2 Instance Configuration

| Setting | Value | Reason |
|---------|-------|--------|
| **Instance Type** | t3.medium | Cần đủ RAM cho OCR model (~4GB) |
| **OS** | Ubuntu 22.04 LTS | Long-term support, stable |
| **Storage** | 30GB gp3 | Fast SSD storage |
| **IAM Role** | EC2ECRReadOnly | Pull images từ ECR |

#### Security Group Configuration

| Port | Protocol | Source | Purpose |
|------|----------|--------|---------|
| 22 | TCP | My IP | SSH access |
| 8000 | TCP | ALB SG | API access |
| 443 | TCP | 0.0.0.0/0 | HTTPS (via ALB) |

---

### Công việc trong tuần

| Thứ | Công việc | Bắt đầu | Kết thúc | Tài liệu |
|-----|-----------|---------|---------|-----------|
| 2 (T3) | Build Docker image cho ocr-api | 05/05 | 05/05 | Docker Docs |
| 3 (T4) | Tạo ECR repository và push image | 06/05 | 06/05 | AWS ECR |
| 4 (T5) | Launch EC2 instance (t3.medium) | 07/05 | 07/05 | AWS EC2 |
| 5 (T6) | Cài Docker và pull image | 08/05 | 08/05 | Docker on EC2 |
| 6 (T7) | Cấu hình security groups | 09/05 | 09/05 | Security Groups |
| 7 (CN) | Test API endpoints với Postman | 10/05 | 10/05 | API Testing |

---

### Kết quả đạt được

- Successfully built and pushed Docker image to Amazon ECR
- Backend API running stably on EC2
- API endpoints `/ocr` và `/translate` hoạt động đúng
- Hiểu complete CI pipeline: Build Push ECR Pull Run

### CI/CD Pipeline Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Code Push  │────▶│ Build Image │────▶│  Push ECR   │────▶│  Pull & Run │
│  (GitHub)  │     │  (Docker)  │     │             │     │  (EC2)     │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
```

---

### Khó khăn & Giải pháp

| Khó khăn | Giải pháp |
|------------|----------|
| Docker image lớn (~2GB) do OCR model | Dùng multi-stage build để giảm size |
| t2.micro không đủ RAM cho OCR | Nâng cấp lên t3.medium |
| IAM permission errors khi pull từ ECR | Tạo IAM role với EC2ECRReadOnly policy |

### Bài học kinh nghiệm

- Luôn kiểm tra **RAM requirements** trước khi chọn instance type
- Sử dụng **multi-stage builds** để tối ưu Docker image size
- IAM roles cho EC2 nên được tạo lúc launch, không phải sau

---

### Kế hoạch tuần tới

- Deploy frontend TypeScript/React lên S3
- Cấu hình CloudFront CDN
- Kết nối frontend với backend API
- Thiết lập CORS cho cross-origin requests

---

### Tài liệu tham khảo

- [Amazon ECR Documentation](https://docs.aws.amazon.com/ecr)
- [Docker Multi-stage Builds](https://docs.docker.com/develop/develop-images/multistage-build)
- [EC2 Instance Types](https://aws.amazon.com/ec2/instance-types)
