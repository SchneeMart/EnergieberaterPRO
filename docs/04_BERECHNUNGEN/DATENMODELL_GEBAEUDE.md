# Datenmodell: Gebäude und Berechnungen

## Übersicht Datenbankstruktur

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ENTITY-RELATIONSHIP OVERVIEW                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Organisation                                                                │
│       │                                                                      │
│       ├── Endkunden                                                          │
│       │       │                                                              │
│       │       └── Gebäude                                                    │
│       │               │                                                      │
│       │               ├── Gebäudeversionen (Zeitreihe)                      │
│       │               │       │                                              │
│       │               │       ├── Zonen                                      │
│       │               │       │       └── Räume                              │
│       │               │       │                                              │
│       │               │       ├── Hüllflächen                                │
│       │               │       │       └── Bauteilschichten                   │
│       │               │       │                                              │
│       │               │       ├── Anlagentechnik                             │
│       │               │       │       ├── Heizung                            │
│       │               │       │       ├── TWW                                │
│       │               │       │       ├── Lüftung                            │
│       │               │       │       └── Solar                              │
│       │               │       │                                              │
│       │               │       └── Berechnungen                               │
│       │               │               ├── Energiebilanz                      │
│       │               │               ├── Heizlast                           │
│       │               │               └── Varianten                          │
│       │               │                                                      │
│       │               └── Projekte                                           │
│       │                       │                                              │
│       │                       ├── Dokumente                                  │
│       │                       └── Aufgaben                                   │
│       │                                                                      │
│       └── Mitarbeiter                                                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Detaillierte Tabellenstrukturen

### 1. Gebäude-Stammdaten

```sql
-- Haupttabelle für Gebäude
CREATE TABLE buildings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID NOT NULL REFERENCES organizations(id),
    customer_id UUID NOT NULL REFERENCES customers(id),

    -- Identifikation
    internal_reference VARCHAR(50),  -- Interne Projektnummer
    external_reference VARCHAR(50),  -- z.B. Kataster-Nr.

    -- Adresse
    street VARCHAR(255) NOT NULL,
    house_number VARCHAR(20) NOT NULL,
    postal_code VARCHAR(10) NOT NULL,
    city VARCHAR(100) NOT NULL,
    district VARCHAR(100),  -- Ortsteil
    federal_state VARCHAR(50) NOT NULL,
    country CHAR(2) NOT NULL DEFAULT 'DE',

    -- Geografische Daten
    latitude DECIMAL(10, 8),
    longitude DECIMAL(11, 8),
    altitude_m INTEGER,  -- Höhe über NN

    -- Klimadaten (aus Standort abgeleitet)
    climate_zone VARCHAR(20),  -- z.B. "TRY Region 4"
    design_outdoor_temp DECIMAL(4,1),  -- Norm-Außentemperatur
    heating_degree_days INTEGER,  -- Gradtagszahl
    solar_radiation_annual DECIMAL(6,1),  -- kWh/(m²·a)

    -- Metadaten
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID REFERENCES users(id),
    is_archived BOOLEAN DEFAULT FALSE,

    CONSTRAINT fk_organization FOREIGN KEY (organization_id)
        REFERENCES organizations(id) ON DELETE CASCADE
);

-- Index für Suche
CREATE INDEX idx_buildings_address ON buildings(postal_code, city);
CREATE INDEX idx_buildings_customer ON buildings(customer_id);
CREATE INDEX idx_buildings_org ON buildings(organization_id);
```

### 2. Gebäudeversionen (Zeitreihe)

```sql
-- Versionen eines Gebäudes (Ist-Zustand, Varianten, Saniert)
CREATE TABLE building_versions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    building_id UUID NOT NULL REFERENCES buildings(id) ON DELETE CASCADE,

    -- Versionsinfo
    version_type VARCHAR(20) NOT NULL,  -- 'ist', 'variante', 'saniert'
    version_number INTEGER NOT NULL,
    version_name VARCHAR(100),  -- z.B. "Ist-Zustand 2024", "Variante A"
    parent_version_id UUID REFERENCES building_versions(id),

    -- Grunddaten
    building_type VARCHAR(30) NOT NULL,  -- EFH, MFH, NWG, etc.
    building_subtype VARCHAR(50),  -- freistehend, Reihenendhaus, etc.
    construction_year INTEGER NOT NULL,
    last_renovation_year INTEGER,

    -- Größen (berechnet oder eingegeben)
    residential_units INTEGER DEFAULT 1,
    commercial_units INTEGER DEFAULT 0,
    floors_above_ground INTEGER NOT NULL,
    floors_below_ground INTEGER DEFAULT 0,

    -- Flächen
    living_area_m2 DECIMAL(10,2),  -- Wohnfläche nach WoFlV
    usable_area_an_m2 DECIMAL(10,2),  -- Nutzfläche AN nach GEG
    gross_floor_area_m2 DECIMAL(10,2),  -- Bruttogeschossfläche
    heated_volume_m3 DECIMAL(10,2),  -- Beheiztes Volumen
    envelope_area_m2 DECIMAL(10,2),  -- Hüllfläche

    -- Kompaktheit
    a_v_ratio DECIMAL(5,3),  -- A/V-Verhältnis
    form_factor DECIMAL(5,3),

    -- Luftdichtheit
    n50_measured DECIMAL(4,2),  -- Blower-Door gemessen
    n50_assumed DECIMAL(4,2),  -- Angesetzter Wert
    blower_door_date DATE,
    blower_door_certificate_id UUID,

    -- Berechnungseinstellungen
    calculation_method VARCHAR(30) DEFAULT 'DIN_V_18599',
    thermal_bridge_method VARCHAR(30) DEFAULT 'pauschal',
    delta_uwb DECIMAL(4,3) DEFAULT 0.10,

    -- Status
    status VARCHAR(20) DEFAULT 'draft',  -- draft, calculated, approved
    locked_at TIMESTAMP,
    locked_by UUID REFERENCES users(id),

    -- Metadaten
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID REFERENCES users(id),

    UNIQUE(building_id, version_number)
);

CREATE INDEX idx_building_versions_building ON building_versions(building_id);
CREATE INDEX idx_building_versions_type ON building_versions(version_type);
```

### 3. Thermische Zonen

```sql
-- Thermische Zonen eines Gebäudes
CREATE TABLE thermal_zones (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    building_version_id UUID NOT NULL REFERENCES building_versions(id) ON DELETE CASCADE,

    zone_number INTEGER NOT NULL,
    zone_name VARCHAR(100) NOT NULL,
    zone_type VARCHAR(30) NOT NULL,  -- beheizt, unbeheizt, unterschiedlich_beheizt

    -- Nutzung
    usage_profile VARCHAR(50),  -- nach DIN V 18599-10
    usage_hours_per_day DECIMAL(4,1),
    usage_days_per_year INTEGER,

    -- Temperaturen
    heating_setpoint_c DECIMAL(4,1) DEFAULT 20.0,
    cooling_setpoint_c DECIMAL(4,1),

    -- Flächen
    floor_area_m2 DECIMAL(10,2) NOT NULL,
    volume_m3 DECIMAL(10,2),

    -- Beleuchtung (für NWG)
    lighting_power_w_m2 DECIMAL(6,2),
    lighting_control VARCHAR(30),

    -- Lüftung
    ventilation_type VARCHAR(30),  -- natuerlich, mechanisch_abluft, zu_abluft
    air_change_rate DECIMAL(4,2),

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(building_version_id, zone_number)
);

CREATE INDEX idx_zones_version ON thermal_zones(building_version_id);
```

### 4. Räume (für Heizlastberechnung)

```sql
-- Einzelne Räume innerhalb einer Zone
CREATE TABLE rooms (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    zone_id UUID NOT NULL REFERENCES thermal_zones(id) ON DELETE CASCADE,

    room_number VARCHAR(20),
    room_name VARCHAR(100) NOT NULL,
    floor_level INTEGER NOT NULL,  -- Geschoss (0 = EG)

    -- Nutzung
    room_type VARCHAR(30) NOT NULL,  -- Wohnraum, Bad, Kueche, etc.

    -- Geometrie
    length_m DECIMAL(6,2),
    width_m DECIMAL(6,2),
    height_m DECIMAL(4,2),
    floor_area_m2 DECIMAL(8,2) NOT NULL,
    volume_m3 DECIMAL(10,2),

    -- Temperaturen
    design_indoor_temp_c DECIMAL(4,1),  -- Auslegungstemperatur

    -- Heizlast-Ergebnisse
    transmission_heat_loss_w DECIMAL(10,1),
    ventilation_heat_loss_w DECIMAL(10,1),
    heating_up_allowance_w DECIMAL(10,1),
    room_heating_load_w DECIMAL(10,1),

    -- Installierte Heizfläche
    installed_heating_power_w DECIMAL(10,1),
    heating_surface_type VARCHAR(30),  -- Heizkoerper, FBH, etc.
    design_flow_temp_c DECIMAL(4,1),
    design_return_temp_c DECIMAL(4,1),

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_rooms_zone ON rooms(zone_id);
```

### 5. Hüllflächen (Bauteile)

```sql
-- Bauteile der thermischen Hülle
CREATE TABLE envelope_components (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    building_version_id UUID NOT NULL REFERENCES building_versions(id) ON DELETE CASCADE,
    zone_id UUID REFERENCES thermal_zones(id),
    room_id UUID REFERENCES rooms(id),

    -- Bauteiltyp
    component_type VARCHAR(30) NOT NULL,
    -- Typen: Aussenwand, Innenwand, Fenster, Tuer, Dach, Flachdach,
    --        Oberste_Geschossdecke, Kellerdecke, Bodenplatte, Dachschraege

    component_name VARCHAR(100) NOT NULL,
    component_code VARCHAR(20),  -- z.B. AW-01, FE-03

    -- Geometrie
    area_m2 DECIMAL(10,2) NOT NULL,
    length_m DECIMAL(8,2),  -- bei linearen Bauteilen
    height_m DECIMAL(6,2),
    width_m DECIMAL(6,2),

    -- Orientierung
    orientation VARCHAR(20),  -- N, NE, E, SE, S, SW, W, NW, horizontal
    tilt_angle_deg DECIMAL(5,2) DEFAULT 90,  -- 90° = vertikal, 0° = horizontal

    -- Wärmetechnische Eigenschaften
    u_value DECIMAL(5,3) NOT NULL,  -- W/(m²K)
    u_value_source VARCHAR(30),  -- berechnet, tabelle, messung

    -- Bei Fenstern zusätzlich
    glazing_type VARCHAR(50),  -- 2-fach, 3-fach, etc.
    frame_material VARCHAR(30),  -- Holz, Kunststoff, Alu, etc.
    frame_fraction DECIMAL(3,2) DEFAULT 0.30,
    ug_value DECIMAL(4,3),  -- U-Wert Verglasung
    uf_value DECIMAL(4,3),  -- U-Wert Rahmen
    psi_g DECIMAL(5,4),  -- ψ-Wert Glasrand
    g_value DECIMAL(3,2),  -- Gesamtenergiedurchlassgrad

    -- Verschattung (Fenster)
    shading_type VARCHAR(30),  -- keine, Jalousie, Rollladen, etc.
    shading_fc_value DECIMAL(3,2),  -- Abminderungsfaktor

    -- Temperaturbedingungen
    adjacent_condition VARCHAR(30),  -- aussenluft, erdreich, unbeheizt, etc.
    adjacent_temp_c DECIMAL(5,2),  -- Temperatur angrenzend
    temperature_factor DECIMAL(4,3),  -- Temperatur-Korrekturfaktor

    -- Referenz auf Bauteilaufbau
    construction_id UUID REFERENCES constructions(id),

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_envelope_version ON envelope_components(building_version_id);
CREATE INDEX idx_envelope_type ON envelope_components(component_type);
```

### 6. Bauteilaufbauten

```sql
-- Vordefinierte Bauteilaufbauten
CREATE TABLE constructions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID REFERENCES organizations(id),  -- NULL = global

    name VARCHAR(100) NOT NULL,
    code VARCHAR(20),
    component_type VARCHAR(30) NOT NULL,
    description TEXT,

    -- Berechnete Werte
    total_thickness_mm DECIMAL(8,2),
    u_value DECIMAL(5,3),
    r_total DECIMAL(6,3),  -- Gesamtwärmedurchlasswiderstand

    -- Feuchteschutz
    condensation_check_passed BOOLEAN,
    sd_value_inside DECIMAL(8,2),
    sd_value_outside DECIMAL(8,2),

    is_template BOOLEAN DEFAULT FALSE,
    source VARCHAR(100),  -- z.B. "DIN 4108-4 Tabelle X"

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Einzelne Schichten eines Bauteilaufbaus
CREATE TABLE construction_layers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    construction_id UUID NOT NULL REFERENCES constructions(id) ON DELETE CASCADE,

    layer_order INTEGER NOT NULL,  -- von außen nach innen
    material_name VARCHAR(100) NOT NULL,

    -- Eigenschaften
    thickness_mm DECIMAL(6,2) NOT NULL,
    lambda_value DECIMAL(6,4) NOT NULL,  -- W/(mK)
    density_kg_m3 DECIMAL(8,2),
    specific_heat_j_kgk INTEGER,
    mu_value DECIMAL(8,2),  -- Wasserdampfdiffusionswiderstandszahl

    -- Referenz auf Materialdatenbank
    material_id UUID REFERENCES materials(id),

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(construction_id, layer_order)
);

CREATE INDEX idx_layers_construction ON construction_layers(construction_id);
```

### 7. Wärmebrücken

```sql
-- Wärmebrücken
CREATE TABLE thermal_bridges (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    building_version_id UUID NOT NULL REFERENCES building_versions(id) ON DELETE CASCADE,

    -- Typ und Bezeichnung
    bridge_type VARCHAR(50) NOT NULL,
    -- Typen: Außenecke, Innenecke, Fensteranschluss, Sockel, Balkon,
    --        Rollladenkasten, Attika, Dachrand, etc.

    bridge_name VARCHAR(100),
    bridge_code VARCHAR(20),

    -- Geometrie
    length_m DECIMAL(8,2) NOT NULL,

    -- Wärmetechnische Eigenschaften
    psi_value DECIMAL(5,4) NOT NULL,  -- W/(mK)
    psi_value_source VARCHAR(30),  -- DIN_4108_Beiblatt_2, berechnet, katalog

    -- Referenz auf Katalog
    catalog_reference VARCHAR(50),  -- z.B. "DIN 4108 Bbl.2 - B.1.3.2"

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_thermal_bridges_version ON thermal_bridges(building_version_id);
```

### 8. Anlagentechnik - Heizung

```sql
-- Wärmeerzeuger
CREATE TABLE heating_systems (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    building_version_id UUID NOT NULL REFERENCES building_versions(id) ON DELETE CASCADE,

    system_number INTEGER DEFAULT 1,
    system_name VARCHAR(100),

    -- Erzeugertyp
    generator_type VARCHAR(50) NOT NULL,
    -- Typen: Gaskessel_NT, Gaskessel_BW, Oelkessel_NT, Oelkessel_BW,
    --        Pellet, Scheitholz, Hackschnitzel, WP_Luft, WP_Sole, WP_Wasser,
    --        Fernwaerme, Elektro_Direkt, BHKW, Solarthermie, etc.

    generator_subtype VARCHAR(50),  -- z.B. modulierend, einstufig

    -- Basisdaten
    manufacturer VARCHAR(100),
    model VARCHAR(100),
    installation_year INTEGER,
    nominal_power_kw DECIMAL(8,2) NOT NULL,
    modulation_min_percent DECIMAL(5,2),

    -- Aufstellung
    location VARCHAR(30),  -- beheizt, unbeheizt
    installation_type VARCHAR(30),  -- bodenstehend, wandhaengend

    -- Effizienz
    efficiency_full_load DECIMAL(5,3),  -- Wirkungsgrad Volllast
    efficiency_part_load DECIMAL(5,3),  -- Wirkungsgrad Teillast
    standby_loss_w DECIMAL(8,2),  -- Bereitschaftsverlust

    -- Wärmepumpe spezifisch
    heat_source VARCHAR(30),  -- Luft, Erdreich, Grundwasser
    cop_a2_w35 DECIMAL(4,2),  -- COP bei A2/W35
    cop_a7_w35 DECIMAL(4,2),  -- COP bei A7/W35
    cop_b0_w35 DECIMAL(4,2),  -- COP bei B0/W35 (Sole)
    scop DECIMAL(4,2),  -- Seasonal COP
    jaz DECIMAL(4,2),  -- Jahresarbeitszahl (berechnet)

    -- Brennwertkessel spezifisch
    condensing BOOLEAN DEFAULT FALSE,
    flue_gas_temp_c DECIMAL(5,1),

    -- Energieträger
    energy_carrier VARCHAR(30) NOT NULL,
    -- Erdgas, Fluessiggas, Heizoel, Pellet, Scheitholz, Hackschnitzel,
    -- Strom, Strom_WP, Fernwaerme, Biomethan, Wasserstoff

    -- Versorgungsanteil
    coverage_heating_percent DECIMAL(5,2) DEFAULT 100,
    coverage_dhw_percent DECIMAL(5,2) DEFAULT 0,

    -- Regelung
    control_type VARCHAR(30),  -- witterungsgefuehrt, raumgefuehrt, etc.
    night_setback BOOLEAN DEFAULT TRUE,

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_heating_version ON heating_systems(building_version_id);

-- Wärmeverteilung
CREATE TABLE heat_distribution (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    heating_system_id UUID NOT NULL REFERENCES heating_systems(id) ON DELETE CASCADE,

    -- Verteilnetz
    distribution_type VARCHAR(30),  -- Einrohr, Zweirohr
    pipe_insulation VARCHAR(30),  -- keine, maessig, gut, GEG
    pipe_length_heated_m DECIMAL(8,2),
    pipe_length_unheated_m DECIMAL(8,2),

    -- Auslegungstemperaturen
    design_flow_temp_c DECIMAL(5,1),  -- z.B. 55
    design_return_temp_c DECIMAL(5,1),  -- z.B. 45
    design_spread_k DECIMAL(4,1),  -- z.B. 10

    -- Pumpen
    pump_type VARCHAR(30),  -- ungeregelt, geregelt, Hocheffizienz
    pump_power_w DECIMAL(8,2),

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Wärmeübergabe
CREATE TABLE heat_emission (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    heating_system_id UUID NOT NULL REFERENCES heating_systems(id) ON DELETE CASCADE,
    zone_id UUID REFERENCES thermal_zones(id),

    -- Übergabetyp
    emission_type VARCHAR(30) NOT NULL,
    -- Heizkoerper, Fussbodenheizung, Wandheizung, Deckenheizung,
    -- Konvektor, Infrarot, Luftheizung

    -- Regelung
    room_control BOOLEAN DEFAULT TRUE,
    control_type VARCHAR(30),  -- Thermostatventil, elektrisch, etc.

    -- Hydraulischer Abgleich
    hydraulic_balancing BOOLEAN DEFAULT FALSE,
    balancing_method VARCHAR(30),  -- Verfahren_A, Verfahren_B

    coverage_percent DECIMAL(5,2) DEFAULT 100,

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 9. Anlagentechnik - Trinkwarmwasser

```sql
-- Trinkwarmwassersystem
CREATE TABLE dhw_systems (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    building_version_id UUID NOT NULL REFERENCES building_versions(id) ON DELETE CASCADE,

    -- Erzeugung
    generation_type VARCHAR(30) NOT NULL,
    -- zentral_mit_heizung, zentral_separat, dezentral_elektrisch,
    -- dezentral_gas, waermepumpe, solar

    -- Referenz auf Heizungssystem (wenn zentral_mit_heizung)
    heating_system_id UUID REFERENCES heating_systems(id),

    -- Eigener Erzeuger (wenn separat)
    generator_type VARCHAR(50),
    generator_power_kw DECIMAL(8,2),
    generator_efficiency DECIMAL(5,3),

    -- Speicher
    storage_present BOOLEAN DEFAULT TRUE,
    storage_volume_l DECIMAL(8,2),
    storage_insulation VARCHAR(30),  -- Standard, verbesser, GEG
    storage_standby_loss_w DECIMAL(8,2),

    -- Zirkulation
    circulation BOOLEAN DEFAULT FALSE,
    circulation_pump_power_w DECIMAL(6,2),
    circulation_control VARCHAR(30),  -- keine, zeitgesteuert, bedarfsgefuehrt

    -- Verteilung
    pipe_length_heated_m DECIMAL(8,2),
    pipe_length_unheated_m DECIMAL(8,2),
    pipe_insulation VARCHAR(30),

    -- Solar-Unterstützung
    solar_support BOOLEAN DEFAULT FALSE,
    solar_coverage_percent DECIMAL(5,2),

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_dhw_version ON dhw_systems(building_version_id);
```

### 10. Anlagentechnik - Lüftung

```sql
-- Lüftungsanlage
CREATE TABLE ventilation_systems (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    building_version_id UUID NOT NULL REFERENCES building_versions(id) ON DELETE CASCADE,
    zone_id UUID REFERENCES thermal_zones(id),

    -- Anlagentyp
    system_type VARCHAR(30) NOT NULL,
    -- Fensterlüftung, Schachtlüftung, Abluftanlage, Zu_Abluft_ohne_WRG,
    -- Zu_Abluft_mit_WRG, dezentral_mit_WRG

    -- Volumenstrom
    supply_air_flow_m3h DECIMAL(10,2),
    exhaust_air_flow_m3h DECIMAL(10,2),
    air_change_rate DECIMAL(4,2),

    -- Wärmerückgewinnung
    heat_recovery BOOLEAN DEFAULT FALSE,
    heat_recovery_type VARCHAR(30),  -- Platte, Rotations, Kreuzstrom
    heat_recovery_efficiency DECIMAL(5,3),  -- z.B. 0.85 = 85%

    -- Elektrische Leistung
    fan_power_supply_w DECIMAL(8,2),
    fan_power_exhaust_w DECIMAL(8,2),
    sfp_value DECIMAL(4,2),  -- Spezifische Ventilatorleistung

    -- Regelung
    control_type VARCHAR(30),  -- konstant, bedarfsgefuehrt, CO2, Feuchte
    bypass_summer BOOLEAN DEFAULT FALSE,

    -- Erdwärmetauscher (optional)
    earth_tube BOOLEAN DEFAULT FALSE,
    earth_tube_length_m DECIMAL(6,2),

    -- Nach- / Vorheizung
    preheating BOOLEAN DEFAULT FALSE,
    preheating_type VARCHAR(30),
    postheating BOOLEAN DEFAULT FALSE,
    postheating_type VARCHAR(30),

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_ventilation_version ON ventilation_systems(building_version_id);
```

### 11. Anlagentechnik - Solaranlagen

```sql
-- Solarthermie
CREATE TABLE solar_thermal_systems (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    building_version_id UUID NOT NULL REFERENCES building_versions(id) ON DELETE CASCADE,

    -- Kollektoren
    collector_type VARCHAR(30) NOT NULL,  -- Flach, Vakuumroehre, Luft
    collector_area_m2 DECIMAL(8,2) NOT NULL,
    collector_orientation VARCHAR(10),  -- S, SO, SW, etc.
    collector_tilt_deg DECIMAL(5,2),

    -- Kenndaten
    eta_0 DECIMAL(4,3),  -- optischer Wirkungsgrad
    a1 DECIMAL(5,3),  -- linearer Wärmeverlustkoeffizient
    a2 DECIMAL(5,4),  -- quadratischer Wärmeverlustkoeffizient

    -- Speicher
    storage_volume_l DECIMAL(8,2),
    storage_type VARCHAR(30),  -- monovalent, bivalent

    -- Nutzung
    usage VARCHAR(30),  -- TWW, Heizungsunterstuetzung, Kombi
    coverage_dhw_percent DECIMAL(5,2),
    coverage_heating_percent DECIMAL(5,2),

    -- Ertrag (berechnet)
    annual_yield_kwh DECIMAL(10,2),

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Photovoltaik
CREATE TABLE pv_systems (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    building_version_id UUID NOT NULL REFERENCES building_versions(id) ON DELETE CASCADE,

    -- Module
    module_type VARCHAR(30),  -- mono, poly, duennschicht
    peak_power_kwp DECIMAL(8,2) NOT NULL,
    module_area_m2 DECIMAL(8,2),
    module_orientation VARCHAR(10),
    module_tilt_deg DECIMAL(5,2),

    -- System
    inverter_power_kw DECIMAL(8,2),
    inverter_efficiency DECIMAL(4,3),

    -- Nutzung
    self_consumption_percent DECIMAL(5,2),  -- Eigenverbrauchsanteil
    feed_in_percent DECIMAL(5,2),  -- Einspeisung

    -- Batteriespeicher
    battery_present BOOLEAN DEFAULT FALSE,
    battery_capacity_kwh DECIMAL(8,2),
    battery_power_kw DECIMAL(8,2),

    -- Ertrag (berechnet)
    annual_yield_kwh DECIMAL(10,2),
    self_consumed_kwh DECIMAL(10,2),

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_solar_thermal_version ON solar_thermal_systems(building_version_id);
CREATE INDEX idx_pv_version ON pv_systems(building_version_id);
```

### 12. Berechnungsergebnisse

```sql
-- Energiebilanz-Ergebnisse
CREATE TABLE energy_calculations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    building_version_id UUID NOT NULL REFERENCES building_versions(id) ON DELETE CASCADE,

    -- Berechnungsinfo
    calculation_type VARCHAR(30) NOT NULL,  -- Energieausweis, iSFP, Heizlast
    calculation_method VARCHAR(30) NOT NULL,  -- DIN_V_18599, DIN_4108_6, etc.
    calculation_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    calculated_by UUID REFERENCES users(id),

    -- Status
    status VARCHAR(20) DEFAULT 'draft',  -- draft, approved, archived
    approved_at TIMESTAMP,
    approved_by UUID REFERENCES users(id),

    -- Wärmeverluste
    transmission_loss_kwh_a DECIMAL(12,2),
    ventilation_loss_kwh_a DECIMAL(12,2),

    -- Gewinne
    solar_gains_kwh_a DECIMAL(12,2),
    internal_gains_kwh_a DECIMAL(12,2),

    -- Nutzenergie
    heating_demand_kwh_a DECIMAL(12,2),  -- Heizwärmebedarf
    cooling_demand_kwh_a DECIMAL(12,2),  -- Kühlbedarf
    dhw_demand_kwh_a DECIMAL(12,2),  -- TWW-Bedarf
    lighting_demand_kwh_a DECIMAL(12,2),  -- Beleuchtung (NWG)

    -- Endenergie (nach Energieträger)
    final_energy_total_kwh_a DECIMAL(12,2),
    final_energy_gas_kwh_a DECIMAL(12,2),
    final_energy_oil_kwh_a DECIMAL(12,2),
    final_energy_electricity_kwh_a DECIMAL(12,2),
    final_energy_biomass_kwh_a DECIMAL(12,2),
    final_energy_district_kwh_a DECIMAL(12,2),
    final_energy_other_kwh_a DECIMAL(12,2),

    -- Primärenergie
    primary_energy_total_kwh_a DECIMAL(12,2),
    primary_energy_non_renewable_kwh_a DECIMAL(12,2),
    primary_energy_renewable_kwh_a DECIMAL(12,2),

    -- Spezifische Werte (pro m² AN)
    final_energy_specific DECIMAL(8,2),  -- kWh/(m²·a)
    primary_energy_specific DECIMAL(8,2),  -- kWh/(m²·a)

    -- CO2-Emissionen
    co2_emissions_kg_a DECIMAL(12,2),
    co2_emissions_specific DECIMAL(8,2),  -- kg/(m²·a)

    -- Effizienzklasse
    efficiency_class CHAR(2),  -- A+, A, B, C, D, E, F, G, H

    -- GEG-Anforderungen (Vergleich)
    geg_requirement_qp DECIMAL(8,2),  -- Anforderungswert QP
    geg_requirement_ht DECIMAL(6,3),  -- Anforderungswert H'T
    geg_fulfilled BOOLEAN,

    -- KfW/BEG (wenn relevant)
    effizienzhaus_level INTEGER,  -- 40, 55, 70, 85
    effizienzhaus_ee BOOLEAN,  -- Erneuerbare Energien
    effizienzhaus_nh BOOLEAN,  -- Nachhaltigkeit

    -- Transmissionswärmeverlust
    ht_value DECIMAL(6,3),  -- H'T in W/(m²K)

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_calculations_version ON energy_calculations(building_version_id);
CREATE INDEX idx_calculations_type ON energy_calculations(calculation_type);

-- Heizlast-Ergebnisse (Zusammenfassung)
CREATE TABLE heating_load_results (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    building_version_id UUID NOT NULL REFERENCES building_versions(id) ON DELETE CASCADE,
    calculation_id UUID REFERENCES energy_calculations(id),

    -- Gesamtergebnis
    building_heating_load_w DECIMAL(12,1),  -- ΦHL
    dhw_heating_load_w DECIMAL(12,1),
    total_load_w DECIMAL(12,1),

    -- Spezifisch
    specific_heating_load_w_m2 DECIMAL(8,2),

    -- Norm-Außentemperatur
    design_outdoor_temp_c DECIMAL(5,2),

    -- Aufteilung
    transmission_load_w DECIMAL(12,1),
    ventilation_load_w DECIMAL(12,1),

    -- Bei MFH: Gleichzeitigkeitsfaktor
    simultaneity_factor DECIMAL(4,3),

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 13. Dokumente und Dateien

```sql
-- Generierte Dokumente
CREATE TABLE documents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
    building_version_id UUID REFERENCES building_versions(id),
    calculation_id UUID REFERENCES energy_calculations(id),

    -- Dokumentinfo
    document_type VARCHAR(50) NOT NULL,
    -- Energieausweis_Bedarf, Energieausweis_Verbrauch, iSFP_Kurzbericht,
    -- iSFP_Umsetzungshilfe, Heizlastberechnung, GEG_Nachweis, etc.

    document_number VARCHAR(50),  -- z.B. DIBt-Registrierungsnummer
    title VARCHAR(255) NOT NULL,
    description TEXT,

    -- Version
    version INTEGER DEFAULT 1,
    status VARCHAR(20) DEFAULT 'draft',  -- draft, final, archived

    -- Dateien
    file_path VARCHAR(500),
    file_size_bytes BIGINT,
    file_type VARCHAR(20),  -- pdf, docx, xlsx

    -- Gültigkeit (z.B. Energieausweis)
    valid_from DATE,
    valid_until DATE,

    -- Digitale Signatur
    is_signed BOOLEAN DEFAULT FALSE,
    signed_at TIMESTAMP,
    signed_by UUID REFERENCES users(id),
    signature_data JSONB,

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID REFERENCES users(id)
);

CREATE INDEX idx_documents_project ON documents(project_id);
CREATE INDEX idx_documents_type ON documents(document_type);

-- Anhänge und Fotos
CREATE TABLE attachments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    building_id UUID REFERENCES buildings(id),
    building_version_id UUID REFERENCES building_versions(id),
    component_id UUID,  -- Referenz auf Bauteil, Anlage, etc.
    component_type VARCHAR(50),  -- Welche Tabelle

    -- Dateiinfo
    file_name VARCHAR(255) NOT NULL,
    file_path VARCHAR(500) NOT NULL,
    file_size_bytes BIGINT,
    mime_type VARCHAR(100),

    -- Metadaten
    category VARCHAR(50),  -- Foto, Plan, Protokoll, Rechnung, etc.
    description TEXT,
    tags TEXT[],

    -- Foto-spezifisch
    taken_at TIMESTAMP,
    location_lat DECIMAL(10, 8),
    location_lng DECIMAL(11, 8),

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID REFERENCES users(id)
);

CREATE INDEX idx_attachments_building ON attachments(building_id);
```

---

## Materialdatenbank

```sql
-- Vorgegebene Materialdaten (aus DIN 4108-4, ÖNORM, etc.)
CREATE TABLE materials (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    -- Identifikation
    code VARCHAR(20) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL,
    name_en VARCHAR(100),
    category VARCHAR(50) NOT NULL,
    -- Kategorien: Mauerwerk, Beton, Holz, Daemmung, Putz, etc.

    subcategory VARCHAR(50),

    -- Wärmetechnische Eigenschaften
    lambda_design DECIMAL(6,4) NOT NULL,  -- Bemessungswert λ [W/(mK)]
    lambda_declared DECIMAL(6,4),  -- Nennwert λD [W/(mK)]

    -- Feuchtetechnische Eigenschaften
    mu_value_dry DECIMAL(8,2),  -- μ (trocken)
    mu_value_wet DECIMAL(8,2),  -- μ (feucht)
    mu_value_design DECIMAL(8,2),  -- Bemessungs-μ

    -- Speichereigenschaften
    density_kg_m3 DECIMAL(8,2),  -- Rohdichte [kg/m³]
    specific_heat_j_kgk INTEGER,  -- spez. Wärmekapazität [J/(kg·K)]

    -- Brandeigenschaften
    fire_class VARCHAR(20),  -- A1, A2, B, C, D, E, F

    -- Quelle
    source VARCHAR(100),  -- z.B. "DIN 4108-4:2020 Tabelle 1"
    source_year INTEGER,

    -- Status
    is_active BOOLEAN DEFAULT TRUE,

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_materials_category ON materials(category);
CREATE INDEX idx_materials_name ON materials(name);

-- Beispieldaten aus DIN 4108-4
INSERT INTO materials (code, name, category, lambda_design, mu_value_dry, density_kg_m3) VALUES
('MW-001', 'Vollziegel (1800)', 'Mauerwerk', 0.81, 10, 1800),
('MW-002', 'Hochlochziegel (1200)', 'Mauerwerk', 0.50, 10, 1200),
('MW-003', 'Kalksandstein (1800)', 'Mauerwerk', 0.99, 15, 1800),
('MW-004', 'Porenbeton (500)', 'Mauerwerk', 0.13, 10, 500),
('BE-001', 'Normalbeton', 'Beton', 2.10, 80, 2400),
('BE-002', 'Stahlbeton', 'Beton', 2.30, 80, 2500),
('DA-001', 'EPS (035)', 'Daemmung', 0.035, 30, 15),
('DA-002', 'Mineralwolle (035)', 'Daemmung', 0.035, 1, 30),
('DA-003', 'PUR/PIR (024)', 'Daemmung', 0.024, 50, 30),
('DA-004', 'Holzfaser (042)', 'Daemmung', 0.042, 5, 160),
('DA-005', 'Zellulose (040)', 'Daemmung', 0.040, 2, 50),
-- ... weitere Materialien
;
```

---

## Views für häufige Abfragen

```sql
-- Gebäudeübersicht mit aktuellen Werten
CREATE VIEW v_building_summary AS
SELECT
    b.id,
    b.street || ' ' || b.house_number AS address,
    b.postal_code,
    b.city,
    c.name AS customer_name,
    bv.building_type,
    bv.construction_year,
    bv.usable_area_an_m2,
    ec.final_energy_specific,
    ec.primary_energy_specific,
    ec.efficiency_class,
    ec.co2_emissions_specific
FROM buildings b
JOIN customers c ON b.customer_id = c.id
LEFT JOIN building_versions bv ON b.id = bv.building_id
    AND bv.version_type = 'ist'
    AND bv.version_number = (
        SELECT MAX(version_number)
        FROM building_versions
        WHERE building_id = b.id AND version_type = 'ist'
    )
LEFT JOIN energy_calculations ec ON bv.id = ec.building_version_id
    AND ec.status = 'approved';

-- Projektfortschritt
CREATE VIEW v_project_progress AS
SELECT
    p.id AS project_id,
    p.name AS project_name,
    p.project_type,
    p.status,
    b.street || ' ' || b.house_number AS building_address,
    c.name AS customer_name,
    COUNT(DISTINCT t.id) FILTER (WHERE t.status = 'completed') AS tasks_completed,
    COUNT(DISTINCT t.id) AS tasks_total,
    ROUND(
        COUNT(DISTINCT t.id) FILTER (WHERE t.status = 'completed')::numeric /
        NULLIF(COUNT(DISTINCT t.id), 0) * 100, 0
    ) AS progress_percent
FROM projects p
JOIN buildings b ON p.building_id = b.id
JOIN customers c ON b.customer_id = c.id
LEFT JOIN tasks t ON p.id = t.project_id
GROUP BY p.id, p.name, p.project_type, p.status, b.street, b.house_number, c.name;
```
