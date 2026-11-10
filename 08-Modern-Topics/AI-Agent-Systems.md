---
title: AI Agent Systems
module: 08 · Modern Topics
lecture: — (added)
level: Advanced
priority: 💡 Edge
status: Not Started
prerequisites: [LLM-Serving-and-Inference, RAG-and-Vector-Databases, Distributed-Transactions]
related: [Design-LLM-Chat-Service, Security-Essentials, Design-Job-Scheduler, Idempotency]
tags: [modern, agents, tool-calling, sandboxing, memory, guardrails, workflows]
---

# AI Agent Systems ➕

## What is it?

An **agent** is an LLM running in a loop: it reads the goal + context, decides to **call a tool** (search, run code, query a DB, call an API), observes the result, and repeats until done. The system-design problem is everything around that loop: orchestration, state, tools, sandboxing, cost, safety, and debugging.

## The agent loop

```text
while not done and steps < max_steps and budget_left:
    action = llm(goal, history, tool_specs)        # think → choose tool or answer
    if action.type == "final": return action.answer
    result = run_tool_sandboxed(action.tool, action.args)   # with auth, timeout, idempotency
    history.append(action, result)                 # memory
```

## Components

| Component | Responsibility | Design notes |
|---|---|---|
| **Orchestrator** | Runs the loop, enforces step/time/token budgets | Long-running → durable workflow engine (Temporal) so a crash resumes mid-task → [[Distributed-Transactions]] |
| **Tool registry** | Tool schemas (JSON schema), auth scopes, rate limits | Standard protocols like **MCP (Model Context Protocol)** expose tools uniformly |
| **Sandbox** | Executes code/browsing in isolation | Firecracker microVMs / gVisor containers, no network by default, CPU/mem/time limits, ephemeral FS |
| **Memory** | Short-term (conversation/scratchpad), long-term (facts, past tasks) | Summarise/compact history; long-term memory in a vector store/DB with user scoping |
| **Retrieval** | Ground the agent in docs | → [[RAG-and-Vector-Databases]] |
| **Guardrails** | Input/output filtering, policy checks, PII redaction | Treat tool outputs and web pages as **untrusted** (prompt injection) |
| **Human-in-the-loop** | Approval for irreversible actions (payments, emails, deletes) | Pause workflow, notify, resume on approval |
| **Tracing & evals** | Record every step (prompt, tool call, result, tokens, latency) | Replay failures; task-success evals in CI |
| **Metering** | Tokens + tool costs per run per tenant | Budgets and hard caps |

## Design concerns interviewers probe

- **Reliability:** LLM outputs are non-deterministic → validate tool args against schemas, retry with feedback, cap steps, fall back.
- **Idempotent tools:** the orchestrator may retry a step after a crash → side-effecting tools need idempotency keys. → [[Idempotency]]
- **Security:** least-privilege credentials per tool, scoped to the user; sandbox egress allow-lists; confirm before irreversible actions; defend against injected instructions in retrieved content.
- **Latency & cost:** parallel tool calls, smaller models for routing/classification, caching tool results, prompt caching of long system prompts.
- **Multi-agent:** planner + specialised workers — adds coordination cost; use only if a single agent with tools fails.

## 📊 Visual model

**Architecture:**

```mermaid
flowchart LR
    U["User / trigger"] --> API["Agent API"] --> WF["Durable orchestrator (workflow engine)<br/>budgets · step limits · checkpoints"]
    WF <--> LLM["LLM gateway"]
    WF --> TR["Tool router (MCP)"]
    TR --> T1["Search / RAG"]
    TR --> T2["Code sandbox (microVM)"]
    TR --> T3["Business APIs (scoped creds)"]
    TR --> T4["Browser (isolated)"]
    WF <--> MEM[("Memory: short-term state + long-term vector store")]
    WF --> HITL["Approval queue for risky actions"]
    WF --> TRACE[("Traces + evals")]
```

**One run, step by step:**

```mermaid
sequenceDiagram
    participant U as User
    participant O as Orchestrator
    participant L as LLM
    participant T as Tools
    participant H as Human approver
    U->>O: "Refund order 991 if it arrived damaged"
    O->>L: goal + tools spec
    L-->>O: call get_order(991)
    O->>T: get_order(991)
    T-->>O: {status: delivered, photos: [...]}
    O->>L: observation
    L-->>O: call classify_damage(photos)
    O->>T: classify_damage
    T-->>O: damaged: 0.94
    L-->>O: call issue_refund(991, ₹1499) — irreversible
    O->>H: approval request
    H-->>O: approved
    O->>T: issue_refund (idempotency key run_77:step_4)
    O-->>U: "Refund issued, reference RF-2231"
```

**Agent run state machine:**

```mermaid
stateDiagram-v2
    [*] --> Planning
    Planning --> CallingTool: tool chosen
    CallingTool --> Observing: result / timeout
    Observing --> Planning: continue
    Planning --> AwaitingApproval: irreversible action
    AwaitingApproval --> CallingTool: approved
    AwaitingApproval --> Done: rejected
    Planning --> Done: final answer
    Planning --> Failed: step / token budget exceeded
    Done --> [*]
    Failed --> [*]
```

**Risk vs autonomy for tools:**

```mermaid
quadrantChart
    title Tool policy
    x-axis Reversible --> Irreversible
    y-axis Low blast radius --> High blast radius
    quadrant-1 Human approval required
    quadrant-2 Rate limit and log
    quadrant-3 Auto run
    quadrant-4 Confirm with user
    Web search: [0.1, 0.1]
    Read DB: [0.15, 0.3]
    Run code in sandbox: [0.25, 0.4]
    Send email: [0.8, 0.45]
    Issue refund: [0.9, 0.7]
    Delete records: [0.95, 0.9]
    Draft document: [0.2, 0.15]
```

## Common mistakes

> [!warning] Traps to avoid
> 1. No step/token budgets → runaway loops and bills.
> 2. Tools with broad credentials ("the agent has admin").
> 3. Running generated code without isolation.
> 4. Non-idempotent side-effecting tools behind a retrying orchestrator.
> 5. No tracing → impossible to debug "why did it do that?".

## Interview understanding

> [!question] What the interviewer is really testing
> - *"Design infrastructure for AI agents."* — Durable orchestrator, tool registry (MCP), sandboxes, memory, guardrails, HITL, tracing/evals, metering.
> - *Tricky:* "How do you make a 2-hour agent task survive a crash?" — Durable workflow engine with checkpointed steps; idempotent tools; resume from last completed step.
> - *Tricky:* "How do you defend against prompt injection from a web page?" — Treat content as data; restrict tool permissions; require confirmation for sensitive actions; separate instructions from retrieved text; output filtering.
> - *Tricky:* "Single agent vs multi-agent?" — Start single with good tools; multi-agent when tasks are parallelisable or need distinct specialisations, accepting coordination cost.

## Mini quiz
1. What's the agent loop?
2. Why a durable workflow engine?
3. What isolates code execution?
4. Which actions need human approval?
5. What's MCP?

> [!success]- Answers (click to reveal)
> 1. LLM decides → tool runs → observation appended → repeat until final answer or budget.
> 2. Long-running, multi-step tasks must survive crashes and resume.
> 3. Sandboxes — microVMs (Firecracker) / gVisor with resource and network limits.
> 4. Irreversible or high-blast-radius ones (payments, deletes, external messages).
> 5. Model Context Protocol — a standard way to expose tools/data to models.

---
**Next:** [[Interview-Playbook-MOC]] · **Up:** [[Modern-Topics-MOC]]

## 📚 Resources
- 📄 [Anthropic — Building effective agents](https://www.anthropic.com/research/building-effective-agents)
- 📄 [Model Context Protocol](https://modelcontextprotocol.io/)
- 📄 [OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)

## My notes
-
