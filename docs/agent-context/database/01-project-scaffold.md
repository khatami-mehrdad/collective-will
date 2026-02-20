# Task: Project Scaffold

## Depends on
Nothing — this is the first task.

## Goal
Create the Python project skeleton: dependency management, directory structure, configuration module, and environment template.

## Files to create

- `pyproject.toml` — project metadata, dependencies, tool config (ruff, mypy, pytest)
- `src/__init__.py`
- `src/config.py` — Pydantic Settings loading all env vars
- `src/models/__init__.py`
- `src/db/__init__.py`
- `src/channels/__init__.py`
- `src/handlers/__init__.py`
- `src/pipeline/__init__.py`
- `src/api/__init__.py`
- `.env.example` — template with all required env vars (no values)
- `.gitignore` — Python, Node, .env, IDE files

## Specification

### pyproject.toml

Use `uv` as the dependency manager. Key dependencies:

```
fastapi
uvicorn[standard]
sqlalchemy[asyncio]
asyncpg
pydantic
pydantic-settings
httpx
hdbscan
scikit-learn
numpy
alembic
python-dotenv
```

Dev dependencies:
```
pytest
pytest-asyncio
mypy
ruff
```

Configure ruff, mypy (strict mode), and pytest in pyproject.toml.

### src/config.py

Use `pydantic_settings.BaseSettings` to load configuration from `.env`:

```python
class Settings(BaseSettings):
    database_url: str
    anthropic_api_key: str
    mistral_api_key: str
    deepseek_api_key: str
    evolution_api_key: str
    evolution_api_url: str = "http://localhost:8080"
    secret_pepper: str          # For HMAC tokenization of WhatsApp IDs

    model_config = SettingsConfigDict(env_file=".env")
```

Provide a `get_settings()` function with `@lru_cache` for singleton access.

### .env.example

```
DATABASE_URL=postgresql+asyncpg://collective:password@localhost:5432/collective_will
ANTHROPIC_API_KEY=
MISTRAL_API_KEY=
DEEPSEEK_API_KEY=
EVOLUTION_API_KEY=
EVOLUTION_API_URL=http://localhost:8080
SECRET_PEPPER=
```

## Constraints

- Do NOT hardcode any secret values.
- Do NOT add dependencies beyond what's listed unless strictly necessary.
- All `__init__.py` files can be empty for now.

## Tests

Write tests in `tests/test_config.py` covering:
- Settings loads successfully when all env vars are present
- Missing required env var raises a clear validation error
- `get_settings()` returns the same instance on repeated calls (caching works)
- `.env.example` contains all keys that `Settings` expects
