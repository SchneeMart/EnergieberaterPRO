# Kommunikations- und Ticketsystem

## Referenzen
- [WebSocket Real-time Communication](https://websockets.spec.whatwg.org/)
- [Matrix Protocol](https://matrix.org/)
- [Open Source Ticketing](https://osticket.com/)

---

## 1. Übersicht

### 1.1 Kommunikations-Architektur

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    KOMMUNIKATIONS-ARCHITEKTUR                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  INTERNE KOMMUNIKATION                  EXTERNE KOMMUNIKATION               │
│  ════════════════════                   ═══════════════════                 │
│                                                                             │
│  ┌─────────────────────┐                ┌─────────────────────┐             │
│  │  Direktnachrichten  │                │  Kunden-Portal      │             │
│  │  • 1:1 Chat         │                │  • Nachrichten      │             │
│  │  • Mentions         │                │  • Dokumente        │             │
│  │  • Reaktionen       │                │  • Status-Updates   │             │
│  └─────────────────────┘                └─────────────────────┘             │
│                                                                             │
│  ┌─────────────────────┐                ┌─────────────────────┐             │
│  │  Gruppenchats       │                │  E-Mail Integration │             │
│  │  • Teams            │                │  • Eingehend        │             │
│  │  • Projekte         │                │  • Ausgehend        │             │
│  │  • Themen           │                │  • Templates        │             │
│  └─────────────────────┘                └─────────────────────┘             │
│                                                                             │
│  ┌─────────────────────┐                ┌─────────────────────┐             │
│  │  Benachrichtigungen │                │  Ticket-System      │             │
│  │  • In-App           │                │  • Support          │             │
│  │  • Push             │                │  • Bug-Reports      │             │
│  │  • E-Mail           │                │  • Feature-Requests │             │
│  └─────────────────────┘                └─────────────────────┘             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 System-Komponenten

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       SYSTEM-KOMPONENTEN                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         FRONTEND                                     │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                  │   │
│  │  │  Chat UI    │  │  Ticket UI  │  │  Notif. UI  │                  │   │
│  │  │  • Messages │  │  • List     │  │  • Bell     │                  │   │
│  │  │  • Threads  │  │  • Detail   │  │  • Dropdown │                  │   │
│  │  │  • Files    │  │  • Forms    │  │  • Toast    │                  │   │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘                  │   │
│  │         └────────────────┼────────────────┘                          │   │
│  │                          │ WebSocket / REST                          │   │
│  └──────────────────────────┼───────────────────────────────────────────┘   │
│                             │                                               │
│  ┌──────────────────────────┼───────────────────────────────────────────┐   │
│  │                     BACKEND SERVICES                                  │   │
│  │  ┌─────────────────────────────────────────────────────────────────┐ │   │
│  │  │                  WebSocket Gateway                               │ │   │
│  │  │  • Connection Management                                         │ │   │
│  │  │  • Message Routing                                               │ │   │
│  │  │  • Presence Tracking                                             │ │   │
│  │  └───────────────────────┬─────────────────────────────────────────┘ │   │
│  │                          │                                           │   │
│  │  ┌───────────────┐ ┌─────┴─────┐ ┌───────────────┐ ┌───────────────┐│   │
│  │  │Message Service│ │Ticket Svc │ │Notification   │ │Email Service  ││   │
│  │  │• Storage      │ │• CRUD     │ │Service        │ │• SMTP/IMAP    ││   │
│  │  │• Search       │ │• Workflow │ │• Routing      │ │• Templates    ││   │
│  │  │• Threads      │ │• SLA      │ │• Delivery     │ │• Tracking     ││   │
│  │  └───────────────┘ └───────────┘ └───────────────┘ └───────────────┘│   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                       DATA STORES                                     │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                   │   │
│  │  │ PostgreSQL  │  │   Redis     │  │ S3/MinIO    │                   │   │
│  │  │ • Messages  │  │ • Presence  │  │ • Files     │                   │   │
│  │  │ • Tickets   │  │ • PubSub    │  │ • Attachments│                  │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘                   │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Messaging-System

### 2.1 WebSocket Gateway

```python
# app/communication/websocket.py
"""WebSocket Gateway für Echtzeit-Kommunikation"""

import asyncio
import json
from typing import Dict, Set, Optional
from dataclasses import dataclass, field
from datetime import datetime
from fastapi import WebSocket, WebSocketDisconnect
import redis.asyncio as redis

@dataclass
class Connection:
    """WebSocket Verbindung"""
    websocket: WebSocket
    user_id: str
    organization_id: str
    connected_at: datetime = field(default_factory=datetime.utcnow)
    subscriptions: Set[str] = field(default_factory=set)

class WebSocketGateway:
    """WebSocket Gateway für alle Echtzeit-Kommunikation"""

    def __init__(self, redis_client: redis.Redis):
        self.redis = redis_client
        self.connections: Dict[str, Connection] = {}  # connection_id -> Connection
        self.user_connections: Dict[str, Set[str]] = {}  # user_id -> connection_ids

    async def handle_connection(self, websocket: WebSocket, user_id: str, org_id: str):
        """Behandelt neue WebSocket-Verbindung"""
        await websocket.accept()

        connection_id = f"{user_id}:{datetime.utcnow().timestamp()}"
        connection = Connection(
            websocket=websocket,
            user_id=user_id,
            organization_id=org_id
        )

        # Registrieren
        self.connections[connection_id] = connection
        if user_id not in self.user_connections:
            self.user_connections[user_id] = set()
        self.user_connections[user_id].add(connection_id)

        # Presence aktualisieren
        await self._set_user_online(user_id, org_id)

        # Standard-Subscriptions
        await self._subscribe(connection_id, f"user:{user_id}")
        await self._subscribe(connection_id, f"org:{org_id}")

        try:
            await self._message_loop(connection_id)
        except WebSocketDisconnect:
            await self._handle_disconnect(connection_id)
        finally:
            await self._cleanup(connection_id)

    async def _message_loop(self, connection_id: str):
        """Message Loop für eine Verbindung"""
        connection = self.connections[connection_id]

        while True:
            data = await connection.websocket.receive_json()
            await self._handle_message(connection_id, data)

    async def _handle_message(self, connection_id: str, message: dict):
        """Verarbeitet eingehende Nachricht"""
        message_type = message.get("type")
        payload = message.get("payload", {})

        handlers = {
            "subscribe": self._handle_subscribe,
            "unsubscribe": self._handle_unsubscribe,
            "send_message": self._handle_send_message,
            "typing": self._handle_typing,
            "read_receipt": self._handle_read_receipt,
            "ping": self._handle_ping,
        }

        handler = handlers.get(message_type)
        if handler:
            await handler(connection_id, payload)

    async def _handle_subscribe(self, connection_id: str, payload: dict):
        """Abonniert Channel"""
        channel = payload.get("channel")
        if channel:
            await self._subscribe(connection_id, channel)

    async def _handle_unsubscribe(self, connection_id: str, payload: dict):
        """Deabonniert Channel"""
        channel = payload.get("channel")
        if channel:
            await self._unsubscribe(connection_id, channel)

    async def _handle_send_message(self, connection_id: str, payload: dict):
        """Sendet Chat-Nachricht"""
        connection = self.connections[connection_id]

        message = {
            "id": str(uuid.uuid4()),
            "type": "chat_message",
            "channel": payload["channel"],
            "sender_id": connection.user_id,
            "content": payload["content"],
            "attachments": payload.get("attachments", []),
            "timestamp": datetime.utcnow().isoformat(),
            "metadata": payload.get("metadata", {})
        }

        # Nachricht speichern
        await self._store_message(message)

        # An alle Abonnenten senden
        await self._broadcast(payload["channel"], message)

    async def _handle_typing(self, connection_id: str, payload: dict):
        """Typing-Indikator"""
        connection = self.connections[connection_id]

        typing_event = {
            "type": "typing",
            "channel": payload["channel"],
            "user_id": connection.user_id,
            "is_typing": payload.get("is_typing", True)
        }

        await self._broadcast(payload["channel"], typing_event, exclude=[connection_id])

    async def _handle_read_receipt(self, connection_id: str, payload: dict):
        """Lesebestätigung"""
        connection = self.connections[connection_id]

        await self._mark_as_read(
            user_id=connection.user_id,
            channel=payload["channel"],
            message_id=payload["message_id"]
        )

        read_event = {
            "type": "read_receipt",
            "channel": payload["channel"],
            "user_id": connection.user_id,
            "message_id": payload["message_id"],
            "timestamp": datetime.utcnow().isoformat()
        }

        await self._broadcast(payload["channel"], read_event)

    async def _handle_ping(self, connection_id: str, payload: dict):
        """Ping/Pong für Keep-alive"""
        connection = self.connections[connection_id]
        await connection.websocket.send_json({"type": "pong"})

    # ========================================================================
    # Subscription Management
    # ========================================================================

    async def _subscribe(self, connection_id: str, channel: str):
        """Abonniert einen Channel"""
        connection = self.connections.get(connection_id)
        if not connection:
            return

        connection.subscriptions.add(channel)
        await self.redis.sadd(f"channel:{channel}:subscribers", connection_id)

    async def _unsubscribe(self, connection_id: str, channel: str):
        """Deabonniert einen Channel"""
        connection = self.connections.get(connection_id)
        if not connection:
            return

        connection.subscriptions.discard(channel)
        await self.redis.srem(f"channel:{channel}:subscribers", connection_id)

    # ========================================================================
    # Broadcasting
    # ========================================================================

    async def _broadcast(self, channel: str, message: dict, exclude: list = None):
        """Sendet Nachricht an alle Channel-Abonnenten"""
        exclude = exclude or []

        # Lokale Subscribers
        for conn_id, connection in self.connections.items():
            if conn_id in exclude:
                continue
            if channel in connection.subscriptions:
                try:
                    await connection.websocket.send_json(message)
                except Exception:
                    pass

        # Über Redis Pub/Sub für andere Server-Instanzen
        await self.redis.publish(
            f"broadcast:{channel}",
            json.dumps(message)
        )

    async def send_to_user(self, user_id: str, message: dict):
        """Sendet Nachricht an bestimmten User"""
        connection_ids = self.user_connections.get(user_id, set())

        for conn_id in connection_ids:
            connection = self.connections.get(conn_id)
            if connection:
                try:
                    await connection.websocket.send_json(message)
                except Exception:
                    pass

    # ========================================================================
    # Presence
    # ========================================================================

    async def _set_user_online(self, user_id: str, org_id: str):
        """Markiert User als online"""
        await self.redis.hset(
            f"presence:{org_id}",
            user_id,
            json.dumps({
                "status": "online",
                "last_seen": datetime.utcnow().isoformat()
            })
        )

        # Presence-Event broadcasten
        await self._broadcast(f"org:{org_id}", {
            "type": "presence",
            "user_id": user_id,
            "status": "online"
        })

    async def _set_user_offline(self, user_id: str, org_id: str):
        """Markiert User als offline"""
        await self.redis.hset(
            f"presence:{org_id}",
            user_id,
            json.dumps({
                "status": "offline",
                "last_seen": datetime.utcnow().isoformat()
            })
        )

        await self._broadcast(f"org:{org_id}", {
            "type": "presence",
            "user_id": user_id,
            "status": "offline"
        })

    async def get_online_users(self, org_id: str) -> list:
        """Gibt Liste der online User zurück"""
        presence_data = await self.redis.hgetall(f"presence:{org_id}")
        online_users = []

        for user_id, data_str in presence_data.items():
            data = json.loads(data_str)
            if data.get("status") == "online":
                online_users.append({
                    "user_id": user_id.decode() if isinstance(user_id, bytes) else user_id,
                    "last_seen": data.get("last_seen")
                })

        return online_users

    # ========================================================================
    # Storage
    # ========================================================================

    async def _store_message(self, message: dict):
        """Speichert Nachricht in DB"""
        # In PostgreSQL speichern
        await self.db.execute("""
            INSERT INTO messages (id, channel, sender_id, content, attachments, metadata, created_at)
            VALUES (:id, :channel, :sender, :content, :attachments, :metadata, :created_at)
        """, {
            "id": message["id"],
            "channel": message["channel"],
            "sender": message["sender_id"],
            "content": message["content"],
            "attachments": json.dumps(message.get("attachments", [])),
            "metadata": json.dumps(message.get("metadata", {})),
            "created_at": message["timestamp"]
        })

    async def _mark_as_read(self, user_id: str, channel: str, message_id: str):
        """Markiert Nachricht als gelesen"""
        await self.db.execute("""
            INSERT INTO message_reads (user_id, channel, message_id, read_at)
            VALUES (:user_id, :channel, :message_id, NOW())
            ON CONFLICT (user_id, message_id) DO NOTHING
        """, {
            "user_id": user_id,
            "channel": channel,
            "message_id": message_id
        })

    # ========================================================================
    # Cleanup
    # ========================================================================

    async def _handle_disconnect(self, connection_id: str):
        """Behandelt Disconnect"""
        connection = self.connections.get(connection_id)
        if not connection:
            return

        # User offline markieren wenn keine anderen Verbindungen
        self.user_connections[connection.user_id].discard(connection_id)
        if not self.user_connections[connection.user_id]:
            await self._set_user_offline(connection.user_id, connection.organization_id)

    async def _cleanup(self, connection_id: str):
        """Cleanup nach Disconnect"""
        connection = self.connections.pop(connection_id, None)
        if not connection:
            return

        # Subscriptions aufräumen
        for channel in connection.subscriptions:
            await self.redis.srem(f"channel:{channel}:subscribers", connection_id)
```

### 2.2 Message Service

```python
# app/communication/messages.py
"""Message Service für Chat-Funktionalität"""

from typing import List, Optional, Dict
from datetime import datetime
from dataclasses import dataclass
from enum import Enum

class ChannelType(Enum):
    DIRECT = "direct"  # 1:1
    GROUP = "group"    # Gruppen-Chat
    PROJECT = "project"  # Projekt-bezogen
    ORGANIZATION = "organization"  # Org-weit

@dataclass
class Channel:
    id: str
    type: ChannelType
    name: Optional[str]
    members: List[str]
    created_by: str
    created_at: datetime
    metadata: Dict = None

@dataclass
class Message:
    id: str
    channel_id: str
    sender_id: str
    content: str
    attachments: List[Dict]
    reply_to: Optional[str]
    created_at: datetime
    edited_at: Optional[datetime]
    is_deleted: bool = False

class MessageService:
    """Service für Nachrichten-Verwaltung"""

    def __init__(self, session, websocket_gateway, file_storage):
        self.session = session
        self.gateway = websocket_gateway
        self.storage = file_storage

    # ========================================================================
    # Channels
    # ========================================================================

    async def create_direct_channel(self, user1_id: str, user2_id: str) -> Channel:
        """Erstellt/findet Direct Channel zwischen zwei Usern"""

        # Prüfe ob Channel existiert
        existing = await self._find_direct_channel(user1_id, user2_id)
        if existing:
            return existing

        channel = Channel(
            id=str(uuid.uuid4()),
            type=ChannelType.DIRECT,
            name=None,
            members=[user1_id, user2_id],
            created_by=user1_id,
            created_at=datetime.utcnow()
        )

        await self._save_channel(channel)
        return channel

    async def create_group_channel(
        self,
        name: str,
        members: List[str],
        created_by: str,
        metadata: Dict = None
    ) -> Channel:
        """Erstellt Gruppen-Channel"""

        channel = Channel(
            id=str(uuid.uuid4()),
            type=ChannelType.GROUP,
            name=name,
            members=members,
            created_by=created_by,
            created_at=datetime.utcnow(),
            metadata=metadata
        )

        await self._save_channel(channel)

        # Benachrichtige Members
        for member_id in members:
            if member_id != created_by:
                await self.gateway.send_to_user(member_id, {
                    "type": "channel_created",
                    "channel": channel.__dict__
                })

        return channel

    async def create_project_channel(self, project_id: str, created_by: str) -> Channel:
        """Erstellt Channel für Projekt"""

        # Projekt-Members als Channel-Members
        project = await self._get_project(project_id)
        members = await self._get_project_members(project_id)

        channel = Channel(
            id=str(uuid.uuid4()),
            type=ChannelType.PROJECT,
            name=f"Projekt: {project.name}",
            members=[m.user_id for m in members],
            created_by=created_by,
            created_at=datetime.utcnow(),
            metadata={"project_id": project_id}
        )

        await self._save_channel(channel)
        return channel

    async def get_user_channels(self, user_id: str) -> List[Channel]:
        """Gibt alle Channels eines Users zurück"""

        result = await self.session.execute("""
            SELECT c.* FROM channels c
            JOIN channel_members cm ON c.id = cm.channel_id
            WHERE cm.user_id = :user_id
            ORDER BY c.last_message_at DESC NULLS LAST
        """, {"user_id": user_id})

        return [self._row_to_channel(row) for row in result.fetchall()]

    # ========================================================================
    # Messages
    # ========================================================================

    async def send_message(
        self,
        channel_id: str,
        sender_id: str,
        content: str,
        attachments: List[Dict] = None,
        reply_to: str = None
    ) -> Message:
        """Sendet Nachricht"""

        # Prüfe Membership
        if not await self._is_channel_member(channel_id, sender_id):
            raise PermissionError("Nicht Mitglied des Channels")

        message = Message(
            id=str(uuid.uuid4()),
            channel_id=channel_id,
            sender_id=sender_id,
            content=content,
            attachments=attachments or [],
            reply_to=reply_to,
            created_at=datetime.utcnow(),
            edited_at=None
        )

        # Speichern
        await self._save_message(message)

        # Channel aktualisieren
        await self._update_channel_activity(channel_id)

        # Über WebSocket senden
        await self.gateway._broadcast(
            f"channel:{channel_id}",
            {
                "type": "new_message",
                "message": message.__dict__
            }
        )

        # Push-Benachrichtigungen für offline User
        await self._send_push_notifications(channel_id, sender_id, content)

        return message

    async def edit_message(
        self,
        message_id: str,
        editor_id: str,
        new_content: str
    ) -> Message:
        """Bearbeitet Nachricht"""

        message = await self._get_message(message_id)

        if message.sender_id != editor_id:
            raise PermissionError("Nur eigene Nachrichten bearbeiten")

        message.content = new_content
        message.edited_at = datetime.utcnow()

        await self._update_message(message)

        # Update broadcasten
        await self.gateway._broadcast(
            f"channel:{message.channel_id}",
            {
                "type": "message_edited",
                "message": message.__dict__
            }
        )

        return message

    async def delete_message(self, message_id: str, deleter_id: str):
        """Löscht Nachricht (soft delete)"""

        message = await self._get_message(message_id)

        # Nur Sender oder Admin darf löschen
        if message.sender_id != deleter_id:
            # Prüfe Admin-Rechte
            if not await self._is_channel_admin(message.channel_id, deleter_id):
                raise PermissionError("Keine Berechtigung")

        message.is_deleted = True
        await self._update_message(message)

        await self.gateway._broadcast(
            f"channel:{message.channel_id}",
            {
                "type": "message_deleted",
                "message_id": message_id
            }
        )

    async def get_channel_messages(
        self,
        channel_id: str,
        user_id: str,
        before: str = None,
        limit: int = 50
    ) -> List[Message]:
        """Lädt Nachrichten eines Channels"""

        if not await self._is_channel_member(channel_id, user_id):
            raise PermissionError("Nicht Mitglied des Channels")

        query = """
            SELECT * FROM messages
            WHERE channel_id = :channel_id
            AND is_deleted = false
        """
        params = {"channel_id": channel_id, "limit": limit}

        if before:
            query += " AND created_at < (SELECT created_at FROM messages WHERE id = :before)"
            params["before"] = before

        query += " ORDER BY created_at DESC LIMIT :limit"

        result = await self.session.execute(query, params)
        return [self._row_to_message(row) for row in result.fetchall()]

    async def search_messages(
        self,
        user_id: str,
        query: str,
        channel_id: str = None,
        limit: int = 20
    ) -> List[Message]:
        """Sucht in Nachrichten"""

        # Nur in Channels suchen, in denen User Mitglied ist
        user_channels = await self.get_user_channels(user_id)
        channel_ids = [c.id for c in user_channels]

        if channel_id:
            if channel_id not in channel_ids:
                return []
            channel_ids = [channel_id]

        result = await self.session.execute("""
            SELECT m.* FROM messages m
            WHERE m.channel_id = ANY(:channel_ids)
            AND m.is_deleted = false
            AND m.content ILIKE :query
            ORDER BY m.created_at DESC
            LIMIT :limit
        """, {
            "channel_ids": channel_ids,
            "query": f"%{query}%",
            "limit": limit
        })

        return [self._row_to_message(row) for row in result.fetchall()]

    # ========================================================================
    # File Attachments
    # ========================================================================

    async def upload_attachment(
        self,
        channel_id: str,
        user_id: str,
        file_data: bytes,
        filename: str,
        content_type: str
    ) -> Dict:
        """Lädt Datei-Anhang hoch"""

        if not await self._is_channel_member(channel_id, user_id):
            raise PermissionError("Nicht Mitglied des Channels")

        # Datei speichern
        file_id = str(uuid.uuid4())
        file_path = f"channels/{channel_id}/attachments/{file_id}/{filename}"

        url = await self.storage.upload(file_path, file_data, content_type)

        return {
            "id": file_id,
            "filename": filename,
            "content_type": content_type,
            "size": len(file_data),
            "url": url
        }
```

---

## 3. Ticket-System

### 3.1 Ticket Service

```python
# app/communication/tickets.py
"""Ticket-System für Support und Bug-Reports"""

from typing import List, Optional, Dict
from datetime import datetime, timedelta
from dataclasses import dataclass, field
from enum import Enum

class TicketType(Enum):
    SUPPORT = "support"
    BUG = "bug"
    FEATURE_REQUEST = "feature_request"
    QUESTION = "question"
    FEEDBACK = "feedback"

class TicketPriority(Enum):
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"
    URGENT = "urgent"

class TicketStatus(Enum):
    NEW = "new"
    OPEN = "open"
    IN_PROGRESS = "in_progress"
    WAITING_FOR_USER = "waiting_for_user"
    RESOLVED = "resolved"
    CLOSED = "closed"

@dataclass
class Ticket:
    id: str
    type: TicketType
    priority: TicketPriority
    status: TicketStatus
    subject: str
    description: str
    reporter_id: str
    organization_id: str
    assignee_id: Optional[str]
    created_at: datetime
    updated_at: datetime
    resolved_at: Optional[datetime]
    tags: List[str] = field(default_factory=list)
    metadata: Dict = field(default_factory=dict)

@dataclass
class TicketComment:
    id: str
    ticket_id: str
    author_id: str
    content: str
    is_internal: bool  # Nur für Support-Team sichtbar
    created_at: datetime
    attachments: List[Dict] = field(default_factory=list)

class TicketService:
    """Service für Ticket-Verwaltung"""

    # SLA-Definitionen
    SLA_RESPONSE_TIMES = {
        TicketPriority.URGENT: timedelta(hours=1),
        TicketPriority.HIGH: timedelta(hours=4),
        TicketPriority.MEDIUM: timedelta(hours=24),
        TicketPriority.LOW: timedelta(hours=72),
    }

    SLA_RESOLUTION_TIMES = {
        TicketPriority.URGENT: timedelta(hours=4),
        TicketPriority.HIGH: timedelta(hours=24),
        TicketPriority.MEDIUM: timedelta(days=3),
        TicketPriority.LOW: timedelta(days=7),
    }

    def __init__(self, session, notification_service, email_service):
        self.session = session
        self.notifications = notification_service
        self.email = email_service

    # ========================================================================
    # Ticket CRUD
    # ========================================================================

    async def create_ticket(
        self,
        reporter_id: str,
        organization_id: str,
        type: TicketType,
        subject: str,
        description: str,
        priority: TicketPriority = TicketPriority.MEDIUM,
        attachments: List[Dict] = None,
        metadata: Dict = None
    ) -> Ticket:
        """Erstellt neues Ticket"""

        ticket = Ticket(
            id=str(uuid.uuid4()),
            type=type,
            priority=priority,
            status=TicketStatus.NEW,
            subject=subject,
            description=description,
            reporter_id=reporter_id,
            organization_id=organization_id,
            assignee_id=None,
            created_at=datetime.utcnow(),
            updated_at=datetime.utcnow(),
            resolved_at=None,
            tags=self._auto_tag(subject, description),
            metadata=metadata or {}
        )

        # Ticket-Nummer generieren
        ticket.metadata["ticket_number"] = await self._generate_ticket_number()

        await self._save_ticket(ticket)

        # Initialen Kommentar mit Beschreibung erstellen
        await self.add_comment(
            ticket_id=ticket.id,
            author_id=reporter_id,
            content=description,
            is_internal=False,
            attachments=attachments
        )

        # Auto-Assignment basierend auf Typ/Tags
        assignee = await self._auto_assign(ticket)
        if assignee:
            await self.assign_ticket(ticket.id, assignee, "system")

        # Benachrichtigungen
        await self._notify_new_ticket(ticket)

        return ticket

    async def update_ticket(
        self,
        ticket_id: str,
        updater_id: str,
        updates: Dict
    ) -> Ticket:
        """Aktualisiert Ticket"""

        ticket = await self.get_ticket(ticket_id)
        old_status = ticket.status

        # Erlaubte Felder
        allowed_fields = ["priority", "status", "subject", "tags", "assignee_id"]

        for field, value in updates.items():
            if field in allowed_fields:
                setattr(ticket, field, value)

        ticket.updated_at = datetime.utcnow()

        # Status-Änderung behandeln
        if "status" in updates and updates["status"] != old_status:
            await self._handle_status_change(ticket, old_status, updates["status"], updater_id)

        await self._update_ticket(ticket)

        return ticket

    async def assign_ticket(
        self,
        ticket_id: str,
        assignee_id: str,
        assigned_by: str
    ) -> Ticket:
        """Weist Ticket zu"""

        ticket = await self.get_ticket(ticket_id)
        old_assignee = ticket.assignee_id

        ticket.assignee_id = assignee_id
        ticket.updated_at = datetime.utcnow()

        if ticket.status == TicketStatus.NEW:
            ticket.status = TicketStatus.OPEN

        await self._update_ticket(ticket)

        # Benachrichtigung
        if assignee_id != old_assignee:
            await self.notifications.notify_user(
                user_id=assignee_id,
                notification_type="ticket_assigned",
                data={
                    "ticket_id": ticket_id,
                    "ticket_number": ticket.metadata.get("ticket_number"),
                    "subject": ticket.subject,
                    "assigned_by": assigned_by
                }
            )

        # Audit Log
        await self._log_ticket_activity(
            ticket_id, "assigned",
            {"old_assignee": old_assignee, "new_assignee": assignee_id},
            assigned_by
        )

        return ticket

    async def close_ticket(
        self,
        ticket_id: str,
        closed_by: str,
        resolution: str = None
    ) -> Ticket:
        """Schließt Ticket"""

        ticket = await self.get_ticket(ticket_id)

        ticket.status = TicketStatus.CLOSED
        ticket.resolved_at = datetime.utcnow()
        ticket.updated_at = datetime.utcnow()

        if resolution:
            ticket.metadata["resolution"] = resolution

        await self._update_ticket(ticket)

        # Zufriedenheits-Umfrage senden
        await self._send_satisfaction_survey(ticket)

        return ticket

    async def get_ticket(self, ticket_id: str) -> Ticket:
        """Lädt Ticket"""
        result = await self.session.execute("""
            SELECT * FROM tickets WHERE id = :id
        """, {"id": ticket_id})
        row = result.fetchone()
        if not row:
            raise ValueError(f"Ticket {ticket_id} nicht gefunden")
        return self._row_to_ticket(row)

    async def list_tickets(
        self,
        organization_id: str = None,
        reporter_id: str = None,
        assignee_id: str = None,
        status: List[TicketStatus] = None,
        type: TicketType = None,
        limit: int = 50,
        offset: int = 0
    ) -> List[Ticket]:
        """Listet Tickets mit Filtern"""

        query = "SELECT * FROM tickets WHERE 1=1"
        params = {"limit": limit, "offset": offset}

        if organization_id:
            query += " AND organization_id = :org_id"
            params["org_id"] = organization_id

        if reporter_id:
            query += " AND reporter_id = :reporter_id"
            params["reporter_id"] = reporter_id

        if assignee_id:
            query += " AND assignee_id = :assignee_id"
            params["assignee_id"] = assignee_id

        if status:
            query += " AND status = ANY(:status)"
            params["status"] = [s.value for s in status]

        if type:
            query += " AND type = :type"
            params["type"] = type.value

        query += " ORDER BY priority DESC, created_at DESC LIMIT :limit OFFSET :offset"

        result = await self.session.execute(query, params)
        return [self._row_to_ticket(row) for row in result.fetchall()]

    # ========================================================================
    # Comments
    # ========================================================================

    async def add_comment(
        self,
        ticket_id: str,
        author_id: str,
        content: str,
        is_internal: bool = False,
        attachments: List[Dict] = None
    ) -> TicketComment:
        """Fügt Kommentar hinzu"""

        comment = TicketComment(
            id=str(uuid.uuid4()),
            ticket_id=ticket_id,
            author_id=author_id,
            content=content,
            is_internal=is_internal,
            created_at=datetime.utcnow(),
            attachments=attachments or []
        )

        await self._save_comment(comment)

        # Ticket aktualisieren
        ticket = await self.get_ticket(ticket_id)
        ticket.updated_at = datetime.utcnow()

        # Status ändern wenn nötig
        if ticket.status == TicketStatus.WAITING_FOR_USER and author_id == ticket.reporter_id:
            ticket.status = TicketStatus.OPEN

        await self._update_ticket(ticket)

        # Benachrichtigungen (nicht für interne Kommentare an Reporter)
        if not is_internal or author_id != ticket.reporter_id:
            await self._notify_comment(ticket, comment)

        return comment

    async def get_comments(
        self,
        ticket_id: str,
        include_internal: bool = False
    ) -> List[TicketComment]:
        """Lädt Kommentare eines Tickets"""

        query = """
            SELECT * FROM ticket_comments
            WHERE ticket_id = :ticket_id
        """

        if not include_internal:
            query += " AND is_internal = false"

        query += " ORDER BY created_at ASC"

        result = await self.session.execute(query, {"ticket_id": ticket_id})
        return [self._row_to_comment(row) for row in result.fetchall()]

    # ========================================================================
    # SLA Tracking
    # ========================================================================

    async def check_sla_breaches(self) -> List[Dict]:
        """Prüft SLA-Verletzungen"""

        now = datetime.utcnow()
        breaches = []

        # Offene Tickets laden
        open_tickets = await self.list_tickets(
            status=[TicketStatus.NEW, TicketStatus.OPEN, TicketStatus.IN_PROGRESS]
        )

        for ticket in open_tickets:
            # Response SLA
            response_deadline = ticket.created_at + self.SLA_RESPONSE_TIMES[ticket.priority]
            if now > response_deadline:
                # Prüfe ob schon geantwortet wurde
                comments = await self.get_comments(ticket.id)
                has_response = any(c.author_id != ticket.reporter_id for c in comments)

                if not has_response:
                    breaches.append({
                        "ticket_id": ticket.id,
                        "ticket_number": ticket.metadata.get("ticket_number"),
                        "breach_type": "response",
                        "deadline": response_deadline,
                        "overdue_by": now - response_deadline
                    })

            # Resolution SLA
            resolution_deadline = ticket.created_at + self.SLA_RESOLUTION_TIMES[ticket.priority]
            if now > resolution_deadline:
                breaches.append({
                    "ticket_id": ticket.id,
                    "ticket_number": ticket.metadata.get("ticket_number"),
                    "breach_type": "resolution",
                    "deadline": resolution_deadline,
                    "overdue_by": now - resolution_deadline
                })

        return breaches

    async def get_sla_status(self, ticket_id: str) -> Dict:
        """Gibt SLA-Status für Ticket zurück"""

        ticket = await self.get_ticket(ticket_id)
        now = datetime.utcnow()

        response_deadline = ticket.created_at + self.SLA_RESPONSE_TIMES[ticket.priority]
        resolution_deadline = ticket.created_at + self.SLA_RESOLUTION_TIMES[ticket.priority]

        # Response Status
        comments = await self.get_comments(ticket.id)
        first_response = next(
            (c for c in comments if c.author_id != ticket.reporter_id),
            None
        )

        response_status = {
            "deadline": response_deadline.isoformat(),
            "met": first_response is not None and first_response.created_at <= response_deadline,
            "response_time": (first_response.created_at - ticket.created_at).total_seconds() if first_response else None
        }

        # Resolution Status
        resolution_status = {
            "deadline": resolution_deadline.isoformat(),
            "met": ticket.resolved_at is not None and ticket.resolved_at <= resolution_deadline if ticket.resolved_at else None,
            "resolution_time": (ticket.resolved_at - ticket.created_at).total_seconds() if ticket.resolved_at else None
        }

        return {
            "ticket_id": ticket_id,
            "priority": ticket.priority.value,
            "response": response_status,
            "resolution": resolution_status
        }

    # ========================================================================
    # Auto-Assignment
    # ========================================================================

    async def _auto_assign(self, ticket: Ticket) -> Optional[str]:
        """Automatische Zuweisung basierend auf Regeln"""

        # Regeln nach Typ
        type_assignees = {
            TicketType.BUG: await self._get_available_developer(),
            TicketType.SUPPORT: await self._get_available_support(),
            TicketType.FEATURE_REQUEST: None,  # Manuell
        }

        assignee = type_assignees.get(ticket.type)

        # Tag-basierte Zuweisung
        if not assignee and ticket.tags:
            assignee = await self._get_assignee_by_tags(ticket.tags)

        return assignee

    async def _get_available_support(self) -> Optional[str]:
        """Findet verfügbaren Support-Mitarbeiter"""
        # Round-Robin oder Load-Balancing
        result = await self.session.execute("""
            SELECT u.id, COUNT(t.id) as open_tickets
            FROM users u
            LEFT JOIN tickets t ON t.assignee_id = u.id
                AND t.status NOT IN ('resolved', 'closed')
            WHERE u.role = 'support'
            GROUP BY u.id
            ORDER BY open_tickets ASC
            LIMIT 1
        """)
        row = result.fetchone()
        return row.id if row else None

    def _auto_tag(self, subject: str, description: str) -> List[str]:
        """Automatisches Tagging basierend auf Inhalt"""
        tags = []
        text = f"{subject} {description}".lower()

        tag_keywords = {
            "berechnung": ["u-wert", "heizlast", "energiebilanz", "berechnung"],
            "bericht": ["bericht", "report", "isfp", "energieausweis"],
            "fehler": ["fehler", "error", "bug", "absturz", "crash"],
            "login": ["login", "anmeldung", "passwort", "authentifizierung"],
            "performance": ["langsam", "performance", "laden", "timeout"],
        }

        for tag, keywords in tag_keywords.items():
            if any(kw in text for kw in keywords):
                tags.append(tag)

        return tags
```

---

## 4. Datenbank-Schema

```sql
-- Kommunikations-bezogene Tabellen

-- Channels
CREATE TABLE channels (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    type VARCHAR(50) NOT NULL,
    name VARCHAR(255),
    created_by UUID REFERENCES users(id),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    last_message_at TIMESTAMPTZ,
    metadata JSONB DEFAULT '{}',

    INDEX idx_channels_type (type)
);

-- Channel Members
CREATE TABLE channel_members (
    channel_id UUID REFERENCES channels(id) ON DELETE CASCADE,
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    joined_at TIMESTAMPTZ DEFAULT NOW(),
    role VARCHAR(50) DEFAULT 'member',
    notifications_enabled BOOLEAN DEFAULT TRUE,

    PRIMARY KEY (channel_id, user_id)
);

-- Messages
CREATE TABLE messages (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    channel_id UUID REFERENCES channels(id) ON DELETE CASCADE,
    sender_id UUID REFERENCES users(id),
    content TEXT NOT NULL,
    attachments JSONB DEFAULT '[]',
    reply_to UUID REFERENCES messages(id),
    is_deleted BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    edited_at TIMESTAMPTZ,

    INDEX idx_messages_channel_time (channel_id, created_at)
);

-- Message Reads
CREATE TABLE message_reads (
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    message_id UUID REFERENCES messages(id) ON DELETE CASCADE,
    read_at TIMESTAMPTZ DEFAULT NOW(),

    PRIMARY KEY (user_id, message_id)
);

-- Tickets
CREATE TABLE tickets (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    type VARCHAR(50) NOT NULL,
    priority VARCHAR(20) NOT NULL,
    status VARCHAR(50) NOT NULL DEFAULT 'new',
    subject VARCHAR(500) NOT NULL,
    description TEXT,
    reporter_id UUID REFERENCES users(id),
    organization_id UUID REFERENCES organizations(id),
    assignee_id UUID REFERENCES users(id),
    tags TEXT[] DEFAULT '{}',
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    resolved_at TIMESTAMPTZ,

    INDEX idx_tickets_org_status (organization_id, status),
    INDEX idx_tickets_assignee (assignee_id, status)
);

-- Ticket Comments
CREATE TABLE ticket_comments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    ticket_id UUID REFERENCES tickets(id) ON DELETE CASCADE,
    author_id UUID REFERENCES users(id),
    content TEXT NOT NULL,
    is_internal BOOLEAN DEFAULT FALSE,
    attachments JSONB DEFAULT '[]',
    created_at TIMESTAMPTZ DEFAULT NOW(),

    INDEX idx_ticket_comments (ticket_id, created_at)
);

-- Ticket Activity Log
CREATE TABLE ticket_activity_log (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    ticket_id UUID REFERENCES tickets(id) ON DELETE CASCADE,
    action VARCHAR(100) NOT NULL,
    details JSONB,
    performed_by UUID REFERENCES users(id),
    performed_at TIMESTAMPTZ DEFAULT NOW(),

    INDEX idx_ticket_activity (ticket_id, performed_at)
);
```

---

## 5. Checkliste

```
✓ WebSocket Gateway für Echtzeit-Kommunikation
✓ Message Service mit Channel-Support
✓ Direct Messages (1:1)
✓ Gruppen-Chats
✓ Projekt-bezogene Channels
✓ Presence-Tracking (online/offline)
✓ Typing-Indikatoren
✓ Lesebestätigungen
✓ File-Attachments
✓ Ticket-System mit CRUD
✓ Ticket-Kommentare
✓ SLA-Tracking
✓ Auto-Assignment
✓ Auto-Tagging
✓ Datenbank-Schema
```

---

*Letzte Aktualisierung: 2025-11-25*
*Version: 1.0.0*
