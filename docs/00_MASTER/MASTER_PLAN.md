# EnergieberaterPRO - Master-Projektplan

## Vision
Eine allumfassende, wissenschaftlich fundierte **SaaS-Webanwendung** für professionelle Energieberatung in **Deutschland und Österreich**, die alle Berechnungen nach deutschen (DIN), österreichischen (ÖNORM) und europäischen Normen durchführt, modernste KI-Integration bietet und dabei benutzerfreundlich, portabel und sicher ist.

---

## 1. Geschäftsmodell und Benutzer-Hierarchie

### 1.1 SaaS-Plattform-Struktur

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  EBENE 1: SYSADMIN = WEBSITEBETREIBER (Plattform-Operator)                   │
│  ════════════════════════════════════════════════════════                    │
│  • Betreibt die SaaS-Plattform "EnergieberaterPRO"                           │
│  • Verkauft Abonnements an Organisationen                                    │
│  • Organisationen sind die KUNDEN des Sysadmins                              │
│  • Hat Vollzugriff auf alle System-Einstellungen                             │
│                                                                              │
│   ┌──────────────────────────────────────────────────────────────────────┐   │
│   │  EBENE 2: ORGANISATION = KUNDE DES SYSADMINS                         │   │
│   │  ════════════════════════════════════════════                         │   │
│   │  • Energieberatungsbüros, die die Plattform abonnieren               │   │
│   │  • Zahlen monatlich/jährlich für die Nutzung                         │   │
│   │  • Haben Mitarbeiter (Org-Admin, Mitarbeiter)                        │   │
│   │  • Haben eigene ENDKUNDEN (Hausbesitzer, Firmen)                     │   │
│   │                                                                      │   │
│   │   ┌──────────────────────────────────────────────────────────────┐   │   │
│   │   │  EBENE 3: ENDKUNDE = KUNDE DER ORGANISATION                  │   │   │
│   │   │  ═══════════════════════════════════════════                  │   │   │
│   │   │  • Hausbesitzer, Unternehmen, WEGs                           │   │   │
│   │   │  • Beauftragen Energieberatungen                             │   │   │
│   │   │  • ⚠️ Haben KEINEN Login auf der Plattform!                   │   │   │
│   │   │  • Werden als Datensatz in Projekten erfasst                 │   │   │
│   │   └──────────────────────────────────────────────────────────────┘   │   │
│   └──────────────────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────────────────┘
```

**Referenz:** Siehe `03_FRONTEND/TERMINOLOGIE_KORREKTUR.md` für detaillierte Begriffsdefinitionen.

---

## 2. Projektziele

### 2.1 Primäre Ziele
- **Vollständige Energieberatungs-Suite**: Alle Berechnungen von Gebäudeenergie bis CO2-Bilanzierung
- **Normkonformität Deutschland**: DIN V 18599, DIN EN 12831, GEG 2024
- **Normkonformität Österreich**: ÖNORM B 8110, ÖNORM H 5050-5060, OIB-Richtlinie 6
- **EU-Normen**: EN ISO 52000-Serie, EN ISO 6946
- **Wissenschaftliche Präzision**: Validierte Berechnungsverfahren mit Quellenangaben
- **Benutzerfreundlichkeit**: Intuitive Oberfläche, keine IT-Kenntnisse erforderlich
- **Portabilität**: Docker-basiert, überall installierbar, offline-fähig (PWA)

### 2.2 Sekundäre Ziele
- KI-gestützte Berichtsgenerierung und Analyse
- Automatisierte Workflows für wiederkehrende Aufgaben
- Integriertes Projektmanagement und Abrechnung
- Multi-Tenant-Architektur mit strikter Datenisolation

---

## 3. Technologie-Stack

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           TECHNOLOGIE-STACK                                      │
├──────────────────────────┬──────────────────────────────────────────────────────┤
│        BACKEND           │                    FRONTEND                           │
│  ┌────────────────────┐  │  ┌────────────────────────────────────────────────┐  │
│  │  FastAPI           │  │  │  Alpine.js + Vanilla JS                        │  │
│  │  Python 3.11+      │  │  │  Tailwind CSS + Custom Design System           │  │
│  │  PostgreSQL 16     │  │  │  D3.js + Chart.js (Visualisierung)             │  │
│  │  Redis 7           │  │  │  Yjs (CRDT Echtzeit-Kollaboration)            │  │
│  │  Celery            │  │  │  PDF.js + html2pdf                             │  │
│  │  MinIO             │  │  │  Lucide Icons                                  │  │
│  └────────────────────┘  │  └────────────────────────────────────────────────┘  │
├──────────────────────────┼──────────────────────────────────────────────────────┤
│          KI              │                 INFRASTRUKTUR                         │
│  ┌────────────────────┐  │  ┌────────────────────────────────────────────────┐  │
│  │  Ollama (lokal)    │  │  │  Docker + Docker Compose                       │  │
│  │  OpenAI (optional) │  │  │  Caddy (Auto-HTTPS)                            │  │
│  │  Tesseract OCR     │  │  │  Prometheus + Grafana                          │  │
│  │  ChromaDB          │  │  │  Loki (Logging)                                │  │
│  │  sentence-transf.  │  │  │  MinIO (S3-kompatibel)                         │  │
│  └────────────────────┘  │  └────────────────────────────────────────────────┘  │
└──────────────────────────┴──────────────────────────────────────────────────────┘
```

### 3.1 Backend
```
Framework:      FastAPI (Python 3.11+)
Datenbank:      PostgreSQL 16 + TimescaleDB (Zeitreihen)
Cache:          Redis 7
Task Queue:     Celery + Redis
Dateispeicher:  MinIO (S3-kompatibel)
Suche:          PostgreSQL Full-Text + pg_trgm
Scheduling:     APScheduler / Celery Beat
```

### 3.2 Frontend
```
Framework:      Vanilla JS + Alpine.js (leichtgewichtig)
Styling:        Tailwind CSS + Custom Design System
Charts:         D3.js + Chart.js
PDF:            PDF.js + html2pdf
Icons:          Lucide Icons (SVG)
Kollaboration:  Yjs (CRDT für Echtzeit-Bearbeitung)
Formulare:      JSON Schema + Alpine-Validierung
PWA:            Service Worker für Offline-Fähigkeit
```

### 3.3 KI-Integration
```
LLM:            Ollama (lokal) / OpenAI API / Claude API (optional)
Embeddings:     sentence-transformers (paraphrase-multilingual)
Vektor-DB:      ChromaDB
OCR:            Tesseract + EasyOCR (Typenschilder)
Computer Vision: OpenCV + YOLO (Plananalyse)
RAG:            LangChain / LlamaIndex
```

### 3.4 Integrationen
```
Buchhaltung:    DATEV-Export, Lexoffice API, sevDesk API, BMD (AT)
Kalender:       CalDAV, Google Calendar API, Microsoft Graph
IoT:            MQTT-Broker, REST-APIs für Smart Meter
E-Mail:         SMTP, IMAP (Eingangsverarbeitung)
Signaturen:     DocuSign API / lokale Signatur (eIDAS)
```

### 3.5 Infrastruktur
```
Container:      Docker + Docker Compose
Reverse Proxy:  Caddy (automatisches HTTPS)
Monitoring:     Prometheus + Grafana
Logging:        Loki + Promtail
Backup:         restic + MinIO
CI/CD:          GitHub Actions / GitLab CI
```

---

## 4. Hauptmodule

### 4.1 Kernmodule (Phase 1)
| Modul | Beschreibung | Priorität |
|-------|--------------|-----------|
| Auth & Benutzer | Anmeldung, Profile, Rollen | KRITISCH |
| Projektmanagement | Projekte, Standorte, Kunden | KRITISCH |
| Gebäudedaten | Gebäudeerfassung, Zonierung | KRITISCH |
| Basisberechnungen | U-Werte, Energiebedarf | KRITISCH |

### 4.2 Berechnungsmodule (Phase 2)
| Modul | Beschreibung | Priorität |
|-------|--------------|-----------|
| Heizlastberechnung | DIN EN 12831 | HOCH |
| Kühllastberechnung | VDI 2078 | HOCH |
| Energiebilanz | DIN V 18599 | KRITISCH |
| Erneuerbare Energien | PV, Solarthermie, WP | HOCH |
| CO2-Bilanzierung | GHG Protocol | HOCH |

### 4.3 Geschäftsmodule (Phase 3)
| Modul | Beschreibung | Priorität |
|-------|--------------|-----------|
| Angebotswesen | Angebote, Vorlagen, digitale Signatur | MITTEL |
| Rechnungswesen | Rechnungen, Mahnwesen, DATEV-Export | MITTEL |
| Zeiterfassung | Automatische Zeiterfassung nach Modulnutzung | MITTEL |
| Dokumentenmanagement | Ablage, Versionierung, Volltextsuche | MITTEL |

### 4.4 Ausgabemodule (Phase 4)
| Modul | Beschreibung | Priorität |
|-------|--------------|-----------|
| Berichtsgenerator | Energieausweise, Audits, iSFP | KRITISCH |
| KI-Textassistent | Beschreibungen, Empfehlungen | HOCH |
| Export | PDF, Excel, XML (BAFA, dena) | HOCH |

### 4.5 Erweiterte Berechnungsmodule (Phase 5)
| Modul | Beschreibung | Priorität |
|-------|--------------|-----------|
| Tauwasserschutz | Glaser-Verfahren, DIN 4108-3 | HOCH |
| Sommerlicher Wärmeschutz | DIN 4108-2, Überhitzungsnachweis | HOCH |
| Lebenszyklusanalyse (LCA) | Graue Energie, CO2-Fußabdruck Baustoffe | MITTEL |
| Tageslichtberechnung | Tageslichtquotienten, EN 17037 | MITTEL |
| Feuchteschutz erweitert | Hygrothermische Simulation | NIEDRIG |

### 4.6 Kollaborationsmodule (Phase 5)
| Modul | Beschreibung | Priorität |
|-------|--------------|-----------|
| Echtzeit-Bearbeitung | CRDT-basiert (Yjs), Multi-User Editing | HOCH |
| Kommentar-System | Inline-Kommentare, @-Erwähnungen | MITTEL |
| Endkunden-Portal | Read-Only Projektstatus, Dokumenten-Upload | MITTEL |
| Kalender-Integration | CalDAV, Google Calendar, Outlook | MITTEL |

### 4.7 Erweiterte KI-Module (Phase 6)
| Modul | Beschreibung | Priorität |
|-------|--------------|-----------|
| OCR & Computer Vision | Plandigitalisierung, Typenschilder lesen | HOCH |
| Predictive Analytics | Verbrauchsprognosen, Wartungsvorhersagen | MITTEL |
| Fördermittel-KI | Automatische Förderprogramm-Suche & Matching | HOCH |
| Sprachassistent | Voice-to-Text Eingabe, Diktieren | NIEDRIG |

### 4.8 Integrationsmodule (Phase 6)
| Modul | Beschreibung | Priorität |
|-------|--------------|-----------|
| Buchhaltung | DATEV, Lexoffice, sevDesk, BMD (AT) | HOCH |
| IoT-Integration | Smart Meter, Wetterstationen, Sensoren | MITTEL |
| No-Code Workflows | Visueller Workflow-Editor, Trigger/Actions | MITTEL |
| BIM-Import | IFC-Dateien, Revit-Integration | NIEDRIG |

### 4.9 Ökosystem-Module (Phase 7)
| Modul | Beschreibung | Priorität |
|-------|--------------|-----------|
| Plugin-System | Erweiterbare Architektur, API für Plugins | MITTEL |
| Vorlagen-Marktplatz | Berichtsvorlagen kaufen/verkaufen | NIEDRIG |
| Benchmarking | Anonymisierter Gebäudevergleich | MITTEL |
| Partner-API | White-Label, Reseller-Integration | NIEDRIG |

---

## 5. Ordnerstruktur (Implementierung)

```
EnergieberaterPRO/
├── docker-compose.yml
├── .env.example
├── data/                      # Persistente Daten (außerhalb Docker)
│   ├── postgres/
│   ├── redis/
│   ├── minio/
│   ├── uploads/
│   └── backups/
├── backend/
│   ├── app/
│   │   ├── api/              # REST API Endpoints
│   │   ├── core/             # Konfiguration, Sicherheit
│   │   ├── models/           # SQLAlchemy Models
│   │   ├── schemas/          # Pydantic Schemas
│   │   ├── services/         # Business Logic
│   │   ├── calculations/     # Energieberechnungen
│   │   ├── ai/               # KI-Integration
│   │   └── utils/            # Hilfsfunktionen
│   ├── tests/
│   ├── alembic/              # DB Migrationen
│   └── requirements.txt
├── frontend/
│   ├── index.html
│   ├── assets/
│   │   ├── css/
│   │   ├── js/
│   │   ├── svg/
│   │   └── fonts/
│   ├── components/           # Wiederverwendbare Komponenten
│   ├── pages/                # Seitenvorlagen
│   └── templates/            # Berichtsvorlagen
├── nginx/                    # oder Caddy
│   └── Caddyfile
└── docs/                     # Diese Dokumentation
```

---

## 6. Datenmodell (Übersicht)

### 6.1 Kernentitäten

**Referenz:** Detailliertes Schema siehe `04_BERECHNUNGEN/DATENMODELL_GEBAEUDE.md`

```
Plattform-Ebene (Sysadmin)
├── Organisationen (organizations)    ← Kunden des Sysadmins
├── Abonnements (subscriptions)
└── Module (modules)

Organisations-Ebene
├── Benutzer (users)
│   ├── Org-Admin
│   └── Mitarbeiter
├── Endkunden (customers)             ← Kunden der Organisation (KEIN LOGIN!)
├── Projekte (projects)
│   ├── Standorte (locations)
│   ├── Gebäude (buildings)
│   │   ├── Gebäudeversionen (building_versions)
│   │   ├── Zonen (zones)
│   │   ├── Bauteile (components)
│   │   └── Anlagen (systems)
│   └── Berechnungen (calculations)
└── Dokumente (documents)

Geschäft (Organisation → Endkunde)
├── Aufträge (orders)
├── Angebote (quotes)
├── Rechnungen (invoices)
└── Zeiteinträge (time_entries)
```

**Länderunterstützung:** Jede Organisation hat ein `country`-Feld (DE/AT) das die anzuwendenden Normen bestimmt.

---

## 7. Entwicklungsphasen

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           ENTWICKLUNGS-ROADMAP                                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  Phase 1 ──▶ Phase 2 ──▶ Phase 3 ──▶ Phase 4 ──▶ Phase 5 ──▶ Phase 6 ──▶ Phase 7│
│  Fundament   Kern       Berechnung  Geschäft   Erweitert   KI & Int.   Ökosystem│
│                                                                                  │
│  ████████    ████████   ████████    ████████   ████████    ████████    ░░░░░░░░ │
│  KRITISCH    KRITISCH   KRITISCH    HOCH       HOCH        MITTEL      NIEDRIG  │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Phase 1: Fundament
- [ ] Docker-Umgebung mit allen Services aufsetzen
- [ ] PostgreSQL mit Row-Level-Security konfigurieren
- [ ] Authentifizierung (JWT, MFA, RBAC) implementieren
- [ ] Grundlegendes UI-Framework (Alpine.js, Tailwind CSS)
- [ ] Design-System und Komponenten-Bibliothek

### Phase 2: Kernfunktionen
- [ ] Multi-Tenant Organisations- und Benutzerverwaltung
- [ ] Projektmanagement mit Vorlagensystem
- [ ] Gebäudeerfassung mit Zonierung
- [ ] Basisberechnungen (U-Werte, Flächen, Volumen)
- [ ] Erste Berichtsvorlagen (PDF-Export)

### Phase 3: Berechnungsmodule
- [ ] Heizlast (DIN EN 12831 / ÖNORM B 8110)
- [ ] Kühllast (VDI 2078)
- [ ] Energiebilanz (DIN V 18599 / ÖNORM H 5050)
- [ ] Erneuerbare Energien (PV, Solarthermie, Wärmepumpen)
- [ ] CO2-Bilanzierung (GHG Protocol)
- [ ] Wirtschaftlichkeitsberechnungen

### Phase 4: Geschäftslogik
- [ ] Endkunden-Verwaltung (CRM-Funktionen)
- [ ] Angebotswesen mit digitaler Signatur
- [ ] Rechnungswesen mit DATEV-Export
- [ ] Automatische Zeiterfassung
- [ ] Dokumentenmanagement mit Versionierung

### Phase 5: Erweiterte Module
- [ ] Tauwasserschutz (Glaser-Verfahren)
- [ ] Sommerlicher Wärmeschutz
- [ ] Lebenszyklusanalyse (LCA)
- [ ] Echtzeit-Kollaboration (Yjs/CRDT)
- [ ] Kommentar-System und Annotationen
- [ ] Endkunden-Portal (Read-Only)

### Phase 6: KI & Integrationen
- [ ] Basis-KI (Textgenerierung, Empfehlungen)
- [ ] OCR/Computer Vision (Pläne, Typenschilder)
- [ ] Fördermittel-KI mit automatischem Matching
- [ ] Predictive Analytics
- [ ] Buchhaltungs-Integrationen (DATEV, Lexoffice, BMD)
- [ ] IoT-Integration (Smart Meter, Sensoren)
- [ ] No-Code Workflow-Editor

### Phase 7: Ökosystem
- [ ] Plugin-System und Erweiterungs-API
- [ ] Vorlagen-Marktplatz
- [ ] Benchmarking-System
- [ ] Partner-API und White-Label
- [ ] BIM-Import (IFC)

---

## 8. Qualitätskriterien

### 8.1 Performance
- Seitenladezeit: < 2 Sekunden
- API-Antwortzeit: < 200ms (95. Perzentil)
- Berechnung Energiebilanz: < 5 Sekunden

### 8.2 Sicherheit
- OWASP Top 10 Compliance
- DSGVO-Konformität
- Verschlüsselung at-rest und in-transit
- Regelmäßige Security Audits

### 8.3 Benutzerfreundlichkeit
- Barrierefreiheit: WCAG 2.1 AA
- Mobile-First Responsive Design
- Maximal 3 Klicks zu jeder Funktion
- Kontexthilfe überall verfügbar

---

## 9. Referenzen & Normen

**Detaillierte Normen-Dokumentation:** Siehe `04_BERECHNUNGEN/NORMEN_UND_ANFORDERUNGEN.md`

### 9.1 Deutsche Normen (DE)
- **DIN V 18599** (Energetische Bewertung von Gebäuden)
- **DIN EN 12831** (Heizlastberechnung)
- **DIN 4108** (Wärmeschutz und Energie-Einsparung)
- **GEG 2024** (Gebäudeenergiegesetz)
- **BEG/BAFA** (Bundesförderung für effiziente Gebäude)

### 9.2 Österreichische Normen (AT)
- **ÖNORM B 8110** (Wärmeschutz im Hochbau)
- **ÖNORM H 5050-5060** (Energieausweis für Gebäude)
- **OIB-Richtlinie 6** (Energieeinsparung und Wärmeschutz)

### 9.3 EU-Normen (DE & AT)
- **EN ISO 52000-Serie** (EPBD Berechnungen)
- **DIN EN ISO 6946** (Wärmedurchlasswiderstand)
- **VDI 2078** (Kühllastberechnung)
- **GHG Protocol** (Treibhausgasbilanzierung)
- **ISO 50001** (Energiemanagement)

---

## 10. Dokumentation-Übersicht

| Ordner | Inhalt |
|--------|--------|
| `00_MASTER/` | Master-Plan, Anforderungskatalog |
| `01_ARCHITEKTUR/` | System-Architektur, Komponenten |
| `02_BACKEND/` | Backend-Architektur, Benutzer-Hierarchie |
| `03_FRONTEND/` | UI-Konzepte, Design-System, Terminologie |
| `04_BERECHNUNGEN/` | Normen, Produkte, Gebäude-Datenmodell |
| `05_GESCHAEFTSLOGIK/` | Geschäftsprozesse, Monetarisierung |
| `06_KI_INTEGRATION/` | KI-Konzept |
| `07_SICHERHEIT/` | OWASP, Multi-Tenancy |
| `08_DATENBANK/` | Datenbank-Schema |
| `09_DEPLOYMENT/` | Docker-Konfiguration |
| `10_UI_UX/` | Design-System |
| `11_TESTING/` | Test-Strategie |
| `12_AUTOMATISIERUNG/` | Automatisierung |
| `13_INTEGRATION/` | MCP, Browser-Erweiterung |

---

*Letzte Aktualisierung: 2025-11-26*
*Version: 2.0.0*
