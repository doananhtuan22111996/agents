# /py-perf — Python / AI Performance Audit
 
Run a performance audit on: **$ARGUMENTS**
(e.g., "chat endpoint is slow", "memory grows over time", "streaming latency high")
 
## Instructions
 
Measure first — don't optimize blindly. AI inference is slow by nature; optimize everything around it.
 
---
 
## Phase 1: Measure
 
### API Response Time
```bash
# Single request latency
curl -w "\nTime: %{time_total}s\n" -X POST http://localhost:8000/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello"}'
 
# Load test with concurrent users
pip install httpx
# or use: hey, wrk, locust
locust -f locustfile.py --headless -u 10 -r 2 --run-time 30s
```
 
### Ollama Inference Speed
```bash
# Baseline: how fast is Ollama directly?
time curl http://localhost:11434/api/generate -d '{
  "model": "llama3",
  "prompt": "Hello",
  "stream": false
}'
# If this is slow → hardware/model issue, not your code
```
 
### Memory Usage
```bash
# Profile memory over time
pip install memray
memray run -o output.bin -m uvicorn main:app
memray flamegraph output.bin
```
 
### CPU / Async Blocking
```bash
# Find blocking calls in async code
pip install py-spy
py-spy top --pid $(pgrep -f uvicorn)
```
 
---
 
## Phase 2: AI / Ollama Latency Checklist
 
### Time to First Token (TTFT)
- [ ] Streaming enabled — user sees first token fast, not waiting for full response
- [ ] Prompt is concise — shorter system prompt = faster first token
- [ ] Context history trimmed — sending 10k tokens of history adds latency
- [ ] Model appropriate for task — `mistral` faster than `llama3:70b` for simple tasks
 
### Model Selection
- [ ] Use smallest model that produces acceptable quality for each task
- [ ] `codellama` for code tasks, `llama3` for general, `mistral` for speed
- [ ] Quantized models (Q4, Q5) for better speed/quality tradeoff
- [ ] Pre-pull models at startup — `ollama pull` in app init, not on first request
 
### Ollama Connection
- [ ] `AsyncClient` reused across requests — not created per request
- [ ] HTTP connection pool configured — not default single connection
- [ ] Ollama running on same machine or fast local network — no cloud latency
 
---
 
## Phase 3: Python Async Checklist
 
- [ ] No blocking I/O in `async def` — no `requests`, no `open()`, no `time.sleep()`
- [ ] All DB calls are async (`asyncpg`, `SQLAlchemy async`, `motor` for MongoDB)
- [ ] File I/O uses `aiofiles` or `asyncio.to_thread()`
- [ ] CPU-heavy work (text processing, parsing) in `asyncio.to_thread()` or separate process
- [ ] `asyncio.gather()` used to run independent operations concurrently
  ```python
  # SLOW: sequential
  result1 = await fetch_user(user_id)
  result2 = await fetch_settings(user_id)
 
  # FAST: concurrent
  result1, result2 = await asyncio.gather(
      fetch_user(user_id),
      fetch_settings(user_id)
  )
  ```
- [ ] Connection pools sized correctly for expected concurrency
 
---
 
## Phase 4: Memory Checklist
 
- [ ] Conversation history capped per session — not growing forever
  ```python
  MAX_HISTORY = 20  # keep last 20 turns
  history = history[-MAX_HISTORY:]
  ```
- [ ] No in-memory cache without eviction policy — use `cachetools.TTLCache`
- [ ] Large responses not stored in memory — stream and discard chunks
- [ ] SQLAlchemy sessions closed after use — no connection leak
- [ ] Background tasks clean up old conversations periodically
 
---
 
## Phase 5: API / FastAPI Checklist
 
- [ ] Pydantic v2 used — significantly faster validation than v1
- [ ] Response models explicitly defined — no unnecessary serialization
- [ ] Streaming uses `StreamingResponse` + async generator — not buffering
  ```python
  from fastapi.responses import StreamingResponse
 
  async def token_generator():
      async for chunk in ollama_client.stream(model, messages):
          yield f"data: {chunk}\n\n"
 
  return StreamingResponse(token_generator(), media_type="text/event-stream")
  ```
- [ ] Middleware stack is minimal — each middleware adds latency
- [ ] `gzip` compression for large non-streaming responses
 
---
 
## Output
 
**Measurements** (before and after):
| Metric | Before | After | Target |
|--------|--------|-------|--------|
| TTFT (time to first token) | ms | ms | < 500ms |
| Full response latency | ms | ms | depends on length |
| Memory (idle) | MB | MB | < 200MB |
| Memory (under load) | MB | MB | stable, no growth |
| Requests/sec | rps | rps | > 10 rps |
 
**Issues Found** (🔴 Critical / 🟡 Medium / 🟢 Minor):
- ...
 
**Changes Made**:
- ...
