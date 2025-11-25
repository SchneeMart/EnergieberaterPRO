# Produkte, Dienstleistungen und Workflows

## Übersicht: Alle Produkte der Plattform

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    PRODUKTKATALOG                                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ENERGIEAUSWEISE                                                             │
│  ├── Bedarfsausweis Wohngebäude                                             │
│  ├── Verbrauchsausweis Wohngebäude                                          │
│  ├── Bedarfsausweis Nichtwohngebäude                                        │
│  ├── Verbrauchsausweis Nichtwohngebäude                                     │
│  └── Energieausweis Österreich (nach OIB)                                   │
│                                                                              │
│  ENERGIEBERATUNG                                                             │
│  ├── iSFP (Individueller Sanierungsfahrplan)                                │
│  ├── Vor-Ort-Beratung (BAFA-gefördert)                                      │
│  ├── Heizungsberatung                                                       │
│  └── Initialberatung                                                        │
│                                                                              │
│  ENERGIEAUDITS                                                               │
│  ├── Energieaudit nach DIN EN 16247 (Unternehmen)                          │
│  ├── Energieaudit nach EDL-G                                                │
│  └── ISO 50001 Begleitung                                                   │
│                                                                              │
│  BERECHNUNGEN                                                                │
│  ├── Heizlastberechnung (DIN EN 12831)                                     │
│  ├── Wärmebrückenberechnung                                                 │
│  ├── Lüftungskonzept (DIN 1946-6)                                          │
│  ├── Wirtschaftlichkeitsberechnung                                          │
│  ├── CO₂-Bilanzierung                                                       │
│  └── Photovoltaik-Simulation                                                │
│                                                                              │
│  NACHWEISE                                                                   │
│  ├── GEG-Nachweis (Neubau)                                                  │
│  ├── GEG-Nachweis (Bestand/Sanierung)                                       │
│  ├── KfW-Effizienzhaus-Nachweis                                             │
│  ├── BAFA-Verwendungsnachweis                                               │
│  └── QNG-Zertifizierung (Nachhaltigkeit)                                    │
│                                                                              │
│  FÖRDERMITTEL                                                                │
│  ├── Fördermittelberatung                                                   │
│  ├── Antragsstellung BEG                                                    │
│  ├── Technische Projektbeschreibung (TPB)                                   │
│  └── Bestätigung nach Durchführung (BnD)                                    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

# Detaillierte Workflow-Dokumentation

## 1. Energieausweis Wohngebäude (Bedarfsausweis)

### 1.1 Übersicht

```
Produkt:        Energiebedarfsausweis für Wohngebäude
Rechtsgrundlage: GEG § 80-86, Anlage 1
Norm:           DIN V 18599 oder DIN 4108-6/4701-10
Gültigkeit:     10 Jahre
Zielgruppe:     Eigentümer, Verkäufer, Vermieter
```

### 1.2 Workflow-Diagramm

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    WORKFLOW: ENERGIEAUSWEIS WOHNGEBÄUDE                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────┐                                                            │
│  │   START     │                                                            │
│  └──────┬──────┘                                                            │
│         │                                                                    │
│         ▼                                                                    │
│  ┌─────────────────────────────────────────────────┐                        │
│  │ PHASE 1: PROJEKTANLAGE                          │                        │
│  ├─────────────────────────────────────────────────┤                        │
│  │ □ Kunde anlegen/auswählen                       │                        │
│  │ □ Gebäudeadresse erfassen                       │                        │
│  │ □ Projekttyp: Energieausweis Bedarf WG         │                        │
│  │ □ Anlass: Verkauf/Vermietung/Neubau/Sanierung  │                        │
│  │ □ Berechnungsverfahren wählen (18599 / 4108-6) │                        │
│  └──────────────────────┬──────────────────────────┘                        │
│                         │                                                    │
│                         ▼                                                    │
│  ┌─────────────────────────────────────────────────┐                        │
│  │ PHASE 2: DATENERFASSUNG GEBÄUDE                 │                        │
│  ├─────────────────────────────────────────────────┤                        │
│  │ □ Gebäudetyp (EFH/MFH/RH/DHH)                   │                        │
│  │ □ Baujahr                                       │                        │
│  │ □ Anzahl Wohneinheiten                          │                        │
│  │ □ Wohnfläche (nach WoFlV)                       │                        │
│  │ □ Nutzfläche AN (nach GEG)                      │                        │
│  │ □ Anzahl Geschosse                              │                        │
│  │ □ Beheizte/unbeheizte Zonen                     │                        │
│  │ □ Kellersituation                               │                        │
│  │ □ Dachsituation                                 │                        │
│  └──────────────────────┬──────────────────────────┘                        │
│                         │                                                    │
│                         ▼                                                    │
│  ┌─────────────────────────────────────────────────┐                        │
│  │ PHASE 3: GEBÄUDEHÜLLE                           │                        │
│  ├─────────────────────────────────────────────────┤                        │
│  │ □ Außenwände (Aufbau, Fläche, U-Wert)          │                        │
│  │ □ Fenster (Typ, Fläche, Uw-Wert, g-Wert)       │                        │
│  │ □ Türen (Typ, Fläche, UD-Wert)                 │                        │
│  │ □ Dach/oberste Geschossdecke                   │                        │
│  │ □ Kellerdecke/Bodenplatte                      │                        │
│  │ □ Wärmebrücken (pauschal/detailliert)          │                        │
│  │ □ Luftdichtheit (gemessen/Standard)            │                        │
│  └──────────────────────┬──────────────────────────┘                        │
│                         │                                                    │
│                         ▼                                                    │
│  ┌─────────────────────────────────────────────────┐                        │
│  │ PHASE 4: ANLAGENTECHNIK                         │                        │
│  ├─────────────────────────────────────────────────┤                        │
│  │ □ Heizungsanlage (Typ, Baujahr, Leistung)      │                        │
│  │ □ Wärmeverteilung (Rohrleitungen, Dämmung)     │                        │
│  │ □ Wärmeübergabe (Heizkörper, FBH)              │                        │
│  │ □ Trinkwarmwasser (zentral/dezentral)          │                        │
│  │ □ Lüftungsanlage (falls vorhanden)             │                        │
│  │ □ Solaranlage (thermisch/PV)                   │                        │
│  │ □ Weitere Erzeuger (Kamin, etc.)               │                        │
│  └──────────────────────┬──────────────────────────┘                        │
│                         │                                                    │
│                         ▼                                                    │
│  ┌─────────────────────────────────────────────────┐                        │
│  │ PHASE 5: BERECHNUNG                             │                        │
│  ├─────────────────────────────────────────────────┤                        │
│  │ → Automatische Berechnung nach DIN V 18599      │                        │
│  │   oder DIN 4108-6/4701-10                       │                        │
│  │                                                  │                        │
│  │ ERGEBNISSE:                                     │                        │
│  │ □ Endenergiebedarf [kWh/(m²·a)]                │                        │
│  │ □ Primärenergiebedarf [kWh/(m²·a)]             │                        │
│  │ □ CO₂-Emissionen [kg/(m²·a)]                   │                        │
│  │ □ Energieeffizienzklasse (A+ bis H)            │                        │
│  │ □ Anforderungswerte GEG (Vergleich)            │                        │
│  └──────────────────────┬──────────────────────────┘                        │
│                         │                                                    │
│                         ▼                                                    │
│  ┌─────────────────────────────────────────────────┐                        │
│  │ PHASE 6: MODERNISIERUNGSEMPFEHLUNGEN            │                        │
│  ├─────────────────────────────────────────────────┤                        │
│  │ □ KI-generierte Vorschläge (optional)          │                        │
│  │ □ Manuelle Anpassung                            │                        │
│  │ □ Kostenabschätzung                             │                        │
│  │ □ Einsparprognose                               │                        │
│  │ □ Förderhinweise                                │                        │
│  └──────────────────────┬──────────────────────────┘                        │
│                         │                                                    │
│                         ▼                                                    │
│  ┌─────────────────────────────────────────────────┐                        │
│  │ PHASE 7: DOKUMENTGENERIERUNG                    │                        │
│  ├─────────────────────────────────────────────────┤                        │
│  │ □ Energieausweis nach Muster GEG Anlage 6      │                        │
│  │ □ Registrierungsnummer (DIBt)                  │                        │
│  │ □ Digitale Signatur                             │                        │
│  │ □ PDF-Export                                    │                        │
│  │ □ Archivierung                                  │                        │
│  └──────────────────────┬──────────────────────────┘                        │
│                         │                                                    │
│                         ▼                                                    │
│  ┌─────────────┐                                                            │
│  │    ENDE     │                                                            │
│  └─────────────┘                                                            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.3 Datenfelder (Detailliert)

```yaml
# Gebäude-Stammdaten
gebaeude:
  adresse:
    strasse: string (required)
    hausnummer: string (required)
    plz: string (required, pattern: /^\d{5}$/)
    ort: string (required)
    bundesland: enum (required)
    land: enum [DE, AT] (required)

  grunddaten:
    gebaeude_typ: enum [EFH, DHH, RH, MFH] (required)
    baujahr: integer (required, range: 1800-2025)
    baujahr_wesentliche_aenderung: integer (optional)
    anzahl_wohneinheiten: integer (required, min: 1)
    anzahl_vollgeschosse: integer (required, min: 1)
    anzahl_kellergeschosse: integer (default: 0)
    anzahl_dachgeschosse: integer (default: 0)

  flaechen:
    wohnflaeche_woflv: float (required, unit: m²)
    nutzflaeche_an: float (calculated, unit: m²)
    bruttogeschossflaeche: float (optional, unit: m²)
    beheiztes_volumen: float (calculated, unit: m³)
    huellflaeche: float (calculated, unit: m²)

  geometrie:
    laenge: float (optional, unit: m)
    breite: float (optional, unit: m)
    geschosshoehe: float (default: 2.5, unit: m)
    firsthoehe: float (optional, unit: m)
    dachneigung: float (optional, unit: °)
    dachform: enum [Satteldach, Walmdach, Pultdach, Flachdach, ...]

# Gebäudehülle
huellflaechen:
  - typ: enum [Aussenwand, Fenster, Dach, Kellerdecke, ...]
    bezeichnung: string
    flaeche: float (required, unit: m²)
    u_wert: float (required, unit: W/(m²K))
    ausrichtung: enum [N, NO, O, SO, S, SW, W, NW, horizontal]
    bauteil_aufbau:
      - schicht: string
        dicke: float (unit: mm)
        lambda: float (unit: W/(mK))

# Bei Fenstern zusätzlich
fenster:
  - typ: enum [Fenster, Fenstertür, Dachfenster]
    verglasungstyp: enum [2-fach, 3-fach, ...]
    rahmenanteil: float (default: 0.3)
    uw_wert: float (required, unit: W/(m²K))
    g_wert: float (required, range: 0-1)
    verschattung:
      typ: enum [keine, Jalousie, Rollladen, ...]
      fc_wert: float

# Wärmebrücken
waermebruecken:
  ansatz: enum [pauschal, detailliert, gleichwertig]
  delta_uwb: float (unit: W/(m²K))  # bei pauschal
  details:  # bei detailliert
    - typ: string
      laenge: float (unit: m)
      psi_wert: float (unit: W/(mK))

# Luftdichtheit
luftdichtheit:
  n50_gemessen: float (optional, unit: 1/h)
  n50_angesetzt: float (calculated, unit: 1/h)
  blower_door_protokoll: file (optional)

# Anlagentechnik
heizung:
  erzeuger:
    - typ: enum [Gaskessel, Ölkessel, Wärmepumpe, Pellet, ...]
      baujahr: integer
      nennleistung: float (unit: kW)
      aufstellort: enum [beheizt, unbeheizt]
      brennwert: boolean
      modulierend: boolean
      # Wärmepumpe spezifisch
      waermequelle: enum [Luft, Erdreich, Grundwasser] (if Wärmepumpe)
      cop: float (if Wärmepumpe)
      jaz: float (calculated, if Wärmepumpe)

  verteilung:
    rohrleitungen:
      daemmung: enum [keine, mässig, gut, EnEV, GEG]
      lage: enum [im_beheizten, im_unbeheizten]
      laenge_beheizt: float (unit: m)
      laenge_unbeheizt: float (unit: m)

  uebergabe:
    typ: enum [Heizkörper, Fussbodenheizung, Wandheizung, ...]
    auslegungstemperatur: enum [90/70, 70/55, 55/45, 35/28, ...]
    einzelraumregelung: boolean
    hydraulischer_abgleich: boolean

trinkwarmwasser:
  erzeugung: enum [zentral_mit_heizung, zentral_separat, dezentral]
  speicher:
    vorhanden: boolean
    volumen: float (unit: l)
    daemmung: enum [keine, mässig, gut]
  zirkulation: boolean
  solar_unterstuetzung: boolean

lueftung:
  typ: enum [keine, Abluft, ZuAbluft_ohne_WRG, ZuAbluft_mit_WRG]
  waermerueckgewinnung_grad: float (if mit_WRG, unit: %)
  luftwechsel: float (unit: 1/h)

solar:
  solarthermie:
    vorhanden: boolean
    kollektor_typ: enum [Flach, Vakuum]
    kollektor_flaeche: float (unit: m²)
    ausrichtung: enum [S, SO, SW, O, W]
    neigung: float (unit: °)
    verwendung: enum [TWW, TWW_und_Heizung]

  photovoltaik:
    vorhanden: boolean
    peak_leistung: float (unit: kWp)
    ausrichtung: enum
    neigung: float (unit: °)
    eigennutzung_anteil: float (unit: %)
    batteriespeicher: boolean
    speicher_kapazitaet: float (if batteriespeicher, unit: kWh)
```

### 1.4 Berechnungsablauf

```python
# Pseudocode für Energiebedarfsberechnung

def berechne_energieausweis(gebaeude, anlagentechnik):
    """
    Berechnung nach DIN V 18599 (vereinfacht dargestellt)
    """

    # 1. Transmissionswärmeverluste
    HT = berechne_transmissionswaermeverluste(
        huellflaechen=gebaeude.huellflaechen,
        waermebruecken=gebaeude.waermebruecken,
        temperatur_innen=20,  # °C
        temperatur_aussen=klimadaten.mittlere_aussentemperatur
    )

    # 2. Lüftungswärmeverluste
    HV = berechne_lueftungswaermeverluste(
        volumen=gebaeude.beheiztes_volumen,
        luftwechsel=bestimme_luftwechsel(gebaeude.luftdichtheit),
        wrg_grad=anlagentechnik.lueftung.waermerueckgewinnung_grad
    )

    # 3. Heizwärmebedarf (Nutzenergie)
    Qh = (HT + HV) * gradtagszahl - solare_gewinne - interne_gewinne

    # 4. Warmwasserbedarf
    Qw = berechne_warmwasserbedarf(
        nutzflaeche=gebaeude.nutzflaeche_an,
        nutzungsprofil='Wohngebäude'
    )

    # 5. Endenergiebedarf
    Qe = berechne_endenergie(
        nutzenergie_heizen=Qh,
        nutzenergie_tww=Qw,
        anlagentechnik=anlagentechnik
    )

    # 6. Primärenergiebedarf
    Qp = berechne_primaerenergie(
        endenergie=Qe,
        energietraeger=anlagentechnik.energietraeger,
        primaerenergiefaktoren=GEG_ANLAGE_9
    )

    # 7. CO2-Emissionen
    CO2 = berechne_co2_emissionen(
        endenergie=Qe,
        energietraeger=anlagentechnik.energietraeger,
        emissionsfaktoren=GEG_ANLAGE_9
    )

    # 8. Effizienzklasse
    klasse = bestimme_effizienzklasse(Qe.spezifisch)

    return {
        'endenergiebedarf': Qe.spezifisch,  # kWh/(m²·a)
        'primaerenergiebedarf': Qp.spezifisch,  # kWh/(m²·a)
        'co2_emissionen': CO2.spezifisch,  # kg/(m²·a)
        'effizienzklasse': klasse,
        'transmissionswaermeverlust': HT,
        'lueftungswaermeverlust': HV,
        'details': {...}
    }
```

---

## 2. Individueller Sanierungsfahrplan (iSFP)

### 2.1 Übersicht

```
Produkt:        Individueller Sanierungsfahrplan
Rechtsgrundlage: BEG-Richtlinie, iSFP-Richtlinie
Förderung:      80% der Beratungskosten (max. 1.300€ EFH / 1.700€ MFH)
Bonus:          +5% auf Einzelmaßnahmen bei Umsetzung nach iSFP
Zielgruppe:     Eigentümer von Wohngebäuden
```

### 2.2 Workflow-Diagramm

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    WORKFLOW: iSFP                                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────┐                                                            │
│  │   START     │                                                            │
│  └──────┬──────┘                                                            │
│         │                                                                    │
│         ▼                                                                    │
│  ┌─────────────────────────────────────────────────┐                        │
│  │ PHASE 1: AUFTRAGSKLÄRUNG                        │                        │
│  ├─────────────────────────────────────────────────┤                        │
│  │ □ Kundenwünsche erfassen                        │                        │
│  │ □ Budget abklären                               │                        │
│  │ □ Zeitrahmen festlegen                          │                        │
│  │ □ Fördermöglichkeiten erläutern                │                        │
│  │ □ Angebot erstellen                             │                        │
│  │ □ Auftrag bestätigen                            │                        │
│  └──────────────────────┬──────────────────────────┘                        │
│                         │                                                    │
│                         ▼                                                    │
│  ┌─────────────────────────────────────────────────┐                        │
│  │ PHASE 2: BESTANDSAUFNAHME VOR ORT               │                        │
│  ├─────────────────────────────────────────────────┤                        │
│  │ □ Termin vereinbaren                            │                        │
│  │ □ Checkliste vorbereiten                        │                        │
│  │                                                  │                        │
│  │ VOR ORT:                                        │                        │
│  │ □ Gebäudegeometrie aufnehmen                    │                        │
│  │ □ Bauteilaufbauten erfassen                     │                        │
│  │ □ Anlagentechnik dokumentieren                  │                        │
│  │ □ Fotos erstellen (alle Bauteile)              │                        │
│  │ □ Verbrauchsdaten erfragen                      │                        │
│  │ □ Mängel dokumentieren                          │                        │
│  │ □ Kundenwünsche vertiefen                       │                        │
│  └──────────────────────┬──────────────────────────┘                        │
│                         │                                                    │
│                         ▼                                                    │
│  ┌─────────────────────────────────────────────────┐                        │
│  │ PHASE 3: IST-ZUSTAND BERECHNUNG                 │                        │
│  ├─────────────────────────────────────────────────┤                        │
│  │ □ Gebäudedaten erfassen (wie Energieausweis)   │                        │
│  │ □ Bilanzierung Ist-Zustand                      │                        │
│  │ □ Vergleich mit Verbrauchsdaten (Plausibilität)│                        │
│  │ □ Schwachstellenanalyse                         │                        │
│  │                                                  │                        │
│  │ ERGEBNIS:                                       │                        │
│  │ □ Energiebilanz Ist-Zustand                    │                        │
│  │ □ Identifizierte Schwachstellen                │                        │
│  │ □ Einsparpotenziale                             │                        │
│  └──────────────────────┬──────────────────────────┘                        │
│                         │                                                    │
│                         ▼                                                    │
│  ┌─────────────────────────────────────────────────┐                        │
│  │ PHASE 4: MASSNAHMENENTWICKLUNG                  │                        │
│  ├─────────────────────────────────────────────────┤                        │
│  │ □ Sanierungsziel definieren (EH 85, 70, 55, 40)│                        │
│  │ □ Sinnvolle Maßnahmen identifizieren           │                        │
│  │ □ Maßnahmen in Pakete bündeln                  │                        │
│  │ □ Reihenfolge festlegen (technisch sinnvoll)   │                        │
│  │                                                  │                        │
│  │ MASSKATEGORIEN:                                 │                        │
│  │ □ Gebäudehülle (Dämmung, Fenster)              │                        │
│  │ □ Anlagentechnik (Heizung, Lüftung)            │                        │
│  │ □ Erneuerbare Energien (Solar, WP)             │                        │
│  │ □ Sommerlicher Wärmeschutz                     │                        │
│  └──────────────────────┬──────────────────────────┘                        │
│                         │                                                    │
│                         ▼                                                    │
│  ┌─────────────────────────────────────────────────┐                        │
│  │ PHASE 5: VARIANTENBERECHNUNG                    │                        │
│  ├─────────────────────────────────────────────────┤                        │
│  │ FÜR JEDES MAẞNAHMENPAKET:                       │                        │
│  │ □ Energiebilanz berechnen                       │                        │
│  │ □ Einsparung gegenüber Ist-Zustand             │                        │
│  │ □ Investitionskosten schätzen                   │                        │
│  │ □ Förderung ermitteln                           │                        │
│  │ □ Wirtschaftlichkeit berechnen                  │                        │
│  │ □ CO₂-Einsparung                               │                        │
│  │                                                  │                        │
│  │ SANIERUNGSFAHRPLAN:                             │                        │
│  │ □ Schritt 1: [Maßnahmenpaket] → EH XX          │                        │
│  │ □ Schritt 2: [Maßnahmenpaket] → EH XX          │                        │
│  │ □ Schritt 3: [Maßnahmenpaket] → EH XX          │                        │
│  │ □ Endzustand: EH XX erreicht                   │                        │
│  └──────────────────────┬──────────────────────────┘                        │
│                         │                                                    │
│                         ▼                                                    │
│  ┌─────────────────────────────────────────────────┐                        │
│  │ PHASE 6: DOKUMENTERSTELLUNG                     │                        │
│  ├─────────────────────────────────────────────────┤                        │
│  │ Dokument 1: "Mein Sanierungsfahrplan" (Kunde)  │                        │
│  │ □ Zusammenfassung Ist-Zustand                   │                        │
│  │ □ Visualisierung Einsparpotenziale             │                        │
│  │ □ Sanierungsschritte mit Zeitplan              │                        │
│  │ □ Kostenschätzung und Förderung                │                        │
│  │ □ Handlungsempfehlungen                         │                        │
│  │                                                  │                        │
│  │ Dokument 2: "Umsetzungshilfe" (Fachhandwerker) │                        │
│  │ □ Technische Details je Maßnahme               │                        │
│  │ □ Mindestanforderungen                          │                        │
│  │ □ Ausführungshinweise                           │                        │
│  │ □ Nachweispflichten                             │                        │
│  └──────────────────────┬──────────────────────────┘                        │
│                         │                                                    │
│                         ▼                                                    │
│  ┌─────────────────────────────────────────────────┐                        │
│  │ PHASE 7: ABSCHLUSSGESPRÄCH                      │                        │
│  ├─────────────────────────────────────────────────┤                        │
│  │ □ Ergebnisse präsentieren                       │                        │
│  │ □ Fragen beantworten                            │                        │
│  │ □ Nächste Schritte besprechen                  │                        │
│  │ □ Förderantrag vorbereiten                      │                        │
│  │ □ Dokumente übergeben                           │                        │
│  └──────────────────────┬──────────────────────────┘                        │
│                         │                                                    │
│                         ▼                                                    │
│  ┌─────────────────────────────────────────────────┐                        │
│  │ PHASE 8: FÖRDERABWICKLUNG                       │                        │
│  ├─────────────────────────────────────────────────┤                        │
│  │ □ BAFA-Antrag erstellen                         │                        │
│  │ □ Beratungsbericht einreichen                   │                        │
│  │ □ Verwendungsnachweis                           │                        │
│  │ □ Auszahlung abwarten                           │                        │
│  └──────────────────────┬──────────────────────────┘                        │
│                         │                                                    │
│                         ▼                                                    │
│  ┌─────────────────────────────────────────────────┐                        │
│  │ PHASE 9: UMSETZUNGSBEGLEITUNG (optional)        │                        │
│  ├─────────────────────────────────────────────────┤                        │
│  │ □ Ausschreibungshilfe                           │                        │
│  │ □ Angebotsvergleich                             │                        │
│  │ □ Baubegleitung                                 │                        │
│  │ □ Abnahme                                       │                        │
│  │ □ BnD für BEG-Förderung                         │                        │
│  └──────────────────────┬──────────────────────────┘                        │
│                         │                                                    │
│                         ▼                                                    │
│  ┌─────────────────────┐                                                    │
│  │        ENDE         │                                                    │
│  └─────────────────────┘                                                    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.3 iSFP-spezifische Anforderungen

```yaml
# Maßnahmen-Definition für iSFP

massnahme:
  id: uuid
  kategorie: enum [
    Dach_Dachschraegen,
    Oberste_Geschossdecke,
    Aussenwand,
    Kellerdecke,
    Bodenplatte,
    Fenster,
    Ausssentüren,
    Heizung,
    Warmwasser,
    Lueftung,
    Solarthermie,
    Photovoltaik,
    Sommerlicher_Waermeschutz
  ]

  bezeichnung: string
  beschreibung: string

  technische_daten:
    u_wert_vorher: float
    u_wert_nachher: float
    flaeche: float
    # oder bei Anlagentechnik:
    erzeuger_vorher: string
    erzeuger_nachher: string

  kosten:
    investition_brutto: float
    foerderung_prozent: float
    foerderung_betrag: float
    eigenanteil: float

  einsparung:
    endenergie: float (unit: kWh/a)
    primaerenergie: float (unit: kWh/a)
    co2: float (unit: kg/a)
    kosten: float (unit: €/a)

  wirtschaftlichkeit:
    amortisation_ohne_foerderung: float (unit: Jahre)
    amortisation_mit_foerderung: float (unit: Jahre)

  ausfuehrung:
    empfohlener_zeitraum: string
    abhaengigkeiten: list[massnahme_id]
    reihenfolge: integer

# Sanierungsfahrplan-Struktur
sanierungsfahrplan:
  gebaeude_id: uuid
  erstellt_am: datetime
  bearbeiter_id: uuid

  ist_zustand:
    endenergiebedarf: float
    primaerenergiebedarf: float
    effizienzklasse: string
    co2_emissionen: float

  ziel_zustand:
    effizienzhaus_stufe: enum [85, 70, 55, 40]

  schritte:
    - schritt_nummer: integer
      bezeichnung: string
      massnahmen: list[massnahme]
      zeitraum: string
      kosten_summe: float
      foerderung_summe: float
      zwischenzustand:
        endenergiebedarf: float
        effizienzklasse: string

  dokumente:
    mein_sanierungsfahrplan: file (PDF)
    umsetzungshilfe: file (PDF)
    anhang: list[file]
```

---

## 3. Heizlastberechnung

### 3.1 Übersicht

```
Produkt:        Heizlastberechnung
Rechtsgrundlage: DIN EN 12831-1 mit nationalem Anhang
Anwendung:      Dimensionierung Heizungsanlage, hydraulischer Abgleich
Ergebnis:       Raum- und Gebäudeheizlast in Watt
```

### 3.2 Workflow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    WORKFLOW: HEIZLASTBERECHNUNG                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  PHASE 1: RAUMWEISE DATENERFASSUNG                                          │
│  ─────────────────────────────────────                                      │
│  □ Raumbezeichnung und Nutzung                                              │
│  □ Raum-Solltemperatur (θint,i)                                             │
│  □ Raumabmessungen (Fläche, Höhe)                                           │
│  □ Angrenzende Räume und deren Temperaturen                                 │
│  □ Außenbauteile mit U-Werten                                               │
│  □ Wärmebrücken                                                             │
│                                                                              │
│  PHASE 2: NORMAUSLEGUNGSTEMPERATUR                                          │
│  ─────────────────────────────────────                                      │
│  □ Standort → θe (aus DIN/TS 12831-1)                                       │
│  □ Deutschland: -10°C bis -18°C je nach Region                              │
│  □ Österreich: nach ÖNORM                                                   │
│                                                                              │
│  PHASE 3: BERECHNUNG JE RAUM                                                │
│  ─────────────────────────────────────                                      │
│  □ Transmissionswärmeverluste ΦT,i                                          │
│  □ Lüftungswärmeverluste ΦV,i                                               │
│  □ Aufheizzuschlag (optional) ΦRH,i                                         │
│  □ Raumheizlast Φ HL,i = ΦT,i + ΦV,i + ΦRH,i                                │
│                                                                              │
│  PHASE 4: GEBÄUDEHEIZLAST                                                   │
│  ─────────────────────────────────────                                      │
│  □ Summe aller Raumheizlasten                                               │
│  □ Trinkwarmwasserzuschlag (optional)                                       │
│  □ Gleichzeitigkeitsfaktor (bei MFH)                                        │
│  □ Ergebnis: Φ HL [kW]                                                      │
│                                                                              │
│  PHASE 5: DOKUMENTATION                                                     │
│  ─────────────────────────────────────                                      │
│  □ Raumliste mit Heizlasten                                                 │
│  □ Zusammenfassung                                                          │
│  □ Grundlagen und Randbedingungen                                           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.3 Datenstruktur Heizlast

```yaml
heizlast_projekt:
  gebaeude_id: uuid
  norm_version: enum [DIN_EN_12831_2017, DIN_EN_12831_2003]

  standort:
    plz: string
    norm_aussentemperatur: float (unit: °C)  # θe

  raeume:
    - id: uuid
      bezeichnung: string
      geschoss: string
      nutzung: enum [Wohnraum, Bad, Kueche, Flur, Keller, ...]

      geometrie:
        grundflaeche: float (unit: m²)
        raumhoehe: float (unit: m)
        volumen: float (calculated, unit: m³)

      temperatur:
        soll_temperatur: float (default nach Nutzung, unit: °C)  # θint,i

      bauteile:
        - typ: enum [Aussenwand, Fenster, Tuer, Dach, Boden, Innenwand]
          bezeichnung: string
          flaeche: float (unit: m²)
          u_wert: float (unit: W/(m²K))
          temperatur_aussen: float (unit: °C)  # θe oder θu
          korrekturfaktor: float (default: 1.0)  # bu, fg, ...

      lueftung:
        luftwechsel: float (unit: 1/h)
        mindest_luftwechsel: float (calculated)
        infiltration: float (calculated)

      ergebnis:
        transmission_verlust: float (unit: W)  # ΦT,i
        lueftung_verlust: float (unit: W)  # ΦV,i
        aufheiz_zuschlag: float (unit: W)  # ΦRH,i
        raum_heizlast: float (unit: W)  # ΦHL,i
        spezifische_heizlast: float (unit: W/m²)

  ergebnis_gesamt:
    gebaeude_heizlast: float (unit: kW)  # ΦHL
    trinkwarmwasser_zuschlag: float (unit: kW)
    gesamt_waermebedarf: float (unit: kW)
```

---

## 4. Energieaudit nach DIN EN 16247

### 4.1 Übersicht

```
Produkt:        Energieaudit für Unternehmen
Rechtsgrundlage: EDL-G, DIN EN 16247-1 bis -5
Pflicht:        Nicht-KMU alle 4 Jahre (oder ISO 50001/EMAS)
Förderung:      Bis 80% (max. 6.000€) für KMU
Zielgruppe:     Unternehmen
```

### 4.2 Workflow Energieaudit

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    WORKFLOW: ENERGIEAUDIT DIN EN 16247                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  PHASE 1: EINLEITENDER KONTAKT                                              │
│  ─────────────────────────────────────                                      │
│  □ Unternehmensprofil erfassen                                              │
│  □ Standorte und Prozesse identifizieren                                    │
│  □ Energieverbrauchsdaten anfordern (3 Jahre)                               │
│  □ Organisatorische Grenzen festlegen                                       │
│  □ Zeitplan und Ressourcen abstimmen                                        │
│  □ Geheimhaltungsvereinbarung                                               │
│                                                                              │
│  PHASE 2: AUFTAKT-BESPRECHUNG                                               │
│  ─────────────────────────────────────                                      │
│  □ Ziele und Erwartungen klären                                             │
│  □ Verantwortlichkeiten festlegen                                           │
│  □ Datenverfügbarkeit prüfen                                                │
│  □ Vorgehensweise erläutern                                                 │
│  □ Termin für Begehung vereinbaren                                          │
│                                                                              │
│  PHASE 3: DATENERFASSUNG                                                    │
│  ─────────────────────────────────────                                      │
│  □ Energierechnungen (Strom, Gas, Öl, Fernwärme)                           │
│  □ Produktionsdaten/Kennzahlen                                              │
│  □ Anlagenverzeichnis                                                       │
│  □ Betriebszeiten                                                           │
│  □ Gebäudedaten                                                             │
│  □ Frühere Audits/Maßnahmen                                                 │
│                                                                              │
│  PHASE 4: AUẞENDIENST/BEGEHUNG                                              │
│  ─────────────────────────────────────                                      │
│  □ Systematische Betriebsbesichtigung                                       │
│  □ Energierelevante Anlagen erfassen                                        │
│  │  ├── Gebäude und Gebäudehülle                                           │
│  │  ├── Heizung/Kühlung/Lüftung (HVAC)                                     │
│  │  ├── Beleuchtung                                                         │
│  │  ├── Druckluft                                                           │
│  │  ├── Elektrische Antriebe                                                │
│  │  ├── Produktionsprozesse                                                 │
│  │  ├── Wärmerückgewinnung                                                  │
│  │  └── Transport/Fuhrpark                                                  │
│  □ Messungen durchführen (optional)                                         │
│  □ Interviews mit Mitarbeitern                                              │
│  □ Fotodokumentation                                                        │
│                                                                              │
│  PHASE 5: ANALYSE                                                           │
│  ─────────────────────────────────────                                      │
│  □ Energiebilanz erstellen                                                  │
│  │  ├── Energieeinsatz nach Energieträger                                  │
│  │  ├── Energieeinsatz nach Verwendung                                     │
│  │  └── Zeitliche Verteilung (Lastgang)                                    │
│  □ Kennzahlen bilden (EnPI)                                                 │
│  □ Benchmarking (Branchenvergleich)                                         │
│  □ Schwachstellenanalyse                                                    │
│  □ Einsparpotenziale identifizieren                                         │
│                                                                              │
│  PHASE 6: MASSNAHMENENTWICKLUNG                                             │
│  ─────────────────────────────────────                                      │
│  FÜR JEDE MASSNAHME:                                                        │
│  □ Technische Beschreibung                                                  │
│  □ Energieeinsparung [kWh/a]                                                │
│  □ Kosteneinsparung [€/a]                                                   │
│  □ Investitionskosten [€]                                                   │
│  □ Amortisationszeit [Jahre]                                                │
│  □ CO₂-Einsparung [t/a]                                                     │
│  □ Umsetzungsempfehlung                                                     │
│  □ Priorität (hoch/mittel/niedrig)                                          │
│                                                                              │
│  PHASE 7: BERICHTERSTATTUNG                                                 │
│  ─────────────────────────────────────                                      │
│  Energieauditbericht nach DIN EN 16247-1:                                   │
│  □ Zusammenfassung                                                          │
│  □ Hintergrund                                                              │
│  □ Energieaudit-Durchführung                                                │
│  □ Energiebilanz und -analyse                                               │
│  □ Bewertungskriterien                                                      │
│  □ Maßnahmenkatalog                                                         │
│  □ Empfehlungen zur Umsetzung                                               │
│  □ Anhänge (Messdaten, Berechnungen)                                        │
│                                                                              │
│  PHASE 8: ABSCHLUSSBESPRECHUNG                                              │
│  ─────────────────────────────────────                                      │
│  □ Ergebnispräsentation                                                     │
│  □ Diskussion der Maßnahmen                                                 │
│  □ Priorisierung abstimmen                                                  │
│  □ Umsetzungsplan skizzieren                                                │
│  □ Berichtsübergabe                                                         │
│                                                                              │
│  PHASE 9: NACHBEREITUNG                                                     │
│  ─────────────────────────────────────                                      │
│  □ BAFA-Meldung (bei Pflichtaudit)                                          │
│  □ Förderantrag (bei KMU-Audit)                                             │
│  □ Umsetzungsbegleitung (optional)                                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Workflow-Verbindungen und Konsistenz

### 5.1 Gemeinsame Datenbasis

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    DATENFLUSS ZWISCHEN PRODUKTEN                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────┐                                                        │
│  │    GEBÄUDE-     │ ◄─── Einmalige Erfassung der Stammdaten               │
│  │   STAMMDATEN    │                                                        │
│  └────────┬────────┘                                                        │
│           │                                                                  │
│           ├──────────────────────────────────────────────────────────────┐  │
│           │                                                              │  │
│           ▼                                                              ▼  │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐           │  │
│  │  ENERGIEAUSWEIS │  │     iSFP        │  │   HEIZLAST      │           │  │
│  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘           │  │
│           │                    │                    │                     │  │
│           │                    ▼                    │                     │  │
│           │           ┌─────────────────┐           │                     │  │
│           └──────────►│ MASSNAHMEN-     │◄──────────┘                     │  │
│                       │ BERECHNUNG      │                                 │  │
│                       └────────┬────────┘                                 │  │
│                                │                                          │  │
│                                ▼                                          │  │
│                       ┌─────────────────┐                                 │  │
│                       │  KFW/BEG-       │                                 │  │
│                       │  NACHWEIS       │                                 │  │
│                       └────────┬────────┘                                 │  │
│                                │                                          │  │
│                                ▼                                          │  │
│                       ┌─────────────────┐                                 │  │
│                       │ FÖRDER-         │                                 │  │
│                       │ ANTRAG          │                                 │  │
│                       └─────────────────┘                                 │  │
│                                                                              │
│  KONSISTENZREGELN:                                                          │
│  ─────────────────                                                          │
│  • Gebäudedaten werden EINMAL erfasst und überall verwendet                │
│  • Änderungen werden automatisch in alle Produkte übernommen               │
│  • Berechnungen bauen aufeinander auf (Ist → Varianten → Nachweis)         │
│  • Versionierung: Jede Änderung wird dokumentiert                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 5.2 Konsistenzprüfungen

```yaml
# Automatische Konsistenzprüfungen

konsistenz_checks:
  gebaeude:
    - check: nutzflaeche_plausibel
      regel: "AN >= 0.32 * Bruttogeschossfläche"
      warnung: "Nutzfläche AN erscheint zu gering"

    - check: huellflaeche_plausibel
      regel: "A/V-Verhältnis zwischen 0.2 und 1.5"
      warnung: "Hüllfläche/Volumen-Verhältnis unplausibel"

    - check: fensterflaeche_plausibel
      regel: "Fensteranteil 10-40% der Außenwandfläche"
      warnung: "Fensteranteil erscheint ungewöhnlich"

  u_werte:
    - check: u_wert_baujahr
      regel: "U-Werte entsprechen Baujahr (Toleranz ±20%)"
      warnung: "U-Wert für Baujahr unplausibel"

    - check: u_wert_minimum
      regel: "U-Werte > 0.1 W/(m²K)"
      fehler: "U-Wert zu niedrig"

    - check: u_wert_maximum
      regel: "U-Werte < 5.0 W/(m²K)"
      fehler: "U-Wert zu hoch"

  anlagentechnik:
    - check: heizung_leistung
      regel: "Nennleistung >= Heizlast"
      warnung: "Heizungsleistung könnte unzureichend sein"

    - check: wp_jaz
      regel: "JAZ Wärmepumpe zwischen 2.0 und 6.0"
      warnung: "Jahresarbeitszahl unplausibel"

  energie:
    - check: verbrauch_bedarf_abgleich
      regel: "Berechneter Bedarf ±30% vom Verbrauch"
      warnung: "Große Abweichung Bedarf/Verbrauch - Daten prüfen"

    - check: primaerenergie_geg
      regel: "Primärenergiebedarf <= GEG-Anforderung"
      info: "Anforderungen werden eingehalten/nicht eingehalten"

  foerderung:
    - check: effizienzhaus_nachweis
      regel: "Alle Effizienzhaus-Kriterien erfüllt"
      fehler: "Effizienzhaus-Stufe nicht erreicht"
      details:
        - "QP <= Referenz * Faktor"
        - "H'T <= Referenz * Faktor"
        - "Sommerlicher Wärmeschutz erfüllt"
```

### 5.3 Workflow-Übergänge

```yaml
# Definition der erlaubten Übergänge zwischen Workflows

workflow_transitions:
  energieausweis:
    kann_starten_von:
      - neues_projekt
      - bestehendes_gebaeude
    kann_übergehen_zu:
      - isfp (mit vorhandenen Gebäudedaten)
      - heizlast (mit vorhandenen Gebäudedaten)
      - geg_nachweis (bei Neubau/Sanierung)
    voraussetzungen:
      - gebaeude_stammdaten_vollstaendig
      - anlagentechnik_erfasst

  isfp:
    kann_starten_von:
      - neues_projekt
      - nach_energieausweis
    kann_übergehen_zu:
      - beg_antrag (pro Maßnahme)
      - baubegleitung (bei Umsetzung)
    voraussetzungen:
      - vor_ort_begehung_durchgefuehrt
      - ist_zustand_berechnet

  heizlast:
    kann_starten_von:
      - neues_projekt
      - nach_energieausweis
      - vor_anlagendimensionierung
    kann_übergehen_zu:
      - hydraulischer_abgleich
      - heizungsauslegung
    voraussetzungen:
      - raeume_vollstaendig_erfasst
      - u_werte_bekannt

  beg_antrag:
    kann_starten_von:
      - nach_isfp
      - nach_beratung
    voraussetzungen:
      - massnahme_definiert
      - kosten_ermittelt
      - technische_mindestanforderungen_geprueft
```
