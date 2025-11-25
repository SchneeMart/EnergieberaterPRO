# KI-Integration - Konzept und Architektur

## 1. Übersicht

### 1.1 Ziele der KI-Integration
- **Textgenerierung**: Automatische Erstellung von Berichtstexten
- **Analyse**: Intelligente Auswertung von Gebäudedaten
- **Empfehlungen**: Maßnahmenvorschläge basierend auf Berechnungen
- **Assistent**: Interaktive Hilfe für Benutzer
- **Datenextraktion**: Automatisches Auslesen von Dokumenten

### 1.2 Architekturprinzipien
- **Lokal-First**: Primär lokale Modelle (Datenschutz)
- **Modular**: Einfacher Austausch von LLM-Backends
- **Fallback**: Graceful Degradation ohne KI
- **Erklärbar**: Transparente Empfehlungen

---

## 2. Technologie-Stack

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           KI-ARCHITEKTUR                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                        APPLICATION LAYER                             │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌───────────┐  │   │
│  │  │   Report    │  │  Analysis   │  │  Recommend  │  │  Chat     │  │   │
│  │  │   Agent     │  │   Agent     │  │    Agent    │  │  Agent    │  │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └───────────┘  │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                        │
│                                    ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         ORCHESTRATION LAYER                          │   │
│  │  ┌───────────────────┐  ┌───────────────────┐  ┌─────────────────┐  │   │
│  │  │   Prompt Manager  │  │   Context Builder │  │   Output Parser │  │   │
│  │  └───────────────────┘  └───────────────────┘  └─────────────────┘  │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                        │
│              ┌─────────────────────┼─────────────────────┐                  │
│              ▼                     ▼                     ▼                  │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐           │
│  │   LLM Client    │   │   Embeddings    │   │   Vector Store  │           │
│  │   (Ollama)      │   │ (Transformers)  │   │   (ChromaDB)    │           │
│  └─────────────────┘   └─────────────────┘   └─────────────────┘           │
│           │                     │                     │                     │
│           └─────────────────────┼─────────────────────┘                     │
│                                 ▼                                           │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                          MODEL LAYER                                 │   │
│  │                                                                      │   │
│  │  Local (Ollama):              Cloud (Optional):                      │   │
│  │  ┌────────────┐               ┌────────────┐                        │   │
│  │  │ Mistral 7B │               │ GPT-4      │                        │   │
│  │  │ Llama2 7B  │               │ Claude 3   │                        │   │
│  │  │ CodeLlama  │               │ Gemini     │                        │   │
│  │  └────────────┘               └────────────┘                        │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Ollama Integration

### 3.1 Konfiguration

```python
# ai/client.py
from typing import Optional, AsyncIterator
import httpx
from pydantic import BaseModel

class OllamaConfig(BaseModel):
    base_url: str = "http://ollama:11434"
    default_model: str = "mistral:7b-instruct"
    timeout: int = 120
    max_tokens: int = 4096
    temperature: float = 0.7

class OllamaClient:
    def __init__(self, config: OllamaConfig):
        self.config = config
        self.client = httpx.AsyncClient(
            base_url=config.base_url,
            timeout=config.timeout
        )

    async def generate(
        self,
        prompt: str,
        model: Optional[str] = None,
        system: Optional[str] = None,
        temperature: Optional[float] = None,
        max_tokens: Optional[int] = None,
        stream: bool = False
    ) -> str | AsyncIterator[str]:
        """
        Generiert Text mit dem LLM

        Args:
            prompt: Der Eingabe-Prompt
            model: Modell (default: mistral:7b-instruct)
            system: System-Prompt
            temperature: Kreativität (0.0-1.0)
            max_tokens: Maximale Ausgabelänge
            stream: Streaming-Ausgabe

        Returns:
            Generierter Text oder Stream
        """
        payload = {
            "model": model or self.config.default_model,
            "prompt": prompt,
            "options": {
                "temperature": temperature or self.config.temperature,
                "num_predict": max_tokens or self.config.max_tokens
            },
            "stream": stream
        }

        if system:
            payload["system"] = system

        if stream:
            return self._stream_generate(payload)
        else:
            response = await self.client.post("/api/generate", json=payload)
            data = response.json()
            return data["response"]

    async def _stream_generate(self, payload: dict) -> AsyncIterator[str]:
        async with self.client.stream("POST", "/api/generate", json=payload) as response:
            async for line in response.aiter_lines():
                if line:
                    data = json.loads(line)
                    if not data.get("done"):
                        yield data.get("response", "")

    async def chat(
        self,
        messages: list[dict],
        model: Optional[str] = None
    ) -> str:
        """
        Chat-Completion API

        messages: [
            {"role": "system", "content": "..."},
            {"role": "user", "content": "..."},
            {"role": "assistant", "content": "..."}
        ]
        """
        payload = {
            "model": model or self.config.default_model,
            "messages": messages
        }

        response = await self.client.post("/api/chat", json=payload)
        data = response.json()
        return data["message"]["content"]

    async def embeddings(
        self,
        text: str,
        model: str = "nomic-embed-text"
    ) -> list[float]:
        """Generiert Embeddings für Text"""
        response = await self.client.post("/api/embeddings", json={
            "model": model,
            "prompt": text
        })
        return response.json()["embedding"]

    async def list_models(self) -> list[dict]:
        """Listet verfügbare Modelle"""
        response = await self.client.get("/api/tags")
        return response.json()["models"]

    async def pull_model(self, model: str) -> None:
        """Lädt ein Modell herunter"""
        await self.client.post("/api/pull", json={"name": model})
```

### 3.2 Modell-Empfehlungen

| Modell | Größe | RAM | Anwendung |
|--------|-------|-----|-----------|
| mistral:7b-instruct | 4.1GB | 8GB | Allgemeine Textgenerierung |
| llama2:7b | 3.8GB | 8GB | Alternative zu Mistral |
| codellama:7b | 3.8GB | 8GB | Code-Generierung |
| nomic-embed-text | 275MB | 2GB | Embeddings |
| mixtral:8x7b | 26GB | 48GB | Komplexe Aufgaben (wenn verfügbar) |

---

## 4. Vector Store (ChromaDB)

### 4.1 Konfiguration

```python
# ai/vectorstore.py
import chromadb
from chromadb.config import Settings
from sentence_transformers import SentenceTransformer

class VectorStore:
    def __init__(self, persist_dir: str = "./data/chromadb"):
        self.client = chromadb.Client(Settings(
            chroma_db_impl="duckdb+parquet",
            persist_directory=persist_dir,
            anonymized_telemetry=False
        ))

        # Embedding-Modell (lokal)
        self.embedder = SentenceTransformer(
            'sentence-transformers/paraphrase-multilingual-mpnet-base-v2'
        )

    def get_collection(self, name: str):
        return self.client.get_or_create_collection(
            name=name,
            metadata={"hnsw:space": "cosine"}
        )

    def add_documents(
        self,
        collection_name: str,
        documents: list[str],
        metadatas: list[dict] = None,
        ids: list[str] = None
    ):
        """Fügt Dokumente zur Collection hinzu"""
        collection = self.get_collection(collection_name)

        # Embeddings generieren
        embeddings = self.embedder.encode(documents).tolist()

        collection.add(
            documents=documents,
            embeddings=embeddings,
            metadatas=metadatas or [{}] * len(documents),
            ids=ids or [str(i) for i in range(len(documents))]
        )

    def query(
        self,
        collection_name: str,
        query_text: str,
        n_results: int = 5,
        where: dict = None
    ) -> list[dict]:
        """Semantische Suche in Collection"""
        collection = self.get_collection(collection_name)

        # Query Embedding
        query_embedding = self.embedder.encode([query_text]).tolist()

        results = collection.query(
            query_embeddings=query_embedding,
            n_results=n_results,
            where=where
        )

        return [
            {
                "document": doc,
                "metadata": meta,
                "distance": dist
            }
            for doc, meta, dist in zip(
                results["documents"][0],
                results["metadatas"][0],
                results["distances"][0]
            )
        ]
```

### 4.2 Collections

```
Collections:
├── report_templates        # Berichtsvorlagen und -texte
├── building_descriptions   # Gebäude-/Bauteilbeschreibungen
├── regulations             # Normen, Gesetze, Richtlinien
├── recommendations         # Sanierungsempfehlungen
├── materials               # Materialbeschreibungen
└── faq                     # Häufig gestellte Fragen
```

---

## 5. Prompt Engineering

### 5.1 System-Prompts

```python
# ai/prompts/system_prompts.py

ENERGY_CONSULTANT_SYSTEM = """
Du bist ein erfahrener Energieberater mit Expertise in:
- Gebäudeenergetik und Bauphysik
- DIN V 18599, GEG, EN-Normen
- Heiz- und Kühllastberechnung
- Erneuerbare Energien und Wärmepumpen
- Sanierungskonzepte und Wirtschaftlichkeit

Deine Aufgaben:
1. Präzise, fachlich korrekte Informationen liefern
2. Komplexe Sachverhalte verständlich erklären
3. Quellenangaben für Normen/Gesetze nennen
4. Keine Spekulationen - bei Unsicherheit nachfragen

Formatierung:
- Strukturierte Antworten mit Überschriften
- Listen für Aufzählungen
- Einheiten immer angeben (kWh, W/m²K, etc.)
"""

REPORT_WRITER_SYSTEM = """
Du bist ein technischer Redakteur für Energieberichte.
Erstelle professionelle, gut lesbare Texte für:
- Energieausweise
- Energieaudit-Berichte
- Sanierungsfahrpläne

Stil:
- Sachlich, aber verständlich
- Fachbegriffe erklären
- Konkrete Zahlen und Fakten verwenden
- Handlungsempfehlungen klar formulieren

Zielgruppe: Gebäudeeigentümer ohne technischen Hintergrund
"""

RECOMMENDATION_SYSTEM = """
Du erstellst Sanierungsempfehlungen basierend auf:
- Aktuellen Gebäudedaten und Berechnungen
- Wirtschaftlichkeit (Amortisation, Förderung)
- Technische Machbarkeit
- Priorisierung nach Einsparpotenzial

Format für jede Empfehlung:
1. Maßnahme (kurz und präzise)
2. Einsparpotenzial (kWh/a, %)
3. Geschätzte Kosten
4. Amortisationszeit
5. Fördermöglichkeiten
6. Technische Hinweise
"""
```

### 5.2 Prompt Templates

```python
# ai/prompts/templates.py
from string import Template

class PromptTemplates:

    BUILDING_DESCRIPTION = Template("""
Erstelle eine präzise Gebäudebeschreibung:

GEBÄUDEDATEN:
- Typ: $building_type
- Baujahr: $construction_year
- Wohnfläche: $heated_area m²
- Geschosse: $floors
- Energieeffizienzklasse: $efficiency_class

HÜLLE:
- Außenwand U-Wert: $u_wall W/(m²K)
- Dach U-Wert: $u_roof W/(m²K)
- Fenster U-Wert: $u_window W/(m²K)

ANLAGENTECHNIK:
- Heizung: $heating_system
- Warmwasser: $dhw_system
- Lüftung: $ventilation_system

KENNWERTE:
- Heizwärmebedarf: $heating_demand kWh/(m²a)
- Primärenergiebedarf: $primary_energy kWh/(m²a)
- CO2-Emissionen: $co2_emissions kg/(m²a)

Erstelle eine 2-3 Absätze lange Beschreibung des energetischen
Zustands für den Energieausweis.
""")

    MEASURE_RECOMMENDATION = Template("""
Analysiere folgende Sanierungsmaßnahme:

MASSNAHME: $measure_name

AUSGANGSZUSTAND:
- Bauteil: $component
- Aktueller U-Wert: $current_u W/(m²K)
- Fläche: $area m²

ZIELZUSTAND:
- Neuer U-Wert: $target_u W/(m²K)
- Erreichte Dämmstärke: $insulation_thickness mm

EINSPARUNG:
- Transmissionswärme: $heat_saving kWh/a
- Heizkosten: $cost_saving €/a
- CO2: $co2_saving kg/a

INVESTITION:
- Geschätzte Kosten: $investment €
- Mögliche Förderung: $funding €

Erstelle eine detaillierte Empfehlung mit:
1. Bewertung der Wirtschaftlichkeit
2. Technische Umsetzungshinweise
3. Priorisierungsempfehlung
""")

    AUDIT_SUMMARY = Template("""
Erstelle eine Zusammenfassung für das Energieaudit:

UNTERNEHMEN: $company_name
STANDORT: $location

ENERGIEVERBRAUCH (Referenzjahr $reference_year):
- Strom: $electricity_kwh kWh ($electricity_cost €)
- Erdgas: $gas_kwh kWh ($gas_cost €)
- Fernwärme: $district_kwh kWh ($district_cost €)

GESAMTENERGIE: $total_kwh kWh
GESAMTKOSTEN: $total_cost €

HAUPTVERBRAUCHER:
$main_consumers

EINSPARPOTENZIAL:
- Technisch: $technical_potential kWh/a ($technical_percent %)
- Wirtschaftlich: $economic_potential kWh/a ($economic_percent %)

MAßNAHMEN:
$measures_summary

Erstelle eine Management-Summary (max. 1 Seite) mit:
1. Kernaussagen zum energetischen Zustand
2. Wichtigste Einsparpotenziale
3. Top-3 Handlungsempfehlungen
4. Nächste Schritte
""")
```

---

## 6. AI Agents

### 6.1 Report Agent

```python
# ai/agents/report_agent.py
from typing import Optional
from app.ai.client import OllamaClient
from app.ai.vectorstore import VectorStore
from app.ai.prompts import PromptTemplates, REPORT_WRITER_SYSTEM

class ReportAgent:
    """
    Agent für die automatische Berichtsgenerierung
    """

    def __init__(
        self,
        llm: OllamaClient,
        vectorstore: VectorStore
    ):
        self.llm = llm
        self.vectorstore = vectorstore
        self.templates = PromptTemplates()

    async def generate_building_description(
        self,
        building_data: dict,
        calculation_results: dict
    ) -> str:
        """
        Generiert Gebäudebeschreibung für Energieausweis
        """
        # Ähnliche Beschreibungen aus Vector Store abrufen
        similar_docs = self.vectorstore.query(
            collection_name="building_descriptions",
            query_text=f"{building_data['building_type']} {building_data['construction_year']}",
            n_results=3
        )

        # Prompt zusammenstellen
        prompt = self.templates.BUILDING_DESCRIPTION.substitute(
            building_type=building_data['building_type'],
            construction_year=building_data['construction_year'],
            heated_area=building_data['heated_area'],
            floors=building_data['floors_above_ground'],
            efficiency_class=calculation_results['efficiency_class'],
            u_wall=building_data['avg_u_wall'],
            u_roof=building_data['avg_u_roof'],
            u_window=building_data['avg_u_window'],
            heating_system=building_data['heating_system'],
            dhw_system=building_data['dhw_system'],
            ventilation_system=building_data.get('ventilation_system', 'Fensterlüftung'),
            heating_demand=calculation_results['heating_demand'],
            primary_energy=calculation_results['primary_energy'],
            co2_emissions=calculation_results['co2_emissions']
        )

        # Kontext aus ähnlichen Dokumenten
        context = "\n".join([doc["document"] for doc in similar_docs])

        # LLM aufrufen
        response = await self.llm.generate(
            prompt=f"BEISPIELE:\n{context}\n\nAUFGABE:\n{prompt}",
            system=REPORT_WRITER_SYSTEM,
            temperature=0.5
        )

        return response

    async def generate_recommendations(
        self,
        building_data: dict,
        calculation_results: dict,
        available_measures: list[dict]
    ) -> list[dict]:
        """
        Generiert Sanierungsempfehlungen
        """
        recommendations = []

        for measure in available_measures:
            prompt = self.templates.MEASURE_RECOMMENDATION.substitute(
                measure_name=measure['name'],
                component=measure['component'],
                current_u=measure['current_u_value'],
                target_u=measure['target_u_value'],
                area=measure['area'],
                insulation_thickness=measure.get('insulation_thickness', '-'),
                heat_saving=measure['energy_saving'],
                cost_saving=measure['cost_saving'],
                co2_saving=measure['co2_saving'],
                investment=measure['investment'],
                funding=measure['funding']
            )

            text = await self.llm.generate(
                prompt=prompt,
                system=RECOMMENDATION_SYSTEM,
                temperature=0.4
            )

            recommendations.append({
                "measure_id": measure['id'],
                "measure_name": measure['name'],
                "generated_text": text,
                "priority": self._calculate_priority(measure)
            })

        return sorted(recommendations, key=lambda x: x['priority'], reverse=True)

    def _calculate_priority(self, measure: dict) -> float:
        """Berechnet Prioritätsscore für Maßnahme"""
        # Amortisation
        amortization = measure['investment'] / max(measure['cost_saving'], 1)

        # Score: Niedrigere Amortisation = höhere Priorität
        score = 10 - min(amortization / 2, 10)

        # Bonus für hohe Einsparung
        if measure['energy_saving'] > 5000:
            score += 2
        if measure['energy_saving'] > 10000:
            score += 3

        return score
```

### 6.2 Analysis Agent

```python
# ai/agents/analysis_agent.py
class AnalysisAgent:
    """
    Agent für intelligente Gebäudeanalyse
    """

    async def analyze_building_efficiency(
        self,
        building_data: dict,
        calculation_results: dict
    ) -> dict:
        """
        Analysiert den energetischen Zustand eines Gebäudes
        """
        prompt = f"""
Analysiere folgendes Gebäude:

Baujahr: {building_data['construction_year']}
Typ: {building_data['building_type']}
Heizwärmebedarf: {calculation_results['heating_demand']} kWh/(m²a)
Primärenergie: {calculation_results['primary_energy']} kWh/(m²a)

Vergleiche mit:
- Typischen Werten für dieses Baujahr
- Anforderungen nach GEG
- Niedrigenergiehaus-Standard

Gib eine strukturierte Analyse mit:
1. Bewertung des Ist-Zustands (gut/mittel/schlecht)
2. Hauptschwachstellen
3. Potenzial für Verbesserungen
4. Empfohlene nächste Schritte
"""

        analysis_text = await self.llm.generate(
            prompt=prompt,
            system=ENERGY_CONSULTANT_SYSTEM,
            temperature=0.3
        )

        # Strukturierte Daten extrahieren
        return {
            "rating": self._extract_rating(analysis_text),
            "weaknesses": self._extract_list(analysis_text, "Schwachstellen"),
            "potential": self._extract_potential(calculation_results),
            "recommendations": self._extract_recommendations(analysis_text),
            "full_analysis": analysis_text
        }

    async def compare_variants(
        self,
        variants: list[dict]
    ) -> dict:
        """
        Vergleicht verschiedene Sanierungsvarianten
        """
        comparison_prompt = self._build_comparison_prompt(variants)

        comparison = await self.llm.generate(
            prompt=comparison_prompt,
            system=ENERGY_CONSULTANT_SYSTEM,
            temperature=0.3
        )

        return {
            "comparison_text": comparison,
            "recommended_variant": self._extract_recommendation(comparison),
            "ranking": self._calculate_ranking(variants)
        }
```

---

## 7. RAG (Retrieval Augmented Generation)

### 7.1 Wissensbasierte Antworten

```python
# ai/rag.py
class RAGService:
    """
    Retrieval Augmented Generation für fachliche Fragen
    """

    def __init__(self, llm: OllamaClient, vectorstore: VectorStore):
        self.llm = llm
        self.vectorstore = vectorstore

    async def answer_question(
        self,
        question: str,
        collections: list[str] = ["regulations", "faq", "recommendations"]
    ) -> dict:
        """
        Beantwortet fachliche Fragen mit Quellenangaben
        """
        # Relevante Dokumente abrufen
        all_docs = []
        for collection in collections:
            docs = self.vectorstore.query(
                collection_name=collection,
                query_text=question,
                n_results=3
            )
            all_docs.extend(docs)

        # Nach Relevanz sortieren
        all_docs.sort(key=lambda x: x["distance"])
        top_docs = all_docs[:5]

        # Kontext aufbauen
        context = "\n\n".join([
            f"[Quelle: {doc['metadata'].get('source', 'Unbekannt')}]\n{doc['document']}"
            for doc in top_docs
        ])

        # Prompt mit Kontext
        prompt = f"""
Beantworte die folgende Frage basierend auf den gegebenen Quellen.
Wenn die Quellen keine ausreichende Information enthalten, sage das ehrlich.
Zitiere relevante Normen oder Gesetze.

QUELLEN:
{context}

FRAGE: {question}

ANTWORT:
"""

        answer = await self.llm.generate(
            prompt=prompt,
            system=ENERGY_CONSULTANT_SYSTEM,
            temperature=0.3
        )

        return {
            "answer": answer,
            "sources": [
                {
                    "source": doc["metadata"].get("source"),
                    "relevance": 1 - doc["distance"]
                }
                for doc in top_docs
            ]
        }
```

### 7.2 Initialisierung der Wissensbasis

```python
# ai/init_knowledge_base.py
async def initialize_knowledge_base(vectorstore: VectorStore):
    """
    Initialisiert die Wissensbasis mit Stammdaten
    """

    # Normen und Vorschriften
    regulations = [
        {
            "text": "Nach GEG §10 darf der Primärenergiebedarf eines Neubaus "
                    "den des Referenzgebäudes nicht überschreiten...",
            "metadata": {"source": "GEG 2024 §10", "type": "regulation"}
        },
        {
            "text": "Die Heizlastberechnung nach DIN EN 12831 berücksichtigt "
                    "Transmissions- und Lüftungswärmeverluste...",
            "metadata": {"source": "DIN EN 12831", "type": "norm"}
        },
        # ... weitere Einträge
    ]

    vectorstore.add_documents(
        collection_name="regulations",
        documents=[r["text"] for r in regulations],
        metadatas=[r["metadata"] for r in regulations]
    )

    # Häufige Fragen
    faq = [
        {
            "text": "Frage: Was ist der Unterschied zwischen Endenergie und "
                    "Primärenergie? Antwort: Endenergie ist die Energie, die "
                    "am Gebäude ankommt (z.B. Gas, Strom). Primärenergie "
                    "berücksichtigt zusätzlich den Aufwand für Förderung, "
                    "Umwandlung und Transport...",
            "metadata": {"source": "FAQ", "type": "faq"}
        },
        # ... weitere Einträge
    ]

    vectorstore.add_documents(
        collection_name="faq",
        documents=[f["text"] for f in faq],
        metadatas=[f["metadata"] for f in faq]
    )
```

---

## 8. API Endpoints

```python
# api/v1/ai.py
from fastapi import APIRouter, Depends
from app.ai.agents import ReportAgent, AnalysisAgent
from app.ai.rag import RAGService

router = APIRouter(prefix="/ai", tags=["AI"])

@router.post("/generate/building-description")
async def generate_building_description(
    building_id: UUID,
    report_agent: ReportAgent = Depends()
) -> dict:
    """Generiert Gebäudebeschreibung für Berichte"""
    pass

@router.post("/generate/recommendations")
async def generate_recommendations(
    calculation_id: UUID,
    report_agent: ReportAgent = Depends()
) -> list[dict]:
    """Generiert Sanierungsempfehlungen"""
    pass

@router.post("/analyze/building")
async def analyze_building(
    building_id: UUID,
    analysis_agent: AnalysisAgent = Depends()
) -> dict:
    """Analysiert Gebäude und gibt Bewertung"""
    pass

@router.post("/chat")
async def chat(
    message: str,
    context: dict = None,
    rag_service: RAGService = Depends()
) -> dict:
    """Chat-Endpunkt für Benutzeranfragen"""
    pass

@router.get("/status")
async def ai_status() -> dict:
    """Status der KI-Komponenten"""
    return {
        "llm_available": True,
        "vectorstore_available": True,
        "models": await llm_client.list_models()
    }
```

---

## 9. Offline-Fähigkeit

### 9.1 Fallback-Strategien

```python
class AIServiceWithFallback:
    """
    KI-Service mit Fallback wenn LLM nicht verfügbar
    """

    async def generate_description(self, data: dict) -> str:
        try:
            return await self.llm_generate(data)
        except Exception:
            return self._template_based_description(data)

    def _template_based_description(self, data: dict) -> str:
        """
        Regelbasierte Beschreibung ohne KI
        """
        templates = {
            "single_family": "Das {year} errichtete Einfamilienhaus...",
            "multi_family": "Das {year} erbaute Mehrfamilienhaus...",
        }

        template = templates.get(data["building_type"], templates["single_family"])
        return template.format(year=data["construction_year"])
```

---

*Letzte Aktualisierung: 2025-11-25*
*Version: 0.1.0*
