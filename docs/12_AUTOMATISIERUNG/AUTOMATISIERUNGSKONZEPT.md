# Automatisierungskonzept EnergieberaterPRO

## Referenzen
- [n8n Workflow Automation](https://n8n.io/)
- [Temporal Workflow Engine](https://temporal.io/)
- [Celery Task Queue](https://docs.celeryq.dev/)
- [Apache Airflow](https://airflow.apache.org/)

---

## 1. Automatisierungs-Philosophie

### 1.1 Kernprinzip: 0% Bürokratie

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    AUTOMATISIERUNGS-PHILOSOPHIE                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ZIEL: Mitarbeiter arbeiten NUR am Inhalt, NIEMALS an Administration       │
│                                                                             │
│  ┌──────────────────────┐         ┌──────────────────────┐                 │
│  │ VORHER               │         │ NACHHER              │                 │
│  ├──────────────────────┤         ├──────────────────────┤                 │
│  │ • Zeit erfassen      │    →    │ • Automatisch        │                 │
│  │ • Berichte erstellen │    →    │ • Vorausgefüllt      │                 │
│  │ • E-Mails schreiben  │    →    │ • KI-generiert       │                 │
│  │ • Rechnungen stellen │    →    │ • Auto-generiert     │                 │
│  │ • Status aktualisieren│   →    │ • Auto-Tracking      │                 │
│  │ • Dokumente sortieren│    →    │ • Auto-Kategorisiert │                 │
│  └──────────────────────┘         └──────────────────────┘                 │
│                                                                             │
│  MESSGRÖSSE: Zeit für Administration pro Projekt < 5 Minuten               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Automatisierungs-Ebenen

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       AUTOMATISIERUNGS-PYRAMIDE                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                         ┌────────────────────┐                              │
│                         │   KI-GESTEUERT     │  ← Kontext-Analyse           │
│                         │   Entscheidungen   │    Empfehlungen              │
│                         └────────────────────┘    Textgenerierung           │
│                    ┌────────────────────────────────┐                       │
│                    │      WORKFLOW-AUTOMATION       │  ← n8n/Temporal       │
│                    │   Prozess-Orchestrierung       │    Event-Chains       │
│                    └────────────────────────────────┘    Multi-Step         │
│               ┌──────────────────────────────────────────┐                  │
│               │         TRIGGER-BASIERT                  │  ← Webhooks      │
│               │   Event → Aktion automatisch             │    Cron Jobs     │
│               └──────────────────────────────────────────┘    Watchers      │
│          ┌────────────────────────────────────────────────────┐             │
│          │              REGELBASIERT                          │  ← Validierung│
│          │   IF Bedingung THEN Aktion                         │    Defaults   │
│          └────────────────────────────────────────────────────┘    Auto-Fill  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Automatische Zeiterfassung

### 2.1 Konzept

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    AUTOMATISCHE ZEITERFASSUNG                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ERFASSUNG OHNE BENUTZERINTERAKTION                                         │
│                                                                             │
│  ┌────────────┐    ┌────────────┐    ┌────────────┐    ┌────────────┐      │
│  │  Session   │───→│  Kontext   │───→│   Modul    │───→│   Zeit     │      │
│  │  Start     │    │  erkennen  │    │  zuordnen  │    │  erfassen  │      │
│  └────────────┘    └────────────┘    └────────────┘    └────────────┘      │
│                                                                             │
│  KONTEXT-ERKENNUNG:                                                         │
│  ─────────────────────────────────────────────────────────────────────────  │
│  • Aktives Projekt aus URL/Session                                          │
│  • Aktives Modul aus Navigation                                             │
│  • Aktive Tätigkeit aus Aktionen                                            │
│  • Automatische Pause-Erkennung (Inaktivität > 5 Min)                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Implementierung

```python
# app/services/time_tracking.py
"""Automatische Zeiterfassung ohne Benutzerinteraktion"""

from datetime import datetime, timedelta
from typing import Optional
from sqlalchemy.ext.asyncio import AsyncSession
from app.models import TimeEntry, User, Project, ActivityType

class AutomaticTimeTracker:
    """Erfasst Arbeitszeit automatisch basierend auf Benutzeraktivität"""

    INACTIVITY_THRESHOLD = timedelta(minutes=5)
    MINIMUM_SESSION_DURATION = timedelta(seconds=30)

    def __init__(self, session: AsyncSession):
        self.session = session
        self._active_entries: dict[str, TimeEntry] = {}

    async def track_activity(
        self,
        user_id: str,
        project_id: str,
        module: str,
        action: str,
        metadata: dict = None
    ) -> None:
        """Trackt Benutzeraktivität und erstellt/aktualisiert Zeiteinträge"""

        entry_key = f"{user_id}:{project_id}:{module}"
        current_time = datetime.utcnow()

        # Prüfe auf aktiven Eintrag
        active_entry = self._active_entries.get(entry_key)

        if active_entry:
            # Prüfe auf Inaktivität
            if current_time - active_entry.last_activity > self.INACTIVITY_THRESHOLD:
                # Session beenden, neue starten
                await self._close_entry(active_entry)
                active_entry = None

        if not active_entry:
            # Neue Session starten
            activity_type = self._determine_activity_type(module, action)

            active_entry = TimeEntry(
                user_id=user_id,
                project_id=project_id,
                module=module,
                activity_type=activity_type,
                started_at=current_time,
                is_automatic=True,
                metadata=metadata or {}
            )
            self.session.add(active_entry)
            self._active_entries[entry_key] = active_entry

        # Aktivität aktualisieren
        active_entry.last_activity = current_time
        active_entry.actions_count = (active_entry.actions_count or 0) + 1

        # Action-Log hinzufügen (für Analyse)
        if not active_entry.action_log:
            active_entry.action_log = []
        active_entry.action_log.append({
            "time": current_time.isoformat(),
            "action": action,
            "metadata": metadata
        })

        await self.session.commit()

    async def _close_entry(self, entry: TimeEntry) -> None:
        """Schließt einen Zeiteintrag ab"""
        entry.ended_at = entry.last_activity
        entry.duration = entry.ended_at - entry.started_at

        # Nur speichern wenn Mindestdauer erreicht
        if entry.duration >= self.MINIMUM_SESSION_DURATION:
            entry.is_completed = True
        else:
            # Zu kurz, verwerfen
            await self.session.delete(entry)

        key = f"{entry.user_id}:{entry.project_id}:{entry.module}"
        self._active_entries.pop(key, None)

        await self.session.commit()

    def _determine_activity_type(self, module: str, action: str) -> ActivityType:
        """Bestimmt Aktivitätstyp aus Modul und Aktion"""
        activity_mapping = {
            "calculations": ActivityType.CALCULATION,
            "reports": ActivityType.REPORT_WRITING,
            "buildings": ActivityType.DATA_ENTRY,
            "customers": ActivityType.CUSTOMER_COMMUNICATION,
            "documents": ActivityType.DOCUMENTATION,
            "ai_assistant": ActivityType.AI_RESEARCH,
        }
        return activity_mapping.get(module, ActivityType.GENERAL)

    async def get_project_summary(
        self,
        project_id: str,
        start_date: datetime = None,
        end_date: datetime = None
    ) -> dict:
        """Erstellt automatische Projektzeitübersicht"""
        query = """
            SELECT
                module,
                activity_type,
                SUM(EXTRACT(EPOCH FROM duration)) as total_seconds,
                COUNT(*) as session_count,
                SUM(actions_count) as total_actions
            FROM time_entries
            WHERE project_id = :project_id
                AND is_completed = true
                AND (:start_date IS NULL OR started_at >= :start_date)
                AND (:end_date IS NULL OR started_at <= :end_date)
            GROUP BY module, activity_type
            ORDER BY total_seconds DESC
        """

        result = await self.session.execute(query, {
            "project_id": project_id,
            "start_date": start_date,
            "end_date": end_date
        })

        return {
            "project_id": project_id,
            "period": {
                "start": start_date.isoformat() if start_date else None,
                "end": end_date.isoformat() if end_date else None
            },
            "modules": [
                {
                    "module": row.module,
                    "activity_type": row.activity_type.value,
                    "total_hours": round(row.total_seconds / 3600, 2),
                    "sessions": row.session_count,
                    "actions": row.total_actions
                }
                for row in result
            ],
            "total_hours": sum(row.total_seconds / 3600 for row in result)
        }


# Middleware für automatisches Tracking
class TimeTrackingMiddleware:
    """ASGI Middleware für automatische Zeiterfassung"""

    def __init__(self, app):
        self.app = app

    async def __call__(self, scope, receive, send):
        if scope["type"] == "http":
            request = Request(scope, receive)

            # Nur für authentifizierte Benutzer
            if hasattr(request.state, "user"):
                user = request.state.user
                path = request.url.path

                # Projekt und Modul aus URL extrahieren
                project_id, module = self._extract_context(path)

                if project_id and module:
                    tracker = AutomaticTimeTracker(request.state.db)
                    await tracker.track_activity(
                        user_id=str(user.id),
                        project_id=project_id,
                        module=module,
                        action=request.method,
                        metadata={
                            "path": path,
                            "query": str(request.query_params)
                        }
                    )

        await self.app(scope, receive, send)

    def _extract_context(self, path: str) -> tuple[Optional[str], Optional[str]]:
        """Extrahiert Projekt-ID und Modul aus URL"""
        import re

        # Pattern: /api/v1/projects/{project_id}/{module}/...
        match = re.match(r"/api/v1/projects/([^/]+)/([^/]+)", path)
        if match:
            return match.group(1), match.group(2)

        return None, None
```

### 2.3 Frontend-Integration

```javascript
// static/js/time-tracking.js
/**
 * Automatische Zeiterfassung im Frontend
 * Trackt Benutzeraktivität ohne explizite Interaktion
 */

class AutoTimeTracker {
    constructor() {
        this.lastActivity = Date.now();
        this.inactivityThreshold = 5 * 60 * 1000; // 5 Minuten
        this.trackingInterval = null;
        this.currentContext = null;

        this.init();
    }

    init() {
        // Event Listener für Aktivität
        ['click', 'keypress', 'scroll', 'mousemove'].forEach(event => {
            document.addEventListener(event, () => this.recordActivity(), { passive: true });
        });

        // Visibility API für Tab-Wechsel
        document.addEventListener('visibilitychange', () => {
            if (document.hidden) {
                this.pause();
            } else {
                this.resume();
            }
        });

        // Kontext aus URL
        this.updateContext();
        window.addEventListener('popstate', () => this.updateContext());

        // Periodisches Heartbeat
        this.trackingInterval = setInterval(() => this.sendHeartbeat(), 30000);
    }

    updateContext() {
        const path = window.location.pathname;
        const match = path.match(/\/projects\/([^/]+)(?:\/([^/]+))?/);

        if (match) {
            this.currentContext = {
                projectId: match[1],
                module: match[2] || 'overview'
            };
        } else {
            this.currentContext = null;
        }
    }

    recordActivity() {
        this.lastActivity = Date.now();
    }

    async sendHeartbeat() {
        if (!this.currentContext) return;

        const timeSinceActivity = Date.now() - this.lastActivity;
        if (timeSinceActivity > this.inactivityThreshold) {
            return; // Inaktiv, kein Heartbeat
        }

        try {
            await fetch('/api/v1/time-tracking/heartbeat', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${this.getToken()}`
                },
                body: JSON.stringify({
                    project_id: this.currentContext.projectId,
                    module: this.currentContext.module,
                    timestamp: new Date().toISOString()
                })
            });
        } catch (error) {
            console.debug('Time tracking heartbeat failed:', error);
        }
    }

    pause() {
        // Tab nicht sichtbar - Tracking pausieren
        clearInterval(this.trackingInterval);
    }

    resume() {
        // Tab wieder sichtbar - Tracking fortsetzen
        this.recordActivity();
        this.trackingInterval = setInterval(() => this.sendHeartbeat(), 30000);
    }

    getToken() {
        return localStorage.getItem('access_token');
    }
}

// Initialisierung
const timeTracker = new AutoTimeTracker();
```

---

## 3. Workflow-Automatisierung

### 3.1 Workflow-Engine Architektur

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       WORKFLOW-ENGINE ARCHITEKTUR                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                         EVENT BUS (Redis Streams)                      │ │
│  └──────────────────┬───────────────────────────────────┬─────────────────┘ │
│                     │                                   │                   │
│         ┌───────────▼───────────┐         ┌────────────▼────────────┐      │
│         │   WORKFLOW TRIGGERS   │         │   SCHEDULED TRIGGERS    │      │
│         │   • project.created   │         │   • Daily Reports       │      │
│         │   • calculation.done  │         │   • Weekly Summaries    │      │
│         │   • invoice.overdue   │         │   • Monthly Billing     │      │
│         └───────────┬───────────┘         └────────────┬────────────┘      │
│                     │                                   │                   │
│         ┌───────────▼───────────────────────────────────▼───────────┐      │
│         │              WORKFLOW ORCHESTRATOR (Temporal)              │      │
│         │   • State Management                                       │      │
│         │   • Retry Logic                                            │      │
│         │   • Error Handling                                         │      │
│         │   • Audit Trail                                            │      │
│         └───────────────────────────────────────────────────────────┘      │
│                                     │                                       │
│         ┌───────────────────────────┼───────────────────────────┐          │
│         │                           │                           │          │
│         ▼                           ▼                           ▼          │
│  ┌──────────────┐          ┌──────────────┐          ┌──────────────┐     │
│  │ EMAIL WORKER │          │ CALC WORKER  │          │ REPORT WORKER│     │
│  │ • Templates  │          │ • Validation │          │ • PDF Gen    │     │
│  │ • SMTP Send  │          │ • Compute    │          │ • Storage    │     │
│  └──────────────┘          └──────────────┘          └──────────────┘     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Vordefinierte Workflows

```python
# app/workflows/definitions.py
"""Vordefinierte Automatisierungs-Workflows"""

from temporalio import workflow, activity
from datetime import timedelta
from typing import List
from dataclasses import dataclass

# ============================================================================
# WORKFLOW 1: Neues Projekt
# ============================================================================

@dataclass
class NewProjectInput:
    project_id: str
    customer_id: str
    created_by: str
    project_type: str

@workflow.defn
class NewProjectWorkflow:
    """Automatisierung bei neuem Projekt"""

    @workflow.run
    async def run(self, input: NewProjectInput) -> dict:
        results = {}

        # 1. Projektordner erstellen
        results["folder"] = await workflow.execute_activity(
            create_project_folder,
            input.project_id,
            start_to_close_timeout=timedelta(seconds=30)
        )

        # 2. Standard-Templates kopieren
        results["templates"] = await workflow.execute_activity(
            copy_project_templates,
            input.project_id,
            input.project_type,
            start_to_close_timeout=timedelta(seconds=60)
        )

        # 3. Willkommens-E-Mail an Kunde
        results["email"] = await workflow.execute_activity(
            send_customer_welcome_email,
            input.customer_id,
            input.project_id,
            start_to_close_timeout=timedelta(seconds=30)
        )

        # 4. Kalender-Eintrag erstellen
        results["calendar"] = await workflow.execute_activity(
            create_project_calendar_entry,
            input.project_id,
            input.created_by,
            start_to_close_timeout=timedelta(seconds=30)
        )

        # 5. Benachrichtigung an Team
        results["notification"] = await workflow.execute_activity(
            notify_team_new_project,
            input.project_id,
            start_to_close_timeout=timedelta(seconds=10)
        )

        return results


# ============================================================================
# WORKFLOW 2: Berechnung abgeschlossen
# ============================================================================

@dataclass
class CalculationCompletedInput:
    calculation_id: str
    project_id: str
    calculation_type: str
    results: dict

@workflow.defn
class CalculationCompletedWorkflow:
    """Automatisierung nach abgeschlossener Berechnung"""

    @workflow.run
    async def run(self, input: CalculationCompletedInput) -> dict:
        results = {}

        # 1. Ergebnisse validieren
        validation = await workflow.execute_activity(
            validate_calculation_results,
            input.results,
            start_to_close_timeout=timedelta(seconds=60)
        )

        if not validation["is_valid"]:
            results["status"] = "validation_failed"
            results["errors"] = validation["errors"]
            return results

        # 2. Ergebnisse in Bericht-Baustein umwandeln
        results["report_block"] = await workflow.execute_activity(
            create_report_block,
            input.calculation_id,
            input.results,
            start_to_close_timeout=timedelta(seconds=30)
        )

        # 3. Sanierungsempfehlungen generieren (KI)
        results["recommendations"] = await workflow.execute_activity(
            generate_recommendations,
            input.calculation_type,
            input.results,
            start_to_close_timeout=timedelta(minutes=2)
        )

        # 4. Projekt-Status aktualisieren
        results["status_update"] = await workflow.execute_activity(
            update_project_progress,
            input.project_id,
            input.calculation_type,
            start_to_close_timeout=timedelta(seconds=10)
        )

        # 5. Bei kritischen Werten: Benachrichtigung
        if self._is_critical(input.results):
            results["alert"] = await workflow.execute_activity(
                send_critical_value_alert,
                input.project_id,
                input.results,
                start_to_close_timeout=timedelta(seconds=30)
            )

        return results

    def _is_critical(self, results: dict) -> bool:
        """Prüft auf kritische Werte"""
        critical_thresholds = {
            "primary_energy_demand": 300,  # kWh/(m²a)
            "u_value": 2.0,  # W/(m²K)
        }
        for key, threshold in critical_thresholds.items():
            if key in results and results[key] > threshold:
                return True
        return False


# ============================================================================
# WORKFLOW 3: Rechnung erstellen
# ============================================================================

@dataclass
class InvoiceGenerationInput:
    project_id: str
    customer_id: str
    invoice_type: str  # "final", "partial", "monthly"

@workflow.defn
class InvoiceGenerationWorkflow:
    """Automatische Rechnungserstellung"""

    @workflow.run
    async def run(self, input: InvoiceGenerationInput) -> dict:
        results = {}

        # 1. Abrechnungsdaten sammeln
        billing_data = await workflow.execute_activity(
            collect_billing_data,
            input.project_id,
            input.invoice_type,
            start_to_close_timeout=timedelta(seconds=60)
        )

        # 2. Positionen berechnen
        positions = await workflow.execute_activity(
            calculate_invoice_positions,
            billing_data,
            start_to_close_timeout=timedelta(seconds=30)
        )

        # 3. Rechnung generieren (PDF)
        results["invoice"] = await workflow.execute_activity(
            generate_invoice_pdf,
            positions,
            input.customer_id,
            start_to_close_timeout=timedelta(minutes=2)
        )

        # 4. Zur Freigabe vorlegen (NICHT automatisch senden!)
        results["approval_request"] = await workflow.execute_activity(
            create_invoice_approval_request,
            results["invoice"]["invoice_id"],
            start_to_close_timeout=timedelta(seconds=10)
        )

        # 5. Warten auf Freigabe (Human-in-the-Loop)
        approval = await workflow.wait_condition(
            lambda: self.approval_status is not None,
            timeout=timedelta(days=7)
        )

        if approval == "approved":
            # 6. Rechnung versenden
            results["sent"] = await workflow.execute_activity(
                send_invoice_email,
                results["invoice"]["invoice_id"],
                start_to_close_timeout=timedelta(seconds=60)
            )

            # 7. Buchhaltungs-Export
            results["accounting"] = await workflow.execute_activity(
                export_to_accounting,
                results["invoice"]["invoice_id"],
                start_to_close_timeout=timedelta(seconds=30)
            )

        return results

    @workflow.signal
    def set_approval(self, status: str):
        self.approval_status = status


# ============================================================================
# WORKFLOW 4: Automatische E-Mail mit Kontext
# ============================================================================

@dataclass
class ContextEmailInput:
    project_id: str
    recipient_id: str
    email_type: str  # "reminder", "update", "completion", "followup"
    context_data: dict

@workflow.defn
class ContextAwareEmailWorkflow:
    """Generiert kontextbezogene E-Mails mit KI-Unterstützung"""

    @workflow.run
    async def run(self, input: ContextEmailInput) -> dict:
        results = {}

        # 1. Projekt-Kontext laden
        context = await workflow.execute_activity(
            load_project_context,
            input.project_id,
            start_to_close_timeout=timedelta(seconds=30)
        )

        # 2. Kunden-Historie laden
        customer_history = await workflow.execute_activity(
            load_customer_communication_history,
            input.recipient_id,
            start_to_close_timeout=timedelta(seconds=30)
        )

        # 3. E-Mail-Text generieren (KI)
        email_content = await workflow.execute_activity(
            generate_email_content,
            {
                "type": input.email_type,
                "project_context": context,
                "customer_history": customer_history,
                "additional_data": input.context_data
            },
            start_to_close_timeout=timedelta(minutes=2)
        )

        results["draft"] = email_content

        # 4. Vorschau erstellen (für manuelle Prüfung optional)
        results["preview"] = await workflow.execute_activity(
            create_email_preview,
            email_content,
            start_to_close_timeout=timedelta(seconds=10)
        )

        # 5. Bei Standard-Mails: Automatisch senden
        if input.email_type in ["reminder", "update"]:
            results["sent"] = await workflow.execute_activity(
                send_email,
                email_content,
                input.recipient_id,
                start_to_close_timeout=timedelta(seconds=60)
            )
        else:
            # Zur manuellen Freigabe
            results["pending_approval"] = True

        return results
```

### 3.3 Workflow-Trigger Konfiguration

```python
# app/workflows/triggers.py
"""Workflow Trigger Konfiguration"""

from typing import Dict, List, Callable
from dataclasses import dataclass
from enum import Enum

class TriggerType(Enum):
    EVENT = "event"
    SCHEDULE = "schedule"
    CONDITION = "condition"
    MANUAL = "manual"

@dataclass
class WorkflowTrigger:
    name: str
    trigger_type: TriggerType
    workflow_name: str
    config: dict
    is_enabled: bool = True

# Vordefinierte Trigger
DEFAULT_TRIGGERS: List[WorkflowTrigger] = [
    # Event-basierte Trigger
    WorkflowTrigger(
        name="new_project_setup",
        trigger_type=TriggerType.EVENT,
        workflow_name="NewProjectWorkflow",
        config={
            "event": "project.created",
            "filter": {}  # Alle Projekte
        }
    ),
    WorkflowTrigger(
        name="calculation_completed",
        trigger_type=TriggerType.EVENT,
        workflow_name="CalculationCompletedWorkflow",
        config={
            "event": "calculation.completed",
            "filter": {"status": "success"}
        }
    ),
    WorkflowTrigger(
        name="report_generated",
        trigger_type=TriggerType.EVENT,
        workflow_name="ReportPostProcessingWorkflow",
        config={
            "event": "report.generated",
            "filter": {}
        }
    ),

    # Zeitgesteuerte Trigger
    WorkflowTrigger(
        name="daily_reminders",
        trigger_type=TriggerType.SCHEDULE,
        workflow_name="DailyReminderWorkflow",
        config={
            "cron": "0 8 * * 1-5",  # Mo-Fr um 8:00
            "timezone": "Europe/Berlin"
        }
    ),
    WorkflowTrigger(
        name="weekly_report",
        trigger_type=TriggerType.SCHEDULE,
        workflow_name="WeeklyReportWorkflow",
        config={
            "cron": "0 18 * * 5",  # Freitag 18:00
            "timezone": "Europe/Berlin"
        }
    ),
    WorkflowTrigger(
        name="monthly_billing",
        trigger_type=TriggerType.SCHEDULE,
        workflow_name="MonthlyBillingWorkflow",
        config={
            "cron": "0 9 1 * *",  # 1. des Monats, 9:00
            "timezone": "Europe/Berlin"
        }
    ),
    WorkflowTrigger(
        name="overdue_invoice_reminder",
        trigger_type=TriggerType.SCHEDULE,
        workflow_name="OverdueInvoiceReminderWorkflow",
        config={
            "cron": "0 10 * * *",  # Täglich 10:00
            "timezone": "Europe/Berlin"
        }
    ),

    # Bedingungsbasierte Trigger
    WorkflowTrigger(
        name="project_deadline_approaching",
        trigger_type=TriggerType.CONDITION,
        workflow_name="DeadlineReminderWorkflow",
        config={
            "condition": "project.deadline - now() < 7 days",
            "check_interval": "1 hour"
        }
    ),
    WorkflowTrigger(
        name="incomplete_project_followup",
        trigger_type=TriggerType.CONDITION,
        workflow_name="ProjectFollowupWorkflow",
        config={
            "condition": "project.status == 'in_progress' AND project.last_activity < now() - 14 days",
            "check_interval": "1 day"
        }
    ),
]


class TriggerManager:
    """Verwaltet Workflow-Trigger"""

    def __init__(self, workflow_client, event_bus):
        self.workflow_client = workflow_client
        self.event_bus = event_bus
        self.triggers: Dict[str, WorkflowTrigger] = {}

    async def register_triggers(self, triggers: List[WorkflowTrigger]):
        """Registriert Trigger"""
        for trigger in triggers:
            await self._setup_trigger(trigger)
            self.triggers[trigger.name] = trigger

    async def _setup_trigger(self, trigger: WorkflowTrigger):
        """Richtet einzelnen Trigger ein"""
        if trigger.trigger_type == TriggerType.EVENT:
            await self.event_bus.subscribe(
                trigger.config["event"],
                lambda data: self._handle_event_trigger(trigger, data)
            )

        elif trigger.trigger_type == TriggerType.SCHEDULE:
            # Cron-Job registrieren
            from apscheduler.schedulers.asyncio import AsyncIOScheduler
            from apscheduler.triggers.cron import CronTrigger

            scheduler = AsyncIOScheduler()
            scheduler.add_job(
                lambda: self._handle_schedule_trigger(trigger),
                CronTrigger.from_crontab(trigger.config["cron"]),
                timezone=trigger.config.get("timezone", "UTC")
            )

        elif trigger.trigger_type == TriggerType.CONDITION:
            # Periodische Bedingungsprüfung
            pass  # Implementierung via Scheduler

    async def _handle_event_trigger(self, trigger: WorkflowTrigger, event_data: dict):
        """Behandelt Event-Trigger"""
        if not trigger.is_enabled:
            return

        # Filter anwenden
        if not self._matches_filter(event_data, trigger.config.get("filter", {})):
            return

        # Workflow starten
        await self.workflow_client.start_workflow(
            trigger.workflow_name,
            event_data,
            id=f"{trigger.name}-{event_data.get('id', 'unknown')}"
        )

    def _matches_filter(self, data: dict, filter_config: dict) -> bool:
        """Prüft ob Daten zum Filter passen"""
        for key, value in filter_config.items():
            if data.get(key) != value:
                return False
        return True
```

---

## 4. Intelligente Formular-Ausfüllung

### 4.1 Auto-Complete Engine

```python
# app/services/auto_complete.py
"""Intelligente Formular-Vorausfüllung"""

from typing import Dict, Any, Optional
from dataclasses import dataclass
from app.models import Project, Building, Customer

@dataclass
class AutoCompleteContext:
    """Kontext für Auto-Complete"""
    project_id: Optional[str] = None
    customer_id: Optional[str] = None
    building_id: Optional[str] = None
    user_id: Optional[str] = None
    form_type: Optional[str] = None

class AutoCompleteEngine:
    """Engine für intelligente Formular-Vorausfüllung"""

    def __init__(self, session, cache):
        self.session = session
        self.cache = cache

    async def get_suggestions(
        self,
        field_name: str,
        context: AutoCompleteContext,
        partial_value: str = ""
    ) -> Dict[str, Any]:
        """Gibt Vorschläge für ein Formularfeld zurück"""

        # 1. Prüfe Cache für häufige Werte
        cache_key = f"autocomplete:{context.user_id}:{field_name}"
        cached = await self.cache.get(cache_key)
        if cached and not partial_value:
            return {"suggestions": cached, "source": "history"}

        # 2. Kontextbasierte Vorschläge
        suggestions = await self._get_contextual_suggestions(field_name, context)

        # 3. Filter bei Teileingabe
        if partial_value:
            suggestions = [
                s for s in suggestions
                if partial_value.lower() in str(s.get("value", "")).lower()
            ]

        return {"suggestions": suggestions[:10], "source": "context"}

    async def _get_contextual_suggestions(
        self,
        field_name: str,
        context: AutoCompleteContext
    ) -> list:
        """Ermittelt kontextbasierte Vorschläge"""

        suggestion_rules = {
            # Adress-Felder: Aus Kundendaten
            "street": self._suggest_from_customer_address,
            "zip_code": self._suggest_from_customer_address,
            "city": self._suggest_from_customer_address,

            # Gebäude-Felder: Aus ähnlichen Projekten
            "construction_year": self._suggest_from_similar_buildings,
            "building_type": self._suggest_building_types,
            "heating_system": self._suggest_heating_systems,

            # Bauteil-Felder: Aus Bauteilkatalog
            "wall_construction": self._suggest_wall_constructions,
            "window_type": self._suggest_window_types,
            "insulation_material": self._suggest_insulation_materials,

            # Berechnungs-Parameter
            "climate_zone": self._suggest_climate_zone,
            "usage_profile": self._suggest_usage_profiles,
        }

        handler = suggestion_rules.get(field_name, self._suggest_from_history)
        return await handler(context)

    async def _suggest_from_customer_address(self, context: AutoCompleteContext) -> list:
        """Adressdaten aus Kundenakte"""
        if not context.customer_id:
            return []

        customer = await self.session.get(Customer, context.customer_id)
        if not customer:
            return []

        return [{
            "value": {
                "street": customer.address_street,
                "zip_code": customer.address_zip,
                "city": customer.address_city
            },
            "label": f"{customer.address_street}, {customer.address_zip} {customer.address_city}",
            "confidence": 0.95
        }]

    async def _suggest_climate_zone(self, context: AutoCompleteContext) -> list:
        """Klimazone aus PLZ ermitteln"""
        if not context.project_id:
            return self._get_default_climate_zones()

        project = await self.session.get(Project, context.project_id)
        if not project or not project.location:
            return self._get_default_climate_zones()

        # PLZ zu Klimazone Mapping
        zip_code = project.location.zip_code
        climate_zone = self._plz_to_climate_zone(zip_code)

        return [{
            "value": climate_zone,
            "label": f"{climate_zone} (basierend auf PLZ {zip_code})",
            "confidence": 0.9
        }]

    def _plz_to_climate_zone(self, zip_code: str) -> str:
        """Mapping PLZ zu DWD Klimazone"""
        # Vereinfachtes Mapping, vollständige Tabelle in Datenbank
        first_two = zip_code[:2]

        climate_mapping = {
            # Norddeutschland
            "20": "TRY01", "21": "TRY01", "22": "TRY01", "23": "TRY02",
            "24": "TRY02", "25": "TRY02", "26": "TRY03", "27": "TRY03",
            # Mitteldeutschland
            "30": "TRY04", "31": "TRY04", "32": "TRY04", "33": "TRY05",
            "34": "TRY05", "35": "TRY05", "36": "TRY06", "37": "TRY06",
            # Süddeutschland
            "70": "TRY12", "71": "TRY12", "72": "TRY12", "73": "TRY13",
            "74": "TRY13", "75": "TRY13", "80": "TRY14", "81": "TRY14",
            "82": "TRY14", "83": "TRY15", "84": "TRY15", "85": "TRY15",
        }

        return climate_mapping.get(first_two, "TRY04")  # Default: Mitteldeutschland

    async def _suggest_from_similar_buildings(self, context: AutoCompleteContext) -> list:
        """Vorschläge aus ähnlichen Gebäuden"""
        if not context.building_id:
            return []

        building = await self.session.get(Building, context.building_id)
        if not building:
            return []

        # Finde ähnliche Gebäude
        similar = await self.session.execute("""
            SELECT construction_year, COUNT(*) as count
            FROM buildings
            WHERE building_type = :building_type
              AND ABS(heated_area - :heated_area) < 50
            GROUP BY construction_year
            ORDER BY count DESC
            LIMIT 5
        """, {
            "building_type": building.building_type,
            "heated_area": building.heated_area
        })

        return [
            {
                "value": row.construction_year,
                "label": f"{row.construction_year} (häufig bei ähnlichen Gebäuden)",
                "confidence": min(0.8, row.count / 10)
            }
            for row in similar
        ]

    async def prefill_form(
        self,
        form_type: str,
        context: AutoCompleteContext
    ) -> Dict[str, Any]:
        """Füllt gesamtes Formular vor"""

        form_fields = self._get_form_fields(form_type)
        prefilled = {}

        for field in form_fields:
            suggestions = await self.get_suggestions(field, context)
            if suggestions["suggestions"]:
                best = suggestions["suggestions"][0]
                if best.get("confidence", 0) > 0.7:
                    prefilled[field] = {
                        "value": best["value"],
                        "auto_filled": True,
                        "confidence": best["confidence"]
                    }

        return prefilled

    def _get_form_fields(self, form_type: str) -> list:
        """Gibt Felder für Formulartyp zurück"""
        form_definitions = {
            "building_data": [
                "construction_year", "building_type", "heated_area",
                "floors", "basement_type", "roof_type"
            ],
            "energy_system": [
                "heating_system", "heating_fuel", "hot_water_system",
                "ventilation_system", "solar_thermal", "photovoltaic"
            ],
            "building_envelope": [
                "wall_construction", "wall_insulation", "roof_insulation",
                "floor_insulation", "window_type", "door_type"
            ],
        }
        return form_definitions.get(form_type, [])
```

### 4.2 Template-Kombination

```python
# app/services/template_engine.py
"""Intelligente Projekt-Template-Kombination"""

from typing import List, Dict, Optional
from dataclasses import dataclass
from enum import Enum

class TemplateCategory(Enum):
    RESIDENTIAL = "residential"
    COMMERCIAL = "commercial"
    INDUSTRIAL = "industrial"
    PUBLIC = "public"

@dataclass
class ProjectTemplate:
    id: str
    name: str
    category: TemplateCategory
    building_types: List[str]
    calculation_types: List[str]
    report_modules: List[str]
    default_checklists: List[str]
    prerequisites: List[str]

class TemplateEngine:
    """Kombiniert Projekt-Templates intelligent"""

    TEMPLATES: Dict[str, ProjectTemplate] = {
        "efh_sanierung": ProjectTemplate(
            id="efh_sanierung",
            name="Einfamilienhaus Sanierung",
            category=TemplateCategory.RESIDENTIAL,
            building_types=["single_family"],
            calculation_types=["energy_demand", "heating_load", "economic"],
            report_modules=["isfp", "energy_certificate", "recommendations"],
            default_checklists=["before_visit", "on_site", "after_analysis"],
            prerequisites=[]
        ),
        "efh_neubau": ProjectTemplate(
            id="efh_neubau",
            name="Einfamilienhaus Neubau",
            category=TemplateCategory.RESIDENTIAL,
            building_types=["single_family"],
            calculation_types=["energy_demand", "heating_load", "summer_heat", "din18599"],
            report_modules=["energy_certificate_new", "kfw_nachweis"],
            default_checklists=["planning_check", "materials_check"],
            prerequisites=[]
        ),
        "mfh_sanierung": ProjectTemplate(
            id="mfh_sanierung",
            name="Mehrfamilienhaus Sanierung",
            category=TemplateCategory.RESIDENTIAL,
            building_types=["multi_family"],
            calculation_types=["energy_demand", "heating_load", "hydraulic_balance"],
            report_modules=["isfp", "tenant_info", "investment_plan"],
            default_checklists=["tenant_survey", "common_areas", "individual_units"],
            prerequisites=[]
        ),
        "nichtwohngebaeude": ProjectTemplate(
            id="nichtwohngebaeude",
            name="Nichtwohngebäude",
            category=TemplateCategory.COMMERCIAL,
            building_types=["office", "retail", "warehouse"],
            calculation_types=["din18599_full", "lighting", "cooling"],
            report_modules=["energy_certificate_nwg", "compliance_report"],
            default_checklists=["zone_definition", "usage_profiles", "technical_systems"],
            prerequisites=["zone_definition"]
        ),
        "energieaudit": ProjectTemplate(
            id="energieaudit",
            name="Energieaudit DIN 16247",
            category=TemplateCategory.COMMERCIAL,
            building_types=["office", "industrial", "retail"],
            calculation_types=["energy_audit", "consumption_analysis", "benchmark"],
            report_modules=["audit_report", "action_plan", "monitoring"],
            default_checklists=["audit_scope", "data_collection", "measurements"],
            prerequisites=["consumption_data_3years"]
        ),
    }

    async def suggest_templates(
        self,
        project_info: Dict
    ) -> List[ProjectTemplate]:
        """Schlägt passende Templates vor"""

        suggestions = []

        for template in self.TEMPLATES.values():
            score = self._calculate_match_score(template, project_info)
            if score > 0.5:
                suggestions.append((template, score))

        # Nach Score sortieren
        suggestions.sort(key=lambda x: x[1], reverse=True)
        return [t for t, _ in suggestions]

    def _calculate_match_score(
        self,
        template: ProjectTemplate,
        project_info: Dict
    ) -> float:
        """Berechnet Übereinstimmungs-Score"""
        score = 0.0
        max_score = 0.0

        # Gebäudetyp
        max_score += 1.0
        if project_info.get("building_type") in template.building_types:
            score += 1.0

        # Kategorie
        max_score += 0.5
        if project_info.get("category") == template.category.value:
            score += 0.5

        # Baujahr (Sanierung vs Neubau)
        max_score += 0.5
        construction_year = project_info.get("construction_year", 0)
        is_new_construction = construction_year >= 2020

        if is_new_construction and "neubau" in template.id:
            score += 0.5
        elif not is_new_construction and "sanierung" in template.id:
            score += 0.5

        return score / max_score if max_score > 0 else 0

    async def combine_templates(
        self,
        template_ids: List[str]
    ) -> Dict:
        """Kombiniert mehrere Templates"""

        combined = {
            "building_types": set(),
            "calculation_types": set(),
            "report_modules": set(),
            "checklists": set(),
            "prerequisites": set(),
        }

        for template_id in template_ids:
            template = self.TEMPLATES.get(template_id)
            if not template:
                continue

            combined["building_types"].update(template.building_types)
            combined["calculation_types"].update(template.calculation_types)
            combined["report_modules"].update(template.report_modules)
            combined["checklists"].update(template.default_checklists)
            combined["prerequisites"].update(template.prerequisites)

        # In Listen konvertieren
        return {k: list(v) for k, v in combined.items()}

    async def create_project_from_template(
        self,
        template_id: str,
        project_data: Dict,
        session
    ) -> Dict:
        """Erstellt Projekt aus Template"""

        template = self.TEMPLATES.get(template_id)
        if not template:
            raise ValueError(f"Template {template_id} nicht gefunden")

        # Projekt erstellen
        project = Project(
            name=project_data["name"],
            template_id=template_id,
            **project_data
        )
        session.add(project)

        # Standard-Berechnungen vorbereiten
        for calc_type in template.calculation_types:
            calculation = Calculation(
                project=project,
                calculation_type=calc_type,
                status="pending"
            )
            session.add(calculation)

        # Checklisten erstellen
        for checklist_id in template.default_checklists:
            checklist = await self._create_checklist_from_template(
                checklist_id, project
            )
            session.add(checklist)

        await session.commit()

        return {
            "project_id": str(project.id),
            "template_id": template_id,
            "calculations_created": len(template.calculation_types),
            "checklists_created": len(template.default_checklists)
        }
```

---

## 5. Automatische Dokumentenerstellung

### 5.1 Report-Baustein-System

```python
# app/services/report_blocks.py
"""Intelligentes Report-Baustein-System"""

from typing import Dict, List, Optional
from dataclasses import dataclass, field
from enum import Enum
import jinja2

class BlockType(Enum):
    HEADER = "header"
    SUMMARY = "summary"
    CALCULATION_RESULT = "calculation_result"
    RECOMMENDATION = "recommendation"
    GRAPH = "graph"
    TABLE = "table"
    IMAGE = "image"
    TEXT = "text"
    CHECKLIST = "checklist"

@dataclass
class ReportBlock:
    id: str
    type: BlockType
    title: str
    template: str
    data_requirements: List[str]
    conditions: Dict = field(default_factory=dict)
    order: int = 0

class ReportBlockEngine:
    """Verwaltet und generiert Report-Bausteine"""

    def __init__(self, template_env: jinja2.Environment):
        self.template_env = template_env
        self.blocks: Dict[str, ReportBlock] = {}
        self._load_default_blocks()

    def _load_default_blocks(self):
        """Lädt Standard-Bausteine"""
        default_blocks = [
            ReportBlock(
                id="header_isfp",
                type=BlockType.HEADER,
                title="iSFP Kopfbereich",
                template="blocks/isfp_header.html",
                data_requirements=["project", "building", "consultant"],
                order=1
            ),
            ReportBlock(
                id="building_summary",
                type=BlockType.SUMMARY,
                title="Gebäudezusammenfassung",
                template="blocks/building_summary.html",
                data_requirements=["building", "envelope_data"],
                order=10
            ),
            ReportBlock(
                id="energy_balance",
                type=BlockType.CALCULATION_RESULT,
                title="Energiebilanz",
                template="blocks/energy_balance.html",
                data_requirements=["calculation_results.energy_demand"],
                order=20
            ),
            ReportBlock(
                id="heating_load_result",
                type=BlockType.CALCULATION_RESULT,
                title="Heizlastberechnung",
                template="blocks/heating_load.html",
                data_requirements=["calculation_results.heating_load"],
                order=21
            ),
            ReportBlock(
                id="sankey_diagram",
                type=BlockType.GRAPH,
                title="Energiefluss-Diagramm",
                template="blocks/sankey.html",
                data_requirements=["calculation_results.energy_flow"],
                order=25
            ),
            ReportBlock(
                id="component_table",
                type=BlockType.TABLE,
                title="Bauteilübersicht",
                template="blocks/component_table.html",
                data_requirements=["building.components"],
                order=30
            ),
            ReportBlock(
                id="recommendation_wall",
                type=BlockType.RECOMMENDATION,
                title="Maßnahme: Außenwanddämmung",
                template="blocks/recommendation_wall.html",
                data_requirements=["recommendations.wall"],
                conditions={"has_wall_potential": True},
                order=40
            ),
            ReportBlock(
                id="recommendation_roof",
                type=BlockType.RECOMMENDATION,
                title="Maßnahme: Dachdämmung",
                template="blocks/recommendation_roof.html",
                data_requirements=["recommendations.roof"],
                conditions={"has_roof_potential": True},
                order=41
            ),
            ReportBlock(
                id="recommendation_windows",
                type=BlockType.RECOMMENDATION,
                title="Maßnahme: Fenstererneuerung",
                template="blocks/recommendation_windows.html",
                data_requirements=["recommendations.windows"],
                conditions={"has_window_potential": True},
                order=42
            ),
            ReportBlock(
                id="recommendation_heating",
                type=BlockType.RECOMMENDATION,
                title="Maßnahme: Heizungstausch",
                template="blocks/recommendation_heating.html",
                data_requirements=["recommendations.heating"],
                conditions={"has_heating_potential": True},
                order=43
            ),
            ReportBlock(
                id="economic_analysis",
                type=BlockType.TABLE,
                title="Wirtschaftlichkeitsberechnung",
                template="blocks/economic_analysis.html",
                data_requirements=["calculation_results.economic"],
                order=50
            ),
            ReportBlock(
                id="funding_options",
                type=BlockType.TABLE,
                title="Fördermöglichkeiten",
                template="blocks/funding.html",
                data_requirements=["funding_data"],
                order=55
            ),
            ReportBlock(
                id="renovation_roadmap",
                type=BlockType.GRAPH,
                title="Sanierungsfahrplan",
                template="blocks/roadmap.html",
                data_requirements=["roadmap_data"],
                order=60
            ),
        ]

        for block in default_blocks:
            self.blocks[block.id] = block

    async def generate_report(
        self,
        report_type: str,
        project_data: Dict,
        options: Dict = None
    ) -> str:
        """Generiert kompletten Bericht aus Bausteinen"""

        # 1. Relevante Bausteine ermitteln
        available_blocks = self._get_blocks_for_report_type(report_type)

        # 2. Nach Bedingungen filtern
        applicable_blocks = [
            block for block in available_blocks
            if self._check_conditions(block, project_data)
        ]

        # 3. Nach Order sortieren
        applicable_blocks.sort(key=lambda b: b.order)

        # 4. Bausteine rendern
        rendered_blocks = []
        for block in applicable_blocks:
            try:
                html = await self._render_block(block, project_data)
                rendered_blocks.append({
                    "id": block.id,
                    "html": html,
                    "type": block.type.value
                })
            except Exception as e:
                # Block überspringen bei Fehler, aber loggen
                print(f"Fehler bei Block {block.id}: {e}")

        # 5. Layout-Template mit Bausteinen füllen
        layout_template = self.template_env.get_template(
            f"layouts/{report_type}.html"
        )

        return layout_template.render(
            blocks=rendered_blocks,
            project=project_data.get("project"),
            meta=options or {}
        )

    def _get_blocks_for_report_type(self, report_type: str) -> List[ReportBlock]:
        """Gibt Bausteine für Berichtstyp zurück"""
        report_block_mapping = {
            "isfp": [
                "header_isfp", "building_summary", "energy_balance",
                "sankey_diagram", "component_table",
                "recommendation_wall", "recommendation_roof",
                "recommendation_windows", "recommendation_heating",
                "economic_analysis", "funding_options", "renovation_roadmap"
            ],
            "energy_certificate": [
                "header_certificate", "building_summary", "energy_balance",
                "heating_load_result", "component_table"
            ],
            "quick_check": [
                "building_summary", "energy_balance", "key_recommendations"
            ],
        }

        block_ids = report_block_mapping.get(report_type, [])
        return [self.blocks[bid] for bid in block_ids if bid in self.blocks]

    def _check_conditions(self, block: ReportBlock, data: Dict) -> bool:
        """Prüft ob Baustein-Bedingungen erfüllt sind"""
        for condition_key, expected_value in block.conditions.items():
            actual_value = self._get_nested_value(data, condition_key)
            if actual_value != expected_value:
                return False
        return True

    async def _render_block(self, block: ReportBlock, data: Dict) -> str:
        """Rendert einzelnen Baustein"""
        template = self.template_env.get_template(block.template)

        # Daten für Block extrahieren
        block_data = {}
        for requirement in block.data_requirements:
            value = self._get_nested_value(data, requirement)
            if value is not None:
                block_data[requirement.split(".")[-1]] = value

        return template.render(**block_data)

    def _get_nested_value(self, data: Dict, key_path: str):
        """Holt verschachtelten Wert aus Dict"""
        keys = key_path.split(".")
        value = data
        for key in keys:
            if isinstance(value, dict):
                value = value.get(key)
            else:
                return None
        return value
```

### 5.2 KI-gestützte Textgenerierung

```python
# app/services/ai_text_generator.py
"""KI-gestützte Textgenerierung für Berichte"""

from typing import Dict, List, Optional
from app.services.ollama_client import OllamaClient

class AITextGenerator:
    """Generiert kontextbezogene Texte für Berichte"""

    def __init__(self, ollama_client: OllamaClient):
        self.ollama = ollama_client

    async def generate_recommendation_text(
        self,
        measure_type: str,
        building_data: Dict,
        calculation_results: Dict,
        style: str = "professional"
    ) -> str:
        """Generiert Maßnahmenempfehlung"""

        prompt = f"""
        Erstelle eine professionelle Maßnahmenempfehlung für einen Energieberatungsbericht.

        Maßnahme: {measure_type}

        Gebäudedaten:
        - Baujahr: {building_data.get('construction_year')}
        - Typ: {building_data.get('building_type')}
        - Beheizte Fläche: {building_data.get('heated_area')} m²

        Berechnungsergebnisse:
        - Aktueller U-Wert: {calculation_results.get('current_u_value')} W/(m²K)
        - Ziel U-Wert: {calculation_results.get('target_u_value')} W/(m²K)
        - Einsparpotenzial: {calculation_results.get('savings_potential')} kWh/a

        Anforderungen:
        - Stil: {style}
        - Sprache: Deutsch
        - Länge: 2-3 Absätze
        - Keine Marketing-Sprache
        - Technisch korrekt
        - Mit konkreten Angaben

        Gib nur den Text zurück, ohne Einleitung oder Erklärung.
        """

        response = await self.ollama.generate(
            prompt=prompt,
            temperature=0.3,  # Wenig Variation für konsistente Ausgabe
            max_tokens=500
        )

        return response.text

    async def generate_summary(
        self,
        project_data: Dict,
        calculation_results: Dict
    ) -> str:
        """Generiert Projektzusammenfassung"""

        prompt = f"""
        Erstelle eine prägnante Zusammenfassung für einen Energieberatungsbericht.

        Projekt: {project_data.get('name')}
        Gebäude: {project_data.get('building', {}).get('type')}
        Standort: {project_data.get('location', {}).get('city')}

        Kernergebnisse:
        - Primärenergiebedarf: {calculation_results.get('primary_energy_demand')} kWh/(m²a)
        - Energieeffizienzklasse: {calculation_results.get('efficiency_class')}
        - CO2-Emissionen: {calculation_results.get('co2_emissions')} kg/(m²a)
        - Empfohlene Maßnahmen: {', '.join(calculation_results.get('recommended_measures', []))}

        Erstelle eine Zusammenfassung in 3-4 Sätzen, die:
        1. Den Ist-Zustand beschreibt
        2. Das Einsparpotenzial hervorhebt
        3. Die wichtigsten Maßnahmen nennt

        Nur den Text zurückgeben.
        """

        response = await self.ollama.generate(
            prompt=prompt,
            temperature=0.2,
            max_tokens=300
        )

        return response.text

    async def generate_email_body(
        self,
        email_type: str,
        context: Dict
    ) -> Dict[str, str]:
        """Generiert E-Mail-Text mit Betreff"""

        email_templates = {
            "project_update": """
                Kontext: Projektfortschritt mitteilen
                Empfänger: {customer_name}
                Projekt: {project_name}
                Status: {project_status}
                Nächste Schritte: {next_steps}
            """,
            "report_ready": """
                Kontext: Bericht ist fertig
                Empfänger: {customer_name}
                Projekt: {project_name}
                Berichtstyp: {report_type}
            """,
            "appointment_reminder": """
                Kontext: Terminerinnerung
                Empfänger: {customer_name}
                Termin: {appointment_date}
                Ort: {location}
                Zweck: {purpose}
            """,
        }

        template = email_templates.get(email_type, "")
        filled_template = template.format(**context)

        prompt = f"""
        Erstelle eine professionelle E-Mail basierend auf folgendem Kontext:

        {filled_template}

        Anforderungen:
        - Professioneller, freundlicher Ton
        - Deutsch
        - Keine übertriebene Höflichkeit
        - Konkret und handlungsorientiert
        - Mit passender Anrede und Grußformel

        Format der Ausgabe (JSON):
        {{
            "subject": "Betreffzeile",
            "body": "E-Mail-Text"
        }}
        """

        response = await self.ollama.generate(
            prompt=prompt,
            temperature=0.4,
            max_tokens=500,
            format="json"
        )

        return response.json()
```

---

## 6. Monitoring und Benachrichtigungen

### 6.1 Automatische Benachrichtigungen

```python
# app/services/notifications.py
"""Automatisches Benachrichtigungssystem"""

from typing import Dict, List, Optional
from dataclasses import dataclass
from enum import Enum
from datetime import datetime, timedelta

class NotificationType(Enum):
    INFO = "info"
    WARNING = "warning"
    ACTION_REQUIRED = "action_required"
    SUCCESS = "success"
    ERROR = "error"

class NotificationChannel(Enum):
    IN_APP = "in_app"
    EMAIL = "email"
    PUSH = "push"
    WEBHOOK = "webhook"

@dataclass
class NotificationRule:
    id: str
    name: str
    trigger_event: str
    condition: str
    notification_type: NotificationType
    channels: List[NotificationChannel]
    template: str
    recipients: List[str]  # "owner", "assignee", "team", "customer"
    cooldown: Optional[timedelta] = None

class NotificationEngine:
    """Engine für automatische Benachrichtigungen"""

    DEFAULT_RULES: List[NotificationRule] = [
        # Projekt-Benachrichtigungen
        NotificationRule(
            id="project_deadline_7d",
            name="Projektfrist in 7 Tagen",
            trigger_event="schedule.daily",
            condition="project.deadline - now() < 7 days AND project.status != 'completed'",
            notification_type=NotificationType.WARNING,
            channels=[NotificationChannel.IN_APP, NotificationChannel.EMAIL],
            template="notifications/deadline_approaching.html",
            recipients=["owner", "assignee"],
            cooldown=timedelta(days=1)
        ),
        NotificationRule(
            id="project_deadline_1d",
            name="Projektfrist morgen",
            trigger_event="schedule.daily",
            condition="project.deadline - now() < 1 day AND project.status != 'completed'",
            notification_type=NotificationType.ACTION_REQUIRED,
            channels=[NotificationChannel.IN_APP, NotificationChannel.EMAIL, NotificationChannel.PUSH],
            template="notifications/deadline_tomorrow.html",
            recipients=["owner", "assignee"],
            cooldown=timedelta(hours=12)
        ),

        # Berechnungs-Benachrichtigungen
        NotificationRule(
            id="calculation_completed",
            name="Berechnung abgeschlossen",
            trigger_event="calculation.completed",
            condition="True",
            notification_type=NotificationType.SUCCESS,
            channels=[NotificationChannel.IN_APP],
            template="notifications/calculation_done.html",
            recipients=["owner"]
        ),
        NotificationRule(
            id="calculation_critical",
            name="Kritische Berechnungswerte",
            trigger_event="calculation.completed",
            condition="result.primary_energy_demand > 300",
            notification_type=NotificationType.WARNING,
            channels=[NotificationChannel.IN_APP, NotificationChannel.EMAIL],
            template="notifications/critical_values.html",
            recipients=["owner", "team"]
        ),

        # Rechnungs-Benachrichtigungen
        NotificationRule(
            id="invoice_overdue",
            name="Rechnung überfällig",
            trigger_event="schedule.daily",
            condition="invoice.due_date < now() AND invoice.status == 'sent'",
            notification_type=NotificationType.ACTION_REQUIRED,
            channels=[NotificationChannel.IN_APP, NotificationChannel.EMAIL],
            template="notifications/invoice_overdue.html",
            recipients=["owner"],
            cooldown=timedelta(days=3)
        ),
        NotificationRule(
            id="invoice_paid",
            name="Zahlung eingegangen",
            trigger_event="invoice.paid",
            condition="True",
            notification_type=NotificationType.SUCCESS,
            channels=[NotificationChannel.IN_APP],
            template="notifications/payment_received.html",
            recipients=["owner"]
        ),

        # Kunden-Benachrichtigungen
        NotificationRule(
            id="customer_inactive",
            name="Inaktiver Kunde",
            trigger_event="schedule.weekly",
            condition="customer.last_activity < now() - 90 days",
            notification_type=NotificationType.INFO,
            channels=[NotificationChannel.IN_APP],
            template="notifications/inactive_customer.html",
            recipients=["owner"],
            cooldown=timedelta(days=30)
        ),

        # System-Benachrichtigungen
        NotificationRule(
            id="report_generated",
            name="Bericht erstellt",
            trigger_event="report.generated",
            condition="True",
            notification_type=NotificationType.SUCCESS,
            channels=[NotificationChannel.IN_APP],
            template="notifications/report_ready.html",
            recipients=["owner"]
        ),
    ]

    def __init__(self, session, notification_sender, template_engine):
        self.session = session
        self.sender = notification_sender
        self.templates = template_engine
        self.rules = {r.id: r for r in self.DEFAULT_RULES}

    async def process_event(self, event_type: str, event_data: Dict):
        """Verarbeitet Event und sendet Benachrichtigungen"""

        # Relevante Regeln finden
        applicable_rules = [
            rule for rule in self.rules.values()
            if rule.trigger_event == event_type
        ]

        for rule in applicable_rules:
            # Bedingung prüfen
            if not await self._evaluate_condition(rule.condition, event_data):
                continue

            # Cooldown prüfen
            if rule.cooldown and await self._is_in_cooldown(rule.id, event_data):
                continue

            # Empfänger ermitteln
            recipients = await self._resolve_recipients(rule.recipients, event_data)

            # Benachrichtigung erstellen und senden
            for recipient in recipients:
                notification = await self._create_notification(rule, event_data, recipient)
                await self._send_notification(notification, rule.channels)

            # Cooldown setzen
            if rule.cooldown:
                await self._set_cooldown(rule.id, event_data, rule.cooldown)

    async def _evaluate_condition(self, condition: str, data: Dict) -> bool:
        """Evaluiert Regel-Bedingung sicher"""
        # Sichere Evaluation mit eingeschränktem Namespace
        safe_namespace = {
            "now": datetime.utcnow,
            "timedelta": timedelta,
            **data
        }

        try:
            return eval(condition, {"__builtins__": {}}, safe_namespace)
        except Exception:
            return False

    async def _resolve_recipients(self, recipient_types: List[str], data: Dict) -> List[str]:
        """Löst Empfänger-Typen in User-IDs auf"""
        recipients = set()

        for recipient_type in recipient_types:
            if recipient_type == "owner":
                if "project" in data:
                    recipients.add(data["project"].owner_id)
            elif recipient_type == "assignee":
                if "project" in data and data["project"].assignee_id:
                    recipients.add(data["project"].assignee_id)
            elif recipient_type == "team":
                # Alle Team-Mitglieder
                if "project" in data:
                    team = await self._get_project_team(data["project"].id)
                    recipients.update(team)
            elif recipient_type == "customer":
                if "customer" in data:
                    recipients.add(data["customer"].contact_email)

        return list(recipients)
```

---

## 7. Checkliste

```
✓ Automatisierungs-Philosophie (0% Bürokratie)
✓ Automatische Zeiterfassung ohne Benutzerinteraktion
✓ Workflow-Engine mit Temporal
✓ Vordefinierte Workflows (Projekt, Berechnung, Rechnung, E-Mail)
✓ Trigger-Konfiguration (Event, Schedule, Condition)
✓ Intelligente Formular-Vorausfüllung
✓ Template-Kombination für Projekte
✓ Report-Baustein-System
✓ KI-gestützte Textgenerierung
✓ Automatisches Benachrichtigungssystem
✓ Monitoring und Alerting
```

---

*Letzte Aktualisierung: 2025-11-25*
*Version: 1.0.0*
