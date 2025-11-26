# Vollständiger Anforderungskatalog

## Version 3.0 - November 2025

---

## 1. Projektvision

### 1.1 Mission
EnergieberaterPRO ist eine **allumfassende, wissenschaftlich fundierte SaaS-Webanwendung** für professionelle Energieberatung in **Deutschland und Österreich**, die:
- Alle Berechnungen nach deutschen (DIN), österreichischen (ÖNORM) und europäischen Normen durchführt
- Maximale Automatisierung bei minimaler Bürokratie bietet
- Als vollständig portable, selbst-gehostete Lösung funktioniert
- Höchste Sicherheitsstandards erfüllt
- Vollständig Open Source ist

**⚠️ PRIORISIERUNG ÖSTERREICH (Phase 1):**
```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  ÖSTERREICH-FIRST STRATEGIE                                                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│  • Default-Konfiguration: country='AT'                                          │
│  • Normen-Implementierung: OIB-Richtlinie 6 + ÖNORM H 5050 VOR DIN-Normen      │
│  • Validatoren primär für österreichische Grenzwerte                           │
│  • Deutsche DIN-Normen werden später als Erweiterung nachgerüstet              │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Benutzer-Hierarchie (SaaS-Modell)

**Referenz:** Siehe `03_FRONTEND/TERMINOLOGIE_KORREKTUR.md` für Details.

| Ebene | Rolle | Beschreibung | Login |
|-------|-------|--------------|-------|
| 1 | **Sysadmin** | Websitebetreiber/Plattform-Operator | ✓ |
| 2 | **Organisation** | Kunden des Sysadmins (Energieberatungsbüros) | ✓ |
| 2a | Org-Admin | Verwaltet Organisation | ✓ |
| 2b | Mitarbeiter | Arbeitet an Projekten | ✓ |
| 3 | **Endkunde** | Kunden der Organisation (Hausbesitzer, Firmen) | ✗ KEIN LOGIN |

⚠️ **WICHTIG:** Endkunden haben keinen Login auf der Plattform! Sie werden als Datensatz in Projekten erfasst.

### 1.3 Kernwerte
```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           KERNWERTE                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  1. WISSENSCHAFTLICHE PRÄZISION                                            │
│     └── Alle Berechnungen normkonform und validierbar                      │
│     └── Quellenangaben für alle Formeln und Faktoren                       │
│     └── Automatische Plausibilitätsprüfungen                               │
│                                                                             │
│  2. MAXIMALE AUTOMATISIERUNG                                               │
│     └── 0% bürokratischer Aufwand für Mitarbeiter                          │
│     └── Intelligente Workflows und Assistenten                             │
│     └── KI-Unterstützung wo sinnvoll                                       │
│                                                                             │
│  3. ABSOLUTE SICHERHEIT                                                    │
│     └── OWASP Top 10 2025 Compliance                                       │
│     └── BSI IT-Grundschutz konform                                         │
│     └── Strikte Mandantentrennung                                          │
│                                                                             │
│  4. VOLLSTÄNDIGE PORTABILITÄT                                              │
│     └── Docker-basiert, überall installierbar                              │
│     └── Offline-fähig (PWA)                                                │
│     └── Keine externen Abhängigkeiten für Kernfunktionen                   │
│                                                                             │
│  5. 100% OPEN SOURCE                                                       │
│     └── Alle Komponenten frei verwendbar                                    │
│     └── Keine proprietären Abhängigkeiten                                  │
│     └── Vollständige Lizenz-Dokumentation                                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Funktionale Anforderungen

### 2.1 Benutzer- und Organisationsverwaltung

#### 2.1.1 Mandantenfähigkeit
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| ORG-001 | Vollständige Datenisolation zwischen Organisationen | KRITISCH |
| ORG-002 | Eigene Subdomain/Branding pro Organisation | HOCH |
| ORG-003 | Organisationsweite Einstellungen und Vorlagen | HOCH |
| ORG-004 | Logo- und Vorlagenbearbeitung durch Org-Admin | HOCH |
| ORG-005 | Organisations-E-Mail-Versand-Konfiguration | HOCH |
| ORG-006 | Hierarchische Organisationsstruktur (Abteilungen) | MITTEL |

#### 2.1.2 Benutzerverwaltung
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| USR-001 | Rollenbasierte Zugriffskontrolle (RBAC) | KRITISCH |
| USR-002 | Granulare Funktionsfreischaltung pro Benutzer | HOCH |
| USR-003 | Multi-Faktor-Authentifizierung (TOTP) | HOCH |
| USR-004 | Single Sign-On (SAML, OIDC) | MITTEL |
| USR-005 | Benutzer-Aktivitätsprotokoll | HOCH |
| USR-006 | Automatische Session-Verwaltung | HOCH |

#### 2.1.3 Webtool-Administration
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| ADM-001 | Globale Systemkonfiguration | KRITISCH |
| ADM-002 | Organisations-Onboarding | HOCH |
| ADM-003 | Modul-/Funktionsfreischaltung pro Organisation | KRITISCH |
| ADM-004 | Abrechnungsverwaltung für Organisationen | HOCH |
| ADM-005 | System-Monitoring und Statistiken | HOCH |
| ADM-006 | Globale Stammdatenverwaltung | MITTEL |

---

### 2.2 Projektmanagement

#### 2.2.1 Projektverwaltung
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| PRJ-001 | Projekttypen (Audit, Ausweis, Konzept, etc.) | KRITISCH |
| PRJ-002 | Intelligente Projektvorlagen-Kombination | HOCH |
| PRJ-003 | Automatische Anforderungsableitung aus Projekttyp | HOCH |
| PRJ-004 | Projekt-Status-Workflow mit Freigaben | HOCH |
| PRJ-005 | Multi-Standort-Projekte | MITTEL |
| PRJ-006 | Projektduplizierung und -vorlagen | HOCH |

#### 2.2.2 Projekttyp-Kategorien (Norm vs. Flex)

```
┌────────────────────────────────────────────────────────────────────────────────┐
│  A. NORM-PROJEKTE                    │  B. FLEX-PROJEKTE                       │
│  ══════════════════                  │  ══════════════════                     │
│  • Starr, geführter Ablauf           │  • Frei gestaltbare Struktur            │
│  • Wizard-basiert im Frontend        │  • Template-Editor für Checklisten      │
│  • Validierung nach Norm-Vorgaben    │  • Eigene Ordnerstrukturen              │
│                                      │                                         │
│  Beispiele:                          │  Beispiele:                             │
│  - Energieausweis AT (ÖNORM H 5055)  │  - Individuelle Sanierungskonzepte      │
│  - Energieausweis DE (GEG 2024)      │  - Kundenspezifische Audits             │
│  - iSFP (BAFA)                       │  - Freie Beratungsprojekte              │
│  - BAFA-Energieaudit                 │  - Machbarkeitsstudien                  │
└────────────────────────────────────────────────────────────────────────────────┘
```

| ID | Anforderung | Priorität |
|----|-------------|-----------|
| PRJ-010 | Explizite Unterscheidung Norm-/Flex-Projekte im Datenmodell | KRITISCH |
| PRJ-011 | Wizard-UI für Norm-Projekte mit Pflichtschritten | HOCH |
| PRJ-012 | Template-Editor für Flex-Projekte (eigene Checklisten) | HOCH |
| PRJ-013 | Vordefinierte Norm-Vorlagen (AT: ÖNORM, DE: GEG) | KRITISCH |
| PRJ-014 | Freie Ordnerstruktur für Flex-Projekte | MITTEL |
| PRJ-015 | Umschaltung Norm↔Flex bei Projektanlage | HOCH |

#### 2.2.3 Intelligente Projektvorlagen
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| TPL-001 | Modulare Vorlagenbausteine | HOCH |
| TPL-002 | Automatische Kombinationslogik | HOCH |
| TPL-003 | Vorlagen pro Organisation anpassbar | MITTEL |
| TPL-004 | Versionierung von Vorlagen | MITTEL |

---

### 2.3 Energieberechnung

#### 2.3.1 Gebäudeerfassung
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| GEB-001 | Vollständige Gebäudedatenerfassung | KRITISCH |
| GEB-002 | Zonierung nach DIN V 18599 | KRITISCH |
| GEB-003 | Bauteilkatalog mit Schichtaufbau | HOCH |
| GEB-004 | Anlagentechnik-Erfassung | KRITISCH |
| GEB-005 | Foto- und Plandokumentation | HOCH |
| GEB-006 | 3D-Gebäudemodell (Import/Export) | NIEDRIG |

#### 2.3.2 Berechnungsmodule - Deutschland (DE)
| ID | Anforderung | Norm | Priorität |
|----|-------------|------|-----------|
| CALC-DE-001 | U-Wert-Berechnung | DIN EN ISO 6946 | KRITISCH |
| CALC-DE-002 | Heizlastberechnung | DIN EN 12831 | KRITISCH |
| CALC-DE-003 | Kühllastberechnung | VDI 2078 | HOCH |
| CALC-DE-004 | Energiebilanz | DIN V 18599 | KRITISCH |
| CALC-DE-005 | Primärenergieberechnung | GEG 2024 | KRITISCH |
| CALC-DE-006 | Energieausweis Wohngebäude | GEG 2024 | KRITISCH |
| CALC-DE-007 | Energieausweis Nichtwohngebäude | GEG 2024 | KRITISCH |
| CALC-DE-008 | iSFP (Sanierungsfahrplan) | BAFA | HOCH |

#### 2.3.3 Berechnungsmodule - Österreich (AT)
| ID | Anforderung | Norm | Priorität |
|----|-------------|------|-----------|
| CALC-AT-001 | U-Wert-Berechnung | ÖNORM B 8110-6 | KRITISCH |
| CALC-AT-002 | Heizwärmebedarf | ÖNORM B 8110-6 | KRITISCH |
| CALC-AT-003 | Kühlbedarf | ÖNORM B 8110-6 | HOCH |
| CALC-AT-004 | Energieausweis Wohngebäude | ÖNORM H 5055 | KRITISCH |
| CALC-AT-005 | Energieausweis Nichtwohngebäude | ÖNORM H 5056 | KRITISCH |
| CALC-AT-006 | Primärenergiebedarf | OIB-RL 6 | KRITISCH |
| CALC-AT-007 | CO2-Emissionen | ÖNORM H 5050 | HOCH |

#### 2.3.4 Berechnungsmodule - Länderneutral (DE & AT)
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| CALC-001 | CO2-Bilanzierung (GHG Protocol) | HOCH |
| CALC-002 | Wirtschaftlichkeitsberechnung | HOCH |
| CALC-003 | Erneuerbare-Energien-Berechnung | HOCH |
| CALC-004 | Lebenszyklusanalyse (LCA) | MITTEL |

#### 2.3.5 Validierung
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| VAL-001 | Automatische Plausibilitätsprüfung | KRITISCH |
| VAL-002 | GEG-Grenzwertprüfung | KRITISCH |
| VAL-003 | Kennwertvergleich mit Referenzen | HOCH |
| VAL-004 | Warnungen bei Ausreißern | HOCH |

---

### 2.4 Berichtswesen

#### 2.4.1 Berichtstypen
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| REP-001 | Energieausweis (Bedarf/Verbrauch) | KRITISCH |
| REP-002 | Energieaudit-Bericht | KRITISCH |
| REP-003 | Sanierungsfahrplan (iSFP) | HOCH |
| REP-004 | Heizlastberechnung-Bericht | HOCH |
| REP-005 | Machbarkeitsstudie | MITTEL |
| REP-006 | Benutzerdefinierte Berichte | MITTEL |

#### 2.4.2 Intelligente Berichtsbausteine
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| BST-001 | Modulare Textbausteine | HOCH |
| BST-002 | KI-generierte Beschreibungen | HOCH |
| BST-003 | Automatische Kennwert-Integration | KRITISCH |
| BST-004 | Dynamische Diagramme und Grafiken | HOCH |
| BST-005 | Bedingte Inhalte (wenn-dann) | MITTEL |

#### 2.4.3 Export
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| EXP-001 | PDF-Export (druckoptimiert) | KRITISCH |
| EXP-002 | Excel-Export | HOCH |
| EXP-003 | XML-Export (BAFA, dena) | HOCH |
| EXP-004 | API-Schnittstellen | MITTEL |

---

### 2.5 Geschäftsprozesse

#### 2.5.1 Endkunden-Verwaltung (Kunden der Organisation)

**Hinweis:** "Kunden" in diesem Kontext = Endkunden der Organisation (Hausbesitzer, Firmen), NICHT Kunden der Plattform.

| ID | Anforderung | Priorität |
|----|-------------|-----------|
| CRM-001 | Endkundentypen (Privat, Geschäft, Öffentlich, WEG) | HOCH |
| CRM-002 | Endkunden-Historie und Projekte | HOCH |
| CRM-003 | Kontaktverlauf | MITTEL |
| CRM-004 | DSGVO-konforme Datenverwaltung | KRITISCH |
| CRM-005 | Endkunden haben KEINEN Login | KRITISCH |

#### 2.5.2 Angebotswesen
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| ANG-001 | Angebotserstellung mit Vorlagen | HOCH |
| ANG-002 | Automatische Kalkulation | HOCH |
| ANG-003 | Digitale Signatur/Annahme | MITTEL |
| ANG-004 | Angebotsverfolgung | MITTEL |

#### 2.5.3 Rechnungswesen
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| INV-001 | Rechnungserstellung | HOCH |
| INV-002 | Automatischer Rechnungsversand (nach Bestätigung) | HOCH |
| INV-003 | Zahlungsverfolgung | HOCH |
| INV-004 | Mahnwesen | MITTEL |
| INV-005 | DATEV-Export | MITTEL |

#### 2.5.4 Zeiterfassung (Automatisch)
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| ZEI-001 | Automatische Zeiterfassung nach Modulnutzung | HOCH |
| ZEI-002 | Projekt-/Aufgabenzuordnung | HOCH |
| ZEI-003 | Keine manuelle Eingabe erforderlich | KRITISCH |
| ZEI-004 | Auswertung nach Modulen/Projekten | HOCH |
| ZEI-005 | Abrechnungsintegration | HOCH |

---

### 2.6 Automatisierung

#### 2.6.1 Workflow-Automatisierung
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| WFL-001 | Geführte Prozesse für alle Projekttypen | HOCH |
| WFL-002 | Automatische Aufgabenerstellung | HOCH |
| WFL-003 | Trigger-basierte Aktionen | MITTEL |
| WFL-004 | Batch-Verarbeitung | MITTEL |

#### 2.6.2 Kommunikations-Automatisierung
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| COM-001 | Auto-Mail-Versand mit KI-Kontext | HOCH |
| COM-002 | Erinnerungen und Benachrichtigungen | HOCH |
| COM-003 | Statusupdates an Kunden | MITTEL |
| COM-004 | Vorlagenverwaltung für E-Mails | HOCH |

#### 2.6.3 Datenerhebung
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| DAT-001 | Browser-Erweiterung für Formularbefüllung | HOCH |
| DAT-002 | Online-Fragebogen für Kunden | HOCH |
| DAT-003 | Dokumenten-OCR und Extraktion | MITTEL |
| DAT-004 | API-Integration externer Datenquellen | MITTEL |

---

### 2.7 KI-Integration

#### 2.7.1 Textgenerierung
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| KI-001 | Berichtstexte generieren (Energieausweise, Audits) | HOCH |
| KI-002 | Maßnahmenempfehlungen mit Kostenabschätzung | HOCH |
| KI-003 | E-Mail-Entwürfe mit Projektkontext | HOCH |
| KI-004 | Zusammenfassungen und Abstracts | MITTEL |
| KI-005 | Mehrsprachige Ausgabe (DE, EN) | NIEDRIG |

#### 2.7.2 Analyse & Predictive Analytics
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| KI-010 | Gebäudeanalyse und Bewertung | HOCH |
| KI-011 | Optimierungsvorschläge | MITTEL |
| KI-012 | Benchmarking mit anonymisierten Daten | MITTEL |
| KI-013 | Verbrauchsprognosen (ML-basiert) | MITTEL |
| KI-014 | Wartungsvorhersagen für Anlagen | NIEDRIG |
| KI-015 | Anomalie-Erkennung bei Verbrauchsdaten | NIEDRIG |

#### 2.7.3 OCR & Computer Vision
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| KI-020 | Dokumenten-OCR (Rechnungen, Pläne) | HOCH |
| KI-021 | Typenschilder-Erkennung (Heizung, PV) | HOCH |
| KI-022 | Plandigitalisierung (Grundrisse) | MITTEL |
| KI-023 | Fotoanalyse Gebäudezustand | NIEDRIG |
| KI-024 | Automatische Bauteil-Erkennung | NIEDRIG |

#### 2.7.4 Fördermittel-KI
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| KI-030 | Automatische Förderprogramm-Suche | HOCH |
| KI-031 | Matching Maßnahme → Förderprogramm | HOCH |
| KI-032 | Förderhöhen-Berechnung | HOCH |
| KI-033 | Antragsunterstützung (Checklisten) | MITTEL |
| KI-034 | Aktualisierung bei Programmänderungen | MITTEL |

#### 2.7.5 Tool-Integration (MCP)
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| MCP-001 | Model Context Protocol Integration | HOCH |
| MCP-002 | Eigene Tool-Erstellung für Agenten | MITTEL |
| MCP-003 | Web-Recherche mittels KI | MITTEL |
| MCP-004 | Automatisierte Datenextraktion | MITTEL |

---

### 2.8 Kollaboration

#### 2.8.1 Dokumentenbearbeitung
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| DOC-001 | Online-Dokumentenbearbeitung | HOCH |
| DOC-002 | Echtzeit-Kollaboration (CRDT) | HOCH |
| DOC-003 | Versionierung | HOCH |
| DOC-004 | Kommentare und Annotationen | MITTEL |

#### 2.8.2 Office-Integration (WOPI/Collabora)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  COLLABORA ONLINE INTEGRATION (WOPI-Protokoll)                                  │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  ┌─────────────┐    WOPI-API     ┌─────────────────────────────────────────────┐│
│  │   Backend   │◄───────────────►│  Collabora Online Development Edition      ││
│  │  (FastAPI)  │                 │  (CODE Docker Container)                   ││
│  └─────────────┘                 └─────────────────────────────────────────────┘│
│        │                                        │                               │
│        │                                        │                               │
│        ▼                                        ▼                               │
│  /api/wopi/files/{file_id}              IFrame im Frontend                      │
│  - CheckFileInfo                        - Live-Bearbeitung .xlsx/.docx         │
│  - GetFile                              - Multi-User-Editing                   │
│  - PutFile                              - Änderungen sofort gespeichert        │
│  - Lock/Unlock                                                                  │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

| ID | Anforderung | Priorität |
|----|-------------|-----------|
| WOPI-001 | Collabora CODE Container im Docker-Stack | HOCH |
| WOPI-002 | WOPI-Endpoints im Backend (/api/wopi/...) | HOCH |
| WOPI-003 | CheckFileInfo, GetFile, PutFile Implementierung | HOCH |
| WOPI-004 | File-Locking für gleichzeitige Bearbeitung | HOCH |
| WOPI-005 | IFrame-Einbettung im Frontend | HOCH |
| WOPI-006 | Unterstützte Formate: .xlsx, .docx, .odt, .ods | HOCH |
| WOPI-007 | Automatisches Speichern bei Änderungen | MITTEL |
| WOPI-008 | Bearbeitungshistorie und Versionen | MITTEL |

#### 2.8.3 Kommunikation
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| MSG-001 | Interne Nachrichten | MITTEL |
| MSG-002 | Gruppenchats/Kanäle | NIEDRIG |
| MSG-003 | @-Erwähnungen | NIEDRIG |

#### 2.8.4 Kalender
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| KAL-001 | Kalenderintegration (CalDAV) | MITTEL |
| KAL-002 | Terminplanung | MITTEL |
| KAL-003 | Ressourcenplanung | NIEDRIG |

---

### 2.9 Dokumentenverwaltung

| ID | Anforderung | Priorität |
|----|-------------|-----------|
| FIL-001 | Strukturierte Ablage | HOCH |
| FIL-002 | Freigabe und Berechtigungen | HOCH |
| FIL-003 | Versionierung | HOCH |
| FIL-004 | Volltextsuche | MITTEL |
| FIL-005 | Automatische Kategorisierung | NIEDRIG |

---

### 2.10 Aufgabenverwaltung

| ID | Anforderung | Priorität |
|----|-------------|-----------|
| TSK-001 | Intelligente Aufgabenerstellung | HOCH |
| TSK-002 | Automatisierungsfokus | HOCH |
| TSK-003 | Abhängigkeiten | MITTEL |
| TSK-004 | Automatische Priorisierung | MITTEL |
| TSK-005 | Integration mit Projekten | HOCH |

---

### 2.11 Ticketsystem

| ID | Anforderung | Priorität |
|----|-------------|-----------|
| TKT-001 | Fehlermeldungen erfassen | HOCH |
| TKT-002 | Screenshot-Funktion | MITTEL |
| TKT-003 | Status-Tracking | HOCH |
| TKT-004 | Eskalation | NIEDRIG |

---

### 2.12 Externe Integrationen

#### 2.12.1 Buchhaltungs-Integration
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| INT-001 | DATEV-Export (Buchungssätze) | HOCH |
| INT-002 | Lexoffice API-Integration | HOCH |
| INT-003 | sevDesk API-Integration | MITTEL |
| INT-004 | BMD-Export (Österreich) | MITTEL |
| INT-005 | Automatische Rechnungsübernahme | MITTEL |
| INT-006 | Kontenrahmen-Mapping (SKR03/04, Österreich) | HOCH |

#### 2.12.2 Kalender-Integration
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| INT-010 | CalDAV-Synchronisation | MITTEL |
| INT-011 | Google Calendar API | MITTEL |
| INT-012 | Microsoft Outlook/Graph API | MITTEL |
| INT-013 | Terminerinnerungen | HOCH |
| INT-014 | Ressourcenplanung (Mitarbeiter) | NIEDRIG |

#### 2.12.3 IoT & Messgeräte
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| INT-020 | Smart-Meter-Anbindung (MQTT) | MITTEL |
| INT-021 | Wetterstationen-Integration | NIEDRIG |
| INT-022 | Raumklima-Sensoren | NIEDRIG |
| INT-023 | Messgeräte-Import (CSV, API) | MITTEL |
| INT-024 | Echtzeit-Monitoring-Dashboard | NIEDRIG |

#### 2.12.4 BIM & CAD
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| INT-030 | IFC-Import (BIM) | NIEDRIG |
| INT-031 | DXF/DWG-Import (Grundrisse) | NIEDRIG |
| INT-032 | gbXML-Import/Export | NIEDRIG |
| INT-033 | Automatische Geometrie-Extraktion | NIEDRIG |

---

### 2.13 Erweiterte Berechnungen

#### 2.13.1 Bauphysik - Erweitert
| ID | Anforderung | Norm | Priorität |
|----|-------------|------|-----------|
| CALC-E-001 | Tauwasserschutz (Glaser-Verfahren) | DIN 4108-3 | HOCH |
| CALC-E-002 | Sommerlicher Wärmeschutz | DIN 4108-2 | HOCH |
| CALC-E-003 | Hygrothermische Simulation | EN 15026 | NIEDRIG |
| CALC-E-004 | Luftdichtheitsprüfung | DIN EN 13829 | MITTEL |
| CALC-E-005 | Schallschutznachweis | DIN 4109 | NIEDRIG |

#### 2.13.2 Tageslicht & Beleuchtung
| ID | Anforderung | Norm | Priorität |
|----|-------------|------|-----------|
| CALC-E-010 | Tageslichtquotient | EN 17037 | MITTEL |
| CALC-E-011 | Beleuchtungsberechnung | DIN V 18599-4 | HOCH |
| CALC-E-012 | Kunstlichtbedarf | DIN EN 12464-1 | MITTEL |

#### 2.13.3 Lebenszyklusanalyse (LCA)
| ID | Anforderung | Norm | Priorität |
|----|-------------|------|-----------|
| CALC-E-020 | Graue Energie Baustoffe | EN 15978 | MITTEL |
| CALC-E-021 | CO2-Fußabdruck Bau | EN 15804 | MITTEL |
| CALC-E-022 | Ökobilanzierung Gebäude | DGNB, BNB | NIEDRIG |
| CALC-E-023 | Recyclingpotenzial | EN 15804+A2 | NIEDRIG |

---

### 2.14 No-Code Automatisierung

| ID | Anforderung | Priorität |
|----|-------------|-----------|
| NCA-001 | Visueller Workflow-Editor | MITTEL |
| NCA-002 | Trigger-Typen (Zeit, Ereignis, Daten) | MITTEL |
| NCA-003 | Vordefinierte Aktionen (E-Mail, Status, etc.) | MITTEL |
| NCA-004 | Bedingte Verzweigungen | MITTEL |
| NCA-005 | Vorlagen-Bibliothek für Workflows | NIEDRIG |
| NCA-006 | Webhook-Integration | NIEDRIG |

---

### 2.15 Benchmarking & Gamification

| ID | Anforderung | Priorität |
|----|-------------|-----------|
| BEN-001 | Anonymisierter Gebäudevergleich | MITTEL |
| BEN-002 | Branchenvergleich (Energieberater) | NIEDRIG |
| BEN-003 | Achievements/Badges System | NIEDRIG |
| BEN-004 | Fortschrittsanzeigen | MITTEL |
| BEN-005 | Effizienz-Rankings (opt-in) | NIEDRIG |

---

### 2.16 Ökosystem & Marktplatz

| ID | Anforderung | Priorität |
|----|-------------|-----------|
| ECO-001 | Plugin-Architektur | MITTEL |
| ECO-002 | Plugin-API und SDK | MITTEL |
| ECO-003 | Vorlagen-Marktplatz | NIEDRIG |
| ECO-004 | Partner-Integrationen | NIEDRIG |
| ECO-005 | White-Label-Optionen | NIEDRIG |
| ECO-006 | Reseller-Portal | NIEDRIG |

---

### 2.17 Endkunden-Portal & Public Links

⚠️ **Hinweis:** Endkunden haben weiterhin KEINEN vollständigen Login! Das Portal ist ein eingeschränkter Zugang.

#### 2.17.1 Basis-Portal
| ID | Anforderung | Priorität |
|----|-------------|-----------|
| EKP-001 | Einmaliger Link-Zugang (Token-basiert) | MITTEL |
| EKP-002 | Read-Only Projektstatus | MITTEL |
| EKP-003 | Dokumenten-Download | MITTEL |
| EKP-004 | Dokumenten-Upload (Rechnungen, Fotos) | MITTEL |
| EKP-005 | Nachrichten an Berater | NIEDRIG |
| EKP-006 | Digitale Auftragsannahme | MITTEL |

#### 2.17.2 Datenerhebungs-Links (Public Links)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  DATENERHEBUNGS-LINKS (Für Endkunden ohne Login)                                │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  Komponente: "Datenerhebungs-Link generieren"                                   │
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  Konfigurationsoptionen:                                                 │    │
│  │                                                                          │    │
│  │  ○ Modus A: "Nur Upload"                                                 │    │
│  │     - Endkunde kann Dateien hochladen (Fotos, Rechnungen, Pläne)        │    │
│  │     - Keine Bearbeitung bestehender Dokumente                           │    │
│  │                                                                          │    │
│  │  ○ Modus B: "Bearbeitung erlaubt" (via Collabora)                        │    │
│  │     - Endkunde kann .xlsx-Checklisten direkt im Browser ausfüllen       │    │
│  │     - Live-Bearbeitung via Collabora/WOPI                               │    │
│  │     - Änderungen werden automatisch gespeichert                         │    │
│  │                                                                          │    │
│  │  Optionen:                                                               │    │
│  │  [x] Gültigkeit (1 Tag / 7 Tage / 30 Tage / Unbegrenzt)                │    │
│  │  [x] Passwortschutz optional                                            │    │
│  │  [x] E-Mail-Benachrichtigung bei Änderungen                             │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

| ID | Anforderung | Priorität |
|----|-------------|-----------|
| PUB-001 | Komponente "Datenerhebungs-Link generieren" | HOCH |
| PUB-002 | Modus "Nur Upload" (Dateien hochladen) | HOCH |
| PUB-003 | Modus "Bearbeitung erlaubt" (via Collabora) | HOCH |
| PUB-004 | Konfigurierbare Gültigkeitsdauer | MITTEL |
| PUB-005 | Optionaler Passwortschutz für Links | MITTEL |
| PUB-006 | E-Mail-Benachrichtigung bei Aktivität | MITTEL |
| PUB-007 | Link-Verwaltung (Übersicht, Deaktivieren) | MITTEL |
| PUB-008 | Audit-Log für Public-Link-Zugriffe | HOCH |

---

## 3. Nicht-funktionale Anforderungen

### 3.1 Sicherheit (OWASP Top 10 2025)

| ID | Anforderung | Standard |
|----|-------------|----------|
| SEC-001 | Broken Access Control Prevention | A01:2025 |
| SEC-002 | Security Misconfiguration Prevention | A02:2025 |
| SEC-003 | Software Supply Chain Security | A03:2025 |
| SEC-004 | Cryptographic Failures Prevention | A04:2025 |
| SEC-005 | Injection Prevention | A05:2025 |
| SEC-006 | Insecure Design Prevention | A06:2025 |
| SEC-007 | Authentication Failures Prevention | A07:2025 |
| SEC-008 | Software Integrity Verification | A08:2025 |
| SEC-009 | Logging and Alerting | A09:2025 |
| SEC-010 | Exception Handling | A10:2025 |

### 3.2 DDoS-Schutz

| ID | Anforderung | Priorität |
|----|-------------|-----------|
| DDOS-001 | Rate Limiting auf allen Ebenen | KRITISCH |
| DDOS-002 | Fail2ban Integration | HOCH |
| DDOS-003 | CrowdSec Community-Schutz | MITTEL |
| DDOS-004 | WAF (mod_security/SafeLine) | HOCH |

### 3.3 Datenisolation

| ID | Anforderung | Priorität |
|----|-------------|-----------|
| ISO-001 | PostgreSQL Row Level Security | KRITISCH |
| ISO-002 | Schema-per-Tenant Option | HOCH |
| ISO-003 | Separate Encryption Keys | HOCH |
| ISO-004 | Audit-Trail pro Mandant | KRITISCH |

### 3.4 Performance

| ID | Anforderung | Zielwert |
|----|-------------|----------|
| PRF-001 | Seitenladezeit | < 2s |
| PRF-002 | API-Antwortzeit (P95) | < 200ms |
| PRF-003 | Berechnungsdauer Energiebilanz | < 5s |
| PRF-004 | PDF-Generierung | < 10s |
| PRF-005 | Gleichzeitige Benutzer | 1000+ |

### 3.5 Verfügbarkeit

| ID | Anforderung | Zielwert |
|----|-------------|----------|
| AVL-001 | Uptime | 99.9% |
| AVL-002 | Offline-Fähigkeit | Vollständig für Kernfunktionen |
| AVL-003 | Backup-Intervall | Täglich + inkrementell |
| AVL-004 | Recovery Time Objective | < 4h |

### 3.6 Barrierefreiheit

| ID | Anforderung | Standard |
|----|-------------|----------|
| A11Y-001 | WCAG 2.1 AA Konformität | Pflicht |
| A11Y-002 | Screenreader-Kompatibilität | Pflicht |
| A11Y-003 | Keyboard-Navigation | Pflicht |
| A11Y-004 | Farbkontrast 4.5:1 | Pflicht |

---

## 4. Monetarisierung

### 4.1 Abrechnungsmodelle

| Modell | Beschreibung | Anwendung |
|--------|--------------|-----------|
| Modulnutzung | Pay-per-Use pro Berechnung/Bericht | Kleine Büros |
| Pauschale | Festpreis pro Projekt | Mittlere Büros |
| Abo | Monatliche Flatrate | Große Büros |
| Enterprise | Individuell | Konzerne |

### 4.2 Freischaltbare Module

```
Basis (kostenlos):
├── Projektverwaltung
├── Kundenverwaltung
└── Basis-Berechnungen

Standard:
├── Energieausweise
├── Heizlastberechnung
├── Basis-Berichte
└── Zeiterfassung

Professional:
├── Energieaudits
├── Sanierungsfahrplan
├── KI-Textgenerierung
├── Kollaboration
└── Erweiterte Automatisierung

Enterprise:
├── Multi-Mandant
├── API-Zugang
├── Custom Branding
├── Dedizierter Support
└── On-Premise Option
```

---

## 5. Technische Anforderungen

### 5.1 Open Source Compliance

| Komponente | Lizenz | Status |
|------------|--------|--------|
| FastAPI | MIT | ✓ Frei |
| PostgreSQL | PostgreSQL License | ✓ Frei |
| Redis | BSD | ✓ Frei |
| Alpine.js | MIT | ✓ Frei |
| Tailwind CSS | MIT | ✓ Frei |
| Yjs | MIT | ✓ Frei |
| Ollama | MIT | ✓ Frei |
| Caddy | Apache 2.0 | ✓ Frei |

### 5.2 Browser-Unterstützung

| Browser | Version |
|---------|---------|
| Chrome | 100+ |
| Firefox | 100+ |
| Safari | 15+ |
| Edge | 100+ |
| Mobile Safari | 15+ |
| Chrome Android | 100+ |

---

## 6. Dokumentationsanforderungen

| ID | Anforderung |
|----|-------------|
| DOK-001 | Vollständige API-Dokumentation (OpenAPI) |
| DOK-002 | Benutzerhandbuch |
| DOK-003 | Administrator-Handbuch |
| DOK-004 | Entwickler-Dokumentation |
| DOK-005 | Berechnungs-Dokumentation mit Quellenangaben |
| DOK-006 | Open-Source-Attribution |

---

*Letzte Aktualisierung: 2025-11-26*
*Version: 3.1.0*
