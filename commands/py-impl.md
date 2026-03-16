# /py-impl — Python / AI Implementation Plan
 
Create an implementation plan for: **$ARGUMENTS**
 
## Instructions
 
Explore the codebase first. Understand existing patterns before writing a single line. This project is a Python AI assistant powered by Ollama — think about LLM interaction, streaming, conversation state, and future extensibility.
 
---
 
## 1. Requirement Check
- What exactly needs to be built? (restate concisely)
- Acceptance criteria?
- What is explicitly out of scope for this iteration?
 
## 2. Codebase Exploration
Before planning, identify:
- Existing modules most similar to this feature — follow their pattern
- Files to create vs modify
- Any new packages needed in `pyproject.toml` / `requirements.txt`?
- Any Ollama model changes or new model requirements?
- Any schema/DB changes? (migration needed?)
 
## 3. Architecture Layer Plan
 
```
API Layer (FastAPI)
  └── router/[feature].py       — HTTP endpoints, request/response models
  └── schemas/[feature].py      — Pydantic input/output models
 
Service Layer
  └── services/[feature].py     — business logic, orchestrates LLM + data
  └── prompts/[feature].py      — prompt templates for this feature
 
LLM Layer
  └── llm/ollama_client.py      — Ollama API wrapper (chat, generate, stream)
  └── llm/context_manager.py    — conversation history, context window limits
 
Data Layer
  └── models/[feature].py       — SQLAlchemy / Pydantic data models
  └── repositories/[feature].py — DB access (if persistence needed)
 
Core / Shared
  └── config.py                 — settings (model name, temperature, etc.)
  └── utils/[helper].py         — shared utilities
```
 
## 4. Data Flow
 
### Chat / Streaming response:
```
HTTP POST /chat
  → router validates request (Pydantic)
    → service builds prompt from template + history
      → context_manager trims history to fit context window
        → ollama_client.stream(model, messages)
          → yields token chunks
        → SSE stream to client
  → service saves conversation turn to DB
```
 
### Non-streaming:
```
HTTP POST /feature
  → router validates
    → service calls ollama_client.generate(model, prompt)
      → returns full response
    → service processes / formats response
  → router returns Pydantic response model
```
 
## 5. Pydantic Models (define upfront)
```python
class ChatRequest(BaseModel):
    message: str
    conversation_id: str | None = None
    model: str = settings.DEFAULT_MODEL
 
class ChatResponse(BaseModel):
    response: str
    conversation_id: str
    model: str
    tokens_used: int | None = None
```
 
## 6. Ollama Integration Pattern
```python
# Prefer streaming for conversational features
async def stream_chat(messages: list[dict], model: str) -> AsyncIterator[str]:
    async with ollama.AsyncClient() as client:
        async for chunk in client.chat(model=model, messages=messages, stream=True):
            if chunk.message.content:
                yield chunk.message.content
 
# Context window management — always trim before sending
def trim_to_context_limit(messages: list[dict], max_tokens: int) -> list[dict]:
    # Keep system prompt + trim oldest messages first
    ...
```
 
## 7. File Plan
| Action | File | Purpose |
|--------|------|---------|
| Create | `routers/feature.py` | FastAPI router |
| Create | `schemas/feature.py` | Pydantic models |
| Create | `services/feature_service.py` | Business logic |
| Create | `prompts/feature_prompt.py` | Prompt templates |
| Modify | `routers/__init__.py` | Register new router |
| Modify | `config.py` | New settings if needed |
 
## 8. Implementation Order
1. Pydantic schemas (request/response contracts)
2. Prompt template(s)
3. Service logic (Ollama call + processing)
4. FastAPI router + endpoint
5. Wire router into app
6. Unit tests: service + prompt logic
7. Manual test: curl or Postman
 
## 9. Edge Cases to Handle
- [ ] Ollama not running / model not pulled → clear error message
- [ ] Context window exceeded → trim conversation history
- [ ] Streaming interrupted mid-response → handle partial output
- [ ] Model returns empty / malformed response → fallback
- [ ] Long-running request timeout → async timeout with `asyncio.wait_for`
- [ ] Concurrent requests → stateless service, no shared mutable state
 
## 10. AI-Specific Considerations
- Prompt is deterministic — same input should produce consistent behavior
- Temperature / top_p in config — not hardcoded per request
- System prompt locked in config — not overridable by user input (security)
- No PII in prompts sent to model if local Ollama (OK), but log carefully
- Test with multiple models (`llama3`, `mistral`, `codellama`) to verify behavior
 
## 11. Future Extensibility
- Will this feature need tool/function calling? Design interface now.
- Will it need RAG? Keep retrieval logic separate from generation logic.
- Will it need memory? Keep conversation storage behind a repository interface.
 
---
 
Proceed in the order above. Follow existing project conventions — check nearest similar feature first.
