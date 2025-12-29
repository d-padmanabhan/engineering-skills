# Configuration Management

## Configuration Precedence

Configuration is applied in order, with later sources overriding earlier ones:

1. **System configuration** (`/etc/gitconfig`, `/etc/app/config.yaml`)
2. **User configuration** (`~/.gitconfig`, `~/.app/config.yaml`)
3. **Local/project configuration** (`.git/config`, `./config.yaml`)
4. **Environment variables** (`GIT_AUTHOR_NAME=...`, `APP_API_KEY=...`)
5. **Command-line parameters** (`git commit --author=...`, `app --api-key=...`)

This pattern is used by Git, SSH, and many Unix tools. Use this hierarchy to allow:
- System defaults
- User preferences
- Project settings
- Runtime environment overrides
- Explicit command-line overrides

## 12-Factor App Principles

1. **Codebase**: One codebase, many deploys
2. **Dependencies**: Explicitly declare and isolate
3. **Config**: Store in environment
4. **Backing services**: Treat as attached resources
5. **Build, release, run**: Strictly separate stages
6. **Processes**: Stateless, share-nothing
7. **Port binding**: Export via port binding
8. **Concurrency**: Scale out via processes
9. **Disposability**: Fast startup, graceful shutdown
10. **Dev/prod parity**: Keep environments similar
11. **Logs**: Treat as event streams
12. **Admin processes**: Run as one-off processes

## Environment Variables

### String Type

Environment variables are **strings** in the OS. Convert them in your application:

```python
# GOOD: Explicit conversion
port = int(os.getenv("PORT", "8080"))
timeout = float(os.getenv("TIMEOUT", "30.0"))
debug = os.getenv("DEBUG") == "1"

# BAD: Assuming type
port = os.getenv("PORT", 8080)  # Wrong! Default should be string
```

### Boolean Values

**Use `1` for true. Anything that is not `1` is false.**

```python
# GOOD: Consistent boolean handling
debug = os.getenv("DEBUG") == "1"
verbose = os.getenv("VERBOSE") == "1"
enabled = os.getenv("FEATURE_ENABLED") == "1"

# BAD: Parsing various strings
debug = os.getenv("DEBUG", "false").lower() in ["true", "1", "yes", "on"]
# This is brittle and opinionated
```

**Rationale:**
- Simple and unambiguous
- Works across all languages
- No parsing ambiguity
- Consistent with Unix conventions

### Required Values

**Fail fast if required configuration is missing:**

```python
# GOOD: Fail fast with clear error
api_key = os.getenv("API_KEY")
if not api_key:
    raise ValueError(
        "API_KEY environment variable is required. "
        "Set it with: export API_KEY=your-key"
    )

# BAD: Silent failure or default
api_key = os.getenv("API_KEY", "")  # What if empty string?
# Later: api_key is empty, fails mysteriously
```

### Default Values

Provide sensible defaults when appropriate:

```python
# GOOD: Sensible defaults
port = int(os.getenv("PORT", "8080"))
log_level = os.getenv("LOG_LEVEL", "INFO")
timeout = int(os.getenv("TIMEOUT", "30"))

# GOOD: No default for required values
api_key = os.getenv("API_KEY")
if not api_key:
    raise ValueError("API_KEY is required")
```

### Bash Examples

```bash
# Load from .env file
export $(grep -v '^#' .env | xargs)

# Required variables with defaults
DATABASE_URL="${DATABASE_URL:?DATABASE_URL is required}"
LOG_LEVEL="${LOG_LEVEL:-INFO}"
PORT="${PORT:-8080}"
```

## Configuration Files

### YAML Configuration
```yaml
# config.yaml
server:
  host: 0.0.0.0
  port: 8080
  
database:
  host: ${DB_HOST:-localhost}
  port: ${DB_PORT:-5432}
  name: ${DB_NAME:-myapp}
  
logging:
  level: ${LOG_LEVEL:-INFO}
  format: json
```

### Environment-Specific Overrides
```
config/
├── default.yaml      # Base config
├── development.yaml  # Dev overrides
├── staging.yaml      # Staging overrides
└── production.yaml   # Prod overrides
```

## Secret Management

### AWS Secrets Manager
```hcl
# Terraform
resource "aws_secretsmanager_secret" "db_password" {
  name = "myapp/db_password"
}

data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = aws_secretsmanager_secret.db_password.id
}
```

### HashiCorp Vault
```bash
# Store secret
vault kv put secret/myapp db_password=supersecret

# Read secret
vault kv get -field=db_password secret/myapp
```

### Kubernetes Secrets
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
stringData:
  username: admin
  password: ${DB_PASSWORD}
```

## Feature Flags

```python
# Simple feature flags
FEATURES = {
    "new_dashboard": os.getenv("FEATURE_NEW_DASHBOARD", "false").lower() == "true",
    "beta_api": os.getenv("FEATURE_BETA_API", "false").lower() == "true",
}

if FEATURES["new_dashboard"]:
    enable_new_dashboard()
```

## Configuration Validation

```python
from pydantic import BaseSettings, validator

class Settings(BaseSettings):
    database_url: str
    redis_url: str = "redis://localhost:6379"
    log_level: str = "INFO"
    
    @validator("log_level")
    def validate_log_level(cls, v):
        allowed = ["DEBUG", "INFO", "WARNING", "ERROR"]
        if v.upper() not in allowed:
            raise ValueError(f"log_level must be one of {allowed}")
        return v.upper()
    
    class Config:
        env_file = ".env"

settings = Settings()
```

## Best Practices

1. **Never commit secrets** to version control
2. **Use environment variables** for configuration that varies by environment
3. **Validate configuration** at startup
4. **Fail fast** if required configuration is missing
5. **Document** all configuration options
6. **Use defaults** for optional settings
7. **Separate secrets** from other configuration
8. **Rotate secrets** regularly
9. **Audit access** to sensitive configuration
10. **Version** configuration changes

