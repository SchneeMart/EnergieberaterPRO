# Konsistenz-Prüfungen und Validierungsregeln

## Übersicht der Validierungsebenen

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        VALIDIERUNGSEBENEN                                        │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  EBENE 1: EINGABEVALIDIERUNG (Sofort bei Eingabe)                               │
│  ═══════════════════════════════════════════════                                │
│  • Pflichtfelder vorhanden                                                      │
│  • Datentypen korrekt                                                           │
│  • Wertebereiche eingehalten                                                    │
│  • Format korrekt (PLZ, Datum, etc.)                                            │
│                                                                                  │
│  EBENE 2: LOGISCHE KONSISTENZ (Bei Speichern)                                   │
│  ════════════════════════════════════════════                                   │
│  • Abhängigkeiten zwischen Feldern                                              │
│  • Summen und Bezüge stimmen                                                    │
│  • Geometrische Plausibilität                                                   │
│                                                                                  │
│  EBENE 3: FACHLICHE PLAUSIBILITÄT (Bei Berechnung)                              │
│  ═══════════════════════════════════════════════                                │
│  • Physikalisch sinnvolle Werte                                                 │
│  • Normkonforme Grenzen                                                         │
│  • Vergleich mit Referenzwerten                                                 │
│                                                                                  │
│  EBENE 4: NORMKONFORMITÄT (Bei Export/Zertifizierung)                           │
│  ════════════════════════════════════════════════                               │
│  • GEG-Anforderungen erfüllt                                                    │
│  • Alle Pflichtangaben vorhanden                                                │
│  • Berechnungsmethodik korrekt                                                  │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Ebene 1: Eingabevalidierung

### Allgemeine Pflichtfeld-Regeln

```typescript
interface FeldValidierung {
  feld: string;
  pflicht: boolean;
  typ: 'text' | 'number' | 'date' | 'enum' | 'array';
  min?: number;
  max?: number;
  pattern?: RegExp;
  enum_werte?: string[];
  fehlermeldung: string;
}

const GEBAEUDE_VALIDIERUNG: FeldValidierung[] = [
  // Adresse
  {
    feld: 'strasse',
    pflicht: true,
    typ: 'text',
    min: 3,
    max: 100,
    fehlermeldung: 'Straße muss zwischen 3 und 100 Zeichen lang sein'
  },
  {
    feld: 'hausnummer',
    pflicht: true,
    typ: 'text',
    pattern: /^[0-9]+[a-zA-Z]?(-[0-9]+[a-zA-Z]?)?$/,
    fehlermeldung: 'Ungültiges Hausnummer-Format (z.B. 12, 12a, 12-14)'
  },
  {
    feld: 'plz',
    pflicht: true,
    typ: 'text',
    pattern: /^[0-9]{4,5}$/,
    fehlermeldung: 'PLZ muss 4 (AT) oder 5 (DE) Ziffern haben'
  },
  {
    feld: 'ort',
    pflicht: true,
    typ: 'text',
    min: 2,
    max: 100,
    fehlermeldung: 'Ort muss zwischen 2 und 100 Zeichen lang sein'
  },
  {
    feld: 'land',
    pflicht: true,
    typ: 'enum',
    enum_werte: ['DE', 'AT'],
    fehlermeldung: 'Land muss DE oder AT sein'
  },

  // Gebäudedaten
  {
    feld: 'baujahr',
    pflicht: true,
    typ: 'number',
    min: 1800,
    max: new Date().getFullYear(),
    fehlermeldung: 'Baujahr muss zwischen 1800 und heute liegen'
  },
  {
    feld: 'gebaeude_typ',
    pflicht: true,
    typ: 'enum',
    enum_werte: ['EFH', 'DHH', 'RH', 'MFH', 'GWNB'],
    fehlermeldung: 'Gebäudetyp muss EFH, DHH, RH, MFH oder GWNB sein'
  },
  {
    feld: 'wohneinheiten',
    pflicht: true,
    typ: 'number',
    min: 1,
    max: 500,
    fehlermeldung: 'Anzahl Wohneinheiten muss zwischen 1 und 500 liegen'
  },

  // Flächen
  {
    feld: 'wohnflaeche',
    pflicht: true,
    typ: 'number',
    min: 20,
    max: 10000,
    fehlermeldung: 'Wohnfläche muss zwischen 20 und 10.000 m² liegen'
  },
  {
    feld: 'nutzflaeche',
    pflicht: false,
    typ: 'number',
    min: 20,
    max: 50000,
    fehlermeldung: 'Nutzfläche muss zwischen 20 und 50.000 m² liegen'
  },
];

// Validator-Funktion
function validiereEingabe(
  daten: Record<string, any>,
  regeln: FeldValidierung[]
): ValidationResult[] {
  const ergebnisse: ValidationResult[] = [];

  for (const regel of regeln) {
    const wert = daten[regel.feld];

    // Pflichtfeld-Prüfung
    if (regel.pflicht && (wert === undefined || wert === null || wert === '')) {
      ergebnisse.push({
        feld: regel.feld,
        gueltig: false,
        meldung: `${regel.feld} ist ein Pflichtfeld`,
        schwere: 'fehler'
      });
      continue;
    }

    if (wert === undefined || wert === null) continue;

    // Typ-spezifische Prüfungen
    switch (regel.typ) {
      case 'number':
        if (typeof wert !== 'number' || isNaN(wert)) {
          ergebnisse.push({
            feld: regel.feld,
            gueltig: false,
            meldung: `${regel.feld} muss eine Zahl sein`,
            schwere: 'fehler'
          });
        } else {
          if (regel.min !== undefined && wert < regel.min) {
            ergebnisse.push({
              feld: regel.feld,
              gueltig: false,
              meldung: regel.fehlermeldung,
              schwere: 'fehler'
            });
          }
          if (regel.max !== undefined && wert > regel.max) {
            ergebnisse.push({
              feld: regel.feld,
              gueltig: false,
              meldung: regel.fehlermeldung,
              schwere: 'fehler'
            });
          }
        }
        break;

      case 'text':
        if (typeof wert !== 'string') {
          ergebnisse.push({
            feld: regel.feld,
            gueltig: false,
            meldung: `${regel.feld} muss ein Text sein`,
            schwere: 'fehler'
          });
        } else {
          if (regel.min && wert.length < regel.min) {
            ergebnisse.push({
              feld: regel.feld,
              gueltig: false,
              meldung: regel.fehlermeldung,
              schwere: 'fehler'
            });
          }
          if (regel.max && wert.length > regel.max) {
            ergebnisse.push({
              feld: regel.feld,
              gueltig: false,
              meldung: regel.fehlermeldung,
              schwere: 'fehler'
            });
          }
          if (regel.pattern && !regel.pattern.test(wert)) {
            ergebnisse.push({
              feld: regel.feld,
              gueltig: false,
              meldung: regel.fehlermeldung,
              schwere: 'fehler'
            });
          }
        }
        break;

      case 'enum':
        if (!regel.enum_werte?.includes(wert)) {
          ergebnisse.push({
            feld: regel.feld,
            gueltig: false,
            meldung: regel.fehlermeldung,
            schwere: 'fehler'
          });
        }
        break;
    }
  }

  return ergebnisse;
}

interface ValidationResult {
  feld: string;
  gueltig: boolean;
  meldung: string;
  schwere: 'fehler' | 'warnung' | 'hinweis';
}
```

---

## Ebene 2: Logische Konsistenz

### Geometrie-Konsistenz

```typescript
interface GeometrieKonsistenz {
  id: string;
  name: string;
  pruefung: (gebaeude: Gebaeude) => ValidationResult | null;
}

const GEOMETRIE_REGELN: GeometrieKonsistenz[] = [
  {
    id: 'GEO_001',
    name: 'Hüllfläche vs. Volumen',
    pruefung: (g) => {
      // Verhältnis A/V für Gebäude typisch 0.3-1.2
      const AV = g.A_huelle / g.V_beheizt;
      if (AV < 0.2 || AV > 1.5) {
        return {
          feld: 'A_huelle',
          gueltig: false,
          meldung: `A/V-Verhältnis ${AV.toFixed(2)} unplausibel (typisch: 0.3-1.2)`,
          schwere: 'warnung'
        };
      }
      return null;
    }
  },
  {
    id: 'GEO_002',
    name: 'NGF vs. BGF',
    pruefung: (g) => {
      const verhaeltnis = g.A_NGF / g.A_BGF;
      if (verhaeltnis < 0.65 || verhaeltnis > 0.95) {
        return {
          feld: 'A_NGF',
          gueltig: false,
          meldung: `NGF/BGF-Verhältnis ${verhaeltnis.toFixed(2)} unüblich (normal: 0.70-0.90)`,
          schwere: 'warnung'
        };
      }
      return null;
    }
  },
  {
    id: 'GEO_003',
    name: 'Fensterfläche vs. Fassade',
    pruefung: (g) => {
      const A_fenster = g.bauteile
        .filter(b => b.typ === 'fenster')
        .reduce((sum, b) => sum + b.flaeche, 0);
      const A_fassade = g.bauteile
        .filter(b => b.typ === 'aussenwand')
        .reduce((sum, b) => sum + b.flaeche, 0);

      const fensteranteil = A_fenster / (A_fenster + A_fassade);

      if (fensteranteil < 0.05) {
        return {
          feld: 'fenster',
          gueltig: false,
          meldung: `Fensteranteil nur ${(fensteranteil*100).toFixed(0)}% - sehr niedrig`,
          schwere: 'warnung'
        };
      }
      if (fensteranteil > 0.60) {
        return {
          feld: 'fenster',
          gueltig: false,
          meldung: `Fensteranteil ${(fensteranteil*100).toFixed(0)}% - ungewöhnlich hoch`,
          schwere: 'warnung'
        };
      }
      return null;
    }
  },
  {
    id: 'GEO_004',
    name: 'Raumhöhe plausibel',
    pruefung: (g) => {
      const mittlere_hoehe = g.V_beheizt / g.A_NGF;
      if (mittlere_hoehe < 2.0 || mittlere_hoehe > 5.0) {
        return {
          feld: 'V_beheizt',
          gueltig: false,
          meldung: `Mittlere Raumhöhe ${mittlere_hoehe.toFixed(1)}m unüblich`,
          schwere: 'warnung'
        };
      }
      return null;
    }
  },
  {
    id: 'GEO_005',
    name: 'Flächensumme Hülle',
    pruefung: (g) => {
      const A_berechnet = g.bauteile
        .filter(b => ['aussenwand', 'dach', 'boden', 'fenster', 'tuer'].includes(b.typ))
        .reduce((sum, b) => sum + b.flaeche, 0);

      const abweichung = Math.abs(A_berechnet - g.A_huelle) / g.A_huelle;

      if (abweichung > 0.10) {
        return {
          feld: 'A_huelle',
          gueltig: false,
          meldung: `Summe Bauteilflächen (${A_berechnet.toFixed(0)}m²) weicht ${(abweichung*100).toFixed(0)}% von A_hülle (${g.A_huelle.toFixed(0)}m²) ab`,
          schwere: 'fehler'
        };
      }
      return null;
    }
  },
  {
    id: 'GEO_006',
    name: 'Geschosshöhe konsistent',
    pruefung: (g) => {
      if (g.geschosse && g.geschosshoehe) {
        const V_erwartet = g.A_BGF * g.geschosshoehe;
        const abweichung = Math.abs(V_erwartet - g.V_beheizt) / g.V_beheizt;

        if (abweichung > 0.15) {
          return {
            feld: 'geschosshoehe',
            gueltig: false,
            meldung: `Volumen aus Geschosszahl und -höhe (${V_erwartet.toFixed(0)}m³) weicht von V_beheizt ab`,
            schwere: 'warnung'
          };
        }
      }
      return null;
    }
  }
];
```

### Anlagentechnik-Konsistenz

```typescript
const ANLAGEN_REGELN: GeometrieKonsistenz[] = [
  {
    id: 'ANL_001',
    name: 'Heizleistung vs. Heizlast',
    pruefung: (g) => {
      if (g.heizlast && g.waermeerzeuger) {
        const P_erzeuger = g.waermeerzeuger.reduce((sum, w) => sum + w.P_n, 0);
        const ueberdimensionierung = P_erzeuger / g.heizlast;

        if (ueberdimensionierung > 2.0) {
          return {
            feld: 'waermeerzeuger',
            gueltig: false,
            meldung: `Heizleistung ${P_erzeuger}kW stark überdimensioniert (Heizlast: ${g.heizlast.toFixed(0)}kW)`,
            schwere: 'warnung'
          };
        }
        if (ueberdimensionierung < 0.9) {
          return {
            feld: 'waermeerzeuger',
            gueltig: false,
            meldung: `Heizleistung ${P_erzeuger}kW könnte zu gering sein (Heizlast: ${g.heizlast.toFixed(0)}kW)`,
            schwere: 'warnung'
          };
        }
      }
      return null;
    }
  },
  {
    id: 'ANL_002',
    name: 'Vorlauftemperatur vs. Wärmeerzeuger',
    pruefung: (g) => {
      const hatWaermepumpe = g.waermeerzeuger?.some(
        w => w.typ.includes('waermepumpe')
      );
      const vorlauftemp = g.verteilung?.theta_vorlauf ?? 55;

      if (hatWaermepumpe && vorlauftemp > 55) {
        return {
          feld: 'verteilung.theta_vorlauf',
          gueltig: false,
          meldung: `Vorlauftemperatur ${vorlauftemp}°C zu hoch für effizienten Wärmepumpenbetrieb`,
          schwere: 'warnung'
        };
      }
      return null;
    }
  },
  {
    id: 'ANL_003',
    name: 'Speichervolumen vs. Kollektorfläche',
    pruefung: (g) => {
      if (g.solarthermie) {
        const A_koll = g.solarthermie.kollektorflaeche;
        const V_sp = g.solarthermie.speichervolumen;
        const verhaeltnis = V_sp / A_koll;

        // Empfohlen: 40-80 l/m² Kollektorfläche
        if (verhaeltnis < 30) {
          return {
            feld: 'solarthermie.speichervolumen',
            gueltig: false,
            meldung: `Speichervolumen zu klein (${verhaeltnis.toFixed(0)} l/m², empfohlen: 50-80 l/m²)`,
            schwere: 'warnung'
          };
        }
        if (verhaeltnis > 100) {
          return {
            feld: 'solarthermie.speichervolumen',
            gueltig: false,
            meldung: `Speichervolumen sehr groß (${verhaeltnis.toFixed(0)} l/m², empfohlen: 50-80 l/m²)`,
            schwere: 'hinweis'
          };
        }
      }
      return null;
    }
  },
  {
    id: 'ANL_004',
    name: 'Lüftungsanlage Volumenstrom',
    pruefung: (g) => {
      if (g.lueftung && g.lueftung.typ !== 'natuerlich') {
        const V_lueftung = g.lueftung.volumenstrom;
        const V_gebaeude = g.V_beheizt;
        const luftwechsel = V_lueftung / V_gebaeude;

        if (luftwechsel < 0.3) {
          return {
            feld: 'lueftung.volumenstrom',
            gueltig: false,
            meldung: `Luftwechsel ${luftwechsel.toFixed(2)}/h zu gering (min. 0.4/h empfohlen)`,
            schwere: 'warnung'
          };
        }
        if (luftwechsel > 2.0) {
          return {
            feld: 'lueftung.volumenstrom',
            gueltig: false,
            meldung: `Luftwechsel ${luftwechsel.toFixed(1)}/h ungewöhnlich hoch`,
            schwere: 'warnung'
          };
        }
      }
      return null;
    }
  },
  {
    id: 'ANL_005',
    name: 'TWW-System definiert',
    pruefung: (g) => {
      const hatTWW = g.tww_bereiter?.length > 0 ||
        g.waermeerzeuger?.some(w => w.auch_tww);

      if (!hatTWW) {
        return {
          feld: 'tww_bereiter',
          gueltig: false,
          meldung: 'Keine Trinkwarmwasserbereitung definiert',
          schwere: 'fehler'
        };
      }
      return null;
    }
  }
];
```

---

## Ebene 3: Fachliche Plausibilität

### U-Wert Plausibilitätsprüfung

```typescript
interface UWertGrenzen {
  typ: string;
  baujahr_von: number;
  baujahr_bis: number;
  U_min: number;
  U_max: number;
  U_typisch: number;
}

const U_WERT_REFERENZEN: UWertGrenzen[] = [
  // Außenwand
  { typ: 'aussenwand', baujahr_von: 1800, baujahr_bis: 1918, U_min: 1.0, U_max: 2.5, U_typisch: 1.7 },
  { typ: 'aussenwand', baujahr_von: 1919, baujahr_bis: 1948, U_min: 0.9, U_max: 2.2, U_typisch: 1.5 },
  { typ: 'aussenwand', baujahr_von: 1949, baujahr_bis: 1968, U_min: 0.8, U_max: 1.8, U_typisch: 1.4 },
  { typ: 'aussenwand', baujahr_von: 1969, baujahr_bis: 1978, U_min: 0.6, U_max: 1.5, U_typisch: 1.0 },
  { typ: 'aussenwand', baujahr_von: 1979, baujahr_bis: 1994, U_min: 0.4, U_max: 1.0, U_typisch: 0.6 },
  { typ: 'aussenwand', baujahr_von: 1995, baujahr_bis: 2001, U_min: 0.25, U_max: 0.6, U_typisch: 0.4 },
  { typ: 'aussenwand', baujahr_von: 2002, baujahr_bis: 2015, U_min: 0.18, U_max: 0.40, U_typisch: 0.28 },
  { typ: 'aussenwand', baujahr_von: 2016, baujahr_bis: 2024, U_min: 0.10, U_max: 0.28, U_typisch: 0.20 },

  // Dach/oberste Geschossdecke
  { typ: 'dach', baujahr_von: 1800, baujahr_bis: 1948, U_min: 0.8, U_max: 2.5, U_typisch: 1.4 },
  { typ: 'dach', baujahr_von: 1949, baujahr_bis: 1978, U_min: 0.5, U_max: 1.5, U_typisch: 0.9 },
  { typ: 'dach', baujahr_von: 1979, baujahr_bis: 1994, U_min: 0.3, U_max: 0.8, U_typisch: 0.5 },
  { typ: 'dach', baujahr_von: 1995, baujahr_bis: 2015, U_min: 0.15, U_max: 0.35, U_typisch: 0.25 },
  { typ: 'dach', baujahr_von: 2016, baujahr_bis: 2024, U_min: 0.08, U_max: 0.20, U_typisch: 0.14 },

  // Fenster
  { typ: 'fenster', baujahr_von: 1800, baujahr_bis: 1978, U_min: 2.5, U_max: 5.5, U_typisch: 4.0 },
  { typ: 'fenster', baujahr_von: 1979, baujahr_bis: 1994, U_min: 2.0, U_max: 3.5, U_typisch: 2.7 },
  { typ: 'fenster', baujahr_von: 1995, baujahr_bis: 2008, U_min: 1.3, U_max: 2.5, U_typisch: 1.8 },
  { typ: 'fenster', baujahr_von: 2009, baujahr_bis: 2024, U_min: 0.7, U_max: 1.5, U_typisch: 1.1 },

  // Kellerdecke/Boden
  { typ: 'boden', baujahr_von: 1800, baujahr_bis: 1978, U_min: 0.6, U_max: 2.0, U_typisch: 1.2 },
  { typ: 'boden', baujahr_von: 1979, baujahr_bis: 2001, U_min: 0.4, U_max: 1.0, U_typisch: 0.7 },
  { typ: 'boden', baujahr_von: 2002, baujahr_bis: 2024, U_min: 0.20, U_max: 0.50, U_typisch: 0.35 },
];

function pruefeUWertPlausibilitaet(
  bauteil: Bauteil,
  baujahr: number
): ValidationResult | null {
  const referenz = U_WERT_REFERENZEN.find(
    r => r.typ === bauteil.typ &&
      baujahr >= r.baujahr_von &&
      baujahr <= r.baujahr_bis
  );

  if (!referenz) return null;

  if (bauteil.U_wert < referenz.U_min * 0.8) {
    return {
      feld: `bauteile.${bauteil.id}.U_wert`,
      gueltig: false,
      meldung: `U-Wert ${bauteil.U_wert} sehr niedrig für ${bauteil.typ} Baujahr ${baujahr} (typisch: ${referenz.U_typisch})`,
      schwere: 'warnung'
    };
  }

  if (bauteil.U_wert > referenz.U_max * 1.2) {
    return {
      feld: `bauteile.${bauteil.id}.U_wert`,
      gueltig: false,
      meldung: `U-Wert ${bauteil.U_wert} sehr hoch für ${bauteil.typ} Baujahr ${baujahr} (typisch: ${referenz.U_typisch})`,
      schwere: 'warnung'
    };
  }

  return null;
}
```

### Energiekennwert-Plausibilität

```typescript
interface EnergiekennwertReferenz {
  gebaeude_typ: string;
  baujahr_von: number;
  baujahr_bis: number;
  q_H_min: number;      // kWh/(m²·a) Heizwärme
  q_H_max: number;
  q_H_typisch: number;
  q_E_min: number;      // kWh/(m²·a) Endenergie
  q_E_max: number;
  q_E_typisch: number;
}

const ENERGIEKENNWERT_REFERENZEN: EnergiekennwertReferenz[] = [
  // Einfamilienhaus
  { gebaeude_typ: 'EFH', baujahr_von: 1800, baujahr_bis: 1948, q_H_min: 150, q_H_max: 350, q_H_typisch: 220, q_E_min: 180, q_E_max: 450, q_E_typisch: 280 },
  { gebaeude_typ: 'EFH', baujahr_von: 1949, baujahr_bis: 1978, q_H_min: 100, q_H_max: 280, q_H_typisch: 180, q_E_min: 120, q_E_max: 350, q_E_typisch: 220 },
  { gebaeude_typ: 'EFH', baujahr_von: 1979, baujahr_bis: 1994, q_H_min: 80, q_H_max: 180, q_H_typisch: 120, q_E_min: 100, q_E_max: 220, q_E_typisch: 150 },
  { gebaeude_typ: 'EFH', baujahr_von: 1995, baujahr_bis: 2001, q_H_min: 50, q_H_max: 120, q_H_typisch: 80, q_E_min: 70, q_E_max: 150, q_E_typisch: 100 },
  { gebaeude_typ: 'EFH', baujahr_von: 2002, baujahr_bis: 2015, q_H_min: 30, q_H_max: 80, q_H_typisch: 50, q_E_min: 50, q_E_max: 100, q_E_typisch: 70 },
  { gebaeude_typ: 'EFH', baujahr_von: 2016, baujahr_bis: 2024, q_H_min: 15, q_H_max: 50, q_H_typisch: 30, q_E_min: 30, q_E_max: 60, q_E_typisch: 45 },

  // Mehrfamilienhaus
  { gebaeude_typ: 'MFH', baujahr_von: 1800, baujahr_bis: 1948, q_H_min: 100, q_H_max: 250, q_H_typisch: 160, q_E_min: 130, q_E_max: 320, q_E_typisch: 200 },
  { gebaeude_typ: 'MFH', baujahr_von: 1949, baujahr_bis: 1978, q_H_min: 80, q_H_max: 200, q_H_typisch: 130, q_E_min: 100, q_E_max: 250, q_E_typisch: 160 },
  { gebaeude_typ: 'MFH', baujahr_von: 1979, baujahr_bis: 1994, q_H_min: 60, q_H_max: 140, q_H_typisch: 90, q_E_min: 80, q_E_max: 180, q_E_typisch: 120 },
  { gebaeude_typ: 'MFH', baujahr_von: 1995, baujahr_bis: 2024, q_H_min: 25, q_H_max: 80, q_H_typisch: 45, q_E_min: 40, q_E_max: 100, q_E_typisch: 60 },
];

function pruefeEnergiekennwerte(
  ergebnis: Energiebilanz,
  gebaeude: Gebaeude
): ValidationResult[] {
  const results: ValidationResult[] = [];

  const referenz = ENERGIEKENNWERT_REFERENZEN.find(
    r => r.gebaeude_typ === gebaeude.gebaeude_typ &&
      gebaeude.baujahr >= r.baujahr_von &&
      gebaeude.baujahr <= r.baujahr_bis
  );

  if (!referenz) return results;

  // Heizwärmebedarf prüfen
  const q_H = ergebnis.Q_h_b / gebaeude.A_N;

  if (q_H < referenz.q_H_min * 0.7) {
    results.push({
      feld: 'Q_h_b',
      gueltig: false,
      meldung: `Heizwärmebedarf ${q_H.toFixed(0)} kWh/(m²·a) sehr niedrig für ${gebaeude.gebaeude_typ} Bj. ${gebaeude.baujahr} (typisch: ${referenz.q_H_typisch})`,
      schwere: 'warnung'
    });
  }

  if (q_H > referenz.q_H_max * 1.3) {
    results.push({
      feld: 'Q_h_b',
      gueltig: false,
      meldung: `Heizwärmebedarf ${q_H.toFixed(0)} kWh/(m²·a) sehr hoch für ${gebaeude.gebaeude_typ} Bj. ${gebaeude.baujahr} (typisch: ${referenz.q_H_typisch})`,
      schwere: 'warnung'
    });
  }

  // Endenergie prüfen
  const q_E = ergebnis.Q_f / gebaeude.A_N;

  if (q_E < referenz.q_E_min * 0.5) {
    results.push({
      feld: 'Q_f',
      gueltig: false,
      meldung: `Endenergiebedarf ${q_E.toFixed(0)} kWh/(m²·a) extrem niedrig - bitte Eingaben prüfen`,
      schwere: 'fehler'
    });
  }

  if (q_E > referenz.q_E_max * 1.5) {
    results.push({
      feld: 'Q_f',
      gueltig: false,
      meldung: `Endenergiebedarf ${q_E.toFixed(0)} kWh/(m²·a) sehr hoch - mögliche Eingabefehler`,
      schwere: 'warnung'
    });
  }

  return results;
}
```

---

## Ebene 4: Normkonformität

### GEG-Anforderungsprüfung

```typescript
interface GEGAnforderung {
  id: string;
  paragraph: string;
  beschreibung: string;
  pruefung: (gebaeude: Gebaeude, ergebnis: Energiebilanz) => ValidationResult | null;
}

const GEG_ANFORDERUNGEN: GEGAnforderung[] = [
  {
    id: 'GEG_10',
    paragraph: '§ 10 Abs. 2',
    beschreibung: 'Primärenergiebedarf ≤ Referenzgebäude',
    pruefung: (g, e) => {
      if (e.Q_p > e.Q_p_ref) {
        return {
          feld: 'Q_p',
          gueltig: false,
          meldung: `GEG § 10: Primärenergiebedarf ${e.q_p_spez.toFixed(0)} kWh/(m²·a) überschreitet Referenzwert ${(e.Q_p_ref/g.A_N).toFixed(0)} kWh/(m²·a)`,
          schwere: 'fehler'
        };
      }
      return null;
    }
  },
  {
    id: 'GEG_11',
    paragraph: '§ 11',
    beschreibung: 'Wärmedurchgangskoeffizient-Höchstwerte',
    pruefung: (g, e) => {
      const ergebnisse: ValidationResult[] = [];

      // GEG Anlage 1 Tabelle 1: Höchstwerte
      const hoechstwerte: Record<string, number> = {
        'aussenwand': 0.28,
        'dach': 0.20,
        'fenster': 1.30,
        'tuer': 1.80,
        'boden_erdberuehrt': 0.35,
      };

      for (const bauteil of g.bauteile) {
        const grenze = hoechstwerte[bauteil.typ];
        if (grenze && bauteil.U_wert > grenze && !bauteil.bestand) {
          return {
            feld: `bauteile.${bauteil.id}.U_wert`,
            gueltig: false,
            meldung: `GEG § 11: U-Wert ${bauteil.name} (${bauteil.U_wert} W/(m²·K)) überschreitet Höchstwert ${grenze} W/(m²·K)`,
            schwere: 'fehler'
          };
        }
      }
      return null;
    }
  },
  {
    id: 'GEG_16',
    paragraph: '§ 16',
    beschreibung: 'Sommerlicher Wärmeschutz',
    pruefung: (g, e) => {
      if (e.sommer_nachweis && !e.sommer_nachweis.erfuellt) {
        return {
          feld: 'sommer_nachweis',
          gueltig: false,
          meldung: `GEG § 16: Sommerlicher Wärmeschutz nicht erfüllt (S_vorh=${e.sommer_nachweis.S_vorh.toFixed(3)} > S_zul=${e.sommer_nachweis.S_zul.toFixed(3)})`,
          schwere: 'fehler'
        };
      }
      return null;
    }
  },
  {
    id: 'GEG_71',
    paragraph: '§ 71',
    beschreibung: '65% Erneuerbare Energien bei Heizungstausch',
    pruefung: (g, e) => {
      // Nur relevant bei Heizungstausch ab 2024
      if (g.heizung_ausgetauscht && g.heizung_austauschjahr >= 2024) {
        const ee_anteil = berechneEEAnteil(g);
        if (ee_anteil < 0.65) {
          return {
            feld: 'waermeerzeuger',
            gueltig: false,
            meldung: `GEG § 71: EE-Anteil ${(ee_anteil*100).toFixed(0)}% unter 65%-Pflicht`,
            schwere: 'fehler'
          };
        }
      }
      return null;
    }
  },
  {
    id: 'GEG_72',
    paragraph: '§ 72',
    beschreibung: 'Verbot von Ölheizungen ab 2026',
    pruefung: (g, e) => {
      const hatOelheizung = g.waermeerzeuger?.some(
        w => w.energietraeger === 'heizoel' && !w.alt_bestand
      );

      if (hatOelheizung && g.heizung_austauschjahr >= 2026) {
        return {
          feld: 'waermeerzeuger',
          gueltig: false,
          meldung: 'GEG § 72: Neuinstallation von Ölheizungen ab 2026 nicht zulässig',
          schwere: 'fehler'
        };
      }
      return null;
    }
  }
];

function pruefeGEGKonformitaet(
  gebaeude: Gebaeude,
  ergebnis: Energiebilanz
): ValidationResult[] {
  const results: ValidationResult[] = [];

  for (const anforderung of GEG_ANFORDERUNGEN) {
    const result = anforderung.pruefung(gebaeude, ergebnis);
    if (result) {
      results.push(result);
    }
  }

  return results;
}
```

### Vollständigkeitsprüfung für Energieausweis

```typescript
interface VollstaendigkeitsFeld {
  feld: string;
  pflicht: boolean;
  abschnitt: string;
  beschreibung: string;
}

const ENERGIEAUSWEIS_PFLICHTFELDER: VollstaendigkeitsFeld[] = [
  // Gebäudedaten
  { feld: 'adresse', pflicht: true, abschnitt: 'Gebäude', beschreibung: 'Vollständige Adresse' },
  { feld: 'baujahr', pflicht: true, abschnitt: 'Gebäude', beschreibung: 'Baujahr' },
  { feld: 'gebaeude_typ', pflicht: true, abschnitt: 'Gebäude', beschreibung: 'Gebäudetyp' },
  { feld: 'A_N', pflicht: true, abschnitt: 'Gebäude', beschreibung: 'Nutzfläche/Wohnfläche' },
  { feld: 'wohneinheiten', pflicht: true, abschnitt: 'Gebäude', beschreibung: 'Anzahl Wohneinheiten' },

  // Bauteile
  { feld: 'bauteile.aussenwand', pflicht: true, abschnitt: 'Gebäudehülle', beschreibung: 'Außenwand-Definition' },
  { feld: 'bauteile.dach', pflicht: true, abschnitt: 'Gebäudehülle', beschreibung: 'Dach/oberste Decke-Definition' },
  { feld: 'bauteile.boden', pflicht: true, abschnitt: 'Gebäudehülle', beschreibung: 'Boden/Kellerdecke-Definition' },
  { feld: 'bauteile.fenster', pflicht: true, abschnitt: 'Gebäudehülle', beschreibung: 'Fenster-Definition' },

  // Anlagentechnik
  { feld: 'waermeerzeuger', pflicht: true, abschnitt: 'Anlagentechnik', beschreibung: 'Wärmeerzeuger für Heizung' },
  { feld: 'tww_bereitung', pflicht: true, abschnitt: 'Anlagentechnik', beschreibung: 'Trinkwarmwasserbereitung' },
  { feld: 'verteilung', pflicht: true, abschnitt: 'Anlagentechnik', beschreibung: 'Wärmeverteilung' },

  // Ergebnisse
  { feld: 'Q_p', pflicht: true, abschnitt: 'Ergebnisse', beschreibung: 'Primärenergiebedarf' },
  { feld: 'Q_f', pflicht: true, abschnitt: 'Ergebnisse', beschreibung: 'Endenergiebedarf' },
  { feld: 'CO2', pflicht: true, abschnitt: 'Ergebnisse', beschreibung: 'CO2-Emissionen' },
  { feld: 'effizienzklasse', pflicht: true, abschnitt: 'Ergebnisse', beschreibung: 'Energieeffizienzklasse' },

  // Modernisierungsempfehlungen
  { feld: 'empfehlungen', pflicht: true, abschnitt: 'Empfehlungen', beschreibung: 'Mindestens eine Modernisierungsempfehlung' },
];

function pruefeVollstaendigkeit(
  energieausweis: Energieausweis
): { vollstaendig: boolean; fehlende: VollstaendigkeitsFeld[] } {
  const fehlende: VollstaendigkeitsFeld[] = [];

  for (const feld of ENERGIEAUSWEIS_PFLICHTFELDER) {
    if (!feld.pflicht) continue;

    const wert = getNestedValue(energieausweis, feld.feld);

    if (wert === undefined || wert === null || wert === '') {
      fehlende.push(feld);
    }

    // Arrays müssen mindestens ein Element haben
    if (Array.isArray(wert) && wert.length === 0) {
      fehlende.push(feld);
    }
  }

  return {
    vollstaendig: fehlende.length === 0,
    fehlende
  };
}

function getNestedValue(obj: any, path: string): any {
  return path.split('.').reduce((current, key) => {
    return current?.[key];
  }, obj);
}
```

---

## Workflow-übergreifende Konsistenz

### Konsistenz zwischen Versionen

```typescript
interface VersionsKonsistenz {
  id: string;
  beschreibung: string;
  pruefung: (
    istZustand: GebaeudeVersion,
    sanierung: GebaeudeVersion
  ) => ValidationResult | null;
}

const VERSIONS_KONSISTENZ: VersionsKonsistenz[] = [
  {
    id: 'VER_001',
    beschreibung: 'Geometrie darf sich nicht ändern',
    pruefung: (ist, san) => {
      if (Math.abs(ist.A_N - san.A_N) > 1) {
        return {
          feld: 'A_N',
          gueltig: false,
          meldung: `Nutzfläche weicht ab: Ist ${ist.A_N}m² vs. Sanierung ${san.A_N}m²`,
          schwere: 'fehler'
        };
      }
      return null;
    }
  },
  {
    id: 'VER_002',
    beschreibung: 'U-Werte sollten sich verbessern',
    pruefung: (ist, san) => {
      for (const bauteil of san.bauteile) {
        const istBauteil = ist.bauteile.find(b => b.id === bauteil.id);
        if (istBauteil && bauteil.U_wert > istBauteil.U_wert) {
          return {
            feld: `bauteile.${bauteil.id}.U_wert`,
            gueltig: false,
            meldung: `U-Wert ${bauteil.name} verschlechtert sich: ${istBauteil.U_wert} → ${bauteil.U_wert}`,
            schwere: 'warnung'
          };
        }
      }
      return null;
    }
  },
  {
    id: 'VER_003',
    beschreibung: 'Energiebedarf sollte sinken',
    pruefung: (ist, san) => {
      if (san.Q_p >= ist.Q_p * 0.95) {
        return {
          feld: 'Q_p',
          gueltig: false,
          meldung: `Primärenergiebedarf sinkt nicht signifikant: ${ist.Q_p.toFixed(0)} → ${san.Q_p.toFixed(0)} kWh/a`,
          schwere: 'warnung'
        };
      }
      return null;
    }
  }
];
```

### Konsistenz zwischen Produkten

```typescript
interface ProduktKonsistenz {
  produkt_a: string;
  produkt_b: string;
  pruefung: (
    a: Projekt,
    b: Projekt
  ) => ValidationResult | null;
}

const PRODUKT_KONSISTENZ: ProduktKonsistenz[] = [
  {
    produkt_a: 'heizlast',
    produkt_b: 'energieausweis',
    pruefung: (heizlast, ea) => {
      // Heizlast sollte zur Anlagenauslegung passen
      const P_heizlast = heizlast.ergebnis.Phi_HL_gesamt;
      const P_anlage = ea.gebaeude.waermeerzeuger
        .reduce((sum, w) => sum + w.P_n * 1000, 0);

      const verhaeltnis = P_anlage / P_heizlast;

      if (verhaeltnis < 0.8 || verhaeltnis > 2.0) {
        return {
          feld: 'waermeerzeuger',
          gueltig: false,
          meldung: `Inkonsistenz: Heizlast ${P_heizlast.toFixed(0)}W, Anlagenleistung ${P_anlage.toFixed(0)}W`,
          schwere: 'warnung'
        };
      }
      return null;
    }
  },
  {
    produkt_a: 'isfp',
    produkt_b: 'energieausweis',
    pruefung: (isfp, ea) => {
      // iSFP-Zielwerte sollten zum Energieausweis passen
      const q_ist = ea.ergebnis.q_p_spez;
      const q_ziel = isfp.ergebnis.q_p_ziel;

      if (q_ziel >= q_ist) {
        return {
          feld: 'isfp.q_p_ziel',
          gueltig: false,
          meldung: `iSFP-Zielwert ${q_ziel} kWh/(m²·a) ist nicht besser als Ist-Zustand ${q_ist.toFixed(0)}`,
          schwere: 'fehler'
        };
      }
      return null;
    }
  }
];
```

---

## UI-Integration der Validierung

### Validierungs-Komponente

```typescript
// React-Komponente für Validierungsanzeige

interface ValidationPanelProps {
  validierungen: ValidationResult[];
  onFeldKlick?: (feld: string) => void;
}

const ValidationPanel: React.FC<ValidationPanelProps> = ({
  validierungen,
  onFeldKlick
}) => {
  const fehler = validierungen.filter(v => v.schwere === 'fehler');
  const warnungen = validierungen.filter(v => v.schwere === 'warnung');
  const hinweise = validierungen.filter(v => v.schwere === 'hinweis');

  return (
    <div className="validation-panel">
      {fehler.length > 0 && (
        <ValidationSection
          titel="Fehler"
          icon="🔴"
          items={fehler}
          onKlick={onFeldKlick}
        />
      )}

      {warnungen.length > 0 && (
        <ValidationSection
          titel="Warnungen"
          icon="🟡"
          items={warnungen}
          onKlick={onFeldKlick}
        />
      )}

      {hinweise.length > 0 && (
        <ValidationSection
          titel="Hinweise"
          icon="🔵"
          items={hinweise}
          onKlick={onFeldKlick}
        />
      )}

      {validierungen.length === 0 && (
        <div className="validation-success">
          <span className="icon">✓</span>
          <span>Alle Prüfungen bestanden</span>
        </div>
      )}
    </div>
  );
};
```

### Echtzeit-Validierung

```typescript
// Hook für Echtzeit-Validierung

function useValidation(daten: any, regeln: ValidationRule[]) {
  const [ergebnisse, setErgebnisse] = useState<ValidationResult[]>([]);
  const [validiert, setValidiert] = useState(false);

  // Debounced Validierung bei Änderungen
  useEffect(() => {
    const timer = setTimeout(() => {
      const results: ValidationResult[] = [];

      for (const regel of regeln) {
        try {
          const result = regel.pruefung(daten);
          if (result && !result.gueltig) {
            results.push(result);
          }
        } catch (e) {
          console.error(`Validierungsregel ${regel.id} fehlgeschlagen:`, e);
        }
      }

      setErgebnisse(results);
      setValidiert(true);
    }, 300);  // 300ms Debounce

    return () => clearTimeout(timer);
  }, [daten, regeln]);

  return {
    ergebnisse,
    validiert,
    hatFehler: ergebnisse.some(e => e.schwere === 'fehler'),
    hatWarnungen: ergebnisse.some(e => e.schwere === 'warnung'),
    istGueltig: !ergebnisse.some(e => e.schwere === 'fehler')
  };
}
```

---

## API für Validierung

```typescript
// POST /api/v1/validation/gebaeude/{id}
interface ValidationRequest {
  ebenen?: ('eingabe' | 'konsistenz' | 'plausibilitaet' | 'norm')[];
  streng?: boolean;  // Auch Warnungen als Fehler behandeln
}

interface ValidationResponse {
  gueltig: boolean;
  ergebnisse: {
    ebene: string;
    ergebnisse: ValidationResult[];
  }[];
  zusammenfassung: {
    fehler: number;
    warnungen: number;
    hinweise: number;
  };
}

// POST /api/v1/validation/energieausweis/{id}
// Prüft Vollständigkeit und Normkonformität für Export

// POST /api/v1/validation/isfp/{id}
// Prüft iSFP-spezifische Anforderungen
```
