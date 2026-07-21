---
title: "Week 4 Worklog"
date: "2026-05-05"
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

## Week 4: Docker Containerization & Backend Deployment

> **Duration:** May 5 - 11, 2026  
> **Theme:** Containerizing & Deploying the Backend

### 📚 Learning Objectives

- Build and push Docker images to Amazon ECR
- Deploy containerized backend to EC2
- Configure security groups and IAM roles
- Set up API endpoints and test with Postman

---

### 🎯 Key AWS Knowledge Acquired

#### Amazon ECR Operations

```bash
# Authenticate Docker to ECR
aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.ap-southeast-1.amazonaws.com

# Create ECR repository
aws ecr create-repository --repository-name ocr-api --region ap-southeast-1

# Tag and push image
docker tag ocr-api:latest <account-id>.dkr.ecr.ap-southeast-1.amazonaws.com/ocr-api:latest
docker push <account-id>.dkr.ecr.ap-southeast-1.amazonaws.com/ocr-api:latest

# Pull image on EC2
docker pull <account-id>.dkr.ecr.ap-southeast-1.amazonaws.com/ocr-api:latest
```

#### EC2 Instance Configuration

| Setting | Value | Reason |
|---------|-------|--------|
| **Instance Type** | t3.medium | Need sufficient RAM for OCR model (~4GB) |
| **OS** | Ubuntu 22.04 LTS | Long-term support, stable |
| **Storage** | 30GB gp3 | Fast SSD storage |
| **IAM Role** | EC2ECRReadOnly | Pull images from ECR |

#### Security Group Configuration

| Port | Protocol | Source | Purpose |
|------|----------|--------|---------|
| 22 | TCP | My IP | SSH access |
| 8000 | TCP | ALB SG | API access |
| 443 | TCP | 0.0.0.0/0 | HTTPS (via ALB) |

---

### 📋 Weekly Task List

| Day | Task | Start | End | Resources |
|-----|------|-------|-----|-----------|
| Mon | Build Docker image for ocr-api | 05/05 | 05/05 | Docker Docs |
| Tue | Create ECR repository and push image | 05/06 | 05/06 | AWS ECR |
| Wed | Launch EC2 instance (t3.medium) | 05/07 | 05/07 | AWS EC2 |
| Thu | Install Docker and pull image | 05/08 | 05/08 | Docker on EC2 |
| Fri | Configure security groups | 05/09 | 05/09 | Security Groups |
| Sat | Test API endpoints with Postman | 05/10 | 05/10 | API Testing |

---

### ✅ Achievements

- Successfully built and pushed Docker image to Amazon ECR
- Backend API running stably on EC2
- API endpoints `/ocr` and `/translate` working correctly
- Understood complete CI pipeline: Build → Push ECR → Pull → Run

### 🔧 CI/CD Pipeline Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Code Push  │────▶│ Build Image │────▶│  Push ECR   │────▶│  Pull & Run │
│  (GitHub)  │     │  (Docker)  │     │             │     │  (EC2)     │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
```

---

### 🚧 Difficulties & Solutions

| Difficulty | Solution |
|------------|----------|
| Large Docker image (~2GB) due to OCR model | Used multi-stage build to reduce size |
| t2.micro insufficient RAM for OCR | Upgraded to t3.medium |
| IAM permission errors pulling from ECR | Created IAM role with EC2ECRReadOnly policy |

### 💡 Lessons Learned

- Always check **RAM requirements** before selecting instance type
- Use **multi-stage builds** to optimize Docker image size
- IAM roles for EC2 should be created at instance launch, not after

---

### 📅 Next Week's Plan

- Deploy frontend TypeScript/React to S3
- Configure CloudFront CDN
- Connect frontend with backend API
- Set up CORS for cross-origin requests

---

### 📖 Reference Materials

- [Amazon ECR Documentation](https://docs.aws.amazon.com/ecr)
- [Docker Multi-stage Builds](https://docs.docker.com/develop/develop-images/multistage-build)
- [EC2 Instance Types](https://aws.amazon.com/ec2/instance-types)
