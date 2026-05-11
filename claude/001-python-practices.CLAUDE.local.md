# Python practices

## Commands
- Install deps: `uv sync`
- Run tests: `uv run pytest`
- Lint: `uv run ruff check .`
- Format: `uv run ruff format .`
- Type-check: `uv run mypy .`

## Runtime & tooling
- Python 3.13.
- Project manager: `uv` — `pyproject.toml` is the single source of truth, versions pinned via `uv.lock`.
- Web framework: FastAPI `0.121.1` (verify against latest stable before adopting in a new project).
- British spelling in code, comments, and identifiers (`colour`, `organise`, `behaviour`).

## Architecture
- Prefer functional and procedural style. Reach for OOP only when it earns its keep — AI agents, reusable frameworks, genuinely stateful components.
- Organise logic into named functions. Avoid long linear scripts and business logic in the global scope.
- Scriptable modules include a `main()` entry point guarded by `if __name__ == "__main__"`.

## File size
- Target ≤ 200 lines per `.py` file.
- 300 lines is the hard ceiling, and only when the module is logically cohesive.
- Split by responsibility, not by line count alone.

## Avoid
- Over-engineering. Don't reach for classes, decorators, or abstractions before they pay for themselves.
- Unstructured blocks with no clear decomposition.
- Module-level globals holding business state.

## Example structure
```python
def load(): ...
def clean(data): ...
def process(data): ...

def main() -> None:
    data = load()
    cleaned = clean(data)
    process(cleaned)

if __name__ == "__main__":
    main()
```
