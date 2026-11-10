---
title: Start Here
topic: Home
level: All
status: Not Started
tags: [home, meta]
related: [System-Design-MOC, SD-Learning-Roadmap, SD-Progress-Tracker, SD-Revision-System]
---

# 🏁 Start Here — How to Use This Vault

## What this vault is

A system design learning system built the same way as my DevOps vault: one concept per note, every note linked, MOCs per module, and an interview layer on top. It follows the 30-lecture syllabus (L1–L30) and adds the topics 2026 interviews actually probe.

## Folder layout

| Folder | What's in it |
|---|---|
| `00-Home` | Maps, roadmap, tracker, revision, trends, where to focus |
| `01`–`07` | Core concepts, in the syllabus order |
| `08-Modern-Topics` | AI-era + differentiator topics (LLM serving, RAG, CRDTs, ranking, geo, sketches) |
| `09-Interview-Playbook` | The framework, level expectations, company patterns, mistakes, LLD primer |
| `10-Case-Studies` | 19 full HLD walkthroughs |
| `11-Interview-Questions` | Conceptual, tricky, scenario, trade-off banks + rapid-fire revision |
| `13-Senior-Engineering` | SDE-2 → SDE-3 skills: migrations, schema evolution, testing, ops, cost, design docs, DDD, leadership |
| `14-Future-Engineering` | What companies will demand in 1–3 years: AI-native SDLC, verification, evals, agent protocols, AI security, platform, data, regulation |
| `12-Glossary`, `99-Resources` | Glossary, books/channels/blogs, cheat sheets |

## The concept-note format

| Section | What it gives you |
|---|---|
| **What is it?** | Plain English |
| **Why does it exist?** | The real problem it solves |
| **Analogy** | A picture for your head — then the technical truth |
| **Interview definition** | The one-liner you say out loud |
| **How it works** | Step by step, with a Mermaid diagram |
| **Trade-offs** | The table interviewers want you to reason through |
| **Who uses it** | Real systems, so your answers sound grounded |
| **Common mistakes** | The traps |
| **Interview understanding** | What they're really testing + tricky follow-ups |
| **Mini quiz** | Answers at the bottom — try first |

## The case-study format

`Clarify → Requirements (FR/NFR) → Estimation → API → Data model → High-level design → Deep dives → Bottlenecks & trade-offs → Follow-ups → What SDE-1 / SDE-2 / Senior should cover`

This is the exact order of [[Interview-Framework]]. Practising every case in the same order is the point — the order becomes muscle memory.

## Priority tags (in each note's frontmatter)

| Tag | Meaning |
|---|---|
| 🔥 **Must** | Asked in almost every loop. Know it cold. |
| ⭐ **Should** | Expected at SDE-2+, or as a deep-dive follow-up. |
| 💡 **Edge** | Differentiator — senior loops, AI companies, or "impress me" moments. |

## Status legend

```text
⬜ Not Started  — haven't opened it
🟡 Learning     — read it once, still fuzzy
🔵 Practicing   — can draw it with the note open
🟢 Comfortable  — can explain it out loud for 3 min, no notes
✅ Mastered     — did a timed mock using it, defended trade-offs under follow-ups
```

## How to study one concept (≈2 hours)

```text
1. Read the note fast.                                      (15 min)
2. Watch ONE video from the note's resources.               (20-40 min)
3. Close everything. Draw the diagram from memory.          (10 min)
4. Explain it out loud as if to an interviewer.             (5 min)
5. Answer the "tricky follow-ups" without peeking.          (15 min)
6. Write 3 lines in "My notes" in your own words.           (5 min)
7. Mini quiz. Under 80%? Back to step 3.                    (10 min)
```

## How to study one case study (≈2.5 hours)

```text
1. Read only the problem statement. Close the note.
2. Set a 45-min timer. Solve it on paper/Excalidraw using [[Interview-Framework]].
3. Open the note. Diff your design against it — section by section.
4. Write down the 2-3 things you missed in "My gaps".
5. Re-explain the deep dives out loud.
6. Revisit in 7 days — see [[SD-Revision-System]].
```

## Obsidian setup

1. **Settings → Files & Links → New link format → Shortest path when possible.** Links are `[[Note-Name]]`.
2. **Graph view** — colour groups per module are already configured. Filter by `tag:#case-study` or `tag:#moc` to see the skeleton.
3. **Canvas boards** (core plugin, already on): [[System-Design-Map.canvas|syllabus map]] and [[Case-Study-Concept-Map.canvas|case-study ↔ concept map]].
4. **Bases** (core plugin, already on): [[SD-Dashboard.base|progress dashboard]] reads the `status` / `priority` properties of every note.
5. Recommended community plugins: **Excalidraw** (draw every HLD yourself — the single best habit), **Spaced Repetition** (turns [[Rapid-Fire-Revision]] into decks), **Advanced Tables**.

## The visual toolkit used in every note

| Visual | Mermaid / Obsidian feature | Used for |
|---|---|---|
| Architecture & flows | `flowchart`, `block-beta` | HLDs, layered stacks, decision trees |
| Request timelines | `sequenceDiagram` | Key flows in every case study |
| Lifecycles | `stateDiagram-v2` | Orders, payments, seats, messages, jobs |
| Data models | `erDiagram` | Every case study's schema |
| Numbers | `xychart-beta` (bar + line), `pie` | Latency budgets, hit ratios, spikes, backoff curves |
| Trade-off maps | `quadrantChart` | CP vs AP, DB landscape, API styles, priorities |
| Traffic flow | `sankey-beta` | CDN → cache → DB, ranking funnels, RAG funnels |
| Plans & timing | `gantt`, `timeline`, `journey` | Study plans, failover drills, 45-min interview |
| Concept maps | `mindmap` | Module overviews |
| Bit layouts | `packet-beta` | Snowflake ID |
| Math | LaTeX `$$…$$` | Availability, quorum, Little's Law, Bloom filter FPR |
| Hide-and-reveal | Collapsible callouts `> [!success]-` | Quiz answers, tricky questions |

> [!note] Mermaid version
> `xychart`, `sankey`, `block` and `packet` diagrams need a recent Obsidian (it ships Mermaid 11). If one shows an error, update Obsidian. The surrounding text and tables still explain the same thing.

## What NOT to do

- ❌ Don't read 10 case studies without solving one under a timer. Reading solutions feels like learning and isn't.
- ❌ Don't memorise architectures. Memorise **trade-offs** — that's what gets graded. See [[Trade-off-Language]].
- ❌ Don't skip [[Estimation-and-Latency-Numbers]]. Numbers are what turn "add a cache" into a decision.
- ❌ Don't start with Kafka internals or Raft. Start with [[Where-to-Focus]].

---
**Next:** [[SD-Learning-Roadmap]] · **Up:** [[System-Design-MOC]]
