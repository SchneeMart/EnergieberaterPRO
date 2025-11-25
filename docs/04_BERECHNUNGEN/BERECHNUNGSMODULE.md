# Berechnungsmodule - Technische Spezifikation

## Übersicht der Berechnungsmodule

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         BERECHNUNGS-ENGINE                                       │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐                │
│  │   MODUL 1       │   │   MODUL 2       │   │   MODUL 3       │                │
│  │   U-Wert        │   │   Wärmebrücken  │   │   Heizlast      │                │
│  │   DIN 4108-4    │   │   DIN 4108-2    │   │   DIN EN 12831  │                │
│  └────────┬────────┘   └────────┬────────┘   └────────┬────────┘                │
│           │                     │                     │                          │
│           └─────────────────────┼─────────────────────┘                          │
│                                 ▼                                                │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │                        MODUL 4: ENERGIEBILANZ                            │    │
│  │                        DIN V 18599 (alle Teile)                          │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │    │
│  │  │ Teil 2   │ │ Teil 3   │ │ Teil 4   │ │ Teil 5   │ │ Teil 6   │       │    │
│  │  │ Zonen    │ │ TWW      │ │ Beleuch- │ │ Heizung  │ │ Lüftung  │       │    │
│  │  │          │ │          │ │ tung     │ │          │ │          │       │    │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘       │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │    │
│  │  │ Teil 7   │ │ Teil 8   │ │ Teil 9   │ │ Teil 10  │ │ Teil 11  │       │    │
│  │  │ Klima    │ │ Kühlung  │ │ Kraft-   │ │ Nutzungs-│ │ Gebäude- │       │    │
│  │  │          │ │          │ │ Wärme    │ │ profile  │ │ automati.│       │    │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘       │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                 │                                                │
│                                 ▼                                                │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐                │
│  │   MODUL 5       │   │   MODUL 6       │   │   MODUL 7       │                │
│  │   Primärenergie │   │   CO2-Emission  │   │   Sommer.       │                │
│  │   GEG Anlage 9  │   │   GEG § 20      │   │   Wärmeschutz   │                │
│  └─────────────────┘   └─────────────────┘   └─────────────────┘                │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## MODUL 1: U-Wert-Berechnung

### Normgrundlage
- **DIN EN ISO 6946**: Wärmedurchlasswiderstand und Wärmedurchgangskoeffizient
- **DIN 4108-4**: Wärme- und feuchtetechnische Bemessungswerte

### Berechnungsalgorithmus

```typescript
interface MaterialSchicht {
  material_id: string;
  name: string;
  dicke_m: number;           // Schichtdicke in Metern
  lambda: number;            // Wärmeleitfähigkeit [W/(m·K)]
  rho: number;               // Rohdichte [kg/m³]
  c: number;                 // spez. Wärmekapazität [J/(kg·K)]
}

interface UWertBerechnung {
  bauteil_id: string;
  schichten: MaterialSchicht[];
  Rsi: number;               // Wärmeübergangswiderstand innen
  Rse: number;               // Wärmeübergangswiderstand außen
}

function berechneUWert(bauteil: UWertBerechnung): number {
  // Schritt 1: Wärmedurchlasswiderstände der Schichten
  let R_gesamt = bauteil.Rsi + bauteil.Rse;

  for (const schicht of bauteil.schichten) {
    // R = d / λ
    const R_schicht = schicht.dicke_m / schicht.lambda;
    R_gesamt += R_schicht;
  }

  // Schritt 2: U-Wert = 1 / R_gesamt
  const U_wert = 1 / R_gesamt;

  // Runden auf 2 Dezimalstellen (normgerecht)
  return Math.round(U_wert * 100) / 100;
}
```

### Wärmeübergangswiderstände nach DIN EN ISO 6946

| Position | Richtung Wärmestrom | Rsi [m²K/W] | Rse [m²K/W] |
|----------|---------------------|-------------|-------------|
| Außenwand | horizontal | 0.13 | 0.04 |
| Dach | aufwärts | 0.10 | 0.04 |
| Kellerdecke | abwärts | 0.17 | 0.04 |
| Innenwand | horizontal | 0.13 | 0.13 |
| Gegen unbeheizt | horizontal | 0.13 | 0.13 |
| Gegen Erdreich | - | 0.17 | 0.00 |

### Sonderfälle

#### 1. Inhomogene Schichten (z.B. Sparrendach)
```typescript
function berechneUWertInhomogen(
  bereiche: { anteil: number; U: number }[]
): number {
  // Oberer Grenzwert (parallele Schaltung)
  let R_upper = 0;
  for (const bereich of bereiche) {
    R_upper += bereich.anteil / (1 / bereich.U);
  }
  R_upper = 1 / R_upper;

  // Unterer Grenzwert (Reihenschaltung) - vereinfacht
  let R_lower = 0;
  for (const bereich of bereiche) {
    R_lower += bereich.anteil * (1 / bereich.U);
  }

  // Mittelwert
  const R_mittel = (R_upper + R_lower) / 2;
  return 1 / R_mittel;
}
```

#### 2. Luftschichten
```typescript
const LUFTSCHICHT_R: Record<string, Record<number, number>> = {
  'ruhend': {
    5: 0.11,
    7: 0.13,
    10: 0.15,
    15: 0.16,
    25: 0.16,
    50: 0.16,
    100: 0.16,
    300: 0.16,
  },
  'schwach_belueftet': {
    // R = 0.15 m²K/W (vereinfacht)
  },
  'stark_belueftet': {
    // Nicht als Dämmschicht anrechenbar
  }
};
```

---

## MODUL 2: Wärmebrücken-Berechnung

### Normgrundlage
- **DIN 4108-2**: Mindestanforderungen an den Wärmeschutz
- **DIN 4108 Beiblatt 2**: Wärmebrückenkatalog

### Berechnungsverfahren

#### Verfahren 1: Pauschaler Zuschlag
```typescript
interface WaermebrueckenPauschal {
  verfahren: 'pauschal';
  qualitaet: 'ohne_nachweis' | 'optimiert' | 'gleichwertig';
}

function getWaermebrueckenZuschlag(
  wb: WaermebrueckenPauschal
): number {
  switch (wb.qualitaet) {
    case 'ohne_nachweis':
      return 0.10;  // ΔUWB = 0.10 W/(m²·K)
    case 'optimiert':
      return 0.05;  // ΔUWB = 0.05 W/(m²·K)
    case 'gleichwertig':
      return 0.03;  // ΔUWB = 0.03 W/(m²·K) (Beiblatt 2)
    default:
      return 0.10;
  }
}
```

#### Verfahren 2: Detaillierter Nachweis
```typescript
interface WaermebrueckeDetail {
  typ: WaermebrueckenTyp;
  laenge_m: number;
  psi_wert: number;           // Längenbezogener Wärmebrückenwert [W/(m·K)]
  chi_wert?: number;          // Punktbezogener Wärmebrückenwert [W/K]
}

type WaermebrueckenTyp =
  | 'aussenwand_bodenplatte'
  | 'aussenwand_decke'
  | 'aussenwand_innenwand'
  | 'aussenwand_fenster'
  | 'dach_aussenwand'
  | 'balkon_durchstoss'
  | 'attika'
  | 'fensterlaibung'
  | 'rolladenkasten';

interface WaermebrueckenKatalog {
  typ: WaermebrueckenTyp;
  konstruktion: string;
  psi_wert_referenz: number;
  bedingungen: string[];
}

const WAERMEBRUECKEN_KATALOG: WaermebrueckenKatalog[] = [
  {
    typ: 'aussenwand_bodenplatte',
    konstruktion: 'Außendämmung bis UK Bodenplatte',
    psi_wert_referenz: 0.10,
    bedingungen: [
      'Dämmung mindestens 1m unter Erdreich',
      'Perimeterdämmung ≥ 80mm'
    ]
  },
  {
    typ: 'aussenwand_fenster',
    konstruktion: 'Fenster in Dämmebene',
    psi_wert_referenz: 0.04,
    bedingungen: [
      'Dämmung Laibung ≥ 30mm',
      'Rahmenüberdeckung ≥ 30mm'
    ]
  },
  // ... weitere Einträge
];

function berechneWaermebrueckenVerlust(
  waermebruecken: WaermebrueckeDetail[],
  flaeche_A: number
): number {
  let summe_psi_l = 0;
  let summe_chi = 0;

  for (const wb of waermebruecken) {
    summe_psi_l += wb.psi_wert * wb.laenge_m;
    if (wb.chi_wert) {
      summe_chi += wb.chi_wert;
    }
  }

  // ΔUWB = (Σ ψ·l + Σ χ) / A
  const delta_UWB = (summe_psi_l + summe_chi) / flaeche_A;

  return delta_UWB;
}
```

---

## MODUL 3: Heizlastberechnung nach DIN EN 12831

### Normgrundlage
- **DIN EN 12831-1**: Raumweise Heizlast
- **DIN/TS 12831-1 Beiblatt 1**: Nationale Ergänzungen Deutschland

### Berechnungsstruktur

```typescript
interface HeizlastRaum {
  raum_id: string;
  name: string;
  theta_int_design: number;    // Raum-Solltemperatur [°C]
  A_floor: number;             // Grundfläche [m²]
  V_room: number;              // Raumvolumen [m³]
  n_min: number;               // Mindestluftwechsel [1/h]
}

interface HeizlastGebaeude {
  gebaeude_id: string;
  theta_ext_design: number;    // Auslegungsaußentemperatur [°C]
  raeume: HeizlastRaum[];
  bauteile: HeizlastBauteil[];
}

interface HeizlastBauteil {
  raum_id: string;
  bauteil_typ: 'aussenwand' | 'fenster' | 'dach' | 'boden' | 'innenwand';
  flaeche: number;
  U_wert: number;
  angrenzend: 'aussen' | 'erdreich' | 'unbeheizt' | 'beheizt';
  theta_angrenzend?: number;   // Temperatur angrenzender Raum/Zone
  f_temperatur?: number;       // Temperaturkorrekturfaktor
}
```

### Berechnungsalgorithmus

```typescript
function berechneRaumHeizlast(
  raum: HeizlastRaum,
  bauteile: HeizlastBauteil[],
  theta_e: number
): HeizlastErgebnis {
  const delta_theta = raum.theta_int_design - theta_e;

  // 1. Transmissionswärmeverluste
  let Phi_T = 0;
  const raumBauteile = bauteile.filter(b => b.raum_id === raum.raum_id);

  for (const bauteil of raumBauteile) {
    const f_temp = bauteil.f_temperatur ??
      berechneTemperaturfaktor(bauteil, raum.theta_int_design, theta_e);

    // H_T = A · U · f_temp
    const H_T = bauteil.flaeche * bauteil.U_wert * f_temp;
    Phi_T += H_T * delta_theta;
  }

  // 2. Wärmebrückenzuschlag
  const delta_UWB = 0.05;  // vereinfacht
  const A_env = raumBauteile
    .filter(b => b.angrenzend === 'aussen')
    .reduce((sum, b) => sum + b.flaeche, 0);
  Phi_T += A_env * delta_UWB * delta_theta;

  // 3. Lüftungswärmeverluste
  const rho_cp = 0.34;  // Wh/(m³·K)
  const V_min = raum.V_room * raum.n_min;
  const Phi_V = rho_cp * V_min * delta_theta;

  // 4. Aufheizzuschlag (optional)
  const f_RH = 1.0;  // kann je nach Nutzung variieren

  // 5. Gesamtheizlast
  const Phi_HL = (Phi_T + Phi_V) * f_RH;

  return {
    raum_id: raum.raum_id,
    Phi_T: Math.round(Phi_T),
    Phi_V: Math.round(Phi_V),
    Phi_HL: Math.round(Phi_HL),
    q_spez: Math.round(Phi_HL / raum.A_floor)  // W/m²
  };
}

function berechneTemperaturfaktor(
  bauteil: HeizlastBauteil,
  theta_int: number,
  theta_e: number
): number {
  switch (bauteil.angrenzend) {
    case 'aussen':
      return 1.0;
    case 'erdreich':
      // f_g1 · f_g2 · G_w nach DIN EN ISO 13370
      return 0.5;  // vereinfacht
    case 'unbeheizt':
      const theta_u = bauteil.theta_angrenzend ??
        (theta_int + theta_e) / 2;
      return (theta_int - theta_u) / (theta_int - theta_e);
    case 'beheizt':
      const theta_adj = bauteil.theta_angrenzend ?? theta_int;
      return (theta_int - theta_adj) / (theta_int - theta_e);
    default:
      return 1.0;
  }
}

interface HeizlastErgebnis {
  raum_id: string;
  Phi_T: number;       // Transmissionswärmeverlust [W]
  Phi_V: number;       // Lüftungswärmeverlust [W]
  Phi_HL: number;      // Norm-Heizlast [W]
  q_spez: number;      // spezifische Heizlast [W/m²]
}
```

### Klimadaten Deutschland (Auslegungsaußentemperaturen)

```typescript
const KLIMADATEN_DE: Record<string, number> = {
  // Auswahl wichtiger Städte
  'Berlin': -14,
  'Hamburg': -12,
  'München': -16,
  'Köln': -10,
  'Frankfurt': -12,
  'Stuttgart': -14,
  'Düsseldorf': -10,
  'Dortmund': -12,
  'Essen': -10,
  'Leipzig': -14,
  'Bremen': -12,
  'Dresden': -16,
  'Hannover': -14,
  'Nürnberg': -16,
  'Duisburg': -10,
  // ... weitere nach PLZ-Bereich
};

// Klimadaten Österreich
const KLIMADATEN_AT: Record<string, number> = {
  'Wien': -13,
  'Graz': -14,
  'Linz': -15,
  'Salzburg': -16,
  'Innsbruck': -16,
  'Klagenfurt': -18,
  'Bregenz': -13,
  // ...
};
```

---

## MODUL 4: Energiebilanz nach DIN V 18599

### Gesamtstruktur der Berechnung

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        ENERGIEBILANZ ABLAUF                                      │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  SCHRITT 1: ZONIERUNG                                                           │
│  ═══════════════════                                                            │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐                               │
│  │ Zone 1  │ │ Zone 2  │ │ Zone 3  │ │ Zone n  │                               │
│  │ Wohnen  │ │ Schlafen│ │ Gewerbe │ │ ...     │                               │
│  └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘                               │
│       └───────────┴───────────┴───────────┘                                     │
│                          │                                                       │
│  SCHRITT 2: NUTZENERGIEBEDARF (je Zone)                                         │
│  ══════════════════════════════════════                                         │
│       ┌──────────────────┴──────────────────┐                                   │
│       ▼                                      ▼                                   │
│  ┌─────────────┐                      ┌─────────────┐                           │
│  │ Q_h,b       │                      │ Q_tw,b      │                           │
│  │ Heizwärme   │                      │ Trinkwarm-  │                           │
│  │             │                      │ wasser      │                           │
│  └──────┬──────┘                      └──────┬──────┘                           │
│         │                                    │                                   │
│  SCHRITT 3: ANLAGENAUFWAND                   │                                   │
│  ═════════════════════════                   │                                   │
│         ▼                                    ▼                                   │
│  ┌─────────────────────────────────────────────────────────────────┐            │
│  │                    WÄRMEERZEUGER                                 │            │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐        │            │
│  │  │ Kessel │ │ WP     │ │ BHKW   │ │ Solar  │ │ Fern-  │        │            │
│  │  │        │ │        │ │        │ │        │ │ wärme  │        │            │
│  │  └────────┘ └────────┘ └────────┘ └────────┘ └────────┘        │            │
│  │                                                                  │            │
│  │  Berechnungen: e_p,g, Q_f, Q_p                                  │            │
│  └─────────────────────────────────────────────────────────────────┘            │
│                          │                                                       │
│  SCHRITT 4: ENDENERGIETRÄGER                                                    │
│  ═══════════════════════════                                                    │
│                          ▼                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐            │
│  │  Q_f,Heizung + Q_f,TWW + Q_f,Lüftung + Q_f,Kühlung + Q_f,Bel.   │            │
│  │  ═══════════════════════════════════════════════════════════════ │            │
│  │                        = Q_f,gesamt                              │            │
│  └─────────────────────────────────────────────────────────────────┘            │
│                          │                                                       │
│  SCHRITT 5: PRIMÄRENERGIE                                                       │
│  ════════════════════════                                                       │
│                          ▼                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐            │
│  │  Q_p = Σ (Q_f,i × f_p,i)                                        │            │
│  └─────────────────────────────────────────────────────────────────┘            │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Zonierung (DIN V 18599-1)

```typescript
interface ThermalZone {
  zone_id: string;
  name: string;
  nutzungsart: Nutzungsart;
  A_NGF: number;              // Nettogrundfläche [m²]
  V_Zone: number;             // Zonenvolumen [m³]
  theta_i_heating: number;    // Heizgrenztemperatur [°C]
  theta_i_cooling: number;    // Kühlgrenztemperatur [°C]
  n_Pers: number;             // Personenbelegung [Pers/m²]
  t_Nutz: number;             // Nutzungszeit [h/d]
  q_int: number;              // interne Wärmelast [W/m²]
}

type Nutzungsart =
  | 'wohnen'                  // Wohngebäude
  | 'buero_einzel'            // Einzelbüro
  | 'buero_gruppen'           // Gruppenbüro
  | 'buero_grossraum'         // Großraumbüro
  | 'konferenz'               // Besprechung/Sitzung
  | 'hoersaal'                // Hörsaal
  | 'restaurant'              // Restaurant
  | 'kantine'                 // Kantine
  | 'verkauf'                 // Einzelhandel
  | 'klassenzimmer'           // Klassenzimmer
  | 'lager'                   // Lager
  | 'wc'                      // WC/Sanitär
  | 'serverraum'              // Rechenzentrum
  | 'sport'                   // Sporthalle
  | 'schwimmbad'              // Schwimmbad
  | 'hotel';                  // Hotelzimmer

// Nutzungsprofile nach DIN V 18599-10
const NUTZUNGSPROFILE: Record<Nutzungsart, NutzungsProfil> = {
  'wohnen': {
    theta_i_h_soll: 20,
    theta_i_c_soll: 26,
    luftwechsel_n: 0.5,
    belegung_pers_m2: 0.03,
    q_int_pers: 80,
    q_int_geraete: 2,
    nutzungsstunden_tag: 24,
    nutzungstage_jahr: 365,
  },
  'buero_einzel': {
    theta_i_h_soll: 21,
    theta_i_c_soll: 26,
    luftwechsel_n: 1.2,
    belegung_pers_m2: 0.1,
    q_int_pers: 75,
    q_int_geraete: 15,
    nutzungsstunden_tag: 11,
    nutzungstage_jahr: 250,
  },
  // ... weitere Profile
};
```

### Heizwärmebedarf (DIN V 18599-2)

```typescript
interface HeizwaermebedarfInput {
  zone: ThermalZone;
  bauteile: EnvelopeComponent[];
  waermebruecken_delta_U: number;
  solargewinne: SolarGain[];
  interne_gewinne: number;
}

interface EnvelopeComponent {
  typ: 'opak' | 'transparent';
  orientierung: Orientation;
  A: number;                  // Fläche [m²]
  U: number;                  // U-Wert [W/(m²·K)]
  g?: number;                 // Gesamtenergiedurchlassgrad (nur Fenster)
  F_c?: number;               // Abminderung Sonnenschutz
  F_F?: number;               // Rahmenanteil (nur Fenster)
}

type Orientation = 'N' | 'NO' | 'O' | 'SO' | 'S' | 'SW' | 'W' | 'NW' | 'horizontal';

function berechneHeizwaermebedarf(
  input: HeizwaermebedarfInput,
  klimadaten: Klimadaten
): number {
  const { zone, bauteile, waermebruecken_delta_U } = input;

  // Monatliche Berechnung
  let Q_h_b_total = 0;

  for (let monat = 0; monat < 12; monat++) {
    const tage = TAGE_PRO_MONAT[monat];
    const theta_e = klimadaten.monatstemperatur[monat];
    const delta_theta = zone.theta_i_heating - theta_e;

    if (delta_theta <= 0) continue;  // kein Heizbedarf

    // 1. Transmissionswärmeverluste
    let H_T = 0;
    for (const bauteil of bauteile) {
      H_T += bauteil.A * bauteil.U;
    }
    // Wärmebrückenzuschlag
    const A_env = bauteile.reduce((s, b) => s + b.A, 0);
    H_T += A_env * waermebruecken_delta_U;

    const Q_T = H_T * delta_theta * tage * 24 / 1000;  // kWh

    // 2. Lüftungswärmeverluste
    const profil = NUTZUNGSPROFILE[zone.nutzungsart];
    const H_V = 0.34 * zone.V_Zone * profil.luftwechsel_n;
    const Q_V = H_V * delta_theta * tage * 24 / 1000;  // kWh

    // 3. Solare Gewinne
    const Q_s = berechneSolargewinne(bauteile, klimadaten, monat);

    // 4. Interne Gewinne
    const q_int = profil.q_int_pers * profil.belegung_pers_m2 +
                  profil.q_int_geraete;
    const Q_int = q_int * zone.A_NGF * tage *
                  profil.nutzungsstunden_tag / 1000;  // kWh

    // 5. Ausnutzungsgrad
    const Q_verlust = Q_T + Q_V;
    const Q_gewinn = Q_s + Q_int;
    const gamma = Q_gewinn / Q_verlust;

    // Zeitkonstante
    const C = zone.A_NGF * 50 * 1000;  // vereinfacht: 50 Wh/(m²·K)
    const tau = C / (H_T + H_V) / 3600;
    const a = 1 + tau / 15;

    const eta = gamma < 1
      ? (1 - Math.pow(gamma, a)) / (1 - Math.pow(gamma, a + 1))
      : a / (a + 1);

    // 6. Heizwärmebedarf
    const Q_h_b_monat = Math.max(0, Q_verlust - eta * Q_gewinn);
    Q_h_b_total += Q_h_b_monat;
  }

  return Q_h_b_total;  // kWh/a
}
```

### Trinkwarmwasserbedarf (DIN V 18599-8)

```typescript
interface TWWBedarf {
  nutzungsart: Nutzungsart;
  A_NGF: number;              // Bezugsfläche [m²]
  theta_W: number;            // Warmwassertemperatur [°C]
  theta_KW: number;           // Kaltwassertemperatur [°C]
}

function berechneTWWBedarf(input: TWWBedarf): number {
  const profil = TWW_PROFILE[input.nutzungsart];

  // Spezifischer Bedarf aus Profil
  const V_w_spez = profil.liter_pro_m2_tag;  // l/(m²·d)

  // Temperaturhub
  const delta_theta = input.theta_W - input.theta_KW;  // meist 35 K

  // Energiebedarf
  // Q_tw,b = V_w · ρ_w · c_w · Δθ · Tage
  const Q_tw_b = V_w_spez * input.A_NGF * 1 * 1.163 / 1000 *
                 delta_theta * 365;  // kWh/a

  return Q_tw_b;
}

const TWW_PROFILE: Record<Nutzungsart, { liter_pro_m2_tag: number }> = {
  'wohnen': { liter_pro_m2_tag: 0.8 },
  'buero_einzel': { liter_pro_m2_tag: 0.1 },
  'buero_gruppen': { liter_pro_m2_tag: 0.1 },
  'hotel': { liter_pro_m2_tag: 1.5 },
  'sport': { liter_pro_m2_tag: 2.0 },
  'schwimmbad': { liter_pro_m2_tag: 5.0 },
  // ...
};
```

---

## MODUL 5: Anlagentechnik

### Wärmeerzeuger-Kennwerte

```typescript
interface Waermeerzeuger {
  typ: WaermeerzeugerTyp;
  P_n: number;                // Nennleistung [kW]
  eta_n?: number;             // Nutzungsgrad bei Nennlast
  eta_100?: number;           // Wirkungsgrad bei Volllast
  eta_30?: number;            // Wirkungsgrad bei 30% Last
  baujahr?: number;
  energietraeger: Energietraeger;
}

type WaermeerzeugerTyp =
  | 'brennwert_gas'
  | 'brennwert_oel'
  | 'niedertemperatur_gas'
  | 'niedertemperatur_oel'
  | 'standard_gas'
  | 'standard_oel'
  | 'pellet'
  | 'hackschnitzel'
  | 'scheitholz'
  | 'waermepumpe_luft'
  | 'waermepumpe_sole'
  | 'waermepumpe_wasser'
  | 'fernwaerme'
  | 'elektro_direkt'
  | 'bhkw_gas'
  | 'bhkw_oel';

type Energietraeger =
  | 'erdgas'
  | 'heizoel'
  | 'fluessiggas'
  | 'steinkohle'
  | 'braunkohle'
  | 'holzpellets'
  | 'hackschnitzel'
  | 'scheitholz'
  | 'strom'
  | 'strom_waermepumpe'
  | 'fernwaerme_fossil'
  | 'fernwaerme_erneuerbar'
  | 'biogas'
  | 'biooel'
  | 'solarthermie'
  | 'umweltwaerme';
```

### Anlagenaufwandszahl

```typescript
interface AnlagenAufwand {
  erzeuger: Waermeerzeuger;
  speicher?: Waermespeicher;
  verteilung: Waermeverteilung;
  uebergabe: Waermeuebergabe;
}

function berechneAnlagenaufwandszahl(
  anlage: AnlagenAufwand,
  Q_h_b: number,
  Q_tw_b: number
): number {
  // 1. Erzeugeraufwandszahl
  const e_g = berechneErzeugeraufwand(anlage.erzeuger, Q_h_b + Q_tw_b);

  // 2. Speicheraufwandszahl
  const e_s = anlage.speicher
    ? berechneSpeicheraufwand(anlage.speicher)
    : 0;

  // 3. Verteilungsaufwandszahl
  const e_d = berechneVerteilungsaufwand(
    anlage.verteilung,
    anlage.uebergabe.temperatur_vorlauf
  );

  // 4. Übergabeaufwandszahl
  const e_ce = berechneUebergabeaufwand(anlage.uebergabe);

  // Gesamtaufwandszahl
  const e_P = (1 + e_s + e_d + e_ce) / e_g;

  return e_P;
}

function berechneErzeugeraufwand(
  erzeuger: Waermeerzeuger,
  Q_bedarf: number
): number {
  // Vereinfachte Berechnung basierend auf Kesseltyp
  const kennwerte: Record<WaermeerzeugerTyp, { eta: number }> = {
    'brennwert_gas': { eta: 0.97 },
    'brennwert_oel': { eta: 0.95 },
    'niedertemperatur_gas': { eta: 0.90 },
    'niedertemperatur_oel': { eta: 0.88 },
    'standard_gas': { eta: 0.85 },
    'standard_oel': { eta: 0.82 },
    'pellet': { eta: 0.88 },
    'waermepumpe_luft': { eta: 3.0 },    // JAZ
    'waermepumpe_sole': { eta: 4.0 },    // JAZ
    'waermepumpe_wasser': { eta: 4.5 },  // JAZ
    'fernwaerme': { eta: 1.0 },
    'elektro_direkt': { eta: 1.0 },
    // ...
  };

  return erzeuger.eta_n ?? kennwerte[erzeuger.typ]?.eta ?? 0.85;
}
```

---

## MODUL 6: Primärenergie- und CO2-Berechnung

### Primärenergiefaktoren (GEG Anlage 9)

```typescript
interface PrimaerenergieFaktor {
  energietraeger: Energietraeger;
  f_p_total: number;          // Primärenergiefaktor nicht erneuerbar
  f_p_ges: number;            // Primärenergiefaktor gesamt
  f_CO2: number;              // CO2-Emissionsfaktor [g/kWh]
}

const PRIMAERENERGIEFAKTOREN_GEG_2024: PrimaerenergieFaktor[] = [
  { energietraeger: 'erdgas', f_p_total: 1.1, f_p_ges: 1.1, f_CO2: 201 },
  { energietraeger: 'heizoel', f_p_total: 1.1, f_p_ges: 1.1, f_CO2: 266 },
  { energietraeger: 'fluessiggas', f_p_total: 1.1, f_p_ges: 1.1, f_CO2: 234 },
  { energietraeger: 'steinkohle', f_p_total: 1.1, f_p_ges: 1.1, f_CO2: 337 },
  { energietraeger: 'braunkohle', f_p_total: 1.2, f_p_ges: 1.2, f_CO2: 407 },
  { energietraeger: 'holzpellets', f_p_total: 0.2, f_p_ges: 1.2, f_CO2: 23 },
  { energietraeger: 'hackschnitzel', f_p_total: 0.2, f_p_ges: 1.2, f_CO2: 20 },
  { energietraeger: 'scheitholz', f_p_total: 0.2, f_p_ges: 1.2, f_CO2: 17 },
  { energietraeger: 'strom', f_p_total: 1.8, f_p_ges: 2.4, f_CO2: 420 },
  { energietraeger: 'strom_waermepumpe', f_p_total: 1.8, f_p_ges: 2.4, f_CO2: 420 },
  { energietraeger: 'fernwaerme_fossil', f_p_total: 0.7, f_p_ges: 1.3, f_CO2: 180 },
  { energietraeger: 'fernwaerme_erneuerbar', f_p_total: 0.0, f_p_ges: 1.0, f_CO2: 0 },
  { energietraeger: 'biogas', f_p_total: 0.5, f_p_ges: 1.5, f_CO2: 100 },
  { energietraeger: 'solarthermie', f_p_total: 0.0, f_p_ges: 1.0, f_CO2: 0 },
  { energietraeger: 'umweltwaerme', f_p_total: 0.0, f_p_ges: 1.0, f_CO2: 0 },
];

function berechnePrimaerenergie(
  endenergien: { energietraeger: Energietraeger; Q_f: number }[]
): PrimaerenergieBilanz {
  let Q_p_total = 0;
  let Q_p_ges = 0;
  let CO2_total = 0;

  const details: EndenergieDetail[] = [];

  for (const endenergie of endenergien) {
    const faktor = PRIMAERENERGIEFAKTOREN_GEG_2024.find(
      f => f.energietraeger === endenergie.energietraeger
    );

    if (!faktor) {
      throw new Error(`Unbekannter Energieträger: ${endenergie.energietraeger}`);
    }

    const Q_p_i = endenergie.Q_f * faktor.f_p_total;
    const Q_p_ges_i = endenergie.Q_f * faktor.f_p_ges;
    const CO2_i = endenergie.Q_f * faktor.f_CO2 / 1000;  // kg

    Q_p_total += Q_p_i;
    Q_p_ges += Q_p_ges_i;
    CO2_total += CO2_i;

    details.push({
      energietraeger: endenergie.energietraeger,
      Q_f: endenergie.Q_f,
      f_p: faktor.f_p_total,
      Q_p: Q_p_i,
      CO2: CO2_i
    });
  }

  return {
    Q_p_total,       // kWh/a - Primärenergie nicht erneuerbar
    Q_p_ges,         // kWh/a - Primärenergie gesamt
    CO2_total,       // kg/a - CO2-Emissionen
    details
  };
}

interface PrimaerenergieBilanz {
  Q_p_total: number;
  Q_p_ges: number;
  CO2_total: number;
  details: EndenergieDetail[];
}

interface EndenergieDetail {
  energietraeger: Energietraeger;
  Q_f: number;
  f_p: number;
  Q_p: number;
  CO2: number;
}
```

### Energieeffizienzklasse

```typescript
type Effizienzklasse = 'A+' | 'A' | 'B' | 'C' | 'D' | 'E' | 'F' | 'G' | 'H';

interface EffizienzklasseGrenzen {
  klasse: Effizienzklasse;
  von: number;
  bis: number;
}

const EFFIZIENZKLASSEN: EffizienzklasseGrenzen[] = [
  { klasse: 'A+', von: 0, bis: 30 },
  { klasse: 'A', von: 30, bis: 50 },
  { klasse: 'B', von: 50, bis: 75 },
  { klasse: 'C', von: 75, bis: 100 },
  { klasse: 'D', von: 100, bis: 130 },
  { klasse: 'E', von: 130, bis: 160 },
  { klasse: 'F', von: 160, bis: 200 },
  { klasse: 'G', von: 200, bis: 250 },
  { klasse: 'H', von: 250, bis: Infinity },
];

function ermittleEffizienzklasse(
  q_E_spez: number  // kWh/(m²·a) Endenergie
): Effizienzklasse {
  const klasse = EFFIZIENZKLASSEN.find(
    k => q_E_spez > k.von && q_E_spez <= k.bis
  );
  return klasse?.klasse ?? 'H';
}
```

---

## MODUL 7: Sommerlicher Wärmeschutz

### Normgrundlage
- **DIN 4108-2**: Sommerlicher Wärmeschutz

### Berechnungsverfahren

```typescript
interface SommerWaermeschutzInput {
  raum_id: string;
  A_NGF: number;                    // Nettogrundfläche [m²]
  fenster: FensterSommer[];
  klimaregion: KlimaregionSommer;
  bauart: 'leicht' | 'mittel' | 'schwer';
  nachtlueftung: boolean;
  passive_kuehlung?: boolean;
}

interface FensterSommer {
  orientierung: Orientation;
  A_w: number;                      // Fensterfläche [m²]
  g_total: number;                  // g-Wert inkl. Sonnenschutz
  F_c: number;                      // Abminderungsfaktor Sonnenschutz
  sonnenschutz_typ: SonnenschutzTyp;
}

type KlimaregionSommer = 'A' | 'B' | 'C';
type SonnenschutzTyp = 'aussen' | 'zwischen' | 'innen' | 'kein';

// Sonneneinstrahlungs-Kennwerte nach DIN 4108-2
const SONNENEINSTRAHLUNG: Record<Orientation, Record<KlimaregionSommer, number>> = {
  'N':  { A: 85, B: 95, C: 105 },
  'NO': { A: 145, B: 160, C: 175 },
  'O':  { A: 275, B: 305, C: 340 },
  'SO': { A: 360, B: 400, C: 445 },
  'S':  { A: 450, B: 500, C: 555 },
  'SW': { A: 360, B: 400, C: 445 },
  'W':  { A: 275, B: 305, C: 340 },
  'NW': { A: 145, B: 160, C: 175 },
  'horizontal': { A: 570, B: 635, C: 705 },
};

function berechneSommerWaermeschutz(
  input: SommerWaermeschutzInput
): SommerWaermeschutzErgebnis {
  // 1. Sonneneintragskennwert berechnen
  let S_vorh = 0;

  for (const fenster of input.fenster) {
    const I = SONNENEINSTRAHLUNG[fenster.orientierung][input.klimaregion];
    const g_eff = fenster.g_total * fenster.F_c;
    S_vorh += fenster.A_w * g_eff * I;
  }

  S_vorh = S_vorh / input.A_NGF;

  // 2. Zulässiger Sonneneintragskennwert
  let S_zul = ermittleSZul(input);

  // 3. Nachweis
  const nachweis_erfuellt = S_vorh <= S_zul;

  return {
    S_vorh,
    S_zul,
    nachweis_erfuellt,
    massnahmen_erforderlich: !nachweis_erfuellt
      ? empfehleMassnahmen(input, S_vorh - S_zul)
      : []
  };
}

function ermittleSZul(input: SommerWaermeschutzInput): number {
  // Grundwert abhängig von Klimaregion
  const basiswerte: Record<KlimaregionSommer, number> = {
    'A': 0.038,
    'B': 0.034,
    'C': 0.029,
  };

  let S_zul = basiswerte[input.klimaregion];

  // Zuschlag für Nachtlüftung
  if (input.nachtlueftung) {
    S_zul *= 1.2;
  }

  // Zuschlag für schwere Bauart
  if (input.bauart === 'schwer') {
    S_zul *= 1.1;
  }

  // Zuschlag für passive Kühlung
  if (input.passive_kuehlung) {
    S_zul *= 1.15;
  }

  return S_zul;
}

interface SommerWaermeschutzErgebnis {
  S_vorh: number;
  S_zul: number;
  nachweis_erfuellt: boolean;
  massnahmen_erforderlich: string[];
}
```

---

## MODUL 8: Referenzgebäudeverfahren (GEG)

### GEG-Anforderungen

```typescript
interface GEGAnforderungen {
  gebaeude_id: string;
  baujahr_neubau: boolean;
  gebaeudekategorie: Gebaeudekategorie;
}

type Gebaeudekategorie =
  | 'wohngebaeude'
  | 'nichtwohngebaeude'
  | 'gemischt';

interface Referenzgebaeude {
  // Geometrie wie Ist-Gebäude
  A_N: number;                      // Nutzfläche [m²]
  V_e: number;                      // beheiztes Volumen [m³]
  A_huelle: number;                 // Hüllfläche [m²]

  // Referenz-U-Werte (GEG Anlage 1)
  U_wand: number;
  U_dach: number;
  U_boden: number;
  U_fenster: number;
  U_tuer: number;
  g_fenster: number;

  // Referenz-Anlagentechnik
  heizung: 'brennwert_gas';
  tww: 'brennwert_gas_zentral';
  lueftung: 'abluft_bedarfsgefuehrt';
}

const REFERENZ_U_WERTE_WOHNGEBAEUDE: Record<string, number> = {
  'aussenwand': 0.28,
  'dach_oberste_decke': 0.20,
  'boden_erdberuehrt': 0.35,
  'boden_gegen_aussen': 0.28,
  'fenster': 1.30,
  'dachfenster': 1.40,
  'haustuer': 1.80,
  'glasdach': 2.00,
};

function berechneReferenzgebaeude(
  istGebaeude: Gebaeude
): number {
  // Referenzgebäude hat gleiche Geometrie
  const ref: Referenzgebaeude = {
    A_N: istGebaeude.A_N,
    V_e: istGebaeude.V_e,
    A_huelle: istGebaeude.A_huelle,
    U_wand: REFERENZ_U_WERTE_WOHNGEBAEUDE['aussenwand'],
    U_dach: REFERENZ_U_WERTE_WOHNGEBAEUDE['dach_oberste_decke'],
    U_boden: REFERENZ_U_WERTE_WOHNGEBAEUDE['boden_erdberuehrt'],
    U_fenster: REFERENZ_U_WERTE_WOHNGEBAEUDE['fenster'],
    U_tuer: REFERENZ_U_WERTE_WOHNGEBAEUDE['haustuer'],
    g_fenster: 0.60,
    heizung: 'brennwert_gas',
    tww: 'brennwert_gas_zentral',
    lueftung: 'abluft_bedarfsgefuehrt'
  };

  // Berechne Primärenergiebedarf Referenzgebäude
  const Q_p_ref = berechneEnergiebedarfReferenz(ref, istGebaeude);

  return Q_p_ref;
}

function pruefeGEGAnforderungen(
  Q_p_ist: number,
  Q_p_ref: number,
  gebaeude: GEGAnforderungen
): GEGPruefungErgebnis {
  // Für Neubau: Q_p ≤ Q_p,ref
  // Für Bestand: Q_p ≤ 1,4 × Q_p,ref (bei Sanierung)

  const grenzwert = gebaeude.baujahr_neubau
    ? Q_p_ref
    : Q_p_ref * 1.4;

  const erfuellt = Q_p_ist <= grenzwert;

  return {
    Q_p_ist,
    Q_p_ref,
    grenzwert,
    erfuellt,
    abweichung_prozent: ((Q_p_ist - grenzwert) / grenzwert) * 100
  };
}

interface GEGPruefungErgebnis {
  Q_p_ist: number;
  Q_p_ref: number;
  grenzwert: number;
  erfuellt: boolean;
  abweichung_prozent: number;
}
```

---

## MODUL 9: Wirtschaftlichkeitsberechnung

### Lebenszykluskostenanalyse

```typescript
interface WirtschaftlichkeitsInput {
  investition: number;              // Investitionskosten [€]
  foerderung: number;               // Fördersumme [€]
  einsparung_energie: number;       // jährliche Einsparung [kWh]
  energiepreis_aktuell: number;     // aktueller Preis [€/kWh]
  preissteigerung: number;          // jährliche Preissteigerung [%]
  lebensdauer: number;              // Nutzungsdauer [Jahre]
  zinssatz: number;                 // Kalkulationszins [%]
  wartungskosten: number;           // jährliche Wartung [€]
}

interface WirtschaftlichkeitsErgebnis {
  amortisationszeit: number;        // Jahre
  kapitalwert: number;              // NPV [€]
  interner_zinsfuss: number;        // IRR [%]
  co2_vermeidungskosten: number;    // €/t CO2
  jaehrliche_ersparnis: number;     // €/a
}

function berechneWirtschaftlichkeit(
  input: WirtschaftlichkeitsInput
): WirtschaftlichkeitsErgebnis {
  const eigenanteil = input.investition - input.foerderung;

  // Jährliche Cashflows berechnen
  const cashflows: number[] = [-eigenanteil];
  let kumuliert = -eigenanteil;
  let amortisationszeit = input.lebensdauer;

  for (let jahr = 1; jahr <= input.lebensdauer; jahr++) {
    // Energiepreis mit Preissteigerung
    const preis = input.energiepreis_aktuell *
      Math.pow(1 + input.preissteigerung / 100, jahr);

    // Jährliche Ersparnis
    const ersparnis = input.einsparung_energie * preis;
    const cashflow = ersparnis - input.wartungskosten;
    cashflows.push(cashflow);

    // Amortisation
    kumuliert += cashflow;
    if (kumuliert >= 0 && amortisationszeit === input.lebensdauer) {
      amortisationszeit = jahr;
    }
  }

  // Kapitalwert (NPV)
  const kapitalwert = berechneNPV(cashflows, input.zinssatz / 100);

  // Interner Zinsfuß (IRR)
  const irr = berechneIRR(cashflows);

  // Durchschnittliche jährliche Ersparnis
  const avg_ersparnis = cashflows.slice(1).reduce((s, c) => s + c, 0) /
    input.lebensdauer;

  return {
    amortisationszeit,
    kapitalwert: Math.round(kapitalwert),
    interner_zinsfuss: Math.round(irr * 100) / 100,
    co2_vermeidungskosten: 0,  // wird separat berechnet
    jaehrliche_ersparnis: Math.round(avg_ersparnis)
  };
}

function berechneNPV(cashflows: number[], r: number): number {
  return cashflows.reduce((npv, cf, t) => {
    return npv + cf / Math.pow(1 + r, t);
  }, 0);
}

function berechneIRR(cashflows: number[]): number {
  // Newton-Raphson-Verfahren
  let irr = 0.1;  // Startwert 10%

  for (let i = 0; i < 100; i++) {
    const npv = berechneNPV(cashflows, irr);
    const d_npv = cashflows.reduce((sum, cf, t) => {
      return sum - t * cf / Math.pow(1 + irr, t + 1);
    }, 0);

    const irr_new = irr - npv / d_npv;

    if (Math.abs(irr_new - irr) < 0.0001) {
      return irr_new * 100;
    }
    irr = irr_new;
  }

  return irr * 100;
}
```

---

## MODUL 10: Photovoltaik-Berechnung

### Ertragsprognose

```typescript
interface PVAnlage {
  module: PVModul[];
  wechselrichter: Wechselrichter;
  standort: Standort;
}

interface PVModul {
  P_peak: number;                   // Nennleistung [Wp]
  anzahl: number;
  neigung: number;                  // Grad
  azimut: number;                   // Grad (0=Süd, 90=West, -90=Ost)
  flaeche: number;                  // m² pro Modul
  eta_stc: number;                  // Wirkungsgrad unter STC
  temp_koeff: number;               // %/K
  degradation: number;              // %/Jahr
}

interface Standort {
  latitude: number;
  longitude: number;
  globalstrahlung: number;          // kWh/(m²·a)
}

function berechnePVErtrag(anlage: PVAnlage): PVErgebnis {
  let jahresertrag = 0;
  const monatsertraege: number[] = [];

  for (const modul of anlage.module) {
    // 1. Einstrahlungskorrektur für Neigung und Azimut
    const f_orient = berechneOrientierungsfaktor(
      modul.neigung,
      modul.azimut,
      anlage.standort.latitude
    );

    // 2. Globalstrahlung auf Modulebene
    const H_modul = anlage.standort.globalstrahlung * f_orient;

    // 3. Performance Ratio (typisch 0.75-0.85)
    const PR = 0.80;

    // 4. Jahresertrag
    const P_peak_total = modul.P_peak * modul.anzahl / 1000;  // kWp
    const E_jahr = P_peak_total * H_modul * PR / 1000;  // kWh

    jahresertrag += E_jahr;
  }

  // Spezifischer Ertrag
  const P_peak_gesamt = anlage.module.reduce(
    (sum, m) => sum + m.P_peak * m.anzahl, 0
  ) / 1000;  // kWp

  const spezifischer_ertrag = jahresertrag / P_peak_gesamt;  // kWh/kWp

  // Eigenverbrauchsquote (vereinfacht)
  const eigenverbrauch = schaetzeEigenverbrauch(
    jahresertrag,
    anlage
  );

  return {
    jahresertrag: Math.round(jahresertrag),
    spezifischer_ertrag: Math.round(spezifischer_ertrag),
    eigenverbrauchsquote: eigenverbrauch,
    P_peak_gesamt,
    monatsertraege
  };
}

function berechneOrientierungsfaktor(
  neigung: number,
  azimut: number,
  latitude: number
): number {
  // Vereinfachte Berechnung
  // Optimum in Deutschland: ca. 35° Neigung, Süd

  const opt_neigung = latitude - 15;  // Faustregel
  const delta_neigung = Math.abs(neigung - opt_neigung);
  const delta_azimut = Math.abs(azimut);

  // Reduktionsfaktoren
  const f_neigung = 1 - 0.005 * delta_neigung;
  const f_azimut = Math.cos(delta_azimut * Math.PI / 180);

  return Math.max(0.5, f_neigung * f_azimut);
}

interface PVErgebnis {
  jahresertrag: number;             // kWh/a
  spezifischer_ertrag: number;      // kWh/kWp
  eigenverbrauchsquote: number;     // %
  P_peak_gesamt: number;            // kWp
  monatsertraege: number[];
}
```

---

## Validierung und Plausibilitätsprüfung

### Automatische Prüfregeln

```typescript
interface Validierungsregel {
  id: string;
  beschreibung: string;
  schwere: 'fehler' | 'warnung' | 'hinweis';
  pruefung: (daten: any) => ValidierungsErgebnis;
}

const VALIDIERUNGSREGELN: Validierungsregel[] = [
  {
    id: 'U_WERT_BEREICH',
    beschreibung: 'U-Wert liegt außerhalb des plausiblen Bereichs',
    schwere: 'fehler',
    pruefung: (bauteil) => {
      const grenzen: Record<string, { min: number; max: number }> = {
        'aussenwand': { min: 0.10, max: 3.0 },
        'dach': { min: 0.10, max: 2.0 },
        'fenster': { min: 0.50, max: 6.0 },
        'boden': { min: 0.15, max: 2.0 },
      };
      const grenze = grenzen[bauteil.typ];
      if (grenze && (bauteil.U < grenze.min || bauteil.U > grenze.max)) {
        return {
          gueltig: false,
          meldung: `U-Wert ${bauteil.U} liegt außerhalb ${grenze.min}-${grenze.max} W/(m²·K)`
        };
      }
      return { gueltig: true };
    }
  },
  {
    id: 'HEIZLAST_SPEZIFISCH',
    beschreibung: 'Spezifische Heizlast unplausibel',
    schwere: 'warnung',
    pruefung: (raum) => {
      const q_spez = raum.Phi_HL / raum.A_floor;
      if (q_spez < 20 || q_spez > 200) {
        return {
          gueltig: false,
          meldung: `Spezifische Heizlast ${q_spez} W/m² unüblich (normal: 20-200 W/m²)`
        };
      }
      return { gueltig: true };
    }
  },
  {
    id: 'FLAECHEN_KONSISTENZ',
    beschreibung: 'Flächensummen inkonsistent',
    schwere: 'fehler',
    pruefung: (gebaeude) => {
      const A_NGF = gebaeude.zonen.reduce((s, z) => s + z.A_NGF, 0);
      const A_BGF = gebaeude.A_BGF;
      const verhaeltnis = A_NGF / A_BGF;

      if (verhaeltnis < 0.7 || verhaeltnis > 0.95) {
        return {
          gueltig: false,
          meldung: `NGF/BGF-Verhältnis ${verhaeltnis.toFixed(2)} unplausibel (normal: 0.70-0.95)`
        };
      }
      return { gueltig: true };
    }
  },
  {
    id: 'PRIMAERENERGIE_GRENZWERT',
    beschreibung: 'Primärenergie überschreitet GEG-Anforderung',
    schwere: 'fehler',
    pruefung: (ergebnis) => {
      if (ergebnis.Q_p_ist > ergebnis.Q_p_ref) {
        return {
          gueltig: false,
          meldung: `Primärenergiebedarf ${ergebnis.Q_p_ist} kWh/(m²·a) überschreitet Referenzwert ${ergebnis.Q_p_ref} kWh/(m²·a)`
        };
      }
      return { gueltig: true };
    }
  },
  {
    id: 'ENERGIETRAEGER_VOLLSTAENDIG',
    beschreibung: 'Nicht alle Energieträger erfasst',
    schwere: 'fehler',
    pruefung: (anlage) => {
      const hatHeizung = anlage.waermeerzeuger?.length > 0;
      const hatTWW = anlage.tww_bereiter?.length > 0 ||
                     anlage.waermeerzeuger?.some(w => w.auch_tww);

      if (!hatHeizung) {
        return {
          gueltig: false,
          meldung: 'Kein Wärmeerzeuger für Heizung definiert'
        };
      }
      if (!hatTWW) {
        return {
          gueltig: false,
          meldung: 'Keine Trinkwarmwasserbereitung definiert'
        };
      }
      return { gueltig: true };
    }
  }
];

interface ValidierungsErgebnis {
  gueltig: boolean;
  meldung?: string;
}

function validiereBerechnung(daten: any): ValidierungsErgebnis[] {
  const ergebnisse: ValidierungsErgebnis[] = [];

  for (const regel of VALIDIERUNGSREGELN) {
    try {
      const ergebnis = regel.pruefung(daten);
      if (!ergebnis.gueltig) {
        ergebnisse.push({
          ...ergebnis,
          regel_id: regel.id,
          schwere: regel.schwere
        } as any);
      }
    } catch (e) {
      // Regel konnte nicht geprüft werden
    }
  }

  return ergebnisse;
}
```

---

## API-Schnittstellen

### Berechnungs-API

```typescript
// POST /api/v1/calculations/u-wert
interface UWertRequest {
  bauteil_id: string;
  schichten: MaterialSchicht[];
  Rsi?: number;
  Rse?: number;
}

interface UWertResponse {
  U_wert: number;
  R_gesamt: number;
  schichten_details: {
    material: string;
    d: number;
    lambda: number;
    R: number;
  }[];
}

// POST /api/v1/calculations/heizlast
interface HeizlastRequest {
  gebaeude_id: string;
  standort_plz?: string;
  theta_ext_design?: number;
}

interface HeizlastResponse {
  Phi_HL_gesamt: number;
  raeume: HeizlastErgebnis[];
  zusammenfassung: {
    Phi_T_gesamt: number;
    Phi_V_gesamt: number;
    q_spez_mittel: number;
  };
}

// POST /api/v1/calculations/energiebilanz
interface EnergiebilanzRequest {
  gebaeude_id: string;
  version_id?: string;          // Ist-Zustand oder Sanierungsvariante
  klimadaten_id?: string;
}

interface EnergiebilanzResponse {
  Q_h_b: number;                // Heizwärmebedarf [kWh/a]
  Q_tw_b: number;               // Trinkwarmwasserbedarf [kWh/a]
  Q_f_gesamt: number;           // Endenergiebedarf [kWh/a]
  Q_p_gesamt: number;           // Primärenergiebedarf [kWh/a]
  CO2_emissionen: number;       // [kg/a]
  effizienzklasse: Effizienzklasse;
  q_p_spez: number;             // [kWh/(m²·a)]
  q_E_spez: number;             // [kWh/(m²·a)]
  monatswerte: MonatsErgebnis[];
  geg_nachweis: GEGPruefungErgebnis;
}
```

---

## Berechnungsreihenfolge und Abhängigkeiten

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    BERECHNUNGSREIHENFOLGE                                        │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  1. GEOMETRIE & ZONIERUNG                                                       │
│     └─→ Flächen, Volumina, Zonen definieren                                     │
│                                                                                  │
│  2. BAUTEIL U-WERTE                                                             │
│     └─→ Alle U-Werte berechnen                                                  │
│         └─→ Abhängig von: Materialien, Schichtaufbau                            │
│                                                                                  │
│  3. WÄRMEBRÜCKEN                                                                │
│     └─→ ΔU_WB oder Detailnachweis                                               │
│         └─→ Abhängig von: Bauteilanschlüsse, Konstruktionsdetails               │
│                                                                                  │
│  4. HEIZLAST (optional für Heizungsauslegung)                                   │
│     └─→ Raumweise Heizlast                                                      │
│         └─→ Abhängig von: U-Werte, Klimadaten, Raumtemperaturen                 │
│                                                                                  │
│  5. NUTZENERGIEBEDARF                                                           │
│     ├─→ Heizwärmebedarf Q_h,b                                                   │
│     │   └─→ Abhängig von: U-Werte, Flächen, Klimadaten, Gewinne                 │
│     └─→ Trinkwarmwasserbedarf Q_tw,b                                            │
│         └─→ Abhängig von: Nutzungsprofil, Fläche                                │
│                                                                                  │
│  6. ANLAGENTECHNIK                                                              │
│     └─→ Aufwandszahlen, Verluste                                                │
│         └─→ Abhängig von: Erzeuger, Speicher, Verteilung                        │
│                                                                                  │
│  7. ENDENERGIEBEDARF                                                            │
│     └─→ Q_f = Q_nutz × Anlagenaufwand                                           │
│         └─→ Abhängig von: Nutzenergie, Anlagentechnik                           │
│                                                                                  │
│  8. PRIMÄRENERGIE & CO2                                                         │
│     └─→ Q_p = Q_f × f_p                                                         │
│         └─→ Abhängig von: Endenergie, Energieträger                             │
│                                                                                  │
│  9. GEG-NACHWEIS                                                                │
│     └─→ Vergleich mit Referenzgebäude                                           │
│         └─→ Abhängig von: Alle vorherigen Berechnungen                          │
│                                                                                  │
│  10. WIRTSCHAFTLICHKEIT (optional)                                              │
│      └─→ Amortisation, NPV, IRR                                                 │
│          └─→ Abhängig von: Investition, Einsparung, Förderung                   │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```
