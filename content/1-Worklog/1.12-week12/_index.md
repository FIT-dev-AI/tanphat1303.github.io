---
title: "Week 12 Worklog"
date: "2026-06-30"
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
---

## Week 12: Load Testing, Auto Scaling & Final Report

> **Duration:** June 30 - July 30, 2026  
> **Theme:** Performance Optimization & Internship Completion

### Learning Objectives

- Perform load testing using Apache JMeter
- Configure Auto Scaling Group for automatic scaling
- Complete comprehensive final internship report
- Present final demo to mentor and Bootcamp team

---

### Key AWS Knowledge Acquired

#### Apache JMeter for Load Testing

| Feature | Description |
|---------|-------------|
| **Protocol Support** | HTTP, HTTPS, JDBC, JMS |
| **Distributed Testing** | Multiple machines for high load |
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

| Component | Description |
|-----------|-------------|
| **Auto Scaling Group** | Manages EC2 instances across AZs |
| **Launch Template** | Instance configuration |
| **Scaling Policies** | Rules for scaling in/out |
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
| **Scale Out** | CPU > 70% for 3 min | Add 1 instance |
| **Scale In** | CPU < 30% for 5 min | Remove 1 instance |
| **Scheduled** | Cron schedule | Set capacity |

#### Performance Testing Results

| Load Level | Response Time | Error Rate | System Behavior |
|------------|---------------|------------|-----------------|
| 50 users | 320ms avg | 0% | Normal |
| 100 users | 580ms avg | 0.2% | Normal |
| 150 users | 1.2s avg | 2% | Slight degradation |
| 200 users | 3.5s avg | 8% | Significant degradation |

---

### Weekly Task List

| Day | Task | Start | End | Resources |
|-----|------|-------|-----|-----------|
| Mon | JMeter setup and test plan creation | 06/30 | 06/30 | JMeter |
| Tue | Run load tests: 50, 100, 200 users | 07/01 | 07/01 | Load Testing |
| Wed | Analyze bottleneck with CloudWatch | 07/02 | 07/02 | CloudWatch |
| Thu | Configure Auto Scaling Group | 07/03 | 07/03 | ASG |
| Fri | Create Launch Template with user data | 07/04 | 07/04 | Launch Template |
| Sat | Verify auto-scaling under load | 07/05 | 07/05 | ASG Testing |
| Week 3-4 | Final report writing | 07/06-07/30 | Final Report |
| Week 4 | Final demo presentation | 07/28-07/30 | Demo |

---

### Achievements

- System handles 100 concurrent users stably
- Auto Scaling Group: 1 to 3 instances under load
- Comprehensive 3-month internship report completed
- Positive mentor feedback
- OCR-CapCut fully deployed and functional

### Load Test Results (Before ASG)

| Metric | 50 Users | 100 Users | 200 Users |
|--------|----------|-----------|-----------|
| Throughput | 150 req/s | 180 req/s | 120 req/s |
| Avg Response | 320ms | 580ms | 1.8s |
| Max Response | 800ms | 2.1s | 5.2s |
| Error Rate | 0% | 0.2% | 8% |

### Auto Scaling Performance (After ASG)

| Metric | 50 Users | 100 Users | 200 Users |
|--------|----------|-----------|-----------|
| Instances | 1 | 2 | 3 |
| Avg Response | 320ms | 350ms | 420ms |
| Max Response | 800ms | 900ms | 1.2s |
| Error Rate | 0% | 0% | 0.5% |

---

### Difficulties & Solutions

| Difficulty | Solution |
|------------|----------|
| JMeter learning curve | Used YouTube tutorials |
| Launch Template script errors | Tested manually first |
| Balancing work and report | Time blocking strategy |

### Key Lessons Learned

- **Load testing** is essential before production
- **Auto Scaling** provides resilience and cost optimization
- **Documentation** is as important as code
- **Continuous learning** is key in cloud engineering

---

### Future Plans

- Pursue **AWS Cloud Practitioner** certification
- Improve OCR-CapCut with video processing
- Apply AWS knowledge to graduation project
- Continue learning advanced AWS services

---

### Skills Acquired Throughout Internship

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

### Reference Materials

- [Apache JMeter](https://jmeter.apache.org)
- [Auto Scaling Documentation](https://docs.aws.amazon.com/autoscaling)
- [Well-Architected Framework](https://aws.amazon.com/architecture)
