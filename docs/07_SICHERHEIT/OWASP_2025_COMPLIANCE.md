# OWASP Top 10 2025 Compliance

## Referenz
- [OWASP Top 10:2025 RC1](https://owasp.org/Top10/2025/0x00_2025-Introduction/)
- [OWASP Foundation](https://owasp.org/www-project-top-ten/)
- Veröffentlicht: 6. November 2025

---

## 1. A01:2025 - Broken Access Control

### 1.1 Risikobeschreibung
Broken Access Control bleibt an der Spitze der OWASP Top 10 2025 und betrifft durchschnittlich 3,73% der getesteten Anwendungen. Server-Side Request Forgery (SSRF) wurde in diese Kategorie integriert.

### 1.2 Maßnahmen

#### Row Level Security (PostgreSQL)
```sql
-- Aktiviere RLS für alle Mandanten-Tabellen
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;

-- Policy für Organisationsisolation
CREATE POLICY org_isolation ON projects
    USING (organization_id = current_setting('app.current_org')::uuid);

-- Policy für Benutzer
CREATE POLICY user_access ON projects
    USING (
        EXISTS (
            SELECT 1 FROM user_roles ur
            JOIN role_permissions rp ON ur.role_id = rp.role_id
            WHERE ur.user_id = current_setting('app.current_user')::uuid
            AND rp.permission = 'projects.read'
        )
    );
```

#### API-Level Access Control
```python
# dependencies/auth.py
from fastapi import Depends, HTTPException, status
from app.core.permissions import Permission, PermissionChecker

async def require_permission(
    permission: Permission,
    current_user: User = Depends(get_current_user),
    permission_checker: PermissionChecker = Depends()
) -> User:
    """Prüft ob Benutzer die erforderliche Berechtigung hat"""
    if not permission_checker.has_permission(
        user_roles=current_user.roles,
        required=permission
    ):
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Insufficient permissions"
        )
    return current_user

# Verwendung in Endpoints
@router.get("/projects/{project_id}")
async def get_project(
    project_id: UUID,
    user: User = Depends(require_permission(Permission.PROJECTS_READ)),
    service: ProjectService = Depends()
):
    # Zusätzliche Prüfung auf Projekt-Ebene
    project = await service.get_with_access_check(project_id, user)
    if not project:
        raise HTTPException(status_code=404)
    return project
```

#### SSRF Prevention
```python
# utils/http_client.py
import ipaddress
from urllib.parse import urlparse

BLOCKED_HOSTS = {
    'localhost', '127.0.0.1', '0.0.0.0',
    '169.254.169.254',  # AWS Metadata
    'metadata.google.internal',  # GCP
}

def validate_url(url: str) -> bool:
    """Verhindert SSRF durch URL-Validierung"""
    parsed = urlparse(url)

    # Nur HTTP(S) erlauben
    if parsed.scheme not in ('http', 'https'):
        return False

    # Blockierte Hosts
    if parsed.hostname in BLOCKED_HOSTS:
        return False

    # Private IP-Bereiche blockieren
    try:
        ip = ipaddress.ip_address(parsed.hostname)
        if ip.is_private or ip.is_loopback or ip.is_link_local:
            return False
    except ValueError:
        pass  # Hostname, keine IP

    return True
```

---

## 2. A02:2025 - Security Misconfiguration

### 2.1 Risikobeschreibung
Security Misconfiguration ist von Platz 5 auf Platz 2 gestiegen. Es betrifft 3,00% der Anwendungen aufgrund zunehmender Konfigurationskomplexität.

### 2.2 Maßnahmen

#### Sichere Default-Konfiguration
```python
# config.py
from pydantic_settings import BaseSettings
from pydantic import SecretStr, validator

class SecuritySettings(BaseSettings):
    # Erzwinge sichere Defaults
    DEBUG: bool = False
    SECRET_KEY: SecretStr

    # Session
    SESSION_COOKIE_SECURE: bool = True
    SESSION_COOKIE_HTTPONLY: bool = True
    SESSION_COOKIE_SAMESITE: str = "Strict"

    # CORS - Restriktiv
    CORS_ORIGINS: list[str] = []
    CORS_ALLOW_CREDENTIALS: bool = False

    # Headers
    HSTS_MAX_AGE: int = 31536000
    CONTENT_SECURITY_POLICY: str = "default-src 'self'"

    @validator('SECRET_KEY')
    def secret_key_min_length(cls, v):
        if len(v.get_secret_value()) < 32:
            raise ValueError('SECRET_KEY must be at least 32 characters')
        return v

    @validator('DEBUG')
    def no_debug_in_prod(cls, v):
        import os
        if os.getenv('ENVIRONMENT') == 'production' and v:
            raise ValueError('DEBUG must be False in production')
        return v
```

#### Security Headers (Caddy)
```caddyfile
# Caddyfile - Security Headers
(security_headers) {
    header {
        # HSTS
        Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"

        # Content Security Policy
        Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: blob:; font-src 'self'; connect-src 'self' wss:; frame-ancestors 'none'; form-action 'self'; base-uri 'self'"

        # Prevent MIME sniffing
        X-Content-Type-Options "nosniff"

        # Clickjacking protection
        X-Frame-Options "DENY"

        # XSS Protection (legacy browsers)
        X-XSS-Protection "1; mode=block"

        # Referrer Policy
        Referrer-Policy "strict-origin-when-cross-origin"

        # Permissions Policy
        Permissions-Policy "geolocation=(), microphone=(), camera=(), payment=(), usb=(), magnetometer=(), gyroscope=(), accelerometer=()"

        # Remove server header
        -Server
    }
}
```

#### Automatische Konfigurationsprüfung
```python
# scripts/security_check.py
def check_security_configuration():
    """Prüft Sicherheitskonfiguration beim Start"""
    issues = []

    # Debug-Modus
    if settings.DEBUG:
        issues.append("CRITICAL: DEBUG mode enabled")

    # Secret Key
    if len(settings.SECRET_KEY.get_secret_value()) < 64:
        issues.append("WARNING: SECRET_KEY should be 64+ characters")

    # Database
    if 'password' in settings.DATABASE_URL.lower():
        if 'localhost' not in settings.DATABASE_URL:
            issues.append("WARNING: Database password in URL for remote host")

    # SSL/TLS
    if not settings.SESSION_COOKIE_SECURE:
        issues.append("CRITICAL: Session cookies not secure")

    return issues
```

---

## 3. A03:2025 - Software Supply Chain Failures (NEU)

### 3.1 Risikobeschreibung
Neue Kategorie, erweitert "Vulnerable and Outdated Components". Umfasst Risiken in Dependencies, Build-Systemen und Distributionsinfrastruktur. Hat die höchsten Exploit- und Impact-Scores.

### 3.2 Maßnahmen

#### Dependency Management
```toml
# pyproject.toml - Pin exact versions
[tool.poetry.dependencies]
python = "^3.11"
fastapi = "0.109.2"
sqlalchemy = "2.0.25"
pydantic = "2.6.1"
# ... alle Versionen explizit pinnen

[tool.poetry.group.dev.dependencies]
safety = "^2.3.5"  # Vulnerability scanner
pip-audit = "^2.7.0"  # Pip vulnerability audit
```

#### Automatische Vulnerability Scans
```yaml
# .github/workflows/security.yml
name: Security Scan

on:
  push:
  schedule:
    - cron: '0 6 * * *'  # Täglich

jobs:
  dependency-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Safety Check
        run: |
          pip install safety
          safety check --full-report

      - name: Run pip-audit
        run: |
          pip install pip-audit
          pip-audit --strict

      - name: Run Trivy
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          severity: 'CRITICAL,HIGH'
```

#### Software Bill of Materials (SBOM)
```python
# scripts/generate_sbom.py
import subprocess
import json

def generate_sbom():
    """Generiert SBOM im CycloneDX Format"""
    result = subprocess.run(
        ['cyclonedx-py', 'poetry', '-o', 'sbom.json'],
        capture_output=True
    )

    with open('sbom.json') as f:
        sbom = json.load(f)

    # Lizenzprüfung
    for component in sbom['components']:
        license_id = component.get('licenses', [{}])[0].get('license', {}).get('id')
        if license_id in PROBLEMATIC_LICENSES:
            print(f"WARNING: {component['name']} uses {license_id}")
```

---

## 4. A04:2025 - Cryptographic Failures

### 4.1 Risikobeschreibung
Betrifft 3,80% der Anwendungen. Führt oft zu sensiblen Datenverlusten oder Systemkompromittierung.

### 4.2 Maßnahmen

#### Password Hashing
```python
# core/security.py
from argon2 import PasswordHasher
from argon2.exceptions import VerifyMismatchError

# Argon2id - Empfohlen von OWASP
ph = PasswordHasher(
    time_cost=3,        # Iterationen
    memory_cost=65536,  # 64 MB
    parallelism=4,      # Threads
    hash_len=32,        # Output-Länge
    salt_len=16         # Salt-Länge
)

def hash_password(password: str) -> str:
    return ph.hash(password)

def verify_password(password: str, hash: str) -> bool:
    try:
        ph.verify(hash, password)
        return True
    except VerifyMismatchError:
        return False
```

#### Datenverschlüsselung
```python
# core/encryption.py
from cryptography.fernet import Fernet
from cryptography.hazmat.primitives.kdf.scrypt import Scrypt
import os

class FieldEncryption:
    """AES-256 Verschlüsselung für sensible Felder"""

    def __init__(self, master_key: bytes):
        self.fernet = Fernet(master_key)

    def encrypt(self, data: str) -> str:
        return self.fernet.encrypt(data.encode()).decode()

    def decrypt(self, encrypted: str) -> str:
        return self.fernet.decrypt(encrypted.encode()).decode()

    @staticmethod
    def derive_key(password: str, salt: bytes) -> bytes:
        """Leitet Schlüssel aus Passwort ab (Scrypt)"""
        kdf = Scrypt(
            salt=salt,
            length=32,
            n=2**14,
            r=8,
            p=1,
        )
        return base64.urlsafe_b64encode(kdf.derive(password.encode()))
```

#### TLS-Konfiguration
```caddyfile
# Caddyfile - TLS
{
    # TLS 1.3 bevorzugen
    servers {
        protocols h1 h2 h3
    }
}

example.com {
    tls {
        protocols tls1.2 tls1.3
        ciphers TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384 TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256
    }
}
```

---

## 5. A05:2025 - Injection

### 5.1 Risikobeschreibung
Umfasst SQL Injection, XSS, Command Injection etc. Eine der am meisten getesteten Kategorien.

### 5.2 Maßnahmen

#### SQL Injection Prevention
```python
# IMMER parameterisierte Queries
from sqlalchemy import select, text

# RICHTIG - SQLAlchemy ORM
async def get_project(project_id: UUID):
    stmt = select(Project).where(Project.id == project_id)
    return await session.execute(stmt)

# RICHTIG - Parameterisiert
async def search_projects(search_term: str):
    stmt = text("SELECT * FROM projects WHERE name ILIKE :search")
    return await session.execute(stmt, {"search": f"%{search_term}%"})

# FALSCH - Niemals String-Konkatenation!
# query = f"SELECT * FROM projects WHERE name = '{user_input}'"
```

#### XSS Prevention
```python
# schemas/common.py
from pydantic import BaseModel, validator
import bleach

class SanitizedText(str):
    """Automatisch bereinigter Text"""

    @classmethod
    def __get_validators__(cls):
        yield cls.validate

    @classmethod
    def validate(cls, v):
        if isinstance(v, str):
            # HTML entfernen
            v = bleach.clean(v, strip=True)
        return cls(v)

class ProjectCreate(BaseModel):
    name: SanitizedText
    description: SanitizedText | None = None
```

#### Command Injection Prevention
```python
# NIEMALS shell=True mit Benutzereingaben
import subprocess
import shlex

def safe_command(filename: str):
    # Validierung
    if not filename.isalnum():
        raise ValueError("Invalid filename")

    # RICHTIG - Liste von Argumenten
    result = subprocess.run(
        ['cat', filename],
        capture_output=True,
        timeout=30
    )

    # FALSCH - Shell mit User-Input
    # subprocess.run(f"cat {filename}", shell=True)
```

---

## 6. A06:2025 - Insecure Design

### 6.1 Risikobeschreibung
Fokussiert auf Design-Fehler, nicht Implementation. Verbesserungen durch Threat Modeling sind sichtbar.

### 6.2 Maßnahmen

#### Threat Modeling
```markdown
## STRIDE Analyse für EnergieberaterPRO

### Spoofing (Identitätsvortäuschung)
- Bedrohung: Angreifer gibt sich als anderer Benutzer aus
- Maßnahme: MFA, sichere Session-Verwaltung

### Tampering (Manipulation)
- Bedrohung: Manipulation von Berechnungsergebnissen
- Maßnahme: Signierte Berichte, Audit-Trail

### Repudiation (Abstreitbarkeit)
- Bedrohung: Benutzer bestreitet Aktionen
- Maßnahme: Vollständiges Audit-Log

### Information Disclosure
- Bedrohung: Datenleck zwischen Mandanten
- Maßnahme: Row Level Security, Encryption

### Denial of Service
- Bedrohung: System-Überlastung
- Maßnahme: Rate Limiting, DDoS-Schutz

### Elevation of Privilege
- Bedrohung: Rechteausweitung
- Maßnahme: RBAC, Least Privilege
```

#### Security by Design Patterns
```python
# Fail Secure
def get_user_permissions(user_id: UUID) -> set[str]:
    """Bei Fehler: Keine Berechtigungen (nicht alle)"""
    try:
        return permission_service.get_permissions(user_id)
    except Exception:
        logger.error(f"Failed to get permissions for {user_id}")
        return set()  # Keine Berechtigungen

# Defense in Depth
class SecureProjectService:
    """Mehrschichtige Sicherheit"""

    async def get_project(self, project_id: UUID, user: User):
        # Schicht 1: API-Level Berechtigung
        if not user.has_permission('projects.read'):
            raise ForbiddenError()

        # Schicht 2: Database RLS
        project = await self.repo.get(project_id)

        # Schicht 3: Explizite Prüfung
        if project.organization_id != user.organization_id:
            raise ForbiddenError()

        return project
```

---

## 7. A07:2025 - Identification and Authentication Failures

### 7.1 Maßnahmen

#### Sichere Authentifizierung
```python
# auth/service.py
from datetime import timedelta

class AuthService:
    MAX_FAILED_ATTEMPTS = 5
    LOCKOUT_DURATION = timedelta(minutes=15)

    async def authenticate(
        self,
        email: str,
        password: str,
        ip_address: str
    ) -> AuthResult:
        # Brute-Force-Schutz
        if await self.is_locked(email):
            raise AccountLockedError()

        user = await self.user_repo.get_by_email(email)

        if not user or not verify_password(password, user.password_hash):
            await self.record_failed_attempt(email, ip_address)
            raise InvalidCredentialsError()

        # MFA prüfen
        if user.mfa_enabled:
            return AuthResult(
                status="mfa_required",
                mfa_token=self.generate_mfa_token(user.id)
            )

        # Session erstellen
        tokens = await self.create_session(user, ip_address)
        return AuthResult(status="success", tokens=tokens)

    async def create_session(self, user: User, ip_address: str):
        # Kurze Access-Token-Lebensdauer
        access_token = create_access_token(
            user_id=user.id,
            expires_delta=timedelta(minutes=15)
        )

        # Refresh Token (rotierbar)
        refresh_token = create_refresh_token(
            user_id=user.id,
            expires_delta=timedelta(days=7)
        )

        return TokenPair(access_token, refresh_token)
```

---

## 8. A08:2025 - Software and Data Integrity Failures

### 8.1 Maßnahmen

#### CI/CD Pipeline Security
```yaml
# .github/workflows/build.yml
jobs:
  build:
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # Signierte Commits prüfen
      - name: Verify commit signatures
        run: |
          git log --show-signature -1

      # Dependencies prüfen
      - name: Verify dependency integrity
        run: |
          pip install pip-tools
          pip-compile --generate-hashes requirements.in
          pip-sync --pip-args='--require-hashes'

      # Build mit Attestation
      - name: Build and sign
        run: |
          docker build --sbom=true --provenance=true -t app:${{ github.sha }} .
```

#### Update-Integrität
```python
# Sichere Update-Prüfung
import hashlib
import requests

def verify_update(update_url: str, expected_hash: str) -> bool:
    """Prüft Integrität von Updates"""
    response = requests.get(update_url)

    actual_hash = hashlib.sha256(response.content).hexdigest()

    return actual_hash == expected_hash
```

---

## 9. A09:2025 - Logging and Alerting Failures

### 9.1 Maßnahmen

#### Strukturiertes Logging
```python
# core/logging.py
import structlog
from datetime import datetime

logger = structlog.get_logger()

def log_security_event(
    event_type: str,
    user_id: str = None,
    ip_address: str = None,
    details: dict = None,
    severity: str = "INFO"
):
    """Protokolliert Sicherheitsereignisse"""
    logger.bind(
        event_type=event_type,
        user_id=user_id,
        ip_address=ip_address,
        timestamp=datetime.utcnow().isoformat(),
        severity=severity,
        **details or {}
    ).info(f"Security event: {event_type}")

# Verwendung
log_security_event(
    event_type="LOGIN_FAILED",
    ip_address=request.client.host,
    details={"email": email, "reason": "invalid_password"}
)
```

#### Alerting
```python
# alerts/security_alerts.py
ALERT_THRESHOLDS = {
    "LOGIN_FAILED": {"count": 10, "window_minutes": 5},
    "PERMISSION_DENIED": {"count": 20, "window_minutes": 10},
    "SQL_INJECTION_ATTEMPT": {"count": 1, "window_minutes": 1},
}

async def check_and_alert(event_type: str):
    """Prüft Schwellenwerte und sendet Alerts"""
    threshold = ALERT_THRESHOLDS.get(event_type)
    if not threshold:
        return

    count = await get_event_count(
        event_type,
        window_minutes=threshold["window_minutes"]
    )

    if count >= threshold["count"]:
        await send_security_alert(
            title=f"Security Alert: {event_type}",
            message=f"{count} events in {threshold['window_minutes']} minutes",
            severity="HIGH"
        )
```

---

## 10. A10:2025 - Mishandling of Exceptional Conditions (NEU)

### 10.1 Risikobeschreibung
Neue Kategorie mit 24 CWEs. Fokussiert auf Fehlerbehandlung, logische Fehler und "Fail Open"-Szenarien.

### 10.2 Maßnahmen

#### Fail Secure Exception Handling
```python
# core/exceptions.py
from fastapi import Request
from fastapi.responses import JSONResponse

async def secure_exception_handler(request: Request, exc: Exception):
    """Sichere Fehlerbehandlung - keine sensiblen Infos preisgeben"""

    # Log mit Details (intern)
    logger.error(
        "Unhandled exception",
        exc_info=exc,
        path=request.url.path,
        method=request.method
    )

    # Generische Antwort an Client
    return JSONResponse(
        status_code=500,
        content={
            "error": {
                "code": "INTERNAL_ERROR",
                "message": "An unexpected error occurred",
                "request_id": request.state.request_id
            }
        }
    )

# Keine Stack Traces in Production
@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    if settings.DEBUG:
        # Nur in Development
        raise exc
    return await secure_exception_handler(request, exc)
```

#### Graceful Degradation
```python
class ResilientService:
    """Service mit Fallback-Verhalten"""

    async def get_with_fallback(self, key: str):
        try:
            # Primäre Datenquelle
            return await self.primary_source.get(key)
        except PrimarySourceError:
            logger.warning(f"Primary source failed for {key}")
            try:
                # Fallback zu Cache
                return await self.cache.get(key)
            except CacheError:
                # Sicherer Default (nicht "fail open")
                raise ServiceUnavailableError("Service temporarily unavailable")
```

---

## Quellen

- [OWASP Top 10:2025 RC1](https://owasp.org/Top10/2025/0x00_2025-Introduction/)
- [Cybersecurity News - OWASP 2025](https://cybersecuritynews.com/owasp-top-10-2025/)
- [eSecurity Planet - OWASP 2025](https://www.esecurityplanet.com/threats/news-owasp-top-10-2025/)

---

*Letzte Aktualisierung: 2025-11-25*
*Version: 1.0.0*
