# /py-test — Python / AI Test Strategy & Test Plan
 
Create a test strategy and test plan for: **$ARGUMENTS**
 
## Instructions
 
Produce a full test plan. AI responses are non-deterministic — always mock the Ollama client in unit tests. Write actual test skeletons, not just descriptions.
 
---
 
**Title**: [PersonalAI] Test Plan — [Feature Name]
**Created**: [today's date]
 
---
 
## Test Scope
- **In scope**: What is being tested?
- **Out of scope**: What is explicitly not tested here?
 
---
 
## Unit Tests
 
**Stack**: pytest + pytest-asyncio + httpx + unittest.mock / pytest-mock
 
### Service Tests (mock Ollama — no real model calls)
```python
import pytest
from unittest.mock import AsyncMock, patch
from services.chat_service import ChatService
 
@pytest.mark.asyncio
async def test_chat_returns_response():
    mock_ollama = AsyncMock()
    mock_ollama.chat.return_value = {"message": {"content": "Hello!"}}
 
    service = ChatService(ollama_client=mock_ollama)
    result = await service.chat("Hello", conversation_id=None)
 
    assert result.response == "Hello!"
    mock_ollama.chat.assert_called_once()
 
@pytest.mark.asyncio
async def test_chat_trims_context_when_history_too_long():
    # Arrange: history with 100 messages
    # Act: call service
    # Assert: only last N messages sent to Ollama
    ...
 
@pytest.mark.asyncio
async def test_chat_handles_ollama_connection_error():
    mock_ollama = AsyncMock()
    mock_ollama.chat.side_effect = ConnectionError("Ollama not running")
 
    service = ChatService(ollama_client=mock_ollama)
 
    with pytest.raises(ServiceUnavailableError):
        await service.chat("Hello", conversation_id=None)
```
 
### Prompt Template Tests
```python
def test_prompt_includes_user_message():
    prompt = build_chat_prompt(
        system="You are a helpful assistant.",
        history=[],
        user_message="What is Python?"
    )
    assert "What is Python?" in prompt[-1]["content"]
    assert prompt[0]["role"] == "system"
 
def test_prompt_preserves_conversation_history():
    history = [
        {"role": "user", "content": "Hi"},
        {"role": "assistant", "content": "Hello!"}
    ]
    prompt = build_chat_prompt(system="...", history=history, user_message="How are you?")
    assert len(prompt) == 4  # system + 2 history + new user message
```
 
### Pydantic Schema Tests
```python
def test_chat_request_requires_message():
    with pytest.raises(ValidationError):
        ChatRequest()  # missing required field
 
def test_chat_request_defaults():
    req = ChatRequest(message="Hello")
    assert req.model == settings.DEFAULT_MODEL
    assert req.conversation_id is None
```
 
### Context Manager Tests
```python
def test_trim_history_keeps_system_prompt():
    messages = [{"role": "system", "content": "..."}] + [
        {"role": "user", "content": f"msg {i}"} for i in range(50)
    ]
    trimmed = trim_to_context_limit(messages, max_tokens=1000)
    assert trimmed[0]["role"] == "system"
    assert len(trimmed) < len(messages)
```
 
---
 
## API Tests (FastAPI)
 
```python
import pytest
from httpx import AsyncClient, ASGITransport
from main import app
 
@pytest.mark.asyncio
async def test_chat_endpoint_returns_200():
    async with AsyncClient(transport=ASGITransport(app=app), base_url="http://test") as client:
        response = await client.post("/chat", json={"message": "Hello"})
    assert response.status_code == 200
    assert "response" in response.json()
 
@pytest.mark.asyncio
async def test_chat_endpoint_rejects_empty_message():
    async with AsyncClient(transport=ASGITransport(app=app), base_url="http://test") as client:
        response = await client.post("/chat", json={"message": ""})
    assert response.status_code == 422  # Pydantic validation error
 
@pytest.mark.asyncio
async def test_streaming_endpoint_emits_chunks():
    async with AsyncClient(transport=ASGITransport(app=app), base_url="http://test") as client:
        async with client.stream("POST", "/chat/stream", json={"message": "Hello"}) as r:
            chunks = [chunk async for chunk in r.aiter_text()]
    assert len(chunks) > 0
```
 
---
 
## Manual Test Scenarios
 
| # | Scenario | Steps | Expected | Status |
|---|----------|-------|----------|--------|
| 1 | Basic chat | Send "Hello" | Gets response | [ ] |
| 2 | Multi-turn | 5 back-and-forth messages | Context maintained | [ ] |
| 3 | Long context | Send 50 messages | Old messages trimmed, no error | [ ] |
| 4 | Ollama down | Stop Ollama, send message | Clear error, no crash | [ ] |
| 5 | Model not pulled | Use unknown model | Clear error message | [ ] |
| 6 | Streaming | Chat with stream=true | Tokens arrive progressively | [ ] |
| 7 | Coding task | Ask to write Python code | Correct, runnable code | [ ] |
| 8 | Read file task | Provide file content, ask question | Correct answer | [ ] |
| 9 | Empty input | Send empty message | 422 validation error | [ ] |
| 10 | Concurrent requests | 10 parallel requests | All complete, no race condition | [ ] |
 
---
 
## pytest Configuration
```toml
# pyproject.toml
[tool.pytest.ini_options]
asyncio_mode = "auto"
testpaths = ["tests"]
 
[tool.coverage.run]
source = ["src"]
omit = ["tests/*"]
```
 
---
 
## AC Coverage Map
| Acceptance Criterion | Test Type | Test Name |
|---------------------|-----------|-----------|
| AC-01 | Unit | `test_chat_returns_response` |
| AC-02 | Manual | Scenario 4 (Ollama down) |
| AC-03 | API | `test_chat_endpoint_returns_200` |
