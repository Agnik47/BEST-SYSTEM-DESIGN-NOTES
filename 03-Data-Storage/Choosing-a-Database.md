---
title: Choosing a Database
module: 03 · Storing Data
lecture: — (added)
level: Intermediate
priority: 🔥 Must
status: Not Started
prerequisites: [SQL-vs-NoSQL, Indexing, Replication, Sharding-and-Partitioning]
related: [Cheat-Sheet-Building-Blocks, Redis-Deep-Dive, Search-Systems, RAG-and-Vector-Databases, Object-Storage]
tags: [data, database-selection, polyglot-persistence]
---

# Choosing a Database ➕

> [!info] Why this is added
> "Which database and why?" is asked in essentially every HLD round. Knowing *categories* isn't enough — you should name a **real product** and a **reason tied to the access pattern**.

## The decision flow

```mermaid
flowchart TD
    Q1{"Need multi-row ACID?<br/>(money, inventory, bookings)"} -->|yes| SQL["Postgres / MySQL<br/>→ Spanner / CockroachDB / Aurora if global scale"]
    Q1 -->|no| Q2{"Access by key only?"}
    Q2 -->|yes, tiny + hot| R["Redis"]
    Q2 -->|yes, huge + durable| DDB["DynamoDB / Cassandra"]
    Q2 -->|no| Q3{"Shape of data / query"}
    Q3 -->|"nested JSON, flexible"| M["MongoDB / Postgres JSONB"]
    Q3 -->|"append-heavy, time-ordered"| C["Cassandra / ScyllaDB / TimescaleDB / ClickHouse"]
    Q3 -->|"full-text search"| ES["Elasticsearch / OpenSearch"]
    Q3 -->|"relationships, multi-hop"| G["Neo4j"]
    Q3 -->|"similarity (embeddings)"| V["pgvector / Pinecone / Milvus / Qdrant"]
    Q3 -->|"analytics over billions of rows"| OLAP["ClickHouse / BigQuery / Snowflake / Druid"]
    Q3 -->|"files, images, video"| S3["S3 / GCS (object storage)"]
```

## Use case → database cheat table

| Use case | Pick | Why |
|---|---|---|
| Users, orders, payments, bookings | **Postgres/MySQL** | ACID, constraints, joins |
| Payments ledger at global scale | Spanner / CockroachDB / sharded Postgres | Strong consistency across regions |
| Sessions, rate-limit counters, leaderboards | **Redis** | In-memory, atomic ops, sorted sets → [[Redis-Deep-Dive]] |
| Shopping cart, user prefs (huge scale) | DynamoDB | Key access, managed, predictable latency |
| Chat messages, activity feeds, IoT | **Cassandra / ScyllaDB** | Massive writes, partition by conversation |
| Product catalogue with varied attributes | MongoDB / Postgres JSONB | Flexible schema |
| Search, autocomplete, log search | **Elasticsearch/OpenSearch** | Inverted index → [[Search-Systems]] |
| Metrics / time series | Prometheus, InfluxDB, TimescaleDB | Time-bucketed compression |
| Clickstream analytics, dashboards | **ClickHouse**, Druid, BigQuery | Columnar OLAP |
| Social graph, fraud rings | Neo4j / Neptune (or adjacency lists in SQL/KV at scale) | Multi-hop traversal |
| Embeddings for RAG / semantic search | **pgvector** (small–mid), Pinecone/Milvus/Qdrant (large) | ANN search → [[RAG-and-Vector-Databases]] |
| Images, videos, backups | **S3** | Cheap, durable blobs → [[Object-Storage]] |
| Config / leader election / service discovery | etcd / ZooKeeper | Consensus, small strongly consistent data → [[Consensus-and-Leader-Election]] |
| Geo queries ("near me") | Redis GEO, PostGIS, Elasticsearch geo | → [[Geospatial-Indexing]] |

## OLTP vs OLAP

| | OLTP | OLAP |
|---|---|---|
| Workload | Many small reads/writes | Few huge scans/aggregations |
| Storage | Row-oriented | **Column-oriented** |
| Examples | Postgres, MySQL, DynamoDB | ClickHouse, BigQuery, Snowflake, Redshift |
| Query | `WHERE id = ?` | `SUM(revenue) GROUP BY city` over 1B rows |

Never run heavy analytics on your OLTP primary — stream changes (CDC) to an OLAP store. → [[Batch-and-Stream-Processing]]

## Polyglot persistence

Real systems use several stores, each for what it's best at — e.g., a food delivery app: Postgres (orders, payments), Redis (sessions, driver locations hot cache), Cassandra (order events/tracking), Elasticsearch (restaurant search), S3 (menu images), ClickHouse (analytics). The cost: more ops, and **keeping them in sync** (CDC/outbox → [[Event-Driven-Architecture]]).

## How to say it in an interview

> "Orders need transactions and are ~2k writes/s, which a single Postgres primary handles; I'll shard by `user_id` later if needed. Delivery-location pings are ~100k writes/s, short-lived, and only queried by proximity, so they go to Redis GEO with a TTL, and a sampled trail goes to Cassandra for history."

Access pattern → number → product → trade-off. Every time.

## 📊 Visual model

```mermaid
mindmap
  root((Pick a store))
    Relational
      Postgres
      MySQL
      Spanner
      CockroachDB
    Key-value
      Redis
      DynamoDB
    Wide-column
      Cassandra
      ScyllaDB
      Bigtable
    Document
      MongoDB
      Firestore
    Search
      Elasticsearch
      OpenSearch
    Analytics OLAP
      ClickHouse
      BigQuery
      Snowflake
    Vector
      pgvector
      Pinecone
      Qdrant
    Blob
      S3
      GCS
    Coordination
      etcd
      ZooKeeper
```

**Access pattern map:**

```mermaid
quadrantChart
    title Pick by read shape and write volume
    x-axis Key lookups --> Rich queries
    y-axis Low write volume --> Massive write volume
    quadrant-1 Search and analytics
    quadrant-2 Wide-column and KV at scale
    quadrant-3 Cache and config
    quadrant-4 Relational OLTP
    Redis: [0.1, 0.4]
    DynamoDB: [0.15, 0.82]
    Cassandra: [0.25, 0.92]
    Postgres: [0.75, 0.35]
    MySQL: [0.7, 0.3]
    MongoDB: [0.55, 0.5]
    Elasticsearch: [0.85, 0.6]
    ClickHouse: [0.92, 0.85]
    etcd: [0.08, 0.08]
```

## Common mistakes

> [!warning] Traps to avoid
> 1. Naming a DB with no reason.
> 2. One DB for everything at large scale ("put the logs in Postgres").
> 3. Too many DBs for a small system (ops cost is real).
> 4. Forgetting the sync problem between stores.

## Mini quiz
1. Best store for a leaderboard?
2. Row vs column store — which for `SUM` over a billion rows?
3. Where do embeddings go for a small RAG app?
4. What do you use for leader election metadata?
5. Chat messages at 1M writes/s?

> [!success]- Answers (click to reveal)
> 1. Redis sorted set.
> 2. Column store (OLAP).
> 3. pgvector in Postgres.
> 4. etcd / ZooKeeper.
> 5. Cassandra/ScyllaDB partitioned by conversation (+ time bucket).

---
**Next:** [[Performance-MOC]] · **Up:** [[Data-Storage-MOC]]

## 📚 Resources
- 📄 [DB-Engines ranking](https://db-engines.com/en/ranking) — *Why:* quick sense of what's widely used.
- 📖 *DDIA* Ch. 3 (OLTP vs OLAP, column storage).

## My notes
-
