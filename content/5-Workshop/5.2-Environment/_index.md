---
title: "Development Environment Setup"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# Workshop 5.2: Development Environment Setup

---

## Learning Objectives

By the end of this module, you will:

- Set up the complete development environment locally
- Configure Docker containers for Redis and MySQL
- Clone and configure the project repositories
- Understand the folder structure
- Run the application in development mode
- Verify all services are communicating correctly

---

## Prerequisites

- Python 3.11 or higher installed
- Node.js 18 or higher installed
- Docker Desktop installed and running
- Git installed
- Code editor (VS Code recommended)

---

## Architecture Context

Before setting up, understand how the components interact in local development:

### Docker Architecture Diagram

![Docker Architecture](/images/5-Workshop/5.2-Environment/Docker_Architecture.png)

### Local Development Architecture Detail

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

### Ports & Services Summary Diagram

![Ports & Services Summary](/images/5-Workshop/5.2-Environment/Ports_Services_Summary.png)

### Ports Table

| Service | Port | Protocol | Description |
|---------|------|----------|-------------|
| **Frontend (Vite)** | 5173 | HTTP | React dev server |
| **Backend (FastAPI)** | 8000 | HTTP/WS | REST API & WebSocket |
| **Swagger UI** | 8000/docs | HTTP | API documentation |
| **Redis** | 6379 | TCP | Message broker |
| **MySQL** | 3306 | TCP | Database |

---

## Step 1: Clone the Project

The project consists of two main repositories:

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

## Step 2: Create Environment Files

### Backend Environment

Create a `.env` file in the `ocr-api` directory:

```bash
cd ocr-api
touch .env
```

Add the following configuration:

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
# Google Gemini API — used as fallback translation provider
API_KEY=your_gemini_api_key
API_MODEL=gemini-2.0-flash

# AWS Translate — primary translation provider (uses AWS_ACCESS_KEY_ID / AWS_SECRET_ACCESS_KEY above)
# AWS_TRANSLATE_REGION defaults to AWS_REGION if not set
AWS_TRANSLATE_REGION=ap-southeast-2

# Google Cloud Translation v2 — last-resort fallback batch provider (optional)
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

Create a `.env` file in the `myapporc` directory:

```bash
cd myapporc
touch .env
```

Add the following configuration:

```bash
VITE_API_URL=http://localhost:8000
VITE_WS_URL=ws://localhost:8000
```

---

## Step 3: Set Up Docker Containers

### Create Docker Compose File

Create `docker-compose.yml` in the `ocr-api` directory:

```bash
cd ocr-api
touch docker-compose.yml
```

Add the following content:

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

### Start Containers

```bash
docker-compose up -d
```

Verify containers are running:

```bash
docker-compose ps
```

Expected output:

```
NAME                        COMMAND                  SERVICE   STATUS
video-localization-redis   "docker-entrypoint.s…"   redis     running (healthy)
video-localization-mysql   "docker-entrypoint.s…"   mysql     running (healthy)
```

---

## Step 4: Backend Setup

### Create Virtual Environment

```bash
cd ocr-api
python -m venv venv

# Windows
venv\Scripts\activate

# Linux/Mac
source venv/bin/activate
```

### Install Dependencies

```bash
pip install -r requirements.txt
```

### Run Database Migrations

```bash
cd ocr-api
alembic upgrade head
```

Expected output:

```
INFO  [alembic.runtime.migration] Context impl MySQLImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> <latest_revision>
```

### Start the Backend Server

```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

You should see:

```
INFO:     Uvicorn running on http://0.0.0.0:8000
INFO:     Application startup complete.
```

### Verify Backend is Running

Open your browser and navigate to:

```
http://localhost:8000/docs
```

You should see the FastAPI automatic documentation.

---

## Step 5: Start Celery Worker

Open a new terminal window:

```bash
cd ocr-api
source venv/bin/activate  # or venv\Scripts\activate on Windows

celery -A celery worker --loglevel=info
```

You should see:

```
[2024-01-01 12:00:00,000: INFO/MainProcess] Connected to redis://localhost:6379/0
[2024-01-01 12:00:00,001: INFO/MainProcess] celery@hostname ready
```

---

## Step 6: Frontend Setup

### Install Dependencies

```bash
cd myapporc
npm install
```

### Start the Development Server

```bash
npm run dev
```

Expected output:

```
  VITE v5.0.0  ready in 500 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: http://192.168.1.x:5173/
```

---

## Step 7: Verify All Services

### Test API Health

```bash
curl http://localhost:8000/api/v1/health
```

Expected response:

```json
{
  "status": "healthy",
  "services": {
    "database": "connected",
    "redis": "connected"
  }
}
```

### Test WebSocket Connection

Create a test HTML file:

```html
<!DOCTYPE html>
<html>
<head>
    <title>WebSocket Test</title>
</head>
<body>
    <h1>WebSocket Test</h1>
    <pre id="output"></pre>
    <script>
        const ws = new WebSocket('ws://localhost:8000/ws/progress/test-video-id');
        ws.onopen = () => {
            document.getElementById('output').textContent = 'Connected!';
        };
        ws.onmessage = (event) => {
            document.getElementById('output').textContent = event.data;
        };
    </script>
</body>
</html>
```

Open the file in a browser and check for "Connected!" message.

---

## Folder Structure

### Backend Structure

```
ocr-api/
├── app/
│   ├── api/v1/endpoints/    # API route handlers
│   ├── core/                # Configuration, database, security
│   ├── models/              # SQLAlchemy models
│   ├── modules/             # Business logic modules
│   │   └── module/          # Individual feature modules
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
│   │   ├── ui/              # UI primitives
│   │   ├── video/           # Video-related components
│   │   └── ...
│   ├── hooks/               # Custom React hooks
│   ├── interfaces/          # TypeScript interfaces
│   ├── pages/               # Page components
│   │   ├── auth/            # Authentication pages
│   │   └── editor/          # Video editor pages
│   ├── store/               # Zustand stores
│   ├── lib/                 # Utilities
│   └── main.tsx             # Application entry point
├── public/                  # Static assets
├── package.json             # Node dependencies
└── vite.config.ts           # Vite configuration
```

---

## Common Issues and Troubleshooting

### Port Already in Use

If you get "Port is already in use" errors:

```bash
# Windows
netstat -ano | findstr :8000
taskkill /PID <pid> /F

# Linux/Mac
lsof -i :8000
kill -9 <pid>
```

### Database Connection Error

If you cannot connect to MySQL:

1. Verify container is running:
   ```bash
   docker-compose ps
   ```

2. Check logs:
   ```bash
   docker-compose logs mysql
   ```

3. Ensure port 3306 is not in use by another MySQL instance.

### Redis Connection Error

If Celery cannot connect to Redis:

1. Verify Redis is running:
   ```bash
   docker-compose ps
   docker-compose logs redis
   ```

2. Test Redis connection:
   ```bash
   docker exec -it video-localization-redis redis-cli ping
   ```
   Expected response: `PONG`

---

## Summary

In this module, you:

- Cloned the project repositories
- Created environment configuration files
- Set up Docker containers for Redis and MySQL
- Installed backend Python dependencies
- Ran database migrations
- Started the FastAPI backend server
- Started the Celery worker
- Installed and started the React frontend
- Verified all services are communicating

---

## Next Steps

Proceed to [Workshop 5.3: Backend Development](5.3-Backend/) to understand the backend API implementation and database models.

---

## Validation Checklist

Before proceeding, verify all components are running:

| Component | URL | Expected |
|-----------|-----|----------|
| FastAPI | http://localhost:8000/docs | Swagger UI |
| Frontend | http://localhost:5173 | React app |
| Redis | localhost:6379 | `PONG` on ping |
| MySQL | localhost:3306 | Connection successful |
| Celery Worker | Terminal output | "ready" message |
