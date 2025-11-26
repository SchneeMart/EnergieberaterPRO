# Energieberechnungs-Module - Übersicht

## Einleitung

EnergieberaterPRO bietet ein umfassendes Berechnungssystem für alle Aspekte der Gebäudeenergetik. Die Module sind normkonform für **Deutschland (DIN)** und **Österreich (ÖNORM)** implementiert und werden automatisch basierend auf dem Land der Organisation ausgewählt.

---

## 1. Berechnungskategorien

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        ENERGIEBERECHNUNGS-ENGINE v2.0                           │
│                     ═══════════════════════════════════                         │
│                                                                                 │
│     DE: DIN V 18599 | DIN EN 12831 | GEG 2024                                  │
│     AT: ÖNORM B 8110 | ÖNORM H 5050 | OIB-RL 6                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ╔═══════════════════════╗  ╔═══════════════════════╗  ╔═══════════════════╗  │
│  ║    BAUPHYSIK          ║  ║   ANLAGENTECHNIK      ║  ║   ERNEUERBARE     ║  │
│  ║   ▬▬▬▬▬▬▬▬▬▬▬▬       ║  ║   ▬▬▬▬▬▬▬▬▬▬▬▬▬▬     ║  ║   ▬▬▬▬▬▬▬▬▬▬▬    ║  │
│  ║ ✓ U-Wert-Berechnung   ║  ║ ✓ Wärmeerzeuger       ║  ║ ✓ Photovoltaik    ║  │
│  ║ ✓ Wärmebrücken (ψ)    ║  ║ ✓ Wärmeverteilung     ║  ║ ✓ Solarthermie    ║  │
│  ║ ✓ Tauwasserschutz     ║  ║ ✓ Wärmeübergabe       ║  ║ ✓ Wärmepumpen     ║  │
│  ║ ✓ Sommerl. Wärmeschutz║  ║ ✓ Trinkwarmwasser     ║  ║ ✓ Biomasse        ║  │
│  ║ ✓ Luftdichtheit       ║  ║ ✓ Lüftung/WRG         ║  ║ ✓ Kleinwind       ║  │
│  ║ ✓ Tageslicht          ║  ║ ✓ Kühlung             ║  ║ ✓ Geothermie      ║  │
│  ║ ○ Hygrothermik        ║  ║ ✓ Beleuchtung         ║  ║ ○ Wasserstoff     ║  │
│  ╚═══════════════════════╝  ╚═══════════════════════╝  ╚═══════════════════╝  │
│                                                                                 │
│  ╔═══════════════════════╗  ╔═══════════════════════╗  ╔═══════════════════╗  │
│  ║   ENERGIEBILANZ       ║  ║    EMISSIONEN         ║  ║  WIRTSCHAFT       ║  │
│  ║   ▬▬▬▬▬▬▬▬▬▬▬▬       ║  ║   ▬▬▬▬▬▬▬▬▬▬▬        ║  ║  ▬▬▬▬▬▬▬▬▬▬▬▬    ║  │
│  ║ ✓ Heizwärmebedarf     ║  ║ ✓ CO2-Bilanz          ║  ║ ✓ Investition     ║  │
│  ║ ✓ Kühlbedarf          ║  ║ ✓ Primärenergie       ║  ║ ✓ Betriebskosten  ║  │
│  ║ ✓ Endenergiebedarf    ║  ║ ✓ Emissionsfaktoren   ║  ║ ✓ Amortisation    ║  │
│  ║ ✓ Primärenergie       ║  ║ ✓ THG-Bilanz          ║  ║ ✓ Kapitalwert     ║  │
│  ║ ✓ Energieausweis      ║  ║ ✓ Scope 1/2/3         ║  ║ ✓ Förderungen     ║  │
│  ║ ○ Lebenszyklusanalyse ║  ║ ○ Graue Energie       ║  ║ ✓ Sensitivität    ║  │
│  ╚═══════════════════════╝  ╚═══════════════════════╝  ╚═══════════════════╝  │
│                                                                                 │
│  Legende: ✓ = Implementiert (Phase 1-4)  ○ = Geplant (Phase 5-7)               │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Normengrundlage

### 2.1 Deutsche Normen (DE)

```
┌────────────────────────────────────────────────────────────────────────────────┐
│                        DIN V 18599 - ÜBERSICHT                                 │
├────────────────────────────────────────────────────────────────────────────────┤
│                                                                                │
│  Teil 1: Allgemeine Bilanzierungsverfahren      ──┐                           │
│  Teil 2: Nutzenergiebedarf Heizen/Kühlen          ├── Grundlagen              │
│  Teil 10: Nutzungsrandbedingungen               ──┘                           │
│                                                                                │
│  Teil 3: Nutzenergiebedarf Beleuchtung          ──┐                           │
│  Teil 4: Endenergiebedarf Beleuchtung             ├── Beleuchtung             │
│                                                                                │
│  Teil 5: Endenergiebedarf Heizsysteme           ──┤                           │
│  Teil 6: Endenergiebedarf Lüftung, TWW            ├── Anlagentechnik          │
│  Teil 7: Endenergiebedarf Kühlung                 │                           │
│  Teil 8: Nutz- und Endenergie TWW               ──┘                           │
│                                                                                │
│  Teil 9: End- und Primärenergie KWK/Strom       ── Stromerzeugung            │
│  Teil 11: Gebäudeautomation                     ── Automation                 │
│                                                                                │
└────────────────────────────────────────────────────────────────────────────────┘
```

| Norm | Titel | Anwendung | Status |
|------|-------|-----------|--------|
| **DIN V 18599** | Energetische Bewertung | Gesamtbilanz | ✓ |
| **DIN EN 12831** | Heizlastberechnung | Auslegung Heizung | ✓ |
| **DIN 4108-2** | Mindestwärmeschutz | Sommerl. Wärmeschutz | ✓ |
| **DIN 4108-3** | Klimabedingter Feuchteschutz | Tauwasser (Glaser) | ✓ |
| **DIN 4108-6** | Wärmedämmung | Anwendungsnorm | ✓ |
| **DIN EN ISO 6946** | Wärmedurchlasswiderstand | U-Wert-Berechnung | ✓ |
| **DIN EN ISO 10077** | Fenster und Türen | Uw-Wert | ✓ |
| **DIN EN ISO 13370** | Wärmeübertragung | Erdberührte Bauteile | ✓ |
| **VDI 2078** | Kühllastberechnung | Auslegung Kühlung | ✓ |

### 2.2 Österreichische Normen (AT)

```
┌────────────────────────────────────────────────────────────────────────────────┐
│                     ÖNORM-STRUKTUR FÜR ENERGIEBERECHNUNG                       │
├────────────────────────────────────────────────────────────────────────────────┤
│                                                                                │
│  ÖNORM B 8110 - Wärmeschutz im Hochbau                                        │
│  ├── Teil 1: Grundlagen                                                       │
│  ├── Teil 2: Wasserdampfdiffusion und Kondensation                           │
│  ├── Teil 3: Feuchteschutz                                                    │
│  ├── Teil 5: Klimamodell und Nutzungsprofile                                 │
│  └── Teil 6: Heiz- und Kühlenergiebedarf                                     │
│                                                                                │
│  ÖNORM H 5050 - Energieausweis-Reihe                                          │
│  ├── H 5050: Gesamtenergieeffizienz                                          │
│  ├── H 5055: Energieausweis Wohngebäude                                      │
│  ├── H 5056: Energieausweis Nichtwohngebäude                                 │
│  └── H 5057: Primärenergie und CO2                                           │
│                                                                                │
│  OIB-Richtlinie 6: Energieeinsparung und Wärmeschutz                         │
│                                                                                │
└────────────────────────────────────────────────────────────────────────────────┘
```

| Norm | Titel | Anwendung | Status |
|------|-------|-----------|--------|
| **ÖNORM B 8110-1** | Wärmeschutz Grundlagen | Mindestanforderungen | ✓ |
| **ÖNORM B 8110-2** | Wasserdampfdiffusion | Tauwasser | ✓ |
| **ÖNORM B 8110-6** | Heiz-/Kühlenergiebedarf | Energiebilanz | ✓ |
| **ÖNORM H 5050** | Gesamtenergieeffizienz | Primärenergie | ✓ |
| **ÖNORM H 5055** | EA Wohngebäude | Energieausweis | ✓ |
| **ÖNORM H 5056** | EA Nichtwohngebäude | Energieausweis | ✓ |
| **ÖNORM H 5057** | CO2-Emissionen | Klimabewertung | ✓ |
| **OIB-RL 6** | Energieeinsparung | Bauordnung | ✓ |

### 2.3 Europäische Normen (DE & AT)

| Norm | Titel | Anwendung |
|------|-------|-----------|
| **EN ISO 52000-1** | EPB Berechnung Überblick | EPBD Framework |
| **EN ISO 52003-1** | EPB Indikatoren | Energiekennwerte |
| **EN ISO 52010-1** | Klimadaten | Außenklima |
| **EN ISO 52016-1** | Energiebedarf Heizen/Kühlen | Monatsbilanz |
| **EN ISO 52018-1** | Indikatoren Hülle | Gebäudehülle |
| **EN 15978** | Nachhaltigkeit Bauwerke | LCA |
| **EN 15804** | Umweltdeklarationen | EPD |
| **EN 17037** | Tageslicht | Beleuchtung |

### 2.4 Gesetzliche Grundlagen

#### Deutschland (DE)
| Gesetz/Verordnung | Relevanz |
|-------------------|----------|
| **GEG 2024** | Gebäudeenergiegesetz (Hauptgesetz) |
| **BEG** | Bundesförderung effiziente Gebäude |
| **EU-EPBD** | Energy Performance of Buildings Directive |
| **EnEV** (historisch) | Für Bestandsgebäude relevant |

#### Österreich (AT)
| Gesetz/Verordnung | Relevanz |
|-------------------|----------|
| **OIB-Richtlinie 6** | Bauordnung Energie |
| **Landesbauordnungen** | Bundeslandspezifisch |
| **EAVG** | Energieausweis-Vorlage-Gesetz |
| **BVG Klimaschutz** | Klimaziele 2040 |

---

## 3. Berechnungsmodule im Detail

### 3.1 Bauphysikalische Berechnungen

#### U-Wert-Berechnung
```
Formel: U = 1 / R_ges

R_ges = R_si + Σ(d_i / λ_i) + R_se

Wo:
  R_si  = Wärmeübergangswiderstand innen [m²K/W]
  d_i   = Schichtdicke [m]
  λ_i   = Wärmeleitfähigkeit [W/(mK)]
  R_se  = Wärmeübergangswiderstand außen [m²K/W]
```

**Eingangsparameter:**
- Schichtaufbau (Material, Dicke)
- Wärmeübergänge (horizontal, aufwärts, abwärts)
- Luftschichten (ruhend, belüftet)

**Ausgabe:**
- U-Wert [W/(m²K)]
- R-Wert [m²K/W]
- Temperaturverlauf durch Bauteil
- Taupunktanalyse

#### Wärmebrückenberechnung
```
Methoden:
1. Pauschaler Zuschlag ΔUWB
2. Gleichwertigkeitsnachweis
3. Detaillierte Berechnung (ψ-Werte)
```

**Katalog Wärmebrücken:**
- Außenecken
- Fensteranschlüsse
- Balkonanschlüsse
- Dachanschlüsse
- Sockelanschlüsse
- Innenwandanschlüsse

### 3.2 Heizlastberechnung (DIN EN 12831)

#### Transmissionswärmeverluste
```
Φ_T,i = (H_T,ie + H_T,iue + H_T,ig + H_T,ij) × (θ_int,i - θ_e)

Wo:
  H_T,ie  = Direkt nach außen [W/K]
  H_T,iue = Über unbeheizte Räume [W/K]
  H_T,ig  = An Erdreich [W/K]
  H_T,ij  = An anders temperierte Räume [W/K]
```

#### Lüftungswärmeverluste
```
Φ_V,i = H_V,i × (θ_int,i - θ_e)

H_V,i = V_i × ρ × c_p × n_min

Wo:
  V_i   = Raumvolumen [m³]
  n_min = Mindestluftwechsel [1/h]
  ρ     = Luftdichte [kg/m³]
  c_p   = Spez. Wärmekapazität [J/(kgK)]
```

#### Aufheizleistung
```
Φ_RH,i = A_i × f_RH

Wo:
  A_i   = Grundfläche [m²]
  f_RH  = Aufheizfaktor [W/m²]
```

**Eingangsparameter:**
- Gebäudegeometrie (alle Bauteile)
- U-Werte aller Bauteile
- Wärmebrücken
- Luftwechsel
- Norm-Außentemperatur
- Raum-Solltemperaturen

**Ausgabe:**
- Heizlast pro Raum [W]
- Heizlast pro Zone [kW]
- Heizlast Gebäude [kW]
- Spezifische Heizlast [W/m²]
- Detailauswertung pro Bauteil

### 3.3 Energiebilanz (DIN V 18599)

#### Monatsbilanzverfahren
```
Q_h,b = Q_sink - η_h × Q_source

Q_sink   = Q_T + Q_V (Wärmeverluste)
Q_source = Q_s + Q_i (Wärmegewinne)
η_h      = Ausnutzungsgrad der Gewinne

Für Kühlung analog mit Kühlbedarf Q_c,b
```

#### Nutzungszonen nach DIN V 18599-10
```
Zonentypen:
- EFH/MFH (Wohngebäude)
- Büro (Einzel, Gruppen, Großraum)
- Besprechung/Konferenz
- Schule/Hörsaal
- Krankenhaus (verschiedene Bereiche)
- Hotel
- Gastronomie
- Verkauf
- Produktion/Werkstatt
- Lager
- Verkehrsflächen
- Sanitär
- Serverraum
- etc. (40+ Profile)
```

**Eingangsparameter pro Zone:**
- Geometrie (Fläche, Volumen, Hüllfläche)
- Bauteile mit U-Werten
- Fenster (g-Wert, Rahmenanteil)
- Nutzungsprofil
- Interne Wärmelasten
- Luftwechsel
- Solltemperaturen

**Ausgabe:**
- Nutzenergiebedarf Heizen [kWh/a]
- Nutzenergiebedarf Kühlen [kWh/a]
- Nutzenergiebedarf Beleuchtung [kWh/a]
- Nutzenergiebedarf TWW [kWh/a]
- Monatliche Verteilung
- Energiebilanz-Diagramme

### 3.4 Anlagentechnik

#### Wärmeerzeuger
```
Systemtypen:
├── Kessel
│   ├── Gas-Brennwert
│   ├── Öl-Brennwert
│   ├── Gas-Niedertemperatur
│   ├── Öl-Niedertemperatur
│   ├── Pellet
│   ├── Hackschnitzel
│   └── Scheitholz
├── Wärmepumpen
│   ├── Luft/Wasser
│   ├── Sole/Wasser
│   ├── Wasser/Wasser
│   └── Luft/Luft
├── KWK
│   ├── Motor-BHKW
│   └── Brennstoffzelle
├── Fernwärme
│   ├── fossil
│   └── erneuerbar
└── Sonstige
    ├── Elektro-Direktheizung
    └── Solarthermie (Unterstützung)
```

**Berechnungsparameter:**
- Erzeugeraufwandszahl e_g
- Aufwandszahl Speicherung e_s
- Aufwandszahl Verteilung e_d
- Hilfsenergiebedarf W_aux

#### Wärmeverteilung
```
Verlustberechnung:
Q_d = Σ(U_j × L_j × Δθ_j × t_j)

Parameter:
- Rohrleitungslängen
- Dämmqualität
- Mediumtemperaturen
- Betriebszeiten
```

#### Lüftungsanlagen
```
Systemtypen:
├── Abluftanlagen
│   ├── Zentral
│   └── Dezentral
├── Zu-/Abluftanlagen
│   ├── Ohne WRG
│   ├── Mit Platten-WT
│   ├── Mit Rotations-WT
│   └── Mit Kreislauf-WT
└── Hybrid
    └── Fensterlüftung + Abluft
```

**Kennwerte:**
- Wärmerückgewinnungsgrad η_WRG
- Spezifische Ventilatorleistung SFP
- Luftvolumenstrom
- Druckverluste

### 3.5 Erneuerbare Energien

#### Photovoltaik
```
Jahresertrag:
E_PV = P_peak × f_perf × H_T / G_STC

Wo:
  P_peak  = Installierte Leistung [kWp]
  f_perf  = Performance Ratio (~0.75-0.85)
  H_T     = Globalstrahlung geneigt [kWh/(m²a)]
  G_STC   = Referenzstrahlung 1000 W/m²
```

**Eingangsparameter:**
- Modulleistung und -fläche
- Ausrichtung (Azimut)
- Neigung
- Verschattung
- Wechselrichter-Wirkungsgrad
- Standort (Klimadaten)

**Ausgabe:**
- Jahresertrag [kWh/a]
- Monatsverteilung
- Eigenverbrauchsquote
- Autarkiegrad
- CO2-Einsparung

#### Solarthermie
```
Solarertrag:
Q_sol = A_ap × η_0 × H_T × f_sol

Mit Systemverlusten und Speicherverlusten
```

**Systemtypen:**
- Flachkollektoren
- Röhrenkollektoren
- Nur TWW
- TWW + Heizungsunterstützung

#### Wärmepumpen
```
Jahresarbeitszahl (JAZ):
JAZ = Q_nutz / W_el

COP bei Normbedingungen (z.B. B0/W35, A2/W35)
```

**Berücksichtigte Faktoren:**
- Quellentemperatur (Luft, Erdreich, Wasser)
- Vorlauftemperatur
- Modulationsbereich
- Abtauenergie (Luft-WP)
- Sperrzeiten

### 3.6 CO2-Bilanzierung

#### Emissionsfaktoren
```
Energieträger         | CO2-Äquivalent [g/kWh]
---------------------|----------------------
Strom (Bundesmix)    | 420 (variabel)
Erdgas               | 201
Heizöl               | 266
Flüssiggas           | 234
Steinkohle           | 335
Braunkohle           | 407
Holzpellets          | 23 (biogen)
Fernwärme (fossil)   | 240 (variabel)
Fernwärme (erneuerb.)| 40 (variabel)
```

#### THG-Bilanz nach GHG Protocol
```
Scope 1: Direkte Emissionen
  - Verbrennung vor Ort
  - Kältemittelleckagen

Scope 2: Indirekte (Energie)
  - Strombezug
  - Fernwärme/Fernkälte

Scope 3: Sonstige indirekte
  - Vorketten Brennstoffe
  - Entsorgung
  - Anfahrt Mitarbeiter
```

### 3.7 Wirtschaftlichkeitsberechnung

#### Investitionsrechnung
```
Kapitalwert (NPV):
NPV = -I_0 + Σ(CF_t / (1+r)^t)

Amortisationszeit:
T_a = I_0 / (E_spar × p_E)

Annuität:
A = I_0 × (q^n × (q-1)) / (q^n - 1)
```

**Kostenarten:**
- Investitionskosten
- Kapitalkosten
- Wartungskosten
- Energiekosten
- Ersatzinvestitionen

#### Fördermittelrechnung

```
┌────────────────────────────────────────────────────────────────────────────────┐
│                       FÖRDERPROGRAMME ÜBERSICHT                                │
├────────────────────────────────────────────────────────────────────────────────┤
│                                                                                │
│  DEUTSCHLAND (BEG)                       │  ÖSTERREICH                         │
│  ═══════════════                         │  ══════════                         │
│                                          │                                     │
│  BEG EM (Einzelmaßnahmen)                │  Sanierungsoffensive                │
│  ├── Gebäudehülle (15%)                  │  ├── Thermische Sanierung (bis 50%)│
│  ├── Anlagentechnik (20-35%)             │  ├── Heizungstausch (bis 75%)      │
│  └── Heizungsoptimierung (15%)           │  └── Photovoltaik (bis 285€/kWp)   │
│                                          │                                     │
│  BEG WG/NWG (Vollsanierung)              │  Landesförderungen                  │
│  ├── Effizienzhaus 85 (5%)               │  ├── Bundeslandspezifisch          │
│  ├── Effizienzhaus 70 (10%)              │  └── Gemeindezuschüsse             │
│  ├── Effizienzhaus 55 (15%)              │                                     │
│  ├── Effizienzhaus 40 (20%)              │  EU-Förderungen                     │
│  └── WPB-Bonus (+10%)                    │  └── Aufbau- und Resilienzplan     │
│                                          │                                     │
│  KfW-Programme                           │  Umweltförderung (UFI)              │
│  ├── Kredite                             │  ├── Gewerbliche Objekte           │
│  └── Tilgungszuschüsse                   │  └── Contracting-Modelle           │
│                                          │                                     │
└────────────────────────────────────────────────────────────────────────────────┘
```

---

### 3.8 Tauwasserschutz (Glaser-Verfahren)

**Norm:** DIN 4108-3 / ÖNORM B 8110-2

```
Tauwassermenge:
m_c = Σ(g_c,i × Δt_i)

Verdunstungsmenge:
m_ev = Σ(g_ev,i × Δt_i)

Kriterium:
m_c - m_ev ≤ m_zul (zulässige Tauwassermenge)

m_zul = {
  0.5 kg/m² für kapillaraktive Schichten
  1.0 kg/m² für nicht kapillaraktive Schichten
}
```

**Eingabeparameter:**
- Schichtaufbau mit sd-Werten
- Innenklima (Temperatur, Feuchte)
- Außenklima (Monatsmittelwerte)
- Randbedingungen nach Bauteiltyp

**Ausgabe:**
- Temperaturverlauf im Bauteil
- Sättigungsdampfdruck-Verlauf
- Tauwasserebenen und -mengen
- Grafische Darstellung (Glaser-Diagramm)
- Nachweis bestanden/nicht bestanden

---

### 3.9 Sommerlicher Wärmeschutz

**Norm:** DIN 4108-2:2013-02

```
Sonneneintragskennwert (vereinfacht):
S = Σ(A_w,j × g_tot,j × F_c,j) / A_G

Nachweis bestanden wenn:
S_vorh ≤ S_zul (abhängig von Klimaregion und Bauweise)

Genaueres Verfahren (Simulation):
- Operative Übertemperaturgradstunden
- Θ_op,max ≤ 27°C (Wohngebäude)
```

**Klimaregionen Deutschland:**
| Region | Kennzeichen | S_zul (leicht) |
|--------|-------------|----------------|
| A | Sommerkühl | 0.113 |
| B | Gemäßigt | 0.080 |
| C | Sommerheiß | 0.058 |

**Eingabeparameter:**
- Verglasungsflächen (Orientierung, Neigung)
- g-Werte und Sonnenschutz (Fc)
- Nettogrundfläche
- Bauweise (leicht/schwer)
- Nachtlüftung (ja/nein)

---

### 3.10 Lebenszyklusanalyse (LCA)

**Norm:** EN 15978, EN 15804

```
┌────────────────────────────────────────────────────────────────────────────────┐
│                    LEBENSZYKLUSPHASEN NACH EN 15978                            │
├────────────────────────────────────────────────────────────────────────────────┤
│                                                                                │
│  PRODUKT        BAU          NUTZUNG              ENTSORGUNG    VORTEILE      │
│  A1-A3          A4-A5        B1-B7                C1-C4         D             │
│  ────────       ─────        ─────                ──────        ────────      │
│  Rohstoffe      Transport    Nutzung              Rückbau       Recycling     │
│  Produktion     Einbau       Instandhaltung       Transport     Wiederverw.   │
│  Transport                   Reparatur            Abfallbeh.    Energie       │
│                              Ersatz               Entsorgung                   │
│                              Umbau                                             │
│                              ENERGIE (B6)                                      │
│                              WASSER (B7)                                       │
│                                                                                │
│  ◄─────────────────── GRAUE ENERGIE ──────────────────►                       │
│                       ◄──── BETRIEBSENERGIE ────►                             │
│                                                                                │
└────────────────────────────────────────────────────────────────────────────────┘
```

**Berechnete Indikatoren:**
| Indikator | Einheit | Beschreibung |
|-----------|---------|--------------|
| GWP | kg CO2-eq | Treibhauspotenzial |
| ODP | kg CFC-11-eq | Ozonabbaupotenzial |
| AP | kg SO2-eq | Versauerungspotenzial |
| EP | kg PO4-eq | Eutrophierungspotenzial |
| PENRT | MJ | Primärenergie nicht-erneuerbar |
| PERT | MJ | Primärenergie erneuerbar |

**Datenquellen:**
- Ökobaudat (Deutschland)
- baubook (Österreich)
- EPD-Datenbanken
- Generische Datensätze

---

### 3.11 Tageslichtberechnung

**Norm:** EN 17037

```
Tageslichtquotient:
D = (E_i / E_a) × 100 [%]

Wo:
  E_i = Beleuchtungsstärke innen [lux]
  E_a = Beleuchtungsstärke außen (Diffusstrahlung) [lux]

Mindestanforderungen EN 17037:
- Minimum: D ≥ 2.0% auf 50% der Nutzfläche
- Mittel:   D ≥ 3.0% auf 50% der Nutzfläche
- Hoch:    D ≥ 5.0% auf 50% der Nutzfläche
```

**Einflussfaktoren:**
- Fensterfläche und -position
- Verglasungstransmission (τv)
- Rahmenfaktor
- Verschattung (Umgebung, Überhänge)
- Raumgeometrie und -reflexion
- Himmelstyp (bedeckt nach CIE)

---

## 4. Berechnungs-Pipeline

### 4.1 Ablaufdiagramm

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         BERECHNUNGS-WORKFLOW                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────┐                                                            │
│  │   INPUT     │                                                            │
│  │   DATEN     │                                                            │
│  └──────┬──────┘                                                            │
│         │                                                                   │
│         ▼                                                                   │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐                   │
│  │ Validierung │────▶│   Parser    │────▶│ Normalisier.│                   │
│  │  (Schema)   │     │             │     │             │                   │
│  └─────────────┘     └─────────────┘     └─────────────┘                   │
│                                                 │                           │
│         ┌───────────────────────────────────────┘                           │
│         ▼                                                                   │
│  ┌─────────────┐                                                            │
│  │ BAUPHYSIK   │                                                            │
│  │ U-Werte     │                                                            │
│  │ Wärmebrück. │                                                            │
│  └──────┬──────┘                                                            │
│         │                                                                   │
│         ├──────────────────┬──────────────────┐                             │
│         ▼                  ▼                  ▼                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                     │
│  │  HEIZLAST   │    │  KÜHLLAST   │    │ BELEUCHTUNG │                     │
│  │ DIN EN 12831│    │  VDI 2078   │    │ DIN V 18599 │                     │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘                     │
│         │                  │                  │                             │
│         └──────────────────┼──────────────────┘                             │
│                            ▼                                                │
│                     ┌─────────────┐                                         │
│                     │ ENERGIEBIL. │                                         │
│                     │ DIN V 18599 │                                         │
│                     │ Nutzenergie │                                         │
│                     └──────┬──────┘                                         │
│                            │                                                │
│         ┌──────────────────┼──────────────────┐                             │
│         ▼                  ▼                  ▼                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                     │
│  │ ERZEUGUNG   │    │ VERTEILUNG  │    │  ERNEUERB.  │                     │
│  │ Kessel, WP  │    │ Rohre, Pumpen│   │  PV, ST, etc│                     │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘                     │
│         │                  │                  │                             │
│         └──────────────────┼──────────────────┘                             │
│                            ▼                                                │
│                     ┌─────────────┐                                         │
│                     │ ENDENERGIE  │                                         │
│                     │ + Hilfsener.│                                         │
│                     └──────┬──────┘                                         │
│                            │                                                │
│         ┌──────────────────┼──────────────────┐                             │
│         ▼                  ▼                  ▼                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                     │
│  │ PRIMÄRENER. │    │    CO2      │    │ WIRTSCHAFT  │                     │
│  │ Faktoren    │    │  Emissionen │    │  Kosten     │                     │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘                     │
│         │                  │                  │                             │
│         └──────────────────┼──────────────────┘                             │
│                            ▼                                                │
│                     ┌─────────────┐                                         │
│                     │  ERGEBNIS   │                                         │
│                     │  Validieren │                                         │
│                     │  Speichern  │                                         │
│                     └─────────────┘                                         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 4.2 Validierung

#### Eingabevalidierung
```
Regeln:
- Plausibilitätsprüfung aller Eingaben
- Grenzwertprüfung (min/max)
- Konsistenzprüfung (Flächen, Volumen)
- Vollständigkeitsprüfung
```

#### Ergebnisvalidierung
```
Prüfungen:
- Kennwertvergleich (typische Bereiche)
- Energiebilanz-Check (Summengleichheit)
- Grenzwertprüfung GEG
- Warnungen bei Ausreißern
```

---

## 5. Datenstrukturen

### 5.1 Berechnungseingabe (Beispiel)
```json
{
  "building": {
    "id": "uuid",
    "type": "single_family",
    "heated_area": 150.0,
    "heated_volume": 450.0,
    "construction_year": 1980
  },
  "zones": [
    {
      "id": "uuid",
      "name": "Wohnbereich",
      "type": "residential",
      "area": 100.0,
      "height": 2.5,
      "target_temp": 20.0
    }
  ],
  "components": [
    {
      "id": "uuid",
      "type": "exterior_wall",
      "area": 120.0,
      "u_value": 0.24,
      "orientation": "north"
    }
  ],
  "systems": [
    {
      "id": "uuid",
      "type": "heating_heat_pump_air",
      "nominal_power": 8.0,
      "cop": 3.5
    }
  ],
  "climate": {
    "location": "12345",
    "design_temp": -12.0,
    "heating_degree_days": 3200
  }
}
```

### 5.2 Berechnungsergebnis (Beispiel)
```json
{
  "calculation_id": "uuid",
  "timestamp": "2025-01-15T10:30:00Z",
  "results": {
    "heat_load": {
      "total_kw": 6.8,
      "specific_w_m2": 45.3,
      "by_zone": [...]
    },
    "energy_demand": {
      "heating_kwh_a": 12500,
      "cooling_kwh_a": 0,
      "dhw_kwh_a": 3000,
      "lighting_kwh_a": 800,
      "total_kwh_a": 16300
    },
    "final_energy": {
      "electricity_kwh_a": 4800,
      "gas_kwh_a": 0
    },
    "primary_energy": {
      "non_renewable_kwh_a": 8160,
      "total_kwh_a": 9600,
      "specific_kwh_m2a": 64.0
    },
    "emissions": {
      "co2_kg_a": 2016,
      "specific_kg_m2a": 13.4
    },
    "kpis": {
      "efficiency_class": "B",
      "geg_compliant": true,
      "renewable_share": 0.65
    }
  }
}
```

---

## 6. Referenzdokumente

- [U-Wert-Berechnung im Detail](./gebaeude/U_WERT_BERECHNUNG.md)
- [Heizlastberechnung im Detail](./heizung/HEIZLAST_DIN_EN_12831.md)
- [Energiebilanz DIN V 18599](./energie/ENERGIEBILANZ_DIN_V_18599.md)
- [Erneuerbare Energien](./erneuerbare/ERNEUERBARE_BERECHNUNG.md)
- [CO2-Bilanzierung](./co2/CO2_BILANZIERUNG.md)
- [Validierung und Normenprüfung](./validierung/VALIDIERUNG.md)

---

*Letzte Aktualisierung: 2025-11-26*
*Version: 2.0.0*
