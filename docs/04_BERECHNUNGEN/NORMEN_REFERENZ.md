# Normen-Referenz für Energieberechnungen

**Version:** 1.1.0
**Stand:** 2025-11-26 (Recherchiert)
**Zielländer:** Österreich (AT) 🇦🇹, Deutschland (DE) 🇩🇪

---

## Inhaltsverzeichnis

1. [Übersicht](#1-übersicht)
2. [Energieausweis / Gebäudeenergie](#2-energieausweis--gebäudeenergie)
3. [Heizlast & Kühllast](#3-heizlast--kühllast)
4. [Lüftung & Klimatechnik](#4-lüftung--klimatechnik)
5. [Wärmeerzeugung](#5-wärmeerzeugung)
6. [Stromerzeugung & Photovoltaik](#6-stromerzeugung--photovoltaik)
7. [Energieeffizienz Industrie](#7-energieeffizienz-industrie)
8. [Druckluft](#8-druckluft)
9. [Vakuumtechnik](#9-vakuumtechnik)
10. [Abwärmenutzung & Wärmerückgewinnung](#10-abwärmenutzung--wärmerückgewinnung)
11. [Wärmetauscher](#11-wärmetauscher)
12. [Power-to-X (PtX)](#12-power-to-x-ptx)
13. [Energieaudits](#13-energieaudits)
14. [Sanierung & Renovierung](#14-sanierung--renovierung)
15. [Bauphysik & Wärmeschutz](#15-bauphysik--wärmeschutz)
16. [Beleuchtung](#16-beleuchtung)
17. [Elektrische Anlagen](#17-elektrische-anlagen)
18. [Förderungen & Nachweise](#18-förderungen--nachweise)

---

## 1. Übersicht

### 1.1 Normen-Hierarchie

```
┌─────────────────────────────────────────────────────────────────┐
│  ÖSTERREICH (AT) - Priorisiert                                  │
├─────────────────────────────────────────────────────────────────┤
│  OIB-Richtlinien      → Bauordnung, verbindlich                │
│  ÖNORM                → Nationale Normen                        │
│  ÖNORM EN             → Europäische Normen (AT-Umsetzung)       │
│  ÖNORM EN ISO         → Internationale Normen (AT-Umsetzung)    │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│  DEUTSCHLAND (DE)                                               │
├─────────────────────────────────────────────────────────────────┤
│  GEG                  → Gebäudeenergiegesetz, verbindlich       │
│  DIN                  → Nationale Normen                        │
│  DIN EN               → Europäische Normen (DE-Umsetzung)       │
│  DIN EN ISO           → Internationale Normen (DE-Umsetzung)    │
│  VDI-Richtlinien      → Technische Regeln (Verein Dt. Ing.)    │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Aktuelle Normenversionen (Stand November 2025)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  WICHTIGE AKTUELLE VERSIONEN (recherchiert November 2025)                        │
├─────────────────────────────────────────────────────────────────────────────────┤
│  ÖSTERREICH:                                                                     │
│  • OIB-Richtlinie 6:       2023 (gültig) → 2025 in Vorbereitung!               │
│    └─ OIB-RL 6 2025: 326 Stellungnahmen bearbeitet, Umsetzung bis Mai 2026     │
│  • ÖNORM H 5056-1:         2024-03-01 (aktuell)                                 │
│  • ÖNORM H 5055:           2011-11-01                                           │
│  • ÖNORM H 5050:           2014                                                 │
│                                                                                  │
│  DEUTSCHLAND:                                                                    │
│  • GEG:                    2024 (gültig) - EPBD-Umsetzung bis Mai 2026 nötig    │
│  • DIN V 18599:            2018-09 (unverändert, aber Pflicht für alle Gebäude) │
│  • DIN EN 16247-1:         2022 (aktualisiert, ISO 50001 angepasst)             │
│  • DIN EN ISO 50001:       2018                                                 │
│  • VDI 4650 Blatt 1:       + Ergänzung VDI-EE 4650 Blatt 1.1 (Oktober 2023)    │
│  • VDI 2078:               2015 (mit TRY 2017 Klimadaten)                       │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 1.3 Wichtige Hinweise

- **Öffentlich verfügbare Informationen:** Diese Referenz listet Normen-Bezeichnungen und deren Anwendungsbereiche
- **Normen-Bezug:** Volltext über Austrian Standards (AT) bzw. Beuth Verlag (DE)
- **Kostenlose Quellen:** OIB-Richtlinien, GEG-Volltext, EU-Verordnungen sind frei verfügbar
- **EPBD 2024:** EU-Gebäuderichtlinie erfordert Umsetzung in DE/AT bis Mai 2026

---

## 2. Energieausweis / Gebäudeenergie

### 2.1 Österreich 🇦🇹

| Norm/Richtlinie | Titel | Anwendung |
|-----------------|-------|-----------|
| **OIB-Richtlinie 6 (2023)** | Energieeinsparung und Wärmeschutz | Zentrales Dokument für Energieausweis AT |
| **OIB-Richtlinie 6 (2025)** | *In Vorbereitung* - Umsetzung EPBD | Ab Mai 2026 in Landesrecht |
| **OIB-Leitfaden** | Energietechnisches Verhalten von Gebäuden | Berechnungsmethodik |
| ÖNORM B 8110-1 | Wärmeschutz im Hochbau - Teil 1: Anforderungen | U-Wert-Anforderungen |
| ÖNORM B 8110-2 | Wärmeschutz im Hochbau - Teil 2: Wasserdampfdiffusion | Feuchteschutz |
| ÖNORM B 8110-3 | Wärmeschutz im Hochbau - Teil 3: Sommerlicher Wärmeschutz | Überhitzungsschutz |
| ÖNORM B 8110-5 | Wärmeschutz im Hochbau - Teil 5: Klimamodell | Klimadaten AT |
| ÖNORM B 8110-6 | Wärmeschutz im Hochbau - Teil 6: Heizwärmebedarf | HWB-Berechnung |
| ÖNORM H 5050 | Gesamtenergieeffizienz von Gebäuden - Berechnung | Endenergiebedarf |
| ÖNORM H 5055 | Gesamtenergieeffizienz - Begleitende Dokumente | Energieausweis-Format |
| ÖNORM H 5056-1 (2024-03) | Gesamtenergieeffizienz - Heiztechnikenergiebedarf | Heizung (aktuell!) |
| ÖNORM H 5057 | Gesamtenergieeffizienz - Raumlufttechnikenergiebedarf | RLT |
| ÖNORM H 5058 | Gesamtenergieeffizienz - Kühlenergiebedarf | Kühlung |
| ÖNORM H 5059 | Gesamtenergieeffizienz - Beleuchtungsenergiebedarf | Beleuchtung (Nicht-Wohnbau) |
| ÖNORM H 5060 | Gesamtenergieeffizienz - Haushaltsstrombedarf | Haushaltsstrom |

### 2.2 Deutschland 🇩🇪

| Norm/Gesetz | Titel | Anwendung |
|-------------|-------|-----------|
| **GEG 2024** | Gebäudeenergiegesetz | Gesetzliche Grundlage (kostenlos online) |
| DIN V 18599-1 | Energetische Bewertung - Teil 1: Allgemeine Bilanzierung | Grundlagen |
| DIN V 18599-2 | Teil 2: Nutzenergiebedarf Heizen/Kühlen | Zonenberechnung |
| DIN V 18599-3 | Teil 3: Nutzenergiebedarf Warmwasser | TWW |
| DIN V 18599-4 | Teil 4: Nutz- und Endenergiebedarf Beleuchtung | Beleuchtung |
| DIN V 18599-5 | Teil 5: Endenergiebedarf Heizsysteme | Heizung |
| DIN V 18599-6 | Teil 6: Endenergiebedarf RLT, Befeuchtung | Lüftung |
| DIN V 18599-7 | Teil 7: Endenergiebedarf Kälteerzeugung | Kühlung |
| DIN V 18599-8 | Teil 8: Nutz- und Endenergiebedarf Warmwasserbereitung | TWW-Details |
| DIN V 18599-9 | Teil 9: End- und Primärenergiebedarf KWK | KWK |
| DIN V 18599-10 | Teil 10: Nutzungsrandbedingungen | Nutzungsprofile |
| DIN V 18599-11 | Teil 11: Gebäudeautomation | Automation |
| DIN V 18599-12 | Teil 12: Tabellenverfahren Wohngebäude | Vereinfacht |
| DIN V 4108-6 | Wärmeschutz - Teil 6: Berechnung Jahresheizwärmebedarf | Alternativ zu 18599 |
| DIN V 4701-10 | Energetische Bewertung Heizungs-/Lüftungsanlagen | Anlagentechnik |

---

## 3. Heizlast & Kühllast

### 3.1 Österreich 🇦🇹

| Norm | Titel | Anwendung |
|------|-------|-----------|
| ÖNORM H 7500-1 | Heizlast - Gebäudeheizlast | Norm-Heizlast Berechnung |
| ÖNORM H 7500-2 | Heizlast - Raumheizlast | Raumweise Berechnung |
| ÖNORM EN 12831-1 | Heizungsanlagen - Heizlastberechnung | EU-Norm (AT-Fassung) |
| ÖNORM H 6040 | Kühllast | Kühllastberechnung |
| ÖNORM EN 15255 | Gebäudekühlung - Kühllast | EU-Norm (AT-Fassung) |

### 3.2 Deutschland 🇩🇪

| Norm | Titel | Anwendung |
|------|-------|-----------|
| DIN EN 12831-1 | Heizungsanlagen - Verfahren Heizlastberechnung | Norm-Heizlast |
| DIN EN 12831-3 | Heizungsanlagen - Trinkwassererwärmung | TWW-Last |
| DIN SPEC 12831-1 | Nationale Ergänzungen | DE-spezifische Werte |
| VDI 2078 | Berechnung der thermischen Lasten | Kühllastberechnung |
| VDI 6007-1 | Raumtemperaturberechnung - Grundlagen | Dynamische Simulation |
| VDI 6007-2 | Raumtemperaturberechnung - Fenster/Glasfassaden | Solare Einträge |
| VDI 6007-3 | Raumtemperaturberechnung - Anlagentechnik | Anlagenkopplung |

---

## 4. Lüftung & Klimatechnik

### 4.1 Österreich 🇦🇹

| Norm | Titel | Anwendung |
|------|-------|-----------|
| ÖNORM H 6000 | Lüftungstechnische Anlagen - Hygieneanforderungen | Hygiene |
| ÖNORM H 6001 | Lüftungstechnische Anlagen - Brandschutz | Brandschutz |
| ÖNORM H 6003 | Lüftungstechnische Anlagen - Bemessung | Auslegung |
| ÖNORM H 6038 | Lüftungstechnische Anlagen - Wohnraumlüftung | Wohnbau |
| ÖNORM EN 16798-1 | Eingangsparameter Innenraumklima | Raumkonditionen |
| ÖNORM EN 16798-3 | Lüftung Nichtwohngebäude | Auslegung |
| ÖNORM EN 13779 | Lüftung Nichtwohngebäude | Anforderungen |

### 4.2 Deutschland 🇩🇪

| Norm | Titel | Anwendung |
|------|-------|-----------|
| DIN EN 16798-1 | Eingangsparameter Innenraumklima | Raumkonditionen |
| DIN EN 16798-3 | Lüftung Nichtwohngebäude - Leistungsanforderungen | Auslegung |
| DIN 1946-4 | RLT-Anlagen in Gebäuden des Gesundheitswesens | Krankenhäuser |
| DIN 1946-6 | Lüftung von Wohnungen | Wohnbau |
| DIN EN 13779 | Lüftung Nichtwohngebäude | Allgemein |
| VDI 2052 | RLT-Anlagen für Küchen | Großküchen |
| VDI 2262 | Luftbeschaffenheit am Arbeitsplatz | Arbeitsschutz |
| VDI 3803 | RLT-Anlagen - Bauliche/technische Anforderungen | Zentrale RLT |
| VDI 6022 | Raumlufttechnik - Hygiene-Anforderungen | Hygiene |

---

## 5. Wärmeerzeugung

### 5.1 Österreich 🇦🇹

| Norm | Titel | Anwendung |
|------|-------|-----------|
| ÖNORM H 5151 | Berechnung Jahresnutzungsgrad Heizkessel | Kessel-Effizienz |
| ÖNORM H 5170 | Planung von Zentralheizungen | Heizungsplanung |
| ÖNORM EN 15316-1 | Heizungsanlagen - Energiebedarf | Berechnung |
| ÖNORM EN 15316-2 | Wärmeabgabesysteme | Übergabe |
| ÖNORM EN 15316-3 | Wärmeverteilungssysteme | Verteilung |
| ÖNORM EN 15316-4-1 | Wärmeerzeugung - Verbrennung | Kessel |
| ÖNORM EN 15316-4-2 | Wärmeerzeugung - Wärmepumpen | Wärmepumpen |
| ÖNORM EN 15316-4-3 | Wärmeerzeugung - Thermische Solar | Solar |
| ÖNORM EN 15316-4-4 | Wärmeerzeugung - KWK | BHKW |
| ÖNORM EN 15316-4-5 | Wärmeerzeugung - Fernwärme | Fernwärme |
| ÖNORM M 7510 | Holzbefeuerte Anlagen | Biomasse |

### 5.2 Deutschland 🇩🇪

| Norm | Titel | Anwendung |
|------|-------|-----------|
| DIN EN 15316-1 bis -5 | Heizungsanlagen (wie AT) | Berechnung |
| DIN EN 12831-3 | Heizlast Trinkwassererwärmung | TWW |
| DIN 4701-10 | Energetische Bewertung | Anlagenbewertung |
| VDI 2067 | Wirtschaftlichkeit gebäudetechnischer Anlagen | Wirtschaftlichkeit |
| VDI 4640 | Thermische Nutzung des Untergrunds | Geothermie |
| VDI 4645 | Planung und Dimensionierung Wärmepumpen | Wärmepumpen |
| VDI 4650-1 | Berechnung Wärmepumpen - Kurzverfahren | WP-JAZ |
| **VDI-EE 4650 Bl. 1.1 (2023)** | Ergänzung: Vorlauftemperaturen bis 60°C | WP-JAZ (aktuell!) |

---

## 6. Stromerzeugung & Photovoltaik

### 6.1 Österreich 🇦🇹

| Norm | Titel | Anwendung |
|------|-------|-----------|
| ÖNORM EN 15316-4-6 | PV-Systeme für Wärmepumpen | PV+WP |
| OVE E 8101 | Errichtung von elektrischen Anlagen | Installation |
| OVE R 6-2-1 | Blitz- und Überspannungsschutz PV | Blitzschutz |
| ÖVE/ÖNORM EN 62446-1 | PV-Systeme - Prüfung, Dokumentation | Inbetriebnahme |

### 6.2 Deutschland 🇩🇪

| Norm | Titel | Anwendung |
|------|-------|-----------|
| DIN EN 62446-1 | PV-Systeme - Prüfung, Dokumentation | Inbetriebnahme |
| DIN VDE 0100-712 | Errichtung Niederspannungsanlagen - PV | Installation |
| VDI 6012-1.1 | Energetische Inspektion Energieanlagen | Inspektion |
| VDE-AR-N 4105 | Erzeugungsanlagen am Niederspannungsnetz | Netzanschluss |
| DIN EN 15316-4-6 | PV-Systeme für Wärmepumpen | PV+WP |

---

## 7. Energieeffizienz Industrie

### 7.1 Österreich 🇦🇹

| Norm | Titel | Anwendung |
|------|-------|-----------|
| ÖNORM EN ISO 50001 | Energiemanagementsysteme | EnMS |
| ÖNORM EN ISO 50002 | Energieaudits | Audit-Durchführung |
| ÖNORM EN ISO 50003 | EnMS-Auditierung und Zertifizierung | Zertifizierung |
| ÖNORM EN ISO 50004 | EnMS - Umsetzungsleitfaden | Implementierung |
| ÖNORM EN ISO 50006 | EnMS - Energieleistungskennzahlen | EnPI |
| ÖNORM EN ISO 50015 | Messung und Verifizierung | M&V |
| ÖNORM EN 16231 | Energieeffizienz-Benchmarking | Benchmarking |
| ÖNORM EN 16247-1 | Energieaudits - Teil 1: Allgemein | Audit allgemein |
| ÖNORM EN 16247-2 | Energieaudits - Teil 2: Gebäude | Gebäude-Audit |
| ÖNORM EN 16247-3 | Energieaudits - Teil 3: Prozesse | Prozess-Audit |
| ÖNORM EN 16247-4 | Energieaudits - Teil 4: Transport | Transport-Audit |
| ÖNORM EN 16247-5 | Energieaudits - Teil 5: Kompetenz | Auditor-Kompetenz |

### 7.2 Deutschland 🇩🇪

| Norm | Titel | Anwendung |
|------|-------|-----------|
| DIN EN ISO 50001-50015 | Energiemanagement (wie AT) | EnMS |
| DIN EN 16247-1 bis -5 | Energieaudits (wie AT) | Audits |
| VDI 4602 | Energiemanagement | Begriffe, Grundlagen |
| VDI 4801 | Ressourceneffizienz in KMU | KMU |

---

## 8. Druckluft

### 8.1 Österreich 🇦🇹

| Norm | Titel | Anwendung |
|------|-------|-----------|
| ÖNORM EN ISO 8573-1 | Druckluft - Qualitätsklassen | Druckluftqualität |
| ÖNORM EN ISO 1217 | Verdrängerkompressoren - Abnahmeprüfung | Leistungsprüfung |
| ÖNORM EN 1012-1 | Kompressoren - Sicherheitsanforderungen | Sicherheit |

### 8.2 Deutschland 🇩🇪

| Norm | Titel | Anwendung |
|------|-------|-----------|
| DIN EN ISO 8573-1 bis -9 | Druckluft - Qualität | Qualitätsklassen |
| DIN EN ISO 1217 | Verdrängerkompressoren - Abnahmeprüfung | Leistungsprüfung |
| DIN EN 1012-1 | Kompressoren - Sicherheit | Sicherheit |
| VDI 4640 | Druckluftanlagen | Energieeffizienz |
| **VDMA 4370** | Druckluft-Energieeffizienz | Branchenstandard |

**Praxishinweis Druckluft:**
- Spezifischer Energiebedarf: ca. 0,08-0,12 kWh/m³ (effiziente Anlage)
- Leckage-Verluste typisch: 20-30% (Altanlagen bis 50%)
- Wärmerückgewinnung: bis 94% der eingesetzten Energie nutzbar

---

## 9. Vakuumtechnik

### 9.1 Allgemein (AT/DE)

| Norm | Titel | Anwendung |
|------|-------|-----------|
| DIN 28400 | Vakuumtechnik - Begriffe | Grundlagen |
| DIN EN ISO 21360 | Vakuumpumpen - Leistungsmessung | Prüfung |
| DIN EN ISO 3529 | Vakuumtechnik - Dichtheit | Leckagen |
| DIN 28426 | Vakuumpumpen - Kenngrößen | Auslegung |

**Praxishinweis Vakuum:**
- Vakuumsysteme sind oft überdimensioniert → Anpassung spart Energie
- Zentrale vs. dezentrale Vakuumerzeugung analysieren
- Leckage-Erkennung wichtig für Effizienz

---

## 10. Abwärmenutzung & Wärmerückgewinnung

### 10.1 Österreich 🇦🇹

| Norm | Titel | Anwendung |
|------|-------|-----------|
| ÖNORM EN 308 | Wärmeaustauscher - Prüfverfahren | Prüfung |
| ÖNORM EN 13141-7 | Lüftung - WRG Leistungsprüfung | WRG-Prüfung |
| ÖNORM H 6038 | Wohnraumlüftung mit WRG | Wohnbau |
| ÖNORM EN 16798-3 | Lüftung - Leistungsanforderungen | WRG-Anforderungen |

### 10.2 Deutschland 🇩🇪

| Norm | Titel | Anwendung |
|------|-------|-----------|
| DIN EN 308 | Wärmeaustauscher - Prüfverfahren | Prüfung |
| DIN EN 13141-7 | Lüftung - WRG Leistungsprüfung | WRG-Prüfung |
| VDI 2071 | Wärmerückgewinnung RLT | WRG-Auslegung |
| VDI 3985 | Abwärmenutzung Industrie | Industrielle Abwärme |
| **VDI 4610** | Energieeffizienz Dämmsysteme | Dämmung Rohre |

**Praxishinweis Abwärme:**
- Abwärme-Potenzial: oft 30-50% des Energieeinsatzes
- Temperaturniveaus: Hochtemperatur (>300°C), Mittel (100-300°C), Niedrig (<100°C)
- Integration: Wärmepumpen können Niedertemperatur-Abwärme aufwerten

---

## 11. Wärmetauscher

### 11.1 Allgemein (AT/DE)

| Norm | Titel | Anwendung |
|------|-------|-----------|
| DIN EN 305 | Wärmeaustauscher - Definitionen | Begriffe |
| DIN EN 306 | Wärmeaustauscher - Messverfahren | Messung |
| DIN EN 307 | Wärmeaustauscher - Daten für Ausschreibung | Ausschreibung |
| DIN EN 308 | Wärmeaustauscher - Prüfverfahren | Leistungsprüfung |
| DIN EN 1148 | Wärmeaustauscher - Terminologie | Begriffe |
| VDI 2069 | Verhinderung von Schäden Wärmetauscher | Schadenverhütung |

**Praxishinweis Wärmetauscher:**
- Dimensionierung: NTU-Methode oder LMTD-Methode
- Fouling-Faktoren berücksichtigen (Verschmutzung)
- Druckverlust vs. Wärmeübertragung optimieren

---

## 12. Power-to-X (PtX)

### 12.1 Wasserstoff (H₂)

| Norm | Titel | Anwendung |
|------|-------|-----------|
| DIN EN ISO 22734 | Wasserstofferzeuger - Elektrolyseure | Elektrolyse |
| DIN EN 17124 | Wasserstoff als Kraftstoff - Qualität | Qualität |
| DIN EN ISO 14687 | Wasserstoff - Produktspezifikation | Spezifikation |
| DIN EN ISO 19880 | Wasserstoff-Tankstellen | Betankung |
| DIN EN ISO 19881 | Druckwasserstoff-Speicherung | Speicherung |

### 12.2 Power-to-Heat / Power-to-Gas

| Norm | Titel | Anwendung |
|------|-------|-----------|
| DVGW G 260 | Gasbeschaffenheit | Einspeise-Qualität |
| DVGW G 262 | Nutzung von Gasen aus regenerativen Quellen | Biomethan/H₂ |
| DIN EN ISO 15403 | Erdgas als Kraftstoff | CNG/LNG |

**Praxishinweis PtX:**
- Elektrolyse-Wirkungsgrad: 60-80% (PEM: 60-70%, Alkalisch: 70-80%)
- Power-to-Heat: COP > 3 bei Wärmepumpen vs. 1,0 bei Elektroheizer
- Sektorenkopplung: H₂ für Industrie, Mobilität, Rückverstromung

---

## 13. Energieaudits

### 13.1 Österreich 🇦🇹

| Norm/Gesetz | Titel | Anwendung |
|-------------|-------|-----------|
| **EEffG** | Bundes-Energieeffizienzgesetz | Gesetzliche Pflicht (kostenlos online) |
| ÖNORM EN 16247-1 bis -5 | Energieaudits | Durchführung |
| ÖNORM EN ISO 50002 | Energieaudits - Anforderungen | Audit-Standard |
| **Monitoringstelle** | Leitfäden und Vorlagen | Praxis-Hilfe (kostenlos) |

### 13.2 Deutschland 🇩🇪

| Norm/Gesetz | Titel | Anwendung |
|-------------|-------|-----------|
| **EDL-G** | Energiedienstleistungsgesetz | Gesetzliche Pflicht |
| **EnEfG (2023)** | Energieeffizienzgesetz | Erweiterte Pflichten! |
| DIN EN 16247-1 (2022) | Energieaudits - Allgemein | Durchführung (aktualisiert!) |
| DIN EN 16247-2 bis -5 | Energieaudits - Spezifisch | Gebäude/Prozess/Transport |
| DIN EN ISO 50002 | Energieaudits - Anforderungen | Audit-Standard |
| **BAFA-Merkblätter** | Merkblätter Energieaudits | Praxis-Hilfe (kostenlos) |

**EnEfG-Pflichten (Stand 2025):**
- Unternehmen > 2,5 GWh/Jahr: Energieaudit-Pflicht
- Unternehmen > 7,5 GWh/Jahr: EnMS (ISO 50001/EMAS) Pflicht seit Juli 2025

---

## 14. Sanierung & Renovierung

### 14.1 Österreich 🇦🇹

| Norm/Richtlinie | Titel | Anwendung |
|-----------------|-------|-----------|
| **OIB-Richtlinie 6** | Energieeinsparung - Sanierung | Anforderungen |
| ÖNORM B 8110-4 | Wärmeschutz - Kondensation Bauteile | Bauphysik Sanierung |
| ÖNORM H 5050 | Gesamtenergieeffizienz | Nachweis |
| **Sanierungsscheck** | Förderrichtlinien | Förderung (kostenlos) |

### 14.2 Deutschland 🇩🇪

| Norm/Gesetz | Titel | Anwendung |
|-------------|-------|-----------|
| **GEG § 48** | Anforderungen bei Änderung Gebäude | Gesetzlich |
| DIN V 18599-11 | Gebäudeautomation | Automation-Potenzial |
| **BEG-Richtlinien** | Bundesförderung effiziente Gebäude | Förderung (kostenlos) |
| **iSFP** | Individueller Sanierungsfahrplan | Beratung |

---

## 15. Bauphysik & Wärmeschutz

### 15.1 Österreich 🇦🇹

| Norm | Titel | Anwendung |
|------|-------|-----------|
| ÖNORM B 8110-1 | Wärmeschutz - Anforderungen | U-Werte |
| ÖNORM B 8110-2 | Wärmeschutz - Wasserdampfdiffusion | Feuchteschutz |
| ÖNORM B 8110-3 | Wärmeschutz - Sommerlicher Wärmeschutz | Überhitzung |
| ÖNORM B 8110-4 | Wärmeschutz - Kondensation | Taupunkt |
| ÖNORM EN ISO 6946 | Bauteile - Wärmedurchlasswiderstand | U-Wert-Berechnung |
| ÖNORM EN ISO 10211 | Wärmebrücken | ψ-Werte |
| ÖNORM EN ISO 13370 | Wärmeübertragung über Erdreich | Bodenplatte |
| ÖNORM EN ISO 13788 | Oberflächentemperatur und Kondensation | Schimmel |
| ÖNORM EN ISO 13789 | Wärmedurchgangskoeffizienten | Transmission |

### 15.2 Deutschland 🇩🇪

| Norm | Titel | Anwendung |
|------|-------|-----------|
| DIN 4108-2 | Wärmeschutz - Mindestanforderungen | Mindestwärmeschutz |
| DIN 4108-3 | Wärmeschutz - Klimabedingter Feuchteschutz | Feuchteschutz |
| DIN 4108-4 | Wärmeschutz - Wärme- und feuchteschutztechnische Bemessungswerte | Rechenwerte |
| DIN 4108-6 | Wärmeschutz - Berechnung Jahresheizwärmebedarf | HWB |
| DIN 4108 Beiblatt 2 | Wärmebrücken | Regeldetails |
| DIN EN ISO 6946 | Bauteile - U-Wert | U-Wert-Berechnung |
| DIN EN ISO 10211 | Wärmebrücken | Detailberechnung |
| DIN EN ISO 13370 | Erdreich | Bodenplatte |

---

## 16. Beleuchtung

### 16.1 Österreich 🇦🇹

| Norm | Titel | Anwendung |
|------|-------|-----------|
| ÖNORM EN 12464-1 | Beleuchtung Arbeitsstätten - Innenräume | Innenbeleuchtung |
| ÖNORM EN 12464-2 | Beleuchtung Arbeitsstätten - Freien | Außenbeleuchtung |
| ÖNORM EN 15193 | Energetische Bewertung Beleuchtung | Energiebedarf |
| ÖNORM H 5059 | Beleuchtungsenergiebedarf | Energieausweis |

### 16.2 Deutschland 🇩🇪

| Norm | Titel | Anwendung |
|------|-------|-----------|
| DIN EN 12464-1 | Beleuchtung Arbeitsstätten - Innenräume | Innenbeleuchtung |
| DIN EN 12464-2 | Beleuchtung Arbeitsstätten - Freien | Außenbeleuchtung |
| DIN V 18599-4 | Nutz- und Endenergiebedarf Beleuchtung | Energiebedarf |
| DIN EN 15193 | Energetische Bewertung Beleuchtung | LENI-Berechnung |
| ASR A3.4 | Beleuchtung | Arbeitsschutz |

---

## 17. Elektrische Anlagen

### 17.1 Österreich 🇦🇹

| Norm | Titel | Anwendung |
|------|-------|-----------|
| OVE E 8101 | Errichtung elektrischer Anlagen | Installation |
| OVE EN 50600 | Rechenzentren | Energieeffizienz RZ |
| ÖNORM EN ISO 50001 | Energiemanagement | Strommessung |

### 17.2 Deutschland 🇩🇪

| Norm | Titel | Anwendung |
|------|-------|-----------|
| DIN VDE 0100 | Errichtung Niederspannungsanlagen | Installation |
| DIN EN 50600 | Rechenzentren | Energieeffizienz RZ |
| VDI 4602 | Energiemanagement | Begriffe |

---

## 18. Förderungen & Nachweise

### 18.1 Österreich 🇦🇹 (Kostenlose Quellen)

| Quelle | URL | Inhalt |
|--------|-----|--------|
| OIB-Richtlinien | oib.or.at | Alle OIB-Richtlinien PDF |
| Monitoringstelle | monitoringstelle.at | EEffG-Leitfäden |
| Klimafonds | klimafonds.gv.at | Förderrichtlinien |
| UFI | umweltfoerderung.at | Umweltförderung |

### 18.2 Deutschland 🇩🇪 (Kostenlose Quellen)

| Quelle | URL | Inhalt |
|--------|-----|--------|
| GEG-Volltext | gesetze-im-internet.de | Gesetzestext |
| BAFA | bafa.de | Energieaudit-Merkblätter |
| KfW | kfw.de | BEG-Förderrichtlinien |
| dena | dena.de | iSFP-Leitfäden |

---

## Anhang A: Normen-Mapping nach Berechnungsmodul

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  Berechnungsmodul          │  Primär-Norm AT         │  Primär-Norm DE        │
├────────────────────────────┼─────────────────────────┼────────────────────────┤
│  Energieausweis Wohnbau    │  OIB-RL 6 + ÖNORM H5050 │  DIN V 18599           │
│  Energieausweis Nicht-WB   │  OIB-RL 6 + ÖNORM H5050 │  DIN V 18599           │
│  Heizlast                  │  ÖNORM H 7500           │  DIN EN 12831          │
│  Kühllast                  │  ÖNORM H 6040           │  VDI 2078              │
│  U-Wert                    │  ÖNORM EN ISO 6946      │  DIN EN ISO 6946       │
│  Wärmebrücken              │  ÖNORM EN ISO 10211     │  DIN 4108 Bbl 2        │
│  Lüftung Wohnbau           │  ÖNORM H 6038           │  DIN 1946-6            │
│  Lüftung Nicht-Wohnbau     │  ÖNORM EN 16798-3       │  DIN EN 16798-3        │
│  WRG-Effizienz             │  ÖNORM EN 13141-7       │  VDI 2071              │
│  Wärmepumpen-JAZ           │  ÖNORM EN 15316-4-2     │  VDI 4650              │
│  Photovoltaik              │  OVE/ÖNORM EN 62446     │  DIN EN 62446          │
│  Energieaudit              │  ÖNORM EN 16247         │  DIN EN 16247          │
│  Druckluft-Effizienz       │  ÖNORM EN ISO 8573      │  VDMA 4370             │
│  Beleuchtung               │  ÖNORM H 5059           │  DIN V 18599-4         │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## Anhang B: Versionierung

| Version | Datum | Änderungen |
|---------|-------|------------|
| 1.0.0 | 2025-01-15 | Initiale Version |
| 1.1.0 | 2025-11-26 | Web-Recherche: OIB-RL 6 2025, ÖNORM H 5056-1:2024, VDI-EE 4650, EnEfG |

---

**Hinweis:** Diese Übersicht dient als Referenz für die benötigten Normen. Die Normentexte selbst müssen bei Austrian Standards (AT) bzw. Beuth Verlag (DE) bezogen werden. Gesetze (GEG, EEffG, OIB-Richtlinien) und behördliche Leitfäden sind kostenlos online verfügbar.

---

## Anhang C: Quellen (Recherche November 2025)

- [OIB-Richtlinien 2025](https://www.oib.or.at/kernaufgaben/oib-richtlinien/richtlinien/oib-richtlinien-2025/) - Status OIB-RL 6 2025
- [GEG 2024/2025](https://www.gesetze-im-internet.de/geg/) - Offizieller Gesetzestext
- [Austrian Standards](https://www.austrian-standards.at) - ÖNORM H 5056-1:2024
- [VDI Richtlinien](https://www.vdi.de/richtlinien) - VDI 2078, VDI 4650
- [BAFA Energieaudit](https://www.bafa.de) - EnEfG-Merkblätter
- [DIN Media](https://www.dinmedia.de) - DIN V 18599 Informationen
