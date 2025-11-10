## Recommended FastAPI project structure (pragmatic, AI-friendly)

Use this as a simple, scalable baseline. Prefer co-locating feature code; keep endpoints thin; avoid over‑engineering.

### Table of contents
- Project tree (canonical)
- Notes (naming, endpoints, configuration, testing)
- Minimal router example
- Wire routers in the app
- Dependency injection guidance
- Services pattern
- Full main.py example (canonical)
- Module-level models and schemas (co‑located option)
- Alternative main.py variant (user snippet)

### Project tree (canonical)
```
<project-name>/
│
├── app/
│   ├── __init__.py
│   ├── main.py                  # FastAPI entrypoint (initialises app, middlewares, lifespan)
│   │
│   ├── api/                     # Versioned routers and request-scoped dependencies
│   │   ├── v1/
│   │   │   ├── endpoints/
│   │   │   │   ├── module1_router.py
│   │   │   │   ├── module2_router.py
│   │   │   │   └── ...
│   │   │   └── dependencies/    # e.g. auth dependency, rate limiters, request id
│   │   └── v2/                   # optional future version
│   │
│   ├── core/                    # Core app setup/configuration
│   │   ├── config.py            # Settings via Pydantic (env + defaults)
│   │   ├── logging.py           # Structured logging config
│   │   ├── security.py          # OAuth2/JWT, password hashing, roles
│   │   └── middleware.py        # Custom middlewares (e.g. correlation id)
│   │
│   ├── models/                  # Domain models (Pydantic/SQLAlchemy)
│   ├── schemas/                 # Request/response Pydantic schemas
│   ├── services/                # Orchestration/business logic layer
│   │
│   ├── module1/                     # LLM provider adapters + prompt tooling
│   ├── module2/                     # Retrieval-augmented generation components
│   │
│   ├── workers/                 # Background tasks and scheduling
│   │   ├── tasks.py             # e.g. ingestion, re-index, batch inference
│   │   └── scheduler.py         # APScheduler/Celery/RQ (choose one)
│   │
│   ├── db/
│   │   ├── base.py              # SQLAlchemy base or ODM base
│   │   ├── session.py           # Engine/session factory
│   │   └── init_db.py           # Seed/admin creation
│   │
│   └── commons/                 # Reusable helpers/utilities
│
├── scripts/
│   ├── prestart.sh              # Optional health checks, migration run
│   └── populate_example_data.py
│
├── configs/
│   ├── settings.example.yaml    # Optional file-based config (alongside .env)
│   ├── logging.ini              # For uvicorn/gunicorn if used
│   └── gunicorn_conf.py         # If deploying with gunicorn/uvicorn workers
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
│
├── Dockerfile
├── .env.example
├── pyproject.toml               # Prefer uv+PEP 621; ruff/mypy/pytest config here
├── README.md
└── Makefile                     # Or simple scripts via `uv run`
```

### Notes
- **Naming**
  - Use `endpoints/` (or `routers/`) for modules that expose `APIRouter`. File names should be valid Python identifiers (e.g. `module1_router.py`, not `module1-router.py`).
  - Replace `module1/`, `module2/` with meaningful domain names as your project grows (e.g. `llm/`, `rag/`, `billing/`). Keep the top-level app simple; only add new directories when a need repeats.

- **Keep endpoints thin**
  - Validate inputs with Pydantic, delegate to services for business logic. Avoid SDK logic in the endpoint files.

- **Configuration**
  - Centralise settings in `app/core/config.py` using `pydantic-settings`. Prefer environment variables; optionally supplement with a `configs/settings.example.yaml`.

- **Testing**
  - Unit test services and adapters without network by default. Use integration tests for API + DB. Keep a small set of end-to-end tests.

### Minimal router example
```python
# app/api/v1/endpoints/module1_router.py
from fastapi import APIRouter, Depends

router = APIRouter(tags=["module1"])

@router.get("/module1/ping")
def ping() -> dict:
    return {"status": "ok"}
```

### Wire routers in the app
```python
# app/main.py
from fastapi import FastAPI
from .api.v1.endpoints import module1_router


def create_app() -> FastAPI:
    app = FastAPI(title="Service")
    app.include_router(module1_router.router, prefix="/api/v1")
    return app


app = create_app()
```

### Dependency injection guidance
- Request-scoped concerns (auth, rate limits, DB session) via FastAPI dependencies under `api/v1/dependencies/`.
- Long‑lived dependencies (e.g. provider clients, caches) constructed at startup and injected into services (constructor injection), then exposed via small provider functions if needed by endpoints.

### Services pattern (thin endpoints, testable logic)
```python
# app/services/example_service.py
class ExampleService:
    def __init__(self, store):
        self._store = store

    def do_work(self, x: int) -> int:
        return x * 2
```

```python
# app/api/v1/endpoints/module2_router.py
from fastapi import APIRouter, Depends
from ...services.example_service import ExampleService

router = APIRouter(tags=["module2"])

def get_example_service() -> ExampleService:
    # Construct or fetch from container; simplified here
    return ExampleService(store=None)

@router.get("/module2/double")
def double(x: int, svc: ExampleService = Depends(get_example_service)) -> dict:
    return {"result": svc.do_work(x)}
```

### Suggested tooling (lightweight)
- Packaging and scripts: `uv`
- Linting: `ruff`; Types: `mypy`; Tests: `pytest`
- Keep it boring; prefer consistency over complexity.

### Full `main.py` example (canonical: CORS, logging, lifespan, env-based run)
```python
# app/main.py
import logging
import os
import ssl
from contextlib import asynccontextmanager

import uvicorn
from dotenv import load_dotenv
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

# Routers (adapt names/files to your project)
from .api.v1.endpoints.database import router as database_router
from .api.v1.endpoints.search import router as search_router
# from .api.v1.endpoints.natural_language import router as natural_language_router
from .api.v1.endpoints.trending import router as trending_router
from .api.v1.endpoints.community import router as community_router
from .api.v1.endpoints.explore import router as explore_router

# DB session lifecycle (Mongo example; swap for SQL as needed)
from .db.session import connect_to_mongo, close_mongo_connection


# Configure logging
logging.basicConfig(
    filename="app.log",
    level=logging.DEBUG,
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s",
)
logger = logging.getLogger(__name__)

load_dotenv()

# Optional: disable SSL verification (avoid in production)
ssl._create_default_https_context = ssl._create_unverified_context

description = """
Example service with database connectivity and multiple routers.
"""

tags_metadata = [
    {"name": "Database", "description": "Database connection endpoints"},
    {"name": "Trending", "description": "Trending insight endpoints"},
    {"name": "Community", "description": "Community search endpoints"},
]


@asynccontextmanager
async def lifespan(app: FastAPI):
    await connect_to_mongo()
    yield
    await close_mongo_connection()


app = FastAPI(
    title="Service",
    description=description,
    version="0.0.1",
    lifespan=lifespan,
    openapi_tags=tags_metadata,
)

origins = ["*"]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE", "OPTIONS"],
    allow_headers=["*"],
    expose_headers=["*"],
)


@app.get("/")
def index():
    return {"message": "backend server is running."}


# Routers
app.include_router(database_router, tags=["Database"])
app.include_router(search_router, tags=["Search"])
# app.include_router(natural_language_router, tags=["Natural Language"])
app.include_router(trending_router, tags=["Trending"], prefix="/trending")
app.include_router(community_router, tags=["Community"], prefix="/community")
app.include_router(explore_router, tags=["Explore"])


if __name__ == "__main__":
    env = os.getenv("ENV", "development")
    if env == "production":
        uvicorn.run("app.main:app", host="0.0.0.0", port=8000, workers=2)
    else:
        uvicorn.run(
            "app.main:app",
            host="127.0.0.1",
            port=8000,
            reload=True,
            log_level="debug",
            workers=1,
        )
```

### Module-level models and schemas (co‑located option)
If your features are clearly bounded (e.g. "module1", "module2"), co‑locating their models/schemas with services keeps changes local and imports clearer.

```
app/
  api/
    v1/
      endpoints/
        module1_router.py
        module2_router.py
  module1/
    __init__.py
    models/            # SQLAlchemy/ODM models specific to module1
    schemas/           # Pydantic request/response models for module1
    services/
      __init__.py
      service.py
    repos.py
  module2/
    __init__.py
    models/
    schemas/
    services/
      __init__.py
      service.py
  models/              # Shared, cross-module DB models (optional)
  schemas/             # Shared, cross-module schemas (optional)
```

- **When to colocate**: Most models are only used by one feature. Keep them under that feature.
- **When to centralise**: A model or schema is shared by multiple features or external packages.

Example endpoint importing co‑located types/services:
```python
# app/api/v1/endpoints/module1_router.py
from fastapi import APIRouter, Depends
from ...module1.schemas.request import CreateItemRequest  # e.g. under module1/schemas/request.py
from ...module1.schemas.response import ItemResponse
from ...module1.services.service import Module1Service

router = APIRouter(tags=["module1"])

def get_service() -> Module1Service:
    return Module1Service()

@router.post("/module1/items", response_model=ItemResponse)
def create_item(payload: CreateItemRequest, svc: Module1Service = Depends(get_service)) -> ItemResponse:
    return svc.create_item(payload)
### Alternative `main.py` variant (user snippet)
```python
from routes.database_routes import database_router
from routes.search_routes import search_router
from database import connect_to_mongo, close_mongo_connection
import uvicorn
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
import ssl
from dotenv import load_dotenv
from contextlib import asynccontextmanager
import logging
import os

# Configure logging
logging.basicConfig(
    filename='app.log',  # Log file name
    level=logging.DEBUG,  # Log level
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'  # Log format
)

logger = logging.getLogger(__name__)
load_dotenv()


# %% ----------------TAG CONFIG ----------------
ssl._create_default_https_context = ssl._create_unverified_context

description = """
The Eurovision song contest analysis project is a comprehensive analysis of the Eurovision song contest, including data retrieval, processing, and analysis.
"""

tags_metadata = [
    {
        "name": "Database",
        "description": "Database connection endpoints",
    },
    {
        "name": "Trending",
        "description": "Trending insight endpoints",
    },
    {
        "name": "Community",
        "description": "Community search endpoints",
    },
]

@asynccontextmanager
async def lifespan(app: FastAPI):
    await connect_to_mongo()
    yield
    await close_mongo_connection()

app = FastAPI(
    title="Multimodal Analytics: Eurovision Song contest",
    description=description,
    version='0.0.1',
    lifespan=lifespan,
)

origins = [
    "*"
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE", "OPTIONS"],  # Specify needed methods
    allow_headers=["*"],
    expose_headers=["*"]
)


@app.get("/")
def index():
    return {"message": "fastapi MMA backend server is running."}

app.include_router(database_router, tags=["Database"])
app.include_router(search_router, tags=["Search"])
app.include_router(explore_router, tags=["Explore"])

# %% ----------------RUN SERVER ----------------
if __name__ == "__main__":
    env = os.getenv("ENV", "development")
    if env == "production":
        uvicorn.run("main:app", host="0.0.0.0", port=8000, workers=2)
    else:
        uvicorn.run("main:app", host="127.0.0.1", port=8000, reload=True, log_level="debug", workers=1)
```