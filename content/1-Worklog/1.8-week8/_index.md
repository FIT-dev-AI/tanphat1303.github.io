---
title: "Week 8 Worklog"
date: "2026-06-02"
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

## Week 8: CI/CD Pipeline with GitHub Actions

> **Duration:** June 2 - 8, 2026  
> **Theme:** Automation & DevOps Best Practices

### Learning Objectives

- Build complete CI/CD pipeline with GitHub Actions
- Automate build, test, and deployment processes
- Configure IAM roles with least privilege principle
- Set up GitHub Secrets for secure credential management

---

### Key AWS Knowledge Acquired

#### GitHub Actions Fundamentals

| Component | Description |
|-----------|-------------|
| **Workflow** | Automated process defined in YAML |
| **Job** | Set of steps executed on a runner |
| **Step** | Individual task (action or shell command) |
| **Action** | Reusable unit of code |
| **Runner** | Server that runs workflows |

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

#### IAM Policy for GitHub Actions (Least Privilege)

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

### Weekly Task List

| Day | Task | Start | End | Resources |
|-----|------|-------|-----|-----------|
| Mon | GitHub Actions fundamentals | 06/02 | 06/02 | GitHub Actions |
| Tue | Create backend workflow | 06/03 | 06/03 | CI/CD |
| Wed | Create frontend workflow | 06/04 | 06/04 | CI/CD |
| Thu | Configure IAM for GitHub Actions | 06/05 | 06/05 | AWS IAM |
| Fri | Set up GitHub Secrets | 06/06 | 06/06 | GitHub Secrets |
| Sat | Test complete pipeline | 06/07 | 06/08 | CI/CD Testing |

---

### Achievements

- Fully automated CI/CD pipeline working end-to-end
- Deployment time reduced from ~30 min manual to ~8 min automated
- Mastered IAM least privilege principle
- Understood GitHub Actions integration with AWS

### Performance Improvement

| Metric | Before (Manual) | After (CI/CD) |
|--------|-----------------|---------------|
| Deployment Time | ~30 minutes | ~8 minutes |
| Human Errors | High | Minimal |
| Rollback Speed | Slow | Fast |

---

### Difficulties & Solutions

| Difficulty | Solution |
|------------|----------|
| YAML syntax errors | Used marketplace actions instead of writing from scratch |
| IAM permissions insufficient | Checked CloudTrail for exact permissions needed |
| CodeDeploy not recognizing EC2 | Added correct tags to EC2 instance |

### Best Practices

- Use **GitHub Actions Marketplace** actions
- Always use **secrets** for sensitive data
- Apply **least privilege** for IAM roles
- Use **environment-specific** configurations

---

### Next Week's Plan

- Set up **Amazon CloudWatch** for monitoring
- Create **CloudWatch Alarms** for notifications
- Configure **CloudWatch Logs** for application logging

---

### Reference Materials

- [GitHub Actions Documentation](https://docs.github.com/actions)
- [AWS Actions for GitHub](https://github.com/aws-actions)
- [CI/CD Best Practices](https://aws.amazon.com/solutions/devops-tools/cicd)
