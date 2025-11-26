# Geschäftslogik-Module

## Terminologie-Hinweis

**⚠️ WICHTIG:** In diesem Dokument bezieht sich "Kunde" immer auf **Endkunden** (Kunden der Organisation), NICHT auf Organisationen (Kunden der Plattform).

| Begriff | Bedeutung | Login |
|---------|-----------|-------|
| **Organisation** | Kunde der Plattform (Energieberatungsbüro) | ✓ |
| **Endkunde** | Kunde der Organisation (Hausbesitzer, Firmen) | ✗ KEIN LOGIN |

**Referenz:** `03_FRONTEND/TERMINOLOGIE_KORREKTUR.md`

---

## 1. Übersicht

### 1.1 Geschäftsprozesse

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       GESCHÄFTSPROZESS-ÜBERSICHT                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  AKQUISE          PROJEKT           DURCHFÜHRUNG      ABRECHNUNG           │
│  ┌─────────┐      ┌─────────┐      ┌─────────┐      ┌─────────┐           │
│  │ Anfrage │─────▶│ Angebot │─────▶│ Projekt │─────▶│Rechnung │           │
│  └─────────┘      └─────────┘      └─────────┘      └─────────┘           │
│       │                │                │                │                 │
│       ▼                ▼                ▼                ▼                 │
│  ┌─────────┐      ┌─────────┐      ┌─────────┐      ┌─────────┐           │
│  │ Kunde   │      │Annahme/ │      │Beratung │      │ Zahlung │           │
│  │ anlegen │      │Ablehnung│      │Berechng.│      │         │           │
│  └─────────┘      └─────────┘      │Bericht  │      └─────────┘           │
│                                    └─────────┘                             │
│                                                                             │
│  ZEITERFASSUNG & DOKUMENTATION durchgängig                                 │
│  ═══════════════════════════════════════════════════════════════════════   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Endkunden-Verwaltung (CRM)

Diese Funktionen verwalten die **Endkunden der Organisation** (Hausbesitzer, Firmen, WEGs).
Endkunden haben **KEINEN Login** auf der Plattform - sie werden als Datensätze erfasst.

### 2.1 Endkundentypen

```python
class CustomerType(Enum):
    PRIVATE = "private"          # Privatkunde
    BUSINESS = "business"        # Geschäftskunde
    PUBLIC = "public"            # Öffentliche Hand

# Datenbedarf pro Typ
CUSTOMER_FIELDS = {
    "private": [
        "salutation", "first_name", "last_name",
        "address", "phone", "email"
    ],
    "business": [
        "company_name", "vat_id", "contact_person",
        "address", "phone", "email", "website"
    ],
    "public": [
        "institution_name", "department",
        "contact_person", "address", "email"
    ]
}
```

### 2.2 Endkunden-Lebenszyklus

```
┌────────────────────────────────────────────────────────────────────────┐
│                    ENDKUNDEN-LEBENSZYKLUS                              │
├────────────────────────────────────────────────────────────────────────┤
│                                                                        │
│   LEAD ─────▶ INTERESSENT ─────▶ KUNDE ─────▶ STAMMKUNDE              │
│                                                                        │
│   • Erstkontakt    • Angebot      • Erstes      • Wiederholte         │
│   • Anfrage        • Beratung       Projekt       Projekte            │
│                                   • Abschluss   • Empfehlungen        │
│                                                                        │
│   Aktivitäten:                                                         │
│   ├── E-Mail-Verlauf                                                   │
│   ├── Telefonate                                                       │
│   ├── Meetings                                                         │
│   └── Notizen                                                          │
│                                                                        │
└────────────────────────────────────────────────────────────────────────┘
```

### 2.3 Endkunden-Service Funktionen

```python
# services/customer_service.py
class CustomerService:

    async def create_customer(self, data: CustomerCreate) -> Customer:
        """Legt neuen Kunden an"""
        # Automatische Kundennummer
        customer_number = await self.generate_customer_number()

        customer = Customer(
            customer_number=customer_number,
            **data.dict()
        )
        return await self.repository.create(customer)

    async def search_customers(
        self,
        query: str,
        filters: CustomerFilters = None
    ) -> list[Customer]:
        """Volltextsuche in Kunden"""
        return await self.repository.search(query, filters)

    async def get_customer_statistics(
        self,
        customer_id: UUID
    ) -> CustomerStats:
        """Statistiken für einen Kunden"""
        return CustomerStats(
            total_projects=await self.count_projects(customer_id),
            total_revenue=await self.sum_invoices(customer_id),
            open_invoices=await self.count_open_invoices(customer_id),
            avg_project_value=await self.avg_project_value(customer_id)
        )

    async def merge_customers(
        self,
        primary_id: UUID,
        secondary_id: UUID
    ) -> Customer:
        """Führt zwei Kundendatensätze zusammen"""
        pass
```

---

## 3. Projektverwaltung

### 3.1 Projekttypen

| Typ | Beschreibung | Typische Dauer |
|-----|--------------|----------------|
| energy_audit | Energieaudit nach EDL-G | 2-4 Wochen |
| energy_certificate | Energieausweis | 1-2 Wochen |
| renovation_concept | Sanierungskonzept | 2-6 Wochen |
| heat_load_calc | Heizlastberechnung | 1-2 Wochen |
| feasibility_study | Machbarkeitsstudie | 2-8 Wochen |
| monitoring | Energiemonitoring | Laufend |
| consulting | Allgemeine Beratung | Variabel |

### 3.2 Projekttyp-Kategorien (Norm vs. Flex)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                      PROJEKTTYP-KATEGORIEN                                       │
├──────────────────────────────────┬──────────────────────────────────────────────┤
│      A. NORM-PROJEKTE            │           B. FLEX-PROJEKTE                   │
├──────────────────────────────────┼──────────────────────────────────────────────┤
│  • Starr, geführter Ablauf       │  • Frei gestaltbare Struktur                 │
│  • Wizard-basiert im Frontend    │  • Template-Editor für eigene Checklisten    │
│  • Validierung nach Norm-Vorgabe │  • Eigene Ordnerstrukturen möglich           │
│  • Pflichtfelder & Schritte      │  • Keine Pflicht-Schritte                    │
│                                  │                                              │
│  Beispiele:                      │  Beispiele:                                  │
│  ├── Energieausweis AT (ÖNORM)   │  ├── Individuelle Sanierungskonzepte         │
│  ├── Energieausweis DE (GEG)     │  ├── Kundenspezifische Audits                │
│  ├── iSFP (BAFA)                 │  ├── Freie Beratungsprojekte                 │
│  └── BAFA-Energieaudit           │  └── Machbarkeitsstudien                     │
└──────────────────────────────────┴──────────────────────────────────────────────┘
```

```python
# models/project.py
class ProjectCategory(str, Enum):
    NORM = "norm"      # Geführt, Wizard-basiert, feste Struktur
    FLEX = "flex"      # Frei, Template-Editor, eigene Struktur

class Project(Base):
    # ... bestehende Felder ...

    # Projekttyp-Kategorie
    category: ProjectCategory = ProjectCategory.NORM

    # Für NORM-Projekte: Vordefinierte Norm-Vorlage
    norm_template_id: Optional[UUID] = None  # z.B. "energieausweis_at_wg"

    # Für FLEX-Projekte: Eigene Vorlage aus Template-Editor
    flex_template_id: Optional[UUID] = None

# Vordefinierte Norm-Vorlagen
NORM_TEMPLATES = {
    "energieausweis_at_wg": {
        "name": "Energieausweis Wohngebäude (ÖNORM H 5055)",
        "country": "AT",
        "steps": [
            "gebaeude_stammdaten",
            "bauteile_erfassung",
            "anlagen_erfassung",
            "berechnung",
            "ausweis_generierung"
        ],
        "required_fields": ["hwb", "peb", "co2"],
        "output": "energieausweis_pdf"
    },
    "energieausweis_de_wg": {
        "name": "Energieausweis Wohngebäude (GEG 2024)",
        "country": "DE",
        "steps": ["..."],
        "required_fields": ["..."],
        "output": "energieausweis_pdf"
    },
    "isfp_de": {
        "name": "Individueller Sanierungsfahrplan (BAFA)",
        "country": "DE",
        "steps": ["..."]
    }
}
```

### 3.3 Projektstatus-Workflow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PROJEKT-WORKFLOW                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌────────┐    ┌────────┐    ┌────────┐    ┌────────┐    ┌────────┐      │
│   │ DRAFT  │───▶│ ACTIVE │───▶│  DONE  │───▶│INVOICED│───▶│ARCHIVED│      │
│   │        │    │        │    │        │    │        │    │        │      │
│   └────────┘    └────────┘    └────────┘    └────────┘    └────────┘      │
│        │             │             │                                       │
│        │             ▼             │                                       │
│        │       ┌────────┐         │                                       │
│        └──────▶│ON HOLD │◀────────┘                                       │
│                │        │                                                  │
│                └────────┘                                                  │
│                     │                                                      │
│                     ▼                                                      │
│               ┌──────────┐                                                 │
│               │CANCELLED │                                                 │
│               └──────────┘                                                 │
│                                                                             │
│   Erlaubte Übergänge:                                                      │
│   draft → active, cancelled                                                │
│   active → done, on_hold, cancelled                                        │
│   on_hold → active, cancelled                                              │
│   done → invoiced, on_hold                                                 │
│   invoiced → archived                                                      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.4 Projektstruktur

```python
# Projekt mit Unterstruktur
Project
├── Location(s)           # Standorte
│   ├── Building(s)       # Gebäude
│   │   ├── Zone(s)       # Zonen
│   │   ├── Component(s)  # Bauteile
│   │   └── System(s)     # Anlagen
│   └── ClimateData       # Klimadaten
├── Calculation(s)        # Berechnungen
├── Report(s)             # Berichte
├── Document(s)           # Dokumente
├── TimeEntry(s)          # Zeiteinträge
└── Invoice(s)            # Rechnungen
```

---

## 4. Angebotswesen

### 4.1 Angebots-Workflow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       ANGEBOTS-WORKFLOW                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌────────┐    ┌────────┐    ┌────────┐    ┌─────────┐                   │
│   │ DRAFT  │───▶│  SENT  │───▶│ VIEWED │───▶│ACCEPTED │──┐                │
│   │        │    │        │    │        │    │         │  │                │
│   └────────┘    └────────┘    └────────┘    └─────────┘  │                │
│                      │                           │        │                │
│                      │                           │        ▼                │
│                      │                           │   ┌─────────┐          │
│                      │                           │   │CONVERTED│          │
│                      │                           │   │(Rechng.)│          │
│                      │                           │   └─────────┘          │
│                      │                           ▼                        │
│                      │                     ┌──────────┐                   │
│                      └────────────────────▶│ REJECTED │                   │
│                                            └──────────┘                   │
│                                                  │                        │
│                      ┌──────────────────────────┘                         │
│                      ▼                                                    │
│                ┌──────────┐                                               │
│                │ EXPIRED  │ (Nach Gültigkeitsdatum)                       │
│                └──────────┘                                               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 4.2 Angebotsberechnung

```python
# services/quote_service.py
class QuoteService:

    async def calculate_quote(
        self,
        items: list[QuoteItemCreate],
        discount_percent: float = 0
    ) -> QuoteCalculation:
        """Berechnet Angebotsumme"""

        subtotal = sum(
            item.quantity * item.unit_price
            for item in items
        )

        discount_amount = subtotal * (discount_percent / 100)
        net_amount = subtotal - discount_amount
        tax_amount = net_amount * (self.tax_rate / 100)
        total = net_amount + tax_amount

        return QuoteCalculation(
            subtotal=subtotal,
            discount_amount=discount_amount,
            net_amount=net_amount,
            tax_amount=tax_amount,
            total=total
        )

    async def convert_to_invoice(
        self,
        quote_id: UUID
    ) -> Invoice:
        """Wandelt akzeptiertes Angebot in Rechnung um"""
        quote = await self.get_quote(quote_id)

        if quote.status != QuoteStatus.ACCEPTED:
            raise BusinessError("Nur akzeptierte Angebote können umgewandelt werden")

        # Neue Rechnung erstellen
        invoice = await self.invoice_service.create_from_quote(quote)

        # Angebot als umgewandelt markieren
        quote.status = QuoteStatus.CONVERTED
        quote.invoice_id = invoice.id
        await self.repository.update(quote)

        return invoice

    async def send_quote(
        self,
        quote_id: UUID,
        send_email: bool = True
    ) -> Quote:
        """Versendet Angebot (PDF + E-Mail)"""
        quote = await self.get_quote(quote_id)

        # PDF generieren
        pdf_path = await self.pdf_service.generate_quote_pdf(quote)

        # Als Dokument speichern
        await self.document_service.save(pdf_path, quote.id)

        # E-Mail senden
        if send_email and quote.customer.email:
            await self.email_service.send_quote(
                to=quote.customer.email,
                quote=quote,
                attachment=pdf_path
            )

        # Status aktualisieren
        quote.status = QuoteStatus.SENT
        quote.sent_at = datetime.utcnow()
        return await self.repository.update(quote)
```

### 4.3 Angebots-Positionen

```yaml
Positions-Typen:
  service:        # Dienstleistung
    - Energieberatung (Stunden)
    - Gebäudeaufnahme (Pauschale)
    - Berechnung (Pauschale)
    - Berichterstellung (Pauschale)

  product:        # Produkt/Auslage
    - Energieausweis-Gebühr
    - Messgeräte-Nutzung
    - Software-Lizenz

  expense:        # Auslagen
    - Fahrtkosten
    - Unterkunft
    - Druckkosten

  discount:       # Rabatt
    - Mengenrabatt
    - Treuerabatt
```

---

## 5. Rechnungswesen

### 5.1 Rechnungstypen

| Typ | Beschreibung |
|-----|--------------|
| invoice | Standard-Rechnung |
| partial | Teilrechnung (z.B. 30% bei Auftragsstart) |
| final | Schlussrechnung |
| credit_note | Gutschrift/Korrektur |
| reminder | Zahlungserinnerung/Mahnung |

### 5.2 Rechnungs-Workflow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       RECHNUNGS-WORKFLOW                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌────────┐    ┌────────┐    ┌────────┐    ┌────────┐                    │
│   │ DRAFT  │───▶│  SENT  │───▶│ VIEWED │───▶│  PAID  │                    │
│   │        │    │        │    │        │    │        │                    │
│   └────────┘    └────────┘    └────────┘    └────────┘                    │
│                      │             │                                       │
│                      │             │                                       │
│                      ▼             ▼                                       │
│                ┌──────────┐  ┌──────────┐                                  │
│                │ OVERDUE  │  │ PARTIAL  │ (Teilzahlung)                    │
│                │          │  │          │                                  │
│                └──────────┘  └──────────┘                                  │
│                      │                                                     │
│                      ▼                                                     │
│   Mahnungen:   1. Mahnung ──▶ 2. Mahnung ──▶ 3. Mahnung ──▶ Inkasso      │
│                                                                             │
│                                                                             │
│   ┌──────────┐                 ┌──────────┐                                │
│   │CANCELLED │                 │ REFUNDED │ (Storno/Gutschrift)            │
│   └──────────┘                 └──────────┘                                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 5.3 Mahnwesen

```python
# services/invoice_service.py
class InvoiceReminderService:

    REMINDER_LEVELS = {
        1: {
            "days_overdue": 7,
            "fee": 0,
            "subject": "Freundliche Zahlungserinnerung"
        },
        2: {
            "days_overdue": 21,
            "fee": 5.00,
            "subject": "2. Zahlungserinnerung"
        },
        3: {
            "days_overdue": 35,
            "fee": 10.00,
            "subject": "Letzte Mahnung vor Inkasso"
        }
    }

    async def process_overdue_invoices(self):
        """Verarbeitet überfällige Rechnungen"""
        overdue = await self.repository.get_overdue_invoices()

        for invoice in overdue:
            days_overdue = (date.today() - invoice.due_date).days
            current_level = invoice.reminder_level

            # Nächste Mahnstufe bestimmen
            for level, config in self.REMINDER_LEVELS.items():
                if level > current_level and days_overdue >= config["days_overdue"]:
                    await self.send_reminder(invoice, level)
                    break

    async def send_reminder(
        self,
        invoice: Invoice,
        level: int
    ):
        """Sendet Mahnung"""
        config = self.REMINDER_LEVELS[level]

        # Mahngebühr hinzufügen
        if config["fee"] > 0:
            invoice.outstanding += config["fee"]

        # Mahnung per E-Mail
        await self.email_service.send_reminder(
            to=invoice.customer.email,
            invoice=invoice,
            level=level
        )

        # Status aktualisieren
        invoice.status = InvoiceStatus.OVERDUE
        invoice.reminder_level = level
        invoice.last_reminder_at = datetime.utcnow()
        await self.repository.update(invoice)
```

### 5.4 Zahlungserfassung

```python
async def record_payment(
    self,
    invoice_id: UUID,
    amount: Decimal,
    payment_date: date = None,
    payment_method: str = "bank_transfer",
    reference: str = None
) -> Invoice:
    """Erfasst Zahlungseingang"""
    invoice = await self.get_invoice(invoice_id)

    # Payment erstellen
    payment = Payment(
        invoice_id=invoice_id,
        amount=amount,
        payment_date=payment_date or date.today(),
        payment_method=payment_method,
        reference=reference
    )
    await self.payment_repository.create(payment)

    # Invoice aktualisieren
    invoice.paid_amount += amount
    invoice.outstanding = invoice.total - invoice.paid_amount

    if invoice.outstanding <= 0:
        invoice.status = InvoiceStatus.PAID
        invoice.paid_at = datetime.utcnow()
    elif invoice.paid_amount > 0:
        invoice.status = InvoiceStatus.PARTIAL

    return await self.repository.update(invoice)
```

---

## 6. Zeiterfassung

### 6.1 Zeiterfassungs-Modell

```python
class TimeEntry:
    """Zeiteintrag für Projekte"""

    id: UUID
    user_id: UUID
    project_id: UUID
    date: date
    start_time: time        # Optional
    end_time: time          # Optional
    duration: int           # Minuten (Pflicht)

    description: str
    activity_type: ActivityType

    is_billable: bool
    hourly_rate: Decimal    # Überschreibt Standard
    total_amount: Decimal   # Berechnet

    status: TimeEntryStatus  # draft, submitted, approved, rejected, invoiced

# Aktivitätstypen
class ActivityType(Enum):
    CONSULTING = "consulting"       # Beratung
    SITE_VISIT = "site_visit"       # Vor-Ort-Termin
    DATA_ENTRY = "data_entry"       # Datenerfassung
    CALCULATION = "calculation"     # Berechnung
    REPORT_WRITING = "report"       # Berichterstellung
    REVIEW = "review"               # Prüfung
    MEETING = "meeting"             # Besprechung
    TRAVEL = "travel"               # Reisezeit
    ADMINISTRATION = "admin"        # Verwaltung
```

### 6.2 Zeiterfassungs-Workflow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ZEITERFASSUNGS-WORKFLOW                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   Mitarbeiter                    Manager                   Buchhaltung     │
│                                                                             │
│   ┌────────┐                                                               │
│   │ DRAFT  │ (Erfassung)                                                   │
│   └───┬────┘                                                               │
│       │                                                                    │
│       ▼                                                                    │
│   ┌─────────┐                                                              │
│   │SUBMITTED│ ───────────────────────────▶ Prüfung                        │
│   └─────────┘                                   │                          │
│                                        ┌────────┴────────┐                 │
│                                        ▼                 ▼                 │
│                                   ┌────────┐        ┌────────┐            │
│   ◀──────────────────────────────│REJECTED│        │APPROVED│             │
│   (Zurück zur Korrektur)         └────────┘        └───┬────┘             │
│                                                        │                  │
│                                                        ▼                  │
│                                                   ┌─────────┐             │
│                                                   │INVOICED │             │
│                                                   └─────────┘             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 6.3 Zeitauswertung

```python
# services/time_service.py
class TimeReportService:

    async def get_project_time_summary(
        self,
        project_id: UUID
    ) -> ProjectTimeSummary:
        """Zusammenfassung der Projektzeiten"""
        entries = await self.repository.get_by_project(project_id)

        total_hours = sum(e.duration for e in entries) / 60
        billable_hours = sum(
            e.duration for e in entries if e.is_billable
        ) / 60
        billable_amount = sum(
            e.total_amount for e in entries if e.is_billable
        )

        # Nach Aktivität gruppieren
        by_activity = {}
        for entry in entries:
            if entry.activity_type not in by_activity:
                by_activity[entry.activity_type] = 0
            by_activity[entry.activity_type] += entry.duration / 60

        # Budget-Vergleich
        project = await self.project_service.get(project_id)
        budget_used = (total_hours / project.budget_hours * 100
                      if project.budget_hours else None)

        return ProjectTimeSummary(
            total_hours=total_hours,
            billable_hours=billable_hours,
            billable_amount=billable_amount,
            by_activity=by_activity,
            budget_hours=project.budget_hours,
            budget_used_percent=budget_used
        )

    async def get_user_timesheet(
        self,
        user_id: UUID,
        start_date: date,
        end_date: date
    ) -> UserTimesheet:
        """Zeitnachweis für einen Benutzer"""
        pass

    async def export_time_report(
        self,
        filters: TimeReportFilters,
        format: str = "excel"
    ) -> bytes:
        """Exportiert Zeitbericht"""
        pass
```

---

## 7. Dokumentenmanagement

### 7.1 Dokumententypen

```python
class DocumentType(Enum):
    # Eingangsdokumente
    FLOOR_PLAN = "floor_plan"           # Grundriss
    ENERGY_BILL = "energy_bill"         # Energierechnung
    INVOICE_RECEIVED = "invoice_recv"    # Eingangsrechnung
    PHOTO = "photo"                      # Foto
    MEASUREMENT = "measurement"          # Messprotokolle
    EXISTING_REPORT = "existing_report"  # Bestehende Gutachten

    # Generierte Dokumente
    QUOTE_PDF = "quote_pdf"              # Angebots-PDF
    INVOICE_PDF = "invoice_pdf"          # Rechnungs-PDF
    REPORT_PDF = "report_pdf"            # Bericht-PDF
    CERTIFICATE_PDF = "certificate_pdf"  # Ausweis-PDF
    EXPORT_FILE = "export_file"          # Export (Excel, XML)

    # Sonstige
    CONTRACT = "contract"                # Vertrag
    CORRESPONDENCE = "correspondence"    # Schriftverkehr
    NOTE = "note"                        # Notiz
```

### 7.2 Speicherstruktur

```
data/uploads/
└── {organization_id}/
    └── {project_id}/
        ├── input/              # Eingangsdokumente
        │   ├── plans/
        │   ├── photos/
        │   └── bills/
        ├── output/             # Generierte Dokumente
        │   ├── reports/
        │   └── exports/
        └── misc/               # Sonstiges
```

### 7.3 Versionierung

```python
class DocumentService:

    async def upload_document(
        self,
        file: UploadFile,
        project_id: UUID,
        document_type: DocumentType,
        metadata: dict = None
    ) -> Document:
        """Lädt Dokument hoch"""

        # Hash für Deduplizierung
        file_hash = await self.calculate_hash(file)

        # Prüfen ob identische Datei existiert
        existing = await self.repository.find_by_hash(file_hash)
        if existing:
            # Nur neue Version anlegen
            return await self.create_version(existing, file)

        # Neue Datei speichern
        path = await self.storage.upload(file, project_id)

        return await self.repository.create(Document(
            project_id=project_id,
            document_type=document_type,
            original_filename=file.filename,
            stored_path=path,
            file_size=file.size,
            mime_type=file.content_type,
            file_hash=file_hash,
            metadata=metadata,
            version=1
        ))

    async def create_version(
        self,
        document: Document,
        file: UploadFile
    ) -> Document:
        """Erstellt neue Version eines Dokuments"""
        document.version += 1
        # Alte Version archivieren
        # Neue Datei speichern
        pass
```

---

## 8. Abrechnungssysteme

### 8.1 Abrechnungsmodelle

```python
class BillingType(Enum):
    FIXED_PRICE = "fixed_price"      # Festpreis
    HOURLY = "hourly"                # Stundenbasis
    DAILY = "daily"                  # Tagessatz
    MILESTONE = "milestone"          # Meilensteine
    SUBSCRIPTION = "subscription"    # Abo/Wartungsvertrag

# Konfiguration pro Typ
BILLING_CONFIG = {
    "fixed_price": {
        "requires_quote": True,
        "time_tracking": "optional",
        "invoice_trigger": "project_complete"
    },
    "hourly": {
        "requires_quote": False,
        "time_tracking": "required",
        "invoice_trigger": "monthly"
    },
    "milestone": {
        "requires_quote": True,
        "time_tracking": "optional",
        "invoice_trigger": "milestone_complete"
    },
    "subscription": {
        "requires_quote": False,
        "time_tracking": "optional",
        "invoice_trigger": "recurring"
    }
}
```

### 8.2 Automatische Rechnungserstellung

```python
class AutoInvoiceService:

    async def process_monthly_invoices(self):
        """Erstellt monatliche Rechnungen für Stundenprojekte"""
        projects = await self.get_hourly_projects()

        for project in projects:
            # Nicht abgerechnete Zeiteinträge
            entries = await self.time_service.get_uninvoiced(
                project_id=project.id,
                end_date=date.today()
            )

            if not entries:
                continue

            # Rechnung erstellen
            invoice = await self.create_invoice_from_time(
                project=project,
                time_entries=entries
            )

            # Zeiteinträge als abgerechnet markieren
            for entry in entries:
                entry.status = TimeEntryStatus.INVOICED
                entry.invoice_item_id = ...

    async def create_milestone_invoice(
        self,
        project_id: UUID,
        milestone: str,
        percentage: int
    ) -> Invoice:
        """Erstellt Meilenstein-Rechnung"""
        project = await self.project_service.get(project_id)

        amount = project.budget_amount * (percentage / 100)

        return await self.invoice_service.create(
            project_id=project_id,
            invoice_type=InvoiceType.PARTIAL,
            items=[
                InvoiceItem(
                    description=f"Meilenstein: {milestone} ({percentage}%)",
                    quantity=1,
                    unit_price=amount
                )
            ]
        )
```

---

## 9. Berichtswesen (Business)

### 9.1 Standard-Berichte

```
Dashboard-Berichte:
├── Umsatzübersicht (Monat/Quartal/Jahr)
├── Projektübersicht (Status, Budget)
├── Auslastung (Team/Einzelperson)
└── Offene Posten

Detailberichte:
├── Projektrentabilität
├── Zeitauswertung pro Projekt
├── Kundenumsatz
├── Maßnahmen-Pipeline
└── Zahlungsübersicht

Export-Formate:
├── PDF (Druckoptimiert)
├── Excel (Analyse)
└── CSV (Import in andere Systeme)
```

### 9.2 KPI-Dashboard

```python
@dataclass
class BusinessKPIs:
    # Finanzen
    revenue_mtd: Decimal          # Umsatz Monat
    revenue_ytd: Decimal          # Umsatz Jahr
    outstanding_invoices: Decimal  # Offene Forderungen
    overdue_invoices: Decimal     # Überfällige Forderungen

    # Projekte
    active_projects: int          # Aktive Projekte
    projects_on_track: int        # Im Plan
    projects_delayed: int         # Verzögert
    projects_completed_mtd: int   # Abgeschlossen (Monat)

    # Auslastung
    billable_hours_mtd: float     # Abrechenbare Stunden
    billable_rate: float          # Abrechenbarkeitsquote
    avg_hourly_rate: Decimal      # Durchschnittlicher Stundensatz

    # Pipeline
    quotes_pending: int           # Offene Angebote
    quotes_value: Decimal         # Wert offener Angebote
    conversion_rate: float        # Annahme-Quote

async def calculate_business_kpis(
    organization_id: UUID,
    period: str = "mtd"
) -> BusinessKPIs:
    """Berechnet Business KPIs"""
    pass
```

---

## 10. Externe Integrationen

### 10.1 Buchhaltungs-Integration

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    BUCHHALTUNGS-INTEGRATIONEN                                    │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  DEUTSCHLAND                                 ÖSTERREICH                          │
│  ══════════                                  ══════════                          │
│  ┌─────────────┐  ┌─────────────┐           ┌─────────────┐                    │
│  │   DATEV     │  │  Lexoffice  │           │    BMD      │                    │
│  │   Export    │  │     API     │           │   Export    │                    │
│  └─────────────┘  └─────────────┘           └─────────────┘                    │
│        │               │                          │                             │
│        │               │                          │                             │
│        ▼               ▼                          ▼                             │
│  ┌───────────────────────────────────────────────────────────────────────┐     │
│  │                    EnergieberaterPRO                                   │     │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │     │
│  │  │  Rechnungen │  │  Zahlungen  │  │   Konten    │  │   Export    │  │     │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  │     │
│  └───────────────────────────────────────────────────────────────────────┘     │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

#### DATEV-Export

```python
# services/integrations/datev.py
class DatevExporter:
    """
    Exportiert Buchungssätze im DATEV-Format
    """

    def __init__(self):
        self.kontenrahmen = "SKR04"  # oder SKR03

    async def export_invoices(
        self,
        organization_id: UUID,
        period: tuple[date, date]
    ) -> bytes:
        """Exportiert Rechnungen als DATEV-CSV"""
        invoices = await self.get_invoices(organization_id, period)

        rows = []
        for invoice in invoices:
            rows.append({
                'Belegdatum': invoice.date.strftime('%d%m'),
                'Buchungstext': f"RE {invoice.invoice_number}",
                'Umsatz': invoice.total,
                'S/H': 'S',
                'Konto': self._get_debitor_konto(invoice.customer),
                'Gegenkonto': self._get_erloes_konto(invoice),
                'Buchungsschlüssel': self._get_buchungsschluessel(invoice.tax_rate)
            })

        return self._to_csv(rows)

    def _get_debitor_konto(self, customer) -> str:
        """Debitorenkonto aus Kundennummer"""
        return f"1{customer.customer_number.zfill(5)}"

    def _get_erloes_konto(self, invoice) -> str:
        """Erlöskonto nach Leistungsart"""
        return {
            'energy_consulting': '8400',
            'energy_certificate': '8401',
            'audit': '8402'
        }.get(invoice.service_type, '8400')
```

#### Lexoffice API

```python
# services/integrations/lexoffice.py
class LexofficeClient:
    """
    Integration mit Lexoffice Buchhaltung
    """

    BASE_URL = "https://api.lexoffice.io/v1"

    async def sync_invoice(self, invoice: Invoice) -> str:
        """Synchronisiert Rechnung zu Lexoffice"""
        payload = {
            "voucherDate": invoice.date.isoformat(),
            "address": self._format_address(invoice.customer),
            "lineItems": [
                {
                    "type": "custom",
                    "name": item.description,
                    "quantity": item.quantity,
                    "unitPrice": {
                        "currency": "EUR",
                        "netAmount": float(item.net_amount),
                        "taxRatePercentage": item.tax_rate
                    }
                }
                for item in invoice.items
            ],
            "totalPrice": {
                "currency": "EUR"
            },
            "taxConditions": {
                "taxType": "net"
            }
        }

        response = await self.client.post(
            f"{self.BASE_URL}/invoices",
            json=payload
        )
        return response.json()["id"]

    async def sync_payment(
        self,
        lexoffice_invoice_id: str,
        payment: Payment
    ):
        """Meldet Zahlungseingang"""
        pass
```

### 10.2 Kalender-Integration

```python
# services/integrations/calendar.py
from caldav import DAVClient

class CalendarIntegration:
    """
    CalDAV-Integration für Terminplanung
    """

    async def sync_project_milestones(
        self,
        project: Project,
        calendar_url: str
    ):
        """Synchronisiert Projekt-Meilensteine mit Kalender"""
        client = DAVClient(url=calendar_url)
        calendar = client.calendar(url=calendar_url)

        for milestone in project.milestones:
            event = self._create_event(milestone)
            calendar.save_event(event)

    async def create_appointment(
        self,
        project_id: UUID,
        customer_id: UUID,
        datetime_start: datetime,
        duration_minutes: int,
        title: str,
        location: str = None
    ) -> CalendarEvent:
        """Erstellt Termin und sendet Einladung"""
        pass

    async def get_availability(
        self,
        user_id: UUID,
        date_range: tuple[date, date]
    ) -> list[TimeSlot]:
        """Prüft Verfügbarkeit aus Kalender"""
        pass
```

### 10.3 IoT & Smart-Meter Integration

```python
# services/integrations/iot.py
import aiomqtt

class IoTDataCollector:
    """
    Sammelt Daten von IoT-Geräten und Smart-Metern
    """

    async def connect_mqtt(self, broker_url: str):
        """Verbindet mit MQTT-Broker"""
        async with aiomqtt.Client(broker_url) as client:
            await client.subscribe("energy/#")

            async for message in client.messages:
                await self._process_message(message)

    async def _process_message(self, message):
        """Verarbeitet eingehende Messwerte"""
        topic = str(message.topic)
        payload = json.loads(message.payload)

        # topic: energy/{building_id}/{meter_type}
        building_id = topic.split('/')[1]
        meter_type = topic.split('/')[2]

        await self.measurement_service.store(
            building_id=building_id,
            meter_type=meter_type,
            timestamp=payload.get('timestamp', datetime.utcnow()),
            value=payload['value'],
            unit=payload.get('unit', 'kWh')
        )

    async def import_csv(
        self,
        building_id: UUID,
        file: UploadFile,
        mapping: dict
    ) -> int:
        """Importiert Messwerte aus CSV"""
        pass
```

### 10.4 E-Mail-Integration

```python
# services/integrations/email.py
class EmailIntegration:
    """
    E-Mail-Versand und -Empfang
    """

    async def send_with_tracking(
        self,
        to: str,
        subject: str,
        body_html: str,
        attachments: list[str] = None,
        track_opens: bool = True,
        track_clicks: bool = True
    ) -> str:
        """Sendet E-Mail mit Tracking"""
        message_id = str(uuid4())

        # Tracking-Pixel einfügen
        if track_opens:
            body_html = self._add_tracking_pixel(body_html, message_id)

        # Links tracken
        if track_clicks:
            body_html = self._wrap_links(body_html, message_id)

        await self.smtp.send(to, subject, body_html, attachments)

        return message_id

    async def process_incoming(self):
        """Verarbeitet eingehende E-Mails (IMAP)"""
        async for email in self.imap.fetch_new():
            # Projekt-Zuordnung versuchen
            project = await self._match_to_project(email)

            if project:
                # Als Korrespondenz speichern
                await self.document_service.save_email(
                    project_id=project.id,
                    email=email
                )
```

---

## 11. No-Code Automatisierung

### 11.1 Workflow-Engine

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         NO-CODE WORKFLOW ENGINE                                  │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  TRIGGER                    BEDINGUNGEN                AKTIONEN                 │
│  ═══════                    ══════════                ════════                  │
│                                                                                 │
│  ┌─────────────┐           ┌─────────────┐          ┌─────────────┐           │
│  │ Zeit-basiert│           │ Feld-Prüfung │          │  E-Mail     │           │
│  │ (Cron)      │    ───▶   │ Status = X   │   ───▶   │  senden     │           │
│  └─────────────┘           │ Betrag > Y   │          └─────────────┘           │
│                            └─────────────┘                                      │
│  ┌─────────────┐                                     ┌─────────────┐           │
│  │ Ereignis    │           ┌─────────────┐          │ Status      │           │
│  │ (Webhook)   │    ───▶   │    UND/ODER  │   ───▶   │ ändern      │           │
│  └─────────────┘           │  Verknüpfung │          └─────────────┘           │
│                            └─────────────┘                                      │
│  ┌─────────────┐                                     ┌─────────────┐           │
│  │ Daten-Änd.  │           ┌─────────────┐          │ Aufgabe     │           │
│  │ (DB Event)  │    ───▶   │ Verzögerung │   ───▶   │ erstellen   │           │
│  └─────────────┘           └─────────────┘          └─────────────┘           │
│                                                                                 │
│  ┌─────────────┐                                     ┌─────────────┐           │
│  │ Manuell     │                                     │ Webhook     │           │
│  │ (Button)    │           ───────────────────▶      │ aufrufen    │           │
│  └─────────────┘                                     └─────────────┘           │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 11.2 Workflow-Definition

```python
# models/workflow.py
class Workflow:
    """No-Code Workflow Definition"""

    id: UUID
    organization_id: UUID
    name: str
    description: str

    trigger: WorkflowTrigger
    conditions: list[WorkflowCondition]
    actions: list[WorkflowAction]

    is_active: bool
    run_count: int
    last_run_at: datetime

class WorkflowTrigger:
    type: TriggerType  # schedule, event, webhook, manual
    config: dict

    # schedule: {"cron": "0 9 * * *"}
    # event: {"entity": "invoice", "event": "status_changed", "to": "overdue"}
    # webhook: {"path": "/webhook/xyz", "method": "POST"}

class WorkflowCondition:
    field: str          # "invoice.total"
    operator: str       # "gt", "eq", "contains", etc.
    value: Any
    logic: str          # "and", "or"

class WorkflowAction:
    type: ActionType
    config: dict
    order: int

    # send_email: {"to": "{{customer.email}}", "template": "reminder"}
    # update_status: {"entity": "invoice", "status": "reminded"}
    # create_task: {"title": "...", "assignee": "..."}
    # webhook: {"url": "...", "method": "POST", "body": {...}}
```

### 11.3 Vordefinierte Workflows

```yaml
# Workflow-Templates
workflows:
  invoice_reminder:
    name: "Automatische Zahlungserinnerung"
    trigger:
      type: schedule
      cron: "0 9 * * *"  # Täglich 9 Uhr
    conditions:
      - field: invoice.status
        operator: eq
        value: sent
      - field: invoice.due_date
        operator: lt
        value: "{{today - 7 days}}"
    actions:
      - type: send_email
        template: payment_reminder
        to: "{{invoice.customer.email}}"
      - type: update_field
        entity: invoice
        field: reminder_count
        value: "{{invoice.reminder_count + 1}}"

  project_completion:
    name: "Projekt-Abschluss Workflow"
    trigger:
      type: event
      entity: project
      event: status_changed
      to: done
    actions:
      - type: send_email
        template: project_completed
        to: "{{project.customer.email}}"
      - type: create_task
        title: "Rechnung erstellen"
        assignee: "{{project.owner}}"
        due_in_days: 3

  new_customer_onboarding:
    name: "Neukunden-Onboarding"
    trigger:
      type: event
      entity: customer
      event: created
    actions:
      - type: send_email
        template: welcome
        to: "{{customer.email}}"
      - type: create_task
        title: "Erstgespräch terminieren"
        assignee: "{{customer.owner}}"
      - type: webhook
        url: "https://crm.example.com/api/new_lead"
        method: POST
```

---

## 12. Endkunden-Portal & Public Links

### 12.1 Portal-Konzept

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                      ENDKUNDEN-PORTAL (Read-Only)                                │
│                                                                                 │
│  ⚠️ KEIN vollständiger Login - Token-basierter Zugang                           │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  PROJEKTSTATUS                                                          │   │
│  │  ═════════════                                                          │   │
│  │                                                                         │   │
│  │  Projekt: Energieberatung Musterstraße 1                               │   │
│  │  Status:  ████████████░░░░ 75% - Bericht in Erstellung                 │   │
│  │                                                                         │   │
│  │  Timeline:                                                              │   │
│  │  ✓ Auftragsannahme          15.11.2025                                 │   │
│  │  ✓ Vor-Ort-Termin           18.11.2025                                 │   │
│  │  ✓ Datenauswertung          20.11.2025                                 │   │
│  │  ◐ Berichterstellung        In Arbeit                                  │   │
│  │  ○ Abschlussgespräch        Geplant 28.11.2025                         │   │
│  │                                                                         │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
│  ┌───────────────────────────┐  ┌───────────────────────────┐                 │
│  │  DOKUMENTE                 │  │  UPLOAD                   │                 │
│  │  ══════════                │  │  ══════                   │                 │
│  │                            │  │                           │                 │
│  │  📄 Angebot_2025-001.pdf  │  │  [Datei auswählen]       │                 │
│  │  📄 Energieausweis.pdf    │  │                           │                 │
│  │                            │  │  Hochladen:              │                 │
│  │  [Download]               │  │  • Energierechnungen     │                 │
│  │                            │  │  • Grundrisse            │                 │
│  └───────────────────────────┘  │  • Fotos                 │                 │
│                                  └───────────────────────────┘                 │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 12.2 Datenerhebungs-Links (Public Links)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│            KOMPONENTE: "DATENERHEBUNGS-LINK GENERIEREN"                         │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  Neuen Datenerhebungs-Link erstellen                                    │   │
│  │  ═════════════════════════════════════                                  │   │
│  │                                                                         │   │
│  │  Modus:                                                                 │   │
│  │  ┌─────────────────────────────────────────────────────────────────┐   │   │
│  │  │  ○ Modus A: "Nur Upload"                                        │   │   │
│  │  │    └─ Endkunde kann Dateien hochladen (Fotos, Rechnungen, Pläne)│   │   │
│  │  │    └─ Keine Bearbeitung bestehender Dokumente                   │   │   │
│  │  │                                                                 │   │   │
│  │  │  ○ Modus B: "Bearbeitung erlaubt" (via Collabora/WOPI)         │   │   │
│  │  │    └─ Endkunde kann .xlsx-Checklisten direkt im Browser füllen │   │   │
│  │  │    └─ Live-Bearbeitung via Collabora Online                     │   │   │
│  │  │    └─ Änderungen werden automatisch gespeichert                 │   │   │
│  │  └─────────────────────────────────────────────────────────────────┘   │   │
│  │                                                                         │   │
│  │  Gültigkeit:  [▼ 30 Tage          ]                                    │   │
│  │               (1 Tag / 7 Tage / 30 Tage / Unbegrenzt)                  │   │
│  │                                                                         │   │
│  │  Optionen:                                                              │   │
│  │  [x] Passwortschutz aktivieren    Passwort: [______________]           │   │
│  │  [x] E-Mail-Benachrichtigung bei Upload/Änderung                       │   │
│  │  [ ] Mehrfache Nutzung erlauben (Single-Use Link)                      │   │
│  │                                                                         │   │
│  │  Dokumente für Bearbeitung freigeben (Modus B):                        │   │
│  │  [x] Gebäudedaten_Checkliste.xlsx                                      │   │
│  │  [x] Energierechnungen_Template.xlsx                                    │   │
│  │  [ ] Anlagenerfassung.xlsx                                              │   │
│  │                                                                         │   │
│  │                              [Link generieren]                          │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
│  Generierter Link:                                                              │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  https://app.energieberaterpro.de/portal/d8f3k2...                     │   │
│  │  [Kopieren] [Per E-Mail senden] [QR-Code anzeigen]                     │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

```python
# services/public_link.py
class PublicLinkService:
    """
    Service für Datenerhebungs-Links (Public Links)
    """

    class LinkMode(str, Enum):
        UPLOAD_ONLY = "upload_only"      # Modus A: Nur Upload
        EDIT_ALLOWED = "edit_allowed"    # Modus B: Bearbeitung via Collabora

    async def generate_data_collection_link(
        self,
        project_id: UUID,
        mode: LinkMode,
        valid_days: int = 30,
        password: Optional[str] = None,
        notify_on_activity: bool = True,
        shared_documents: list[UUID] = None,  # Für EDIT_ALLOWED Modus
        single_use: bool = False
    ) -> PublicLink:
        """Generiert Datenerhebungs-Link"""
        token = secrets.token_urlsafe(32)
        expires_at = datetime.utcnow() + timedelta(days=valid_days) if valid_days else None

        link = PublicLink(
            token=token,
            project_id=project_id,
            mode=mode,
            expires_at=expires_at,
            password_hash=self._hash_password(password) if password else None,
            notify_on_activity=notify_on_activity,
            shared_document_ids=shared_documents or [],
            single_use=single_use,
            access_count=0
        )

        await self.repository.create(link)

        return link

    async def access_link(
        self,
        token: str,
        password: Optional[str] = None
    ) -> PublicLinkAccess:
        """Validiert und gibt Zugang zum Link"""
        link = await self.repository.get_by_token(token)

        if not link:
            raise LinkNotFound()

        if link.expires_at and link.expires_at < datetime.utcnow():
            raise LinkExpired()

        if link.single_use and link.access_count > 0:
            raise LinkAlreadyUsed()

        if link.password_hash and not self._verify_password(password, link.password_hash):
            raise InvalidPassword()

        # Zugriff protokollieren
        link.access_count += 1
        link.last_access_at = datetime.utcnow()
        await self.repository.update(link)

        # Benachrichtigung senden
        if link.notify_on_activity:
            await self._notify_project_owner(link, "access")

        return PublicLinkAccess(
            project_id=link.project_id,
            mode=link.mode,
            can_upload=True,
            can_edit=link.mode == LinkMode.EDIT_ALLOWED,
            shared_documents=link.shared_document_ids
        )

    async def get_collabora_url(
        self,
        token: str,
        document_id: UUID
    ) -> str:
        """Generiert Collabora IFrame URL für Dokument-Bearbeitung"""
        link = await self.repository.get_by_token(token)

        if link.mode != LinkMode.EDIT_ALLOWED:
            raise EditNotAllowed()

        if document_id not in link.shared_document_ids:
            raise DocumentNotShared()

        # WOPI URL generieren
        wopi_src = f"{self.base_url}/api/wopi/files/{document_id}?access_token={token}"
        collabora_url = f"{self.collabora_url}/browser/dist/cool.html?WOPISrc={quote(wopi_src)}"

        return collabora_url
```

### 12.3 WOPI-Integration für Public Links

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                COLLABORA/WOPI INTEGRATION FÜR PUBLIC LINKS                       │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────┐                    ┌──────────────────────────────────────┐   │
│  │  Endkunde   │                    │  Collabora Online (CODE Container)  │   │
│  │  (Browser)  │                    │                                      │   │
│  └──────┬──────┘                    └──────────────────┬───────────────────┘   │
│         │                                              │                        │
│         │ 1. Öffnet Link                               │                        │
│         ▼                                              │                        │
│  ┌──────────────────────────────────────────────┐     │                        │
│  │  Frontend: Public Link Portal                 │     │                        │
│  │  ┌──────────────────────────────────────────┐│     │                        │
│  │  │ IFrame mit Collabora                      ││     │                        │
│  │  │                                          ││◄────┘                        │
│  │  │  ┌──────────────────────────────────┐   ││                               │
│  │  │  │ Checkliste.xlsx                   │   ││                               │
│  │  │  │ Live-Bearbeitung im Browser      │   ││                               │
│  │  │  └──────────────────────────────────┘   ││                               │
│  │  └──────────────────────────────────────────┘│                               │
│  └──────────────────────────────────────────────┘                               │
│         │                                                                       │
│         │ 2. Collabora lädt/speichert via WOPI                                 │
│         ▼                                                                       │
│  ┌──────────────────────────────────────────────┐                               │
│  │  Backend: WOPI Endpoints                      │                               │
│  │  /api/wopi/files/{file_id}                   │                               │
│  │                                               │                               │
│  │  • CheckFileInfo - Dateiinfos abrufen        │                               │
│  │  • GetFile - Datei laden                      │                               │
│  │  • PutFile - Änderungen speichern            │                               │
│  │  • Lock/Unlock - Concurrent Editing           │                               │
│  └──────────────────────────────────────────────┘                               │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

```python
# api/wopi.py
from fastapi import APIRouter, Request, Response

router = APIRouter(prefix="/api/wopi")

@router.get("/files/{file_id}")
async def check_file_info(
    file_id: UUID,
    access_token: str,
    request: Request
):
    """WOPI CheckFileInfo - Dateiinformationen"""
    # Token validieren (kann normaler Token oder Public Link Token sein)
    access = await validate_wopi_access(access_token, file_id)

    document = await document_service.get(file_id)

    return {
        "BaseFileName": document.original_filename,
        "Size": document.file_size,
        "OwnerId": str(document.created_by),
        "UserId": str(access.user_id or "public"),
        "UserFriendlyName": access.user_name or "Endkunde",
        "UserCanWrite": access.can_edit,
        "UserCanNotWriteRelative": True,
        "LastModifiedTime": document.updated_at.isoformat(),
        "SHA256": document.file_hash,
        "Version": str(document.version)
    }

@router.get("/files/{file_id}/contents")
async def get_file(
    file_id: UUID,
    access_token: str
):
    """WOPI GetFile - Datei herunterladen"""
    await validate_wopi_access(access_token, file_id)
    content = await storage_service.get_file(file_id)
    return Response(content=content, media_type="application/octet-stream")

@router.post("/files/{file_id}/contents")
async def put_file(
    file_id: UUID,
    access_token: str,
    request: Request
):
    """WOPI PutFile - Datei speichern"""
    access = await validate_wopi_access(access_token, file_id, require_write=True)

    content = await request.body()
    await storage_service.update_file(file_id, content)

    # Benachrichtigung bei Änderung durch Endkunde
    if access.is_public_link:
        await notify_project_owner(file_id, "document_edited")

    return Response(status_code=200)
```

### 12.4 Token-basierter Zugang

```python
# services/customer_portal.py
class CustomerPortalService:
    """
    Endkunden-Portal mit Token-Zugang (KEIN Login!)
    """

    async def generate_access_link(
        self,
        project_id: UUID,
        valid_days: int = 30
    ) -> str:
        """Generiert einmaligen Zugangslink"""
        token = secrets.token_urlsafe(32)
        expires_at = datetime.utcnow() + timedelta(days=valid_days)

        await self.repository.create_token(
            token=token,
            project_id=project_id,
            expires_at=expires_at
        )

        return f"https://app.energieberaterpro.de/portal/{token}"

    async def get_portal_data(
        self,
        token: str
    ) -> PortalData:
        """Lädt Portal-Daten für Token"""
        access = await self.repository.get_by_token(token)

        if not access or access.expires_at < datetime.utcnow():
            raise AccessDenied()

        project = await self.project_service.get(access.project_id)

        return PortalData(
            project_name=project.name,
            status=project.status,
            progress=project.progress_percent,
            milestones=project.milestones,
            documents=await self._get_shared_documents(project.id),
            can_upload=project.status == ProjectStatus.ACTIVE
        )

    async def handle_upload(
        self,
        token: str,
        file: UploadFile
    ) -> Document:
        """Verarbeitet Upload vom Endkunden"""
        access = await self.repository.get_by_token(token)
        # ... Validierung ...

        return await self.document_service.upload(
            file=file,
            project_id=access.project_id,
            document_type=DocumentType.CUSTOMER_UPLOAD,
            uploaded_by_customer=True
        )
```

---

*Letzte Aktualisierung: 2025-11-26*
*Version: 2.1.0*
