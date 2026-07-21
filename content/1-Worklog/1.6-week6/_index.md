---
title: "Week 6 Worklog"
date: "2026-05-19"
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

## Week 6: VPC Networking & Security

> **Duration:** May 19 - 25, 2026  
> **Theme:** Network Isolation & Security Architecture

### Learning Objectives

- Understand VPC components: Subnets, Route Tables, Internet Gateway, NAT Gateway
- Create VPC with public and private subnets across multiple AZs
- Deploy EC2 backend in private subnet behind Application Load Balancer
- Implement security best practices

---

### Key AWS Knowledge Acquired

#### Amazon VPC Components

| Component | Purpose | Can be accessed from Internet? |
|-----------|---------|-------------------------------|
| **VPC** | Virtual Private Cloud - isolated network | No |
| **Subnet** | Range of IP addresses in a VPC | Depends (public/private) |
| **Internet Gateway (IGW)** | Allows VPC to connect to internet | - |
| **NAT Gateway** | Allows private subnet instances to access internet | - |
| **Route Table** | Routes traffic within VPC | - |
| **Security Group** | Virtual firewall for EC2 (stateful) | - |
| **NACL** | Network Access Control List (stateless) | - |

#### VPC Architecture

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

| Feature | Description |
|---------|-------------|
| **Health Checks** | Monitor instance health and route traffic only to healthy instances |
| **SSL Termination** | Handle HTTPS at ALB level |
| **Target Groups** | Group instances for different routing rules |
| **Cross-Zone Load Balancing** | Distribute traffic across AZs |

#### Security Group vs NACL

| Aspect | Security Group | NACL |
|--------|---------------|------|
| Stateful?** | Yes (auto allow response) | No |
| Scope | Instance level | Subnet level |
| Evaluation | All rules | Rules in order |
| Default | Deny all inbound | Allow all |

---

### Weekly Task List

| Day | Task | Start | End | Resources |
|-----|------|-------|-----|-----------|
| Mon | VPC Fundamentals: Subnets, IGW, NAT, Route Tables | 05/19 | 05/19 | AWS VPC |
| Tue | Create VPC with CIDR 10.0.0.0/16, 2 public + 2 private subnets | 05/20 | 05/20 | VPC Design |
| Wed | Deploy EC2 in private subnet | 05/21 | 05/21 | Network Architecture |
| Thu | Create ALB in public subnet | 05/22 | 05/22 | AWS ALB |
| Fri | Configure security groups properly | 05/23 | 05/23 | Security |
| Sat | System testing after changes | 05/24 | 05/24 | Integration Test |

---

### Achievements

- EC2 backend protected in private subnet, not directly exposed to internet
- ALB distributing traffic with automatic health checks
- Deep understanding of VPC networking
- System stable after architecture improvements

### Security Improvements

| Before | After |
|--------|-------|
| EC2 with public IP | EC2 in private subnet only |
| Direct internet access | Through ALB only |
| No network isolation | VPC isolation across AZs |

---

### Difficulties & Solutions

| Difficulty | Solution |
|------------|----------|
| Abstract VPC concepts | Drew network diagram before configuring |
| ALB health check failures | Configured correct health check path (/health) |
| NAT Gateway costs | Only enable when needed, disable after use |

### Cost Optimization

- NAT Gateway: ~$0.045/GB data processed
- ALB: ~$0.0225/hour + LCU charges
- Use **NAT Instance** (EC2) for lower costs in dev environments

---

### Next Week's Plan

- Connect **Amazon RDS PostgreSQL** for storing user history
- Update backend API to interact with database
- Learn **AWS Secrets Manager** for credential management

---

### Reference Materials

- [Amazon VPC Documentation](https://docs.aws.amazon.com/vpc)
- [VPC Networking Best Practices](https://aws.amazon.com/architecture)
- [Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application)
