---
title: "Week 9 Worklog"
date: "2026-06-09"
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

## Week 9: CloudWatch Monitoring & Alerts

> **Duration:** June 9 - 15, 2026  
> **Theme:** Observability & System Monitoring

### Learning Objectives

- Set up CloudWatch for application monitoring (logs and metrics)
- Create CloudWatch Dashboard with key metrics
- Configure CloudWatch Alarms for proactive notifications
- Implement structured logging for easier debugging

---

### Key AWS Knowledge Acquired

#### Amazon CloudWatch Components

| Component | Purpose | Data Retention |
|-----------|---------|----------------|
| **Metrics** | Numerical data about AWS resources | 15 months |
| **Logs** | Application and system logs | Configurable (1 day - indefinite) |
| **Alarms** | Trigger notifications based on metrics | Until deleted |
| **Dashboards** | Customizable visualizations | Until deleted |

#### CloudWatch Metrics for EC2

| Metric | Description |
|--------|-------------|
| **CPUUtilization** | CPU usage percentage |
| **NetworkIn/Out** | Network traffic in bytes |
| **DiskRead/Write** | Disk I/O operations |
| **StatusCheckFailed** | Instance health status |

#### Custom Metrics with CloudWatch Agent

```json
{
  "agent": {
    "metrics_collection_interval": 60
  },
  "metrics": {
    "namespace": "OCRApp",
    "metrics_collection_interval": 60,
    "metrics": [
      {
        "id": "app_metrics",
        "metric_stat": {
          "stat": "Average",
          "period": 60,
          "unit": "Count"
        },
        "dimensions": [
          {
            "name": "InstanceId",
            "value": "i-1234567890abcdef0"
          }
        ]
      }
    ]
  }
}
```

#### Structured Logging

```python
import logging
import json
from datetime import datetime

class StructuredFormatter(logging.Formatter):
    def format(self, record):
        log_data = {
            "timestamp": datetime.utcnow().isoformat(),
            "level": record.levelname,
            "message": record.getMessage(),
            "logger": record.name,
            "request_id": getattr(record, "request_id", None),
            "user_id": getattr(record, "user_id", None)
        }
        return json.dumps(log_data)

# Usage
logger = logging.getLogger("ocr_api")
handler = logging.StreamHandler()
handler.setFormatter(StructuredFormatter())
logger.addHandler(handler)
logger.info("OCR processing completed", extra={"request_id": "req-123", "user_id": "user-456"})
```

#### CloudWatch Alarms

| Alarm | Threshold | Action |
|-------|-----------|--------|
| **HighCPU** | CPU > 80% for 3 consecutive minutes | Send SNS notification |
| **HighErrorRate** | Error rate > 5% | Send SNS notification |
| **HighLatency** | Latency > 3000ms | Send SNS notification |
| **DiskSpaceLow** | Disk > 85% | Send SNS notification |

#### SNS for Notifications

```python
import boto3

sns = boto3.client('sns')

# Publish notification
sns.publish(
    TopicArn='arn:aws:sns:ap-southeast-1:123456789:ocr-alerts',
    Subject='OCR App Alert: High CPU',
    Message='CPU utilization exceeded 80% on instance i-1234567890abcdef0'
)
```

---

### Weekly Task List

| Day | Task | Start | End | Resources |
|-----|------|-------|-----|-----------|
| Mon | Integrate CloudWatch Logs in Python backend | 06/09 | 06/09 | CloudWatch Logs |
| Tue | Create Log Groups and structured logs | 06/10 | 06/10 | Log Groups |
| Wed | Create CloudWatch Dashboard | 06/11 | 06/11 | Dashboard |
| Thu | Configure CloudWatch Alarms | 06/12 | 06/12 | CloudWatch Alarms |
| Fri | Set up SNS topic for notifications | 06/13 | 06/13 | AWS SNS |
| Sat | Install CloudWatch Agent for custom metrics | 06/14 | 06/14 | CloudWatch Agent |
| Sun | Structured logging implementation | 06/15 | 06/15 | Logging Best Practices |

---

### Achievements

- CloudWatch Dashboard provides real-time system health overview
- Received test email alerts when simulating high load
- Structured logs enable easy error tracing by request ID
- Understood observability: metrics, logs, and traces

### Monitoring Dashboard Widgets

| Widget | Metric | Refresh |
|--------|--------|---------|
| **CPU Usage** | CPUUtilization | 1 min |
| **Memory Usage** | MemUsage | 1 min |
| **Request Count** | API Requests | 1 min |
| **Error Rate** | 4xx/5xx Errors | 1 min |
| **Response Time** | Latency p99 | 1 min |

---

### Difficulties & Solutions

| Difficulty | Solution |
|------------|----------|
| CloudWatch Agent JSON config errors | Used Configuration Wizard |
| EC2 memory metrics not available | Installed CloudWatch Agent with IAM role |
| Too much log noise | Implemented log levels (INFO, WARNING, ERROR) |

### Best Practices

- Use **structured logging** (JSON format) for easier querying
- Set appropriate **alarm thresholds** based on baseline metrics
- Create **dashboards** for different personas (dev, ops, management)
- Use **Log Insights** for ad-hoc queries

---

### Next Week's Plan

- Research **Amazon Textract** for improved OCR accuracy
- Integrate **Amazon Translate** for multi-language support
- Compare self-built vs AWS managed services

---

### Reference Materials

- [Amazon CloudWatch Documentation](https://docs.aws.amazon.com/cloudwatch)
- [CloudWatch Agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/download-CloudWatch-Agent-on-EC2-Instance.html)
- [CloudWatch Logs Insights](https://docs.aws.amazon.com/CloudWatchLogs/latest/APIReference/LogInsights.html)
