# Kollaborative Dokumentenbearbeitung

## Referenzen
- [Yjs CRDT Framework](https://yjs.dev/)
- [y-websocket Provider](https://github.com/yjs/y-websocket)
- [Tiptap Editor](https://tiptap.dev/)
- [Hocuspocus Server](https://tiptap.dev/hocuspocus)

---

## 1. Architektur-Übersicht

### 1.1 CRDT-basierte Synchronisation

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    KOLLABORATIVE BEARBEITUNG                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  CONFLICT-FREE REPLICATED DATA TYPES (CRDT)                                 │
│  ═══════════════════════════════════════════                                │
│                                                                             │
│  Vorteile:                                                                  │
│  • Keine Konflikte bei gleichzeitiger Bearbeitung                          │
│  • Offline-fähig (Änderungen werden gemerged)                              │
│  • Echtzeit-Synchronisation                                                │
│  • Mathematisch bewiesen korrekt                                           │
│                                                                             │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                  │
│  │  Client A    │    │   Server     │    │  Client B    │                  │
│  │              │◄──►│  (Yjs Doc)   │◄──►│              │                  │
│  │  Yjs Doc     │    │              │    │  Yjs Doc     │                  │
│  │  (Replica)   │    │  WebSocket   │    │  (Replica)   │                  │
│  └──────────────┘    └──────────────┘    └──────────────┘                  │
│         │                   │                   │                          │
│         ▼                   ▼                   ▼                          │
│  ┌──────────────────────────────────────────────────────┐                  │
│  │              Automatisches Merging                    │                  │
│  │     Änderungen werden ohne Konflikte zusammengeführt │                  │
│  └──────────────────────────────────────────────────────┘                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 System-Architektur

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       SYSTEM-ARCHITEKTUR                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         FRONTEND                                     │   │
│  ├─────────────────────────────────────────────────────────────────────┤   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                  │   │
│  │  │  Tiptap     │  │  Yjs        │  │  Awareness  │                  │   │
│  │  │  Editor     │  │  Document   │  │  (Cursors)  │                  │   │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘                  │   │
│  │         │                │                │                          │   │
│  │         └────────────────┼────────────────┘                          │   │
│  │                          │                                           │   │
│  │                 ┌────────▼────────┐                                  │   │
│  │                 │  y-websocket    │                                  │   │
│  │                 │  Provider       │                                  │   │
│  │                 └────────┬────────┘                                  │   │
│  └──────────────────────────┼──────────────────────────────────────────┘   │
│                             │ WebSocket                                     │
│  ┌──────────────────────────┼──────────────────────────────────────────┐   │
│  │                         BACKEND                                      │   │
│  ├──────────────────────────┼──────────────────────────────────────────┤   │
│  │                 ┌────────▼────────┐                                  │   │
│  │                 │  Hocuspocus     │                                  │   │
│  │                 │  Server         │                                  │   │
│  │                 └────────┬────────┘                                  │   │
│  │                          │                                           │   │
│  │         ┌────────────────┼────────────────┐                          │   │
│  │         │                │                │                          │   │
│  │  ┌──────▼──────┐  ┌──────▼──────┐  ┌──────▼──────┐                  │   │
│  │  │ PostgreSQL  │  │   Redis     │  │    Auth     │                  │   │
│  │  │ (Persist)   │  │  (Pubsub)   │  │  (Verify)   │                  │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘                  │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Backend: Hocuspocus Server

### 2.1 Server-Implementierung

```python
# app/collaboration/hocuspocus_server.py
"""Hocuspocus-kompatibler WebSocket Server für Yjs"""

import asyncio
import json
from typing import Dict, Set, Optional
from dataclasses import dataclass, field
from fastapi import WebSocket, WebSocketDisconnect
import y_py as Y  # Python Yjs Bindings

@dataclass
class Document:
    """Yjs Document Wrapper"""
    name: str
    ydoc: Y.YDoc
    connections: Set[WebSocket] = field(default_factory=set)
    awareness: Dict[int, dict] = field(default_factory=dict)
    last_saved: float = 0

class CollaborationServer:
    """WebSocket Server für Echtzeit-Kollaboration"""

    def __init__(self, db_session, redis_client, auth_service):
        self.db = db_session
        self.redis = redis_client
        self.auth = auth_service
        self.documents: Dict[str, Document] = {}
        self.client_ids: Dict[WebSocket, int] = {}
        self._client_id_counter = 0

    async def handle_connection(self, websocket: WebSocket, document_name: str):
        """Behandelt neue WebSocket-Verbindung"""

        # Authentifizierung
        token = websocket.query_params.get("token")
        user = await self.auth.verify_token(token)
        if not user:
            await websocket.close(code=4001, reason="Unauthorized")
            return

        # Berechtigung prüfen
        if not await self._check_permission(user, document_name):
            await websocket.close(code=4003, reason="Forbidden")
            return

        await websocket.accept()

        # Client-ID zuweisen
        client_id = self._get_next_client_id()
        self.client_ids[websocket] = client_id

        # Dokument laden oder erstellen
        doc = await self._get_or_create_document(document_name)
        doc.connections.add(websocket)

        try:
            # Initialer Sync
            await self._send_sync_step1(websocket, doc)

            # Message Loop
            while True:
                data = await websocket.receive_bytes()
                await self._handle_message(websocket, doc, data, user, client_id)

        except WebSocketDisconnect:
            await self._handle_disconnect(websocket, doc, client_id)
        except Exception as e:
            print(f"WebSocket error: {e}")
            await self._handle_disconnect(websocket, doc, client_id)

    async def _get_or_create_document(self, name: str) -> Document:
        """Lädt oder erstellt Dokument"""
        if name not in self.documents:
            ydoc = Y.YDoc()

            # Aus DB laden
            stored = await self._load_from_db(name)
            if stored:
                Y.apply_update(ydoc, stored)

            self.documents[name] = Document(name=name, ydoc=ydoc)

        return self.documents[name]

    async def _handle_message(
        self,
        websocket: WebSocket,
        doc: Document,
        data: bytes,
        user: dict,
        client_id: int
    ):
        """Verarbeitet eingehende Nachricht"""
        message_type = data[0]

        if message_type == 0:  # Sync
            await self._handle_sync(websocket, doc, data[1:])

        elif message_type == 1:  # Awareness
            await self._handle_awareness(websocket, doc, data[1:], user, client_id)

        elif message_type == 2:  # Auth (optional)
            pass  # Bereits beim Connect gehandhabt

    async def _handle_sync(self, websocket: WebSocket, doc: Document, data: bytes):
        """Sync-Protokoll"""
        sync_type = data[0]

        if sync_type == 0:  # Sync Step 1 (Request)
            state_vector = data[1:]
            diff = Y.encode_state_as_update(doc.ydoc, state_vector)
            await websocket.send_bytes(bytes([0, 1]) + diff)

        elif sync_type == 1:  # Sync Step 2 (Response/Update)
            update = data[1:]
            Y.apply_update(doc.ydoc, update)

            # An alle anderen Clients broadcasten
            for conn in doc.connections:
                if conn != websocket:
                    await conn.send_bytes(bytes([0, 2]) + update)

            # Periodisch speichern
            await self._maybe_persist(doc)

        elif sync_type == 2:  # Update
            update = data[1:]
            Y.apply_update(doc.ydoc, update)

            for conn in doc.connections:
                if conn != websocket:
                    await conn.send_bytes(bytes([0, 2]) + update)

            await self._maybe_persist(doc)

    async def _handle_awareness(
        self,
        websocket: WebSocket,
        doc: Document,
        data: bytes,
        user: dict,
        client_id: int
    ):
        """Awareness-Updates (Cursor-Position, etc.)"""
        # Awareness-State parsen und speichern
        awareness_update = self._parse_awareness(data)

        # User-Info hinzufügen
        awareness_update["user"] = {
            "name": user.get("name", "Unbekannt"),
            "color": self._get_user_color(client_id),
            "avatar": user.get("avatar_url")
        }

        doc.awareness[client_id] = awareness_update

        # An alle anderen broadcasten
        broadcast_data = self._encode_awareness(client_id, awareness_update)
        for conn in doc.connections:
            if conn != websocket:
                await conn.send_bytes(bytes([1]) + broadcast_data)

    async def _send_sync_step1(self, websocket: WebSocket, doc: Document):
        """Sendet initialen Sync"""
        # Gesamten State senden
        state = Y.encode_state_as_update(doc.ydoc)
        await websocket.send_bytes(bytes([0, 2]) + state)

        # Awareness aller verbundenen Clients senden
        for cid, awareness in doc.awareness.items():
            data = self._encode_awareness(cid, awareness)
            await websocket.send_bytes(bytes([1]) + data)

    async def _handle_disconnect(self, websocket: WebSocket, doc: Document, client_id: int):
        """Behandelt Disconnect"""
        doc.connections.discard(websocket)
        self.client_ids.pop(websocket, None)

        # Awareness entfernen
        if client_id in doc.awareness:
            del doc.awareness[client_id]
            # Disconnect an andere broadcasten
            for conn in doc.connections:
                await conn.send_bytes(bytes([1, 0]) + client_id.to_bytes(4, 'big'))

        # Dokument speichern wenn leer
        if not doc.connections:
            await self._persist_document(doc)
            # Optional: Dokument aus Memory entfernen
            # del self.documents[doc.name]

    async def _maybe_persist(self, doc: Document):
        """Speichert Dokument wenn nötig"""
        import time
        now = time.time()

        # Alle 30 Sekunden speichern
        if now - doc.last_saved > 30:
            await self._persist_document(doc)
            doc.last_saved = now

    async def _persist_document(self, doc: Document):
        """Speichert Dokument in DB"""
        state = Y.encode_state_as_update(doc.ydoc)

        await self.db.execute("""
            INSERT INTO document_states (document_name, state, updated_at)
            VALUES (:name, :state, NOW())
            ON CONFLICT (document_name)
            DO UPDATE SET state = :state, updated_at = NOW()
        """, {"name": doc.name, "state": state})

        await self.db.commit()

    async def _load_from_db(self, name: str) -> Optional[bytes]:
        """Lädt Dokument aus DB"""
        result = await self.db.execute("""
            SELECT state FROM document_states WHERE document_name = :name
        """, {"name": name})
        row = result.fetchone()
        return row.state if row else None

    async def _check_permission(self, user: dict, document_name: str) -> bool:
        """Prüft Berechtigung für Dokument"""
        # Dokument-Namen parsen: "project:{project_id}:report:{report_id}"
        parts = document_name.split(":")

        if parts[0] == "project":
            project_id = parts[1]
            # Prüfe Projekt-Zugehörigkeit
            result = await self.db.execute("""
                SELECT 1 FROM project_members
                WHERE project_id = :project_id AND user_id = :user_id
            """, {"project_id": project_id, "user_id": user["id"]})
            return result.fetchone() is not None

        return False

    def _get_next_client_id(self) -> int:
        self._client_id_counter += 1
        return self._client_id_counter

    def _get_user_color(self, client_id: int) -> str:
        """Generiert konsistente Farbe für Client"""
        colors = [
            "#FF6B6B", "#4ECDC4", "#45B7D1", "#96CEB4",
            "#FFEAA7", "#DDA0DD", "#98D8C8", "#F7DC6F"
        ]
        return colors[client_id % len(colors)]

    def _parse_awareness(self, data: bytes) -> dict:
        """Parst Awareness-Update"""
        # Vereinfachte Implementation
        return json.loads(data.decode('utf-8'))

    def _encode_awareness(self, client_id: int, awareness: dict) -> bytes:
        """Enkodiert Awareness-Update"""
        return json.dumps({"clientId": client_id, **awareness}).encode('utf-8')
```

### 2.2 FastAPI Integration

```python
# app/api/collaboration.py
"""Collaboration API Endpoints"""

from fastapi import APIRouter, WebSocket, Depends
from app.collaboration.hocuspocus_server import CollaborationServer
from app.dependencies import get_collab_server

router = APIRouter(prefix="/collaboration", tags=["collaboration"])

@router.websocket("/ws/{document_name}")
async def websocket_endpoint(
    websocket: WebSocket,
    document_name: str,
    server: CollaborationServer = Depends(get_collab_server)
):
    """WebSocket Endpoint für Kollaboration"""
    await server.handle_connection(websocket, document_name)


@router.get("/documents/{document_name}/participants")
async def get_participants(
    document_name: str,
    server: CollaborationServer = Depends(get_collab_server)
):
    """Gibt aktive Teilnehmer zurück"""
    doc = server.documents.get(document_name)
    if not doc:
        return {"participants": []}

    return {
        "participants": [
            {
                "client_id": cid,
                "user": awareness.get("user", {}),
                "cursor": awareness.get("cursor")
            }
            for cid, awareness in doc.awareness.items()
        ]
    }
```

---

## 3. Frontend: Tiptap + Yjs

### 3.1 Editor-Komponente

```javascript
// static/js/collaborative-editor.js
/**
 * Kollaborativer Rich-Text-Editor mit Tiptap und Yjs
 */

import { Editor } from '@tiptap/core';
import StarterKit from '@tiptap/starter-kit';
import Collaboration from '@tiptap/extension-collaboration';
import CollaborationCursor from '@tiptap/extension-collaboration-cursor';
import * as Y from 'yjs';
import { WebsocketProvider } from 'y-websocket';

class CollaborativeEditor {
    constructor(element, documentName, options = {}) {
        this.element = element;
        this.documentName = documentName;
        this.options = options;

        this.ydoc = null;
        this.provider = null;
        this.editor = null;

        this.init();
    }

    async init() {
        // Yjs Document
        this.ydoc = new Y.Doc();

        // WebSocket Provider
        const wsUrl = `${window.location.protocol === 'https:' ? 'wss:' : 'ws:'}//${window.location.host}/collaboration/ws/${this.documentName}`;

        this.provider = new WebsocketProvider(
            wsUrl,
            this.documentName,
            this.ydoc,
            {
                params: {
                    token: localStorage.getItem('access_token')
                }
            }
        );

        // Awareness für Cursor-Positionen
        const currentUser = await this.getCurrentUser();

        this.provider.awareness.setLocalStateField('user', {
            name: currentUser.name,
            color: this.getRandomColor(),
            avatar: currentUser.avatar_url
        });

        // Tiptap Editor
        this.editor = new Editor({
            element: this.element,
            extensions: [
                StarterKit.configure({
                    history: false // Yjs hat eigene History
                }),
                Collaboration.configure({
                    document: this.ydoc
                }),
                CollaborationCursor.configure({
                    provider: this.provider,
                    user: {
                        name: currentUser.name,
                        color: this.getRandomColor()
                    }
                }),
                // Weitere Extensions...
                ...this.getAdditionalExtensions()
            ],
            editorProps: {
                attributes: {
                    class: 'collaborative-editor-content'
                }
            },
            onUpdate: ({ editor }) => {
                this.handleUpdate(editor);
            }
        });

        // Event Listener
        this.setupEventListeners();
    }

    getAdditionalExtensions() {
        return [
            // Tabellen
            Table.configure({ resizable: true }),
            TableRow,
            TableHeader,
            TableCell,

            // Bilder
            Image.configure({
                inline: true,
                allowBase64: true
            }),

            // Links
            Link.configure({
                openOnClick: false
            }),

            // Placeholder
            Placeholder.configure({
                placeholder: 'Beginnen Sie zu schreiben...'
            }),

            // Task Lists
            TaskList,
            TaskItem.configure({
                nested: true
            }),

            // Highlight
            Highlight.configure({
                multicolor: true
            }),

            // Text Alignment
            TextAlign.configure({
                types: ['heading', 'paragraph']
            })
        ];
    }

    setupEventListeners() {
        // Verbindungsstatus
        this.provider.on('status', ({ status }) => {
            this.element.dispatchEvent(new CustomEvent('connection-status', {
                detail: { connected: status === 'connected' }
            }));
        });

        // Sync-Status
        this.provider.on('sync', (isSynced) => {
            this.element.dispatchEvent(new CustomEvent('sync-status', {
                detail: { synced: isSynced }
            }));
        });

        // Awareness-Updates (andere User)
        this.provider.awareness.on('change', () => {
            const states = Array.from(this.provider.awareness.getStates().values());
            this.element.dispatchEvent(new CustomEvent('participants-changed', {
                detail: { participants: states.map(s => s.user).filter(Boolean) }
            }));
        });
    }

    handleUpdate(editor) {
        // Autosave triggern (Debounced)
        clearTimeout(this._saveTimeout);
        this._saveTimeout = setTimeout(() => {
            this.element.dispatchEvent(new CustomEvent('content-changed', {
                detail: {
                    html: editor.getHTML(),
                    json: editor.getJSON()
                }
            }));
        }, 1000);
    }

    async getCurrentUser() {
        const response = await fetch('/api/v1/users/me');
        return response.json();
    }

    getRandomColor() {
        const colors = [
            '#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4',
            '#FFEAA7', '#DDA0DD', '#98D8C8', '#F7DC6F'
        ];
        return colors[Math.floor(Math.random() * colors.length)];
    }

    // Public API

    getHTML() {
        return this.editor.getHTML();
    }

    getJSON() {
        return this.editor.getJSON();
    }

    setContent(content) {
        this.editor.commands.setContent(content);
    }

    focus() {
        this.editor.commands.focus();
    }

    destroy() {
        this.provider.destroy();
        this.editor.destroy();
        this.ydoc.destroy();
    }

    // Toolbar Commands

    toggleBold() {
        this.editor.chain().focus().toggleBold().run();
    }

    toggleItalic() {
        this.editor.chain().focus().toggleItalic().run();
    }

    toggleHeading(level) {
        this.editor.chain().focus().toggleHeading({ level }).run();
    }

    toggleBulletList() {
        this.editor.chain().focus().toggleBulletList().run();
    }

    toggleOrderedList() {
        this.editor.chain().focus().toggleOrderedList().run();
    }

    insertTable(rows = 3, cols = 3) {
        this.editor.chain().focus().insertTable({ rows, cols, withHeaderRow: true }).run();
    }

    insertImage(src, alt = '') {
        this.editor.chain().focus().setImage({ src, alt }).run();
    }

    setLink(href) {
        this.editor.chain().focus().setLink({ href }).run();
    }

    undo() {
        // Yjs Undo
        if (this.ydoc.undoManager) {
            this.ydoc.undoManager.undo();
        }
    }

    redo() {
        if (this.ydoc.undoManager) {
            this.ydoc.undoManager.redo();
        }
    }
}

// Export
window.CollaborativeEditor = CollaborativeEditor;
```

### 3.2 Editor UI-Komponente

```html
<!-- templates/components/collaborative_editor.html -->
<div class="collaborative-editor-wrapper"
     x-data="collaborativeEditorComponent('{{ document_name }}')"
     x-init="init()">

    <!-- Toolbar -->
    <div class="editor-toolbar">
        <div class="toolbar-group">
            <button @click="editor.toggleBold()"
                    :class="{ 'active': isActive('bold') }"
                    title="Fett (Ctrl+B)">
                <svg><!-- Bold Icon --></svg>
            </button>
            <button @click="editor.toggleItalic()"
                    :class="{ 'active': isActive('italic') }"
                    title="Kursiv (Ctrl+I)">
                <svg><!-- Italic Icon --></svg>
            </button>
            <button @click="editor.toggleUnderline()"
                    :class="{ 'active': isActive('underline') }"
                    title="Unterstrichen (Ctrl+U)">
                <svg><!-- Underline Icon --></svg>
            </button>
        </div>

        <div class="toolbar-separator"></div>

        <div class="toolbar-group">
            <button @click="editor.toggleHeading(1)"
                    :class="{ 'active': isActive('heading', { level: 1 }) }"
                    title="Überschrift 1">
                H1
            </button>
            <button @click="editor.toggleHeading(2)"
                    :class="{ 'active': isActive('heading', { level: 2 }) }"
                    title="Überschrift 2">
                H2
            </button>
            <button @click="editor.toggleHeading(3)"
                    :class="{ 'active': isActive('heading', { level: 3 }) }"
                    title="Überschrift 3">
                H3
            </button>
        </div>

        <div class="toolbar-separator"></div>

        <div class="toolbar-group">
            <button @click="editor.toggleBulletList()"
                    :class="{ 'active': isActive('bulletList') }"
                    title="Aufzählung">
                <svg><!-- List Icon --></svg>
            </button>
            <button @click="editor.toggleOrderedList()"
                    :class="{ 'active': isActive('orderedList') }"
                    title="Nummerierung">
                <svg><!-- Numbered List Icon --></svg>
            </button>
            <button @click="showTableDialog = true" title="Tabelle einfügen">
                <svg><!-- Table Icon --></svg>
            </button>
        </div>

        <div class="toolbar-separator"></div>

        <div class="toolbar-group">
            <button @click="editor.undo()" title="Rückgängig (Ctrl+Z)">
                <svg><!-- Undo Icon --></svg>
            </button>
            <button @click="editor.redo()" title="Wiederholen (Ctrl+Y)">
                <svg><!-- Redo Icon --></svg>
            </button>
        </div>

        <!-- Spacer -->
        <div class="toolbar-spacer"></div>

        <!-- Participants -->
        <div class="toolbar-participants">
            <template x-for="participant in participants" :key="participant.name">
                <div class="participant-avatar"
                     :style="{ backgroundColor: participant.color }"
                     :title="participant.name">
                    <span x-text="participant.name.charAt(0).toUpperCase()"></span>
                </div>
            </template>
        </div>

        <!-- Connection Status -->
        <div class="connection-status" :class="{ 'connected': isConnected, 'disconnected': !isConnected }">
            <span x-text="isConnected ? 'Verbunden' : 'Offline'"></span>
        </div>
    </div>

    <!-- Editor Content -->
    <div class="editor-container">
        <div x-ref="editorElement" class="editor-content"></div>
    </div>

    <!-- Status Bar -->
    <div class="editor-status-bar">
        <span class="word-count" x-text="wordCount + ' Wörter'"></span>
        <span class="char-count" x-text="charCount + ' Zeichen'"></span>
        <span class="save-status" x-text="saveStatus"></span>
    </div>

    <!-- Table Dialog -->
    <div class="modal" x-show="showTableDialog" x-cloak>
        <div class="modal-content">
            <h3>Tabelle einfügen</h3>
            <div class="form-group">
                <label>Zeilen:</label>
                <input type="number" x-model="tableRows" min="1" max="20">
            </div>
            <div class="form-group">
                <label>Spalten:</label>
                <input type="number" x-model="tableCols" min="1" max="10">
            </div>
            <div class="modal-actions">
                <button @click="showTableDialog = false">Abbrechen</button>
                <button @click="insertTable()" class="primary">Einfügen</button>
            </div>
        </div>
    </div>
</div>

<script>
function collaborativeEditorComponent(documentName) {
    return {
        documentName,
        editor: null,
        isConnected: false,
        isSynced: false,
        participants: [],
        wordCount: 0,
        charCount: 0,
        saveStatus: 'Gespeichert',
        showTableDialog: false,
        tableRows: 3,
        tableCols: 3,

        init() {
            this.editor = new CollaborativeEditor(
                this.$refs.editorElement,
                this.documentName
            );

            // Event Listeners
            this.$refs.editorElement.addEventListener('connection-status', (e) => {
                this.isConnected = e.detail.connected;
            });

            this.$refs.editorElement.addEventListener('sync-status', (e) => {
                this.isSynced = e.detail.synced;
            });

            this.$refs.editorElement.addEventListener('participants-changed', (e) => {
                this.participants = e.detail.participants;
            });

            this.$refs.editorElement.addEventListener('content-changed', (e) => {
                this.updateCounts(e.detail.html);
                this.saveStatus = 'Speichert...';
                // Wird automatisch durch Yjs synchronisiert
                setTimeout(() => {
                    this.saveStatus = 'Gespeichert';
                }, 500);
            });
        },

        isActive(type, attributes = {}) {
            if (!this.editor?.editor) return false;
            return this.editor.editor.isActive(type, attributes);
        },

        updateCounts(html) {
            const text = html.replace(/<[^>]*>/g, '');
            this.charCount = text.length;
            this.wordCount = text.trim().split(/\s+/).filter(Boolean).length;
        },

        insertTable() {
            this.editor.insertTable(this.tableRows, this.tableCols);
            this.showTableDialog = false;
        },

        destroy() {
            if (this.editor) {
                this.editor.destroy();
            }
        }
    };
}
</script>
```

---

## 4. Spezielle Dokument-Typen

### 4.1 Bericht-Editor mit Bausteinen

```javascript
// static/js/report-editor.js
/**
 * Spezialisierter Editor für Energieberichte
 * Kombiniert Yjs mit Baustein-System
 */

class ReportEditor extends CollaborativeEditor {
    constructor(element, documentName, options = {}) {
        super(element, documentName, options);
        this.blocks = [];
    }

    getAdditionalExtensions() {
        return [
            ...super.getAdditionalExtensions(),

            // Report Block Extension
            ReportBlock.configure({
                onBlockInsert: (block) => this.handleBlockInsert(block),
                onBlockRemove: (blockId) => this.handleBlockRemove(blockId)
            }),

            // Calculation Result Extension
            CalculationResult.configure({
                renderCalculation: (data) => this.renderCalculation(data)
            }),

            // Editable Variables
            EditableVariable.configure({
                variables: this.options.variables || {}
            })
        ];
    }

    async handleBlockInsert(block) {
        // Block-Daten laden
        const response = await fetch(`/api/v1/report-blocks/${block.id}`);
        const blockData = await response.json();

        // Block in Editor einfügen
        this.editor.chain()
            .focus()
            .insertContent({
                type: 'reportBlock',
                attrs: {
                    blockId: block.id,
                    blockType: block.type
                },
                content: this.parseBlockContent(blockData.template)
            })
            .run();
    }

    parseBlockContent(template) {
        // Template-Variablen ersetzen
        // z.B. {{building.heated_area}} → tatsächlicher Wert
        const variables = this.options.projectData || {};

        let content = template;
        content = content.replace(/\{\{([^}]+)\}\}/g, (match, path) => {
            const value = this.getNestedValue(variables, path);
            return value !== undefined ? value : match;
        });

        return content;
    }

    getNestedValue(obj, path) {
        return path.split('.').reduce((current, key) => current?.[key], obj);
    }

    renderCalculation(data) {
        // Berechnungsergebnis als formatierte Tabelle
        return `
            <div class="calculation-result" data-calculation-id="${data.id}">
                <h4>${data.title}</h4>
                <table>
                    ${Object.entries(data.results).map(([key, value]) => `
                        <tr>
                            <td>${key}</td>
                            <td>${value.value} ${value.unit}</td>
                        </tr>
                    `).join('')}
                </table>
            </div>
        `;
    }
}
```

---

## 5. Datenbank-Schema

```sql
-- Kollaborations-bezogene Tabellen

-- Document States (Yjs Persistenz)
CREATE TABLE document_states (
    document_name VARCHAR(255) PRIMARY KEY,
    state BYTEA NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Document Snapshots (Versionierung)
CREATE TABLE document_snapshots (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    document_name VARCHAR(255) NOT NULL,
    state BYTEA NOT NULL,
    version INTEGER NOT NULL,
    created_by UUID REFERENCES users(id),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    label VARCHAR(255),

    UNIQUE(document_name, version)
);

-- Document Access Log
CREATE TABLE document_access_log (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    document_name VARCHAR(255) NOT NULL,
    user_id UUID REFERENCES users(id),
    action VARCHAR(50) NOT NULL, -- 'open', 'edit', 'close'
    timestamp TIMESTAMPTZ DEFAULT NOW(),
    duration_seconds INTEGER,

    INDEX idx_doc_access_time (document_name, timestamp)
);

-- Active Sessions (für Awareness)
CREATE TABLE document_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    document_name VARCHAR(255) NOT NULL,
    user_id UUID REFERENCES users(id),
    client_id INTEGER NOT NULL,
    connected_at TIMESTAMPTZ DEFAULT NOW(),
    last_activity TIMESTAMPTZ DEFAULT NOW(),
    cursor_position JSONB,

    UNIQUE(document_name, client_id)
);

-- Comments (Inline-Kommentare)
CREATE TABLE document_comments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    document_name VARCHAR(255) NOT NULL,
    user_id UUID REFERENCES users(id),
    content TEXT NOT NULL,
    position JSONB NOT NULL, -- {from: number, to: number}
    resolved BOOLEAN DEFAULT FALSE,
    resolved_by UUID REFERENCES users(id),
    resolved_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),

    INDEX idx_doc_comments (document_name, resolved)
);

-- Comment Replies
CREATE TABLE document_comment_replies (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    comment_id UUID REFERENCES document_comments(id) ON DELETE CASCADE,
    user_id UUID REFERENCES users(id),
    content TEXT NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

---

## 6. Checkliste

```
✓ CRDT-basierte Architektur mit Yjs
✓ Hocuspocus-kompatibler WebSocket Server
✓ Tiptap Editor Integration
✓ Echtzeit-Cursor und Awareness
✓ Offline-Fähigkeit durch Yjs
✓ Automatische Konfliktauflösung
✓ Persistenz in PostgreSQL
✓ Versionierung mit Snapshots
✓ Spezialisierter Bericht-Editor
✓ Kommentar-System
✓ Datenbank-Schema
```

---

*Letzte Aktualisierung: 2025-11-25*
*Version: 1.0.0*
