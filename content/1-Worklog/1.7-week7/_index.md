---
title: "Week 7 Worklog"
date: "2026-05-26"
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

## Week 7: RDS Integration & Secrets Manager

> **Duration:** May 26 - June 1, 2026  
> **Theme:** Database Layer & Secure Credential Management

### 📚 Learning Objectives

- Set up Amazon RDS PostgreSQL for storing user translation history
- Design database schema with proper relationships
- Integrate AWS Secrets Manager for secure credential management
- Implement API endpoints for CRUD operations on user data

---

### 🎯 Key AWS Knowledge Acquired

#### Amazon RDS (Relational Database Service)

| Feature | Description |
|---------|-------------|
| **Managed Service** | AWS handles patching, backups, failover |
| **Multi-AZ** | High availability with synchronous replication |
| **Read Replicas** | Scale read operations |
| **Automated Backups** | Point-in-time recovery |
| **Encryption** | AES-256 encryption at rest |
| **Parameter Groups** | Configure database settings |

#### RDS Instance Classes

| Class | Use Case | vCPU | RAM |
|-------|----------|------|-----|
| **db.t3.micro** | Dev/Test | 2 | 1 GB |
| **db.t3.small** | Small apps | 2 | 2 GB |
| **db.t3.medium** | Medium apps | 2 | 4 GB |
| **db.m5.large** | Production | 2 | 8 GB |

#### AWS Secrets Manager

| Feature | Description |
|---------|-------------|
| **Automatic Rotation** | Rotate credentials automatically |
| **Encryption** | Encrypt secrets using KMS |
| **Access Control** | IAM policies for access |
| **Audit** | CloudTrail integration |
| **SDK Integration** | Easy integration with boto3 |

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

### 📋 Weekly Task List

| Day | Task | Start | End | Resources |
|-----|------|-------|-----|-----------|
| Mon | Create RDS PostgreSQL instance | 05/26 | 05/26 | AWS RDS |
| Tue | Design database schema | 05/27 | 05/27 | Database Design |
| Wed | Configure RDS security group | 05/28 | 05/28 | Security |
| Thu | Set up Secrets Manager | 05/29 | 05/29 | Secrets Manager |
| Fri | Update backend for DB integration | 05/30 | 05/30 | boto3 SDK |
| Sat | Implement history API endpoints | 06/01 | 06/01 | API Development |
| Sun | E2E testing | 06/02 | 06/02 | Integration |

---

### ✅ Achievements

- RDS PostgreSQL running in private subnet
- User translation history saved and retrieved successfully
- Database credentials managed securely via Secrets Manager
- Learned boto3 SDK for AWS services integration

### 🔐 Security Improvements

| Before | After |
|--------|-------|
| Hardcoded credentials | Credentials in Secrets Manager |
| No database | Persistent storage with backups |
| No data isolation | User-specific data access |

---

### 🚧 Difficulties & Solutions

| Difficulty | Solution |
|------------|----------|
| RDS connection failed | Added security group rule for port 5432 |
| Schema design not optimal | Referenced database design patterns |
| Secrets auto-rotation broke connection | Disabled in dev, enabled in production |

### 💡 Best Practices

- Always place RDS in **private subnet**
- Use **Secrets Manager** for all credentials
- Enable **automated backups** for production
- Use **read replicas** for read-heavy workloads

---

### 📅 Next Week's Plan

- Build **CI/CD pipeline** with GitHub Actions
- Automate build and deployment process
- Set up **AWS CloudWatch** monitoring

---

### 📖 Reference Materials

- [Amazon RDS Documentation](https://docs.aws.amazon.com/rds)
- [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager)
- [boto3 Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
