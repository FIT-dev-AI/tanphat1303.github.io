---
title: "Bản đề xuất"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
layout: proposal
---

<div class="proposal-header">
  <div class="proposal-badge">
    <i class="fas fa-rocket"></i>
    <span>AWS Cloud Project</span>
  </div>
  <h1>Video Localization Platform</h1>
  <p class="subtitle">Hệ thống Tự động Hóa Dịch Thuật & Lồng Tiếng Video</p>
  <div class="proposal-tags">
    <span class="tag"><i class="fas fa-cloud"></i> AWS Serverless</span>
    <span class="tag"><i class="fas fa-brain"></i> AI-Powered</span>
    <span class="tag"><i class="fas fa-video"></i> Video Processing</span>
  </div>
</div>

<div class="proposal-metrics">
  <div class="metric-card">
    <div class="metric-icon success">
      <i class="fas fa-clock"></i>
    </div>
    <div class="metric-value">5-15 phút</div>
    <div class="metric-label">Thời gian xử lý/video</div>
  </div>
  <div class="metric-card">
    <div class="metric-icon primary">
      <i class="fas fa-dollar-sign"></i>
    </div>
    <div class="metric-value">$17-30</div>
    <div class="metric-label">Chi phí hàng tháng</div>
  </div>
  <div class="metric-card">
    <div class="metric-icon warning">
      <i class="fas fa-tachometer-alt"></i>
    </div>
    <div class="metric-value">90%+</div>
    <div class="metric-label">Tiết kiệm chi phí</div>
  </div>
  <div class="metric-card">
    <div class="metric-icon info">
      <i class="fas fa-calendar-check"></i>
    </div>
    <div class="metric-value">14 tuần</div>
    <div class="metric-label">Thời gian triển khai</div>
  </div>
</div>

<div class="proposal-content">

## 1. Tóm tắt điều hành

Video Localization Platform là hệ thống dựa trên đám mây, được thiết kế để tự động hóa quy trình dịch thuật và lồng tiếng video cho những người tạo nội dung đa ngôn ngữ. Hệ thống tận dụng kiến trúc serverless trên AWS kết hợp với xử lý tác vụ bất đồng bộ thông qua Celery và Redis.

Nền tảng chấp nhận video được upload qua giao diện web, tự động trích xuất phụ đề sử dụng PaddleOCR (cho video có text nhúng) hoặc Google Cloud Speech-to-Text (cho nội dung chỉ có âm thanh), dịch nội dung trích xuất bằng Google Gemini API, và tạo audio lồng tiếng sử dụng gTTS hoặc ElevenLabs.

### Công nghệ cốt lõi

| Công nghệ | Mô tả |
|-----------|--------|
| AWS Serverless | Hạ tầng có thể mở rộng và tiết kiệm chi phí |
| Celery + Redis | Xử lý tác vụ bất đồng bộ |
| PaddleOCR | Trích xuất phụ đề thông minh |
| Google Gemini | Dịch thuật đa ngôn ngữ |
| ElevenLabs | Tổng hợp giọng nói tự nhiên |
| FFmpeg | Render video chuyên nghiệp |

---

## 2. Tuyên bố vấn đề

### Những thách thức hiện tại

| Thách thức | Tác động |
|------------|----------|
| Trích xuất phụ đề thủ công | Quy trình theo dõi bằng mắt tốn thời gian |
| Dịch thuật từng câu | 4-8 giờ mỗi video |
| Đồng bộ thời gian | Công việc thủ công phức tạp |
| Chi phí lồng tiếng | $100-500 mỗi phút |
| Xử lý nhiều video | Không thể mở rộng |

### Giải pháp đề xuất

Pipeline tích hợp, end-to-end xử lý từ upload video đến render cuối cùng:

1. **PaddleOCR** — Trích xuất phụ đề thông minh từ video có sẵn
2. **Google Cloud STT** — Nhận dạng giọng nói cho video không có phụ đề
3. **Google Gemini** — Dịch thuật nhận biết ngữ cảnh
4. **ElevenLabs TTS** — Tổng hợp giọng nói tự nhiên cho lồng tiếng
5. **FFmpeg Processing** — Render video chuyên nghiệp

### Lợi ích và ROI

| Chỉ số | Truyền thống | Nền tảng của chúng tôi |
|--------|--------------|-------------------------|
| Thời gian xử lý | 4-8 giờ/video | **5-15 phút/video** |
| Chi phí hàng tháng | $500-2000 | **$17-30** |
| Tính nhất quán dịch | Biến đổi | Đảm bảo |
| Thời gian hoàn vốn | Không bao giờ | **1-2 tháng** |

---

## 3. Kiến trúc giải pháp

Nền tảng tuân theo kiến trúc lấy cảm hứng từ microservices với sự phân tách rõ ràng giữa frontend, backend API, workers bất đồng bộ, và các dịch vụ lưu trữ.

![Video Localization Platform Architecture](/images/2-Proposal/aws_demo.jpg)

### Các dịch vụ AWS được sử dụng

| Dịch vụ | Mục đích |
|---------|----------|
| **Amazon S3** | Video đầu vào, video đầu ra, file phụ đề (.srt) — 3 bucket riêng biệt |
| **Amazon RDS MySQL** | Thông tin video, người dùng, trạng thái xử lý |
| **Amazon SES** | Thông báo email khi xử lý hoàn thành |
| **AWS Region** | ap-southeast-1 (Singapore) |

### Các dịch vụ bên thứ ba

| Dịch vụ | Chức năng |
|---------|----------|
| **PaddleOCR** | Nhận dạng ký tự quang học để trích xuất phụ đề |
| **Google Cloud STT** | Nhận dạng giọng nói cho video không có phụ đề |
| **Google Gemini** | Dịch thuật nhận biết ngữ cảnh |
| **ElevenLabs** | Tổng hợp giọng nói tự nhiên cho lồng tiếng |
| **FFmpeg** | Xác thực định dạng và render video |

### Thiết kế thành phần

- **Frontend**: React 18 + TypeScript + Vite — upload video, trình chỉnh sửa phụ đề (auto-save), xem trước video
- **Backend**: FastAPI + Celery — REST API, tác vụ bất đồng bộ, WebSocket cập nhật thời gian thực
- **Pipeline**: 2 giai đoạn xử lý (OCR/STT → Dịch thuật → TTS → Render Video)
- **Storage**: S3 (3 bucket theo chức năng) + Redis (cache & pub/sub)

---

## 4. Triển khai kỹ thuật

### Các giai đoạn triển khai

| Giai đoạn | Thời gian | Deliverables |
|-----------|-----------|--------------|
| **Nền tảng** | 4 tuần | Project setup, hạ tầng, basic upload |
| **Stage 1** | 6 tuần | OCR/STT, dịch thuật, theo dõi tiến độ |
| **Stage 2** | 4 tuần | TTS, video rendering, trình chỉnh sửa phụ đề |
| **Hoàn thiện** | 2 tuần | Xử lý lỗi, tối ưu, tài liệu |

### Tech Stack

| Lớp | Công nghệ |
|-----|-----------|
| **Frontend** | React 18, TypeScript, Vite, Zustand, TailwindCSS, Radix UI |
| **Backend** | Python 3.11+, FastAPI, Celery, Redis, SQLAlchemy, Pydantic |
| **OCR/STT** | PaddleOCR, Google Cloud Speech-to-Text API |
| **Dịch thuật** | Google Gemini API |
| **TTS** | gTTS, ElevenLabs API |
| **Video** | FFmpeg, MoviePy |
| **Hạ tầng** | AWS S3, RDS MySQL, SES, Docker, Redis |

---

## 5. Ước tính ngân sách

### Chi phí hàng tháng AWS (Development Tier)

| Dịch vụ | Sử dụng | Chi phí |
|---------|---------|---------|
| Amazon S3 | 50 GB storage, 10 GB transfer | $1.50 |
| Amazon RDS MySQL | db.t3.micro, 20 GB storage | $15.00 |
| Amazon SES | 62,000 emails đầu tiên miễn phí | $0.00 |
| Data Transfer | 10 GB outbound | $0.90 |
| **Tổng AWS** | | **$17.40** |

### Chi phí API bên thứ ba

| Dịch vụ | Free Tier | Sử dụng trả phí |
|---------|-----------|-----------------|
| Google Cloud STT | 60 phút/tháng | $1.50/15 phút |
| Google Gemini | 15 requests/phút | $0.001/1K ký tự |
| ElevenLabs | 10,000 ký tự/tháng | $5.00/100K ký tự |
| gTTS | Unlimited | $0.00 |

**Tổng ước tính hàng tháng**: $20-30 tùy thuộc vào dung lượng sử dụng.

---

## 6. Đánh giá rủi ro

### Ma trận rủi ro

| Rủi ro | Ảnh hưởng | Xác suất | Giảm thiểu |
|--------|-----------|-----------|------------|
| Gemini API rate limiting | Cao | Trung bình | Exponential backoff, queue requests |
| OCR quality trên phông phức tạp | Cao | Thấp | Fallback sang STT, cho phép sửa thủ công |
| Video format không tương thích | Trung bình | Cao | FFprobe validation, auto-transcode |
| Celery worker failure | Trung bình | Thấp | Redis persistence, automatic retry |
| Cost overrun từ API usage | Trung bình | Trung bình | Usage monitoring, budget alerts |

### Kế hoạch dự phòng

- **Primary API failure**: Automatic fallback sang secondary provider
- **Worker crash**: Các tác vụ vẫn queued cho đến khi worker recovery
- **Database unavailable**: Return cached status, queue writes

---

## 7. Kết quả kỳ vọng

### Deliverables kỹ thuật

- Ứng dụng web cho video upload và chỉnh sửa phụ đề
- Trích xuất phụ đề tự động sử dụng OCR hoặc STT
- Dịch thuật nhận biết ngữ cảnh qua Gemini API
- Tổng hợp giọng nói sử dụng gTTS hoặc ElevenLabs
- Video rendering cuối cùng với phụ đề và/hoặc audio dubbed
- Theo dõi tiến độ thời gian thực qua WebSocket
- Thông báo email khi hoàn thành

### Các định dạng đầu ra

- Videos với soft subtitle overlay
- Videos với hard subtitles (burned vào video stream)
- Videos với dubbed audio
- File phụ đề SRT ở cả ngôn ngữ nguồn và đích
- Presigned download links cho nội dung đã hoàn thành

### Giá trị thực tiễn

- **Khả năng mở rộng đa ngôn ngữ** — Mở rộng phạm vi tiếp cận nội dung video toàn cầu
- **Giảm chi phí** — Tiết kiệm 90%+ so với dịch vụ truyền thống
- **Tích hợp quy trình** — Phù hợp với quy trình sản xuất nội dung
- **Tính nhất quán** — Đồng nhất thuật ngữ trên tất cả video

</div>
