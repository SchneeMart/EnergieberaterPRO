# Sicherheitskonzept

## 1. Übersicht

### 1.1 Sicherheitsziele
- **Vertraulichkeit**: Schutz sensibler Gebäude- und Kundendaten
- **Integrität**: Korrektheit von Berechnungen und Berichten
- **Verfügbarkeit**: Zuverlässiger Zugang zum System
- **Authentizität**: Eindeutige Identifikation von Benutzern
- **Nachvollziehbarkeit**: Lückenlose Audit-Logs

### 1.2 Compliance-Anforderungen
- DSGVO (Datenschutz-Grundverordnung)
- IT-Sicherheitsgesetz
- OWASP Top 10
- BSI IT-Grundschutz (optional)

---

## 2. Authentifizierung

### 2.1 Passwort-Richtlinien

```
Mindestanforderungen:
- Länge: 12+ Zeichen
- Komplexität: Groß, Klein, Zahl, Sonderzeichen
- Keine häufigen Passwörter (Liste)
- Keine Wiederverwendung (letzte 10)
- Ablauf: 365 Tage (konfigurierbar)
```

### 2.2 Password Hashing

```python
# core/security.py
from passlib.context import CryptContext
import bcrypt

# Bcrypt mit Work Factor 12
pwd_context = CryptContext(
    schemes=["bcrypt"],
    deprecated="auto",
    bcrypt__rounds=12
)

def hash_password(password: str) -> str:
    """Hasht ein Passwort sicher"""
    return pwd_context.hash(password)

def verify_password(plain: str, hashed: str) -> bool:
    """Verifiziert ein Passwort"""
    return pwd_context.verify(plain, hashed)

def check_password_strength(password: str) -> dict:
    """Prüft Passwortstärke"""
    issues = []

    if len(password) < 12:
        issues.append("Mindestens 12 Zeichen erforderlich")
    if not any(c.isupper() for c in password):
        issues.append("Mindestens ein Großbuchstabe erforderlich")
    if not any(c.islower() for c in password):
        issues.append("Mindestens ein Kleinbuchstabe erforderlich")
    if not any(c.isdigit() for c in password):
        issues.append("Mindestens eine Zahl erforderlich")
    if not any(c in "!@#$%^&*()_+-=[]{}|;:,.<>?" for c in password):
        issues.append("Mindestens ein Sonderzeichen erforderlich")

    # Prüfung gegen häufige Passwörter
    if password.lower() in COMMON_PASSWORDS:
        issues.append("Dieses Passwort ist zu häufig")

    return {
        "valid": len(issues) == 0,
        "issues": issues,
        "strength": calculate_entropy(password)
    }
```

### 2.3 JWT Token

```python
# core/security.py
from datetime import datetime, timedelta
from jose import jwt, JWTError
from pydantic import BaseModel

class TokenPayload(BaseModel):
    sub: str                    # User ID
    org: str | None = None      # Organization ID
    roles: list[str] = []       # Rollen
    exp: datetime               # Ablaufzeit
    iat: datetime               # Ausstellungszeit
    jti: str                    # Token ID (für Invalidierung)

class TokenService:
    def __init__(self, secret_key: str, algorithm: str = "HS256"):
        self.secret_key = secret_key
        self.algorithm = algorithm

    def create_access_token(
        self,
        user_id: str,
        organization_id: str = None,
        roles: list[str] = None,
        expires_delta: timedelta = timedelta(minutes=15)
    ) -> str:
        """Erstellt Access Token (kurzlebig)"""
        now = datetime.utcnow()
        payload = {
            "sub": user_id,
            "org": organization_id,
            "roles": roles or [],
            "exp": now + expires_delta,
            "iat": now,
            "jti": str(uuid.uuid4()),
            "type": "access"
        }
        return jwt.encode(payload, self.secret_key, algorithm=self.algorithm)

    def create_refresh_token(
        self,
        user_id: str,
        expires_delta: timedelta = timedelta(days=7)
    ) -> str:
        """Erstellt Refresh Token (langlebig)"""
        now = datetime.utcnow()
        payload = {
            "sub": user_id,
            "exp": now + expires_delta,
            "iat": now,
            "jti": str(uuid.uuid4()),
            "type": "refresh"
        }
        return jwt.encode(payload, self.secret_key, algorithm=self.algorithm)

    def verify_token(self, token: str) -> TokenPayload | None:
        """Verifiziert und dekodiert Token"""
        try:
            payload = jwt.decode(
                token,
                self.secret_key,
                algorithms=[self.algorithm]
            )

            # Prüfen ob Token auf Blacklist
            if self.is_blacklisted(payload["jti"]):
                return None

            return TokenPayload(**payload)
        except JWTError:
            return None

    def blacklist_token(self, jti: str, exp: datetime):
        """Setzt Token auf Blacklist (für Logout)"""
        # In Redis speichern mit TTL bis Token abläuft
        redis.setex(f"token_blacklist:{jti}", exp - datetime.utcnow(), "1")

    def is_blacklisted(self, jti: str) -> bool:
        return redis.exists(f"token_blacklist:{jti}")
```

### 2.4 Multi-Faktor-Authentifizierung (MFA)

```python
# core/mfa.py
import pyotp
import qrcode
from io import BytesIO

class MFAService:
    def __init__(self, issuer: str = "EnergieberaterPRO"):
        self.issuer = issuer

    def generate_secret(self) -> str:
        """Generiert neuen TOTP Secret"""
        return pyotp.random_base32()

    def get_provisioning_uri(self, secret: str, email: str) -> str:
        """Generiert URI für Authenticator-App"""
        totp = pyotp.TOTP(secret)
        return totp.provisioning_uri(
            name=email,
            issuer_name=self.issuer
        )

    def generate_qr_code(self, secret: str, email: str) -> bytes:
        """Generiert QR-Code für Authenticator-App"""
        uri = self.get_provisioning_uri(secret, email)
        qr = qrcode.QRCode(version=1, box_size=10, border=5)
        qr.add_data(uri)
        qr.make(fit=True)

        img = qr.make_image(fill_color="black", back_color="white")
        buffer = BytesIO()
        img.save(buffer, format="PNG")
        return buffer.getvalue()

    def verify_code(self, secret: str, code: str) -> bool:
        """Verifiziert TOTP Code"""
        totp = pyotp.TOTP(secret)
        return totp.verify(code, valid_window=1)  # ±30 Sekunden

    def generate_backup_codes(self, count: int = 10) -> list[str]:
        """Generiert Backup-Codes"""
        import secrets
        return [secrets.token_hex(4).upper() for _ in range(count)]
```

### 2.5 Brute-Force-Schutz

```python
# core/rate_limit.py
from datetime import datetime, timedelta

class LoginRateLimiter:
    """
    Schützt gegen Brute-Force-Angriffe
    """

    def __init__(self, redis_client):
        self.redis = redis_client

    async def check_login_attempt(
        self,
        identifier: str,  # IP oder E-Mail
        max_attempts: int = 5,
        window_seconds: int = 300  # 5 Minuten
    ) -> tuple[bool, int]:
        """
        Prüft ob Login-Versuch erlaubt

        Returns:
            (erlaubt, verbleibende_versuche)
        """
        key = f"login_attempts:{identifier}"

        # Aktuelle Anzahl abrufen
        attempts = int(await self.redis.get(key) or 0)

        if attempts >= max_attempts:
            # Gesperrt - Zeit bis Entsperrung
            ttl = await self.redis.ttl(key)
            return False, ttl

        return True, max_attempts - attempts

    async def record_failed_attempt(
        self,
        identifier: str,
        window_seconds: int = 300
    ):
        """Zeichnet fehlgeschlagenen Versuch auf"""
        key = f"login_attempts:{identifier}"

        pipe = self.redis.pipeline()
        pipe.incr(key)
        pipe.expire(key, window_seconds)
        await pipe.execute()

    async def record_successful_login(self, identifier: str):
        """Setzt Zähler nach erfolgreichem Login zurück"""
        await self.redis.delete(f"login_attempts:{identifier}")

    async def lock_account(
        self,
        user_id: str,
        duration: timedelta = timedelta(hours=1)
    ):
        """Sperrt Account temporär"""
        await self.redis.setex(
            f"account_locked:{user_id}",
            int(duration.total_seconds()),
            "1"
        )

    async def is_account_locked(self, user_id: str) -> bool:
        return await self.redis.exists(f"account_locked:{user_id}")
```

---

## 3. Autorisierung (RBAC)

### 3.1 Rollen-Hierarchie

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            ROLLEN-HIERARCHIE                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────┐                                                            │
│  │  SUPERADMIN │  ← System-Administrator (Plattform)                       │
│  └──────┬──────┘                                                            │
│         │                                                                   │
│         ▼                                                                   │
│  ┌─────────────┐                                                            │
│  │    ADMIN    │  ← Organisations-Administrator                             │
│  └──────┬──────┘                                                            │
│         │                                                                   │
│         ├──────────────────┬──────────────────┐                             │
│         ▼                  ▼                  ▼                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                     │
│  │   MANAGER   │    │  CONSULTANT │    │   VIEWER    │                     │
│  │             │    │             │    │             │                     │
│  │ Projektleit.│    │  Berater    │    │ Nur Lesen   │                     │
│  └─────────────┘    └─────────────┘    └─────────────┘                     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Berechtigungen

```python
# core/permissions.py
from enum import Enum
from typing import Set

class Permission(str, Enum):
    # Projekte
    PROJECTS_READ = "projects.read"
    PROJECTS_CREATE = "projects.create"
    PROJECTS_UPDATE = "projects.update"
    PROJECTS_DELETE = "projects.delete"

    # Berechnungen
    CALCULATIONS_READ = "calculations.read"
    CALCULATIONS_CREATE = "calculations.create"
    CALCULATIONS_RUN = "calculations.run"

    # Berichte
    REPORTS_READ = "reports.read"
    REPORTS_CREATE = "reports.create"
    REPORTS_APPROVE = "reports.approve"

    # Geschäft
    QUOTES_READ = "quotes.read"
    QUOTES_CREATE = "quotes.create"
    QUOTES_APPROVE = "quotes.approve"
    INVOICES_READ = "invoices.read"
    INVOICES_CREATE = "invoices.create"

    # Benutzer
    USERS_READ = "users.read"
    USERS_CREATE = "users.create"
    USERS_UPDATE = "users.update"
    USERS_DELETE = "users.delete"

    # Organisation
    ORGANIZATION_READ = "organization.read"
    ORGANIZATION_UPDATE = "organization.update"

    # Admin
    ADMIN_ACCESS = "admin.access"
    SYSTEM_CONFIG = "system.config"

ROLE_PERMISSIONS: dict[str, Set[Permission]] = {
    "superadmin": {Permission(p) for p in Permission},  # Alle

    "admin": {
        Permission.PROJECTS_READ,
        Permission.PROJECTS_CREATE,
        Permission.PROJECTS_UPDATE,
        Permission.PROJECTS_DELETE,
        Permission.CALCULATIONS_READ,
        Permission.CALCULATIONS_CREATE,
        Permission.CALCULATIONS_RUN,
        Permission.REPORTS_READ,
        Permission.REPORTS_CREATE,
        Permission.REPORTS_APPROVE,
        Permission.QUOTES_READ,
        Permission.QUOTES_CREATE,
        Permission.QUOTES_APPROVE,
        Permission.INVOICES_READ,
        Permission.INVOICES_CREATE,
        Permission.USERS_READ,
        Permission.USERS_CREATE,
        Permission.USERS_UPDATE,
        Permission.USERS_DELETE,
        Permission.ORGANIZATION_READ,
        Permission.ORGANIZATION_UPDATE,
    },

    "manager": {
        Permission.PROJECTS_READ,
        Permission.PROJECTS_CREATE,
        Permission.PROJECTS_UPDATE,
        Permission.CALCULATIONS_READ,
        Permission.CALCULATIONS_CREATE,
        Permission.CALCULATIONS_RUN,
        Permission.REPORTS_READ,
        Permission.REPORTS_CREATE,
        Permission.REPORTS_APPROVE,
        Permission.QUOTES_READ,
        Permission.QUOTES_CREATE,
        Permission.INVOICES_READ,
        Permission.USERS_READ,
    },

    "consultant": {
        Permission.PROJECTS_READ,
        Permission.PROJECTS_CREATE,
        Permission.PROJECTS_UPDATE,
        Permission.CALCULATIONS_READ,
        Permission.CALCULATIONS_CREATE,
        Permission.CALCULATIONS_RUN,
        Permission.REPORTS_READ,
        Permission.REPORTS_CREATE,
        Permission.QUOTES_READ,
    },

    "viewer": {
        Permission.PROJECTS_READ,
        Permission.CALCULATIONS_READ,
        Permission.REPORTS_READ,
    }
}

class PermissionChecker:
    """Prüft Berechtigungen für Benutzer"""

    def has_permission(
        self,
        user_roles: list[str],
        required: Permission
    ) -> bool:
        """Prüft ob Benutzer berechtigt ist"""
        for role in user_roles:
            if role in ROLE_PERMISSIONS:
                if required in ROLE_PERMISSIONS[role]:
                    return True
        return False

    def get_user_permissions(self, user_roles: list[str]) -> Set[Permission]:
        """Gibt alle Berechtigungen des Benutzers zurück"""
        permissions = set()
        for role in user_roles:
            if role in ROLE_PERMISSIONS:
                permissions.update(ROLE_PERMISSIONS[role])
        return permissions
```

### 3.3 Row-Level Security

```python
# Projekt-spezifische Zugriffskontrolle
class ProjectAccessControl:
    """
    Kontrolle auf Projektebene
    """

    async def can_access_project(
        self,
        user_id: str,
        project_id: str,
        required_permission: Permission
    ) -> bool:
        """
        Prüft ob Benutzer auf Projekt zugreifen darf

        Zugriff wenn:
        1. Benutzer ist in gleicher Organisation wie Projekt
        2. Benutzer hat erforderliche Berechtigung
        3. Benutzer ist Projektzugewiesen (optional)
        """
        # Organisation prüfen
        user_org = await self.get_user_organization(user_id)
        project_org = await self.get_project_organization(project_id)

        if user_org != project_org:
            return False

        # Berechtigung prüfen
        user_roles = await self.get_user_roles(user_id)
        if not self.has_permission(user_roles, required_permission):
            return False

        return True
```

---

## 4. Datenschutz & DSGVO

### 4.1 Personenbezogene Daten

```
Kategorien:
├── Benutzer
│   ├── Name, E-Mail, Telefon
│   ├── Passwort (gehashed)
│   └── Login-Historie
├── Kunden
│   ├── Name, Adresse, Kontakt
│   ├── Rechnungsdaten
│   └── Projekthistorie
└── Gebäude
    └── Adresse (ggf. mit Eigentümer)
```

### 4.2 Verarbeitungsverzeichnis

```yaml
Verarbeitungstätigkeit: Energieberatung
Verantwortlicher: [Organisation]
Zweck: Durchführung von Energieberatungen
Rechtsgrundlage: Art. 6 Abs. 1 lit. b DSGVO (Vertrag)
Kategorien betroffener Personen:
  - Kunden
  - Gebäudeeigentümer
Kategorien personenbezogener Daten:
  - Kontaktdaten
  - Gebäudeadressen
  - Verbrauchsdaten
Empfänger: Keine (außer auf Wunsch des Kunden)
Drittlandtransfer: Nein (alle Daten in EU)
Löschfristen:
  - Aktive Projekte: Aufbewahrung während Vertrag
  - Abgeschlossene Projekte: 10 Jahre (Aufbewahrungspflicht)
  - Benutzerkonten: Bei Löschung sofort
TOMs: Siehe Sicherheitskonzept
```

### 4.3 Daten-Export (Art. 20 DSGVO)

```python
# services/privacy_service.py
class PrivacyService:
    """DSGVO-Compliance-Funktionen"""

    async def export_user_data(self, user_id: str) -> dict:
        """
        Exportiert alle Daten eines Benutzers (Datenportabilität)
        """
        return {
            "user": await self.get_user_data(user_id),
            "projects": await self.get_user_projects(user_id),
            "time_entries": await self.get_user_time_entries(user_id),
            "audit_log": await self.get_user_audit_log(user_id),
            "exported_at": datetime.utcnow().isoformat()
        }

    async def anonymize_user_data(self, user_id: str):
        """
        Anonymisiert Benutzerdaten (Recht auf Löschung)
        """
        # Benutzer anonymisieren
        await self.db.execute("""
            UPDATE users SET
                email = CONCAT('deleted_', id, '@anonymized.local'),
                first_name = 'Gelöscht',
                last_name = 'Benutzer',
                phone = NULL,
                password_hash = 'DELETED'
            WHERE id = :user_id
        """, {"user_id": user_id})

        # Audit-Log Eintrag
        await self.audit_log.log(
            action="USER_ANONYMIZED",
            user_id=user_id
        )

    async def get_consent_status(self, user_id: str) -> dict:
        """Gibt Einwilligungsstatus zurück"""
        pass

    async def update_consent(
        self,
        user_id: str,
        consent_type: str,
        granted: bool
    ):
        """Aktualisiert Einwilligung"""
        pass
```

---

## 5. Input Validation & Output Encoding

### 5.1 Request Validation

```python
# schemas/validation.py
from pydantic import BaseModel, validator, Field
import bleach
import re

class ProjectCreate(BaseModel):
    name: str = Field(..., min_length=3, max_length=255)
    description: str | None = Field(None, max_length=5000)

    @validator('name')
    def sanitize_name(cls, v):
        # HTML entfernen
        v = bleach.clean(v, strip=True)
        # Gefährliche Zeichen entfernen
        v = re.sub(r'[<>"\']', '', v)
        return v

    @validator('description')
    def sanitize_description(cls, v):
        if v:
            # Erlaubte HTML-Tags
            v = bleach.clean(
                v,
                tags=['p', 'br', 'b', 'i', 'ul', 'ol', 'li'],
                strip=True
            )
        return v

class CalculationInput(BaseModel):
    u_value: float = Field(..., gt=0, lt=10)
    area: float = Field(..., gt=0, lt=100000)
    temperature: float = Field(..., ge=-50, le=50)

    @validator('u_value', 'area', 'temperature')
    def check_numeric_bounds(cls, v, field):
        if not isinstance(v, (int, float)):
            raise ValueError(f"{field.name} muss numerisch sein")
        return v
```

### 5.2 SQL Injection Prevention

```python
# IMMER parameterisierte Queries verwenden
# NIE String-Konkatenation für SQL

# FALSCH (SQL Injection möglich):
query = f"SELECT * FROM users WHERE email = '{email}'"

# RICHTIG (parameterisiert):
query = "SELECT * FROM users WHERE email = :email"
result = await db.execute(query, {"email": email})

# Mit SQLAlchemy ORM:
user = await session.execute(
    select(User).where(User.email == email)
)
```

### 5.3 XSS Prevention

```python
# templates/base.html (Jinja2 Auto-Escape)
# Automatisches Escaping ist standardmäßig aktiviert

{{ user.name }}  {# Wird automatisch escaped #}
{{ user.bio | safe }}  {# NUR wenn sicher! #}

# In JavaScript:
# Immer textContent statt innerHTML
element.textContent = userInput;  // Sicher
element.innerHTML = userInput;    // GEFÄHRLICH!
```

---

## 6. Verschlüsselung

### 6.1 Transport-Verschlüsselung (TLS)

```
Caddy Konfiguration:
- TLS 1.3 bevorzugt
- TLS 1.2 als Minimum
- Starke Cipher Suites
- HSTS aktiviert
- OCSP Stapling

Automatische Zertifikate:
- Let's Encrypt
- Auto-Renewal
- HTTP Challenge
```

### 6.2 Daten-Verschlüsselung at Rest

```python
# utils/crypto.py
from cryptography.fernet import Fernet
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
from cryptography.hazmat.primitives import hashes
import base64
import os

class FieldEncryption:
    """
    Verschlüsselung für sensible Datenbankfelder
    """

    def __init__(self, master_key: bytes):
        self.fernet = Fernet(master_key)

    def encrypt(self, plaintext: str) -> str:
        """Verschlüsselt einen String"""
        return self.fernet.encrypt(plaintext.encode()).decode()

    def decrypt(self, ciphertext: str) -> str:
        """Entschlüsselt einen String"""
        return self.fernet.decrypt(ciphertext.encode()).decode()

    @staticmethod
    def derive_key(password: str, salt: bytes) -> bytes:
        """Leitet Schlüssel aus Passwort ab"""
        kdf = PBKDF2HMAC(
            algorithm=hashes.SHA256(),
            length=32,
            salt=salt,
            iterations=480000,
        )
        key = base64.urlsafe_b64encode(kdf.derive(password.encode()))
        return key

# Verwendung für sensible Felder:
class Customer(Base):
    __tablename__ = "customers"

    # Normale Felder
    id = Column(UUID, primary_key=True)
    name = Column(String)

    # Verschlüsselte Felder
    _tax_id = Column("tax_id", String)  # Verschlüsselt gespeichert

    @property
    def tax_id(self):
        if self._tax_id:
            return encryption.decrypt(self._tax_id)
        return None

    @tax_id.setter
    def tax_id(self, value):
        if value:
            self._tax_id = encryption.encrypt(value)
        else:
            self._tax_id = None
```

### 6.3 Backup-Verschlüsselung

```bash
# Backup mit GPG-Verschlüsselung
pg_dump energieberater | gzip | gpg --symmetric --cipher-algo AES256 > backup.sql.gz.gpg

# Restore
gpg --decrypt backup.sql.gz.gpg | gunzip | psql energieberater
```

---

## 7. Audit Logging

### 7.1 Log-Struktur

```python
# core/audit.py
from datetime import datetime
from enum import Enum
from pydantic import BaseModel

class AuditAction(str, Enum):
    # Auth
    LOGIN_SUCCESS = "auth.login.success"
    LOGIN_FAILED = "auth.login.failed"
    LOGOUT = "auth.logout"
    PASSWORD_CHANGED = "auth.password.changed"
    MFA_ENABLED = "auth.mfa.enabled"

    # Daten
    CREATE = "data.create"
    READ = "data.read"
    UPDATE = "data.update"
    DELETE = "data.delete"

    # Berichte
    REPORT_GENERATED = "report.generated"
    REPORT_EXPORTED = "report.exported"

    # Admin
    USER_CREATED = "admin.user.created"
    USER_DELETED = "admin.user.deleted"
    ROLE_CHANGED = "admin.role.changed"

class AuditLog(BaseModel):
    id: str
    timestamp: datetime
    action: AuditAction
    user_id: str | None
    organization_id: str | None
    resource_type: str | None
    resource_id: str | None
    ip_address: str | None
    user_agent: str | None
    details: dict | None
    success: bool

class AuditService:
    async def log(
        self,
        action: AuditAction,
        user_id: str = None,
        resource_type: str = None,
        resource_id: str = None,
        details: dict = None,
        success: bool = True
    ):
        entry = AuditLog(
            id=str(uuid.uuid4()),
            timestamp=datetime.utcnow(),
            action=action,
            user_id=user_id,
            organization_id=self.get_org_from_context(),
            resource_type=resource_type,
            resource_id=resource_id,
            ip_address=self.get_client_ip(),
            user_agent=self.get_user_agent(),
            details=details,
            success=success
        )

        # In Datenbank speichern
        await self.db.execute(
            "INSERT INTO audit_log VALUES (:id, :timestamp, ...)",
            entry.dict()
        )

        # Für kritische Aktionen: Auch in externes System
        if action in CRITICAL_ACTIONS:
            await self.send_to_siem(entry)
```

### 7.2 Log-Retention

```
Aufbewahrungsfristen:
├── Security Events: 2 Jahre
├── Auth Events: 1 Jahr
├── Data Access: 90 Tage
└── Debug Logs: 30 Tage

Archivierung:
- Nach 90 Tagen: Komprimierung
- Nach 1 Jahr: Archiv-Storage
- Nach Frist: Sichere Löschung
```

---

## 8. Security Headers

```python
# Caddy Configuration
(security_headers) {
    header {
        # HSTS
        Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"

        # XSS Protection
        X-XSS-Protection "1; mode=block"

        # Content Type Options
        X-Content-Type-Options "nosniff"

        # Frame Options
        X-Frame-Options "DENY"

        # Referrer Policy
        Referrer-Policy "strict-origin-when-cross-origin"

        # Permissions Policy
        Permissions-Policy "geolocation=(), microphone=(), camera=()"

        # Content Security Policy
        Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: blob:; font-src 'self'; connect-src 'self' wss:; frame-ancestors 'none';"
    }
}
```

---

## 9. Sicherheits-Checkliste

### 9.1 Deployment-Checkliste

```
[ ] Alle Default-Passwörter geändert
[ ] DEBUG-Modus deaktiviert
[ ] HTTPS erzwungen
[ ] Security Headers konfiguriert
[ ] Firewall-Regeln eingerichtet
[ ] Backup-Verschlüsselung aktiviert
[ ] Log-Rotation konfiguriert
[ ] Monitoring eingerichtet
[ ] Notfall-Kontakte dokumentiert
```

### 9.2 Regelmäßige Prüfungen

```
Täglich:
- Automatisierte Security Scans
- Log-Analyse auf Anomalien

Wöchentlich:
- Dependency-Updates prüfen
- Backup-Tests

Monatlich:
- Access Review
- Vulnerability Scan

Jährlich:
- Penetration Test
- Security Audit
- Notfallplan-Test
```

---

*Letzte Aktualisierung: 2025-11-25*
*Version: 0.1.0*
