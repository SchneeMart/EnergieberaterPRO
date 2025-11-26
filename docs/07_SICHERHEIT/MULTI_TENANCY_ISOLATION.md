# Multi-Tenancy mit strikter Datenisolation

## Referenzen
- [AWS Multi-Tenant Isolation](https://aws.amazon.com/blogs/database/multi-tenant-data-isolation-with-postgresql-row-level-security/)
- [PostgreSQL RLS Best Practices](https://www.crunchydata.com/blog/row-level-security-for-tenants-in-postgres)
- [Strict Data Isolation Patterns](https://medium.com/@moyo.sore.oluwa/strict-data-isolation-in-multitenant-systems-with-postgresql-aa615052fe80)

---

## 1. Architekturübersicht

### 1.1 Isolationsebenen

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     MULTI-TENANCY ISOLATION LAYERS                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  EBENE 5: NETZWERK                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  • Separate Docker Networks                                          │   │
│  │  • Interne Services nicht von außen erreichbar                      │   │
│  │  • TLS für alle Verbindungen                                         │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  EBENE 4: APPLICATION LAYER                                                │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  • JWT mit Organization ID                                           │   │
│  │  • Middleware setzt Tenant-Context                                  │   │
│  │  • API-Level Access Control                                          │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  EBENE 3: SERVICE LAYER                                                    │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  • Explizite Organization-Prüfung                                    │   │
│  │  • Keine Cross-Tenant Queries                                        │   │
│  │  • Tenant-aware Caching                                              │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  EBENE 2: DATABASE LAYER (PostgreSQL RLS)                                  │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  • Row Level Security auf ALLEN Tabellen                            │   │
│  │  • Policies für SELECT, INSERT, UPDATE, DELETE                       │   │
│  │  • Unbypassbare letzte Verteidigungslinie                           │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  EBENE 1: ENCRYPTION AT REST                                               │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  • Separate Encryption Keys pro Tenant (optional)                    │   │
│  │  • Verschlüsselte sensible Spalten                                   │   │
│  │  • Verschlüsselte Backups                                            │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Isolationsgarantien

```
GARANTIE 1: Kein Tenant kann Daten eines anderen Tenants sehen
GARANTIE 2: Kein Tenant kann Daten eines anderen Tenants modifizieren
GARANTIE 3: Auch bei Anwendungsfehlern greift DB-Level-Schutz
GARANTIE 4: Admins können nur auf ihren Tenant zugreifen
GARANTIE 5: API-Keys sind Tenant-gebunden
```

---

## 2. PostgreSQL Row Level Security

### 2.1 Grundkonfiguration

```sql
-- Tenant-Context Variable
-- Wird bei jeder Request-Verarbeitung gesetzt
SET app.current_org = 'org-uuid-here';
SET app.current_user = 'user-uuid-here';

-- WICHTIG: Separater DB-User für die Anwendung
-- (Nicht der Table Owner - sonst wird RLS umgangen)
CREATE USER app_user WITH PASSWORD 'secure_password';
GRANT CONNECT ON DATABASE energieberater TO app_user;
GRANT USAGE ON SCHEMA public TO app_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_user;
```

### 2.2 RLS Policies für alle Tenant-Tabellen

```sql
-- ============================================
-- ORGANIZATIONS (Basis-Tabelle)
-- ============================================
CREATE TABLE organizations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    settings JSONB DEFAULT '{}',
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Keine RLS auf organizations selbst (wird über User gesteuert)

-- ============================================
-- USERS
-- ============================================
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
    email VARCHAR(255) UNIQUE NOT NULL,
    -- ...
);

ALTER TABLE users ENABLE ROW LEVEL SECURITY;
ALTER TABLE users FORCE ROW LEVEL SECURITY;

-- Policy: User sieht nur eigene Organisation
CREATE POLICY users_org_isolation ON users
    USING (organization_id = current_setting('app.current_org')::uuid);

-- ============================================
-- PROJECTS
-- ============================================
CREATE TABLE projects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    -- ...
);

ALTER TABLE projects ENABLE ROW LEVEL SECURITY;
ALTER TABLE projects FORCE ROW LEVEL SECURITY;

-- SELECT Policy
CREATE POLICY projects_select ON projects
    FOR SELECT
    USING (organization_id = current_setting('app.current_org')::uuid);

-- INSERT Policy - verhindert Einfügen für andere Orgs
CREATE POLICY projects_insert ON projects
    FOR INSERT
    WITH CHECK (organization_id = current_setting('app.current_org')::uuid);

-- UPDATE Policy
CREATE POLICY projects_update ON projects
    FOR UPDATE
    USING (organization_id = current_setting('app.current_org')::uuid)
    WITH CHECK (organization_id = current_setting('app.current_org')::uuid);

-- DELETE Policy
CREATE POLICY projects_delete ON projects
    FOR DELETE
    USING (organization_id = current_setting('app.current_org')::uuid);

-- ============================================
-- BUILDINGS (über Project verbunden)
-- ============================================
CREATE TABLE buildings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    location_id UUID NOT NULL REFERENCES locations(id) ON DELETE CASCADE,
    organization_id UUID NOT NULL REFERENCES organizations(id),
    -- ...
);

ALTER TABLE buildings ENABLE ROW LEVEL SECURITY;
ALTER TABLE buildings FORCE ROW LEVEL SECURITY;

CREATE POLICY buildings_isolation ON buildings
    USING (organization_id = current_setting('app.current_org')::uuid);

-- ============================================
-- TEMPLATE für alle weiteren Tabellen
-- ============================================
-- Jede Tabelle mit Tenant-Daten MUSS:
-- 1. organization_id Spalte haben
-- 2. RLS aktiviert haben
-- 3. FORCE RLS aktiviert haben (auch für Table Owner)
-- 4. Policies für alle Operationen haben
```

### 2.3 RLS für hierarchische Daten

```sql
-- Für Tabellen die über Parent-Child indirekt zum Tenant gehören
-- Beispiel: zones -> buildings -> locations -> projects -> organization

CREATE TABLE zones (
    id UUID PRIMARY KEY,
    building_id UUID REFERENCES buildings(id) ON DELETE CASCADE,
    organization_id UUID NOT NULL REFERENCES organizations(id),
    -- WICHTIG: Redundante org_id für effiziente RLS
    -- ...
);

-- Alternative: Join-basierte Policy (langsamer)
CREATE POLICY zones_via_building ON zones
    USING (
        EXISTS (
            SELECT 1 FROM buildings b
            WHERE b.id = zones.building_id
            AND b.organization_id = current_setting('app.current_org')::uuid
        )
    );

-- Bessere Alternative: Direkte org_id (schneller)
CREATE POLICY zones_direct ON zones
    USING (organization_id = current_setting('app.current_org')::uuid);
```

### 2.4 Restrictive Policies (AND statt OR)

```sql
-- Standard: Permissive Policies werden mit OR kombiniert
-- Manchmal brauchen wir AND-Kombination

-- Beispiel: User muss SOWOHL in richtiger Org sein
-- ALS AUCH die richtige Rolle haben

-- Policy 1: Org-Isolation (Permissive)
CREATE POLICY projects_org ON projects
    FOR SELECT
    USING (organization_id = current_setting('app.current_org')::uuid);

-- Policy 2: Zusätzliche Einschränkung (Restrictive)
CREATE POLICY projects_role ON projects
    AS RESTRICTIVE
    FOR SELECT
    USING (
        EXISTS (
            SELECT 1 FROM user_roles ur
            WHERE ur.user_id = current_setting('app.current_user')::uuid
            AND ur.role_name IN ('admin', 'manager', 'consultant')
        )
    );

-- Ergebnis: Benutzer muss BEIDE Policies erfüllen
```

---

## 3. Application Layer Integration

### 3.1 Middleware für Tenant-Context

```python
# middleware/tenant.py
from fastapi import Request
from sqlalchemy import text

class TenantMiddleware:
    """Setzt Tenant-Context für jede Request"""

    async def __call__(self, request: Request, call_next):
        # Token validieren und User extrahieren
        token_payload = await validate_token(request)

        if token_payload:
            # Tenant-Context in Request-State speichern
            request.state.organization_id = token_payload.org
            request.state.user_id = token_payload.sub

        response = await call_next(request)
        return response


class DatabaseTenantContext:
    """Context Manager für DB-Session mit RLS"""

    def __init__(self, session, organization_id: str, user_id: str):
        self.session = session
        self.org_id = organization_id
        self.user_id = user_id

    async def __aenter__(self):
        # Setze PostgreSQL Session-Variablen
        await self.session.execute(
            text("SET app.current_org = :org"),
            {"org": str(self.org_id)}
        )
        await self.session.execute(
            text("SET app.current_user = :user"),
            {"user": str(self.user_id)}
        )
        return self.session

    async def __aexit__(self, *args):
        # Reset für Connection Pool
        await self.session.execute(text("RESET app.current_org"))
        await self.session.execute(text("RESET app.current_user"))
```

### 3.2 Dependency Injection

```python
# dependencies/database.py
from fastapi import Depends, Request
from sqlalchemy.ext.asyncio import AsyncSession

async def get_tenant_db(
    request: Request,
    session: AsyncSession = Depends(get_session)
) -> AsyncSession:
    """
    Gibt DB-Session mit gesetztem Tenant-Context zurück.
    RLS ist damit aktiv.
    """
    org_id = getattr(request.state, 'organization_id', None)
    user_id = getattr(request.state, 'user_id', None)

    if not org_id:
        raise HTTPException(401, "Not authenticated")

    async with DatabaseTenantContext(session, org_id, user_id) as tenant_session:
        yield tenant_session


# Verwendung in Endpoints
@router.get("/projects")
async def list_projects(
    db: AsyncSession = Depends(get_tenant_db)
):
    # Query ist automatisch auf Tenant gefiltert durch RLS
    result = await db.execute(select(Project))
    return result.scalars().all()
```

### 3.3 Explizite Service-Layer-Prüfung (Defense in Depth)

```python
# services/project_service.py
class ProjectService:
    """Service mit expliziter Tenant-Prüfung"""

    def __init__(self, db: AsyncSession, current_org: UUID, current_user: UUID):
        self.db = db
        self.current_org = current_org
        self.current_user = current_user

    async def get(self, project_id: UUID) -> Project:
        """Holt Projekt mit expliziter Org-Prüfung"""
        project = await self.db.get(Project, project_id)

        # Explizite Prüfung (zusätzlich zu RLS)
        if project and project.organization_id != self.current_org:
            # Sollte nie passieren wegen RLS
            # Aber: Defense in Depth
            logger.error(
                f"RLS bypass attempt: User {self.current_user} "
                f"tried to access project {project_id} "
                f"from org {project.organization_id}"
            )
            raise ForbiddenError()

        return project

    async def create(self, data: ProjectCreate) -> Project:
        """Erstellt Projekt immer in aktueller Org"""
        project = Project(
            organization_id=self.current_org,  # Erzwungen
            created_by=self.current_user,
            **data.dict()
        )
        self.db.add(project)
        await self.db.commit()
        return project
```

---

## 4. Tenant-aware Caching

```python
# cache/tenant_cache.py
from redis.asyncio import Redis

class TenantAwareCache:
    """Cache mit Tenant-Isolation"""

    def __init__(self, redis: Redis):
        self.redis = redis

    def _key(self, org_id: str, key: str) -> str:
        """Tenant-prefixed Cache Key"""
        return f"tenant:{org_id}:{key}"

    async def get(self, org_id: str, key: str) -> bytes | None:
        return await self.redis.get(self._key(org_id, key))

    async def set(
        self,
        org_id: str,
        key: str,
        value: bytes,
        ttl: int = 3600
    ):
        await self.redis.setex(
            self._key(org_id, key),
            ttl,
            value
        )

    async def delete_tenant_cache(self, org_id: str):
        """Löscht allen Cache eines Tenants"""
        pattern = f"tenant:{org_id}:*"
        async for key in self.redis.scan_iter(pattern):
            await self.redis.delete(key)
```

---

## 5. Audit Trail pro Tenant

```sql
-- Audit Log mit Tenant-Isolation
CREATE TABLE audit_log (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID NOT NULL REFERENCES organizations(id),
    user_id UUID REFERENCES users(id),
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(100),
    resource_id UUID,
    old_data JSONB,
    new_data JSONB,
    ip_address INET,
    timestamp TIMESTAMPTZ DEFAULT NOW()
);

-- RLS auch auf Audit Log!
ALTER TABLE audit_log ENABLE ROW LEVEL SECURITY;
ALTER TABLE audit_log FORCE ROW LEVEL SECURITY;

CREATE POLICY audit_isolation ON audit_log
    USING (organization_id = current_setting('app.current_org')::uuid);

-- Index für Performance
CREATE INDEX idx_audit_org_time ON audit_log (organization_id, timestamp DESC);
```

---

## 6. File Storage Isolation

```python
# storage/tenant_storage.py
class TenantFileStorage:
    """Dateispeicher mit Tenant-Isolation"""

    def __init__(self, minio_client, bucket: str):
        self.client = minio_client
        self.bucket = bucket

    def _path(self, org_id: str, path: str) -> str:
        """Tenant-prefixed Pfad"""
        return f"{org_id}/{path}"

    async def upload(
        self,
        org_id: str,
        filename: str,
        data: bytes
    ) -> str:
        """Upload mit Tenant-Präfix"""
        path = self._path(org_id, filename)
        await self.client.put_object(
            self.bucket,
            path,
            data
        )
        return path

    async def download(
        self,
        org_id: str,
        path: str
    ) -> bytes:
        """Download mit Tenant-Prüfung"""
        full_path = self._path(org_id, path)

        # Verhindere Path Traversal
        if '..' in path or not full_path.startswith(f"{org_id}/"):
            raise ForbiddenError("Invalid path")

        return await self.client.get_object(self.bucket, full_path)
```

---

## 7. Testing der Isolation

```python
# tests/test_tenant_isolation.py
import pytest
from uuid import uuid4

@pytest.fixture
async def org_a(db):
    """Organisation A"""
    return await create_organization(db, name="Org A")

@pytest.fixture
async def org_b(db):
    """Organisation B"""
    return await create_organization(db, name="Org B")

@pytest.fixture
async def project_in_org_a(db, org_a):
    """Projekt in Org A"""
    async with tenant_context(db, org_a.id):
        return await create_project(db, name="Project A")

class TestTenantIsolation:
    """Tests für Mandantentrennung"""

    async def test_org_b_cannot_see_org_a_projects(
        self, db, org_a, org_b, project_in_org_a
    ):
        """Org B kann Projekte von Org A nicht sehen"""
        async with tenant_context(db, org_b.id):
            projects = await db.execute(select(Project))
            assert len(projects.scalars().all()) == 0

    async def test_org_b_cannot_access_org_a_project_directly(
        self, db, org_a, org_b, project_in_org_a
    ):
        """Org B kann Projekt von Org A nicht direkt abrufen"""
        async with tenant_context(db, org_b.id):
            result = await db.execute(
                select(Project).where(Project.id == project_in_org_a.id)
            )
            assert result.scalar_one_or_none() is None

    async def test_org_b_cannot_insert_into_org_a(self, db, org_a, org_b):
        """Org B kann nicht in Org A einfügen"""
        async with tenant_context(db, org_b.id):
            with pytest.raises(Exception):
                # Versuch, Projekt mit falscher org_id einzufügen
                project = Project(
                    organization_id=org_a.id,  # Falsche Org!
                    name="Hack attempt"
                )
                db.add(project)
                await db.commit()

    async def test_rls_blocks_sql_injection_attempt(self, db, org_a, org_b):
        """RLS blockt auch SQL Injection Versuche"""
        async with tenant_context(db, org_b.id):
            # Selbst wenn jemand die Query manipulieren könnte
            result = await db.execute(
                text("SELECT * FROM projects WHERE '1'='1'")
            )
            # RLS filtert trotzdem nach current_org
            assert len(result.fetchall()) == 0
```

---

## 8. Schema-per-Tenant (Optional für höchste Sicherheit)

```sql
-- Für hochsensible Daten: Kombination Schema + RLS

-- Schema pro Tenant erstellen
CREATE SCHEMA tenant_abc123;

-- Tabellen im Tenant-Schema
CREATE TABLE tenant_abc123.sensitive_data (
    id UUID PRIMARY KEY,
    data TEXT
);

-- RLS zusätzlich als zweite Ebene
ALTER TABLE tenant_abc123.sensitive_data ENABLE ROW LEVEL SECURITY;

-- Anwendung setzt search_path dynamisch
SET search_path TO tenant_abc123, public;
```

---

## Quellen

- [AWS Multi-Tenant Isolation](https://aws.amazon.com/blogs/database/multi-tenant-data-isolation-with-postgresql-row-level-security/)
- [Crunchy Data RLS Guide](https://www.crunchydata.com/blog/row-level-security-for-tenants-in-postgres)
- [Leapcell Multi-Tenant Guide](https://leapcell.io/blog/achieving-robust-multi-tenant-data-isolation-with-postgresql-row-level-security)
- [Permit.io RLS Implementation](https://www.permit.io/blog/postgres-rls-implementation-guide)

---

*Letzte Aktualisierung: 2025-11-25*
*Version: 1.0.0*
