---
title: The Story of an App Growing Up
module: 02 · Scaling
lecture: L5
level: Beginner
priority: 🔥 Must
status: Not Started
prerequisites: [How-an-App-Works, Estimation-and-Latency-Numbers]
related: [Scaling-Servers-Stateless-and-SPOF, Load-Balancing, Caching, Replication, Sharding-and-Partitioning, CDN, Message-Queues]
tags: [scaling, architecture-evolution, course-map]
---

# The Story of an App Growing Up (1 → 10M users)

## What is it?

A narrative of how architecture evolves as traffic grows — each step adding **one component to fix one specific bottleneck**. It's also the map of this whole vault.

## Why it matters

Interviewers love "start simple and evolve". Showing you'd **not** build the 10M-user architecture for 1k users is a maturity signal. And each stage gives you the *reason* for a component, which is exactly what's graded.

## The story

```mermaid
flowchart TB
    subgraph S1["Stage 1 · ~100 users"]
      U1["Users"] --> M1["1 server: app + DB"]
    end
    subgraph S2["Stage 2 · ~10k"]
      U2["Users"] --> A2["App server"] --> D2[("DB server")]
    end
    subgraph S3["Stage 3 · ~100k"]
      U3["Users"] --> LB3["Load balancer"] --> A3a["App 1"] & A3b["App 2"] & A3c["App 3"]
      A3a & A3b & A3c --> D3[("DB")]
    end
    subgraph S4["Stage 4 · ~1M"]
      U4["Users"] --> CDN4["CDN"]
      U4 --> LB4["LB"] --> AP4["Stateless app fleet"]
      AP4 --> R4[("Redis cache")]
      AP4 --> P4[("Primary")] --> RR4[("Read replicas")]
      AP4 --> Q4["Queue"] --> W4["Workers"]
    end
    subgraph S5["Stage 5 · ~10M+"]
      U5["Users"] --> G5["GeoDNS"] --> RG1["Region A"] & RG2["Region B"]
      RG1 --> SH5[("Sharded DB")]
    end
```

| Stage | Bottleneck hit | Fix | Note |
|---|---|---|---|
| 1 | Everything on one box; app and DB fight for CPU/RAM | Separate DB server | — |
| 2 | One app server is a SPOF and CPU-bound | Multiple app servers + **load balancer** | [[Load-Balancing]] |
| 3 | Sessions stored in server memory break when LB switches servers | Make servers **stateless** (sessions → Redis/JWT) | [[Scaling-Servers-Stateless-and-SPOF]] |
| 4 | DB overwhelmed by repeat reads | **Cache** (Redis) | [[Caching]] |
| 5 | Still too many reads; DB is a SPOF | **Read replicas** + failover | [[Replication]] |
| 6 | Images/JS hog bandwidth, far users are slow | **Object storage + CDN** | [[Object-Storage]] · [[CDN]] |
| 7 | Slow tasks (email, video encode) block requests; spikes crush the system | **Message queue** + workers | [[Message-Queues]] |
| 8 | Write volume / data size exceeds one primary | **Sharding** | [[Sharding-and-Partitioning]] |
| 9 | Region outage; global latency | **Multi-region** | [[Multi-Region-and-Disaster-Recovery]] |
| 10 | One codebase, 200 engineers stepping on each other | **Microservices** (org scaling, not traffic!) | [[Microservices-vs-Monolith]] |

## The key insight

**Scaling is removing the current bottleneck, measuring, and finding the next one.** Order matters: cache before sharding (sharding is expensive and permanent); vertical scaling before horizontal for the DB (it's simpler).

## Real examples

- **Instagram** ran on a small Django + Postgres setup with a handful of engineers to tens of millions of users, then sharded Postgres by user ID.
- **Stack Overflow** famously served huge traffic from a small number of powerful (vertically scaled) SQL Server machines with heavy caching.
- **WhatsApp** scaled on Erlang with a tiny team by optimising per-server connection counts.

Lesson: the "right" architecture is the simplest one that meets the numbers.

## 📊 Visual model

```mermaid
timeline
    title Architecture evolution by user count
    section Small
        ~100 users : One box — app + DB
        ~10k users : Separate DB server
    section Growing
        ~100k users : Load balancer + N stateless servers : Sessions moved to Redis
        ~1M users : Cache : Read replicas : CDN + object storage : Queue + workers
    section Large
        ~10M users : Sharding : Multi-region : Microservices for team scaling
```

**The bottleneck ladder** — fix the cheapest thing first:

```mermaid
flowchart LR
    A["Slow queries?"] -->|"index"| B["Repeat reads?"] -->|"cache"| C["Read volume?"] -->|"replicas"| D["Static bytes?"] -->|"CDN"| E["Spiky/slow work?"] -->|"queue"| F["Write volume / size?"] -->|"shard"| G["Region risk / global latency?"] -->|"multi-region"| H["Done (for now)"]
    style F fill:#ffe0e0
    style G fill:#ffe0e0
```

Red = expensive and hard to undo. Everything before them is cheap and reversible.

## Common mistakes

> [!warning] Traps to avoid
> 1. Starting the interview at Stage 5.
> 2. Adding microservices to solve a *traffic* problem (they solve *team* problems).
> 3. Sharding before trying caching, indexing, replicas, and a bigger box.
> 4. Forgetting that each component is new failure surface + cost.

## Interview understanding

> [!question] What the interviewer is really testing
> - *"How would you scale this to 10× users?"* — Walk this table: identify the current bottleneck *with a number*, apply the next fix.
> - *Tricky:* "Why not shard from day one?" — Cross-shard queries, resharding pain, operational cost; YAGNI until a single primary (with vertical scaling + replicas + cache) can't keep up.
> - *Tricky:* "What's the first thing that breaks at 10× traffic?" — Usually the database (connections, write throughput) — not the app servers.

## Mini quiz
1. What problem does the load balancer solve that a bigger server doesn't?
2. Why must servers become stateless before adding more of them?
3. Cache or shard first? Why?
4. What problem do microservices actually solve?
5. Which component handles traffic spikes by buffering?

> [!success]- Answers (click to reveal)
> 1. Redundancy (no SPOF) + horizontal capacity.
> 2. Otherwise the LB sends a user to a server without their session.
> 3. Cache — cheap, reversible; sharding is costly and hard to undo.
> 4. Organisational scaling: independent deploys and ownership.
> 5. A message queue.

---
**Next:** [[Scaling-Servers-Stateless-and-SPOF]] · **Up:** [[Scaling-MOC]]

## 📚 Resources
- 📖 *System Design Interview Vol 1* (Alex Xu), Ch. 1 "Scale from zero to millions of users" — the canonical version of this story.
- 🎥 Search "scale from zero to millions of users" on ByteByteGo.

## My notes
-
