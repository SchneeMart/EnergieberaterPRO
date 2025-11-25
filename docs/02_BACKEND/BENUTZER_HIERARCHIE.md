# Benutzer-Hierarchie und Berechtigungssystem

## Referenzen
- [RBAC Best Practices](https://csrc.nist.gov/projects/role-based-access-control)
- [Multi-Tenant Authorization](https://auth0.com/docs/architecture-scenarios/multiple-tenants)

---

## 1. Ebenen-Übersicht

### 1.1 Drei-Ebenen-Hierarchie

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    BENUTZER-HIERARCHIE                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                    ┌─────────────────────────────────┐                      │
│                    │         SYSADMIN                │  ← Plattform-Ebene   │
│                    │   (System-Administrator)        │                      │
│                    │                                 │                      │
│                    │   • Vollzugriff auf ALLES      │                      │
│                    │   • Alle Organisationen        │                      │
│                    │   • System-Konfiguration       │                      │
│                    │   • Billing aller Orgs         │                      │
│                    │   • Support-Zugang             │                      │
│                    └───────────────┬─────────────────┘                      │
│                                    │                                        │
│           ┌────────────────────────┼────────────────────────┐              │
│           │                        │                        │              │
│           ▼                        ▼                        ▼              │
│  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐       │
│  │  ORGANISATION A │     │  ORGANISATION B │     │  ORGANISATION C │       │
│  └────────┬────────┘     └────────┬────────┘     └────────┬────────┘       │
│           │                       │                       │                │
│           ▼                       ▼                       ▼                │
│  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐       │
│  │   ORG-ADMIN     │     │   ORG-ADMIN     │     │   ORG-ADMIN     │  ← Org│
│  │                 │     │                 │     │                 │   Ebene│
│  │ • Org-Einstellg.│     │ • Org-Einstellg.│     │ • Org-Einstellg.│       │
│  │ • Benutzer verw.│     │ • Benutzer verw.│     │ • Benutzer verw.│       │
│  │ • Kunden/Aufträg│     │ • Kunden/Aufträg│     │ • Kunden/Aufträg│       │
│  │ • Billing/Rech. │     │ • Billing/Rech. │     │ • Billing/Rech. │       │
│  └────────┬────────┘     └────────┬────────┘     └────────┬────────┘       │
│           │                       │                       │                │
│     ┌─────┴─────┐           ┌─────┴─────┐           ┌─────┴─────┐          │
│     │           │           │           │           │           │          │
│     ▼           ▼           ▼           ▼           ▼           ▼          │
│  ┌──────┐  ┌──────┐     ┌──────┐  ┌──────┐     ┌──────┐  ┌──────┐         │
│  │ MA 1 │  │ MA 2 │     │ MA 1 │  │ MA 2 │     │ MA 1 │  │ MA 2 │  ← User │
│  │      │  │      │     │      │  │      │     │      │  │      │   Ebene │
│  │Projek│  │Projek│     │Projek│  │Projek│     │Projek│  │Projek│         │
│  │te    │  │te    │     │te    │  │te    │     │te    │  │te    │         │
│  └──────┘  └──────┘     └──────┘  └──────┘     └──────┘  └──────┘         │
│                                                                             │
│  MA = Mitarbeiter                                                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Detaillierte Rollenbeschreibung

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         ROLLEN-DEFINITION                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ╔═══════════════════════════════════════════════════════════════════════╗  │
│  ║  EBENE 1: SYSADMIN (System-Administrator)                             ║  │
│  ╠═══════════════════════════════════════════════════════════════════════╣  │
│  ║                                                                       ║  │
│  ║  ZUSTÄNDIGKEIT: Gesamte Plattform                                     ║  │
│  ║                                                                       ║  │
│  ║  RECHTE:                                                              ║  │
│  ║  ├── Organisationen                                                   ║  │
│  ║  │   ├── Erstellen / Löschen / Sperren                               ║  │
│  ║  │   ├── Einstellungen ändern                                        ║  │
│  ║  │   ├── Module freischalten/sperren                                 ║  │
│  ║  │   └── Zugriff auf ALLE Daten (Support-Modus)                      ║  │
│  ║  │                                                                    ║  │
│  ║  ├── Benutzer (über alle Orgs)                                       ║  │
│  ║  │   ├── Alle Benutzer verwalten                                     ║  │
│  ║  │   ├── Passwörter zurücksetzen                                     ║  │
│  ║  │   └── Sessions beenden                                            ║  │
│  ║  │                                                                    ║  │
│  ║  ├── Billing (Plattform-weit)                                        ║  │
│  ║  │   ├── Tarife und Preise festlegen                                 ║  │
│  ║  │   ├── Rechnungen aller Orgs einsehen                              ║  │
│  ║  │   ├── Zahlungen verwalten                                         ║  │
│  ║  │   └── Gutschriften erstellen                                      ║  │
│  ║  │                                                                    ║  │
│  ║  ├── System-Konfiguration                                            ║  │
│  ║  │   ├── Globale Einstellungen                                       ║  │
│  ║  │   ├── E-Mail-Server                                               ║  │
│  ║  │   ├── Speicher-Limits                                             ║  │
│  ║  │   ├── Feature-Flags                                               ║  │
│  ║  │   └── Wartungsmodus                                               ║  │
│  ║  │                                                                    ║  │
│  ║  ├── Kunden/Aufträge (über alle Orgs)                                ║  │
│  ║  │   ├── Einsehen aller Kunden                                       ║  │
│  ║  │   ├── Einsehen aller Projekte                                     ║  │
│  ║  │   ├── Statistiken & Reports                                       ║  │
│  ║  │   └── Datenexport für Compliance                                  ║  │
│  ║  │                                                                    ║  │
│  ║  └── Support                                                         ║  │
│  ║      ├── Impersonate User (als Benutzer agieren)                     ║  │
│  ║      ├── Ticket-Eskalation                                           ║  │
│  ║      └── System-Logs einsehen                                        ║  │
│  ║                                                                       ║  │
│  ╚═══════════════════════════════════════════════════════════════════════╝  │
│                                                                             │
│  ╔═══════════════════════════════════════════════════════════════════════╗  │
│  ║  EBENE 2: ORG-ADMIN (Organisations-Administrator)                     ║  │
│  ╠═══════════════════════════════════════════════════════════════════════╣  │
│  ║                                                                       ║  │
│  ║  ZUSTÄNDIGKEIT: Eigene Organisation                                   ║  │
│  ║                                                                       ║  │
│  ║  RECHTE:                                                              ║  │
│  ║  ├── Organisation                                                     ║  │
│  ║  │   ├── Firmeneinstellungen (Logo, Templates, E-Mail)               ║  │
│  ║  │   ├── Abrechnungseinstellungen                                    ║  │
│  ║  │   └── Module aktivieren (innerhalb Tarif)                         ║  │
│  ║  │                                                                    ║  │
│  ║  ├── Benutzer                                                        ║  │
│  ║  │   ├── Mitarbeiter einladen/entfernen                              ║  │
│  ║  │   ├── Rollen zuweisen                                             ║  │
│  ║  │   ├── Berechtigungen verwalten                                    ║  │
│  ║  │   └── Aktivitätslogs einsehen                                     ║  │
│  ║  │                                                                    ║  │
│  ║  ├── Kunden                                                          ║  │
│  ║  │   ├── Alle Kunden der Org verwalten                               ║  │
│  ║  │   ├── Kunden-Portal aktivieren                                    ║  │
│  ║  │   └── Kundendaten exportieren                                     ║  │
│  ║  │                                                                    ║  │
│  ║  ├── Projekte/Aufträge                                               ║  │
│  ║  │   ├── Alle Projekte einsehen                                      ║  │
│  ║  │   ├── Projekte zuweisen                                           ║  │
│  ║  │   └── Projektvorlagen verwalten                                   ║  │
│  ║  │                                                                    ║  │
│  ║  ├── Finanzen                                                        ║  │
│  ║  │   ├── Rechnungen erstellen/verwalten                              ║  │
│  ║  │   ├── Angebote verwalten                                          ║  │
│  ║  │   ├── Umsatzstatistiken                                           ║  │
│  ║  │   └── DATEV-Export                                                ║  │
│  ║  │                                                                    ║  │
│  ║  └── Reports                                                         ║  │
│  ║      ├── Team-Auswertungen                                           ║  │
│  ║      ├── Zeiterfassungs-Reports                                      ║  │
│  ║      └── Produktivitätsanalysen                                      ║  │
│  ║                                                                       ║  │
│  ╚═══════════════════════════════════════════════════════════════════════╝  │
│                                                                             │
│  ╔═══════════════════════════════════════════════════════════════════════╗  │
│  ║  EBENE 3: MITARBEITER (Standard-Benutzer)                             ║  │
│  ╠═══════════════════════════════════════════════════════════════════════╣  │
│  ║                                                                       ║  │
│  ║  ZUSTÄNDIGKEIT: Zugewiesene Projekte                                  ║  │
│  ║                                                                       ║  │
│  ║  RECHTE:                                                              ║  │
│  ║  ├── Projekte                                                        ║  │
│  ║  │   ├── Eigene/zugewiesene Projekte bearbeiten                      ║  │
│  ║  │   ├── Neue Projekte erstellen (wenn erlaubt)                      ║  │
│  ║  │   └── Berechnungen durchführen                                    ║  │
│  ║  │                                                                    ║  │
│  ║  ├── Kunden                                                          ║  │
│  ║  │   ├── Eigene Kunden verwalten                                     ║  │
│  ║  │   └── Neue Kunden anlegen (wenn erlaubt)                          ║  │
│  ║  │                                                                    ║  │
│  ║  ├── Berichte                                                        ║  │
│  ║  │   ├── Berichte erstellen                                          ║  │
│  ║  │   └── Berichte exportieren                                        ║  │
│  ║  │                                                                    ║  │
│  ║  ├── Zeiterfassung                                                   ║  │
│  ║  │   ├── Eigene Zeit erfassen (automatisch)                          ║  │
│  ║  │   └── Eigene Auswertungen einsehen                                ║  │
│  ║  │                                                                    ║  │
│  ║  └── Profil                                                          ║  │
│  ║      ├── Eigene Einstellungen                                        ║  │
│  ║      └── Passwort ändern                                             ║  │
│  ║                                                                       ║  │
│  ╚═══════════════════════════════════════════════════════════════════════╝  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Sysadmin-Interface

### 2.1 Sysadmin Dashboard

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      SYSADMIN DASHBOARD                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  NAVIGATION (Sysadmin)                                               │   │
│  │                                                                      │   │
│  │  📊 Dashboard           → Systemübersicht, KPIs, Health              │   │
│  │  🏢 Organisationen      → Alle Orgs verwalten                        │   │
│  │  👥 Benutzer            → Alle User verwalten                        │   │
│  │  📋 Aufträge            → Alle Projekte über alle Orgs               │   │
│  │  👤 Kunden              → Alle Kunden über alle Orgs                 │   │
│  │  💰 Billing             → Tarife, Rechnungen, Zahlungen              │   │
│  │  📦 Module              → Feature-Management                         │   │
│  │  ⚙️  System             → Globale Konfiguration                      │   │
│  │  🎫 Support             → Ticket-Übersicht, Eskalationen             │   │
│  │  📈 Analytics           → Plattform-Statistiken                      │   │
│  │  📝 Audit Log           → Alle Aktivitäten                           │   │
│  │                                                                      │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Sysadmin-Funktionen Implementation

```python
# app/admin/sysadmin_service.py
"""Sysadmin Service - Vollzugriff auf alle Plattform-Funktionen"""

from typing import List, Optional, Dict, Any
from datetime import datetime, timedelta
from enum import Enum

class SysadminService:
    """Service für Sysadmin-Funktionen"""

    def __init__(self, session, cache, audit_logger):
        self.session = session
        self.cache = cache
        self.audit = audit_logger

    # ========================================================================
    # Organisationen verwalten
    # ========================================================================

    async def list_all_organizations(
        self,
        status: str = None,
        search: str = None,
        sort_by: str = "created_at",
        limit: int = 50,
        offset: int = 0
    ) -> Dict:
        """Listet alle Organisationen"""

        query = """
            SELECT
                o.*,
                COUNT(DISTINCT u.id) as user_count,
                COUNT(DISTINCT p.id) as project_count,
                COUNT(DISTINCT c.id) as customer_count,
                s.tier as subscription_tier,
                s.status as subscription_status
            FROM organizations o
            LEFT JOIN users u ON u.organization_id = o.id AND u.is_active = true
            LEFT JOIN projects p ON p.organization_id = o.id
            LEFT JOIN customers c ON c.organization_id = o.id
            LEFT JOIN subscriptions s ON s.organization_id = o.id
            WHERE 1=1
        """
        params = {"limit": limit, "offset": offset}

        if status:
            query += " AND o.status = :status"
            params["status"] = status

        if search:
            query += " AND (o.name ILIKE :search OR o.email ILIKE :search)"
            params["search"] = f"%{search}%"

        query += f" GROUP BY o.id, s.tier, s.status ORDER BY o.{sort_by} DESC"
        query += " LIMIT :limit OFFSET :offset"

        result = await self.session.execute(query, params)
        orgs = result.fetchall()

        # Gesamtzahl
        count_result = await self.session.execute(
            "SELECT COUNT(*) FROM organizations WHERE 1=1" +
            (" AND status = :status" if status else ""),
            {"status": status} if status else {}
        )

        return {
            "organizations": [self._org_to_dict(o) for o in orgs],
            "total": count_result.scalar(),
            "limit": limit,
            "offset": offset
        }

    async def get_organization_details(self, org_id: str) -> Dict:
        """Detaillierte Org-Informationen für Sysadmin"""

        org = await self._get_org(org_id)

        # Benutzer
        users = await self.session.execute("""
            SELECT u.*, COUNT(p.id) as project_count
            FROM users u
            LEFT JOIN projects p ON p.created_by = u.id
            WHERE u.organization_id = :org_id
            GROUP BY u.id
        """, {"org_id": org_id})

        # Billing
        billing = await self._get_org_billing(org_id)

        # Nutzungsstatistiken
        usage = await self._get_org_usage(org_id)

        # Letzte Aktivitäten
        activities = await self._get_org_activities(org_id, limit=20)

        return {
            "organization": org,
            "users": [dict(u) for u in users.fetchall()],
            "billing": billing,
            "usage": usage,
            "recent_activities": activities
        }

    async def create_organization(
        self,
        admin_id: str,
        data: Dict
    ) -> Dict:
        """Erstellt neue Organisation (Sysadmin)"""

        org = Organization(
            id=str(uuid.uuid4()),
            name=data["name"],
            email=data["email"],
            status="active",
            created_at=datetime.utcnow(),
            settings=data.get("settings", {})
        )

        await self._save_org(org)

        # Ersten Admin-User erstellen
        if data.get("admin_email"):
            await self._create_org_admin(org.id, data["admin_email"])

        # Audit Log
        await self.audit.log(
            action="org_created",
            actor_id=admin_id,
            entity_type="organization",
            entity_id=org.id,
            details={"name": org.name}
        )

        return {"organization": org, "status": "created"}

    async def suspend_organization(
        self,
        admin_id: str,
        org_id: str,
        reason: str
    ) -> Dict:
        """Sperrt Organisation"""

        org = await self._get_org(org_id)
        org.status = "suspended"
        org.suspended_at = datetime.utcnow()
        org.suspension_reason = reason

        await self._update_org(org)

        # Alle User-Sessions beenden
        await self._invalidate_org_sessions(org_id)

        # Audit Log
        await self.audit.log(
            action="org_suspended",
            actor_id=admin_id,
            entity_type="organization",
            entity_id=org_id,
            details={"reason": reason}
        )

        return {"status": "suspended"}

    async def update_organization_settings(
        self,
        admin_id: str,
        org_id: str,
        settings: Dict
    ) -> Dict:
        """Aktualisiert Org-Einstellungen (Sysadmin kann ALLES ändern)"""

        org = await self._get_org(org_id)
        old_settings = org.settings.copy()

        # Alle Einstellungen können geändert werden
        for key, value in settings.items():
            org.settings[key] = value

        await self._update_org(org)

        # Audit Log
        await self.audit.log(
            action="org_settings_updated",
            actor_id=admin_id,
            entity_type="organization",
            entity_id=org_id,
            details={"old": old_settings, "new": org.settings}
        )

        return {"settings": org.settings}

    # ========================================================================
    # Benutzer verwalten (über alle Orgs)
    # ========================================================================

    async def list_all_users(
        self,
        org_id: str = None,
        role: str = None,
        status: str = None,
        search: str = None,
        limit: int = 50,
        offset: int = 0
    ) -> Dict:
        """Listet alle Benutzer über alle Organisationen"""

        query = """
            SELECT u.*, o.name as organization_name
            FROM users u
            JOIN organizations o ON u.organization_id = o.id
            WHERE 1=1
        """
        params = {"limit": limit, "offset": offset}

        if org_id:
            query += " AND u.organization_id = :org_id"
            params["org_id"] = org_id

        if role:
            query += " AND u.role = :role"
            params["role"] = role

        if status == "active":
            query += " AND u.is_active = true"
        elif status == "inactive":
            query += " AND u.is_active = false"

        if search:
            query += " AND (u.email ILIKE :search OR u.name ILIKE :search)"
            params["search"] = f"%{search}%"

        query += " ORDER BY u.created_at DESC LIMIT :limit OFFSET :offset"

        result = await self.session.execute(query, params)

        return {
            "users": [dict(u) for u in result.fetchall()],
            "limit": limit,
            "offset": offset
        }

    async def impersonate_user(
        self,
        admin_id: str,
        target_user_id: str
    ) -> Dict:
        """Sysadmin agiert als anderer Benutzer (Support)"""

        target_user = await self._get_user(target_user_id)

        # Impersonation Token erstellen
        impersonation_token = await self._create_impersonation_token(
            admin_id=admin_id,
            target_user_id=target_user_id,
            expires_in=timedelta(hours=1)
        )

        # Audit Log (WICHTIG!)
        await self.audit.log(
            action="user_impersonation_started",
            actor_id=admin_id,
            entity_type="user",
            entity_id=target_user_id,
            details={
                "target_email": target_user.email,
                "target_org": target_user.organization_id
            },
            severity="high"
        )

        return {
            "impersonation_token": impersonation_token,
            "target_user": {
                "id": target_user.id,
                "email": target_user.email,
                "name": target_user.name,
                "organization_id": target_user.organization_id
            },
            "expires_at": (datetime.utcnow() + timedelta(hours=1)).isoformat()
        }

    async def reset_user_password(
        self,
        admin_id: str,
        user_id: str,
        send_email: bool = True
    ) -> Dict:
        """Setzt Benutzer-Passwort zurück"""

        user = await self._get_user(user_id)

        # Temporäres Passwort generieren
        temp_password = self._generate_temp_password()
        hashed = self._hash_password(temp_password)

        user.hashed_password = hashed
        user.must_change_password = True
        await self._update_user(user)

        # E-Mail senden
        if send_email:
            await self.email_service.send_password_reset(
                user.email,
                temp_password
            )

        # Audit Log
        await self.audit.log(
            action="password_reset_by_admin",
            actor_id=admin_id,
            entity_type="user",
            entity_id=user_id
        )

        return {
            "status": "password_reset",
            "email_sent": send_email,
            "temp_password": temp_password if not send_email else None
        }

    # ========================================================================
    # Kunden über alle Organisationen
    # ========================================================================

    async def list_all_customers(
        self,
        org_id: str = None,
        search: str = None,
        limit: int = 50,
        offset: int = 0
    ) -> Dict:
        """Listet alle Kunden über alle Organisationen"""

        query = """
            SELECT
                c.*,
                o.name as organization_name,
                COUNT(p.id) as project_count
            FROM customers c
            JOIN organizations o ON c.organization_id = o.id
            LEFT JOIN projects p ON p.customer_id = c.id
            WHERE 1=1
        """
        params = {"limit": limit, "offset": offset}

        if org_id:
            query += " AND c.organization_id = :org_id"
            params["org_id"] = org_id

        if search:
            query += " AND (c.name ILIKE :search OR c.email ILIKE :search)"
            params["search"] = f"%{search}%"

        query += " GROUP BY c.id, o.name"
        query += " ORDER BY c.created_at DESC LIMIT :limit OFFSET :offset"

        result = await self.session.execute(query, params)

        return {
            "customers": [dict(c) for c in result.fetchall()],
            "limit": limit,
            "offset": offset
        }

    async def get_customer_details_admin(self, customer_id: str) -> Dict:
        """Kunden-Details für Sysadmin (mit Org-Info)"""

        customer = await self.session.execute("""
            SELECT
                c.*,
                o.name as organization_name,
                o.id as organization_id
            FROM customers c
            JOIN organizations o ON c.organization_id = o.id
            WHERE c.id = :id
        """, {"id": customer_id})

        row = customer.fetchone()
        if not row:
            raise ValueError("Kunde nicht gefunden")

        # Projekte des Kunden
        projects = await self.session.execute("""
            SELECT * FROM projects WHERE customer_id = :customer_id
        """, {"customer_id": customer_id})

        return {
            "customer": dict(row),
            "projects": [dict(p) for p in projects.fetchall()]
        }

    # ========================================================================
    # Aufträge/Projekte über alle Organisationen
    # ========================================================================

    async def list_all_projects(
        self,
        org_id: str = None,
        status: str = None,
        project_type: str = None,
        search: str = None,
        limit: int = 50,
        offset: int = 0
    ) -> Dict:
        """Listet alle Projekte über alle Organisationen"""

        query = """
            SELECT
                p.*,
                o.name as organization_name,
                c.name as customer_name,
                u.name as created_by_name
            FROM projects p
            JOIN organizations o ON p.organization_id = o.id
            LEFT JOIN customers c ON p.customer_id = c.id
            LEFT JOIN users u ON p.created_by = u.id
            WHERE 1=1
        """
        params = {"limit": limit, "offset": offset}

        if org_id:
            query += " AND p.organization_id = :org_id"
            params["org_id"] = org_id

        if status:
            query += " AND p.status = :status"
            params["status"] = status

        if project_type:
            query += " AND p.project_type = :project_type"
            params["project_type"] = project_type

        if search:
            query += " AND p.name ILIKE :search"
            params["search"] = f"%{search}%"

        query += " ORDER BY p.created_at DESC LIMIT :limit OFFSET :offset"

        result = await self.session.execute(query, params)

        return {
            "projects": [dict(p) for p in result.fetchall()],
            "limit": limit,
            "offset": offset
        }

    # ========================================================================
    # Billing (Plattform-weit)
    # ========================================================================

    async def get_billing_overview(self) -> Dict:
        """Billing-Übersicht für Sysadmin"""

        # MRR (Monthly Recurring Revenue)
        mrr = await self.session.execute("""
            SELECT
                s.tier,
                COUNT(*) as count,
                SUM(CASE
                    WHEN s.tier = 'starter' THEN 29
                    WHEN s.tier = 'professional' THEN 79
                    WHEN s.tier = 'business' THEN 199
                    ELSE 0
                END) as revenue
            FROM subscriptions s
            WHERE s.status = 'active'
            GROUP BY s.tier
        """)

        # Offene Rechnungen
        open_invoices = await self.session.execute("""
            SELECT
                COUNT(*) as count,
                SUM(total) as total
            FROM invoices
            WHERE status IN ('sent', 'overdue')
        """)

        # Überfällige Rechnungen
        overdue = await self.session.execute("""
            SELECT
                i.*,
                o.name as organization_name
            FROM invoices i
            JOIN organizations o ON i.organization_id = o.id
            WHERE i.status = 'overdue'
            ORDER BY i.due_date ASC
            LIMIT 20
        """)

        return {
            "mrr_by_tier": [dict(r) for r in mrr.fetchall()],
            "open_invoices": dict(open_invoices.fetchone()),
            "overdue_invoices": [dict(r) for r in overdue.fetchall()]
        }

    async def update_organization_subscription(
        self,
        admin_id: str,
        org_id: str,
        tier: str,
        reason: str
    ) -> Dict:
        """Ändert Org-Subscription (Sysadmin kann frei ändern)"""

        org = await self._get_org(org_id)
        subscription = await self._get_subscription(org_id)

        old_tier = subscription.tier
        subscription.tier = tier
        subscription.updated_at = datetime.utcnow()
        subscription.updated_by = admin_id

        await self._update_subscription(subscription)

        # Module entsprechend freischalten
        await self._sync_modules_for_tier(org_id, tier)

        # Audit Log
        await self.audit.log(
            action="subscription_changed_by_admin",
            actor_id=admin_id,
            entity_type="subscription",
            entity_id=subscription.id,
            details={
                "org_id": org_id,
                "old_tier": old_tier,
                "new_tier": tier,
                "reason": reason
            }
        )

        return {"status": "updated", "new_tier": tier}

    # ========================================================================
    # System-Konfiguration
    # ========================================================================

    async def get_system_config(self) -> Dict:
        """Lädt globale System-Konfiguration"""

        config = await self.cache.hgetall("system:config")

        # Defaults falls nicht gesetzt
        defaults = {
            "maintenance_mode": "false",
            "registration_enabled": "true",
            "max_users_per_org": "100",
            "max_storage_per_org_gb": "50",
            "default_subscription_tier": "free",
            "email_provider": "smtp",
            "ai_provider": "ollama",
            "feature_flags": "{}",
        }

        return {**defaults, **config}

    async def update_system_config(
        self,
        admin_id: str,
        config: Dict
    ) -> Dict:
        """Aktualisiert System-Konfiguration"""

        for key, value in config.items():
            await self.cache.hset("system:config", key, str(value))

        # Audit Log
        await self.audit.log(
            action="system_config_updated",
            actor_id=admin_id,
            entity_type="system",
            entity_id="config",
            details=config,
            severity="critical"
        )

        return {"status": "updated", "config": config}

    async def set_maintenance_mode(
        self,
        admin_id: str,
        enabled: bool,
        message: str = None
    ) -> Dict:
        """Aktiviert/Deaktiviert Wartungsmodus"""

        await self.cache.hset("system:config", "maintenance_mode", str(enabled).lower())
        if message:
            await self.cache.hset("system:config", "maintenance_message", message)

        # Alle verbundenen Clients benachrichtigen
        await self._broadcast_maintenance_notification(enabled, message)

        # Audit Log
        await self.audit.log(
            action="maintenance_mode_changed",
            actor_id=admin_id,
            entity_type="system",
            entity_id="maintenance",
            details={"enabled": enabled, "message": message},
            severity="critical"
        )

        return {"maintenance_mode": enabled}

    # ========================================================================
    # Analytics
    # ========================================================================

    async def get_platform_analytics(self, period: str = "30d") -> Dict:
        """Plattform-weite Analysen"""

        # Zeitraum berechnen
        days = int(period.replace("d", ""))
        start_date = datetime.utcnow() - timedelta(days=days)

        # Wachstum
        growth = await self.session.execute("""
            SELECT
                DATE(created_at) as date,
                COUNT(*) FILTER (WHERE entity = 'organization') as new_orgs,
                COUNT(*) FILTER (WHERE entity = 'user') as new_users,
                COUNT(*) FILTER (WHERE entity = 'project') as new_projects
            FROM (
                SELECT created_at, 'organization' as entity FROM organizations WHERE created_at >= :start
                UNION ALL
                SELECT created_at, 'user' as entity FROM users WHERE created_at >= :start
                UNION ALL
                SELECT created_at, 'project' as entity FROM projects WHERE created_at >= :start
            ) combined
            GROUP BY DATE(created_at)
            ORDER BY date
        """, {"start": start_date})

        # Aktivität
        activity = await self.session.execute("""
            SELECT
                DATE(timestamp) as date,
                COUNT(*) as actions
            FROM audit_log
            WHERE timestamp >= :start
            GROUP BY DATE(timestamp)
            ORDER BY date
        """, {"start": start_date})

        # Top Organisationen
        top_orgs = await self.session.execute("""
            SELECT
                o.name,
                COUNT(DISTINCT p.id) as projects,
                COUNT(DISTINCT u.id) as users
            FROM organizations o
            LEFT JOIN projects p ON p.organization_id = o.id
            LEFT JOIN users u ON u.organization_id = o.id
            GROUP BY o.id
            ORDER BY projects DESC
            LIMIT 10
        """)

        return {
            "period": period,
            "growth": [dict(r) for r in growth.fetchall()],
            "activity": [dict(r) for r in activity.fetchall()],
            "top_organizations": [dict(r) for r in top_orgs.fetchall()]
        }
```

---

## 3. Organisations-Admin Interface

### 3.1 Org-Admin Funktionen

```python
# app/admin/org_admin_service.py
"""Organisations-Admin Service"""

class OrgAdminService:
    """Service für Org-Admin-Funktionen"""

    def __init__(self, session, org_id: str):
        self.session = session
        self.org_id = org_id

    # ========================================================================
    # Benutzer-Verwaltung
    # ========================================================================

    async def list_users(self, include_inactive: bool = False) -> List[Dict]:
        """Listet Benutzer der eigenen Organisation"""

        query = """
            SELECT u.*, COUNT(p.id) as project_count
            FROM users u
            LEFT JOIN projects p ON p.created_by = u.id
            WHERE u.organization_id = :org_id
        """

        if not include_inactive:
            query += " AND u.is_active = true"

        query += " GROUP BY u.id ORDER BY u.name"

        result = await self.session.execute(query, {"org_id": self.org_id})
        return [dict(u) for u in result.fetchall()]

    async def invite_user(
        self,
        admin_id: str,
        email: str,
        role: str,
        permissions: Dict = None
    ) -> Dict:
        """Lädt neuen Benutzer ein"""

        # Prüfe Benutzer-Limit
        current_count = await self._count_users()
        limit = await self._get_user_limit()

        if current_count >= limit:
            raise ValueError(f"Benutzer-Limit erreicht ({limit})")

        # Einladung erstellen
        invitation = Invitation(
            id=str(uuid.uuid4()),
            organization_id=self.org_id,
            email=email,
            role=role,
            permissions=permissions or {},
            invited_by=admin_id,
            created_at=datetime.utcnow(),
            expires_at=datetime.utcnow() + timedelta(days=7)
        )

        await self._save_invitation(invitation)

        # E-Mail senden
        await self.email_service.send_invitation(
            email=email,
            org_name=await self._get_org_name(),
            invitation_id=invitation.id
        )

        return {"invitation_id": invitation.id, "status": "sent"}

    async def update_user_role(
        self,
        admin_id: str,
        user_id: str,
        new_role: str
    ) -> Dict:
        """Ändert Benutzer-Rolle"""

        user = await self._get_user(user_id)

        # Prüfe ob User zur Org gehört
        if user.organization_id != self.org_id:
            raise PermissionError("Benutzer gehört nicht zu dieser Organisation")

        old_role = user.role
        user.role = new_role
        await self._update_user(user)

        return {"user_id": user_id, "old_role": old_role, "new_role": new_role}

    async def remove_user(self, admin_id: str, user_id: str) -> Dict:
        """Entfernt Benutzer aus Organisation"""

        user = await self._get_user(user_id)

        if user.organization_id != self.org_id:
            raise PermissionError("Benutzer gehört nicht zu dieser Organisation")

        # Soft-Delete
        user.is_active = False
        user.deactivated_at = datetime.utcnow()
        user.deactivated_by = admin_id
        await self._update_user(user)

        # Sessions invalidieren
        await self._invalidate_user_sessions(user_id)

        return {"status": "removed"}

    # ========================================================================
    # Organisations-Einstellungen
    # ========================================================================

    async def get_org_settings(self) -> Dict:
        """Lädt Organisations-Einstellungen"""

        org = await self._get_org()

        return {
            "name": org.name,
            "email": org.email,
            "logo_url": org.logo_url,
            "address": org.address,
            "settings": org.settings,
            "email_settings": org.email_settings,
            "template_settings": org.template_settings
        }

    async def update_org_settings(
        self,
        admin_id: str,
        settings: Dict
    ) -> Dict:
        """Aktualisiert Organisations-Einstellungen"""

        org = await self._get_org()

        # Nur bestimmte Felder dürfen geändert werden
        allowed_fields = [
            "name", "email", "logo_url", "address",
            "email_settings", "template_settings",
            "default_project_settings", "branding"
        ]

        for key, value in settings.items():
            if key in allowed_fields:
                if key in ["email_settings", "template_settings", "branding"]:
                    org.settings[key] = value
                else:
                    setattr(org, key, value)

        await self._update_org(org)

        return {"status": "updated"}

    async def update_email_settings(
        self,
        admin_id: str,
        smtp_settings: Dict
    ) -> Dict:
        """Konfiguriert E-Mail-Versand für Organisation"""

        org = await self._get_org()

        org.email_settings = {
            "smtp_host": smtp_settings.get("host"),
            "smtp_port": smtp_settings.get("port", 587),
            "smtp_user": smtp_settings.get("user"),
            "smtp_password": smtp_settings.get("password"),  # Verschlüsselt speichern!
            "from_email": smtp_settings.get("from_email"),
            "from_name": smtp_settings.get("from_name", org.name),
            "use_tls": smtp_settings.get("use_tls", True)
        }

        await self._update_org(org)

        return {"status": "updated"}

    async def upload_logo(self, admin_id: str, file_data: bytes, filename: str) -> Dict:
        """Lädt Organisations-Logo hoch"""

        # Validierung
        allowed_types = ["image/png", "image/jpeg", "image/svg+xml"]
        # ... Validierung ...

        # Upload
        path = f"organizations/{self.org_id}/logo/{filename}"
        url = await self.storage.upload(path, file_data)

        org = await self._get_org()
        org.logo_url = url
        await self._update_org(org)

        return {"logo_url": url}

    # ========================================================================
    # Kunden-Verwaltung
    # ========================================================================

    async def list_customers(
        self,
        search: str = None,
        limit: int = 50,
        offset: int = 0
    ) -> Dict:
        """Listet alle Kunden der Organisation"""

        query = """
            SELECT
                c.*,
                COUNT(p.id) as project_count,
                MAX(p.created_at) as last_project_date
            FROM customers c
            LEFT JOIN projects p ON p.customer_id = c.id
            WHERE c.organization_id = :org_id
        """
        params = {"org_id": self.org_id, "limit": limit, "offset": offset}

        if search:
            query += " AND (c.name ILIKE :search OR c.email ILIKE :search)"
            params["search"] = f"%{search}%"

        query += " GROUP BY c.id ORDER BY c.name LIMIT :limit OFFSET :offset"

        result = await self.session.execute(query, params)

        return {
            "customers": [dict(c) for c in result.fetchall()],
            "limit": limit,
            "offset": offset
        }

    # ========================================================================
    # Auftrags-/Projekt-Verwaltung
    # ========================================================================

    async def list_all_projects(
        self,
        status: str = None,
        assignee_id: str = None,
        limit: int = 50,
        offset: int = 0
    ) -> Dict:
        """Listet alle Projekte der Organisation"""

        query = """
            SELECT
                p.*,
                c.name as customer_name,
                u.name as assignee_name
            FROM projects p
            LEFT JOIN customers c ON p.customer_id = c.id
            LEFT JOIN users u ON p.assignee_id = u.id
            WHERE p.organization_id = :org_id
        """
        params = {"org_id": self.org_id, "limit": limit, "offset": offset}

        if status:
            query += " AND p.status = :status"
            params["status"] = status

        if assignee_id:
            query += " AND p.assignee_id = :assignee_id"
            params["assignee_id"] = assignee_id

        query += " ORDER BY p.created_at DESC LIMIT :limit OFFSET :offset"

        result = await self.session.execute(query, params)

        return {
            "projects": [dict(p) for p in result.fetchall()],
            "limit": limit,
            "offset": offset
        }

    async def assign_project(
        self,
        admin_id: str,
        project_id: str,
        assignee_id: str
    ) -> Dict:
        """Weist Projekt einem Mitarbeiter zu"""

        project = await self._get_project(project_id)

        if project.organization_id != self.org_id:
            raise PermissionError("Projekt gehört nicht zu dieser Organisation")

        assignee = await self._get_user(assignee_id)
        if assignee.organization_id != self.org_id:
            raise PermissionError("Mitarbeiter gehört nicht zu dieser Organisation")

        project.assignee_id = assignee_id
        await self._update_project(project)

        # Benachrichtigung
        await self.notifications.notify_user(
            user_id=assignee_id,
            type="project_assigned",
            data={"project_id": project_id, "project_name": project.name}
        )

        return {"status": "assigned"}

    # ========================================================================
    # Finanzen
    # ========================================================================

    async def get_financial_overview(self, period: str = "month") -> Dict:
        """Finanz-Übersicht für Org-Admin"""

        # Zeitraum
        if period == "month":
            start = datetime.utcnow().replace(day=1)
        elif period == "quarter":
            month = ((datetime.utcnow().month - 1) // 3) * 3 + 1
            start = datetime.utcnow().replace(month=month, day=1)
        else:  # year
            start = datetime.utcnow().replace(month=1, day=1)

        # Umsatz
        revenue = await self.session.execute("""
            SELECT
                SUM(total) as total_revenue,
                COUNT(*) as invoice_count
            FROM invoices
            WHERE organization_id = :org_id
            AND status = 'paid'
            AND paid_at >= :start
        """, {"org_id": self.org_id, "start": start})

        # Offene Rechnungen
        open_invoices = await self.session.execute("""
            SELECT SUM(total) as amount, COUNT(*) as count
            FROM invoices
            WHERE organization_id = :org_id
            AND status IN ('sent', 'overdue')
        """, {"org_id": self.org_id})

        # Zeiterfassung
        time_tracked = await self.session.execute("""
            SELECT SUM(duration) as total_hours
            FROM time_entries
            WHERE organization_id = :org_id
            AND started_at >= :start
        """, {"org_id": self.org_id, "start": start})

        return {
            "period": period,
            "revenue": dict(revenue.fetchone()),
            "open_invoices": dict(open_invoices.fetchone()),
            "time_tracked_hours": time_tracked.fetchone().total_hours / 3600 if time_tracked else 0
        }
```

---

## 4. Multi-Window Synchronisation

### 4.1 BroadcastChannel API

```javascript
// static/js/multi-window-sync.js
/**
 * Multi-Window Synchronisation
 * Synchronisiert Zustand zwischen mehreren Browser-Tabs/Fenstern
 */

class MultiWindowSync {
    constructor() {
        this.channel = null;
        this.windowId = this.generateWindowId();
        this.state = {};
        this.listeners = new Map();

        this.init();
    }

    init() {
        // BroadcastChannel für Tab-übergreifende Kommunikation
        if ('BroadcastChannel' in window) {
            this.channel = new BroadcastChannel('energieberater_sync');
            this.channel.onmessage = (event) => this.handleMessage(event.data);
        }

        // Fallback: localStorage Events
        window.addEventListener('storage', (event) => {
            if (event.key?.startsWith('sync_')) {
                this.handleStorageEvent(event);
            }
        });

        // Visibility Change - Sync bei Tab-Wechsel
        document.addEventListener('visibilitychange', () => {
            if (document.visibilityState === 'visible') {
                this.requestStateSync();
            }
        });

        // Fenster-Schließen
        window.addEventListener('beforeunload', () => {
            this.broadcast({
                type: 'window_closed',
                windowId: this.windowId
            });
        });

        // Initial State Request
        this.requestStateSync();
    }

    generateWindowId() {
        return `window_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
    }

    // ========================================================================
    // Messaging
    // ========================================================================

    broadcast(message) {
        message.senderId = this.windowId;
        message.timestamp = Date.now();

        // BroadcastChannel
        if (this.channel) {
            this.channel.postMessage(message);
        }

        // localStorage Fallback
        localStorage.setItem(
            `sync_${message.type}_${Date.now()}`,
            JSON.stringify(message)
        );

        // Cleanup alte localStorage Items
        this.cleanupStorage();
    }

    handleMessage(message) {
        // Eigene Nachrichten ignorieren
        if (message.senderId === this.windowId) return;

        console.log('[MultiWindowSync] Received:', message.type);

        switch (message.type) {
            case 'state_update':
                this.handleStateUpdate(message);
                break;

            case 'state_request':
                this.handleStateRequest(message);
                break;

            case 'user_action':
                this.handleUserAction(message);
                break;

            case 'logout':
                this.handleLogout(message);
                break;

            case 'data_changed':
                this.handleDataChanged(message);
                break;

            case 'notification':
                this.handleNotification(message);
                break;
        }
    }

    handleStorageEvent(event) {
        if (!event.newValue) return;

        try {
            const message = JSON.parse(event.newValue);
            this.handleMessage(message);
        } catch (e) {
            console.error('[MultiWindowSync] Storage parse error:', e);
        }
    }

    // ========================================================================
    // State Synchronisation
    // ========================================================================

    updateState(key, value, broadcast = true) {
        this.state[key] = value;

        if (broadcast) {
            this.broadcast({
                type: 'state_update',
                key: key,
                value: value
            });
        }

        // Listener benachrichtigen
        const listeners = this.listeners.get(key) || [];
        listeners.forEach(callback => callback(value));
    }

    handleStateUpdate(message) {
        this.state[message.key] = message.value;

        // Listener benachrichtigen
        const listeners = this.listeners.get(message.key) || [];
        listeners.forEach(callback => callback(message.value));
    }

    requestStateSync() {
        this.broadcast({
            type: 'state_request'
        });
    }

    handleStateRequest(message) {
        // Eigenen State an anfragendes Fenster senden
        this.broadcast({
            type: 'state_update',
            key: '_full_state',
            value: this.state,
            targetWindow: message.senderId
        });
    }

    onStateChange(key, callback) {
        if (!this.listeners.has(key)) {
            this.listeners.set(key, []);
        }
        this.listeners.get(key).push(callback);

        // Cleanup-Funktion zurückgeben
        return () => {
            const listeners = this.listeners.get(key);
            const index = listeners.indexOf(callback);
            if (index > -1) {
                listeners.splice(index, 1);
            }
        };
    }

    // ========================================================================
    // Spezifische Sync-Events
    // ========================================================================

    // Projekt gewechselt
    syncProjectChange(projectId) {
        this.broadcast({
            type: 'user_action',
            action: 'project_changed',
            data: { projectId }
        });
    }

    // Daten wurden geändert
    syncDataChange(entityType, entityId, action) {
        this.broadcast({
            type: 'data_changed',
            entityType,
            entityId,
            action  // 'created', 'updated', 'deleted'
        });
    }

    handleDataChanged(message) {
        // Event für UI-Updates
        window.dispatchEvent(new CustomEvent('data-sync', {
            detail: {
                entityType: message.entityType,
                entityId: message.entityId,
                action: message.action
            }
        }));
    }

    // Logout in einem Tab -> Alle Tabs ausloggen
    syncLogout() {
        this.broadcast({
            type: 'logout'
        });
    }

    handleLogout(message) {
        // Sofort ausloggen
        window.location.href = '/logout?reason=session_ended_other_tab';
    }

    // Notification
    syncNotification(notification) {
        this.broadcast({
            type: 'notification',
            notification
        });
    }

    handleNotification(message) {
        // Nur anzeigen wenn Tab aktiv ist
        if (document.visibilityState === 'visible') {
            window.dispatchEvent(new CustomEvent('show-notification', {
                detail: message.notification
            }));
        }
    }

    // ========================================================================
    // Helpers
    // ========================================================================

    cleanupStorage() {
        const now = Date.now();
        const maxAge = 5000; // 5 Sekunden

        for (let i = localStorage.length - 1; i >= 0; i--) {
            const key = localStorage.key(i);
            if (key?.startsWith('sync_')) {
                try {
                    const data = JSON.parse(localStorage.getItem(key));
                    if (now - data.timestamp > maxAge) {
                        localStorage.removeItem(key);
                    }
                } catch (e) {
                    localStorage.removeItem(key);
                }
            }
        }
    }
}

// ============================================================================
// Integration mit Alpine.js
// ============================================================================

document.addEventListener('alpine:init', () => {
    Alpine.store('multiWindow', {
        sync: new MultiWindowSync(),

        // Reaktiver Zustand
        activeProject: null,
        notifications: [],

        init() {
            // State-Änderungen beobachten
            this.sync.onStateChange('activeProject', (value) => {
                this.activeProject = value;
            });

            // Data-Sync Events
            window.addEventListener('data-sync', (e) => {
                this.handleDataSync(e.detail);
            });
        },

        setActiveProject(projectId) {
            this.activeProject = projectId;
            this.sync.updateState('activeProject', projectId);
            this.sync.syncProjectChange(projectId);
        },

        handleDataSync(detail) {
            // UI-Updates triggern
            console.log('[Store] Data sync:', detail);
            // z.B. Listen neu laden
            if (detail.entityType === 'project') {
                this.$dispatch('refresh-projects');
            }
        }
    });
});

// Globale Instanz
window.multiWindowSync = new MultiWindowSync();
```

### 4.2 WebSocket Integration für Multi-Window

```javascript
// static/js/websocket-multi-window.js
/**
 * WebSocket mit Multi-Window Koordination
 * Nur ein Tab hält die WebSocket-Verbindung
 */

class SharedWebSocket {
    constructor(url) {
        this.url = url;
        this.ws = null;
        this.isLeader = false;
        this.listeners = new Map();

        // BroadcastChannel für Tab-Koordination
        this.channel = new BroadcastChannel('websocket_coordination');
        this.channel.onmessage = (e) => this.handleCoordinationMessage(e.data);

        this.init();
    }

    async init() {
        // Leader Election
        await this.electLeader();

        if (this.isLeader) {
            this.connect();
        }

        // Heartbeat für Leader-Erkennung
        setInterval(() => this.sendHeartbeat(), 5000);
    }

    async electLeader() {
        // Prüfe ob bereits ein Leader existiert
        const leaderCheck = await this.checkForLeader();

        if (!leaderCheck) {
            // Kein Leader -> Selbst Leader werden
            this.isLeader = true;
            this.announceLeadership();
        }
    }

    checkForLeader() {
        return new Promise((resolve) => {
            const timeout = setTimeout(() => resolve(false), 500);

            const handler = (e) => {
                if (e.data.type === 'leader_heartbeat') {
                    clearTimeout(timeout);
                    this.channel.removeEventListener('message', handler);
                    resolve(true);
                }
            };

            this.channel.addEventListener('message', handler);

            this.channel.postMessage({
                type: 'leader_check'
            });
        });
    }

    announceLeadership() {
        this.channel.postMessage({
            type: 'leader_heartbeat',
            windowId: window.multiWindowSync?.windowId
        });
    }

    sendHeartbeat() {
        if (this.isLeader) {
            this.announceLeadership();
        }
    }

    handleCoordinationMessage(message) {
        switch (message.type) {
            case 'leader_check':
                if (this.isLeader) {
                    this.announceLeadership();
                }
                break;

            case 'leader_heartbeat':
                // Anderer Tab ist Leader
                if (!this.isLeader && this.ws) {
                    this.ws.close();
                    this.ws = null;
                }
                break;

            case 'ws_message':
                // WebSocket-Nachricht von Leader erhalten
                if (!this.isLeader) {
                    this.dispatchMessage(message.data);
                }
                break;

            case 'ws_send':
                // Anderer Tab will Nachricht senden
                if (this.isLeader && this.ws?.readyState === WebSocket.OPEN) {
                    this.ws.send(JSON.stringify(message.data));
                }
                break;

            case 'leader_lost':
                // Leader wurde geschlossen -> neue Election
                this.electLeader();
                break;
        }
    }

    connect() {
        if (this.ws) return;

        this.ws = new WebSocket(this.url);

        this.ws.onopen = () => {
            console.log('[SharedWS] Connected as leader');
            this.broadcastStatus('connected');
        };

        this.ws.onmessage = (event) => {
            const data = JSON.parse(event.data);

            // An alle Tabs weiterleiten
            this.channel.postMessage({
                type: 'ws_message',
                data: data
            });

            // Lokal verarbeiten
            this.dispatchMessage(data);
        };

        this.ws.onclose = () => {
            console.log('[SharedWS] Disconnected');
            this.broadcastStatus('disconnected');

            // Reconnect nach 5s
            if (this.isLeader) {
                setTimeout(() => this.connect(), 5000);
            }
        };

        this.ws.onerror = (error) => {
            console.error('[SharedWS] Error:', error);
        };
    }

    send(data) {
        if (this.isLeader && this.ws?.readyState === WebSocket.OPEN) {
            this.ws.send(JSON.stringify(data));
        } else {
            // An Leader senden
            this.channel.postMessage({
                type: 'ws_send',
                data: data
            });
        }
    }

    subscribe(eventType, callback) {
        if (!this.listeners.has(eventType)) {
            this.listeners.set(eventType, []);
        }
        this.listeners.get(eventType).push(callback);

        return () => {
            const callbacks = this.listeners.get(eventType);
            const index = callbacks.indexOf(callback);
            if (index > -1) callbacks.splice(index, 1);
        };
    }

    dispatchMessage(data) {
        const eventType = data.type || 'message';
        const callbacks = this.listeners.get(eventType) || [];
        callbacks.forEach(cb => cb(data));

        // Wildcard Listener
        const wildcardCallbacks = this.listeners.get('*') || [];
        wildcardCallbacks.forEach(cb => cb(data));
    }

    broadcastStatus(status) {
        this.channel.postMessage({
            type: 'ws_status',
            status: status
        });
    }

    close() {
        if (this.isLeader) {
            this.channel.postMessage({ type: 'leader_lost' });
            this.ws?.close();
        }
    }
}

// Initialisierung
window.addEventListener('DOMContentLoaded', () => {
    const token = localStorage.getItem('access_token');
    if (token) {
        window.sharedWS = new SharedWebSocket(
            `wss://${window.location.host}/ws?token=${token}`
        );
    }
});
```

---

## 5. Datenbank-Schema Erweiterung

```sql
-- Benutzer-Rollen und Berechtigungen

-- Benutzer-Tabelle Erweiterung
ALTER TABLE users ADD COLUMN IF NOT EXISTS role VARCHAR(50) DEFAULT 'user';
ALTER TABLE users ADD COLUMN IF NOT EXISTS permissions JSONB DEFAULT '{}';
ALTER TABLE users ADD COLUMN IF NOT EXISTS is_sysadmin BOOLEAN DEFAULT FALSE;

-- Sysadmins (separate Tabelle für extra Sicherheit)
CREATE TABLE IF NOT EXISTS sysadmins (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID UNIQUE REFERENCES users(id),
    access_level VARCHAR(50) DEFAULT 'full',  -- full, readonly, support
    created_at TIMESTAMPTZ DEFAULT NOW(),
    created_by UUID REFERENCES users(id),
    mfa_required BOOLEAN DEFAULT TRUE,

    CONSTRAINT sysadmin_user_unique UNIQUE(user_id)
);

-- Impersonation Log (kritisch für Audit)
CREATE TABLE IF NOT EXISTS impersonation_log (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    admin_id UUID REFERENCES users(id) NOT NULL,
    target_user_id UUID REFERENCES users(id) NOT NULL,
    started_at TIMESTAMPTZ DEFAULT NOW(),
    ended_at TIMESTAMPTZ,
    actions_performed JSONB DEFAULT '[]',
    ip_address INET,
    user_agent TEXT
);

-- Org-Admin Einladungen
CREATE TABLE IF NOT EXISTS invitations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID REFERENCES organizations(id),
    email VARCHAR(255) NOT NULL,
    role VARCHAR(50) NOT NULL,
    permissions JSONB DEFAULT '{}',
    invited_by UUID REFERENCES users(id),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    expires_at TIMESTAMPTZ NOT NULL,
    accepted_at TIMESTAMPTZ,
    accepted_by UUID REFERENCES users(id)
);

-- RLS Policies für Ebenen

-- Sysadmins können alles sehen
CREATE POLICY sysadmin_full_access ON organizations
    FOR ALL
    TO authenticated
    USING (
        EXISTS (SELECT 1 FROM sysadmins WHERE user_id = current_setting('app.current_user')::uuid)
    );

-- Org-Admins können nur eigene Org sehen
CREATE POLICY org_admin_access ON organizations
    FOR ALL
    TO authenticated
    USING (
        id = current_setting('app.current_org')::uuid
        AND EXISTS (
            SELECT 1 FROM users
            WHERE id = current_setting('app.current_user')::uuid
            AND role IN ('org_admin', 'owner')
        )
    );

-- Mitarbeiter nur Lesezugriff auf Org
CREATE POLICY user_read_org ON organizations
    FOR SELECT
    TO authenticated
    USING (
        id = current_setting('app.current_org')::uuid
    );
```

---

## 6. Checkliste

```
✓ 3-Ebenen-Hierarchie definiert (Sysadmin, Org-Admin, Mitarbeiter)
✓ Sysadmin-Service mit Vollzugriff
✓ Impersonation für Support
✓ Org-Admin-Service
✓ Multi-Window-Synchronisation mit BroadcastChannel
✓ Shared WebSocket mit Leader Election
✓ Datenbank-Schema für Rollen
✓ RLS Policies für Ebenen
✓ Audit-Logging für kritische Aktionen
```

---

*Letzte Aktualisierung: 2025-11-25*
*Version: 1.0.0*
