---
title: "Week 10 Worklog"
date: "2026-06-16"
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

## Week 10: AWS AI Services - Textract & Translate

> **Duration:** June 16 - 22, 2026  
> **Theme:** AI/ML Services Integration

### 📚 Learning Objectives

- Integrate Amazon Textract for improved OCR accuracy
- Implement Amazon Translate for multi-language support
- Perform cost-benefit analysis between self-built and AWS managed services
- Update frontend with language selection feature

---

### 🎯 Key AWS Knowledge Acquired

#### Amazon Textract

| Feature | Description |
|---------|-------------|
| **DetectDocumentText** | Extracts printed and handwritten text |
| **AnalyzeDocument** | Extracts text, tables, and forms |
| **AnalyzeExpense** | Extracts data from receipts and invoices |
| **AnalyzeID** | Extracts data from identity documents |
| **Async Operations** | For large documents |

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

| Feature | Description |
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

#### Cost Comparison

| Service | Pricing | 1000 pages/month |
|---------|---------|------------------|
| **Textract** | $0.015/page (OCR) | $15.00 |
| **Translate** | $0.000015/character | ~$0.50 |
| **Self-hosted OCR** | EC2 + Storage | ~$30-50/month |

---

### 📋 Weekly Task List

| Day | Task | Start | End | Resources |
|-----|------|-------|-----|-----------|
| Mon | Research Textract API features | 06/16 | 06/16 | AWS Textract |
| Tue | Integrate Textract into backend | 06/17 | 06/17 | boto3 SDK |
| Wed | Research Translate API | 06/18 | 06/18 | AWS Translate |
| Thu | Implement Translate integration | 06/19 | 06/19 | Translation API |
| Fri | Benchmark: accuracy vs cost | 06/20 | 06/20 | Cost Analysis |
| Sat | Update frontend for language selection | 06/21 | 06/21 | Frontend Dev |
| Sun | Testing and documentation | 06/22 | 06/22 | Testing |

---

### ✅ Achievements

- Textract improved OCR accuracy significantly
- Amazon Translate supports 75+ languages
- Processing time improved ~40% for high-res images
- Learned cost management for AI/ML services

### 📊 Benchmark Results

| Metric | Open Source OCR | Amazon Textract |
|--------|-----------------|-----------------|
| Accuracy | 87% | 98% |
| Processing Time | 3.2s | 1.8s |
| Cost/1000 pages | $0 | $15 |
| Language Support | 5 | 100+ |

---

### 🚧 Difficulties & Solutions

| Difficulty | Solution |
|------------|----------|
| Textract pricing concerns | Estimated usage, used only for necessary cases |
| Complex response parsing | Wrote helper functions for extraction |
| Vietnamese character encoding | Ensured UTF-8 throughout pipeline |

### 💡 Cost Optimization Tips

- Use **Async Textract** for large documents (cheaper)
- Cache **common translations** to reduce API calls
- Use **batch operations** when possible
- Set up **billing alerts** to monitor spend

---

### 📅 Next Week's Plan

- Write comprehensive **technical documentation**
- Perform **security review** with Security Hub
- Optimize costs using **Cost Explorer**
- Prepare **midterm report** for mentor

---

### 📖 Reference Materials

- [Amazon Textract Documentation](https://docs.aws.amazon.com/textract)
- [Amazon Translate Documentation](https://docs.aws.amazon.com/translate)
- [AWS AI Services Pricing](https://aws.amazon.com/textract/pricing)
