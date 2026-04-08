# /py-review — Python / AI Code Review
 
Review the current code changes (or: $ARGUMENTS) against the full checklist.
 
## Instructions
 
Run `git diff HEAD` or inspect the specified files. Be critical — catch issues before opening a PR.
 
---
 
## ✅ Correctness
- [ ] Logic correct for all inputs: empty string, None, very long text, special characters
- [ ] Async functions use `await` correctly — no blocking calls inside `async def`
- [ ] No `asyncio.run()` inside an already running event loop
- [ ] Generator / `AsyncIterator` used correctly for streaming — no buffering entire response
- [ ] Pydantic models validate all edge cases (optional fields, default values)
- [ ] No mutable default arguments: `def f(x=[])` → use `def f(x=None)`
 
## 🔒 Security
- [ ] No secrets, API keys, or tokens in source code — use `.env` + `python-dotenv`
- [ ] No user input injected directly into system prompts (prompt injection risk)
- [ ] User input sanitized before being sent to Ollama
- [ ] No sensitive data (PII, passwords) logged via `print()` or `logging`
- [ ] FastAPI endpoints validate input with Pydantic — no raw `request.body` parsing
- [ ] CORS configured correctly — not `allow_origins=["*"]` in production
- [ ] No `eval()`, `exec()`, or `subprocess` with user-controlled input
 
## ⚡ Performance
- [ ] Streaming used for conversational responses — not waiting for full completion
- [ ] No synchronous blocking calls in async context (`time.sleep` → `asyncio.sleep`)
- [ ] Conversation history trimmed before each Ollama call — no unbounded context growth
- [ ] DB queries use async ORM (SQLAlchemy async) — not blocking the event loop
- [ ] No N+1 query patterns — batch fetch where possible
- [ ] Heavy CPU work (text processing, embeddings) offloaded to `asyncio.to_thread()`
- [ ] Ollama client reused (connection pooling) — not created per request
 
## 🛡️ Error Handling
- [ ] Ollama connection errors caught and return meaningful HTTP error (503, not 500)
- [ ] Model not found / not pulled caught gracefully — clear user-facing message
- [ ] Pydantic `ValidationError` handled at API boundary — not leaked as 500
- [ ] Streaming errors mid-response handled — don't silently drop partial output
- [ ] `asyncio.TimeoutError` handled for long-running model calls
- [ ] All exceptions logged with `logger.exception()` before re-raising
 
## 🧪 Tests
- [ ] Service logic tested with mocked Ollama client — no real model calls in unit tests
- [ ] Pydantic models tested with valid + invalid inputs
- [ ] Async tests use `@pytest.mark.asyncio` and `pytest-asyncio`
- [ ] FastAPI endpoints tested with `httpx.AsyncClient` + `TestClient`
- [ ] Streaming endpoints tested for correct chunk emission
- [ ] Tests are deterministic — AI responses mocked, not using real model
 
## 🧹 Code Quality (Python)
- [ ] Type hints on all function signatures (`def f(x: str) -> dict:`)
- [ ] No `Any` type unless genuinely unavoidable
- [ ] f-strings used for string formatting — not `%` or `.format()`
- [ ] List/dict comprehensions preferred over `map()`/`filter()` for readability
- [ ] `pathlib.Path` used for file paths — not `os.path`
- [ ] No bare `except:` — always catch specific exception types
- [ ] No dead code, commented-out code, or `print()` debug statements left in
- [ ] Docstrings on public functions and classes
 
## 🏗️ Architecture
- [ ] Prompt templates in dedicated `prompts/` module — not hardcoded in service
- [ ] Ollama client abstracted behind interface — swappable without touching services
- [ ] Business logic in service layer — not in FastAPI router
- [ ] Conversation history managed in a dedicated module — not scattered across services
- [ ] Config values from `settings` object — not hardcoded strings/numbers
- [ ] No circular imports
 
## 🤖 AI / LLM Specific
- [ ] System prompt locked and not modifiable by user input
- [ ] Model name and temperature from config — not hardcoded per call
- [ ] Context window limit enforced before sending to Ollama
- [ ] Response parsing handles empty / unexpected model output
- [ ] Prompt templates versioned or named — easy to track changes
- [ ] Different behavior tested across models (llama3, mistral, codellama)
 
---
 
## Summary
 
**Issues Found**
- 🔴 Must Fix: ...
- 🟡 Should Fix: ...
- 🟢 Nice to Have: ...
 
**Overall**: Ready to merge | Needs changes | Significant rework needed
