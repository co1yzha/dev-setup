## FastAPI AI project structure (large, pragmatic)

A practical, scalable layout for AI-heavy FastAPI backends. Designed to stay simple, avoid over‑engineering, and grow cleanly.

### Goals
- **Clarity**: Clear boundaries between API, core app setup, business logic, and AI concerns
- **Testability**: Domain-first, dependency-injected services with thin endpoints
- **Replaceability**: Swap LLM/vector providers behind adapters
- **Observability**: First-class logging, metrics, tracing

### Directory tree (suggested)
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
│   │   │   │   ├── health.py
│   │   │   │   ├── auth.py
│   │   │   │   ├── inference.py # LLM/generation endpoints
│   │   │   │   ├── rag.py       # Retrieval-augmented generation endpoints
│   │   │   │   └── admin.py
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
│   │   ├── inference_service.py # Prompting/calls to llm providers
│   │   ├── rag_service.py       # Retrieval + generation orchestration
│   │   └── monitoring_service.py# Metrics/evaluations aggregation
│   │
│   ├── llm/                     # LLM provider adapters + prompt tooling
│   │   ├── providers/
│   │   │   ├── base.py          # Abstract interface for generation
│   │   │   ├── openai.py
│   │   │   ├── anthropic.py
│   │   │   ├── bedrock.py
│   │   │   └── local_ollama.py
│   │   ├── prompts/
│   │   │   ├── templates/       # Prompt templates (Jinja/format strings)
│   │   │   └── builders.py      # Utilities to compose prompts
│   │   └── utils.py             # Token/price accounting, truncation, safety checks
│   │
│   ├── rag/                     # Retrieval-augmented generation components
│   │   ├── retrievers/          # Vector/hybrid/keyword retrievers
│   │   │   ├── vectorstore_retriever.py
│   │   │   └── hybrid_retriever.py
│   │   ├── chunking/
│   │   │   └── strategies.py    # Chunking/splitting strategies
│   │   ├── indexing/
│   │   │   └── pipeline.py      # Indexing/ingestion pipeline
│   │   └── eval/
│   │       ├── metrics.py       # Faithfulness/answer relevance/latency
│   │       └── suites/          # Reusable eval suites & fixtures
│   │
│   ├── vectorstores/            # Vendor adapters (swap without touching services)
│   │   ├── chroma.py
│   │   ├── pinecone.py
│   │   └── opensearch.py
│   │
│   ├── events/                  # Async events/bus for decoupling
│   │   ├── bus.py               # Simple pub/sub abstraction
│   │   └── handlers/            # Side-effects (logging, analytics, audit)
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
│   ├── commons/                 # Reusable helpers/utilities
│   │   ├── utils.py
│   │   ├── errors.py            # AppError types and exception mapping
│   │   └── rate_limit.py
│   │
│   └── telemetry/               # Observability
│       ├── tracing.py           # OpenTelemetry setup
│       ├── metrics.py           # Prometheus/OTel metrics
│       └── ai_observability.py  # Model/feature metrics (latency, cost)
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

### Notes on key directories
- **app/api**: Keep endpoints thin. Validate with Pydantic, delegate to services. Group by version to enable non-breaking iteration.
- **app/core**: Single source of truth for settings and app wiring. Aim for pure functions returning configured components (e.g. `get_app()`).
- **app/services**: Orchestrate use cases. Inject providers (LLM, vector store) via constructor or FastAPI dependencies.
- **app/llm**: Define a small interface in `providers/base.py` (e.g. `generate`, `stream_generate`). Add adapters per vendor. Keep token accounting/safety checks here.
- **app/rag**: Separate retrieval concerns (chunking, indexing, retrieving, evaluation). The `rag_service` composes these for endpoints.
- **app/vectorstores**: Keep normalised CRUD/search against your chosen vector DB. Avoid leaking SDK specifics outside adapters.
- **app/events**: Use a minimal pub/sub (in-process or message broker) to decouple audit/analytics side-effects from request path.
- **app/workers**: Choose one worker stack (APScheduler for simple, Celery/RQ for distributed). Avoid mixing.
- **app/telemetry**: Treat observability as a feature. Emit latency, cost per request, cache hit-rate, and model usage.

### Configuration and dependency injection
- **Settings**: Centralise in `core/config.py` using `pydantic-settings` (env first; optional YAML for structured config).
- **DI**: Prefer FastAPI dependencies for request‑scoped concerns (auth, rate‑limit). Prefer constructor injection for services.
- **Feature flags**: Start simple (env vars). Add a file/provider only when needed.

### Data and storage
- Start with a single relational database (PostgreSQL) unless requirements demand otherwise.
- Use SQLAlchemy (sync or async) with a clean `session` helper. Keep migrations with Alembic when needed.
- Vector store: begin with an embedded option (e.g. Chroma) in dev, swap to managed (e.g. Pinecone) via adapter in prod.

### Testing
- **Unit**: Services and adapters with provider fakes. No network by default.
- **Integration**: API routes + DB + vector store (test container).
- **E2E**: Golden RAG inputs/outputs with tolerant assertions (semantic similarity, not exact strings).

### Tooling (suggested defaults)
- **Package & scripts**: `uv` for fast, reproducible environments and script running.
- **Lint**: `ruff`; **types**: `mypy`; **tests**: `pytest` with coverage.
- **Formatting**: `ruff format` (or `black` if preferred) — be consistent.

### Minimal provider interface (example)
```python
# app/llm/providers/base.py
from abc import ABC, abstractmethod
from typing import Iterable, Optional


class LLMProvider(ABC):
    """Thin interface to allow provider swapping without touching services."""

    @abstractmethod
    def generate(self, prompt: str, *, system: Optional[str] = None, **kwargs) -> str:
        ...

    @abstractmethod
    def stream_generate(self, prompt: str, *, system: Optional[str] = None, **kwargs) -> Iterable[str]:
        ...
```

### FastAPI app entry (shape)
```python
# app/main.py
from fastapi import FastAPI
from .api.v1.endpoints import health, inference, rag
from .core import middleware, logging as logging_setup


def create_app() -> FastAPI:
    logging_setup.configure_logging()
    app = FastAPI(title="AI Service")
    middleware.add_middlewares(app)
    app.include_router(health.router, prefix="/api/v1")
    app.include_router(inference.router, prefix="/api/v1")
    app.include_router(rag.router, prefix="/api/v1")
    return app


app = create_app()
```

### Scaling tips (when needed)
- **Caching**: Prompt/result cache at service layer; vector search cache for hot queries.
- **Cost controls**: Token budgeting per endpoint; guardrails on max output tokens; emit per‑request cost metrics.
- **Safety**: Input/output moderation pipeline (only if required by your domain/regulation).
- **Background**: Move ingestion, re‑indexing, and batch labelling to `workers`.

Keep it boring. Add new directories only when there is a clear, repeated need.


