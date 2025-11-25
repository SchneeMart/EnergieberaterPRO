# Geschäftslogik-Module

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

## 2. Kundenverwaltung (CRM)

### 2.1 Kundentypen

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

### 2.2 Kundenlebenszyklus

```
┌────────────────────────────────────────────────────────────────────────┐
│                      KUNDENLEBENSZYKLUS                                │
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

### 2.3 Kundenservice Funktionen

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

### 3.2 Projektstatus-Workflow

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

### 3.3 Projektstruktur

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

*Letzte Aktualisierung: 2025-11-25*
*Version: 0.1.0*
