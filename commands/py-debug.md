# /py-debug — Python / AI Debug Session
 
Debug the following issue: **$ARGUMENTS**
 
## Instructions
 
Structured debugging for Python + Ollama AI. Work through each phase. Don't guess — confirm with evidence before moving on.
 
---
 
## Phase 1: Reproduce
 
**Goal**: Reliable reproduction before touching code.
 
- Does it reproduce every time or intermittently?
- Is it a crash, wrong output, hang, or performance issue?
- Which endpoint / function / model triggers it?
- Does it happen with all models or a specific one?
 
Collect:
```bash
# Run with full traceback visible
python -m uvicorn main:app --reload --log-level debug
 
# Check if Ollama is running and model is available
ollama list
ollama ps
 
# Test Ollama directly (bypass your code)
curl http://localhost:11434/api/chat -d '{
  "model": "llama3",
  "messages": [{"role": "user", "content": "Hello"}]
}'
```
 
---
 
## Phase 2: Locate
 
**Goal**: Narrow down which layer the problem is in.
 
```
Is Ollama responding correctly? → test with curl above
  Yes → problem is in your code
  No  → Ollama issue (model, config, resources)
 
Is it a Python exception? → check full traceback
Is it wrong output? → check prompt template and model response
Is it a hang? → likely async blocking or no timeout
Is it only under load? → likely concurrency or connection pool issue
```
 
Narrow with targeted logging:
```python
import logging
logger = logging.getLogger(__name__)
 
logger.debug("Prompt sent to Ollama: %s", messages)
logger.debug("Raw Ollama response: %s", response)
logger.debug("Conversation history length: %d", len(history))
```
 
Enable asyncio debug mode to catch blocking calls:
```bash
PYTHONASYNCIODEBUG=1 python -m uvicorn main:app --reload
```
 
---
 
## Phase 3: Understand
 
**Goal**: Know exactly WHY it fails before writing a fix.
 
Common Python / AI culprits:
 
- **Async blocking**: `time.sleep()`, `requests.get()`, synchronous file I/O inside `async def` → blocks event loop
- **Context window exceeded**: sending too much history → Ollama error or truncated response
- **Prompt injection**: user input containing "Ignore previous instructions..." → unexpected model behavior
- **Streaming not consumed**: generator not iterated → response never sent
- **Pydantic validation silently coercing**: wrong type passed in, Pydantic converts it silently
- **Event loop issues**: `asyncio.run()` called inside already-running loop → `RuntimeError`
- **Ollama timeout**: model takes > N seconds, no timeout set → request hangs forever
- **Memory leak**: conversation history growing unbounded in memory → OOM over time
- **Model hallucination**: not a bug — model is non-deterministic, improve prompt instead
 
Use `pdb` / `breakpoint()` for sync code:
```python
breakpoint()  # drops into interactive debugger at this line
```
 
Use `asyncio` debugger for async:
```python
import asyncio
asyncio.get_event_loop().set_debug(True)
```
 
---
 
## Phase 4: Fix
 
**Goal**: Smallest correct change that fixes root cause, not the symptom.
 
```python
# Blocking call in async → fix with asyncio.to_thread
# WRONG
async def process(text: str) -> str:
    return heavy_cpu_work(text)  # blocks event loop
 
# RIGHT
async def process(text: str) -> str:
    return await asyncio.to_thread(heavy_cpu_work, text)
 
# No timeout → fix with asyncio.wait_for
# WRONG
response = await ollama_client.chat(model, messages)
 
# RIGHT
response = await asyncio.wait_for(
    ollama_client.chat(model, messages),
    timeout=30.0
)
```
 
---
 
## Phase 5: Verify
 
After the fix:
- [ ] Reproduce original scenario — bug gone?
- [ ] Run tests: `pytest`
- [ ] Test with multiple Ollama models
- [ ] Test edge cases: empty input, long input, Ollama down, concurrent requests
- [ ] Check logs — no new warnings or unhandled exceptions
- [ ] Run with `PYTHONASYNCIODEBUG=1` — no new blocking warnings
 
---
 
## Phase 6: Document
 
If non-trivial:
- Add a code comment explaining WHY
- Use `/doc` to write a learning note to Notion
- Add a regression test to prevent recurrence
