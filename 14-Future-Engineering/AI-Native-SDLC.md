---
title: The AI-Native Software Development Lifecycle
module: 14 · Future Engineering
lecture: — (added)
level: Intermediate
priority: 🔥 Must
status: Not Started
prerequisites: [Design-Docs-and-ADRs, Testing-Distributed-Systems]
related: [Verification-Engineering, Context-Engineering-and-Agent-Protocols, AI-Security-Engineering, Technical-Leadership-for-Engineers, Market-Research-2026]
tags: [future, ai, coding-agents, sdlc, spec-driven, code-review]
---

# The AI-Native Software Development Lifecycle

## What is it?

How software gets built when **AI coding agents** (Claude Code, Cursor, GitHub Copilot agent mode, OpenAI Codex, Gemini CLI and similar) write a large share of the code. The engineer's role shifts from typing implementation to **specifying, directing, reviewing and verifying**.

## Why it matters

- 84% of developers use or plan to use AI tools, but 46% don't trust the accuracy of their output (Stack Overflow 2025).
- Gartner expects "AI-native software engineering", where most code is AI-generated, in the medium term, and says ~80% of engineers must upskill through 2027.
- Interviews are changing too. Karat reports a shift to human-led, "AI-native" interviews that test how engineers *collaborate with* AI.
- Evidence is mixed on speed. A 2025 METR randomised study found experienced open-source developers were ~19% **slower** with AI tools on their own repos, while *believing* they were faster. **Process and verification decide whether AI helps.**

## Analogy

You become a **tech lead for a team of very fast, very confident interns**. They write a lot, quickly, and are sometimes brilliantly right and sometimes confidently wrong. Your value is clear instructions, good guardrails, and catching mistakes before they ship.

## Interview definition

> "AI-native development means specs and tests become the primary artifacts: I write a clear spec and acceptance tests, let agents implement in small, reviewable steps inside guardrails (CI, linters, security scans, sandboxes), and verify every change before merge. I measure outcomes like lead time, change-failure rate and escaped defects, not lines of code."

## The workflow

```mermaid
flowchart LR
    S["1 Spec<br/>problem, constraints, acceptance tests, non-goals"] --> P["2 Plan<br/>agent proposes plan; human reviews"]
    P --> I["3 Implement<br/>agent works in small steps, in a sandbox / branch"]
    I --> V["4 Verify<br/>tests, types, linters, SAST, evals, run it"]
    V --> R["5 Human review<br/>design, failure modes, security, explainability"]
    R --> D["6 Deploy safely<br/>flags, canary, monitoring"]
    V -->|"fails"| I
    R -->|"changes"| I
```

## Spec-driven development

Write the **what and why** precisely before generating the **how**:
- Requirements + acceptance criteria (Given/When/Then), API contracts, data model, edge cases, non-goals.
- Tools formalising this: GitHub Spec Kit, AWS Kiro, and repo-level instruction files (`AGENTS.md`, `CLAUDE.md`, `.cursor/rules`) that tell agents your conventions, commands and boundaries.
- The spec doubles as the review checklist and test plan. → [[Design-Docs-and-ADRs]]

```markdown
<!-- AGENTS.md (repo root) -->
## Build & test
- `pnpm i && pnpm test` must pass; `pnpm lint` and `pnpm typecheck` clean
## Conventions
- Money in integer paise; never floats. All POST handlers take Idempotency-Key.
## Boundaries
- Never edit /migrations without a matching down migration.
- Never add dependencies without approval; never touch .env or secrets.
```

## Guardrails that make agents safe
| Guardrail | Why |
|---|---|
| Small PRs / small tasks | Reviewable; agents drift on big tasks |
| Tests written or reviewed by humans first | Agents tend to write tests that assert their own behaviour |
| CI: types, lint, tests, SAST, dependency + licence scan, secret scan | Mechanical checks catch a large share of agent mistakes |
| Sandboxed execution, least-privilege tokens | Agents run commands; limit blast radius → [[AI-Security-Engineering]] |
| Dependency allow-list | Hallucinated package names → "slopsquatting" supply-chain attacks → [[Supply-Chain-Security-and-Post-Quantum]] |
| Provenance labels on AI-generated code | Audit and compliance |
| Human owner per change | Someone must be able to explain it |

## 📊 Visual model

**Where engineering time shifts (illustrative):**

```mermaid
%%{init: {"themeVariables": {"xyChart": {"plotColorPalette": "#E15759, #59A14F"}}}}%%
xychart-beta
    title "Share of effort by activity (%) — 2022 (bars) vs AI-native (line)"
    x-axis ["Spec and design", "Writing code", "Reviewing code", "Testing and verification", "Debugging prod", "Coordination"]
    y-axis "% of effort" 0 --> 45
    bar [10, 40, 12, 15, 13, 10]
    line [22, 12, 20, 25, 11, 10]
```

**Who does what:**

```mermaid
sequenceDiagram
    participant H as Engineer
    participant A as Coding agent
    participant CI as CI guardrails
    participant R as Reviewer
    H->>A: spec + acceptance tests + AGENTS.md context
    A->>H: plan (files, approach, risks)
    H->>A: approve / adjust plan
    A->>CI: implement in branch, run tests
    CI-->>A: 2 failing tests, 1 SAST finding
    A->>CI: fix, re-run → green
    A->>H: PR with summary + test evidence
    H->>H: review design, failure modes, security
    H->>R: second review (risky change)
    R-->>H: approve → canary deploy
```

**Which tasks to delegate:**

```mermaid
quadrantChart
    title Delegation map
    x-axis Low ambiguity --> High ambiguity
    y-axis Low risk --> High risk
    quadrant-1 Human-led, AI assists
    quadrant-2 AI drafts, strict review
    quadrant-3 Delegate fully with CI
    quadrant-4 Pair with AI, explore
    Boilerplate CRUD: [0.1, 0.15]
    Test scaffolding: [0.2, 0.2]
    Refactor with tests: [0.3, 0.4]
    Data migration: [0.4, 0.85]
    Auth changes: [0.35, 0.9]
    New architecture: [0.9, 0.8]
    Prototype UI: [0.7, 0.2]
    Incident fix in prod: [0.75, 0.95]
```

## Measuring whether AI helps
Use DORA metrics (lead time, deploy frequency, change-failure rate, time to restore) plus escaped defects, review time and rework rate. DORA's 2025 research on AI-assisted development frames AI as an **amplifier**: strong practices get stronger, weak ones get weaker.

## Skills to build
1. Writing precise specs and acceptance tests.
2. Decomposing work into agent-sized tasks.
3. Reading and reviewing code faster than you write it.
4. Verification depth. → [[Verification-Engineering]]
5. Context engineering for your repo (instruction files, MCP tools). → [[Context-Engineering-and-Agent-Protocols]]
6. Knowing the fundamentals well enough to spot plausible-but-wrong code. **This vault is that foundation.**

## Common mistakes

> [!warning] Traps to avoid
> 1. "Vibe coding" production systems: merging code nobody understands.
> 2. Letting agents write both the code and the only tests.
> 3. Giant agent PRs that can't be reviewed.
> 4. Measuring productivity by lines of code or PR count.
> 5. Giving agents prod credentials or unrestricted shell access.

## Interview understanding

> [!question] What the interviewer is really testing
> - *"How do you use AI in your workflow?"* — Spec → plan → small steps → verification → review; give a concrete example where you caught a subtle AI bug.
> - *Tricky:* "Does AI make developers faster?" — Depends; controlled studies show mixed results; gains come from process, verification and task fit.
> - *Tricky:* "What's slopsquatting?" — Attackers register package names that LLMs hallucinate, so blindly installing suggested packages pulls malware.
> - *Tricky:* "What should never be fully delegated?" — Security-critical, irreversible or ambiguous work (auth, migrations, money, architecture).

## Mini quiz
1. Six steps of the AI-native workflow?
2. What goes in AGENTS.md?
3. Why shouldn't the agent write the only tests?
4. Which metrics measure AI impact properly?
5. What did the METR 2025 study find?

> [!success]- Answers (click to reveal)
> 1. Spec → plan → implement → verify → human review → safe deploy.
> 2. Build/test commands, conventions, boundaries and forbidden actions for agents.
> 3. Tests may encode the agent's misunderstanding, so they pass while the behaviour is wrong.
> 4. DORA metrics (lead time, deploy frequency, change-failure rate, restore time) + escaped defects, rework.
> 5. Experienced devs were ~19% slower with AI on their own repos while believing they were faster.

---
**Next:** [[Verification-Engineering]] · **Up:** [[Future-Engineering-MOC]]

## 📚 Resources
- 📄 [METR — Measuring the impact of early-2025 AI on experienced open-source developer productivity](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/)
- 📄 [DORA research](https://dora.dev/research/)
- 📄 [AGENTS.md](https://agents.md/)
- 📄 [GitHub Spec Kit](https://github.com/github/spec-kit)
- 📄 [Stack Overflow 2025 Developer Survey — AI](https://survey.stackoverflow.co/2025/ai)

## My notes
-
