# Open Source Lizenzen und Attribution

## Referenzen
- [Choose a License](https://choosealicense.com/licenses/)
- [Open Source License Comparison](https://www.mend.io/blog/open-source-licenses-comparison-guide/)
- [TechCrunch - Open Source Licenses](https://techcrunch.com/2025/01/12/open-source-licenses-everything-you-need-to-know/)

---

## 1. Lizenzstrategie

### 1.1 Grundsätze

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     OPEN SOURCE LIZENZSTRATEGIE                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  PRINZIP 1: Nur permissive Lizenzen für Kernkomponenten                    │
│  ─────────────────────────────────────────────────────────────────────────  │
│  • MIT, Apache 2.0, BSD - unbedenklich                                      │
│  • GPL/LGPL - nur isoliert verwenden                                        │
│  • AGPL - vermeiden (SaaS-Klausel)                                          │
│                                                                             │
│  PRINZIP 2: Keine proprietären Abhängigkeiten                               │
│  ─────────────────────────────────────────────────────────────────────────  │
│  • Alle Kernfunktionen mit Open Source umsetzbar                            │
│  • Keine Lock-in-Risiken                                                    │
│  • Selbst-Hosting jederzeit möglich                                         │
│                                                                             │
│  PRINZIP 3: Vollständige Attribution                                        │
│  ─────────────────────────────────────────────────────────────────────────  │
│  • LICENSES.md im Repository                                                │
│  • Attribution in der UI (Footer/About)                                     │
│  • SBOM (Software Bill of Materials)                                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Lizenztypen-Übersicht

| Lizenz | Typ | Kommerzielle Nutzung | Modifikation | Distribution | Patent Grant | Copyleft |
|--------|-----|---------------------|--------------|--------------|--------------|----------|
| MIT | Permissive | ✓ | ✓ | ✓ | ✗ | ✗ |
| Apache 2.0 | Permissive | ✓ | ✓ | ✓ | ✓ | ✗ |
| BSD 2/3 | Permissive | ✓ | ✓ | ✓ | ✗ | ✗ |
| PostgreSQL | Permissive | ✓ | ✓ | ✓ | ✗ | ✗ |
| LGPL | Weak Copyleft | ✓ | ✓ | ✓ | ✗ | Teilweise |
| GPL | Strong Copyleft | ✓ | ✓ | ✓ | ✗ | Ja |
| AGPL | Strong Copyleft | ✓ | ✓ | ✓ | ✗ | Ja (auch SaaS) |

---

## 2. Verwendete Komponenten

### 2.1 Backend

| Komponente | Version | Lizenz | Verwendung | Risiko |
|------------|---------|--------|------------|--------|
| Python | 3.11+ | PSF | Runtime | ✓ Kein |
| FastAPI | 0.109+ | MIT | Web Framework | ✓ Kein |
| SQLAlchemy | 2.0+ | MIT | ORM | ✓ Kein |
| Pydantic | 2.6+ | MIT | Validation | ✓ Kein |
| Alembic | 1.13+ | MIT | Migrations | ✓ Kein |
| Celery | 5.3+ | BSD | Task Queue | ✓ Kein |
| uvicorn | 0.27+ | BSD | ASGI Server | ✓ Kein |
| httpx | 0.26+ | BSD | HTTP Client | ✓ Kein |
| passlib | 1.7+ | BSD | Password Hashing | ✓ Kein |
| python-jose | 3.3+ | MIT | JWT | ✓ Kein |
| Jinja2 | 3.1+ | BSD | Templates | ✓ Kein |
| WeasyPrint | 60+ | BSD | PDF Generation | ✓ Kein |
| NumPy | 1.26+ | BSD | Calculations | ✓ Kein |
| Pandas | 2.2+ | BSD | Data Processing | ✓ Kein |

### 2.2 Frontend

| Komponente | Version | Lizenz | Verwendung | Risiko |
|------------|---------|--------|------------|--------|
| Alpine.js | 3.x | MIT | Reactivity | ✓ Kein |
| Tailwind CSS | 3.x | MIT | Styling | ✓ Kein |
| D3.js | 7.x | ISC | Visualizations | ✓ Kein |
| Chart.js | 4.x | MIT | Charts | ✓ Kein |
| Lucide Icons | Latest | ISC | Icons | ✓ Kein |
| Day.js | 1.x | MIT | Date/Time | ✓ Kein |
| PDF.js | 4.x | Apache 2.0 | PDF Viewer | ✓ Kein |
| Yjs | 13.x | MIT | CRDT/Collab | ✓ Kein |

### 2.3 Datenbanken & Services

| Komponente | Version | Lizenz | Verwendung | Risiko |
|------------|---------|--------|------------|--------|
| PostgreSQL | 16 | PostgreSQL | Database | ✓ Kein |
| Redis | 7 | BSD | Cache/Queue | ✓ Kein |
| MinIO | Latest | AGPL-3.0 | Object Storage | ⚠️ Prüfen |
| Ollama | Latest | MIT | LLM | ✓ Kein |
| ChromaDB | Latest | Apache 2.0 | Vector DB | ✓ Kein |

### 2.4 Infrastructure

| Komponente | Version | Lizenz | Verwendung | Risiko |
|------------|---------|--------|------------|--------|
| Docker | Latest | Apache 2.0 | Container | ✓ Kein |
| Caddy | 2.x | Apache 2.0 | Reverse Proxy | ✓ Kein |
| Prometheus | Latest | Apache 2.0 | Monitoring | ✓ Kein |
| Grafana | Latest | AGPL-3.0 | Dashboards | ⚠️ Prüfen |

---

## 3. Sonderfall AGPL (MinIO, Grafana)

### 3.1 AGPL-Implikationen

Die AGPL (Affero GPL) erfordert, dass bei Nutzung über ein Netzwerk (SaaS) der Quellcode verfügbar gemacht werden muss.

**Für EnergieberaterPRO:**
- MinIO und Grafana werden als **separate Services** betrieben
- Sie sind **nicht in die Hauptanwendung integriert**
- Der EnergieberaterPRO-Code bleibt **unabhängig**
- Bei Self-Hosting: Keine Auswirkungen
- Bei SaaS: MinIO/Grafana-Quellcode ist ohnehin öffentlich

### 3.2 Alternativen (falls nötig)

| AGPL-Komponente | Alternative | Lizenz |
|-----------------|-------------|--------|
| MinIO | SeaweedFS | Apache 2.0 |
| MinIO | LakeFS | Apache 2.0 |
| Grafana | Apache Superset | Apache 2.0 |
| Grafana | Metabase (CE) | AGPL (gleich) |

---

## 4. Attribution (LICENSES.md)

```markdown
# Third-Party Licenses

EnergieberaterPRO verwendet folgende Open-Source-Komponenten:

## Backend

### FastAPI
- License: MIT
- Copyright (c) 2018 Sebastián Ramírez
- https://github.com/tiangolo/fastapi

### SQLAlchemy
- License: MIT
- Copyright (c) 2005-2024 SQLAlchemy authors and contributors
- https://github.com/sqlalchemy/sqlalchemy

### Pydantic
- License: MIT
- Copyright (c) 2017-2024 Samuel Colvin and other contributors
- https://github.com/pydantic/pydantic

### Celery
- License: BSD 3-Clause
- Copyright (c) 2009-2024, Ask Solem & contributors
- https://github.com/celery/celery

### NumPy
- License: BSD 3-Clause
- Copyright (c) 2005-2024, NumPy Developers
- https://github.com/numpy/numpy

### WeasyPrint
- License: BSD 3-Clause
- Copyright (c) 2011-2024 Simon Sapin and contributors
- https://github.com/Kozea/WeasyPrint

## Frontend

### Alpine.js
- License: MIT
- Copyright (c) 2019-2024 Caleb Porzio
- https://github.com/alpinejs/alpine

### Tailwind CSS
- License: MIT
- Copyright (c) Tailwind Labs, Inc.
- https://github.com/tailwindlabs/tailwindcss

### D3.js
- License: ISC
- Copyright (c) 2010-2024 Mike Bostock
- https://github.com/d3/d3

### Yjs
- License: MIT
- Copyright (c) 2015-2024 Kevin Jahns
- https://github.com/yjs/yjs

### Lucide Icons
- License: ISC
- Copyright (c) 2020-2024 Lucide Contributors
- https://github.com/lucide-icons/lucide

## Databases

### PostgreSQL
- License: PostgreSQL License
- Copyright (c) 1996-2024 The PostgreSQL Global Development Group
- https://www.postgresql.org/about/licence/

### Redis
- License: BSD 3-Clause
- Copyright (c) 2006-2024, Salvatore Sanfilippo
- https://github.com/redis/redis

## Infrastructure

### Docker
- License: Apache 2.0
- Copyright (c) 2013-2024 Docker, Inc.
- https://github.com/docker/docker-ce

### Caddy
- License: Apache 2.0
- Copyright (c) 2015-2024 Light Code Labs
- https://github.com/caddyserver/caddy

---

The full text of each license can be found in the LICENSES/ directory.
```

---

## 5. SBOM (Software Bill of Materials)

### 5.1 Generierung

```bash
# CycloneDX SBOM generieren
pip install cyclonedx-bom
cyclonedx-py poetry -o sbom.json

# SPDX Format
pip install spdx-tools
# ...
```

### 5.2 CI/CD Integration

```yaml
# .github/workflows/sbom.yml
name: Generate SBOM

on:
  release:
    types: [published]

jobs:
  sbom:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Generate CycloneDX SBOM
        uses: CycloneDX/gh-python-generate-sbom@v2
        with:
          input: ./pyproject.toml
          output: ./sbom.json
          format: json

      - name: Upload SBOM
        uses: actions/upload-artifact@v4
        with:
          name: sbom
          path: sbom.json

      - name: Attach to Release
        uses: softprops/action-gh-release@v1
        with:
          files: sbom.json
```

---

## 6. Lizenzprüfung in CI/CD

```python
# scripts/check_licenses.py
"""Prüft Lizenzen aller Dependencies"""

ALLOWED_LICENSES = {
    'MIT', 'MIT License',
    'Apache 2.0', 'Apache Software License',
    'BSD', 'BSD License', 'BSD-2-Clause', 'BSD-3-Clause',
    'ISC', 'ISC License',
    'PSF', 'Python Software Foundation License',
    'PostgreSQL',
    'MPL 2.0', 'Mozilla Public License 2.0',
}

FORBIDDEN_LICENSES = {
    'GPL', 'GPLv2', 'GPLv3',
    'AGPL', 'AGPLv3',
    'SSPL',
    'Proprietary',
}

def check_licenses():
    import pkg_resources

    issues = []

    for pkg in pkg_resources.working_set:
        license_name = pkg.get_metadata('PKG-INFO').split('License: ')[1].split('\n')[0]

        if license_name in FORBIDDEN_LICENSES:
            issues.append(f"FORBIDDEN: {pkg.project_name} ({license_name})")
        elif license_name not in ALLOWED_LICENSES:
            issues.append(f"UNKNOWN: {pkg.project_name} ({license_name})")

    return issues

if __name__ == '__main__':
    issues = check_licenses()
    if issues:
        print("License issues found:")
        for issue in issues:
            print(f"  - {issue}")
        exit(1)
    print("All licenses OK")
```

---

## 7. UI Attribution

```html
<!-- Footer mit Attribution -->
<footer class="app-footer">
    <div class="footer-content">
        <p>EnergieberaterPRO &copy; 2025</p>
        <p class="text-sm text-gray-500">
            Powered by Open Source:
            <a href="/licenses">View Licenses</a>
        </p>
    </div>
</footer>

<!-- About/Info Modal -->
<div class="about-modal">
    <h2>EnergieberaterPRO</h2>
    <p>Version 1.0.0</p>

    <h3>Open Source</h3>
    <p>Diese Software verwendet Open-Source-Komponenten.</p>
    <a href="/licenses" class="btn">Lizenzen anzeigen</a>

    <h3>Credits</h3>
    <ul>
        <li>FastAPI - Sebastián Ramírez</li>
        <li>Alpine.js - Caleb Porzio</li>
        <li>Tailwind CSS - Tailwind Labs</li>
        <!-- ... -->
    </ul>
</div>
```

---

## 8. Checkliste

```
✓ Alle Dependencies geprüft
✓ Keine GPL/AGPL im Kern
✓ LICENSES.md erstellt
✓ SBOM generiert
✓ CI/CD Lizenzprüfung
✓ UI Attribution
✓ Dokumentation vollständig
```

---

*Letzte Aktualisierung: 2025-11-25*
*Version: 1.0.0*
