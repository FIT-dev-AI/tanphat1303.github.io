---
title: "Worklog Tuần 10"
date: "2026-06-16"
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

## Tuần 10: AWS AI Services - Textract & Translate

> **Thời gian:** 16/06 - 22/06/2026  
> **Chủ đề:** AI/ML Services Integration

### Mục tiêu học tập

- Tích hợp Amazon Textract để cải thiện OCR accuracy
- Implement Amazon Translate cho multi-language support
- Phân tích chi phí - lợi ích giữa self-built và AWS managed services
- Cập nhật frontend với tính năng chọn ngôn ngữ

---

### Kiến thức AWS đã học

#### Amazon Textract

| Tính năng | Mô tả |
|---------|-------------|
| **DetectDocumentText** | Trích xuất printed và handwritten text |
| **AnalyzeDocument** | Trích xuất text, tables, và forms |
| **AnalyzeExpense** | Trích xuất data từ receipts và invoices |
| **AnalyzeID** | Trích xuất data từ identity documents |
| **Async Operations** | Cho large documents |

#### Textract vs Open Source OCR

| Aspect | Amazon Textract | Tesseract/EasyOCR |
|--------|-----------------|-------------------|
| **Accuracy** | ~98% | ~85-90% |
| **Languages** | 100+ languages | Limited |
| **Tables/Forms** | Native support | Complex parsing |
| **Cost** | Pay-per-page | Free (self-hosted) |
| **Setup** | Managed service | Self-managed |

#### Textract API Usage

```python
import boto3

textract = boto3.client('textract', region_name='ap-southeast-1')

# Detect text from document
response = textract.detect_document_text(
    Document={
        'S3Object': {
            'Bucket': 'my-bucket',
            'Name': 'document.jpg'
        }
    }
)

# Extract text blocks
for block in response['Blocks']:
    if block['BlockType'] == 'LINE':
        print(f"{block['Confidence']:.2f}%: {block['Text']}")
```

#### Amazon Translate

| Tính năng | Mô tả |
|---------|-------------|
| **75+ Languages** | Broad language coverage |
| **Real-time** | Low latency translation |
| **Batch** | High-volume translation |
| **Custom Terms** | Industry-specific terminology |
| **Domain Adaptation** | Domain-specific models |

#### Translate API Usage

```python
import boto3

translate = boto3.client('translate', region_name='ap-southeast-1')

# Translate text
result = translate.translate_text(
    Text="Hello, how are you?",
    SourceLanguageCode="en",
    TargetLanguageCode="vi"
)

print(f"Original: {result['SourceLanguageCode']}")
print(f"Translated: {result['TranslatedText']}")
print(f"Target: {result['TargetLanguageCode']}")

# Batch translation
result = translate.translate_document(
    Document={
        'S3Object': {
            'Bucket': 'my-bucket',
            'Name': 'document.txt'
        }
    },
    TargetLanguageCode="vi"
)
```

#### So sánh Chi phí

| Service | Pricing | 1000 pages/tháng |
|---------|---------|------------------|
| **Textract** | $0.015/page (OCR) | $15.00 |
| **Translate** | $0.000015/character | ~$0.50 |
| **Self-hosted OCR** | EC2 + Storage | ~$30-50/month |

---

### Công việc trong tuần

| Thứ | Công việc | Bắt đầu | Kết thúc | Tài liệu |
|-----|-----------|---------|---------|-----------|
| 2 (T3) | Nghiên cứu Textract API features | 16/06 | 16/06 | AWS Textract |
| 3 (T4) | Tích hợp Textract vào backend | 17/06 | 17/06 | boto3 SDK |
| 4 (T5) | Nghiên cứu Translate API | 18/06 | 18/06 | AWS Translate |
| 5 (T6) | Implement Translate integration | 19/06 | 19/06 | Translation API |
| 6 (T7) | Benchmark: accuracy vs cost | 20/06 | 20/06 | Cost Analysis |
| 7 (CN) | Cập nhật frontend cho language selection | 21/06 | 21/06 | Frontend Dev |
| 8 (T2) | Testing và documentation | 22/06 | 22/06 | Testing |

---

### Kết quả đạt được

- Textract cải thiện OCR accuracy đáng kể
- Amazon Translate hỗ trợ 75+ ngôn ngữ
- Thời gian xử lý cải thiện ~40% cho ảnh độ phân giải cao
- Học cách quản lý chi phí cho AI/ML services

### Benchmark Results

| Metric | Open Source OCR | Amazon Textract |
|--------|-----------------|-----------------|
| Accuracy | 87% | 98% |
| Processing Time | 3.2s | 1.8s |
| Cost/1000 pages | $0 | $15 |
| Language Support | 5 | 100+ |

---

### Khó khăn & Giải pháp

| Khó khăn | Giải pháp |
|------------|----------|
| Textract pricing concerns | Estimated usage, chỉ dùng cho necessary cases |
| Complex response parsing | Viết helper functions cho extraction |
| Vietnamese character encoding | Đảm bảo UTF-8 throughout pipeline |

### Cost Optimization Tips

- Sử dụng **Async Textract** cho large documents (rẻ hơn)
- Cache **common translations** để giảm API calls
- Sử dụng **batch operations** khi có thể
- Thiết lập **billing alerts** để monitor spend

---

### Kế hoạch tuần tới

- Viết comprehensive **technical documentation**
- Thực hiện **security review** với Security Hub
- Tối ưu chi phí sử dụng **Cost Explorer**
- Chuẩn bị **midterm report** cho mentor

---

### Tài liệu tham khảo

- [Amazon Textract Documentation](https://docs.aws.amazon.com/textract)
- [Amazon Translate Documentation](https://docs.aws.amazon.com/translate)
- [AWS AI Services Pricing](https://aws.amazon.com/textract/pricing)
