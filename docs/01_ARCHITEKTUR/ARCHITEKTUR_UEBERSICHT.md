# Systemarchitektur - Übersicht

## 1. Architekturprinzipien

### 1.1 Leitprinzipien
- **Modularität**: Unabhängige, austauschbare Komponenten
- **Skalierbarkeit**: Horizontal skalierbar bei Bedarf
- **Portabilität**: Läuft auf jedem System mit Docker
- **Datensouveränität**: Alle Daten bleiben lokal
- **Offline-Fähigkeit**: Kernfunktionen ohne Internet nutzbar

### 1.2 Architekturstil
- **Schichtenarchitektur** mit klarer Trennung
- **API-First Design** für flexible Frontend-Entwicklung
- **Event-Driven** für asynchrone Verarbeitung
- **Domain-Driven Design** für Geschäftslogik

---

## 2. Systemübersicht

```
┌─────────────────────────────────────────────────────────────────┐
│                        FRONTEND LAYER                           │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              Single Page Application (SPA)                │  │
│  │         HTML5 + Alpine.js + Tailwind CSS + D3.js         │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      REVERSE PROXY (Caddy)                      │
│              Automatisches HTTPS, Load Balancing                │
└─────────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│   API Server    │ │  Static Files   │ │   WebSocket     │
│   (FastAPI)     │ │    (Caddy)      │ │   (FastAPI)     │
└─────────────────┘ └─────────────────┘ └─────────────────┘
              │                               │
              └───────────────┬───────────────┘
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       APPLICATION LAYER                          │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌───────────┐ │
│  │   Auth      │ │  Projects   │ │ Calculations│ │  Reports  │ │
│  │  Service    │ │  Service    │ │   Service   │ │  Service  │ │
│  └─────────────┘ └─────────────┘ └─────────────┘ └───────────┘ │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌───────────┐ │
│  │  Business   │ │     AI      │ │   Export    │ │ Scheduler │ │
│  │  Service    │ │   Service   │ │   Service   │ │  Service  │ │
│  └─────────────┘ └─────────────┘ └─────────────┘ └───────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                         DATA LAYER                               │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌───────────┐ │
│  │ PostgreSQL  │ │    Redis    │ │    MinIO    │ │ ChromaDB  │ │
│  │ (Haupt-DB)  │ │   (Cache)   │ │  (Dateien)  │ │ (Vektoren)│ │
│  └─────────────┘ └─────────────┘ └─────────────┘ └───────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     PERSISTENT STORAGE                           │
│                    ./data/ (Host Volume)                         │
│         Alle Daten außerhalb der Docker-Container               │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Komponentenbeschreibung

### 3.1 Frontend Layer

#### Single Page Application
- **Technologie**: Vanilla JavaScript mit Alpine.js
- **Vorteile**:
  - Kein Build-Prozess erforderlich
  - Geringe Bundle-Größe (~50KB)
  - Einfache Wartung und Anpassung
- **Komponenten**:
  - Router (clientseitig)
  - State Management (Alpine.store)
  - HTTP-Client (Fetch API)
  - WebSocket-Client

### 3.2 Reverse Proxy (Caddy)

#### Funktionen
- **Automatisches HTTPS**: Let's Encrypt Integration
- **Routing**: API vs. Static Content
- **Kompression**: Gzip/Brotli
- **Security Headers**: CSP, HSTS, etc.
- **Rate Limiting**: Schutz vor DDoS

### 3.3 Application Layer

#### API Server (FastAPI)
```
Kernfunktionen:
- REST API Endpoints
- OpenAPI Dokumentation (automatisch)
- Request Validation (Pydantic)
- Dependency Injection
- Background Tasks
```

#### Services
| Service | Verantwortung |
|---------|---------------|
| Auth | Authentifizierung, Autorisierung, JWT |
| Projects | Projektverwaltung, Standorte, Gebäude |
| Calculations | Alle Energieberechnungen |
| Reports | Berichtsgenerierung, Vorlagen |
| Business | Angebote, Rechnungen, Zeiterfassung |
| AI | LLM-Integration, Textgenerierung |
| Export | PDF, Excel, XML Export |
| Scheduler | Cronjobs, Hintergrundaufgaben |

### 3.4 Data Layer

#### PostgreSQL
- **Verwendung**: Alle strukturierten Daten
- **Erweiterungen**:
  - `uuid-ossp`: UUID-Generierung
  - `pgcrypto`: Verschlüsselung
  - `pg_trgm`: Volltextsuche
- **Features**:
  - JSONB für flexible Daten
  - Partitionierung für Zeitserien
  - Row-Level Security

#### Redis
- **Verwendung**:
  - Session Storage
  - Cache für Berechnungen
  - Rate Limiting Counter
  - Pub/Sub für WebSockets
  - Task Queue (Celery)

#### MinIO
- **Verwendung**:
  - Dokumente und Dateien
  - Generierte Berichte
  - Backup-Archive
- **S3-Kompatibel**: Standard-APIs

#### ChromaDB
- **Verwendung**:
  - Vektorspeicher für KI
  - Semantische Suche
  - RAG für Berichtstexte

---

## 4. Datenfluss

### 4.1 Typischer Request-Flow

```
┌──────────┐    ┌───────────┐    ┌──────────┐    ┌─────────────┐
│  Client  │───▶│   Caddy   │───▶│  FastAPI │───▶│   Service   │
│ (Browser)│    │  (Proxy)  │    │  (API)   │    │   Layer     │
└──────────┘    └───────────┘    └──────────┘    └─────────────┘
                                       │               │
                                       ▼               ▼
                                 ┌──────────┐    ┌─────────────┐
                                 │  Cache   │◀──▶│  Database   │
                                 │ (Redis)  │    │ (Postgres)  │
                                 └──────────┘    └─────────────┘
```

### 4.2 Berechnungs-Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                     BERECHNUNGS-PIPELINE                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Eingabedaten        2. Validierung       3. Normauswahl    │
│  ┌─────────────┐        ┌─────────────┐      ┌─────────────┐   │
│  │ Gebäudedaten│───────▶│  Pydantic   │─────▶│ DIN V 18599 │   │
│  │ Anlagen     │        │  Schemas    │      │ DIN EN 12831│   │
│  │ Klimadaten  │        │  Prüfung    │      │ VDI 2078    │   │
│  └─────────────┘        └─────────────┘      └─────────────┘   │
│                                                     │           │
│  6. Speicherung         5. Validierung       4. Berechnung     │
│  ┌─────────────┐        ┌─────────────┐      ┌─────────────┐   │
│  │ PostgreSQL  │◀───────│ Plausibilität│◀────│ Engine      │   │
│  │ + Cache     │        │ Grenzwerte  │      │ (NumPy)     │   │
│  └─────────────┘        └─────────────┘      └─────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.3 Berichtsgenerierungs-Flow

```
┌───────────────┐     ┌───────────────┐     ┌───────────────┐
│  Berechnung   │────▶│   Template    │────▶│   Renderer    │
│  Ergebnisse   │     │   Engine      │     │   (Jinja2)    │
└───────────────┘     └───────────────┘     └───────────────┘
                                                   │
       ┌───────────────────────────────────────────┤
       ▼                   ▼                       ▼
┌─────────────┐     ┌─────────────┐         ┌─────────────┐
│  KI-Text-   │     │   HTML/CSS  │         │    PDF      │
│  Generator  │────▶│   Aufberei- │────────▶│  Generator  │
│  (Ollama)   │     │   tung      │         │ (WeasyPrint)│
└─────────────┘     └─────────────┘         └─────────────┘
```

---

## 5. Sicherheitsarchitektur

### 5.1 Schichten

```
┌─────────────────────────────────────────────────────────────────┐
│ PERIMETER: Caddy Reverse Proxy                                  │
│ - TLS 1.3 Terminierung                                          │
│ - Rate Limiting                                                 │
│ - IP-Filterung (optional)                                       │
│ - Security Headers (CSP, HSTS)                                  │
├─────────────────────────────────────────────────────────────────┤
│ APPLICATION: FastAPI                                            │
│ - JWT Token Validierung                                         │
│ - RBAC (Role-Based Access Control)                              │
│ - Input Validation (Pydantic)                                   │
│ - CORS Kontrolle                                                │
├─────────────────────────────────────────────────────────────────┤
│ DATA: PostgreSQL                                                │
│ - Row-Level Security                                            │
│ - Verschlüsselte Spalten (sensible Daten)                       │
│ - Connection Pooling mit SSL                                    │
│ - Audit Logging                                                 │
├─────────────────────────────────────────────────────────────────┤
│ STORAGE: Filesystem                                             │
│ - Dateiverschlüsselung (optional)                               │
│ - Berechtigungskontrolle                                        │
│ - Backup-Verschlüsselung                                        │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Deployment-Architektur

### 6.1 Docker Compose Struktur

```yaml
services:
  # Reverse Proxy
  caddy:
    ports: ["80:80", "443:443"]

  # Backend
  api:
    replicas: 2  # Skalierbar

  worker:
    replicas: 2  # Hintergrundaufgaben

  # Datenbanken
  postgres:
    volumes: ["./data/postgres:/var/lib/postgresql/data"]

  redis:
    volumes: ["./data/redis:/data"]

  minio:
    volumes: ["./data/minio:/data"]

  # KI (optional)
  ollama:
    volumes: ["./data/ollama:/root/.ollama"]

  chromadb:
    volumes: ["./data/chromadb:/chroma"]
```

### 6.2 Netzwerk-Isolation

```
┌─────────────────────────────────────────────────────────────────┐
│                       Docker Network                             │
├─────────────────────────────────────────────────────────────────┤
│  frontend-net (Extern zugänglich)                               │
│  ┌─────────┐                                                    │
│  │  Caddy  │ ◄── Port 80/443                                   │
│  └─────────┘                                                    │
│       │                                                         │
├───────┼─────────────────────────────────────────────────────────┤
│  backend-net (Intern)                                           │
│       │                                                         │
│  ┌────┴────┐  ┌─────────┐  ┌─────────┐                        │
│  │   API   │  │  Worker │  │  Beat   │                        │
│  └─────────┘  └─────────┘  └─────────┘                         │
│       │           │            │                                │
├───────┼───────────┼────────────┼────────────────────────────────┤
│  data-net (Nur Datenbanken)                                     │
│       │           │            │                                │
│  ┌────┴───────────┴────────────┴────┐                          │
│  │  Postgres  │  Redis  │  MinIO    │                          │
│  └──────────────────────────────────┘                           │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. Erweiterbarkeit

### 7.1 Plugin-Architektur
- Module können unabhängig hinzugefügt werden
- Definierte Schnittstellen (Interfaces)
- Event-System für lose Kopplung

### 7.2 API-Versionierung
- URL-Prefix: `/api/v1/`, `/api/v2/`
- Breaking Changes nur in neuen Versionen
- Deprecation Policy: 6 Monate

---

## 8. Referenzdokumente

- [Systemkomponenten](./system/KOMPONENTEN_DETAIL.md)
- [Datenfluss-Diagramme](./datenfluss/DATENFLUSS_DETAIL.md)
- [Integrationsschnittstellen](./integration/INTEGRATION_DETAIL.md)

---

*Letzte Aktualisierung: 2025-11-25*
*Version: 0.1.0*
