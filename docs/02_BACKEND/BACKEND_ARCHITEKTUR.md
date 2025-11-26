# Backend-Architektur

## 1. Technologie-Stack

### 1.1 Kernkomponenten
```
Python 3.11+
├── FastAPI 0.104+          # Web-Framework
├── SQLAlchemy 2.0+         # ORM
├── Pydantic 2.0+           # Validierung
├── Alembic                 # Migrationen
├── Celery 5.3+             # Task Queue
├── Redis                   # Cache & Broker
└── uvicorn                 # ASGI Server
```

### 1.2 Zusätzliche Bibliotheken
```
Berechnung:
├── NumPy                   # Numerische Berechnungen
├── Pandas                  # Datenverarbeitung
└── SciPy                   # Wissenschaftliche Funktionen

Berichte:
├── Jinja2                  # Templates
├── WeasyPrint              # PDF-Generierung
└── openpyxl                # Excel-Export

KI:
├── httpx                   # Async HTTP (Ollama)
├── chromadb                # Vektor-DB
└── sentence-transformers   # Embeddings
```

---

## 2. Projektstruktur

```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py                    # FastAPI Application Factory
│   ├── config.py                  # Einstellungen (Pydantic Settings)
│   ├── dependencies.py            # Dependency Injection
│   │
│   ├── api/                       # API Endpoints
│   │   ├── __init__.py
│   │   ├── router.py              # Hauptrouter
│   │   ├── middleware.py          # Custom Middleware
│   │   │
│   │   └── v1/                    # API Version 1
│   │       ├── __init__.py
│   │       ├── auth.py            # Authentifizierung
│   │       ├── users.py           # Benutzerverwaltung
│   │       ├── organizations.py   # Organisationen
│   │       ├── projects.py        # Projekte
│   │       ├── customers.py       # Kunden
│   │       ├── locations.py       # Standorte
│   │       ├── buildings.py       # Gebäude
│   │       ├── zones.py           # Zonen
│   │       ├── components.py      # Bauteile
│   │       ├── systems.py         # Anlagen
│   │       ├── calculations.py    # Berechnungen
│   │       ├── reports.py         # Berichte
│   │       ├── quotes.py          # Angebote
│   │       ├── invoices.py        # Rechnungen
│   │       ├── time_entries.py    # Zeiterfassung
│   │       ├── documents.py       # Dokumente
│   │       ├── materials.py       # Stammdaten Materialien
│   │       ├── climate.py         # Klimadaten
│   │       └── ai.py              # KI-Funktionen
│   │
│   ├── core/                      # Kernfunktionen
│   │   ├── __init__.py
│   │   ├── security.py            # JWT, Password Hashing
│   │   ├── permissions.py         # RBAC System
│   │   ├── exceptions.py          # Custom Exceptions
│   │   ├── events.py              # Event System
│   │   └── logging.py             # Logging Konfiguration
│   │
│   ├── models/                    # SQLAlchemy Models
│   │   ├── __init__.py
│   │   ├── base.py                # Base Model mit Mixins
│   │   ├── user.py                # User, Role, Session
│   │   ├── organization.py        # Organization, Settings
│   │   ├── project.py             # Project, Location
│   │   ├── customer.py            # Customer
│   │   ├── building.py            # Building, Zone
│   │   ├── component.py           # BuildingComponent
│   │   ├── system.py              # BuildingSystem
│   │   ├── calculation.py         # Calculation, Result
│   │   ├── report.py              # Report, Template
│   │   ├── business.py            # Quote, Invoice, TimeEntry
│   │   ├── document.py            # Document
│   │   └── reference.py           # Material, ClimateData
│   │
│   ├── schemas/                   # Pydantic Schemas
│   │   ├── __init__.py
│   │   ├── common.py              # Gemeinsame Schemas
│   │   ├── user.py
│   │   ├── organization.py
│   │   ├── project.py
│   │   ├── customer.py
│   │   ├── building.py
│   │   ├── component.py
│   │   ├── system.py
│   │   ├── calculation.py
│   │   ├── report.py
│   │   ├── business.py
│   │   └── ai.py
│   │
│   ├── services/                  # Business Logic Layer
│   │   ├── __init__.py
│   │   ├── base.py                # Base Service Klasse
│   │   ├── auth_service.py
│   │   ├── user_service.py
│   │   ├── organization_service.py
│   │   ├── project_service.py
│   │   ├── customer_service.py
│   │   ├── building_service.py
│   │   ├── calculation_service.py
│   │   ├── report_service.py
│   │   ├── quote_service.py
│   │   ├── invoice_service.py
│   │   ├── time_service.py
│   │   ├── document_service.py
│   │   └── notification_service.py
│   │
│   ├── calculations/              # Berechnungs-Engine
│   │   ├── __init__.py
│   │   ├── base.py                # Basis-Berechnungsklasse
│   │   ├── validators.py          # Eingabe-Validierung
│   │   │
│   │   ├── thermal/               # Wärmetechnik
│   │   │   ├── __init__.py
│   │   │   ├── u_value.py         # U-Wert-Berechnung
│   │   │   ├── thermal_bridge.py  # Wärmebrücken
│   │   │   ├── heat_load.py       # Heizlast DIN EN 12831
│   │   │   └── cooling_load.py    # Kühllast VDI 2078
│   │   │
│   │   ├── energy/                # Energiebilanz
│   │   │   ├── __init__.py
│   │   │   ├── din18599.py        # DIN V 18599 Hauptberechnung
│   │   │   ├── zones.py           # Zonenbilanzierung
│   │   │   ├── heating.py         # Heizsysteme
│   │   │   ├── cooling.py         # Kühlsysteme
│   │   │   ├── ventilation.py     # Lüftungssysteme
│   │   │   ├── dhw.py             # Trinkwarmwasser
│   │   │   ├── lighting.py        # Beleuchtung
│   │   │   └── primary_energy.py  # Primärenergie
│   │   │
│   │   ├── renewable/             # Erneuerbare Energien
│   │   │   ├── __init__.py
│   │   │   ├── photovoltaic.py    # PV-Berechnung
│   │   │   ├── solar_thermal.py   # Solarthermie
│   │   │   ├── heat_pump.py       # Wärmepumpen
│   │   │   └── biomass.py         # Biomasse
│   │   │
│   │   ├── emissions/             # CO2/THG
│   │   │   ├── __init__.py
│   │   │   ├── co2_balance.py     # CO2-Bilanz
│   │   │   └── ghg_protocol.py    # Scope 1/2/3
│   │   │
│   │   ├── economic/              # Wirtschaftlichkeit
│   │   │   ├── __init__.py
│   │   │   ├── investment.py      # Investitionsrechnung
│   │   │   ├── operation.py       # Betriebskosten
│   │   │   └── funding.py         # Förderung
│   │   │
│   │   └── data/                  # Stammdaten für Berechnungen
│   │       ├── __init__.py
│   │       ├── materials.json     # Baustoff-Kennwerte
│   │       ├── emission_factors.json
│   │       ├── primary_energy_factors.json
│   │       └── usage_profiles.json
│   │
│   ├── ai/                        # KI-Integration
│   │   ├── __init__.py
│   │   ├── client.py              # LLM Client (Ollama/OpenAI)
│   │   ├── embeddings.py          # Embedding Service
│   │   ├── vectorstore.py         # ChromaDB Integration
│   │   ├── prompts/               # Prompt Templates
│   │   │   ├── __init__.py
│   │   │   ├── report_text.py
│   │   │   ├── recommendations.py
│   │   │   └── analysis.py
│   │   └── agents/                # AI Agents
│   │       ├── __init__.py
│   │       ├── report_agent.py
│   │       └── analysis_agent.py
│   │
│   ├── reports/                   # Berichtsgenerierung
│   │   ├── __init__.py
│   │   ├── generator.py           # Report Generator
│   │   ├── pdf.py                 # PDF Rendering
│   │   ├── excel.py               # Excel Export
│   │   ├── xml.py                 # XML Export (BAFA etc.)
│   │   └── templates/             # Jinja2 Templates
│   │       ├── energy_certificate/
│   │       ├── energy_audit/
│   │       ├── renovation_roadmap/
│   │       └── heat_load_report/
│   │
│   ├── storage/                   # Dateispeicherung
│   │   ├── __init__.py
│   │   ├── minio_client.py        # MinIO S3 Client
│   │   └── file_service.py        # Datei-Operationen
│   │
│   ├── tasks/                     # Celery Tasks
│   │   ├── __init__.py
│   │   ├── celery_app.py          # Celery Konfiguration
│   │   ├── calculation_tasks.py   # Berechnung im Hintergrund
│   │   ├── report_tasks.py        # Berichtsgenerierung
│   │   ├── export_tasks.py        # Export Tasks
│   │   ├── email_tasks.py         # E-Mail Versand
│   │   ├── backup_tasks.py        # Backup Tasks
│   │   └── cleanup_tasks.py       # Aufräum-Tasks
│   │
│   └── utils/                     # Hilfsfunktionen
│       ├── __init__.py
│       ├── validators.py          # Allgemeine Validatoren
│       ├── converters.py          # Einheiten-Konvertierung
│       ├── formatters.py          # Formatierung
│       ├── date_utils.py          # Datum/Zeit
│       └── crypto.py              # Verschlüsselung
│
├── tests/                         # Tests
│   ├── __init__.py
│   ├── conftest.py                # pytest Fixtures
│   ├── unit/
│   │   ├── test_calculations/
│   │   ├── test_services/
│   │   └── test_utils/
│   ├── integration/
│   │   ├── test_api/
│   │   └── test_tasks/
│   └── e2e/
│
├── alembic/                       # Datenbank-Migrationen
│   ├── env.py
│   ├── script.py.mako
│   └── versions/
│
├── scripts/                       # Hilfsskripte
│   ├── init_db.py                 # Datenbank initialisieren
│   ├── seed_data.py               # Testdaten
│   └── migrate.py                 # Migrationen ausführen
│
├── requirements.txt               # Dependencies
├── requirements-dev.txt           # Dev Dependencies
├── pytest.ini                     # pytest Konfiguration
├── pyproject.toml                 # Projekt-Konfiguration
└── Dockerfile                     # Docker Build
```

---

## 3. API-Design

### 3.1 REST API Konventionen

```
Endpunkt-Schema:
GET    /api/v1/{resource}          # Liste
POST   /api/v1/{resource}          # Erstellen
GET    /api/v1/{resource}/{id}     # Einzeln lesen
PUT    /api/v1/{resource}/{id}     # Vollständig aktualisieren
PATCH  /api/v1/{resource}/{id}     # Teilweise aktualisieren
DELETE /api/v1/{resource}/{id}     # Löschen

Verschachtelte Ressourcen:
GET    /api/v1/projects/{id}/buildings
POST   /api/v1/projects/{id}/calculations

Aktionen:
POST   /api/v1/calculations/{id}/run
POST   /api/v1/reports/{id}/generate
POST   /api/v1/quotes/{id}/convert-to-invoice
```

### 3.2 Request/Response Format

```python
# Erfolgreiche Antwort (einzelnes Objekt)
{
    "data": {
        "id": "uuid",
        "type": "project",
        "attributes": {...}
    },
    "meta": {
        "timestamp": "2025-01-15T10:30:00Z"
    }
}

# Erfolgreiche Antwort (Liste)
{
    "data": [...],
    "meta": {
        "total": 100,
        "page": 1,
        "per_page": 20,
        "pages": 5
    }
}

# Fehlerantwort
{
    "error": {
        "code": "VALIDATION_ERROR",
        "message": "Eingabedaten ungültig",
        "details": [
            {
                "field": "u_value",
                "message": "U-Wert muss positiv sein"
            }
        ]
    }
}
```

### 3.3 Authentifizierung

```
Header: Authorization: Bearer <JWT_TOKEN>

JWT Payload:
{
    "sub": "user_id",
    "org": "organization_id",
    "roles": ["consultant"],
    "exp": 1705312200,
    "iat": 1705308600
}

Token-Typen:
- Access Token: 15 Minuten
- Refresh Token: 7 Tage
```

### 3.4 Pagination

```
Query Parameter:
?page=1&per_page=20&sort=-created_at&filter[status]=active

Response Header:
X-Total-Count: 100
X-Page: 1
X-Per-Page: 20
Link: <...?page=2>; rel="next", <...?page=5>; rel="last"
```

---

## 4. Service Layer

### 4.1 Base Service Pattern

```python
# services/base.py
from typing import Generic, TypeVar, Type, Optional, List
from sqlalchemy.orm import Session
from app.models.base import BaseModel
from app.schemas.common import PaginationParams

ModelType = TypeVar("ModelType", bound=BaseModel)
CreateSchemaType = TypeVar("CreateSchemaType")
UpdateSchemaType = TypeVar("UpdateSchemaType")

class BaseService(Generic[ModelType, CreateSchemaType, UpdateSchemaType]):
    def __init__(self, model: Type[ModelType], db: Session):
        self.model = model
        self.db = db

    async def get(self, id: UUID) -> Optional[ModelType]:
        """Einzelnen Datensatz abrufen"""
        pass

    async def get_multi(
        self,
        pagination: PaginationParams,
        filters: dict = None
    ) -> List[ModelType]:
        """Liste mit Pagination und Filtern"""
        pass

    async def create(self, obj_in: CreateSchemaType) -> ModelType:
        """Neuen Datensatz erstellen"""
        pass

    async def update(
        self,
        id: UUID,
        obj_in: UpdateSchemaType
    ) -> ModelType:
        """Datensatz aktualisieren"""
        pass

    async def delete(self, id: UUID) -> bool:
        """Datensatz löschen"""
        pass
```

### 4.2 Calculation Service

```python
# services/calculation_service.py
class CalculationService(BaseService):

    async def run_calculation(
        self,
        calculation_id: UUID,
        user_id: UUID
    ) -> CalculationResult:
        """
        Führt eine Berechnung durch:
        1. Eingabedaten laden
        2. Validieren
        3. Berechnungs-Engine aufrufen
        4. Ergebnisse validieren
        5. Speichern
        """
        pass

    async def validate_inputs(
        self,
        calculation_type: str,
        input_data: dict
    ) -> ValidationResult:
        """Validiert Eingabedaten vor Berechnung"""
        pass

    async def get_calculation_status(
        self,
        calculation_id: UUID
    ) -> CalculationStatus:
        """Status einer laufenden Berechnung"""
        pass
```

---

## 5. Berechnungs-Engine

### 5.1 Basis-Klasse

```python
# calculations/base.py
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Dict, Any, List

@dataclass
class CalculationInput:
    building_data: Dict
    zone_data: List[Dict]
    component_data: List[Dict]
    system_data: List[Dict]
    climate_data: Dict
    options: Dict = None

@dataclass
class CalculationOutput:
    results: Dict[str, Any]
    summary: Dict[str, Any]
    warnings: List[str]
    calculation_time_ms: int
    version: str
    standard: str

class BaseCalculation(ABC):
    """Basisklasse für alle Berechnungen"""

    STANDARD: str = ""
    VERSION: str = ""

    def __init__(self):
        self.warnings: List[str] = []
        self.errors: List[str] = []

    @abstractmethod
    def validate(self, input_data: CalculationInput) -> bool:
        """Eingabedaten validieren"""
        pass

    @abstractmethod
    def calculate(self, input_data: CalculationInput) -> CalculationOutput:
        """Berechnung durchführen"""
        pass

    def add_warning(self, message: str):
        self.warnings.append(message)
```

### 5.2 U-Wert-Berechnung

```python
# calculations/thermal/u_value.py
from typing import List
from dataclasses import dataclass

@dataclass
class Layer:
    material_id: str
    thickness: float      # [m]
    lambda_value: float   # [W/(mK)]
    name: str = ""

class UValueCalculation(BaseCalculation):
    """
    U-Wert-Berechnung nach DIN EN ISO 6946
    """

    STANDARD = "DIN EN ISO 6946"
    VERSION = "2018-03"

    # Wärmeübergangswiderstände nach Tabelle 7
    RSI_HORIZONTAL = 0.13  # m²K/W
    RSI_UPWARD = 0.10
    RSI_DOWNWARD = 0.17
    RSE_EXTERIOR = 0.04
    RSE_INTERIOR = 0.13

    def calculate_r_value(
        self,
        layers: List[Layer],
        heat_flow_direction: str = "horizontal"
    ) -> float:
        """
        Berechnet Gesamtwärmedurchlasswiderstand R

        R = Rsi + Σ(d/λ) + Rse
        """
        # Rsi nach Wärmestromrichtung
        if heat_flow_direction == "upward":
            rsi = self.RSI_UPWARD
        elif heat_flow_direction == "downward":
            rsi = self.RSI_DOWNWARD
        else:
            rsi = self.RSI_HORIZONTAL

        # Schichtwiderstand
        r_layers = sum(
            layer.thickness / layer.lambda_value
            for layer in layers
        )

        # Rse (außen)
        rse = self.RSE_EXTERIOR

        return rsi + r_layers + rse

    def calculate_u_value(
        self,
        layers: List[Layer],
        heat_flow_direction: str = "horizontal"
    ) -> float:
        """
        Berechnet U-Wert [W/(m²K)]

        U = 1 / R
        """
        r_value = self.calculate_r_value(layers, heat_flow_direction)
        return 1 / r_value

    def calculate_temperature_profile(
        self,
        layers: List[Layer],
        temp_inside: float,
        temp_outside: float
    ) -> List[dict]:
        """
        Berechnet Temperaturverlauf durch Bauteil
        für Taupunktanalyse
        """
        pass
```

### 5.3 Heizlast-Berechnung

```python
# calculations/thermal/heat_load.py
class HeatLoadCalculation(BaseCalculation):
    """
    Heizlastberechnung nach DIN EN 12831-1
    """

    STANDARD = "DIN EN 12831-1"
    VERSION = "2017"

    def calculate_transmission_heat_loss(
        self,
        components: List[ComponentData],
        temp_int: float,
        temp_ext: float
    ) -> float:
        """
        Transmissionswärmeverlust Φ_T

        Φ_T = Σ(A * U * f) * Δθ
        """
        h_t = 0  # Transmissionswärmetransferkoefffizient

        for comp in components:
            f_factor = self._get_temperature_factor(
                comp.adjacent_to,
                temp_int,
                temp_ext
            )
            h_t += comp.area * comp.u_value * f_factor

        return h_t * (temp_int - temp_ext)

    def calculate_ventilation_heat_loss(
        self,
        volume: float,
        air_change_rate: float,
        temp_int: float,
        temp_ext: float
    ) -> float:
        """
        Lüftungswärmeverlust Φ_V

        Φ_V = V * n * ρ * cp * Δθ
        """
        rho_cp = 0.34  # Wh/(m³K) für Luft
        h_v = volume * air_change_rate * rho_cp
        return h_v * (temp_int - temp_ext)

    def calculate_room_heat_load(
        self,
        room: RoomData
    ) -> RoomHeatLoad:
        """Berechnet Heizlast für einen Raum"""
        pass

    def calculate_building_heat_load(
        self,
        building: BuildingData
    ) -> BuildingHeatLoad:
        """Berechnet Heizlast für gesamtes Gebäude"""
        pass
```

---

## 6. Celery Tasks

### 6.1 Konfiguration

```python
# tasks/celery_app.py
from celery import Celery

celery_app = Celery(
    "energieberater",
    broker="redis://redis:6379/3",
    backend="redis://redis:6379/3",
)

celery_app.conf.update(
    task_serializer="json",
    accept_content=["json"],
    result_serializer="json",
    timezone="Europe/Berlin",
    enable_utc=True,
    task_track_started=True,
    task_time_limit=600,  # 10 Minuten
    worker_prefetch_multiplier=1,
    task_routes={
        "app.tasks.calculation_tasks.*": {"queue": "calculations"},
        "app.tasks.report_tasks.*": {"queue": "reports"},
        "app.tasks.email_tasks.*": {"queue": "emails"},
    },
)

# Periodische Tasks
celery_app.conf.beat_schedule = {
    "cleanup-expired-sessions": {
        "task": "app.tasks.cleanup_tasks.cleanup_sessions",
        "schedule": 3600.0,  # Jede Stunde
    },
    "backup-database": {
        "task": "app.tasks.backup_tasks.backup_database",
        "schedule": crontab(hour=2, minute=0),  # 02:00 Uhr
    },
}
```

### 6.2 Berechnungs-Tasks

```python
# tasks/calculation_tasks.py
from celery import shared_task
from app.services.calculation_service import CalculationService

@shared_task(
    bind=True,
    max_retries=3,
    default_retry_delay=60
)
def run_calculation_async(
    self,
    calculation_id: str,
    user_id: str
):
    """
    Führt Berechnung asynchron aus
    """
    try:
        service = CalculationService()
        result = service.run_calculation(
            calculation_id=calculation_id,
            user_id=user_id
        )
        return {"status": "success", "result_id": str(result.id)}
    except Exception as exc:
        self.retry(exc=exc)

@shared_task
def run_batch_calculations(
    project_id: str,
    calculation_type: str
):
    """
    Führt Batch-Berechnungen für ein Projekt aus
    """
    pass
```

---

## 7. Error Handling

### 7.1 Exception Klassen

```python
# core/exceptions.py
class EnergieberaterException(Exception):
    """Basis-Exception"""
    code: str = "INTERNAL_ERROR"
    status_code: int = 500

class ValidationError(EnergieberaterException):
    code = "VALIDATION_ERROR"
    status_code = 422

class NotFoundError(EnergieberaterException):
    code = "NOT_FOUND"
    status_code = 404

class AuthenticationError(EnergieberaterException):
    code = "AUTHENTICATION_ERROR"
    status_code = 401

class AuthorizationError(EnergieberaterException):
    code = "AUTHORIZATION_ERROR"
    status_code = 403

class CalculationError(EnergieberaterException):
    code = "CALCULATION_ERROR"
    status_code = 400

class RateLimitError(EnergieberaterException):
    code = "RATE_LIMIT_EXCEEDED"
    status_code = 429
```

### 7.2 Exception Handler

```python
# api/middleware.py
from fastapi import Request
from fastapi.responses import JSONResponse

async def exception_handler(
    request: Request,
    exc: EnergieberaterException
):
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "error": {
                "code": exc.code,
                "message": str(exc),
                "request_id": request.state.request_id
            }
        }
    )
```

---

## 8. Testing

### 8.1 Test-Struktur

```python
# tests/conftest.py
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from app.main import create_app
from app.models.base import Base

@pytest.fixture(scope="session")
def engine():
    return create_engine("postgresql://test:test@localhost/test")

@pytest.fixture
def db_session(engine):
    Base.metadata.create_all(engine)
    Session = sessionmaker(bind=engine)
    session = Session()
    yield session
    session.rollback()
    Base.metadata.drop_all(engine)

@pytest.fixture
def client(db_session):
    app = create_app()
    with TestClient(app) as client:
        yield client
```

### 8.2 Unit Tests für Berechnungen

```python
# tests/unit/test_calculations/test_u_value.py
import pytest
from app.calculations.thermal.u_value import UValueCalculation, Layer

class TestUValueCalculation:

    def test_simple_wall(self):
        """Test U-Wert einer einfachen Wand"""
        calc = UValueCalculation()
        layers = [
            Layer("brick", 0.24, 0.7, "Mauerwerk"),
            Layer("insulation", 0.16, 0.035, "Dämmung"),
            Layer("plaster", 0.02, 0.7, "Putz"),
        ]
        u_value = calc.calculate_u_value(layers)
        assert 0.18 <= u_value <= 0.22  # Erwarteter Bereich

    def test_geg_requirement(self):
        """Test ob U-Wert GEG-Anforderung erfüllt"""
        pass
```

---

*Letzte Aktualisierung: 2025-11-25*
*Version: 0.1.0*
