---
title: Future Engineering MOC (2027–2029)
topic: Future Engineering
module: 14
level: Advanced
status: Not Started
tags: [moc, future, ai-era, 2027]
related: [System-Design-MOC, Market-Research-2026, SDE-Career-Ladder, Senior-Engineering-MOC, Modern-Topics-MOC]
---

# 🔭 14 · Future Engineering MOC — What Companies Will Demand in 1–3 Years ➕

> [!important] The thesis (from [[Market-Research-2026]])
> AI is making **code cheap** and **judgment expensive**. Over the next 1–3 years the engineers in demand will be the ones who can (1) direct and *verify* AI-written software, (2) build reliable, safe, affordable AI features, and (3) run the platforms, data and security those features depend on. India's GCCs already report their biggest shortfalls in exactly these areas: AI/data (43%) and platform engineering (38%), plus MLOps, AI observability, FinOps and security.

## How to use this module

You **don't** need all of it. Read the overview of every note once (awareness), then pick **one spike** to go deep on and prove with a project. With my stack (Node/Next/Python + Gemini), the natural spike is the **AI engineering track**.

## The notes

| # | Note | Track | Why it will matter |
|---|---|---|---|
| 1 | [[AI-Native-SDLC]] | Everyone | How software gets built when agents write much of the code |
| 2 | [[Verification-Engineering]] | Everyone | Verification is the scarce skill when generation is cheap |
| 3 | [[LLM-Evals-and-AI-Observability]] | AI eng | The #1 signal of real LLM production experience |
| 4 | [[Context-Engineering-and-Agent-Protocols]] | AI eng | MCP, A2A, tool design, memory: the plumbing of agents |
| 5 | [[AI-Security-Engineering]] | AI eng / security | Prompt injection and excessive agency are the new SQLi |
| 6 | [[AI-Cost-and-Small-Models]] | AI eng / platform | Routing, distillation, on-device: making AI affordable |
| 7 | [[Platform-Engineering]] | Platform | Internal developer platforms; a 38% talent gap |
| 8 | [[Data-Engineering-for-AI]] | Data | AI is only as good as its data pipelines and contracts |
| 9 | [[Edge-Wasm-and-eBPF]] | Platform / infra | Compute moving to the edge and into the kernel |
| 10 | [[Supply-Chain-Security-and-Post-Quantum]] | Security | SBOMs, signing, and the crypto migration ahead |
| 11 | [[Privacy-and-AI-Regulation]] | Everyone (senior) | DPDP, GDPR, EU AI Act are architecture requirements |

## The map

```mermaid
mindmap
  root((Engineer of 2028))
    Build with AI
      AI-native SDLC
      Verification engineering
      Specs and design docs
    Build AI features
      Evals and observability
      Context and agent protocols
      AI security
      Cost and small models
      RAG and agents from module 08
    Run the foundations
      Platform engineering
      Data engineering for AI
      Edge Wasm eBPF
      FinOps from module 13
    Stay trustworthy
      Supply chain security
      Post-quantum readiness
      Privacy and AI regulation
```

**Pick your spike:**

```mermaid
quadrantChart
    title Future tracks — demand vs fit for a full-stack JS and Python dev
    x-axis Lower fit with my stack --> Higher fit with my stack
    y-axis Lower demand --> Higher demand
    quadrant-1 Best spike
    quadrant-2 High demand, more ramp-up
    quadrant-3 Awareness only
    quadrant-4 Nice side skill
    AI engineering: [0.85, 0.92]
    Platform engineering: [0.45, 0.8]
    Data engineering for AI: [0.55, 0.75]
    AI security: [0.6, 0.7]
    Edge and Wasm: [0.7, 0.35]
    eBPF kernel: [0.2, 0.3]
    Post-quantum crypto: [0.25, 0.4]
```

> [!note] Positions are my judgment based on the demand data in [[Market-Research-2026]] and the Node/Next/Python + LLM stack, not survey data.

## Suggested order (12 weeks, alongside core prep)

```mermaid
gantt
    title Future-skills sprint (2-3 hrs/day)
    dateFormat YYYY-MM-DD
    axisFormat W%W
    section Everyone
    AI-native SDLC + verification        :f1, 2026-12-01, 14d
    section AI engineering spike
    Evals and observability              :f2, after f1, 14d
    Context engineering + MCP            :f3, after f2, 10d
    AI security                          :f4, after f3, 10d
    Cost + small models                  :f5, after f4, 7d
    section Awareness
    Platform, data, edge, supply chain, regulation :f6, after f5, 21d
    section Evidence
    Ship agent + eval suite + write-up   :crit, f7, 2027-01-15, 45d
```

## The evidence that gets you hired (portfolio ideas)
1. An **MCP server** for a real tool you use (e.g., your JobPilot data) with OAuth and tests. → [[Context-Engineering-and-Agent-Protocols]]
2. A **RAG or agent app with an eval suite in CI**, a cost-per-task dashboard, and a write-up of what the evals caught. → [[LLM-Evals-and-AI-Observability]]
3. A **prompt-injection red-team report** on your own app with fixes. → [[AI-Security-Engineering]]
4. A **design doc** for one of the above with alternatives, cost and rollout. → [[Design-Docs-and-ADRs]]

**Prev module:** [[Senior-Engineering-MOC]] · **Up:** [[System-Design-MOC]] · **Builds on:** [[Modern-Topics-MOC]]
