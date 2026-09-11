# Phase 0 — Python Engineering for Agentic AI

## Objective

Build production-grade Python skills specifically for AI/Agentic AI engineering.

You already have DevOps, cloud, Kubernetes, Terraform, CI/CD, security, and observability experience. Therefore, this roadmap skips most beginner Python material and focuses on the engineering capabilities required to build reliable AI services and agent platforms.

## Six-Week Roadmap

| Week | Focus | Main Outcome |
|---|---|---|
| 1 | Python typing + clean code | Strongly typed agent state and domain models |
| 2 | Async Python | Understand and implement asynchronous I/O |
| 3 | Concurrency + multiprocessing | Choose the right execution model |
| 4 | Pydantic + FastAPI | Build validated production APIs |
| 5 | REST APIs + WebSockets | Build synchronous and real-time interfaces |
| 6 | pytest + logging + errors + packaging | Production-quality software |

---

# Week 1 — Python Typing + Clean Code

### Learn

- Type hints
- Optional
- Union
- Literal
- TypedDict
- Generic
- Protocol
- Callable
- TypeAlias
- TypeVar

### Why this matters for AI

Agent systems contain a lot of structured state.

```python
from typing import TypedDict

class AgentState(TypedDict):
    user_query: str
    current_step: str
    tool_result: str | None
    retry_count: int
```

### Exercise

Create `agent_state.py` and define:

- `AgentState`
- `ToolCall`
- `ToolResult`
- `AgentResponse`
- `ErrorResponse`

You should be able to explain `TypedDict`, `TypeVar`, `Generic`, `Protocol`, `Callable`, `Optional`, and `Literal`.

---

# Week 2 — Async Python

Async programming is extremely important for Agentic AI because agents frequently wait on external systems.

### Learn

```text
async
await
asyncio.run()
asyncio.create_task()
asyncio.gather()
asyncio.wait()
asyncio.sleep()
asyncio.Queue
asyncio.Semaphore
asyncio.Lock
```

### Example

```python
import asyncio

async def fetch_data(name):
    print(f"Starting {name}")
    await asyncio.sleep(2)
    print(f"Finished {name}")
    return name

async def main():
    results = await asyncio.gather(
        fetch_data("AWS"),
        fetch_data("Kubernetes"),
        fetch_data("GitHub"),
    )
    print(results)

asyncio.run(main())
```

Independent I/O operations can overlap, which is useful for agents making multiple external calls.

### Exercise

Create `async_devops_checks.py`.

Simulate:

- `check_aws()`
- `check_kubernetes()`
- `check_github()`
- `check_prometheus()`

Run them sequentially, then with `asyncio.gather()`, and compare execution time.

---

# Week 3 — Concurrency + Multiprocessing

Understand the difference between:

- Concurrency
- Parallelism
- Async I/O
- Threads
- Processes

### Mental model

| Work | Preferred approach |
|---|---|
| API calls | asyncio |
| Database calls | asyncio |
| HTTP requests | asyncio |
| Waiting for LLM | asyncio |
| File I/O | asyncio/thread depending on library |
| CPU-heavy processing | multiprocessing |
| Heavy computation | multiprocessing |
| Independent external operations | concurrency |

### Learn

```python
asyncio.gather()
asyncio.create_task()
asyncio.Semaphore()

concurrent.futures
ThreadPoolExecutor
ProcessPoolExecutor

multiprocessing
```

### Critical concept

> Async does not automatically mean faster.

Async mainly improves throughput when tasks spend time waiting for I/O. CPU-heavy work needs parallel execution or specialized workers.

### Exercises

Create:

- `io_concurrency.py`
- `cpu_parallelism.py`

Compare sequential execution, async/threaded execution for I/O work, and thread/process approaches for CPU-heavy work.

---

# Week 4 — Pydantic + FastAPI

Now convert your Python knowledge into an actual service.

## Pydantic

Learn:

- `BaseModel`
- `Field`
- Validation
- Nested models
- Optional fields
- Enums
- Custom validators
- Serialization
- JSON schema
- Settings/configuration

Example:

```python
from pydantic import BaseModel

class UserRequest(BaseModel):
    query: str
    max_tokens: int = 500
```

## FastAPI

```python
from fastapi import FastAPI

app = FastAPI()

@app.post("/ask")
async def ask(request: UserRequest):
    return {
        "answer": f"Processing: {request.query}"
    }
```

### Learn

- Path parameters
- Query parameters
- Request bodies
- Response models
- Validation
- Dependency injection
- Middleware
- Background tasks
- Authentication
- Exception handlers
- OpenAPI

### Exercise

Create:

```text
POST /analyze
```

Request:

```json
{
  "service": "payment-api",
  "environment": "production"
}
```

Response:

```json
{
  "service": "payment-api",
  "status": "degraded",
  "root_cause": "High memory utilization",
  "confidence": 0.91
}
```

---

# Week 5 — REST APIs + WebSockets

## REST

Build:

```text
POST   /agents
GET    /agents/{id}
POST   /agents/{id}/execute
GET    /agents/{id}/status
DELETE /agents/{id}
```

Understand:

```text
GET
POST
PUT
PATCH
DELETE
```

And:

```text
200 OK
201 Created
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
422 Validation Error
429 Too Many Requests
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
```

## WebSockets

REST is request/response:

```text
Client ───── request ─────> Server
Client <──── response ───── Server
```

WebSocket maintains a persistent connection:

```text
Client ═══════════════════ Server
```

Useful for:

- Agent streaming
- LLM tokens
- Agent progress
- Tool execution updates
- Real-time logs

Example:

```text
Connected
→ Starting analysis
→ Checking Kubernetes
→ Checking AWS
→ Checking GitHub
→ Checking metrics
→ Correlating results
→ Analysis complete
```

### Exercise

Create:

```text
WS /agents/{id}/stream
```

and stream agent progress events.

---

# Week 6 — pytest + Logging + Error Handling + Packaging

## pytest

Learn:

- Assertions
- Fixtures
- Parametrize
- Mocks
- Async tests
- Integration tests
- Test coverage

Example:

```python
def test_add():
    assert add(2, 3) == 5
```

Async test:

```python
@pytest.mark.asyncio
async def test_agent():
    result = await agent.run("hello")
    assert result.status == "success"
```

### Testing target

Build:

```text
Unit tests
+
Integration tests
+
API tests
+
Agent evaluation tests
```

---

## Logging

Avoid:

```python
print("Something happened")
```

Use structured logging:

```python
import logging

logger = logging.getLogger(__name__)

logger.info(
    "Tool execution started",
    extra={
        "tool": "kubernetes",
        "agent_id": "123"
    }
)
```

Learn:

- Log levels
- Structured logs
- Correlation IDs
- Request IDs
- JSON logging
- Sensitive-data filtering
- Centralized logging

---

## Error Handling

Understand:

- Expected errors
- Unexpected errors
- Transient errors
- Permanent errors
- Validation errors
- Timeouts
- Dependency failures

Production pattern:

```text
Retry
 ↓
Exponential backoff
 ↓
Timeout
 ↓
Circuit breaker
 ↓
Fallback
```

For agents:

```text
LLM failure
     ↓
Retry?
     ↓
Alternative model?
     ↓
Fallback?
     ↓
Human?
```

---

## Packaging

Learn modern Python project structure:

```text
agent-platform/
├── pyproject.toml
├── README.md
├── src/
│   └── agent_platform/
│       ├── api/
│       ├── agents/
│       ├── models/
│       ├── tools/
│       ├── services/
│       ├── config/
│       └── utils/
├── tests/
│   ├── unit/
│   └── integration/
└── .github/
    └── workflows/
```

Understand:

- `pyproject.toml`
- Virtual environments
- Dependencies
- Development dependencies
- Editable installs
- Package versions
- Semantic versioning

---

# Capstone Project — Async DevOps Intelligence API

Combine everything into one project.

```text
                 User
                   │
                   ▼
              FastAPI API
                   │
             Agent Service
                   │
       ┌───────────┼───────────┐
       ↓           ↓           ↓
      AWS        GitHub       K8s
       │           │           │
       └───────────┼───────────┘
                   ↓
              Agent Result
                   │
              WebSocket
                   ↓
                  User
```

## API

```text
POST /analyze
```

Request:

```json
{
  "service": "payment-api",
  "environment": "production"
}
```

Backend concurrently simulates:

- AWS check
- Kubernetes check
- GitHub check
- Prometheus check

Response:

```json
{
  "service": "payment-api",
  "status": "degraded",
  "root_cause": "High memory utilization",
  "confidence": 0.91
}
```

WebSocket:

```text
Connected
→ Starting analysis
→ Checking Kubernetes
→ Checking AWS
→ Checking GitHub
→ Checking metrics
→ Correlating results
→ Analysis complete
```

Add:

- Pydantic validation
- asyncio
- Concurrency
- Structured logging
- Custom exceptions
- pytest
- `pyproject.toml`
- Dockerfile

---

# Recommended Project Structure

```text
async-devops-intelligence/
├── pyproject.toml
├── README.md
├── Dockerfile
├── .gitignore
├── src/
│   └── app/
│       ├── main.py
│       ├── api/
│       │   ├── routes.py
│       │   └── websocket.py
│       ├── agents/
│       │   └── analyzer.py
│       ├── services/
│       │   ├── aws.py
│       │   ├── kubernetes.py
│       │   ├── github.py
│       │   └── prometheus.py
│       ├── models/
│       │   ├── requests.py
│       │   ├── responses.py
│       │   └── state.py
│       ├── exceptions/
│       │   └── errors.py
│       ├── config/
│       │   └── settings.py
│       └── utils/
│           └── logging.py
└── tests/
    ├── unit/
    └── integration/
```

---

# Learning Method

For every topic:

```text
1. Understand the concept
          ↓
2. Write a tiny example
          ↓
3. Break it intentionally
          ↓
4. Debug it
          ↓
5. Use it in the project
          ↓
6. Explain it without notes
```

Do not just memorize syntax.

---

# Definition of Done

## Python

- [ ] Write clean typed Python
- [ ] Design typed agent state
- [ ] Use Pydantic effectively
- [ ] Understand async execution
- [ ] Use asyncio correctly
- [ ] Explain concurrency vs parallelism
- [ ] Choose threads vs processes vs async
- [ ] Write production error handling
- [ ] Implement retries and timeouts

## APIs

- [ ] Build FastAPI services
- [ ] Design REST APIs
- [ ] Validate requests with Pydantic
- [ ] Design response models
- [ ] Understand authentication concepts
- [ ] Implement WebSockets
- [ ] Stream agent progress

## Production Engineering

- [ ] Write unit tests
- [ ] Write async tests
- [ ] Write integration tests
- [ ] Mock external dependencies
- [ ] Implement structured logging
- [ ] Add correlation/request IDs
- [ ] Package Python projects
- [ ] Use `pyproject.toml`
- [ ] Dockerize the application
- [ ] Build CI for tests and quality

---

# Interview-Level Questions

## Python

1. What is the difference between `is` and `==`?
2. What are mutable and immutable objects?
3. What are decorators?
4. What are generators?
5. What is a context manager?
6. What is the GIL?
7. What is duck typing?
8. `TypedDict` vs Pydantic model?
9. `Protocol` vs inheritance?
10. When should you use `dataclass`?

## Async

1. What does `async` mean?
2. What does `await` do?
3. How does the event loop work?
4. What happens when you call a blocking function inside async code?
5. `asyncio.gather()` vs `create_task()`?
6. When should you use a semaphore?
7. What causes an async deadlock?
8. Async vs threading?
9. Async vs multiprocessing?
10. How would you concurrently call 100 APIs while limiting concurrency to 10?

## FastAPI

1. Why FastAPI?
2. How does dependency injection work?
3. How does request validation work?
4. How do middleware and exception handlers work?
5. How would you implement authentication?
6. How would you handle long-running agent tasks?
7. REST vs WebSocket?
8. How would you stream LLM output?

## Production

1. How do you design retries?
2. What is exponential backoff?
3. When should you not retry?
4. How do you prevent duplicate operations?
5. How do you correlate logs across services?
6. How do you test external APIs?
7. How do you package and version a Python service?
8. How do you gracefully shut down async workers?

---

# Connection to Agentic AI

| Python Skill | Future Agentic AI Application |
|---|---|
| `typing` | Agent state and tool schemas |
| `async/await` | Concurrent LLM/tool calls |
| `asyncio` | Agent orchestration |
| Pydantic | Structured agent inputs/outputs |
| FastAPI | Agent APIs |
| REST | Agent service integration |
| WebSockets | Streaming agent execution |
| Concurrency | Parallel tools and agents |
| Multiprocessing | CPU-heavy workloads |
| pytest | Agent regression tests |
| Logging | Agent traces and debugging |
| Error handling | Agent retries/fallbacks |
| Packaging | Reusable agent services |

---

# Standard to Aim For

Do not aim for:

> "I completed a Python course."

Aim for:

> "I can build a typed, asynchronous, tested, observable, packaged FastAPI service that can safely orchestrate concurrent AI tools."

Then progress through:

```text
Phase 0
Python Engineering
       ↓
Phase 1
LLM Engineering
       ↓
Phase 2
RAG + Knowledge Systems
       ↓
Phase 3
Agent Fundamentals
       ↓
Phase 4
MCP + Tool Systems
       ↓
Phase 5
Agent Orchestration
       ↓
Phase 6
Advanced Agent Architecture
       ↓
Phase 7
Context Engineering
       ↓
Phase 8
AI Evaluation
       ↓
Phase 9
Agent Security
       ↓
Phase 10
LLMOps / AI Infrastructure
       ↓
Phase 11
AI + Kubernetes
       ↓
Staff-Level AI Platform Engineering
```

# North Star

> **Don't aim to become someone who knows Agentic AI.**
>
> **Become the engineer who can design, build, secure, deploy, evaluate, operate, and scale Agentic AI systems in production.**
