---
title: "Week 1 Worklog"
date: "2026-04-17"
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

## Week 1: AWS Cloud Foundation & Getting Started

> **Duration:** April 17 - July 30, 2026  
> **Theme:** AWS Fundamentals & Cloud Computing Basics

### 📚 Learning Objectives

- Master core cloud computing concepts and AWS ecosystem
- Understand AWS Global Infrastructure (Regions, AZs, Edge Locations)
- Get familiar with AWS Management Console and security best practices
- Complete AWS Cloud Practitioner learning path

---

### 🎯 Key AWS Knowledge Acquired

#### Cloud Computing Fundamentals
| Concept | Description |
|---------|-------------|
| **Cloud Computing** | On-demand delivery of IT resources via the internet with pay-as-you-go pricing |
| **IaaS** | Infrastructure as a Service - provides virtualized computing resources (EC2, S3) |
| **PaaS** | Platform as a Service - provides platform for developing applications (Beanstalk, Lambda) |
| **SaaS** | Software as a Service - provides fully managed applications (Google Workspace, Salesforce) |
| **Cloud Deployment Models** | Public Cloud, Private Cloud, Hybrid Cloud, Multi-Cloud |

#### AWS Global Infrastructure
| Component | Purpose |
|-----------|---------|
| **Region** | Geographic area containing multiple Availability Zones (e.g., us-east-1, ap-southeast-1) |
| **Availability Zone (AZ)** | One or more discrete data centers with redundant power, networking, and connectivity |
| **Edge Location** | CDN endpoints for caching content closer to users (CloudFront, Route 53) |
| **Local Zone** | Extension of AWS Region for latency-sensitive applications |
| **Wavelength** | For applications requiring ultra-low latency at the edge of 5G networks |

---

### 📋 Weekly Task List

| Day | Task | Start | End | Resources |
|-----|------|-------|-----|-----------|
| Thu | Cloud Computing Overview: IaaS/PaaS/SaaS models | 04/17 | 04/17 | AWS Cloud Journey |
| Mon | AWS Global Infrastructure: Regions, AZs, Edge Locations | 04/20 | 04/20 | AWS Documentation |
| Tue | AWS Management Console Navigation | 04/21 | 04/21 | AWS Console |
| Wed | AWS Account Security: IAM, MFA, Root Account Protection | 04/22 | 04/22 | IAM Best Practices |
| Thu | Practice Labs: Create EC2, S3 Bucket, Connect via SSH | 04/23 | 04/23 | Bootcamp Labs |

---

### ✅ Achievements

- Successfully created AWS account with proper security configurations
- Activated MFA (Multi-Factor Authentication) for root account
- Created IAM user with appropriate permissions
- Received **$100 AWS Credit** for learning purposes
- Completed AWS Cloud Practitioner learning path fundamentals

### 🔐 Security Best Practices Implemented

```bash
# Enable MFA for root account
aws iam create-virtual-mfa-device --virtual-mfa-device-name "root-mfa"

# Create IAM user with least privilege
aws iam create-user --user-name "intern-user"
aws iam attach-user-policy --user-name "intern-user" --policy-arn "arn:aws:iam::aws:policy/ReadOnlyAccess"
```

---

### 🚧 Difficulties & Solutions

| Difficulty | Solution |
|------------|----------|
| Overwhelmed by many AWS services | Focused on core services first: EC2, S3, IAM, VPC |
| IAM concepts (Users, Roles, Policies) confusing | Created visual diagram to understand relationships |
| Security group configuration errors | Documented common port configurations |

---

### 📅 Next Week's Plan

- Deep dive into **Amazon VPC** networking concepts
- Practice **SSH connectivity** to EC2 instances
- Learn **Linux command line basics** for server management

---

### 📖 Reference Materials

- [AWS Cloud Practitioner Essentials](https://explore.skillbuilder.aws/learn/course/134/aws-cloud-practitioner-essentials)
- [AWS Documentation](https://docs.aws.amazon.com)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture)
