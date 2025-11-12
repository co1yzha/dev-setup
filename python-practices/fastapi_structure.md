# 🧭 FastAPI Project Structure Guide

This document describes the **recommended structure** for this FastAPI-based backend.  
The layout is designed for **AI-driven, modular services** with clean separation between API, business logic, and infrastructure.  
It is lightweight, scalable, and suitable for multi-module projects such as LLM inference, RAG pipelines, analytics, or other service integrations.

---

## ⚡️ Quick Start

```bash
# 1. Clone and install
git clone <your-repo-url> my-fastapi-app
cd my-fastapi-app
uv sync

# 2. Set environment variables
cp .env.example .env

# 3. Run locally
uvicorn app.main:app --reload

# 4. Run tests
pytest -v
```

---

## 🎯 Design Principles

- **Clarity** – each directory has a clear purpose.
- **Modularity** – features are grouped into independent modules.
- **Testability** – keep endpoints thin and business logic isolated.
- **Replaceability** – easily swap AI/vector/database providers.
- **Observability** – first-class support for logging, metrics, and tracing.

---

## 📁 Directory Overview

**Add __init__.py under each folder**
```
<project-name>/
│
├── app/
│   ├── __init__.py
│   ├── main.py                    # FastAPI entrypoint (creates the app)
│   │
│   ├── api/                       # HTTP API layer (thin routers)
│   │   ├── v1/
│   │   │   ├── endpoints/         # Routers for versioned endpoints
│   │   │   │   ├── health.py
│   │   │   │   ├── inference_router.py
│   │   │   │   ├── rag_router.py
│   │   │   │   └── admin_router.py
│   │   │   └── dependencies/      # Auth, rate limiters, DB sessions, etc.
│   │   └── v2/                    # Future API versions (optional)
│   │
│   ├── core/                      # Global app setup and configuration
│   │   ├── config.py              # Settings via pydantic-settings
│   │   ├── logging.py             # Structured logging
│   │   ├── security.py            # JWT / OAuth2 setup
│   │   └── middleware.py          # Custom middleware registration
│   │
│   ├── commons/                   # Shared helpers and utilities
│   │   ├── utils.py
│   │   ├── errors.py
│   │   └── decorators.py
│   │
│   ├── db/                        # Database setup and lifecycle
│   │   ├── base.py
│   │   ├── session.py
│   │   └── init_db.py
│   │
│   ├── workers/                   # Background tasks / schedulers
│   │   ├── tasks.py
│   │   └── scheduler.py
│   │
│   ├── services/                  # Shared or cross-cutting logic
│   │   ├── user_service.py
│   │   ├── monitoring_service.py
│   │   └── ai_service.py
│   │
│   ├── modules/                   # Feature-level modules (isolated logic)
│   │   ├── llm/
│   │   │   ├── models/
│   │   │   ├── schemas/
│   │   │   ├── services/
│   │   │   ├── providers/
│   │   │   └── prompts/
│   │   │
│   │   ├── rag/
│   │   │   ├── models/
│   │   │   ├── schemas/
│   │   │   ├── services/
│   │   │   ├── retrievers/
│   │   │   └── indexing/
│   │   │
│   │   └── analytics/
│   │       ├── models/
│   │       ├── schemas/
│   │       └── services/
│   │
│   └── telemetry/                 # Observability
│       ├── tracing.py
│       ├── metrics.py
│       └── ai_observability.py
│
├── scripts/                       # Local or deployment scripts
│   ├── prestart.sh
│   └── populate_example_data.py
│
├── configs/                       # Optional YAML / INI configs
│   ├── settings.example.yaml
│   ├── logging.ini
│   └── gunicorn_conf.py
│
├── tests/                         # Unit / integration / e2e tests
│   ├── unit/
│   ├── integration/
│   └── e2e/
│
├── Dockerfile
├── .env.example
├── pyproject.toml
├── README.md
└── Makefile
```

---

## 🧩 How It Works

### 1. Application Entry Point

`app/main.py` creates the FastAPI app, configures logging and middleware, and mounts routers.

```python
from fastapi import FastAPI
from app.core import middleware, lifespan
from app.core.logging import configure_logging, get_logger
from app.api.router import api_router

def create_app() -> FastAPI:
    """Create and configure the FastAPI app."""
    configure_logging()

    app = FastAPI(
        title="AI Backend Service",
        description="LLM + RAG APIs backed by MongoDB",
        version="1.0.0",
        lifespan=lifespan.lifespan,       # ← lifespan moved out
        openapi_tags=api_router.tags_metadata,  # ← tags defined in api/
    )

    middleware.add_middlewares(app)
    app.include_router(api_router.router)  # all endpoints handled here

    @app.get("/")
    def root():
        return {"service": "ai-backend", "routers": ["health", "inference", "rag"]}

    return app


app = create_app()

# optional runner for local development
if __name__ == "__main__":
    import uvicorn
    uvicorn.run("app.main:app", host="0.0.0.0", port=8080, reload=True)
```


#### app/api/router.py
```python
from fastapi import APIRouter
from app.api.v1.endpoints.health import router as health_router
from app.api.v1.endpoints.inference import router as inference_router
from app.api.v1.endpoints.rag import router as rag_router

tags_metadata = [
    {"name": "health", "description": "Health and readiness probes."},
    {"name": "inference", "description": "LLM inference endpoints."},
    {"name": "rag", "description": "Retrieval-augmented generation APIs."},
]

router = APIRouter(prefix="/api/v1")
router.include_router(health_router, tags=["health"])
router.include_router(inference_router, tags=["inference"])
router.include_router(rag_router, tags=["rag"])
```

---

### 2. app/core/

#### app/core/lifespan.py
```python
from contextlib import asynccontextmanager
from fastapi import FastAPI
from app.db.mongo import get_async_client, close_async_client
from core.logging import get_logger

logger = get_logger(__name__)

@asynccontextmanager
async def lifespan(app: FastAPI):
    client = get_async_client()
    logger.info("Async MongoDB client initialised)
    try:
        yield
    finally:
        close_async_client()
        logger.info("Async MongoDB client closed.)
```

#### app/core/logging.py
```python
import logging
import os
import sys
from typing import Optional

def configure_logging(level: Optional[str] = None) -> None:
    level = (level or os.getenv("LOG_LEVEL", "INFO")).upper()
    app_name = os.getenv("APP_NAME", "fastapi-app")

    logging.basicConfig(
        level=level,
        format=f"{app_name} %(asctime)s | %(levelname)s | %(name)s | %(message)s",
        datefmt="%Y-%m-%d %H:%M:%S",
        handlers=[logging.StreamHandler(sys.stdout)],
        force=True,  # reset any prior config (useful under reload)
    )

    # Tame Uvicorn verbosity (keeps things quiet without custom handlers)
    logging.getLogger("uvicorn.access").setLevel("WARNING")
    logging.getLogger("uvicorn.error").setLevel("INFO")
    logging.getLogger("fastapi").setLevel("INFO")

def get_logger(name: str) -> logging.Logger:
    return logging.getLogger(name)
```
use log anywhere in the app

```python
from app.core.logging import get_logger

logger = get_logger(__name__)

class LLMService:
    def generate(self, query: str) -> str:
        logger.info("Generating LLM response...")
        try:
            # do something...
            result = f"Processed: {query}"
            logger.debug(f"Generated result: {result}")
            return result
        except Exception as e:
            logger.exception(f"Error in LLM generation: {e}")
            raise
```

#### app/core/middleware.py
```
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

def add_middlewares(app: FastAPI):
    app.add_middleware(
        CORSMiddleware,
        allow_origins=["*"],
        allow_credentials=True,
        allow_methods=["*"],
        allow_headers=["*"],
    )
```


#### Configuration [Optional]

Centralised in `app/core/config.py` using `pydantic-settings`.

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    app_name: str = "AI Backend"
    openai_api_key: str
    database_url: str

    class Config:
        env_file = ".env"

settings = Settings()
```


---

### 3. Thin Endpoints, Fat Services

Endpoints only validate and route data:

```python
# app/api/v1/endpoints/inference_router.py
from fastapi import APIRouter, Depends
from app.modules.llm.schemas.request import LLMQuery
from app.modules.llm.schemas.response import LLMResponse
from app.modules.llm.services.llm_service import LLMService

router = APIRouter(tags=["LLM"])

def get_service() -> LLMService:
    return LLMService()

@router.post("/llm/infer", response_model=LLMResponse)
def infer(payload: LLMQuery, svc: LLMService = Depends(get_service)) -> LLMResponse:
    return svc.generate(payload)
```

Business logic lives in `app/modules/<module>/services/`:

```python
# app/modules/llm/services/llm_service.py
class LLMService:
    def generate(self, query):
        return {"text": f"Processed: {query.prompt}"}
```

---

### 4. Module-Level Models and Schemas

Each module keeps its own models and schemas for easy maintenance.

```
app/
  modules/
    llm/
      models/
        llm_model.py
      schemas/
        request.py
        response.py
      services/
        llm_service.py
```

Use shared models only when multiple modules depend on the same data structure.

---

### 5. Dependency Injection

- **Request-scoped** dependencies (auth, rate limits, DB sessions): under `api/v1/dependencies/`  
- **Long-lived** dependencies (LLM clients, caches): created at startup and injected into services

This pattern keeps services testable and endpoints clean.

---


### 6. Testing

- **Unit tests** → `tests/unit/`  
- **Integration tests** → `tests/integration/`  
- **E2E tests** → `tests/e2e/`  

Example:

```bash
pytest -v --maxfail=1 --disable-warnings
```

---

### 7. Tooling

| Purpose | Tool | Config |
|----------|------|--------|
| Packaging | uv | pyproject.toml |
| Linting | ruff | [tool.ruff] |
| Type checking | mypy | [tool.mypy] |
| Tests | pytest | [tool.pytest.ini_options] |

---

### 8. Extending the App

To add a new module:

1. Create `app/modules/<new_module>/`
2. Add `models/`, `schemas/`, and `services/`
3. Create a router in `app/api/v1/endpoints/<new_module>_router.py`
4. Register it in `app/main.py`

---

## ✅ Summary

- Use `app/` for clarity and convention  
- Keep endpoints simple; logic lives in services  
- Co-locate models and schemas by feature  
- Shared utilities in `commons/`, config in `core/`  
- Expand only when needed

---

_This structure scales from prototypes to production AI microservices._
