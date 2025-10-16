# Best Practices for Python Development with uv

REF:  https://docs.astral.sh/uv/

**Recommendation on uv Adoption**
- Ongoing team projects: Stick with our current setup (pyenv + venv + pip + requirements.txt). It remains the most robust, mature, and flexible solution for most Python teams—especially for ongoing, multi-developer projects.
- New/Research projects: Feel free to experiment with uv and let me know your thoughts. If  we switch tools, we’ll decide and move as a team.
- Current: not forced to use it, but need to be aware its existance. 

**Summary:**
Test uv on your own, but for production, we stay with the classic approach until (and unless) we all agree to change.

## Table of Contents
- [Best Practices for Python Development with uv](#best-practices-for-python-development-with-uv)
  - [Table of Contents](#table-of-contents)
  - [1. Introduction](#1-introduction)
    - [What is uv?](#what-is-uv)
    - [Core Features](#core-features)
  - [2. Installation](#2-installation)
    - [Recommended Method](#recommended-method)
  - [3. Basic Usage Modes](#3-basic-usage-modes)
    - [A. As pip/venv Replacement](#a-as-pipvenv-replacement)
    - [B. As Project/Dependency Manager (pyproject.toml Workflow)](#b-as-projectdependency-manager-pyprojecttoml-workflow)
    - [C. As Python Version Manager](#c-as-python-version-manager)
    - [D. CLI Tools (uv tool and uvx)](#d-cli-tools-uv-tool-and-uvx)
    - [E. Workspace/Monorepo Support](#e-workspacemonorepo-support)
  - [4. CI/CD \& Docker Integration](#4-cicd--docker-integration)
    - [A. GitHub Actions (Sample)](#a-github-actions-sample)
    - [B. Dockerfile Example](#b-dockerfile-example)
  - [5. Example Workflow](#5-example-workflow)
  - [7.  Further Reading \& Resources](#7--further-reading--resources)

---

## 1. Introduction

### What is uv?
- An ultra-fast, all-in-one Python project manager, dependency resolver, package installer, and Python version manager written in Rust.
- Developed by Astral, creators of ruff.
- Designed to replace tools like pip, virtualenv, pipx, pyenv, pip-tools, and Poetry/PDM, with a modern, unified interface.


### Core Features
- pip-compatible CLI: Use uv pip ... as a drop-in for classic workflows.
- Project/Monorepo Management: pyproject.toml-driven, supports workspaces.
- Python Version Management: Install and pin multiple Python versions.
- Global CLI Tooling: Like pipx—run or install CLI tools globally.
- Script Runner: Run single-file scripts with dependencies.
- Extensible: Great for CI, Docker, VS Code, and advanced workflows.
---

## 2. Installation

### Recommended Method

- macOS/Linux:
    ``` 
    curl -LsSf https://astral.sh/uv/install.sh | sh
    ``` 

- Windows (PowerShell):
    ```
    irm https://astral.sh/uv/install.ps1 | iex
    ```

The script automatically adds uv to your path.


- Updating uv
    ```
    uv self update
    ```

---

## 3. Basic Usage Modes

### A. As pip/venv Replacement
- Create a Virtual Environment:
    ```
    uv venv           # Creates .venv with default Python
    uv venv --python 3.12  # Specify Python version (downloads if missing)
    ```
    
- Activate Environment:
  ```
  source .venv/bin/activate      # macOS/Linux
  .venv\Scripts\activate         # Windows
  ```

- Install/Remove/List Packages:
    ```
    uv pip install requests flask
    uv pip uninstall flask
    uv pip list
    ```

- requirements.txt Workflow:
    ```
    uv pip install -r requirements.txt
    uv pip freeze > requirements.txt
    ```

- run script
  ```
  uv run python3 script.py
  ```
  Equals to
  ```
  source .venv/bin/activate
  python3 script.py
  ```

- Compile and Sync (Reproducibility):
    ```
    uv pip compile requirements.in -o requirements.txt  # Like pip-compile
    uv pip sync requirements.txt    # Like pip-sync (removes extras)
    ```


### B. As Project/Dependency Manager (pyproject.toml Workflow)
- Start a New Project:
    ```
    uv init 
    ```

- Add Dependencies:
    ```
    uv add requests
    uv add --dev pytest
    uv add --optional httpx --group network
    ```

- Remove Dependencies:
    ```
    uv remove requests
    ```

- Lock & Sync:
    ```
    uv lock      # Update uv.lock from pyproject.toml
    uv sync      # Sync .venv to lockfile
    ```

- Run Commands in Environment:
    ```
    uv run pytest
    uv run python script.py
    ```

- Show Dependency Tree:
    ```
    uv tree
    ```



### C. As Python Version Manager 

- Install & List Python Versions:
    ```
    uv python install 3.12 3.11
    uv python list
    ```

- Pin Version for Project:
    ```
    uv python pin 3.12    # writes .python-version
    ```

- Check Which Python is Used:
    ```
    uv python which
    ```


### D. CLI Tools (uv tool and uvx)
- Run a Tool Ephemerally (like npx):
    ```
    uvx pycowsay "Hello"
    ```

- Install a CLI Tool Globally:
    ```
    uv tool install ruff
    ruff --version
    ```

- Remove a Tool:
    ```
    uv tool uninstall ruff
    ```


### E. Workspace/Monorepo Support
- Add Multiple Projects:
In pyproject.toml, define [tool.uv.workspace] for monorepo-style management.
- Details: https://docs.astral.sh/uv/guides/workspaces/



---

## 4. CI/CD & Docker Integration

### A. GitHub Actions (Sample)
```yaml
jobs:
    test:
        runs-on: ubuntu-latest
        steps:
        - uses: actions/checkout@v4
        - run: curl -LsSf https://astral.sh/uv/install.sh | sh
        - run: uv sync
        - run: uv run pytest
```    

### B. Dockerfile Example
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY pyproject.toml uv.lock ./
RUN pip install uv && uv sync
COPY . .
CMD ["uv", "run", "python", "main.py"]
```
- Tip: Use multi-stage builds and layer ordering to cache dependencies for faster rebuilds.

---



## 5. Example Workflow

- New Project
    ```
    uv init myproject
    cd myproject
    uv python pin 3.12
    uv add fastapi uvicorn --dev pytest
    uv run pytest
    uv remove pytest
    uv build
    # In VS Code, select the .venv interpreter and go!
    ```

- For existing projects:
  ```
  uv pip install -r requirements.txt
  ```
  Then switch to uv add/remove, uv lock, and uv sync for future work.

---

## 7.  Further Reading & Resources
- Official uv Documentation https://docs.astral.sh/uv/
- uv GitHub Repo https://github.com/astral-sh/uv
- uv in Docker https://docs.astral.sh/uv/guides/docker/
- uv for Monorepos https://docs.astral.sh/uv/guides/docker/
  