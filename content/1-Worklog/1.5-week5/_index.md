---
title: "Week 5 Worklog"
date: "2026-05-12"
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

## Week 5: Frontend Deployment & CloudFront CDN

> **Duration:** May 12 - 18, 2026  
> **Theme:** Static Website Hosting & Content Delivery

### Learning Objectives

- Deploy React frontend to S3 as static website hosting
- Configure CloudFront CDN for global content delivery
- Set up CORS for cross-origin API requests
- Connect frontend with deployed backend API

---

### Key AWS Knowledge Acquired

#### Amazon S3 Static Website Hosting

| Feature | Description |
|---------|-------------|
| **Static Hosting** | Host HTML, CSS, JS without a web server |
| **Index Document** | Default page (usually index.html) |
| **Error Document** | Custom 404 error page |
| **Routing** | Support for Single Page Applications (SPA) |

#### S3 Configuration for Static Website

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

| Feature | Benefit |
|---------|---------|
| **Global Edge Network** | 450+ edge locations worldwide |
| **SSL/TLS Encryption** | HTTPS for all content |
| **Cache Control** | Reduce origin load, improve performance |
| **Invalidation** | Remove cached content when needed |
| **Price Class** | Optimize cost by selecting edge locations |

#### CloudFront + S3 Architecture

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

### Weekly Task List

| Day | Task | Start | End | Resources |
|-----|------|-------|-----|-----------|
| Mon | Build React frontend for production | 05/12 | 05/12 | React Build |
| Tue | Create S3 bucket with static hosting | 05/13 | 05/13 | AWS S3 |
| Wed | Upload build artifacts, create CloudFront | 05/14 | 05/14 | CloudFront |
| Thu | Configure CORS on S3 and backend | 05/15 | 05/15 | CORS Config |
| Fri | Update frontend API endpoint | 05/16 | 05/16 | Environment |
| Sat | End-to-end testing | 05/17 | 05/17 | E2E Testing |

---

### Achievements

- Frontend accessible via CloudFront URL with HTTPS
- Page load time improved significantly with CloudFront caching
- Complete flow working: Upload API Translation Display
- Understood CORS configuration for cross-origin requests

### Performance Metrics

| Metric | Without CDN | With CloudFront |
|--------|-------------|-----------------|
| TTFB | ~800ms | ~50ms |
| Cache Hit Ratio | 0% | ~95% |
| Global Availability | Slow in APAC | Fast globally |

---

### Difficulties & Solutions

| Difficulty | Solution |
|------------|----------|
| CORS errors when calling API | Added CORS headers to FastAPI and S3 |
| CloudFront cache outdated | Used invalidation (/*) after deploy |
| React build environment variables | Created .env.production before build |

### Best Practices

- Always **invalidate cache** after deploying new version
- Use **environment variables** for API endpoints
- Set **Cache-Control headers** appropriately

---

### Next Week's Plan

- Deep dive into **Amazon VPC** networking
- Deploy EC2 backend in private subnet
- Set up Application Load Balancer (ALB)
- Implement VPC security best practices

---

### Reference Materials

- [S3 Static Website Hosting](https://docs.aws.amazon.com/AmazonS3/latest/userguide/HostingWebsiteOnS3Setup.html)
- [CloudFront Documentation](https://docs.aws.amazon.com/cloudfront)
- [CORS Configuration](https://docs.aws.amazon.com/AmazonS3/latest/userguide/cors.html)
