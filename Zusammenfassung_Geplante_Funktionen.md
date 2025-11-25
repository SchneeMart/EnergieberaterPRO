# Zusammenfassung: Geplante Funktionen (EnergieberaterPRO)

Dieses Dokument fasst alle geplanten Funktionen der SaaS-Plattform "EnergieberaterPRO" zusammen, basierend auf der bestehenden Projektdokumentation.

---

### 1. Kernplattform & Administration (Für den Plattform-Betreiber/Sysadmin)
- **Mandantenfähige Organisationsverwaltung:** Vollständige Verwaltung (Erstellen, Bearbeiten, Sperren, Löschen) der Kunden der Plattform (Energieberatungsbüros).
- **Abonnement- & Abrechnungsmanagement:** Verwaltung von Abonnements (z.B. Starter, Professional), Abrechnung der Gebühren an die Organisationen und Verwaltung von Zahlungen.
- **Modul-Management:** Verwaltung von optionalen Zusatzmodulen (z.B. erweiterte KI-Funktionen), die Organisationen zu ihrem Abonnement hinzufügen können.
- **Systemüberwachung & -einstellungen:** Zentrales Dashboard mit Systemstatus, globalen Konfigurationen und Monitoring der Plattform-Performance.
- **Support-Werkzeuge:** Funktionen wie "Impersonation" (Anmelden als Benutzer einer Organisation) zur Unterstützung bei Problemen und Zugriff auf zentrale Audit-Logs.

### 2. Organisations- & Benutzerverwaltung (Für Energieberatungsbüros)
- **Benutzer- & Rollenmanagement (RBAC):** Verwaltung von Mitarbeitern mit einem granularen Rollen- und Berechtigungssystem (z.B. Org-Admin, Manager, Berater, Nur-Lesen).
- **Teams & Zugriffssteuerung:** Möglichkeit, Mitarbeiter in Teams zu organisieren und den Zugriff auf Projekte und Kunden zu steuern.
- **Organisations-Einstellungen:** Konfiguration von unternehmensspezifischen Daten, Branding (Logo), Dokumentvorlagen und Standardwerten.
- **Sicherheit:** Verwaltung von Sicherheitsrichtlinien, inklusive der Aktivierung von Multi-Faktor-Authentifizierung (MFA) für alle Mitarbeiter.

### 3. CRM & Geschäftsprozesse
- **Endkunden-Management (CRM):** Verwaltung der eigenen Kunden (Hausbesitzer, Firmen, WEGs), inklusive Kontaktdaten, Historie und zugehörigen Projekten. **Wichtig:** Endkunden haben keinen eigenen Login.
- **Projektmanagement:** Detaillierte Verwaltung von Energieberatungsprojekten von der Anlage bis zum Abschluss, inklusive Status-Tracking, Budgetierung und Zuweisung von Mitarbeitern.
- **Angebotswesen:** Erstellung, Versand und Nachverfolgung von Angeboten an Endkunden. Akzeptierte Angebote können direkt in Rechnungen umgewandelt werden.
- **Rechnungswesen:** Erstellung verschiedener Rechnungstypen (Teil-, Schlussrechnung, Gutschrift) und Nachverfolgung des Zahlungsstatus.
- **Automatisiertes Mahnwesen:** Mehrstufiges System zur automatischen Erinnerung bei überfälligen Rechnungen.
- **Zeiterfassung:** Erfassung von Arbeitszeiten auf Projektebene, Unterscheidung zwischen abrechenbaren und nicht-abrechenbaren Stunden sowie detaillierte Auswertungen.

### 4. Gebäudedaten & Berechnungs-Engine
- **Detaillierte Gebäudemodellierung:** Erfassung von Standorten, Gebäuden, thermischen Zonen, Hüllflächen-Bauteilen und der gesamten Anlagentechnik (Heizung, Lüftung, Kühlung, TWW, Erneuerbare).
- **Gebäude-Versionierung:** Möglichkeit, verschiedene Zustände eines Gebäudes zu verwalten (z.B. "Ist-Zustand" und mehrere "Sanierungsvarianten") und diese miteinander zu vergleichen.
- **Normkonforme Berechnungs-Engine:**
    - **U-Werte:** Nach DIN EN ISO 6946.
    - **Heizlast:** Nach DIN EN 12831.
    - **Energiebilanz:** Umfassende Bilanzierung nach DIN V 18599.
    - **Ökobilanz:** CO2-Bilanzierung nach GHG Protocol.
    - **Wirtschaftlichkeit:** Berechnung von Investitionskosten, Betriebskosten und Amortisationszeiten.
- **Stammdaten-Bibliotheken:** Vorbefüllte und erweiterbare Datenbanken für Baumaterialien und Klimadaten.

### 5. Berichterstellung & Ausgabe
- **Flexibler Berichtsgenerator:** Erstellung von offiziellen Dokumenten (z.B. Energieausweise, Sanierungsfahrpläne) und individuellen Berichten auf Basis von Vorlagen.
- **Vielfältige Export-Formate:** Export von Berichten und Daten als PDF, Excel und standardisierten XML-Formaten (z.B. für BAFA-Anträge).

### 6. KI-Integration
- **Textgenerierung:** KI-gestützte Erstellung von beschreibenden Texten für Berichte (z.B. Gebäudebeschreibung, Maßnahmenempfehlungen).
- **Intelligente Analyse:** Automatische Analyse des energetischen Zustands eines Gebäudes, Identifikation von Schwachstellen und Vergleich von Sanierungsvarianten.
- **Wissens-Assistent (RAG):** Ein Chat-Assistent, der fachliche Fragen auf Basis einer Wissensdatenbank mit Normen, Gesetzen und Fachwissen beantwortet und Quellenangaben liefert.

### 7. Technische Architektur & Sicherheit
- **Multi-Tenancy & Datenisolation:** Strikte Trennung der Daten zwischen den einzelnen Organisationen.
- **Sicherheit:** Umfassendes Sicherheitskonzept nach OWASP Top 10, inklusive starker Authentifizierung (MFA), Brute-Force-Schutz, Verschlüsselung (at-rest, in-transit) und detailliertem Audit-Logging.
- **Offline-Fähigkeit (PWA):** Kernfunktionen der Anwendung sind auch ohne aktive Internetverbindung nutzbar.
- **API-First-Design:** Eine gut dokumentierte REST-API ermöglicht die Integration mit anderen Systemen.