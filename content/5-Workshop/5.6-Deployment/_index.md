---
title: "Deployment"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

# Workshop 5.6: Deployment and Operations

---

## Learning Objectives

By the end of this module, you will:

- Deploy the backend to AWS services
- Configure Docker for production
- Set up CI/CD pipelines
- Implement monitoring and logging
- Configure security best practices
- Understand cost optimization strategies

---

## Prerequisites

- Workshops 5.1-5.5 completed
- AWS account with appropriate permissions
- Domain name (optional for production)

---

## Architecture Context

Production deployment leverages AWS managed services:

![AWS Architecture](/images/5-Workshop/5.6-Deployment/AWS.png)

---

## Step 1: Docker Configuration

### Backend Dockerfile

```dockerfile
# ocr-api/Dockerfile
FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    ffmpeg \
    libsm6 \
    libxext6 \
    libgl1-mesa-glx \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements first for better caching
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Preload OCR models (optional, reduces first-run time)
RUN python preload_ocr_model.py || true

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/api/v1/health || exit 1

# Run the application
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Docker Compose for Production

```yaml
# docker-compose.yml
version: '3.8'

services:
  backend:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: video-localization-api
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
      - AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
      - REDIS_URL=redis://redis:6379/0
      - CELERY_BROKER_URL=redis://redis:6379/0
    depends_on:
      - redis
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/api/v1/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  celery-worker:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: video-localization-worker
    command: celery -A celery worker --loglevel=info
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
      - AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
      - REDIS_URL=redis://redis:6379/0
      - CELERY_BROKER_URL=redis://redis:6379/0
    depends_on:
      - redis
    restart: unless-stopped
    volumes:
      - /tmp:/tmp  # For temporary video files

  redis:
    image: redis:7-alpine
    container_name: video-localization-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    restart: unless-stopped

volumes:
  redis_data:
```

---

## Step 2: AWS ECR Repository

### Create ECR Repository

```bash
# Create ECR repository
aws ecr create-repository \
    --repository-name video-localization-api \
    --region ap-southeast-2

# Get login password
aws ecr get-login-password --region ap-southeast-2 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.ap-southeast-2.amazonaws.com
```

### Build and Push Docker Image

```bash
# Build the image
docker build -t video-localization-api:latest .

# Tag for ECR
docker tag video-localization-api:latest <account-id>.dkr.ecr.ap-southeast-2.amazonaws.com/video-localization-api:latest

# Push to ECR
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
  "containerDefinitions": [
    {
      "name": "backend",
      "image": "<account-id>.dkr.ecr.ap-southeast-2.amazonaws.com/video-localization-api:latest",
      "essential": true,
      "portMappings": [
        {
          "containerPort": 8000,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "DATABASE_URL",
          "value": "mysql+pymysql://user:pass@host:3306/db"
        },
        {
          "name": "REDIS_URL",
          "value": "redis://redis-host:6379/0"
        }
      ],
      "secrets": [
        {
          "name": "AWS_ACCESS_KEY_ID",
          "valueFrom": "arn:aws:secretsmanager:region:account:secret:key:AWSCURRENT"
        },
        {
          "name": "AWS_SECRET_ACCESS_KEY",
          "valueFrom": "arn:aws:secretsmanager:region:account:secret:key:AWSCURRENT"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/video-localization",
          "awslogs-region": "ap-southeast-2",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

### ECS Service Configuration

```bash
# Register task definition
aws ecs register-task-definition \
    --cli-input-json file://task-definition.json \
    --region ap-southeast-2

# Create ECS cluster
aws ecs create-cluster \
    --cluster-name video-localization-cluster \
    --region ap-southeast-2

# Create service
aws ecs create-service \
    --cluster video-localization-cluster \
    --service-name video-localization-api \
    --task-definition video-localization-api:1 \
    --desired-count 2 \
    --launch-type FARGATE \
    --network-configuration "awsvpcConfiguration={subnets=[subnet-xxx],securityGroups=[sg-xxx]}" \
    --region ap-southeast-2
```

---

## Step 4: AWS RDS Configuration

### Database Creation

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
    --vpc-security-group-ids sg-xxx \
    --db-subnet-group-name your-db-subnet-group \
    --region ap-southeast-2 \
    --backup-retention-period 7 \
    --multi-az
```

### Database Migration

```bash
# Run migrations via ECS task or locally
DATABASE_URL="mysql+pymysql://user:pass@host:3306/db" \
    alembic upgrade head
```

---

## Step 5: S3 Bucket Configuration

### Create Buckets

```bash
# Input video bucket
aws s3 mb s3://ocr-video-bucket-prod --region ap-southeast-2

# Output video bucket
aws s3 mb s3://video-sub-ft-prod --region ap-southeast-2

# SRT storage bucket
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

### Bucket Policies

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::video-sub-ft-prod/*"
    }
  ]
}
```

---

## Step 6: AWS Secrets Manager

### Store Secrets

```bash
# Store database password
aws secretsmanager create-secret \
    --name video-localization/db-password \
    --secret-string '{"password":"YourPassword123"}' \
    --region ap-southeast-2

# Store API keys
# Note: Amazon Translate uses the AWS credentials above and does not need a separate key.
aws secretsmanager create-secret \
    --name video-localization/api-keys \
    --secret-string '{"gemini_key":"your-key","elevenlabs_key":"your-key","gcp_translate_credentials_json":"<base64-service-account-json>"}' \
    --region ap-southeast-2

# Store AWS credentials
aws secretsmanager create-secret \
    --name video-localization/aws-credentials \
    --secret-string '{"AWS_ACCESS_KEY_ID":"xxx","AWS_SECRET_ACCESS_KEY":"xxx"}' \
    --region ap-southeast-2
```

---

## Step 7: CloudWatch Logging

### Log Group Configuration

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

# Create filter for errors
aws logs put-metric-filter \
    --log-group-name /ecs/video-localization \
    --filter-name error-filter \
    --filter-pattern "ERROR" \
    --metric-transformations " [{ \"metricName\": \"error-count\", \"metricNamespace\": \"VideoLocalization\", \"metricValue\": \"1\" }]" \
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
      
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1
      
      - name: Build Docker image
        run: |
          docker build -t video-localization-api:${{ github.sha }} .
          docker tag video-localization-api:${{ github.sha }} ${{ steps.login-ecr.outputs.registry }}/video-localization-api:${{ github.sha }}
      
      - name: Push to ECR
        run: |
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

## Step 9: Monitoring and Alerts

### CloudWatch Dashboard

Create a custom dashboard with:
- API latency
- Error rates
- CPU/Memory utilization
- Celery task success rate
- S3 operation metrics

### Budget Alerts

```bash
# Create budget alert
aws budgets create-budget \
    --account-id <account-id> \
    --budget '{"BudgetName":"video-localization-monthly","BudgetLimit":{"Amount":"50","Unit":"USD"},"TimeUnit":"MONTHLY","BudgetType":"COST"}' \
    --notifications-with-subscribers '[{"Notification":{"NotificationType":"ACTUAL","ComparisonOperator":"GREATER_THAN","Threshold":80},"Subscribers":[{"SubscriptionType":"EMAIL","Address":"your-email@example.com"}]}]' \
    --region ap-southeast-2
```

---

## Step 10: Security Best Practices

### IAM Role for ECS Task

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": [
        "arn:aws:s3:::ocr-video-bucket-prod/*",
        "arn:aws:s3:::video-sub-ft-prod/*",
        "arn:aws:s3:::srt-input-storage-prod/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "ses:SendEmail"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue"
      ],
      "Resource": "arn:aws:secretsmanager:ap-southeast-2:<account>:secret:video-localization/*"
    }
  ]
}
```

### Security Groups

```bash
# ECS task security group
aws ec2 create-security-group \
    --group-name video-localization-ecs-sg \
    --description "Security group for ECS tasks" \
    --vpc-id vpc-xxx

# Allow only necessary ports
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxx \
    --protocol tcp \
    --port 8000 \
    --cidr 0.0.0.0/0
```

---

## Cost Optimization

### Recommendations

| Optimization | Impact | Estimated Savings |
|-------------|--------|-------------------|
| Use Fargate Spot | 50-70% | $50-100/month |
| S3 Intelligent Tiering | Medium | $10-20/month |
| RDS reserved instances | 30-40% | $20-40/month |
| CloudWatch log retention | Low | $5/month |

### Right-sizing

| Component | Development | Production |
|-----------|-------------|------------|
| ECS CPU | 1024 | 2048 |
| ECS Memory | 2048 | 4096 |
| RDS Instance | db.t3.micro | db.t3.small |
| Celery Workers | 1 | 2-4 |

---

## Validation

### Health Check

```bash
# Test API health
curl https://api.example.com/api/v1/health

# Expected response
{
  "status": "healthy",
  "services": {
    "database": "connected",
    "redis": "connected",
    "s3": "connected"
  }
}
```

### End-to-End Test

```bash
# 1. Upload a test video
curl -X POST https://api.example.com/api/v1/videos/upload \
  -H "Authorization: Bearer <token>" \
  -F "file=@test_video.mp4"

# 2. Monitor progress
# Watch CloudWatch logs or WebSocket

# 3. Verify completion
curl https://api.example.com/api/v1/videos/<video_id> \
  -H "Authorization: Bearer <token>"
```

---

## Summary

In this module, you learned:

- **Docker Configuration**: Production-ready Dockerfile and compose
- **AWS ECR**: Container registry setup
- **ECS/Fargate**: Serverless container deployment
- **RDS**: Managed database configuration
- **S3**: Storage with security policies
- **Secrets Manager**: Secure credential management
- **CloudWatch**: Logging and monitoring
- **CI/CD**: Automated deployment pipeline
- **Security**: IAM roles and security groups
- **Cost Optimization**: Strategies to reduce costs

---

## Cleanup

To avoid ongoing charges, clean up resources:

```bash
# Delete ECS service
aws ecs delete-service --cluster video-localization-cluster --service video-localization-api --force

# Delete RDS instance
aws rds delete-db-instance --db-instance-identifier video-localization-db --skip-final-snapshot

# Delete S3 buckets
aws s3 rb s3://ocr-video-bucket-prod --force
aws s3 rb s3://video-sub-ft-prod --force
aws s3 rb s3://srt-input-storage-prod --force

# Delete CloudWatch log group
aws logs delete-log-group --log-group-name /ecs/video-localization

# Delete ECR repository
aws ecr delete-repository --repository-name video-localization-api --force
```

---

## References

- [AWS ECS Documentation](https://docs.aws.amazon.com/ecs/)
- [AWS Fargate](https://docs.aws.amazon.com/AmazonECS/latest/userguide/what-is-fargate.html)
- [AWS RDS MySQL](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_MySQL.html)
- [Docker Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
