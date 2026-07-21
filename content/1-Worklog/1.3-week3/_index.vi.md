---
title: "Worklog Tuần 3"
date: "2026-04-28"
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

## Tuần 3: Phân tích dự án & Thiết kế kiến trúc AWS

> **Thời gian:** 28/04 - 04/05/2026  
> **Chủ đề:** OCR-CapCut Project Planning

### Mục tiêu học tập

- Phân tích kiến trúc dự án OCR-CapCut (Frontend + Backend)
- Hiểu về Docker containerization và lợi ích của nó
- Thiết kế kiến trúc triển khai AWS cho dự án
- Tìm hiểu Amazon ECR để lưu trữ container images

---

### Dự án: Kiến trúc OCR-CapCut

#### Tổng quan hệ thống

![Tổng quan hệ thống](/images/5-Workshop/5.1-Workshop-overview/Kien-truc-he-thong-tong-quan.png)

#### Phân tích các thành phần

| Thành phần | Công nghệ | Mục đích |
|-----------|------------|---------|
| **Frontend** | TypeScript + React | Giao diện người dùng upload ảnh/video |
| **Backend API** | Python + FastAPI | Xử lý OCR requests và translation |
| **OCR Engine** | EasyOCR / Tesseract | Trích xuất text từ ảnh |
| **Translation** | Google Translate API | Dịch đa ngôn ngữ |
| **Database** | PostgreSQL (RDS) | Lưu trữ lịch sử người dùng |

---

### Kiến thức AWS đã học

#### Amazon ECR (Elastic Container Registry)

| Tính năng | Mô tả |
|---------|-------------|
| **Fully Managed** | AWS quản lý scaling, availability, và security |
| **Image Scanning** | Scan images để tìm vulnerabilities khi push |
| **Lifecycle Policies** | Tự động dọn dẹp images cũ |
| **Immutable Tags** | Ngăn overwriting images với cùng tag |

#### Docker Fundamentals cho AWS

```dockerfile
# Multi-stage Dockerfile cho Python FastAPI backend
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

#### Thiết kế Kiến trúc triển khai AWS

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

### Công việc trong tuần

| Thứ | Công việc | Bắt đầu | Kết thúc | Tài liệu |
|-----|-----------|---------|---------|-----------|
| 2 (T3) | Phân tích dự án: Đọc và hiểu source code OCR-CapCut | 28/04 | 28/04 | Project Docs |
| 3 (T4) | Xác định thành phần: Frontend, Backend, Database | 29/04 | 29/04 | Architecture Review |
| 4 (T5) | Docker Fundamentals: Nghiên cứu các Dockerfile | 30/04 | 30/04 | Docker Docs |
| 5 (T6) | Nghiên cứu ECR: Tìm hiểu Amazon ECR | 01/05 | 01/05 | AWS ECR Docs |
| 6 (T7) | Thiết kế kiến trúc: Vẽ sơ đồ triển khai AWS | 02/05 | 02/05 | Mentor Guidance |

---

### Kết quả đạt được

- Hiểu luồng hoàn chỉnh: User Upload OCR Translation Result Display
- Tạo sơ đồ kiến trúc AWS chi tiết cho OCR-CapCut
- Xác định tất cả AWS services cần thiết: EC2, S3, ECR, ALB, IAM, VPC
- Được mentor phê duyệt kế hoạch triển khai

### Các AWS Services được sử dụng

| Service | Mục đích | Chi phí ước tính/tháng |
|---------|---------|------------------------ |
| EC2 (t3.medium) | Backend server | ~$30 |
| S3 | Static website hosting | ~$5 |
| CloudFront | CDN distribution | ~$10 |
| ECR | Container registry | Miễn phí |
| RDS (db.t3.micro) | Database | ~$15 |
| ALB | Load balancing | ~$20 |

---

### Khó khăn & Giải pháp

| Khó khăn | Giải pháp |
|------------|----------|
| Codebase lớn với nhiều thư viện | Chạy local trước để hiểu luồng |
| Hiểu Docker images | Đọc Dockerfile theo từng dòng |
| Chọn phương án kiến trúc | Tham khảo AWS AI/ML reference architectures |

---

### Kế hoạch tuần tới

- Build Docker image cho ocr-api backend
- Push image lên Amazon ECR
- Tạo EC2 instance và deploy backend
- Cấu hình security groups và networking

---

### Tài liệu tham khảo

- [Amazon ECR Documentation](https://docs.aws.amazon.com/ecr)
- [Docker Documentation](https://docs.docker.com)
- [AWS AI Services Reference Architectures](https://aws.amazon.com/solutions/reference-architectures)
