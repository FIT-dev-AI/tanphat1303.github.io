---
title: "Backend Development"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

# Workshop 5.3: Backend Development

---

## Mục tiêu học tập

Sau khi hoàn thành module này, bạn sẽ:

- Hiểu cấu trúc ứng dụng FastAPI
- Implement REST API endpoints cho video management
- Configure Celery cho asynchronous task processing
- Set up Redis Pub/Sub cho real-time updates
- Design database models với SQLAlchemy
- Implement WebSocket endpoints cho progress tracking
- Understand JWT authentication flow
- Nhận biết cách backend dispatch translation work cho **multi-provider pipeline** dẫn đầu bởi Amazon Translate

---

## Điều kiện tiên quyết

- Hoàn thành Workshops 5.1 và 5.2
- FastAPI framework understanding
- Basic SQLAlchemy knowledge
- Celery basics

---

## Architecture Context

Backend đóng vai trò orchestration layer giữa frontend và processing workers:

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Frontend   │────▶│   FastAPI    │────▶│     S3       │
│   (React)    │◀────│   Backend   │◀────│   Buckets    │
└──────────────┘     └──────┬───────┘     └──────────────┘
       ▲                    │
       │ WebSocket          │ Celery Tasks
       │                    ▼
┌──────────────┐     ┌──────────────┐
│    Redis     │◀────│   Celery     │
│   Pub/Sub    │     │   Workers    │
└──────────────┘     └──────────────┘
```

---

## Step 1: Application Entry Point

```python
# app/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.api.v1.api import api_router

app = FastAPI(
    title="Video Localization API",
    description="API for automated video translation and dubbing",
    version="1.0.0"
)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(api_router, prefix="/api/v1")
```

---

## Step 2: API Router Structure

API được tổ chức theo hierarchy:

```
app/api/v1/
├── api.py          # Main router
└── endpoints/
    ├── auth.py     # Authentication endpoints
    ├── user.py     # User management
    └── video.py    # Video operations
```

![API Endpoints](/images/5-Workshop/5.3-Backend/API_Endpoints_Chi_tiet.png)

### Main Router Configuration

```python
# app/api/v1/api.py
from fastapi import APIRouter
from app.api.v1.endpoints import auth, user, video

api_router = APIRouter()

api_router.include_router(auth.router, prefix="/auth", tags=["Authentication"])
api_router.include_router(user.router, prefix="/users", tags=["Users"])
api_router.include_router(video.router, prefix="/videos", tags=["Videos"])
```

---

## Step 3: Video API Endpoints

### Upload Video

```python
# app/api/v1/endpoints/video.py
@router.post("/upload", response_model=VideoResponse)
async def upload_video(
    file: UploadFile = File(...),
    db: Session = Depends(get_db)
):
    """Upload a new video for processing."""
    
    if not file.content_type.startswith("video/"):
        raise HTTPException(status_code=400, detail="File must be a video")
    
    # Upload to S3
    s3_key = await video_service.upload_video_to_s3(file)
    
    # Create database record
    video_data = VideoCreate(
        title=file.filename,
        file_name=file.filename,
        file_url=f"s3://{s3_key}",
        status="PROCESSING"
    )
    video = await video_service.create_video(db, video_data)
    
    # Dispatch Celery task
    process_video_stage_1.delay(
        video_id=str(video.video_id),
        s3_key=s3_key,
        original_filename=file.filename
    )
    
    return video
```

### Upload Video Sequence Flow

![Sequence Upload Video](/images/5-Workshop/5.3-Backend/Sequence_Upload_Video.png)

---

## Step 3: Database Models

### Video Model

```python
# app/models/video.py
class Video(Base):
    __tablename__ = "videos"
    
    id = Column(Integer, primary_key=True, index=True)
    video_id = Column(String(36), unique=True, index=True, default=lambda: str(uuid.uuid4()))
    
    # User relationship
    user_id = Column(Integer, ForeignKey("users.id"))
    user = relationship("User", back_populates="videos")
    
    # File information
    title = Column(String(255))
    file_name = Column(String(255))
    display_name = Column(String(255))
    file_url = Column(Text)
    video_url = Column(Text)
    video_tts_url = Column(Text)
    
    # Processing status
    status = Column(String(50), default="PROCESSING")
    processing_percentage = Column(Integer, default=0)
    
    # Subtitle information
    srt_original = Column(Text)
    srt_translated = Column(Text)
    srt_translated_url = Column(Text)
    
    # Voice configuration
    voice_used = Column(String(100))
    
    # Timestamps
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
```

---

## Step 4: Celery Tasks

```python
# app/tasks/video_tasks.py
@celery_app.task(
    bind=True,
    name="process_video_stage_1",
    autoretry_for=(Exception,),
    retry_kwargs={'max_retries': 3, 'countdown': 60},
    retry_backoff=True,
    retry_backoff_max=600,
    retry_jitter=True
)
def process_video_stage_1(self, video_id: str, s3_key: str, original_filename: str):
    """
    Stage 1: OCR + Translation
    """
    # Download video from S3
    report_progress(video_id, 10, "PROCESSING", "Downloading video...")
    
    # Validate and transcode if needed
    report_progress(video_id, 20, "PROCESSING", "Validating video format...")
    
    # Extract subtitles
    report_progress(video_id, 50, "PROCESSING", "Extracting subtitles...")
    
    # Translate subtitles
    report_progress(video_id, 80, "PROCESSING", "Translating subtitles...")
    
    # Update database
    report_progress(video_id, 100, "AWAITING_REVIEW", "Processing complete!")
```

### Celery Task Flow

![Celery Flow](/images/5-Workshop/5.3-Backend/Luong_Celery.png)

### Error Handling & Retry Mechanism

![Error Handling & Retry](/images/5-Workshop/5.3-Backend/Error_Handling_Retry.png)

---

## Step 5: Redis Pub/Sub Progress Reporter

```python
# app/utils/progress_reporter.py
class ProgressReporter:
    def __init__(self):
        self.redis_client = redis.Redis(host='localhost', port=6379, db=0)
        self.channel_prefix = "progress"
    
    def publish(self, video_id: str, percentage: int, status: str, 
                message: str, stage: Optional[str] = None):
        channel = f"{self.channel_prefix}:{video_id}"
        data = {
            "video_id": video_id,
            "percentage": percentage,
            "status": status,
            "message": message,
        }
        if stage:
            data["stage"] = stage
        
        self.redis_client.publish(channel, json.dumps(data))

progress_reporter = ProgressReporter()

def report_progress(video_id: str, percentage: int, status: str, message: str, stage: Optional[str] = None):
    progress_reporter.publish(video_id, percentage, status, message, stage)
```

---

## Step 6: WebSocket Implementation

```python
# app/websocket/progress.py
manager = ConnectionManager()

@router.websocket("/ws/progress/{video_id}")
async def websocket_progress(websocket: WebSocket, video_id: str, token: str = None):
    """WebSocket endpoint for real-time progress updates."""
    
    if token:
        try:
            verify_token(token)
        except:
            await websocket.close(code=4001)
            return
    
    await manager.connect(websocket, video_id)
    
    try:
        while True:
            data = await websocket.receive_text()
            if data == "ping":
                await websocket.send_text("pong")
    except WebSocketDisconnect:
        manager.disconnect(websocket, video_id)
```

---

## Step 7: JWT Authentication

```python
# app/core/security.py
def create_access_token(data: dict, expires_delta: timedelta = None):
    to_encode = data.copy()
    expire = datetime.utcnow() + (expires_delta or timedelta(minutes=30))
    to_encode.update({"exp": expire})
    
    encoded_jwt = jwt.encode(
        to_encode, 
        settings.SECRET_KEY, 
        algorithm=settings.ALGORITHM
    )
    return encoded_jwt

def verify_token(token: str):
    try:
        payload = jwt.decode(token, settings.SECRET_KEY, algorithms=[settings.ALGORITHM])
        return payload
    except JWTError:
        raise HTTPException(status_code=401, detail="Invalid token")
```

---

## Code References

| Component | File | Purpose |
|-----------|------|---------|
| Main App | `app/main.py` | FastAPI configuration |
| Video API | `app/api/v1/endpoints/video.py` | Video CRUD operations |
| Models | `app/models/video.py` | Database schema |
| Tasks | `app/tasks/video_tasks.py` | Async processing (OCR + translation + SRT) |
| Translation Adapter | `app/utils/aws_translate.py` | Amazon Translate primary client (`translate_text`, `translate_batch`) |
| Orchestrator | `app/modules/module/module_speech_to_text.py` | Multi-provider router: Amazon → Gemini → GCP v2 |
| Retry Classifier | `app/utils/provider_failure.py` | Tenacity-driven retry boundary với category classification |
| Progress | `app/utils/progress_reporter.py` | Redis Pub/Sub |
| WebSocket | `app/websocket/progress.py` | Real-time updates |

> **Lưu ý về translation:** Stage 1 dispatch `process_video_stage_1` sẽ gọi **multi-provider translation pipeline** (Amazon Translate primary → Gemini fallback → GCP v2 last resort). Mọi call đến Amazon Translate đều đi qua `app/utils/aws_translate.py`, dùng chung AWS credentials với S3/SES — không cần API key riêng. Xem chi tiết tại [Workshop 5.4](5.4-AI-Pipeline/).

---

## Tóm tắt

Trong module này, bạn đã học:

- **FastAPI Structure**: Cách ứng dụng được tổ chức
- **REST API Design**: CRUD operations cho video management
- **Celery Integration**: Asynchronous task processing
- **Database Models**: SQLAlchemy schema cho videos và subtitles
- **Redis Pub/Sub**: Real-time progress broadcasting
- **WebSocket**: Bidirectional communication với frontend
- **JWT Authentication**: Secure API access
- **Translation dispatch**: Celery `process_video_stage_1` delegate mỗi subtitle segment cho `app/utils/aws_translate.py` (Amazon Translate primary) với Gemini và GCP v2 làm fallback tự động — xem chi tiết tại [Workshop 5.4](5.4-AI-Pipeline/).

---

## Bước tiếp theo

Tiếp tục đến [Workshop 5.4: AI/ML Pipeline](5.4-AI-Pipeline/) để hiểu cách subtitle extraction, translation, và TTS được implement.
