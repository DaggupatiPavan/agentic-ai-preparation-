# System Design Mastery — AI & Agentic Systems

## Objective

Master the distributed-systems concepts required to design reliable, scalable Agentic AI and AI Platform systems.

The goal is not to memorize architecture diagrams. The goal is to understand **why a design works, where it fails, how it scales, and what trade-offs it introduces**.

---

# 1. Mastery Map

```text
                         SYSTEM DESIGN
                              │
       ┌──────────────────────┼──────────────────────┐
       │                      │                      │
   Scalability            Reliability            Consistency
       │                      │                      │
       ├─ Caching             ├─ Retries             ├─ Eventual consistency
       ├─ Queues              ├─ Circuit breakers    ├─ Idempotency
       ├─ Rate limiting       ├─ Fault tolerance     └─ Distributed locks
       └─ Load balancing
                              │
                              ▼
                   Event-Driven Architecture
                              │
                              ▼
                    Production AI Systems
```

---

# 2. Scalability

## What You Must Understand

Scalability is the ability of a system to handle increasing load while maintaining acceptable performance and reliability.

Learn:

- Vertical scaling
- Horizontal scaling
- Stateless services
- Load balancing
- Auto scaling
- Database scaling
- Read replicas
- Sharding
- Partitioning
- Connection pooling
- Backpressure
- Capacity planning

## Vertical vs Horizontal

### Vertical

```text
4 CPU / 8 GB
      ↓
16 CPU / 32 GB
```

Advantages:

- Simple
- Minimal architectural changes

Limitations:

- Hardware limits
- Larger failure domain
- Often more expensive at scale

### Horizontal

```text
          Load Balancer
          /     |     \
         ↓      ↓      ↓
      API-1   API-2   API-3
```

Advantages:

- Better availability
- Elastic scaling
- Smaller failure domains

For cloud-native systems, horizontal scaling should become your default mental model.

---

# 3. Caching

Caching reduces latency and load on expensive downstream systems.

## Basic Architecture

```text
Client
  ↓
API
  ↓
Cache ── HIT ──→ Response
  │
 MISS
  ↓
Database
  ↓
Cache
  ↓
Response
```

## Learn

- Cache-aside
- Write-through
- Write-back
- Read-through
- TTL
- Eviction
- LRU
- Cache invalidation
- Cache stampede
- Hot keys
- Distributed caching

## Example

```text
Request
  ↓
Redis
  ↓
HIT → return
  ↓
MISS
  ↓
PostgreSQL
  ↓
Store in Redis
  ↓
Return
```

## AI Application

Cache:

- Embeddings
- Retrieval results
- Model responses
- Tool results
- Agent context
- Configuration

But never blindly cache responses when the underlying data or permissions can change.

---

# 4. Queues

Queues decouple producers and consumers.

```text
Producer
   ↓
 Message Queue
   ↓
 ┌───────┬───────┐
 ↓       ↓       ↓
Worker  Worker  Worker
```

Learn:

- Producer
- Consumer
- Acknowledgement
- Visibility timeout
- Retry queues
- Dead-letter queues
- Consumer groups
- Ordering
- Partitioning
- Backpressure

## Why Queues Matter

Without a queue:

```text
API → Slow Service
```

The API waits.

With a queue:

```text
API
 ↓
Queue
 ↓
Worker
```

The API can respond quickly while the worker processes the task asynchronously.

## AI Application

Use queues for:

- Long-running agent jobs
- Document processing
- Embedding generation
- Batch inference
- Evaluation jobs
- Incident analysis
- Tool execution
- Notifications

---

# 5. Retries

Distributed systems fail temporarily.

Examples:

```text
Network timeout
503
Rate limit
Database connection failure
LLM provider timeout
```

A retry strategy should be intentional.

## Exponential Backoff

```text
Attempt 1 → wait 1s
Attempt 2 → wait 2s
Attempt 3 → wait 4s
Attempt 4 → wait 8s
```

Add jitter to prevent many clients from retrying simultaneously.

## Never blindly retry

Do not automatically retry:

- Invalid requests
- Authentication failures
- Permission errors
- Permanent validation failures
- Non-idempotent operations without protection

---

# 6. Idempotency

An operation is idempotent when executing it multiple times produces the same intended result as executing it once.

Example:

```text
POST /payments
```

Network timeout occurs.

Client doesn't know whether payment succeeded.

Client retries.

Without idempotency:

```text
Payment → ₹10,000
Retry   → ₹10,000

Total = ₹20,000
```

With an idempotency key:

```text
Request
Idempotency-Key: abc123
       ↓
Server
       ↓
Already processed?
       ↓
YES → return previous result
```

## AI Application

Idempotency is essential for agents performing actions such as:

```text
Deploy application
Create infrastructure
Create GitHub PR
Restart service
Send notification
Create ticket
```

An agent must not accidentally execute the same destructive action twice.

---

# 7. Circuit Breakers

A circuit breaker prevents repeated calls to an unhealthy dependency.

```text
             Healthy
                │
                ▼
          ┌───────────┐
          │   CLOSED  │
          └─────┬─────┘
                │
          failures increase
                ↓
          ┌───────────┐
          │    OPEN   │
          └─────┬─────┘
                │
             timeout
                ↓
          ┌───────────┐
          │ HALF OPEN │
          └─────┬─────┘
             │       │
          success   failure
             │       │
             ↓       ↓
          CLOSED    OPEN
```

## Why?

Without a circuit breaker:

```text
API
 ↓
Dependency failing
 ↓
Retry
 ↓
Retry
 ↓
Retry
 ↓
More traffic
 ↓
Dependency becomes even worse
```

With a circuit breaker:

```text
Dependency unhealthy
        ↓
Circuit opens
        ↓
Fail fast
        ↓
Protect dependency + application
```

---

# 8. Rate Limiting

Rate limiting controls how much traffic a client can generate.

Example:

```text
100 requests/minute/user
```

Learn:

- Fixed window
- Sliding window
- Token bucket
- Leaky bucket
- Distributed rate limiting

## Token Bucket

```text
        Tokens
       ↓ ↓ ↓ ↓
     ┌─────────┐
     │ Bucket  │
     └────┬────┘
          ↓
       Request
          ↓
        API
```

## AI Application

Rate limiting is critical because LLM calls can be expensive.

Limit:

```text
Requests/user
Tokens/minute
LLM calls/agent
Tool calls/agent
Concurrent agent executions
```

---

# 9. Distributed Locks

A distributed lock coordinates multiple workers accessing the same resource.

Problem:

```text
Worker A ──┐
           ├──→ Same resource
Worker B ──┘
```

Both may execute the same operation.

With a distributed lock:

```text
Worker A
   ↓
Acquire lock
   ↓
Execute
   ↓
Release

Worker B
   ↓
Wait
```

Learn:

- Lock ownership
- Lease/TTL
- Lock expiration
- Deadlocks
- Fencing tokens
- Redis-based locks
- Database locks
- Consensus concepts

## AI Application

Useful when:

- Two agents may modify the same resource
- Only one deployment should run
- One incident should be remediated at a time
- A shared job must have a single owner

---

# 10. Eventual Consistency

In distributed systems, different replicas may temporarily have different states.

```text
              Database
                 │
       ┌─────────┼─────────┐
       ↓         ↓         ↓
    Replica A Replica B Replica C

Time T:
A = v2
B = v1
C = v1

Later:

A = v2
B = v2
C = v2
```

The system eventually converges.

## Learn

- Strong consistency
- Eventual consistency
- Read-after-write consistency
- Conflict resolution
- Replication lag
- CAP theorem
- Consistency trade-offs

## AI Application

Agent systems often combine:

```text
Vector DB
PostgreSQL
Redis
GitHub
Kubernetes
Cloud APIs
Observability systems
```

These systems do not necessarily update at exactly the same time.

Your agent architecture must tolerate stale information.

---

# 11. Event-Driven Architecture

Instead of services directly calling each other for every operation:

```text
Service A
   ↓
Service B
   ↓
Service C
```

Use events:

```text
Service A
   ↓
Event Bus
   ├────→ Service B
   ├────→ Service C
   └────→ Service D
```

## Learn

- Events
- Commands
- Event producers
- Event consumers
- Topics
- Partitions
- Consumer groups
- Event ordering
- At-least-once delivery
- At-most-once delivery
- Exactly-once concepts
- Event replay
- Dead-letter queues
- Schema evolution

## AI Example

```text
Deployment Event
       ↓
     Kafka
       ↓
 ┌─────┼───────────┐
 ↓     ↓           ↓
Agent Metrics   Security
 ↓     ↓           ↓
      Analysis
          ↓
      Incident Agent
```

This architecture is powerful for autonomous operations.

---

# 12. Fault Tolerance

Fault tolerance means the system continues providing useful service despite component failures.

Assume everything will fail.

Potential failures:

```text
Application
Database
Cache
Network
DNS
Cloud API
LLM provider
Kubernetes node
GPU
Message broker
External API
```

## Learn

- Redundancy
- Replication
- Failover
- Health checks
- Graceful degradation
- Timeouts
- Retries
- Circuit breakers
- Bulkheads
- Disaster recovery
- Backup/restore
- Multi-AZ architecture
- RTO
- RPO

---

# 13. Bulkhead Pattern

Prevent one failing workload from consuming all resources.

```text
                Application
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
        Pool A      Pool B     Pool C
         API        Agent      Reports
```

If Pool B fails:

```text
Pool A → healthy
Pool B → failing
Pool C → healthy
```

This is especially useful in agent platforms where one workload can unexpectedly consume large amounts of CPU, memory, tokens, or tool calls.

---

# 14. Putting Everything Together

## Production Agent Platform

```text
                         Users
                           │
                           ▼
                     API Gateway
                           │
                     Rate Limiter
                           │
                     Load Balancer
                           │
                  ┌────────┴────────┐
                  │                 │
             Agent API-1       Agent API-2
                  │                 │
                  └────────┬────────┘
                           │
                     Agent Runtime
                           │
          ┌────────────────┼────────────────┐
          │                │                │
       Redis             Queue          PostgreSQL
       Cache              │                │
          │          ┌────┼────┐           │
          │          ↓    ↓    ↓           │
          │       Worker Worker Worker     │
          │          │    │    │           │
          └──────────┴────┼────┴───────────┘
                          │
                     Tool Gateway
                          │
                         MCP
             ┌────────────┼────────────┐
             ↓            ↓            ↓
            AWS         GitHub        K8s
                          │
                     External APIs
                          │
                     Circuit Breaker
                          │
                         LLM
```

---

# 15. How the Patterns Work Together

Imagine an AI agent receives:

> "Investigate why checkout-service is failing and fix it."

A production system might execute:

```text
Request
  ↓
Rate limiter
  ↓
Authentication
  ↓
Agent API
  ↓
Idempotency check
  ↓
Agent state
  ↓
Parallel investigation
  ├── Kubernetes
  ├── Prometheus
  ├── GitHub
  └── AWS
          │
          ↓
       Results
          │
      Correlation
          │
       Decision
          │
    Approval required?
       /            YES        NO
      │          │
    Human      Execute
                │
          Distributed lock
                │
             Deploy
                │
          Circuit breaker
                │
             Verify
                │
          Emit event
                │
          Audit + metrics
```

This is the level of system thinking you should develop.

---

# 16. Design Checklist

Whenever you design a system, ask:

## Scale

- How many users?
- Requests per second?
- Peak traffic?
- Read/write ratio?
- Data volume?
- Growth rate?

## Performance

- What is the latency target?
- What can be cached?
- What can run asynchronously?
- What can run in parallel?

## Reliability

- What happens when the database fails?
- What happens when the LLM fails?
- What happens when an external API times out?
- Where are retries?
- Where are circuit breakers?
- Where are fallbacks?

## Consistency

- Do we need strong consistency?
- Can eventual consistency work?
- What happens with stale data?
- How are conflicts resolved?

## Operations

- How do we deploy?
- How do we rollback?
- What are the SLOs?
- What metrics matter?
- How do we trace requests?
- How do we debug failures?

## Security

- Who can access the system?
- What permissions does the agent have?
- How are secrets stored?
- Can a tool perform destructive operations?
- Is human approval required?

## Cost

For AI systems also ask:

- How many tokens per request?
- How many tool calls?
- What is the cost per task?
- Can responses be cached?
- Can smaller models handle some tasks?
- Can workloads be batched?

---

# 17. AI-Specific System Design

Traditional system design:

```text
Request
 ↓
API
 ↓
Database
 ↓
Response
```

Agentic system design:

```text
Request
 ↓
Agent
 ↓
Plan
 ↓
Context
 ↓
Model
 ↓
Tool
 ↓
Observation
 ↓
Model
 ↓
Tool
 ↓
Observation
 ↓
Evaluation
 ↓
Final response
```

This introduces new dimensions:

```text
Token cost
Context size
Agent state
Tool latency
Tool reliability
Model latency
Model routing
Agent loops
Agent termination
Evaluation
Safety
```

You need to design for all of them.

---

# 18. Practice Problems

Master these progressively.

## Level 1

### Design a URL Shortener

Focus:

- caching
- database
- scalability
- rate limiting

### Design a Rate Limiter

Focus:

- Redis
- distributed state
- token bucket
- concurrency

### Design a Notification System

Focus:

- queues
- retries
- idempotency
- dead-letter queues

---

## Level 2

### Design a Job Processing System

```text
API
 ↓
Queue
 ↓
Workers
 ↓
Database
```

Add:

- retries
- DLQ
- idempotency
- autoscaling

### Design a Distributed Cache

Study:

- partitioning
- replication
- eviction
- consistency

---

## Level 3

### Design an AI Chat Platform

Requirements:

- 100K users
- streaming responses
- conversation history
- rate limiting
- caching
- multiple LLM providers

---

## Level 4

### Design an AI Agent Platform

Requirements:

- 10K concurrent agents
- multiple LLM providers
- MCP tools
- asynchronous tasks
- persistent state
- WebSockets
- evaluation
- observability
- security
- cost controls

---

## Level 5 — Signature Project

### Design an Autonomous AI SRE Platform

```text
                    User
                      │
                 API Gateway
                      │
                 Agent Router
                      │
                Agent Runtime
                      │
          ┌───────────┼───────────┐
          ↓           ↓           ↓
       Planner     Investigator  Memory
          │           │           │
          └───────────┼───────────┘
                      ↓
                  Tool Gateway
                      │
              ┌───────┼────────┐
              ↓       ↓        ↓
             K8s     AWS      GitHub
              │       │        │
              └───────┼────────┘
                      ↓
                   Verifier
                      │
                Human Approval
                      │
                   Executor
                      │
                Deployment
                      │
                  Monitoring
                      │
               Event Bus / Audit
```

Your design must cover:

- scalability
- caching
- queues
- retries
- idempotency
- circuit breakers
- rate limiting
- distributed locks
- eventual consistency
- event-driven architecture
- fault tolerance
- authentication
- authorization
- observability
- AI evaluation
- cost control

---

# 19. Interview Framework

When asked to design a system, follow:

```text
1. Clarify requirements
        ↓
2. Estimate scale
        ↓
3. Define APIs
        ↓
4. Define data model
        ↓
5. Draw high-level architecture
        ↓
6. Explain request flow
        ↓
7. Identify bottlenecks
        ↓
8. Add caching
        ↓
9. Add queues
        ↓
10. Add reliability
        ↓
11. Add consistency strategy
        ↓
12. Add security
        ↓
13. Add observability
        ↓
14. Discuss trade-offs
```

Never jump straight into drawing Kubernetes pods.

---

# 20. What "Mastery" Means

You have not mastered system design because you can draw:

```text
Client → LB → API → DB
```

You have mastered it when you can answer:

> Why is the database the bottleneck?

> Why did you choose a queue?

> Why eventual consistency?

> What happens if Redis fails?

> What happens if the queue delivers the same message twice?

> How do you prevent duplicate agent actions?

> What happens if an LLM call takes 90 seconds?

> What if 10,000 agents call the same tool?

> What if a downstream API returns 503 for 10 minutes?

> How do you recover after a region failure?

> How do you control AI token cost?

> How do you prevent one agent from exhausting the entire platform?

---

# 21. Your Mastery Target

The progression should be:

```text
Beginner
   ↓
Understand individual patterns
   ↓
Combine 2–3 patterns
   ↓
Design complete distributed systems
   ↓
Design AI systems
   ↓
Design agentic systems
   ↓
Design production AI platforms
   ↓
Explain trade-offs like a Staff Engineer
```

## Final Standard

> **Don't memorize system-design patterns.**
>
> **Understand the failure they solve.**

When you see:

```text
Slow dependency
```

you should think:

```text
Timeout
+
Retry
+
Circuit breaker
+
Fallback
```

When you see:

```text
Traffic spike
```

think:

```text
Load balancing
+
Horizontal scaling
+
Caching
+
Queue/backpressure
+
Rate limiting
```

When you see:

```text
Duplicate execution
```

think:

```text
Idempotency
+
Distributed lock
+
Unique constraint
```

When you see:

```text
Agent performing production actions
```

think:

```text
Least privilege
+
Tool authorization
+
Idempotency
+
Approval gates
+
Audit
+
Rollback
```

---

# North Star

> **Become the engineer who can reason about systems under load, failure, concurrency, inconsistency, security constraints, and cost — and then apply that thinking to AI and autonomous agents.**

This system-design foundation will directly support the next stages:

```text
System Design
      ↓
LLM Engineering
      ↓
RAG
      ↓
Agent Runtime
      ↓
MCP / Tools
      ↓
Multi-Agent Systems
      ↓
AI Evaluation
      ↓
AI Security
      ↓
LLMOps
      ↓
AI Infrastructure
      ↓
Staff-Level AI Platform Engineering
```
