# Docker Deployment

## 1. Übersicht

### 1.1 Container-Architektur

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DOCKER COMPOSE STACK                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                      FRONTEND NETWORK (extern)                       │   │
│  │                                                                      │   │
│  │  ┌─────────────┐                                                    │   │
│  │  │    caddy    │ ← Port 80, 443                                     │   │
│  │  │(Rev. Proxy) │                                                    │   │
│  │  └──────┬──────┘                                                    │   │
│  │         │                                                            │   │
│  └─────────┼────────────────────────────────────────────────────────────┘   │
│            │                                                                │
│  ┌─────────┼────────────────────────────────────────────────────────────┐   │
│  │         │           BACKEND NETWORK (intern)                         │   │
│  │         │                                                            │   │
│  │         ▼                                                            │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                  │   │
│  │  │     api     │  │   worker    │  │    beat     │                  │   │
│  │  │  (FastAPI)  │  │  (Celery)   │  │  (Scheduler)│                  │   │
│  │  │  Port 8000  │  │             │  │             │                  │   │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘                  │   │
│  │         │                │                │                          │   │
│  └─────────┼────────────────┼────────────────┼──────────────────────────┘   │
│            │                │                │                              │
│  ┌─────────┼────────────────┼────────────────┼──────────────────────────┐   │
│  │         │           DATA NETWORK (intern)                            │   │
│  │         │                │                │                          │   │
│  │         ▼                ▼                ▼                          │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │   │
│  │  │  postgres   │  │    redis    │  │    minio    │  │   ollama    │ │   │
│  │  │  Port 5432  │  │  Port 6379  │  │  Port 9000  │  │ Port 11434  │ │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘ │   │
│  │                                                                      │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                    HOST VOLUMES (./data)                             │   │
│  │                                                                      │   │
│  │  ./data/postgres/   ./data/redis/   ./data/minio/   ./data/ollama/  │   │
│  │  ./data/uploads/    ./data/backups/ ./data/logs/    ./data/chromadb/│   │
│  │                                                                      │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Docker Compose Konfiguration

### 2.1 docker-compose.yml

```yaml
# docker-compose.yml
version: "3.9"

services:
  # ===========================================
  # REVERSE PROXY
  # ===========================================
  caddy:
    image: caddy:2-alpine
    container_name: energieberater-caddy
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - ./frontend:/srv/frontend:ro
      - ./data/caddy/data:/data
      - ./data/caddy/config:/config
    networks:
      - frontend
      - backend
    depends_on:
      - api
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # ===========================================
  # BACKEND API
  # ===========================================
  api:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: energieberater-api
    restart: unless-stopped
    environment:
      - DATABASE_URL=postgresql://energieberater:${DB_PASSWORD}@postgres:5432/energieberater
      - REDIS_URL=redis://redis:6379/0
      - MINIO_ENDPOINT=minio:9000
      - MINIO_ACCESS_KEY=${MINIO_ACCESS_KEY}
      - MINIO_SECRET_KEY=${MINIO_SECRET_KEY}
      - OLLAMA_BASE_URL=http://ollama:11434
      - SECRET_KEY=${SECRET_KEY}
      - ALLOWED_HOSTS=${ALLOWED_HOSTS:-localhost}
      - DEBUG=${DEBUG:-false}
    volumes:
      - ./data/uploads:/app/uploads
      - ./data/logs:/app/logs
    networks:
      - backend
      - data
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: "2"
          memory: 2G

  # ===========================================
  # CELERY WORKER
  # ===========================================
  worker:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: energieberater-worker
    restart: unless-stopped
    command: celery -A app.tasks worker -l INFO -Q default,calculations,reports
    environment:
      - DATABASE_URL=postgresql://energieberater:${DB_PASSWORD}@postgres:5432/energieberater
      - REDIS_URL=redis://redis:6379/0
      - MINIO_ENDPOINT=minio:9000
      - MINIO_ACCESS_KEY=${MINIO_ACCESS_KEY}
      - MINIO_SECRET_KEY=${MINIO_SECRET_KEY}
      - OLLAMA_BASE_URL=http://ollama:11434
    volumes:
      - ./data/uploads:/app/uploads
      - ./data/logs:/app/logs
    networks:
      - backend
      - data
    depends_on:
      - api
      - redis
    deploy:
      resources:
        limits:
          cpus: "2"
          memory: 2G

  # ===========================================
  # CELERY BEAT (Scheduler)
  # ===========================================
  beat:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: energieberater-beat
    restart: unless-stopped
    command: celery -A app.tasks beat -l INFO
    environment:
      - DATABASE_URL=postgresql://energieberater:${DB_PASSWORD}@postgres:5432/energieberater
      - REDIS_URL=redis://redis:6379/0
    networks:
      - backend
      - data
    depends_on:
      - worker

  # ===========================================
  # POSTGRESQL DATABASE
  # ===========================================
  postgres:
    image: postgres:16-alpine
    container_name: energieberater-postgres
    restart: unless-stopped
    environment:
      - POSTGRES_USER=energieberater
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_DB=energieberater
      - PGDATA=/var/lib/postgresql/data/pgdata
    volumes:
      - ./data/postgres:/var/lib/postgresql/data
      - ./scripts/init-db.sql:/docker-entrypoint-initdb.d/init.sql:ro
    networks:
      - data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U energieberater"]
      interval: 10s
      timeout: 5s
      retries: 5
    deploy:
      resources:
        limits:
          cpus: "1"
          memory: 1G

  # ===========================================
  # REDIS CACHE
  # ===========================================
  redis:
    image: redis:7-alpine
    container_name: energieberater-redis
    restart: unless-stopped
    command: redis-server --appendonly yes --maxmemory 512mb --maxmemory-policy allkeys-lru
    volumes:
      - ./data/redis:/data
    networks:
      - data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: 512M

  # ===========================================
  # MINIO OBJECT STORAGE
  # ===========================================
  minio:
    image: minio/minio:latest
    container_name: energieberater-minio
    restart: unless-stopped
    command: server /data --console-address ":9001"
    environment:
      - MINIO_ROOT_USER=${MINIO_ACCESS_KEY}
      - MINIO_ROOT_PASSWORD=${MINIO_SECRET_KEY}
    volumes:
      - ./data/minio:/data
    networks:
      - data
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 10s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: 512M

  # ===========================================
  # OLLAMA LLM (Optional)
  # ===========================================
  ollama:
    image: ollama/ollama:latest
    container_name: energieberater-ollama
    restart: unless-stopped
    volumes:
      - ./data/ollama:/root/.ollama
    networks:
      - data
    profiles:
      - ai  # Nur mit --profile ai starten
    deploy:
      resources:
        limits:
          cpus: "4"
          memory: 16G
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]

  # ===========================================
  # CHROMADB VECTOR DATABASE (Optional)
  # ===========================================
  chromadb:
    image: chromadb/chroma:latest
    container_name: energieberater-chromadb
    restart: unless-stopped
    volumes:
      - ./data/chromadb:/chroma/chroma
    networks:
      - data
    profiles:
      - ai
    deploy:
      resources:
        limits:
          cpus: "1"
          memory: 2G

  # ===========================================
  # BACKUP SERVICE
  # ===========================================
  backup:
    image: prodrigestivill/postgres-backup-local:16-alpine
    container_name: energieberater-backup
    restart: unless-stopped
    environment:
      - POSTGRES_HOST=postgres
      - POSTGRES_USER=energieberater
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_DB=energieberater
      - SCHEDULE=@daily
      - BACKUP_KEEP_DAYS=7
      - BACKUP_KEEP_WEEKS=4
      - BACKUP_KEEP_MONTHS=6
      - HEALTHCHECK_PORT=8080
    volumes:
      - ./data/backups:/backups
    networks:
      - data
    depends_on:
      - postgres

# ===========================================
# NETWORKS
# ===========================================
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true
  data:
    driver: bridge
    internal: true

# ===========================================
# VOLUMES (für named volumes, falls gewünscht)
# ===========================================
# volumes:
#   postgres_data:
#   redis_data:
#   minio_data:
```

### 2.2 Caddyfile

```caddyfile
# Caddyfile
{
    # E-Mail für Let's Encrypt
    email {$ACME_EMAIL:admin@example.com}

    # Staging für Tests
    # acme_ca https://acme-staging-v02.api.letsencrypt.org/directory
}

{$DOMAIN:localhost} {
    # Logging
    log {
        output file /var/log/caddy/access.log
        format json
    }

    # Security Headers
    header {
        Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
        X-Content-Type-Options "nosniff"
        X-Frame-Options "DENY"
        X-XSS-Protection "1; mode=block"
        Referrer-Policy "strict-origin-when-cross-origin"
        -Server
    }

    # Health Check
    handle /health {
        respond "OK" 200
    }

    # API Routes
    handle /api/* {
        reverse_proxy api:8000 {
            health_uri /health
            health_interval 30s
        }
    }

    # WebSocket
    handle /ws/* {
        reverse_proxy api:8000
    }

    # MinIO (nur für File Downloads)
    handle /s3/* {
        uri strip_prefix /s3
        reverse_proxy minio:9000
    }

    # Static Files (Frontend)
    handle {
        root * /srv/frontend
        file_server
        try_files {path} /index.html
    }

    # Kompression
    encode gzip zstd

    # Rate Limiting
    rate_limit {
        zone api_limit {
            key {remote_host}
            events 100
            window 1m
        }
    }
}

# MinIO Console (Optional, nur intern)
minio.{$DOMAIN:localhost} {
    reverse_proxy minio:9001
}
```

### 2.3 Backend Dockerfile

```dockerfile
# backend/Dockerfile
FROM python:3.11-slim as builder

WORKDIR /build

# Build Dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Python Dependencies
COPY requirements.txt .
RUN pip wheel --no-cache-dir --wheel-dir /wheels -r requirements.txt

# ===========================================
# Production Image
# ===========================================
FROM python:3.11-slim

WORKDIR /app

# Runtime Dependencies
RUN apt-get update && apt-get install -y \
    libpq5 \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Non-root User
RUN useradd -m -s /bin/bash appuser

# Install Python packages
COPY --from=builder /wheels /wheels
RUN pip install --no-cache /wheels/*

# Application Code
COPY --chown=appuser:appuser . .

# Directories
RUN mkdir -p /app/uploads /app/logs && \
    chown -R appuser:appuser /app

USER appuser

# Health Check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## 3. Umgebungsvariablen

### 3.1 .env.example

```bash
# .env.example
# Kopieren nach .env und anpassen

# ===========================================
# ALLGEMEIN
# ===========================================
DOMAIN=localhost
ACME_EMAIL=admin@example.com
DEBUG=false
ALLOWED_HOSTS=localhost,127.0.0.1

# ===========================================
# SICHERHEIT
# ===========================================
# Generieren mit: openssl rand -hex 32
SECRET_KEY=your-super-secret-key-change-me

# ===========================================
# DATENBANK
# ===========================================
DB_PASSWORD=strong-database-password

# ===========================================
# MINIO
# ===========================================
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=strong-minio-password

# ===========================================
# OPTIONAL: E-MAIL
# ===========================================
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=
SMTP_PASSWORD=
SMTP_FROM=noreply@example.com

# ===========================================
# OPTIONAL: OPENAI (wenn nicht Ollama)
# ===========================================
# OPENAI_API_KEY=sk-...
```

---

## 4. Installation

### 4.1 Systemvoraussetzungen

```
Minimum:
- 4 CPU Cores
- 8 GB RAM
- 50 GB SSD
- Docker 24+
- Docker Compose 2.20+

Empfohlen (mit KI):
- 8 CPU Cores
- 32 GB RAM
- 100 GB SSD
- NVIDIA GPU (optional)
```

### 4.2 Installationsschritte

```bash
#!/bin/bash
# install.sh

# 1. Repository klonen
git clone https://github.com/example/energieberaterpro.git
cd energieberaterpro

# 2. Umgebungsvariablen konfigurieren
cp .env.example .env
# Editieren Sie .env mit Ihren Werten:
# nano .env

# 3. Verzeichnisse erstellen
mkdir -p data/{postgres,redis,minio,ollama,chromadb,uploads,backups,logs}
chmod 777 data/postgres  # PostgreSQL braucht Schreibrechte

# 4. Docker Images bauen
docker compose build

# 5. Datenbank initialisieren
docker compose up -d postgres
sleep 10  # Warten bis DB bereit
docker compose exec api python scripts/init_db.py
docker compose exec api alembic upgrade head

# 6. Alle Services starten
docker compose up -d

# 7. Status prüfen
docker compose ps
docker compose logs -f

# 8. (Optional) KI-Services starten
docker compose --profile ai up -d

# 9. (Optional) LLM-Modell laden
docker compose exec ollama ollama pull mistral:7b-instruct
```

### 4.3 Erste Schritte nach Installation

```bash
# Admin-Benutzer erstellen
docker compose exec api python scripts/create_admin.py

# System-Health prüfen
curl http://localhost/health
curl http://localhost/api/v1/health

# Logs überwachen
docker compose logs -f api worker

# Bei Problemen
docker compose down
docker compose up -d --force-recreate
```

---

## 5. Updates & Wartung

### 5.1 Update-Prozess

```bash
#!/bin/bash
# update.sh

# 1. Backup erstellen
docker compose exec postgres pg_dump -U energieberater energieberater > backup_$(date +%Y%m%d).sql

# 2. Neue Version holen
git pull origin main

# 3. Container stoppen
docker compose down

# 4. Images neu bauen
docker compose build --no-cache

# 5. Migrationen ausführen
docker compose up -d postgres
sleep 10
docker compose exec api alembic upgrade head

# 6. Alle Services starten
docker compose up -d

# 7. Health Check
sleep 30
curl http://localhost/api/v1/health
```

### 5.2 Backup & Restore

```bash
# Manuelles Backup
docker compose exec postgres pg_dump -U energieberater energieberater | gzip > backup.sql.gz

# Restore
gunzip -c backup.sql.gz | docker compose exec -T postgres psql -U energieberater energieberater

# Dateien sichern
tar -czvf uploads_backup.tar.gz data/uploads/
tar -czvf minio_backup.tar.gz data/minio/
```

### 5.3 Logs & Monitoring

```bash
# Alle Logs
docker compose logs -f

# Spezifischer Service
docker compose logs -f api
docker compose logs -f worker

# Log-Rotation (in Docker Daemon konfigurieren)
# /etc/docker/daemon.json:
# {
#   "log-driver": "json-file",
#   "log-opts": {
#     "max-size": "10m",
#     "max-file": "3"
#   }
# }
```

---

## 6. Skalierung

### 6.1 Horizontal Scaling

```yaml
# docker-compose.override.yml für Skalierung
services:
  api:
    deploy:
      replicas: 3

  worker:
    deploy:
      replicas: 5
```

```bash
# Skalieren
docker compose up -d --scale api=3 --scale worker=5
```

### 6.2 Load Balancing (Caddy)

```caddyfile
# Caddyfile mit Load Balancing
{$DOMAIN} {
    handle /api/* {
        reverse_proxy api:8000 {
            lb_policy round_robin
            health_uri /health
            health_interval 10s
        }
    }
}
```

---

## 7. Troubleshooting

### 7.1 Häufige Probleme

```bash
# Container startet nicht
docker compose logs <service>
docker compose exec <service> sh  # In Container gehen

# Datenbank-Verbindung
docker compose exec api python -c "from app.db import engine; print(engine.connect())"

# Redis-Verbindung
docker compose exec redis redis-cli ping

# Speicherplatz
df -h
docker system df
docker system prune -a  # VORSICHT: Löscht ungenutzte Images

# Port bereits belegt
lsof -i :80
lsof -i :443
```

### 7.2 Reset

```bash
# Kompletter Reset (ALLE DATEN WERDEN GELÖSCHT!)
docker compose down -v
rm -rf data/*
# Dann neu installieren
```

---

*Letzte Aktualisierung: 2025-11-25*
*Version: 0.1.0*
