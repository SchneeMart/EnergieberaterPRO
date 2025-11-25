# EnergieberaterPRO - Master-Projektplan

## Vision
Eine allumfassende, wissenschaftlich fundierte Webanwendung für professionelle Energieberatung, die alle Berechnungen nach deutschen und europäischen Normen durchführt, modernste KI-Integration bietet und dabei benutzerfreundlich, portabel und sicher ist.

---

## 1. Projektziele

### 1.1 Primäre Ziele
- **Vollständige Energieberatungs-Suite**: Alle Berechnungen von Gebäudeenergie bis CO2-Bilanzierung
- **Normkonformität**: DIN V 18599, GEG 2024, EN ISO 52000-Serie
- **Wissenschaftliche Präzision**: Validierte Berechnungsverfahren mit Quellenangaben
- **Benutzerfreundlichkeit**: Intuitive Oberfläche, keine IT-Kenntnisse erforderlich
- **Portabilität**: Docker-basiert, überall installierbar, offline-fähig

### 1.2 Sekundäre Ziele
- KI-gestützte Berichtsgenerierung und Analyse
- Automatisierte Workflows für wiederkehrende Aufgaben
- Integriertes Projektmanagement und Abrechnung
- Mandantenfähigkeit für Beratungsunternehmen

---

## 2. Technologie-Stack

### 2.1 Backend
```
Framework:      FastAPI (Python 3.11+)
Datenbank:      PostgreSQL 16 + TimescaleDB
Cache:          Redis 7
Task Queue:     Celery + Redis
Dateispeicher:  MinIO (S3-kompatibel)
```

### 2.2 Frontend
```
Framework:      Vanilla JS + Alpine.js (leichtgewichtig)
Styling:        Tailwind CSS + Custom Design System
Charts:         D3.js + Chart.js
PDF:            PDF.js + html2pdf
Icons:          Lucide Icons (SVG)
```

### 2.3 KI-Integration
```
LLM:            Ollama (lokal) / OpenAI API (optional)
Embeddings:     sentence-transformers (lokal)
Vektor-DB:      ChromaDB
```

### 2.4 Infrastruktur
```
Container:      Docker + Docker Compose
Reverse Proxy:  Caddy (automatisches HTTPS)
Monitoring:     Prometheus + Grafana
Logging:        Loki
```

---

## 3. Hauptmodule

### 3.1 Kernmodule (Phase 1)
| Modul | Beschreibung | Priorität |
|-------|--------------|-----------|
| Auth & Benutzer | Anmeldung, Profile, Rollen | KRITISCH |
| Projektmanagement | Projekte, Standorte, Kunden | KRITISCH |
| Gebäudedaten | Gebäudeerfassung, Zonierung | KRITISCH |
| Basisberechnungen | U-Werte, Energiebedarf | KRITISCH |

### 3.2 Berechnungsmodule (Phase 2)
| Modul | Beschreibung | Priorität |
|-------|--------------|-----------|
| Heizlastberechnung | DIN EN 12831 | HOCH |
| Kühllastberechnung | VDI 2078 | HOCH |
| Energiebilanz | DIN V 18599 | KRITISCH |
| Erneuerbare Energien | PV, Solarthermie, WP | HOCH |
| CO2-Bilanzierung | GHG Protocol | HOCH |

### 3.3 Geschäftsmodule (Phase 3)
| Modul | Beschreibung | Priorität |
|-------|--------------|-----------|
| Angebotswesen | Angebote, Vorlagen | MITTEL |
| Rechnungswesen | Rechnungen, Mahnwesen | MITTEL |
| Zeiterfassung | Projektzeiten, Auswertung | MITTEL |
| Dokumentenmanagement | Ablage, Versionierung | MITTEL |

### 3.4 Ausgabemodule (Phase 4)
| Modul | Beschreibung | Priorität |
|-------|--------------|-----------|
| Berichtsgenerator | Energieausweise, Audits | KRITISCH |
| KI-Textassistent | Beschreibungen, Empfehlungen | HOCH |
| Export | PDF, Excel, XML (BAFA) | HOCH |

---

## 4. Ordnerstruktur (Implementierung)

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

## 5. Datenmodell (Übersicht)

### 5.1 Kernentitäten
```
Benutzer (users)
├── Profile (profiles)
├── Organisationen (organizations)
└── Rollen (roles)

Projekte (projects)
├── Standorte (locations)
├── Gebäude (buildings)
│   ├── Zonen (zones)
│   ├── Bauteile (components)
│   └── Anlagen (systems)
├── Berechnungen (calculations)
└── Dokumente (documents)

Geschäft (business)
├── Kunden (customers)
├── Angebote (quotes)
├── Rechnungen (invoices)
└── Zeiteinträge (time_entries)
```

---

## 6. Entwicklungsphasen

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

## 7. Qualitätskriterien

### 7.1 Performance
- Seitenladezeit: < 2 Sekunden
- API-Antwortzeit: < 200ms (95. Perzentil)
- Berechnung Energiebilanz: < 5 Sekunden

### 7.2 Sicherheit
- OWASP Top 10 Compliance
- DSGVO-Konformität
- Verschlüsselung at-rest und in-transit
- Regelmäßige Security Audits

### 7.3 Benutzerfreundlichkeit
- Barrierefreiheit: WCAG 2.1 AA
- Mobile-First Responsive Design
- Maximal 3 Klicks zu jeder Funktion
- Kontexthilfe überall verfügbar

---

## 8. Referenzen & Normen

### 8.1 Technische Normen
- DIN V 18599 (Energetische Bewertung von Gebäuden)
- DIN EN 12831 (Heizlastberechnung)
- DIN EN ISO 6946 (Wärmedurchlasswiderstand)
- VDI 2078 (Kühllastberechnung)
- GEG 2024 (Gebäudeenergiegesetz)

### 8.2 Weitere Standards
- EN ISO 52000-Serie (EPBD Berechnungen)
- GHG Protocol (Treibhausgasbilanzierung)
- ISO 50001 (Energiemanagement)

---

## 9. Nächste Schritte

1. **Architektur detaillieren** → `01_ARCHITEKTUR/`
2. **Datenmodell ausarbeiten** → `08_DATENBANK/`
3. **Berechnungsalgorithmen spezifizieren** → `04_ENERGIEBERECHNUNG/`
4. **UI/UX Design erstellen** → `10_UI_UX/`
5. **Docker-Setup definieren** → `09_DEPLOYMENT/`

---

*Letzte Aktualisierung: 2025-11-25*
*Version: 0.1.0*
