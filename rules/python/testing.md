# Python Testing

## Framework: pytest

```bash
# Run tests
uv run pytest tests/ -v

# With coverage (80%+ required)
uv run pytest tests/ --cov=src --cov-report=term-missing

# Run specific test
uv run pytest tests/test_users.py -v

# Run async tests
uv run pytest tests/ -v --asyncio-mode=auto

# With race detection via markers
uv run pytest tests/ -m "not slow"
```

## Test Organization

```python
import pytest

@pytest.fixture
def sample_user():
    return {"id": "1", "name": "Test User", "email": "test@example.com"}

class TestUserService:
    def test_create_user(self, sample_user):
        result = create_user(sample_user)
        assert result.id == "1"

    def test_create_user_invalid_email(self):
        with pytest.raises(ValidationError):
            create_user({"email": "invalid"})

    @pytest.mark.parametrize("role,expected", [
        ("admin", True),
        ("user", False),
    ])
    def test_is_admin(self, role, expected):
        assert is_admin(role) == expected
```

## Key Packages

pytest, pytest-asyncio, pytest-cov, pytest-mock, httpx, factory-boy

Scope: `**/*.py`, `**/*.pyi`
