# Monetarisierungsmodelle EnergieberaterPRO

## Referenzen
- [Stripe Billing](https://stripe.com/billing)
- [SaaS Pricing Models](https://www.priceintelligently.com/)
- [Usage-Based Billing](https://www.chargebee.com/resources/glossaries/usage-based-pricing/)

---

## 1. Übersicht Geschäftsmodell

### 1.1 Monetarisierungsstrategie

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      MONETARISIERUNGSSTRATEGIE                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  FLEXIBLES HYBRID-MODELL                                                    │
│  ═══════════════════════                                                    │
│                                                                             │
│  Organisation wählt Abrechnungsmodell:                                      │
│                                                                             │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐           │
│  │   FLAT RATE     │   │  PAY-PER-USE    │   │   SUBSCRIPTION  │           │
│  │                 │   │                 │   │                 │           │
│  │  Fester Preis   │   │ Zahle nur was   │   │  Monatlich/     │           │
│  │  pro Monat/Jahr │   │ du nutzt        │   │  Jährlich       │           │
│  │                 │   │                 │   │                 │           │
│  │  ✓ Planbar      │   │  ✓ Flexibel     │   │  ✓ Skalierbar   │           │
│  │  ✓ Unbegrenzt   │   │  ✓ Einstieg     │   │  ✓ Features     │           │
│  │  ✓ All-in-One   │   │  ✓ Kostenkontro.│   │  ✓ Support      │           │
│  └─────────────────┘   └─────────────────┘   └─────────────────┘           │
│                                                                             │
│  PRINZIPIEN:                                                                │
│  ─────────────────────────────────────────────────────────────────────────  │
│  • Keine versteckten Kosten                                                 │
│  • Transparente Preisgestaltung                                             │
│  • Modul-basierte Freischaltung                                             │
│  • Automatische Rechnungsstellung (nach Freigabe)                           │
│  • Keine Lock-in-Effekte (Export jederzeit möglich)                         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Preisstruktur-Übersicht

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PREISSTRUKTUR                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  BASIS (Kostenlos)           PROFESSIONAL              ENTERPRISE           │
│  ──────────────────          ────────────────          ─────────────────    │
│  • 3 Projekte/Monat          • Unbegrenzt Projekte     • Alles aus Pro      │
│  • Basis-Berechnungen        • Alle Berechnungen       • Multi-Tenant       │
│  • 1 Benutzer                • 5 Benutzer              • Unbegrenzt User    │
│  • Community Support         • E-Mail Support          • Priority Support   │
│                              • API Zugang              • SLA 99.9%          │
│  €0                          • KI-Funktionen           • On-Premise Option  │
│                                                        • Dedicated Instance │
│                              ab €49/Monat                                   │
│                                                        Individuell          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Abrechnungsmodelle im Detail

### 2.1 Modell A: Flatrate

```python
# app/billing/models/flatrate.py
"""Flatrate-Abrechnungsmodell"""

from dataclasses import dataclass
from decimal import Decimal
from typing import List, Optional
from enum import Enum
from datetime import date

class FlatrateInterval(Enum):
    MONTHLY = "monthly"
    QUARTERLY = "quarterly"
    YEARLY = "yearly"

@dataclass
class FlatratePlan:
    """Flatrate-Tarifplan"""
    id: str
    name: str
    interval: FlatrateInterval
    base_price: Decimal
    included_users: int
    included_storage_gb: int
    features: List[str]
    modules: List[str]
    discount_yearly: Decimal = Decimal("0.15")  # 15% Rabatt bei Jahresabo

    def get_price(self, interval: FlatrateInterval) -> Decimal:
        """Berechnet Preis für Intervall"""
        if interval == FlatrateInterval.MONTHLY:
            return self.base_price
        elif interval == FlatrateInterval.QUARTERLY:
            return self.base_price * 3 * Decimal("0.95")  # 5% Rabatt
        elif interval == FlatrateInterval.YEARLY:
            return self.base_price * 12 * (1 - self.discount_yearly)
        return self.base_price

# Vordefinierte Pläne
FLATRATE_PLANS = {
    "starter": FlatratePlan(
        id="starter",
        name="Starter",
        interval=FlatrateInterval.MONTHLY,
        base_price=Decimal("49.00"),
        included_users=2,
        included_storage_gb=10,
        features=[
            "Projektverwaltung",
            "Basis-Berechnungen",
            "Standard-Berichte",
            "E-Mail-Support",
        ],
        modules=[
            "projects",
            "calculations_basic",
            "reports_basic",
            "customers",
        ]
    ),
    "professional": FlatratePlan(
        id="professional",
        name="Professional",
        interval=FlatrateInterval.MONTHLY,
        base_price=Decimal("99.00"),
        included_users=5,
        included_storage_gb=50,
        features=[
            "Alle Starter-Features",
            "Erweiterte Berechnungen",
            "iSFP-Modul",
            "KI-Textassistent",
            "API-Zugang",
            "Prioritäts-Support",
        ],
        modules=[
            "projects",
            "calculations_full",
            "reports_full",
            "customers",
            "isfp",
            "ai_assistant",
            "api_access",
            "time_tracking",
        ]
    ),
    "business": FlatratePlan(
        id="business",
        name="Business",
        interval=FlatrateInterval.MONTHLY,
        base_price=Decimal("199.00"),
        included_users=15,
        included_storage_gb=200,
        features=[
            "Alle Professional-Features",
            "Unbegrenzte Projekte",
            "Energieaudit DIN 16247",
            "Team-Verwaltung",
            "Workflow-Automatisierung",
            "White-Label Option",
            "Phone-Support",
        ],
        modules=[
            "*",  # Alle Module
        ]
    ),
}


class FlatrateSubscription:
    """Verwaltet Flatrate-Abonnements"""

    def __init__(self, session, payment_provider):
        self.session = session
        self.payment = payment_provider

    async def create_subscription(
        self,
        organization_id: str,
        plan_id: str,
        interval: FlatrateInterval,
        payment_method_id: str
    ) -> dict:
        """Erstellt neues Flatrate-Abonnement"""

        plan = FLATRATE_PLANS.get(plan_id)
        if not plan:
            raise ValueError(f"Plan {plan_id} nicht gefunden")

        price = plan.get_price(interval)

        # Zahlungsprovider-Subscription erstellen
        subscription = await self.payment.create_subscription(
            customer_id=organization_id,
            price_amount=int(price * 100),  # Cent
            interval=interval.value,
            payment_method_id=payment_method_id
        )

        # In DB speichern
        db_subscription = Subscription(
            organization_id=organization_id,
            plan_id=plan_id,
            interval=interval.value,
            status="active",
            current_period_start=date.today(),
            external_id=subscription["id"]
        )
        self.session.add(db_subscription)
        await self.session.commit()

        # Module freischalten
        await self._activate_modules(organization_id, plan.modules)

        return {
            "subscription_id": str(db_subscription.id),
            "plan": plan.name,
            "price": str(price),
            "interval": interval.value,
            "modules_activated": plan.modules
        }

    async def _activate_modules(self, organization_id: str, modules: List[str]):
        """Schaltet Module für Organisation frei"""
        if "*" in modules:
            # Alle Module freischalten
            await self.session.execute("""
                UPDATE organization_modules
                SET is_active = true
                WHERE organization_id = :org_id
            """, {"org_id": organization_id})
        else:
            for module in modules:
                await self.session.execute("""
                    INSERT INTO organization_modules (organization_id, module_id, is_active)
                    VALUES (:org_id, :module, true)
                    ON CONFLICT (organization_id, module_id)
                    DO UPDATE SET is_active = true
                """, {"org_id": organization_id, "module": module})
```

### 2.2 Modell B: Pay-per-Use

```python
# app/billing/models/usage_based.py
"""Nutzungsbasierte Abrechnung"""

from dataclasses import dataclass
from decimal import Decimal
from typing import Dict, List
from datetime import datetime, date
from enum import Enum

class UsageUnit(Enum):
    PROJECT = "project"
    CALCULATION = "calculation"
    REPORT = "report"
    AI_REQUEST = "ai_request"
    STORAGE_GB = "storage_gb"
    API_CALL = "api_call"
    USER_SEAT = "user_seat"

@dataclass
class UsagePrice:
    """Preis pro Nutzungseinheit"""
    unit: UsageUnit
    price_per_unit: Decimal
    free_tier: int = 0
    volume_discounts: Dict[int, Decimal] = None  # {threshold: discount_percent}

# Preisliste
USAGE_PRICES: Dict[UsageUnit, UsagePrice] = {
    UsageUnit.PROJECT: UsagePrice(
        unit=UsageUnit.PROJECT,
        price_per_unit=Decimal("9.90"),
        free_tier=3,
        volume_discounts={10: Decimal("0.10"), 50: Decimal("0.20"), 100: Decimal("0.30")}
    ),
    UsageUnit.CALCULATION: UsagePrice(
        unit=UsageUnit.CALCULATION,
        price_per_unit=Decimal("2.50"),
        free_tier=10,
        volume_discounts={50: Decimal("0.15"), 200: Decimal("0.25")}
    ),
    UsageUnit.REPORT: UsagePrice(
        unit=UsageUnit.REPORT,
        price_per_unit=Decimal("4.90"),
        free_tier=3,
        volume_discounts={20: Decimal("0.10"), 100: Decimal("0.20")}
    ),
    UsageUnit.AI_REQUEST: UsagePrice(
        unit=UsageUnit.AI_REQUEST,
        price_per_unit=Decimal("0.10"),
        free_tier=100,
        volume_discounts={1000: Decimal("0.20"), 10000: Decimal("0.40")}
    ),
    UsageUnit.STORAGE_GB: UsagePrice(
        unit=UsageUnit.STORAGE_GB,
        price_per_unit=Decimal("0.50"),
        free_tier=5,
        volume_discounts={100: Decimal("0.25")}
    ),
    UsageUnit.API_CALL: UsagePrice(
        unit=UsageUnit.API_CALL,
        price_per_unit=Decimal("0.001"),
        free_tier=10000,
        volume_discounts={100000: Decimal("0.30")}
    ),
}


class UsageTracker:
    """Trackt Nutzung für Abrechnung"""

    def __init__(self, session, cache):
        self.session = session
        self.cache = cache

    async def record_usage(
        self,
        organization_id: str,
        unit: UsageUnit,
        quantity: int = 1,
        metadata: Dict = None
    ):
        """Zeichnet Nutzung auf"""

        usage_record = UsageRecord(
            organization_id=organization_id,
            unit=unit.value,
            quantity=quantity,
            timestamp=datetime.utcnow(),
            metadata=metadata or {}
        )
        self.session.add(usage_record)

        # Cache aktualisieren für schnelle Abfragen
        cache_key = f"usage:{organization_id}:{unit.value}:{date.today().isoformat()}"
        await self.cache.incr(cache_key, quantity)
        await self.cache.expire(cache_key, 86400 * 35)  # 35 Tage

        await self.session.commit()

    async def get_usage_summary(
        self,
        organization_id: str,
        period_start: date,
        period_end: date
    ) -> Dict:
        """Ermittelt Nutzungszusammenfassung für Zeitraum"""

        result = await self.session.execute("""
            SELECT unit, SUM(quantity) as total
            FROM usage_records
            WHERE organization_id = :org_id
              AND timestamp >= :start
              AND timestamp < :end
            GROUP BY unit
        """, {
            "org_id": organization_id,
            "start": period_start,
            "end": period_end
        })

        usage = {row.unit: row.total for row in result}

        return {
            "organization_id": organization_id,
            "period": {
                "start": period_start.isoformat(),
                "end": period_end.isoformat()
            },
            "usage": usage
        }


class UsageBillingCalculator:
    """Berechnet nutzungsbasierte Rechnungen"""

    def calculate_invoice(
        self,
        usage_summary: Dict,
        prices: Dict[UsageUnit, UsagePrice] = None
    ) -> Dict:
        """Berechnet Rechnungspositionen aus Nutzung"""

        prices = prices or USAGE_PRICES
        line_items = []
        total = Decimal("0")

        for unit_str, quantity in usage_summary.get("usage", {}).items():
            unit = UsageUnit(unit_str)
            price_info = prices.get(unit)

            if not price_info:
                continue

            # Free Tier abziehen
            billable_quantity = max(0, quantity - price_info.free_tier)

            if billable_quantity == 0:
                continue

            # Volumenrabatt ermitteln
            discount = Decimal("0")
            if price_info.volume_discounts:
                for threshold, disc in sorted(price_info.volume_discounts.items(), reverse=True):
                    if billable_quantity >= threshold:
                        discount = disc
                        break

            # Preis berechnen
            unit_price = price_info.price_per_unit * (1 - discount)
            line_total = unit_price * billable_quantity

            line_items.append({
                "description": self._get_unit_description(unit),
                "quantity": billable_quantity,
                "unit_price": str(unit_price),
                "discount_percent": str(discount * 100),
                "total": str(line_total)
            })

            total += line_total

        return {
            "line_items": line_items,
            "subtotal": str(total),
            "tax_rate": "19",
            "tax_amount": str(total * Decimal("0.19")),
            "total": str(total * Decimal("1.19"))
        }

    def _get_unit_description(self, unit: UsageUnit) -> str:
        """Gibt lesbare Beschreibung für Einheit zurück"""
        descriptions = {
            UsageUnit.PROJECT: "Projekte",
            UsageUnit.CALCULATION: "Berechnungen",
            UsageUnit.REPORT: "Berichte",
            UsageUnit.AI_REQUEST: "KI-Anfragen",
            UsageUnit.STORAGE_GB: "Speicher (GB)",
            UsageUnit.API_CALL: "API-Aufrufe",
            UsageUnit.USER_SEAT: "Benutzerlizenzen",
        }
        return descriptions.get(unit, unit.value)
```

### 2.3 Modell C: Subscription mit Tiers

```python
# app/billing/models/subscription.py
"""Gestaffeltes Abo-Modell"""

from dataclasses import dataclass, field
from decimal import Decimal
from typing import Dict, List, Optional
from enum import Enum

class SubscriptionTier(Enum):
    FREE = "free"
    STARTER = "starter"
    PROFESSIONAL = "professional"
    BUSINESS = "business"
    ENTERPRISE = "enterprise"

@dataclass
class TierLimits:
    """Limits pro Tier"""
    projects_per_month: int
    calculations_per_month: int
    reports_per_month: int
    users: int
    storage_gb: int
    ai_requests_per_month: int
    api_calls_per_day: int

@dataclass
class SubscriptionPlan:
    """Abo-Plan mit Tiers"""
    tier: SubscriptionTier
    name: str
    monthly_price: Decimal
    yearly_price: Decimal
    limits: TierLimits
    features: List[str]
    modules: List[str]
    support_level: str
    sla: Optional[str] = None

SUBSCRIPTION_PLANS: Dict[SubscriptionTier, SubscriptionPlan] = {
    SubscriptionTier.FREE: SubscriptionPlan(
        tier=SubscriptionTier.FREE,
        name="Free",
        monthly_price=Decimal("0"),
        yearly_price=Decimal("0"),
        limits=TierLimits(
            projects_per_month=3,
            calculations_per_month=10,
            reports_per_month=3,
            users=1,
            storage_gb=1,
            ai_requests_per_month=50,
            api_calls_per_day=100
        ),
        features=[
            "Basis-Projektverwaltung",
            "Standard-Berechnungen",
            "PDF-Export",
        ],
        modules=["projects", "calculations_basic", "reports_basic"],
        support_level="community"
    ),
    SubscriptionTier.STARTER: SubscriptionPlan(
        tier=SubscriptionTier.STARTER,
        name="Starter",
        monthly_price=Decimal("29.00"),
        yearly_price=Decimal("290.00"),
        limits=TierLimits(
            projects_per_month=20,
            calculations_per_month=100,
            reports_per_month=20,
            users=2,
            storage_gb=10,
            ai_requests_per_month=500,
            api_calls_per_day=1000
        ),
        features=[
            "Alles aus Free",
            "Erweiterte Berechnungen",
            "Kundenverwaltung",
            "E-Mail-Templates",
        ],
        modules=[
            "projects", "calculations_standard", "reports_standard",
            "customers", "email_templates"
        ],
        support_level="email"
    ),
    SubscriptionTier.PROFESSIONAL: SubscriptionPlan(
        tier=SubscriptionTier.PROFESSIONAL,
        name="Professional",
        monthly_price=Decimal("79.00"),
        yearly_price=Decimal("790.00"),
        limits=TierLimits(
            projects_per_month=100,
            calculations_per_month=500,
            reports_per_month=100,
            users=5,
            storage_gb=50,
            ai_requests_per_month=2000,
            api_calls_per_day=10000
        ),
        features=[
            "Alles aus Starter",
            "iSFP-Modul",
            "DIN V 18599 komplett",
            "KI-Textassistent",
            "Zeiterfassung",
            "API-Zugang",
        ],
        modules=[
            "projects", "calculations_full", "reports_full",
            "customers", "isfp", "din18599", "ai_assistant",
            "time_tracking", "api_access"
        ],
        support_level="priority_email"
    ),
    SubscriptionTier.BUSINESS: SubscriptionPlan(
        tier=SubscriptionTier.BUSINESS,
        name="Business",
        monthly_price=Decimal("199.00"),
        yearly_price=Decimal("1990.00"),
        limits=TierLimits(
            projects_per_month=-1,  # Unbegrenzt
            calculations_per_month=-1,
            reports_per_month=-1,
            users=20,
            storage_gb=200,
            ai_requests_per_month=10000,
            api_calls_per_day=100000
        ),
        features=[
            "Alles aus Professional",
            "Energieaudit DIN 16247",
            "Team-Verwaltung",
            "Workflow-Automatisierung",
            "White-Label",
            "Custom Reports",
        ],
        modules=["*"],
        support_level="phone",
        sla="99.5%"
    ),
    SubscriptionTier.ENTERPRISE: SubscriptionPlan(
        tier=SubscriptionTier.ENTERPRISE,
        name="Enterprise",
        monthly_price=Decimal("-1"),  # Individuell
        yearly_price=Decimal("-1"),
        limits=TierLimits(
            projects_per_month=-1,
            calculations_per_month=-1,
            reports_per_month=-1,
            users=-1,
            storage_gb=-1,
            ai_requests_per_month=-1,
            api_calls_per_day=-1
        ),
        features=[
            "Alles aus Business",
            "Dedicated Instance",
            "On-Premise Option",
            "Custom Integrationen",
            "SSO/SAML",
            "Audit Logs",
            "Dedicated Account Manager",
        ],
        modules=["*"],
        support_level="dedicated",
        sla="99.9%"
    ),
}


class SubscriptionManager:
    """Verwaltet Abonnements"""

    def __init__(self, session, payment_provider, usage_tracker):
        self.session = session
        self.payment = payment_provider
        self.usage = usage_tracker

    async def check_limit(
        self,
        organization_id: str,
        resource: str,
        requested: int = 1
    ) -> Dict:
        """Prüft ob Limit erreicht ist"""

        subscription = await self._get_subscription(organization_id)
        plan = SUBSCRIPTION_PLANS.get(subscription.tier)

        if not plan:
            return {"allowed": False, "reason": "Kein gültiger Plan"}

        limit = getattr(plan.limits, resource, 0)

        # Unbegrenzt
        if limit == -1:
            return {"allowed": True, "remaining": -1}

        # Aktuelle Nutzung
        current_usage = await self.usage.get_current_period_usage(
            organization_id, resource
        )

        remaining = limit - current_usage
        allowed = remaining >= requested

        return {
            "allowed": allowed,
            "limit": limit,
            "used": current_usage,
            "remaining": remaining,
            "upgrade_suggestion": self._get_upgrade_suggestion(plan.tier) if not allowed else None
        }

    async def upgrade(
        self,
        organization_id: str,
        new_tier: SubscriptionTier,
        payment_method_id: str
    ) -> Dict:
        """Upgrade auf höheren Tier"""

        current_sub = await self._get_subscription(organization_id)
        current_plan = SUBSCRIPTION_PLANS[current_sub.tier]
        new_plan = SUBSCRIPTION_PLANS[new_tier]

        # Prüfe ob Upgrade
        tier_order = list(SubscriptionTier)
        if tier_order.index(new_tier) <= tier_order.index(current_sub.tier):
            raise ValueError("Nur Upgrades möglich, für Downgrade Support kontaktieren")

        # Prorata-Berechnung
        prorate_amount = self._calculate_prorate(current_sub, current_plan, new_plan)

        # Zahlungsupdate
        await self.payment.update_subscription(
            subscription_id=current_sub.external_id,
            new_price=int(new_plan.monthly_price * 100),
            prorate_amount=int(prorate_amount * 100)
        )

        # DB Update
        current_sub.tier = new_tier
        current_sub.upgraded_at = datetime.utcnow()
        await self.session.commit()

        # Module freischalten
        await self._sync_modules(organization_id, new_plan.modules)

        return {
            "success": True,
            "new_tier": new_tier.value,
            "prorate_amount": str(prorate_amount),
            "new_limits": plan.limits.__dict__
        }

    def _calculate_prorate(
        self,
        subscription,
        old_plan: SubscriptionPlan,
        new_plan: SubscriptionPlan
    ) -> Decimal:
        """Berechnet anteiligen Betrag für Upgrade"""
        from datetime import date

        days_in_period = 30  # Vereinfacht
        days_remaining = (subscription.current_period_end - date.today()).days

        old_daily = old_plan.monthly_price / days_in_period
        new_daily = new_plan.monthly_price / days_in_period

        difference_daily = new_daily - old_daily
        prorate = difference_daily * days_remaining

        return max(Decimal("0"), prorate)

    def _get_upgrade_suggestion(self, current_tier: SubscriptionTier) -> Optional[str]:
        """Gibt Upgrade-Vorschlag"""
        tier_order = list(SubscriptionTier)
        current_idx = tier_order.index(current_tier)

        if current_idx < len(tier_order) - 1:
            next_tier = tier_order[current_idx + 1]
            next_plan = SUBSCRIPTION_PLANS[next_tier]
            return f"Upgrade auf {next_plan.name} für mehr Kapazität"

        return None
```

---

## 3. Modul-Freischaltung

### 3.1 Modul-Katalog

```python
# app/billing/modules.py
"""Modul-Verwaltung und Freischaltung"""

from dataclasses import dataclass
from decimal import Decimal
from typing import List, Optional, Dict
from enum import Enum

class ModuleCategory(Enum):
    CORE = "core"
    CALCULATION = "calculation"
    REPORT = "report"
    AUTOMATION = "automation"
    INTEGRATION = "integration"
    AI = "ai"
    ADMIN = "admin"

@dataclass
class Module:
    """Einzelnes Modul"""
    id: str
    name: str
    description: str
    category: ModuleCategory
    standalone_price: Optional[Decimal]  # None = nicht einzeln kaufbar
    dependencies: List[str]
    features: List[str]
    is_addon: bool = False  # Zusatzmodul zu Abo

MODULE_CATALOG: Dict[str, Module] = {
    # === CORE Module ===
    "projects": Module(
        id="projects",
        name="Projektverwaltung",
        description="Grundlegende Projektverwaltung",
        category=ModuleCategory.CORE,
        standalone_price=None,  # Immer enthalten
        dependencies=[],
        features=["Projekte erstellen", "Status-Tracking", "Dateiverwaltung"]
    ),
    "customers": Module(
        id="customers",
        name="Kundenverwaltung",
        description="CRM-Funktionen für Kunden",
        category=ModuleCategory.CORE,
        standalone_price=Decimal("9.90"),
        dependencies=["projects"],
        features=["Kundenakte", "Kontakthistorie", "Dokumente"]
    ),

    # === BERECHNUNGS-Module ===
    "calculations_basic": Module(
        id="calculations_basic",
        name="Basis-Berechnungen",
        description="U-Wert, einfache Heizlast",
        category=ModuleCategory.CALCULATION,
        standalone_price=None,
        dependencies=["projects"],
        features=["U-Wert", "Vereinfachte Heizlast", "Energiekennwerte"]
    ),
    "calculations_standard": Module(
        id="calculations_standard",
        name="Standard-Berechnungen",
        description="DIN EN 12831, Energiebilanz",
        category=ModuleCategory.CALCULATION,
        standalone_price=Decimal("19.90"),
        dependencies=["calculations_basic"],
        features=["Heizlast DIN EN 12831", "Energiebilanz", "Wirtschaftlichkeit"]
    ),
    "calculations_full": Module(
        id="calculations_full",
        name="Vollständige Berechnungen",
        description="Alle Berechnungsmodule",
        category=ModuleCategory.CALCULATION,
        standalone_price=Decimal("39.90"),
        dependencies=["calculations_standard"],
        features=["Sommerlicher Wärmeschutz", "Kühllast VDI 2078", "Tageslicht"]
    ),
    "din18599": Module(
        id="din18599",
        name="DIN V 18599",
        description="Vollständige Bilanzierung nach DIN V 18599",
        category=ModuleCategory.CALCULATION,
        standalone_price=Decimal("49.90"),
        dependencies=["calculations_full"],
        features=["Zonenbilanz", "Alle Nutzungsprofile", "Nichtwohngebäude"]
    ),
    "energy_audit": Module(
        id="energy_audit",
        name="Energieaudit DIN 16247",
        description="Energieaudit für Unternehmen",
        category=ModuleCategory.CALCULATION,
        standalone_price=Decimal("79.90"),
        dependencies=["din18599"],
        features=["Audit-Workflow", "Maßnahmenplanung", "Benchmark"]
    ),

    # === REPORT-Module ===
    "reports_basic": Module(
        id="reports_basic",
        name="Basis-Berichte",
        description="Standard-PDF-Berichte",
        category=ModuleCategory.REPORT,
        standalone_price=None,
        dependencies=["projects"],
        features=["PDF-Export", "Basis-Vorlagen"]
    ),
    "reports_standard": Module(
        id="reports_standard",
        name="Standard-Berichte",
        description="Erweiterte Berichtsvorlagen",
        category=ModuleCategory.REPORT,
        standalone_price=Decimal("14.90"),
        dependencies=["reports_basic"],
        features=["Energieausweis", "Kundenberichte", "Anpassbare Vorlagen"]
    ),
    "reports_full": Module(
        id="reports_full",
        name="Vollständige Berichte",
        description="Alle Berichtstypen inkl. iSFP",
        category=ModuleCategory.REPORT,
        standalone_price=Decimal("29.90"),
        dependencies=["reports_standard"],
        features=["iSFP", "BAFA-konforme Berichte", "White-Label"]
    ),
    "isfp": Module(
        id="isfp",
        name="iSFP-Modul",
        description="Individueller Sanierungsfahrplan",
        category=ModuleCategory.REPORT,
        standalone_price=Decimal("39.90"),
        dependencies=["reports_full", "calculations_full"],
        features=["iSFP-Generator", "BAFA-Schnittstelle", "Maßnahmenpakete"]
    ),

    # === KI-Module ===
    "ai_assistant": Module(
        id="ai_assistant",
        name="KI-Assistent",
        description="KI-gestützte Textgenerierung",
        category=ModuleCategory.AI,
        standalone_price=Decimal("19.90"),
        dependencies=[],
        features=["Textvorschläge", "Berichtstexte", "E-Mail-Entwürfe"],
        is_addon=True
    ),
    "ai_analysis": Module(
        id="ai_analysis",
        name="KI-Analyse",
        description="Automatische Gebäudeanalyse",
        category=ModuleCategory.AI,
        standalone_price=Decimal("29.90"),
        dependencies=["ai_assistant"],
        features=["Foto-Analyse", "Schadenserkennung", "Empfehlungen"],
        is_addon=True
    ),

    # === AUTOMATISIERUNG ===
    "time_tracking": Module(
        id="time_tracking",
        name="Zeiterfassung",
        description="Automatische Zeiterfassung",
        category=ModuleCategory.AUTOMATION,
        standalone_price=Decimal("9.90"),
        dependencies=["projects"],
        features=["Auto-Tracking", "Auswertungen", "Export"]
    ),
    "workflows": Module(
        id="workflows",
        name="Workflow-Automatisierung",
        description="Automatisierte Abläufe",
        category=ModuleCategory.AUTOMATION,
        standalone_price=Decimal("29.90"),
        dependencies=["projects"],
        features=["Trigger", "Aktionen", "E-Mail-Automation"]
    ),
    "invoicing": Module(
        id="invoicing",
        name="Rechnungsstellung",
        description="Angebote und Rechnungen",
        category=ModuleCategory.AUTOMATION,
        standalone_price=Decimal("19.90"),
        dependencies=["customers", "time_tracking"],
        features=["Angebote", "Rechnungen", "Mahnwesen", "DATEV-Export"]
    ),

    # === INTEGRATION ===
    "api_access": Module(
        id="api_access",
        name="API-Zugang",
        description="REST API für Integrationen",
        category=ModuleCategory.INTEGRATION,
        standalone_price=Decimal("29.90"),
        dependencies=[],
        features=["REST API", "Webhooks", "API-Keys"],
        is_addon=True
    ),
    "calendar_sync": Module(
        id="calendar_sync",
        name="Kalender-Sync",
        description="Kalender-Integration",
        category=ModuleCategory.INTEGRATION,
        standalone_price=Decimal("4.90"),
        dependencies=[],
        features=["Google Calendar", "Outlook", "CalDAV"],
        is_addon=True
    ),
    "email_integration": Module(
        id="email_integration",
        name="E-Mail-Integration",
        description="E-Mail senden und empfangen",
        category=ModuleCategory.INTEGRATION,
        standalone_price=Decimal("9.90"),
        dependencies=["customers"],
        features=["SMTP/IMAP", "Templates", "Tracking"]
    ),
}


class ModuleManager:
    """Verwaltet Modul-Freischaltungen"""

    def __init__(self, session):
        self.session = session
        self.catalog = MODULE_CATALOG

    async def get_available_modules(
        self,
        organization_id: str
    ) -> Dict[str, Dict]:
        """Gibt verfügbare Module mit Status zurück"""

        # Aktive Module laden
        active_modules = await self._get_active_modules(organization_id)

        result = {}
        for module_id, module in self.catalog.items():
            result[module_id] = {
                "id": module.id,
                "name": module.name,
                "description": module.description,
                "category": module.category.value,
                "price": str(module.standalone_price) if module.standalone_price else "included",
                "is_active": module_id in active_modules,
                "can_activate": self._can_activate(module, active_modules),
                "dependencies": module.dependencies,
                "features": module.features
            }

        return result

    async def activate_module(
        self,
        organization_id: str,
        module_id: str,
        payment_method_id: Optional[str] = None
    ) -> Dict:
        """Aktiviert Modul für Organisation"""

        module = self.catalog.get(module_id)
        if not module:
            raise ValueError(f"Modul {module_id} nicht gefunden")

        active_modules = await self._get_active_modules(organization_id)

        # Prüfe Abhängigkeiten
        for dep in module.dependencies:
            if dep not in active_modules:
                raise ValueError(f"Abhängigkeit {dep} muss zuerst aktiviert werden")

        # Prüfe ob kostenpflichtig
        if module.standalone_price and module.standalone_price > 0:
            if not payment_method_id:
                return {
                    "requires_payment": True,
                    "price": str(module.standalone_price),
                    "module": module_id
                }

            # Zahlung durchführen
            # ... Payment Logic ...

        # Modul aktivieren
        await self.session.execute("""
            INSERT INTO organization_modules (organization_id, module_id, is_active, activated_at)
            VALUES (:org_id, :module_id, true, NOW())
            ON CONFLICT (organization_id, module_id)
            DO UPDATE SET is_active = true, activated_at = NOW()
        """, {"org_id": organization_id, "module_id": module_id})

        await self.session.commit()

        return {
            "success": True,
            "module": module_id,
            "features_unlocked": module.features
        }

    def _can_activate(self, module: Module, active_modules: set) -> bool:
        """Prüft ob Modul aktivierbar ist"""
        return all(dep in active_modules for dep in module.dependencies)

    async def _get_active_modules(self, organization_id: str) -> set:
        """Lädt aktive Module einer Organisation"""
        result = await self.session.execute("""
            SELECT module_id FROM organization_modules
            WHERE organization_id = :org_id AND is_active = true
        """, {"org_id": organization_id})

        return {row.module_id for row in result}
```

---

## 4. Rechnungsstellung

### 4.1 Automatische Rechnungserstellung

```python
# app/billing/invoicing.py
"""Rechnungserstellung und -verwaltung"""

from dataclasses import dataclass, field
from decimal import Decimal
from typing import List, Dict, Optional
from datetime import date, datetime
from enum import Enum
import uuid

class InvoiceStatus(Enum):
    DRAFT = "draft"
    PENDING_APPROVAL = "pending_approval"
    APPROVED = "approved"
    SENT = "sent"
    PAID = "paid"
    OVERDUE = "overdue"
    CANCELLED = "cancelled"

@dataclass
class InvoiceLineItem:
    description: str
    quantity: Decimal
    unit_price: Decimal
    tax_rate: Decimal = Decimal("19")

    @property
    def net_amount(self) -> Decimal:
        return self.quantity * self.unit_price

    @property
    def tax_amount(self) -> Decimal:
        return self.net_amount * (self.tax_rate / 100)

    @property
    def gross_amount(self) -> Decimal:
        return self.net_amount + self.tax_amount

@dataclass
class Invoice:
    id: str
    organization_id: str
    customer_id: str
    invoice_number: str
    invoice_date: date
    due_date: date
    status: InvoiceStatus
    line_items: List[InvoiceLineItem]
    notes: str = ""
    payment_terms: str = "Zahlbar innerhalb von 14 Tagen"

    @property
    def subtotal(self) -> Decimal:
        return sum(item.net_amount for item in self.line_items)

    @property
    def tax_total(self) -> Decimal:
        return sum(item.tax_amount for item in self.line_items)

    @property
    def total(self) -> Decimal:
        return self.subtotal + self.tax_total


class InvoiceGenerator:
    """Generiert Rechnungen automatisch"""

    def __init__(self, session, template_engine, pdf_generator):
        self.session = session
        self.templates = template_engine
        self.pdf = pdf_generator

    async def generate_subscription_invoice(
        self,
        organization_id: str,
        period_start: date,
        period_end: date
    ) -> Invoice:
        """Generiert Abo-Rechnung"""

        org = await self._get_organization(organization_id)
        subscription = await self._get_subscription(organization_id)
        plan = SUBSCRIPTION_PLANS[subscription.tier]

        line_items = [
            InvoiceLineItem(
                description=f"{plan.name} Abonnement ({period_start.strftime('%m/%Y')})",
                quantity=Decimal("1"),
                unit_price=plan.monthly_price
            )
        ]

        # Zusätzliche Benutzer
        extra_users = await self._count_extra_users(organization_id, plan.limits.users)
        if extra_users > 0:
            line_items.append(InvoiceLineItem(
                description=f"Zusätzliche Benutzer ({extra_users})",
                quantity=Decimal(str(extra_users)),
                unit_price=Decimal("9.90")
            ))

        # Zusätzlicher Speicher
        extra_storage = await self._get_extra_storage(organization_id, plan.limits.storage_gb)
        if extra_storage > 0:
            line_items.append(InvoiceLineItem(
                description=f"Zusätzlicher Speicher ({extra_storage} GB)",
                quantity=Decimal(str(extra_storage)),
                unit_price=Decimal("0.50")
            ))

        invoice = Invoice(
            id=str(uuid.uuid4()),
            organization_id=organization_id,
            customer_id=org.billing_customer_id,
            invoice_number=await self._generate_invoice_number(organization_id),
            invoice_date=date.today(),
            due_date=date.today() + timedelta(days=14),
            status=InvoiceStatus.PENDING_APPROVAL,
            line_items=line_items
        )

        await self._save_invoice(invoice)

        return invoice

    async def generate_usage_invoice(
        self,
        organization_id: str,
        period_start: date,
        period_end: date
    ) -> Invoice:
        """Generiert nutzungsbasierte Rechnung"""

        org = await self._get_organization(organization_id)
        usage_summary = await self.usage_tracker.get_usage_summary(
            organization_id, period_start, period_end
        )

        calculator = UsageBillingCalculator()
        billing_data = calculator.calculate_invoice(usage_summary)

        line_items = [
            InvoiceLineItem(
                description=item["description"],
                quantity=Decimal(str(item["quantity"])),
                unit_price=Decimal(item["unit_price"])
            )
            for item in billing_data["line_items"]
        ]

        if not line_items:
            return None  # Keine Nutzung, keine Rechnung

        invoice = Invoice(
            id=str(uuid.uuid4()),
            organization_id=organization_id,
            customer_id=org.billing_customer_id,
            invoice_number=await self._generate_invoice_number(organization_id),
            invoice_date=date.today(),
            due_date=date.today() + timedelta(days=14),
            status=InvoiceStatus.PENDING_APPROVAL,
            line_items=line_items,
            notes=f"Nutzungszeitraum: {period_start.strftime('%d.%m.%Y')} - {period_end.strftime('%d.%m.%Y')}"
        )

        await self._save_invoice(invoice)

        return invoice

    async def approve_and_send(
        self,
        invoice_id: str,
        approved_by: str
    ) -> Dict:
        """Genehmigt und versendet Rechnung (MANUELL!)"""

        invoice = await self._get_invoice(invoice_id)

        if invoice.status != InvoiceStatus.PENDING_APPROVAL:
            raise ValueError(f"Rechnung kann nicht gesendet werden (Status: {invoice.status})")

        # PDF generieren
        pdf_content = await self._generate_pdf(invoice)

        # E-Mail vorbereiten
        customer = await self._get_customer(invoice.customer_id)

        email_data = {
            "to": customer.billing_email,
            "subject": f"Rechnung {invoice.invoice_number}",
            "template": "invoice_email",
            "attachments": [
                {
                    "filename": f"Rechnung_{invoice.invoice_number}.pdf",
                    "content": pdf_content
                }
            ],
            "context": {
                "customer_name": customer.name,
                "invoice_number": invoice.invoice_number,
                "amount": str(invoice.total),
                "due_date": invoice.due_date.strftime("%d.%m.%Y")
            }
        }

        # Status aktualisieren
        invoice.status = InvoiceStatus.APPROVED

        # Audit-Log
        await self._log_action(invoice_id, "approved", approved_by)

        await self.session.commit()

        return {
            "invoice_id": invoice_id,
            "status": "approved",
            "email_preview": email_data,
            "message": "Rechnung genehmigt. Klicken Sie auf 'Senden' um die E-Mail zu versenden."
        }

    async def send_invoice(
        self,
        invoice_id: str,
        sent_by: str
    ) -> Dict:
        """Versendet genehmigte Rechnung"""

        invoice = await self._get_invoice(invoice_id)

        if invoice.status != InvoiceStatus.APPROVED:
            raise ValueError("Rechnung muss zuerst genehmigt werden")

        # E-Mail senden
        await self.email_service.send_invoice_email(invoice)

        # Status aktualisieren
        invoice.status = InvoiceStatus.SENT
        invoice.sent_at = datetime.utcnow()

        # Audit-Log
        await self._log_action(invoice_id, "sent", sent_by)

        await self.session.commit()

        return {
            "invoice_id": invoice_id,
            "status": "sent",
            "sent_to": invoice.customer.billing_email
        }

    async def _generate_invoice_number(self, organization_id: str) -> str:
        """Generiert eindeutige Rechnungsnummer"""
        year = date.today().year

        result = await self.session.execute("""
            SELECT COUNT(*) + 1 as next_num
            FROM invoices
            WHERE organization_id = :org_id
              AND EXTRACT(YEAR FROM invoice_date) = :year
        """, {"org_id": organization_id, "year": year})

        next_num = result.scalar()

        return f"RE-{year}-{next_num:05d}"
```

### 4.2 Zahlungs-Integration

```python
# app/billing/payment.py
"""Zahlungsabwicklung"""

from abc import ABC, abstractmethod
from decimal import Decimal
from typing import Dict, Optional
import stripe
from datetime import datetime

class PaymentProvider(ABC):
    """Abstrakte Basis für Zahlungsanbieter"""

    @abstractmethod
    async def create_customer(self, organization_data: Dict) -> str:
        """Erstellt Kunden beim Zahlungsanbieter"""
        pass

    @abstractmethod
    async def create_payment_method(self, customer_id: str, payment_data: Dict) -> str:
        """Speichert Zahlungsmethode"""
        pass

    @abstractmethod
    async def charge(self, customer_id: str, amount: int, description: str) -> Dict:
        """Führt Zahlung durch"""
        pass

    @abstractmethod
    async def create_subscription(self, customer_id: str, plan_id: str) -> Dict:
        """Erstellt Abo"""
        pass


class StripePaymentProvider(PaymentProvider):
    """Stripe-Implementation"""

    def __init__(self, api_key: str, webhook_secret: str):
        stripe.api_key = api_key
        self.webhook_secret = webhook_secret

    async def create_customer(self, organization_data: Dict) -> str:
        customer = stripe.Customer.create(
            email=organization_data["email"],
            name=organization_data["name"],
            metadata={
                "organization_id": organization_data["id"]
            }
        )
        return customer.id

    async def create_payment_method(self, customer_id: str, payment_data: Dict) -> str:
        # Payment Method aus Frontend (Stripe Elements)
        pm_id = payment_data.get("payment_method_id")

        # An Kunden anhängen
        stripe.PaymentMethod.attach(pm_id, customer=customer_id)

        # Als Standard setzen
        stripe.Customer.modify(
            customer_id,
            invoice_settings={"default_payment_method": pm_id}
        )

        return pm_id

    async def charge(self, customer_id: str, amount: int, description: str) -> Dict:
        intent = stripe.PaymentIntent.create(
            amount=amount,  # In Cent
            currency="eur",
            customer=customer_id,
            description=description,
            automatic_payment_methods={"enabled": True}
        )

        return {
            "payment_id": intent.id,
            "status": intent.status,
            "client_secret": intent.client_secret
        }

    async def create_subscription(
        self,
        customer_id: str,
        price_id: str,
        trial_days: int = 0
    ) -> Dict:
        subscription = stripe.Subscription.create(
            customer=customer_id,
            items=[{"price": price_id}],
            trial_period_days=trial_days if trial_days > 0 else None,
            payment_behavior="default_incomplete",
            expand=["latest_invoice.payment_intent"]
        )

        return {
            "subscription_id": subscription.id,
            "status": subscription.status,
            "current_period_end": datetime.fromtimestamp(subscription.current_period_end),
            "client_secret": subscription.latest_invoice.payment_intent.client_secret
                if subscription.latest_invoice.payment_intent else None
        }

    async def cancel_subscription(self, subscription_id: str, immediately: bool = False) -> Dict:
        if immediately:
            subscription = stripe.Subscription.delete(subscription_id)
        else:
            subscription = stripe.Subscription.modify(
                subscription_id,
                cancel_at_period_end=True
            )

        return {
            "subscription_id": subscription.id,
            "status": subscription.status,
            "cancel_at": datetime.fromtimestamp(subscription.cancel_at) if subscription.cancel_at else None
        }

    async def handle_webhook(self, payload: bytes, signature: str) -> Dict:
        """Verarbeitet Stripe Webhooks"""
        try:
            event = stripe.Webhook.construct_event(
                payload, signature, self.webhook_secret
            )
        except stripe.error.SignatureVerificationError:
            raise ValueError("Invalid webhook signature")

        handlers = {
            "invoice.paid": self._handle_invoice_paid,
            "invoice.payment_failed": self._handle_payment_failed,
            "customer.subscription.deleted": self._handle_subscription_cancelled,
            "customer.subscription.updated": self._handle_subscription_updated,
        }

        handler = handlers.get(event.type)
        if handler:
            return await handler(event.data.object)

        return {"status": "ignored", "event_type": event.type}

    async def _handle_invoice_paid(self, invoice) -> Dict:
        """Verarbeitet bezahlte Rechnung"""
        return {
            "event": "invoice_paid",
            "customer_id": invoice.customer,
            "amount": invoice.amount_paid,
            "invoice_id": invoice.id
        }

    async def _handle_payment_failed(self, invoice) -> Dict:
        """Verarbeitet fehlgeschlagene Zahlung"""
        return {
            "event": "payment_failed",
            "customer_id": invoice.customer,
            "invoice_id": invoice.id,
            "attempt_count": invoice.attempt_count
        }
```

---

## 5. Admin-Konfiguration

### 5.1 Organisations-Einstellungen

```python
# app/billing/admin.py
"""Admin-Funktionen für Billing"""

from typing import Dict, List, Optional
from dataclasses import dataclass
from datetime import date

@dataclass
class OrganizationBillingConfig:
    """Billing-Konfiguration einer Organisation"""
    billing_model: str  # "flatrate", "usage", "subscription"
    billing_interval: str  # "monthly", "quarterly", "yearly"
    payment_method: str  # "invoice", "card", "sepa"
    tax_id: Optional[str] = None
    billing_email: Optional[str] = None
    billing_address: Optional[Dict] = None
    custom_pricing: Optional[Dict] = None  # Für Enterprise
    credit_limit: Optional[Decimal] = None


class BillingAdminService:
    """Admin-Service für Billing-Verwaltung"""

    def __init__(self, session, payment_provider):
        self.session = session
        self.payment = payment_provider

    async def configure_organization_billing(
        self,
        organization_id: str,
        config: OrganizationBillingConfig,
        admin_user_id: str
    ) -> Dict:
        """Konfiguriert Billing für Organisation"""

        # Validierung
        if config.billing_model not in ["flatrate", "usage", "subscription"]:
            raise ValueError("Ungültiges Abrechnungsmodell")

        # Bestehende Config laden oder erstellen
        existing = await self._get_billing_config(organization_id)

        if existing:
            # Update
            await self.session.execute("""
                UPDATE organization_billing_config
                SET billing_model = :model,
                    billing_interval = :interval,
                    payment_method = :payment,
                    tax_id = :tax_id,
                    billing_email = :email,
                    billing_address = :address,
                    custom_pricing = :custom,
                    updated_at = NOW(),
                    updated_by = :admin
                WHERE organization_id = :org_id
            """, {
                "org_id": organization_id,
                "model": config.billing_model,
                "interval": config.billing_interval,
                "payment": config.payment_method,
                "tax_id": config.tax_id,
                "email": config.billing_email,
                "address": config.billing_address,
                "custom": config.custom_pricing,
                "admin": admin_user_id
            })
        else:
            # Insert
            await self.session.execute("""
                INSERT INTO organization_billing_config
                (organization_id, billing_model, billing_interval, payment_method,
                 tax_id, billing_email, billing_address, custom_pricing, created_by)
                VALUES (:org_id, :model, :interval, :payment, :tax_id, :email,
                        :address, :custom, :admin)
            """, {
                "org_id": organization_id,
                "model": config.billing_model,
                "interval": config.billing_interval,
                "payment": config.payment_method,
                "tax_id": config.tax_id,
                "email": config.billing_email,
                "address": config.billing_address,
                "custom": config.custom_pricing,
                "admin": admin_user_id
            })

        await self.session.commit()

        # Audit Log
        await self._log_config_change(organization_id, config, admin_user_id)

        return {"success": True, "organization_id": organization_id}

    async def grant_module_access(
        self,
        organization_id: str,
        module_id: str,
        granted_by: str,
        expires_at: Optional[date] = None,
        reason: str = ""
    ) -> Dict:
        """Gewährt Modul-Zugang (Admin-Override)"""

        await self.session.execute("""
            INSERT INTO organization_modules
            (organization_id, module_id, is_active, granted_by, expires_at, grant_reason)
            VALUES (:org_id, :module, true, :granted_by, :expires, :reason)
            ON CONFLICT (organization_id, module_id)
            DO UPDATE SET is_active = true, granted_by = :granted_by,
                         expires_at = :expires, grant_reason = :reason
        """, {
            "org_id": organization_id,
            "module": module_id,
            "granted_by": granted_by,
            "expires": expires_at,
            "reason": reason
        })

        await self.session.commit()

        return {
            "success": True,
            "module": module_id,
            "expires_at": expires_at.isoformat() if expires_at else None
        }

    async def get_billing_dashboard(self) -> Dict:
        """Dashboard für Billing-Admin"""

        stats = await self.session.execute("""
            SELECT
                COUNT(DISTINCT o.id) as total_organizations,
                COUNT(DISTINCT CASE WHEN s.tier != 'free' THEN o.id END) as paying_customers,
                SUM(CASE
                    WHEN s.tier = 'starter' THEN 29
                    WHEN s.tier = 'professional' THEN 79
                    WHEN s.tier = 'business' THEN 199
                    ELSE 0
                END) as monthly_recurring_revenue,
                COUNT(DISTINCT CASE WHEN s.tier = 'free' THEN o.id END) as free_tier,
                COUNT(DISTINCT CASE WHEN s.tier = 'starter' THEN o.id END) as starter_tier,
                COUNT(DISTINCT CASE WHEN s.tier = 'professional' THEN o.id END) as pro_tier,
                COUNT(DISTINCT CASE WHEN s.tier = 'business' THEN o.id END) as business_tier
            FROM organizations o
            LEFT JOIN subscriptions s ON o.id = s.organization_id AND s.status = 'active'
        """)

        row = stats.fetchone()

        # Offene Rechnungen
        open_invoices = await self.session.execute("""
            SELECT COUNT(*) as count, SUM(total) as amount
            FROM invoices
            WHERE status IN ('sent', 'overdue')
        """)
        inv_row = open_invoices.fetchone()

        return {
            "organizations": {
                "total": row.total_organizations,
                "paying": row.paying_customers,
                "free": row.free_tier
            },
            "mrr": float(row.monthly_recurring_revenue or 0),
            "tier_distribution": {
                "free": row.free_tier,
                "starter": row.starter_tier,
                "professional": row.pro_tier,
                "business": row.business_tier
            },
            "open_invoices": {
                "count": inv_row.count,
                "amount": float(inv_row.amount or 0)
            }
        }
```

---

## 6. Frontend-Integration

### 6.1 Preisübersicht-Komponente

```html
<!-- templates/components/pricing_table.html -->
<div class="pricing-container" x-data="pricingTable()">
    <!-- Billing Interval Toggle -->
    <div class="billing-toggle">
        <button @click="interval = 'monthly'"
                :class="{ 'active': interval === 'monthly' }">
            Monatlich
        </button>
        <button @click="interval = 'yearly'"
                :class="{ 'active': interval === 'yearly' }">
            Jährlich
            <span class="discount-badge">-15%</span>
        </button>
    </div>

    <!-- Pricing Cards -->
    <div class="pricing-grid">
        <template x-for="plan in plans" :key="plan.id">
            <div class="pricing-card" :class="{ 'recommended': plan.recommended }">
                <div class="plan-header">
                    <h3 x-text="plan.name"></h3>
                    <div class="plan-price">
                        <span class="currency">€</span>
                        <span class="amount" x-text="getPrice(plan)"></span>
                        <span class="interval">/Monat</span>
                    </div>
                    <p class="plan-description" x-text="plan.description"></p>
                </div>

                <ul class="plan-features">
                    <template x-for="feature in plan.features">
                        <li>
                            <svg class="check-icon"><!-- ... --></svg>
                            <span x-text="feature"></span>
                        </li>
                    </template>
                </ul>

                <div class="plan-limits">
                    <div class="limit-item">
                        <span class="limit-label">Projekte</span>
                        <span class="limit-value" x-text="formatLimit(plan.limits.projects)"></span>
                    </div>
                    <div class="limit-item">
                        <span class="limit-label">Benutzer</span>
                        <span class="limit-value" x-text="formatLimit(plan.limits.users)"></span>
                    </div>
                    <div class="limit-item">
                        <span class="limit-label">Speicher</span>
                        <span class="limit-value" x-text="plan.limits.storage + ' GB'"></span>
                    </div>
                </div>

                <button class="plan-cta"
                        @click="selectPlan(plan)"
                        :disabled="plan.id === currentPlan">
                    <span x-text="plan.id === currentPlan ? 'Aktueller Plan' : 'Auswählen'"></span>
                </button>
            </div>
        </template>
    </div>

    <!-- Module Add-ons -->
    <div class="addons-section" x-show="showAddons">
        <h3>Zusatzmodule</h3>
        <div class="addons-grid">
            <template x-for="addon in addons" :key="addon.id">
                <div class="addon-card">
                    <div class="addon-info">
                        <h4 x-text="addon.name"></h4>
                        <p x-text="addon.description"></p>
                    </div>
                    <div class="addon-price">
                        <span x-text="'€' + addon.price + '/Monat'"></span>
                        <button @click="toggleAddon(addon)"
                                :class="{ 'active': isAddonActive(addon.id) }">
                            <span x-text="isAddonActive(addon.id) ? 'Aktiv' : 'Hinzufügen'"></span>
                        </button>
                    </div>
                </div>
            </template>
        </div>
    </div>
</div>

<script>
function pricingTable() {
    return {
        interval: 'monthly',
        currentPlan: null,
        plans: [],
        addons: [],
        activeAddons: new Set(),

        async init() {
            // Pläne laden
            const response = await fetch('/api/v1/billing/plans');
            const data = await response.json();
            this.plans = data.plans;
            this.addons = data.addons;
            this.currentPlan = data.current_plan;
            this.activeAddons = new Set(data.active_addons);
        },

        getPrice(plan) {
            if (this.interval === 'yearly') {
                return (plan.yearly_price / 12).toFixed(0);
            }
            return plan.monthly_price;
        },

        formatLimit(limit) {
            return limit === -1 ? 'Unbegrenzt' : limit;
        },

        async selectPlan(plan) {
            // Checkout starten
            window.location.href = `/billing/checkout?plan=${plan.id}&interval=${this.interval}`;
        },

        isAddonActive(addonId) {
            return this.activeAddons.has(addonId);
        },

        async toggleAddon(addon) {
            if (this.isAddonActive(addon.id)) {
                // Deaktivieren
                await fetch(`/api/v1/billing/addons/${addon.id}`, { method: 'DELETE' });
                this.activeAddons.delete(addon.id);
            } else {
                // Aktivieren (mit Checkout)
                window.location.href = `/billing/addon-checkout?addon=${addon.id}`;
            }
        }
    };
}
</script>
```

---

## 7. Datenbank-Schema

```sql
-- Billing-bezogene Tabellen

-- Subscriptions
CREATE TABLE subscriptions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID NOT NULL REFERENCES organizations(id),
    tier VARCHAR(50) NOT NULL,
    status VARCHAR(50) NOT NULL DEFAULT 'active',
    billing_interval VARCHAR(20) NOT NULL,
    current_period_start DATE NOT NULL,
    current_period_end DATE NOT NULL,
    cancel_at_period_end BOOLEAN DEFAULT FALSE,
    external_id VARCHAR(255),  -- Stripe Subscription ID
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),

    UNIQUE(organization_id)
);

-- Organization Modules
CREATE TABLE organization_modules (
    organization_id UUID NOT NULL REFERENCES organizations(id),
    module_id VARCHAR(100) NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    activated_at TIMESTAMPTZ DEFAULT NOW(),
    expires_at DATE,
    granted_by UUID REFERENCES users(id),
    grant_reason TEXT,

    PRIMARY KEY (organization_id, module_id)
);

-- Usage Records
CREATE TABLE usage_records (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID NOT NULL REFERENCES organizations(id),
    unit VARCHAR(50) NOT NULL,
    quantity INTEGER NOT NULL DEFAULT 1,
    timestamp TIMESTAMPTZ DEFAULT NOW(),
    metadata JSONB DEFAULT '{}',

    INDEX idx_usage_org_unit_time (organization_id, unit, timestamp)
);

-- Invoices
CREATE TABLE invoices (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID NOT NULL REFERENCES organizations(id),
    customer_id UUID,
    invoice_number VARCHAR(50) NOT NULL UNIQUE,
    invoice_date DATE NOT NULL,
    due_date DATE NOT NULL,
    status VARCHAR(50) NOT NULL DEFAULT 'draft',
    subtotal DECIMAL(10, 2) NOT NULL,
    tax_amount DECIMAL(10, 2) NOT NULL,
    total DECIMAL(10, 2) NOT NULL,
    notes TEXT,
    sent_at TIMESTAMPTZ,
    paid_at TIMESTAMPTZ,
    external_id VARCHAR(255),  -- Stripe Invoice ID
    created_at TIMESTAMPTZ DEFAULT NOW(),

    INDEX idx_invoices_org_status (organization_id, status)
);

-- Invoice Line Items
CREATE TABLE invoice_line_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    invoice_id UUID NOT NULL REFERENCES invoices(id) ON DELETE CASCADE,
    description TEXT NOT NULL,
    quantity DECIMAL(10, 4) NOT NULL,
    unit_price DECIMAL(10, 2) NOT NULL,
    tax_rate DECIMAL(5, 2) DEFAULT 19.00,
    net_amount DECIMAL(10, 2) NOT NULL,
    position INTEGER NOT NULL DEFAULT 0
);

-- Billing Config
CREATE TABLE organization_billing_config (
    organization_id UUID PRIMARY KEY REFERENCES organizations(id),
    billing_model VARCHAR(50) NOT NULL,
    billing_interval VARCHAR(20) NOT NULL,
    payment_method VARCHAR(50) NOT NULL,
    tax_id VARCHAR(50),
    billing_email VARCHAR(255),
    billing_address JSONB,
    custom_pricing JSONB,
    credit_limit DECIMAL(10, 2),
    stripe_customer_id VARCHAR(255),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id)
);

-- Payment Methods
CREATE TABLE payment_methods (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID NOT NULL REFERENCES organizations(id),
    type VARCHAR(50) NOT NULL,  -- card, sepa, invoice
    is_default BOOLEAN DEFAULT FALSE,
    external_id VARCHAR(255),  -- Stripe PaymentMethod ID
    last_four VARCHAR(4),
    brand VARCHAR(50),
    expires_at DATE,
    created_at TIMESTAMPTZ DEFAULT NOW(),

    INDEX idx_payment_methods_org (organization_id)
);

-- Billing Audit Log
CREATE TABLE billing_audit_log (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID NOT NULL REFERENCES organizations(id),
    action VARCHAR(100) NOT NULL,
    entity_type VARCHAR(50),
    entity_id UUID,
    old_value JSONB,
    new_value JSONB,
    performed_by UUID REFERENCES users(id),
    performed_at TIMESTAMPTZ DEFAULT NOW(),
    ip_address INET,

    INDEX idx_billing_audit_org_time (organization_id, performed_at)
);
```

---

## 8. Checkliste

```
✓ Flexible Monetarisierungsmodelle (Flatrate, Pay-per-Use, Subscription)
✓ Modul-basierte Freischaltung
✓ Automatische Rechnungserstellung (mit manueller Freigabe)
✓ Stripe-Integration für Zahlungen
✓ Tier-basierte Limits und Features
✓ Addon-Module einzeln zubuchbar
✓ Admin-Dashboard für Billing
✓ Organisations-spezifische Konfiguration
✓ Audit-Logging für alle Billing-Aktionen
✓ Frontend-Komponenten für Preisauswahl
✓ Datenbank-Schema definiert
```

---

*Letzte Aktualisierung: 2025-11-25*
*Version: 1.0.0*
