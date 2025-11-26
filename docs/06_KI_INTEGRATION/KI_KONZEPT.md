# KI-Integration - Konzept und Architektur

## Einleitung

EnergieberaterPRO integriert KI-Funktionen auf allen Ebenen der Anwendung - von der Texterstellung über Computer Vision bis hin zu Predictive Analytics. Die Architektur folgt dem "Lokal-First" Prinzip für maximalen Datenschutz.

---

## 1. Übersicht

### 1.1 Ziele der KI-Integration

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           KI-FUNKTIONEN ÜBERSICHT                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ╔═══════════════════════╗  ╔═══════════════════════╗  ╔═══════════════════╗  │
│  ║   TEXTGENERIERUNG     ║  ║   COMPUTER VISION     ║  ║   ANALYTICS       ║  │
│  ║   ▬▬▬▬▬▬▬▬▬▬▬▬▬      ║  ║   ▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬   ║  ║   ▬▬▬▬▬▬▬▬▬▬▬    ║  │
│  ║ ✓ Berichtstexte       ║  ║ ✓ Dokumenten-OCR      ║  ║ ✓ Benchmarking    ║  │
│  ║ ✓ Empfehlungen        ║  ║ ✓ Typenschilder       ║  ║ ✓ Prognosen       ║  │
│  ║ ✓ E-Mail-Entwürfe     ║  ║ ○ Plandigitalisierung ║  ║ ○ Anomalien       ║  │
│  ║ ✓ Zusammenfassungen   ║  ║ ○ Fotoanalyse         ║  ║ ○ Wartung         ║  │
│  ╚═══════════════════════╝  ╚═══════════════════════╝  ╚═══════════════════╝  │
│                                                                                 │
│  ╔═══════════════════════╗  ╔═══════════════════════╗  ╔═══════════════════╗  │
│  ║   FÖRDERMITTEL-KI     ║  ║   RAG-SYSTEM          ║  ║   ASSISTENT       ║  │
│  ║   ▬▬▬▬▬▬▬▬▬▬▬▬▬      ║  ║   ▬▬▬▬▬▬▬▬▬▬▬        ║  ║   ▬▬▬▬▬▬▬▬▬▬▬    ║  │
│  ║ ✓ Programm-Suche      ║  ║ ✓ Normen-Wissen       ║  ║ ✓ Chat-Bot        ║  │
│  ║ ✓ Matching            ║  ║ ✓ FAQ-Antworten       ║  ║ ✓ Kontexthilfe    ║  │
│  ║ ✓ Förderhöhen         ║  ║ ✓ Quellenangaben      ║  ║ ○ Sprachassistent ║  │
│  ║ ○ Antragsassistent    ║  ║ ○ Semantische Suche   ║  ║ ○ Multi-Modal     ║  │
│  ╚═══════════════════════╝  ╚═══════════════════════╝  ╚═══════════════════╝  │
│                                                                                 │
│  Legende: ✓ = Phase 1-4  ○ = Phase 5-7                                         │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

**Kernfunktionen:**
- **Textgenerierung**: Automatische Erstellung von Berichtstexten
- **Computer Vision**: OCR, Typenschilder-Erkennung, Plananalyse
- **Predictive Analytics**: Verbrauchsprognosen, Anomalie-Erkennung
- **Fördermittel-KI**: Automatisches Matching Maßnahme → Förderprogramm
- **RAG-System**: Wissensbasierte Antworten mit Quellenangaben
- **Assistent**: Interaktiver Chat-Bot mit Kontextverständnis

### 1.2 Architekturprinzipien

| Prinzip | Beschreibung |
|---------|--------------|
| **Lokal-First** | Primär lokale Modelle (Ollama) für maximalen Datenschutz |
| **Modular** | Einfacher Austausch von LLM-Backends (Ollama ↔ OpenAI ↔ Claude) |
| **Fallback** | Graceful Degradation ohne KI (Templates, Regeln) |
| **Erklärbar** | Transparente Empfehlungen mit Begründung |
| **Effizient** | Caching, Batch-Processing, optimierte Prompts |

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

## 8. OCR & Computer Vision

### 8.1 Architektur

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        COMPUTER VISION PIPELINE                                  │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  INPUT                PREPROCESSING          EXTRACTION           OUTPUT        │
│  ─────                ─────────────          ──────────           ──────        │
│                                                                                 │
│  ┌─────────┐         ┌─────────────┐        ┌───────────┐        ┌──────────┐ │
│  │ Dokument │────────▶│ Bildkorrektur│───────▶│ Tesseract │───────▶│ JSON     │ │
│  │ (PDF)    │         │ Entzerrung   │        │ OCR       │        │ Text     │ │
│  └─────────┘         └─────────────┘        └───────────┘        └──────────┘ │
│                                                                                 │
│  ┌─────────┐         ┌─────────────┐        ┌───────────┐        ┌──────────┐ │
│  │ Typen-  │────────▶│ Ausschnitt  │───────▶│ EasyOCR   │───────▶│ Geräte-  │ │
│  │ schild  │         │ Erkennung   │        │           │        │ daten    │ │
│  └─────────┘         └─────────────┘        └───────────┘        └──────────┘ │
│                                                                                 │
│  ┌─────────┐         ┌─────────────┐        ┌───────────┐        ┌──────────┐ │
│  │ Grundriss│───────▶│ Vektorisier.│───────▶│ YOLO/     │───────▶│ Raum-    │ │
│  │ (Foto)  │         │             │        │ LayoutLM  │        │ geometrie│ │
│  └─────────┘         └─────────────┘        └───────────┘        └──────────┘ │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 8.2 Typenschilder-Erkennung

```python
# ai/vision/nameplate.py
from PIL import Image
import easyocr
import re

class NameplateExtractor:
    """
    Extrahiert Daten von Typenschildern (Heizung, Wärmepumpe, PV)
    """

    def __init__(self):
        self.reader = easyocr.Reader(['de', 'en'])

        # Muster für typische Felder
        self.patterns = {
            'leistung': r'(\d+(?:[,.]\d+)?)\s*(kW|W)',
            'baujahr': r'(19|20)\d{2}',
            'hersteller': r'(Viessmann|Buderus|Vaillant|Bosch|Wolf|...)',
            'modell': r'(?:Typ|Type|Model)[:\s]*([A-Z0-9\-]+)',
            'cop': r'COP[:\s]*(\d+[,.]\d+)',
            'scop': r'SCOP[:\s]*(\d+[,.]\d+)',
        }

    async def extract(self, image: Image) -> dict:
        """Extrahiert strukturierte Daten aus Typenschild"""
        # OCR durchführen
        results = self.reader.readtext(image)
        text = ' '.join([r[1] for r in results])

        # Felder extrahieren
        extracted = {}
        for field, pattern in self.patterns.items():
            match = re.search(pattern, text, re.IGNORECASE)
            if match:
                extracted[field] = match.group(1)

        return {
            'raw_text': text,
            'extracted': extracted,
            'confidence': self._calculate_confidence(results)
        }
```

### 8.3 Dokumenten-OCR

```python
# ai/vision/document_ocr.py
import pytesseract
from pdf2image import convert_from_path

class DocumentOCR:
    """
    OCR für Rechnungen, Energieausweise, Pläne
    """

    async def process_invoice(self, pdf_path: str) -> dict:
        """Extrahiert Daten aus Energierechnungen"""
        images = convert_from_path(pdf_path)

        extracted = {
            'verbrauch': [],
            'kosten': [],
            'zeitraum': None,
            'anbieter': None
        }

        for image in images:
            text = pytesseract.image_to_string(image, lang='deu')

            # Strukturierte Extraktion
            extracted['verbrauch'].extend(
                self._extract_consumption(text)
            )
            extracted['kosten'].extend(
                self._extract_costs(text)
            )

        return extracted
```

---

## 9. Predictive Analytics

### 9.1 Verbrauchsprognosen

```python
# ai/analytics/prediction.py
from sklearn.ensemble import GradientBoostingRegressor
import numpy as np

class ConsumptionPredictor:
    """
    ML-basierte Verbrauchsprognosen
    """

    def __init__(self):
        self.model = GradientBoostingRegressor(
            n_estimators=100,
            max_depth=5,
            learning_rate=0.1
        )

    def prepare_features(self, building_data: dict, weather_data: dict) -> np.array:
        """
        Feature Engineering für Prognosemodell
        """
        return np.array([
            building_data['heated_area'],
            building_data['u_wall'],
            building_data['u_roof'],
            building_data['construction_year'],
            weather_data['hdd'],  # Heizgradtage
            weather_data['mean_temp'],
            # ... weitere Features
        ])

    async def predict_annual_consumption(
        self,
        building_data: dict,
        weather_scenario: str = 'normal'
    ) -> dict:
        """
        Prognostiziert Jahresverbrauch
        """
        weather = self._get_weather_scenario(weather_scenario)
        features = self.prepare_features(building_data, weather)

        prediction = self.model.predict([features])[0]
        confidence = self._calculate_confidence_interval(features)

        return {
            'predicted_kwh': prediction,
            'confidence_low': confidence[0],
            'confidence_high': confidence[1],
            'scenario': weather_scenario
        }
```

### 9.2 Anomalie-Erkennung

```python
# ai/analytics/anomaly.py
from sklearn.ensemble import IsolationForest

class AnomalyDetector:
    """
    Erkennt ungewöhnliche Verbrauchsmuster
    """

    def __init__(self):
        self.model = IsolationForest(
            contamination=0.05,
            random_state=42
        )

    async def detect_anomalies(
        self,
        consumption_history: list[dict]
    ) -> list[dict]:
        """
        Findet Anomalien in Verbrauchsdaten
        """
        features = self._prepare_time_series(consumption_history)
        predictions = self.model.fit_predict(features)

        anomalies = []
        for i, pred in enumerate(predictions):
            if pred == -1:  # Anomalie erkannt
                anomalies.append({
                    'timestamp': consumption_history[i]['timestamp'],
                    'value': consumption_history[i]['value'],
                    'expected_range': self._get_expected_range(i),
                    'severity': self._calculate_severity(consumption_history[i])
                })

        return anomalies
```

---

## 10. Fördermittel-KI

### 10.1 Förderprogramm-Matching

```python
# ai/funding/matcher.py
from app.ai.client import OllamaClient
from app.ai.vectorstore import VectorStore

class FundingMatcher:
    """
    Matcht Sanierungsmaßnahmen mit Förderprogrammen
    """

    def __init__(self, llm: OllamaClient, vectorstore: VectorStore):
        self.llm = llm
        self.vectorstore = vectorstore

    async def find_programs(
        self,
        measure: dict,
        location: dict,
        building_data: dict
    ) -> list[dict]:
        """
        Findet passende Förderprogramme
        """
        # Kontext aufbauen
        query = f"""
        Maßnahme: {measure['name']}
        Kategorie: {measure['category']}
        Land: {location['country']}
        Bundesland: {location['state']}
        Gebäudetyp: {building_data['type']}
        """

        # Relevante Programme aus Vector Store
        programs = self.vectorstore.query(
            collection_name="funding_programs",
            query_text=query,
            n_results=10,
            where={"country": location['country'], "active": True}
        )

        # LLM-basiertes Matching und Ranking
        matched = []
        for program in programs:
            eligibility = await self._check_eligibility(
                measure, program, building_data
            )
            if eligibility['eligible']:
                funding = await self._calculate_funding(
                    measure, program, building_data
                )
                matched.append({
                    'program': program,
                    'eligibility': eligibility,
                    'funding': funding,
                    'score': eligibility['confidence']
                })

        return sorted(matched, key=lambda x: x['score'], reverse=True)

    async def _calculate_funding(
        self,
        measure: dict,
        program: dict,
        building_data: dict
    ) -> dict:
        """
        Berechnet erwartete Förderhöhe
        """
        base_rate = program['metadata'].get('rate', 0)
        max_amount = program['metadata'].get('max_amount', float('inf'))

        # Bonusfaktoren
        bonuses = 0
        if building_data.get('worst_performing'):
            bonuses += program['metadata'].get('wpb_bonus', 0)
        if measure.get('exceeds_requirements'):
            bonuses += program['metadata'].get('efficiency_bonus', 0)

        total_rate = min(base_rate + bonuses, 0.5)  # Max 50%

        eligible_costs = measure['investment']
        funding_amount = min(eligible_costs * total_rate, max_amount)

        return {
            'rate': total_rate,
            'amount': funding_amount,
            'eligible_costs': eligible_costs,
            'bonuses_applied': bonuses > 0
        }
```

### 10.2 Förderdatenbank

```
Collections in ChromaDB:
├── funding_programs_de          # BEG, KfW, Landesförderungen
│   ├── metadata: {country, state, category, rate, max_amount, deadline}
│   └── document: Programmbeschreibung, Voraussetzungen
├── funding_programs_at          # Sanierungsoffensive, Landesförderungen
│   ├── metadata: {country, state, category, rate, max_amount, deadline}
│   └── document: Programmbeschreibung, Voraussetzungen
└── funding_faq                  # Häufige Fragen zu Förderungen
```

### 10.3 Automatische Aktualisierung

```python
# ai/funding/updater.py
class FundingDatabaseUpdater:
    """
    Hält Förderdatenbank aktuell
    """

    async def update_from_sources(self):
        """
        Aktualisiert aus offiziellen Quellen
        """
        sources = [
            'https://www.bafa.de/...',
            'https://www.kfw.de/...',
            'https://www.umweltfoerderung.at/...'
        ]

        for source in sources:
            # Web-Scraping oder API-Abruf
            data = await self._fetch_source(source)

            # LLM-basierte Strukturierung
            structured = await self._structure_with_llm(data)

            # Upsert in Vector Store
            await self._update_vectorstore(structured)

    async def notify_changes(self):
        """
        Benachrichtigt über Programmänderungen
        """
        changes = await self._detect_changes()

        for change in changes:
            await self._send_notification(
                type=change['type'],  # 'new', 'modified', 'expired'
                program=change['program']
            )
```

---

## 11. API Endpoints

```python
# api/v1/ai.py
from fastapi import APIRouter, Depends
from app.ai.agents import ReportAgent, AnalysisAgent
from app.ai.rag import RAGService
from app.ai.vision import NameplateExtractor, DocumentOCR
from app.ai.funding import FundingMatcher
from app.ai.analytics import ConsumptionPredictor, AnomalyDetector

router = APIRouter(prefix="/ai", tags=["AI"])

# --- Textgenerierung ---
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

# --- Analyse ---
@router.post("/analyze/building")
async def analyze_building(
    building_id: UUID,
    analysis_agent: AnalysisAgent = Depends()
) -> dict:
    """Analysiert Gebäude und gibt Bewertung"""
    pass

# --- Computer Vision ---
@router.post("/ocr/nameplate")
async def extract_nameplate(
    image: UploadFile,
    extractor: NameplateExtractor = Depends()
) -> dict:
    """Extrahiert Daten aus Typenschild-Foto"""
    pass

@router.post("/ocr/document")
async def process_document(
    file: UploadFile,
    doc_type: str,  # 'invoice', 'energy_certificate', 'plan'
    ocr: DocumentOCR = Depends()
) -> dict:
    """OCR für verschiedene Dokumenttypen"""
    pass

# --- Fördermittel ---
@router.post("/funding/match")
async def match_funding(
    measure_id: UUID,
    matcher: FundingMatcher = Depends()
) -> list[dict]:
    """Findet passende Förderprogramme"""
    pass

@router.get("/funding/programs")
async def list_funding_programs(
    country: str,
    category: str = None
) -> list[dict]:
    """Listet verfügbare Förderprogramme"""
    pass

# --- Analytics ---
@router.post("/analytics/predict")
async def predict_consumption(
    building_id: UUID,
    predictor: ConsumptionPredictor = Depends()
) -> dict:
    """Prognostiziert Jahresverbrauch"""
    pass

@router.post("/analytics/anomalies")
async def detect_anomalies(
    building_id: UUID,
    detector: AnomalyDetector = Depends()
) -> list[dict]:
    """Erkennt Verbrauchsanomalien"""
    pass

# --- Chat & RAG ---
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
        "ocr_available": True,
        "models": await llm_client.list_models()
    }
```

---

## 12. Offline-Fähigkeit

### 12.1 Fallback-Strategien

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

*Letzte Aktualisierung: 2025-11-26*
*Version: 2.0.0*
