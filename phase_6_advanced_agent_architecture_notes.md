# Phase 6 — Advanced Agent Architecture

## Duration: 2–3 Months

## Mission

Move from building individual agents to designing **advanced, reliable, long-horizon agent systems**.

Phase 3 taught the agent loop.

Phase 4 taught tools and MCP.

Phase 5 taught orchestration.

Phase 6 focuses on the deeper intelligence and architecture behind advanced agents:

```text
Planning
+
Task Decomposition
+
Reasoning Strategies
+
Reflection
+
Self-Critique
+
Search
+
Memory
+
Multi-Agent Architecture
+
Long-Horizon Execution
```

## Golden Rule

> **Do not create multiple agents simply because you can.**

The best architecture is the **simplest architecture that reliably solves the problem**.

---

# 1. Phase 6 Mental Model

A basic agent:

```text
Goal
 ↓
Decide
 ↓
Tool
 ↓
Observe
 ↓
Final
```

An advanced agent:

```text
Goal
 ↓
Understand
 ↓
Decompose
 ↓
Plan
 ↓
Execute
 ↓
Observe
 ↓
Evaluate
 ↓
Reflect
 ↓
Re-plan
 ↓
Execute
 ↓
Verify
 ↓
Remember
 ↓
Final
```

For complex tasks:

```text
                    Goal
                     │
                     ▼
                Task Decomposer
                     │
                     ▼
                   Planner
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       Research     Code      Analysis
          │          │          │
          └──────────┼──────────┘
                     ▼
                  Reviewer
                     │
                     ▼
                 Reflection
                     │
                ┌────┴────┐
                ▼         ▼
              Good      Improve
                │         │
                │         └────► Planner
                ▼
              Final
```

---

# 2. What You Need to Master

## Planning

Learn:

- ReAct
- Plan-and-execute
- hierarchical planning
- task decomposition
- dynamic re-planning
- dependency-aware planning
- plan validation
- plan repair

## Reasoning / Verification

Learn:

- reflection
- self-critique
- verification
- evaluator loops
- search concepts
- candidate generation
- candidate ranking

## Memory

Understand:

- conversation memory
- working memory
- episodic memory
- semantic memory
- procedural memory
- long-term memory

## Multi-Agent Architecture

Learn when to use:

- single agent
- supervisor → workers
- router → specialists
- parallel agents
- sequential agents
- debate / critique
- planner → executor

---

# 3. Planning Fundamentals

Planning answers:

> What should the system do to achieve the goal?

Example:

```text
Goal:
Investigate a production incident.

Plan:
1. Identify affected service
2. Query error rate
3. Inspect pods
4. Read logs
5. Check recent deployment
6. Correlate evidence
7. Verify root cause
8. Recommend remediation
```

Planning is not necessarily a fixed list.

A strong agent can modify the plan when new evidence appears.

---

# 4. ReAct

ReAct combines reasoning-oriented decision making with actions and observations.

Conceptually:

```text
Goal
 ↓
Reason about next action
 ↓
Act
 ↓
Observe
 ↓
Reason again
 ↓
Act
 ↓
Observe
```

Example:

```text
User:
Why is checkout-api failing?

Agent:
Need current workload status.

Action:
get_pods()

Observation:
3 pods CrashLoopBackOff.

Agent:
Need application error.

Action:
get_logs()

Observation:
database connection timeout.

Agent:
Need database health.

Action:
get_rds_metrics()

Observation:
RDS healthy.

Agent:
Need Kubernetes service configuration.

Action:
get_service()

Observation:
service endpoints missing.

Conclusion:
Likely service discovery/configuration issue.
```

The key property:

> **The next action depends on the latest observation.**

---

# 5. ReAct Strengths

Useful when:

- environment is uncertain
- tool results change the plan
- investigation is dynamic
- actions depend on observations

Weaknesses:

- many model calls
- latency
- cost
- possible loops
- difficult long-horizon planning

Use safeguards:

```text
max iterations
timeouts
cost limits
progress checks
```

---

# 6. Plan-and-Execute

Instead of deciding every step independently:

```text
Goal
 ↓
Planner
 ↓
Plan
 ↓
Executor
 ↓
Results
 ↓
Final
```

Example:

```text
Planner:

1. Query Prometheus
2. Inspect Kubernetes
3. Check recent deployment
4. Read logs
5. Correlate findings
```

Executor executes the plan.

---

# 7. ReAct vs Plan-and-Execute

| Strategy | ReAct | Plan-and-Execute |
|---|---|---|
| Planning | Continuous | Upfront |
| Adaptability | High | Medium |
| LLM calls | More | Potentially fewer |
| Long tasks | Can become noisy | Better structure |
| Dynamic environment | Strong | Requires re-planning |
| Debugging | Action-by-action | Plan-level |

In production, hybrid approaches are often useful:

```text
Initial plan
 ↓
Execute
 ↓
Observe
 ↓
Repair plan
 ↓
Continue
```

---

# 8. Dynamic Re-Planning

Initial plan:

```text
1. Check pods
2. Check logs
3. Check database
```

Observation:

```text
Pods cannot resolve service DNS.
```

New plan:

```text
1. Check service
2. Check endpoints
3. Check CoreDNS
4. Check network policy
```

The plan changed because reality changed.

---

# 9. Hierarchical Planning

Large goals can be decomposed into levels.

Example:

```text
Goal:
Resolve production checkout incident.
        │
        ├── Understand incident
        │     ├── Metrics
        │     ├── Logs
        │     └── Deployment
        │
        ├── Identify root cause
        │     ├── Application
        │     ├── Network
        │     └── Database
        │
        └── Remediate
              ├── Choose action
              ├── Approval
              └── Verify
```

This is hierarchical planning.

---

# 10. Task Decomposition

A good decomposition creates tasks that are:

- independently understandable
- independently executable where possible
- verifiable
- bounded
- appropriately sized

Bad:

```text
Fix everything.
```

Better:

```text
1. Identify failing service
2. Determine failure symptom
3. Find likely dependency
4. Verify dependency
5. Propose remediation
```

---

# 11. Dependency-Aware Planning

Tasks can form a DAG.

```text
A ─────► C
│
└─────► B ─────► D
```

Example:

```text
Get deployment
      │
      ├────► Get pods
      │
      └────► Get recent deployment
                    │
                    ▼
                  Verify
```

Only execute a task when its dependencies are satisfied.

---

# 12. Plan Validation

Do not blindly execute generated plans.

Validate:

```text
Are all tools available?
Are dependencies valid?
Are actions authorized?
Is the plan complete?
Are destructive actions present?
Is the plan within budget?
```

Example:

```python
def validate_plan(plan, tools, policy):

    for step in plan.steps:

        if step.tool not in tools:
            raise ValueError("Unknown tool")

        if not policy.allows(step):
            raise PermissionError(
                "Unauthorized action"
            )
```

---

# 13. Plan Repair

Suppose:

```text
Plan:
Check Prometheus
Check Grafana
Check K8s
```

Prometheus is unavailable.

Repair:

```text
Check Grafana
Check K8s
Check CloudWatch
```

Plan repair should preserve completed work.

Do not restart from zero unless necessary.

---

# 14. Reflection

Reflection asks:

> How well did I perform?

Example:

```text
Task:
Diagnose API failure.

Result:
Agent concluded database failure.

Evidence:
Database healthy.

Reflection:
Conclusion is unsupported.

Action:
Re-open investigation.
```

Reflection is a feedback mechanism.

---

# 15. Reflection Loop

```text
Execute
 ↓
Result
 ↓
Reflect
 ↓
Good?
 ├── YES → Continue
 └── NO → Improve
             │
             ▼
           Re-plan
```

Reflection should have a concrete purpose.

Do not add reflection simply to increase LLM calls.

---

# 16. Self-Critique

Self-critique asks the agent to challenge its own output.

Example:

```text
Proposed diagnosis:
"Database is overloaded."

Critique:
- CPU is normal.
- Connections are normal.
- No database errors.

Conclusion:
Diagnosis is weak.
```

Then:

```text
Search for stronger evidence.
```

---

# 17. Reflection vs Self-Critique

### Reflection

Broad:

```text
Did the process work?
```

### Self-critique

Specific:

```text
Is this answer/plan correct?
What evidence contradicts it?
```

Both can feed an evaluator.

---

# 18. Verification

Never confuse:

```text
Agent believes it succeeded
```

with:

```text
System actually succeeded
```

Example:

```text
Agent:
Deployment restarted successfully.

Verification:
Pods still unhealthy.
```

Therefore:

```text
Action
 ↓
Verification
 ↓
Evidence
```

---

# 19. Verification Strategies

Use:

```text
state comparison
health checks
metrics
tests
tool results
independent evaluator
```

For infrastructure:

```text
Before
 ↓
Action
 ↓
After
 ↓
Compare
```

Example:

```text
Before:
5xx = 15%

Restart

After:
5xx = 0.2%

Verified.
```

---

# 20. Tree Search Concepts

You do not need to immediately implement full research-grade search algorithms.

Understand the concept.

Suppose the agent has:

```text
Goal
 │
 ├── Strategy A
 │     ├── Tool 1
 │     └── Tool 2
 │
 ├── Strategy B
 │     ├── Tool 3
 │     └── Tool 4
 │
 └── Strategy C
       ├── Tool 5
       └── Tool 6
```

The agent can explore candidate paths and compare them.

Core concepts:

```text
state
action
branch
candidate
score
expand
prune
select
```

---

# 21. Search vs Normal Planning

Normal:

```text
Choose one plan
 ↓
Execute
```

Search:

```text
Generate candidates
 ↓
Evaluate candidates
 ↓
Select promising candidate
 ↓
Expand
 ↓
Evaluate
```

Useful when:

- multiple plausible strategies exist
- mistakes are expensive
- candidate solutions can be scored

---

# 22. Search Costs

Search can explode.

If:

```text
5 choices
×
5 choices
×
5 choices
```

you already have:

```text
125 possible paths
```

At depth 5:

```text
5^5 = 3125
```

Therefore use:

```text
pruning
beam search concepts
heuristics
budget
depth limits
candidate scoring
```

---

# 23. Task Decomposition Strategies

### Sequential decomposition

```text
A → B → C
```

### Parallel decomposition

```text
   ┌→ A
Goal
   ├→ B
   └→ C
```

### Hierarchical

```text
Goal
 ├── Subgoal A
 │    ├── A1
 │    └── A2
 └── Subgoal B
      ├── B1
      └── B2
```

Choose based on dependencies.

---

# 24. Memory Fundamentals

Agents need memory because not everything belongs in the current context.

Think of memory as:

```text
Information
 ↓
Store
 ↓
Retrieve
 ↓
Use
```

Different memory types solve different problems.

---

# 25. Conversation Memory

Stores conversation context.

Example:

```text
User:
My service is checkout-api.

Later:
Check its logs.
```

Conversation memory allows:

```text
"its"
→ checkout-api
```

Use for:

- ongoing dialogue
- user intent
- recent interaction context

---

# 26. Working Memory

Temporary information required for the current task.

Example:

```text
Current goal
Current plan
Recent tool results
Current errors
Current hypothesis
```

Working memory should usually be short-lived.

---

# 27. Episodic Memory

Stores experiences/events.

Example:

```text
Incident:
checkout-api
Date:
2026-09-01

Cause:
bad configuration

Resolution:
rollback
```

Later:

```text
New checkout incident
 ↓
Retrieve similar incident
```

Episodic memory is:

> **What happened before?**

---

# 28. Semantic Memory

Stores generalized knowledge.

Example:

```text
Checkout API depends on PostgreSQL.
```

or:

```text
CrashLoopBackOff often indicates repeated
container startup failure.
```

Semantic memory is:

> **What do we know?**

---

# 29. Procedural Memory

Stores how to perform tasks.

Example:

```text
Incident remediation procedure:

1. Check metrics
2. Check pods
3. Check logs
4. Verify dependency
5. Request approval
6. Roll back
7. Verify
```

Procedural memory is:

> **How do we do it?**

---

# 30. Long-Term Memory

Persistent knowledge across tasks and sessions.

Can include:

```text
preferences
facts
past incidents
learned patterns
procedures
successful strategies
```

Long-term memory requires:

```text
storage
retrieval
versioning
privacy
access control
forgetting/deletion policies
```

---

# 31. Memory Architecture

A production memory system:

```text
Agent
 │
 ▼
Memory Manager
 │
 ├── Working Memory
 │
 ├── Episodic Store
 │
 ├── Semantic Store
 │
 └── Procedural Store
 │
 ▼
Retriever
 │
 ▼
Relevant Memory
 │
 ▼
Agent Context
```

Do not retrieve every memory for every task.

---

# 32. Memory Retrieval

Query:

```text
Why is checkout-api failing?
```

Possible memories:

```text
Recent incident
Historical incident
Known architecture
Runbook
Previous remediation
```

Rank by:

```text
relevance
recency
confidence
scope
tenant
permissions
```

---

# 33. Memory Safety

Memory can become harmful if it stores:

- incorrect information
- stale information
- secrets
- private information
- malicious instructions

Treat memory as data, not absolute truth.

Use:

```text
source
timestamp
confidence
version
permissions
```

---

# 34. Memory Write Policy

Do not automatically save everything.

A memory candidate should pass:

```text
Is it useful later?
Is it stable?
Is it trustworthy?
Is it safe to store?
Is it allowed by policy?
```

Example:

```text
Temporary tool result
→ working memory

Verified incident pattern
→ episodic/semantic memory

Secret
→ never memory
```

---

# 35. Single-Agent Architecture

Start with:

```text
              Agent
                │
        ┌───────┼────────┐
        ▼       ▼        ▼
      Tools   Memory   Evaluator
```

Advantages:

- simple
- cheap
- easier debugging
- less coordination
- lower latency

Default choice.

---

# 36. Supervisor → Workers

Use when tasks require real specialization.

```text
                  Supervisor
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
    Researcher      Coder         Tester
        │             │             │
        └─────────────┼─────────────┘
                      ▼
                   Reviewer
                      │
                      ▼
                     User
```

Supervisor controls:

- task assignment
- state
- routing
- completion
- aggregation

---

# 37. Router → Specialists

Use when one request belongs clearly to a domain.

```text
                  Router
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
         AWS        K8s       DB
       Specialist Specialist Specialist
```

Example:

```text
"EC2 instance unhealthy"
→ AWS

"Pod crashing"
→ Kubernetes

"Slow SQL query"
→ Database
```

Simple and efficient.

---

# 38. Parallel Agents

Use when tasks are independent.

```text
                    Goal
                     │
              ┌──────┼──────┐
              ▼      ▼      ▼
             AWS     K8s   Security
              │      │      │
              └──────┼──────┘
                     ▼
                  Synthesis
```

Good for:

- independent analysis
- research
- security/performance reviews
- evidence gathering

---

# 39. Sequential Agents

Use when each stage depends on the previous.

```text
Researcher
    ↓
Coder
    ↓
Tester
    ↓
Reviewer
```

Example:

```text
Research
 ↓
Implement
 ↓
Test
 ↓
Review
```

Do not parallelize dependent work.

---

# 40. Debate / Critique

Example:

```text
              Proposal
                 │
        ┌────────┴────────┐
        ▼                 ▼
    Agent A            Agent B
    argues              critiques
        │                 │
        └────────┬────────┘
                 ▼
             Judge
                 │
                 ▼
              Final
```

Useful when:

- correctness matters
- multiple solutions exist
- independent review is valuable

Costs:

- additional tokens
- latency
- coordination
- possible correlated mistakes

---

# 41. Planner → Executor

Architecture:

```text
              Planner
                 │
              Plan
                 │
                 ▼
             Executor
                 │
        ┌────────┼────────┐
        ▼        ▼        ▼
      Tool A   Tool B   Tool C
        │        │        │
        └────────┼────────┘
                 ▼
              Results
                 │
                 ▼
              Planner
```

Useful when:

- planning is complex
- execution is mostly deterministic
- separation improves reliability

---

# 42. Choosing the Architecture

Use this decision tree:

```text
Can one agent solve it?
       │
      YES
       │
       ▼
Single Agent

       NO
       │
       ▼
Are tasks independent?
       │
      YES ──► Parallel Agents

       NO
       │
       ▼
Is there domain specialization?
       │
      YES ──► Router / Specialists

       NO
       │
       ▼
Do you need centralized coordination?
       │
      YES ──► Supervisor / Workers

       NO
       │
       ▼
Planner → Executor
```

---

# 43. The Cost of Multi-Agent Systems

More agents do not automatically mean better results.

Costs include:

```text
more LLM calls
more tokens
more latency
more state
more coordination
more failure modes
more observability
more security boundaries
```

Before adding an agent ask:

```text
What problem does this agent solve
that another node/tool cannot?
```

---

# 44. Multi-Agent Failure Modes

### Infinite delegation

```text
Supervisor
 ↓
Worker
 ↓
Supervisor
 ↓
Worker
...
```

### Conflicting decisions

```text
AWS Agent:
scale up

K8s Agent:
rollback
```

### Duplicate work

```text
Researcher A → same research
Researcher B → same research
```

### Context explosion

```text
5 agents
 ×
full history
```

### Permission confusion

```text
Worker accidentally gets production write access
```

---

# 45. Preventing Multi-Agent Loops

Track:

```text
agent
task
parent
status
handoff_count
```

Set:

```text
max_handoffs
max depth
max total agent calls
```

Example:

```python
if state["handoff_count"] > 8:
    return "stop"
```

---

# 46. Agent Contracts

Each specialist should have a clear contract.

Example:

```text
Kubernetes Agent

Input:
Kubernetes incident context

Responsibilities:
- inspect workload
- inspect logs
- analyze events

Not responsible for:
- AWS remediation
- Terraform apply
```

This reduces overlap.

---

# 47. Shared State vs Private State

## Shared state

Useful for:

```text
supervisor
workers
```

But can become messy.

## Private state

Each agent has its own context.

Useful for:

```text
specialists
security isolation
different models
```

Then return only necessary output.

---

# 48. Agent Handoffs

A handoff should contain:

```text
task
context
required output
constraints
permissions
deadline
```

Example:

```json
{
  "task": "Analyze Kubernetes failure",
  "context": {
    "service": "checkout-api",
    "namespace": "production"
  },
  "required_output": [
    "root_cause",
    "evidence",
    "recommendation"
  ]
}
```

---

# 49. Agent Result Contracts

Do not pass free-form text between agents when possible.

Use:

```python
class InvestigationResult(BaseModel):

    root_cause: str | None

    confidence: float

    evidence: list[str]

    recommendations: list[str]
```

This makes multi-agent systems composable.

---

# 50. Supervisor Architecture

A strong supervisor should:

```text
Understand
 ↓
Decompose
 ↓
Assign
 ↓
Track
 ↓
Evaluate
 ↓
Reassign
 ↓
Synthesize
```

Not:

```text
"Ask everyone and combine text."
```

---

# 51. Multi-Agent Example — AI SRE

```text
                         Supervisor
                              │
          ┌───────────────────┼──────────────────┐
          ▼                   ▼                  ▼
     K8s Specialist       AWS Specialist     Observability
          │                   │                  │
          └───────────────────┼──────────────────┘
                              ▼
                           Evidence
                              │
                              ▼
                          Critic Agent
                              │
                              ▼
                          Supervisor
                              │
                         Human Approval
                              │
                              ▼
                        Remediation
                              │
                              ▼
                           Verify
```

---

# 52. Advanced AI Coding System

Another useful architecture:

```text
                     Supervisor
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
          Researcher   Coder      Tester
              │          │          │
              └──────────┼──────────┘
                         ▼
                      Reviewer
                         │
                         ▼
                       User
```

Use only when the work genuinely benefits from specialization.

---

# 53. Advanced Agent State

Example:

```python
class AdvancedAgentState(TypedDict):

    goal: str

    objectives: list[str]

    plan: list[dict]

    current_task: str | None

    observations: list[dict]

    evidence: list[dict]

    hypotheses: list[dict]

    agent_results: list[dict]

    memories: list[dict]

    reflection: dict | None

    verification: dict | None

    approval: dict | None

    iteration: int

    cost: float

    status: str
```

This is becoming a real agent operating state.

---

# 54. Hypothesis-Driven Agents

Instead of:

```text
randomly inspect tools
```

maintain hypotheses.

Example:

```text
H1: Database failure
H2: Kubernetes networking failure
H3: Bad deployment
```

Gather evidence:

```text
Evidence → H1
Evidence → H2
Evidence → H3
```

Score hypotheses.

```text
H1: 0.20
H2: 0.75
H3: 0.40
```

Investigate the strongest uncertain hypothesis next.

This is powerful for AI SRE systems.

---

# 55. Confidence

Agent output should distinguish:

```text
fact
inference
uncertainty
```

Example:

```text
Fact:
Pod is CrashLoopBackOff.

Fact:
Logs contain database timeout.

Inference:
Application cannot reach database.

Confidence:
0.82
```

Avoid false precision. Confidence should be meaningful only if your evaluation method supports it.

---

# 56. Evidence Graph

For complex investigations:

```text
                Incident
                   │
          ┌────────┼─────────┐
          ▼        ▼         ▼
        Metric    Log      Deploy
          │        │         │
          └────────┼─────────┘
                   ▼
               Hypothesis
                   │
                   ▼
                Evidence
                   │
                   ▼
                Conclusion
```

This is more reliable than treating the final answer as unsupported text.

---

# 57. Advanced Planning Loop

A mature agent can use:

```text
Goal
 ↓
Task Decomposition
 ↓
Candidate Plans
 ↓
Plan Evaluation
 ↓
Select Plan
 ↓
Execute
 ↓
Observe
 ↓
Verify
 ↓
Reflect
 ↓
     ┌───────────────┐
     │               │
   Success         Failure
     │               │
     ▼               ▼
  Memory          Re-plan
     │               │
     └───────┬───────┘
             ▼
           Final
```

---

# 58. Long-Horizon Tasks

Examples:

```text
Migrate service to EKS
```

Could require:

```text
inventory
design
Terraform
testing
deployment
validation
rollback
documentation
```

This can take hours.

Therefore use:

```text
persistent state
checkpoints
subtasks
verification
approval
recovery
```

---

# 59. Durable Long-Horizon Architecture

```text
                 User
                  │
                  ▼
               Run API
                  │
                  ▼
                Queue
                  │
                  ▼
             Agent Worker
                  │
                  ▼
                Graph
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
      State    Memory     Tools
        │         │         │
        └─────────┼─────────┘
                  ▼
             Checkpoint
                  │
                  ▼
              Continue
```

---

# 60. Advanced Memory + Planning

Memory can improve planning:

```text
Current Goal
     │
     ▼
Retrieve relevant memories
     │
     ▼
Planner
     │
     ▼
Plan
     │
     ▼
Execute
     │
     ▼
Verify
     │
     ▼
Store useful experience
```

Example:

```text
Past incident:
Service failed because a NetworkPolicy blocked DB traffic.

Current incident:
Similar symptoms.

Agent:
Check NetworkPolicy early.
```

Memory should influence priorities, not blindly dictate conclusions.

---

# 61. Procedural Memory for DevOps

Create reusable operational procedures:

```text
Kubernetes incident procedure
AWS EC2 incident procedure
Terraform change procedure
CI/CD failure procedure
Database incident procedure
```

Agent can retrieve the appropriate procedure.

This combines:

```text
RAG
+
Memory
+
Planning
+
Tools
```

---

# 62. Episodic Memory for Incident Management

Store:

```text
incident
timeline
observations
actions
root cause
resolution
verification
```

Future agent:

```text
Current incident
 ↓
Retrieve similar incidents
 ↓
Generate hypotheses
 ↓
Investigate
```

This can become an AI incident-learning system.

---

# 63. Semantic Memory for Platform Knowledge

Store stable knowledge:

```text
Architecture
Dependencies
Ownership
Service metadata
Known constraints
Runbooks
Infrastructure relationships
```

Potential sources:

```text
Backstage
Git
Kubernetes
Terraform
Documentation
CMDB
Databases
```

This is highly aligned with Platform Engineering.

---

# 64. Procedural + Semantic + Episodic

Example:

```text
Semantic:
checkout-api depends on PostgreSQL.

Procedural:
To troubleshoot checkout-api:
check metrics → pods → logs → DB.

Episodic:
On 2026-09-01, checkout-api failed because
a configuration change broke DB connectivity.
```

Together they create a much richer AI platform.

---

# 65. Advanced Project 1 — Planning Agent

Build:

```text
Goal
 ↓
Task Decomposition
 ↓
Plan
 ↓
Plan Validator
 ↓
Executor
 ↓
Verifier
```

Use a DevOps task:

```text
Prepare migration of an application to EKS.
```

No destructive execution.

---

# 66. Advanced Project 2 — Reflection Agent

Build:

```text
Agent
 ↓
Answer
 ↓
Critic
 ↓
Score
 ↓
Improve
 ↓
Final
```

Test on:

```text
Kubernetes troubleshooting
AWS architecture
Terraform plans
```

---

# 67. Advanced Project 3 — Memory System

Implement:

```text
working memory
episodic memory
semantic memory
procedural memory
```

Use:

```text
PostgreSQL
+
pgvector
```

or another suitable storage layer.

Create APIs:

```text
store_memory()
search_memory()
get_memory()
update_memory()
archive_memory()
```

---

# 68. Advanced Project 4 — Multi-Agent SRE

Build:

```text
                 Supervisor
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
        K8s          AWS       Observability
       Agent        Agent          Agent
          │           │           │
          └───────────┼───────────┘
                      ▼
                   Critic
                      │
                      ▼
                  Supervisor
                      │
                 Human Review
                      │
                      ▼
                  Remediator
                      │
                      ▼
                    Verify
```

---

# 69. Advanced Project 5 — AI Incident Commander

User:

```text
Production checkout service is failing.
```

System:

```text
1. Triage
2. Create investigation plan
3. Delegate
4. Collect evidence
5. Critique findings
6. Identify root cause
7. Recommend remediation
8. Request approval
9. Execute
10. Verify
11. Create incident summary
12. Store useful memory
```

This should be your major Phase 6 project.

---

# 70. Capstone Architecture

```text
                              User
                               │
                               ▼
                       AI Incident Commander
                               │
                         ┌─────┴─────┐
                         ▼           ▼
                    Planner       Memory
                         │           │
                         └─────┬─────┘
                               ▼
                           Supervisor
                               │
          ┌────────────────────┼────────────────────┐
          ▼                    ▼                    ▼
      K8s Agent             AWS Agent          Observability
          │                    │                    │
          └────────────────────┼────────────────────┘
                               ▼
                           Evidence Store
                               │
                               ▼
                            Critic
                               │
                         ┌─────┴─────┐
                         ▼           ▼
                       Good        Weak
                         │           │
                         │           ▼
                         │         Re-plan
                         │           │
                         └───────────┘
                               │
                               ▼
                         Human Approval
                               │
                               ▼
                          Remediation
                               │
                               ▼
                           Verification
                               │
                               ▼
                           Memory Write
                               │
                               ▼
                              END
```

---

# 71. Phase 6 Production Requirements

Your capstone should include:

## Planning

- [ ] decomposition
- [ ] planning
- [ ] re-planning
- [ ] plan validation
- [ ] plan repair

## Reasoning

- [ ] ReAct
- [ ] reflection
- [ ] self-critique
- [ ] verification
- [ ] hypothesis tracking

## Memory

- [ ] working
- [ ] episodic
- [ ] semantic
- [ ] procedural
- [ ] long-term
- [ ] memory retrieval
- [ ] memory write policy

## Multi-Agent

- [ ] single agent
- [ ] router
- [ ] supervisor
- [ ] parallel
- [ ] sequential
- [ ] planner/executor
- [ ] critique/debate

## Reliability

- [ ] checkpoints
- [ ] persistence
- [ ] retries
- [ ] timeouts
- [ ] cancellation
- [ ] idempotency
- [ ] loop detection

## Security

- [ ] least privilege
- [ ] agent isolation
- [ ] tool authorization
- [ ] memory access control
- [ ] audit logs
- [ ] human approval

---

# 72. Observability

Track:

```text
agent_run
plan
task
node
agent
tool
memory retrieval
memory write
LLM call
reflection
re-plan
approval
execution
verification
```

Metrics:

```text
planning_latency
planning_cost
agent_calls
tool_calls
memory_hits
memory_misses
replan_count
reflection_count
task_success
unsafe_action
human_approval
total_cost
```

---

# 73. Advanced Failure Modes

## Planning failure

```text
bad decomposition
```

## Execution failure

```text
tool unavailable
```

## Memory failure

```text
wrong memory retrieved
```

## Coordination failure

```text
agents disagree
```

## Verification failure

```text
agent declares success incorrectly
```

## Reflection failure

```text
critic approves bad answer
```

## Search failure

```text
search space explodes
```

Design each failure explicitly.

---

# 74. Architecture Principle

A useful separation:

```text
Planner
→ decides what should happen.

Orchestrator
→ controls how it happens.

Tool/MCP layer
→ performs external actions.

Memory
→ provides relevant historical/contextual information.

Evaluator
→ determines whether the result is good.

Policy
→ determines what is allowed.

Human
→ controls high-risk decisions.
```

Do not collapse everything into one giant "agent."

---

# 75. Framework Mapping

After mastering concepts, map them to frameworks.

| Concept | LangGraph | OpenAI Agents SDK | LlamaIndex |
|---|---|---|---|
| State | Graph state | Agent/session state | Workflow/context |
| Planning | Nodes/LLM | Agent behavior | Agent/workflow |
| Routing | Conditional edges | Handoffs/routing | Workflow routing |
| Memory | Checkpoints + external memory | Session/state + external memory | Data/memory integrations |
| Reflection | Graph node/loop | Agent/evaluator pattern | Workflow pattern |
| Multi-agent | Subgraphs/supervisors | Agents/handoffs | Agent/workflows |
| Human approval | Interrupts | Approval/tool patterns | Workflow pause/control |
| Parallelism | Parallel branches | Concurrent execution patterns | Workflow parallelism |

Framework APIs evolve. Keep the conceptual model independent.

---

# 76. What NOT to Do

### Don't create 10 agents.

```text
Agent A
Agent B
Agent C
...
```

without a reason.

### Don't add reflection everywhere.

More calls ≠ more intelligence.

### Don't store everything in memory.

Memory quality matters more than memory quantity.

### Don't let agents freely delegate.

Use explicit contracts and limits.

### Don't confuse planning with execution.

Keep responsibilities clear.

### Don't trust self-critique blindly.

A model can critique itself incorrectly.

Use independent verification where it matters.

---

# 77. Architecture Decision Checklist

Before introducing another agent ask:

```text
[ ] Is the task genuinely specialized?
[ ] Does it require a different context?
[ ] Does it require different permissions?
[ ] Can it run independently?
[ ] Does it justify extra latency?
[ ] Does it justify extra cost?
[ ] Can a normal tool/node solve it?
[ ] Can a deterministic workflow solve it?
```

If most answers are "no":

> Do not add the agent.

---

# 78. 10-Week Study Plan

## Weeks 1–2 — Advanced Planning

Learn:

```text
ReAct
plan-and-execute
task decomposition
dynamic planning
hierarchical planning
plan validation
plan repair
```

Build:

```text
Planning Agent
```

---

## Weeks 3–4 — Reflection + Search

Learn:

```text
reflection
self-critique
verification
hypothesis reasoning
tree-search concepts
candidate ranking
pruning
```

Build:

```text
Research + Critique Agent
```

---

## Weeks 5–6 — Memory

Learn:

```text
working memory
conversation memory
episodic
semantic
procedural
long-term
retrieval
memory write policies
```

Build:

```text
AI Incident Memory System
```

---

## Weeks 7–8 — Multi-Agent Architecture

Learn:

```text
router
supervisor
workers
parallel agents
sequential agents
handoffs
planner/executor
debate
```

Build:

```text
Multi-Agent AI SRE
```

---

## Weeks 9–10 — Capstone

Combine:

```text
Planning
+
Reflection
+
Memory
+
Multi-Agent
+
MCP
+
Human Approval
+
Verification
```

Build:

> **AI Incident Commander**

---

# 79. Daily Practice

Assuming 2–3 hours/day:

### 30 minutes

Study one concept.

### 60–90 minutes

Implement it.

### 30 minutes

Break it intentionally.

Examples:

```text
bad plan
wrong memory
agent disagreement
worker timeout
repeated delegation
false verification
```

### 15–30 minutes

Explain the architecture aloud.

---

# 80. Interview Questions

## Planning

1. What is ReAct?
2. ReAct vs plan-and-execute?
3. What is hierarchical planning?
4. How do you decompose a complex task?
5. How do you validate an agent plan?
6. How do you repair a failed plan?
7. How do you prevent planning loops?

## Reflection

8. What is reflection?
9. What is self-critique?
10. When is reflection useful?
11. Why can self-critique fail?
12. How do you independently verify an agent's conclusion?

## Search

13. What is tree search?
14. Why can search become expensive?
15. What is pruning?
16. What is beam-search-style reasoning?
17. When should you use search instead of direct planning?

## Memory

18. Working vs episodic memory?
19. Episodic vs semantic memory?
20. What is procedural memory?
21. How would you design long-term agent memory?
22. How do you prevent stale memory from harming decisions?
23. What should never be stored in agent memory?

## Multi-Agent

24. Single agent vs multi-agent?
25. Router vs supervisor?
26. Sequential vs parallel agents?
27. Planner vs executor?
28. When does debate help?
29. How do agents share state?
30. How do you prevent delegation loops?

## Production

31. How do you scale long-running agents?
32. How do you persist state?
33. How do you handle worker failure?
34. How do you control cost?
35. How do you secure agent-to-agent communication?
36. How do you observe a multi-agent run?
37. How do you evaluate whether an agent architecture is actually better?

---

# 81. Senior System Design Question

> Design an autonomous AI Incident Commander for a Kubernetes/AWS platform.

Expected architecture:

```text
                         User
                          │
                          ▼
                  Incident Commander
                          │
                ┌─────────┴─────────┐
                ▼                   ▼
             Planner              Memory
                │                   │
                └─────────┬─────────┘
                          ▼
                      Supervisor
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
            K8s          AWS        Observability
            Agent        Agent          Agent
             │            │             │
             └────────────┼─────────────┘
                          ▼
                       Evidence
                          │
                          ▼
                        Critic
                          │
                     ┌────┴────┐
                     ▼         ▼
                  Accept     Re-plan
                     │         │
                     ▼         └────► Planner
                 Approval
                     │
                     ▼
                 Remediation
                     │
                     ▼
                  Verify
                     │
                     ▼
                Memory Write
                     │
                     ▼
                    END
```

Discuss:

- state
- planning
- memory
- MCP
- tool permissions
- multi-agent coordination
- retries
- checkpoints
- human approval
- verification
- observability
- cost
- latency
- scaling
- failure recovery

---

# 82. Definition of Done

You are finished with Phase 6 when you can:

### Planning

- [ ] Explain ReAct
- [ ] Explain plan-and-execute
- [ ] Implement task decomposition
- [ ] Implement hierarchical planning
- [ ] Implement dynamic re-planning
- [ ] Validate plans
- [ ] Repair plans

### Reasoning

- [ ] Implement reflection
- [ ] Implement self-critique
- [ ] Implement verification
- [ ] Understand hypothesis-driven investigation
- [ ] Understand tree-search concepts
- [ ] Understand pruning

### Memory

- [ ] Conversation memory
- [ ] Working memory
- [ ] Episodic memory
- [ ] Semantic memory
- [ ] Procedural memory
- [ ] Long-term memory
- [ ] Retrieval
- [ ] Memory write policies
- [ ] Memory security

### Multi-Agent

- [ ] Single-agent architecture
- [ ] Router
- [ ] Supervisor
- [ ] Workers
- [ ] Parallel agents
- [ ] Sequential agents
- [ ] Planner/executor
- [ ] Debate/critique
- [ ] Agent contracts
- [ ] Handoffs
- [ ] Delegation limits

### Production

- [ ] Durable execution
- [ ] Checkpoints
- [ ] Persistence
- [ ] Cancellation
- [ ] Cost control
- [ ] Latency optimization
- [ ] Observability
- [ ] Security
- [ ] Human approval
- [ ] Verification

---

# 83. Phase 6 Final Mental Model

You should now think about an advanced agent like this:

```text
                           GOAL
                             │
                             ▼
                       UNDERSTAND
                             │
                             ▼
                       DECOMPOSE
                             │
                             ▼
                          PLAN
                             │
                   ┌─────────┴─────────┐
                   ▼                   ▼
                MEMORY              SEARCH
                   │                   │
                   └─────────┬─────────┘
                             ▼
                        ORCHESTRATE
                             │
          ┌──────────────────┼──────────────────┐
          ▼                  ▼                  ▼
       Specialist         Specialist         Specialist
          │                  │                  │
          └──────────────────┼──────────────────┘
                             ▼
                          EVIDENCE
                             │
                             ▼
                         REFLECT
                             │
                       ┌─────┴─────┐
                       ▼           ▼
                    Correct     Incorrect
                       │           │
                       ▼           ▼
                    Verify      Re-plan
                       │           │
                       └─────┬─────┘
                             ▼
                         APPROVAL
                             │
                             ▼
                         EXECUTE
                             │
                             ▼
                         VERIFY
                             │
                             ▼
                      STORE MEMORY
                             │
                             ▼
                           FINAL
```

---

# 84. North Star

The goal is not:

> "I know how to build multi-agent demos."

The goal is:

> **I can decide whether a problem needs one agent, a workflow, or a multi-agent architecture—and then design the planning, memory, orchestration, verification, security, and reliability required to make it work in production.**

A top-tier Agentic AI engineer should be able to look at a complex problem and ask:

```text
What is the goal?

Can deterministic logic solve it?

Does it need an agent?

What needs planning?

What can run in parallel?

What needs memory?

What should be delegated?

Why does each agent exist?

What happens if agents disagree?

How do we verify success?

What should be persisted?

What requires human approval?

How do we recover?

How much will it cost?

How do we observe it?
```

That is advanced agent architecture.

---

# Phase 6 → Phase 7 Transition

After Phase 6, move to:

> **Phase 7 — Context Engineering**

You will learn how to control exactly what information the model receives and when.

Topics:

```text
context windows
context compression
context selection
context routing
context caching
memory/context interaction
long-context strategies
structured context
dynamic context
retrieval-aware context
agent context budgets
```

The progression becomes:

```text
Phase 3
Agent Fundamentals
       ↓
Phase 4
Tool Calling + MCP
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
Agent Evaluation
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
AI Platform / Agent Architect
```

## Final Principle

> **Do not maximize the number of agents. Maximize the reliability of the system.**

A single well-designed agent with excellent tools, memory, planning and verification can be better than ten poorly coordinated agents.
