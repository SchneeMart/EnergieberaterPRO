# Frontend-Architektur

## 1. Technologie-Entscheidungen

### 1.1 Warum Vanilla JS + Alpine.js?

| Kriterium | Vanilla JS + Alpine | React/Vue/Angular |
|-----------|---------------------|-------------------|
| Bundle-Größe | ~15KB (Alpine) | 100-300KB |
| Build-Prozess | Keiner nötig | Webpack/Vite |
| Lernkurve | Gering | Hoch |
| Wartbarkeit | Hoch | Mittel |
| Performance | Sehr gut | Gut |
| SEO | Gut | SSR nötig |
| Offline-Fähigkeit | Einfach | Komplex |

### 1.2 Tech Stack

```
Core:
├── Alpine.js 3.x          # Reaktivität
├── Petite-Vue (optional)  # Alternative zu Alpine
└── HTMX (optional)        # Server-driven UI

Styling:
├── Tailwind CSS 3.x       # Utility-First CSS
├── Custom Design System   # Eigene Komponenten
└── Lucide Icons           # SVG Icons

Charts & Visualisierung:
├── D3.js                  # Komplexe Visualisierungen
├── Chart.js               # Standard-Charts
└── Plotly.js              # Interaktive Charts

Utilities:
├── Day.js                 # Datum/Zeit
├── IMask                  # Eingabemasken
├── SortableJS             # Drag & Drop
└── PDF.js                 # PDF-Anzeige
```

---

## 2. Projektstruktur

```
frontend/
├── index.html                      # Einstiegspunkt
├── app.html                        # Hauptanwendung (nach Login)
│
├── assets/
│   ├── css/
│   │   ├── main.css               # Haupt-CSS (Tailwind)
│   │   ├── design-system.css      # Design System Tokens
│   │   ├── components.css         # Komponenten-Styles
│   │   ├── forms.css              # Formular-Styles
│   │   ├── tables.css             # Tabellen-Styles
│   │   └── print.css              # Druck-Styles
│   │
│   ├── js/
│   │   ├── app.js                 # Haupt-JavaScript
│   │   ├── router.js              # Client-Side Router
│   │   ├── store.js               # State Management
│   │   ├── api.js                 # API Client
│   │   ├── auth.js                # Authentifizierung
│   │   ├── utils.js               # Hilfsfunktionen
│   │   ├── validators.js          # Formular-Validierung
│   │   └── charts.js              # Chart-Konfiguration
│   │
│   ├── svg/
│   │   ├── logo.svg
│   │   ├── icons/                 # Lucide Icons (selektiv)
│   │   └── illustrations/         # Illustrationen
│   │
│   └── fonts/
│       └── inter/                 # Inter Font Family
│
├── components/                     # Wiederverwendbare Komponenten
│   ├── layout/
│   │   ├── header.html
│   │   ├── sidebar.html
│   │   ├── footer.html
│   │   └── breadcrumb.html
│   │
│   ├── ui/
│   │   ├── button.html
│   │   ├── card.html
│   │   ├── modal.html
│   │   ├── dropdown.html
│   │   ├── tabs.html
│   │   ├── accordion.html
│   │   ├── toast.html
│   │   ├── tooltip.html
│   │   ├── badge.html
│   │   ├── avatar.html
│   │   ├── progress.html
│   │   └── spinner.html
│   │
│   ├── forms/
│   │   ├── input.html
│   │   ├── select.html
│   │   ├── checkbox.html
│   │   ├── radio.html
│   │   ├── textarea.html
│   │   ├── file-upload.html
│   │   ├── date-picker.html
│   │   ├── number-input.html
│   │   └── autocomplete.html
│   │
│   ├── data/
│   │   ├── table.html
│   │   ├── data-grid.html
│   │   ├── list.html
│   │   ├── tree.html
│   │   └── pagination.html
│   │
│   ├── charts/
│   │   ├── bar-chart.html
│   │   ├── line-chart.html
│   │   ├── pie-chart.html
│   │   ├── sankey.html
│   │   └── gauge.html
│   │
│   └── domain/                    # Fachspezifische Komponenten
│       ├── building-card.html
│       ├── zone-editor.html
│       ├── component-form.html
│       ├── calculation-result.html
│       ├── energy-label.html
│       └── co2-indicator.html
│
├── pages/                          # Seitenvorlagen
│   ├── auth/
│   │   ├── login.html
│   │   ├── register.html
│   │   ├── forgot-password.html
│   │   └── reset-password.html
│   │
│   ├── dashboard/
│   │   ├── index.html
│   │   ├── widgets/
│   │   │   ├── project-summary.html
│   │   │   ├── recent-activity.html
│   │   │   ├── kpi-cards.html
│   │   │   └── calendar.html
│   │   └── charts/
│   │
│   ├── projects/
│   │   ├── list.html
│   │   ├── detail.html
│   │   ├── create.html
│   │   └── settings.html
│   │
│   ├── buildings/
│   │   ├── list.html
│   │   ├── detail.html
│   │   ├── zones.html
│   │   ├── components.html
│   │   └── systems.html
│   │
│   ├── calculations/
│   │   ├── list.html
│   │   ├── u-value.html
│   │   ├── heat-load.html
│   │   ├── energy-balance.html
│   │   ├── renewable.html
│   │   └── results.html
│   │
│   ├── reports/
│   │   ├── list.html
│   │   ├── generator.html
│   │   ├── preview.html
│   │   └── templates.html
│   │
│   ├── business/
│   │   ├── customers/
│   │   ├── quotes/
│   │   ├── invoices/
│   │   └── time-tracking/
│   │
│   ├── settings/
│   │   ├── profile.html
│   │   ├── organization.html
│   │   ├── users.html
│   │   ├── integrations.html
│   │   └── preferences.html
│   │
│   └── admin/
│       ├── dashboard.html
│       ├── users.html
│       ├── organizations.html
│       └── system.html
│
├── templates/                      # Server-seitige Templates
│   ├── emails/
│   │   ├── welcome.html
│   │   ├── password-reset.html
│   │   └── invoice.html
│   │
│   └── reports/
│       ├── energy-certificate/
│       ├── audit-report/
│       └── heat-load/
│
└── service-worker.js              # PWA Service Worker
```

---

## 3. Design System

### 3.1 Farben

```css
/* design-system.css */
:root {
  /* Primärfarbe (Blau) */
  --color-primary-50: #eff6ff;
  --color-primary-100: #dbeafe;
  --color-primary-200: #bfdbfe;
  --color-primary-300: #93c5fd;
  --color-primary-400: #60a5fa;
  --color-primary-500: #3b82f6;
  --color-primary-600: #2563eb;
  --color-primary-700: #1d4ed8;
  --color-primary-800: #1e40af;
  --color-primary-900: #1e3a8a;

  /* Sekundärfarbe (Grün - Energie) */
  --color-secondary-50: #ecfdf5;
  --color-secondary-500: #10b981;
  --color-secondary-700: #047857;

  /* Neutralfarben */
  --color-gray-50: #f9fafb;
  --color-gray-100: #f3f4f6;
  --color-gray-200: #e5e7eb;
  --color-gray-300: #d1d5db;
  --color-gray-400: #9ca3af;
  --color-gray-500: #6b7280;
  --color-gray-600: #4b5563;
  --color-gray-700: #374151;
  --color-gray-800: #1f2937;
  --color-gray-900: #111827;

  /* Semantische Farben */
  --color-success: #10b981;
  --color-warning: #f59e0b;
  --color-error: #ef4444;
  --color-info: #3b82f6;

  /* Energieeffizienz-Skala */
  --color-efficiency-a-plus: #00a651;
  --color-efficiency-a: #4cb848;
  --color-efficiency-b: #bdd62e;
  --color-efficiency-c: #fff100;
  --color-efficiency-d: #ffb819;
  --color-efficiency-e: #f47920;
  --color-efficiency-f: #ed1c24;
  --color-efficiency-g: #be1622;
  --color-efficiency-h: #6d1f19;
}
```

### 3.2 Typografie

```css
:root {
  /* Font Families */
  --font-sans: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  --font-mono: 'JetBrains Mono', 'Fira Code', monospace;

  /* Font Sizes */
  --text-xs: 0.75rem;     /* 12px */
  --text-sm: 0.875rem;    /* 14px */
  --text-base: 1rem;      /* 16px */
  --text-lg: 1.125rem;    /* 18px */
  --text-xl: 1.25rem;     /* 20px */
  --text-2xl: 1.5rem;     /* 24px */
  --text-3xl: 1.875rem;   /* 30px */
  --text-4xl: 2.25rem;    /* 36px */

  /* Line Heights */
  --leading-tight: 1.25;
  --leading-normal: 1.5;
  --leading-relaxed: 1.75;

  /* Font Weights */
  --font-normal: 400;
  --font-medium: 500;
  --font-semibold: 600;
  --font-bold: 700;
}
```

### 3.3 Spacing & Layout

```css
:root {
  /* Spacing Scale */
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

  /* Border Radius */
  --radius-sm: 0.25rem;
  --radius-md: 0.375rem;
  --radius-lg: 0.5rem;
  --radius-xl: 0.75rem;
  --radius-2xl: 1rem;
  --radius-full: 9999px;

  /* Shadows */
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
  --shadow-xl: 0 20px 25px -5px rgba(0, 0, 0, 0.1);

  /* Layout */
  --sidebar-width: 280px;
  --header-height: 64px;
  --max-content-width: 1280px;
}
```

---

## 4. Alpine.js Integration

### 4.1 Store (State Management)

```javascript
// assets/js/store.js
document.addEventListener('alpine:init', () => {
  // Globaler App-Store
  Alpine.store('app', {
    user: null,
    organization: null,
    isLoading: false,
    notifications: [],

    init() {
      // Beim Start User laden
      this.loadUser();
    },

    async loadUser() {
      const token = localStorage.getItem('access_token');
      if (!token) return;

      try {
        const response = await api.get('/users/me');
        this.user = response.data;
        this.organization = response.data.organization;
      } catch (error) {
        this.logout();
      }
    },

    logout() {
      localStorage.removeItem('access_token');
      localStorage.removeItem('refresh_token');
      this.user = null;
      window.location.href = '/login.html';
    },

    addNotification(message, type = 'info') {
      const id = Date.now();
      this.notifications.push({ id, message, type });
      setTimeout(() => this.removeNotification(id), 5000);
    },

    removeNotification(id) {
      this.notifications = this.notifications.filter(n => n.id !== id);
    }
  });

  // Projekt-Store
  Alpine.store('projects', {
    items: [],
    current: null,
    filters: {
      status: 'all',
      search: ''
    },

    async load() {
      const response = await api.get('/projects', {
        params: this.filters
      });
      this.items = response.data;
    },

    async loadById(id) {
      const response = await api.get(`/projects/${id}`);
      this.current = response.data;
    }
  });

  // Berechnungs-Store
  Alpine.store('calculations', {
    current: null,
    results: null,
    isRunning: false,

    async run(calculationId) {
      this.isRunning = true;
      try {
        const response = await api.post(
          `/calculations/${calculationId}/run`
        );
        this.results = response.data;
      } finally {
        this.isRunning = false;
      }
    }
  });
});
```

### 4.2 Komponenten

```html
<!-- components/ui/modal.html -->
<template x-component="modal">
  <div
    x-show="open"
    x-transition:enter="transition ease-out duration-300"
    x-transition:enter-start="opacity-0"
    x-transition:enter-end="opacity-100"
    x-transition:leave="transition ease-in duration-200"
    x-transition:leave-start="opacity-100"
    x-transition:leave-end="opacity-0"
    class="fixed inset-0 z-50 overflow-y-auto"
    @keydown.escape.window="open = false"
  >
    <!-- Backdrop -->
    <div
      class="fixed inset-0 bg-black/50"
      @click="open = false"
    ></div>

    <!-- Modal -->
    <div class="flex min-h-full items-center justify-center p-4">
      <div
        x-show="open"
        x-transition:enter="transition ease-out duration-300"
        x-transition:enter-start="opacity-0 scale-95"
        x-transition:enter-end="opacity-100 scale-100"
        class="relative bg-white rounded-xl shadow-xl max-w-lg w-full"
        @click.stop
      >
        <!-- Header -->
        <div class="flex items-center justify-between p-4 border-b">
          <h3 class="text-lg font-semibold" x-text="title"></h3>
          <button @click="open = false" class="text-gray-400 hover:text-gray-600">
            <svg class="w-5 h-5">...</svg>
          </button>
        </div>

        <!-- Content -->
        <div class="p-4">
          <slot></slot>
        </div>

        <!-- Footer -->
        <div class="flex justify-end gap-3 p-4 border-t">
          <button
            @click="open = false"
            class="btn btn-secondary"
          >
            Abbrechen
          </button>
          <button
            @click="$dispatch('confirm')"
            class="btn btn-primary"
          >
            Bestätigen
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
```

### 4.3 Formular-Komponente

```html
<!-- components/forms/number-input.html -->
<div
  x-data="{
    value: '',
    min: null,
    max: null,
    step: 1,
    unit: '',
    error: '',

    validate() {
      if (this.min !== null && this.value < this.min) {
        this.error = `Minimum ist ${this.min}`;
        return false;
      }
      if (this.max !== null && this.value > this.max) {
        this.error = `Maximum ist ${this.max}`;
        return false;
      }
      this.error = '';
      return true;
    },

    increment() {
      this.value = Math.min(
        (parseFloat(this.value) || 0) + this.step,
        this.max || Infinity
      );
    },

    decrement() {
      this.value = Math.max(
        (parseFloat(this.value) || 0) - this.step,
        this.min || -Infinity
      );
    }
  }"
  class="form-group"
>
  <label :for="$id('input')" class="form-label">
    <span x-text="label"></span>
    <span x-show="required" class="text-red-500">*</span>
  </label>

  <div class="relative flex">
    <button
      @click="decrement()"
      class="px-3 bg-gray-100 border border-r-0 rounded-l-lg hover:bg-gray-200"
      type="button"
    >
      <svg class="w-4 h-4"><!-- minus icon --></svg>
    </button>

    <input
      :id="$id('input')"
      type="number"
      x-model="value"
      :min="min"
      :max="max"
      :step="step"
      @blur="validate()"
      class="form-input rounded-none text-center"
      :class="{ 'border-red-500': error }"
    />

    <button
      @click="increment()"
      class="px-3 bg-gray-100 border border-l-0 hover:bg-gray-200"
      type="button"
    >
      <svg class="w-4 h-4"><!-- plus icon --></svg>
    </button>

    <span
      x-show="unit"
      class="px-3 bg-gray-50 border border-l-0 rounded-r-lg flex items-center text-gray-500"
      x-text="unit"
    ></span>
  </div>

  <p x-show="error" x-text="error" class="mt-1 text-sm text-red-500"></p>
</div>
```

---

## 5. API Client

```javascript
// assets/js/api.js
const API_BASE = '/api/v1';

class ApiClient {
  constructor() {
    this.baseUrl = API_BASE;
  }

  getHeaders() {
    const headers = {
      'Content-Type': 'application/json',
    };

    const token = localStorage.getItem('access_token');
    if (token) {
      headers['Authorization'] = `Bearer ${token}`;
    }

    return headers;
  }

  async request(method, path, options = {}) {
    const url = `${this.baseUrl}${path}`;
    const config = {
      method,
      headers: this.getHeaders(),
      ...options
    };

    if (options.body) {
      config.body = JSON.stringify(options.body);
    }

    try {
      const response = await fetch(url, config);

      // Token abgelaufen?
      if (response.status === 401) {
        const refreshed = await this.refreshToken();
        if (refreshed) {
          return this.request(method, path, options);
        }
        Alpine.store('app').logout();
        throw new Error('Session abgelaufen');
      }

      const data = await response.json();

      if (!response.ok) {
        throw new ApiError(data.error);
      }

      return data;

    } catch (error) {
      console.error('API Error:', error);
      Alpine.store('app').addNotification(
        error.message || 'Ein Fehler ist aufgetreten',
        'error'
      );
      throw error;
    }
  }

  async refreshToken() {
    const refreshToken = localStorage.getItem('refresh_token');
    if (!refreshToken) return false;

    try {
      const response = await fetch(`${this.baseUrl}/auth/refresh`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ refresh_token: refreshToken })
      });

      if (response.ok) {
        const data = await response.json();
        localStorage.setItem('access_token', data.access_token);
        return true;
      }
    } catch (e) {
      // ignore
    }

    return false;
  }

  // Convenience Methods
  get(path, options) {
    return this.request('GET', path, options);
  }

  post(path, body, options) {
    return this.request('POST', path, { body, ...options });
  }

  put(path, body, options) {
    return this.request('PUT', path, { body, ...options });
  }

  patch(path, body, options) {
    return this.request('PATCH', path, { body, ...options });
  }

  delete(path, options) {
    return this.request('DELETE', path, options);
  }

  // Datei-Upload
  async upload(path, file, onProgress) {
    const formData = new FormData();
    formData.append('file', file);

    const xhr = new XMLHttpRequest();

    return new Promise((resolve, reject) => {
      xhr.upload.addEventListener('progress', (e) => {
        if (e.lengthComputable && onProgress) {
          onProgress(Math.round((e.loaded / e.total) * 100));
        }
      });

      xhr.addEventListener('load', () => {
        if (xhr.status >= 200 && xhr.status < 300) {
          resolve(JSON.parse(xhr.responseText));
        } else {
          reject(new Error('Upload fehlgeschlagen'));
        }
      });

      xhr.addEventListener('error', () => {
        reject(new Error('Netzwerkfehler'));
      });

      xhr.open('POST', `${this.baseUrl}${path}`);
      xhr.setRequestHeader('Authorization', `Bearer ${localStorage.getItem('access_token')}`);
      xhr.send(formData);
    });
  }
}

window.api = new ApiClient();
```

---

## 6. Router

```javascript
// assets/js/router.js
class Router {
  constructor() {
    this.routes = [];
    this.currentRoute = null;

    window.addEventListener('popstate', () => this.handleRoute());
    document.addEventListener('click', (e) => {
      if (e.target.matches('[data-link]')) {
        e.preventDefault();
        this.navigate(e.target.href);
      }
    });
  }

  addRoute(path, handler, options = {}) {
    this.routes.push({
      path: new RegExp(`^${path.replace(/:\w+/g, '([^/]+)')}$`),
      handler,
      ...options
    });
  }

  async handleRoute() {
    const path = window.location.pathname;

    for (const route of this.routes) {
      const match = path.match(route.path);
      if (match) {
        // Auth Check
        if (route.requiresAuth && !Alpine.store('app').user) {
          this.navigate('/login.html');
          return;
        }

        // Route Parameter extrahieren
        const params = match.slice(1);

        // Handler aufrufen
        await route.handler(params);
        this.currentRoute = route;
        return;
      }
    }

    // 404
    this.show404();
  }

  navigate(url) {
    history.pushState(null, '', url);
    this.handleRoute();
  }

  async loadPage(url) {
    const response = await fetch(url);
    const html = await response.text();

    const container = document.getElementById('app-content');
    container.innerHTML = html;

    // Alpine.js neu initialisieren für neuen Content
    Alpine.initTree(container);
  }

  show404() {
    document.getElementById('app-content').innerHTML = `
      <div class="flex flex-col items-center justify-center h-full">
        <h1 class="text-4xl font-bold text-gray-300">404</h1>
        <p class="text-gray-500 mt-2">Seite nicht gefunden</p>
        <a href="/" class="btn btn-primary mt-4">Zur Startseite</a>
      </div>
    `;
  }
}

// Router initialisieren
const router = new Router();

router.addRoute('/', () => router.loadPage('/pages/dashboard/index.html'), { requiresAuth: true });
router.addRoute('/projects', () => router.loadPage('/pages/projects/list.html'), { requiresAuth: true });
router.addRoute('/projects/:id', (params) => router.loadPage(`/pages/projects/detail.html?id=${params[0]}`), { requiresAuth: true });
router.addRoute('/buildings/:id', (params) => router.loadPage(`/pages/buildings/detail.html?id=${params[0]}`), { requiresAuth: true });
router.addRoute('/calculations/:type', (params) => router.loadPage(`/pages/calculations/${params[0]}.html`), { requiresAuth: true });
router.addRoute('/login', () => router.loadPage('/pages/auth/login.html'));

// Initial Route
document.addEventListener('DOMContentLoaded', () => router.handleRoute());
```

---

## 7. Fachspezifische Komponenten

### 7.1 Energieeffizienz-Label

```html
<!-- components/domain/energy-label.html -->
<div
  x-data="{
    value: 0,
    getClass() {
      if (this.value <= 30) return 'A+';
      if (this.value <= 50) return 'A';
      if (this.value <= 75) return 'B';
      if (this.value <= 100) return 'C';
      if (this.value <= 130) return 'D';
      if (this.value <= 160) return 'E';
      if (this.value <= 200) return 'F';
      if (this.value <= 250) return 'G';
      return 'H';
    },
    getColor() {
      const colors = {
        'A+': 'bg-efficiency-a-plus',
        'A': 'bg-efficiency-a',
        'B': 'bg-efficiency-b',
        'C': 'bg-efficiency-c',
        'D': 'bg-efficiency-d',
        'E': 'bg-efficiency-e',
        'F': 'bg-efficiency-f',
        'G': 'bg-efficiency-g',
        'H': 'bg-efficiency-h'
      };
      return colors[this.getClass()];
    }
  }"
  class="energy-label"
>
  <!-- Skala -->
  <div class="flex flex-col w-48">
    <template x-for="cls in ['A+', 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H']">
      <div
        class="h-6 flex items-center px-2 text-white text-sm font-medium"
        :class="[
          `bg-efficiency-${cls.toLowerCase().replace('+', '-plus')}`,
          cls === getClass() ? 'font-bold scale-105 shadow-lg z-10' : 'opacity-70'
        ]"
      >
        <span x-text="cls"></span>
      </div>
    </template>
  </div>

  <!-- Wert -->
  <div class="ml-4 flex flex-col items-center">
    <span class="text-4xl font-bold" x-text="value"></span>
    <span class="text-sm text-gray-500">kWh/(m²a)</span>
  </div>
</div>
```

### 7.2 U-Wert-Editor

```html
<!-- components/domain/u-value-editor.html -->
<div
  x-data="{
    layers: [],
    rsi: 0.13,
    rse: 0.04,

    addLayer() {
      this.layers.push({
        id: Date.now(),
        material: '',
        thickness: 0,
        lambda: 0.04
      });
    },

    removeLayer(id) {
      this.layers = this.layers.filter(l => l.id !== id);
    },

    calculateRValue() {
      const rLayers = this.layers.reduce((sum, layer) => {
        return sum + (layer.thickness / 1000) / layer.lambda;
      }, 0);
      return this.rsi + rLayers + this.rse;
    },

    calculateUValue() {
      return (1 / this.calculateRValue()).toFixed(3);
    },

    moveLayer(fromIndex, toIndex) {
      const layer = this.layers.splice(fromIndex, 1)[0];
      this.layers.splice(toIndex, 0, layer);
    }
  }"
  class="bg-white rounded-xl shadow-sm border p-6"
>
  <h3 class="text-lg font-semibold mb-4">Schichtaufbau</h3>

  <!-- Schichten Liste -->
  <div class="space-y-3">
    <template x-for="(layer, index) in layers" :key="layer.id">
      <div class="flex items-center gap-3 p-3 bg-gray-50 rounded-lg">
        <!-- Drag Handle -->
        <button class="cursor-move text-gray-400">
          <svg class="w-5 h-5"><!-- grip icon --></svg>
        </button>

        <!-- Material -->
        <select
          x-model="layer.material"
          @change="layer.lambda = materials[layer.material]?.lambda || 0.04"
          class="form-select flex-1"
        >
          <option value="">Material wählen...</option>
          <option value="concrete">Beton</option>
          <option value="brick">Mauerwerk</option>
          <option value="eps">EPS Dämmung</option>
          <option value="mineral_wool">Mineralwolle</option>
          <!-- weitere Materialien -->
        </select>

        <!-- Dicke -->
        <div class="w-24">
          <input
            type="number"
            x-model.number="layer.thickness"
            class="form-input"
            placeholder="mm"
          />
        </div>

        <!-- Lambda -->
        <div class="w-24">
          <input
            type="number"
            x-model.number="layer.lambda"
            step="0.001"
            class="form-input"
          />
        </div>

        <!-- Löschen -->
        <button
          @click="removeLayer(layer.id)"
          class="text-red-500 hover:text-red-700"
        >
          <svg class="w-5 h-5"><!-- trash icon --></svg>
        </button>
      </div>
    </template>
  </div>

  <!-- Schicht hinzufügen -->
  <button
    @click="addLayer()"
    class="mt-3 flex items-center gap-2 text-primary-600 hover:text-primary-700"
  >
    <svg class="w-5 h-5"><!-- plus icon --></svg>
    Schicht hinzufügen
  </button>

  <!-- Ergebnis -->
  <div class="mt-6 pt-6 border-t">
    <div class="grid grid-cols-2 gap-4">
      <div>
        <span class="text-sm text-gray-500">R-Wert</span>
        <p class="text-2xl font-semibold" x-text="calculateRValue().toFixed(3) + ' m²K/W'"></p>
      </div>
      <div>
        <span class="text-sm text-gray-500">U-Wert</span>
        <p class="text-2xl font-semibold text-primary-600" x-text="calculateUValue() + ' W/(m²K)'"></p>
      </div>
    </div>
  </div>
</div>
```

---

## 8. Charts & Visualisierungen

### 8.1 Energiebilanz-Chart

```javascript
// assets/js/charts/energy-balance.js
function createEnergyBalanceChart(container, data) {
  const width = container.clientWidth;
  const height = 400;

  // D3.js Sankey-Diagramm für Energieflüsse
  const sankey = d3.sankey()
    .nodeWidth(20)
    .nodePadding(10)
    .extent([[1, 1], [width - 1, height - 1]]);

  const { nodes, links } = sankey({
    nodes: data.nodes.map(d => Object.assign({}, d)),
    links: data.links.map(d => Object.assign({}, d))
  });

  const svg = d3.select(container)
    .append('svg')
    .attr('viewBox', [0, 0, width, height]);

  // Links (Flüsse)
  svg.append('g')
    .selectAll('path')
    .data(links)
    .join('path')
    .attr('d', d3.sankeyLinkHorizontal())
    .attr('stroke-width', d => Math.max(1, d.width))
    .attr('fill', 'none')
    .attr('stroke', d => d.color || '#3b82f6')
    .attr('stroke-opacity', 0.5);

  // Nodes (Kategorien)
  svg.append('g')
    .selectAll('rect')
    .data(nodes)
    .join('rect')
    .attr('x', d => d.x0)
    .attr('y', d => d.y0)
    .attr('height', d => d.y1 - d.y0)
    .attr('width', d => d.x1 - d.x0)
    .attr('fill', d => d.color || '#1f2937');

  // Labels
  svg.append('g')
    .selectAll('text')
    .data(nodes)
    .join('text')
    .attr('x', d => d.x0 < width / 2 ? d.x1 + 6 : d.x0 - 6)
    .attr('y', d => (d.y1 + d.y0) / 2)
    .attr('dy', '0.35em')
    .attr('text-anchor', d => d.x0 < width / 2 ? 'start' : 'end')
    .text(d => `${d.name}: ${d.value} kWh`);
}
```

---

*Letzte Aktualisierung: 2025-11-25*
*Version: 0.1.0*
