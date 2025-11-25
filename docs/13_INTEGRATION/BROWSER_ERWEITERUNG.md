# Browser-Erweiterung für Formular-Autofill

## Referenzen
- [Chrome Extension Documentation](https://developer.chrome.com/docs/extensions/)
- [Firefox WebExtensions](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions)
- [Manifest V3](https://developer.chrome.com/docs/extensions/mv3/)

---

## 1. Übersicht

### 1.1 Funktionsumfang

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    BROWSER-ERWEITERUNG                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  HAUPTFUNKTIONEN:                                                           │
│  ═══════════════                                                            │
│                                                                             │
│  1. FORMULAR-AUTOFILL                                                       │
│     • BAFA-Formulare automatisch ausfüllen                                  │
│     • KfW-Anträge vorausfüllen                                             │
│     • Behörden-Formulare (Bauamt, etc.)                                    │
│     • Energieausweis-Datenbanken                                           │
│                                                                             │
│  2. DATEN-IMPORT                                                            │
│     • Gebäudedaten aus Webseiten extrahieren                               │
│     • Produktdaten (Fenster, Dämmung) importieren                          │
│     • Normwerte nachschlagen                                               │
│                                                                             │
│  3. SCHNELLZUGRIFF                                                          │
│     • Direkter Zugang zu EnergieberaterPRO                                 │
│     • Kontextbezogene Aktionen                                             │
│     • Projektauswahl                                                        │
│                                                                             │
│  4. WEB CLIPPER                                                             │
│     • Webseiten als Referenz speichern                                     │
│     • Screenshots zu Projekten hinzufügen                                  │
│     • Produktinformationen archivieren                                     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Architektur

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    EXTENSION ARCHITEKTUR                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                      BROWSER EXTENSION                               │   │
│  ├─────────────────────────────────────────────────────────────────────┤   │
│  │                                                                      │   │
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐              │   │
│  │  │  Popup UI   │    │  Background │    │  Content    │              │   │
│  │  │             │    │  Worker     │    │  Scripts    │              │   │
│  │  │  • Login    │    │  • API Comm │    │  • Form     │              │   │
│  │  │  • Projekt  │    │  • Storage  │    │    Detection│              │   │
│  │  │  • Actions  │    │  • Auth     │    │  • Autofill │              │   │
│  │  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘              │   │
│  │         │                  │                  │                      │   │
│  │         └──────────────────┼──────────────────┘                      │   │
│  │                            │ Message Passing                         │   │
│  └────────────────────────────┼─────────────────────────────────────────┘   │
│                               │                                             │
│                               │ HTTPS                                       │
│                               ▼                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                   ENERGIEBERATER PRO API                             │   │
│  │  • /api/v1/extension/auth                                            │   │
│  │  • /api/v1/extension/projects                                        │   │
│  │  • /api/v1/extension/autofill-data                                   │   │
│  │  • /api/v1/extension/clip                                            │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Extension Manifest

### 2.1 Manifest V3

```json
// manifest.json
{
    "manifest_version": 3,
    "name": "EnergieberaterPRO Assistant",
    "version": "1.0.0",
    "description": "Autofill für Förderanträge und Behördenformulare",

    "permissions": [
        "activeTab",
        "storage",
        "contextMenus",
        "notifications"
    ],

    "host_permissions": [
        "https://förderdatenbank.de/*",
        "https://www.bafa.de/*",
        "https://www.kfw.de/*",
        "https://www.energieausweis-online.de/*",
        "https://app.energieberaterpro.de/*"
    ],

    "background": {
        "service_worker": "background.js",
        "type": "module"
    },

    "action": {
        "default_popup": "popup/popup.html",
        "default_icon": {
            "16": "icons/icon-16.png",
            "32": "icons/icon-32.png",
            "48": "icons/icon-48.png",
            "128": "icons/icon-128.png"
        },
        "default_title": "EnergieberaterPRO"
    },

    "content_scripts": [
        {
            "matches": [
                "https://förderdatenbank.de/*",
                "https://www.bafa.de/*",
                "https://www.kfw.de/*"
            ],
            "js": ["content/autofill.js"],
            "css": ["content/autofill.css"],
            "run_at": "document_idle"
        },
        {
            "matches": ["<all_urls>"],
            "js": ["content/clipper.js"],
            "run_at": "document_idle"
        }
    ],

    "icons": {
        "16": "icons/icon-16.png",
        "32": "icons/icon-32.png",
        "48": "icons/icon-48.png",
        "128": "icons/icon-128.png"
    },

    "commands": {
        "_execute_action": {
            "suggested_key": {
                "default": "Ctrl+Shift+E",
                "mac": "Command+Shift+E"
            },
            "description": "EnergieberaterPRO öffnen"
        },
        "autofill": {
            "suggested_key": {
                "default": "Ctrl+Shift+F",
                "mac": "Command+Shift+F"
            },
            "description": "Formular ausfüllen"
        },
        "clip": {
            "suggested_key": {
                "default": "Ctrl+Shift+C",
                "mac": "Command+Shift+C"
            },
            "description": "Seite speichern"
        }
    },

    "options_page": "options/options.html",

    "web_accessible_resources": [
        {
            "resources": ["icons/*", "content/*.css"],
            "matches": ["<all_urls>"]
        }
    ]
}
```

---

## 3. Background Service Worker

```javascript
// background.js
/**
 * Background Service Worker für Extension
 */

// ============================================================================
// State Management
// ============================================================================

let authState = {
    isAuthenticated: false,
    accessToken: null,
    user: null,
    organization: null
};

let activeProject = null;

// ============================================================================
// Initialization
// ============================================================================

chrome.runtime.onInstalled.addListener(() => {
    console.log('EnergieberaterPRO Extension installed');

    // Context Menus erstellen
    createContextMenus();

    // Auth-Status laden
    loadAuthState();
});

function createContextMenus() {
    chrome.contextMenus.create({
        id: 'ebpro-clip',
        title: 'Zu EnergieberaterPRO hinzufügen',
        contexts: ['page', 'selection', 'image']
    });

    chrome.contextMenus.create({
        id: 'ebpro-autofill',
        title: 'Formular mit Projektdaten füllen',
        contexts: ['page']
    });

    chrome.contextMenus.create({
        id: 'ebpro-lookup',
        title: 'In Materialdatenbank suchen',
        contexts: ['selection']
    });
}

// ============================================================================
// Authentication
// ============================================================================

async function loadAuthState() {
    const stored = await chrome.storage.local.get(['authState', 'activeProject']);

    if (stored.authState) {
        authState = stored.authState;
    }

    if (stored.activeProject) {
        activeProject = stored.activeProject;
    }

    // Token validieren
    if (authState.accessToken) {
        const isValid = await validateToken(authState.accessToken);
        if (!isValid) {
            authState = { isAuthenticated: false, accessToken: null, user: null };
            await chrome.storage.local.remove(['authState']);
        }
    }
}

async function validateToken(token) {
    try {
        const response = await fetch(`${API_BASE_URL}/api/v1/users/me`, {
            headers: { 'Authorization': `Bearer ${token}` }
        });
        return response.ok;
    } catch {
        return false;
    }
}

async function login(email, password) {
    try {
        const response = await fetch(`${API_BASE_URL}/api/v1/auth/login`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ email, password })
        });

        if (!response.ok) {
            throw new Error('Login fehlgeschlagen');
        }

        const data = await response.json();

        authState = {
            isAuthenticated: true,
            accessToken: data.access_token,
            user: data.user,
            organization: data.organization
        };

        await chrome.storage.local.set({ authState });

        return { success: true };
    } catch (error) {
        return { success: false, error: error.message };
    }
}

async function logout() {
    authState = { isAuthenticated: false, accessToken: null, user: null };
    activeProject = null;
    await chrome.storage.local.remove(['authState', 'activeProject']);
}

// ============================================================================
// API Communication
// ============================================================================

const API_BASE_URL = 'https://app.energieberaterpro.de';

async function apiRequest(endpoint, options = {}) {
    if (!authState.accessToken) {
        throw new Error('Nicht authentifiziert');
    }

    const response = await fetch(`${API_BASE_URL}${endpoint}`, {
        ...options,
        headers: {
            'Authorization': `Bearer ${authState.accessToken}`,
            'Content-Type': 'application/json',
            ...options.headers
        }
    });

    if (response.status === 401) {
        await logout();
        throw new Error('Session abgelaufen');
    }

    return response.json();
}

async function getProjects() {
    return apiRequest('/api/v1/extension/projects?limit=20');
}

async function getAutofillData(projectId, formType) {
    return apiRequest(`/api/v1/extension/autofill-data?project_id=${projectId}&form_type=${formType}`);
}

async function saveClip(clipData) {
    return apiRequest('/api/v1/extension/clip', {
        method: 'POST',
        body: JSON.stringify(clipData)
    });
}

// ============================================================================
// Message Handling
// ============================================================================

chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
    handleMessage(message, sender).then(sendResponse);
    return true; // Async response
});

async function handleMessage(message, sender) {
    switch (message.type) {
        case 'GET_AUTH_STATE':
            return { authState, activeProject };

        case 'LOGIN':
            return await login(message.email, message.password);

        case 'LOGOUT':
            await logout();
            return { success: true };

        case 'GET_PROJECTS':
            return await getProjects();

        case 'SET_ACTIVE_PROJECT':
            activeProject = message.project;
            await chrome.storage.local.set({ activeProject });
            return { success: true };

        case 'GET_AUTOFILL_DATA':
            return await getAutofillData(message.projectId, message.formType);

        case 'SAVE_CLIP':
            return await saveClip(message.clipData);

        case 'LOOKUP_MATERIAL':
            return await lookupMaterial(message.query);

        default:
            return { error: 'Unknown message type' };
    }
}

// ============================================================================
// Context Menu Handlers
// ============================================================================

chrome.contextMenus.onClicked.addListener(async (info, tab) => {
    switch (info.menuItemId) {
        case 'ebpro-clip':
            await handleClipAction(info, tab);
            break;

        case 'ebpro-autofill':
            await handleAutofillAction(tab);
            break;

        case 'ebpro-lookup':
            await handleLookupAction(info.selectionText);
            break;
    }
});

async function handleClipAction(info, tab) {
    if (!authState.isAuthenticated) {
        chrome.notifications.create({
            type: 'basic',
            iconUrl: 'icons/icon-48.png',
            title: 'Nicht angemeldet',
            message: 'Bitte melden Sie sich zuerst an.'
        });
        return;
    }

    const clipData = {
        url: tab.url,
        title: tab.title,
        type: info.srcUrl ? 'image' : (info.selectionText ? 'selection' : 'page'),
        content: info.selectionText || info.srcUrl || null,
        project_id: activeProject?.id
    };

    try {
        await saveClip(clipData);
        chrome.notifications.create({
            type: 'basic',
            iconUrl: 'icons/icon-48.png',
            title: 'Gespeichert',
            message: `Inhalt wurde ${activeProject ? `zu "${activeProject.name}"` : ''} hinzugefügt.`
        });
    } catch (error) {
        chrome.notifications.create({
            type: 'basic',
            iconUrl: 'icons/icon-48.png',
            title: 'Fehler',
            message: error.message
        });
    }
}

async function handleAutofillAction(tab) {
    chrome.tabs.sendMessage(tab.id, {
        type: 'TRIGGER_AUTOFILL',
        projectId: activeProject?.id
    });
}

async function handleLookupAction(query) {
    const result = await lookupMaterial(query);

    if (result.materials && result.materials.length > 0) {
        // Popup mit Ergebnissen öffnen
        chrome.windows.create({
            url: `lookup/results.html?query=${encodeURIComponent(query)}`,
            type: 'popup',
            width: 400,
            height: 500
        });
    }
}

async function lookupMaterial(query) {
    return apiRequest(`/api/v1/materials/search?q=${encodeURIComponent(query)}`);
}

// ============================================================================
// Keyboard Shortcuts
// ============================================================================

chrome.commands.onCommand.addListener(async (command) => {
    const [tab] = await chrome.tabs.query({ active: true, currentWindow: true });

    switch (command) {
        case 'autofill':
            await handleAutofillAction(tab);
            break;

        case 'clip':
            await handleClipAction({ selectionText: null }, tab);
            break;
    }
});
```

---

## 4. Content Scripts

### 4.1 Autofill Script

```javascript
// content/autofill.js
/**
 * Content Script für Formular-Autofill
 */

// ============================================================================
// Form Detection
// ============================================================================

const KNOWN_FORMS = {
    'bafa.de': {
        patterns: {
            '/beg/': 'bafa_beg',
            '/isfp/': 'bafa_isfp',
            '/energieberatung/': 'bafa_beratung'
        },
        fieldMappings: {
            'bafa_beg': {
                'vorname': { selector: '#vorname, [name="vorname"]', dataKey: 'customer.first_name' },
                'nachname': { selector: '#nachname, [name="nachname"]', dataKey: 'customer.last_name' },
                'strasse': { selector: '#strasse, [name="strasse"]', dataKey: 'building.address.street' },
                'plz': { selector: '#plz, [name="plz"]', dataKey: 'building.address.zip_code' },
                'ort': { selector: '#ort, [name="ort"]', dataKey: 'building.address.city' },
                'baujahr': { selector: '#baujahr, [name="baujahr"]', dataKey: 'building.construction_year' },
                'wohnflaeche': { selector: '#wohnflaeche, [name="wohnflaeche"]', dataKey: 'building.heated_area' },
            }
        }
    },
    'kfw.de': {
        patterns: {
            '/antrag/': 'kfw_antrag',
            '/foerderung/': 'kfw_foerderung'
        },
        fieldMappings: {
            'kfw_antrag': {
                'gebaeude_typ': { selector: '[name="gebaeudetyp"]', dataKey: 'building.building_type' },
                'massnahme': { selector: '[name="massnahme"]', dataKey: 'project.measure_type' },
                'investitionskosten': { selector: '[name="investitionskosten"]', dataKey: 'calculation.investment_cost' }
            }
        }
    }
};

class FormAutofiller {
    constructor() {
        this.currentForm = null;
        this.formData = null;

        this.init();
    }

    init() {
        // Formular-Typ erkennen
        this.detectFormType();

        // Autofill-Button einfügen
        this.injectAutofillButton();

        // Message Listener
        chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
            if (message.type === 'TRIGGER_AUTOFILL') {
                this.triggerAutofill(message.projectId);
            }
        });
    }

    detectFormType() {
        const hostname = window.location.hostname.replace('www.', '');
        const pathname = window.location.pathname;

        const siteConfig = KNOWN_FORMS[hostname];
        if (!siteConfig) return;

        for (const [pattern, formType] of Object.entries(siteConfig.patterns)) {
            if (pathname.includes(pattern)) {
                this.currentForm = {
                    type: formType,
                    site: hostname,
                    mappings: siteConfig.fieldMappings[formType]
                };
                console.log('[EBPro] Form detected:', this.currentForm.type);
                break;
            }
        }
    }

    injectAutofillButton() {
        if (!this.currentForm) return;

        // Button-Container erstellen
        const container = document.createElement('div');
        container.id = 'ebpro-autofill-container';
        container.innerHTML = `
            <button id="ebpro-autofill-btn" class="ebpro-btn">
                <img src="${chrome.runtime.getURL('icons/icon-32.png')}" alt="">
                <span>Mit EnergieberaterPRO ausfüllen</span>
            </button>
            <div id="ebpro-autofill-status"></div>
        `;

        // In Seite einfügen
        const form = document.querySelector('form');
        if (form) {
            form.insertBefore(container, form.firstChild);
        } else {
            document.body.insertBefore(container, document.body.firstChild);
        }

        // Event Listener
        document.getElementById('ebpro-autofill-btn').addEventListener('click', () => {
            this.showProjectSelector();
        });
    }

    async showProjectSelector() {
        // Auth prüfen
        const response = await chrome.runtime.sendMessage({ type: 'GET_AUTH_STATE' });

        if (!response.authState?.isAuthenticated) {
            this.showStatus('Bitte zuerst in der Extension anmelden.', 'error');
            return;
        }

        // Projekte laden
        const projects = await chrome.runtime.sendMessage({ type: 'GET_PROJECTS' });

        // Projekt-Auswahl-Dialog
        this.showProjectDialog(projects.projects);
    }

    showProjectDialog(projects) {
        const dialog = document.createElement('div');
        dialog.id = 'ebpro-project-dialog';
        dialog.innerHTML = `
            <div class="ebpro-dialog-overlay"></div>
            <div class="ebpro-dialog-content">
                <h3>Projekt auswählen</h3>
                <div class="ebpro-project-list">
                    ${projects.map(p => `
                        <div class="ebpro-project-item" data-id="${p.id}">
                            <div class="project-name">${p.name}</div>
                            <div class="project-info">${p.customer_name || ''} - ${p.address || ''}</div>
                        </div>
                    `).join('')}
                </div>
                <button class="ebpro-dialog-close">Abbrechen</button>
            </div>
        `;

        document.body.appendChild(dialog);

        // Event Listener
        dialog.querySelectorAll('.ebpro-project-item').forEach(item => {
            item.addEventListener('click', () => {
                this.selectProject(item.dataset.id);
                dialog.remove();
            });
        });

        dialog.querySelector('.ebpro-dialog-close').addEventListener('click', () => {
            dialog.remove();
        });

        dialog.querySelector('.ebpro-dialog-overlay').addEventListener('click', () => {
            dialog.remove();
        });
    }

    async selectProject(projectId) {
        this.showStatus('Lade Projektdaten...', 'loading');

        try {
            // Autofill-Daten laden
            const data = await chrome.runtime.sendMessage({
                type: 'GET_AUTOFILL_DATA',
                projectId: projectId,
                formType: this.currentForm.type
            });

            this.formData = data;
            await this.fillForm();

            this.showStatus('Formular ausgefüllt!', 'success');
        } catch (error) {
            this.showStatus('Fehler: ' + error.message, 'error');
        }
    }

    async triggerAutofill(projectId) {
        if (!projectId) {
            this.showProjectSelector();
            return;
        }

        await this.selectProject(projectId);
    }

    async fillForm() {
        if (!this.formData || !this.currentForm?.mappings) {
            return;
        }

        let filledCount = 0;

        for (const [fieldName, config] of Object.entries(this.currentForm.mappings)) {
            const element = document.querySelector(config.selector);
            if (!element) continue;

            const value = this.getNestedValue(this.formData, config.dataKey);
            if (value === undefined || value === null) continue;

            // Feld ausfüllen
            await this.fillField(element, value);
            filledCount++;

            // Highlight
            this.highlightField(element);
        }

        console.log(`[EBPro] Filled ${filledCount} fields`);
    }

    async fillField(element, value) {
        const tagName = element.tagName.toLowerCase();

        if (tagName === 'input') {
            const type = element.type.toLowerCase();

            if (type === 'checkbox') {
                element.checked = Boolean(value);
            } else if (type === 'radio') {
                const radio = document.querySelector(`input[name="${element.name}"][value="${value}"]`);
                if (radio) radio.checked = true;
            } else {
                element.value = value;
            }
        } else if (tagName === 'select') {
            element.value = value;
        } else if (tagName === 'textarea') {
            element.value = value;
        }

        // Events triggern für Validierung
        element.dispatchEvent(new Event('input', { bubbles: true }));
        element.dispatchEvent(new Event('change', { bubbles: true }));
    }

    highlightField(element) {
        element.classList.add('ebpro-filled');

        // Animation
        element.style.transition = 'background-color 0.3s';
        element.style.backgroundColor = '#e8f5e9';

        setTimeout(() => {
            element.style.backgroundColor = '';
        }, 2000);
    }

    getNestedValue(obj, path) {
        return path.split('.').reduce((current, key) => current?.[key], obj);
    }

    showStatus(message, type) {
        const status = document.getElementById('ebpro-autofill-status');
        if (!status) return;

        status.textContent = message;
        status.className = `ebpro-status ebpro-status-${type}`;

        if (type !== 'loading') {
            setTimeout(() => {
                status.textContent = '';
                status.className = '';
            }, 5000);
        }
    }
}

// Initialisierung
const autofiller = new FormAutofiller();
```

### 4.2 Web Clipper Script

```javascript
// content/clipper.js
/**
 * Web Clipper für Seiteninhalte
 */

class WebClipper {
    constructor() {
        this.isActive = false;
        this.selectedElements = [];

        this.init();
    }

    init() {
        // Keyboard Shortcut
        document.addEventListener('keydown', (e) => {
            if (e.ctrlKey && e.shiftKey && e.key === 'C') {
                this.toggleClipMode();
            }
        });

        // Message Listener
        chrome.runtime.onMessage.addListener((message) => {
            if (message.type === 'ACTIVATE_CLIPPER') {
                this.activateClipMode();
            }
        });
    }

    toggleClipMode() {
        this.isActive ? this.deactivateClipMode() : this.activateClipMode();
    }

    activateClipMode() {
        this.isActive = true;
        document.body.classList.add('ebpro-clip-mode');

        // Overlay
        this.createOverlay();

        // Element-Hover-Handler
        document.addEventListener('mouseover', this.handleHover.bind(this), true);
        document.addEventListener('click', this.handleClick.bind(this), true);
        document.addEventListener('keydown', this.handleKeydown.bind(this), true);
    }

    deactivateClipMode() {
        this.isActive = false;
        document.body.classList.remove('ebpro-clip-mode');

        // Cleanup
        this.removeOverlay();
        document.querySelectorAll('.ebpro-highlight').forEach(el => {
            el.classList.remove('ebpro-highlight');
        });

        document.removeEventListener('mouseover', this.handleHover.bind(this), true);
        document.removeEventListener('click', this.handleClick.bind(this), true);
    }

    createOverlay() {
        const overlay = document.createElement('div');
        overlay.id = 'ebpro-clip-overlay';
        overlay.innerHTML = `
            <div class="ebpro-clip-toolbar">
                <span>Clip-Modus aktiv - Klicken Sie auf Elemente zum Auswählen</span>
                <button id="ebpro-clip-save">Speichern</button>
                <button id="ebpro-clip-cancel">Abbrechen (Esc)</button>
            </div>
        `;
        document.body.appendChild(overlay);

        document.getElementById('ebpro-clip-save').addEventListener('click', () => {
            this.saveClip();
        });

        document.getElementById('ebpro-clip-cancel').addEventListener('click', () => {
            this.deactivateClipMode();
        });
    }

    removeOverlay() {
        document.getElementById('ebpro-clip-overlay')?.remove();
    }

    handleHover(e) {
        if (!this.isActive) return;

        // Vorherige Highlights entfernen
        document.querySelectorAll('.ebpro-hover').forEach(el => {
            el.classList.remove('ebpro-hover');
        });

        // Neues Element highlighten
        if (!e.target.closest('#ebpro-clip-overlay')) {
            e.target.classList.add('ebpro-hover');
        }
    }

    handleClick(e) {
        if (!this.isActive) return;

        // Toolbar-Clicks ignorieren
        if (e.target.closest('#ebpro-clip-overlay')) return;

        e.preventDefault();
        e.stopPropagation();

        const element = e.target;

        if (element.classList.contains('ebpro-selected')) {
            // Deselektieren
            element.classList.remove('ebpro-selected');
            this.selectedElements = this.selectedElements.filter(el => el !== element);
        } else {
            // Selektieren
            element.classList.add('ebpro-selected');
            this.selectedElements.push(element);
        }
    }

    handleKeydown(e) {
        if (e.key === 'Escape') {
            this.deactivateClipMode();
        }
    }

    async saveClip() {
        if (this.selectedElements.length === 0) {
            alert('Bitte wählen Sie mindestens ein Element aus.');
            return;
        }

        // Content zusammenstellen
        const content = this.selectedElements.map(el => {
            if (el.tagName === 'IMG') {
                return { type: 'image', src: el.src, alt: el.alt };
            } else if (el.tagName === 'TABLE') {
                return { type: 'table', html: el.outerHTML, data: this.parseTable(el) };
            } else {
                return { type: 'text', text: el.innerText, html: el.innerHTML };
            }
        });

        const clipData = {
            url: window.location.href,
            title: document.title,
            clipped_at: new Date().toISOString(),
            content: content
        };

        try {
            await chrome.runtime.sendMessage({
                type: 'SAVE_CLIP',
                clipData: clipData
            });

            alert('Inhalt wurde gespeichert!');
            this.deactivateClipMode();
        } catch (error) {
            alert('Fehler beim Speichern: ' + error.message);
        }
    }

    parseTable(table) {
        const data = [];
        const rows = table.querySelectorAll('tr');

        rows.forEach(row => {
            const rowData = [];
            row.querySelectorAll('th, td').forEach(cell => {
                rowData.push(cell.innerText.trim());
            });
            data.push(rowData);
        });

        return data;
    }
}

// Initialisierung
const clipper = new WebClipper();
```

---

## 5. Popup UI

```html
<!-- popup/popup.html -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <link rel="stylesheet" href="popup.css">
</head>
<body>
    <div id="app">
        <!-- Login View -->
        <div id="login-view" class="view">
            <div class="logo">
                <img src="../icons/icon-48.png" alt="EnergieberaterPRO">
                <h1>EnergieberaterPRO</h1>
            </div>

            <form id="login-form">
                <div class="form-group">
                    <label>E-Mail</label>
                    <input type="email" id="email" required>
                </div>
                <div class="form-group">
                    <label>Passwort</label>
                    <input type="password" id="password" required>
                </div>
                <button type="submit" class="btn-primary">Anmelden</button>
                <div id="login-error" class="error"></div>
            </form>
        </div>

        <!-- Main View -->
        <div id="main-view" class="view hidden">
            <div class="header">
                <div class="user-info">
                    <span id="user-name"></span>
                    <span id="org-name"></span>
                </div>
                <button id="logout-btn" class="btn-icon" title="Abmelden">
                    <svg><!-- Logout Icon --></svg>
                </button>
            </div>

            <div class="section">
                <h3>Aktives Projekt</h3>
                <div id="active-project">
                    <span class="no-project">Kein Projekt ausgewählt</span>
                </div>
                <button id="select-project-btn" class="btn-secondary">
                    Projekt auswählen
                </button>
            </div>

            <div class="section">
                <h3>Schnellaktionen</h3>
                <div class="quick-actions">
                    <button id="action-autofill" class="action-btn">
                        <svg><!-- Form Icon --></svg>
                        <span>Formular ausfüllen</span>
                    </button>
                    <button id="action-clip" class="action-btn">
                        <svg><!-- Clip Icon --></svg>
                        <span>Seite speichern</span>
                    </button>
                    <button id="action-open" class="action-btn">
                        <svg><!-- External Icon --></svg>
                        <span>App öffnen</span>
                    </button>
                </div>
            </div>

            <div class="footer">
                <a href="#" id="settings-link">Einstellungen</a>
                <a href="https://app.energieberaterpro.de/help" target="_blank">Hilfe</a>
            </div>
        </div>

        <!-- Project Selector -->
        <div id="project-selector" class="view hidden">
            <div class="header">
                <button id="back-btn" class="btn-icon">
                    <svg><!-- Back Icon --></svg>
                </button>
                <h2>Projekt auswählen</h2>
            </div>

            <div class="search-box">
                <input type="text" id="project-search" placeholder="Suchen...">
            </div>

            <div id="project-list" class="project-list">
                <!-- Dynamisch gefüllt -->
            </div>
        </div>
    </div>

    <script src="popup.js"></script>
</body>
</html>
```

```javascript
// popup/popup.js
/**
 * Popup UI Logic
 */

class PopupApp {
    constructor() {
        this.views = {
            login: document.getElementById('login-view'),
            main: document.getElementById('main-view'),
            projectSelector: document.getElementById('project-selector')
        };

        this.state = {
            authState: null,
            activeProject: null,
            projects: []
        };

        this.init();
    }

    async init() {
        // Auth-Status laden
        const response = await chrome.runtime.sendMessage({ type: 'GET_AUTH_STATE' });
        this.state.authState = response.authState;
        this.state.activeProject = response.activeProject;

        // View anzeigen
        if (this.state.authState?.isAuthenticated) {
            this.showMainView();
        } else {
            this.showLoginView();
        }

        // Event Listener
        this.setupEventListeners();
    }

    setupEventListeners() {
        // Login
        document.getElementById('login-form').addEventListener('submit', (e) => {
            e.preventDefault();
            this.handleLogin();
        });

        // Logout
        document.getElementById('logout-btn').addEventListener('click', () => {
            this.handleLogout();
        });

        // Projekt auswählen
        document.getElementById('select-project-btn').addEventListener('click', () => {
            this.showProjectSelector();
        });

        document.getElementById('back-btn').addEventListener('click', () => {
            this.showMainView();
        });

        // Quick Actions
        document.getElementById('action-autofill').addEventListener('click', () => {
            this.triggerAutofill();
        });

        document.getElementById('action-clip').addEventListener('click', () => {
            this.triggerClip();
        });

        document.getElementById('action-open').addEventListener('click', () => {
            chrome.tabs.create({ url: 'https://app.energieberaterpro.de' });
        });

        // Projekt-Suche
        document.getElementById('project-search').addEventListener('input', (e) => {
            this.filterProjects(e.target.value);
        });
    }

    async handleLogin() {
        const email = document.getElementById('email').value;
        const password = document.getElementById('password').value;
        const errorEl = document.getElementById('login-error');

        errorEl.textContent = '';

        const response = await chrome.runtime.sendMessage({
            type: 'LOGIN',
            email,
            password
        });

        if (response.success) {
            const state = await chrome.runtime.sendMessage({ type: 'GET_AUTH_STATE' });
            this.state.authState = state.authState;
            this.showMainView();
        } else {
            errorEl.textContent = response.error || 'Anmeldung fehlgeschlagen';
        }
    }

    async handleLogout() {
        await chrome.runtime.sendMessage({ type: 'LOGOUT' });
        this.state.authState = null;
        this.state.activeProject = null;
        this.showLoginView();
    }

    showView(viewName) {
        Object.values(this.views).forEach(v => v.classList.add('hidden'));
        this.views[viewName].classList.remove('hidden');
    }

    showLoginView() {
        this.showView('login');
    }

    showMainView() {
        this.showView('main');

        // User Info
        document.getElementById('user-name').textContent = this.state.authState?.user?.name || '';
        document.getElementById('org-name').textContent = this.state.authState?.organization?.name || '';

        // Aktives Projekt
        this.updateActiveProjectDisplay();
    }

    updateActiveProjectDisplay() {
        const container = document.getElementById('active-project');

        if (this.state.activeProject) {
            container.innerHTML = `
                <div class="project-item active">
                    <div class="project-name">${this.state.activeProject.name}</div>
                    <div class="project-info">${this.state.activeProject.customer_name || ''}</div>
                </div>
            `;
        } else {
            container.innerHTML = '<span class="no-project">Kein Projekt ausgewählt</span>';
        }
    }

    async showProjectSelector() {
        this.showView('projectSelector');

        // Projekte laden
        const response = await chrome.runtime.sendMessage({ type: 'GET_PROJECTS' });
        this.state.projects = response.projects || [];

        this.renderProjects(this.state.projects);
    }

    renderProjects(projects) {
        const list = document.getElementById('project-list');

        list.innerHTML = projects.map(p => `
            <div class="project-item" data-id="${p.id}">
                <div class="project-name">${p.name}</div>
                <div class="project-info">${p.customer_name || ''} - ${p.status}</div>
            </div>
        `).join('');

        // Click Handler
        list.querySelectorAll('.project-item').forEach(item => {
            item.addEventListener('click', () => {
                this.selectProject(item.dataset.id);
            });
        });
    }

    filterProjects(query) {
        const filtered = this.state.projects.filter(p =>
            p.name.toLowerCase().includes(query.toLowerCase()) ||
            (p.customer_name && p.customer_name.toLowerCase().includes(query.toLowerCase()))
        );
        this.renderProjects(filtered);
    }

    async selectProject(projectId) {
        const project = this.state.projects.find(p => p.id === projectId);

        await chrome.runtime.sendMessage({
            type: 'SET_ACTIVE_PROJECT',
            project
        });

        this.state.activeProject = project;
        this.showMainView();
    }

    async triggerAutofill() {
        const [tab] = await chrome.tabs.query({ active: true, currentWindow: true });

        chrome.tabs.sendMessage(tab.id, {
            type: 'TRIGGER_AUTOFILL',
            projectId: this.state.activeProject?.id
        });

        window.close();
    }

    async triggerClip() {
        const [tab] = await chrome.tabs.query({ active: true, currentWindow: true });

        chrome.tabs.sendMessage(tab.id, {
            type: 'ACTIVATE_CLIPPER'
        });

        window.close();
    }
}

// Initialisierung
const app = new PopupApp();
```

---

## 6. Checkliste

```
✓ Manifest V3 Konfiguration
✓ Background Service Worker
✓ Content Scripts für Autofill
✓ Web Clipper Funktionalität
✓ Popup UI mit Login
✓ Projektauswahl
✓ BAFA/KfW Formular-Mappings
✓ Context Menus
✓ Keyboard Shortcuts
✓ Materialdatenbank-Lookup
```

---

*Letzte Aktualisierung: 2025-11-25*
*Version: 1.0.0*
