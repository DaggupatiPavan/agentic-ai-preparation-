# Phase 6 — Advanced Agent Architecture

## Duration: 2–3 Months

## Mission

Move from building individual agents to designing **advanced, reliable, long-horizon agent systems**.

Phase 3 taught the agent loop.

Phase 4 taught tools and MCP.

Phase 5 taught orchestration.

Phase 6 focuses on:

```text
Planning
Task Decomposition
Reasoning Strategies
Reflection
Self-Critique
Search
Memory
Multi-Agent Architecture
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

# 2. Advanced Planning

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

A strong agent can modify the plan when new evidence appears.

---

# 3. ReAct

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

# 4. ReAct Strengths and Weaknesses

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

# 5. Plan-and-Execute

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

Executor performs the plan.

---

# 6. ReAct vs Plan-and-Execute

| Strategy | ReAct | Plan-and-Execute |
|---|---|---|
| Planning | Continuous | Upfront |
| Adaptability | High | Medium |
| LLM calls | More | Potentially fewer |
| Long tasks | Can become noisy | Better structure |
| Dynamic environment | Strong | Requires re-planning |
| Debugging | Action-by-action | Plan-level |

A production system can combine both:

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

# 7. Hierarchical Planning

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

---

# 8. Task Decomposition

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

# 9. Dependency-Aware Planning

Tasks can form a DAG:

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

# 10. Plan Validation

Never blindly execute generated plans.

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
            raise PermissionError("Unauthorized action")
```

---

# 11. Plan Repair

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

# 12. Reflection

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

# 13. Self-Critique

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

Action:
Search for stronger evidence.
```

---

# 14. Reflection vs Self-Critique

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

Use an evaluator to determine whether another iteration is worthwhile.

Do not add reflection simply to increase LLM calls.

---

# 15. Verification

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

---

# 16. Hypothesis-Driven Agents

Instead of randomly inspecting tools, maintain hypotheses.

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

Then prioritize the strongest uncertain hypothesis.

This is powerful for AI SRE systems.

---

# 17. Tree-Search Concepts

You do not need to immediately implement advanced research-grade search algorithms.

Understand the concept:

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

Search is useful when multiple plausible strategies need comparison.

---

# 18. Search Cost

Search can explode.

If:

```text
5 choices × 5 choices × 5 choices
```

there are:

```text
125 possible paths
```

At depth 5:

```text
5^5 = 3125
```

Therefore understand:

```text
pruning
beam-search concepts
heuristics
budget
depth limits
candidate scoring
```

---

# 19. Memory Fundamentals

Agents need memory because not everything belongs in the current context.

Think:

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

# 20. Conversation Memory

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
- recent user intent
- recent interaction context

---

# 21. Working Memory

Temporary information required for the current task.

Example:

```text
Current goal
Current plan
Recent tool results
Current errors
Current hypothesis
```

Working memory is usually short-lived.

---

# 22. Episodic Memory

Stores experiences/events.

Example:

```text
Incident:
checkout-api

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

# 23. Semantic Memory

Stores generalized knowledge.

Example:

```text
Checkout API depends on PostgreSQL.
```

Semantic memory is:

> **What do we know?**

Sources can include:

```text
architecture
documentation
service metadata
verified facts
```

---

# 24. Procedural Memory

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

# 25. Long-Term Memory

Persistent information across sessions:

```text
past incidents
verified patterns
procedures
stable architecture facts
useful user preferences
```

Long-term memory requires:

```text
storage
retrieval
versioning
access control
privacy
quality control
```

Do not automatically store everything.

---

# 26. Memory Architecture

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

---

# 27. Memory Retrieval

Query:

```text
Why is checkout-api failing?
```

Potential memories:

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
permissions
```

---

# 28. Memory Safety

Memory can contain:

- incorrect information
- stale information
- secrets
- private information
- malicious instructions

Treat memory as data, not absolute truth.

Store metadata:

```text
source
timestamp
confidence
version
permissions
```

---

# 29. Memory Write Policy

Before storing:

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

# 30. Single-Agent Architecture

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

This should be the default choice.

---

# 31. Supervisor → Workers

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

# 32. Router → Specialists

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

---

# 33. Parallel Agents

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

# 34. Sequential Agents

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

# 35. Debate / Critique

Example:

```text
              Proposal
                 │
        ┌────────┴────────┐
        ▼                 ▼
    Agent A            Agent B
    proposes             critiques
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

# 36. Planner → Executor

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

Useful when planning is complex but execution can be controlled.

---

# 37. Architecture Selection

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

# 38. The Cost of Multi-Agent Systems

More agents mean:

```text
more LLM calls
more tokens
more latency
more state
more coordination
more failure modes
more security boundaries
more observability
```

Before adding an agent ask:

> **What problem does this agent solve that another node, tool, or workflow cannot?**

---

# 39. Multi-Agent Failure Modes

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
5 agents × full history
```

### Permission confusion

```text
Worker accidentally gets production write access
```

---

# 40. Preventing Delegation Loops

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
max handoffs
max depth
max total agent calls
```

Example:

```python
if state["handoff_count"] > 8:
    return "stop"
```

---

# 41. Agent Contracts

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

Clear boundaries reduce overlap.

---

# 42. Shared State vs Private State

## Shared state

Useful for:

```text
supervisor
workers
```

But can become messy.

## Private state

Useful for:

```text
specialists
security isolation
different models
```

Return only necessary output to the parent.

---

# 43. Agent Handoffs

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

# 44. Agent Result Contracts

Use structured results.

```python
class InvestigationResult(BaseModel):

    root_cause: str | None
    confidence: float
    evidence: list[str]
    recommendations: list[str]
```

This makes multi-agent systems composable.

---

# 45. Supervisor Responsibilities

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

# 46. AI SRE Multi-Agent Example

```text
                         Supervisor
                              │
          ┌───────────────────┼──────────────────┐
          ▼                   ▼                  ▼
     K8s Agent             AWS Agent        Observability
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

# 47. Advanced Agent State

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

This represents an operating state for a serious agent system.

---

# 48. Long-Horizon Tasks

Example:

```text
Migrate service to EKS
```

Potential subtasks:

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

Long-horizon tasks require:

```text
persistent state
checkpoints
subtasks
verification
approval
recovery
```

---

# 49. Durable Long-Horizon Architecture

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

# 50. Memory + Planning

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

Memory should influence priorities, not blindly determine conclusions.

---

# 51. Procedural Memory for DevOps

Create reusable procedures:

```text
Kubernetes incident procedure
AWS EC2 incident procedure
Terraform change procedure
CI/CD failure procedure
Database incident procedure
```

Agent retrieves the appropriate procedure.

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

# 52. Episodic Memory for Incidents

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

# 53. Semantic Memory for Platform Knowledge

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

# 54. Procedural + Semantic + Episodic

Example:

```text
Semantic:
checkout-api depends on PostgreSQL.

Procedural:
To troubleshoot checkout-api:
check metrics → pods → logs → DB.

Episodic:
A previous checkout-api incident was caused by
a configuration change that broke DB connectivity.
```

Together they create richer agent context.

---

# 55. Hands-On Project 1 — Planning Agent

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

DevOps task:

```text
Prepare migration of an application to EKS.
```

Do not perform destructive execution.

---

# 56. Hands-On Project 2 — Reflection Agent

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

# 57. Hands-On Project 3 — Memory System

Implement:

```text
working memory
episodic memory
semantic memory
procedural memory
```

Potential storage:

```text
PostgreSQL
+
pgvector
```

Create:

```text
store_memory()
search_memory()
get_memory()
update_memory()
archive_memory()
```

---

# 58. Hands-On Project 4 — Multi-Agent SRE

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

# 59. Capstone — AI Incident Commander

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

---

# 60. Capstone Architecture

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

# 61. Production Requirements

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
- [ ] tree-search concepts

## Memory

- [ ] conversation
- [ ] working
- [ ] episodic
- [ ] semantic
- [ ] procedural
- [ ] long-term
- [ ] retrieval
- [ ] memory write policy

## Multi-Agent

- [ ] single agent
- [ ] router
- [ ] supervisor
- [ ] workers
- [ ] parallel
- [ ] sequential
- [ ] planner/executor
- [ ] debate/critique
- [ ] contracts
- [ ] handoffs

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

# 62. Observability

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

# 63. Advanced Failure Modes

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

# 64. Architecture Separation

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

# 65. Architecture Decision Checklist

Before introducing another agent:

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

> **Do not add the agent.**

---

# 66. 10-Week Study Plan

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
conversation memory
working memory
episodic memory
semantic memory
procedural memory
long-term memory
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

# 67. Daily Practice

Assuming 2–3 hours/day:

### 30 minutes

Study one concept.

### 60–90 minutes

Implement it.

### 30 minutes

Break it intentionally:

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

# 68. Interview Questions

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

# 69. Senior System Design Question

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

# 70. Definition of Done

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

# 71. North Star

The goal is not:

> "I know how to build multi-agent demos."

The goal is:

> **I can decide whether a problem needs one agent, a workflow, or a multi-agent architecture—and then design the planning, memory, orchestration, verification, security, and reliability required to make it work in production.**

A top-tier Agentic AI engineer should be able to ask:

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

Next:

> **Phase 7 — Context Engineering**

Topics:

```text
context windows
context compression
context selection
memory retrieval
tool result compression
conversation summarization
state management
prompt construction
information prioritization
agent trajectory management
context security
context budgeting
context observability
```

The progression:

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

A single well-designed agent with excellent tools, memory, planning, and verification can be better than ten poorly coordinated agents.
