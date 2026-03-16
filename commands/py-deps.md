# /py-deps — Python Dependency Management
 
Manage Python dependencies for: **$ARGUMENTS**
(e.g., "add LangChain", "upgrade all packages", "audit unused deps", "set up project from scratch")
 
## Instructions
 
Use `uv` as the primary package manager — it's significantly faster than pip and handles virtual envs cleanly. Fall back to pip + `requirements.txt` if the project already uses it.
 
---
 
## Audit Current State
 
```bash
# List installed packages and versions
uv pip list
# or
pip list
 
# Check for outdated packages
pip list --outdated
# or with uv:
uv pip list --outdated
 
# Check for security vulnerabilities
pip install pip-audit
pip-audit
```
 
---
 
## Project Setup (uv — recommended)
 
```bash
# Install uv
curl -LsSf https://astral.sh/uv/install.sh | sh
 
# Create project
uv init personal-ai
cd personal-ai
 
# Create and activate venv
uv venv
source .venv/bin/activate  # macOS/Linux
.venv\Scripts\activate     # Windows
 
# Add a dependency
uv add fastapi
uv add "ollama>=0.3.0"
uv add --dev pytest pytest-asyncio httpx
```
 
---
 
## Core Stack for Personal AI Project
 
```bash
# Runtime
uv add fastapi uvicorn[standard]   # API server
uv add ollama                       # Ollama Python client
uv add pydantic pydantic-settings  # Data models + config
uv add sqlalchemy[asyncio] aiosqlite  # Async DB (SQLite dev)
uv add python-dotenv                # .env support
uv add httpx                        # Async HTTP client
 
# Dev / test
uv add --dev pytest pytest-asyncio pytest-cov
uv add --dev ruff mypy              # Linting + type checking
uv add --dev pre-commit             # Git hooks
```
 
---
 
## pyproject.toml Structure
 
```toml
[project]
name = "personal-ai"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = [
    "fastapi>=0.111.0",
    "uvicorn[standard]>=0.30.0",
    "ollama>=0.3.0",
    "pydantic>=2.0.0",
    "pydantic-settings>=2.0.0",
    "sqlalchemy[asyncio]>=2.0.0",
    "aiosqlite>=0.20.0",
    "python-dotenv>=1.0.0",
]
 
[project.optional-dependencies]
dev = [
    "pytest>=8.0.0",
    "pytest-asyncio>=0.23.0",
    "pytest-cov>=5.0.0",
    "httpx>=0.27.0",
    "ruff>=0.5.0",
    "mypy>=1.10.0",
]
 
[tool.ruff]
line-length = 100
target-version = "py311"
 
[tool.ruff.lint]
select = ["E", "F", "I", "UP", "B", "SIM"]
 
[tool.mypy]
python_version = "3.11"
strict = true
 
[tool.pytest.ini_options]
asyncio_mode = "auto"
testpaths = ["tests"]
```
 
---
 
## Upgrade Dependencies
 
```bash
# Upgrade a specific package
uv add "fastapi>=0.115.0"
 
# Upgrade all (careful — check breaking changes)
uv lock --upgrade
uv sync
```
 
For each upgrade:
1. Check CHANGELOG for breaking changes
2. `uv sync` to install
3. `pytest` — run tests
4. `ruff check .` — no new lint errors
5. `mypy .` — no new type errors
 
---
 
## Remove a Dependency
 
```bash
uv remove package-name
# Updates pyproject.toml and uv.lock automatically
```
 
---
 
## Useful AI / LLM Packages (add when needed)
 
| Purpose | Package | Notes |
|---------|---------|-------|
| Vector DB | `chromadb` | Local embeddings + search |
| Embeddings | `sentence-transformers` | Local embedding models |
| LLM framework | `langchain` | If orchestration gets complex |
| LLM framework | `llama-index` | Better for RAG |
| Streaming | Built into `ollama` | No extra package needed |
| Rate limiting | `slowapi` | FastAPI rate limiter |
| Background tasks | `celery` + `redis` | Heavy async jobs |
| Caching | `cachetools` | In-memory TTL cache |
 
---
 
## Output
| Action | Package | Old Version | New Version | Notes |
|--------|---------|-------------|-------------|-------|
| Added | ollama | — | 0.3.0 | Ollama Python client |
| Upgraded | fastapi | 0.111.0 | 0.115.0 | No breaking changes |
| Removed | requests | 2.31.0 | — | Replaced by httpx |
