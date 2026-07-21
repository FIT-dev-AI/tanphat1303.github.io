---
title: "Backend Development"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

# Workshop 5.3: Backend Development

---

## Learning Objectives

By the end of this module, you will:

- Understand the FastAPI application structure
- Implement REST API endpoints for video management
- Configure Celery for asynchronous task processing
- Set up Redis Pub/Sub for real-time updates
- Design database models with SQLAlchemy
- Implement WebSocket endpoints for progress tracking
- Understand JWT authentication flow

---

## Prerequisites

- Workshop 5.1 and 5.2 completed
- FastAPI framework understanding
- Basic SQLAlchemy knowledge
- Celery basics

---

## Architecture Context

The backend serves as the orchestration layer between the frontend and processing workers:

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

The FastAPI application is configured in `app/main.py`:

```python
# app/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.api.v1.api import api_router
from app.core.database import engine
from app.models import video, user  # Import to register models

app = FastAPI(
    title="Video Localization API",
    description="API for automated video translation and dubbing",
    version="1.0.0"
)

# CORS middleware for frontend access
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173"],  # Vite dev server
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Include API routes
app.include_router(api_router, prefix="/api/v1")
```

---

## Step 2: API Router Structure

The API is organized with a router hierarchy:

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

The video endpoints handle the core functionality:

### Upload Video

```python
# app/api/v1/endpoints/video.py
from fastapi import APIRouter, UploadFile, File, Depends, HTTPException
from sqlalchemy.orm import Session
from app.core.database import get_db
from app.schemas.video import VideoCreate, VideoResponse
from app.service import video_service
from app.tasks.video_tasks import process_video_stage_1

router = APIRouter()

@router.post("/upload", response_model=VideoResponse)
async def upload_video(
    file: UploadFile = File(...),
    db: Session = Depends(get_db)
):
    """
    Upload a new video for processing.
    
    The video is stored in S3 and a Celery task is dispatched
    for Stage 1 processing (OCR/STT + Translation).
    """
    # Validate file type
    if not file.content_type.startswith("video/"):
        raise HTTPException(
            status_code=400,
            detail="File must be a video"
        )
    
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

### Get Video Status

```python
@router.get("/{video_id}", response_model=VideoResponse)
async def get_video(
    video_id: str,
    db: Session = Depends(get_db)
):
    """Get video details and current processing status."""
    video = await video_service.get_video(db, video_id)
    if not video:
        raise HTTPException(status_code=404, detail="Video not found")
    return video
```

### List User Videos

```python
@router.get("/", response_model=list[VideoResponse])
async def list_videos(
    skip: int = 0,
    limit: int = 10,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    """List all videos for the authenticated user."""
    videos = await video_service.get_user_videos(
        db, user_id=current_user.id, skip=skip, limit=limit
    )
    return videos
```

---

## Step 4: Database Models

### Video Model

```python
# app/models/video.py
from sqlalchemy import Column, String, Integer, DateTime, ForeignKey, Text
from sqlalchemy.orm import relationship
from datetime import datetime
from app.core.database import Base

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
    display_name = Column(String(255))  # Original filename
    file_url = Column(Text)  # S3 URL for original
    video_url = Column(Text)  # S3 URL for processed
    video_tts_url = Column(Text)  # S3 URL for final (with TTS)
    
    # Processing status
    status = Column(String(50), default="PROCESSING")
    processing_percentage = Column(Integer, default=0)
    
    # Subtitle information
    srt_original = Column(Text)
    srt_translated = Column(Text)
    srt_translated_url = Column(Text)
    
    # Voice configuration
    voice_used = Column(String(100))
    
    # Thumbnail
    thumbnail_url = Column(Text)
    thumbnail_s3_key = Column(String(255))
    
    # Timestamps
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    
    # Relationships
    srt_records = relationship("SRT", back_populates="video", cascade="all, delete-orphan")
    tts_records = relationship("VIDEO_TTS", back_populates="video", cascade="all, delete-orphan")
```

### SRT Model

```python
# app/models/video.py (continued)
class SRT(Base):
    __tablename__ = "srt_records"
    
    id = Column(Integer, primary_key=True, index=True)
    video_id = Column(Integer, ForeignKey("videos.id"))
    
    # Language pair
    source_language = Column(String(10))
    target_language = Column(String(10))
    
    # File references
    srt_original_s3_key = Column(String(255))
    srt_translated_s3_key = Column(String(255))
    
    # Timestamps
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    
    # Relationship
    video = relationship("Video", back_populates="srt_records")
```

---

## Step 5: Celery Configuration

### Celery App Setup

```python
# celery.py
from celery import Celery
import os

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "app.core.config")

celery_app = Celery(
    "video_localization",
    broker=os.getenv("CELERY_BROKER_URL", "redis://localhost:6379/0"),
    backend=os.getenv("CELERY_RESULT_BACKEND", "redis://localhost:6379/0"),
    include=["app.tasks.video_tasks"]
)

celery_app.conf.update(
    task_serializer="json",
    accept_content=["json"],
    result_serializer="json",
    timezone="UTC",
    enable_utc=True,
    task_track_started=True,
    task_time_limit=3600,  # 1 hour max
    task_soft_time_limit=3300,  # 55 minutes soft limit
)
```

### Video Processing Tasks

```python
# app/tasks/video_tasks.py
from celery import shared_task
from celery.signals import task_prerun, task_postrun

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
    
    Steps:
    1. Download video from S3
    2. Validate format with FFprobe
    3. Transcode if necessary
    4. Extract subtitles (OCR or STT)
    5. Translate subtitles
    6. Upload SRT to S3
    7. Update database status
    8. Send notification
    """
    from app.core.database import SessionLocal
    from app.modules.video_process import extract_subtitles, translate_srt
    from app.utils.progress_reporter import report_progress
    
    db = SessionLocal()
    
    try:
        # Report initial progress
        report_progress(video_id, 5, "PROCESSING", "Starting video processing...")
        
        # Download video from S3
        report_progress(video_id, 10, "PROCESSING", "Downloading video...")
        video_path = download_video_from_s3(s3_key)
        
        # Validate and transcode if needed
        report_progress(video_id, 15, "PROCESSING", "Validating video format...")
        video_analysis = analyze_video(video_path)
        
        if not is_format_compatible(video_analysis):
            report_progress(video_id, 20, "PROCESSING", "Transcoding video...")
            video_path = transcode_to_mp4_h264(video_path)
        
        # Extract subtitles
        report_progress(video_id, 40, "PROCESSING", "Extracting subtitles...")
        subtitles = extract_subtitles(video_path)
        
        # Translate subtitles
        report_progress(video_id, 70, "PROCESSING", "Translating subtitles...")
        translated = translate_srt(subtitles, target_lang='vi')
        
        # Upload to S3
        report_progress(video_id, 85, "PROCESSING", "Uploading results...")
        srt_url = upload_srt_to_s3(translated)
        
        # Update database
        report_progress(video_id, 95, "PROCESSING", "Updating database...")
        update_video_status(db, video_id, status="AWAITING_REVIEW", srt_url=srt_url)
        
        # Send notification
        report_progress(video_id, 100, "AWAITING_REVIEW", "Processing complete!")
        
        send_completion_email(video_id)
        
    finally:
        db.close()
```

### Celery Task Flow

![Celery Flow](/images/5-Workshop/5.3-Backend/Luong_Celery.png)

### Error Handling & Retry Mechanism

![Error Handling & Retry](/images/5-Workshop/5.3-Backend/Error_Handling_Retry.png)

---

## Step 6: Redis Pub/Sub Progress Reporter

### Progress Reporter Implementation

```python
# app/utils/progress_reporter.py
import redis
import json
from typing import Optional

class ProgressReporter:
    def __init__(self):
        self.redis_client = redis.Redis(host='localhost', port=6379, db=0)
        self.channel_prefix = "progress"
    
    def publish(self, video_id: str, percentage: int, status: str, 
                message: str, stage: Optional[str] = None,
                error_code: Optional[str] = None):
        """Publish progress update to Redis Pub/Sub."""
        channel = f"{self.channel_prefix}:{video_id}"
        
        data = {
            "video_id": video_id,
            "percentage": percentage,
            "status": status,
            "message": message,
        }
        
        if stage:
            data["stage"] = stage
        if error_code:
            data["error_code"] = error_code
        
        self.redis_client.publish(channel, json.dumps(data))

# Singleton instance
progress_reporter = ProgressReporter()

def report_progress(video_id: str, percentage: int, status: str,
                   message: str, stage: Optional[str] = None,
                   error_code: Optional[str] = None):
    """Convenience function for reporting progress."""
    progress_reporter.publish(video_id, percentage, status, message, stage, error_code)
```

---

## Step 7: WebSocket Implementation

### WebSocket Endpoint

```python
# app/websocket/progress.py
from fastapi import WebSocket, WebSocketDisconnect
from app.core.socket_manager import ConnectionManager
from app.core.security import verify_token

manager = ConnectionManager()

@router.websocket("/ws/progress/{video_id}")
async def websocket_progress(websocket: WebSocket, video_id: str, token: str = None):
    """WebSocket endpoint for real-time progress updates."""
    
    # Optional authentication
    if token:
        try:
            verify_token(token)
        except:
            await websocket.close(code=4001)
            return
    
    await manager.connect(websocket, video_id)
    
    try:
        while True:
            # Keep connection alive
            data = await websocket.receive_text()
            # Echo back for ping/pong
            if data == "ping":
                await websocket.send_text("pong")
    except WebSocketDisconnect:
        manager.disconnect(websocket, video_id)
```

### Connection Manager

```python
# app/core/socket_manager.py
from fastapi import WebSocket
from typing import Dict, Set
import asyncio

class ConnectionManager:
    def __init__(self):
        # video_id -> set of websocket connections
        self.active_connections: Dict[str, Set[WebSocket]] = {}
        self.redis_subscriber = None
    
    async def connect(self, websocket: WebSocket, video_id: str):
        await websocket.accept()
        
        if video_id not in self.active_connections:
            self.active_connections[video_id] = set()
            # Subscribe to Redis channel for this video
            self._subscribe_to_redis(video_id)
        
        self.active_connections[video_id].add(websocket)
    
    def disconnect(self, websocket: WebSocket, video_id: str):
        if video_id in self.active_connections:
            self.active_connections[video_id].discard(websocket)
            if not self.active_connections[video_id]:
                del self.active_connections[video_id]
                self._unsubscribe_from_redis(video_id)
    
    async def send_to_video(self, video_id: str, message: str):
        """Send message to all connections watching a video."""
        if video_id in self.active_connections:
            dead_connections = []
            for connection in self.active_connections[video_id]:
                try:
                    await connection.send_text(message)
                except:
                    dead_connections.append(connection)
            
            # Clean up dead connections
            for conn in dead_connections:
                self.disconnect(conn, video_id)
```

---

## Step 8: Authentication

### JWT Token Generation

```python
# app/core/security.py
from datetime import datetime, timedelta
from jose import JWTError, jwt
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def create_access_token(data: dict, expires_delta: timedelta = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=30)
    
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

### Authenticated Endpoint

```python
# app/api/v1/endpoints/auth.py
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from app.core.security import verify_token
from app.models.user import User

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/api/v1/auth/login")

async def get_current_user(token: str = Depends(oauth2_scheme)) -> User:
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    
    try:
        payload = verify_token(token)
        user_id: int = payload.get("sub")
        if user_id is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception
    
    user = db.query(User).filter(User.id == user_id).first()
    if user is None:
        raise credentials_exception
    return user
```

---

## Code References

| Component | File | Purpose |
|-----------|------|---------|
| Main App | `app/main.py` | FastAPI configuration |
| Router | `app/api/v1/api.py` | Route organization |
| Video API | `app/api/v1/endpoints/video.py` | Video CRUD operations |
| Models | `app/models/video.py` | Database schema |
| Celery Config | `celery.py` | Task queue setup |
| Tasks | `app/tasks/video_tasks.py` | Async processing |
| Progress | `app/utils/progress_reporter.py` | Redis Pub/Sub |
| WebSocket | `app/websocket/progress.py` | Real-time updates |
| Security | `app/core/security.py` | JWT authentication |

---

## Summary

In this module, you learned:

- **FastAPI Structure**: How the application is organized
- **REST API Design**: CRUD operations for video management
- **Celery Integration**: Asynchronous task processing
- **Database Models**: SQLAlchemy schema for videos and subtitles
- **Redis Pub/Sub**: Real-time progress broadcasting
- **WebSocket**: Bidirectional communication with frontend
- **JWT Authentication**: Secure API access

---

## Next Steps

Proceed to [Workshop 5.4: AI/ML Pipeline](5.4-AI-Pipeline/) to understand how subtitle extraction, translation, and TTS are implemented.

---

## Validation

Test the API endpoints:

```bash
# Upload a video
curl -X POST http://localhost:8000/api/v1/videos/upload \
  -H "Authorization: Bearer <token>" \
  -F "file=@test_video.mp4"

# Check video status
curl http://localhost:8000/api/v1/videos/<video_id> \
  -H "Authorization: Bearer <token>"
```

Monitor Celery worker logs to see task processing.
