---
title: "Proposal"
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
  <p class="subtitle">Automated Video Translation & Dubbing System</p>
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
    <div class="metric-value">5-15 minutes</div>
    <div class="metric-label">Processing time/video</div>
  </div>
  <div class="metric-card">
    <div class="metric-icon primary">
      <i class="fas fa-dollar-sign"></i>
    </div>
    <div class="metric-value">$17-30</div>
    <div class="metric-label">Monthly cost</div>
  </div>
  <div class="metric-card">
    <div class="metric-icon warning">
      <i class="fas fa-tachometer-alt"></i>
    </div>
    <div class="metric-value">90%+</div>
    <div class="metric-label">Cost savings</div>
  </div>
  <div class="metric-card">
    <div class="metric-icon info">
      <i class="fas fa-calendar-check"></i>
    </div>
    <div class="metric-value">14 weeks</div>
    <div class="metric-label">Implementation time</div>
  </div>
</div>

<div class="proposal-content">

## 1. Executive Summary

Video Localization Platform is a cloud-based system designed to automate the video translation and dubbing workflow for multilingual content creators. The system leverages serverless architecture on AWS combined with asynchronous task processing through Celery and Redis.

The platform accepts videos uploaded via web interface, automatically extracts subtitles using PaddleOCR (for videos with embedded text) or Google Cloud Speech-to-Text (for audio-only content), translates extracted content using Google Gemini API, and generates dubbed audio using gTTS or ElevenLabs.

### Core Technologies

| Technology | Description |
|------------|-------------|
| AWS Serverless | Scalable and cost-efficient infrastructure |
| Celery + Redis | Asynchronous task processing |
| PaddleOCR | Intelligent subtitle extraction |
| Google Gemini | Multi-language translation |
| ElevenLabs | Natural voice synthesis |
| FFmpeg | Professional video rendering |

---

## 2. Problem Statement

### Current Challenges

| Challenge | Impact |
|-----------|--------|
| Manual subtitle extraction | Time-consuming eye-tracking process |
| Sentence-by-sentence translation | 4-8 hours per video |
| Timing synchronization | Complex manual work |
| Professional dubbing costs | $100-500 per minute |
| Multiple video processing | Does not scale |

### Proposed Solution

An integrated, end-to-end pipeline processing from video upload to final rendering:

1. **PaddleOCR** — Intelligent subtitle extraction from videos with existing subtitles
2. **Google Cloud STT** — Speech recognition for subtitle-less videos
3. **Google Gemini** — Context-aware translation
4. **ElevenLabs TTS** — Natural-sounding voice synthesis for dubbing
5. **FFmpeg Processing** — Professional video rendering

### Benefits & ROI

| Metric | Traditional | Our Platform |
|--------|-------------|--------------|
| Processing Time | 4-8 hours/video | **5-15 minutes/video** |
| Monthly Cost | $500-2000 | **$17-30** |
| Translation Consistency | Variable | Guaranteed |
| Break-even Period | Never | **1-2 months** |

---

## 3. Solution Architecture

The platform follows a microservices-inspired architecture with clear separation between frontend, backend API, asynchronous workers, and storage services.

![Video Localization Platform Architecture](/images/2-Proposal/aws_demo.jpg)

### AWS Services

| Service | Purpose |
|---------|---------|
| **Amazon S3** | Input videos, output videos, subtitle files (.srt) — 3 dedicated buckets |
| **Amazon RDS MySQL** | Video info, users, processing status |
| **Amazon SES** | Email notifications on processing completion |
| **AWS Region** | ap-southeast-1 (Singapore) |

### Third-Party Services

| Service | Function |
|---------|----------|
| **PaddleOCR** | Optical character recognition for subtitle extraction |
| **Google Cloud STT** | Speech recognition for videos without subtitles |
| **Google Gemini** | Context-aware translation |
| **ElevenLabs** | Natural voice synthesis for dubbing |
| **FFmpeg** | Format validation and video rendering |

### Component Design

- **Frontend**: React 18 + TypeScript + Vite — video upload, subtitle editor (auto-save), video preview
- **Backend**: FastAPI + Celery — REST API, async tasks, WebSocket real-time updates
- **Pipeline**: 2-stage processing (OCR/STT → Translation → TTS → Video Render)
- **Storage**: S3 (3 buckets by function) + Redis (caching & pub/sub)

---

## 4. Technical Implementation

### Implementation Phases

| Phase | Duration | Deliverables |
|-------|----------|--------------|
| **Platform** | 4 weeks | Project setup, infrastructure, basic upload |
| **Stage 1** | 6 weeks | OCR/STT, translation, progress tracking |
| **Stage 2** | 4 weeks | TTS, video rendering, subtitle editor |
| **Finalization** | 2 weeks | Error handling, optimization, documentation |

### Technical Stack

| Layer | Technologies |
|-------|--------------|
| **Frontend** | React 18, TypeScript, Vite, Zustand, TailwindCSS, Radix UI |
| **Backend** | Python 3.11+, FastAPI, Celery, Redis, SQLAlchemy, Pydantic |
| **OCR/STT** | PaddleOCR, Google Cloud Speech-to-Text API |
| **Translation** | Google Gemini API |
| **TTS** | gTTS, ElevenLabs API |
| **Video** | FFmpeg, MoviePy |
| **Infrastructure** | AWS S3, RDS MySQL, SES, Docker, Redis |

---

## 5. Budget Estimation

### AWS Monthly Costs (Development Tier)

| Service | Usage | Monthly Cost |
|---------|-------|--------------|
| Amazon S3 | 50 GB storage, 10 GB transfer | $1.50 |
| Amazon RDS MySQL | db.t3.micro, 20 GB storage | $15.00 |
| Amazon SES | 62,000 free emails | $0.00 |
| Data Transfer | 10 GB outbound | $0.90 |
| **Total AWS** | | **$17.40** |

### Third-Party API Costs

| Service | Free Tier | Paid Usage |
|---------|-----------|------------|
| Google Cloud STT | 60 minutes/month | $1.50/15 minutes |
| Google Gemini | 15 requests/minute | $0.001/1K chars |
| ElevenLabs | 10,000 chars/month | $5.00/100K chars |
| gTTS | Unlimited | $0.00 |

**Estimated Total Monthly**: $20-30 depending on usage volume.

---

## 6. Risk Assessment

### Risk Matrix

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Gemini API rate limiting | High | Medium | Exponential backoff, queue requests |
| Poor OCR with complex fonts | High | Low | STT fallback, manual editing allowed |
| Incompatible video formats | Medium | High | FFprobe validation, auto-transcode |
| Celery worker failure | Medium | Low | Redis persistence, automatic retry |
| Cost overrun from API usage | Medium | Medium | Usage monitoring, budget alerts |

### Contingency Plans

- **Primary API failure**: Automatic fallback to secondary provider
- **Worker crash**: Tasks remain queued until worker recovery
- **Database unavailable**: Return cached status, queue writes

---

## 7. Expected Outcomes

### Technical Deliverables

- Web application for video upload and subtitle editing
- Automatic subtitle extraction using OCR or STT
- Context-aware translation via Gemini API
- Voice synthesis using gTTS or ElevenLabs
- Final video rendering with subtitles and/or dubbed audio
- Real-time progress tracking via WebSocket
- Email notifications on completion

### Output Formats

- Videos with soft subtitle overlay
- Videos with hard subtitles (burned into video stream)
- Videos with dubbed audio
- SRT subtitle files in both source and target languages
- Presigned download links for completed content

### Practical Value

- **Multilingual Scalability** — Expand video content reach globally
- **Cost Reduction** — 90%+ savings vs. traditional services
- **Workflow Integration** — Seamless fit into content production pipelines
- **Consistency** — Terminology uniformity across all videos

</div>
