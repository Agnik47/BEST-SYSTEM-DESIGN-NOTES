---
title: Technical Leadership for Engineers (SDE-2 → SDE-3)
module: 13 · Senior Engineering
lecture: — (added)
level: Advanced
priority: ⭐ Should
status: Not Started
prerequisites: [Design-Docs-and-ADRs, Operational-Excellence-and-Incidents]
related: [SDE-Career-Ladder, Level-Expectations, Large-Scale-Migrations, Cost-Aware-Architecture, AI-Native-SDLC]
tags: [senior, leadership, scoping, estimation, tech-debt, mentoring, influence]
---

# Technical Leadership for Engineers

## What is it?

The non-coding half of a senior engineer's job: turning vague problems into scoped projects, estimating and sequencing work, managing tech debt, making build-vs-buy calls, mentoring, and **influencing decisions without authority**.

## Why it matters now

- It's the clearest line between SDE-2 and SDE-3 (see [[SDE-Career-Ladder]]).
- With AI agents doing more implementation, **judgment, scoping and review** are what remain scarce. Companies are cutting coordination layers (Amazon removed management layers; Canva merged teams into small empowered groups), so ICs are expected to lead more.

## Analogy

A film director doesn't operate every camera. They decide the story, cast, schedule and trade-offs, unblock the crew, and are accountable for the movie.

## Interview definition

> "Technical leadership is multiplying a team's impact: framing the right problem, choosing a simple design, de-risking early, sequencing delivery, keeping quality high through reviews and standards, and growing other engineers, mostly through influence rather than authority."

## The skills

### 1. Scoping ambiguous problems
Restate the problem as a measurable goal → find the riskiest assumption → design a thin vertical slice that tests it → define milestones with user-visible value each.

### 2. Estimation
- Break into ≤ 3-day tasks; estimate ranges (best/likely/worst), not single numbers.
- Add explicit buffers for integration, migrations, reviews, on-call.
- Re-estimate when you learn; communicate changes early.
- AI-era twist: implementation is faster, but **review, verification and integration** don't shrink as much. Budget for them.

### 3. Prioritisation
Impact vs effort, with **reversibility** as a third axis (Amazon's one-way vs two-way doors: decide two-way doors fast; slow down on one-way doors).

### 4. Tech debt management
Make it visible (register with cost of delay), tie paydown to business outcomes ("deploys 3× faster", "pages down 60%"), reserve capacity (~15–20% per sprint), prefer incremental refactors in the path of feature work.

### 5. Build vs buy
| Build when | Buy / use OSS when |
|---|---|
| Core differentiator | Commodity (auth, email, payments gateway, observability) |
| Unique constraints (scale, compliance) | Vendor meets 80%+ and is cheaper than team-time |
| Long-term ownership is acceptable | You need it this quarter |

### 6. Code review & standards
Review for **correctness, design, failure modes, tests, readability**, not style (automate that). For AI-generated PRs: demand tests and explanations; reject code nobody can explain. → [[AI-Native-SDLC]]

### 7. Mentoring & raising the bar
Pairing, design-doc feedback, letting juniors own incidents with backup, interview bar-raising.

### 8. Influence without authority
Write things down, bring data, understand the other team's incentives, offer options not ultimatums, disagree and commit once decided.

## 📊 Visual model

**Impact vs effort vs reversibility:**

```mermaid
quadrantChart
    title Prioritising work
    x-axis Low effort --> High effort
    y-axis Low impact --> High impact
    quadrant-1 Plan carefully
    quadrant-2 Do now
    quadrant-3 Batch or skip
    quadrant-4 Avoid
    Add SLO alerts: [0.2, 0.8]
    Fix flaky tests: [0.3, 0.6]
    Rewrite service in Rust: [0.9, 0.35]
    DB migration to sharded: [0.85, 0.85]
    Rename internal fields: [0.4, 0.1]
    Remove dead feature flags: [0.15, 0.4]
```

**Thin vertical slices beat horizontal layers:**

```mermaid
flowchart LR
    subgraph Horizontal["❌ Horizontal: value only at the end"]
      H1["All DB work"] --> H2["All APIs"] --> H3["All UI"] --> H4["Value 🎉 (month 4)"]
    end
    subgraph Vertical["✅ Vertical slices: value every 2 weeks"]
      V1["Slice 1: one flow end to end"] --> V2["Slice 2: + edge cases"] --> V3["Slice 3: + scale"] --> V4["Slice 4: + polish"]
    end
```

**Estimate as ranges (cone of uncertainty):**

```mermaid
%%{init: {"themeVariables": {"xyChart": {"plotColorPalette": "#E15759, #4E79A7, #59A14F"}}}}%%
xychart-beta
    title "Estimate range narrows as you learn (weeks)"
    x-axis ["Idea", "Design doc", "Prototype", "Mid-build", "Code complete"]
    y-axis "weeks" 0 --> 20
    line [18, 12, 9, 7.5, 6.5]
    line [8, 7, 6.5, 6.5, 6.2]
    line [3, 4, 5, 5.5, 6]
```

> [!note] Line 1 = worst case, line 2 = likely, line 3 = best case. Communicate the range, not the best case.

**Tech debt as a portfolio:**

```mermaid
mindmap
  root((Tech debt))
    Deliberate
      Shortcut for launch
      Tracked with payback date
    Accidental
      Learned a better design later
    Bit rot
      Outdated deps
      Dead flags
    Operational
      Toil
      Noisy alerts
    Paydown tactics
      20 percent capacity
      Refactor in path of features
      Tie to metrics
```

## Behavioural questions this note prepares you for
- "Tell me about a time you disagreed with a technical decision." → data, alternatives, disagree-and-commit.
- "A project you scoped from an ambiguous ask." → goal, slices, risks, outcome with numbers.
- "How do you handle tech debt vs features?" → visibility, business framing, reserved capacity.
- "How did you help someone grow?" → concrete mentoring, their outcome.
- "Biggest mistake / failed project?" → what you learned, what you changed systemically.

## Common mistakes

> [!warning] Traps to avoid
> 1. Being the hero who fixes everything personally (doesn't scale, burns out).
> 2. Single-number estimates that become deadlines.
> 3. "Rewrite it" as the answer to every debt.
> 4. Approving AI-generated code you can't explain.
> 5. Winning arguments instead of reaching good decisions.

## Mini quiz
1. One-way vs two-way door decisions?
2. Why thin vertical slices?
3. How do you justify tech-debt work?
4. What to review in an AI-generated PR?
5. When to build vs buy?

> [!success]- Answers (click to reveal)
> 1. Irreversible decisions need care; reversible ones should be made fast.
> 2. Deliver value and learning early; de-risk integration.
> 3. Quantify the cost of delay and tie paydown to business metrics (speed, reliability, cost).
> 4. Correctness, failure modes, tests that prove behaviour, security, and whether the author can explain it.
> 5. Build core differentiators; buy commodities where vendors meet needs at lower total cost.

---
**Up:** [[Senior-Engineering-MOC]] · **Next module:** [[Future-Engineering-MOC]]

## 📚 Resources
- 📖 *The Staff Engineer's Path* (Tanya Reilly) — the best book on IC leadership.
- 📖 *An Elegant Puzzle* (Will Larson) and [staffeng.com](https://staffeng.com/)
- 📄 [Gergely Orosz — The Pragmatic Engineer](https://newsletter.pragmaticengineer.com/)
- 📄 Jeff Bezos, 2015 Amazon shareholder letter — "Type 1 vs Type 2 decisions" (the origin of one-way/two-way doors; search the title to read it)

## My notes
-
