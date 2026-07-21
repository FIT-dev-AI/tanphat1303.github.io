---
title: "Week 3 Worklog"
date: "2026-04-28"
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

## Week 3: Project Analysis & AWS Architecture Design

> **Duration:** April 28 - May 4, 2026  
> **Theme:** OCR-CapCut Project Planning

### 📚 Learning Objectives

- Analyze OCR-CapCut project architecture (Frontend + Backend)
- Understand Docker containerization and its benefits
- Design AWS deployment architecture for the project
- Learn about Amazon ECR for container image storage

---

### 🎯 Project: OCR-CapCut Architecture

#### System Overview

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│   User Upload   │────▶│  Frontend (S3 +  │────▶│  Backend API    │
│   Image/Video   │     │   CloudFront)    │     │  (EC2 + Docker) │
└─────────────────┘     └──────────────────┘     └────────┬────────┘
                                                          │
                                                          ▼
                                                ┌─────────────────┐
                                                │  OCR + Translate │
                                                │  (Python/FastAPI)│
                                                └─────────────────┘
```

#### Component Analysis

| Component | Technology | Purpose |
|-----------|------------|---------|
| **Frontend** | TypeScript + React | User interface for image/video upload |
| **Backend API** | Python + FastAPI | Handle OCR requests and translation |
| **OCR Engine** | EasyOCR / Tesseract | Text extraction from images |
| **Translation** | Google Translate API | Multi-language translation |
| **Database** | PostgreSQL (RDS) | Store user history |

---

### 🎯 Key AWS Knowledge Acquired

#### Amazon ECR (Elastic Container Registry)

| Feature | Description |
|---------|-------------|
| **Fully Managed** | AWS handles scaling, availability, and security |
| **Image Scanning** | Scan images for vulnerabilities on push |
| **Lifecycle Policies** | Automatically clean up old images |
| **Immutable Tags** | Prevent overwriting images with same tag |

#### Docker Fundamentals for AWS

```dockerfile
# Multi-stage Dockerfile for Python FastAPI backend
FROM python:3.11-slim as builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages
COPY --from=builder /usr/local/bin /usr/local/bin
COPY . .
EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

#### AWS Deployment Architecture Design

```
                            ┌──────────────────┐
                            │   Amazon S3       │
                            │  (Static Assets)  │
                            └────────┬─────────┘
                                     │
                            ┌────────▼─────────┐
                            │  CloudFront CDN  │
                            └────────┬─────────┘
                                     │
                        ┌────────────▼────────────┐
                        │   Application Load       │
                        │   Balancer (ALB)         │
                        └────────────┬────────────┘
                                     │
                        ┌────────────▼────────────┐
                        │   VPC (Private Subnet)   │
                        │   ┌──────────────────┐  │
                        │   │  EC2 Instances   │  │
                        │   │  (Docker)       │  │
                        │   └──────────────────┘  │
                        └─────────────────────────┘
```

---

### 📋 Weekly Task List

| Day | Task | Start | End | Resources |
|-----|------|-------|-----|-----------|
| Mon | Project Analysis: Read and understand OCR-CapCut source code | 04/28 | 04/28 | Project Docs |
| Tue | Component Identification: Frontend, Backend, Database | 04/29 | 04/29 | Architecture Review |
| Wed | Docker Fundamentals: Study existing Dockerfiles | 04/30 | 04/30 | Docker Docs |
| Thu | ECR Research: Learn about Amazon ECR for image storage | 05/01 | 05/01 | AWS ECR Docs |
| Fri | Architecture Design: Design AWS deployment diagram | 05/02 | 05/02 | Mentor Guidance |

---

### ✅ Achievements

- Understood complete workflow: User Upload → OCR → Translation → Result Display
- Created detailed AWS architecture diagram for OCR-CapCut
- Identified all required AWS services: EC2, S3, ECR, ALB, IAM, VPC
- Received mentor approval for deployment plan

### 📊 AWS Services Utilized

| Service | Purpose | Monthly Cost (Estimate) |
|---------|---------|------------------------|
| EC2 (t3.medium) | Backend server | ~$30 |
| S3 | Static website hosting | ~$5 |
| CloudFront | CDN distribution | ~$10 |
| ECR | Container registry | Free |
| RDS (db.t3.micro) | Database | ~$15 |
| ALB | Load balancing | ~$20 |

---

### 🚧 Difficulties & Solutions

| Difficulty | Solution |
|------------|----------|
| Large codebase with multiple libraries | Ran locally first to understand flow |
| Understanding Docker images | Studied Dockerfile line by line |
| Choosing between architecture options | Referenced AWS AI/ML reference architectures |

---

### 📅 Next Week's Plan

- Build Docker image for ocr-api backend
- Push image to Amazon ECR
- Create EC2 instance and deploy backend
- Configure security groups and networking

---

### 📖 Reference Materials

- [Amazon ECR Documentation](https://docs.aws.amazon.com/ecr)
- [Docker Documentation](https://docs.docker.com)
- [AWS AI Services Reference Architectures](https://aws.amazon.com/solutions/reference-architectures)
