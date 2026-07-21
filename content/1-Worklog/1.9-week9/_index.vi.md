---
title: "Worklog Tuần 9"
date: "2026-06-09"
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

## Tuần 9: CloudWatch Monitoring & Alerts

> **Thời gian:** 09/06 - 15/06/2026  
> **Chủ đề:** Observability & System Monitoring

### Mục tiêu học tập

- Thiết lập CloudWatch để monitor ứng dụng (logs và metrics)
- Tạo CloudWatch Dashboard với các metrics quan trọng
- Cấu hình CloudWatch Alarms để nhận notifications chủ động
- Implement structured logging để debug dễ dàng hơn

---

### Kiến thức AWS đã học

#### Amazon CloudWatch Components

| Thành phần | Mục đích | Data Retention |
|-----------|---------|----------------|
| **Metrics** | Dữ liệu số về AWS resources | 15 months |
| **Logs** | Application và system logs | Configurable (1 day - indefinite) |
| **Alarms** | Trigger notifications dựa trên metrics | Cho đến khi deleted |
| **Dashboards** | Visualizations tùy chỉnh | Cho đến khi deleted |

#### CloudWatch Metrics cho EC2

| Metric | Mô tả |
|--------|-------------|
| **CPUUtilization** | CPU usage percentage |
| **NetworkIn/Out** | Network traffic in bytes |
| **DiskRead/Write** | Disk I/O operations |
| **StatusCheckFailed** | Instance health status |

#### Custom Metrics với CloudWatch Agent

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
| **HighCPU** | CPU > 80% trong 3 phút liên tiếp | Gửi SNS notification |
| **HighErrorRate** | Error rate > 5% | Gửi SNS notification |
| **HighLatency** | Latency > 3000ms | Gửi SNS notification |
| **DiskSpaceLow** | Disk > 85% | Gửi SNS notification |

#### SNS cho Notifications

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

### Công việc trong tuần

| Thứ | Công việc | Bắt đầu | Kết thúc | Tài liệu |
|-----|-----------|---------|---------|-----------|
| 2 (T3) | Tích hợp CloudWatch Logs trong Python backend | 09/06 | 09/06 | CloudWatch Logs |
| 3 (T4) | Tạo Log Groups và structured logs | 10/06 | 10/06 | Log Groups |
| 4 (T5) | Tạo CloudWatch Dashboard | 11/06 | 11/06 | Dashboard |
| 5 (T6) | Cấu hình CloudWatch Alarms | 12/06 | 12/06 | CloudWatch Alarms |
| 6 (T7) | Thiết lập SNS topic cho notifications | 13/06 | 13/06 | AWS SNS |
| 7 (CN) | Cài CloudWatch Agent cho custom metrics | 14/06 | 14/06 | CloudWatch Agent |
| 8 (T2) | Structured logging implementation | 15/06 | 15/06 | Logging Best Practices |

---

### Kết quả đạt được

- CloudWatch Dashboard cung cấp real-time system health overview
- Nhận được test email alerts khi simulate high load
- Structured logs cho phép dễ dàng trace errors theo request ID
- Hiểu về observability: metrics, logs, và traces

### Monitoring Dashboard Widgets

| Widget | Metric | Refresh |
|--------|--------|---------|
| **CPU Usage** | CPUUtilization | 1 min |
| **Memory Usage** | MemUsage | 1 min |
| **Request Count** | API Requests | 1 min |
| **Error Rate** | 4xx/5xx Errors | 1 min |
| **Response Time** | Latency p99 | 1 min |

---

### Khó khăn & Giải pháp

| Khó khăn | Giải pháp |
|------------|----------|
| CloudWatch Agent JSON config errors | Dùng Configuration Wizard |
| EC2 memory metrics không có sẵn | Cài CloudWatch Agent với IAM role |
| Quá nhiều log noise | Implement log levels (INFO, WARNING, ERROR) |

### Best Practices

- Sử dụng **structured logging** (JSON format) để query dễ hơn
- Đặt **alarm thresholds** phù hợp dựa trên baseline metrics
- Tạo **dashboards** cho different personas (dev, ops, management)
- Sử dụng **Log Insights** cho ad-hoc queries

---

### Kế hoạch tuần tới

- Nghiên cứu **Amazon Textract** để cải thiện OCR accuracy
- Tích hợp **Amazon Translate** cho multi-language support
- So sánh self-built vs AWS managed services

---

### Tài liệu tham khảo

- [Amazon CloudWatch Documentation](https://docs.aws.amazon.com/cloudwatch)
- [CloudWatch Agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/download-CloudWatch-Agent-on-EC2-Instance.html)
- [CloudWatch Logs Insights](https://docs.aws.amazon.com/CloudWatchLogs/latest/APIReference/LogInsights.html)
