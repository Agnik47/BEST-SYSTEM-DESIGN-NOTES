---
title: Design Typeahead / Autocomplete
module: 10 · Case Studies
lecture: Practice (L11, L21)
level: Intermediate
priority: ⭐ Should
status: Not Started
prerequisites: [Caching, Search-Systems, Batch-and-Stream-Processing]
related: [Probabilistic-Data-Structures, Design-Leaderboard-Top-K, CDN]
tags: [case-study, trie, top-k, autocomplete, caching]
---

# ⌨️ Design Typeahead / Autocomplete

> [!abstract] Problem
> As the user types in a search box ("biry…"), show the top 5–10 suggestions within ~100 ms, ranked by popularity (and personalisation), updated as trends change.

## 1. Clarifying questions

- Suggest **queries** or **entities** (products/restaurants)? → *queries, with entity boosts*
- Personalised? → *mostly global + light personalisation*
- Freshness of trends? → *within ~15 minutes for trending terms*
- Languages / typo tolerance? → *English + Hinglish, basic typo tolerance*

## 2. Requirements

| Functional | Non-functional |
|---|---|
| Top-k suggestions per prefix | p99 < 100 ms end to end (≈ 20 ms server) |
| Ranked by popularity, trends | Very read-heavy: every keystroke is a request |
| Filter offensive terms | Highly available; stale by minutes is fine |

## 3. Estimation

```text
50M DAU × 10 searches × ~6 keystrokes (after debounce) = 3B requests/day ≈ 35k/s avg, ~100k/s peak
Distinct queries tracked: ~100M; prefixes of length ≤ 15 → top-k per prefix precomputed
Cache: top 10 suggestions per popular prefix ≈ 10M prefixes × 500 B ≈ 5 GB → fits in memory
```

```mermaid
%%{init: {"themeVariables": {"xyChart": {"plotColorPalette": "#4E79A7, #E15759, #59A14F, #F28E2B, #B07AA1"}}}}%%
xychart-beta
    title "Requests per search vs debounce (illustrative)"
    x-axis ["No debounce", "100 ms", "200 ms", "300 ms"]
    y-axis "requests per typed query" 0 --> 12
    bar [11, 7, 5, 3]
```

## 4. API

```http
GET /v1/suggest?q=biry&limit=8&locale=en-IN&city=BLR
→ { "suggestions": ["biryani", "biryani near me", "biryani by kilo", ...] }
Cache-Control: public, max-age=60    (edge-cacheable for non-personalised prefixes)
```

## 5. Data structures

**Trie with top-k cached at each node** — lookup is O(length of prefix), no subtree walk at query time:

```mermaid
flowchart TD
    R(("root")) --> B(("b"))
    B --> BI(("bi"))
    BI --> BIR(("bir<br/>top: biryani, birthday cake, biriyani"))
    BIR --> BIRY(("biry<br/>top: biryani, biryani near me"))
    B --> BU(("bu<br/>top: burger, butter chicken"))
    BU --> BUR(("bur<br/>top: burger, burger king"))
```

In practice: build offline into a **prefix → top-k** map stored in a KV/Redis or a serialized trie loaded into memory on suggestion servers (sharded by prefix range).

```mermaid
erDiagram
    QUERY_COUNT {
        string query PK
        bigint count_7d
        bigint count_1h
        float score
    }
    PREFIX_TOPK {
        string prefix PK
        string locale PK
        string suggestions "JSON top 10"
        timestamp built_at
    }
```

## 6. High-level design

```mermaid
flowchart LR
    U["Search box<br/>(debounce 150 ms, client cache)"] --> CDN["CDN / edge cache<br/>(popular prefixes)"]
    CDN --> SS["Suggestion service<br/>(in-memory trie / prefix map)"]
    SS --> PZ["Personalisation blend<br/>(recent searches)"]
    subgraph Offline["Data pipeline"]
      LOG["Search logs"] --> K[["Kafka"]]
      K --> AGG["Aggregator<br/>batch (daily) + stream (5-min trends)"]
      AGG --> FIL["Filter: profanity, low counts"]
      FIL --> BLD["Trie / top-k builder"]
      BLD --> SNAP[("Snapshot in S3")]
    end
    SNAP -->|"hot reload"| SS
```

## 7. Key flow

```mermaid
sequenceDiagram
    participant U as User
    participant C as Client cache
    participant E as CDN edge
    participant S as Suggest svc
    U->>C: types "b" "bi" "bir" (debounced → "bir")
    C->>E: GET /suggest?q=bir
    E-->>C: hit (max-age 60) → suggestions
    U->>C: "biry"
    C->>E: GET /suggest?q=biry
    E->>S: miss
    S->>S: prefixMap["biry"] → top 10 (O(1))
    S-->>E: suggestions
    E-->>C: cache + return
```

## 8. Deep dives

### Ranking score
`score = w1·log(count_7d) + w2·trend(count_1h vs baseline) + boosts (locale/city) − penalties`. Trending terms come from the streaming path (e.g., "ipl final tickets"). Top-k per prefix via Count-Min + heap in the stream. → [[Probabilistic-Data-Structures]] · [[Design-Leaderboard-Top-K]]

### Updating without hurting reads
Never mutate the live trie per query. Rebuild periodically (batch daily + small trend overlay every few minutes), write a snapshot, servers **hot-swap** (blue/green in memory).

### Sharding
By prefix range (a–c, d–f, …) — but "s" is huge vs "x" → split by first 2 chars or by hash of prefix with replication; each shard replicated for availability.

### Typos & languages
Fuzzy matching (edit distance 1) for short prefixes is expensive → precompute common misspellings mapping; transliteration for Hinglish ("biryani"/"biriyani").

### Personalisation
Blend global top-k with the user's recent searches (client-side or a small per-user store) — keep the global path cacheable.

## 9. Bottlenecks & trade-offs

| Choice | Trade-off |
|---|---|
| Precomputed top-k per node | Memory vs O(1) query |
| Edge caching | Huge offload vs personalisation (keep personal blend on client) |
| Rebuild frequency | Freshness vs compute |
| Debounce length | Fewer requests vs perceived snappiness |

## 10. Follow-ups & tricky questions

- *"Why not query Elasticsearch on every keystroke?"* — Works with edge n-grams at moderate scale, but a precomputed prefix map is cheaper and faster at 100k/s.
- *"Remove a harmful suggestion immediately?"* — Blocklist checked at serve time (overrides snapshot) + purge CDN keys.
- *"Long-tail prefixes with no data?"* — Fall back to search-engine results or entity names.

## 11. What each level should cover

| SDE-1 | SDE-2 | Senior |
|---|---|---|
| Trie, top-k, caching | Precomputed top-k per node, offline pipeline, sharding, debounce + CDN | Trend streaming, ranking signals, multilingual, safe-content controls |

## 🧠 Concepts used
[[Caching]] · [[CDN]] · [[Search-Systems]] · [[Batch-and-Stream-Processing]] · [[Probabilistic-Data-Structures]] · [[Sharding-and-Partitioning]]

## My gaps
-

---
**Next:** [[Design-File-Storage-Sync]] · **Up:** [[Case-Studies-MOC]]
