---
title: "Week 2 Worklog"
date: "2026-04-24"
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

## Week 2: SSH, Linux Operations & EC2 Basics

> **Duration:** April 24 - 30, 2026  
> **Theme:** Server Management & Command Line Mastery

### Learning Objectives

- Master SSH connectivity to EC2 instances using best practices
- Understand Linux server administration fundamentals
- Learn EC2 instance types and use cases
- Practice secure file transfer between local and remote servers

---

### Key AWS Knowledge Acquired

#### Amazon EC2 Deep Dive

| Feature | Description |
|---------|-------------|
| **Instance Types** | General Purpose (T, M), Compute Optimized (C), Memory Optimized (R, X), Storage Optimized (D, I) |
| **Instance Lifecycle** | Pending → Running → Stopping → Stopped → Shutting-down → Terminated |
| **AMI (Amazon Machine Image)** | Pre-configured template for your instance (OS, applications) |
| **Instance State** | Running, Stopped, Terminated - costs only for Running state |
| **EBS Volumes** | Persistent block storage attached to EC2 |

#### EC2 Instance Types Comparison

| Type | Use Case | Specs |
|------|----------|-------|
| **T3** | General purpose, burstable performance | 2-4 vCPUs, 1-16 GB RAM |
| **M5** | Balanced compute and memory | 4-96 vCPUs, 16-384 GB RAM |
| **C5** | Compute intensive workloads | 4-72 vCPUs, 8-144 GB RAM |
| **R5** | Memory intensive applications | 4-192 vCPUs, 16-768 GB RAM |

#### SSH & Key Pair Management

```bash
# Connect to EC2 using key pair
ssh -i "your-key.pem" ec2-user@<public-ip>

# Secure your key pair (Linux/Mac)
chmod 400 your-key.pem

# Copy files to EC2
scp -i "your-key.pem" local-file.txt ec2-user@<public-ip>:/home/ec2-user/

# SSH with key forwarding
ssh -A -i "your-key.pem" ec2-user@<public-ip>
```

---

### Weekly Task List

| Day | Task | Start | End | Resources |
|-----|------|-------|-----|-----------|
| Thu | SSH Fundamentals: Key pair (.pem) authentication | 04/24 | 04/24 | AWS EC2 Docs |
| Mon | Connect to EC2 using MobaXterm and VSCode Remote | 04/27 | 04/27 | SSH Best Practices |
| Tue | Linux Operations: File creation, package installation, navigation | 04/28 | 04/28 | Linux Reference |
| Wed | File Transfer: SCP/SFTP between local and EC2 | 04/29 | 04/29 | AWS Transfer Guide |
| Thu | Advanced Labs: EC2 management and troubleshooting | 04/30 | 04/30 | Bootcamp Labs |

---

### Achievements

- Successfully connected to EC2 instance via SSH using secure key pair authentication
- Mastered Linux command line for server administration
- Understood EC2 instance lifecycle and billing model
- Learned to use MobaXterm for stable SSH connections (preferred over VSCode)
- Performed file transfers using SCP and SFTP protocols

### Linux Commands Mastered

```bash
# System Information
uname -a              # Kernel version
cat /etc/os-release   # OS information
free -h               # Memory usage
df -h                 # Disk usage

# Package Management (Ubuntu/Debian)
sudo apt update && sudo apt upgrade -y
sudo apt install <package-name>
sudo systemctl start|stop|restart <service>

# File Operations
mkdir, cp, mv, rm, chmod, chown
tar -czvf archive.tar.gz /path/to/dir
find / -name "filename"

# Process Management
ps aux | grep <process>
top or htop
kill -9 <pid>
```

---

### Difficulties & Solutions

| Difficulty | Solution |
|------------|----------|
| VSCode SSH permission errors with .pem file | Used MobaXterm instead - more stable |
| Security group port 22 not open | Double-checked inbound rules before connecting |
| Slow Linux command line learning curve | Practiced daily with cheat sheets |

### Best Practices Learned

- **Never** expose port 22 to the world (0.0.0.0/0)
- Use **bastion host** or **Systems Manager Session Manager** for production
- Always **stop** instances when not in use to save costs
- Keep key pairs **secure** and never commit to version control

---

### Next Week's Plan

- Analyze **OCR-CapCut project architecture** (frontend + backend)
- Study **Docker containerization** concepts
- Design **AWS deployment architecture**

---

### Reference Materials

- [Amazon EC2 Documentation](https://docs.aws.amazon.com/ec2)
- [Connect to Linux instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstances.html)
- [SSH Essentials](https://www.digitalocean.com/community/tutorials/ssh-essentials)
