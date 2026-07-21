---
title: "Worklog Tuần 12"
date: "2026-06-30"
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
---

## Tuần 12: Load Testing, Auto Scaling & Báo cáo cuối kỳ

> **Thời gian:** 30/06 - 30/07/2026  
> **Chủ đề:** Performance Optimization & Hoàn thành thực tập

### 📚 Mục tiêu học tập

- Thực hiện load testing sử dụng Apache JMeter
- Cấu hình Auto Scaling Group cho automatic scaling
- Hoàn thành comprehensive final internship report
- Present final demo cho mentor và Bootcamp team

---

### 🎯 Kiến thức AWS đã học

#### Apache JMeter cho Load Testing

| Tính năng | Mô tả |
|---------|-------------|
| **Protocol Support** | HTTP, HTTPS, JDBC, JMS |
| **Distributed Testing** | Multiple machines cho high load |
| **Assertions** | Validate responses |
| **Reports** | HTML dashboards, graphs |

#### JMeter Test Plan Structure

```
Test Plan
├── Thread Group (Users, Ramp-up, Duration)
│   ├── HTTP Request Defaults
│   ├── Cookie Manager
│   └── HTTP Requests
│       ├── Home Page
│       ├── OCR Upload
│       └── Translation Result
├── Listeners
│   ├── View Results Tree
│   ├── Summary Report
│   └── Graph Results
└── Timers
    └── Constant Throughput Timer
```

#### JMeter Configuration

```xml
<!-- Thread Group Configuration -->
<ThreadGroup guiclass="ThreadGroupGui" testclass="ThreadGroup">
    <stringProp name="ThreadGroup.on_sample_error">continue</stringProp>
    <elementProp name="ThreadGroup.main_controller">
        <boolProp name="Controller.loop_controller.continue_forever">false</boolProp>
        <stringProp name="LoopController.loops">10</stringProp>
    </elementProp>
    <stringProp name="ThreadGroup.num_threads">100</stringProp>
    <stringProp name="ThreadGroup.ramp_time">60</stringProp>
    <stringProp name="ThreadGroup.duration">300</stringProp>
</ThreadGroup>

<!-- HTTP Request -->
<HTTPSamplerProxy guiclass="HttpTestSampleGui" testclass="HTTPSamplerProxy">
    <stringProp name="HTTPSampler.domain">api.example.com</stringProp>
    <stringProp name="HTTPSampler.path">/api/ocr</stringProp>
    <stringProp name="HTTPSampler.method">POST</stringProp>
    <boolProp name="HTTPSampler.auto_redirects">false</boolProp>
</HTTPSamplerProxy>
```

#### Amazon EC2 Auto Scaling

| Thành phần | Mô tả |
|-----------|-------------|
| **Auto Scaling Group** | Quản lý EC2 instances across AZs |
| **Launch Template** | Instance configuration |
| **Scaling Policies** | Rules cho scaling in/out |
| **Health Checks** | Instance health monitoring |

#### Auto Scaling Policy

```json
{
  "AutoScalingGroupName": "ocr-backend-asg",
  "MinSize": 1,
  "MaxSize": 5,
  "DesiredCapacity": 1,
  "AvailabilityZones": ["ap-southeast-1a", "ap-southeast-1b"],
  "LaunchTemplate": {
    "LaunchTemplateId": "lt-1234567890",
    "Version": "$Latest"
  },
  "TargetGroupARNs": ["arn:aws:elasticloadbalancing:..."],
  "HealthCheckType": "ELB",
  "HealthCheckGracePeriod": 60,
  "TerminationPolicies": ["OldestInstance"]
}
```

#### Scaling Policies

| Policy | Trigger | Action |
|--------|---------|--------|
| **Scale Out** | CPU > 70% trong 3 min | Thêm 1 instance |
| **Scale In** | CPU < 30% trong 5 min | Xóa 1 instance |
| **Scheduled** | Cron schedule | Đặt capacity |

#### Performance Testing Results

| Load Level | Response Time | Error Rate | System Behavior |
|------------|---------------|------------|-----------------|
| 50 users | 320ms avg | 0% | Normal |
| 100 users | 580ms avg | 0.2% | Normal |
| 150 users | 1.2s avg | 2% | Slight degradation |
| 200 users | 3.5s avg | 8% | Significant degradation |

---

### 📋 Công việc trong tuần

| Thứ | Công việc | Bắt đầu | Kết thúc | Tài liệu |
|-----|-----------|---------|---------|-----------|
| 2 (T3) | JMeter setup và tạo test plan | 30/06 | 30/06 | JMeter |
| 3 (T4) | Chạy load tests: 50, 100, 200 users | 01/07 | 01/07 | Load Testing |
| 4 (T5) | Phân tích bottleneck với CloudWatch | 02/07 | 02/07 | CloudWatch |
| 5 (T6) | Cấu hình Auto Scaling Group | 03/07 | 03/07 | ASG |
| 6 (T7) | Tạo Launch Template với user data | 04/07 | 04/07 | Launch Template |
| 7 (CN) | Xác nhận auto-scaling under load | 05/07 | 05/07 | ASG Testing |
| Tuần 3-4 | Viết báo cáo cuối kỳ | 06/07-30/07 | Final Report |
| Tuần 4 | Final demo presentation | 28/07-30/07 | Demo |

---

### ✅ Kết quả đạt được

- Hệ thống xử lý 100 concurrent users ổn định
- Auto Scaling Group: 1 → 3 instances khi load cao
- Comprehensive 3-month internship report hoàn thành
- Mentor feedback tích cực
- OCR-CapCut fully deployed và functional

### 📊 Load Test Results (Before ASG)

| Metric | 50 Users | 100 Users | 200 Users |
|--------|----------|-----------|-----------|
| Throughput | 150 req/s | 180 req/s | 120 req/s |
| Avg Response | 320ms | 580ms | 1.8s |
| Max Response | 800ms | 2.1s | 5.2s |
| Error Rate | 0% | 0.2% | 8% |

### 📊 Auto Scaling Performance (After ASG)

| Metric | 50 Users | 100 Users | 200 Users |
|--------|----------|-----------|-----------|
| Instances | 1 | 2 | 3 |
| Avg Response | 320ms | 350ms | 420ms |
| Max Response | 800ms | 900ms | 1.2s |
| Error Rate | 0% | 0% | 0.5% |

---

### 🚧 Khó khăn & Giải pháp

| Khó khăn | Giải pháp |
|------------|----------|
| JMeter learning curve | Sử dụng YouTube tutorials |
| Launch Template script errors | Test manually trước |
| Cân bằng work và report | Time blocking strategy |

### 💡 Bài học quan trọng

- **Load testing** là essential trước khi lên production
- **Auto Scaling** cung cấp resilience và cost optimization
- **Documentation** quan trọng ngang code
- **Continuous learning** là key trong cloud engineering

---

### 🎯 Kế hoạch tương lai

- Theo đuổi **AWS Cloud Practitioner** certification
- Cải thiện OCR-CapCut với video processing
- Áp dụng AWS knowledge vào đồ án tốt nghiệp
- Tiếp tục học advanced AWS services

---

### 📚 Kỹ năng đạt được trong suốt kỳ thực tập

| Category | Skills |
|----------|--------|
| **Compute** | EC2, ECS, Lambda, Auto Scaling |
| **Storage** | S3, EBS, EFS |
| **Database** | RDS PostgreSQL |
| **Networking** | VPC, ALB, CloudFront, Route 53 |
| **Security** | IAM, Secrets Manager, Security Hub |
| **CI/CD** | GitHub Actions, CodeDeploy |
| **Monitoring** | CloudWatch, Alarms, Logs |
| **AI/ML** | Textract, Translate |
| **Email** | SES |

---

### 📖 Tài liệu tham khảo

- [Apache JMeter](https://jmeter.apache.org)
- [Auto Scaling Documentation](https://docs.aws.amazon.com/autoscaling)
- [Well-Architected Framework](https://aws.amazon.com/architecture)
