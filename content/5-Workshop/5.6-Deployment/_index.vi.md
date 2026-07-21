---
title: "Deployment"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

# Workshop 5.6: Deployment và Operations

---

## Mục tiêu học tập

Sau khi hoàn thành module này, bạn sẽ:

- Deploy backend lên AWS services
- Configure Docker cho production
- Set up CI/CD pipelines
- Implement monitoring và logging
- Configure security best practices
- Understand cost optimization strategies

---

## Điều kiện tiên quyết

- Hoàn thành Workshops 5.1-5.5
- AWS account với appropriate permissions
- Domain name (tùy chọn cho production)

---

## Kiến trúc Production

![AWS Architecture](/images/5-Workshop/5.6-Deployment/AWS.png)

---

## Step 1: Docker Configuration

### Backend Dockerfile

```dockerfile
# ocr-api/Dockerfile
FROM python:3.11-slim

WORKDIR /app

RUN apt-get update && apt-get install -y \
    ffmpeg libsm6 libxext6 libgl1-mesa-glx \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .
RUN python preload_ocr_model.py || true

EXPOSE 8000
HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/api/v1/health || exit 1

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## Step 2: AWS ECR Repository

```bash
# Create ECR repository
aws ecr create-repository \
    --repository-name video-localization-api \
    --region ap-southeast-2

# Login to ECR
aws ecr get-login-password --region ap-southeast-2 | \
    docker login --username AWS --password-stdin <account-id>.dkr.ecr.ap-southeast-2.amazonaws.com

# Build và push
docker build -t video-localization-api:latest .
docker tag video-localization-api:latest <account-id>.dkr.ecr.ap-southeast-2.amazonaws.com/video-localization-api:latest
docker push <account-id>.dkr.ecr.ap-southeast-2.amazonaws.com/video-localization-api:latest
```

---

## Step 3: AWS ECS/Fargate Deployment

### Task Definition

```json
{
  "family": "video-localization-api",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "2048",
  "memory": "4096",
  "containerDefinitions": [{
    "name": "backend",
    "image": "<account-id>.dkr.ecr.ap-southeast-2.amazonaws.com/video-localization-api:latest",
    "essential": true,
    "portMappings": [{
      "containerPort": 8000,
      "protocol": "tcp"
    }],
    "logConfiguration": {
      "logDriver": "awslogs",
      "options": {
        "awslogs-group": "/ecs/video-localization",
        "awslogs-region": "ap-southeast-2",
        "awslogs-stream-prefix": "ecs"
      }
    }
  }]
}
```

---

## Step 4: AWS RDS Configuration

```bash
# Create RDS MySQL instance
aws rds create-db-instance \
    --db-instance-identifier video-localization-db \
    --db-instance-class db.t3.micro \
    --engine mysql \
    --engine-version 8.0 \
    --allocated-storage 20 \
    --master-username admin \
    --master-user-password YourPassword123 \
    --backup-retention-period 7 \
    --multi-az
```

---

## Step 5: S3 Bucket Configuration

```bash
# Create buckets
aws s3 mb s3://ocr-video-bucket-prod --region ap-southeast-2
aws s3 mb s3://video-sub-ft-prod --region ap-southeast-2
aws s3 mb s3://srt-input-storage-prod --region ap-southeast-2

# Enable versioning
aws s3api put-bucket-versioning \
    --bucket ocr-video-bucket-prod \
    --versioning-configuration Status=Enabled

# Enable encryption
aws s3api put-bucket-encryption \
    --bucket ocr-video-bucket-prod \
    --server-side-encryption-configuration '{"Rules":[{"ApplyServerSideEncryptionByDefault":{"SSEAlgorithm":"AES256"}}]}'
```

---

## Step 6: AWS Secrets Manager

```bash
# Store database password
aws secretsmanager create-secret \
    --name video-localization/db-password \
    --secret-string '{"password":"YourPassword123"}' \
    --region ap-southeast-2

# Store API keys
# Lưu ý: Amazon Translate dùng chung AWS credentials ở trên, không cần key riêng.
aws secretsmanager create-secret \
    --name video-localization/api-keys \
    --secret-string '{"gemini_key":"xxx","elevenlabs_key":"xxx","gcp_translate_credentials_json":"<base64-service-account-json>"}' \
    --region ap-southeast-2
```

---

## Step 7: CloudWatch Logging

```bash
# Create log group
aws logs create-log-group \
    --log-group-name /ecs/video-localization \
    --region ap-southeast-2

# Set retention policy
aws logs put-retention-policy \
    --log-group-name /ecs/video-localization \
    --retention-in-days 30 \
    --region ap-southeast-2
```

---

## Step 8: CI/CD Pipeline

### GitHub Actions Workflow

```yaml
# .github/workflows/deploy.yml
name: Deploy to AWS

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-southeast-2
      
      - name: Build và Push to ECR
        run: |
          docker build -t video-localization-api:${{ github.sha }} .
          docker push ${{ steps.login-ecr.outputs.registry }}/video-localization-api:${{ github.sha }}
      
      - name: Deploy to ECS
        run: |
          aws ecs update-service \
            --cluster video-localization-cluster \
            --service video-localization-api \
            --force-new-deployment
```

### CI/CD Pipeline Flow

![CI/CD Pipeline](/images/5-Workshop/5.6-Deployment/ci_cd.png)

---

## Step 9: Cost Optimization

| Optimization | Impact | Estimated Savings |
|-------------|--------|-------------------|
| Use Fargate Spot | 50-70% | $50-100/tháng |
| S3 Intelligent Tiering | Medium | $10-20/tháng |
| RDS reserved instances | 30-40% | $20-40/tháng |

---

## Tóm tắt

Trong module này, bạn đã học:

- **Docker Configuration**: Production-ready Dockerfile và compose
- **AWS ECR**: Container registry setup
- **ECS/Fargate**: Serverless container deployment
- **RDS**: Managed database configuration
- **S3**: Storage với security policies
- **Secrets Manager**: Secure credential management
- **CloudWatch**: Logging và monitoring
- **CI/CD**: Automated deployment pipeline
- **Security**: IAM roles và security groups
- **Cost Optimization**: Strategies để reduce costs

---

## Cleanup

```bash
# Delete ECS service
aws ecs delete-service --cluster video-localization-cluster --service video-localization-api --force

# Delete RDS instance
aws rds delete-db-instance --db-instance-identifier video-localization-db --skip-final-snapshot

# Delete S3 buckets
aws s3 rb s3://ocr-video-bucket-prod --force
aws s3 rb s3://video-sub-ft-prod --force
aws s3 rb s3://srt-input-storage-prod --force
```

---

## Tham khảo

- [AWS ECS Documentation](https://docs.aws.amazon.com/ecs/)
- [AWS Fargate](https://docs.aws.amazon.com/AmazonECS/latest/userguide/what-is-fargate.html)
- [AWS RDS MySQL](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_MySQL.html)
- [Docker Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
