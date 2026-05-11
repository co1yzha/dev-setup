# Python project structure and size guidelines

# 📏 Python Script Length & Structure Guidelines

## 🧰 Runtime & Tooling Baseline
- **Python**: 3.13 (target runtime)
- **Project manager**: `uv` (single source of truth is `pyproject.toml`, versions pinned via `uv.lock`)
- **Web framework**: FastAPI `0.121.1` (pin this version for new services)


## 🔢 Script Length
- Keep each `.py` script no more than **300 lines**.
  - ✅ Prefer ≤ **200 lines** for readability and maintainability.
  - 📌 Allow up to **300 lines** only if the module is logically cohesive.

## 🧩 Code Structure
- DO NOT OVERENGINEERING the scripts or structure.
- Use **functions** to organise logic. Avoid long, linear scripts.
- Include a `main()` entry point for scriptable modules.
- Use **British spell** for coding


## 🚫 Avoid Anti-Patterns
- Avoid unstructured code blocks with no clear decomposition.
- Don't let business logic sprawl in the global scope.

## 🧱 Object-Oriented Development
- **Use OOP only when necessary and benefits for practice**, such as for:
  - AI agents
  - Reusable frameworks or stateful components
- Otherwise, prefer functional style and procedural scripting.

## ✅ Examples

### ✅ Structured and Modular
```python
def process():
    ...

def main():
    data = load()
    cleaned = clean(data)
    ...

if __name__ == "__main__":
    main()
```
