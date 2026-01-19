# VAANI Engine - Deployment Guide

**Complete deployment documentation for local development, staging, and production environments**

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Environment Configuration](#environment-configuration)
3. [Local Development Setup](#local-development-setup)
4. [Docker Deployment](#docker-deployment)
5. [Production Deployment](#production-deployment)
6. [Database Setup](#database-setup)
7. [Twilio Configuration](#twilio-configuration)
8. [SSL/TLS Configuration](#ssltls-configuration)
9. [Monitoring Setup](#monitoring-setup)
10. [Backup & Recovery](#backup--recovery)
11. [Troubleshooting](#troubleshooting)

---

## 1. Prerequisites

### 1.1 System Requirements

**Minimum Requirements (Development):**
```
CPU: 4 cores
RAM: 8 GB
Disk: 50 GB SSD
OS: Linux (Ubuntu 22.04), macOS (12+), or Windows 11 with WSL2
```

**Recommended Requirements (Production):**
```
CPU: 16 cores
RAM: 32 GB
Disk: 200 GB SSD
OS: Ubuntu 22.04 LTS Server
GPU: NVIDIA GPU with 8GB+ VRAM (optional, for self-hosted voice models)
```

### 1.2 Required Software

**Development Environment:**
```bash
# Core dependencies
- Python 3.11 or higher
- PostgreSQL 15+
- Redis 7+
- Git 2.30+

# Optional (recommended)
- Docker 24.0+
- Docker Compose 2.20+
- Node.js 18+ (for admin dashboard)
```

**Installation Commands:**

**Ubuntu/Debian:**
```bash
# Update package list
sudo apt update

# Install Python 3.11
sudo apt install -y python3.11 python3.11-venv python3-pip

# Install PostgreSQL
sudo apt install -y postgresql postgresql-contrib

# Install Redis
sudo apt install -y redis-server

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# Install Docker Compose
sudo apt install -y docker-compose-plugin
```

**macOS:**
```bash
# Install Homebrew (if not already installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Python 3.11
brew install python@3.11

# Install PostgreSQL
brew install postgresql@15
brew services start postgresql@15

# Install Redis
brew install redis
brew services start redis

# Install Docker Desktop
brew install --cask docker
```

### 1.3 External Service Accounts

Create accounts and obtain API keys for:

| Service | Purpose | Signup URL |
|---------|---------|------------|
| **Twilio** | Voice gateway | https://www.twilio.com/try-twilio |
| **OpenAI** | GPT-4o language model | https://platform.openai.com/signup |
| **Deepgram** | Speech-to-text | https://console.deepgram.com/signup |
| **Cartesia** | Text-to-speech | https://cartesia.ai/signup |
| **SendGrid** (optional) | Email notifications | https://signup.sendgrid.com/ |

---

## 2. Environment Configuration

### 2.1 Environment Variables

Create a `.env` file in the project root:

```bash
# Copy example environment file
cp .env.example .env
```

**Complete `.env` Configuration:**

```bash
# ============================================
# Application Settings
# ============================================
APP_NAME=VAANI Engine
ENVIRONMENT=development  # development, staging, production
DEBUG=true
LOG_LEVEL=INFO
BASE_URL=http://localhost:8000

# ============================================
# Database Configuration
# ============================================
DATABASE_URL=postgresql+asyncpg://vaani_user:your_password@localhost:5432/vaani_db
DATABASE_POOL_SIZE=20
DATABASE_MAX_OVERFLOW=10

# ============================================
# Redis Configuration
# ============================================
REDIS_URL=redis://localhost:6379/0
REDIS_MAX_CONNECTIONS=50

# ============================================
# Twilio Configuration
# ============================================
TWILIO_ACCOUNT_SID=ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_AUTH_TOKEN=your_auth_token_here
TWILIO_PHONE_NUMBER=+1234567890
TWILIO_STATUS_CALLBACK_URL=https://your-domain.com/api/v1/voice/status

# ============================================
# OpenAI Configuration
# ============================================
OPENAI_API_KEY=sk-proj-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
OPENAI_MODEL=gpt-4o
OPENAI_MAX_TOKENS=500
OPENAI_TEMPERATURE=0.7

# ============================================
# Deepgram Configuration (STT)
# ============================================
DEEPGRAM_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
DEEPGRAM_MODEL=nova-2
DEEPGRAM_LANGUAGE=multi  # auto-detect Hindi/English/Hinglish

# ============================================
# Cartesia Configuration (TTS)
# ============================================
CARTESIA_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
CARTESIA_VOICE_ID=indian-female-warm
CARTESIA_MODEL=sonic-english

# ============================================
# Security Settings
# ============================================
JWT_SECRET_KEY=your-super-secret-jwt-key-change-this-in-production
JWT_ALGORITHM=HS256
JWT_EXPIRATION_MINUTES=60

# Encryption for sensitive patient data
ENCRYPTION_KEY=your-32-byte-encryption-key-base64-encoded
ENCRYPTION_ALGORITHM=AES-256-GCM

# ============================================
# Voice Engine Settings
# ============================================
MAX_CONCURRENT_SESSIONS=50
SESSION_TIMEOUT_SECONDS=600
AUDIO_BUFFER_SECONDS=30
VAD_THRESHOLD=0.5
SILENCE_DURATION_MS=700

# ============================================
# SendGrid Configuration (Optional)
# ============================================
SENDGRID_API_KEY=SG.xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
SENDGRID_FROM_EMAIL=noreply@vaani-engine.com
SENDGRID_FROM_NAME=VAANI Engine

# ============================================
# SMS Configuration (via Twilio)
# ============================================
SMS_ENABLED=true
SMS_APPOINTMENT_REMINDER_HOURS=24

# ============================================
# Monitoring & Analytics
# ============================================
SENTRY_DSN=https://xxxxx@sentry.io/xxxxx  # Optional error tracking
PROMETHEUS_PORT=9090

# ============================================
# CORS Settings
# ============================================
CORS_ORIGINS=http://localhost:3000,https://admin.vaani-engine.com
CORS_ALLOW_CREDENTIALS=true

# ============================================
# Rate Limiting
# ============================================
RATE_LIMIT_PER_MINUTE=60
RATE_LIMIT_BURST=10

# ============================================
# Feature Flags
# ============================================
FEATURE_VOICE_ANALYTICS=true
FEATURE_SENTIMENT_ANALYSIS=true
FEATURE_CALL_RECORDING=false  # Disabled by default (privacy)
```

### 2.2 Environment-Specific Configurations

**Development (.env.development):**
```bash
ENVIRONMENT=development
DEBUG=true
LOG_LEVEL=DEBUG
BASE_URL=http://localhost:8000
DATABASE_URL=postgresql+asyncpg://vaani_user:dev_password@localhost:5432/vaani_dev
```

**Staging (.env.staging):**
```bash
ENVIRONMENT=staging
DEBUG=false
LOG_LEVEL=INFO
BASE_URL=https://staging.vaani-engine.com
DATABASE_URL=postgresql+asyncpg://vaani_user:staging_password@staging-db:5432/vaani_staging
```

**Production (.env.production):**
```bash
ENVIRONMENT=production
DEBUG=false
LOG_LEVEL=WARNING
BASE_URL=https://api.vaani-engine.com
DATABASE_URL=postgresql+asyncpg://vaani_user:prod_password@prod-db:5432/vaani_prod
```

---

## 3. Local Development Setup

### 3.1 Clone Repository

```bash
# Clone the repository
git clone https://github.com/mdkashifakram/VAANI_Engine.git
cd VAANI_Engine

# Create and activate virtual environment
python3.11 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Upgrade pip
pip install --upgrade pip
```

### 3.2 Install Dependencies

```bash
# Install production dependencies
pip install -r requirements.txt

# Install development dependencies
pip install -r requirements-dev.txt
```

**requirements.txt:**
```
fastapi==0.104.1
uvicorn[standard]==0.24.0
sqlalchemy[asyncio]==2.0.23
asyncpg==0.29.0
redis[hiredis]==5.0.1
python-dotenv==1.0.0
pydantic==2.5.0
pydantic-settings==2.1.0
twilio==8.10.0
openai==1.3.5
httpx==0.25.1
python-multipart==0.0.6
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
alembic==1.12.1
psycopg2-binary==2.9.9
prometheus-client==0.19.0
```

**requirements-dev.txt:**
```
pytest==7.4.3
pytest-asyncio==0.21.1
pytest-cov==4.1.0
black==23.11.0
flake8==6.1.0
mypy==1.7.1
isort==5.12.0
pre-commit==3.5.0
httpx==0.25.1
faker==20.1.0
```

### 3.3 Database Setup

```bash
# Start PostgreSQL (if not running)
# Ubuntu/Debian:
sudo systemctl start postgresql

# macOS:
brew services start postgresql@15

# Create database and user
sudo -u postgres psql << EOF
CREATE USER vaani_user WITH PASSWORD 'your_password';
CREATE DATABASE vaani_db OWNER vaani_user;
GRANT ALL PRIVILEGES ON DATABASE vaani_db TO vaani_user;
EOF

# Run database migrations
alembic upgrade head

# Seed initial data (optional)
python scripts/seed_database.py
```

### 3.4 Redis Setup

```bash
# Start Redis
# Ubuntu/Debian:
sudo systemctl start redis-server

# macOS:
brew services start redis

# Verify Redis is running
redis-cli ping
# Expected output: PONG
```

### 3.5 Start Development Server

```bash
# Start FastAPI development server with auto-reload
uvicorn src.main:app --reload --host 0.0.0.0 --port 8000

# Alternative: Use the provided script
chmod +x scripts/run_dev.sh
./scripts/run_dev.sh
```

**scripts/run_dev.sh:**
```bash
#!/bin/bash
set -e

echo "Starting VAANI Engine (Development Mode)..."

# Activate virtual environment
source venv/bin/activate

# Load environment variables
export $(cat .env | grep -v '^#' | xargs)

# Run database migrations
echo "Running database migrations..."
alembic upgrade head

# Start server
echo "Starting FastAPI server..."
uvicorn src.main:app \
  --reload \
  --host 0.0.0.0 \
  --port 8000 \
  --log-level debug
```

### 3.6 Verify Installation

```bash
# Test health endpoint
curl http://localhost:8000/health

# Expected response:
# {
#   "status": "healthy",
#   "timestamp": "2026-01-20T10:00:00Z",
#   "version": "2.0.0",
#   "components": {
#     "database": "connected",
#     "redis": "connected"
#   }
# }

# Test API documentation
open http://localhost:8000/docs
```

---

## 4. Docker Deployment

### 4.1 Docker Compose (Recommended)

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  backend:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: vaani-backend
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql+asyncpg://vaani_user:vaani_pass@db:5432/vaani_db
      - REDIS_URL=redis://redis:6379/0
    env_file:
      - .env
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    volumes:
      - ./logs:/app/logs
    restart: unless-stopped
    networks:
      - vaani-network

  db:
    image: postgres:15-alpine
    container_name: vaani-postgres
    environment:
      POSTGRES_DB: vaani_db
      POSTGRES_USER: vaani_user
      POSTGRES_PASSWORD: vaani_pass
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./scripts/init_db.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U vaani_user -d vaani_db"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped
    networks:
      - vaani-network

  redis:
    image: redis:7-alpine
    container_name: vaani-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes
    restart: unless-stopped
    networks:
      - vaani-network

  nginx:
    image: nginx:alpine
    container_name: vaani-nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/ssl:/etc/nginx/ssl
    depends_on:
      - backend
    restart: unless-stopped
    networks:
      - vaani-network

volumes:
  postgres_data:
  redis_data:

networks:
  vaani-network:
    driver: bridge
```

### 4.2 Dockerfile

**Dockerfile:**
```dockerfile
# Multi-stage build for production optimization
FROM python:3.11-slim as builder

# Set working directory
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir --user -r requirements.txt

# Final stage
FROM python:3.11-slim

WORKDIR /app

# Install runtime dependencies
RUN apt-get update && apt-get install -y \
    libpq5 \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Copy Python dependencies from builder
COPY --from=builder /root/.local /root/.local

# Copy application code
COPY . .

# Make scripts executable
RUN chmod +x scripts/*.sh

# Set environment variables
ENV PATH=/root/.local/bin:$PATH
ENV PYTHONUNBUFFERED=1
ENV PYTHONDONTWRITEBYTECODE=1

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

# Run database migrations and start server
CMD ["sh", "-c", "alembic upgrade head && uvicorn src.main:app --host 0.0.0.0 --port 8000"]
```

### 4.3 Build and Run with Docker Compose

```bash
# Build images
docker-compose build

# Start all services
docker-compose up -d

# View logs
docker-compose logs -f backend

# Check service status
docker-compose ps

# Stop services
docker-compose down

# Stop and remove volumes (clean slate)
docker-compose down -v
```

### 4.4 Docker Production Optimization

**docker-compose.prod.yml:**
```yaml
version: '3.8'

services:
  backend:
    build:
      context: .
      dockerfile: Dockerfile.prod
      args:
        - BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ')
        - VERSION=2.0.0
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
    environment:
      - ENVIRONMENT=production
      - WORKERS=4
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  # ... (db, redis configurations similar to above)
```

---

## 5. Production Deployment

### 5.1 Cloud Provider Options

#### Option A: AWS Deployment

**Architecture:**
```
┌─────────────────────────────────────────────┐
│          Application Load Balancer          │
│         (HTTPS termination, WAF)            │
└───────────────────┬─────────────────────────┘
                    │
        ┌───────────┴───────────┐
        │                       │
        ▼                       ▼
┌───────────────┐       ┌───────────────┐
│   ECS Task 1  │       │   ECS Task 2  │
│ (FastAPI app) │       │ (FastAPI app) │
└───────────────┘       └───────────────┘
        │                       │
        └───────────┬───────────┘
                    │
        ┌───────────┴───────────┐
        │                       │
        ▼                       ▼
┌───────────────┐       ┌───────────────┐
│  RDS Postgres │       │ ElastiCache   │
│   (Primary)   │       │   (Redis)     │
└───────────────┘       └───────────────┘
```

**AWS ECS Deployment:**

**task-definition.json:**
```json
{
  "family": "vaani-backend",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "1024",
  "memory": "2048",
  "containerDefinitions": [
    {
      "name": "vaani-app",
      "image": "your-ecr-repo/vaani-engine:latest",
      "portMappings": [
        {
          "containerPort": 8000,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "ENVIRONMENT",
          "value": "production"
        }
      ],
      "secrets": [
        {
          "name": "DATABASE_URL",
          "valueFrom": "arn:aws:secretsmanager:region:account:secret:vaani/database-url"
        },
        {
          "name": "OPENAI_API_KEY",
          "valueFrom": "arn:aws:secretsmanager:region:account:secret:vaani/openai-key"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/vaani-backend",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "healthCheck": {
        "command": ["CMD-SHELL", "curl -f http://localhost:8000/health || exit 1"],
        "interval": 30,
        "timeout": 5,
        "retries": 3,
        "startPeriod": 60
      }
    }
  ]
}
```

**Deployment Commands:**
```bash
# Build and push Docker image to ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin your-account.dkr.ecr.us-east-1.amazonaws.com

docker build -t vaani-engine:latest .
docker tag vaani-engine:latest your-account.dkr.ecr.us-east-1.amazonaws.com/vaani-engine:latest
docker push your-account.dkr.ecr.us-east-1.amazonaws.com/vaani-engine:latest

# Register task definition
aws ecs register-task-definition --cli-input-json file://task-definition.json

# Update ECS service
aws ecs update-service \
  --cluster vaani-cluster \
  --service vaani-backend-service \
  --task-definition vaani-backend:2 \
  --force-new-deployment
```

#### Option B: Google Cloud Platform (GCP)

**Cloud Run Deployment:**
```bash
# Build with Cloud Build
gcloud builds submit --tag gcr.io/your-project/vaani-engine

# Deploy to Cloud Run
gcloud run deploy vaani-backend \
  --image gcr.io/your-project/vaani-engine \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --set-env-vars ENVIRONMENT=production \
  --set-secrets DATABASE_URL=vaani-db-url:latest,OPENAI_API_KEY=openai-key:latest \
  --memory 2Gi \
  --cpu 2 \
  --min-instances 2 \
  --max-instances 10 \
  --concurrency 80
```

#### Option C: Kubernetes (Any Cloud)

**kubernetes/deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vaani-backend
  namespace: vaani
spec:
  replicas: 3
  selector:
    matchLabels:
      app: vaani-backend
  template:
    metadata:
      labels:
        app: vaani-backend
        version: v2.0.0
    spec:
      containers:
      - name: backend
        image: vaani-engine:2.0.0
        ports:
        - containerPort: 8000
          name: http
        env:
        - name: ENVIRONMENT
          value: "production"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: vaani-secrets
              key: database-url
        - name: REDIS_URL
          valueFrom:
            configMapKeyRef:
              name: vaani-config
              key: redis-url
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: 8000
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 2
---
apiVersion: v1
kind: Service
metadata:
  name: vaani-backend-service
  namespace: vaani
spec:
  selector:
    app: vaani-backend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8000
  type: LoadBalancer
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: vaani-backend-hpa
  namespace: vaani
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: vaani-backend
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

**Deploy to Kubernetes:**
```bash
# Create namespace
kubectl create namespace vaani

# Create secrets
kubectl create secret generic vaani-secrets \
  --from-literal=database-url='postgresql+asyncpg://...' \
  --from-literal=openai-api-key='sk-...' \
  --namespace vaani

# Apply configurations
kubectl apply -f kubernetes/deployment.yaml
kubectl apply -f kubernetes/service.yaml
kubectl apply -f kubernetes/ingress.yaml

# Check deployment status
kubectl get pods -n vaani
kubectl get svc -n vaani

# View logs
kubectl logs -f deployment/vaani-backend -n vaani
```

---

## 6. Database Setup

### 6.1 Database Migrations

**Alembic Configuration:**

**alembic.ini:**
```ini
[alembic]
script_location = alembic
prepend_sys_path = .
version_path_separator = os

sqlalchemy.url = driver://user:pass@localhost/dbname

[loggers]
keys = root,sqlalchemy,alembic

[handlers]
keys = console

[formatters]
keys = generic

[logger_root]
level = WARN
handlers = console
qualname =

[logger_sqlalchemy]
level = WARN
handlers =
qualname = sqlalchemy.engine

[logger_alembic]
level = INFO
handlers =
qualname = alembic

[handler_console]
class = StreamHandler
args = (sys.stderr,)
level = NOTSET
formatter = generic

[formatter_generic]
format = %(levelname)-5.5s [%(name)s] %(message)s
datefmt = %H:%M:%S
```

**Migration Commands:**
```bash
# Create new migration
alembic revision --autogenerate -m "Add new table"

# Apply migrations
alembic upgrade head

# Rollback migration
alembic downgrade -1

# View migration history
alembic history

# View current version
alembic current
```

### 6.2 Database Backup

**Automated Backup Script (scripts/backup_db.sh):**
```bash
#!/bin/bash
set -e

# Configuration
BACKUP_DIR="/backups/postgres"
DB_NAME="vaani_db"
DB_USER="vaani_user"
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
BACKUP_FILE="${BACKUP_DIR}/${DB_NAME}_${TIMESTAMP}.sql.gz"
RETENTION_DAYS=7

# Create backup directory
mkdir -p $BACKUP_DIR

# Perform backup
echo "Starting database backup..."
pg_dump -U $DB_USER -d $DB_NAME | gzip > $BACKUP_FILE

# Verify backup
if [ -f "$BACKUP_FILE" ]; then
    SIZE=$(du -h "$BACKUP_FILE" | cut -f1)
    echo "Backup completed successfully: $BACKUP_FILE ($SIZE)"
else
    echo "Backup failed!"
    exit 1
fi

# Remove old backups
find $BACKUP_DIR -name "${DB_NAME}_*.sql.gz" -mtime +$RETENTION_DAYS -delete
echo "Cleaned up backups older than $RETENTION_DAYS days"

# Upload to S3 (optional)
if [ ! -z "$AWS_S3_BUCKET" ]; then
    aws s3 cp $BACKUP_FILE s3://$AWS_S3_BUCKET/backups/postgres/
    echo "Backup uploaded to S3"
fi
```

**Cron Job for Daily Backups:**
```bash
# Add to crontab (crontab -e)
0 2 * * * /path/to/VAANI_Engine/scripts/backup_db.sh >> /var/log/vaani_backup.log 2>&1
```

### 6.3 Database Restore

```bash
# Restore from backup
gunzip -c /backups/postgres/vaani_db_20260120_020000.sql.gz | psql -U vaani_user -d vaani_db

# Restore from S3 backup
aws s3 cp s3://your-bucket/backups/postgres/vaani_db_20260120_020000.sql.gz - | gunzip | psql -U vaani_user -d vaani_db
```

---

## 7. Twilio Configuration

### 7.1 Phone Number Setup

**Step 1: Purchase Phone Number**
```
1. Log into Twilio Console: https://console.twilio.com/
2. Navigate to Phone Numbers → Buy a Number
3. Search for number in desired country (India: +91)
4. Select number with Voice capability
5. Purchase number
```

**Step 2: Configure Voice Webhooks**
```
1. Go to Phone Numbers → Manage → Active Numbers
2. Click on your purchased number
3. Under "Voice & Fax" section:

   A CALL COMES IN:
   - Webhook: https://your-domain.com/api/v1/voice/incoming
   - HTTP Method: POST

   CALL STATUS CHANGES:
   - Webhook: https://your-domain.com/api/v1/voice/status
   - HTTP Method: POST

4. Save configuration
```

### 7.2 Test Call Flow

**Test Script (scripts/test_call.py):**
```python
from twilio.rest import Client
import os
from dotenv import load_dotenv

load_dotenv()

client = Client(
    os.getenv("TWILIO_ACCOUNT_SID"),
    os.getenv("TWILIO_AUTH_TOKEN")
)

# Make test call
call = client.calls.create(
    to="+919876543210",  # Your test number
    from_=os.getenv("TWILIO_PHONE_NUMBER"),
    url="https://your-domain.com/api/v1/voice/incoming"
)

print(f"Call initiated: {call.sid}")
print(f"Status: {call.status}")
```

### 7.3 ngrok for Local Testing

```bash
# Install ngrok
brew install ngrok  # macOS
# or
sudo apt install ngrok  # Ubuntu

# Start ngrok tunnel
ngrok http 8000

# Copy HTTPS URL and update Twilio webhook
# Example: https://abc123.ngrok.io/api/v1/voice/incoming
```

---

## 8. SSL/TLS Configuration

### 8.1 Nginx SSL Configuration

**nginx/nginx.conf:**
```nginx
upstream vaani_backend {
    least_conn;
    server backend:8000;
}

server {
    listen 80;
    server_name api.vaani-engine.com;

    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name api.vaani-engine.com;

    # SSL certificates
    ssl_certificate /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;

    # SSL configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # WebSocket support
    location /api/v1/voice/stream {
        proxy_pass http://vaani_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket timeout
        proxy_read_timeout 600s;
        proxy_send_timeout 600s;
    }

    # HTTP endpoints
    location / {
        proxy_pass http://vaani_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Timeout settings
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
    limit_req zone=api_limit burst=20 nodelay;
}
```

### 8.2 Let's Encrypt SSL Certificate

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d api.vaani-engine.com

# Auto-renewal (add to crontab)
0 3 * * * certbot renew --quiet --post-hook "systemctl reload nginx"
```

---

## 9. Monitoring Setup

### 9.1 Prometheus Configuration

**prometheus.yml:**
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'vaani-backend'
    static_configs:
      - targets: ['backend:8000']
    metrics_path: '/metrics'

  - job_name: 'postgres'
    static_configs:
      - targets: ['postgres-exporter:9187']

  - job_name: 'redis'
    static_configs:
      - targets: ['redis-exporter:9121']
```

### 9.2 Grafana Dashboard

**grafana/dashboards/vaani-overview.json:**
```json
{
  "dashboard": {
    "title": "VAANI Engine Overview",
    "panels": [
      {
        "title": "Active Voice Sessions",
        "targets": [
          {
            "expr": "voice_active_sessions"
          }
        ]
      },
      {
        "title": "End-to-End Latency (p95)",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(voice_e2e_latency_ms_bucket[5m]))"
          }
        ]
      }
    ]
  }
}
```

---

## 10. Backup & Recovery

### 10.1 Automated Backup Strategy

**Backup Schedule:**
- Database: Daily at 2 AM UTC
- Configuration files: On change
- Application logs: Continuous to S3
- Redis snapshots: Every 6 hours

**scripts/full_backup.sh:**
```bash
#!/bin/bash
set -e

TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
BACKUP_ROOT="/backups/${TIMESTAMP}"

# Create backup directory
mkdir -p $BACKUP_ROOT

# Backup database
pg_dump -U vaani_user vaani_db | gzip > "${BACKUP_ROOT}/database.sql.gz"

# Backup Redis
redis-cli --rdb "${BACKUP_ROOT}/redis_dump.rdb"

# Backup configuration
tar -czf "${BACKUP_ROOT}/config.tar.gz" .env nginx/ kubernetes/

# Upload to S3
aws s3 sync $BACKUP_ROOT s3://vaani-backups/${TIMESTAMP}/

echo "Backup completed: $BACKUP_ROOT"
```

### 10.2 Disaster Recovery

**Recovery Time Objective (RTO):** 1 hour
**Recovery Point Objective (RPO):** 24 hours

**Recovery Steps:**
```bash
# 1. Restore database
aws s3 cp s3://vaani-backups/latest/database.sql.gz - | gunzip | psql -U vaani_user -d vaani_db

# 2. Restore Redis
aws s3 cp s3://vaani-backups/latest/redis_dump.rdb /var/lib/redis/dump.rdb
systemctl restart redis

# 3. Redeploy application
kubectl apply -f kubernetes/

# 4. Verify health
curl https://api.vaani-engine.com/health
```

---

## 11. Troubleshooting

### 11.1 Common Issues

**Issue: Database connection errors**
```bash
# Check PostgreSQL status
sudo systemctl status postgresql

# Verify connection
psql -U vaani_user -d vaani_db -h localhost

# Check logs
sudo tail -f /var/log/postgresql/postgresql-15-main.log
```

**Issue: Redis connection errors**
```bash
# Check Redis status
redis-cli ping

# Check Redis logs
sudo tail -f /var/log/redis/redis-server.log

# Flush Redis (caution: development only)
redis-cli FLUSHALL
```

**Issue: High latency on voice calls**
```bash
# Check voice engine metrics
curl http://localhost:8000/metrics | grep voice_latency

# Check active sessions
curl http://localhost:8000/admin/sessions

# Review logs for bottlenecks
docker-compose logs -f backend | grep "latency"
```

**Issue: Twilio webhook failures**
```bash
# Check Twilio debugger
# https://console.twilio.com/us1/monitor/logs/debugger

# Test webhook locally
curl -X POST http://localhost:8000/api/v1/voice/incoming \
  -d "CallSid=CAtest123" \
  -d "From=+919876543210" \
  -d "To=+911234567890"
```

### 11.2 Debug Mode

**Enable debug logging:**
```bash
# Update .env
LOG_LEVEL=DEBUG
DEBUG=true

# Restart application
docker-compose restart backend

# View detailed logs
docker-compose logs -f backend
```

### 11.3 Performance Profiling

```bash
# Install profiling tools
pip install py-spy

# Profile running process
py-spy top --pid <process_id>

# Generate flame graph
py-spy record -o profile.svg --pid <process_id>
```

---

## Deployment Checklist

### Pre-Deployment
- [ ] All environment variables configured
- [ ] Database migrations applied
- [ ] SSL certificates installed
- [ ] Twilio webhooks configured
- [ ] External API keys validated
- [ ] Backup strategy implemented
- [ ] Monitoring dashboards created

### Post-Deployment
- [ ] Health check passing
- [ ] Test call successful
- [ ] Database connectivity verified
- [ ] Redis caching working
- [ ] Metrics being collected
- [ ] Logs being aggregated
- [ ] SSL certificate valid
- [ ] Auto-scaling configured

---

**Document Version**: 1.0
**Last Updated**: 2026-01-20
**Maintained By**: VAANI DevOps Team
