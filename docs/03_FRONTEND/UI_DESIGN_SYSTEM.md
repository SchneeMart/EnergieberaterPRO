# UI Design System - EnergieberaterPRO

## Konsistenz-Richtlinien

Dieses Dokument definiert die einheitlichen UI/UX-Standards für die gesamte Plattform.

## 1. Farb-System

```css
/* /static/css/variables.css */

:root {
    /* Primärfarben */
    --primary-50: #eff6ff;
    --primary-100: #dbeafe;
    --primary-200: #bfdbfe;
    --primary-300: #93c5fd;
    --primary-400: #60a5fa;
    --primary-500: #3b82f6;  /* Hauptfarbe */
    --primary-600: #2563eb;
    --primary-700: #1d4ed8;
    --primary-800: #1e40af;
    --primary-900: #1e3a8a;

    /* Graustufen */
    --gray-50: #f9fafb;
    --gray-100: #f3f4f6;
    --gray-200: #e5e7eb;
    --gray-300: #d1d5db;
    --gray-400: #9ca3af;
    --gray-500: #6b7280;
    --gray-600: #4b5563;
    --gray-700: #374151;
    --gray-800: #1f2937;
    --gray-900: #111827;

    /* Semantische Farben */
    --success-50: #ecfdf5;
    --success-500: #10b981;
    --success-600: #059669;

    --warning-50: #fffbeb;
    --warning-500: #f59e0b;
    --warning-600: #d97706;

    --danger-50: #fef2f2;
    --danger-500: #ef4444;
    --danger-600: #dc2626;

    --info-50: #eff6ff;
    --info-500: #3b82f6;
    --info-600: #2563eb;

    /* Energieeffizienz-Klassen */
    --efficiency-a-plus: #00a651;
    --efficiency-a: #4cb848;
    --efficiency-b: #bfd730;
    --efficiency-c: #fff200;
    --efficiency-d: #fdb913;
    --efficiency-e: #f37021;
    --efficiency-f: #ed1c24;
    --efficiency-g: #b2232f;
    --efficiency-h: #8b0000;
}
```

## 2. Typografie

```css
/* Schriftarten */
--font-sans: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
--font-mono: 'JetBrains Mono', 'Fira Code', monospace;

/* Schriftgrößen */
--text-xs: 0.75rem;      /* 12px */
--text-sm: 0.875rem;     /* 14px */
--text-base: 1rem;       /* 16px */
--text-lg: 1.125rem;     /* 18px */
--text-xl: 1.25rem;      /* 20px */
--text-2xl: 1.5rem;      /* 24px */
--text-3xl: 1.875rem;    /* 30px */
--text-4xl: 2.25rem;     /* 36px */

/* Zeilenhöhen */
--leading-tight: 1.25;
--leading-normal: 1.5;
--leading-relaxed: 1.75;

/* Schriftgewichte */
--font-normal: 400;
--font-medium: 500;
--font-semibold: 600;
--font-bold: 700;
```

## 3. Abstände

```css
/* Einheitliches Spacing-System (4px Basis) */
--space-0: 0;
--space-1: 0.25rem;   /* 4px */
--space-2: 0.5rem;    /* 8px */
--space-3: 0.75rem;   /* 12px */
--space-4: 1rem;      /* 16px */
--space-5: 1.25rem;   /* 20px */
--space-6: 1.5rem;    /* 24px */
--space-8: 2rem;      /* 32px */
--space-10: 2.5rem;   /* 40px */
--space-12: 3rem;     /* 48px */
--space-16: 4rem;     /* 64px */
```

## 4. Schatten

```css
--shadow-sm: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
--shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
--shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
--shadow-xl: 0 20px 25px -5px rgba(0, 0, 0, 0.1);
```

## 5. Radien

```css
--radius-sm: 0.25rem;   /* 4px */
--radius-md: 0.375rem;  /* 6px */
--radius-lg: 0.5rem;    /* 8px */
--radius-xl: 0.75rem;   /* 12px */
--radius-2xl: 1rem;     /* 16px */
--radius-full: 9999px;
```

---

## Komponenten-Bibliothek

### Buttons

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

<!-- Mit Icon -->
<button class="btn btn-primary">
    <svg class="btn-icon"><use href="#icon-plus"/></svg>
    Hinzufügen
</button>

<!-- Größen -->
<button class="btn btn-primary btn-sm">Klein</button>
<button class="btn btn-primary">Normal</button>
<button class="btn btn-primary btn-lg">Groß</button>

<!-- Loading -->
<button class="btn btn-primary" disabled>
    <span class="spinner"></span>
    Wird gespeichert...
</button>
```

```css
/* Button Styles */
.btn {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: var(--space-2);
    padding: var(--space-2) var(--space-4);
    font-size: var(--text-sm);
    font-weight: var(--font-medium);
    line-height: 1.5;
    border-radius: var(--radius-md);
    border: 1px solid transparent;
    cursor: pointer;
    transition: all 0.15s ease;
}

.btn:disabled {
    opacity: 0.5;
    cursor: not-allowed;
}

.btn-primary {
    background: var(--primary-600);
    color: white;
}

.btn-primary:hover:not(:disabled) {
    background: var(--primary-700);
}

.btn-secondary {
    background: var(--gray-100);
    color: var(--gray-700);
}

.btn-secondary:hover:not(:disabled) {
    background: var(--gray-200);
}

.btn-outline {
    background: transparent;
    border-color: var(--gray-300);
    color: var(--gray-700);
}

.btn-outline:hover:not(:disabled) {
    background: var(--gray-50);
    border-color: var(--gray-400);
}

.btn-danger {
    background: var(--danger-600);
    color: white;
}

.btn-sm { padding: var(--space-1) var(--space-3); font-size: var(--text-xs); }
.btn-lg { padding: var(--space-3) var(--space-6); font-size: var(--text-base); }
```

### Formulare

```html
<!-- Text Input -->
<div class="form-group">
    <label for="name" class="form-label">Name</label>
    <input type="text" id="name" class="form-input" placeholder="Ihr Name">
    <span class="form-hint">Optional: Wird auf Berichten angezeigt</span>
</div>

<!-- Mit Fehler -->
<div class="form-group error">
    <label for="email" class="form-label">E-Mail</label>
    <input type="email" id="email" class="form-input" value="ungültig">
    <span class="form-error">Bitte geben Sie eine gültige E-Mail-Adresse ein</span>
</div>

<!-- Select -->
<div class="form-group">
    <label for="type" class="form-label">Gebäudetyp</label>
    <select id="type" class="form-select">
        <option value="">Bitte wählen</option>
        <option value="efh">Einfamilienhaus</option>
        <option value="mfh">Mehrfamilienhaus</option>
    </select>
</div>

<!-- Textarea -->
<div class="form-group">
    <label for="notes" class="form-label">Notizen</label>
    <textarea id="notes" class="form-textarea" rows="4"></textarea>
</div>

<!-- Checkbox -->
<div class="form-group">
    <label class="form-checkbox">
        <input type="checkbox">
        <span class="checkbox-mark"></span>
        <span>Ich akzeptiere die AGB</span>
    </label>
</div>

<!-- Radio Group -->
<fieldset class="form-group">
    <legend class="form-label">Heizungsart</legend>
    <div class="radio-group">
        <label class="form-radio">
            <input type="radio" name="heating" value="gas">
            <span class="radio-mark"></span>
            <span>Gas</span>
        </label>
        <label class="form-radio">
            <input type="radio" name="heating" value="oil">
            <span class="radio-mark"></span>
            <span>Öl</span>
        </label>
        <label class="form-radio">
            <input type="radio" name="heating" value="wp">
            <span class="radio-mark"></span>
            <span>Wärmepumpe</span>
        </label>
    </div>
</fieldset>
```

```css
/* Form Styles */
.form-group {
    margin-bottom: var(--space-4);
}

.form-label {
    display: block;
    font-size: var(--text-sm);
    font-weight: var(--font-medium);
    color: var(--gray-700);
    margin-bottom: var(--space-1);
}

.form-input,
.form-select,
.form-textarea {
    width: 100%;
    padding: var(--space-2) var(--space-3);
    font-size: var(--text-sm);
    border: 1px solid var(--gray-300);
    border-radius: var(--radius-md);
    background: white;
    transition: border-color 0.15s, box-shadow 0.15s;
}

.form-input:focus,
.form-select:focus,
.form-textarea:focus {
    outline: none;
    border-color: var(--primary-500);
    box-shadow: 0 0 0 3px var(--primary-100);
}

.form-group.error .form-input {
    border-color: var(--danger-500);
}

.form-error {
    display: block;
    font-size: var(--text-xs);
    color: var(--danger-600);
    margin-top: var(--space-1);
}

.form-hint {
    display: block;
    font-size: var(--text-xs);
    color: var(--gray-500);
    margin-top: var(--space-1);
}
```

### Cards

```html
<!-- Standard Card -->
<div class="card">
    <div class="card-header">
        <h3 class="card-title">Projektübersicht</h3>
        <button class="btn btn-ghost btn-sm">Alle anzeigen</button>
    </div>
    <div class="card-body">
        <!-- Content -->
    </div>
    <div class="card-footer">
        <button class="btn btn-primary">Speichern</button>
    </div>
</div>

<!-- Stat Card -->
<div class="stat-card">
    <div class="stat-icon bg-primary">
        <svg><use href="#icon-folder"/></svg>
    </div>
    <div class="stat-content">
        <span class="stat-value">24</span>
        <span class="stat-label">Aktive Projekte</span>
    </div>
</div>
```

```css
.card {
    background: white;
    border: 1px solid var(--gray-200);
    border-radius: var(--radius-lg);
    overflow: hidden;
}

.card-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: var(--space-4);
    border-bottom: 1px solid var(--gray-200);
}

.card-title {
    font-size: var(--text-base);
    font-weight: var(--font-semibold);
    margin: 0;
}

.card-body {
    padding: var(--space-4);
}

.card-footer {
    padding: var(--space-4);
    background: var(--gray-50);
    border-top: 1px solid var(--gray-200);
}

.stat-card {
    display: flex;
    align-items: center;
    gap: var(--space-4);
    padding: var(--space-4);
    background: white;
    border: 1px solid var(--gray-200);
    border-radius: var(--radius-lg);
}

.stat-icon {
    width: 48px;
    height: 48px;
    border-radius: var(--radius-md);
    display: flex;
    align-items: center;
    justify-content: center;
}

.stat-icon.bg-primary { background: var(--primary-100); color: var(--primary-600); }
.stat-icon.bg-success { background: var(--success-50); color: var(--success-600); }
.stat-icon.bg-warning { background: var(--warning-50); color: var(--warning-600); }

.stat-value {
    display: block;
    font-size: var(--text-2xl);
    font-weight: var(--font-bold);
    color: var(--gray-900);
}

.stat-label {
    font-size: var(--text-sm);
    color: var(--gray-500);
}
```

### Tabellen

```html
<div class="table-container">
    <table class="table">
        <thead>
            <tr>
                <th>Projekt</th>
                <th>Kunde</th>
                <th>Status</th>
                <th>Fällig</th>
                <th class="text-right">Aktionen</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>
                    <div class="table-cell-main">
                        <span class="cell-title">Energieberatung EFH</span>
                        <span class="cell-sub">PRJ-2024-001</span>
                    </div>
                </td>
                <td>Max Mustermann</td>
                <td><span class="badge badge-success">Aktiv</span></td>
                <td>15.12.2024</td>
                <td class="text-right">
                    <button class="btn btn-ghost btn-sm">Öffnen</button>
                </td>
            </tr>
        </tbody>
    </table>
</div>
```

```css
.table-container {
    overflow-x: auto;
}

.table {
    width: 100%;
    border-collapse: collapse;
}

.table th,
.table td {
    padding: var(--space-3) var(--space-4);
    text-align: left;
    border-bottom: 1px solid var(--gray-200);
}

.table th {
    font-size: var(--text-xs);
    font-weight: var(--font-medium);
    color: var(--gray-500);
    text-transform: uppercase;
    letter-spacing: 0.05em;
    background: var(--gray-50);
}

.table tbody tr:hover {
    background: var(--gray-50);
}

.table-cell-main {
    display: flex;
    flex-direction: column;
}

.cell-title {
    font-weight: var(--font-medium);
    color: var(--gray-900);
}

.cell-sub {
    font-size: var(--text-xs);
    color: var(--gray-500);
}
```

### Badges

```html
<span class="badge">Standard</span>
<span class="badge badge-primary">Primär</span>
<span class="badge badge-success">Aktiv</span>
<span class="badge badge-warning">Ausstehend</span>
<span class="badge badge-danger">Überfällig</span>
<span class="badge badge-info">Neu</span>

<!-- Energieeffizienz -->
<span class="badge-efficiency badge-a-plus">A+</span>
<span class="badge-efficiency badge-a">A</span>
<span class="badge-efficiency badge-b">B</span>
<span class="badge-efficiency badge-c">C</span>
<span class="badge-efficiency badge-d">D</span>
<span class="badge-efficiency badge-e">E</span>
<span class="badge-efficiency badge-f">F</span>
<span class="badge-efficiency badge-g">G</span>
<span class="badge-efficiency badge-h">H</span>
```

```css
.badge {
    display: inline-flex;
    align-items: center;
    padding: var(--space-1) var(--space-2);
    font-size: var(--text-xs);
    font-weight: var(--font-medium);
    border-radius: var(--radius-full);
    background: var(--gray-100);
    color: var(--gray-700);
}

.badge-primary { background: var(--primary-100); color: var(--primary-700); }
.badge-success { background: var(--success-50); color: var(--success-600); }
.badge-warning { background: var(--warning-50); color: var(--warning-600); }
.badge-danger { background: var(--danger-50); color: var(--danger-600); }
.badge-info { background: var(--info-50); color: var(--info-600); }

/* Energieeffizienz-Badges */
.badge-efficiency {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 32px;
    height: 32px;
    font-size: var(--text-sm);
    font-weight: var(--font-bold);
    color: white;
    border-radius: var(--radius-sm);
}

.badge-a-plus { background: var(--efficiency-a-plus); }
.badge-a { background: var(--efficiency-a); }
.badge-b { background: var(--efficiency-b); color: var(--gray-900); }
.badge-c { background: var(--efficiency-c); color: var(--gray-900); }
.badge-d { background: var(--efficiency-d); }
.badge-e { background: var(--efficiency-e); }
.badge-f { background: var(--efficiency-f); }
.badge-g { background: var(--efficiency-g); }
.badge-h { background: var(--efficiency-h); }
```

### Alerts & Notifications

```html
<!-- Inline Alert -->
<div class="alert alert-info">
    <svg class="alert-icon"><use href="#icon-info"/></svg>
    <div class="alert-content">
        <strong>Info:</strong> Die Berechnung wurde gestartet.
    </div>
</div>

<div class="alert alert-success">
    <svg class="alert-icon"><use href="#icon-check-circle"/></svg>
    <div class="alert-content">
        Projekt erfolgreich gespeichert.
    </div>
    <button class="alert-close">&times;</button>
</div>

<div class="alert alert-warning">
    <svg class="alert-icon"><use href="#icon-alert-triangle"/></svg>
    <div class="alert-content">
        <strong>Achtung:</strong> Einige Pflichtfelder sind nicht ausgefüllt.
    </div>
</div>

<div class="alert alert-danger">
    <svg class="alert-icon"><use href="#icon-x-circle"/></svg>
    <div class="alert-content">
        <strong>Fehler:</strong> Die Berechnung konnte nicht durchgeführt werden.
    </div>
</div>

<!-- Toast Notification -->
<div class="toast toast-success">
    <svg class="toast-icon"><use href="#icon-check"/></svg>
    <span>Änderungen gespeichert</span>
</div>
```

```css
.alert {
    display: flex;
    align-items: flex-start;
    gap: var(--space-3);
    padding: var(--space-4);
    border-radius: var(--radius-md);
    border: 1px solid;
}

.alert-icon {
    width: 20px;
    height: 20px;
    flex-shrink: 0;
}

.alert-content {
    flex: 1;
}

.alert-close {
    background: none;
    border: none;
    font-size: 1.25rem;
    cursor: pointer;
    opacity: 0.5;
}

.alert-info {
    background: var(--info-50);
    border-color: var(--info-500);
    color: var(--info-600);
}

.alert-success {
    background: var(--success-50);
    border-color: var(--success-500);
    color: var(--success-600);
}

.alert-warning {
    background: var(--warning-50);
    border-color: var(--warning-500);
    color: var(--warning-600);
}

.alert-danger {
    background: var(--danger-50);
    border-color: var(--danger-500);
    color: var(--danger-600);
}

/* Toast */
.toast {
    position: fixed;
    bottom: var(--space-4);
    right: var(--space-4);
    display: flex;
    align-items: center;
    gap: var(--space-2);
    padding: var(--space-3) var(--space-4);
    background: var(--gray-800);
    color: white;
    border-radius: var(--radius-md);
    box-shadow: var(--shadow-lg);
    animation: toast-in 0.3s ease;
}

@keyframes toast-in {
    from {
        transform: translateY(100%);
        opacity: 0;
    }
    to {
        transform: translateY(0);
        opacity: 1;
    }
}
```

### Modale

```html
<!-- Modal -->
<div class="modal-backdrop" x-show="modalOpen" @click="modalOpen = false">
    <div class="modal" @click.stop>
        <div class="modal-header">
            <h2 class="modal-title">Projekt erstellen</h2>
            <button class="modal-close" @click="modalOpen = false">&times;</button>
        </div>
        <div class="modal-body">
            <!-- Content -->
        </div>
        <div class="modal-footer">
            <button class="btn btn-secondary" @click="modalOpen = false">Abbrechen</button>
            <button class="btn btn-primary">Erstellen</button>
        </div>
    </div>
</div>
```

```css
.modal-backdrop {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 1000;
}

.modal {
    background: white;
    border-radius: var(--radius-lg);
    width: 100%;
    max-width: 500px;
    max-height: 90vh;
    overflow: hidden;
    box-shadow: var(--shadow-xl);
}

.modal-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: var(--space-4);
    border-bottom: 1px solid var(--gray-200);
}

.modal-title {
    font-size: var(--text-lg);
    font-weight: var(--font-semibold);
    margin: 0;
}

.modal-close {
    background: none;
    border: none;
    font-size: 1.5rem;
    color: var(--gray-400);
    cursor: pointer;
}

.modal-body {
    padding: var(--space-4);
    overflow-y: auto;
}

.modal-footer {
    display: flex;
    justify-content: flex-end;
    gap: var(--space-2);
    padding: var(--space-4);
    background: var(--gray-50);
    border-top: 1px solid var(--gray-200);
}
```

---

## Layout-Komponenten

### App Shell (Nach Login)

```html
<div class="app-shell">
    <!-- Sidebar -->
    <aside class="sidebar">
        <!-- Navigation -->
    </aside>

    <!-- Main Area -->
    <div class="main-area">
        <!-- Header -->
        <header class="app-header">
            <!-- Breadcrumbs, Search, User Menu -->
        </header>

        <!-- Page Content -->
        <main class="page-content">
            <!-- Content -->
        </main>
    </div>
</div>
```

```css
.app-shell {
    display: flex;
    min-height: 100vh;
}

.sidebar {
    width: 260px;
    background: var(--gray-900);
    color: white;
    display: flex;
    flex-direction: column;
    position: fixed;
    height: 100vh;
    z-index: 100;
}

.main-area {
    flex: 1;
    margin-left: 260px;
    display: flex;
    flex-direction: column;
    min-height: 100vh;
}

.app-header {
    height: 60px;
    background: white;
    border-bottom: 1px solid var(--gray-200);
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 var(--space-6);
    position: sticky;
    top: 0;
    z-index: 50;
}

.page-content {
    flex: 1;
    padding: var(--space-6);
    background: var(--gray-50);
}
```

### Page Header

```html
<div class="page-header">
    <div class="page-header-left">
        <h1 class="page-title">Projekte</h1>
        <p class="page-description">Verwalten Sie Ihre Energieberatungsprojekte</p>
    </div>
    <div class="page-header-actions">
        <button class="btn btn-secondary">
            <svg class="btn-icon"><use href="#icon-download"/></svg>
            Export
        </button>
        <button class="btn btn-primary">
            <svg class="btn-icon"><use href="#icon-plus"/></svg>
            Neues Projekt
        </button>
    </div>
</div>
```

```css
.page-header {
    display: flex;
    align-items: flex-start;
    justify-content: space-between;
    margin-bottom: var(--space-6);
}

.page-title {
    font-size: var(--text-2xl);
    font-weight: var(--font-bold);
    color: var(--gray-900);
    margin: 0 0 var(--space-1);
}

.page-description {
    color: var(--gray-500);
    margin: 0;
}

.page-header-actions {
    display: flex;
    gap: var(--space-2);
}
```

---

## Benutzerfreundlichkeit-Richtlinien

### 1. Konsistente Aktionen

| Aktion | Primäre Platzierung | Sekundäre Platzierung |
|--------|---------------------|----------------------|
| Erstellen/Neu | Oben rechts auf der Seite | In leeren Zuständen |
| Speichern | Unten rechts im Formular | Sticky Footer bei langen Formularen |
| Abbrechen | Links vom Speichern-Button | |
| Löschen | In Dropdown-Menü | Nie prominent |
| Bearbeiten | Inline oder in Zeilen-Aktionen | |

### 2. Farbsemantik

- **Blau (Primary)**: Hauptaktionen, Links
- **Grün (Success)**: Bestätigungen, positive Zustände
- **Gelb (Warning)**: Warnungen, ausstehende Aktionen
- **Rot (Danger)**: Fehler, destruktive Aktionen
- **Grau**: Neutrale Informationen

### 3. Feedback

```javascript
// Toast-System für Feedback
const toast = {
    success(message) {
        this.show(message, 'success');
    },
    error(message) {
        this.show(message, 'error');
    },
    info(message) {
        this.show(message, 'info');
    },
    show(message, type) {
        const el = document.createElement('div');
        el.className = `toast toast-${type}`;
        el.innerHTML = `
            <svg class="toast-icon"><use href="#icon-${type === 'success' ? 'check' : type}"/></svg>
            <span>${message}</span>
        `;
        document.body.appendChild(el);
        setTimeout(() => el.remove(), 3000);
    }
};

// Verwendung
toast.success('Projekt gespeichert');
toast.error('Fehler beim Speichern');
```

### 4. Loading States

```html
<!-- Button Loading -->
<button class="btn btn-primary" disabled>
    <span class="spinner"></span>
    Wird geladen...
</button>

<!-- Skeleton Loading -->
<div class="skeleton skeleton-text"></div>
<div class="skeleton skeleton-text w-75"></div>
<div class="skeleton skeleton-rect h-200"></div>

<!-- Full Page Loading -->
<div class="loading-overlay">
    <div class="loading-spinner"></div>
    <p>Daten werden geladen...</p>
</div>
```

### 5. Leere Zustände

```html
<div class="empty-state">
    <svg class="empty-icon"><use href="#icon-folder"/></svg>
    <h3>Keine Projekte vorhanden</h3>
    <p>Erstellen Sie Ihr erstes Projekt, um loszulegen.</p>
    <button class="btn btn-primary">
        <svg class="btn-icon"><use href="#icon-plus"/></svg>
        Projekt erstellen
    </button>
</div>
```

```css
.empty-state {
    text-align: center;
    padding: var(--space-12) var(--space-4);
}

.empty-icon {
    width: 64px;
    height: 64px;
    color: var(--gray-300);
    margin-bottom: var(--space-4);
}

.empty-state h3 {
    font-size: var(--text-lg);
    color: var(--gray-900);
    margin-bottom: var(--space-2);
}

.empty-state p {
    color: var(--gray-500);
    margin-bottom: var(--space-4);
}
```

### 6. Tastaturnavigation

```css
/* Focus States */
*:focus-visible {
    outline: 2px solid var(--primary-500);
    outline-offset: 2px;
}

/* Skip Links */
.skip-link {
    position: absolute;
    top: -100%;
    left: 0;
    padding: var(--space-2) var(--space-4);
    background: var(--primary-600);
    color: white;
    z-index: 9999;
}

.skip-link:focus {
    top: 0;
}
```

### 7. Responsive Breakpoints

```css
/* Mobile First */
/* Base styles für Mobile */

/* Tablet */
@media (min-width: 768px) {
    /* Tablet-spezifische Styles */
}

/* Desktop */
@media (min-width: 1024px) {
    /* Desktop-spezifische Styles */
}

/* Large Desktop */
@media (min-width: 1280px) {
    /* Große Bildschirme */
}
```

---

## Barrierefreiheit (WCAG 2.1 AA)

1. **Kontrast**: Mindestens 4.5:1 für normalen Text, 3:1 für großen Text
2. **Fokus-Indikatoren**: Sichtbar für alle interaktiven Elemente
3. **Alt-Texte**: Für alle informativen Bilder
4. **ARIA-Labels**: Für Icons und nicht-textuelle Elemente
5. **Keyboard-Navigation**: Alle Funktionen per Tastatur erreichbar
6. **Error-Messages**: Mit `aria-live` für Screenreader

```html
<!-- Beispiel: Barrierefreier Button -->
<button class="btn btn-primary" aria-label="Neues Projekt erstellen">
    <svg aria-hidden="true"><use href="#icon-plus"/></svg>
    <span>Neu</span>
</button>

<!-- Beispiel: Formular mit Fehlern -->
<div class="form-group error" role="alert">
    <label for="email">E-Mail</label>
    <input type="email" id="email" aria-invalid="true" aria-describedby="email-error">
    <span id="email-error" class="form-error">Bitte geben Sie eine gültige E-Mail ein</span>
</div>
```
