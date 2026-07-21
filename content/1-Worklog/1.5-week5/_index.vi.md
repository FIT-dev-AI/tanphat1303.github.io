---
title: "Worklog Tuần 5"
date: "2026-05-12"
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

## Tuần 5: Triển khai Frontend & CloudFront CDN

> **Thời gian:** 12/05 - 18/05/2026  
> **Chủ đề:** Static Website Hosting & Content Delivery

### Mục tiêu học tập

- Deploy React frontend lên S3 với static website hosting
- Cấu hình CloudFront CDN cho việc phân phối nội dung toàn cầu
- Thiết lập CORS cho cross-origin API requests
- Kết nối frontend với backend API đã deploy

---

### Kiến thức AWS đã học

#### Amazon S3 Static Website Hosting

| Tính năng | Mô tả |
|---------|-------------|
| **Static Hosting** | Host HTML, CSS, JS không cần web server |
| **Index Document** | Trang mặc định (thường là index.html) |
| **Error Document** | Trang lỗi 404 tùy chỉnh |
| **Routing** | Hỗ trợ Single Page Applications (SPA) |

#### Cấu hình S3 cho Static Website

```json
{
  "IndexDocument": {
    "Suffix": "index.html"
  },
  "ErrorDocument": {
    "Key": "error.html"
  },
  "RoutingRules": [
    {
      "RedirectRule": {
        "ReplaceKeyPrefixWith": ""
      },
      "Condition": {
        "HttpErrorCodeReturnedEquals": "404"
      }
    }
  ]
}
```

#### Amazon CloudFront CDN

| Tính năng | Lợi ích |
|---------|---------|
| **Global Edge Network** | 450+ edge locations toàn cầu |
| **SSL/TLS Encryption** | HTTPS cho tất cả nội dung |
| **Cache Control** | Giảm tải origin, cải thiện performance |
| **Invalidation** | Xóa cached content khi cần |
| **Price Class** | Tối ưu chi phí bằng cách chọn edge locations |

#### Kiến trúc CloudFront + S3

```
User Request
     │
     ▼
┌──────────────────┐
│  CloudFront CDN  │──Cache Hit──▶ Return Cached Content
└────────┬─────────┘
         │ Cache Miss
         ▼
┌──────────────────┐
│  S3 Bucket       │──▶ Return Static Content
│  (Origin)       │
└──────────────────┘
```

---

### Công việc trong tuần

| Thứ | Công việc | Bắt đầu | Kết thúc | Tài liệu |
|-----|-----------|---------|---------|-----------|
| 2 (T3) | Build React frontend cho production | 12/05 | 12/05 | React Build |
| 3 (T4) | Tạo S3 bucket với static hosting | 13/05 | 13/05 | AWS S3 |
| 4 (T5) | Upload build artifacts, tạo CloudFront | 14/05 | 14/05 | CloudFront |
| 5 (T6) | Cấu hình CORS trên S3 và backend | 15/05 | 15/05 | CORS Config |
| 6 (T7) | Cập nhật frontend API endpoint | 16/05 | 16/05 | Environment |
| 7 (CN) | End-to-end testing | 17/05 | 17/05 | E2E Testing |

---

### Kết quả đạt được

- Frontend accessible qua CloudFront URL với HTTPS
- Thời gian tải trang cải thiện đáng kể với CloudFront caching
- Luồng hoàn chỉnh hoạt động: Upload API Translation Display
- Hiểu cách cấu hình CORS cho cross-origin requests

### Performance Metrics

| Metric | Không CDN | Với CloudFront |
|--------|-------------|-----------------|
| TTFB | ~800ms | ~50ms |
| Cache Hit Ratio | 0% | ~95% |
| Global Availability | Chậm ở APAC | Nhanh toàn cầu |

---

### Khó khăn & Giải pháp

| Khó khăn | Giải pháp |
|------------|----------|
| Lỗi CORS khi gọi API | Thêm CORS headers vào FastAPI và S3 |
| CloudFront cache outdate | Dùng invalidation (/*) sau khi deploy |
| React build environment variables | Tạo .env.production trước khi build |

### Best Practices

- Luôn **invalidate cache** sau khi deploy phiên bản mới
- Sử dụng **environment variables** cho API endpoints
- Đặt **Cache-Control headers** phù hợp

---

### Kế hoạch tuần tới

- Tìm hiểu sâu hơn về **Amazon VPC** networking
- Deploy EC2 backend trong private subnet
- Thiết lập Application Load Balancer (ALB)
- Implement VPC security best practices

---

### Tài liệu tham khảo

- [S3 Static Website Hosting](https://docs.aws.amazon.com/AmazonS3/latest/userguide/HostingWebsiteOnS3Setup.html)
- [CloudFront Documentation](https://docs.aws.amazon.com/cloudfront)
- [CORS Configuration](https://docs.aws.amazon.com/AmazonS3/latest/userguide/cors.html)
