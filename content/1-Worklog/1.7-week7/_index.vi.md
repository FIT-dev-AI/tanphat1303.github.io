---
title: "Worklog Tuần 7"
date: "2026-05-26"
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

## Tuần 7: Tích hợp RDS & Secrets Manager

> **Thời gian:** 26/05 - 01/06/2026  
> **Chủ đề:** Database Layer & Secure Credential Management

### 📚 Mục tiêu học tập

- Thiết lập Amazon RDS PostgreSQL để lưu trữ lịch sử dịch thuật
- Thiết kế database schema với các mối quan hệ phù hợp
- Tích hợp AWS Secrets Manager cho quản lý credentials bảo mật
- Implement API endpoints cho CRUD operations trên user data

---

### 🎯 Kiến thức AWS đã học

#### Amazon RDS (Relational Database Service)

| Tính năng | Mô tả |
|---------|-------------|
| **Managed Service** | AWS xử lý patching, backups, failover |
| **Multi-AZ** | High availability với synchronous replication |
| **Read Replicas** | Scale read operations |
| **Automated Backups** | Point-in-time recovery |
| **Encryption** | AES-256 encryption at rest |
| **Parameter Groups** | Cấu hình database settings |

#### RDS Instance Classes

| Class | Use Case | vCPU | RAM |
|-------|----------|------|-----|
| **db.t3.micro** | Dev/Test | 2 | 1 GB |
| **db.t3.small** | Small apps | 2 | 2 GB |
| **db.t3.medium** | Medium apps | 2 | 4 GB |
| **db.m5.large** | Production | 2 | 8 GB |

#### AWS Secrets Manager

| Tính năng | Mô tả |
|---------|-------------|
| **Automatic Rotation** | Rotate credentials tự động |
| **Encryption** | Encrypt secrets sử dụng KMS |
| **Access Control** | IAM policies cho access |
| **Audit** | CloudTrail integration |
| **SDK Integration** | Dễ dàng tích hợp với boto3 |

#### Secrets Manager with boto3

```python
import boto3
import json

def get_db_credentials():
    client = boto3.client('secretsmanager')
    response = client.get_secret_value(
        SecretId='prod/ocr-app/database'
    )
    return json.loads(response['SecretString'])

# Usage in connection
creds = get_db_credentials()
conn = psycopg2.connect(
    host=creds['host'],
    dbname=creds['dbname'],
    user=creds['username'],
    password=creds['password']
)
```

#### Database Schema Design

```sql
-- Users table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- OCR History table
CREATE TABLE ocr_history (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    source_text TEXT,
    translated_text TEXT,
    source_lang VARCHAR(10),
    target_lang VARCHAR(10),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Index for faster queries
CREATE INDEX idx_ocr_history_user_id ON ocr_history(user_id);
CREATE INDEX idx_ocr_history_created_at ON ocr_history(created_at);
```

---

### 📋 Công việc trong tuần

| Thứ | Công việc | Bắt đầu | Kết thúc | Tài liệu |
|-----|-----------|---------|---------|-----------|
| 2 (T3) | Tạo RDS PostgreSQL instance | 26/05 | 26/05 | AWS RDS |
| 3 (T4) | Thiết kế database schema | 27/05 | 27/05 | Database Design |
| 4 (T5) | Cấu hình RDS security group | 28/05 | 28/05 | Security |
| 5 (T6) | Thiết lập Secrets Manager | 29/05 | 29/05 | Secrets Manager |
| 6 (T7) | Update backend cho DB integration | 30/05 | 30/05 | boto3 SDK |
| 7 (CN) | Implement history API endpoints | 01/06 | 01/06 | API Development |
| 8 (T2) | E2E testing | 02/06 | 02/06 | Integration |

---

### ✅ Kết quả đạt được

- RDS PostgreSQL chạy trong private subnet
- User translation history được lưu và truy xuất thành công
- Database credentials được quản lý bảo mật qua Secrets Manager
- Học được boto3 SDK để tích hợp AWS services

### 🔐 Cải thiện bảo mật

| Trước | Sau |
|--------|-------|
| Hardcoded credentials | Credentials trong Secrets Manager |
| Không có database | Persistent storage với backups |
| Không có data isolation | User-specific data access |

---

### 🚧 Khó khăn & Giải pháp

| Khó khăn | Giải pháp |
|------------|----------|
| Kết nối RDS thất bại | Thêm security group rule cho port 5432 |
| Schema design không tối ưu | Tham khảo database design patterns |
| Secrets auto-rotation làm đứt kết nối | Tắt trong dev, bật trong production |

### 💡 Best Practices

- Luôn đặt RDS trong **private subnet**
- Sử dụng **Secrets Manager** cho tất cả credentials
- Enable **automated backups** cho production
- Sử dụng **read replicas** cho read-heavy workloads

---

### 📅 Kế hoạch tuần tới

- Xây dựng **CI/CD pipeline** với GitHub Actions
- Tự động hóa quá trình build và deployment
- Thiết lập **AWS CloudWatch** monitoring

---

### 📖 Tài liệu tham khảo

- [Amazon RDS Documentation](https://docs.aws.amazon.com/rds)
- [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager)
- [boto3 Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
