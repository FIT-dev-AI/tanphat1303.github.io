---
title: "Workshop Overview"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# Workshop 5.1: Project Overview and Architecture

---

## Learning Objectives

By the end of this module, you will:

- Understand the business problem this platform solves
- Comprehend the end-to-end video localization workflow
- Identify all AWS services and third-party integrations
- Read and interpret the system architecture diagram
- Understand the data flow between components

---

## Prerequisites

- Basic understanding of cloud computing concepts
- Familiarity with REST APIs
- Basic knowledge of video formats and codecs
- Understanding of database concepts

---

## Architecture Context

### The Business Problem

Video content localization traditionally requires multiple manual steps:

1. **Subtitle Extraction**: Extracting text from video frames using OCR software
2. **Translation**: Converting text from source to target language
3. **Timing Synchronization**: Aligning translated text with video timestamps
4. **Voice Recording**: Recording or generating voice-over audio
5. **Final Rendering**: Merging all elements into a complete localized video

This manual process is time-consuming, error-prone, and does not scale when handling multiple videos.

### The Solution

The Video Localization Platform automates this workflow through a cloud-native architecture that handles video upload through final render. The system uses:

- **PaddleOCR** for optical character recognition from video frames
- **Google Cloud Speech-to-Text** as an alternative for audio-based subtitle extraction
- **Multi-Provider Translation Pipeline** (AWS Translate → Google Gemini → Google Cloud Translation v2) for resilient, context-aware translation
- **gTTS/ElevenLabs** for voice synthesis
- **FFmpeg** for video transcoding and rendering

---

## System Architecture

### High-Level Architecture Diagram

![System Architecture Overview](/images/5-Workshop/5.1-Workshop-overview/Kien-truc-he-thong-tong-quan.png)

### Component Overview

| Layer | Component | Technology | Purpose |
|-------|-----------|------------|---------|
| **Frontend** | Web Application | React 18, TypeScript, Vite | User interface for upload, editing, preview |
| **Frontend** | State Management | Zustand | Client-side state with persistence |
| **Frontend** | Real-time Updates | WebSocket | Live progress tracking |
| **Backend** | API Gateway | FastAPI | REST endpoints, authentication |
| **Backend** | Task Queue | Celery + Redis | Asynchronous processing |
| **Backend** | Database | MySQL (RDS) | Video metadata, user data |
| **Storage** | Object Storage | Amazon S3 | Video files, subtitles |
| **Storage** | Cache/Pub-Sub | Redis | Message broker, real-time events |
| **Processing** | OCR Engine | PaddleOCR | Text extraction from frames |
| **Processing** | Speech-to-Text | Google Cloud STT | Audio transcription |
| **Processing** | Translation Pipeline | AWS Translate (Primary) → Gemini → GCP Translate v2 | Resilient multi-provider translation |
| **Processing** | TTS | gTTS, ElevenLabs | Voice synthesis |
| **Processing** | Video Processing | FFmpeg, MoviePy | Transcoding, rendering |
| **Notification** | Email Service | Amazon SES | Completion notifications |

---

## Processing Pipeline

### Two-Stage Architecture

The platform uses a two-stage pipeline design that separates resource-intensive processing from user interaction:

![Two-Stage Pipeline](/images/5-Workshop/5.1-Workshop-overview/Luong-pipeline-2-giai-doan.png)

#### Stage 1: Extraction and Translation

```
Video Upload → Format Validation → OCR/STT Detection → Subtitle Extraction → Translation → SRT Upload
```

**Steps:**
1. User uploads video to S3
2. System validates video format using FFprobe
3. If format incompatible (HEVC, ProRes), auto-transcode to H.264/AAC
4. Detect whether to use OCR (embedded subtitles) or STT (audio only)
5. Extract subtitles using selected method
6. Translate subtitles using the multi-provider translation pipeline (AWS Translate → Gemini → GCP Translate v2)
7. Upload translated SRT to S3
8. Send email notification via SES

**Status Transition:**
```
PROCESSING → AWAITING_REVIEW
```

#### Stage 2: Rendering and Final Output

```
User Edits → Approval → TTS Generation → Video Merge → Final Render → Download
```

**Steps:**
1. User reviews and edits subtitles in browser
2. User selects voice for dubbing (optional)
3. User clicks "Approve" to trigger Stage 2
4. Generate TTS audio from translated text
5. Merge video with subtitles and/or audio
6. Upload final video to S3
7. Send completion email

**Status Transition:**
```
AWAITING_REVIEW → STAGE2_PROCESSING → COMPLETED
```

---

## AWS Services Deep Dive

### Amazon S3

The platform uses three separate S3 buckets for logical data separation:

| Bucket | Purpose | Content Type |
|--------|---------|--------------|
| `ocr-video-bucket-*` | Input videos | Original uploads |
| `video-sub-ft-*` | Output videos | Final rendered videos |
| `srt-input-storage-*` | Subtitle files | SRT files (source + translated) |

**Security:** All bucket access uses presigned URLs with expiration to prevent unauthorized access.

### Amazon RDS MySQL

The database stores:

- **Video records**: Metadata, status, processing percentage, file URLs
- **SRT records**: Subtitle file references, language pairs
- **VIDEO_TTS records**: Final output tracking, voice generation details
- **User records**: Authentication and authorization

**Key fields:**
- `video_id`: Unique identifier (UUID)
- `status`: Current processing state
- `processing_percentage`: Real-time progress
- `file_url`, `video_url`, `video_tts_url`: S3 URL references

### Amazon Translate

Amazon Translate is the **primary translation engine** in the multi-provider pipeline. It runs as a fully managed neural machine translation service inside the AWS account — no external API key, no egress outside the region, predictable per-character pricing.

**Role in the pipeline:**
- Primary provider for AWS-eligible language pairs (English ↔ Vietnamese, Korean ↔ Vietnamese, Japanese ↔ Vietnamese, Chinese ↔ Vietnamese, etc.)
- Returns a `SourceLanguageCode` for auto-detection results, used downstream to decide whether a segment needs Gemini fallback
- Per-segment invocation — failure is contained to a single subtitle line and triggers fallback rather than aborting the whole batch

**Integration:**
- Client built via `boto3.client('translate', region_name=settings.AWS_TRANSLATE_REGION)` in `app/utils/aws_translate.py`
- Single source of truth for primary translation calls; the STT orchestrator imports `translate_text` / `translate_batch`
- `Config(retries={'max_attempts': 1})` disables the hidden botocore retry — Tenacity owns the retry boundary (3 attempts, exponential backoff 1–10s)
- Retry policy is classification-driven (`provider_failure.classify_failure`): only TIMEOUT / RATE_LIMIT / QUOTA_EXCEEDED / SERVICE_UNAVAILABLE / INTERNAL_ERROR categories retry; AUTHENTICATION / PERMISSION / INVALID_REQUEST raise immediately
- Language code normalization converts BCP-47 codes (`en-US`, `cmn-Hans-CN`, `ko-KR`) to Amazon Translate's two-letter format before each call

**Why Amazon Translate first:**
- **Latency** — typical response time is in the low hundreds of milliseconds per segment, critical when translating 200+ subtitle lines per video
- **Cost predictability** — flat per-character pricing, no quota surprises
- **AWS-native** — no third-party credentials to manage, no data leaving the AWS network
- **Determinism** — reproducible output for the same input, which keeps subtitle diffs stable across re-runs

**Code reference:**
```python
from app.utils.aws_translate import translate_text, translate_batch
from app.utils.aws_translate import normalize_language_code

# Single segment
result = translate_text("Hello world", source_lang="auto", target_lang="vi")

# Batch (sequential — Amazon Translate has no native batch API)
results = translate_batch(["Hello", "World"], source_lang="en", target_lang="vi")
```

### Amazon SES

Email notifications sent at two points:

1. **Stage 1 completion**: Notifies user to review subtitles
2. **Stage 2 completion**: Notifies user video is ready for download

**Email content includes:**
- Video name
- Completion timestamp
- Status
- Download link (presigned URL)

### Redis

Dual role in the architecture:

1. **Message Broker**: Celery workers pull tasks from Redis queue
2. **Pub/Sub**: Real-time progress events broadcast to WebSocket clients

**Channels:**
- `progress:{video_id}`: Individual video progress updates
- `status:{video_id}`: Status change notifications

---

## Third-Party Service Integrations

### PaddleOCR

**Purpose:** Extract text from video frames (subtitles)

**Configuration:**
- Singleton pattern with model caching
- Language support: Chinese, Latin, Korean, Japanese, Arabic
- Default: Latin (English, Vietnamese)
- Optimized for 1 FPS sampling rate

**Code reference:**
```python
from app.modules.video_process import get_ocr_instance
ocr = get_ocr_instance(lang='latin')
```

### Google Cloud Speech-to-Text

**Purpose:** Transcribe audio when no embedded subtitles exist

**Use case:** Videos without hard-coded subtitles but with clear audio

**API:** `process_audio_to_subtitles()` in `module_speech_to_text.py`

### Translation Pipeline

The platform uses a **multi-provider translation pipeline** to maximize availability and quality. Each subtitle segment is routed through the providers in priority order; if one fails, the orchestrator transparently falls over to the next.

| Priority | Provider | Role | Why |
|----------|----------|------|-----|
| 1 (Primary) | **Amazon Translate** | First attempt | Low latency, native AWS integration, deterministic cost, predictable quality for short subtitle lines |
| 2 (Fallback 1) | **Google Gemini API** (`gemini-2.0-flash`) | Context-aware fallback | Handles nuanced segments and language pairs where Amazon Translate is ineligible (e.g. mixed-language lines) |
| 3 (Fallback 2) | **Google Cloud Translation v2** | Last-resort shared batch | Single batched `translate` call invoked once per STT batch — guarantees a result even when both above providers fail |

**Architecture highlights:**

- **Provider ownership contract** — Each adapter (`_call_amazon_translate`, `_call_gemini_translate`, `_call_google_translate_batch`) performs exactly one provider operation. No nested fallback inside an adapter. No silent source-text return.
- **Routing logic** — AWS-eligible segments try Amazon first, then Gemini. AWS-ineligible segments (mixed / unknown / null source) skip Amazon and go straight to Gemini. Segments failing both end up in the shared GCP v2 batch.
- **Retry policy (TASK 9)** — Per-provider retry is driven by `provider_failure.classify_failure`. Only retryable categories (TIMEOUT, RATE_LIMIT, QUOTA_EXCEEDED, SERVICE_UNAVAILABLE, INTERNAL_ERROR, etc.) are retried with exponential backoff. Auth/permission/invalid-request errors raise immediately. botocore's hidden retry is disabled via `Config(retries={'max_attempts': 1})` so the Tenacity layer is the single retry boundary.
- **Observability** — Each attempt is logged with `provider`, `status` (ATTEMPTED / SUCCESS / FAILED), latency and `attempted_providers` chain for full traceability.
- **Deterministic failure** — When all providers are exhausted, the orchestrator raises `TranslationFailure` (never returns the source text), so the caller can decide whether to retry the whole segment, skip it, or surface the error.

**Code references:**

```python
# Primary adapter
from app.utils.aws_translate import translate_text, translate_batch

# Orchestrator (chooses provider per segment)
from app.modules.module.module_speech_to_text import translate_stt_segment, translate_stt_segments
```

### gTTS and ElevenLabs

**Purpose:** Text-to-Speech for dubbing

| Provider | Quality | Cost | Use Case |
|----------|---------|------|----------|
| gTTS | Standard | Free | Default option |
| ElevenLabs | Natural | Paid | Premium voices |

**Code reference:**
```python
from app.modules.module.module_text_to_speech_v4_parallel import generate_audio_from_srt
```

---

## Real-Time Progress Tracking

### WebSocket Architecture

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Browser    │────▶│   FastAPI    │◀────│    Redis     │
│   (React)    │◀────│   Backend   │────▶│   Pub/Sub    │
└──────────────┘     └──────────────┘     └──────────────┘
       ▲                    │                    ▲
       │                    │                    │
       └────────────────────┴────────────────────┘
                    WebSocket
```

### Progress Data Structure

```typescript
interface ProgressData {
  video_id: string;
  percentage: number;    // 0-100
  status: VideoStatus;  // PROCESSING | AWAITING_REVIEW | etc.
  message: string;      // Human-readable status
  stage?: string;       // STAGE1_INIT | STAGE1_DOWNLOAD | etc.
  error_code?: string;   // For error states
}
```

### Frontend Integration

```typescript
// From useVideoProgress.ts
const ws = new WebSocket(`${WS_URL}/progress/${videoId}`);
ws.onmessage = (event) => {
  const progress: ProgressData = JSON.parse(event.data);
  updateProgress(progress);
};
```

---

## Video Format Handling

### Validation with FFprobe

Before processing, all videos are validated:

```python
from app.utils.video_validator import analyze_video, is_format_compatible

analysis = analyze_video(video_path)
is_compatible, reason = is_format_compatible(analysis)
```

### Supported Formats

| Status | Formats | Handling |
|--------|---------|----------|
| **Compatible** | MP4, MOV (H.264 + AAC) | Fast copy with optimization |
| **Requires Transcode** | HEVC, ProRes, VP9 | FFmpeg to H.264/AAC |
| **Unsupported** | Others | Error with guidance |

### Auto-Transcode Logic

```python
if not is_actually_compatible:
    transcode_to_mp4_h264(input_path, output_path, force_transcode=True)
```

---

## Security Considerations

### Authentication

- JWT tokens for API authentication
- Token expiration and refresh
- WebSocket authentication via query parameters

### Authorization

- User can only access their own videos
- S3 access via presigned URLs (time-limited)

### Data Protection

- Database credentials in environment variables
- API keys managed via AWS Secrets Manager (recommended)
- No sensitive data in logs

---

## Summary

In this module, you learned:

- **Business Context**: Why automated video localization solves real problems
- **Architecture Overview**: The two-stage pipeline design
- **AWS Services**: S3, RDS, SES, and their roles
- **Third-Party Integrations**: OCR, STT, Translation, TTS
- **Real-Time Updates**: WebSocket and Redis Pub/Sub
- **Video Processing**: FFprobe validation and FFmpeg transcoding

---

## Next Steps

Proceed to [Workshop 5.2: Development Environment Setup](5.2-Environment/) to configure your local development environment.

---

## References

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Celery Documentation](https://docs.celeryproject.org/)
- [PaddleOCR](https://github.com/PaddlePaddle/PaddleOCR)
- [Amazon Translate](https://aws.amazon.com/translate/)
- [Google Gemini API](https://ai.google.dev/)
- [Google Cloud Translation](https://cloud.google.com/translate)
- [FFmpeg Documentation](https://ffmpeg.org/documentation.html)
