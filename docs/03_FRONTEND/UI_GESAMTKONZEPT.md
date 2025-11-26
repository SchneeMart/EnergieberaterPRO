# UI/UX Gesamtkonzept - EnergieberaterPRO

## Korrigierte Benutzer-Hierarchie

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        PLATTFORM-HIERARCHIE                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  SYSADMIN (Websitebetreiber)                                                │
│  └── Betreibt die SaaS-Plattform "EnergieberaterPRO"                        │
│      └── Verkauft Abonnements an Organisationen                             │
│                                                                              │
│  ORGANISATIONEN (Kunden des Sysadmins)                                      │
│  └── Energieberatungsbüros, die die Plattform nutzen                        │
│      ├── Org-Admin: Verwaltet die Organisation                              │
│      └── Mitarbeiter: Arbeiten mit der Plattform                            │
│                                                                              │
│  ENDKUNDEN (Kunden der Organisationen)                                      │
│  └── Hausbesitzer, Unternehmen, die Energieberatung beauftragen             │
│      └── Haben KEINEN eigenen Login auf der Plattform                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Begriffsdefinitionen (Konsistente Terminologie)

| Begriff | Bedeutung | Verwendet in |
|---------|-----------|--------------|
| **Sysadmin** | Betreiber der Plattform | Backend, Admin-Panel |
| **Organisation** | Kunde des Sysadmins (Energieberatungsbüro) | Überall |
| **Org-Admin** | Administrator einer Organisation | Org-Verwaltung |
| **Mitarbeiter** | Angestellter einer Organisation | Arbeits-UI |
| **Endkunde** | Kunde einer Organisation | Projektdaten |
| **Abonnement/Plan** | Vertrag Sysadmin ↔ Organisation | Billing |
| **Auftrag** | Vertrag Organisation ↔ Endkunde | Projekte |

---

# Teil 1: Landing Page (Nicht angemeldet)

## Seitenstruktur

```
Landing Page (/)
├── Header (Navigation)
├── Hero Section
├── Features Section
├── Pricing Section
├── Testimonials
├── FAQ
├── CTA Section
└── Footer

Weitere öffentliche Seiten:
├── /features - Detaillierte Features
├── /pricing - Preisübersicht
├── /about - Über uns
├── /contact - Kontakt
├── /demo - Demo anfordern
├── /login - Anmeldung
├── /register - Registrierung
├── /legal/impressum
├── /legal/datenschutz
└── /legal/agb
```

## Landing Page HTML

```html
<!-- /templates/public/landing.html -->
<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="EnergieberaterPRO - Die professionelle Software für Energieberater. Berechnungen, Berichte, Kundenverwaltung in einer Plattform.">
    <title>EnergieberaterPRO - Software für Energieberater</title>
    <link rel="stylesheet" href="/static/css/landing.css">
    <link rel="icon" href="/static/favicon.ico">
</head>
<body>
    <!-- Header -->
    <header class="landing-header">
        <div class="container">
            <a href="/" class="logo">
                <img src="/static/img/logo.svg" alt="EnergieberaterPRO">
            </a>

            <nav class="main-nav">
                <a href="#features">Features</a>
                <a href="#pricing">Preise</a>
                <a href="/about">Über uns</a>
                <a href="/contact">Kontakt</a>
            </nav>

            <div class="header-actions">
                <a href="/login" class="btn btn-ghost">Anmelden</a>
                <a href="/register" class="btn btn-primary">Kostenlos testen</a>
            </div>

            <button class="mobile-menu-toggle" aria-label="Menü öffnen">
                <span></span><span></span><span></span>
            </button>
        </div>
    </header>

    <!-- Hero Section -->
    <section class="hero">
        <div class="container">
            <div class="hero-content">
                <h1>Die All-in-One Software für <span class="highlight">Energieberater</span></h1>
                <p class="hero-subtitle">
                    Berechnungen nach DIN V 18599, automatische Berichte, Kundenverwaltung
                    und intelligente Workflows – alles in einer Plattform.
                </p>
                <div class="hero-cta">
                    <a href="/register" class="btn btn-primary btn-lg">
                        14 Tage kostenlos testen
                    </a>
                    <a href="/demo" class="btn btn-outline btn-lg">
                        <svg class="icon"><use href="#icon-play"/></svg>
                        Demo ansehen
                    </a>
                </div>
                <p class="hero-note">Keine Kreditkarte erforderlich • Sofort startklar</p>
            </div>
            <div class="hero-image">
                <img src="/static/img/landing/hero-dashboard.png"
                     alt="EnergieberaterPRO Dashboard"
                     loading="eager">
            </div>
        </div>
    </section>

    <!-- Trust Badges -->
    <section class="trust-badges">
        <div class="container">
            <p>Vertraut von über <strong>500+ Energieberatern</strong> in Deutschland</p>
            <div class="badges">
                <img src="/static/img/badges/dena.svg" alt="dena">
                <img src="/static/img/badges/bafa.svg" alt="BAFA">
                <img src="/static/img/badges/tuv.svg" alt="TÜV geprüft">
                <img src="/static/img/badges/dsgvo.svg" alt="DSGVO konform">
            </div>
        </div>
    </section>

    <!-- Problem/Solution -->
    <section class="problem-solution">
        <div class="container">
            <div class="problem">
                <h2>Kennen Sie das?</h2>
                <ul class="problem-list">
                    <li>
                        <svg class="icon text-danger"><use href="#icon-x"/></svg>
                        Stundenlange manuelle Berechnungen
                    </li>
                    <li>
                        <svg class="icon text-danger"><use href="#icon-x"/></svg>
                        Excel-Chaos und verstreute Dokumente
                    </li>
                    <li>
                        <svg class="icon text-danger"><use href="#icon-x"/></svg>
                        Zeitaufwändige Berichtserstellung
                    </li>
                    <li>
                        <svg class="icon text-danger"><use href="#icon-x"/></svg>
                        Komplizierte Zeiterfassung
                    </li>
                </ul>
            </div>
            <div class="solution">
                <h2>Mit EnergieberaterPRO:</h2>
                <ul class="solution-list">
                    <li>
                        <svg class="icon text-success"><use href="#icon-check"/></svg>
                        Automatische Berechnungen nach allen Normen
                    </li>
                    <li>
                        <svg class="icon text-success"><use href="#icon-check"/></svg>
                        Alles an einem Ort, perfekt organisiert
                    </li>
                    <li>
                        <svg class="icon text-success"><use href="#icon-check"/></svg>
                        Berichte per Klick generieren
                    </li>
                    <li>
                        <svg class="icon text-success"><use href="#icon-check"/></svg>
                        0% Bürokratie durch Automatisierung
                    </li>
                </ul>
            </div>
        </div>
    </section>

    <!-- Features Section -->
    <section id="features" class="features">
        <div class="container">
            <div class="section-header">
                <h2>Alles, was Sie brauchen</h2>
                <p>Professionelle Werkzeuge für Ihre tägliche Arbeit</p>
            </div>

            <div class="features-grid">
                <div class="feature-card">
                    <div class="feature-icon bg-blue">
                        <svg><use href="#icon-calculator"/></svg>
                    </div>
                    <h3>Normgerechte Berechnungen</h3>
                    <p>DIN V 18599, DIN EN 12831, GEG 2024 – alle Berechnungen nach aktuellen Normen, automatisch aktualisiert.</p>
                </div>

                <div class="feature-card">
                    <div class="feature-icon bg-green">
                        <svg><use href="#icon-file-text"/></svg>
                    </div>
                    <h3>Intelligente Berichte</h3>
                    <p>Erstellen Sie professionelle Energieausweise und Beratungsberichte mit nur wenigen Klicks.</p>
                </div>

                <div class="feature-card">
                    <div class="feature-icon bg-purple">
                        <svg><use href="#icon-users"/></svg>
                    </div>
                    <h3>Kundenverwaltung</h3>
                    <p>Behalten Sie alle Kunden, Projekte und Dokumente übersichtlich im Blick.</p>
                </div>

                <div class="feature-card">
                    <div class="feature-icon bg-yellow">
                        <svg><use href="#icon-clock"/></svg>
                    </div>
                    <h3>Automatische Zeiterfassung</h3>
                    <p>Keine manuelle Eingabe mehr – die Zeit wird automatisch erfasst, während Sie arbeiten.</p>
                </div>

                <div class="feature-card">
                    <div class="feature-icon bg-red">
                        <svg><use href="#icon-sparkles"/></svg>
                    </div>
                    <h3>KI-Assistent</h3>
                    <p>Ihr persönlicher Helfer für Normen, Berechnungen und Textvorschläge.</p>
                </div>

                <div class="feature-card">
                    <div class="feature-icon bg-teal">
                        <svg><use href="#icon-shield"/></svg>
                    </div>
                    <h3>DSGVO-konform</h3>
                    <p>Ihre Daten sind sicher – gehostet in Deutschland mit höchsten Sicherheitsstandards.</p>
                </div>
            </div>

            <div class="features-cta">
                <a href="/features" class="btn btn-outline">
                    Alle Features entdecken →
                </a>
            </div>
        </div>
    </section>

    <!-- How It Works -->
    <section class="how-it-works">
        <div class="container">
            <div class="section-header">
                <h2>So einfach geht's</h2>
                <p>In 3 Schritten zur professionellen Energieberatung</p>
            </div>

            <div class="steps">
                <div class="step">
                    <div class="step-number">1</div>
                    <div class="step-content">
                        <h3>Registrieren</h3>
                        <p>Erstellen Sie Ihr Konto in weniger als 2 Minuten. Keine Kreditkarte erforderlich.</p>
                    </div>
                </div>

                <div class="step-connector"></div>

                <div class="step">
                    <div class="step-number">2</div>
                    <div class="step-content">
                        <h3>Projekt anlegen</h3>
                        <p>Erfassen Sie Gebäudedaten und lassen Sie Berechnungen automatisch durchführen.</p>
                    </div>
                </div>

                <div class="step-connector"></div>

                <div class="step">
                    <div class="step-number">3</div>
                    <div class="step-content">
                        <h3>Bericht erstellen</h3>
                        <p>Generieren Sie professionelle Berichte und Energieausweise per Klick.</p>
                    </div>
                </div>
            </div>
        </div>
    </section>

    <!-- Pricing Section -->
    <section id="pricing" class="pricing">
        <div class="container">
            <div class="section-header">
                <h2>Transparente Preise</h2>
                <p>Wählen Sie den Plan, der zu Ihrem Büro passt</p>
            </div>

            <div class="billing-toggle">
                <span :class="{ 'active': !annual }">Monatlich</span>
                <button class="toggle" @click="annual = !annual" :class="{ 'active': annual }">
                    <span class="toggle-slider"></span>
                </button>
                <span :class="{ 'active': annual }">Jährlich <span class="discount">-20%</span></span>
            </div>

            <div class="pricing-grid" x-data="{ annual: true }">
                <!-- Starter Plan -->
                <div class="pricing-card">
                    <div class="pricing-header">
                        <h3>Starter</h3>
                        <p class="pricing-description">Für Einzelberater</p>
                    </div>
                    <div class="pricing-price">
                        <span class="price" x-text="annual ? '39' : '49'">39</span>
                        <span class="currency">€</span>
                        <span class="period">/Monat</span>
                    </div>
                    <ul class="pricing-features">
                        <li><svg><use href="#icon-check"/></svg> 1 Benutzer</li>
                        <li><svg><use href="#icon-check"/></svg> 50 Projekte/Jahr</li>
                        <li><svg><use href="#icon-check"/></svg> Alle Berechnungen</li>
                        <li><svg><use href="#icon-check"/></svg> Basis-Berichte</li>
                        <li><svg><use href="#icon-check"/></svg> 5 GB Speicher</li>
                        <li><svg><use href="#icon-check"/></svg> E-Mail-Support</li>
                    </ul>
                    <a href="/register?plan=starter" class="btn btn-outline btn-block">
                        Kostenlos testen
                    </a>
                </div>

                <!-- Professional Plan -->
                <div class="pricing-card featured">
                    <div class="pricing-badge">Beliebt</div>
                    <div class="pricing-header">
                        <h3>Professional</h3>
                        <p class="pricing-description">Für kleine Teams</p>
                    </div>
                    <div class="pricing-price">
                        <span class="price" x-text="annual ? '79' : '99'">79</span>
                        <span class="currency">€</span>
                        <span class="period">/Monat</span>
                    </div>
                    <ul class="pricing-features">
                        <li><svg><use href="#icon-check"/></svg> Bis 5 Benutzer</li>
                        <li><svg><use href="#icon-check"/></svg> Unbegrenzte Projekte</li>
                        <li><svg><use href="#icon-check"/></svg> Alle Berechnungen</li>
                        <li><svg><use href="#icon-check"/></svg> Premium-Berichte</li>
                        <li><svg><use href="#icon-check"/></svg> 50 GB Speicher</li>
                        <li><svg><use href="#icon-check"/></svg> KI-Assistent</li>
                        <li><svg><use href="#icon-check"/></svg> Prioritäts-Support</li>
                    </ul>
                    <a href="/register?plan=professional" class="btn btn-primary btn-block">
                        Kostenlos testen
                    </a>
                </div>

                <!-- Enterprise Plan -->
                <div class="pricing-card">
                    <div class="pricing-header">
                        <h3>Enterprise</h3>
                        <p class="pricing-description">Für große Büros</p>
                    </div>
                    <div class="pricing-price">
                        <span class="price" x-text="annual ? '149' : '189'">149</span>
                        <span class="currency">€</span>
                        <span class="period">/Monat</span>
                    </div>
                    <ul class="pricing-features">
                        <li><svg><use href="#icon-check"/></svg> Unbegrenzte Benutzer</li>
                        <li><svg><use href="#icon-check"/></svg> Unbegrenzte Projekte</li>
                        <li><svg><use href="#icon-check"/></svg> Alle Berechnungen</li>
                        <li><svg><use href="#icon-check"/></svg> White-Label Berichte</li>
                        <li><svg><use href="#icon-check"/></svg> 500 GB Speicher</li>
                        <li><svg><use href="#icon-check"/></svg> KI-Assistent Premium</li>
                        <li><svg><use href="#icon-check"/></svg> Dedizierter Support</li>
                        <li><svg><use href="#icon-check"/></svg> API-Zugang</li>
                    </ul>
                    <a href="/contact?subject=enterprise" class="btn btn-outline btn-block">
                        Kontakt aufnehmen
                    </a>
                </div>
            </div>

            <p class="pricing-note">
                Alle Preise zzgl. MwSt. • 14 Tage kostenlos testen • Jederzeit kündbar
            </p>
        </div>
    </section>

    <!-- Testimonials -->
    <section class="testimonials">
        <div class="container">
            <div class="section-header">
                <h2>Das sagen unsere Kunden</h2>
            </div>

            <div class="testimonials-grid">
                <div class="testimonial-card">
                    <div class="testimonial-content">
                        <p>"Mit EnergieberaterPRO spare ich pro Woche mindestens 10 Stunden. Die automatische Zeiterfassung ist ein Gamechanger!"</p>
                    </div>
                    <div class="testimonial-author">
                        <img src="/static/img/testimonials/person1.jpg" alt="">
                        <div>
                            <strong>Thomas Müller</strong>
                            <span>Energieberater, München</span>
                        </div>
                    </div>
                </div>

                <div class="testimonial-card">
                    <div class="testimonial-content">
                        <p>"Endlich eine Software, die wirklich für Energieberater gemacht ist. Die Berechnungen sind immer normkonform und aktuell."</p>
                    </div>
                    <div class="testimonial-author">
                        <img src="/static/img/testimonials/person2.jpg" alt="">
                        <div>
                            <strong>Sarah Weber</strong>
                            <span>Ing.-Büro Weber, Hamburg</span>
                        </div>
                    </div>
                </div>

                <div class="testimonial-card">
                    <div class="testimonial-content">
                        <p>"Der KI-Assistent hilft mir bei komplexen Normen und spart mir stundenlange Recherche. Absolut empfehlenswert!"</p>
                    </div>
                    <div class="testimonial-author">
                        <img src="/static/img/testimonials/person3.jpg" alt="">
                        <div>
                            <strong>Michael Schmidt</strong>
                            <span>Energie-Consulting Schmidt</span>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </section>

    <!-- FAQ -->
    <section class="faq">
        <div class="container">
            <div class="section-header">
                <h2>Häufig gestellte Fragen</h2>
            </div>

            <div class="faq-list" x-data="{ open: null }">
                <div class="faq-item">
                    <button class="faq-question" @click="open = open === 1 ? null : 1">
                        <span>Kann ich EnergieberaterPRO kostenlos testen?</span>
                        <svg class="icon" :class="{ 'rotated': open === 1 }"><use href="#icon-chevron-down"/></svg>
                    </button>
                    <div class="faq-answer" x-show="open === 1" x-collapse>
                        <p>Ja! Sie können alle Funktionen 14 Tage lang kostenlos und unverbindlich testen. Es ist keine Kreditkarte erforderlich.</p>
                    </div>
                </div>

                <div class="faq-item">
                    <button class="faq-question" @click="open = open === 2 ? null : 2">
                        <span>Sind meine Daten sicher?</span>
                        <svg class="icon" :class="{ 'rotated': open === 2 }"><use href="#icon-chevron-down"/></svg>
                    </button>
                    <div class="faq-answer" x-show="open === 2" x-collapse>
                        <p>Absolut. Ihre Daten werden in deutschen Rechenzentren gehostet und sind DSGVO-konform. Wir verwenden modernste Verschlüsselung und regelmäßige Backups.</p>
                    </div>
                </div>

                <div class="faq-item">
                    <button class="faq-question" @click="open = open === 3 ? null : 3">
                        <span>Welche Normen werden unterstützt?</span>
                        <svg class="icon" :class="{ 'rotated': open === 3 }"><use href="#icon-chevron-down"/></svg>
                    </button>
                    <div class="faq-answer" x-show="open === 3" x-collapse>
                        <p>Wir unterstützen alle relevanten Normen: DIN V 18599, DIN EN 12831, GEG 2024, DIN V 4108-6 und viele mehr. Die Normen werden regelmäßig aktualisiert.</p>
                    </div>
                </div>

                <div class="faq-item">
                    <button class="faq-question" @click="open = open === 4 ? null : 4">
                        <span>Kann ich jederzeit kündigen?</span>
                        <svg class="icon" :class="{ 'rotated': open === 4 }"><use href="#icon-chevron-down"/></svg>
                    </button>
                    <div class="faq-answer" x-show="open === 4" x-collapse>
                        <p>Ja, Sie können monatliche Abonnements jederzeit zum Monatsende kündigen. Bei jährlicher Zahlung ist eine Kündigung zum Ablauf des Jahres möglich.</p>
                    </div>
                </div>

                <div class="faq-item">
                    <button class="faq-question" @click="open = open === 5 ? null : 5">
                        <span>Gibt es eine Einrichtungsgebühr?</span>
                        <svg class="icon" :class="{ 'rotated': open === 5 }"><use href="#icon-chevron-down"/></svg>
                    </button>
                    <div class="faq-answer" x-show="open === 5" x-collapse>
                        <p>Nein, es gibt keine versteckten Kosten oder Einrichtungsgebühren. Sie zahlen nur den monatlichen oder jährlichen Preis.</p>
                    </div>
                </div>
            </div>
        </div>
    </section>

    <!-- CTA Section -->
    <section class="cta-section">
        <div class="container">
            <div class="cta-content">
                <h2>Bereit, effizienter zu arbeiten?</h2>
                <p>Starten Sie noch heute Ihre kostenlose Testversion.</p>
                <div class="cta-buttons">
                    <a href="/register" class="btn btn-white btn-lg">
                        Kostenlos testen
                    </a>
                    <a href="/demo" class="btn btn-outline-white btn-lg">
                        Demo vereinbaren
                    </a>
                </div>
            </div>
        </div>
    </section>

    <!-- Footer -->
    <footer class="landing-footer">
        <div class="container">
            <div class="footer-grid">
                <div class="footer-brand">
                    <img src="/static/img/logo-white.svg" alt="EnergieberaterPRO">
                    <p>Die professionelle Software für Energieberater in Deutschland.</p>
                    <div class="social-links">
                        <a href="#" aria-label="LinkedIn"><svg><use href="#icon-linkedin"/></svg></a>
                        <a href="#" aria-label="Twitter"><svg><use href="#icon-twitter"/></svg></a>
                        <a href="#" aria-label="YouTube"><svg><use href="#icon-youtube"/></svg></a>
                    </div>
                </div>

                <div class="footer-links">
                    <h4>Produkt</h4>
                    <ul>
                        <li><a href="/features">Features</a></li>
                        <li><a href="/pricing">Preise</a></li>
                        <li><a href="/demo">Demo</a></li>
                        <li><a href="/changelog">Updates</a></li>
                    </ul>
                </div>

                <div class="footer-links">
                    <h4>Ressourcen</h4>
                    <ul>
                        <li><a href="/help">Hilfe-Center</a></li>
                        <li><a href="/blog">Blog</a></li>
                        <li><a href="/webinars">Webinare</a></li>
                        <li><a href="/api">API-Dokumentation</a></li>
                    </ul>
                </div>

                <div class="footer-links">
                    <h4>Unternehmen</h4>
                    <ul>
                        <li><a href="/about">Über uns</a></li>
                        <li><a href="/careers">Karriere</a></li>
                        <li><a href="/contact">Kontakt</a></li>
                        <li><a href="/partner">Partner werden</a></li>
                    </ul>
                </div>

                <div class="footer-links">
                    <h4>Rechtliches</h4>
                    <ul>
                        <li><a href="/legal/impressum">Impressum</a></li>
                        <li><a href="/legal/datenschutz">Datenschutz</a></li>
                        <li><a href="/legal/agb">AGB</a></li>
                        <li><a href="/legal/cookies">Cookie-Einstellungen</a></li>
                    </ul>
                </div>
            </div>

            <div class="footer-bottom">
                <p>&copy; 2025 EnergieberaterPRO. Alle Rechte vorbehalten.</p>
                <div class="footer-badges">
                    <img src="/static/img/badges/ssl.svg" alt="SSL verschlüsselt">
                    <img src="/static/img/badges/hosted-germany.svg" alt="Gehostet in Deutschland">
                </div>
            </div>
        </div>
    </footer>

    <!-- Scripts -->
    <script src="/static/js/alpine.min.js" defer></script>
    <script src="/static/js/landing.js" defer></script>
</body>
</html>
```

## Landing Page CSS

```css
/* /static/css/landing.css */

/* Variables */
:root {
    --color-primary: #2563eb;
    --color-primary-dark: #1d4ed8;
    --color-secondary: #10b981;
    --color-text: #1f2937;
    --color-text-light: #6b7280;
    --color-bg: #ffffff;
    --color-bg-alt: #f9fafb;
    --color-border: #e5e7eb;
    --font-sans: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
    --shadow-sm: 0 1px 2px rgba(0,0,0,0.05);
    --shadow-md: 0 4px 6px rgba(0,0,0,0.1);
    --shadow-lg: 0 10px 15px rgba(0,0,0,0.1);
    --radius: 0.5rem;
    --radius-lg: 1rem;
}

/* Reset & Base */
*, *::before, *::after {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
}

html {
    scroll-behavior: smooth;
}

body {
    font-family: var(--font-sans);
    color: var(--color-text);
    line-height: 1.6;
    background: var(--color-bg);
}

.container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 1.5rem;
}

/* Typography */
h1, h2, h3, h4 {
    font-weight: 700;
    line-height: 1.2;
}

h1 { font-size: clamp(2rem, 5vw, 3.5rem); }
h2 { font-size: clamp(1.5rem, 4vw, 2.5rem); }
h3 { font-size: 1.25rem; }

/* Buttons */
.btn {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: 0.5rem;
    padding: 0.75rem 1.5rem;
    border-radius: var(--radius);
    font-weight: 600;
    font-size: 1rem;
    text-decoration: none;
    transition: all 0.2s;
    cursor: pointer;
    border: none;
}

.btn-primary {
    background: var(--color-primary);
    color: white;
}

.btn-primary:hover {
    background: var(--color-primary-dark);
}

.btn-outline {
    background: transparent;
    border: 2px solid var(--color-primary);
    color: var(--color-primary);
}

.btn-outline:hover {
    background: var(--color-primary);
    color: white;
}

.btn-ghost {
    background: transparent;
    color: var(--color-text);
}

.btn-ghost:hover {
    background: var(--color-bg-alt);
}

.btn-white {
    background: white;
    color: var(--color-primary);
}

.btn-outline-white {
    background: transparent;
    border: 2px solid white;
    color: white;
}

.btn-lg {
    padding: 1rem 2rem;
    font-size: 1.125rem;
}

.btn-block {
    width: 100%;
}

/* Header */
.landing-header {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    z-index: 1000;
    background: rgba(255, 255, 255, 0.95);
    backdrop-filter: blur(10px);
    border-bottom: 1px solid var(--color-border);
}

.landing-header .container {
    display: flex;
    align-items: center;
    justify-content: space-between;
    height: 70px;
}

.landing-header .logo img {
    height: 36px;
}

.main-nav {
    display: flex;
    gap: 2rem;
}

.main-nav a {
    color: var(--color-text);
    text-decoration: none;
    font-weight: 500;
    transition: color 0.2s;
}

.main-nav a:hover {
    color: var(--color-primary);
}

.header-actions {
    display: flex;
    gap: 0.75rem;
}

.mobile-menu-toggle {
    display: none;
}

/* Hero */
.hero {
    padding: 8rem 0 4rem;
    background: linear-gradient(135deg, #f0f9ff 0%, #e0f2fe 100%);
}

.hero .container {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 4rem;
    align-items: center;
}

.hero-content h1 {
    margin-bottom: 1.5rem;
}

.hero-content .highlight {
    color: var(--color-primary);
}

.hero-subtitle {
    font-size: 1.25rem;
    color: var(--color-text-light);
    margin-bottom: 2rem;
}

.hero-cta {
    display: flex;
    gap: 1rem;
    margin-bottom: 1rem;
}

.hero-note {
    font-size: 0.875rem;
    color: var(--color-text-light);
}

.hero-image img {
    width: 100%;
    border-radius: var(--radius-lg);
    box-shadow: var(--shadow-lg);
}

/* Trust Badges */
.trust-badges {
    padding: 3rem 0;
    background: white;
    text-align: center;
}

.trust-badges p {
    color: var(--color-text-light);
    margin-bottom: 1.5rem;
}

.badges {
    display: flex;
    justify-content: center;
    gap: 3rem;
    flex-wrap: wrap;
}

.badges img {
    height: 40px;
    opacity: 0.6;
    filter: grayscale(100%);
    transition: all 0.3s;
}

.badges img:hover {
    opacity: 1;
    filter: grayscale(0%);
}

/* Section Headers */
.section-header {
    text-align: center;
    max-width: 600px;
    margin: 0 auto 3rem;
}

.section-header h2 {
    margin-bottom: 0.75rem;
}

.section-header p {
    color: var(--color-text-light);
    font-size: 1.125rem;
}

/* Problem/Solution */
.problem-solution {
    padding: 5rem 0;
}

.problem-solution .container {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 4rem;
}

.problem-list, .solution-list {
    list-style: none;
}

.problem-list li, .solution-list li {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    padding: 0.75rem 0;
    font-size: 1.125rem;
}

.problem-list .icon { color: #ef4444; }
.solution-list .icon { color: #10b981; }

/* Features */
.features {
    padding: 5rem 0;
    background: var(--color-bg-alt);
}

.features-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 2rem;
}

.feature-card {
    background: white;
    padding: 2rem;
    border-radius: var(--radius-lg);
    box-shadow: var(--shadow-sm);
    transition: transform 0.3s, box-shadow 0.3s;
}

.feature-card:hover {
    transform: translateY(-5px);
    box-shadow: var(--shadow-md);
}

.feature-icon {
    width: 56px;
    height: 56px;
    border-radius: var(--radius);
    display: flex;
    align-items: center;
    justify-content: center;
    margin-bottom: 1.25rem;
}

.feature-icon svg {
    width: 28px;
    height: 28px;
    color: white;
}

.feature-icon.bg-blue { background: #2563eb; }
.feature-icon.bg-green { background: #10b981; }
.feature-icon.bg-purple { background: #8b5cf6; }
.feature-icon.bg-yellow { background: #f59e0b; }
.feature-icon.bg-red { background: #ef4444; }
.feature-icon.bg-teal { background: #14b8a6; }

.feature-card h3 {
    margin-bottom: 0.75rem;
}

.feature-card p {
    color: var(--color-text-light);
}

.features-cta {
    text-align: center;
    margin-top: 3rem;
}

/* How It Works */
.how-it-works {
    padding: 5rem 0;
}

.steps {
    display: flex;
    justify-content: center;
    align-items: flex-start;
    gap: 1rem;
}

.step {
    text-align: center;
    max-width: 280px;
}

.step-number {
    width: 64px;
    height: 64px;
    border-radius: 50%;
    background: var(--color-primary);
    color: white;
    font-size: 1.5rem;
    font-weight: 700;
    display: flex;
    align-items: center;
    justify-content: center;
    margin: 0 auto 1.5rem;
}

.step-connector {
    width: 80px;
    height: 2px;
    background: var(--color-border);
    margin-top: 32px;
}

.step h3 {
    margin-bottom: 0.5rem;
}

.step p {
    color: var(--color-text-light);
}

/* Pricing */
.pricing {
    padding: 5rem 0;
    background: var(--color-bg-alt);
}

.billing-toggle {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 1rem;
    margin-bottom: 3rem;
}

.toggle {
    width: 56px;
    height: 28px;
    background: var(--color-border);
    border-radius: 14px;
    border: none;
    cursor: pointer;
    position: relative;
    transition: background 0.3s;
}

.toggle.active {
    background: var(--color-primary);
}

.toggle-slider {
    position: absolute;
    top: 2px;
    left: 2px;
    width: 24px;
    height: 24px;
    background: white;
    border-radius: 50%;
    transition: transform 0.3s;
}

.toggle.active .toggle-slider {
    transform: translateX(28px);
}

.discount {
    background: #dcfce7;
    color: #16a34a;
    padding: 0.125rem 0.5rem;
    border-radius: 9999px;
    font-size: 0.75rem;
    font-weight: 600;
}

.pricing-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 2rem;
    max-width: 1000px;
    margin: 0 auto;
}

.pricing-card {
    background: white;
    border-radius: var(--radius-lg);
    padding: 2rem;
    box-shadow: var(--shadow-sm);
    position: relative;
}

.pricing-card.featured {
    border: 2px solid var(--color-primary);
    transform: scale(1.05);
    box-shadow: var(--shadow-lg);
}

.pricing-badge {
    position: absolute;
    top: -12px;
    left: 50%;
    transform: translateX(-50%);
    background: var(--color-primary);
    color: white;
    padding: 0.25rem 1rem;
    border-radius: 9999px;
    font-size: 0.75rem;
    font-weight: 600;
}

.pricing-header {
    text-align: center;
    margin-bottom: 1.5rem;
}

.pricing-description {
    color: var(--color-text-light);
}

.pricing-price {
    text-align: center;
    margin-bottom: 2rem;
}

.pricing-price .price {
    font-size: 3rem;
    font-weight: 700;
}

.pricing-price .currency {
    font-size: 1.5rem;
    vertical-align: super;
}

.pricing-price .period {
    color: var(--color-text-light);
}

.pricing-features {
    list-style: none;
    margin-bottom: 2rem;
}

.pricing-features li {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    padding: 0.5rem 0;
}

.pricing-features svg {
    width: 20px;
    height: 20px;
    color: var(--color-secondary);
}

.pricing-note {
    text-align: center;
    color: var(--color-text-light);
    margin-top: 2rem;
}

/* Testimonials */
.testimonials {
    padding: 5rem 0;
}

.testimonials-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 2rem;
}

.testimonial-card {
    background: white;
    padding: 2rem;
    border-radius: var(--radius-lg);
    box-shadow: var(--shadow-sm);
}

.testimonial-content p {
    font-style: italic;
    margin-bottom: 1.5rem;
}

.testimonial-author {
    display: flex;
    align-items: center;
    gap: 1rem;
}

.testimonial-author img {
    width: 48px;
    height: 48px;
    border-radius: 50%;
    object-fit: cover;
}

.testimonial-author strong {
    display: block;
}

.testimonial-author span {
    color: var(--color-text-light);
    font-size: 0.875rem;
}

/* FAQ */
.faq {
    padding: 5rem 0;
    background: var(--color-bg-alt);
}

.faq-list {
    max-width: 800px;
    margin: 0 auto;
}

.faq-item {
    background: white;
    border-radius: var(--radius);
    margin-bottom: 1rem;
    overflow: hidden;
}

.faq-question {
    width: 100%;
    padding: 1.25rem 1.5rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
    background: none;
    border: none;
    font-size: 1rem;
    font-weight: 600;
    text-align: left;
    cursor: pointer;
}

.faq-question .icon {
    width: 20px;
    height: 20px;
    transition: transform 0.3s;
}

.faq-question .icon.rotated {
    transform: rotate(180deg);
}

.faq-answer {
    padding: 0 1.5rem 1.25rem;
    color: var(--color-text-light);
}

/* CTA Section */
.cta-section {
    padding: 5rem 0;
    background: linear-gradient(135deg, var(--color-primary) 0%, var(--color-primary-dark) 100%);
    color: white;
    text-align: center;
}

.cta-content h2 {
    margin-bottom: 0.75rem;
}

.cta-content p {
    opacity: 0.9;
    margin-bottom: 2rem;
}

.cta-buttons {
    display: flex;
    justify-content: center;
    gap: 1rem;
}

/* Footer */
.landing-footer {
    background: #111827;
    color: #9ca3af;
    padding: 4rem 0 2rem;
}

.footer-grid {
    display: grid;
    grid-template-columns: 2fr 1fr 1fr 1fr 1fr;
    gap: 2rem;
    margin-bottom: 3rem;
}

.footer-brand img {
    height: 32px;
    margin-bottom: 1rem;
}

.footer-brand p {
    margin-bottom: 1.5rem;
}

.social-links {
    display: flex;
    gap: 1rem;
}

.social-links a {
    width: 36px;
    height: 36px;
    border-radius: 50%;
    background: rgba(255,255,255,0.1);
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    transition: background 0.3s;
}

.social-links a:hover {
    background: var(--color-primary);
}

.footer-links h4 {
    color: white;
    margin-bottom: 1rem;
}

.footer-links ul {
    list-style: none;
}

.footer-links li {
    margin-bottom: 0.5rem;
}

.footer-links a {
    color: #9ca3af;
    text-decoration: none;
    transition: color 0.2s;
}

.footer-links a:hover {
    color: white;
}

.footer-bottom {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-top: 2rem;
    border-top: 1px solid rgba(255,255,255,0.1);
}

.footer-badges {
    display: flex;
    gap: 1rem;
}

.footer-badges img {
    height: 28px;
}

/* Responsive */
@media (max-width: 1024px) {
    .hero .container,
    .problem-solution .container {
        grid-template-columns: 1fr;
        text-align: center;
    }

    .hero-cta {
        justify-content: center;
    }

    .features-grid,
    .testimonials-grid {
        grid-template-columns: repeat(2, 1fr);
    }

    .pricing-grid {
        grid-template-columns: 1fr;
        max-width: 400px;
    }

    .pricing-card.featured {
        transform: none;
    }

    .footer-grid {
        grid-template-columns: repeat(2, 1fr);
    }
}

@media (max-width: 768px) {
    .main-nav, .header-actions {
        display: none;
    }

    .mobile-menu-toggle {
        display: block;
    }

    .hero {
        padding: 6rem 0 3rem;
    }

    .features-grid,
    .testimonials-grid {
        grid-template-columns: 1fr;
    }

    .steps {
        flex-direction: column;
        align-items: center;
    }

    .step-connector {
        width: 2px;
        height: 40px;
        margin: 0;
    }

    .cta-buttons {
        flex-direction: column;
    }

    .footer-grid {
        grid-template-columns: 1fr;
        text-align: center;
    }

    .social-links {
        justify-content: center;
    }

    .footer-bottom {
        flex-direction: column;
        gap: 1rem;
        text-align: center;
    }
}
```

---

# Teil 2: Registrierung & Login

```html
<!-- /templates/public/register.html -->
<div class="auth-page">
    <div class="auth-container">
        <div class="auth-card">
            <div class="auth-header">
                <a href="/" class="logo">
                    <img src="/static/img/logo.svg" alt="EnergieberaterPRO">
                </a>
                <h1>Kostenlos registrieren</h1>
                <p>Starten Sie Ihre 14-tägige Testversion</p>
            </div>

            <form class="auth-form" x-data="registerForm()" @submit.prevent="submit">
                <!-- Step 1: Account -->
                <div class="form-step" x-show="step === 1">
                    <div class="form-group">
                        <label for="email">E-Mail-Adresse</label>
                        <input type="email" id="email" x-model="form.email" required
                               placeholder="ihre@email.de">
                        <span class="error" x-show="errors.email" x-text="errors.email"></span>
                    </div>

                    <div class="form-group">
                        <label for="password">Passwort</label>
                        <input type="password" id="password" x-model="form.password" required
                               placeholder="Mindestens 8 Zeichen">
                        <div class="password-strength">
                            <div class="strength-bar" :class="passwordStrength"></div>
                        </div>
                    </div>

                    <div class="form-group">
                        <label for="password_confirm">Passwort bestätigen</label>
                        <input type="password" id="password_confirm" x-model="form.password_confirm" required>
                    </div>

                    <button type="button" @click="nextStep" class="btn btn-primary btn-block">
                        Weiter
                    </button>
                </div>

                <!-- Step 2: Organization -->
                <div class="form-step" x-show="step === 2">
                    <div class="form-group">
                        <label for="org_name">Firmenname</label>
                        <input type="text" id="org_name" x-model="form.org_name" required
                               placeholder="Ihr Unternehmen">
                    </div>

                    <div class="form-group">
                        <label for="first_name">Vorname</label>
                        <input type="text" id="first_name" x-model="form.first_name" required>
                    </div>

                    <div class="form-group">
                        <label for="last_name">Nachname</label>
                        <input type="text" id="last_name" x-model="form.last_name" required>
                    </div>

                    <div class="form-group">
                        <label for="phone">Telefon (optional)</label>
                        <input type="tel" id="phone" x-model="form.phone">
                    </div>

                    <div class="form-row">
                        <button type="button" @click="prevStep" class="btn btn-outline">
                            Zurück
                        </button>
                        <button type="button" @click="nextStep" class="btn btn-primary">
                            Weiter
                        </button>
                    </div>
                </div>

                <!-- Step 3: Plan Selection -->
                <div class="form-step" x-show="step === 3">
                    <div class="plan-selection">
                        <label class="plan-option" :class="{ 'selected': form.plan === 'starter' }">
                            <input type="radio" name="plan" value="starter" x-model="form.plan">
                            <div class="plan-content">
                                <strong>Starter</strong>
                                <span>39€/Monat</span>
                                <span class="plan-features">1 Benutzer • 50 Projekte</span>
                            </div>
                        </label>

                        <label class="plan-option" :class="{ 'selected': form.plan === 'professional' }">
                            <input type="radio" name="plan" value="professional" x-model="form.plan">
                            <div class="plan-content">
                                <strong>Professional</strong>
                                <span>79€/Monat</span>
                                <span class="plan-features">5 Benutzer • Unbegrenzt</span>
                            </div>
                            <span class="plan-badge">Empfohlen</span>
                        </label>

                        <label class="plan-option" :class="{ 'selected': form.plan === 'enterprise' }">
                            <input type="radio" name="plan" value="enterprise" x-model="form.plan">
                            <div class="plan-content">
                                <strong>Enterprise</strong>
                                <span>149€/Monat</span>
                                <span class="plan-features">Unbegrenzt • API</span>
                            </div>
                        </label>
                    </div>

                    <div class="form-group">
                        <label class="checkbox-label">
                            <input type="checkbox" x-model="form.accept_terms" required>
                            <span>Ich akzeptiere die <a href="/legal/agb" target="_blank">AGB</a> und
                            <a href="/legal/datenschutz" target="_blank">Datenschutzerklärung</a></span>
                        </label>
                    </div>

                    <div class="form-row">
                        <button type="button" @click="prevStep" class="btn btn-outline">
                            Zurück
                        </button>
                        <button type="submit" class="btn btn-primary" :disabled="submitting">
                            <span x-show="!submitting">Registrieren</span>
                            <span x-show="submitting">Wird erstellt...</span>
                        </button>
                    </div>
                </div>
            </form>

            <div class="auth-footer">
                <p>Bereits ein Konto? <a href="/login">Anmelden</a></p>
            </div>
        </div>

        <div class="auth-features">
            <h2>Ihre Vorteile</h2>
            <ul>
                <li>
                    <svg><use href="#icon-check"/></svg>
                    14 Tage kostenlos testen
                </li>
                <li>
                    <svg><use href="#icon-check"/></svg>
                    Keine Kreditkarte erforderlich
                </li>
                <li>
                    <svg><use href="#icon-check"/></svg>
                    Sofort einsatzbereit
                </li>
                <li>
                    <svg><use href="#icon-check"/></svg>
                    Jederzeit kündbar
                </li>
            </ul>
        </div>
    </div>
</div>
```

```html
<!-- /templates/public/login.html -->
<div class="auth-page">
    <div class="auth-container login">
        <div class="auth-card">
            <div class="auth-header">
                <a href="/" class="logo">
                    <img src="/static/img/logo.svg" alt="EnergieberaterPRO">
                </a>
                <h1>Willkommen zurück</h1>
                <p>Melden Sie sich in Ihrem Konto an</p>
            </div>

            <form class="auth-form" x-data="loginForm()" @submit.prevent="submit">
                <div class="form-group">
                    <label for="email">E-Mail-Adresse</label>
                    <input type="email" id="email" x-model="email" required autofocus>
                </div>

                <div class="form-group">
                    <label for="password">
                        Passwort
                        <a href="/forgot-password" class="forgot-link">Vergessen?</a>
                    </label>
                    <input type="password" id="password" x-model="password" required>
                </div>

                <div class="form-group">
                    <label class="checkbox-label">
                        <input type="checkbox" x-model="remember">
                        <span>Angemeldet bleiben</span>
                    </label>
                </div>

                <div class="error-message" x-show="error" x-text="error"></div>

                <button type="submit" class="btn btn-primary btn-block" :disabled="submitting">
                    <span x-show="!submitting">Anmelden</span>
                    <span x-show="submitting">Wird angemeldet...</span>
                </button>
            </form>

            <div class="auth-footer">
                <p>Noch kein Konto? <a href="/register">Kostenlos registrieren</a></p>
            </div>
        </div>
    </div>
</div>
```
