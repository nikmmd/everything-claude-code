# Python Coding Style

## Conventions

- Follow PEP 8
- Type annotations on all function signatures
- Docstrings for public modules, classes, and functions (Google style)

## Immutability

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class User:
    name: str
    email: str

# Update by creating new instance
updated = User(name="new_name", email=user.email)
```

Use `NamedTuple` for lightweight immutable data:

```python
from typing import NamedTuple

class Point(NamedTuple):
    x: float
    y: float
```

## Error Handling

```python
try:
    result = perform_operation()
except SpecificError as e:
    logger.error("Operation failed", exc_info=True)
    raise AppError("User-friendly message") from e
```

## Validation

Use Pydantic for schema-based validation:

```python
from pydantic import BaseModel, EmailStr

class CreateUser(BaseModel):
    email: EmailStr
    name: str = Field(min_length=1, max_length=100)
```

## Tools

- **Formatting**: `ruff format` or `black`
- **Linting**: `ruff check`
- **Type checking**: `mypy` or `pyright`
- **Import sorting**: handled by ruff
