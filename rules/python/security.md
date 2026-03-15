# Python Security

## Secret Management

```python
import os

api_key = os.environ["API_KEY"]  # Raises KeyError if missing
```

Or with explicit validation:

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    api_key: str
    database_url: str

    class Config:
        env_file = ".env"
```

## Static Analysis

```bash
bandit -r src/      # Security vulnerability scanning
ruff check src/     # Includes security-related rules
```

## SQL Safety

Always use parameterized queries:

```python
# GOOD
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))

# BAD
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")
```

Scope: `**/*.py`, `**/*.pyi`
