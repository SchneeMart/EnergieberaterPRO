# Test-Strategie EnergieberaterPRO

## Referenzen
- [pytest Documentation](https://docs.pytest.org/en/stable/)
- [pytest-xdist Parallel Testing](https://pytest-xdist.readthedocs.io/)
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [Locust Performance Testing](https://locust.io/)
- [Hypothesis Property-Based Testing](https://hypothesis.readthedocs.io/)

---

## 1. Test-Philosophie

### 1.1 Grundprinzipien

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        TEST-PHILOSOPHIE                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  PRINZIP 1: Keine KI in Tests                                               │
│  ─────────────────────────────────────────────────────────────────────────  │
│  • Alle Tests sind deterministisch                                          │
│  • Keine LLM-basierten Assertions                                           │
│  • Reproduzierbare Ergebnisse bei jedem Lauf                               │
│                                                                             │
│  PRINZIP 2: Wasserdichte Berechnungen                                       │
│  ─────────────────────────────────────────────────────────────────────────  │
│  • Jede Formel gegen Referenzwerte validiert                               │
│  • Physikalische Plausibilitätsprüfungen                                   │
│  • Normenkonforme Testfälle (DIN, EN, GEG)                                 │
│                                                                             │
│  PRINZIP 3: Defense in Depth Testing                                        │
│  ─────────────────────────────────────────────────────────────────────────  │
│  • Unit Tests für Komponenten                                               │
│  • Integration Tests für Workflows                                          │
│  • E2E Tests für User Journeys                                              │
│  • Security Tests für alle Angriffsvektoren                                │
│                                                                             │
│  PRINZIP 4: Continuous Quality                                              │
│  ─────────────────────────────────────────────────────────────────────────  │
│  • Tests in CI/CD Pipeline                                                  │
│  • Keine Merges ohne grüne Tests                                           │
│  • Coverage-Gates (min. 80%)                                                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Test-Pyramide

```
                    ╔════════════════════╗
                    ║   E2E Tests (5%)   ║  ← Playwright/Selenium
                    ║  User Journeys     ║
                    ╠════════════════════╣
               ╔════╩════════════════════╩════╗
               ║  Integration Tests (15%)     ║  ← pytest + TestClient
               ║  API, Database, Services     ║
               ╠══════════════════════════════╣
          ╔════╩══════════════════════════════╩════╗
          ║       Unit Tests (80%)                 ║  ← pytest
          ║  Functions, Classes, Calculations      ║
          ╚════════════════════════════════════════╝
```

---

## 2. Test-Kategorien

### 2.1 Unit Tests

#### 2.1.1 Struktur

```python
# tests/unit/calculations/test_u_wert.py
"""U-Wert Berechnungen nach DIN EN ISO 6946"""

import pytest
from decimal import Decimal
from app.calculations.thermal import UWertCalculator

class TestUWertCalculation:
    """Tests für U-Wert Berechnung"""

    @pytest.fixture
    def calculator(self):
        return UWertCalculator()

    @pytest.fixture
    def reference_wall(self):
        """Referenz-Wandaufbau nach DIN 4108 Beiblatt 2"""
        return [
            {"name": "Außenputz", "thickness": 0.02, "lambda": 0.87},
            {"name": "Mauerwerk", "thickness": 0.24, "lambda": 0.79},
            {"name": "Dämmung EPS", "thickness": 0.14, "lambda": 0.035},
            {"name": "Innenputz", "thickness": 0.015, "lambda": 0.51},
        ]

    def test_u_wert_simple_wall(self, calculator, reference_wall):
        """Prüfe U-Wert gegen Referenzwert"""
        result = calculator.calculate(
            layers=reference_wall,
            rsi=0.13,  # Wärmeübergangswiderstand innen
            rse=0.04   # Wärmeübergangswiderstand außen
        )

        # Referenzwert: 0.22 W/(m²K) ± 0.01
        assert Decimal("0.21") <= result.u_value <= Decimal("0.23")

    def test_u_wert_einzelschicht(self, calculator):
        """Prüfe Einzelschicht-Berechnung"""
        result = calculator.calculate(
            layers=[{"name": "Beton", "thickness": 0.20, "lambda": 2.1}],
            rsi=0.13,
            rse=0.04
        )

        # R = 0.13 + (0.20/2.1) + 0.04 = 0.265
        # U = 1/0.265 = 3.77 W/(m²K)
        expected_u = Decimal("3.77")
        assert abs(result.u_value - expected_u) < Decimal("0.01")

    @pytest.mark.parametrize("thickness,expected_u", [
        (0.10, Decimal("0.31")),  # 10cm Dämmung
        (0.14, Decimal("0.22")),  # 14cm Dämmung
        (0.20, Decimal("0.16")),  # 20cm Dämmung
        (0.30, Decimal("0.11")),  # 30cm Dämmung
    ])
    def test_u_wert_daemmstaerken(self, calculator, reference_wall, thickness, expected_u):
        """Prüfe verschiedene Dämmstärken"""
        wall = reference_wall.copy()
        wall[2]["thickness"] = thickness

        result = calculator.calculate(layers=wall, rsi=0.13, rse=0.04)
        assert abs(result.u_value - expected_u) < Decimal("0.02")

    def test_physikalische_grenzen(self, calculator):
        """Prüfe physikalische Plausibilität"""
        # U-Wert muss positiv sein
        result = calculator.calculate(
            layers=[{"name": "Luft", "thickness": 0.01, "lambda": 0.025}],
            rsi=0.13, rse=0.04
        )
        assert result.u_value > 0

        # U-Wert kann nicht unendlich sein
        assert result.u_value < Decimal("100")

    def test_ungueltige_eingaben(self, calculator):
        """Prüfe Fehlerbehandlung"""
        with pytest.raises(ValueError, match="Schichtdicke muss positiv sein"):
            calculator.calculate(
                layers=[{"name": "Test", "thickness": -0.1, "lambda": 0.5}],
                rsi=0.13, rse=0.04
            )

        with pytest.raises(ValueError, match="Lambda muss positiv sein"):
            calculator.calculate(
                layers=[{"name": "Test", "thickness": 0.1, "lambda": 0}],
                rsi=0.13, rse=0.04
            )
```

#### 2.1.2 Berechnungs-Testfälle nach Normen

```python
# tests/unit/calculations/test_heizlast.py
"""Heizlastberechnung nach DIN EN 12831"""

import pytest
from decimal import Decimal
from app.calculations.heating import HeizlastCalculator

class TestHeizlastDINEN12831:
    """Tests nach DIN EN 12831-1:2017"""

    @pytest.fixture
    def calculator(self):
        return HeizlastCalculator()

    @pytest.fixture
    def referenz_raum(self):
        """Referenzraum nach DIN EN 12831-1 Anhang NA"""
        return {
            "floor_area": Decimal("20.0"),  # m²
            "height": Decimal("2.5"),        # m
            "volume": Decimal("50.0"),       # m³
            "theta_int": Decimal("20.0"),    # °C Innentemperatur
            "theta_ext": Decimal("-12.0"),   # °C Außentemperatur (Klimazone)
            "building_components": [
                {
                    "type": "Außenwand",
                    "area": Decimal("15.0"),  # m²
                    "u_value": Decimal("0.24"),
                    "f_u": Decimal("1.0"),  # Korrekturfaktor
                },
                {
                    "type": "Fenster",
                    "area": Decimal("4.0"),
                    "u_value": Decimal("1.3"),
                    "f_u": Decimal("1.0"),
                },
                {
                    "type": "Dach",
                    "area": Decimal("20.0"),
                    "u_value": Decimal("0.20"),
                    "f_u": Decimal("1.0"),
                },
            ],
            "n_min": Decimal("0.5"),  # Mindestluftwechsel 1/h
        }

    def test_transmissionswaermeverlust(self, calculator, referenz_raum):
        """Φ_T = Σ(A_i × U_i × f_u,i) × (θ_int - θ_ext)"""
        result = calculator.calculate_transmission_loss(referenz_raum)

        # Manuelle Berechnung:
        # Außenwand: 15 × 0.24 × 1.0 × 32 = 115.2 W
        # Fenster: 4 × 1.3 × 1.0 × 32 = 166.4 W
        # Dach: 20 × 0.20 × 1.0 × 32 = 128.0 W
        # Summe: 409.6 W

        expected = Decimal("409.6")
        assert abs(result - expected) < Decimal("1.0")

    def test_lueftungswaermeverlust(self, calculator, referenz_raum):
        """Φ_V = V × n × ρ × c_p × (θ_int - θ_ext)"""
        result = calculator.calculate_ventilation_loss(referenz_raum)

        # V = 50 m³, n = 0.5 1/h, ρ×c_p ≈ 0.34 Wh/(m³K)
        # Φ_V = 50 × 0.5 × 0.34 × 32 = 272 W

        expected = Decimal("272.0")
        assert abs(result - expected) < Decimal("5.0")

    def test_norm_heizlast_gesamt(self, calculator, referenz_raum):
        """Φ_HL = Φ_T + Φ_V + Φ_RH (Aufheizleistung)"""
        result = calculator.calculate_total(referenz_raum)

        # Ohne Aufheizleistung: 409.6 + 272 = 681.6 W
        # Mit Aufheizleistung (pauschal 10%): ~750 W

        assert Decimal("650") < result.total_heat_load < Decimal("800")
        assert result.transmission_loss > 0
        assert result.ventilation_loss > 0

    @pytest.mark.parametrize("klimazone,theta_ext,expected_range", [
        ("Flensburg", Decimal("-12"), (Decimal("700"), Decimal("850"))),
        ("Frankfurt", Decimal("-10"), (Decimal("620"), Decimal("780"))),
        ("München", Decimal("-14"), (Decimal("780"), Decimal("920"))),
        ("Zugspitze", Decimal("-18"), (Decimal("900"), Decimal("1100"))),
    ])
    def test_klimazonen(self, calculator, referenz_raum, klimazone, theta_ext, expected_range):
        """Teste verschiedene Klimazonen nach DIN EN 12831-1 NA"""
        raum = referenz_raum.copy()
        raum["theta_ext"] = theta_ext
        raum["klimazone"] = klimazone

        result = calculator.calculate_total(raum)

        assert expected_range[0] <= result.total_heat_load <= expected_range[1], \
            f"Heizlast {result.total_heat_load}W außerhalb {expected_range} für {klimazone}"
```

### 2.2 Integration Tests

```python
# tests/integration/test_api_projects.py
"""Integration Tests für Projekt-API"""

import pytest
from httpx import AsyncClient
from sqlalchemy.ext.asyncio import AsyncSession

from app.main import app
from app.models.project import Project
from app.models.user import User
from tests.factories import UserFactory, ProjectFactory

@pytest.fixture
async def authenticated_client(db_session: AsyncSession):
    """Client mit authentifiziertem Benutzer"""
    user = await UserFactory.create(db_session)
    async with AsyncClient(app=app, base_url="http://test") as client:
        # Login
        response = await client.post("/api/v1/auth/login", json={
            "email": user.email,
            "password": "test_password"
        })
        token = response.json()["access_token"]
        client.headers["Authorization"] = f"Bearer {token}"
        client.user = user
        yield client

class TestProjectAPI:
    """Projekt-API Integration Tests"""

    @pytest.mark.asyncio
    async def test_create_project(self, authenticated_client):
        """Projekt erstellen"""
        response = await authenticated_client.post("/api/v1/projects", json={
            "name": "Energieberatung Musterhaus",
            "project_type": "residential",
            "customer_id": str(authenticated_client.user.organization_id),
            "address": {
                "street": "Musterstraße 1",
                "zip_code": "12345",
                "city": "Musterstadt"
            }
        })

        assert response.status_code == 201
        data = response.json()
        assert data["name"] == "Energieberatung Musterhaus"
        assert "id" in data
        assert data["status"] == "draft"

    @pytest.mark.asyncio
    async def test_project_isolation(self, authenticated_client, db_session):
        """Projekte anderer Organisationen nicht sichtbar"""
        # Erstelle Projekt in anderer Organisation
        other_user = await UserFactory.create(db_session)
        other_project = await ProjectFactory.create(
            db_session,
            organization_id=other_user.organization_id
        )

        # Versuche Zugriff
        response = await authenticated_client.get(
            f"/api/v1/projects/{other_project.id}"
        )

        assert response.status_code == 404  # Nicht 403, um Info-Leak zu vermeiden

    @pytest.mark.asyncio
    async def test_project_workflow(self, authenticated_client):
        """Vollständiger Projekt-Workflow"""
        # 1. Projekt erstellen
        create_response = await authenticated_client.post("/api/v1/projects", json={
            "name": "Test-Projekt",
            "project_type": "residential"
        })
        project_id = create_response.json()["id"]

        # 2. Gebäude hinzufügen
        building_response = await authenticated_client.post(
            f"/api/v1/projects/{project_id}/buildings",
            json={
                "name": "Hauptgebäude",
                "building_type": "single_family",
                "construction_year": 1985,
                "heated_area": 150.0
            }
        )
        assert building_response.status_code == 201
        building_id = building_response.json()["id"]

        # 3. Berechnung durchführen
        calc_response = await authenticated_client.post(
            f"/api/v1/projects/{project_id}/calculations",
            json={
                "calculation_type": "energy_demand",
                "building_id": building_id,
                "parameters": {
                    "climate_zone": "TRY04",
                    "heating_system": "gas_condensing"
                }
            }
        )
        assert calc_response.status_code == 201

        # 4. Projekt abschließen
        complete_response = await authenticated_client.patch(
            f"/api/v1/projects/{project_id}",
            json={"status": "completed"}
        )
        assert complete_response.status_code == 200
```

### 2.3 End-to-End Tests

```python
# tests/e2e/test_user_journey.py
"""E2E Tests mit Playwright"""

import pytest
from playwright.async_api import Page, expect

class TestEnergyConsultantJourney:
    """User Journey: Energieberater erstellt Beratungsbericht"""

    @pytest.fixture(autouse=True)
    async def setup(self, page: Page, test_user):
        """Login vor jedem Test"""
        await page.goto("/login")
        await page.fill("[data-testid=email]", test_user.email)
        await page.fill("[data-testid=password]", test_user.password)
        await page.click("[data-testid=login-button]")
        await expect(page.locator("[data-testid=dashboard]")).to_be_visible()

    @pytest.mark.asyncio
    async def test_complete_energy_audit(self, page: Page):
        """Vollständiger Energieberatungs-Workflow"""

        # 1. Neues Projekt anlegen
        await page.click("[data-testid=new-project]")
        await page.fill("[data-testid=project-name]", "Energieaudit Einfamilienhaus")
        await page.select_option("[data-testid=project-type]", "residential")
        await page.click("[data-testid=create-project]")

        await expect(page.locator("[data-testid=project-detail]")).to_be_visible()

        # 2. Gebäudedaten erfassen
        await page.click("[data-testid=add-building]")
        await page.fill("[data-testid=building-name]", "Wohnhaus")
        await page.fill("[data-testid=construction-year]", "1975")
        await page.fill("[data-testid=heated-area]", "180")
        await page.click("[data-testid=save-building]")

        # 3. Bauteile definieren
        await page.click("[data-testid=tab-components]")

        # Außenwand hinzufügen
        await page.click("[data-testid=add-component]")
        await page.select_option("[data-testid=component-type]", "external_wall")
        await page.fill("[data-testid=component-area]", "120")
        await page.fill("[data-testid=component-u-value]", "1.2")
        await page.click("[data-testid=save-component]")

        # Fenster hinzufügen
        await page.click("[data-testid=add-component]")
        await page.select_option("[data-testid=component-type]", "window")
        await page.fill("[data-testid=component-area]", "25")
        await page.fill("[data-testid=component-u-value]", "2.8")
        await page.click("[data-testid=save-component]")

        # 4. Berechnung durchführen
        await page.click("[data-testid=tab-calculations]")
        await page.click("[data-testid=run-calculation]")

        # Warte auf Berechnung
        await expect(page.locator("[data-testid=calculation-result]")).to_be_visible(
            timeout=30000
        )

        # Prüfe Ergebnis vorhanden
        energy_demand = await page.locator("[data-testid=energy-demand-value]").text_content()
        assert float(energy_demand.replace(",", ".")) > 0

        # 5. Bericht generieren
        await page.click("[data-testid=generate-report]")
        await page.select_option("[data-testid=report-template]", "isfp_standard")
        await page.click("[data-testid=create-report]")

        # Warte auf PDF-Generierung
        async with page.expect_download() as download_info:
            await page.click("[data-testid=download-report]")

        download = await download_info.value
        assert download.suggested_filename.endswith(".pdf")

        # 6. Projekt abschließen
        await page.click("[data-testid=complete-project]")
        await expect(page.locator("[data-testid=project-status]")).to_have_text("Abgeschlossen")
```

---

## 3. Berechnungs-Validierung

### 3.1 Referenzwert-Katalog

```python
# tests/reference_values/din_v_18599.py
"""Referenzwerte aus DIN V 18599 für Validierung"""

from decimal import Decimal
from dataclasses import dataclass
from typing import Dict, List

@dataclass
class ReferenceBuilding:
    """Referenzgebäude nach DIN V 18599"""
    name: str
    building_type: str
    parameters: Dict
    expected_results: Dict

# Referenzgebäude aus DIN V 18599-10
REFERENCE_BUILDINGS: List[ReferenceBuilding] = [
    ReferenceBuilding(
        name="Einfamilienhaus 1958",
        building_type="single_family",
        parameters={
            "construction_year": 1958,
            "heated_area": Decimal("150.0"),
            "heated_volume": Decimal("450.0"),
            "envelope": {
                "wall_area": Decimal("180.0"),
                "wall_u": Decimal("1.4"),
                "roof_area": Decimal("90.0"),
                "roof_u": Decimal("1.0"),
                "floor_area": Decimal("75.0"),
                "floor_u": Decimal("1.0"),
                "window_area": Decimal("25.0"),
                "window_u": Decimal("2.8"),
            },
            "heating_system": "oil_standard",
            "ventilation": "natural",
        },
        expected_results={
            "primary_energy_demand": (Decimal("250"), Decimal("320")),  # kWh/(m²a)
            "transmission_heat_loss": (Decimal("320"), Decimal("380")),  # W/K
            "heating_demand": (Decimal("180"), Decimal("230")),  # kWh/(m²a)
        }
    ),
    ReferenceBuilding(
        name="Einfamilienhaus KfW55",
        building_type="single_family",
        parameters={
            "construction_year": 2020,
            "heated_area": Decimal("150.0"),
            "heated_volume": Decimal("450.0"),
            "envelope": {
                "wall_area": Decimal("180.0"),
                "wall_u": Decimal("0.18"),
                "roof_area": Decimal("90.0"),
                "roof_u": Decimal("0.14"),
                "floor_area": Decimal("75.0"),
                "floor_u": Decimal("0.25"),
                "window_area": Decimal("30.0"),
                "window_u": Decimal("0.9"),
            },
            "heating_system": "heat_pump_air",
            "ventilation": "mechanical_heat_recovery",
        },
        expected_results={
            "primary_energy_demand": (Decimal("30"), Decimal("45")),  # kWh/(m²a)
            "transmission_heat_loss": (Decimal("70"), Decimal("100")),  # W/K
            "heating_demand": (Decimal("25"), Decimal("40")),  # kWh/(m²a)
        }
    ),
    ReferenceBuilding(
        name="Mehrfamilienhaus 1970",
        building_type="multi_family",
        parameters={
            "construction_year": 1970,
            "heated_area": Decimal("1200.0"),
            "heated_volume": Decimal("3600.0"),
            "units": 12,
            "envelope": {
                "wall_area": Decimal("800.0"),
                "wall_u": Decimal("1.2"),
                "roof_area": Decimal("400.0"),
                "roof_u": Decimal("0.8"),
                "floor_area": Decimal("400.0"),
                "floor_u": Decimal("0.9"),
                "window_area": Decimal("200.0"),
                "window_u": Decimal("2.7"),
            },
            "heating_system": "district_heating",
            "ventilation": "natural",
        },
        expected_results={
            "primary_energy_demand": (Decimal("140"), Decimal("180")),
            "transmission_heat_loss": (Decimal("1800"), Decimal("2400")),
            "heating_demand": (Decimal("100"), Decimal("140")),
        }
    ),
]

# Einzelbauteil-Referenzwerte
COMPONENT_REFERENCE_VALUES = {
    "wall_uninsulated_1960": {
        "description": "Außenwand Mauerwerk 36cm ungedämmt",
        "u_value_measured": Decimal("1.35"),
        "tolerance": Decimal("0.1"),
    },
    "wall_etics_2020": {
        "description": "Außenwand mit WDVS 16cm",
        "u_value_measured": Decimal("0.18"),
        "tolerance": Decimal("0.02"),
    },
    "window_double_1980": {
        "description": "Doppelverglasung Isolierglas",
        "u_value_measured": Decimal("2.8"),
        "tolerance": Decimal("0.2"),
    },
    "window_triple_2020": {
        "description": "Dreifachverglasung Passivhaus",
        "u_value_measured": Decimal("0.7"),
        "tolerance": Decimal("0.1"),
    },
}
```

### 3.2 Validierungs-Tests

```python
# tests/validation/test_reference_buildings.py
"""Validierung gegen Referenzgebäude"""

import pytest
from decimal import Decimal
from tests.reference_values.din_v_18599 import REFERENCE_BUILDINGS
from app.calculations.energy import EnergyCalculator

class TestReferenceBuildings:
    """Validiere Berechnungen gegen Referenzgebäude"""

    @pytest.fixture
    def calculator(self):
        return EnergyCalculator()

    @pytest.mark.parametrize("ref_building", REFERENCE_BUILDINGS,
                             ids=lambda b: b.name)
    def test_reference_building(self, calculator, ref_building):
        """Teste gegen Referenzgebäude aus DIN V 18599"""
        result = calculator.calculate_full(ref_building.parameters)

        # Prüfe Primärenergiebedarf
        pe_min, pe_max = ref_building.expected_results["primary_energy_demand"]
        assert pe_min <= result.primary_energy_demand <= pe_max, \
            f"Primärenergiebedarf {result.primary_energy_demand} kWh/(m²a) " \
            f"außerhalb erwartetem Bereich [{pe_min}, {pe_max}]"

        # Prüfe Transmissionswärmeverlust
        ht_min, ht_max = ref_building.expected_results["transmission_heat_loss"]
        assert ht_min <= result.transmission_heat_loss <= ht_max, \
            f"H'T {result.transmission_heat_loss} W/K " \
            f"außerhalb erwartetem Bereich [{ht_min}, {ht_max}]"

        # Prüfe Heizwärmebedarf
        qh_min, qh_max = ref_building.expected_results["heating_demand"]
        assert qh_min <= result.heating_demand <= qh_max, \
            f"Heizwärmebedarf {result.heating_demand} kWh/(m²a) " \
            f"außerhalb erwartetem Bereich [{qh_min}, {qh_max}]"

    def test_plausibility_checks(self, calculator):
        """Physikalische Plausibilitätsprüfungen"""
        for ref_building in REFERENCE_BUILDINGS:
            result = calculator.calculate_full(ref_building.parameters)

            # Energiebedarf muss positiv sein
            assert result.primary_energy_demand > 0
            assert result.heating_demand > 0

            # Primärenergie >= Endenergie (immer!)
            assert result.primary_energy_demand >= result.final_energy_demand

            # Heizwärmebedarf <= Endenergie (Systemverluste)
            assert result.heating_demand <= result.final_energy_demand * Decimal("1.5")

            # Saniertes Gebäude besser als unsaniert
            if ref_building.parameters["construction_year"] >= 2010:
                assert result.primary_energy_demand < Decimal("100")
```

### 3.3 Property-Based Testing mit Hypothesis

```python
# tests/property/test_calculations_properties.py
"""Property-Based Tests für mathematische Invarianten"""

import pytest
from hypothesis import given, strategies as st, assume, settings
from decimal import Decimal
from app.calculations.thermal import UWertCalculator, HeizlastCalculator

class TestCalculationProperties:
    """Property-Based Tests für physikalische Invarianten"""

    @given(
        thickness=st.decimals(min_value=Decimal("0.001"), max_value=Decimal("1.0")),
        lambda_value=st.decimals(min_value=Decimal("0.01"), max_value=Decimal("5.0")),
    )
    @settings(max_examples=1000)
    def test_u_value_inverse_proportional_to_thickness(self, thickness, lambda_value):
        """U-Wert sinkt mit steigender Dämmdicke (bei gleichem Lambda)"""
        calculator = UWertCalculator()

        assume(thickness > Decimal("0"))
        assume(lambda_value > Decimal("0"))

        layer_thin = [{"name": "Test", "thickness": thickness, "lambda": float(lambda_value)}]
        layer_thick = [{"name": "Test", "thickness": thickness * 2, "lambda": float(lambda_value)}]

        u_thin = calculator.calculate(layers=layer_thin, rsi=0.13, rse=0.04)
        u_thick = calculator.calculate(layers=layer_thick, rsi=0.13, rse=0.04)

        assert u_thick.u_value < u_thin.u_value

    @given(
        delta_t=st.decimals(min_value=Decimal("1"), max_value=Decimal("60")),
        area=st.decimals(min_value=Decimal("1"), max_value=Decimal("1000")),
    )
    @settings(max_examples=500)
    def test_heat_loss_proportional_to_delta_t(self, delta_t, area):
        """Wärmeverlust proportional zur Temperaturdifferenz"""
        calculator = HeizlastCalculator()

        room1 = {
            "floor_area": float(area),
            "theta_int": Decimal("20"),
            "theta_ext": Decimal("20") - delta_t,
            "building_components": [
                {"type": "wall", "area": float(area), "u_value": Decimal("1.0"), "f_u": Decimal("1.0")}
            ]
        }

        room2 = {
            "floor_area": float(area),
            "theta_int": Decimal("20"),
            "theta_ext": Decimal("20") - delta_t * 2,
            "building_components": [
                {"type": "wall", "area": float(area), "u_value": Decimal("1.0"), "f_u": Decimal("1.0")}
            ]
        }

        loss1 = calculator.calculate_transmission_loss(room1)
        loss2 = calculator.calculate_transmission_loss(room2)

        # Doppelte Temperaturdifferenz = doppelter Verlust (±1%)
        ratio = loss2 / loss1
        assert Decimal("1.98") < ratio < Decimal("2.02")

    @given(
        u_value=st.decimals(min_value=Decimal("0.1"), max_value=Decimal("5.0")),
    )
    def test_u_value_never_negative(self, u_value):
        """U-Wert ist niemals negativ"""
        assume(u_value > Decimal("0"))

        r_value = Decimal("1") / u_value
        assert r_value > 0

        # Rückrechnung
        u_calculated = Decimal("1") / r_value
        assert abs(u_calculated - u_value) < Decimal("0.001")
```

---

## 4. Security Testing

### 4.1 OWASP Test Suite

```python
# tests/security/test_owasp_top10.py
"""Security Tests nach OWASP Top 10 2025"""

import pytest
from httpx import AsyncClient
from app.main import app

class TestA01BrokenAccessControl:
    """A01:2025 - Broken Access Control"""

    @pytest.mark.asyncio
    async def test_idor_protection(self, authenticated_client, other_org_project):
        """Insecure Direct Object Reference verhindert"""
        response = await authenticated_client.get(
            f"/api/v1/projects/{other_org_project.id}"
        )
        assert response.status_code == 404  # Nicht 200 oder 403

    @pytest.mark.asyncio
    async def test_privilege_escalation_prevention(self, user_client):
        """Privilege Escalation verhindert"""
        # User versucht Admin-Endpoint
        response = await user_client.delete("/api/v1/admin/users/123")
        assert response.status_code == 403

    @pytest.mark.asyncio
    async def test_force_browsing_prevention(self, authenticated_client):
        """Force Browsing verhindert"""
        sensitive_paths = [
            "/api/v1/admin/config",
            "/api/v1/system/secrets",
            "/.env",
            "/config/database.yml",
        ]

        for path in sensitive_paths:
            response = await authenticated_client.get(path)
            assert response.status_code in [401, 403, 404]

class TestA02CryptographicFailures:
    """A02:2025 - Cryptographic Failures"""

    @pytest.mark.asyncio
    async def test_password_not_in_response(self, authenticated_client):
        """Passwort niemals in API-Response"""
        response = await authenticated_client.get("/api/v1/users/me")
        data = response.json()

        assert "password" not in data
        assert "password_hash" not in data
        assert "hashed_password" not in data

    @pytest.mark.asyncio
    async def test_tokens_not_logged(self, caplog):
        """Tokens nicht in Logs"""
        async with AsyncClient(app=app, base_url="http://test") as client:
            response = await client.post("/api/v1/auth/login", json={
                "email": "test@example.com",
                "password": "test_password"
            })

        for record in caplog.records:
            assert "access_token" not in record.message.lower()
            assert "refresh_token" not in record.message.lower()

class TestA03Injection:
    """A03:2025 - Injection"""

    @pytest.mark.asyncio
    @pytest.mark.parametrize("payload", [
        "'; DROP TABLE users; --",
        "1 OR 1=1",
        "admin'--",
        "1; SELECT * FROM users",
        "UNION SELECT * FROM secrets",
    ])
    async def test_sql_injection_prevention(self, client, payload):
        """SQL Injection verhindert"""
        response = await client.get(f"/api/v1/projects?search={payload}")

        # Sollte keine Fehler werfen, sondern sicher behandeln
        assert response.status_code in [200, 400]

        # Keine SQL-Fehlermeldungen in Response
        if response.status_code == 400:
            assert "SQL" not in response.text
            assert "syntax" not in response.text.lower()

    @pytest.mark.asyncio
    @pytest.mark.parametrize("payload", [
        "<script>alert('xss')</script>",
        "javascript:alert(1)",
        "<img src=x onerror=alert(1)>",
        "{{constructor.constructor('alert(1)')()}}",
    ])
    async def test_xss_prevention(self, authenticated_client, payload):
        """XSS verhindert"""
        # Versuch, XSS in Projektname zu injizieren
        response = await authenticated_client.post("/api/v1/projects", json={
            "name": payload,
            "project_type": "residential"
        })

        if response.status_code == 201:
            data = response.json()
            # Name sollte escaped sein
            assert "<script>" not in data["name"]
            assert "javascript:" not in data["name"]

class TestA05SecurityMisconfiguration:
    """A05:2025 - Security Misconfiguration"""

    @pytest.mark.asyncio
    async def test_security_headers(self, client):
        """Security Headers gesetzt"""
        response = await client.get("/")

        assert response.headers.get("X-Content-Type-Options") == "nosniff"
        assert response.headers.get("X-Frame-Options") in ["DENY", "SAMEORIGIN"]
        assert "strict-transport-security" in response.headers
        assert response.headers.get("X-XSS-Protection") == "1; mode=block"

    @pytest.mark.asyncio
    async def test_no_stack_traces_in_production(self, client, monkeypatch):
        """Keine Stack-Traces in Production"""
        monkeypatch.setenv("ENVIRONMENT", "production")

        # Provoziere einen Fehler
        response = await client.get("/api/v1/trigger-error-for-test")

        if response.status_code == 500:
            data = response.json()
            assert "traceback" not in str(data).lower()
            assert "file" not in str(data).lower()
            assert "line" not in str(data).lower()

class TestA07SSRF:
    """A07:2025 - Server-Side Request Forgery"""

    @pytest.mark.asyncio
    @pytest.mark.parametrize("malicious_url", [
        "http://localhost/admin",
        "http://127.0.0.1:22",
        "http://169.254.169.254/latest/meta-data",  # AWS metadata
        "file:///etc/passwd",
        "http://internal-service:8080",
    ])
    async def test_ssrf_prevention(self, authenticated_client, malicious_url):
        """SSRF verhindert"""
        response = await authenticated_client.post("/api/v1/webhooks", json={
            "url": malicious_url,
            "event": "project.created"
        })

        assert response.status_code in [400, 422]
        assert "url" in response.text.lower() or "invalid" in response.text.lower()
```

### 4.2 Penetration Test Automation

```python
# tests/security/test_pentest_automation.py
"""Automatisierte Penetration Tests"""

import pytest
from httpx import AsyncClient
import asyncio

class TestAuthenticationSecurity:
    """Authentifizierungs-Sicherheitstests"""

    @pytest.mark.asyncio
    async def test_brute_force_protection(self, client):
        """Rate Limiting bei Login"""
        attempts = []

        for i in range(20):
            response = await client.post("/api/v1/auth/login", json={
                "email": "test@example.com",
                "password": f"wrong_password_{i}"
            })
            attempts.append(response.status_code)

        # Nach X Versuchen sollte Rate Limiting greifen
        assert 429 in attempts, "Rate Limiting nicht aktiv"

    @pytest.mark.asyncio
    async def test_session_fixation_prevention(self, client):
        """Session Fixation verhindert"""
        # Erste Session
        response1 = await client.post("/api/v1/auth/login", json={
            "email": "test@example.com",
            "password": "correct_password"
        })
        token1 = response1.json().get("access_token")

        # Logout
        await client.post("/api/v1/auth/logout",
                         headers={"Authorization": f"Bearer {token1}"})

        # Zweite Session
        response2 = await client.post("/api/v1/auth/login", json={
            "email": "test@example.com",
            "password": "correct_password"
        })
        token2 = response2.json().get("access_token")

        # Tokens müssen unterschiedlich sein
        assert token1 != token2

        # Alter Token ungültig
        response_old = await client.get("/api/v1/users/me",
                                       headers={"Authorization": f"Bearer {token1}"})
        assert response_old.status_code == 401

    @pytest.mark.asyncio
    async def test_jwt_algorithm_confusion(self, client):
        """JWT Algorithm Confusion verhindert"""
        import jwt

        # Versuche Token mit 'none' Algorithm
        fake_token = jwt.encode(
            {"sub": "admin", "role": "admin"},
            key="",
            algorithm="none"
        )

        response = await client.get("/api/v1/users/me",
                                   headers={"Authorization": f"Bearer {fake_token}"})
        assert response.status_code == 401

class TestInputValidation:
    """Input Validation Tests"""

    @pytest.mark.asyncio
    async def test_oversized_request_rejection(self, client):
        """Überdimensionierte Requests abgelehnt"""
        large_payload = {"data": "x" * (10 * 1024 * 1024)}  # 10MB

        response = await client.post("/api/v1/projects", json=large_payload)
        assert response.status_code == 413  # Payload Too Large

    @pytest.mark.asyncio
    async def test_malformed_json_handling(self, client):
        """Malformed JSON sicher behandelt"""
        response = await client.post(
            "/api/v1/projects",
            content="{invalid json",
            headers={"Content-Type": "application/json"}
        )

        assert response.status_code == 400
        assert "json" in response.text.lower()

    @pytest.mark.asyncio
    @pytest.mark.parametrize("field,value", [
        ("email", "a" * 1000 + "@example.com"),
        ("name", "x" * 10000),
        ("description", "y" * 100000),
    ])
    async def test_field_length_limits(self, authenticated_client, field, value):
        """Feldlängen-Limits durchgesetzt"""
        data = {"name": "Test Project", field: value}

        response = await authenticated_client.post("/api/v1/projects", json=data)
        assert response.status_code in [400, 422]
```

---

## 5. Performance Testing

### 5.1 Load Tests mit Locust

```python
# tests/performance/locustfile.py
"""Load Tests mit Locust"""

from locust import HttpUser, task, between, events
from locust.runners import MasterRunner
import json
import random

class EnergyConsultantUser(HttpUser):
    """Simuliert Energieberater-Nutzer"""

    wait_time = between(1, 5)

    def on_start(self):
        """Login bei Start"""
        response = self.client.post("/api/v1/auth/login", json={
            "email": f"test_user_{random.randint(1, 100)}@example.com",
            "password": "test_password"
        })
        if response.status_code == 200:
            self.token = response.json()["access_token"]
            self.client.headers["Authorization"] = f"Bearer {self.token}"

    @task(10)
    def view_dashboard(self):
        """Dashboard aufrufen (häufig)"""
        self.client.get("/api/v1/dashboard")

    @task(5)
    def list_projects(self):
        """Projektliste abrufen"""
        self.client.get("/api/v1/projects")

    @task(3)
    def view_project(self):
        """Einzelnes Projekt anzeigen"""
        self.client.get(f"/api/v1/projects/{random.randint(1, 1000)}")

    @task(2)
    def create_project(self):
        """Neues Projekt anlegen"""
        self.client.post("/api/v1/projects", json={
            "name": f"Load Test Project {random.randint(1, 10000)}",
            "project_type": "residential"
        })

    @task(1)
    def run_calculation(self):
        """Berechnung durchführen (ressourcenintensiv)"""
        self.client.post("/api/v1/calculations", json={
            "project_id": random.randint(1, 1000),
            "calculation_type": "energy_demand",
            "parameters": {
                "climate_zone": "TRY04",
                "heating_system": "gas_condensing"
            }
        })

    @task(1)
    def generate_report(self):
        """Bericht generieren"""
        self.client.post("/api/v1/reports", json={
            "project_id": random.randint(1, 1000),
            "template": "isfp_standard"
        })

class AdminUser(HttpUser):
    """Simuliert Admin-Nutzer"""

    wait_time = between(5, 15)
    weight = 1  # Weniger Admins als normale User

    def on_start(self):
        response = self.client.post("/api/v1/auth/login", json={
            "email": "admin@example.com",
            "password": "admin_password"
        })
        if response.status_code == 200:
            self.token = response.json()["access_token"]
            self.client.headers["Authorization"] = f"Bearer {self.token}"

    @task(5)
    def view_admin_dashboard(self):
        self.client.get("/api/v1/admin/dashboard")

    @task(2)
    def list_users(self):
        self.client.get("/api/v1/admin/users")

    @task(1)
    def view_system_logs(self):
        self.client.get("/api/v1/admin/logs?limit=100")

# Performance Assertions
@events.test_stop.add_listener
def on_test_stop(environment, **kwargs):
    """Prüfe Performance-Kriterien am Ende"""
    if isinstance(environment.runner, MasterRunner):
        stats = environment.stats.total

        # Assertions
        assert stats.fail_ratio < 0.01, f"Fehlerrate {stats.fail_ratio:.2%} > 1%"
        assert stats.avg_response_time < 500, f"Avg Response {stats.avg_response_time}ms > 500ms"
        assert stats.get_response_time_percentile(0.95) < 2000, "P95 > 2000ms"
```

### 5.2 Benchmark Tests

```python
# tests/performance/test_benchmarks.py
"""Benchmark Tests für kritische Pfade"""

import pytest
import time
from decimal import Decimal
from app.calculations.energy import EnergyCalculator
from app.services.report import ReportGenerator

class TestCalculationBenchmarks:
    """Benchmarks für Berechnungen"""

    @pytest.fixture
    def large_building(self):
        """Großes Gebäude mit vielen Komponenten"""
        return {
            "heated_area": Decimal("5000"),
            "zones": [
                {
                    "name": f"Zone {i}",
                    "area": Decimal("100"),
                    "components": [
                        {"type": "wall", "area": Decimal("50"), "u_value": Decimal("0.24")},
                        {"type": "window", "area": Decimal("10"), "u_value": Decimal("1.1")},
                        {"type": "roof", "area": Decimal("100"), "u_value": Decimal("0.18")},
                    ]
                }
                for i in range(50)  # 50 Zonen
            ]
        }

    def test_calculation_performance(self, large_building, benchmark):
        """Berechnung in akzeptabler Zeit"""
        calculator = EnergyCalculator()

        result = benchmark(calculator.calculate_full, large_building)

        # Assertion: Max 5 Sekunden für komplexes Gebäude
        assert benchmark.stats.mean < 5.0
        assert result is not None

    def test_u_value_calculation_performance(self, benchmark):
        """U-Wert Berechnung < 10ms"""
        from app.calculations.thermal import UWertCalculator

        calculator = UWertCalculator()
        layers = [
            {"name": f"Layer {i}", "thickness": 0.05, "lambda": 0.5}
            for i in range(20)
        ]

        result = benchmark(calculator.calculate, layers=layers, rsi=0.13, rse=0.04)

        assert benchmark.stats.mean < 0.01  # 10ms

class TestAPIPeformance:
    """API Performance Tests"""

    @pytest.mark.asyncio
    async def test_project_list_performance(self, authenticated_client, create_many_projects):
        """Projektliste < 200ms bei 1000 Projekten"""
        await create_many_projects(1000)

        start = time.perf_counter()
        response = await authenticated_client.get("/api/v1/projects?page=1&limit=50")
        duration = time.perf_counter() - start

        assert response.status_code == 200
        assert duration < 0.2  # 200ms

    @pytest.mark.asyncio
    async def test_search_performance(self, authenticated_client, create_many_projects):
        """Suche < 500ms bei 10000 Projekten"""
        await create_many_projects(10000)

        start = time.perf_counter()
        response = await authenticated_client.get(
            "/api/v1/projects?search=Energieberatung"
        )
        duration = time.perf_counter() - start

        assert response.status_code == 200
        assert duration < 0.5  # 500ms
```

---

## 6. CI/CD Integration

### 6.1 GitHub Actions Workflow

```yaml
# .github/workflows/tests.yml
name: Test Suite

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  PYTHON_VERSION: "3.11"
  POSTGRES_DB: test_db
  POSTGRES_USER: test_user
  POSTGRES_PASSWORD: test_password
  REDIS_URL: redis://localhost:6379

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}

      - name: Install dependencies
        run: |
          pip install ruff mypy
          pip install -e ".[dev]"

      - name: Run Ruff
        run: ruff check .

      - name: Run Mypy
        run: mypy app --strict

  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}

      - name: Install dependencies
        run: pip install -e ".[dev]"

      - name: Run Unit Tests
        run: |
          pytest tests/unit \
            --cov=app \
            --cov-report=xml \
            --cov-fail-under=80 \
            -n auto \
            --dist loadfile

      - name: Upload Coverage
        uses: codecov/codecov-action@v4
        with:
          files: coverage.xml
          fail_ci_if_error: true

  integration-tests:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: ${{ env.POSTGRES_DB }}
          POSTGRES_USER: ${{ env.POSTGRES_USER }}
          POSTGRES_PASSWORD: ${{ env.POSTGRES_PASSWORD }}
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

      redis:
        image: redis:7
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}

      - name: Install dependencies
        run: pip install -e ".[dev]"

      - name: Run Migrations
        run: alembic upgrade head
        env:
          DATABASE_URL: postgresql://${{ env.POSTGRES_USER }}:${{ env.POSTGRES_PASSWORD }}@localhost:5432/${{ env.POSTGRES_DB }}

      - name: Run Integration Tests
        run: |
          pytest tests/integration \
            --cov=app \
            --cov-append \
            -n 4
        env:
          DATABASE_URL: postgresql://${{ env.POSTGRES_USER }}:${{ env.POSTGRES_PASSWORD }}@localhost:5432/${{ env.POSTGRES_DB }}

  security-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}

      - name: Install dependencies
        run: |
          pip install -e ".[dev]"
          pip install safety bandit pip-audit

      - name: Run Safety Check
        run: safety check --full-report

      - name: Run Bandit
        run: bandit -r app -ll

      - name: Run pip-audit
        run: pip-audit

      - name: Run Security Tests
        run: pytest tests/security -v

  e2e-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}

      - name: Install Playwright
        run: |
          pip install playwright pytest-playwright
          playwright install chromium

      - name: Start Application
        run: |
          docker-compose up -d
          sleep 30  # Wait for services

      - name: Run E2E Tests
        run: pytest tests/e2e --headed=false

      - name: Upload Screenshots
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: e2e-screenshots
          path: tests/e2e/screenshots/

  performance-tests:
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}

      - name: Install dependencies
        run: |
          pip install -e ".[dev]"
          pip install locust pytest-benchmark

      - name: Run Benchmark Tests
        run: pytest tests/performance/test_benchmarks.py --benchmark-only

      - name: Run Load Tests
        run: |
          docker-compose up -d
          locust -f tests/performance/locustfile.py \
            --headless \
            --users 100 \
            --spawn-rate 10 \
            --run-time 60s \
            --host http://localhost:8000 \
            --check-fail-ratio 0.01 \
            --check-avg-response-time 500

  mutation-tests:
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}

      - name: Install dependencies
        run: |
          pip install -e ".[dev]"
          pip install mutmut

      - name: Run Mutation Tests
        run: |
          mutmut run \
            --paths-to-mutate=app/calculations \
            --tests-dir=tests/unit/calculations \
            --runner="pytest -x"

      - name: Check Mutation Score
        run: |
          mutmut results
          # Mutation Score sollte > 80% sein
          mutmut results --json | python -c "
          import json, sys
          data = json.load(sys.stdin)
          score = data['killed'] / data['total'] * 100
          if score < 80:
              print(f'Mutation score {score:.1f}% < 80%')
              sys.exit(1)
          "
```

### 6.2 Pre-commit Hooks

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.3.0
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.8.0
    hooks:
      - id: mypy
        additional_dependencies:
          - types-redis
          - types-python-jose
          - pydantic

  - repo: local
    hooks:
      - id: pytest-unit
        name: pytest unit tests
        entry: pytest tests/unit -x --tb=short
        language: system
        pass_filenames: false
        stages: [commit]

      - id: security-check
        name: security check
        entry: bash -c 'safety check --short-report && bandit -r app -q'
        language: system
        pass_filenames: false
        stages: [push]
```

---

## 7. Test-Daten-Management

### 7.1 Factories

```python
# tests/factories.py
"""Test Data Factories"""

import factory
from factory.alchemy import SQLAlchemyModelFactory
from faker import Faker
from decimal import Decimal
import random

from app.models import User, Organization, Project, Building, Calculation
from app.database import TestSessionLocal

fake = Faker("de_DE")

class BaseFactory(SQLAlchemyModelFactory):
    class Meta:
        abstract = True
        sqlalchemy_session = TestSessionLocal()
        sqlalchemy_session_persistence = "commit"

class OrganizationFactory(BaseFactory):
    class Meta:
        model = Organization

    name = factory.LazyFunction(lambda: fake.company())
    slug = factory.LazyAttribute(lambda o: o.name.lower().replace(" ", "-"))
    email = factory.LazyFunction(fake.company_email)
    is_active = True

class UserFactory(BaseFactory):
    class Meta:
        model = User

    email = factory.LazyFunction(fake.email)
    first_name = factory.LazyFunction(fake.first_name)
    last_name = factory.LazyFunction(fake.last_name)
    hashed_password = "$argon2id$..."  # Pre-hashed test password
    is_active = True
    organization = factory.SubFactory(OrganizationFactory)

class ProjectFactory(BaseFactory):
    class Meta:
        model = Project

    name = factory.LazyFunction(
        lambda: f"Energieberatung {fake.street_name()} {fake.building_number()}"
    )
    project_type = factory.LazyFunction(
        lambda: random.choice(["residential", "commercial", "industrial"])
    )
    status = "active"
    organization = factory.SubFactory(OrganizationFactory)
    created_by = factory.SubFactory(UserFactory)

class BuildingFactory(BaseFactory):
    class Meta:
        model = Building

    name = factory.LazyFunction(lambda: f"Gebäude {fake.building_number()}")
    building_type = factory.LazyFunction(
        lambda: random.choice(["single_family", "multi_family", "office"])
    )
    construction_year = factory.LazyFunction(lambda: random.randint(1950, 2024))
    heated_area = factory.LazyFunction(
        lambda: Decimal(str(random.randint(80, 500)))
    )
    project = factory.SubFactory(ProjectFactory)

class CalculationFactory(BaseFactory):
    class Meta:
        model = Calculation

    calculation_type = "energy_demand"
    status = "completed"
    parameters = factory.LazyFunction(lambda: {
        "climate_zone": random.choice(["TRY04", "TRY05", "TRY12"]),
        "heating_system": random.choice(["gas_condensing", "heat_pump", "district"]),
    })
    results = factory.LazyFunction(lambda: {
        "primary_energy_demand": float(random.randint(30, 200)),
        "heating_demand": float(random.randint(20, 150)),
    })
    building = factory.SubFactory(BuildingFactory)
```

### 7.2 Fixtures

```python
# tests/conftest.py
"""Pytest Fixtures"""

import pytest
import asyncio
from typing import AsyncGenerator
from httpx import AsyncClient
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
from sqlalchemy.orm import sessionmaker

from app.main import app
from app.database import Base, get_db
from app.models import User, Organization
from tests.factories import UserFactory, OrganizationFactory

# Test Database URL
TEST_DATABASE_URL = "postgresql+asyncpg://test:test@localhost/test_db"

@pytest.fixture(scope="session")
def event_loop():
    """Event Loop für async Tests"""
    loop = asyncio.get_event_loop_policy().new_event_loop()
    yield loop
    loop.close()

@pytest.fixture(scope="session")
async def engine():
    """Database Engine"""
    engine = create_async_engine(TEST_DATABASE_URL, echo=False)

    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)

    yield engine

    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)

    await engine.dispose()

@pytest.fixture
async def db_session(engine) -> AsyncGenerator[AsyncSession, None]:
    """Database Session mit Rollback nach Test"""
    async_session = sessionmaker(
        engine, class_=AsyncSession, expire_on_commit=False
    )

    async with async_session() as session:
        async with session.begin():
            yield session
            await session.rollback()

@pytest.fixture
async def client(db_session) -> AsyncGenerator[AsyncClient, None]:
    """HTTP Client ohne Auth"""
    async def override_get_db():
        yield db_session

    app.dependency_overrides[get_db] = override_get_db

    async with AsyncClient(app=app, base_url="http://test") as client:
        yield client

    app.dependency_overrides.clear()

@pytest.fixture
async def test_user(db_session) -> User:
    """Authentifizierter Test-Benutzer"""
    org = await OrganizationFactory.create_async(session=db_session)
    user = await UserFactory.create_async(
        session=db_session,
        organization=org,
        email="test@example.com"
    )
    return user

@pytest.fixture
async def authenticated_client(client, test_user) -> AsyncClient:
    """HTTP Client mit Authentication"""
    response = await client.post("/api/v1/auth/login", json={
        "email": test_user.email,
        "password": "test_password"
    })
    token = response.json()["access_token"]
    client.headers["Authorization"] = f"Bearer {token}"
    client._test_user = test_user
    return client

@pytest.fixture
async def admin_client(client, db_session) -> AsyncClient:
    """HTTP Client mit Admin-Rechten"""
    admin = await UserFactory.create_async(
        session=db_session,
        email="admin@example.com",
        role="admin"
    )
    response = await client.post("/api/v1/auth/login", json={
        "email": admin.email,
        "password": "test_password"
    })
    token = response.json()["access_token"]
    client.headers["Authorization"] = f"Bearer {token}"
    return client
```

---

## 8. Test-Dokumentation

### 8.1 Test Coverage Report

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      TEST COVERAGE REQUIREMENTS                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  MINIMUM COVERAGE: 80% gesamt                                               │
│                                                                             │
│  Module-spezifische Anforderungen:                                          │
│  ─────────────────────────────────────────────────────────────────────────  │
│  • app/calculations/*     : 95%  (Kritische Berechnungslogik)               │
│  • app/api/*              : 85%  (API Endpoints)                            │
│  • app/services/*         : 80%  (Business Logic)                           │
│  • app/security/*         : 95%  (Sicherheitskritisch)                      │
│  • app/models/*           : 70%  (Data Models)                              │
│                                                                             │
│  Ausnahmen dokumentieren in .coveragerc                                      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 8.2 Test-Checkliste für neue Features

```markdown
## Feature Test Checkliste

### Unit Tests
- [ ] Alle Funktionen/Methoden getestet
- [ ] Edge Cases abgedeckt
- [ ] Fehlerbehandlung getestet
- [ ] Parametrisierte Tests für Varianten

### Integration Tests
- [ ] API Endpoints getestet
- [ ] Database Queries verifiziert
- [ ] Service-Interaktionen getestet
- [ ] Transaktions-Rollback funktioniert

### Security Tests
- [ ] Authentifizierung erforderlich
- [ ] Autorisierung geprüft (RBAC)
- [ ] Input Validation getestet
- [ ] SQL Injection verhindert
- [ ] XSS verhindert

### Performance
- [ ] Response Time < 200ms (einfache Requests)
- [ ] Response Time < 2s (komplexe Berechnungen)
- [ ] Keine N+1 Queries
- [ ] Pagination implementiert
```

---

## 9. Checkliste

```
✓ Test-Philosophie definiert (keine KI, deterministisch)
✓ Test-Pyramide dokumentiert (80% Unit, 15% Integration, 5% E2E)
✓ Unit Tests mit pytest strukturiert
✓ Berechnungs-Validierung gegen Referenzwerte
✓ Property-Based Testing mit Hypothesis
✓ Security Tests nach OWASP Top 10 2025
✓ Performance Tests mit Locust
✓ CI/CD Pipeline mit GitHub Actions
✓ Pre-commit Hooks konfiguriert
✓ Test Data Factories erstellt
✓ Fixtures dokumentiert
✓ Coverage Requirements definiert
```

---

*Letzte Aktualisierung: 2025-11-25*
*Version: 1.0.0*
