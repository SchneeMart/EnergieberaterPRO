# Datenmodell - Vollständige Spezifikation

## Terminologie-Hinweis

**⚠️ WICHTIG:**
- **Organisationen** = Kunden der Plattform (Energieberatungsbüros) → HABEN LOGIN
- **Customers (Endkunden)** = Kunden der Organisationen (Hausbesitzer, Firmen) → KEIN LOGIN

**Referenzen:**
- `03_FRONTEND/TERMINOLOGIE_KORREKTUR.md` - Detaillierte Terminologie
- `04_BERECHNUNGEN/DATENMODELL_GEBAEUDE.md` - Detailliertes Gebäude-Schema mit Versionierung

---

## 1. Übersicht

### 1.1 Entity-Relationship-Diagramm (Konzept)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              BENUTZERVERWALTUNG                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐                │
│  │ Organization │────▶│     User     │◀────│     Role     │                │
│  │              │     │              │     │              │                │
│  └──────────────┘     └──────────────┘     └──────────────┘                │
│         │                    │                    │                         │
│         │                    │                    │                         │
│         ▼                    ▼                    ▼                         │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐                │
│  │   Settings   │     │   Profile    │     │  Permission  │                │
│  └──────────────┘     └──────────────┘     └──────────────┘                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                              PROJEKTVERWALTUNG                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐                │
│  │   Customer   │────▶│   Project    │◀────│   Location   │                │
│  │              │     │              │     │              │                │
│  └──────────────┘     └──────────────┘     └──────────────┘                │
│                              │                    │                         │
│         ┌────────────────────┼────────────────────┤                         │
│         ▼                    ▼                    ▼                         │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐                │
│  │   Document   │     │   Building   │     │  ClimateData │                │
│  └──────────────┘     └──────────────┘     └──────────────┘                │
│                              │                                              │
│         ┌────────────────────┼────────────────────┐                         │
│         ▼                    ▼                    ▼                         │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐                │
│  │     Zone     │     │  Component   │     │    System    │                │
│  └──────────────┘     └──────────────┘     └──────────────┘                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                              BERECHNUNGEN                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐                │
│  │ Calculation  │────▶│    Result    │────▶│    Report    │                │
│  │              │     │              │     │              │                │
│  └──────────────┘     └──────────────┘     └──────────────┘                │
│         │                                                                   │
│         ▼                                                                   │
│  ┌──────────────────────────────────────────────────────────┐              │
│  │                    Calculation Types                      │              │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐     │              │
│  │  │ UValue  │  │HeatLoad │  │ Energy  │  │   CO2   │     │              │
│  │  │  Calc   │  │  Calc   │  │ Balance │  │ Balance │     │              │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘     │              │
│  └──────────────────────────────────────────────────────────┘              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                              GESCHÄFTSPROZESSE                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐                │
│  │    Quote     │────▶│   Invoice    │────▶│   Payment    │                │
│  │  (Angebot)   │     │ (Rechnung)   │     │  (Zahlung)   │                │
│  └──────────────┘     └──────────────┘     └──────────────┘                │
│         │                    │                                              │
│         ▼                    ▼                                              │
│  ┌──────────────┐     ┌──────────────┐                                     │
│  │  QuoteItem   │     │ InvoiceItem  │                                     │
│  └──────────────┘     └──────────────┘                                     │
│                                                                             │
│  ┌──────────────┐     ┌──────────────┐                                     │
│  │  TimeEntry   │────▶│   Expense    │                                     │
│  │(Zeiterfassg.)│     │   (Kosten)   │                                     │
│  └──────────────┘     └──────────────┘                                     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Tabellendefinitionen

### 2.1 Benutzerverwaltung

#### organizations (Kunden der Plattform - HABEN LOGIN)
```sql
CREATE TABLE organizations (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    logo_url VARCHAR(500),
    address JSONB,
    contact JSONB,
    settings JSONB DEFAULT '{}',

    -- Länderunterstützung (DE/AT)
    country VARCHAR(2) DEFAULT 'DE' CHECK (country IN ('DE', 'AT')),
    -- DE = Deutschland (DIN, GEG)
    -- AT = Österreich (ÖNORM, OIB)

    subscription_plan VARCHAR(50) DEFAULT 'free',
    subscription_valid_until TIMESTAMP WITH TIME ZONE,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Indizes
CREATE INDEX idx_organizations_slug ON organizations(slug);
CREATE INDEX idx_organizations_active ON organizations(is_active);
CREATE INDEX idx_organizations_country ON organizations(country);
```

#### users
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    organization_id UUID REFERENCES organizations(id) ON DELETE SET NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    phone VARCHAR(50),
    avatar_url VARCHAR(500),
    language VARCHAR(10) DEFAULT 'de',
    timezone VARCHAR(50) DEFAULT 'Europe/Berlin',
    is_active BOOLEAN DEFAULT TRUE,
    is_verified BOOLEAN DEFAULT FALSE,
    is_superuser BOOLEAN DEFAULT FALSE,
    last_login TIMESTAMP WITH TIME ZONE,
    failed_login_attempts INTEGER DEFAULT 0,
    locked_until TIMESTAMP WITH TIME ZONE,
    mfa_enabled BOOLEAN DEFAULT FALSE,
    mfa_secret VARCHAR(255),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Indizes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_organization ON users(organization_id);
CREATE INDEX idx_users_active ON users(is_active);
```

#### roles
```sql
CREATE TABLE roles (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    permissions JSONB DEFAULT '[]',
    is_system_role BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,

    UNIQUE(organization_id, name)
);

-- Vordefinierte Rollen
INSERT INTO roles (name, description, permissions, is_system_role) VALUES
('admin', 'Vollzugriff', '["*"]', TRUE),
('manager', 'Projektverwaltung', '["projects.*", "reports.*", "calculations.*"]', TRUE),
('consultant', 'Berater', '["projects.read", "projects.write", "calculations.*", "reports.read"]', TRUE),
('viewer', 'Nur Lesen', '["projects.read", "reports.read"]', TRUE);
```

#### user_roles
```sql
CREATE TABLE user_roles (
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    role_id UUID REFERENCES roles(id) ON DELETE CASCADE,
    assigned_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    assigned_by UUID REFERENCES users(id),

    PRIMARY KEY (user_id, role_id)
);
```

#### sessions
```sql
CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    token_hash VARCHAR(255) NOT NULL,
    ip_address INET,
    user_agent TEXT,
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    last_used_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_sessions_user ON sessions(user_id);
CREATE INDEX idx_sessions_token ON sessions(token_hash);
CREATE INDEX idx_sessions_expires ON sessions(expires_at);
```

---

### 2.2 Projektverwaltung

#### customers (Endkunden der Organisationen - ⚠️ KEIN LOGIN!)

**WICHTIG:** Diese Tabelle enthält die Endkunden der Organisationen (Hausbesitzer, Firmen, WEGs).
Endkunden haben **KEINEN Login** auf der Plattform - sie werden nur als Datensätze erfasst.

```sql
CREATE TABLE customers (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
    customer_number VARCHAR(50),
    customer_type VARCHAR(20) CHECK (customer_type IN ('private', 'business', 'public', 'weg')),
    -- private = Privatkunde (Hausbesitzer)
    -- business = Geschäftskunde (Firma)
    -- public = Öffentliche Hand
    -- weg = Wohnungseigentümergemeinschaft

    -- Privatkunden
    salutation VARCHAR(20),
    first_name VARCHAR(100),
    last_name VARCHAR(100),

    -- Geschäftskunden
    company_name VARCHAR(255),
    vat_id VARCHAR(50),

    -- Kontakt
    email VARCHAR(255),
    phone VARCHAR(50),
    mobile VARCHAR(50),

    -- Adresse
    street VARCHAR(255),
    house_number VARCHAR(20),
    postal_code VARCHAR(20),
    city VARCHAR(100),
    country VARCHAR(100) DEFAULT 'Deutschland',

    -- Zusätzlich
    notes TEXT,
    tags JSONB DEFAULT '[]',

    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,

    UNIQUE(organization_id, customer_number)
);

CREATE INDEX idx_customers_organization ON customers(organization_id);
CREATE INDEX idx_customers_search ON customers USING gin(
    to_tsvector('german', coalesce(first_name, '') || ' ' ||
                          coalesce(last_name, '') || ' ' ||
                          coalesce(company_name, ''))
);
```

#### projects
```sql
CREATE TABLE projects (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
    customer_id UUID REFERENCES customers(id) ON DELETE SET NULL,
    created_by UUID REFERENCES users(id),

    project_number VARCHAR(50),
    name VARCHAR(255) NOT NULL,
    description TEXT,

    project_type VARCHAR(50) CHECK (project_type IN (
        'energy_audit',           -- Energieaudit
        'energy_certificate',     -- Energieausweis
        'renovation_concept',     -- Sanierungskonzept
        'heat_load_calc',         -- Heizlastberechnung
        'feasibility_study',      -- Machbarkeitsstudie
        'monitoring',             -- Energiemonitoring
        'consulting',             -- Beratung
        'other'                   -- Sonstiges
    )),

    status VARCHAR(30) CHECK (status IN (
        'draft',        -- Entwurf
        'active',       -- Aktiv
        'on_hold',      -- Pausiert
        'completed',    -- Abgeschlossen
        'cancelled',    -- Abgebrochen
        'archived'      -- Archiviert
    )) DEFAULT 'draft',

    priority VARCHAR(20) CHECK (priority IN ('low', 'medium', 'high', 'urgent')),

    -- Zeitplanung
    start_date DATE,
    due_date DATE,
    completed_date DATE,

    -- Budget
    budget_hours DECIMAL(10, 2),
    budget_amount DECIMAL(12, 2),
    billing_type VARCHAR(30) CHECK (billing_type IN (
        'fixed_price',      -- Festpreis
        'hourly',           -- Stundenbasis
        'daily',            -- Tagessatz
        'milestone',        -- Meilensteine
        'subscription'      -- Abo
    )),
    hourly_rate DECIMAL(10, 2),

    -- Metadaten
    tags JSONB DEFAULT '[]',
    custom_fields JSONB DEFAULT '{}',

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,

    UNIQUE(organization_id, project_number)
);

CREATE INDEX idx_projects_organization ON projects(organization_id);
CREATE INDEX idx_projects_customer ON projects(customer_id);
CREATE INDEX idx_projects_status ON projects(status);
CREATE INDEX idx_projects_type ON projects(project_type);
```

#### locations
```sql
CREATE TABLE locations (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    project_id UUID REFERENCES projects(id) ON DELETE CASCADE,

    name VARCHAR(255) NOT NULL,
    description TEXT,

    -- Adresse
    street VARCHAR(255),
    house_number VARCHAR(20),
    postal_code VARCHAR(20) NOT NULL,
    city VARCHAR(100) NOT NULL,
    state VARCHAR(100),
    country VARCHAR(100) DEFAULT 'Deutschland',

    -- Geokoordinaten
    latitude DECIMAL(10, 8),
    longitude DECIMAL(11, 8),
    altitude DECIMAL(8, 2),  -- Höhe über NN in Metern

    -- Klimadaten
    climate_zone VARCHAR(20),          -- Klimazone nach DIN
    climate_data_source VARCHAR(100),  -- Quelle der Klimadaten
    heating_degree_days INTEGER,       -- Heizgradtage
    cooling_degree_days INTEGER,       -- Kühlgradtage

    -- Umgebung
    urban_context VARCHAR(30) CHECK (urban_context IN (
        'rural',        -- Ländlich
        'suburban',     -- Vorstadt
        'urban',        -- Städtisch
        'city_center'   -- Stadtzentrum
    )),
    terrain_type VARCHAR(30),
    shading_objects JSONB DEFAULT '[]',

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_locations_project ON locations(project_id);
CREATE INDEX idx_locations_postal ON locations(postal_code);
```

---

### 2.3 Gebäudedaten

#### buildings

**Hinweis:** Für detailliertes Schema mit Gebäude-Versionierung (Ist-Zustand, Sanierungsvarianten)
siehe `04_BERECHNUNGEN/DATENMODELL_GEBAEUDE.md`.

```sql
CREATE TABLE buildings (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    location_id UUID REFERENCES locations(id) ON DELETE CASCADE,

    name VARCHAR(255) NOT NULL,
    description TEXT,

    -- Gebäudetyp
    building_type VARCHAR(50) CHECK (building_type IN (
        'single_family',        -- Einfamilienhaus
        'two_family',           -- Zweifamilienhaus
        'multi_family',         -- Mehrfamilienhaus
        'apartment_block',      -- Wohnblock
        'terraced',             -- Reihenhaus
        'office',               -- Bürogebäude
        'retail',               -- Einzelhandel
        'industrial',           -- Industriegebäude
        'warehouse',            -- Lager
        'school',               -- Schule
        'hospital',             -- Krankenhaus
        'hotel',                -- Hotel
        'mixed_use',            -- Mischnutzung
        'other'                 -- Sonstiges
    )),

    -- Baujahr & Sanierung
    construction_year INTEGER,
    last_renovation_year INTEGER,
    heritage_protected BOOLEAN DEFAULT FALSE,

    -- Dimensionen
    gross_floor_area DECIMAL(12, 2),      -- Bruttogeschossfläche [m²]
    net_floor_area DECIMAL(12, 2),        -- Nettogeschossfläche [m²]
    heated_area DECIMAL(12, 2),           -- Beheizte Fläche [m²]
    cooled_area DECIMAL(12, 2),           -- Gekühlte Fläche [m²]
    building_volume DECIMAL(14, 2),       -- Gebäudevolumen [m³]
    heated_volume DECIMAL(14, 2),         -- Beheiztes Volumen [m³]
    envelope_area DECIMAL(12, 2),         -- Hüllfläche [m²]

    -- Geschosse
    floors_above_ground INTEGER,
    floors_below_ground INTEGER,
    floor_height DECIMAL(5, 2),           -- Geschosshöhe [m]

    -- Kompaktheit
    a_v_ratio DECIMAL(6, 4),              -- A/V-Verhältnis [1/m]
    form_factor DECIMAL(6, 4),

    -- Ausrichtung
    orientation DECIMAL(5, 2),            -- Hauptausrichtung in Grad (0=Nord)

    -- Nutzungsprofil
    occupancy_hours_weekday INTEGER,
    occupancy_hours_weekend INTEGER,
    occupants_count INTEGER,

    -- Status
    status VARCHAR(30) CHECK (status IN ('planned', 'existing', 'demolition')),

    -- Zusätzliche Daten
    photos JSONB DEFAULT '[]',
    floor_plans JSONB DEFAULT '[]',
    custom_data JSONB DEFAULT '{}',

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_buildings_location ON buildings(location_id);
CREATE INDEX idx_buildings_type ON buildings(building_type);
```

#### zones
```sql
CREATE TABLE zones (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    building_id UUID REFERENCES buildings(id) ON DELETE CASCADE,
    parent_zone_id UUID REFERENCES zones(id) ON DELETE SET NULL,

    name VARCHAR(255) NOT NULL,
    zone_number VARCHAR(20),
    description TEXT,

    -- Zonierung nach DIN V 18599
    zone_type VARCHAR(50) CHECK (zone_type IN (
        'residential',              -- Wohnbereich
        'office',                   -- Büro
        'conference',               -- Besprechungsraum
        'lobby',                    -- Foyer/Empfang
        'circulation',              -- Verkehrsfläche
        'sanitary',                 -- Sanitär
        'kitchen_residential',      -- Küche (Wohnen)
        'kitchen_commercial',       -- Küche (Gewerbe)
        'restaurant',               -- Restaurant
        'retail',                   -- Verkauf
        'warehouse',                -- Lager
        'production',               -- Produktion
        'server_room',              -- Serverraum
        'laboratory',               -- Labor
        'classroom',                -- Klassenzimmer
        'auditorium',               -- Hörsaal
        'sports',                   -- Sporthalle
        'swimming_pool',            -- Schwimmbad
        'hospital_room',            -- Patientenzimmer
        'operating_room',           -- OP
        'hotel_room',               -- Hotelzimmer
        'other'                     -- Sonstiges
    )),

    -- Nutzungsprofil
    usage_profile VARCHAR(50),       -- Nutzungsprofil-Code nach DIN V 18599-10

    -- Flächen
    floor_area DECIMAL(10, 2),       -- Zonenfläche [m²]
    ceiling_height DECIMAL(5, 2),    -- Raumhöhe [m]
    volume DECIMAL(12, 2),           -- Zonenvolumen [m³]

    -- Thermische Eigenschaften
    target_temp_heating DECIMAL(4, 1) DEFAULT 20.0,   -- Solltemperatur Heizen [°C]
    target_temp_cooling DECIMAL(4, 1) DEFAULT 26.0,   -- Solltemperatur Kühlen [°C]
    min_air_change DECIMAL(5, 2),                     -- Mindestluftwechsel [1/h]
    internal_gains_persons DECIMAL(8, 2),             -- Wärmeabgabe Personen [W]
    internal_gains_equipment DECIMAL(8, 2),           -- Wärmeabgabe Geräte [W]
    internal_gains_lighting DECIMAL(8, 2),            -- Wärmeabgabe Beleuchtung [W]

    -- Betriebszeiten
    operating_hours JSONB DEFAULT '{}',  -- {monday: {start: "08:00", end: "18:00"}, ...}
    occupancy JSONB DEFAULT '{}',        -- Belegungsprofil

    -- Zuordnungen
    floor_number INTEGER,

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_zones_building ON zones(building_id);
CREATE INDEX idx_zones_type ON zones(zone_type);
```

#### building_components (Bauteile)
```sql
CREATE TABLE building_components (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    building_id UUID REFERENCES buildings(id) ON DELETE CASCADE,
    zone_id UUID REFERENCES zones(id) ON DELETE SET NULL,

    name VARCHAR(255) NOT NULL,
    component_code VARCHAR(50),

    -- Bauteiltyp
    component_type VARCHAR(50) CHECK (component_type IN (
        'exterior_wall',           -- Außenwand
        'interior_wall',           -- Innenwand
        'roof',                    -- Dach
        'ceiling',                 -- Decke
        'floor_ground',            -- Bodenplatte/Kellerdecke
        'floor_unheated',          -- Decke zu unbeheizt
        'floor_exterior',          -- Decke zu Außenluft
        'window',                  -- Fenster
        'door_exterior',           -- Außentür
        'door_interior',           -- Innentür
        'skylight',                -- Dachfenster
        'curtain_wall',            -- Vorhangfassade
        'thermal_bridge'           -- Wärmebrücke
    )),

    -- Geometrie
    area DECIMAL(10, 2),                 -- Fläche [m²]
    length DECIMAL(8, 2),                -- Länge [m] (für Wärmebrücken)
    height DECIMAL(6, 2),                -- Höhe [m]
    width DECIMAL(6, 2),                 -- Breite [m]
    thickness DECIMAL(6, 4),             -- Dicke [m]

    -- Ausrichtung
    orientation VARCHAR(20) CHECK (orientation IN (
        'north', 'north_east', 'east', 'south_east',
        'south', 'south_west', 'west', 'north_west',
        'horizontal_up', 'horizontal_down'
    )),
    tilt_angle DECIMAL(5, 2),            -- Neigung [°] (0=horizontal, 90=vertikal)

    -- Grenzt an
    adjacent_to VARCHAR(30) CHECK (adjacent_to IN (
        'exterior',          -- Außenluft
        'ground',            -- Erdreich
        'unheated',          -- Unbeheizt
        'heated',            -- Beheizt (andere Zone)
        'neighbor'           -- Nachbargebäude
    )),
    adjacent_zone_id UUID REFERENCES zones(id),

    -- Thermische Eigenschaften
    u_value DECIMAL(6, 4),               -- U-Wert [W/(m²K)]
    u_value_calculated BOOLEAN DEFAULT FALSE,
    r_value DECIMAL(8, 4),               -- R-Wert [m²K/W]

    -- Für Fenster
    g_value DECIMAL(4, 3),               -- Gesamtenergiedurchlassgrad [-]
    frame_fraction DECIMAL(4, 3),        -- Rahmenanteil [-]
    shading_factor DECIMAL(4, 3),        -- Verschattungsfaktor [-]

    -- Für Wärmebrücken
    psi_value DECIMAL(6, 4),             -- Längenbezogener Wärmebrückenverlust [W/(mK)]
    chi_value DECIMAL(6, 4),             -- Punktförmiger Wärmebrückenverlust [W/K]

    -- Konstruktion (Schichten)
    layers JSONB DEFAULT '[]',           -- [{material_id, thickness, lambda, ...}, ...]

    -- Qualität
    condition VARCHAR(20) CHECK (condition IN ('good', 'moderate', 'poor', 'critical')),
    year_of_installation INTEGER,
    notes TEXT,

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_components_building ON building_components(building_id);
CREATE INDEX idx_components_zone ON building_components(zone_id);
CREATE INDEX idx_components_type ON building_components(component_type);
```

#### building_systems (Anlagentechnik)
```sql
CREATE TABLE building_systems (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    building_id UUID REFERENCES buildings(id) ON DELETE CASCADE,

    name VARCHAR(255) NOT NULL,
    system_code VARCHAR(50),

    -- Systemtyp
    system_type VARCHAR(50) CHECK (system_type IN (
        -- Heizung
        'heating_boiler_gas',           -- Gas-Heizkessel
        'heating_boiler_oil',           -- Öl-Heizkessel
        'heating_boiler_biomass',       -- Biomasse-Kessel
        'heating_heat_pump_air',        -- Luft-Wärmepumpe
        'heating_heat_pump_ground',     -- Erdwärmepumpe
        'heating_heat_pump_water',      -- Wasser-Wärmepumpe
        'heating_district',             -- Fernwärme
        'heating_electric',             -- Elektroheizung
        'heating_chp',                  -- BHKW

        -- Warmwasser
        'dhw_central',                  -- Zentrale WW-Bereitung
        'dhw_decentral',                -- Dezentrale WW-Bereitung
        'dhw_solar',                    -- Solarthermie WW
        'dhw_heat_pump',                -- WW-Wärmepumpe

        -- Kühlung
        'cooling_chiller',              -- Kältemaschine
        'cooling_split',                -- Split-Klimaanlage
        'cooling_vrv',                  -- VRV/VRF-System
        'cooling_absorption',           -- Absorptionskälte
        'cooling_free',                 -- Freie Kühlung

        -- Lüftung
        'ventilation_exhaust',          -- Abluftanlage
        'ventilation_supply_exhaust',   -- Zu-/Abluftanlage
        'ventilation_hrv',              -- Mit Wärmerückgewinnung

        -- Beleuchtung
        'lighting_standard',            -- Standardbeleuchtung
        'lighting_led',                 -- LED-Beleuchtung
        'lighting_daylight',            -- Tageslichtnutzung

        -- Erneuerbare
        'renewable_pv',                 -- Photovoltaik
        'renewable_solar_thermal',      -- Solarthermie
        'renewable_wind',               -- Kleinwindanlage

        -- Speicher
        'storage_battery',              -- Batteriespeicher
        'storage_thermal',              -- Wärmespeicher

        -- Sonstiges
        'other'
    )),

    -- Allgemeine Eigenschaften
    manufacturer VARCHAR(255),
    model VARCHAR(255),
    installation_year INTEGER,
    expected_lifetime INTEGER,          -- Erwartete Lebensdauer [Jahre]

    -- Leistungsdaten
    nominal_power DECIMAL(12, 2),       -- Nennleistung [kW]
    efficiency DECIMAL(5, 3),           -- Wirkungsgrad/COP [-]
    efficiency_part_load DECIMAL(5, 3), -- Teillastwirkungsgrad [-]
    capacity DECIMAL(12, 2),            -- Speicherkapazität [kWh]

    -- Für Heizung
    flow_temperature DECIMAL(5, 1),     -- Vorlauftemperatur [°C]
    return_temperature DECIMAL(5, 1),   -- Rücklauftemperatur [°C]
    fuel_type VARCHAR(50),              -- Brennstoffart
    fuel_consumption DECIMAL(12, 2),    -- Brennstoffverbrauch [kWh/a]

    -- Für Lüftung
    air_flow_rate DECIMAL(10, 2),       -- Volumenstrom [m³/h]
    heat_recovery_rate DECIMAL(5, 3),   -- Wärmerückgewinnungsgrad [-]
    sfp DECIMAL(6, 2),                  -- Spezifische Ventilatorleistung [W/(m³/h)]

    -- Für PV
    peak_power DECIMAL(10, 2),          -- Spitzenleistung [kWp]
    module_area DECIMAL(10, 2),         -- Modulfläche [m²]
    orientation_pv DECIMAL(5, 2),       -- Ausrichtung [°]
    tilt_pv DECIMAL(5, 2),              -- Neigung [°]

    -- Für Speicher
    usable_capacity DECIMAL(10, 2),     -- Nutzbare Kapazität [kWh]
    charge_efficiency DECIMAL(5, 3),    -- Ladeeffizienz [-]
    discharge_efficiency DECIMAL(5, 3), -- Entladeeffizienz [-]

    -- Versorgungsbereiche
    served_zones JSONB DEFAULT '[]',    -- [zone_id, ...]
    coverage_percentage DECIMAL(5, 2),  -- Deckungsanteil [%]

    -- Zustand
    condition VARCHAR(20) CHECK (condition IN ('new', 'good', 'moderate', 'poor', 'critical')),
    last_maintenance DATE,
    next_maintenance DATE,
    notes TEXT,

    -- Wirtschaftlichkeit
    investment_cost DECIMAL(12, 2),
    annual_cost DECIMAL(10, 2),

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_systems_building ON building_systems(building_id);
CREATE INDEX idx_systems_type ON building_systems(system_type);
```

---

### 2.4 Berechnungen

#### calculations
```sql
CREATE TABLE calculations (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
    building_id UUID REFERENCES buildings(id) ON DELETE CASCADE,
    created_by UUID REFERENCES users(id),

    calculation_type VARCHAR(50) CHECK (calculation_type IN (
        'u_value',              -- U-Wert-Berechnung
        'heat_load',            -- Heizlastberechnung DIN EN 12831
        'cooling_load',         -- Kühllastberechnung VDI 2078
        'energy_balance',       -- Energiebilanz DIN V 18599
        'primary_energy',       -- Primärenergiebedarf
        'co2_emissions',        -- CO2-Bilanzierung
        'economic',             -- Wirtschaftlichkeitsberechnung
        'renewable_share',      -- Erneuerbare-Energien-Anteil
        'lca',                  -- Lebenszyklusanalyse
        'custom'                -- Benutzerdefiniert
    )) NOT NULL,

    name VARCHAR(255) NOT NULL,
    description TEXT,
    version INTEGER DEFAULT 1,

    -- Berechnungsgrundlage
    standard VARCHAR(100),              -- Verwendete Norm
    standard_version VARCHAR(50),       -- Normversion
    calculation_method VARCHAR(100),    -- Berechnungsverfahren

    -- Eingabedaten (strukturiert)
    input_data JSONB NOT NULL,

    -- Ergebnisse
    results JSONB,
    summary JSONB,

    -- Validierung
    is_valid BOOLEAN,
    validation_errors JSONB DEFAULT '[]',
    validation_warnings JSONB DEFAULT '[]',

    -- Status
    status VARCHAR(30) CHECK (status IN (
        'draft',
        'calculating',
        'completed',
        'failed',
        'archived'
    )) DEFAULT 'draft',

    -- Berechnet
    calculated_at TIMESTAMP WITH TIME ZONE,
    calculation_duration INTEGER,       -- Dauer in Millisekunden

    -- Metadaten
    tags JSONB DEFAULT '[]',
    notes TEXT,

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_calculations_project ON calculations(project_id);
CREATE INDEX idx_calculations_building ON calculations(building_id);
CREATE INDEX idx_calculations_type ON calculations(calculation_type);
CREATE INDEX idx_calculations_status ON calculations(status);
```

#### calculation_results (detaillierte Ergebnisse)
```sql
CREATE TABLE calculation_results (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    calculation_id UUID REFERENCES calculations(id) ON DELETE CASCADE,

    result_type VARCHAR(100) NOT NULL,
    result_key VARCHAR(255) NOT NULL,

    -- Werte
    value_numeric DECIMAL(18, 6),
    value_text TEXT,
    value_json JSONB,

    -- Einheit
    unit VARCHAR(50),

    -- Zeitbezug (für zeitabhängige Werte)
    period_type VARCHAR(20) CHECK (period_type IN ('annual', 'monthly', 'daily', 'hourly')),
    period_value INTEGER,               -- Monat 1-12, Tag 1-365, Stunde 0-23

    -- Referenz
    component_id UUID REFERENCES building_components(id),
    zone_id UUID REFERENCES zones(id),
    system_id UUID REFERENCES building_systems(id),

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_results_calculation ON calculation_results(calculation_id);
CREATE INDEX idx_results_type ON calculation_results(result_type);
CREATE INDEX idx_results_key ON calculation_results(result_key);
```

---

### 2.5 Berichte

#### reports
```sql
CREATE TABLE reports (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
    created_by UUID REFERENCES users(id),

    report_type VARCHAR(50) CHECK (report_type IN (
        'energy_certificate_demand',    -- Energieausweis Bedarf
        'energy_certificate_consumption', -- Energieausweis Verbrauch
        'energy_audit',                  -- Energieaudit
        'renovation_roadmap',            -- Sanierungsfahrplan
        'heat_load_report',              -- Heizlastberechnung
        'feasibility_study',             -- Machbarkeitsstudie
        'monitoring_report',             -- Monitoringbericht
        'custom'                         -- Benutzerdefiniert
    )) NOT NULL,

    name VARCHAR(255) NOT NULL,
    description TEXT,

    -- Template
    template_id UUID,                   -- Referenz auf Vorlage
    template_version INTEGER,

    -- Inhalte
    content_data JSONB,                 -- Strukturierte Inhalte
    generated_html TEXT,                -- Generiertes HTML

    -- KI-generierte Texte
    ai_generated_sections JSONB DEFAULT '{}',

    -- Status
    status VARCHAR(30) CHECK (status IN (
        'draft',
        'generating',
        'review',
        'approved',
        'published',
        'archived'
    )) DEFAULT 'draft',

    -- Dateien
    file_path VARCHAR(500),             -- PDF-Pfad
    file_size INTEGER,
    file_hash VARCHAR(64),

    -- Validierung (für offizielle Dokumente)
    is_official BOOLEAN DEFAULT FALSE,
    certificate_number VARCHAR(100),
    valid_until DATE,

    -- Freigabe
    approved_by UUID REFERENCES users(id),
    approved_at TIMESTAMP WITH TIME ZONE,

    -- Versionierung
    version INTEGER DEFAULT 1,
    previous_version_id UUID REFERENCES reports(id),

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_reports_project ON reports(project_id);
CREATE INDEX idx_reports_type ON reports(report_type);
CREATE INDEX idx_reports_status ON reports(status);
```

---

### 2.6 Geschäftsprozesse

#### quotes (Angebote)
```sql
CREATE TABLE quotes (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
    project_id UUID REFERENCES projects(id) ON DELETE SET NULL,
    customer_id UUID REFERENCES customers(id) ON DELETE SET NULL,
    created_by UUID REFERENCES users(id),

    quote_number VARCHAR(50) NOT NULL,
    reference VARCHAR(100),

    -- Daten
    title VARCHAR(255) NOT NULL,
    introduction TEXT,
    terms TEXT,
    notes TEXT,

    -- Beträge
    subtotal DECIMAL(12, 2) DEFAULT 0,
    discount_percent DECIMAL(5, 2) DEFAULT 0,
    discount_amount DECIMAL(12, 2) DEFAULT 0,
    tax_rate DECIMAL(5, 2) DEFAULT 19,
    tax_amount DECIMAL(12, 2) DEFAULT 0,
    total DECIMAL(12, 2) DEFAULT 0,

    -- Währung
    currency VARCHAR(3) DEFAULT 'EUR',

    -- Status
    status VARCHAR(30) CHECK (status IN (
        'draft',
        'sent',
        'viewed',
        'accepted',
        'rejected',
        'expired',
        'converted'         -- In Rechnung umgewandelt
    )) DEFAULT 'draft',

    -- Termine
    valid_until DATE,
    sent_at TIMESTAMP WITH TIME ZONE,
    accepted_at TIMESTAMP WITH TIME ZONE,

    -- Verknüpfung zu Rechnung
    invoice_id UUID,        -- Wird nach Umwandlung gesetzt

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,

    UNIQUE(organization_id, quote_number)
);

CREATE INDEX idx_quotes_organization ON quotes(organization_id);
CREATE INDEX idx_quotes_customer ON quotes(customer_id);
CREATE INDEX idx_quotes_status ON quotes(status);
```

#### quote_items
```sql
CREATE TABLE quote_items (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    quote_id UUID REFERENCES quotes(id) ON DELETE CASCADE,

    position INTEGER NOT NULL,
    item_type VARCHAR(30) CHECK (item_type IN (
        'service',      -- Dienstleistung
        'product',      -- Produkt
        'expense',      -- Auslagen
        'discount',     -- Rabatt
        'text'          -- Nur Text
    )),

    description TEXT NOT NULL,
    details TEXT,

    quantity DECIMAL(10, 3) DEFAULT 1,
    unit VARCHAR(30),               -- Stück, Stunden, Tage, m², etc.
    unit_price DECIMAL(12, 2),
    total DECIMAL(12, 2),

    tax_rate DECIMAL(5, 2),

    sort_order INTEGER DEFAULT 0
);

CREATE INDEX idx_quote_items_quote ON quote_items(quote_id);
```

#### invoices (Rechnungen)
```sql
CREATE TABLE invoices (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
    project_id UUID REFERENCES projects(id) ON DELETE SET NULL,
    customer_id UUID REFERENCES customers(id) ON DELETE SET NULL,
    quote_id UUID REFERENCES quotes(id) ON DELETE SET NULL,
    created_by UUID REFERENCES users(id),

    invoice_number VARCHAR(50) NOT NULL,
    reference VARCHAR(100),

    -- Rechnungsart
    invoice_type VARCHAR(30) CHECK (invoice_type IN (
        'invoice',          -- Rechnung
        'partial',          -- Teilrechnung
        'final',            -- Schlussrechnung
        'credit_note',      -- Gutschrift
        'reminder'          -- Mahnung
    )) DEFAULT 'invoice',

    -- Daten
    title VARCHAR(255) NOT NULL,
    introduction TEXT,
    terms TEXT,
    notes TEXT,

    -- Beträge
    subtotal DECIMAL(12, 2) DEFAULT 0,
    discount_percent DECIMAL(5, 2) DEFAULT 0,
    discount_amount DECIMAL(12, 2) DEFAULT 0,
    tax_rate DECIMAL(5, 2) DEFAULT 19,
    tax_amount DECIMAL(12, 2) DEFAULT 0,
    total DECIMAL(12, 2) DEFAULT 0,

    -- Bereits gezahlt (für Teilrechnungen)
    paid_amount DECIMAL(12, 2) DEFAULT 0,
    outstanding DECIMAL(12, 2) DEFAULT 0,

    -- Währung
    currency VARCHAR(3) DEFAULT 'EUR',

    -- Status
    status VARCHAR(30) CHECK (status IN (
        'draft',
        'sent',
        'viewed',
        'paid',
        'partial',
        'overdue',
        'cancelled',
        'refunded'
    )) DEFAULT 'draft',

    -- Termine
    invoice_date DATE DEFAULT CURRENT_DATE,
    due_date DATE,
    sent_at TIMESTAMP WITH TIME ZONE,
    paid_at TIMESTAMP WITH TIME ZONE,

    -- Mahnwesen
    reminder_level INTEGER DEFAULT 0,
    last_reminder_at TIMESTAMP WITH TIME ZONE,

    -- Datei
    file_path VARCHAR(500),

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,

    UNIQUE(organization_id, invoice_number)
);

CREATE INDEX idx_invoices_organization ON invoices(organization_id);
CREATE INDEX idx_invoices_customer ON invoices(customer_id);
CREATE INDEX idx_invoices_status ON invoices(status);
CREATE INDEX idx_invoices_due ON invoices(due_date);
```

#### invoice_items
```sql
CREATE TABLE invoice_items (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    invoice_id UUID REFERENCES invoices(id) ON DELETE CASCADE,

    position INTEGER NOT NULL,
    item_type VARCHAR(30),

    description TEXT NOT NULL,
    details TEXT,

    quantity DECIMAL(10, 3) DEFAULT 1,
    unit VARCHAR(30),
    unit_price DECIMAL(12, 2),
    total DECIMAL(12, 2),

    tax_rate DECIMAL(5, 2),

    -- Verknüpfung zu Zeiterfassung
    time_entry_ids JSONB DEFAULT '[]',

    sort_order INTEGER DEFAULT 0
);

CREATE INDEX idx_invoice_items_invoice ON invoice_items(invoice_id);
```

#### time_entries (Zeiterfassung)
```sql
CREATE TABLE time_entries (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    project_id UUID REFERENCES projects(id) ON DELETE CASCADE,

    -- Zeit
    date DATE NOT NULL,
    start_time TIME,
    end_time TIME,
    duration INTEGER NOT NULL,          -- Dauer in Minuten

    -- Beschreibung
    description TEXT,
    activity_type VARCHAR(50),          -- Beratung, Berechnung, Bericht, etc.

    -- Abrechenbarkeit
    is_billable BOOLEAN DEFAULT TRUE,
    hourly_rate DECIMAL(10, 2),
    total_amount DECIMAL(10, 2),

    -- Status
    status VARCHAR(30) CHECK (status IN (
        'draft',
        'submitted',
        'approved',
        'rejected',
        'invoiced'
    )) DEFAULT 'draft',

    -- Verknüpfung
    invoice_item_id UUID REFERENCES invoice_items(id),

    -- Genehmigung
    approved_by UUID REFERENCES users(id),
    approved_at TIMESTAMP WITH TIME ZONE,

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_time_entries_user ON time_entries(user_id);
CREATE INDEX idx_time_entries_project ON time_entries(project_id);
CREATE INDEX idx_time_entries_date ON time_entries(date);
CREATE INDEX idx_time_entries_status ON time_entries(status);
```

---

### 2.7 Stammdaten

#### materials (Baumaterialien)
```sql
CREATE TABLE materials (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),

    name VARCHAR(255) NOT NULL,
    name_en VARCHAR(255),
    category VARCHAR(100),

    -- Thermische Eigenschaften
    lambda_value DECIMAL(8, 5),         -- Wärmeleitfähigkeit [W/(mK)]
    density DECIMAL(10, 2),             -- Rohdichte [kg/m³]
    specific_heat DECIMAL(8, 2),        -- Spez. Wärmekapazität [J/(kgK)]

    -- Feuchtetechnisch
    mu_value DECIMAL(10, 2),            -- Wasserdampfdiffusionswiderstand [-]

    -- Mechanisch
    compressive_strength DECIMAL(10, 2),

    -- Ökologisch
    gwp DECIMAL(10, 4),                 -- Global Warming Potential [kgCO2eq/kg]
    primary_energy_non_renewable DECIMAL(10, 2),

    -- Quelle
    source VARCHAR(255),
    source_year INTEGER,

    is_active BOOLEAN DEFAULT TRUE,

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_materials_category ON materials(category);
CREATE INDEX idx_materials_name ON materials USING gin(to_tsvector('german', name));
```

#### climate_data (Klimadaten)
```sql
CREATE TABLE climate_data (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),

    postal_code VARCHAR(10),
    city VARCHAR(100),
    region VARCHAR(100),
    country VARCHAR(100) DEFAULT 'Deutschland',

    -- Jährliche Werte
    heating_degree_days_15 INTEGER,     -- HGT 15/15
    heating_degree_days_20 INTEGER,     -- HGT 20/12
    cooling_degree_days INTEGER,

    -- Auslegungstemperaturen
    design_temp_heating DECIMAL(5, 2),  -- Norm-Außentemperatur Heizen
    design_temp_cooling DECIMAL(5, 2),  -- Norm-Außentemperatur Kühlen

    -- Monatliche Durchschnittswerte
    monthly_temp JSONB,                 -- [Jan, Feb, ..., Dez]
    monthly_radiation JSONB,            -- Globalstrahlung pro Monat

    -- Quelle
    source VARCHAR(255),
    reference_period VARCHAR(50),       -- z.B. "1991-2020"

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_climate_postal ON climate_data(postal_code);
CREATE INDEX idx_climate_city ON climate_data(city);
```

---

## 3. Views

### 3.1 Projektübersicht
```sql
CREATE VIEW v_project_overview AS
SELECT
    p.id,
    p.project_number,
    p.name,
    p.status,
    p.project_type,
    o.name as organization_name,
    c.company_name,
    c.first_name || ' ' || c.last_name as customer_name,
    COUNT(DISTINCT b.id) as building_count,
    COUNT(DISTINCT calc.id) as calculation_count,
    COUNT(DISTINCT r.id) as report_count,
    SUM(te.duration) / 60.0 as total_hours,
    p.created_at,
    p.updated_at
FROM projects p
LEFT JOIN organizations o ON p.organization_id = o.id
LEFT JOIN customers c ON p.customer_id = c.id
LEFT JOIN locations l ON l.project_id = p.id
LEFT JOIN buildings b ON b.location_id = l.id
LEFT JOIN calculations calc ON calc.project_id = p.id
LEFT JOIN reports r ON r.project_id = p.id
LEFT JOIN time_entries te ON te.project_id = p.id
GROUP BY p.id, o.name, c.company_name, c.first_name, c.last_name;
```

### 3.2 Energiekennwerte
```sql
CREATE VIEW v_building_energy_kpis AS
SELECT
    b.id as building_id,
    b.name,
    b.building_type,
    b.heated_area,
    b.a_v_ratio,
    -- Aggregierte Berechnungsergebnisse
    (SELECT value_numeric FROM calculation_results cr
     JOIN calculations c ON cr.calculation_id = c.id
     WHERE c.building_id = b.id
     AND c.calculation_type = 'energy_balance'
     AND cr.result_key = 'primary_energy_demand'
     ORDER BY c.calculated_at DESC LIMIT 1
    ) as primary_energy_demand,
    -- Weitere KPIs...
    b.updated_at
FROM buildings b;
```

---

## 4. Trigger & Funktionen

### 4.1 Automatische Timestamps
```sql
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Anwenden auf alle relevanten Tabellen
CREATE TRIGGER update_users_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();
-- ... weitere Tabellen
```

### 4.2 Audit Logging
```sql
CREATE TABLE audit_log (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    table_name VARCHAR(100) NOT NULL,
    record_id UUID NOT NULL,
    action VARCHAR(20) NOT NULL,
    old_data JSONB,
    new_data JSONB,
    user_id UUID,
    ip_address INET,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE OR REPLACE FUNCTION audit_trigger()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        INSERT INTO audit_log (table_name, record_id, action, new_data)
        VALUES (TG_TABLE_NAME, NEW.id, 'INSERT', row_to_json(NEW));
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO audit_log (table_name, record_id, action, old_data, new_data)
        VALUES (TG_TABLE_NAME, NEW.id, 'UPDATE', row_to_json(OLD), row_to_json(NEW));
    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO audit_log (table_name, record_id, action, old_data)
        VALUES (TG_TABLE_NAME, OLD.id, 'DELETE', row_to_json(OLD));
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;
```

---

*Letzte Aktualisierung: 2025-11-25*
*Version: 1.0.0*
