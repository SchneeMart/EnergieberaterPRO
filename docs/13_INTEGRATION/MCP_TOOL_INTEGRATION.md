# MCP und Tool-Integration

## Referenzen
- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [MCP SDK](https://github.com/modelcontextprotocol/sdk)
- [OpenAI Function Calling](https://platform.openai.com/docs/guides/function-calling)
- [Anthropic Tool Use](https://docs.anthropic.com/claude/docs/tool-use)

---

## 1. MCP-Übersicht

### 1.1 Model Context Protocol

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    MODEL CONTEXT PROTOCOL (MCP)                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  MCP ist ein offener Standard für die Kommunikation zwischen                │
│  KI-Modellen und externen Tools/Datenquellen.                               │
│                                                                             │
│  ┌─────────────┐       MCP Protocol       ┌─────────────┐                  │
│  │             │◄────────────────────────►│             │                  │
│  │  KI-Modell  │                          │  MCP Server │                  │
│  │  (Client)   │                          │  (Tools)    │                  │
│  │             │                          │             │                  │
│  └─────────────┘                          └─────────────┘                  │
│                                                                             │
│  KERNKONZEPTE:                                                              │
│  ─────────────────────────────────────────────────────────────────────────  │
│                                                                             │
│  1. TOOLS                                                                   │
│     Funktionen die das Modell aufrufen kann                                │
│     z.B. calculate_u_value, search_projects, generate_report               │
│                                                                             │
│  2. RESOURCES                                                               │
│     Daten die dem Modell zur Verfügung stehen                              │
│     z.B. Projektdaten, Normwerte, Kundendaten                              │
│                                                                             │
│  3. PROMPTS                                                                 │
│     Vordefinierte Prompt-Templates                                         │
│     z.B. Berichtstexte, E-Mail-Vorlagen                                    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Architektur in EnergieberaterPRO

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    MCP INTEGRATION ARCHITEKTUR                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                        FRONTEND                                       │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                   │  │
│  │  │  Chat UI    │  │  Tool UI    │  │  Result UI  │                   │  │
│  │  │             │  │  (Trigger)  │  │  (Display)  │                   │  │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘                   │  │
│  │         └────────────────┼────────────────┘                          │  │
│  │                          │                                           │  │
│  └──────────────────────────┼───────────────────────────────────────────┘  │
│                             │ REST/WebSocket                               │
│  ┌──────────────────────────┼───────────────────────────────────────────┐  │
│  │                     AI GATEWAY                                        │  │
│  │  ┌─────────────────────────────────────────────────────────────────┐ │  │
│  │  │                   MCP Router                                     │ │  │
│  │  │  • Tool Discovery                                                │ │  │
│  │  │  • Request Routing                                               │ │  │
│  │  │  • Response Aggregation                                          │ │  │
│  │  └───────────────────────┬─────────────────────────────────────────┘ │  │
│  │                          │                                           │  │
│  │  ┌───────────────────────┼───────────────────────────────────────┐   │  │
│  │  │                       │                                       │   │  │
│  │  ▼                       ▼                                       ▼   │  │
│  │  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐       │  │
│  │  │Calc Tools│    │Data Tools│    │Report    │    │External  │       │  │
│  │  │          │    │          │    │Tools     │    │Tools     │       │  │
│  │  │• U-Wert  │    │• Projects│    │• Generate│    │• Weather │       │  │
│  │  │• Heizlast│    │• Buildings│   │• Format  │    │• Geodata │       │  │
│  │  │• Energie │    │• Customers│   │• Export  │    │• Funding │       │  │
│  │  └──────────┘    └──────────┘    └──────────┘    └──────────┘       │  │
│  │                                                                      │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                     LLM PROVIDERS                                     │  │
│  │  ┌──────────┐    ┌──────────┐    ┌──────────┐                        │  │
│  │  │  Ollama  │    │  Claude  │    │  OpenAI  │                        │  │
│  │  │  (Local) │    │  (API)   │    │  (API)   │                        │  │
│  │  └──────────┘    └──────────┘    └──────────┘                        │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. MCP Server Implementation

### 2.1 MCP Server Kern

```python
# app/mcp/server.py
"""MCP Server für EnergieberaterPRO"""

import json
from typing import Dict, List, Any, Optional, Callable
from dataclasses import dataclass, field
from enum import Enum
import asyncio

class ToolType(Enum):
    CALCULATION = "calculation"
    DATA = "data"
    REPORT = "report"
    EXTERNAL = "external"
    UTILITY = "utility"

@dataclass
class Tool:
    """MCP Tool Definition"""
    name: str
    description: str
    tool_type: ToolType
    parameters: Dict[str, Any]  # JSON Schema
    handler: Callable
    requires_auth: bool = True
    rate_limit: Optional[int] = None  # Calls per minute

@dataclass
class Resource:
    """MCP Resource Definition"""
    uri: str
    name: str
    description: str
    mime_type: str = "application/json"

@dataclass
class PromptTemplate:
    """MCP Prompt Template"""
    name: str
    description: str
    template: str
    arguments: List[Dict[str, Any]]

class MCPServer:
    """Model Context Protocol Server"""

    def __init__(self, session, config):
        self.session = session
        self.config = config
        self.tools: Dict[str, Tool] = {}
        self.resources: Dict[str, Resource] = {}
        self.prompts: Dict[str, PromptTemplate] = {}

        self._register_default_tools()
        self._register_default_resources()
        self._register_default_prompts()

    # ========================================================================
    # Tool Registration
    # ========================================================================

    def register_tool(self, tool: Tool):
        """Registriert ein Tool"""
        self.tools[tool.name] = tool

    def _register_default_tools(self):
        """Registriert Standard-Tools"""

        # Berechnungs-Tools
        self.register_tool(Tool(
            name="calculate_u_value",
            description="Berechnet den U-Wert eines Bauteils nach DIN EN ISO 6946",
            tool_type=ToolType.CALCULATION,
            parameters={
                "type": "object",
                "properties": {
                    "layers": {
                        "type": "array",
                        "description": "Schichtaufbau des Bauteils",
                        "items": {
                            "type": "object",
                            "properties": {
                                "name": {"type": "string"},
                                "thickness": {"type": "number", "description": "Dicke in Metern"},
                                "lambda": {"type": "number", "description": "Wärmeleitfähigkeit W/(mK)"}
                            },
                            "required": ["thickness", "lambda"]
                        }
                    },
                    "rsi": {"type": "number", "default": 0.13},
                    "rse": {"type": "number", "default": 0.04}
                },
                "required": ["layers"]
            },
            handler=self._calculate_u_value
        ))

        self.register_tool(Tool(
            name="calculate_heating_load",
            description="Berechnet die Heizlast nach DIN EN 12831",
            tool_type=ToolType.CALCULATION,
            parameters={
                "type": "object",
                "properties": {
                    "building_id": {"type": "string", "description": "Gebäude-ID"},
                    "zone_id": {"type": "string", "description": "Zonen-ID (optional)"},
                    "climate_zone": {"type": "string", "description": "Klimazone (z.B. TRY04)"}
                },
                "required": ["building_id"]
            },
            handler=self._calculate_heating_load
        ))

        self.register_tool(Tool(
            name="calculate_energy_balance",
            description="Berechnet die Energiebilanz eines Gebäudes",
            tool_type=ToolType.CALCULATION,
            parameters={
                "type": "object",
                "properties": {
                    "building_id": {"type": "string"},
                    "calculation_standard": {
                        "type": "string",
                        "enum": ["DIN_V_18599", "DIN_4108", "simplified"]
                    }
                },
                "required": ["building_id"]
            },
            handler=self._calculate_energy_balance
        ))

        # Daten-Tools
        self.register_tool(Tool(
            name="search_projects",
            description="Sucht Projekte nach verschiedenen Kriterien",
            tool_type=ToolType.DATA,
            parameters={
                "type": "object",
                "properties": {
                    "query": {"type": "string", "description": "Suchbegriff"},
                    "status": {"type": "string", "enum": ["active", "completed", "archived"]},
                    "project_type": {"type": "string"},
                    "limit": {"type": "integer", "default": 10}
                }
            },
            handler=self._search_projects
        ))

        self.register_tool(Tool(
            name="get_building_data",
            description="Lädt Gebäudedaten inkl. Bauteile und Anlagentechnik",
            tool_type=ToolType.DATA,
            parameters={
                "type": "object",
                "properties": {
                    "building_id": {"type": "string"},
                    "include_components": {"type": "boolean", "default": True},
                    "include_systems": {"type": "boolean", "default": True}
                },
                "required": ["building_id"]
            },
            handler=self._get_building_data
        ))

        self.register_tool(Tool(
            name="get_material_properties",
            description="Gibt Materialeigenschaften aus Datenbank zurück",
            tool_type=ToolType.DATA,
            parameters={
                "type": "object",
                "properties": {
                    "material_name": {"type": "string"},
                    "category": {"type": "string"},
                    "property": {"type": "string", "description": "Spezifische Eigenschaft (z.B. lambda, density)"}
                }
            },
            handler=self._get_material_properties
        ))

        # Report-Tools
        self.register_tool(Tool(
            name="generate_report_section",
            description="Generiert einen Berichtsabschnitt",
            tool_type=ToolType.REPORT,
            parameters={
                "type": "object",
                "properties": {
                    "project_id": {"type": "string"},
                    "section_type": {
                        "type": "string",
                        "enum": ["summary", "building_description", "energy_analysis",
                                "recommendations", "economic_analysis", "conclusion"]
                    },
                    "style": {"type": "string", "enum": ["formal", "simplified", "technical"]}
                },
                "required": ["project_id", "section_type"]
            },
            handler=self._generate_report_section
        ))

        self.register_tool(Tool(
            name="format_calculation_result",
            description="Formatiert Berechnungsergebnisse für Bericht",
            tool_type=ToolType.REPORT,
            parameters={
                "type": "object",
                "properties": {
                    "calculation_id": {"type": "string"},
                    "format": {"type": "string", "enum": ["table", "text", "chart_data"]}
                },
                "required": ["calculation_id"]
            },
            handler=self._format_calculation_result
        ))

        # Externe Tools
        self.register_tool(Tool(
            name="get_weather_data",
            description="Lädt Wetterdaten für Standort",
            tool_type=ToolType.EXTERNAL,
            parameters={
                "type": "object",
                "properties": {
                    "location": {"type": "string", "description": "PLZ oder Stadtname"},
                    "data_type": {"type": "string", "enum": ["current", "climate", "degree_days"]}
                },
                "required": ["location"]
            },
            handler=self._get_weather_data,
            rate_limit=60
        ))

        self.register_tool(Tool(
            name="search_funding_programs",
            description="Sucht passende Förderprogramme",
            tool_type=ToolType.EXTERNAL,
            parameters={
                "type": "object",
                "properties": {
                    "measure_type": {"type": "string", "description": "Art der Maßnahme"},
                    "building_type": {"type": "string"},
                    "state": {"type": "string", "description": "Bundesland"}
                },
                "required": ["measure_type"]
            },
            handler=self._search_funding_programs,
            rate_limit=30
        ))

    # ========================================================================
    # Tool Handlers
    # ========================================================================

    async def _calculate_u_value(self, params: Dict, context: Dict) -> Dict:
        """U-Wert Berechnung"""
        from app.calculations.thermal import UWertCalculator

        calculator = UWertCalculator()
        result = calculator.calculate(
            layers=params["layers"],
            rsi=params.get("rsi", 0.13),
            rse=params.get("rse", 0.04)
        )

        return {
            "u_value": float(result.u_value),
            "r_total": float(result.r_total),
            "layer_resistances": [
                {"name": l["name"], "r": float(r)}
                for l, r in zip(params["layers"], result.layer_resistances)
            ],
            "unit": "W/(m²K)",
            "standard": "DIN EN ISO 6946"
        }

    async def _calculate_heating_load(self, params: Dict, context: Dict) -> Dict:
        """Heizlast Berechnung"""
        from app.calculations.heating import HeizlastCalculator

        # Gebäudedaten laden
        building = await self._load_building(params["building_id"], context)

        calculator = HeizlastCalculator()
        result = calculator.calculate(building, params.get("climate_zone", "TRY04"))

        return {
            "total_heating_load": float(result.total),
            "transmission_loss": float(result.transmission_loss),
            "ventilation_loss": float(result.ventilation_loss),
            "specific_heating_load": float(result.specific),
            "zones": [
                {"name": z.name, "load": float(z.load)}
                for z in result.zones
            ],
            "unit": "W",
            "standard": "DIN EN 12831"
        }

    async def _search_projects(self, params: Dict, context: Dict) -> Dict:
        """Projektsuche"""
        org_id = context.get("organization_id")

        query = """
            SELECT id, name, project_type, status, created_at
            FROM projects
            WHERE organization_id = :org_id
        """
        query_params = {"org_id": org_id}

        if params.get("query"):
            query += " AND name ILIKE :query"
            query_params["query"] = f"%{params['query']}%"

        if params.get("status"):
            query += " AND status = :status"
            query_params["status"] = params["status"]

        query += " ORDER BY created_at DESC LIMIT :limit"
        query_params["limit"] = params.get("limit", 10)

        result = await self.session.execute(query, query_params)
        projects = result.fetchall()

        return {
            "count": len(projects),
            "projects": [
                {
                    "id": str(p.id),
                    "name": p.name,
                    "type": p.project_type,
                    "status": p.status,
                    "created_at": p.created_at.isoformat()
                }
                for p in projects
            ]
        }

    async def _get_building_data(self, params: Dict, context: Dict) -> Dict:
        """Gebäudedaten laden"""
        building_id = params["building_id"]

        # Basis-Daten
        building = await self.session.get(Building, building_id)
        if not building:
            return {"error": "Gebäude nicht gefunden"}

        result = {
            "id": str(building.id),
            "name": building.name,
            "type": building.building_type,
            "construction_year": building.construction_year,
            "heated_area": float(building.heated_area),
            "floors": building.floors,
            "climate_zone": building.climate_zone
        }

        if params.get("include_components", True):
            components = await self._load_components(building_id)
            result["components"] = components

        if params.get("include_systems", True):
            systems = await self._load_systems(building_id)
            result["systems"] = systems

        return result

    async def _get_material_properties(self, params: Dict, context: Dict) -> Dict:
        """Material-Eigenschaften aus Datenbank"""
        # Aus statischer Materialdatenbank laden
        from app.data.materials import MATERIAL_DATABASE

        material_name = params.get("material_name", "").lower()
        category = params.get("category")

        results = []
        for mat_id, mat_data in MATERIAL_DATABASE.items():
            if material_name in mat_data["name"].lower():
                if category and mat_data.get("category") != category:
                    continue
                results.append(mat_data)

        return {"materials": results, "count": len(results)}

    async def _generate_report_section(self, params: Dict, context: Dict) -> Dict:
        """Berichtsabschnitt generieren"""
        project_id = params["project_id"]
        section_type = params["section_type"]

        # Projektdaten laden
        project_data = await self._load_project_data(project_id, context)

        # Template für Sektion laden
        template = self._get_section_template(section_type, params.get("style", "formal"))

        # Mit Daten füllen
        content = template.format(**project_data)

        return {
            "section_type": section_type,
            "content": content,
            "data_used": list(project_data.keys())
        }

    async def _get_weather_data(self, params: Dict, context: Dict) -> Dict:
        """Wetterdaten abrufen"""
        location = params["location"]
        data_type = params.get("data_type", "climate")

        # Aus Klimadatenbank oder externem Service
        climate_data = await self._fetch_climate_data(location)

        if data_type == "climate":
            return {
                "location": location,
                "design_temperature": climate_data["theta_e"],
                "heating_degree_days": climate_data["hdd"],
                "climate_zone": climate_data["try_zone"],
                "solar_radiation": climate_data["solar_annual"]
            }
        elif data_type == "degree_days":
            return {
                "location": location,
                "heating_degree_days": climate_data["hdd"],
                "cooling_degree_days": climate_data.get("cdd", 0),
                "reference_temperature": 20
            }

        return climate_data

    async def _search_funding_programs(self, params: Dict, context: Dict) -> Dict:
        """Förderprogramme suchen"""
        measure_type = params["measure_type"]
        building_type = params.get("building_type", "residential")

        # Aus Förderdatenbank
        programs = await self._query_funding_database(measure_type, building_type)

        return {
            "count": len(programs),
            "programs": programs,
            "note": "Stand: November 2025. Aktualität prüfen!"
        }

    # ========================================================================
    # MCP Protocol Methods
    # ========================================================================

    def get_capabilities(self) -> Dict:
        """Gibt Server-Capabilities zurück"""
        return {
            "capabilities": {
                "tools": {"listChanged": True},
                "resources": {"subscribe": True, "listChanged": True},
                "prompts": {"listChanged": True}
            },
            "serverInfo": {
                "name": "energieberater-pro",
                "version": "1.0.0"
            }
        }

    def list_tools(self) -> List[Dict]:
        """Listet alle verfügbaren Tools"""
        return [
            {
                "name": tool.name,
                "description": tool.description,
                "inputSchema": tool.parameters
            }
            for tool in self.tools.values()
        ]

    async def call_tool(self, name: str, arguments: Dict, context: Dict) -> Dict:
        """Führt Tool aus"""
        tool = self.tools.get(name)
        if not tool:
            return {"error": f"Tool '{name}' nicht gefunden"}

        # Rate Limiting prüfen
        if tool.rate_limit:
            if not await self._check_rate_limit(name, tool.rate_limit, context):
                return {"error": "Rate limit erreicht"}

        try:
            result = await tool.handler(arguments, context)
            return {"content": [{"type": "text", "text": json.dumps(result, default=str)}]}
        except Exception as e:
            return {"error": str(e), "isError": True}

    def list_resources(self) -> List[Dict]:
        """Listet alle Ressourcen"""
        return [
            {
                "uri": res.uri,
                "name": res.name,
                "description": res.description,
                "mimeType": res.mime_type
            }
            for res in self.resources.values()
        ]

    async def read_resource(self, uri: str, context: Dict) -> Dict:
        """Liest Ressource"""
        # URI parsen: energieberater://projects/{id}
        parts = uri.replace("energieberater://", "").split("/")

        if parts[0] == "projects" and len(parts) > 1:
            project = await self._load_project_data(parts[1], context)
            return {"contents": [{"uri": uri, "mimeType": "application/json", "text": json.dumps(project)}]}

        return {"error": f"Ressource '{uri}' nicht gefunden"}

    def list_prompts(self) -> List[Dict]:
        """Listet Prompt-Templates"""
        return [
            {
                "name": prompt.name,
                "description": prompt.description,
                "arguments": prompt.arguments
            }
            for prompt in self.prompts.values()
        ]

    def get_prompt(self, name: str, arguments: Dict = None) -> Dict:
        """Gibt Prompt-Template zurück"""
        prompt = self.prompts.get(name)
        if not prompt:
            return {"error": f"Prompt '{name}' nicht gefunden"}

        # Template mit Argumenten füllen
        text = prompt.template
        if arguments:
            text = text.format(**arguments)

        return {
            "description": prompt.description,
            "messages": [{"role": "user", "content": {"type": "text", "text": text}}]
        }

    # ========================================================================
    # Resource Registration
    # ========================================================================

    def _register_default_resources(self):
        """Registriert Standard-Ressourcen"""
        self.resources["projects"] = Resource(
            uri="energieberater://projects",
            name="Projekte",
            description="Liste aller Projekte der Organisation"
        )

        self.resources["materials"] = Resource(
            uri="energieberater://materials",
            name="Materialdatenbank",
            description="Baustoff-Eigenschaften und Lambda-Werte"
        )

        self.resources["norms"] = Resource(
            uri="energieberater://norms",
            name="Normen-Referenz",
            description="Relevante DIN/EN Normen und Grenzwerte"
        )

    def _register_default_prompts(self):
        """Registriert Prompt-Templates"""
        self.prompts["analyze_building"] = PromptTemplate(
            name="analyze_building",
            description="Analysiert ein Gebäude und gibt Empfehlungen",
            template="""Analysiere das folgende Gebäude und gib Sanierungsempfehlungen:

Gebäude: {building_name}
Baujahr: {construction_year}
Typ: {building_type}
Beheizte Fläche: {heated_area} m²

Aktuelle Kennwerte:
- Primärenergiebedarf: {primary_energy} kWh/(m²a)
- Heizlast: {heating_load} W
- U-Werte: {u_values}

Bitte analysiere:
1. Schwachstellen der Gebäudehülle
2. Potenzial der Anlagentechnik
3. Priorisierte Maßnahmenempfehlungen
4. Geschätzte Einsparpotenziale""",
            arguments=[
                {"name": "building_name", "required": True},
                {"name": "construction_year", "required": True},
                {"name": "building_type", "required": True},
                {"name": "heated_area", "required": True},
                {"name": "primary_energy", "required": False},
                {"name": "heating_load", "required": False},
                {"name": "u_values", "required": False}
            ]
        )

        self.prompts["write_report_text"] = PromptTemplate(
            name="write_report_text",
            description="Generiert Berichtstext für einen Abschnitt",
            template="""Erstelle einen {style} Berichtstext für den Abschnitt "{section}".

Kontext:
{context}

Anforderungen:
- Sachlich und präzise
- Fachlich korrekt
- {length} Zeichen
- Keine Marketing-Sprache""",
            arguments=[
                {"name": "section", "required": True},
                {"name": "style", "required": False, "default": "formal"},
                {"name": "context", "required": True},
                {"name": "length", "required": False, "default": "500-800"}
            ]
        )
```

### 2.2 MCP HTTP Transport

```python
# app/mcp/transport.py
"""MCP HTTP Transport Layer"""

from fastapi import APIRouter, Request, Response, Depends, HTTPException
from fastapi.responses import JSONResponse, StreamingResponse
import json

from app.mcp.server import MCPServer
from app.dependencies import get_mcp_server, get_current_user

router = APIRouter(prefix="/mcp", tags=["mcp"])

@router.get("/")
async def get_server_info(server: MCPServer = Depends(get_mcp_server)):
    """Server-Informationen und Capabilities"""
    return server.get_capabilities()

@router.get("/tools")
async def list_tools(server: MCPServer = Depends(get_mcp_server)):
    """Liste aller verfügbaren Tools"""
    return {"tools": server.list_tools()}

@router.post("/tools/{tool_name}")
async def call_tool(
    tool_name: str,
    request: Request,
    server: MCPServer = Depends(get_mcp_server),
    user: dict = Depends(get_current_user)
):
    """Tool aufrufen"""
    body = await request.json()
    arguments = body.get("arguments", {})

    context = {
        "user_id": user["id"],
        "organization_id": user["organization_id"],
        "request_id": request.headers.get("X-Request-ID")
    }

    result = await server.call_tool(tool_name, arguments, context)

    if result.get("isError"):
        raise HTTPException(status_code=400, detail=result.get("error"))

    return result

@router.get("/resources")
async def list_resources(server: MCPServer = Depends(get_mcp_server)):
    """Liste aller Ressourcen"""
    return {"resources": server.list_resources()}

@router.get("/resources/{resource_uri:path}")
async def read_resource(
    resource_uri: str,
    server: MCPServer = Depends(get_mcp_server),
    user: dict = Depends(get_current_user)
):
    """Ressource lesen"""
    context = {
        "user_id": user["id"],
        "organization_id": user["organization_id"]
    }

    result = await server.read_resource(f"energieberater://{resource_uri}", context)

    if result.get("error"):
        raise HTTPException(status_code=404, detail=result["error"])

    return result

@router.get("/prompts")
async def list_prompts(server: MCPServer = Depends(get_mcp_server)):
    """Liste aller Prompt-Templates"""
    return {"prompts": server.list_prompts()}

@router.get("/prompts/{prompt_name}")
async def get_prompt(
    prompt_name: str,
    request: Request,
    server: MCPServer = Depends(get_mcp_server)
):
    """Prompt-Template abrufen"""
    arguments = dict(request.query_params)
    result = server.get_prompt(prompt_name, arguments)

    if result.get("error"):
        raise HTTPException(status_code=404, detail=result["error"])

    return result
```

---

## 3. Custom Tool Creation

### 3.1 Tool Builder

```python
# app/mcp/custom_tools.py
"""Custom Tool Creation für Organisationen"""

from typing import Dict, List, Optional
from dataclasses import dataclass
import json

@dataclass
class CustomToolDefinition:
    """Definition eines Custom Tools"""
    id: str
    organization_id: str
    name: str
    description: str
    parameters: Dict  # JSON Schema
    implementation_type: str  # "http", "script", "workflow"
    implementation: Dict  # URL, Script, oder Workflow-ID
    is_active: bool = True
    created_by: str = None

class CustomToolManager:
    """Verwaltet Custom Tools"""

    def __init__(self, session, mcp_server):
        self.session = session
        self.mcp_server = mcp_server

    async def create_tool(
        self,
        organization_id: str,
        definition: Dict,
        created_by: str
    ) -> CustomToolDefinition:
        """Erstellt Custom Tool"""

        # Validierung
        self._validate_definition(definition)

        tool = CustomToolDefinition(
            id=str(uuid.uuid4()),
            organization_id=organization_id,
            name=definition["name"],
            description=definition["description"],
            parameters=definition["parameters"],
            implementation_type=definition["implementation_type"],
            implementation=definition["implementation"],
            created_by=created_by
        )

        # In DB speichern
        await self._save_tool(tool)

        # In MCP Server registrieren
        self._register_in_mcp(tool)

        return tool

    def _validate_definition(self, definition: Dict):
        """Validiert Tool-Definition"""
        required_fields = ["name", "description", "parameters", "implementation_type", "implementation"]
        for field in required_fields:
            if field not in definition:
                raise ValueError(f"Feld '{field}' fehlt")

        # Parameter als JSON Schema validieren
        if not isinstance(definition["parameters"], dict):
            raise ValueError("Parameters muss ein JSON Schema sein")

        # Implementation-Type prüfen
        valid_types = ["http", "script", "workflow"]
        if definition["implementation_type"] not in valid_types:
            raise ValueError(f"implementation_type muss einer von {valid_types} sein")

    def _register_in_mcp(self, tool: CustomToolDefinition):
        """Registriert Tool im MCP Server"""
        from app.mcp.server import Tool, ToolType

        handler = self._create_handler(tool)

        mcp_tool = Tool(
            name=f"custom_{tool.organization_id}_{tool.name}",
            description=tool.description,
            tool_type=ToolType.UTILITY,
            parameters=tool.parameters,
            handler=handler
        )

        self.mcp_server.register_tool(mcp_tool)

    def _create_handler(self, tool: CustomToolDefinition):
        """Erstellt Handler-Funktion für Tool"""

        if tool.implementation_type == "http":
            return self._create_http_handler(tool)
        elif tool.implementation_type == "script":
            return self._create_script_handler(tool)
        elif tool.implementation_type == "workflow":
            return self._create_workflow_handler(tool)

    def _create_http_handler(self, tool: CustomToolDefinition):
        """HTTP-basierter Handler"""
        async def handler(params: Dict, context: Dict) -> Dict:
            import httpx

            url = tool.implementation["url"]
            method = tool.implementation.get("method", "POST")
            headers = tool.implementation.get("headers", {})

            async with httpx.AsyncClient() as client:
                response = await client.request(
                    method=method,
                    url=url,
                    json=params,
                    headers=headers,
                    timeout=30
                )
                return response.json()

        return handler

    def _create_script_handler(self, tool: CustomToolDefinition):
        """Script-basierter Handler (sandboxed)"""
        async def handler(params: Dict, context: Dict) -> Dict:
            # Sandboxed Python execution
            # WICHTIG: Sicherheit beachten!
            script = tool.implementation["script"]

            # Eingeschränkter Namespace
            safe_globals = {
                "params": params,
                "context": context,
                "__builtins__": {
                    "len": len, "str": str, "int": int, "float": float,
                    "list": list, "dict": dict, "sum": sum, "min": min, "max": max,
                    "round": round, "abs": abs, "sorted": sorted
                }
            }

            result = {}
            exec(script, safe_globals, result)
            return result.get("output", {})

        return handler

    def _create_workflow_handler(self, tool: CustomToolDefinition):
        """Workflow-basierter Handler"""
        async def handler(params: Dict, context: Dict) -> Dict:
            workflow_id = tool.implementation["workflow_id"]

            # Workflow starten
            from app.workflows.engine import workflow_engine
            result = await workflow_engine.run_workflow(
                workflow_id,
                inputs=params,
                context=context
            )
            return result

        return handler

    async def list_tools(self, organization_id: str) -> List[CustomToolDefinition]:
        """Listet Custom Tools einer Organisation"""
        result = await self.session.execute("""
            SELECT * FROM custom_tools
            WHERE organization_id = :org_id AND is_active = true
        """, {"org_id": organization_id})

        return [self._row_to_tool(row) for row in result.fetchall()]

    async def delete_tool(self, tool_id: str, organization_id: str):
        """Löscht Custom Tool"""
        await self.session.execute("""
            UPDATE custom_tools
            SET is_active = false
            WHERE id = :id AND organization_id = :org_id
        """, {"id": tool_id, "org_id": organization_id})

        # Aus MCP Server entfernen
        tool = await self._get_tool(tool_id)
        if tool:
            tool_name = f"custom_{tool.organization_id}_{tool.name}"
            del self.mcp_server.tools[tool_name]

        await self.session.commit()
```

---

## 4. Frontend Integration

### 4.1 AI Chat mit Tool Use

```javascript
// static/js/ai-chat.js
/**
 * AI Chat Interface mit Tool Integration
 */

class AIChatInterface {
    constructor(container, options = {}) {
        this.container = container;
        this.options = options;
        this.messages = [];
        this.tools = [];

        this.init();
    }

    async init() {
        // Tools laden
        await this.loadTools();

        // UI rendern
        this.render();

        // Event Listener
        this.setupEventListeners();
    }

    async loadTools() {
        const response = await fetch('/mcp/tools');
        const data = await response.json();
        this.tools = data.tools;
    }

    render() {
        this.container.innerHTML = `
            <div class="ai-chat">
                <div class="chat-messages" id="chat-messages"></div>

                <div class="chat-input-area">
                    <div class="tool-suggestions" id="tool-suggestions"></div>

                    <div class="input-row">
                        <textarea
                            id="chat-input"
                            placeholder="Fragen Sie den KI-Assistenten..."
                            rows="1"
                        ></textarea>
                        <button id="send-button" class="send-btn">
                            <svg><!-- Send Icon --></svg>
                        </button>
                    </div>

                    <div class="quick-actions">
                        <button data-action="analyze-building">Gebäude analysieren</button>
                        <button data-action="calculate-u-value">U-Wert berechnen</button>
                        <button data-action="find-funding">Förderung finden</button>
                    </div>
                </div>
            </div>
        `;
    }

    setupEventListeners() {
        const input = this.container.querySelector('#chat-input');
        const sendBtn = this.container.querySelector('#send-button');
        const quickActions = this.container.querySelectorAll('[data-action]');

        // Enter zum Senden
        input.addEventListener('keydown', (e) => {
            if (e.key === 'Enter' && !e.shiftKey) {
                e.preventDefault();
                this.sendMessage();
            }
        });

        // Send Button
        sendBtn.addEventListener('click', () => this.sendMessage());

        // Quick Actions
        quickActions.forEach(btn => {
            btn.addEventListener('click', () => {
                this.handleQuickAction(btn.dataset.action);
            });
        });

        // Tool-Vorschläge bei @-Mention
        input.addEventListener('input', (e) => {
            this.handleToolMention(e.target.value);
        });
    }

    async sendMessage() {
        const input = this.container.querySelector('#chat-input');
        const message = input.value.trim();

        if (!message) return;

        // User-Nachricht anzeigen
        this.addMessage('user', message);
        input.value = '';

        // An API senden
        try {
            const response = await this.callAI(message);
            await this.processResponse(response);
        } catch (error) {
            this.addMessage('error', 'Fehler bei der Verarbeitung: ' + error.message);
        }
    }

    async callAI(message) {
        const response = await fetch('/api/v1/ai/chat', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${this.getToken()}`
            },
            body: JSON.stringify({
                message,
                context: this.getContext(),
                tools: this.tools.map(t => t.name)
            })
        });

        return response.json();
    }

    async processResponse(response) {
        // Tool Calls verarbeiten
        if (response.tool_calls) {
            for (const toolCall of response.tool_calls) {
                await this.executeToolCall(toolCall);
            }
        }

        // Antwort anzeigen
        if (response.content) {
            this.addMessage('assistant', response.content);
        }
    }

    async executeToolCall(toolCall) {
        // Tool-Ausführung anzeigen
        this.addMessage('tool', `Führe aus: ${toolCall.name}...`, { loading: true });

        try {
            const result = await fetch(`/mcp/tools/${toolCall.name}`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${this.getToken()}`
                },
                body: JSON.stringify({ arguments: toolCall.arguments })
            });

            const data = await result.json();

            // Ergebnis anzeigen
            this.updateLastToolMessage(toolCall.name, data);

            return data;
        } catch (error) {
            this.updateLastToolMessage(toolCall.name, { error: error.message });
            throw error;
        }
    }

    addMessage(type, content, options = {}) {
        const messagesContainer = this.container.querySelector('#chat-messages');

        const messageEl = document.createElement('div');
        messageEl.className = `chat-message ${type}`;
        messageEl.dataset.id = Date.now();

        if (type === 'tool') {
            messageEl.innerHTML = `
                <div class="tool-indicator ${options.loading ? 'loading' : ''}">
                    <svg class="tool-icon"><!-- Tool Icon --></svg>
                    <span>${content}</span>
                    ${options.loading ? '<span class="spinner"></span>' : ''}
                </div>
                <div class="tool-result"></div>
            `;
        } else {
            messageEl.innerHTML = `
                <div class="message-content">${this.formatContent(content)}</div>
            `;
        }

        messagesContainer.appendChild(messageEl);
        messagesContainer.scrollTop = messagesContainer.scrollHeight;
    }

    updateLastToolMessage(toolName, result) {
        const messages = this.container.querySelectorAll('.chat-message.tool');
        const lastTool = messages[messages.length - 1];

        if (lastTool) {
            const indicator = lastTool.querySelector('.tool-indicator');
            indicator.classList.remove('loading');
            indicator.querySelector('.spinner')?.remove();

            const resultEl = lastTool.querySelector('.tool-result');
            resultEl.innerHTML = this.formatToolResult(toolName, result);
        }
    }

    formatToolResult(toolName, result) {
        if (result.error) {
            return `<div class="tool-error">${result.error}</div>`;
        }

        // Tool-spezifische Formatierung
        switch (toolName) {
            case 'calculate_u_value':
                return this.formatUValueResult(result);
            case 'calculate_heating_load':
                return this.formatHeatingLoadResult(result);
            case 'search_projects':
                return this.formatProjectsResult(result);
            default:
                return `<pre>${JSON.stringify(result, null, 2)}</pre>`;
        }
    }

    formatUValueResult(result) {
        const content = JSON.parse(result.content[0].text);
        return `
            <div class="result-card u-value">
                <div class="result-value">${content.u_value.toFixed(2)} ${content.unit}</div>
                <div class="result-label">U-Wert nach ${content.standard}</div>
                <table class="result-details">
                    <tr><th>Bauteil</th><th>R [m²K/W]</th></tr>
                    ${content.layer_resistances.map(l =>
                        `<tr><td>${l.name}</td><td>${l.r.toFixed(3)}</td></tr>`
                    ).join('')}
                    <tr class="total"><td>Gesamt R</td><td>${content.r_total.toFixed(3)}</td></tr>
                </table>
            </div>
        `;
    }

    handleToolMention(text) {
        // @tool_name Syntax erkennen
        const match = text.match(/@(\w*)$/);

        if (match) {
            const partial = match[1].toLowerCase();
            const suggestions = this.tools.filter(t =>
                t.name.toLowerCase().includes(partial)
            );

            this.showToolSuggestions(suggestions);
        } else {
            this.hideToolSuggestions();
        }
    }

    showToolSuggestions(tools) {
        const container = this.container.querySelector('#tool-suggestions');

        if (tools.length === 0) {
            container.style.display = 'none';
            return;
        }

        container.innerHTML = tools.map(t => `
            <div class="tool-suggestion" data-tool="${t.name}">
                <strong>@${t.name}</strong>
                <span>${t.description}</span>
            </div>
        `).join('');

        container.style.display = 'block';

        // Click Handler
        container.querySelectorAll('.tool-suggestion').forEach(el => {
            el.addEventListener('click', () => {
                this.insertToolMention(el.dataset.tool);
            });
        });
    }

    hideToolSuggestions() {
        this.container.querySelector('#tool-suggestions').style.display = 'none';
    }

    insertToolMention(toolName) {
        const input = this.container.querySelector('#chat-input');
        input.value = input.value.replace(/@\w*$/, `@${toolName} `);
        input.focus();
        this.hideToolSuggestions();
    }

    handleQuickAction(action) {
        const prompts = {
            'analyze-building': 'Analysiere das aktuelle Gebäude und gib Sanierungsempfehlungen.',
            'calculate-u-value': 'Berechne den U-Wert für folgendes Bauteil: ',
            'find-funding': 'Welche Förderprogramme gibt es für '
        };

        const input = this.container.querySelector('#chat-input');
        input.value = prompts[action] || '';
        input.focus();
    }

    getContext() {
        // Aktueller Projektkontext
        return {
            project_id: this.options.projectId,
            building_id: this.options.buildingId,
            current_view: window.location.pathname
        };
    }

    getToken() {
        return localStorage.getItem('access_token');
    }

    formatContent(content) {
        // Markdown-ähnliche Formatierung
        return content
            .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
            .replace(/\*(.*?)\*/g, '<em>$1</em>')
            .replace(/`(.*?)`/g, '<code>$1</code>')
            .replace(/\n/g, '<br>');
    }
}

// Export
window.AIChatInterface = AIChatInterface;
```

---

## 5. Checkliste

```
✓ MCP Server Implementierung
✓ Tool-Registration und -Discovery
✓ Resource-Management
✓ Prompt-Templates
✓ HTTP Transport Layer
✓ Berechnungs-Tools (U-Wert, Heizlast, Energiebilanz)
✓ Daten-Tools (Projekte, Gebäude, Materialien)
✓ Report-Tools (Generierung, Formatierung)
✓ Externe Tools (Wetter, Förderungen)
✓ Custom Tool Creation
✓ Frontend Chat Interface
✓ Tool-Suggestions und Quick Actions
```

---

*Letzte Aktualisierung: 2025-11-25*
*Version: 1.0.0*
