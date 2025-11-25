# PWA und Offline-Konzept EnergieberaterPRO

## Referenzen
- [Service Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)
- [Workbox](https://developer.chrome.com/docs/workbox/)
- [IndexedDB API](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
- [Cache API](https://developer.mozilla.org/en-US/docs/Web/API/Cache)
- [Background Sync API](https://developer.mozilla.org/en-US/docs/Web/API/Background_Synchronization_API)

---

## 1. Offline-First Philosophie

### 1.1 Kernprinzip

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       OFFLINE-FIRST ARCHITEKTUR                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  GRUNDSATZ: Die Anwendung funktioniert IMMER - mit oder ohne Internet       │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                         ANWENDUNGSFALL                                │  │
│  ├───────────────────────────────────────────────────────────────────────┤  │
│  │                                                                       │  │
│  │  Energieberater vor Ort beim Kunden:                                  │  │
│  │  • Kein WLAN verfügbar                                                │  │
│  │  • Mobilfunk-Empfang schlecht (Keller, Dachboden)                    │  │
│  │  • Muss trotzdem:                                                     │  │
│  │    - Gebäudedaten erfassen                                           │  │
│  │    - Berechnungen durchführen                                        │  │
│  │    - Fotos dokumentieren                                             │  │
│  │    - Notizen machen                                                  │  │
│  │                                                                       │  │
│  │  → ALLES muss offline funktionieren!                                  │  │
│  │                                                                       │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  STRATEGIE:                                                                 │
│  ─────────────────────────────────────────────────────────────────────────  │
│  1. Cache-First für statische Ressourcen                                   │
│  2. Network-First mit Offline-Fallback für API-Daten                       │
│  3. IndexedDB für lokale Datenspeicherung                                  │
│  4. Background Sync für verzögerte Synchronisation                         │
│  5. Konfliktauflösung bei gleichzeitiger Bearbeitung                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Caching-Strategien

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        CACHING-STRATEGIEN                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  CACHE-FIRST (Statische Assets)                                             │
│  ───────────────────────────────                                            │
│  Request → Cache → [Hit: Return] / [Miss: Network → Cache → Return]        │
│                                                                             │
│  Verwendet für:                                                             │
│  • HTML-Templates                                                           │
│  • CSS, JavaScript                                                          │
│  • Bilder, Icons, Fonts                                                     │
│  • Berechnungs-Bibliotheken                                                 │
│  • Normwerte-Datenbanken                                                    │
│                                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│                                                                             │
│  NETWORK-FIRST (Dynamische Daten)                                           │
│  ─────────────────────────────────                                          │
│  Request → Network → [Success: Cache + Return] / [Fail: Cache → Return]    │
│                                                                             │
│  Verwendet für:                                                             │
│  • Projektdaten                                                             │
│  • Kundendaten                                                              │
│  • Berechnungsergebnisse                                                    │
│  • Benutzereinstellungen                                                    │
│                                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│                                                                             │
│  STALE-WHILE-REVALIDATE (Semi-statisch)                                     │
│  ─────────────────────────────────────────                                  │
│  Request → Cache → Return → Network → Update Cache                          │
│                                                                             │
│  Verwendet für:                                                             │
│  • Projektlisten                                                            │
│  • Dashboard-Daten                                                          │
│  • Benutzer-Profile                                                         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Service Worker

### 2.1 Service Worker Implementierung

```javascript
// static/sw.js
/**
 * Service Worker für EnergieberaterPRO
 * Ermöglicht vollständige Offline-Funktionalität
 */

const CACHE_VERSION = 'v1.0.0';
const STATIC_CACHE = `energieberater-static-${CACHE_VERSION}`;
const DYNAMIC_CACHE = `energieberater-dynamic-${CACHE_VERSION}`;
const DATA_CACHE = `energieberater-data-${CACHE_VERSION}`;

// Statische Assets zum Pre-Caching
const STATIC_ASSETS = [
    '/',
    '/index.html',
    '/offline.html',
    '/manifest.json',

    // CSS
    '/static/css/main.css',
    '/static/css/components.css',
    '/static/css/print.css',

    // JavaScript
    '/static/js/app.js',
    '/static/js/alpine.min.js',
    '/static/js/calculations.js',
    '/static/js/offline-manager.js',
    '/static/js/sync-manager.js',
    '/static/js/idb-wrapper.js',

    // Berechnungs-Bibliotheken (kritisch für Offline!)
    '/static/js/lib/thermal-calculations.js',
    '/static/js/lib/energy-balance.js',
    '/static/js/lib/heating-load.js',
    '/static/js/lib/u-value-calculator.js',

    // Normwert-Datenbanken
    '/static/data/climate-zones.json',
    '/static/data/building-materials.json',
    '/static/data/usage-profiles.json',
    '/static/data/energy-factors.json',

    // Icons und Bilder
    '/static/images/logo.svg',
    '/static/images/icons/icon-192.png',
    '/static/images/icons/icon-512.png',

    // Fonts
    '/static/fonts/inter-var.woff2',
];

// API-Routen die gecacht werden sollen
const CACHEABLE_API_ROUTES = [
    '/api/v1/projects',
    '/api/v1/customers',
    '/api/v1/buildings',
    '/api/v1/calculations',
    '/api/v1/users/me',
    '/api/v1/settings',
];

// ============================================================================
// INSTALLATION
// ============================================================================

self.addEventListener('install', (event) => {
    console.log('[SW] Installing Service Worker...');

    event.waitUntil(
        caches.open(STATIC_CACHE)
            .then((cache) => {
                console.log('[SW] Pre-caching static assets');
                return cache.addAll(STATIC_ASSETS);
            })
            .then(() => {
                console.log('[SW] Static assets cached');
                return self.skipWaiting();
            })
            .catch((error) => {
                console.error('[SW] Pre-cache failed:', error);
            })
    );
});

// ============================================================================
// AKTIVIERUNG
// ============================================================================

self.addEventListener('activate', (event) => {
    console.log('[SW] Activating Service Worker...');

    event.waitUntil(
        Promise.all([
            // Alte Caches löschen
            caches.keys().then((cacheNames) => {
                return Promise.all(
                    cacheNames
                        .filter((name) => {
                            return name.startsWith('energieberater-') &&
                                   name !== STATIC_CACHE &&
                                   name !== DYNAMIC_CACHE &&
                                   name !== DATA_CACHE;
                        })
                        .map((name) => {
                            console.log('[SW] Deleting old cache:', name);
                            return caches.delete(name);
                        })
                );
            }),
            // Sofort alle Clients übernehmen
            self.clients.claim()
        ])
    );
});

// ============================================================================
// FETCH HANDLER
// ============================================================================

self.addEventListener('fetch', (event) => {
    const { request } = event;
    const url = new URL(request.url);

    // Nur gleicher Origin
    if (url.origin !== self.location.origin) {
        return;
    }

    // API-Requests
    if (url.pathname.startsWith('/api/')) {
        event.respondWith(handleApiRequest(request));
        return;
    }

    // Statische Assets
    if (isStaticAsset(url.pathname)) {
        event.respondWith(handleStaticRequest(request));
        return;
    }

    // HTML-Seiten (Navigation)
    if (request.mode === 'navigate') {
        event.respondWith(handleNavigationRequest(request));
        return;
    }

    // Fallback: Network-First
    event.respondWith(handleDynamicRequest(request));
});

// ============================================================================
// REQUEST HANDLER
// ============================================================================

/**
 * Cache-First für statische Assets
 */
async function handleStaticRequest(request) {
    const cached = await caches.match(request);
    if (cached) {
        return cached;
    }

    try {
        const response = await fetch(request);
        if (response.ok) {
            const cache = await caches.open(STATIC_CACHE);
            cache.put(request, response.clone());
        }
        return response;
    } catch (error) {
        console.error('[SW] Static fetch failed:', error);
        // Fallback für Bilder
        if (request.destination === 'image') {
            return caches.match('/static/images/placeholder.svg');
        }
        throw error;
    }
}

/**
 * Network-First für API-Requests mit Offline-Fallback
 */
async function handleApiRequest(request) {
    const url = new URL(request.url);

    // Nur GET-Requests cachen
    if (request.method !== 'GET') {
        return handleMutationRequest(request);
    }

    try {
        const response = await fetch(request);

        if (response.ok && isCacheableApiRoute(url.pathname)) {
            const cache = await caches.open(DATA_CACHE);
            cache.put(request, response.clone());

            // Auch in IndexedDB für komplexere Abfragen
            const data = await response.clone().json();
            await storeInIndexedDB(url.pathname, data);
        }

        return response;
    } catch (error) {
        console.log('[SW] Network failed, trying cache:', url.pathname);

        // Aus Cache laden
        const cached = await caches.match(request);
        if (cached) {
            // Header hinzufügen um Offline-Status anzuzeigen
            const headers = new Headers(cached.headers);
            headers.set('X-Offline-Cache', 'true');
            headers.set('X-Cache-Date', cached.headers.get('date') || 'unknown');

            return new Response(cached.body, {
                status: cached.status,
                statusText: cached.statusText,
                headers: headers
            });
        }

        // Fallback: Leere Antwort mit Offline-Status
        return new Response(JSON.stringify({
            error: 'offline',
            message: 'Keine Internetverbindung. Daten nicht verfügbar.',
            cached: false
        }), {
            status: 503,
            headers: {
                'Content-Type': 'application/json',
                'X-Offline': 'true'
            }
        });
    }
}

/**
 * Mutation-Requests (POST, PUT, DELETE) für Background Sync
 */
async function handleMutationRequest(request) {
    try {
        const response = await fetch(request);
        return response;
    } catch (error) {
        // Offline: Request für Background Sync speichern
        console.log('[SW] Mutation failed, queuing for sync');

        const requestData = {
            url: request.url,
            method: request.method,
            headers: Object.fromEntries(request.headers.entries()),
            body: await request.clone().text(),
            timestamp: Date.now()
        };

        await queueForSync(requestData);

        // Optimistische Antwort
        return new Response(JSON.stringify({
            status: 'queued',
            message: 'Änderung wird synchronisiert sobald online.',
            offline: true,
            syncId: requestData.timestamp
        }), {
            status: 202,
            headers: {
                'Content-Type': 'application/json',
                'X-Offline-Queued': 'true'
            }
        });
    }
}

/**
 * Navigation-Requests mit Offline-Fallback
 */
async function handleNavigationRequest(request) {
    try {
        const response = await fetch(request);
        if (response.ok) {
            const cache = await caches.open(DYNAMIC_CACHE);
            cache.put(request, response.clone());
        }
        return response;
    } catch (error) {
        // Versuche gecachte Version
        const cached = await caches.match(request);
        if (cached) {
            return cached;
        }

        // Offline-Seite
        return caches.match('/offline.html');
    }
}

/**
 * Dynamische Requests mit Stale-While-Revalidate
 */
async function handleDynamicRequest(request) {
    const cached = await caches.match(request);

    const networkPromise = fetch(request)
        .then((response) => {
            if (response.ok) {
                const cache = caches.open(DYNAMIC_CACHE);
                cache.then((c) => c.put(request, response.clone()));
            }
            return response;
        })
        .catch(() => null);

    // Cached Version sofort zurückgeben, im Hintergrund aktualisieren
    if (cached) {
        networkPromise; // Fire and forget
        return cached;
    }

    const networkResponse = await networkPromise;
    if (networkResponse) {
        return networkResponse;
    }

    throw new Error('No cached version and network failed');
}

// ============================================================================
// BACKGROUND SYNC
// ============================================================================

self.addEventListener('sync', (event) => {
    console.log('[SW] Background sync triggered:', event.tag);

    if (event.tag === 'sync-mutations') {
        event.waitUntil(processSyncQueue());
    }
});

async function processSyncQueue() {
    const queue = await getSyncQueue();

    for (const item of queue) {
        try {
            const response = await fetch(item.url, {
                method: item.method,
                headers: item.headers,
                body: item.body
            });

            if (response.ok) {
                await removeFromSyncQueue(item.timestamp);
                // Client benachrichtigen
                await notifyClients({
                    type: 'sync-success',
                    syncId: item.timestamp,
                    url: item.url
                });
            }
        } catch (error) {
            console.error('[SW] Sync failed for item:', item.timestamp);
            // Wird beim nächsten Sync erneut versucht
        }
    }
}

// ============================================================================
// INDEXEDDB HELPERS
// ============================================================================

const DB_NAME = 'energieberater-offline';
const DB_VERSION = 1;

function openDatabase() {
    return new Promise((resolve, reject) => {
        const request = indexedDB.open(DB_NAME, DB_VERSION);

        request.onerror = () => reject(request.error);
        request.onsuccess = () => resolve(request.result);

        request.onupgradeneeded = (event) => {
            const db = event.target.result;

            // Sync Queue
            if (!db.objectStoreNames.contains('sync-queue')) {
                db.createObjectStore('sync-queue', { keyPath: 'timestamp' });
            }

            // Cached API Data
            if (!db.objectStoreNames.contains('api-cache')) {
                const store = db.createObjectStore('api-cache', { keyPath: 'url' });
                store.createIndex('timestamp', 'timestamp');
            }

            // Offline Edits (nicht synchronisierte Änderungen)
            if (!db.objectStoreNames.contains('offline-edits')) {
                const store = db.createObjectStore('offline-edits', { keyPath: 'id', autoIncrement: true });
                store.createIndex('entity', ['entityType', 'entityId']);
                store.createIndex('timestamp', 'timestamp');
            }
        };
    });
}

async function storeInIndexedDB(url, data) {
    const db = await openDatabase();
    const tx = db.transaction('api-cache', 'readwrite');
    const store = tx.objectStore('api-cache');

    await store.put({
        url: url,
        data: data,
        timestamp: Date.now()
    });
}

async function queueForSync(requestData) {
    const db = await openDatabase();
    const tx = db.transaction('sync-queue', 'readwrite');
    const store = tx.objectStore('sync-queue');

    await store.add(requestData);

    // Background Sync registrieren
    if ('sync' in self.registration) {
        await self.registration.sync.register('sync-mutations');
    }
}

async function getSyncQueue() {
    const db = await openDatabase();
    const tx = db.transaction('sync-queue', 'readonly');
    const store = tx.objectStore('sync-queue');

    return new Promise((resolve, reject) => {
        const request = store.getAll();
        request.onsuccess = () => resolve(request.result);
        request.onerror = () => reject(request.error);
    });
}

async function removeFromSyncQueue(timestamp) {
    const db = await openDatabase();
    const tx = db.transaction('sync-queue', 'readwrite');
    const store = tx.objectStore('sync-queue');

    await store.delete(timestamp);
}

// ============================================================================
// HELPER FUNCTIONS
// ============================================================================

function isStaticAsset(pathname) {
    const staticExtensions = ['.css', '.js', '.png', '.jpg', '.jpeg', '.gif', '.svg', '.woff', '.woff2', '.json'];
    return staticExtensions.some(ext => pathname.endsWith(ext)) ||
           pathname.startsWith('/static/');
}

function isCacheableApiRoute(pathname) {
    return CACHEABLE_API_ROUTES.some(route => pathname.startsWith(route));
}

async function notifyClients(message) {
    const clients = await self.clients.matchAll();
    for (const client of clients) {
        client.postMessage(message);
    }
}

// ============================================================================
// PUSH NOTIFICATIONS
// ============================================================================

self.addEventListener('push', (event) => {
    const data = event.data?.json() || {};

    const options = {
        body: data.body || 'Neue Benachrichtigung',
        icon: '/static/images/icons/icon-192.png',
        badge: '/static/images/icons/badge-72.png',
        vibrate: [100, 50, 100],
        data: data.data || {},
        actions: data.actions || []
    };

    event.waitUntil(
        self.registration.showNotification(data.title || 'EnergieberaterPRO', options)
    );
});

self.addEventListener('notificationclick', (event) => {
    event.notification.close();

    const data = event.notification.data;

    if (data.url) {
        event.waitUntil(
            clients.openWindow(data.url)
        );
    }
});
```

### 2.2 Service Worker Registration

```javascript
// static/js/sw-register.js
/**
 * Service Worker Registration und Update-Handling
 */

class ServiceWorkerManager {
    constructor() {
        this.registration = null;
        this.updateAvailable = false;
    }

    async register() {
        if (!('serviceWorker' in navigator)) {
            console.warn('Service Worker nicht unterstützt');
            return false;
        }

        try {
            this.registration = await navigator.serviceWorker.register('/sw.js', {
                scope: '/'
            });

            console.log('Service Worker registriert:', this.registration.scope);

            // Update-Listener
            this.registration.addEventListener('updatefound', () => {
                this.handleUpdateFound();
            });

            // Auf Messages vom SW hören
            navigator.serviceWorker.addEventListener('message', (event) => {
                this.handleMessage(event.data);
            });

            // Periodische Update-Checks
            setInterval(() => {
                this.registration.update();
            }, 60 * 60 * 1000); // Stündlich

            return true;
        } catch (error) {
            console.error('Service Worker Registration fehlgeschlagen:', error);
            return false;
        }
    }

    handleUpdateFound() {
        const newWorker = this.registration.installing;

        newWorker.addEventListener('statechange', () => {
            if (newWorker.state === 'installed' && navigator.serviceWorker.controller) {
                // Neue Version verfügbar
                this.updateAvailable = true;
                this.showUpdateNotification();
            }
        });
    }

    showUpdateNotification() {
        // Event für UI
        window.dispatchEvent(new CustomEvent('sw-update-available', {
            detail: {
                update: () => this.applyUpdate()
            }
        }));
    }

    applyUpdate() {
        if (!this.registration.waiting) {
            return;
        }

        // Signal an neuen SW, die Kontrolle zu übernehmen
        this.registration.waiting.postMessage({ type: 'SKIP_WAITING' });

        // Seite neu laden wenn SW aktiv wird
        navigator.serviceWorker.addEventListener('controllerchange', () => {
            window.location.reload();
        });
    }

    handleMessage(message) {
        switch (message.type) {
            case 'sync-success':
                window.dispatchEvent(new CustomEvent('sync-completed', {
                    detail: message
                }));
                break;

            case 'sync-failed':
                window.dispatchEvent(new CustomEvent('sync-failed', {
                    detail: message
                }));
                break;

            case 'cache-updated':
                window.dispatchEvent(new CustomEvent('cache-updated', {
                    detail: message
                }));
                break;
        }
    }

    async triggerSync() {
        if (this.registration && 'sync' in this.registration) {
            await this.registration.sync.register('sync-mutations');
        }
    }

    async clearCaches() {
        const cacheNames = await caches.keys();
        await Promise.all(
            cacheNames
                .filter(name => name.startsWith('energieberater-'))
                .map(name => caches.delete(name))
        );
    }
}

// Initialisierung
const swManager = new ServiceWorkerManager();

document.addEventListener('DOMContentLoaded', () => {
    swManager.register();
});

// Export für andere Module
window.swManager = swManager;
```

---

## 3. Offline-Datenmanagement

### 3.1 IndexedDB Wrapper

```javascript
// static/js/idb-wrapper.js
/**
 * IndexedDB Wrapper für Offline-Datenspeicherung
 */

class OfflineDatabase {
    constructor() {
        this.dbName = 'energieberater-data';
        this.version = 1;
        this.db = null;
    }

    async open() {
        if (this.db) return this.db;

        return new Promise((resolve, reject) => {
            const request = indexedDB.open(this.dbName, this.version);

            request.onerror = () => reject(request.error);
            request.onsuccess = () => {
                this.db = request.result;
                resolve(this.db);
            };

            request.onupgradeneeded = (event) => {
                const db = event.target.result;
                this.createStores(db);
            };
        });
    }

    createStores(db) {
        // Projekte
        if (!db.objectStoreNames.contains('projects')) {
            const store = db.createObjectStore('projects', { keyPath: 'id' });
            store.createIndex('organization', 'organization_id');
            store.createIndex('status', 'status');
            store.createIndex('updated', 'updated_at');
        }

        // Gebäude
        if (!db.objectStoreNames.contains('buildings')) {
            const store = db.createObjectStore('buildings', { keyPath: 'id' });
            store.createIndex('project', 'project_id');
        }

        // Berechnungen
        if (!db.objectStoreNames.contains('calculations')) {
            const store = db.createObjectStore('calculations', { keyPath: 'id' });
            store.createIndex('project', 'project_id');
            store.createIndex('building', 'building_id');
        }

        // Kunden
        if (!db.objectStoreNames.contains('customers')) {
            const store = db.createObjectStore('customers', { keyPath: 'id' });
            store.createIndex('organization', 'organization_id');
        }

        // Offline-Änderungen (Pending Sync)
        if (!db.objectStoreNames.contains('pendingChanges')) {
            const store = db.createObjectStore('pendingChanges', { keyPath: 'id', autoIncrement: true });
            store.createIndex('entity', ['entityType', 'entityId']);
            store.createIndex('timestamp', 'timestamp');
        }

        // Medien/Fotos
        if (!db.objectStoreNames.contains('media')) {
            const store = db.createObjectStore('media', { keyPath: 'id' });
            store.createIndex('project', 'project_id');
            store.createIndex('synced', 'is_synced');
        }

        // Metadaten (letzte Sync-Zeit, etc.)
        if (!db.objectStoreNames.contains('metadata')) {
            db.createObjectStore('metadata', { keyPath: 'key' });
        }
    }

    // ========================================================================
    // CRUD Operations
    // ========================================================================

    async getAll(storeName, indexName = null, query = null) {
        const db = await this.open();
        const tx = db.transaction(storeName, 'readonly');
        const store = tx.objectStore(storeName);

        return new Promise((resolve, reject) => {
            let request;

            if (indexName && query !== null) {
                const index = store.index(indexName);
                request = index.getAll(query);
            } else {
                request = store.getAll();
            }

            request.onsuccess = () => resolve(request.result);
            request.onerror = () => reject(request.error);
        });
    }

    async get(storeName, id) {
        const db = await this.open();
        const tx = db.transaction(storeName, 'readonly');
        const store = tx.objectStore(storeName);

        return new Promise((resolve, reject) => {
            const request = store.get(id);
            request.onsuccess = () => resolve(request.result);
            request.onerror = () => reject(request.error);
        });
    }

    async put(storeName, data) {
        const db = await this.open();
        const tx = db.transaction(storeName, 'readwrite');
        const store = tx.objectStore(storeName);

        return new Promise((resolve, reject) => {
            const request = store.put(data);
            request.onsuccess = () => resolve(request.result);
            request.onerror = () => reject(request.error);
        });
    }

    async delete(storeName, id) {
        const db = await this.open();
        const tx = db.transaction(storeName, 'readwrite');
        const store = tx.objectStore(storeName);

        return new Promise((resolve, reject) => {
            const request = store.delete(id);
            request.onsuccess = () => resolve();
            request.onerror = () => reject(request.error);
        });
    }

    async clear(storeName) {
        const db = await this.open();
        const tx = db.transaction(storeName, 'readwrite');
        const store = tx.objectStore(storeName);

        return new Promise((resolve, reject) => {
            const request = store.clear();
            request.onsuccess = () => resolve();
            request.onerror = () => reject(request.error);
        });
    }

    // ========================================================================
    // Bulk Operations
    // ========================================================================

    async bulkPut(storeName, items) {
        const db = await this.open();
        const tx = db.transaction(storeName, 'readwrite');
        const store = tx.objectStore(storeName);

        const promises = items.map(item => {
            return new Promise((resolve, reject) => {
                const request = store.put(item);
                request.onsuccess = () => resolve();
                request.onerror = () => reject(request.error);
            });
        });

        await Promise.all(promises);

        return new Promise((resolve, reject) => {
            tx.oncomplete = () => resolve();
            tx.onerror = () => reject(tx.error);
        });
    }

    // ========================================================================
    // Pending Changes Management
    // ========================================================================

    async addPendingChange(entityType, entityId, operation, data) {
        return this.put('pendingChanges', {
            entityType,
            entityId,
            operation, // 'create', 'update', 'delete'
            data,
            timestamp: Date.now(),
            retryCount: 0
        });
    }

    async getPendingChanges() {
        return this.getAll('pendingChanges');
    }

    async removePendingChange(id) {
        return this.delete('pendingChanges', id);
    }

    async hasPendingChanges() {
        const changes = await this.getPendingChanges();
        return changes.length > 0;
    }

    // ========================================================================
    // Metadata
    // ========================================================================

    async setMetadata(key, value) {
        return this.put('metadata', { key, value, updated: Date.now() });
    }

    async getMetadata(key) {
        const result = await this.get('metadata', key);
        return result?.value;
    }

    async getLastSyncTime(entityType) {
        return this.getMetadata(`lastSync_${entityType}`);
    }

    async setLastSyncTime(entityType, timestamp = Date.now()) {
        return this.setMetadata(`lastSync_${entityType}`, timestamp);
    }
}

// Singleton-Instanz
const offlineDB = new OfflineDatabase();
export default offlineDB;
```

### 3.2 Sync Manager

```javascript
// static/js/sync-manager.js
/**
 * Synchronisations-Manager für Offline/Online-Wechsel
 */

import offlineDB from './idb-wrapper.js';

class SyncManager {
    constructor() {
        this.isSyncing = false;
        this.syncQueue = [];
        this.conflictResolver = new ConflictResolver();

        this.init();
    }

    init() {
        // Online/Offline Events
        window.addEventListener('online', () => this.handleOnline());
        window.addEventListener('offline', () => this.handleOffline());

        // Sync-Events vom Service Worker
        window.addEventListener('sync-completed', (e) => this.handleSyncCompleted(e.detail));

        // Periodische Sync-Prüfung
        setInterval(() => this.checkAndSync(), 30000); // Alle 30 Sekunden
    }

    async handleOnline() {
        console.log('[Sync] Online - Starting sync...');
        window.dispatchEvent(new CustomEvent('connection-status', { detail: { online: true } }));

        await this.syncAll();
    }

    handleOffline() {
        console.log('[Sync] Offline mode activated');
        window.dispatchEvent(new CustomEvent('connection-status', { detail: { online: false } }));
    }

    async checkAndSync() {
        if (!navigator.onLine || this.isSyncing) return;

        const hasPending = await offlineDB.hasPendingChanges();
        if (hasPending) {
            await this.syncPendingChanges();
        }
    }

    // ========================================================================
    // Full Sync
    // ========================================================================

    async syncAll() {
        if (this.isSyncing) return;
        this.isSyncing = true;

        try {
            // 1. Pending Changes hochladen
            await this.syncPendingChanges();

            // 2. Neue Daten herunterladen
            await this.downloadUpdates();

            // 3. Metadaten aktualisieren
            await offlineDB.setMetadata('lastFullSync', Date.now());

            window.dispatchEvent(new CustomEvent('sync-complete', {
                detail: { success: true }
            }));
        } catch (error) {
            console.error('[Sync] Full sync failed:', error);
            window.dispatchEvent(new CustomEvent('sync-complete', {
                detail: { success: false, error: error.message }
            }));
        } finally {
            this.isSyncing = false;
        }
    }

    // ========================================================================
    // Upload Pending Changes
    // ========================================================================

    async syncPendingChanges() {
        const pendingChanges = await offlineDB.getPendingChanges();

        if (pendingChanges.length === 0) {
            console.log('[Sync] No pending changes');
            return;
        }

        console.log(`[Sync] Syncing ${pendingChanges.length} pending changes...`);

        // Nach Timestamp sortieren
        pendingChanges.sort((a, b) => a.timestamp - b.timestamp);

        for (const change of pendingChanges) {
            try {
                await this.processPendingChange(change);
                await offlineDB.removePendingChange(change.id);
            } catch (error) {
                console.error('[Sync] Failed to sync change:', change.id, error);

                // Konflikt-Handling
                if (error.status === 409) {
                    await this.handleConflict(change, error);
                } else {
                    // Retry-Counter erhöhen
                    change.retryCount++;
                    if (change.retryCount < 3) {
                        await offlineDB.put('pendingChanges', change);
                    } else {
                        // Nach 3 Versuchen: User benachrichtigen
                        window.dispatchEvent(new CustomEvent('sync-error', {
                            detail: { change, error }
                        }));
                    }
                }
            }
        }
    }

    async processPendingChange(change) {
        const endpoints = {
            'projects': '/api/v1/projects',
            'buildings': '/api/v1/buildings',
            'calculations': '/api/v1/calculations',
            'customers': '/api/v1/customers',
        };

        const baseUrl = endpoints[change.entityType];
        if (!baseUrl) {
            throw new Error(`Unknown entity type: ${change.entityType}`);
        }

        let url = baseUrl;
        let method = 'POST';

        switch (change.operation) {
            case 'create':
                method = 'POST';
                break;
            case 'update':
                url = `${baseUrl}/${change.entityId}`;
                method = 'PUT';
                break;
            case 'delete':
                url = `${baseUrl}/${change.entityId}`;
                method = 'DELETE';
                break;
        }

        const response = await fetch(url, {
            method,
            headers: {
                'Content-Type': 'application/json',
                'X-Offline-Sync': 'true',
                'X-Offline-Timestamp': change.timestamp.toString()
            },
            body: change.operation !== 'delete' ? JSON.stringify(change.data) : undefined
        });

        if (!response.ok) {
            const error = new Error(`Sync failed: ${response.status}`);
            error.status = response.status;
            error.response = await response.json();
            throw error;
        }

        return response.json();
    }

    // ========================================================================
    // Download Updates
    // ========================================================================

    async downloadUpdates() {
        const entityTypes = ['projects', 'buildings', 'calculations', 'customers'];

        for (const entityType of entityTypes) {
            await this.downloadEntityUpdates(entityType);
        }
    }

    async downloadEntityUpdates(entityType) {
        const lastSync = await offlineDB.getLastSyncTime(entityType) || 0;

        const response = await fetch(
            `/api/v1/${entityType}?updated_since=${new Date(lastSync).toISOString()}`,
            {
                headers: {
                    'X-Sync-Request': 'true'
                }
            }
        );

        if (!response.ok) {
            throw new Error(`Failed to download ${entityType}: ${response.status}`);
        }

        const data = await response.json();

        if (data.items && data.items.length > 0) {
            await offlineDB.bulkPut(entityType, data.items);
            console.log(`[Sync] Downloaded ${data.items.length} ${entityType}`);
        }

        await offlineDB.setLastSyncTime(entityType);
    }

    // ========================================================================
    // Conflict Resolution
    // ========================================================================

    async handleConflict(localChange, error) {
        const serverVersion = error.response?.serverVersion;

        const resolution = await this.conflictResolver.resolve(
            localChange.entityType,
            localChange.data,
            serverVersion
        );

        switch (resolution.strategy) {
            case 'keep-local':
                // Force-Update mit lokaler Version
                localChange.data._forceUpdate = true;
                await this.processPendingChange(localChange);
                break;

            case 'keep-server':
                // Server-Version übernehmen
                await offlineDB.put(localChange.entityType, serverVersion);
                break;

            case 'merge':
                // Zusammenführen
                const merged = resolution.mergedData;
                localChange.data = merged;
                await this.processPendingChange(localChange);
                break;

            case 'manual':
                // Benutzer entscheiden lassen
                window.dispatchEvent(new CustomEvent('sync-conflict', {
                    detail: {
                        localData: localChange.data,
                        serverData: serverVersion,
                        resolve: async (chosenData) => {
                            localChange.data = chosenData;
                            await this.processPendingChange(localChange);
                            await offlineDB.removePendingChange(localChange.id);
                        }
                    }
                }));
                break;
        }
    }
}


/**
 * Konfliktauflösung
 */
class ConflictResolver {
    async resolve(entityType, localData, serverData) {
        // Automatische Strategien basierend auf Änderungstyp

        // Gleiche Daten = kein Konflikt
        if (JSON.stringify(localData) === JSON.stringify(serverData)) {
            return { strategy: 'keep-server' };
        }

        // Unterschiedliche Felder geändert = Merge möglich
        const localChangedFields = this.getChangedFields(localData);
        const serverChangedFields = this.getChangedFields(serverData);

        const conflictingFields = localChangedFields.filter(f => serverChangedFields.includes(f));

        if (conflictingFields.length === 0) {
            // Kein Feldkonflikt - automatisch mergen
            return {
                strategy: 'merge',
                mergedData: { ...serverData, ...localData }
            };
        }

        // Bei kritischen Feldern: manuelle Auflösung
        const criticalFields = ['status', 'owner_id', 'calculation_results'];
        const hasCriticalConflict = conflictingFields.some(f => criticalFields.includes(f));

        if (hasCriticalConflict) {
            return { strategy: 'manual' };
        }

        // Standard: Neuere Version gewinnt
        if (localData.updated_at > serverData.updated_at) {
            return { strategy: 'keep-local' };
        } else {
            return { strategy: 'keep-server' };
        }
    }

    getChangedFields(data) {
        // Vereinfacht: Alle nicht-null Felder
        return Object.keys(data).filter(k => data[k] != null && !k.startsWith('_'));
    }
}

// Singleton-Instanz
const syncManager = new SyncManager();
export default syncManager;
```

---

## 4. Offline-fähige Berechnungen

### 4.1 Lokale Berechnungs-Engine

```javascript
// static/js/calculations/offline-calculator.js
/**
 * Offline-fähige Berechnungs-Engine
 * Alle Berechnungen können ohne Server durchgeführt werden
 */

import { UValueCalculator } from './u-value.js';
import { HeatingLoadCalculator } from './heating-load.js';
import { EnergyBalanceCalculator } from './energy-balance.js';
import { EconomicCalculator } from './economic.js';

class OfflineCalculationEngine {
    constructor() {
        this.uValue = new UValueCalculator();
        this.heatingLoad = new HeatingLoadCalculator();
        this.energyBalance = new EnergyBalanceCalculator();
        this.economic = new EconomicCalculator();

        // Normwert-Datenbanken laden
        this.dataLoaded = this.loadReferenceData();
    }

    async loadReferenceData() {
        try {
            const [climateZones, materials, usageProfiles, energyFactors] = await Promise.all([
                this.fetchJSON('/static/data/climate-zones.json'),
                this.fetchJSON('/static/data/building-materials.json'),
                this.fetchJSON('/static/data/usage-profiles.json'),
                this.fetchJSON('/static/data/energy-factors.json'),
            ]);

            this.climateZones = climateZones;
            this.materials = materials;
            this.usageProfiles = usageProfiles;
            this.energyFactors = energyFactors;

            console.log('[Calculator] Reference data loaded');
            return true;
        } catch (error) {
            console.error('[Calculator] Failed to load reference data:', error);
            return false;
        }
    }

    async fetchJSON(url) {
        // Versuche aus Cache, dann aus Netzwerk
        const cached = await caches.match(url);
        if (cached) {
            return cached.json();
        }

        const response = await fetch(url);
        return response.json();
    }

    // ========================================================================
    // U-Wert Berechnung
    // ========================================================================

    calculateUValue(layers, rsi = 0.13, rse = 0.04) {
        return this.uValue.calculate(layers, rsi, rse);
    }

    // ========================================================================
    // Heizlast nach DIN EN 12831
    // ========================================================================

    calculateHeatingLoad(roomData) {
        const climateData = this.getClimateData(roomData.climate_zone);

        return this.heatingLoad.calculate({
            ...roomData,
            theta_ext: climateData.design_temperature,
            climate_correction: climateData.correction_factor
        });
    }

    getClimateData(zoneId) {
        return this.climateZones[zoneId] || this.climateZones['TRY04']; // Default: Mitteldeutschland
    }

    // ========================================================================
    // Energiebilanz
    // ========================================================================

    calculateEnergyBalance(buildingData) {
        return this.energyBalance.calculate(buildingData, {
            climateZones: this.climateZones,
            usageProfiles: this.usageProfiles,
            energyFactors: this.energyFactors
        });
    }

    // ========================================================================
    // Wirtschaftlichkeit
    // ========================================================================

    calculateEconomics(measureData, energySavings) {
        return this.economic.calculate(measureData, energySavings);
    }

    // ========================================================================
    // Vollständige Gebäudeberechnung
    // ========================================================================

    async calculateBuilding(building) {
        await this.dataLoaded;

        const results = {
            building_id: building.id,
            calculated_at: new Date().toISOString(),
            offline: !navigator.onLine,
            calculations: {}
        };

        // 1. U-Werte aller Bauteile
        results.calculations.u_values = {};
        for (const component of building.components || []) {
            if (component.layers) {
                results.calculations.u_values[component.id] = this.calculateUValue(
                    component.layers,
                    component.rsi,
                    component.rse
                );
            }
        }

        // 2. Heizlast pro Raum
        results.calculations.heating_load = {};
        let totalHeatingLoad = 0;

        for (const zone of building.zones || []) {
            const load = this.calculateHeatingLoad({
                ...zone,
                climate_zone: building.climate_zone,
                components: zone.components.map(c => ({
                    ...c,
                    u_value: results.calculations.u_values[c.component_id]?.u_value || c.u_value
                }))
            });
            results.calculations.heating_load[zone.id] = load;
            totalHeatingLoad += load.total;
        }

        results.calculations.total_heating_load = totalHeatingLoad;

        // 3. Energiebilanz
        results.calculations.energy_balance = this.calculateEnergyBalance({
            ...building,
            heating_load: totalHeatingLoad,
            u_values: results.calculations.u_values
        });

        // 4. Kennwerte
        const heatedArea = building.heated_area || 1;
        results.calculations.key_values = {
            specific_heating_load: totalHeatingLoad / heatedArea,
            primary_energy_demand: results.calculations.energy_balance.primary_energy / heatedArea,
            final_energy_demand: results.calculations.energy_balance.final_energy / heatedArea,
            co2_emissions: results.calculations.energy_balance.co2_emissions / heatedArea,
            efficiency_class: this.determineEfficiencyClass(
                results.calculations.energy_balance.primary_energy / heatedArea
            )
        };

        return results;
    }

    determineEfficiencyClass(primaryEnergyDemand) {
        const classes = [
            { max: 30, class: 'A+' },
            { max: 50, class: 'A' },
            { max: 75, class: 'B' },
            { max: 100, class: 'C' },
            { max: 130, class: 'D' },
            { max: 160, class: 'E' },
            { max: 200, class: 'F' },
            { max: 250, class: 'G' },
            { max: Infinity, class: 'H' }
        ];

        for (const c of classes) {
            if (primaryEnergyDemand <= c.max) {
                return c.class;
            }
        }
        return 'H';
    }
}

// Singleton
const calculator = new OfflineCalculationEngine();
export default calculator;
```

---

## 5. PWA Manifest

```json
// static/manifest.json
{
    "name": "EnergieberaterPRO",
    "short_name": "EnergiePRO",
    "description": "Professionelle Software für Energieberater - Offline-fähig",
    "start_url": "/",
    "display": "standalone",
    "background_color": "#ffffff",
    "theme_color": "#2563eb",
    "orientation": "any",
    "scope": "/",

    "icons": [
        {
            "src": "/static/images/icons/icon-72.png",
            "sizes": "72x72",
            "type": "image/png",
            "purpose": "any maskable"
        },
        {
            "src": "/static/images/icons/icon-96.png",
            "sizes": "96x96",
            "type": "image/png"
        },
        {
            "src": "/static/images/icons/icon-128.png",
            "sizes": "128x128",
            "type": "image/png"
        },
        {
            "src": "/static/images/icons/icon-192.png",
            "sizes": "192x192",
            "type": "image/png",
            "purpose": "any maskable"
        },
        {
            "src": "/static/images/icons/icon-384.png",
            "sizes": "384x384",
            "type": "image/png"
        },
        {
            "src": "/static/images/icons/icon-512.png",
            "sizes": "512x512",
            "type": "image/png",
            "purpose": "any maskable"
        }
    ],

    "screenshots": [
        {
            "src": "/static/images/screenshots/dashboard.png",
            "sizes": "1280x720",
            "type": "image/png",
            "form_factor": "wide",
            "label": "Dashboard-Ansicht"
        },
        {
            "src": "/static/images/screenshots/mobile.png",
            "sizes": "750x1334",
            "type": "image/png",
            "form_factor": "narrow",
            "label": "Mobile Ansicht"
        }
    ],

    "shortcuts": [
        {
            "name": "Neues Projekt",
            "short_name": "Projekt",
            "description": "Neues Projekt erstellen",
            "url": "/projects/new",
            "icons": [{ "src": "/static/images/icons/shortcut-project.png", "sizes": "96x96" }]
        },
        {
            "name": "Schnellberechnung",
            "short_name": "Rechner",
            "description": "Schnelle U-Wert Berechnung",
            "url": "/tools/u-value-calculator",
            "icons": [{ "src": "/static/images/icons/shortcut-calc.png", "sizes": "96x96" }]
        }
    ],

    "categories": ["business", "productivity", "utilities"],

    "related_applications": [],
    "prefer_related_applications": false,

    "share_target": {
        "action": "/share-target",
        "method": "POST",
        "enctype": "multipart/form-data",
        "params": {
            "title": "title",
            "text": "text",
            "url": "url",
            "files": [
                {
                    "name": "images",
                    "accept": ["image/*"]
                }
            ]
        }
    },

    "file_handlers": [
        {
            "action": "/open-file",
            "accept": {
                "application/pdf": [".pdf"],
                "image/*": [".jpg", ".jpeg", ".png"]
            }
        }
    ],

    "launch_handler": {
        "client_mode": "focus-existing"
    },

    "handle_links": "preferred"
}
```

---

## 6. Offline-UI-Komponenten

### 6.1 Offline-Status-Anzeige

```html
<!-- templates/components/offline_indicator.html -->
<div x-data="offlineIndicator()" x-init="init()">
    <!-- Status-Bar -->
    <div class="offline-status-bar"
         :class="{ 'online': isOnline, 'offline': !isOnline, 'syncing': isSyncing }"
         x-show="!isOnline || isSyncing || hasPendingChanges">

        <div class="status-content">
            <template x-if="!isOnline">
                <span>
                    <svg class="status-icon"><!-- Offline Icon --></svg>
                    Offline-Modus aktiv
                </span>
            </template>

            <template x-if="isOnline && isSyncing">
                <span>
                    <svg class="status-icon spinning"><!-- Sync Icon --></svg>
                    Synchronisiere...
                </span>
            </template>

            <template x-if="isOnline && !isSyncing && hasPendingChanges">
                <span>
                    <svg class="status-icon"><!-- Pending Icon --></svg>
                    <span x-text="pendingCount + ' Änderungen warten auf Sync'"></span>
                    <button @click="triggerSync()" class="sync-button">Jetzt synchronisieren</button>
                </span>
            </template>
        </div>

        <div class="status-details" x-show="showDetails">
            <div class="detail-item">
                <span class="label">Letzte Synchronisation:</span>
                <span class="value" x-text="lastSyncFormatted"></span>
            </div>
            <div class="detail-item" x-show="hasPendingChanges">
                <span class="label">Ausstehende Änderungen:</span>
                <span class="value" x-text="pendingCount"></span>
            </div>
            <div class="detail-item">
                <span class="label">Cache-Status:</span>
                <span class="value" x-text="cacheStatus"></span>
            </div>
        </div>
    </div>

    <!-- Toast für Sync-Fehler -->
    <div class="sync-error-toast"
         x-show="syncError"
         x-transition
         @click="syncError = null">
        <svg class="error-icon"><!-- Error Icon --></svg>
        <span x-text="syncError"></span>
    </div>
</div>

<script>
function offlineIndicator() {
    return {
        isOnline: navigator.onLine,
        isSyncing: false,
        hasPendingChanges: false,
        pendingCount: 0,
        lastSync: null,
        showDetails: false,
        syncError: null,
        cacheStatus: 'Wird geladen...',

        init() {
            // Event Listener
            window.addEventListener('online', () => this.handleOnline());
            window.addEventListener('offline', () => this.handleOffline());
            window.addEventListener('sync-complete', (e) => this.handleSyncComplete(e.detail));
            window.addEventListener('sync-error', (e) => this.handleSyncError(e.detail));
            window.addEventListener('connection-status', (e) => {
                this.isOnline = e.detail.online;
            });

            // Initial check
            this.checkPendingChanges();
            this.checkCacheStatus();

            // Periodische Prüfung
            setInterval(() => this.checkPendingChanges(), 10000);
        },

        handleOnline() {
            this.isOnline = true;
            this.isSyncing = true;
        },

        handleOffline() {
            this.isOnline = false;
            this.isSyncing = false;
        },

        handleSyncComplete(detail) {
            this.isSyncing = false;
            this.lastSync = Date.now();
            this.checkPendingChanges();

            if (detail.success) {
                this.syncError = null;
            }
        },

        handleSyncError(detail) {
            this.syncError = detail.error || 'Synchronisation fehlgeschlagen';
            this.isSyncing = false;

            // Auto-hide nach 5 Sekunden
            setTimeout(() => {
                this.syncError = null;
            }, 5000);
        },

        async checkPendingChanges() {
            const changes = await offlineDB.getPendingChanges();
            this.hasPendingChanges = changes.length > 0;
            this.pendingCount = changes.length;
        },

        async checkCacheStatus() {
            if ('storage' in navigator && 'estimate' in navigator.storage) {
                const { usage, quota } = await navigator.storage.estimate();
                const usedMB = Math.round(usage / 1024 / 1024);
                const quotaMB = Math.round(quota / 1024 / 1024);
                this.cacheStatus = `${usedMB} MB / ${quotaMB} MB verwendet`;
            } else {
                this.cacheStatus = 'Nicht verfügbar';
            }
        },

        triggerSync() {
            if (window.syncManager) {
                this.isSyncing = true;
                window.syncManager.syncAll();
            }
        },

        get lastSyncFormatted() {
            if (!this.lastSync) return 'Noch nie';

            const diff = Date.now() - this.lastSync;
            if (diff < 60000) return 'Gerade eben';
            if (diff < 3600000) return `vor ${Math.round(diff / 60000)} Minuten`;
            if (diff < 86400000) return `vor ${Math.round(diff / 3600000)} Stunden`;
            return new Date(this.lastSync).toLocaleDateString('de-DE');
        }
    };
}
</script>

<style>
.offline-status-bar {
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
    padding: 8px 16px;
    font-size: 14px;
    z-index: 1000;
    transition: all 0.3s ease;
}

.offline-status-bar.online {
    background: #10b981;
    color: white;
}

.offline-status-bar.offline {
    background: #ef4444;
    color: white;
}

.offline-status-bar.syncing {
    background: #3b82f6;
    color: white;
}

.status-icon {
    width: 16px;
    height: 16px;
    margin-right: 8px;
    vertical-align: middle;
}

.status-icon.spinning {
    animation: spin 1s linear infinite;
}

@keyframes spin {
    from { transform: rotate(0deg); }
    to { transform: rotate(360deg); }
}

.sync-button {
    margin-left: 16px;
    padding: 4px 12px;
    background: rgba(255,255,255,0.2);
    border: 1px solid rgba(255,255,255,0.5);
    border-radius: 4px;
    color: white;
    cursor: pointer;
}

.sync-button:hover {
    background: rgba(255,255,255,0.3);
}

.sync-error-toast {
    position: fixed;
    bottom: 60px;
    left: 50%;
    transform: translateX(-50%);
    background: #dc2626;
    color: white;
    padding: 12px 24px;
    border-radius: 8px;
    box-shadow: 0 4px 12px rgba(0,0,0,0.3);
    cursor: pointer;
}
</style>
```

---

## 7. Checkliste

```
✓ Offline-First Philosophie definiert
✓ Caching-Strategien dokumentiert (Cache-First, Network-First, Stale-While-Revalidate)
✓ Service Worker implementiert
✓ IndexedDB für lokale Datenspeicherung
✓ Background Sync für verzögerte Synchronisation
✓ Konfliktauflösung bei gleichzeitiger Bearbeitung
✓ Offline-fähige Berechnungs-Engine
✓ PWA Manifest konfiguriert
✓ Offline-Status-UI-Komponenten
✓ Normwert-Datenbanken lokal verfügbar
✓ Push Notifications integriert
```

---

*Letzte Aktualisierung: 2025-11-25*
*Version: 1.0.0*
