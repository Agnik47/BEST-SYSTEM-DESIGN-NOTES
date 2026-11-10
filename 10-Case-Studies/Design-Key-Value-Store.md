---
title: Design a Distributed Key-Value Store (Dynamo / Cassandra)
module: 10 · Case Studies
lecture: Practice (L9, L10, L16, L17)
level: Advanced
priority: ⭐ Should
status: Not Started
prerequisites: [Consistent-Hashing, Replication, CAP-and-PACELC, Indexing]
related: [Consistency-Models, Clocks-and-Ordering, Probabilistic-Data-Structures, Consensus-and-Leader-Election]
tags: [case-study, key-value, dynamo, quorum, consistent-hashing, lsm]
---

# 🗝️ Design a Distributed Key-Value Store

> [!abstract] Problem
> Build `put(key, value)` / `get(key)` that scales to petabytes and millions of ops/s across many nodes, stays available when nodes fail, with tunable consistency. Essentially: design Amazon Dynamo / Cassandra.

## 1. Clarifying questions

- Value size? → *≤ 1 MB, typical 1 KB*
- Consistency? → *tunable; default eventually consistent, option for quorum*
- Availability priority? → *always writable (AP)*
- Single region? → *start single region, multi-AZ*

## 2. Requirements

| Functional | Non-functional |
|---|---|
| `put`, `get`, `delete` | p99 < 10 ms |
| TTL optional | Highly available for writes (AP) |
| Tunable consistency per request | Horizontal scale, automatic rebalancing |
| | Durable (replication factor 3) |

## 3. Estimation

```text
100 TB data × RF 3 = 300 TB raw → at ~4 TB usable/node ≈ 75 nodes (+ headroom ≈ 100)
1M ops/s → ~10k ops/s per node — comfortable for an LSM engine on NVMe
```

## 4. The building blocks (this is the whole answer)

| Problem | Technique | Note |
|---|---|---|
| Partitioning | **Consistent hashing + vnodes** | [[Consistent-Hashing]] |
| High availability for writes | Replicate to N nodes on the ring (preference list) | [[Replication]] |
| Tunable consistency | **Quorum** W, R with W + R > N for strong-ish reads | |
| Concurrent writes | **Vector clocks** (conflict detection) or LWW timestamps | [[Clocks-and-Ordering]] |
| Temporary failures | **Sloppy quorum + hinted handoff** | |
| Permanent failures / drift | **Anti-entropy with Merkle trees** | |
| Failure detection & membership | **Gossip protocol** | |
| Storage engine | **LSM tree**: commit log + memtable + SSTables + compaction | [[Indexing]] |
| Fast negative lookups | **Bloom filters** per SSTable | [[Probabilistic-Data-Structures]] |

## 📊 Visual model

**Ring with replication (N = 3):**

```mermaid
flowchart LR
    subgraph Ring["Hash ring (clockwise)"]
      A["Node A"] --> B["Node B"] --> C["Node C"] --> D["Node D"] --> E["Node E"] --> A
    end
    K["key 'user:42' hashes here →"] -.-> B
    B -->|"replica 2"| C
    C -->|"replica 3"| D
```

Key lands on B (coordinator); replicas on the next two distinct nodes C and D.

**Quorum write and read (N=3, W=2, R=2):**

```mermaid
sequenceDiagram
    participant Cl as Client
    participant Co as Coordinator (B)
    participant C as Replica C
    participant D as Replica D
    Cl->>Co: put(k, v, ctx)
    Co->>Co: write locally (commit log + memtable)
    Co->>C: replicate
    Co->>D: replicate
    C-->>Co: ack
    Note over Co: W=2 reached (B + C) → success
    Co-->>Cl: OK
    D-->>Co: ack (late)
    Cl->>Co: get(k)
    Co->>C: read
    Co->>D: read
    C-->>Co: v (version 7)
    D-->>Co: v_old (version 6)
    Note over Co: R=2 → return newest, read-repair D
    Co-->>Cl: v
```

**Consistency vs latency knobs:**

```mermaid
quadrantChart
    title Choosing W and R for N equals 3
    x-axis Low latency --> High latency
    y-axis Weak consistency --> Strong consistency
    quadrant-1 Strong and slower
    quadrant-2 Ideal
    quadrant-3 Fast and eventual
    quadrant-4 Slow and weak
    W1 R1: [0.12, 0.15]
    W2 R1: [0.4, 0.35]
    W1 R2: [0.35, 0.38]
    W2 R2: [0.55, 0.75]
    W3 R1: [0.75, 0.8]
    W3 R3: [0.95, 0.95]
```

**LSM write/read path inside a node:**

```mermaid
flowchart LR
    W["put"] --> CL["Commit log (append, fsync)"] --> MT["Memtable (sorted, RAM)"]
    MT -->|"full → flush"| SS1["SSTable L0"]
    SS1 -->|"compaction"| SS2["SSTable L1..Ln"]
    R["get"] --> MT
    R --> BF{{"Bloom filter: maybe in SSTable?"}}
    BF -->|"no"| SKIP["skip file"]
    BF -->|"maybe"| SS1 & SS2
```

## 5. Deep dives

### Hinted handoff
If replica D is down, the coordinator writes to the next healthy node E with a **hint** ("this belongs to D"); E forwards when D returns. Keeps writes available (sloppy quorum) at the cost of weaker consistency during failures.

### Merkle-tree anti-entropy
Each node builds a hash tree over key ranges; replicas compare roots and descend only into differing branches → sync only the divergent keys, cheaply.

### Gossip
Every second each node exchanges membership/heartbeat state with a few random peers → cluster-wide view converges in O(log N) rounds without a central coordinator. Phi-accrual failure detection.

### Conflict resolution
Vector clocks → return siblings for the application to merge (Dynamo cart) **or** LWW by timestamp (Cassandra; simple, can lose writes). CRDT types for counters/sets. → [[Consistency-Models]]

### Deletes
Tombstones (a delete marker) replicated like writes; purged after `gc_grace` during compaction — otherwise deleted data can be "resurrected" by a replica that missed the delete.

## 6. Bottlenecks & trade-offs

| Choice | Trade-off |
|---|---|
| AP default | Always writable vs conflicts/staleness |
| LWW vs vector clocks | Simplicity vs never silently losing concurrent writes |
| LSM engine | Write throughput vs read amplification + compaction I/O |
| Leaderless vs leader-per-range (Raft, e.g. TiKV) | Availability vs linearizable ops |

## 7. Follow-ups & tricky questions

- *"How do you add a node?"* — It takes vnodes from others; data streams over; consistent hashing limits movement.
- *"Can you offer linearizable ops?"* — Lightweight transactions (Paxos per key in Cassandra) or a Raft group per range — at latency cost.
- *"Hot key?"* — Client-side caching, key splitting, read from any replica (R=1 for that key).
- *"Range queries?"* — Hash partitioning breaks them; use order-preserving partitioning within a partition key (clustering columns) or a range-partitioned design.

## 8. What each level should cover

| SDE-1 | SDE-2 | Senior |
|---|---|---|
| Hashing to nodes, replication, basic get/put | Consistent hashing + vnodes, quorum math, hinted handoff, LSM basics | Vector clocks vs LWW, Merkle anti-entropy, gossip, tombstones, linearizable options |

## 🧠 Concepts used
[[Consistent-Hashing]] · [[Replication]] · [[CAP-and-PACELC]] · [[Consistency-Models]] · [[Clocks-and-Ordering]] · [[Indexing]] · [[Probabilistic-Data-Structures]] · [[Consensus-and-Leader-Election]]

## My gaps
-

---
**Next:** [[Design-Food-Delivery]] · **Up:** [[Case-Studies-MOC]]
