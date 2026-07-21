---
title: "Worklog Tuần 6"
date: "2026-05-19"
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

## Tuần 6: VPC Networking & Bảo mật

> **Thời gian:** 19/05 - 25/05/2026  
> **Chủ đề:** Network Isolation & Security Architecture

### Mục tiêu học tập

- Hiểu các thành phần VPC: Subnets, Route Tables, Internet Gateway, NAT Gateway
- Tạo VPC với public và private subnets across multiple AZs
- Deploy EC2 backend trong private subnet behind Application Load Balancer
- Implement security best practices

---

### Kiến thức AWS đã học

#### Các thành phần Amazon VPC

| Thành phần | Mục đích | Có thể truy cập từ Internet? |
|-----------|---------|-------------------------------|
| **VPC** | Virtual Private Cloud - network cô lập | Không |
| **Subnet** | Range của IP addresses trong VPC | Tùy thuộc (public/private) |
| **Internet Gateway (IGW)** | Cho phép VPC kết nối internet | - |
| **NAT Gateway** | Cho phép private subnet instances truy cập internet | - |
| **Route Table** | Định tuyến traffic trong VPC | - |
| **Security Group** | Virtual firewall cho EC2 (stateful) | - |
| **NACL** | Network Access Control List (stateless) | - |

#### Kiến trúc VPC

```
┌─────────────────────────────────────────────────────────────────────┐
│                           VPC (10.0.0.0/16)                          │
│                                                                      │
│  ┌─────────────────────────────┐    ┌─────────────────────────────┐ │
│  │    Public Subnet 1          │    │    Public Subnet 2          │ │
│  │    (10.0.1.0/24)           │    │    (10.0.2.0/24)           │ │
│  │    ap-southeast-1a          │    │    ap-southeast-1b          │ │
│  │                             │    │                             │ │
│  │  ┌───────────────────────┐ │    │                             │ │
│  │  │   ALB (Internet-facing)│ │    │                             │ │
│  │  └───────────────────────┘ │    │                             │ │
│  └─────────────┬───────────────┘    └─────────────────────────────┘ │
│                │                                                        │
│                │                                                        │
│  ┌─────────────▼───────────────┐    ┌─────────────────────────────┐ │
│  │    Private Subnet 1         │    │    Private Subnet 2         │ │
│  │    (10.0.11.0/24)          │    │    (10.0.12.0/24)          │ │
│  │    ap-southeast-1a          │    │    ap-southeast-1b          │ │
│  │                             │    │                             │ │
│  │  ┌───────────────────────┐ │    │                             │ │
│  │  │   EC2 (Backend)       │ │    │                             │ │
│  │  └───────────────────────┘ │    │                             │ │
│  └─────────────────────────────┘    └─────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

#### Application Load Balancer

| Tính năng | Mô tả |
|---------|-------------|
| **Health Checks** | Monitor instance health và route traffic chỉ đến healthy instances |
| **SSL Termination** | Handle HTTPS ở ALB level |
| **Target Groups** | Group instances cho different routing rules |
| **Cross-Zone Load Balancing** | Phân phối traffic across AZs |

#### Security Group vs NACL

| Aspect | Security Group | NACL |
|--------|---------------|------|
| Stateful?** | Có (auto allow response) | Không |
| Scope | Instance level | Subnet level |
| Evaluation | All rules | Rules in order |
| Default | Deny all inbound | Allow all |

---

### Công việc trong tuần

| Thứ | Công việc | Bắt đầu | Kết thúc | Tài liệu |
|-----|-----------|---------|---------|-----------|
| 2 (T3) | VPC Fundamentals: Subnets, IGW, NAT, Route Tables | 19/05 | 19/05 | AWS VPC |
| 3 (T4) | Tạo VPC với CIDR 10.0.0.0/16, 2 public + 2 private subnets | 20/05 | 20/05 | VPC Design |
| 4 (T5) | Deploy EC2 in private subnet | 21/05 | 21/05 | Network Architecture |
| 5 (T6) | Tạo ALB in public subnet | 22/05 | 22/05 | AWS ALB |
| 6 (T7) | Cấu hình security groups properly | 23/05 | 23/05 | Security |
| 7 (CN) | System testing after changes | 24/05 | 24/05 | Integration Test |

---

### Kết quả đạt được

- EC2 backend protected in private subnet, not directly exposed to internet
- ALB phân phối traffic với automatic health checks
- Hiểu sâu về VPC networking
- Hệ thống ổn định sau khi cải thiện kiến trúc

### Cải thiện bảo mật

| Trước | Sau |
|--------|-------|
| EC2 with public IP | EC2 in private subnet only |
| Direct internet access | Through ALB only |
| No network isolation | VPC isolation across AZs |

---

### Khó khăn & Giải pháp

| Khó khăn | Giải pháp |
|------------|----------|
| Khái niệm VPC trừu tượng | Vẽ sơ đồ mạng trước khi cấu hình |
| ALB health check failures | Cấu hình đúng health check path (/health) |
| NAT Gateway costs | Chỉ enable khi cần, disable sau khi dùng |

### Tối ưu chi phí

- NAT Gateway: ~$0.045/GB data processed
- ALB: ~$0.0225/hour + LCU charges
- Sử dụng **NAT Instance** (EC2) cho chi phí thấp hơn trong dev environments

---

### Kế hoạch tuần tới

- Kết nối **Amazon RDS PostgreSQL** để lưu trữ user history
- Cập nhật backend API để tương tác với database
- Tìm hiểu **AWS Secrets Manager** cho credential management

---

### Tài liệu tham khảo

- [Amazon VPC Documentation](https://docs.aws.amazon.com/vpc)
- [VPC Networking Best Practices](https://aws.amazon.com/architecture)
- [Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application)
