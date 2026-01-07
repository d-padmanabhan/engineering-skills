# OWASP Security Patterns

## A01: Broken Access Control

```python
# GOOD: Check authorization on every request
@app.route("/api/users/<user_id>")
@requires_auth
def get_user(user_id):
    if not current_user.can_access(user_id):
        abort(403)
    return User.get(user_id)

# BAD: No authorization check
@app.route("/api/users/<user_id>")
def get_user(user_id):
    return User.get(user_id)  # Anyone can access any user!
```

## A02: Cryptographic Failures

```python
# GOOD: Use strong hashing for passwords
from argon2 import PasswordHasher
ph = PasswordHasher()
hash = ph.hash(password)
ph.verify(hash, password)

# BAD: Weak hashing
import hashlib
hash = hashlib.md5(password.encode()).hexdigest()  # Never!
```

## A03: Injection

```python
# GOOD: Parameterized query
cursor.execute(
    "SELECT * FROM users WHERE id = %s",
    (user_id,)
)

# BAD: String interpolation
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")  # SQL injection!

# GOOD: ORM usage
User.query.filter_by(id=user_id).first()
```

## A04: Insecure Design

```python
# GOOD: Validate business logic
def transfer_money(from_account, to_account, amount):
    if amount <= 0:
        raise ValueError("Amount must be positive")
    if from_account.balance < amount:
        raise InsufficientFunds()
    if from_account.owner != current_user:
        raise Unauthorized()
    # Process transfer
```

## A05: Security Misconfiguration

```yaml
# GOOD: Secure defaults
DEBUG: false
ALLOWED_HOSTS: ["app.acme.com"]
SESSION_COOKIE_SECURE: true
SESSION_COOKIE_HTTPONLY: true
CSRF_COOKIE_SECURE: true
```

## A06: Vulnerable Components

```bash
# Regular dependency scanning
pip-audit
npm audit
govulncheck ./...

# Automated updates
dependabot enabled
```

## A07: Identification and Authentication Failures

```python
# GOOD: Secure session management
session.permanent = True
app.config["PERMANENT_SESSION_LIFETIME"] = timedelta(hours=1)
app.config["SESSION_COOKIE_SECURE"] = True
app.config["SESSION_COOKIE_HTTPONLY"] = True
app.config["SESSION_COOKIE_SAMESITE"] = "Lax"

# GOOD: MFA implementation
def verify_mfa(user, code):
    totp = pyotp.TOTP(user.mfa_secret)
    return totp.verify(code)
```

## A08: Software and Data Integrity Failures

```yaml
# GOOD: Verify dependencies
# package-lock.json, Pipfile.lock, go.sum

# GOOD: Sign artifacts
cosign sign myimage:latest
```

## A09: Security Logging and Monitoring Failures

```python
# GOOD: Log security events
logger.info("Login attempt", extra={
    "user": username,
    "ip": request.remote_addr,
    "success": authenticated,
    "mfa_used": mfa_verified
})

# Alert on anomalies
if failed_login_count > 5:
    alert("Possible brute force attack")
```

## A10: Server-Side Request Forgery (SSRF)

```python
# GOOD: Validate URLs
from urllib.parse import urlparse

ALLOWED_HOSTS = ["api.acme.com", "cdn.acme.com"]

def safe_fetch(url):
    parsed = urlparse(url)
    if parsed.hostname not in ALLOWED_HOSTS:
        raise ValueError("Host not allowed")
    if parsed.scheme not in ["https"]:
        raise ValueError("HTTPS required")
    return requests.get(url)
```

## Input Validation

```python
from pydantic import BaseModel, EmailStr, validator

class UserInput(BaseModel):
    name: str
    email: EmailStr
    age: int
    
    @validator("name")
    def name_alphanumeric(cls, v):
        if not v.replace(" ", "").isalnum():
            raise ValueError("Name must be alphanumeric")
        return v
    
    @validator("age")
    def age_reasonable(cls, v):
        if not 0 <= v <= 150:
            raise ValueError("Age must be 0-150")
        return v
```
