---
title: "Worklog Tuần 2"
date: "2026-04-24"
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

## Tuần 2: SSH, Linux Operations & Cơ bản EC2

> **Thời gian:** 24/04 - 30/04/2026  
> **Chủ đề:** Server Management & Command Line Mastery

### 📚 Mục tiêu học tập

- Thành thạo kết nối SSH tới EC2 instances với best practices
- Hiểu fundamentals về quản trị Linux server
- Tìm hiểu các loại EC2 instance và use cases
- Thực hành secure file transfer giữa local và remote servers

---

### 🎯 Kiến thức AWS đã học

#### Amazon EC2 Deep Dive

| Tính năng | Mô tả |
|---------|-------------|
| **Instance Types** | General Purpose (T, M), Compute Optimized (C), Memory Optimized (R, X), Storage Optimized (D, I) |
| **Instance Lifecycle** | Pending → Running → Stopping → Stopped → Shutting-down → Terminated |
| **AMI** | Template đã cấu hình sẵn cho instance (OS, applications) |
| **Instance State** | Running, Stopped, Terminated - chỉ tính phí khi Running |
| **EBS Volumes** | Block storage gắn vào EC2, persist qua các lần restart |

#### So sánh EC2 Instance Types

| Loại | Use Case | Specs |
|------|----------|-------|
| **T3** | General purpose, burstable performance | 2-4 vCPUs, 1-16 GB RAM |
| **M5** | Cân bằng compute và memory | 4-96 vCPUs, 16-384 GB RAM |
| **C5** | Compute intensive workloads | 4-72 vCPUs, 8-144 GB RAM |
| **R5** | Memory intensive applications | 4-192 vCPUs, 16-768 GB RAM |

#### SSH & Key Pair Management

```bash
# Kết nối EC2 bằng key pair
ssh -i "your-key.pem" ec2-user@<public-ip>

# Bảo mật key pair (Linux/Mac)
chmod 400 your-key.pem

# Copy file lên EC2
scp -i "your-key.pem" local-file.txt ec2-user@<public-ip>:/home/ec2-user/

# SSH với key forwarding
ssh -A -i "your-key.pem" ec2-user@<public-ip>
```

---

### 📋 Công việc trong tuần

| Thứ | Công việc | Bắt đầu | Kết thúc | Tài liệu |
|-----|-----------|---------|---------|-----------|
| 5 (CN) | SSH Fundamentals: Key pair authentication | 24/04 | 24/04 | AWS EC2 Docs |
| 2 (T2) | Kết nối EC2 bằng MobaXterm và VSCode Remote | 27/04 | 27/04 | SSH Best Practices |
| 3 (T3) | Linux Operations: File creation, packages, navigation | 28/04 | 28/04 | Linux Reference |
| 4 (T4) | File Transfer: SCP/SFTP giữa local và EC2 | 29/04 | 29/04 | AWS Transfer Guide |
| 5 (T5) | Advanced Labs: EC2 management và troubleshooting | 30/04 | 30/04 | Bootcamp Labs |

---

### ✅ Kết quả đạt được

- Kết nối thành công tới EC2 instance qua SSH với xác thực key pair
- Thành thạo Linux command line cho quản trị server
- Hiểu EC2 instance lifecycle và billing model
- Sử dụng MobaXterm cho kết nối SSH ổn định hơn
- Thực hiện file transfers bằng SCP và SFTP protocols

### 🔧 Các lệnh Linux đã thành thạo

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

### 🚧 Khó khăn & Giải pháp

| Khó khăn | Giải pháp |
|------------|----------|
| VSCode SSH lỗi permission với file .pem | Dùng MobaXterm thay thế - ổn định hơn |
| Security group port 22 chưa mở | Kiểm tra inbound rules trước khi kết nối |
| Chậm trong việc học Linux command line | Thực hành hàng ngày với cheat sheets |

### 💡 Best Practices đã học

- **Không bao giờ** expose port 22 ra thế giới (0.0.0.0/0)
- Sử dụng **bastion host** hoặc **Systems Manager Session Manager** cho production
- Luôn **stop** instances khi không sử dụng để tiết kiệm chi phí
- Giữ key pairs **bảo mật** và không commit vào version control

---

### 📅 Kế hoạch tuần tới

- Phân tích **kiến trúc dự án OCR-CapCut** (frontend + backend)
- Nghiên cứu **Docker containerization** concepts
- Thiết kế **AWS deployment architecture**

---

### 📖 Tài liệu tham khảo

- [Amazon EC2 Documentation](https://docs.aws.amazon.com/ec2)
- [Connect to Linux instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstances.html)
- [SSH Essentials](https://www.digitalocean.com/community/tutorials/ssh-essentials)
