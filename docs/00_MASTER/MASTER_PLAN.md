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

### 3.1 Backend
```
Framework:      FastAPI (Python 3.11+)
Datenbank:      PostgreSQL 16 + TimescaleDB
Cache:          Redis 7
Task Queue:     Celery + Redis
Dateispeicher:  MinIO (S3-kompatibel)
```

### 3.2 Frontend
```
Framework:      Vanilla JS + Alpine.js (leichtgewichtig)
Styling:        Tailwind CSS + Custom Design System
Charts:         D3.js + Chart.js
PDF:            PDF.js + html2pdf
Icons:          Lucide Icons (SVG)
```

### 3.3 KI-Integration
```
LLM:            Ollama (lokal) / OpenAI API (optional)
Embeddings:     sentence-transformers (lokal)
Vektor-DB:      ChromaDB
```

### 3.4 Infrastruktur
```
Container:      Docker + Docker Compose
Reverse Proxy:  Caddy (automatisches HTTPS)
Monitoring:     Prometheus + Grafana
Logging:        Loki
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
| Angebotswesen | Angebote, Vorlagen | MITTEL |
| Rechnungswesen | Rechnungen, Mahnwesen | MITTEL |
| Zeiterfassung | Projektzeiten, Auswertung | MITTEL |
| Dokumentenmanagement | Ablage, Versionierung | MITTEL |

### 4.4 Ausgabemodule (Phase 4)
| Modul | Beschreibung | Priorität |
|-------|--------------|-----------|
| Berichtsgenerator | Energieausweise, Audits | KRITISCH |
| KI-Textassistent | Beschreibungen, Empfehlungen | HOCH |
| Export | PDF, Excel, XML (BAFA) | HOCH |

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

### Phase 1: Fundament (Wochen 1-4)
- [ ] Docker-Umgebung aufsetzen
- [ ] Datenbank-Schema erstellen
- [ ] Authentifizierung implementieren
- [ ] Grundlegendes UI-Framework

### Phase 2: Kernfunktionen (Wochen 5-12)
- [ ] Projektmanagement
- [ ] Gebäudeerfassung
- [ ] Basisberechnungen (U-Werte, Flächen)
- [ ] Erste Berichte

### Phase 3: Berechnungsmodule (Wochen 13-24)
- [ ] Heizlast (DIN EN 12831)
- [ ] Energiebilanz (DIN V 18599)
- [ ] Erneuerbare Energien
- [ ] CO2-Bilanzierung

### Phase 4: Geschäftslogik (Wochen 25-32)
- [ ] Angebotswesen
- [ ] Rechnungswesen
- [ ] Zeiterfassung
- [ ] Dokumentenmanagement

### Phase 5: KI & Optimierung (Wochen 33-40)
- [ ] KI-Textgenerierung
- [ ] Automatisierte Analysen
- [ ] Performance-Optimierung
- [ ] Finale Tests

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

*Letzte Aktualisierung: 2025-11-25*
*Version: 1.0.0*
