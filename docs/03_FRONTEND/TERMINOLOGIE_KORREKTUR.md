# Terminologie-Korrektur und Klarstellung

## Korrigierte Hierarchie

Die Plattform EnergieberaterPRO hat folgende Benutzer-Hierarchie:

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│   EBENE 1: SYSADMIN = WEBSITEBETREIBER                                       │
│   ════════════════════════════════════                                       │
│   • Betreibt die SaaS-Plattform "EnergieberaterPRO"                          │
│   • Verkauft Abonnements an Organisationen                                   │
│   • Organisationen sind die KUNDEN des Sysadmins                             │
│   • Kann ALLES auf der Website einstellen                                    │
│                                                                              │
│   ┌──────────────────────────────────────────────────────────────────────┐   │
│   │                                                                      │   │
│   │   EBENE 2: ORGANISATION = KUNDE DES SYSADMINS                        │   │
│   │   ═══════════════════════════════════════════                        │   │
│   │   • Energieberatungsbüros, die die Plattform abonnieren              │   │
│   │   • Zahlen monatlich/jährlich für die Nutzung                        │   │
│   │   • Haben eigene Mitarbeiter                                         │   │
│   │   • Haben eigene ENDKUNDEN (Hausbesitzer, Firmen)                    │   │
│   │                                                                      │   │
│   │   Benutzerrollen innerhalb einer Organisation:                       │   │
│   │   ├── ORG-ADMIN: Verwaltet die Organisation                          │   │
│   │   └── MITARBEITER: Arbeiten an Projekten                             │   │
│   │                                                                      │   │
│   │   ┌──────────────────────────────────────────────────────────────┐   │   │
│   │   │                                                              │   │   │
│   │   │   EBENE 3: ENDKUNDE = KUNDE DER ORGANISATION                 │   │   │
│   │   │   ════════════════════════════════════════════               │   │   │
│   │   │   • Hausbesitzer, Unternehmen, WEGs                          │   │   │
│   │   │   • Beauftragen Energieberatungen                            │   │   │
│   │   │   • Haben KEINEN eigenen Login auf der Plattform!            │   │   │
│   │   │   • Werden als Datensatz in Projekten erfasst                │   │   │
│   │   │                                                              │   │   │
│   │   └──────────────────────────────────────────────────────────────┘   │   │
│   │                                                                      │   │
│   └──────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

## Begriffsdefinitionen

### Plattform-Ebene (Sysadmin)

| Begriff | Bedeutung | Beispiel |
|---------|-----------|----------|
| **Sysadmin** | Der Betreiber der Plattform | EnergieberaterPRO GmbH |
| **Organisation** | Ein Kunde des Sysadmins | "Müller Energieberatung" |
| **Abonnement** | Vertrag zwischen Sysadmin und Organisation | Professional-Plan, 79€/Monat |
| **Modul** | Zusatzfunktion, die der Sysadmin verkauft | KI-Assistent, 15€/Monat |

### Organisations-Ebene (Org-Admin + Mitarbeiter)

| Begriff | Bedeutung | Beispiel |
|---------|-----------|----------|
| **Org-Admin** | Administrator einer Organisation | Chef des Büros |
| **Mitarbeiter** | Angestellter einer Organisation | Energieberater, Sekretärin |
| **Endkunde** | Kunde der Organisation | Familie Schmidt, Firma ABC GmbH |
| **Auftrag** | Vertrag zwischen Organisation und Endkunde | Energieberatung für EFH |
| **Projekt** | Arbeit an einem Gebäude für einen Endkunden | iSFP für Musterstraße 1 |

## Sysadmin-Sicht: Organisationen sind Kunden

Der Sysadmin verwaltet **Organisationen** als seine Kunden:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    SYSADMIN DASHBOARD                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  "Meine Kunden" (= Organisationen)                                          │
│  ─────────────────────────────────                                          │
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │ Organisation          │ Plan        │ Benutzer │ MRR      │ Status  │   │
│  ├──────────────────────────────────────────────────────────────────────┤   │
│  │ Müller Energieberatung│ Professional│ 3        │ 79€      │ Aktiv   │   │
│  │ Schmidt & Partner     │ Enterprise  │ 12       │ 149€     │ Aktiv   │   │
│  │ Grün Consulting       │ Starter     │ 1        │ 39€      │ Trial   │   │
│  │ EcoEnergy GmbH        │ Professional│ 5        │ 79€      │ Aktiv   │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│  Gesamtumsatz (MRR): 346€                                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**WICHTIG**: Der Sysadmin sieht die Endkunden der Organisationen **nur im Support-Modus** oder für Statistiken. Die Endkunden sind nicht die direkten Kunden des Sysadmins!

## Organisations-Sicht: Endkunden sind ihre Kunden

Eine Organisation verwaltet **Endkunden** als ihre Kunden:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ORG-ADMIN DASHBOARD                                       │
│                    (Müller Energieberatung)                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  "Meine Kunden" (= Endkunden)                                               │
│  ────────────────────────────                                               │
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │ Endkunde              │ Typ     │ Projekte │ Umsatz    │ Status     │   │
│  ├──────────────────────────────────────────────────────────────────────┤   │
│  │ Familie Schmidt       │ Privat  │ 2        │ 1.500€    │ Aktiv      │   │
│  │ ABC GmbH             │ Gewerbe │ 1        │ 3.200€    │ Aktiv      │   │
│  │ WEG Musterstraße     │ WEG     │ 1        │ 4.800€    │ Ausstehend │   │
│  │ Herr Müller          │ Privat  │ 1        │ 800€      │ Abgeschl.  │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│  Gesamtumsatz: 10.300€                                                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Navigation und Menü-Struktur

### Sysadmin-Menü

```
Sysadmin-Panel
├── Dashboard
│   ├── Umsatz-Übersicht (von Organisationen)
│   ├── Neue Registrierungen
│   └── System-Status
│
├── Kunden (= Organisationen)
│   ├── Alle Organisationen
│   ├── Neue Registrierungen
│   ├── Ausstehende Zahlungen
│   └── Gesperrte Organisationen
│
├── Abrechnung
│   ├── Rechnungen
│   ├── Zahlungen
│   ├── Pläne & Preise
│   └── Gutscheine
│
├── Module
│   ├── Verfügbare Module
│   ├── Freischaltungen
│   └── Nutzungsstatistik
│
├── System
│   ├── Globale Einstellungen
│   ├── E-Mail-Konfiguration
│   ├── Sicherheit
│   └── Wartungsmodus
│
├── Support
│   ├── Tickets
│   ├── Impersonation
│   └── Audit-Logs
│
└── Website
    ├── Landing Page
    ├── Pricing
    ├── Feature-Seiten
    └── Rechtliches
```

### Org-Admin-Menü

```
Organisation-Panel
├── Dashboard
│   ├── Projekt-Übersicht
│   ├── Team-Aktivität
│   └── Umsatz (von Endkunden)
│
├── Kunden (= Endkunden)
│   ├── Alle Kunden
│   ├── Neuer Kunde
│   └── Import
│
├── Projekte
│   ├── Alle Projekte
│   ├── Aktive
│   └── Vorlagen
│
├── Aufträge
│   ├── Pipeline
│   ├── Angebote
│   └── Rechnungen
│
├── Team
│   ├── Mitarbeiter
│   ├── Einladen
│   └── Rollen
│
├── Einstellungen
│   ├── Organisation
│   ├── Branding
│   ├── E-Mail
│   └── Templates
│
└── Mein Plan
    ├── Aktueller Plan
    ├── Module
    └── Rechnungen
```

### Mitarbeiter-Menü

```
Arbeitsbereich
├── Start (Dashboard)
├── Meine Projekte
├── Meine Kunden (zugewiesene Endkunden)
├── Dokumente
├── Berechnungen
├── Kalender
├── Aufgaben
├── KI-Assistent
└── Wissensdatenbank
```

## Berechtigungsmatrix

### Was sieht wer?

| Funktion | Sysadmin | Org-Admin | Mitarbeiter |
|----------|----------|-----------|-------------|
| Alle Organisationen | ✓ | ✗ | ✗ |
| Eigene Organisation | ✓ | ✓ | ✓ (nur lesen) |
| Org-Abrechnung (Plan) | ✓ | ✓ | ✗ |
| Alle Endkunden einer Org | ✓ (Support) | ✓ | ✗ |
| Zugewiesene Endkunden | ✓ | ✓ | ✓ |
| Alle Projekte einer Org | ✓ (Support) | ✓ | ✗ |
| Zugewiesene Projekte | ✓ | ✓ | ✓ |
| Team verwalten | ✗ | ✓ | ✗ |
| System-Konfiguration | ✓ | ✗ | ✗ |
| Plattform-Preise ändern | ✓ | ✗ | ✗ |

### Wer verwaltet was?

| Entität | Erstellt von | Verwaltet von | Gelöscht von |
|---------|--------------|---------------|--------------|
| Organisation | Sysadmin / Registrierung | Sysadmin, Org-Admin | Sysadmin |
| Benutzer | Org-Admin | Org-Admin | Org-Admin |
| Endkunde | Org-Admin, Mitarbeiter | Org-Admin, Mitarbeiter | Org-Admin |
| Projekt | Org-Admin, Mitarbeiter | Org-Admin, zugewiesener MA | Org-Admin |
| Dokument | Mitarbeiter | Ersteller | Org-Admin, Ersteller |
| Plan/Modul | Sysadmin | Sysadmin | Sysadmin |

## Konsistenz-Regeln für die UI

### 1. Sprachliche Konsistenz

**Im Sysadmin-Panel:**
- "Kunden" = Organisationen
- "Abonnenten" = Organisationen mit aktivem Plan
- "Umsatz" = Einnahmen von Organisationen

**Im Org-Admin/Mitarbeiter-Panel:**
- "Kunden" = Endkunden
- "Aufträge" = Aufträge von Endkunden
- "Umsatz" = Einnahmen von Endkunden

### 2. Icon-Konsistenz

| Konzept | Icon | Verwendung |
|---------|------|------------|
| Organisation | 🏢 Building | Sysadmin-Panel |
| Endkunde | 👤 User | Org-Panel |
| Projekt | 📁 Folder | Überall |
| Berechnung | 🧮 Calculator | Überall |
| Dokument | 📄 File | Überall |

### 3. Farb-Konsistenz

| Bereich | Primärfarbe | Verwendung |
|---------|-------------|------------|
| Sysadmin | Rot/Dunkel | #991B1B |
| Org-Admin | Blau | #2563EB |
| Mitarbeiter | Grün | #059669 |

### 4. Breadcrumb-Muster

**Sysadmin:**
```
Sysadmin > Kunden > Müller Energieberatung > Details
```

**Org-Admin:**
```
Organisation > Kunden > Familie Schmidt > Projekte
```

**Mitarbeiter:**
```
Projekte > Energieberatung EFH Schmidt > Berechnungen
```

## Checkliste für UI-Entwicklung

- [ ] "Kunde" im Sysadmin-Panel = Organisation
- [ ] "Kunde" im Org-Panel = Endkunde
- [ ] Endkunden haben keinen Login
- [ ] Sysadmin-Panel hat eigene Farbgebung
- [ ] Breadcrumbs zeigen korrekte Hierarchie
- [ ] Menü-Struktur entspricht der Hierarchie
- [ ] Berechtigungen werden korrekt geprüft
- [ ] Keine Vermischung der Terminologie
