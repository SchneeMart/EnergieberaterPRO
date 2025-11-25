# Systemkomponenten - Detailbeschreibung

## 1. Caddy Reverse Proxy

### 1.1 Konfiguration
```
Primäre Funktionen:
- Automatisches HTTPS via Let's Encrypt
- HTTP/2 und HTTP/3 Support
- Automatische Kompression
- Security Headers
```

### 1.2 Routing-Regeln
| Pfad | Ziel | Beschreibung |
|------|------|--------------|
| `/api/*` | api:8000 | Backend API |
| `/ws/*` | api:8000 | WebSocket |
| `/s3/*` | minio:9000 | Datei-Downloads |
| `/*` | static files | Frontend |

### 1.3 Sicherheitskonfiguration
```
Headers:
- Strict-Transport-Security: max-age=31536000
- X-Content-Type-Options: nosniff
- X-Frame-Options: DENY
- Content-Security-Policy: [detailliert]
- Referrer-Policy: strict-origin-when-cross-origin
```

### 1.4 Rate Limiting
```
Regeln:
- Login: 5 Versuche / Minute / IP
- API allgemein: 100 Requests / Minute / Token
- Upload: 10 Requests / Minute / User
- Report-Generierung: 5 / Minute / User
```

---

## 2. FastAPI Backend

### 2.1 Struktur
```
app/
├── __init__.py
├── main.py                 # Application Factory
├── config.py               # Settings (Pydantic)
├── dependencies.py         # Dependency Injection
│
├── api/                    # API Layer
│   ├── __init__.py
│   ├── router.py           # Haupt-Router
│   ├── v1/                 # API Version 1
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   ├── users.py
│   │   ├── projects.py
│   │   ├── buildings.py
│   │   ├── calculations.py
│   │   ├── reports.py
│   │   ├── business.py
│   │   └── ai.py
│   └── websocket/
│       └── handlers.py
│
├── core/                   # Kernfunktionen
│   ├── security.py         # JWT, Hashing
│   ├── permissions.py      # RBAC
│   ├── exceptions.py       # Custom Exceptions
│   └── middleware.py       # Custom Middleware
│
├── models/                 # SQLAlchemy Models
│   ├── __init__.py
│   ├── base.py             # Base Model
│   ├── user.py
│   ├── project.py
│   ├── building.py
│   ├── calculation.py
│   └── business.py
│
├── schemas/                # Pydantic Schemas
│   ├── __init__.py
│   ├── user.py
│   ├── project.py
│   ├── building.py
│   ├── calculation.py
│   └── business.py
│
├── services/               # Business Logic
│   ├── __init__.py
│   ├── auth_service.py
│   ├── project_service.py
│   ├── building_service.py
│   ├── calculation_service.py
│   ├── report_service.py
│   ├── business_service.py
│   └── ai_service.py
│
├── calculations/           # Berechnungs-Engine
│   ├── __init__.py
│   ├── base.py             # Basis-Klassen
│   ├── thermal/            # Wärmetechnik
│   ├── energy/             # Energiebilanz
│   ├── renewable/          # Erneuerbare
│   └── emissions/          # CO2
│
├── ai/                     # KI-Integration
│   ├── __init__.py
│   ├── llm.py              # LLM Client
│   ├── embeddings.py       # Embedding Service
│   ├── prompts.py          # Prompt Templates
│   └── agents.py           # AI Agents
│
└── utils/                  # Hilfsfunktionen
    ├── __init__.py
    ├── validators.py
    ├── converters.py
    └── helpers.py
```

### 2.2 Middleware-Stack
```python
# Reihenfolge der Middleware (von außen nach innen)
1. CORSMiddleware         # CORS-Handling
2. TrustedHostMiddleware  # Host-Validierung
3. GZipMiddleware         # Kompression
4. RequestIDMiddleware    # Request-Tracking
5. TimingMiddleware       # Performance-Logging
6. AuthMiddleware         # JWT-Validierung
7. RateLimitMiddleware    # Rate Limiting
```

### 2.3 Error Handling
```python
# Hierarchie der Exceptions
HTTPException
├── AuthenticationError (401)
├── AuthorizationError (403)
├── NotFoundError (404)
├── ValidationError (422)
├── BusinessLogicError (400)
├── CalculationError (400)
└── InternalError (500)
```

---

## 3. Celery Worker

### 3.1 Aufgaben-Kategorien
```
Priorität HOCH:
- Berichts-Generierung
- Export-Aufgaben
- E-Mail-Versand

Priorität MITTEL:
- Berechnungs-Batch
- Datenimport
- Backup-Erstellung

Priorität NIEDRIG:
- Cleanup-Tasks
- Statistik-Aggregation
- KI-Training
```

### 3.2 Queue-Konfiguration
```python
CELERY_TASK_QUEUES = {
    'high': Queue('high', routing_key='high'),
    'default': Queue('default', routing_key='default'),
    'low': Queue('low', routing_key='low'),
}
```

### 3.3 Retry-Strategie
```python
@celery.task(
    bind=True,
    max_retries=3,
    default_retry_delay=60,  # 1 Minute
    retry_backoff=True,      # Exponentiell
    retry_jitter=True        # Randomisierung
)
```

---

## 4. PostgreSQL Datenbank

### 4.1 Erweiterungen
```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";    -- UUIDs
CREATE EXTENSION IF NOT EXISTS "pgcrypto";     -- Verschlüsselung
CREATE EXTENSION IF NOT EXISTS "pg_trgm";      -- Fuzzy Search
CREATE EXTENSION IF NOT EXISTS "btree_gin";    -- GIN Indexes
```

### 4.2 Connection Pool
```
Konfiguration:
- Pool Size: 20
- Max Overflow: 10
- Pool Recycle: 300s
- Pool Pre-Ping: True
```

### 4.3 Backup-Strategie
```
Automatische Backups:
- Vollbackup: Täglich um 02:00
- Inkrementell: Alle 6 Stunden
- WAL-Archivierung: Kontinuierlich
- Aufbewahrung: 30 Tage
```

---

## 5. Redis Cache

### 5.1 Verwendungszwecke
```
Datenbank 0: Session Storage
Datenbank 1: Cache (Berechnungen)
Datenbank 2: Rate Limiting
Datenbank 3: Celery Broker
Datenbank 4: Pub/Sub (WebSocket)
```

### 5.2 Cache-Strategien
```python
# TTL-Konfiguration
CACHE_TTL = {
    'user_profile': 3600,        # 1 Stunde
    'project_summary': 1800,     # 30 Minuten
    'calculation_result': 7200,  # 2 Stunden
    'report_data': 3600,         # 1 Stunde
    'climate_data': 86400,       # 24 Stunden
}
```

### 5.3 Invalidierungsregeln
```
Automatische Invalidierung bei:
- Benutzer-Update → user_profile:*
- Projekt-Update → project_*
- Gebäude-Update → building_*, calculation_*
- Berechnung-Update → calculation_*
```

---

## 6. MinIO Object Storage

### 6.1 Bucket-Struktur
```
Buckets:
├── documents/           # Hochgeladene Dokumente
│   ├── {org_id}/
│   │   └── {project_id}/
├── reports/             # Generierte Berichte
│   ├── {year}/
│   │   └── {month}/
├── templates/           # Vorlagen
│   ├── reports/
│   └── documents/
├── exports/             # Temporäre Exports
│   └── {date}/
└── backups/             # DB-Backups
    └── {date}/
```

### 6.2 Lifecycle-Regeln
```
- exports/*: Löschen nach 24 Stunden
- backups/*: Verschieben zu Glacier nach 30 Tagen
- reports/*: Versionierung aktiviert
```

### 6.3 Zugriffskontrolle
```
Policies:
- documents: Nur authentifizierte User
- reports: Lese-Zugriff nach Projekt-Berechtigung
- templates: Nur Admins schreiben
```

---

## 7. Ollama LLM

### 7.1 Modell-Konfiguration
```
Empfohlene Modelle:
- Mistral 7B: Allgemeine Textgenerierung
- CodeLlama 7B: Code-Generierung
- Llama2 13B: Komplexe Analysen (wenn RAM verfügbar)

Minimale Anforderungen:
- RAM: 16 GB (7B Modelle)
- RAM: 32 GB (13B Modelle)
- GPU: Optional aber empfohlen
```

### 7.2 API-Endpunkte
```
POST /api/generate     # Textgenerierung
POST /api/embeddings   # Embeddings
POST /api/chat         # Chat-Completion
GET  /api/tags         # Verfügbare Modelle
```

### 7.3 Prompt-Management
```
Verwendung:
- Berichtstexte generieren
- Maßnahmenempfehlungen
- Zusammenfassungen
- Übersetzungen
```

---

## 8. ChromaDB Vektordatenbank

### 8.1 Collections
```
Collections:
├── report_templates     # Berichtsvorlagen
├── building_components  # Bauteil-Beschreibungen
├── regulations          # Normen und Vorschriften
├── recommendations      # Maßnahmenempfehlungen
└── user_documents       # Hochgeladene Dokumente
```

### 8.2 Embedding-Konfiguration
```python
# sentence-transformers
EMBEDDING_MODEL = "sentence-transformers/paraphrase-multilingual-mpnet-base-v2"
EMBEDDING_DIMENSION = 768
```

### 8.3 Suchstrategien
```
- Semantische Suche: Ähnliche Dokumente finden
- Hybrid-Suche: Kombination aus Keyword + Semantisch
- RAG: Retrieval Augmented Generation für KI
```

---

## 9. Monitoring & Logging

### 9.1 Prometheus Metriken
```
Kategorien:
- http_requests_total
- http_request_duration_seconds
- calculation_duration_seconds
- cache_hits_total
- cache_misses_total
- db_connections_active
- celery_tasks_total
```

### 9.2 Grafana Dashboards
```
Dashboards:
├── System Overview
├── API Performance
├── Database Metrics
├── Cache Efficiency
├── Worker Status
└── Business KPIs
```

### 9.3 Logging-Format
```json
{
  "timestamp": "2025-01-15T10:30:00Z",
  "level": "INFO",
  "service": "api",
  "request_id": "uuid",
  "user_id": "uuid",
  "message": "...",
  "extra": {}
}
```

---

*Letzte Aktualisierung: 2025-11-25*
