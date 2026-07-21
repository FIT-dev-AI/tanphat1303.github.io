---
title: "Worklog Tuần 1"
date: "2026-04-17"
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

## Tuần 1: AWS Cloud Foundation & Làm quen

> **Thời gian:** 17/04 - 30/07/2026  
> **Chủ đề:** AWS Fundamentals & Cloud Computing Basics

### 📚 Mục tiêu học tập

- Nắm vững các khái niệm cơ bản về điện toán đám mây và hệ sinh thái AWS
- Hiểu AWS Global Infrastructure (Regions, AZs, Edge Locations)
- Làm quen với AWS Management Console và các best practices bảo mật
- Hoàn thành lộ trình học AWS Cloud Practitioner

---

### 🎯 Kiến thức AWS đã học

#### Fundamentals về Cloud Computing

| Khái niệm | Mô tả |
|-----------|--------|
| **Cloud Computing** | Cung cấp tài nguyên IT theo yêu cầu qua internet với thanh toán theo usage |
| **IaaS** | Infrastructure as a Service - cung cấp tài nguyên máy chủ ảo (EC2, S3) |
| **PaaS** | Platform as a Service - cung cấp nền tảng phát triển ứng dụng (Beanstalk, Lambda) |
| **SaaS** | Software as a Service - cung cấp ứng dụng được quản lý hoàn toàn |
| **Cloud Deployment Models** | Public Cloud, Private Cloud, Hybrid Cloud, Multi-Cloud |

#### AWS Global Infrastructure

| Thành phần | Mục đích |
|-----------|-----------|
| **Region** | Khu vực địa lý chứa nhiều Availability Zones |
| **Availability Zone (AZ)** | Một hoặc nhiều data center với nguồn điện, mạng dự phòng |
| **Edge Location** | CDN endpoints để cache nội dung gần người dùng |
| **Local Zone** | Mở rộng của AWS Region cho ứng dụng nhạy cảm về độ trễ |
| **Wavelength** | Cho ứng dụng cần độ trễ cực thấp tại edge của mạng 5G |

---

### 📋 Công việc trong tuần

| Thứ | Công việc | Bắt đầu | Kết thúc | Tài liệu |
|-----|-----------|---------|---------|-----------|
| 5 (CN) | Cloud Computing Overview: IaaS/PaaS/SaaS | 17/04 | 17/04 | AWS Cloud Journey |
| 2 (T2) | AWS Global Infrastructure: Regions, AZs | 20/04 | 20/04 | AWS Documentation |
| 3 (T3) | AWS Management Console Navigation | 21/04 | 21/04 | AWS Console |
| 4 (T4) | AWS Account Security: IAM, MFA | 22/04 | 22/04 | IAM Best Practices |
| 5 (T5) | Practice Labs: EC2, S3, SSH | 23/04 | 23/04 | Bootcamp Labs |

---

### ✅ Kết quả đạt được

- Tạo thành công tài khoản AWS với cấu hình bảo mật
- Kích hoạt MFA (Multi-Factor Authentication) cho root account
- Tạo IAM user với các quyền phù hợp
- Nhận được **$100 AWS Credit** để sử dụng cho việc học tập
- Hoàn thành fundamentals của AWS Cloud Practitioner learning path

### 🔐 Best Practices bảo mật đã triển khai

```bash
# Enable MFA cho root account
aws iam create-virtual-mfa-device --virtual-mfa-device-name "root-mfa"

# Tạo IAM user với least privilege
aws iam create-user --user-name "intern-user"
aws iam attach-user-policy --user-name "intern-user" --policy-arn "arn:aws:iam::aws:policy/ReadOnlyAccess"
```

---

### 🚧 Khó khăn & Giải pháp

| Khó khăn | Giải pháp |
|------------|----------|
| Bị ngợp bởi quá nhiều AWS services | Tập trung vào core services trước: EC2, S3, IAM, VPC |
| Khái niệm IAM (Users, Roles, Policies) khó hiểu | Vẽ sơ đồ để hiểu mối quan hệ |
| Lỗi cấu hình security group | Ghi chép lại các port thông dụng |

---

### 📅 Kế hoạch tuần tới

- Tìm hiểu sâu hơn về **Amazon VPC** networking
- Thực hành **SSH connectivity** đến EC2 instances
- Học **Linux command line basics** để quản lý server

---

### 📖 Tài liệu tham khảo

- [AWS Cloud Practitioner Essentials](https://explore.skillbuilder.aws/learn/course/134/aws-cloud-practitioner-essentials)
- [AWS Documentation](https://docs.aws.amazon.com)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture)
