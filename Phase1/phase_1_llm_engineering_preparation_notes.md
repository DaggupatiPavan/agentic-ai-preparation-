# Phase 1 — LLM Engineering Preparation Notes

## Duration

**6–8 weeks**

## Objective

Become highly comfortable using Large Language Models as a **production systems component**, not merely as an API.

By the end of this phase, you should understand:

```text
Prompt
  ↓
Tokenizer
  ↓
Tokens
  ↓
Transformer
  ↓
Attention
  ↓
Model inference
  ↓
Sampling / decoding
  ↓
Structured output / tool call
  ↓
Application
```

You should also understand what happens around the model:

```text
Application
    ↓
Provider abstraction
    ↓
Model routing
    ↓
LLM API / self-hosted model
    ↓
Streaming / batching
    ↓
Observability
    ↓
Evaluation
    ↓
Cost + latency optimization
```

---

# 1. Learning Strategy

Use:

```text
30% Theory
70% Hands-on
```

For every concept:

```text
Understand
   ↓
Write a tiny experiment
   ↓
Measure it
   ↓
Break it
   ↓
Debug it
   ↓
Use it in a project
   ↓
Explain it without notes
```

Do not try to memorize terminology.

Your goal is to understand **what happens, why it happens, and what engineering decision it creates**.

---

# 2. Week-by-Week Plan

| Week | Focus | Deliverable |
|---|---|---|
| 1 | Tokenization + Transformer fundamentals | Tokenization lab + transformer notes |
| 2 | Attention + context windows | Attention/context experiments |
| 3 | Embeddings + retrieval basics | Semantic-search mini project |
| 4 | Inference + sampling | Model parameter experimentation |
| 5 | Structured output + tool calling | Typed tool-calling agent |
| 6 | Streaming + batching | Streaming/batch inference service |
| 7 | Quantization + fine-tuning concepts | Model optimization report |
| 8 | Provider abstraction + production architecture | Multi-provider LLM gateway |

If you have only 6 weeks, combine:

```text
Weeks 1–2 → Fundamentals
Week 3     → Embeddings
Week 4     → Inference + structured output
Week 5     → Tools + streaming
Week 6     → Provider abstraction + production project
```

---

# 3. Tokenization

## What is a Token?

An LLM generally does not directly process raw text.

Conceptually:

```text
"Hello world"
      ↓
Tokenizer
      ↓
["Hello", " world"]
      ↓
Token IDs
      ↓
[15496, 995]
```

Actual tokenization depends on the model/tokenizer.

## Learn

- tokens
- token IDs
- vocabulary
- subword tokenization
- BPE
- tokenization of numbers
- tokenization of code
- tokenization of different languages
- input tokens
- output tokens
- special tokens

## Why Tokenization Matters

Tokenization affects:

- cost
- context usage
- latency
- maximum input size
- maximum output size
- prompt design

Example:

```text
1 user request
+
10,000-token context
+
4,000-token output
=
14,000+ tokens
```

This directly affects model cost and latency.

## Hands-On

Build:

```text
tokenizer_lab.py
```

Experiment with:

```text
Short English sentence
Long English sentence
JSON
Python code
Kubernetes YAML
Indian languages
Numbers
URLs
Logs
```

Record:

```text
Input characters
Token count
Characters/token
```

### Questions

- Why is a word not necessarily one token?
- Why can the same text have different token counts across models?
- Why does tokenization matter for cost?

---

# 4. Transformer Fundamentals

You do not need to become a research scientist, but you must understand the architecture conceptually.

Basic flow:

```text
Text
 ↓
Tokenizer
 ↓
Token IDs
 ↓
Embeddings
 ↓
Transformer layers
 ↓
Logits
 ↓
Sampling / decoding
 ↓
Generated tokens
```

## Learn

- embeddings
- positional information
- self-attention
- feed-forward networks
- residual connections
- normalization
- transformer layers
- logits
- probability distribution
- autoregressive generation

## Important Mental Model

A transformer repeatedly transforms representations of tokens so the model can predict what comes next.

For a simplified example:

```text
"The server returned a"

        ↓

Model probabilities

200       0.42
500       0.31
404       0.08
timeout   0.05
...
```

Decoding determines what token is selected.

---

# 5. Attention

Attention is one of the most important concepts to understand.

Example:

```text
"The application crashed because it ran out of memory."

                    ↑
              "it" refers to
              "application"
```

Attention allows token representations to incorporate information from other relevant tokens.

## Learn

```text
Query
Key
Value
```

Conceptually:

```text
Q × K
  ↓
Attention scores
  ↓
Softmax
  ↓
Weighted Values
  ↓
New representation
```

## Understand

- self-attention
- multi-head attention
- causal attention
- attention weights
- computational cost
- why long contexts can be expensive

## Interview Question

> Why can't an LLM simply process an infinitely large context window?

Your answer should discuss:

- computational cost
- memory requirements
- attention complexity
- inference latency
- quality degradation / context utilization challenges

---

# 6. Context Windows

A context window is the amount of tokenized information a model can process within a request/generation context.

Conceptually:

```text
┌───────────────────────────────┐
│ System instructions           │
│ User request                  │
│ Conversation history          │
│ Retrieved documents           │
│ Tool results                  │
│ Agent state                   │
│ Requested output              │
└───────────────────────────────┘
```

Everything competes for context.

## Learn

- input context
- output limits
- context window
- truncation
- context compression
- summarization
- retrieval
- context selection
- long-context trade-offs

## Agentic AI Connection

A bad agent might send:

```text
10,000 logs
+
500 documents
+
100 tool outputs
+
entire conversation
```

A better agent sends:

```text
Relevant logs
+
Relevant documents
+
Compressed history
+
Verified tool results
+
Current task state
```

This becomes **context engineering**.

---

# 7. Embeddings

Embeddings transform information into numerical vectors.

Conceptually:

```text
"Pod is restarting"
       ↓
   Embedding Model
       ↓
[0.12, -0.84, 0.33, ...]
```

Similar meanings tend to produce nearby vectors.

```text
"Pod is restarting"
        ●
                   ● "Container keeps crashing"

                         ●
                    "Stock price"
```

## Learn

- embedding models
- vector dimensions
- cosine similarity
- dot product
- Euclidean distance
- semantic similarity
- normalization
- embedding storage

## Hands-On

Create:

```text
embedding_lab.py
```

Compare:

```text
"Pod is crashing"
"Container keeps restarting"
"Kubernetes workload is failing"
"Pizza recipe"
```

Measure similarity.

---

# 8. Inference

Inference means using a trained model to generate output.

Understand:

```text
Prompt
 ↓
Tokenization
 ↓
Model
 ↓
Logits
 ↓
Sampling / decoding
 ↓
Token
 ↓
Next token
 ↓
Repeat
```

## Learn

- prefill
- decode
- autoregressive generation
- latency
- throughput
- tokens/sec
- time to first token (TTFT)
- time per output token
- batching
- GPU utilization
- KV cache

## Important Metrics

### TTFT

```text
Request
   ↓
   ↓
First token
```

Time to first token affects perceived responsiveness.

### Throughput

```text
tokens / second
requests / second
```

### Total latency

```text
network
+
queue
+
prefill
+
generation
+
post-processing
```

---

# 9. Temperature

Temperature controls how sharply or randomly the probability distribution is sampled.

Conceptually:

```text
Low temperature
      ↓
More deterministic

High temperature
      ↓
More diverse / random
```

For example:

```text
temperature = 0
```

is commonly used when deterministic behavior is preferred.

Higher values can be useful for:

- brainstorming
- creative writing
- diverse candidate generation

Lower values are generally preferable for:

- extraction
- classification
- structured workflows
- tool selection
- deterministic operations

Do not assume that temperature alone guarantees deterministic output.

---

# 10. Top-p

Top-p, or nucleus sampling, limits token selection to a probability mass.

Conceptually:

```text
Token probabilities

A = 0.50
B = 0.25
C = 0.15
D = 0.05
E = 0.05

top_p = 0.90

Candidate set:
A + B + C = 0.90
```

Learn:

- temperature
- top-p
- greedy decoding
- sampling
- deterministic vs stochastic generation

Understand the difference between:

```text
Temperature → reshapes probabilities

Top-p → restricts candidate probability mass
```

---

# 11. Structured Output

LLMs naturally generate text.

Production applications often require:

```json
{
  "severity": "HIGH",
  "service": "payment-api",
  "confidence": 0.94
}
```

Learn:

- JSON output
- JSON Schema
- Pydantic models
- schema validation
- structured generation
- validation failures
- retry strategies

## Example

```python
from pydantic import BaseModel

class Incident(BaseModel):
    severity: str
    service: str
    confidence: float
```

Your application should validate model output rather than blindly trusting it.

Architecture:

```text
LLM
 ↓
Structured output
 ↓
Pydantic validation
 ↓
Valid?
 ├── Yes → continue
 └── No  → retry / repair / fallback
```

---

# 12. Tool Calling

Tool calling allows the model to request an external operation.

Example:

```text
User
 ↓
LLM
 ↓
"I need Kubernetes pod status"
 ↓
Tool call
 ↓
Kubernetes API
 ↓
Tool result
 ↓
LLM
 ↓
Final answer
```

## Tool Definition

Conceptually:

```python
def get_pod_status(
    namespace: str,
    pod_name: str
):
    ...
```

The model should receive a schema describing:

```text
Tool name
Description
Arguments
Argument types
```

## Critical Production Concepts

Learn:

- schema validation
- authorization
- tool allowlists
- timeouts
- retries
- idempotency
- tool result validation
- audit logging
- permission boundaries

Never treat an LLM's requested tool action as automatically trusted.

---

# 13. Reasoning

Do not reduce reasoning to:

> "Ask the model to think harder."

For production engineering, understand:

```text
Task
 ↓
Decomposition
 ↓
Intermediate state
 ↓
Tool usage
 ↓
Observation
 ↓
Verification
 ↓
Final result
```

Study:

- chain-of-thought concepts
- reasoning models
- planning
- decomposition
- reflection
- verification
- tool-assisted reasoning
- agent state

## Important Principle

Your application should care about:

```text
Correct result
+
Evidence
+
Tool execution
+
Validation
```

rather than relying blindly on hidden reasoning.

---

# 14. Streaming

Instead of waiting for the complete answer:

```text
Request
    ↓
............... 10 seconds
    ↓
Complete response
```

stream:

```text
Request
 ↓
Token
 ↓
Token
 ↓
Token
 ↓
Token
 ↓
...
```

## Learn

- streaming APIs
- Server-Sent Events (SSE)
- WebSockets
- token streaming
- buffering
- backpressure
- cancellation
- partial responses
- error handling during streams

## AI Application

Streaming is important for:

- chat
- agent progress
- long-running tasks
- coding agents
- tool execution status

---

# 15. Batch Inference

Batching combines multiple inference requests.

```text
Request A ─┐
Request B ─┼──→ Batch → Model
Request C ─┤
Request D ─┘
```

Benefits can include:

- better hardware utilization
- higher throughput
- lower cost per request

Trade-off:

```text
More batching
     ↓
Better throughput
     ↓
Potentially higher waiting time
```

Understand:

- static batching
- dynamic batching
- continuous batching
- throughput vs latency
- batch size
- queueing

---

# 16. Quantization

Quantization reduces numerical precision used to represent model weights/activations.

Conceptually:

```text
FP32
 ↓
FP16 / BF16
 ↓
INT8
 ↓
INT4
```

Potential benefits:

- lower memory usage
- faster inference in supported environments
- lower infrastructure cost

Potential trade-offs:

- quality degradation
- hardware/software compatibility
- implementation complexity

## Learn

- FP32
- FP16
- BF16
- INT8
- INT4
- weight quantization
- activation quantization
- quantization-aware concepts

You do not need to implement a quantization algorithm from scratch initially.

---

# 17. Fine-Tuning Concepts

You do not need to become a model-training engineer in this phase.

Understand when fine-tuning is appropriate.

## Learn

- pretraining
- supervised fine-tuning
- instruction tuning
- LoRA
- QLoRA
- adapters
- dataset preparation
- training/validation split
- overfitting
- catastrophic forgetting
- evaluation

## Decision Framework

Before fine-tuning ask:

```text
Can prompting solve it?
       ↓
Can RAG solve it?
       ↓
Can tool calling solve it?
       ↓
Can better context solve it?
       ↓
Can model selection solve it?
       ↓
Only then consider fine-tuning
```

Fine-tuning should not be the default solution for missing knowledge.

---

# 18. Provider Abstraction

Avoid tightly coupling your application to one model provider.

Conceptual interface:

```python
from typing import Protocol

class LLMProvider(Protocol):

    async def generate(...):
        ...

    async def stream(...):
        ...

    async def embed(...):
        ...
```

Then implementations can include:

```text
OpenAIProvider
AnthropicProvider
GoogleProvider
LocalProvider
```

Your application talks to:

```text
LLMProvider
```

instead of directly depending everywhere on one vendor SDK.

---

# 19. Why Provider Abstraction Matters

It enables:

### Model Routing

```text
Simple task
   ↓
Cheap model

Complex task
   ↓
Powerful model
```

### Fallback

```text
Primary provider
      ↓
   Failure
      ↓
Secondary provider
```

### Cost Optimization

```text
Classification
 → small model

Reasoning
 → advanced model

Embeddings
 → embedding model
```

### Experimentation

```text
Model A
vs
Model B
vs
Model C
```

without rewriting your entire application.

---

# 20. Recommended Architecture

```text
                         Application
                              │
                       LLM Gateway
                              │
                       Model Router
                              │
              ┌───────────────┼───────────────┐
              ↓               ↓               ↓
        Provider A       Provider B       Provider C
              │               │               │
            Model           Model           Model
```

The gateway should manage:

- authentication
- provider selection
- routing
- retries
- timeouts
- rate limits
- fallbacks
- logging
- metrics
- cost tracking
- request IDs
- model configuration

---

# 21. Recommended Python Interface

A practical abstraction can look like:

```python
from dataclasses import dataclass
from typing import Protocol, AsyncIterator


@dataclass
class GenerateRequest:
    model: str
    messages: list[dict]
    temperature: float = 0.0
    max_tokens: int | None = None


@dataclass
class GenerateResponse:
    text: str
    model: str
    input_tokens: int
    output_tokens: int
    latency_ms: float


class LLMProvider(Protocol):

    async def generate(
        self,
        request: GenerateRequest,
    ) -> GenerateResponse:
        ...

    async def stream(
        self,
        request: GenerateRequest,
    ) -> AsyncIterator[str]:
        ...

    async def embed(
        self,
        texts: list[str],
    ) -> list[list[float]]:
        ...
```

This becomes the foundation for your future AI platform.

---

# 22. Production LLM Gateway

Eventually build:

```text
                     Client
                       │
                       ▼
                  FastAPI
                       │
                Authentication
                       │
                 Rate Limiter
                       │
                 LLM Gateway
                       │
                Model Router
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
     Provider A     Provider B     Local Model
        │              │              │
        └──────────────┼──────────────┘
                       ↓
                 Response Layer
                       │
              ┌────────┼────────┐
              ↓        ↓        ↓
           Metrics   Logging   Cost
```

---

# 23. LLM Reliability Patterns

Your LLM layer should eventually support:

```text
Timeout
 ↓
Retry with backoff
 ↓
Provider fallback
 ↓
Circuit breaker
 ↓
Graceful degradation
```

Also implement:

- request cancellation
- rate limiting
- token limits
- output validation
- idempotency where appropriate
- provider-specific error mapping
- structured logs

---

# 24. LLM Observability

Track at minimum:

```text
Request count
Error count
Latency
TTFT
Input tokens
Output tokens
Total tokens
Cost
Model
Provider
Status
Retries
Fallbacks
```

Example:

```text
Request ID: abc123
Model: model-x
Provider: provider-a
Input tokens: 3,200
Output tokens: 650
TTFT: 820 ms
Total latency: 5.2 s
Retries: 0
Cost: $0.XX
Status: success
```

Later integrate:

```text
OpenTelemetry
Prometheus
Grafana
Distributed tracing
LLM tracing/evaluation platforms
```

---

# 25. Cost Engineering

LLM systems have a unique cost dimension.

Track:

```text
Cost / request
Cost / user
Cost / agent task
Cost / successful task
Cost / 1K tokens
```

Optimization levers:

```text
Smaller model
      ↓
Prompt optimization
      ↓
Context reduction
      ↓
Caching
      ↓
Batching
      ↓
Routing
      ↓
Quantization
      ↓
Self-hosting when justified
```

Do not optimize cost by blindly choosing the cheapest model.

Optimize:

> **Cost per successful task.**

---

# 26. Performance Engineering

Measure:

```text
Latency
Throughput
TTFT
Tokens/sec
Concurrent requests
Queue time
Provider latency
Tool latency
```

Think:

```text
Total latency =
Queue
+
Network
+
Model prefill
+
Model decode
+
Tool calls
+
Post-processing
```

For agent systems:

```text
Agent latency =
LLM call 1
+
Tool 1
+
LLM call 2
+
Tool 2
+
LLM call 3
```

Therefore, reducing unnecessary agent loops can dramatically improve latency and cost.

---

# 27. Security

LLM engineering must include security from the beginning.

Learn:

- prompt injection
- sensitive data leakage
- insecure tool calling
- credential exposure
- malicious documents
- model output validation
- tenant isolation
- provider credential management

Never place secrets in prompts.

Never trust model-generated tool arguments without validation.

Never grant an agent more permissions than necessary.

---

# 28. Hands-On Labs

## Lab 1 — Tokenization

Build:

```text
tokenizer_lab/
├── tokenizer.py
├── examples.txt
└── README.md
```

Measure token counts for:

- English
- code
- JSON
- YAML
- logs
- URLs
- multilingual text

---

## Lab 2 — Embeddings

Build:

```text
embedding_search/
├── embed.py
├── search.py
└── README.md
```

Store embeddings and perform semantic similarity search.

---

## Lab 3 — Sampling

Build an experiment that compares:

```text
temperature:
0
0.2
0.7
1.0

top_p:
0.5
0.8
0.95
1.0
```

Run the same prompts and record output differences.

---

## Lab 4 — Structured Output

Build:

```text
incident_parser/
```

Input:

```text
"Kubernetes payment-api has 5 CrashLoopBackOff pods."
```

Output:

```json
{
  "service": "payment-api",
  "severity": "HIGH",
  "issue": "CrashLoopBackOff",
  "pod_count": 5
}
```

Validate using Pydantic.

---

## Lab 5 — Tool Calling

Create tools:

```text
get_pod_status()
get_pod_logs()
get_deployment()
get_metrics()
```

Build a simple agent that chooses the appropriate tool.

---

## Lab 6 — Streaming

Build:

```text
FastAPI
   ↓
LLM
   ↓
SSE/WebSocket
   ↓
Client
```

Stream generated output token-by-token or chunk-by-chunk.

---

## Lab 7 — Provider Abstraction

Implement:

```text
LLMProvider
     │
 ┌───┼───────────┐
 ↓   ↓           ↓
A    B         Local
```

The application should be able to switch providers through configuration.

---

# 29. Phase 1 Capstone

# Production LLM Gateway

Build a production-style service.

## Requirements

```text
POST /generate
POST /embed
POST /stream
GET  /models
GET  /health
```

### Architecture

```text
                         Client
                           │
                           ▼
                        FastAPI
                           │
                     Authentication
                           │
                      Rate Limiter
                           │
                       LLM Gateway
                           │
                      Model Router
                           │
              ┌────────────┼────────────┐
              ↓            ↓            ↓
          Provider A   Provider B   Local/vLLM
              │            │            │
              └────────────┼────────────┘
                           ↓
                     Response Handler
                           │
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
          Logging       Metrics        Cost
```

## Must Have

- Async Python
- FastAPI
- Pydantic
- Provider abstraction
- Streaming
- Timeouts
- Retries
- Circuit breaker
- Rate limiting
- Structured logging
- Metrics
- Token tracking
- Cost tracking
- Error normalization
- Unit tests
- Integration tests
- Docker
- CI/CD

---

# 30. Suggested Repository

```text
llm-gateway/
│
├── pyproject.toml
├── README.md
├── Dockerfile
├── docker-compose.yml
│
├── src/
│   └── llm_gateway/
│       ├── main.py
│       │
│       ├── api/
│       │   ├── generate.py
│       │   ├── embeddings.py
│       │   └── streaming.py
│       │
│       ├── providers/
│       │   ├── base.py
│       │   ├── provider_a.py
│       │   ├── provider_b.py
│       │   └── local.py
│       │
│       ├── routing/
│       │   └── router.py
│       │
│       ├── models/
│       │   ├── requests.py
│       │   └── responses.py
│       │
│       ├── resilience/
│       │   ├── retry.py
│       │   ├── timeout.py
│       │   └── circuit_breaker.py
│       │
│       ├── observability/
│       │   ├── logging.py
│       │   └── metrics.py
│       │
│       └── config/
│           └── settings.py
│
└── tests/
    ├── unit/
    └── integration/
```

---

# 31. Interview Preparation

You should eventually be able to answer:

## Fundamentals

1. What is a token?
2. Why do LLMs use tokens?
3. What is a transformer?
4. What is attention?
5. What are Q, K, and V?
6. What is an embedding?
7. What is inference?
8. What is a context window?
9. What is TTFT?
10. What is token throughput?

## Sampling

11. What does temperature do?
12. What does top-p do?
13. Temperature vs top-p?
14. How would you make an output more deterministic?

## Production

15. How would you implement LLM retries?
16. When should you fallback to another provider?
17. How would you track LLM cost?
18. How would you handle provider rate limits?
19. How would you stream responses?
20. How would you batch requests?
21. How would you design an LLM gateway?
22. How would you route requests between models?
23. How would you handle provider outages?
24. How would you monitor LLM performance?

## Agentic AI

25. How does tool calling work?
26. How do you validate tool arguments?
27. How do you prevent dangerous tool execution?
28. How do you maintain agent state?
29. How do you reduce context size?
30. How do you evaluate agent responses?
31. How do you prevent an agent from entering an infinite loop?
32. How do you make agent actions idempotent?

---

# 32. Definition of Done

Do not move to Phase 2 until you can confidently:

- [ ] Explain tokenization
- [ ] Explain transformer architecture conceptually
- [ ] Explain attention and Q/K/V
- [ ] Explain context windows
- [ ] Generate and compare embeddings
- [ ] Explain inference
- [ ] Explain TTFT and throughput
- [ ] Explain temperature
- [ ] Explain top-p
- [ ] Produce validated structured output
- [ ] Implement tool calling
- [ ] Stream model output
- [ ] Explain batching
- [ ] Explain quantization
- [ ] Explain fine-tuning vs RAG
- [ ] Design an LLM provider abstraction
- [ ] Implement retries and fallbacks
- [ ] Track tokens and cost
- [ ] Implement LLM observability
- [ ] Build and Dockerize an LLM gateway
- [ ] Write unit and integration tests
- [ ] Explain the architecture in an interview

---

# 33. Final Mental Model

You should be able to visualize LLM engineering as:

```text
                     USER
                       │
                       ▼
                 APPLICATION
                       │
                 LLM GATEWAY
                       │
                 MODEL ROUTER
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
     Model A         Model B        Local Model
        │              │              │
        └──────────────┼──────────────┘
                       ↓
                    INFERENCE
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
      Streaming     Sampling     Batching
          │            │            │
          └────────────┼────────────┘
                       ↓
                STRUCTURED OUTPUT
                       │
                  TOOL CALLING
                       │
                    AGENT
                       │
              ┌────────┴────────┐
              ↓                 ↓
           Memory              Tools
              │                 │
              └────────┬────────┘
                       ↓
                  Observability
                       │
              ┌────────┼────────┐
              ↓        ↓        ↓
           Metrics    Logs     Cost
```

---

# North Star

> **Don't become someone who can merely call an LLM API.**

Become the engineer who understands the entire lifecycle:

```text
Tokenization
     ↓
Transformer
     ↓
Attention
     ↓
Context
     ↓
Embeddings
     ↓
Inference
     ↓
Sampling
     ↓
Structured Output
     ↓
Tool Calling
     ↓
Streaming
     ↓
Batching
     ↓
Routing
     ↓
Reliability
     ↓
Observability
     ↓
Cost Optimization
     ↓
Production LLM Platform
```

The final goal of Phase 1 is:

> **Build an LLM Gateway that abstracts providers, supports async generation/streaming/embeddings, handles failures, measures tokens/latency/cost, validates outputs, and is ready to become the model layer underneath your future Agentic AI platform.**
