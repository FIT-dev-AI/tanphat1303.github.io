---
title: "Development Environment Setup"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# Workshop 5.2: Thiết lập Môi trường Phát triển

---

## Mục tiêu học tập

Sau khi hoàn thành module này, bạn sẽ:

- Thiết lập môi trường phát triển hoàn chỉnh cục bộ
- Cấu hình Docker containers cho Redis và MySQL
- Clone và cấu hình các repositories của dự án
- Hiểu cấu trúc thư mục
- Chạy ứng dụng ở chế độ phát triển
- Xác minh tất cả các services đang giao tiếp đúng

---

## Điều kiện tiên quyết

- Python 3.11 hoặc cao hơn đã cài đặt
- Node.js 18 hoặc cao hơn đã cài đặt
- Docker Desktop đã cài đặt và đang chạy
- Git đã cài đặt
- Code editor (VS Code được khuyến nghị)

---

## Bối cảnh kiến trúc

Trước khi thiết lập, hiểu cách các components tương tác trong môi trường local development:

### Sơ đồ Kiến trúc Docker

![Kiến trúc Docker](/images/5-Workshop/5.2-Environment/Docker_Architecture.png)

### Chi tiết Kiến trúc Local Development

{{< mermaid >}}
graph TB
    subgraph Development["🐳 Docker Container"]
        subgraph Data["Data Layer"]
            Redis["🟢 Redis<br/>:6379"]
            MySQL["🔵 MySQL<br/>:3306"]
        end
    end

    subgraph Backend["⚙️ Backend (ocr-api)"]
        API["🐍 FastAPI<br/>:8000"]
        Celery["⚡ Celery<br/>Worker"]
        WS["🔌 WebSocket<br/>:8000"]
    end

    subgraph Frontend["🎨 Frontend (myapporc)"]
        React["⚛️ React<br/>:5173"]
    end

    React -->|"HTTP/WS| :5173 → :8000"| API
    API -->|"Publish Tasks"| Redis
    Celery -->|"Subscribe| :6379"| Redis
    API -->|"SQL Queries| :3306"| MySQL
    Celery -->|"Results"| Redis
    API -->|"Pub/Sub"| WS
    React -->|"Real-time| WS"| WS

    style Redis fill:#dc382d,stroke:#333,stroke-width:2px,color:#fff
    style MySQL fill:#00758f,stroke:#333,stroke-width:2px,color:#fff
    style API fill:#009688,stroke:#333,stroke-width:2px,color:#fff
    style React fill:#61dafb,stroke:#333,stroke-width:2px
{{< /mermaid >}}

---

### Sơ đồ Ports & Services Summary

![Ports & Services Summary](/images/5-Workshop/5.2-Environment/Ports_Services_Summary.png)

### Bảng Ports

| Service | Port | Protocol | Description |
|---------|------|----------|-------------|
| **Frontend (Vite)** | 5173 | HTTP | React dev server |
| **Backend (FastAPI)** | 8000 | HTTP/WS | REST API & WebSocket |
| **Swagger UI** | 8000/docs | HTTP | API documentation |
| **Redis** | 6379 | TCP | Message broker |
| **MySQL** | 3306 | TCP | Database |

---

## Bước 1: Clone Project

### Backend Repository

```bash
cd /path/to/your/workspace
git clone <repository-url> ocr-api
cd ocr-api
```

### Frontend Repository

```bash
cd /path/to/your/workspace
git clone <repository-url> myapporc
cd myapporc
```

---

## Bước 2: Tạo Environment Files

### Backend Environment

Tạo file `.env` trong thư mục `ocr-api`:

```bash
cd ocr-api
touch .env
```

Thêm các cấu hình sau:

```bash
# Database Configuration
DATABASE_URL=mysql+pymysql://admin:Admin1303@localhost:3306/db_sub_video

# AWS Configuration
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
AWS_BUCKET_INPUT_VIDEO=ocr-video-bucket-2004
AWS_BUCKET_VIDEO_SUB=video-sub-ft-2004
AWS_BUCKET_INPUT_SRT=srt-input-storage-ft-2004
AWS_REGION=ap-southeast-2

# API Keys
# Google Gemini API — dùng làm fallback translation provider
API_KEY=your_gemini_api_key
API_MODEL=gemini-2.0-flash

# AWS Translate — primary translation provider (dùng chung AWS_ACCESS_KEY_ID / AWS_SECRET_ACCESS_KEY phía trên)
# AWS_TRANSLATE_REGION mặc định lấy theo AWS_REGION nếu không được set
AWS_TRANSLATE_REGION=ap-southeast-2

# Google Cloud Translation v2 — last-resort fallback batch provider (tùy chọn)
# GOOGLE_APPLICATION_CREDENTIALS=/path/to/gcp-service-account.json

# JWT Configuration
SECRET_KEY=your_secret_key_min_32_chars
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# Redis Configuration
REDIS_URL=redis://localhost:6379/0

# SES Configuration
AWS_SES_FROM_EMAIL=noreply@example.com

# Celery Configuration
CELERY_BROKER_URL=redis://localhost:6379/0
CELERY_RESULT_BACKEND=redis://localhost:6379/0
```

### Frontend Environment

Tạo file `.env` trong thư mục `myapporc`:

```bash
cd myapporc
touch .env
```

```bash
VITE_API_URL=http://localhost:8000
VITE_WS_URL=ws://localhost:8000
```

---

## Bước 3: Thiết lập Docker Containers

### Tạo Docker Compose File

```yaml
version: '3.8'

services:
  redis:
    image: redis:7-alpine
    container_name: video-localization-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  mysql:
    image: mysql:8.0
    container_name: video-localization-mysql
    environment:
      MYSQL_ROOT_PASSWORD: Admin1303
      MYSQL_DATABASE: db_sub_video
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  redis_data:
  mysql_data:
```

### Khởi động Containers

```bash
docker-compose up -d
```

Xác minh containers đang chạy:

```bash
docker-compose ps
```

---

## Bước 4: Backend Setup

### Tạo Virtual Environment

```bash
cd ocr-api
python -m venv venv

# Windows
venv\Scripts\activate

# Linux/Mac
source venv/bin/activate
```

### Cài đặt Dependencies

```bash
pip install -r requirements.txt
```

### Chạy Database Migrations

```bash
cd ocr-api
alembic upgrade head
```

### Khởi động Backend Server

```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### Xác minh Backend đang chạy

Mở trình duyệt và truy cập:

```
http://localhost:8000/docs
```

---

## Bước 5: Khởi động Celery Worker

Mở terminal mới:

```bash
cd ocr-api
source venv/bin/activate

celery -A celery worker --loglevel=info
```

---

## Bước 6: Frontend Setup

### Cài đặt Dependencies

```bash
cd myapporc
npm install
```

### Khởi động Development Server

```bash
npm run dev
```

---

## Bước 7: Xác minh tất cả Services

### Test API Health

```bash
curl http://localhost:8000/api/v1/health
```

### Test WebSocket Connection

Tạo file HTML test và mở trong trình duyệt.

---

## Cấu trúc thư mục

### Backend Structure

```
ocr-api/
├── app/
│   ├── api/v1/endpoints/    # API route handlers
│   ├── core/                # Configuration, database, security
│   ├── models/              # SQLAlchemy models
│   ├── modules/             # Business logic modules
│   ├── schemas/             # Pydantic schemas
│   ├── services/            # Service layer
│   ├── tasks/               # Celery tasks
│   ├── utils/               # Utility functions
│   ├── websocket/           # WebSocket handlers
│   └── main.py              # Application entry point
├── alembic/                 # Database migrations
├── celery.py                # Celery configuration
├── docker-compose.yml       # Docker services
├── requirements.txt         # Python dependencies
└── .env                     # Environment variables
```

### Frontend Structure

```
myapporc/
├── src/
│   ├── components/           # React components
│   ├── hooks/               # Custom React hooks
│   ├── interfaces/          # TypeScript interfaces
│   ├── pages/               # Page components
│   ├── store/               # Zustand stores
│   ├── lib/                 # Utilities
│   └── main.tsx             # Application entry point
├── public/                  # Static assets
├── package.json             # Node dependencies
└── vite.config.ts           # Vite configuration
```

---

## Common Issues và Troubleshooting

### Port đã được sử dụng

```bash
# Windows
netstat -ano | findstr :8000
taskkill /PID <pid> /F
```

### Database Connection Error

1. Xác minh container đang chạy:
   ```bash
   docker-compose ps
   ```

2. Kiểm tra logs:
   ```bash
   docker-compose logs mysql
   ```

### Redis Connection Error

1. Xác minh Redis đang chạy:
   ```bash
   docker exec -it video-localization-redis redis-cli ping
   ```
   Expected: `PONG`

---

## Tóm tắt

Trong module này, bạn đã:

- Clone các project repositories
- Tạo các file cấu hình môi trường
- Thiết lập Docker containers cho Redis và MySQL
- Cài đặt backend Python dependencies
- Chạy database migrations
- Khởi động FastAPI backend server
- Khởi động Celery worker
- Cài đặt và khởi động React frontend
- Xác minh tất cả services đang giao tiếp

---

## Bước tiếp theo

Tiếp tục đến [Workshop 5.3: Backend Development](5.3-Backend/) để hiểu implementation backend API và database models.

---

## Validation Checklist

| Component | URL | Expected |
|-----------|-----|----------|
| FastAPI | http://localhost:8000/docs | Swagger UI |
| Frontend | http://localhost:5173 | React app |
| Redis | localhost:6379 | `PONG` on ping |
| MySQL | localhost:3306 | Connection successful |
| Celery Worker | Terminal output | "ready" message |
