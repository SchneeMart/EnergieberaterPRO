# Design System - EnergieberaterPRO

## 1. Design-Philosophie

### 1.1 Grundprinzipien

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DESIGN-PRINZIPIEN                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  1. KLARHEIT                                                                │
│     └── Komplexe Daten verständlich präsentieren                           │
│     └── Eindeutige visuelle Hierarchie                                     │
│     └── Konsistente Muster für wiederkehrende Aufgaben                    │
│                                                                             │
│  2. EFFIZIENZ                                                               │
│     └── Häufige Aktionen mit minimalen Klicks                              │
│     └── Kontextbezogene Hilfe                                              │
│     └── Keyboard-Shortcuts für Power-User                                   │
│                                                                             │
│  3. PROFESSIONALITÄT                                                        │
│     └── Seriöses, technisches Erscheinungsbild                             │
│     └── Präzise Darstellung von Kennwerten                                 │
│     └── Druckoptimierte Ausgaben                                            │
│                                                                             │
│  4. ZUGÄNGLICHKEIT                                                          │
│     └── WCAG 2.1 AA Konformität                                            │
│     └── Responsive für alle Geräte                                          │
│     └── Keine Barrieren für Screenreader                                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Zielgruppe

```
Primär:
├── Energieberater (Experten)
│   └── Tägliche intensive Nutzung
│   └── Komplexe Berechnungen
│   └── Berichterstellung
│
├── Projektmanager
│   └── Übersicht über Projekte
│   └── Zeit- und Kostenkontrolle
│
└── Administratoren
    └── Benutzerverwaltung
    └── Systemkonfiguration

Sekundär:
└── Kunden (Lesezugriff auf Berichte)
```

---

## 2. Farbsystem

### 2.1 Primärfarben

```css
/* Blau - Hauptfarbe (Vertrauen, Technik) */
--primary-50:  #eff6ff;
--primary-100: #dbeafe;
--primary-200: #bfdbfe;
--primary-300: #93c5fd;
--primary-400: #60a5fa;
--primary-500: #3b82f6;  /* Standard */
--primary-600: #2563eb;  /* Hover */
--primary-700: #1d4ed8;  /* Active */
--primary-800: #1e40af;
--primary-900: #1e3a8a;
```

### 2.2 Sekundärfarben

```css
/* Grün - Energie, Nachhaltigkeit */
--secondary-50:  #ecfdf5;
--secondary-100: #d1fae5;
--secondary-200: #a7f3d0;
--secondary-300: #6ee7b7;
--secondary-400: #34d399;
--secondary-500: #10b981;  /* Standard */
--secondary-600: #059669;
--secondary-700: #047857;
--secondary-800: #065f46;
--secondary-900: #064e3b;
```

### 2.3 Semantische Farben

```css
/* Status-Farben */
--success:  #10b981;  /* Grün - Erfolg */
--warning:  #f59e0b;  /* Orange - Warnung */
--error:    #ef4444;  /* Rot - Fehler */
--info:     #3b82f6;  /* Blau - Information */

/* Hintergründe für Status */
--success-bg: #ecfdf5;
--warning-bg: #fffbeb;
--error-bg:   #fef2f2;
--info-bg:    #eff6ff;
```

### 2.4 Energieeffizienz-Skala

```css
/* Offizielle Farbskala für Energieausweise */
--efficiency-a-plus: #00a651;  /* A+ */
--efficiency-a:      #4cb848;  /* A  */
--efficiency-b:      #bdd62e;  /* B  */
--efficiency-c:      #fff100;  /* C  */
--efficiency-d:      #ffb819;  /* D  */
--efficiency-e:      #f47920;  /* E  */
--efficiency-f:      #ed1c24;  /* F  */
--efficiency-g:      #be1622;  /* G  */
--efficiency-h:      #6d1f19;  /* H  */
```

### 2.5 Neutralfarben

```css
/* Graustufen */
--gray-50:  #f9fafb;  /* Hintergrund */
--gray-100: #f3f4f6;  /* Karten */
--gray-200: #e5e7eb;  /* Borders */
--gray-300: #d1d5db;  /* Disabled */
--gray-400: #9ca3af;  /* Placeholder */
--gray-500: #6b7280;  /* Sekundärtext */
--gray-600: #4b5563;  /* Text */
--gray-700: #374151;  /* Überschriften */
--gray-800: #1f2937;  /* Primärtext */
--gray-900: #111827;  /* Stark */

/* Spezielle Hintergründe */
--bg-app:     #f3f4f6;  /* App-Hintergrund */
--bg-sidebar: #1f2937;  /* Sidebar dunkel */
--bg-card:    #ffffff;  /* Karten */
```

---

## 3. Typografie

### 3.1 Schriftfamilien

```css
/* Primäre Schrift: Inter (Google Fonts) */
--font-sans: 'Inter', -apple-system, BlinkMacSystemFont,
             'Segoe UI', Roboto, 'Helvetica Neue', sans-serif;

/* Monospace für Zahlen und Code */
--font-mono: 'JetBrains Mono', 'Fira Code', 'SF Mono',
             Consolas, monospace;
```

### 3.2 Schriftgrößen

```css
/* Überschriften */
--text-4xl: 2.25rem;    /* 36px - Seitentitel */
--text-3xl: 1.875rem;   /* 30px - Sektionen */
--text-2xl: 1.5rem;     /* 24px - Untersektionen */
--text-xl:  1.25rem;    /* 20px - Karten-Titel */
--text-lg:  1.125rem;   /* 18px - Hervorgehoben */

/* Fließtext */
--text-base: 1rem;      /* 16px - Standard */
--text-sm:   0.875rem;  /* 14px - Sekundär */
--text-xs:   0.75rem;   /* 12px - Labels, Captions */

/* Schriftgewichte */
--font-normal:   400;
--font-medium:   500;
--font-semibold: 600;
--font-bold:     700;
```

### 3.3 Typografie-Klassen

```css
/* Überschriften */
.heading-1 {
    font-size: var(--text-4xl);
    font-weight: var(--font-bold);
    line-height: 1.2;
    color: var(--gray-900);
}

.heading-2 {
    font-size: var(--text-2xl);
    font-weight: var(--font-semibold);
    line-height: 1.3;
    color: var(--gray-800);
}

.heading-3 {
    font-size: var(--text-xl);
    font-weight: var(--font-semibold);
    line-height: 1.4;
    color: var(--gray-800);
}

/* Textstile */
.text-body {
    font-size: var(--text-base);
    line-height: 1.6;
    color: var(--gray-700);
}

.text-secondary {
    font-size: var(--text-sm);
    color: var(--gray-500);
}

.text-caption {
    font-size: var(--text-xs);
    color: var(--gray-400);
    text-transform: uppercase;
    letter-spacing: 0.05em;
}

/* Zahlen (tabellarisch) */
.text-numeric {
    font-family: var(--font-mono);
    font-feature-settings: 'tnum' on, 'lnum' on;
}
```

---

## 4. Spacing & Layout

### 4.1 Spacing-Skala

```css
/* 4px Basis-Skala */
--space-0:  0;
--space-1:  0.25rem;   /* 4px */
--space-2:  0.5rem;    /* 8px */
--space-3:  0.75rem;   /* 12px */
--space-4:  1rem;      /* 16px */
--space-5:  1.25rem;   /* 20px */
--space-6:  1.5rem;    /* 24px */
--space-8:  2rem;      /* 32px */
--space-10: 2.5rem;    /* 40px */
--space-12: 3rem;      /* 48px */
--space-16: 4rem;      /* 64px */
--space-20: 5rem;      /* 80px */
--space-24: 6rem;      /* 96px */
```

### 4.2 Layout-Konstanten

```css
/* Breiten */
--sidebar-width:      280px;
--sidebar-collapsed:  64px;
--content-max-width:  1280px;
--form-max-width:     640px;
--modal-sm:           400px;
--modal-md:           560px;
--modal-lg:           720px;
--modal-xl:           960px;

/* Höhen */
--header-height:      64px;
--footer-height:      48px;
--input-height:       40px;
--input-height-sm:    32px;
--input-height-lg:    48px;
```

### 4.3 Grid-System

```css
/* CSS Grid für Dashboards */
.grid-dashboard {
    display: grid;
    grid-template-columns: repeat(12, 1fr);
    gap: var(--space-6);
}

/* Responsive Breakpoints */
@media (min-width: 640px)  { /* sm */ }
@media (min-width: 768px)  { /* md */ }
@media (min-width: 1024px) { /* lg */ }
@media (min-width: 1280px) { /* xl */ }
@media (min-width: 1536px) { /* 2xl */ }
```

---

## 5. Komponenten

### 5.1 Buttons

```html
<!-- Primär -->
<button class="btn btn-primary">Speichern</button>

<!-- Sekundär -->
<button class="btn btn-secondary">Abbrechen</button>

<!-- Outline -->
<button class="btn btn-outline">Details</button>

<!-- Ghost -->
<button class="btn btn-ghost">Mehr</button>

<!-- Danger -->
<button class="btn btn-danger">Löschen</button>

<!-- Größen -->
<button class="btn btn-primary btn-sm">Klein</button>
<button class="btn btn-primary btn-lg">Groß</button>

<!-- Mit Icon -->
<button class="btn btn-primary">
    <svg>...</svg>
    Hinzufügen
</button>

<!-- Loading -->
<button class="btn btn-primary loading">
    <span class="spinner"></span>
    Wird gespeichert...
</button>
```

```css
.btn {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: var(--space-2);
    padding: 0 var(--space-4);
    height: var(--input-height);
    font-size: var(--text-sm);
    font-weight: var(--font-medium);
    border-radius: var(--radius-lg);
    transition: all 0.15s ease;
    cursor: pointer;
}

.btn-primary {
    background: var(--primary-600);
    color: white;
}
.btn-primary:hover {
    background: var(--primary-700);
}

.btn-secondary {
    background: var(--gray-100);
    color: var(--gray-700);
}

.btn-outline {
    border: 1px solid var(--gray-300);
    background: transparent;
}

.btn-ghost {
    background: transparent;
    color: var(--gray-600);
}
.btn-ghost:hover {
    background: var(--gray-100);
}
```

### 5.2 Formulare

```html
<!-- Text Input -->
<div class="form-group">
    <label for="name" class="form-label">
        Projektname
        <span class="required">*</span>
    </label>
    <input
        type="text"
        id="name"
        class="form-input"
        placeholder="Bezeichnung eingeben"
    />
    <p class="form-hint">Maximal 100 Zeichen</p>
</div>

<!-- Mit Fehler -->
<div class="form-group has-error">
    <label class="form-label">E-Mail</label>
    <input type="email" class="form-input" />
    <p class="form-error">Ungültige E-Mail-Adresse</p>
</div>

<!-- Nummer mit Einheit -->
<div class="form-group">
    <label class="form-label">Fläche</label>
    <div class="input-with-unit">
        <input type="number" class="form-input" />
        <span class="input-unit">m²</span>
    </div>
</div>

<!-- Select -->
<div class="form-group">
    <label class="form-label">Gebäudetyp</label>
    <select class="form-select">
        <option value="">Bitte wählen...</option>
        <option value="efh">Einfamilienhaus</option>
        <option value="mfh">Mehrfamilienhaus</option>
    </select>
</div>

<!-- Checkbox -->
<label class="form-checkbox">
    <input type="checkbox" />
    <span class="checkmark"></span>
    <span class="label-text">Ich akzeptiere die AGB</span>
</label>
```

```css
.form-group {
    margin-bottom: var(--space-5);
}

.form-label {
    display: block;
    margin-bottom: var(--space-2);
    font-size: var(--text-sm);
    font-weight: var(--font-medium);
    color: var(--gray-700);
}

.form-input,
.form-select {
    width: 100%;
    height: var(--input-height);
    padding: 0 var(--space-3);
    font-size: var(--text-base);
    border: 1px solid var(--gray-300);
    border-radius: var(--radius-lg);
    background: white;
    transition: border-color 0.15s, box-shadow 0.15s;
}

.form-input:focus,
.form-select:focus {
    outline: none;
    border-color: var(--primary-500);
    box-shadow: 0 0 0 3px var(--primary-100);
}

.has-error .form-input {
    border-color: var(--error);
}

.form-error {
    margin-top: var(--space-1);
    font-size: var(--text-sm);
    color: var(--error);
}

.input-with-unit {
    position: relative;
}
.input-unit {
    position: absolute;
    right: var(--space-3);
    top: 50%;
    transform: translateY(-50%);
    color: var(--gray-400);
    font-size: var(--text-sm);
}
```

### 5.3 Karten

```html
<!-- Standard Card -->
<div class="card">
    <div class="card-header">
        <h3 class="card-title">Projektübersicht</h3>
        <button class="btn btn-ghost btn-sm">
            <svg><!-- more icon --></svg>
        </button>
    </div>
    <div class="card-body">
        <!-- Inhalt -->
    </div>
    <div class="card-footer">
        <button class="btn btn-secondary">Abbrechen</button>
        <button class="btn btn-primary">Speichern</button>
    </div>
</div>

<!-- Stat Card -->
<div class="card stat-card">
    <div class="stat-icon">
        <svg><!-- icon --></svg>
    </div>
    <div class="stat-content">
        <span class="stat-label">Primärenergie</span>
        <span class="stat-value">127.5</span>
        <span class="stat-unit">kWh/(m²a)</span>
    </div>
    <div class="stat-trend up">
        <svg><!-- arrow up --></svg>
        -15% vs. Vorjahr
    </div>
</div>
```

```css
.card {
    background: var(--bg-card);
    border-radius: var(--radius-xl);
    box-shadow: var(--shadow-sm);
    overflow: hidden;
}

.card-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: var(--space-4) var(--space-5);
    border-bottom: 1px solid var(--gray-100);
}

.card-title {
    font-size: var(--text-lg);
    font-weight: var(--font-semibold);
    color: var(--gray-800);
}

.card-body {
    padding: var(--space-5);
}

.card-footer {
    display: flex;
    justify-content: flex-end;
    gap: var(--space-3);
    padding: var(--space-4) var(--space-5);
    border-top: 1px solid var(--gray-100);
    background: var(--gray-50);
}
```

### 5.4 Tabellen

```html
<div class="table-container">
    <table class="table">
        <thead>
            <tr>
                <th>
                    <button class="sort-btn">
                        Projekt
                        <svg><!-- sort icon --></svg>
                    </button>
                </th>
                <th class="text-right">Heizlast</th>
                <th class="text-right">Primärenergie</th>
                <th>Status</th>
                <th></th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>
                    <div class="cell-with-sub">
                        <span class="cell-main">Musterstraße 1</span>
                        <span class="cell-sub">Einfamilienhaus</span>
                    </div>
                </td>
                <td class="text-right text-numeric">8.500 W</td>
                <td class="text-right text-numeric">127.5 kWh/(m²a)</td>
                <td>
                    <span class="badge badge-success">Aktiv</span>
                </td>
                <td>
                    <button class="btn btn-ghost btn-sm">
                        <svg><!-- dots icon --></svg>
                    </button>
                </td>
            </tr>
        </tbody>
    </table>
</div>
```

```css
.table {
    width: 100%;
    border-collapse: collapse;
}

.table th {
    text-align: left;
    padding: var(--space-3) var(--space-4);
    font-size: var(--text-xs);
    font-weight: var(--font-semibold);
    text-transform: uppercase;
    letter-spacing: 0.05em;
    color: var(--gray-500);
    background: var(--gray-50);
    border-bottom: 1px solid var(--gray-200);
}

.table td {
    padding: var(--space-4);
    border-bottom: 1px solid var(--gray-100);
}

.table tr:hover {
    background: var(--gray-50);
}
```

### 5.5 Badges & Tags

```html
<!-- Status Badges -->
<span class="badge badge-success">Aktiv</span>
<span class="badge badge-warning">Entwurf</span>
<span class="badge badge-error">Überfällig</span>
<span class="badge badge-info">Neu</span>

<!-- Energieklasse Badge -->
<span class="efficiency-badge efficiency-b">B</span>

<!-- Tags -->
<div class="tag-list">
    <span class="tag">Wohngebäude</span>
    <span class="tag">BEG-Förderung</span>
    <span class="tag removable">
        Neubau
        <button class="tag-remove">×</button>
    </span>
</div>
```

```css
.badge {
    display: inline-flex;
    align-items: center;
    padding: var(--space-1) var(--space-2);
    font-size: var(--text-xs);
    font-weight: var(--font-medium);
    border-radius: var(--radius-full);
}

.badge-success {
    background: var(--success-bg);
    color: var(--success);
}

.efficiency-badge {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 32px;
    height: 32px;
    font-size: var(--text-lg);
    font-weight: var(--font-bold);
    color: white;
    border-radius: var(--radius-md);
}

.efficiency-b {
    background: var(--efficiency-b);
    color: var(--gray-900);
}
```

---

## 6. Icons

### 6.1 Icon-System (Lucide)

```html
<!-- Lucide Icons - inline SVG für beste Performance -->
<svg
    class="icon"
    xmlns="http://www.w3.org/2000/svg"
    width="24"
    height="24"
    viewBox="0 0 24 24"
    fill="none"
    stroke="currentColor"
    stroke-width="2"
    stroke-linecap="round"
    stroke-linejoin="round"
>
    <!-- Icon path -->
</svg>

<!-- Größen -->
<svg class="icon icon-sm">...</svg>  <!-- 16px -->
<svg class="icon">...</svg>          <!-- 24px (Standard) -->
<svg class="icon icon-lg">...</svg>  <!-- 32px -->
```

### 6.2 Wichtige Icons

```
Navigation:
├── home         - Dashboard
├── folder       - Projekte
├── building-2   - Gebäude
├── calculator   - Berechnungen
├── file-text    - Berichte
├── users        - Kunden
├── settings     - Einstellungen

Aktionen:
├── plus         - Hinzufügen
├── edit         - Bearbeiten
├── trash-2      - Löschen
├── download     - Herunterladen
├── upload       - Hochladen
├── search       - Suchen
├── filter       - Filtern

Status:
├── check-circle - Erfolg
├── alert-circle - Warnung
├── x-circle     - Fehler
├── info         - Information
├── loader-2     - Laden

Energie:
├── zap          - Energie
├── thermometer  - Temperatur
├── sun          - Solar
├── wind         - Lüftung
├── flame        - Heizung
├── snowflake    - Kühlung
```

---

## 7. Animationen

### 7.1 Transitions

```css
/* Standard Transition */
--transition-fast:   150ms ease;
--transition-base:   200ms ease;
--transition-slow:   300ms ease;

/* Easing */
--ease-in:     cubic-bezier(0.4, 0, 1, 1);
--ease-out:    cubic-bezier(0, 0, 0.2, 1);
--ease-in-out: cubic-bezier(0.4, 0, 0.2, 1);
```

### 7.2 Keyframe Animationen

```css
/* Fade In */
@keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
}

/* Slide Up */
@keyframes slideUp {
    from {
        opacity: 0;
        transform: translateY(10px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

/* Spinner */
@keyframes spin {
    to { transform: rotate(360deg); }
}

/* Pulse */
@keyframes pulse {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.5; }
}

/* Verwendung */
.animate-fade-in {
    animation: fadeIn var(--transition-base) forwards;
}

.spinner {
    animation: spin 1s linear infinite;
}
```

---

## 8. Responsive Design

### 8.1 Breakpoints

```css
/* Mobile First */
/* Default: < 640px (Mobile) */

/* Small (sm): >= 640px */
@media (min-width: 640px) { }

/* Medium (md): >= 768px (Tablet) */
@media (min-width: 768px) { }

/* Large (lg): >= 1024px (Desktop) */
@media (min-width: 1024px) { }

/* Extra Large (xl): >= 1280px */
@media (min-width: 1280px) { }

/* 2XL: >= 1536px (Wide) */
@media (min-width: 1536px) { }
```

### 8.2 Responsive Patterns

```css
/* Mobile: Stack, Desktop: Side-by-Side */
.responsive-split {
    display: flex;
    flex-direction: column;
    gap: var(--space-4);
}

@media (min-width: 1024px) {
    .responsive-split {
        flex-direction: row;
    }
}

/* Sidebar ausblenden auf Mobile */
.sidebar {
    position: fixed;
    left: -100%;
    transition: left var(--transition-base);
}

.sidebar.open {
    left: 0;
}

@media (min-width: 1024px) {
    .sidebar {
        position: static;
        left: auto;
    }
}
```

---

## 9. Barrierefreiheit

### 9.1 WCAG Anforderungen

```
WCAG 2.1 AA Anforderungen:
├── Farbkontrast: Mindestens 4.5:1 für Text
├── Fokus-Indikatoren: Sichtbar für Tastaturnutzer
├── Alt-Texte: Für alle informativen Bilder
├── Labels: Für alle Formularelemente
├── Hierarchie: Logische Überschriftenstruktur
└── Keyboard: Alle Funktionen per Tastatur erreichbar
```

### 9.2 Fokus-Styles

```css
/* Fokus-Ring */
:focus-visible {
    outline: 2px solid var(--primary-500);
    outline-offset: 2px;
}

/* Buttons */
.btn:focus-visible {
    box-shadow: 0 0 0 3px var(--primary-200);
}

/* Skip-Link */
.skip-link {
    position: absolute;
    left: -9999px;
}

.skip-link:focus {
    left: var(--space-4);
    top: var(--space-4);
    z-index: 9999;
}
```

### 9.3 ARIA Patterns

```html
<!-- Beschreibende Labels -->
<button aria-label="Projekt löschen">
    <svg aria-hidden="true">...</svg>
</button>

<!-- Live Regions -->
<div
    role="status"
    aria-live="polite"
    aria-atomic="true"
>
    Änderungen gespeichert
</div>

<!-- Modal -->
<div
    role="dialog"
    aria-modal="true"
    aria-labelledby="modal-title"
>
    <h2 id="modal-title">Projekt erstellen</h2>
</div>

<!-- Tabs -->
<div role="tablist">
    <button
        role="tab"
        aria-selected="true"
        aria-controls="panel-1"
    >
        Tab 1
    </button>
</div>
<div
    role="tabpanel"
    id="panel-1"
    aria-labelledby="tab-1"
>
    Inhalt
</div>
```

---

## 10. Dark Mode (Optional)

```css
/* CSS Custom Properties für Dark Mode */
@media (prefers-color-scheme: dark) {
    :root {
        --bg-app:     #0f172a;
        --bg-card:    #1e293b;
        --bg-sidebar: #0f172a;

        --gray-50:  #1e293b;
        --gray-100: #334155;
        --gray-200: #475569;
        --gray-300: #64748b;
        --gray-400: #94a3b8;
        --gray-500: #cbd5e1;
        --gray-600: #e2e8f0;
        --gray-700: #f1f5f9;
        --gray-800: #f8fafc;
        --gray-900: #ffffff;
    }
}

/* Manuelle Toggle */
[data-theme="dark"] {
    /* Dark Mode Variablen */
}
```

---

*Letzte Aktualisierung: 2025-11-25*
*Version: 0.1.0*
