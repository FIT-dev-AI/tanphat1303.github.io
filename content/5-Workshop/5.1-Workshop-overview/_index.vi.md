---
title: "Workshop Overview"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# Workshop 5.1: Tổng quan dự án và Kiến trúc

---

## Mục tiêu học tập

Sau khi hoàn thành module này, bạn sẽ:

- Hiểu bài toán kinh doanh mà nền tảng này giải quyết
- Nắm vững workflow dịch thuật và lồng tiếng video end-to-end
- Xác định tất cả các dịch vụ AWS và tích hợp bên thứ ba
- Đọc và diễn giải sơ đồ kiến trúc hệ thống
- Hiểu luồng dữ liệu giữa các thành phần

---

## Điều kiện tiên quyết

- Hiểu biết cơ bản về cloud computing
- Quen thuộc với REST APIs
- Kiến thức cơ bản về định dạng và codec video
- Hiểu về khái niệm database

---

## Kiến trúc hệ thống

### Sơ đồ Kiến trúc Tổng quan

![Kiến trúc hệ thống tổng quan](/images/5-Workshop/5.1-Workshop-overview/Kien-truc-he-thong-tong-quan.png)

### Tổng quan các thành phần

| Layer | Component | Technology | Purpose |
|-------|-----------|------------|---------|
| **Frontend** | Web Application | React 18, TypeScript, Vite | Giao diện người dùng cho upload, chỉnh sửa, xem trước |
| **Frontend** | State Management | Zustand | Trạng thái phía client với persistence |
| **Frontend** | Real-time Updates | WebSocket | Theo dõi tiến độ trực tiếp |
| **Backend** | API Gateway | FastAPI | REST endpoints, xác thực |
| **Backend** | Task Queue | Celery + Redis | Xử lý bất đồng bộ |
| **Backend** | Database | MySQL (RDS) | Video metadata, user data |
| **Storage** | Object Storage | Amazon S3 | Video files, subtitles |
| **Storage** | Cache/Pub-Sub | Redis | Message broker, real-time events |
| **Processing** | OCR Engine | PaddleOCR | Trích xuất text từ frames |
| **Processing** | Speech-to-Text | Google Cloud STT | Audio transcription |
| **Processing** | Translation Pipeline | AWS Translate (Primary) → Gemini → GCP Translate v2 | Bộ dịch đa nhà cung cấp có failover |
| **Processing** | TTS | gTTS, ElevenLabs | Tổng hợp giọng nói |
| **Processing** | Video Processing | FFmpeg, MoviePy | Transcoding, rendering |
| **Notification** | Email Service | Amazon SES | Thông báo hoàn thành |

---

## Processing Pipeline

### Kiến trúc hai giai đoạn

Nền tảng sử dụng thiết kế pipeline hai giai đoạn tách biệt xử lý tốn tài nguyên khỏi tương tác người dùng:

![Luồng pipeline 2 giai đoạn](/images/5-Workshop/5.1-Workshop-overview/Luong-pipeline-2-giai-doan.png)

#### Giai đoạn 1: Trích xuất và Dịch thuật

```
Video Upload → Format Validation → OCR/STT Detection → Subtitle Extraction → Translation → SRT Upload
```

**Các bước:**
1. Người dùng upload video lên S3
2. Hệ thống xác thực định dạng video sử dụng FFprobe
3. Nếu định dạng không tương thích (HEVC, ProRes), tự động transcode sang H.264/AAC
4. Phát hiện nên dùng OCR (phụ đề nhúng) hay STT (chỉ audio)
5. Trích xuất phụ đề sử dụng phương pháp được chọn
6. Dịch phụ đề thông qua Translation Pipeline đa nhà cung cấp (AWS Translate → Gemini → GCP Translate v2)
7. Upload SRT đã dịch lên S3
8. Gửi thông báo email qua SES

**Chuyển đổi trạng thái:**
```
PROCESSING → AWAITING_REVIEW
```

#### Giai đoạn 2: Rendering và Output cuối cùng

```
User Edits → Approval → TTS Generation → Video Merge → Final Render → Download
```

**Các bước:**
1. Người dùng xem lại và chỉnh sửa phụ đề trong trình duyệt
2. Người dùng chọn giọng cho dubbing (tùy chọn)
3. Người dùng click "Approve" để kích hoạt Giai đoạn 2
4. Tạo TTS audio từ text đã dịch
5. Ghép video với phụ đề và/hoặc audio
6. Upload video cuối cùng lên S3
7. Gửi thông báo hoàn thành

**Chuyển đổi trạng thái:**
```
AWAITING_REVIEW → STAGE2_PROCESSING → COMPLETED
```

---

## Chi tiết các dịch vụ AWS

### Amazon S3

Nền tảng sử dụng ba bucket S3 riêng biệt cho việc phân tách dữ liệu logic:

| Bucket | Mục đích | Loại nội dung |
|--------|-----------|--------------|
| `ocr-video-bucket-*` | Video đầu vào | Original uploads |
| `video-sub-ft-*` | Video đầu ra | Final rendered videos |
| `srt-input-storage-*` | File phụ đề | SRT files (source + translated) |

**Bảo mật:** Tất cả bucket access sử dụng presigned URLs với expiration để ngăn chặn truy cập trái phép.

### Amazon RDS MySQL

Database lưu trữ:

- **Video records**: Metadata, status, processing percentage, file URLs
- **SRT records**: Subtitle file references, language pairs
- **VIDEO_TTS records**: Final output tracking, voice generation details
- **User records**: Authentication và authorization

**Các trường quan trọng:**
- `video_id`: Unique identifier (UUID)
- `status`: Current processing state
- `processing_percentage`: Real-time progress
- `file_url`, `video_url`, `video_tts_url`: S3 URL references

### Amazon Translate

Amazon Translate là **translation engine chính** trong multi-provider pipeline. Đây là dịch vụ neural machine translation được quản lý hoàn toàn bởi AWS — không cần API key bên ngoài, không có dữ liệu ra khỏi region, giá theo ký tự xác định trước.

**Vai trò trong pipeline:**
- Primary provider cho các cặp ngôn ngữ AWS-eligible (Anh ↔ Việt, Hàn ↔ Việt, Nhật ↔ Việt, Trung ↔ Việt, …)
- Trả về `SourceLanguageCode` cho kết quả auto-detect, dùng để quyết định segment có cần Gemini fallback hay không
- Gọi theo từng segment — lỗi được cô lập trong một dòng phụ đề, kích hoạt fallback thay vì hủy cả batch

**Tích hợp:**
- Client khởi tạo qua `boto3.client('translate', region_name=settings.AWS_TRANSLATE_REGION)` trong `app/utils/aws_translate.py`
- Là nguồn duy nhất cho các call primary translation; STT orchestrator import `translate_text` / `translate_batch`
- `Config(retries={'max_attempts': 1})` disable botocore hidden retry — Tenacity sở hữu ranh giới retry (3 lần, exponential backoff 1–10s)
- Retry policy dựa trên phân loại lỗi (`provider_failure.classify_failure`): chỉ retry các category TIMEOUT / RATE_LIMIT / QUOTA_EXCEEDED / SERVICE_UNAVAILABLE / INTERNAL_ERROR; AUTHENTICATION / PERMISSION / INVALID_REQUEST raise ngay
- Normalize mã ngôn ngữ chuyển BCP-47 (`en-US`, `cmn-Hans-CN`, `ko-KR`) sang định dạng hai chữ cái của Amazon Translate trước mỗi call

**Vì sao ưu tiên Amazon Translate:**
- **Latency** — thời gian phản hồi thường chỉ vài trăm ms mỗi segment, cần thiết khi dịch 200+ dòng phụ đề mỗi video
- **Chi phí xác định** — giá flat theo ký tự, không có quota surprise
- **AWS-native** — không phải quản lý credentials bên thứ ba, dữ liệu không rời khỏi AWS network
- **Tính deterministic** — output reproducible cho cùng input, giữ subtitle diff ổn định qua các lần re-run

**Code tham khảo:**
```python
from app.utils.aws_translate import translate_text, translate_batch
from app.utils.aws_translate import normalize_language_code

# Một segment
result = translate_text("Hello world", source_lang="auto", target_lang="vi")

# Batch (tuần tự — Amazon Translate không có batch API native)
results = translate_batch(["Hello", "World"], source_lang="en", target_lang="vi")
```

### Amazon SES

Email notifications được gửi tại hai thời điểm:

1. **Stage 1 completion**: Thông báo cho user xem lại phụ đề
2. **Stage 2 completion**: Thông báo cho user video đã sẵn sàng tải về

**Email content bao gồm:**
- Video name
- Completion timestamp
- Status
- Download link (presigned URL)

### Redis

Vai trò kép trong kiến trúc:

1. **Message Broker**: Celery workers pull tasks từ Redis queue
2. **Pub/Sub**: Real-time progress events broadcast đến WebSocket clients

**Channels:**
- `progress:{video_id}`: Individual video progress updates
- `status:{video_id}`: Status change notifications

---

## Tích hợp dịch vụ bên thứ ba

### PaddleOCR

**Mục đích:** Trích xuất text từ video frames (phụ đề)

**Cấu hình:**
- Singleton pattern với model caching
- Hỗ trợ ngôn ngữ: Chinese, Latin, Korean, Japanese, Arabic
- Default: Latin (English, Vietnamese)
- Tối ưu cho tốc độ lấy mẫu 1 FPS

### Google Cloud Speech-to-Text

**Mục đích:** Phiên âm audio khi không có phụ đề nhúng

**Use case:** Videos không có phụ đề hard-coded nhưng có audio rõ ràng

### Translation Pipeline (Bộ dịch đa nhà cung cấp)

Hệ thống sử dụng **Translation Pipeline đa nhà cung cấp** để tối đa hóa độ khả dụng và chất lượng. Mỗi đoạn phụ đề được định tuyến qua các provider theo thứ tự ưu tiên; nếu một provider lỗi, orchestrator sẽ tự động failover sang provider tiếp theo.

| Ưu tiên | Provider | Vai trò | Lý do sử dụng |
|---------|----------|---------|----------------|
| 1 (Primary) | **Amazon Translate** | Lần thử đầu tiên | Độ trễ thấp, tích hợp native với AWS, chi phí xác định, chất lượng ổn định với các dòng phụ đề ngắn |
| 2 (Fallback 1) | **Google Gemini API** (`gemini-2.0-flash`) | Fallback nhận biết ngữ cảnh | Xử lý các đoạn phức tạp hoặc cặp ngôn ngữ mà Amazon Translate không đủ điều kiện (ví dụ dòng đa ngôn ngữ) |
| 3 (Fallback 2) | **Google Cloud Translation v2** | Last-resort shared batch | Một `translate` call dạng batch duy nhất cho cả STT batch — đảm bảo có kết quả khi cả hai provider trên đều thất bại |

**Các điểm kiến trúc quan trọng:**

- **Provider ownership contract** — Mỗi adapter (`_call_amazon_translate`, `_call_gemini_translate`, `_call_google_translate_batch`) chỉ thực hiện đúng một operation của provider. Không có nested fallback bên trong adapter. Không trả về source text khi lỗi.
- **Routing logic** — Segment AWS-eligible sẽ thử Amazon trước, rồi Gemini. Segment AWS-ineligible (mixed / unknown / null source) bỏ qua Amazon và đi thẳng tới Gemini. Nếu cả hai đều fail, index sẽ được gom vào shared GCP v2 batch.
- **Retry policy (TASK 9)** — Retry theo provider được điều khiển bởi `provider_failure.classify_failure`. Chỉ những category retryable (TIMEOUT, RATE_LIMIT, QUOTA_EXCEEDED, SERVICE_UNAVAILABLE, INTERNAL_ERROR, …) mới được retry với exponential backoff. Lỗi auth/permission/invalid-request raise ngay lập tức. botocore hidden retry bị disable qua `Config(retries={'max_attempts': 1})` để Tenacity là ranh giới retry duy nhất.
- **Observability** — Mỗi attempt được log kèm `provider`, `status` (ATTEMPTED / SUCCESS / FAILED), latency và chuỗi `attempted_providers` để truy vết đầy đủ.
- **Deterministic failure** — Khi tất cả provider đều cạn kiệt, orchestrator raise `TranslationFailure` (không bao giờ trả về source text), để caller quyết định retry cả segment, bỏ qua, hoặc surface lỗi lên UI.

**Code tham khảo:**

```python
# Primary adapter
from app.utils.aws_translate import translate_text, translate_batch

# Orchestrator (chọn provider theo từng segment)
from app.modules.module.module_speech_to_text import translate_stt_segment, translate_stt_segments
```

### gTTS và ElevenLabs

**Mục đích:** Text-to-Speech cho dubbing

| Provider | Chất lượng | Chi phí | Use Case |
|----------|-----------|---------|----------|
| gTTS | Standard | Miễn phí | Tùy chọn mặc định |
| ElevenLabs | Tự nhiên | Trả phí | Giọng premium |

---

## Theo dõi tiến độ Real-time

### Kiến trúc WebSocket

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

### Cấu trúc dữ liệu Progress

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

---

## Xử lý định dạng Video

### Validation với FFprobe

Trước khi xử lý, tất cả videos được xác thực:

```python
from app.utils.video_validator import analyze_video, is_format_compatible

analysis = analyze_video(video_path)
is_compatible, reason = is_format_compatible(analysis)
```

### Các định dạng được hỗ trợ

| Status | Formats | Xử lý |
|--------|---------|--------|
| **Tương thích** | MP4, MOV (H.264 + AAC) | Fast copy với optimization |
| **Cần Transcode** | HEVC, ProRes, VP9 | FFmpeg sang H.264/AAC |
| **Không hỗ trợ** | Others | Error với hướng dẫn |

---

## Bảo mật

### Xác thực

- JWT tokens cho API authentication
- Token expiration và refresh
- WebSocket authentication qua query parameters

### Ủy quyền

- User chỉ có thể truy cập videos của chính họ
- S3 access qua presigned URLs (time-limited)

### Bảo vệ dữ liệu

- Database credentials trong environment variables
- API keys quản lý qua AWS Secrets Manager (recommended)
- Không có sensitive data trong logs

---

## Tóm tắt

Trong module này, bạn đã học:

- **Bối cảnh kinh doanh**: Tại sao dịch thuật video tự động giải quyết vấn đề thực
- **Tổng quan kiến trúc**: Thiết kế pipeline hai giai đoạn
- **AWS Services**: S3, RDS, SES và vai trò của chúng
- **Tích hợp bên thứ ba**: OCR, STT, Translation, TTS
- **Real-time Updates**: WebSocket và Redis Pub/Sub
- **Xử lý Video**: FFprobe validation và FFmpeg transcoding

---

## Bước tiếp theo

Tiếp tục đến [Workshop 5.2: Thiết lập môi trường phát triển](5.2-Environment/) để cấu hình môi trường phát triển cục bộ.

---

## Tham khảo

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Celery Documentation](https://docs.celeryproject.org/)
- [PaddleOCR](https://github.com/PaddlePaddle/PaddleOCR)
- [Amazon Translate](https://aws.amazon.com/translate/)
- [Google Gemini API](https://ai.google.dev/)
- [Google Cloud Translation](https://cloud.google.com/translate)
- [FFmpeg Documentation](https://ffmpeg.org/documentation.html)
