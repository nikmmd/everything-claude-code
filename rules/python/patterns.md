# Python Patterns

## Protocol (Structural Typing)

```python
from typing import Protocol

class Repository(Protocol):
    def find_by_id(self, id: str) -> dict | None: ...
    def save(self, entity: dict) -> dict: ...
```

## Data Transfer Objects

```python
from dataclasses import dataclass, field
from datetime import datetime

@dataclass
class CreateUserDTO:
    email: str
    name: str
    created_at: datetime = field(default_factory=datetime.utcnow)
```

## Resource Management

```python
# Context managers for cleanup
async with aiohttp.ClientSession() as session:
    async with session.get(url) as response:
        data = await response.json()

# Generators for lazy evaluation
def read_large_file(path: str):
    with open(path) as f:
        yield from f
```

## FastAPI Patterns

```python
from fastapi import APIRouter, Depends, HTTPException

router = APIRouter(prefix="/api/v1")

@router.get("/items/{item_id}")
async def get_item(
    item_id: str,
    db: Database = Depends(get_db),
) -> ItemResponse:
    item = await db.find_by_id(item_id)
    if not item:
        raise HTTPException(status_code=404, detail="Item not found")
    return ItemResponse.model_validate(item)
```
