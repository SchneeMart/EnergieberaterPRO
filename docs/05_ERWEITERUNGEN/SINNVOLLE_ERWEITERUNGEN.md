# Sinnvolle Erweiterungen für EnergieberaterPRO

## Übersicht der geplanten Erweiterungen

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        ERWEITERUNGS-ROADMAP                                      │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  PHASE 1: KERN-FUNKTIONEN (MVP)                                                 │
│  ══════════════════════════════                                                 │
│  ✓ Energieausweis DE (Bedarfsausweis)                                           │
│  ✓ Heizlastberechnung                                                           │
│  ✓ U-Wert-Berechnung                                                            │
│  ✓ Grundlegende Kundenverwaltung                                                │
│                                                                                  │
│  PHASE 2: ERWEITERTE PRODUKTE                                                   │
│  ════════════════════════════                                                   │
│  □ iSFP (individueller Sanierungsfahrplan)                                      │
│  □ Energieausweis AT (ÖNORM)                                                    │
│  □ Energieaudit DIN EN 16247                                                    │
│  □ Verbrauchsausweis                                                            │
│                                                                                  │
│  PHASE 3: PREMIUM-FEATURES                                                      │
│  ═════════════════════════                                                      │
│  □ KI-Assistent                                                                 │
│  □ Automatische Dokumentenerkennung (OCR)                                       │
│  □ Förderungsoptimierung                                                        │
│  □ Thermografie-Integration                                                     │
│                                                                                  │
│  PHASE 4: INTEGRATION & AUTOMATISIERUNG                                         │
│  ═══════════════════════════════════════                                        │
│  □ CAD-Import (DXF, IFC)                                                        │
│  □ BIM-Integration                                                              │
│  □ Schnittstelle zu Bauämtern                                                   │
│  □ Automatische BAFA-Meldung                                                    │
│                                                                                  │
│  PHASE 5: MARKTPLATZ & ÖKOSYSTEM                                                │
│  ════════════════════════════════                                               │
│  □ Handwerker-Marktplatz                                                        │
│  □ Produkt-Datenbank der Hersteller                                             │
│  □ Finanzierungs-Partner                                                        │
│  □ White-Label-Lösung                                                           │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 1. KI-Assistent

### Funktionsbeschreibung

Der KI-Assistent unterstützt Energieberater bei der täglichen Arbeit durch intelligente Analyse und Empfehlungen.

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           KI-ASSISTENT FEATURES                                  │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  1. AUTOMATISCHE DATENEXTRAKTION                                                │
│     ━━━━━━━━━━━━━━━━━━━━━━━━━━━                                                 │
│     • Fotos → Bauteil-Erkennung (Fenster, Heizung, etc.)                        │
│     • Pläne → Flächen- und Maßextraktion                                        │
│     • Rechnungen → Verbrauchsdaten-Extraktion                                   │
│     • Protokolle → Anlagendaten-Übernahme                                       │
│                                                                                  │
│  2. INTELLIGENTE EMPFEHLUNGEN                                                   │
│     ━━━━━━━━━━━━━━━━━━━━━━━━━                                                   │
│     • Optimale Sanierungsreihenfolge                                            │
│     • Wirtschaftlichste Maßnahmen                                               │
│     • Fördermittel-Optimierung                                                  │
│     • Wärmebrücken-Erkennung in Fotos                                           │
│                                                                                  │
│  3. CHAT-INTERFACE                                                              │
│     ━━━━━━━━━━━━━━━                                                             │
│     • Normen-Fragen beantworten                                                 │
│     • Berechnungen erklären                                                     │
│     • Dokumentation automatisch erstellen                                       │
│     • Kundenanfragen vorbereiten                                                │
│                                                                                  │
│  4. QUALITÄTSSICHERUNG                                                          │
│     ━━━━━━━━━━━━━━━━━━                                                          │
│     • Plausibilitätsprüfung aller Eingaben                                      │
│     • Vergleich mit ähnlichen Projekten                                         │
│     • Fehler-Erkennung vor Export                                               │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Technische Umsetzung

```typescript
interface KIAssistentKonfiguration {
  // MCP-Server Integration
  mcp_server: {
    name: 'energieberater-ki';
    tools: MCPTool[];
    resources: MCPResource[];
  };

  // Modelle
  modelle: {
    chat: 'claude-3-5-sonnet';           // Für Chat und Empfehlungen
    vision: 'claude-3-5-sonnet';         // Für Bild-Analyse
    extraction: 'claude-3-5-haiku';      // Für schnelle Extraktion
  };

  // Kontext-Fenster
  kontext: {
    max_tokens: 200000;
    projekt_kontext: boolean;            // Aktuelles Projekt mitgeben
    normen_kontext: boolean;             // Relevante Normen einbetten
    historie_tage: 30;                   // Projekt-Historie
  };
}

// MCP Tools für den Energieberater-Assistenten
const MCP_TOOLS: MCPTool[] = [
  {
    name: 'berechne_u_wert',
    description: 'Berechnet den U-Wert eines Bauteils basierend auf Schichtaufbau',
    input_schema: {
      type: 'object',
      properties: {
        schichten: {
          type: 'array',
          items: {
            type: 'object',
            properties: {
              material: { type: 'string' },
              dicke_mm: { type: 'number' }
            }
          }
        }
      },
      required: ['schichten']
    }
  },
  {
    name: 'suche_foerderung',
    description: 'Sucht passende Förderprogramme für eine Maßnahme',
    input_schema: {
      type: 'object',
      properties: {
        massnahme_typ: { type: 'string' },
        bundesland: { type: 'string' },
        gebaeude_typ: { type: 'string' }
      },
      required: ['massnahme_typ']
    }
  },
  {
    name: 'schaetze_kosten',
    description: 'Schätzt Kosten für eine Sanierungsmaßnahme',
    input_schema: {
      type: 'object',
      properties: {
        massnahme_typ: { type: 'string' },
        flaeche_m2: { type: 'number' },
        qualitaet: { type: 'string', enum: ['standard', 'premium'] }
      },
      required: ['massnahme_typ', 'flaeche_m2']
    }
  },
  {
    name: 'analysiere_foto',
    description: 'Analysiert ein Foto und extrahiert relevante Gebäudedaten',
    input_schema: {
      type: 'object',
      properties: {
        foto_base64: { type: 'string' },
        fokus: {
          type: 'string',
          enum: ['fassade', 'fenster', 'dach', 'heizung', 'thermografie']
        }
      },
      required: ['foto_base64']
    }
  }
];
```

### Beispiel-Interaktionen

```markdown
## Beispiel 1: Bauteil-Identifikation

**Nutzer lädt Foto hoch**

**KI-Assistent:**
"Ich habe das Foto analysiert und folgende Bauteile erkannt:

1. **Fenster** (5 Stück sichtbar)
   - Geschätztes Alter: 1995-2005 (Isolierglas, Kunststoffrahmen)
   - Geschätzter U-Wert: 1.6-2.0 W/(m²·K)
   - Zustand: Mäßig (Dichtungen sollten geprüft werden)

2. **Fassade**
   - Typ: Verputztes Mauerwerk ohne Dämmung
   - Geschätzte Wandstärke: 36cm
   - Geschätzter U-Wert: 1.2-1.5 W/(m²·K)

Soll ich diese Werte in das Projekt übernehmen?"

---

## Beispiel 2: Förder-Optimierung

**Nutzer:** "Welche Förderung bekomme ich für eine Wärmepumpe?"

**KI-Assistent:**
"Basierend auf den Projektdaten (EFH, Baujahr 1978, Gasheizung):

**BAFA-Förderung für Wärmepumpe:**
- Basis-Förderung: 30% (max. 30.000€)
- Effizienz-Bonus: +5% (natürliches Kältemittel)
- Heizungs-Tausch-Bonus: +20% (Austausch Gas/Öl)
- **Gesamt: 55%** der förderfähigen Kosten

**Geschätzte Kosten und Förderung:**
- Luft-Wasser-WP inkl. Installation: ca. 28.000€
- Förderung (55%): ca. 15.400€
- **Eigenanteil: ca. 12.600€**

⚠️ **Hinweis:** Die Förderung muss VOR Auftragsvergabe beantragt werden!

Soll ich die Förderdetails in die Wirtschaftlichkeitsberechnung übernehmen?"
```

### Kosten-Modell für KI-Modul

```typescript
interface KIModulPreise {
  // Als Zusatzmodul für Organisationen
  basis: {
    name: 'KI-Assistent Basis';
    preis_monat: 29;
    features: [
      'Chat-Assistent für Fragen',
      '50 Foto-Analysen/Monat',
      'Förder-Empfehlungen'
    ];
  };

  pro: {
    name: 'KI-Assistent Pro';
    preis_monat: 79;
    features: [
      'Alles aus Basis',
      'Unbegrenzte Foto-Analysen',
      'Automatische Datenextraktion',
      'OCR für Dokumente',
      'Individuelle Trainings'
    ];
  };

  // Token-basierte Abrechnung als Alternative
  pay_per_use: {
    chat_pro_token: 0.00002;
    bild_analyse: 0.10;
    dokument_extraktion: 0.25;
  };
}
```

---

## 2. Automatische Dokumentenerkennung (OCR)

### Unterstützte Dokumenttypen

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        DOKUMENT-ERKENNUNG                                        │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  RECHNUNGEN & ABRECHNUNGEN                                                      │
│  ═════════════════════════                                                      │
│  • Heizkostenabrechnung → Verbrauchsdaten                                       │
│  • Stromrechnung → Stromverbrauch, PV-Einspeisung                               │
│  • Gasrechnung → Gasverbrauch in kWh                                            │
│  • Öllieferung → Ölverbrauch in Litern                                          │
│                                                                                  │
│  TECHNISCHE DOKUMENTE                                                           │
│  ════════════════════                                                           │
│  • Schornsteinfeger-Protokoll → Kesseltyp, Wirkungsgrad                         │
│  • Wartungsprotokolle → Anlagendaten                                            │
│  • Heizlast-Berechnungen → Vorhandene Werte                                     │
│  • Energieausweise (alt) → Vergleichswerte                                      │
│                                                                                  │
│  BAUPLÄNE & ZEICHNUNGEN                                                         │
│  ═══════════════════════                                                        │
│  • Grundrisse → Raumflächen, Maße                                               │
│  • Schnitte → Geschosshöhen                                                     │
│  • Ansichten → Fensterflächen, Proportionen                                     │
│                                                                                  │
│  PRODUKTDATENBLÄTTER                                                            │
│  ════════════════════                                                           │
│  • Fenster-Zertifikate → U_w, g-Wert                                            │
│  • Dämmstoffe → Lambda, Dicke                                                   │
│  • Heizgeräte → Leistung, Wirkungsgrad                                          │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Technische Umsetzung

```typescript
interface OCRPipeline {
  // Schritt 1: Dokumenten-Klassifikation
  klassifikation: {
    modell: 'document-classifier';
    klassen: [
      'heizkostenabrechnung',
      'stromrechnung',
      'gasrechnung',
      'schornsteinfeger',
      'energieausweis',
      'grundriss',
      'datenblatt',
      'unbekannt'
    ];
  };

  // Schritt 2: Layout-Erkennung
  layout: {
    modell: 'layout-parser';
    features: ['tabellen', 'listen', 'formulare', 'diagramme'];
  };

  // Schritt 3: Text-Extraktion
  ocr: {
    engine: 'tesseract' | 'google-vision' | 'azure-ocr';
    sprachen: ['de', 'en'];
    konfidenz_schwelle: 0.85;
  };

  // Schritt 4: Strukturierte Extraktion
  extraktion: {
    modell: 'claude-3-5-haiku';
    schema_pro_dokumenttyp: Record<string, JSONSchema>;
  };
}

// Beispiel: Heizkostenabrechnung-Schema
const HEIZKOSTENABRECHNUNG_SCHEMA: JSONSchema = {
  type: 'object',
  properties: {
    abrechnungszeitraum: {
      type: 'object',
      properties: {
        von: { type: 'string', format: 'date' },
        bis: { type: 'string', format: 'date' }
      }
    },
    gesamtverbrauch: {
      type: 'object',
      properties: {
        heizung_kwh: { type: 'number' },
        warmwasser_kwh: { type: 'number' },
        gesamt_kwh: { type: 'number' }
      }
    },
    einheiten: {
      type: 'object',
      properties: {
        heizung_einheiten: { type: 'number' },
        warmwasser_m3: { type: 'number' }
      }
    },
    kosten: {
      type: 'object',
      properties: {
        heizung_euro: { type: 'number' },
        warmwasser_euro: { type: 'number' },
        gesamt_euro: { type: 'number' }
      }
    },
    energietraeger: {
      type: 'string',
      enum: ['erdgas', 'heizoel', 'fernwaerme', 'strom', 'pellets']
    },
    wohnflaeche_m2: { type: 'number' }
  }
};

// Verarbeitungs-Workflow
async function verarbeiteHochgeladenesDokument(
  file: File,
  projektId: string
): Promise<DokumentVerarbeitungsErgebnis> {
  // 1. Klassifizieren
  const klasse = await klassifiziereDokument(file);

  // 2. OCR durchführen
  const text = await extrahiereText(file);

  // 3. Strukturiert extrahieren
  const schema = SCHEMAS_PRO_KLASSE[klasse];
  const daten = await extrahiereStrukturiert(text, schema);

  // 4. Validieren
  const validierung = await validiereDaten(daten, klasse);

  // 5. In Projekt übernehmen (mit Bestätigung)
  return {
    dokument_typ: klasse,
    extrahierte_daten: daten,
    konfidenz: validierung.konfidenz,
    vorschau: erstelleVorschau(daten),
    aktionen: [
      {
        label: 'In Projekt übernehmen',
        action: () => uebernehmeDaten(projektId, daten)
      },
      {
        label: 'Manuell bearbeiten',
        action: () => oeffneEditor(daten)
      }
    ]
  };
}
```

---

## 3. Förderungsoptimierung

### Förderdatenbank

```typescript
interface Foerderprogramm {
  id: string;
  name: string;
  geber: 'bafa' | 'kfw' | 'land' | 'kommune';
  bundesland?: string;
  zielgruppe: ('privat' | 'gewerbe' | 'kommunen')[];
  massnahmen: MassnahmenTyp[];
  foerderart: 'zuschuss' | 'kredit' | 'tilgungszuschuss';
  foerdersatz: {
    basis_prozent: number;
    max_betrag?: number;
    min_betrag?: number;
    boni?: {
      name: string;
      prozent: number;
      bedingung: string;
    }[];
  };
  anforderungen: {
    technisch: string[];
    formell: string[];
    fristen: {
      antrag_vor_beginn: boolean;
      verwendungsnachweis_monate: number;
    };
  };
  kombinierbar: string[];           // IDs kombinierbarer Programme
  nicht_kombinierbar: string[];
  gueltig_von: string;
  gueltig_bis?: string;
  link: string;
}

// Beispiel: BAFA Heizungsförderung 2024
const BAFA_HEIZUNG_2024: Foerderprogramm = {
  id: 'bafa-heizung-2024',
  name: 'Bundesförderung für effiziente Gebäude - Einzelmaßnahmen (BEG EM)',
  geber: 'bafa',
  zielgruppe: ['privat', 'gewerbe'],
  massnahmen: ['waermepumpe', 'solarthermie', 'biomasseheizung', 'brennstoffzelle'],
  foerderart: 'zuschuss',
  foerdersatz: {
    basis_prozent: 30,
    max_betrag: 30000,
    boni: [
      {
        name: 'Effizienz-Bonus',
        prozent: 5,
        bedingung: 'Wärmepumpe mit natürlichem Kältemittel oder Erdwärme/Wasser'
      },
      {
        name: 'Klimageschwindigkeits-Bonus',
        prozent: 20,
        bedingung: 'Austausch funktionierende Gas/Öl/Kohle/Nachtspeicher-Heizung'
      },
      {
        name: 'Einkommens-Bonus',
        prozent: 30,
        bedingung: 'Haushaltseinkommen unter 40.000€/Jahr'
      }
    ]
  },
  anforderungen: {
    technisch: [
      'Wärmepumpe: JAZ ≥ 3.0',
      'Biomasse: η_N ≥ 0.81 (Scheitholz) / 0.86 (Pellets)',
      'Einbau durch Fachunternehmen',
      'Hydraulischer Abgleich Verfahren B'
    ],
    formell: [
      'Registrierung auf BAFA-Portal',
      'Liegenschaftsnachweis',
      'Kostenvoranschläge'
    ],
    fristen: {
      antrag_vor_beginn: true,
      verwendungsnachweis_monate: 36
    }
  },
  kombinierbar: ['kfw-261', 'kfw-262'],
  nicht_kombinierbar: ['bafa-heizung-alt'],
  gueltig_von: '2024-01-01',
  link: 'https://www.bafa.de/beg-em'
};
```

### Optimierungs-Algorithmus

```typescript
interface FoerderOptimierung {
  massnahmen: Massnahme[];
  kunde: {
    typ: 'privat' | 'gewerbe';
    bundesland: string;
    einkommen_unter_40k?: boolean;
    wohneigentum: boolean;
  };
  gebaeude: {
    baujahr: number;
    typ: string;
    aktuelle_heizung: string;
    denkmalschutz: boolean;
  };
}

interface OptimierungsErgebnis {
  varianten: FoerderVariante[];
  empfehlung: FoerderVariante;
  hinweise: string[];
}

interface FoerderVariante {
  name: string;
  programme: {
    programm: Foerderprogramm;
    massnahmen: string[];
    foerderbetrag: number;
    bedingungen_erfuellt: boolean;
    fehlende_bedingungen: string[];
  }[];
  gesamt_foerderung: number;
  gesamt_investition: number;
  eigenanteil: number;
  jaehrliche_einsparung: number;
  amortisation_jahre: number;
}

async function optimiereFoerderung(
  input: FoerderOptimierung
): Promise<OptimierungsErgebnis> {
  // 1. Passende Programme filtern
  const passendeProgramme = await filtereProgramme(
    input.massnahmen,
    input.kunde,
    input.gebaeude
  );

  // 2. Kombinationen generieren
  const kombinationen = generiereKombinationen(
    passendeProgramme,
    input.massnahmen
  );

  // 3. Jede Kombination bewerten
  const varianten: FoerderVariante[] = [];

  for (const kombi of kombinationen) {
    const variante = berechneVariante(kombi, input);
    if (variante.gesamt_foerderung > 0) {
      varianten.push(variante);
    }
  }

  // 4. Nach Gesamtförderung sortieren
  varianten.sort((a, b) => b.gesamt_foerderung - a.gesamt_foerderung);

  // 5. Empfehlung ermitteln (beste Kombination aus Förderung und Komplexität)
  const empfehlung = ermittleEmpfehlung(varianten);

  return {
    varianten: varianten.slice(0, 5),  // Top 5
    empfehlung,
    hinweise: generiereHinweise(empfehlung, input)
  };
}

function generiereHinweise(
  empfehlung: FoerderVariante,
  input: FoerderOptimierung
): string[] {
  const hinweise: string[] = [];

  // Einkommens-Bonus prüfen
  if (input.kunde.typ === 'privat' && input.kunde.einkommen_unter_40k === undefined) {
    hinweise.push(
      '💡 Bei Haushaltseinkommen unter 40.000€/Jahr ist ein zusätzlicher Einkommens-Bonus von 30% möglich.'
    );
  }

  // Klimabonus prüfen
  if (input.gebaeude.aktuelle_heizung.includes('gas') ||
      input.gebaeude.aktuelle_heizung.includes('öl')) {
    hinweise.push(
      '💡 Beim Austausch der bestehenden Gas-/Ölheizung erhalten Sie den Klimageschwindigkeits-Bonus (+20%).'
    );
  }

  // Denkmalschutz
  if (input.gebaeude.denkmalschutz) {
    hinweise.push(
      '⚠️ Bei Denkmalschutz können abweichende Anforderungen gelten. Abstimmung mit Denkmalbehörde erforderlich.'
    );
  }

  // Frist-Hinweis
  hinweise.push(
    '⚠️ Wichtig: Förderantrag muss VOR Auftragsvergabe gestellt werden!'
  );

  return hinweise;
}
```

---

## 4. Thermografie-Integration

### Funktionsbeschreibung

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        THERMOGRAFIE-MODUL                                        │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  BILDAUFNAHME                                                                   │
│  ═══════════                                                                    │
│  • Import von Thermografie-Kameras (FLIR, Testo, etc.)                          │
│  • Smartphone-Thermografie (FLIR One, Seek Thermal)                             │
│  • RAW-Format-Unterstützung für genaue Temperaturen                             │
│                                                                                  │
│  AUTOMATISCHE ANALYSE                                                           │
│  ════════════════════                                                           │
│  • Wärmebrücken-Erkennung                                                       │
│  • Leckage-Identifikation                                                       │
│  • Fenster-Qualität bewerten                                                    │
│  • Feuchtigkeit erkennen                                                        │
│                                                                                  │
│  DOKUMENTATION                                                                  │
│  ═════════════                                                                  │
│  • Automatische Beschriftung                                                    │
│  • Temperaturprofile                                                            │
│  • Vergleich IR / Sichtbild                                                     │
│  • Export für Gutachten                                                         │
│                                                                                  │
│  VERKNÜPFUNG MIT BERECHNUNG                                                     │
│  ═══════════════════════════                                                    │
│  • Wärmebrückenzuschlag aus Thermografie ableiten                               │
│  • Qualitätskontrolle der Eingaben                                              │
│  • Nachher-Dokumentation bei Sanierung                                          │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Technische Integration

```typescript
interface ThermografieAnalyse {
  bild: {
    id: string;
    typ: 'radiometrisch' | 'screenshot';
    aufnahme_datum: Date;
    aussentemperatur: number;
    innentemperatur: number;
    emissionsgrad: number;
    kamera_modell: string;
  };

  analyse: {
    min_temp: number;
    max_temp: number;
    avg_temp: number;
    hotspots: ThermoSpot[];
    coldspots: ThermoSpot[];
    waermebruecken: Waermebruecke[];
  };

  bewertung: {
    qualitaet: 'gut' | 'mittel' | 'schlecht';
    handlungsbedarf: boolean;
    empfehlungen: string[];
  };
}

interface ThermoSpot {
  x: number;              // Position im Bild (%)
  y: number;
  temperatur: number;
  flaeche_prozent: number;
  klassifikation: 'waermebruecke' | 'leckage' | 'feuchtigkeit' | 'normal';
}

async function analysiereThermografieBild(
  bild: ArrayBuffer,
  metadaten: ThermografieMetadaten
): Promise<ThermografieAnalyse> {
  // 1. Radiometrische Daten extrahieren (wenn vorhanden)
  const tempMatrix = await extrahiereTemperaturMatrix(bild);

  // 2. Bildanalyse mit KI
  const kiAnalyse = await analysiereThermoBild(bild, {
    kontext: 'gebaeude_fassade',
    fokus: ['waermebruecken', 'fenster', 'anschluesse']
  });

  // 3. Temperaturbereiche analysieren
  const statistik = berechneTemperaturStatistik(tempMatrix);

  // 4. Auffälligkeiten identifizieren
  const spots = identifiziereSpots(tempMatrix, statistik);

  // 5. Wärmebrücken klassifizieren
  const waermebruecken = klassifiziereWaermebruecken(
    spots.coldspots,
    kiAnalyse.erkannte_bauteile
  );

  // 6. Bewertung erstellen
  const bewertung = bewerteFassade(statistik, waermebruecken);

  return {
    bild: {
      id: generateId(),
      typ: tempMatrix ? 'radiometrisch' : 'screenshot',
      aufnahme_datum: new Date(),
      aussentemperatur: metadaten.theta_e,
      innentemperatur: metadaten.theta_i,
      emissionsgrad: metadaten.epsilon ?? 0.95,
      kamera_modell: metadaten.kamera
    },
    analyse: {
      ...statistik,
      hotspots: spots.hotspots,
      coldspots: spots.coldspots,
      waermebruecken
    },
    bewertung
  };
}
```

---

## 5. CAD-Import und BIM-Integration

### Unterstützte Formate

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        CAD/BIM INTEGRATION                                       │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  2D-FORMATE                                                                     │
│  ══════════                                                                     │
│  • DXF/DWG → AutoCAD, BricsCAD, etc.                                            │
│  • PDF → Pläne als PDF (mit Maßstab)                                            │
│  • SVG → Vektorgrafiken                                                         │
│                                                                                  │
│  3D-FORMATE                                                                     │
│  ══════════                                                                     │
│  • IFC (Industry Foundation Classes) → BIM-Standard                             │
│  • gbXML → Energieanalyse-Format                                                │
│  • SketchUp → .skp-Dateien                                                      │
│                                                                                  │
│  EXTRAHIERBARE DATEN                                                            │
│  ════════════════════                                                           │
│  • Raumflächen und Volumina                                                     │
│  • Bauteilflächen nach Orientierung                                             │
│  • Fenster- und Türpositionen                                                   │
│  • Schichtaufbauten (bei IFC)                                                   │
│  • Zonierung (bei gbXML)                                                        │
│                                                                                  │
│  EXPORT                                                                         │
│  ══════                                                                         │
│  • IFC mit Energiedaten angereichert                                            │
│  • gbXML für Energiesimulation                                                  │
│  • PDF-Dokumentation mit 3D-Ansichten                                           │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### IFC-Import Workflow

```typescript
interface IFCImportErgebnis {
  projekt: {
    name: string;
    adresse: string;
    autor: string;
  };

  gebaeude: {
    stockwerke: IFCStockwerk[];
    A_BGF: number;
    V_brutto: number;
  };

  bauteile: IFCBauteil[];

  zonen: IFCZone[];

  warnungen: string[];
}

interface IFCBauteil {
  ifc_id: string;
  ifc_typ: string;                    // IfcWall, IfcWindow, etc.
  name: string;
  flaeche: number;
  orientierung?: Orientation;
  schichten?: IFCSchicht[];
  eigenschaften: Record<string, any>;
}

async function importiereIFC(
  datei: File
): Promise<IFCImportErgebnis> {
  // 1. IFC parsen
  const ifcModel = await parseIFC(datei);

  // 2. Gebäude-Hierarchie aufbauen
  const gebaeude = extrahiereGebaeudeStruktur(ifcModel);

  // 3. Bauteile extrahieren
  const bauteile = extrahiereBauteile(ifcModel);

  // 4. Thermische Eigenschaften wenn vorhanden
  for (const bauteil of bauteile) {
    const thermalProps = findeThermalProperties(ifcModel, bauteil.ifc_id);
    if (thermalProps) {
      bauteil.U_wert = thermalProps.U;
      bauteil.schichten = thermalProps.layers;
    }
  }

  // 5. Flächen berechnen
  berechneFlaechen(gebaeude, bauteile);

  // 6. Mapping zu EnergieberaterPRO-Datenmodell
  return mappeZuProjekt(gebaeude, bauteile);
}
```

---

## 6. Handwerker-Marktplatz

### Konzept

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        HANDWERKER-MARKTPLATZ                                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  FÜR ENERGIEBERATER                                                             │
│  ══════════════════                                                             │
│  • Qualifizierte Handwerker nach Gewerk finden                                  │
│  • Angebote direkt aus der Berechnung anfordern                                 │
│  • Bewertungen und Referenzen einsehen                                          │
│  • Koordination der Gewerke                                                     │
│                                                                                  │
│  FÜR HANDWERKER                                                                 │
│  ═══════════════                                                                │
│  • Qualifizierte Anfragen erhalten                                              │
│  • Detaillierte Leistungsbeschreibung vorab                                     │
│  • Keine Kaltakquise nötig                                                      │
│  • Eigene Referenzen präsentieren                                               │
│                                                                                  │
│  GEWERKE                                                                        │
│  ═══════                                                                        │
│  • Heizung/Sanitär (Wärmepumpe, Solar, etc.)                                    │
│  • Dachdecker (Dämmung, PV-Montage)                                             │
│  • Fensterbau (Fensteraustausch)                                                │
│  • Maler/Fassade (WDVS, Putz)                                                   │
│  • Elektro (PV-Anschluss, Wallbox)                                              │
│  • Zimmerei (Dachausbau, Holzbau)                                               │
│                                                                                  │
│  GESCHÄFTSMODELL                                                                │
│  ══════════════                                                                 │
│  • Handwerker: Provision bei vermitteltem Auftrag (3-5%)                        │
│  • Energieberater: Kostenlos                                                    │
│  • Premium-Listung für Handwerker (monatlich)                                   │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Produkt-Datenbank der Hersteller

### Datenbank-Struktur

```typescript
interface ProduktDatenbank {
  kategorien: {
    fenster: FensterProdukt[];
    daemmstoffe: DaemmstoffProdukt[];
    heizsysteme: HeizsystemProdukt[];
    lueftung: LueftungsProdukt[];
    solarthermie: SolarthermieProdukt[];
    photovoltaik: PVProdukt[];
  };
}

interface FensterProdukt {
  id: string;
  hersteller: string;
  produktname: string;
  serie: string;

  // Technische Daten
  U_w: number;                        // W/(m²·K)
  U_g: number;                        // Verglasung
  U_f: number;                        // Rahmen
  g_wert: number;
  psi_g: number;                      // Glasrandverbund
  rahmenanteil: number;

  // Optionen
  verglasung_optionen: {
    zweifach: { U_g: number; g: number };
    dreifach: { U_g: number; g: number };
  };

  rahmenmaterial: 'kunststoff' | 'holz' | 'alu' | 'holz_alu';

  // Zertifizierung
  ce_kennzeichnung: boolean;
  passivhaus_zertifikat?: 'phA' | 'phB' | 'phC';

  // Preise (Richtpreise)
  preis_ab: number;                   // €/m²
  preis_bis: number;

  // Dokumentation
  datenblatt_url: string;
  einbauanleitung_url?: string;
}

interface DaemmstoffProdukt {
  id: string;
  hersteller: string;
  produktname: string;

  // Technische Daten
  lambda: number;                     // W/(m·K)
  dicken_mm: number[];               // Verfügbare Dicken
  roh_dichte: number;                // kg/m³
  waermekapazitaet: number;          // J/(kg·K)

  // Brandschutz
  brandklasse: 'A1' | 'A2' | 'B1' | 'B2' | 'C' | 'D' | 'E' | 'F';

  // Feuchteschutz
  diffusionswiderstand: number;      // µ-Wert
  wasseraufnahme: number;            // kg/m²

  // Anwendung
  anwendungen: ('aussenwand' | 'innendaemmung' | 'dach' | 'kellerdecke' | 'perimeter')[];

  // Ökologie
  recycling_faehig: boolean;
  oekobilanz?: {
    gwp: number;                     // Global Warming Potential
    penrt: number;                   // Nicht-erneuerbare Primärenergie
  };

  // Preise
  preis_pro_m2: Record<number, number>;  // Dicke → €/m²

  // Dokumentation
  datenblatt_url: string;
  epd_url?: string;                  // Environmental Product Declaration
}

interface HeizsystemProdukt {
  id: string;
  hersteller: string;
  produktname: string;
  typ: WaermeerzeugerTyp;

  // Leistung
  P_nenn_kW: number;
  P_min_kW?: number;                 // Modulationsbereich
  P_max_kW?: number;

  // Effizienz
  eta_s?: number;                    // Jahreszeitbedingte Raumheizungs-Energieeffizienz
  cop_a7w35?: number;                // Wärmepumpe: A7/W35
  cop_a2w35?: number;                // Wärmepumpe: A2/W35
  cop_b0w35?: number;                // Sole-WP: B0/W35
  scop?: number;                     // Seasonal COP
  eta_ww?: number;                   // Warmwasserbereitung

  // Schall
  schallleistung_db?: number;
  schallleistung_innen_db?: number;

  // Kältemittel (Wärmepumpen)
  kaeltemittel?: string;
  gwp_kaeltemittel?: number;

  // Abmessungen
  hoehe_mm: number;
  breite_mm: number;
  tiefe_mm: number;
  gewicht_kg: number;

  // Förderung
  bafa_foerderbar: boolean;
  energielabel: 'A+++' | 'A++' | 'A+' | 'A' | 'B' | 'C' | 'D';

  // Preise
  preis_uvp?: number;
  preis_strasse?: number;

  // Dokumentation
  datenblatt_url: string;
  montageanleitung_url?: string;
}

// API für Produktsuche
async function sucheProdukte(
  kategorie: string,
  filter: ProduktFilter
): Promise<Produkt[]> {
  const query = buildQuery(kategorie, filter);
  return await produktDb.search(query);
}

interface ProduktFilter {
  hersteller?: string[];
  U_max?: number;
  lambda_max?: number;
  P_min?: number;
  P_max?: number;
  foerderbar?: boolean;
  preis_max?: number;
  nur_mit_preis?: boolean;
}
```

---

## 8. Schnittstellen zu Behörden

### BAFA-Schnittstelle

```typescript
interface BAFASchnittstelle {
  // Antragstellung
  antrag: {
    erstellen: (daten: BAFAAntrag) => Promise<string>;  // Antragsnummer
    status: (antragsnummer: string) => Promise<BAFAStatus>;
    verwendungsnachweis: (antragsnummer: string, nachweis: Nachweis) => Promise<void>;
  };

  // Energieberater-Registrierung
  dena: {
    abfrage: (plz: string) => Promise<Energieberater[]>;
    verifizierung: (berater_id: string) => Promise<boolean>;
  };
}

interface BAFAAntrag {
  antragsteller: {
    name: string;
    adresse: Adresse;
    geburtsdatum?: string;
    steuernummer?: string;
  };

  vorhaben: {
    typ: BAFAMassnahmenTyp;
    beschreibung: string;
    beginn_geplant: string;
    ende_geplant: string;
  };

  gebaeude: {
    adresse: Adresse;
    baujahr: number;
    typ: string;
    wohneinheiten: number;
  };

  kosten: {
    gesamt_brutto: number;
    gesamt_netto: number;
    foerderfaehig: number;
  };

  energieberater: {
    dena_id: string;
    name: string;
  };

  dokumente: {
    kostenvoranschlaege: File[];
    energieberatungsbericht?: File;
    liegenschaftsnachweis: File;
  };
}

// Automatischer Export für BAFA-Portal
async function exportiereFuerBAFA(
  projekt: Projekt,
  massnahmen: Massnahme[]
): Promise<BAFAExportPaket> {
  const antrag = erstelleAntragsdaten(projekt, massnahmen);
  const dokumente = sammleErforderlicheDokumente(projekt);

  // Validierung gegen BAFA-Anforderungen
  const validierung = validiereBAFAAnforderungen(antrag);

  if (!validierung.gueltig) {
    throw new Error(`BAFA-Anforderungen nicht erfüllt: ${validierung.fehler.join(', ')}`);
  }

  return {
    antrag,
    dokumente,
    checkliste: erstelleCheckliste(antrag),
    hinweise: generiereBAFAHinweise(antrag)
  };
}
```

### DIBt-Schnittstelle (Energieausweis-Registrierung)

```typescript
interface DIBtSchnittstelle {
  // Energieausweis registrieren
  registrieren: (
    ausweis: Energieausweis,
    aussteller: Aussteller
  ) => Promise<{
    registriernummer: string;
    gueltig_bis: string;
  }>;

  // Registriernummer validieren
  validieren: (registriernummer: string) => Promise<{
    gueltig: boolean;
    ausweis_typ: string;
    ausstellungsdatum: string;
  }>;
}

async function registriereEnergieausweis(
  energieausweis: Energieausweis
): Promise<string> {
  // 1. Vollständigkeit prüfen
  const vollstaendig = pruefeVollstaendigkeit(energieausweis);
  if (!vollstaendig.ok) {
    throw new Error(`Energieausweis unvollständig: ${vollstaendig.fehlend.join(', ')}`);
  }

  // 2. Beim DIBt registrieren
  const registrierung = await dibt.registrieren(energieausweis, {
    aussteller_id: energieausweis.aussteller.dena_id,
    gebaeude_adresse: energieausweis.gebaeude.adresse,
    ausweis_typ: energieausweis.typ
  });

  // 3. Registriernummer im Ausweis speichern
  energieausweis.registriernummer = registrierung.registriernummer;
  energieausweis.gueltig_bis = registrierung.gueltig_bis;

  return registrierung.registriernummer;
}
```

---

## 9. Mobile App (PWA-Erweiterung)

### Offline-Funktionen

```typescript
interface MobileFeatures {
  // Vor-Ort-Erfassung
  erfassung: {
    foto_mit_metadaten: boolean;      // GPS, Kompass, Datum
    sprachnotizen: boolean;
    schnellerfassung: boolean;        // Vordefinierte Templates
    offline_sync: boolean;
  };

  // Kamera-Features
  kamera: {
    bauteil_erkennung: boolean;
    mass_schaetzung: boolean;         // AR-basiert
    thermografie: boolean;            // Mit Zusatzgerät
  };

  // Sync
  synchronisation: {
    automatisch: boolean;
    manuell: boolean;
    konfliktloesung: 'server' | 'lokal' | 'manuell';
  };
}

// Service Worker für Offline-Betrieb
const OFFLINE_CACHE = {
  // Statische Assets
  static: [
    '/app/shell.html',
    '/app/main.js',
    '/app/styles.css',
    '/app/icons/*'
  ],

  // Datenbank für Offline-Daten
  indexedDB: {
    projekte: 'projekte_offline',
    bilder: 'bilder_offline',
    berechnungen: 'berechnungen_cache'
  },

  // Materialdatenbank cachen
  materialdb: {
    version: '2024.1',
    groesse_mb: 15
  }
};

// Synchronisation bei Verbindungsherstellung
async function synchronisiereOfflineDaten(): Promise<SyncErgebnis> {
  const pendingChanges = await db.getPendingChanges();

  const ergebnisse: SyncErgebnis = {
    hochgeladen: 0,
    konflikte: [],
    fehler: []
  };

  for (const change of pendingChanges) {
    try {
      if (change.typ === 'create') {
        await api.create(change.entity, change.data);
      } else if (change.typ === 'update') {
        const serverVersion = await api.getVersion(change.entity, change.id);

        if (serverVersion > change.basis_version) {
          // Konflikt!
          ergebnisse.konflikte.push({
            entity: change.entity,
            id: change.id,
            lokal: change.data,
            server: await api.get(change.entity, change.id)
          });
        } else {
          await api.update(change.entity, change.id, change.data);
        }
      }

      ergebnisse.hochgeladen++;
      await db.markSynced(change.id);
    } catch (error) {
      ergebnisse.fehler.push({
        change,
        error: error.message
      });
    }
  }

  return ergebnisse;
}
```

---

## 10. White-Label-Lösung

### Konfigurierbare Elemente

```typescript
interface WhiteLabelConfig {
  // Branding
  branding: {
    firmenname: string;
    logo_url: string;
    favicon_url: string;
    primary_color: string;
    secondary_color: string;
    schriftart?: string;
  };

  // Domain
  domain: {
    custom_domain: string;            // z.B. "portal.ihre-firma.de"
    ssl_zertifikat: 'managed' | 'custom';
  };

  // Features
  features: {
    modul_energieausweis: boolean;
    modul_isfp: boolean;
    modul_heizlast: boolean;
    modul_ki: boolean;
    marktplatz: boolean;
    api_zugang: boolean;
  };

  // E-Mail
  email: {
    absender_name: string;
    absender_email: string;
    reply_to: string;
    smtp_custom?: SMTPConfig;
  };

  // Dokumente
  dokumente: {
    briefkopf_template?: string;
    fusszeile_template?: string;
    eigene_agb_url?: string;
    eigene_datenschutz_url?: string;
  };

  // Preise (für Reseller)
  preise: {
    eigene_preise: boolean;
    aufschlag_prozent?: number;
    eigene_plaene?: PreisPlan[];
  };
}

// White-Label Deployment
interface WhiteLabelDeployment {
  tenant_id: string;
  config: WhiteLabelConfig;
  status: 'aktiv' | 'einrichtung' | 'pausiert';

  // DNS-Konfiguration
  dns: {
    cname_ziel: string;              // z.B. "tenant123.energieberaterpro.de"
    ssl_status: 'pending' | 'issued' | 'error';
  };

  // Isolierung
  daten_isolierung: 'shared' | 'dedicated';
  backup_retention_tage: number;
}
```

---

## Priorisierungs-Matrix

| Erweiterung | Aufwand | Impact | Umsatz | Priorität |
|-------------|---------|--------|--------|-----------|
| KI-Assistent | Hoch | Sehr hoch | Hoch | 1 |
| Förderungsoptimierung | Mittel | Sehr hoch | Mittel | 2 |
| OCR Dokumentenerkennung | Mittel | Hoch | Mittel | 3 |
| Mobile App (PWA) | Mittel | Hoch | Mittel | 4 |
| Thermografie | Mittel | Mittel | Mittel | 5 |
| Handwerker-Marktplatz | Hoch | Hoch | Hoch | 6 |
| CAD/BIM-Import | Hoch | Mittel | Niedrig | 7 |
| Produkt-Datenbank | Hoch | Mittel | Mittel | 8 |
| BAFA-Schnittstelle | Mittel | Mittel | Niedrig | 9 |
| White-Label | Hoch | Niedrig | Hoch | 10 |

---

## Technische Voraussetzungen

### Infrastruktur-Anforderungen

```yaml
# Erweiterungen benötigen zusätzliche Services

ki_assistent:
  - claude_api_key
  - gpu_server: optional    # Für lokale Modelle
  - embedding_db: pgvector  # Für RAG

ocr:
  - tesseract: 5.x
  - vision_api: google | azure | aws
  - storage: s3_compatible

thermografie:
  - flir_sdk: optional
  - image_processing: sharp | jimp

cad_import:
  - ifc_parser: ifc.js | xeokit
  - dxf_parser: dxf-parser
  - pdf_renderer: pdf.js

marktplatz:
  - payment_gateway: stripe | mollie
  - messaging: websocket
  - search: elasticsearch | meilisearch

whitelabel:
  - wildcard_ssl: letsencrypt
  - cdn: cloudflare | fastly
  - multi_tenant_db: dedicated | shared
```

---

## Fazit

Die geplanten Erweiterungen bauen auf dem Kern-System auf und erweitern EnergieberaterPRO zu einer umfassenden Plattform für alle Aspekte der Energieberatung. Die Priorisierung fokussiert zunächst auf Features mit hohem Kundennutzen (KI, Förderung) und baut dann das Ökosystem aus (Marktplatz, Hersteller).
