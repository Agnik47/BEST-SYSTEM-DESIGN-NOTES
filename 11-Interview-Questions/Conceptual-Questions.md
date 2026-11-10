---
title: Conceptual Questions
module: 11 · Interview Questions
level: Beginner
priority: 🔥 Must
status: Not Started
tags: [interview, questions, conceptual]
related: [Tricky-Questions, Rapid-Fire-Revision, SD-Glossary]
---

# 📘 Conceptual Questions (30-second answers)

> Answer out loud first, then click the answer to check. Each links to its concept note.

## Foundations

**1. What is system design? HLD vs LLD?**
> [!success]- Answer
> Defining components, data flow, interfaces and storage to meet functional and non-functional requirements at scale. HLD = services, DBs, caches, queues and how they connect; LLD = classes, interfaces, patterns inside a component. → [[What-is-System-Design]]

**2. Functional vs non-functional requirements?**
> [!success]- Answer
> FR = what the system does (send message). NFR = how well (p99 < 200 ms, 99.99% available, eventual consistency OK). FRs drive APIs; NFRs drive architecture.

**3. Latency vs throughput?**
> [!success]- Answer
> Latency = time for one request; throughput = requests per second. Batching usually raises throughput but adds latency. → [[Estimation-and-Latency-Numbers]]

**4. Why p99 over average?**
> [!success]- Answer
> Averages hide tail pain; with fan-out, most user requests hit at least one slow backend call. SLOs are written on percentiles.

**5. What happens when you type a URL and press Enter?**
> [!success]- Answer
> DNS lookup (cached at browser/OS/resolver) → TCP handshake → TLS handshake → HTTP request → CDN/LB → app server → cache/DB → response → browser parses HTML, fetches assets (CDN), renders. → [[How-an-App-Works]]

**6. TCP vs UDP?**
> [!success]- Answer
> TCP: connection, ordered, reliable, congestion control (APIs, DBs). UDP: connectionless, best effort, low overhead (video calls, DNS, games, QUIC). → [[Network-Protocols]]

## Scaling

**7. Vertical vs horizontal scaling?**
> [!success]- Answer
> Bigger machine vs more machines. Vertical is simple but capped and a SPOF; horizontal is near-unlimited and redundant but needs stateless services, LBs and distributed data. → [[Scaling-Servers-Stateless-and-SPOF]]

**8. What is a stateless service and why does it matter?**
> [!success]- Answer
> It keeps no per-client state in memory between requests (sessions in Redis/JWT, files in S3), so any instance can serve any request — enabling autoscaling, easy deploys and failover.

**9. What does a load balancer do? L4 vs L7?**
> [!success]- Answer
> Spreads traffic across healthy backends (capacity + availability). L4 routes by IP/port; L7 understands HTTP and routes by path/host/headers, terminates TLS. → [[Load-Balancing]]

**10. What is a SPOF and how do you remove it?**
> [!success]- Answer
> A component whose failure takes the system down. Remove via redundancy across AZs + health checks + automatic failover (LB pairs, DB replicas, multi-AZ).

## Data

**11. SQL vs NoSQL — when each?**
> [!success]- Answer
> SQL for relationships, transactions, ad-hoc queries (orders, payments). NoSQL for massive scale with known access patterns or flexible schemas (KV sessions, wide-column chat, documents for catalogues). → [[SQL-vs-NoSQL]]

**12. ACID?**
> [!success]- Answer
> Atomicity (all or nothing), Consistency (constraints hold), Isolation (concurrent txns don't interfere), Durability (committed survives crashes).

**13. What is an index and its cost?**
> [!success]- Answer
> A data structure (usually B-tree) to find rows without scanning; O(log N) lookups. Costs extra storage and slows writes (every index updated). → [[Indexing]]

**14. Replication vs sharding?**
> [!success]- Answer
> Replication = copies of the same data (availability, read scaling). Sharding = splitting different data across machines (write + storage scaling). Usually both. → [[Replication]] · [[Sharding-and-Partitioning]]

**15. What is consistent hashing?**
> [!success]- Answer
> Mapping keys and nodes onto a ring so adding/removing a node moves only ~1/N of keys; vnodes even out load. → [[Consistent-Hashing]]

**16. What is replication lag and a problem it causes?**
> [!success]- Answer
> Delay before followers apply leader writes. Causes read-your-writes violations (user doesn't see their own update). → [[Consistency-Models]]

## Performance

**17. What is caching and the cache-aside pattern?**
> [!success]- Answer
> Keep hot data in fast storage. Cache-aside: read cache → on miss read DB and fill cache; on write update DB and delete the cache key. → [[Caching]]

**18. LRU vs LFU?**
> [!success]- Answer
> LRU evicts least recently used (recency); LFU evicts least frequently used (popularity). LRU is the default; LFU suits stable hot sets.

**19. What is a CDN?**
> [!success]- Answer
> Geographically distributed edge caches serving content near users — cuts latency, offloads origin, absorbs DDoS. → [[CDN]]

**20. Why store files in object storage rather than a DB?**
> [!success]- Answer
> Cheaper, massively durable, CDN-friendly, keeps the DB small; clients upload/download directly via presigned URLs. → [[Object-Storage]]

## Communication

**21. REST principles?**
> [!success]- Answer
> Resources as nouns, HTTP methods as verbs, stateless requests, standard status codes, cacheable GETs. → [[API-Design]]

**22. What is idempotency and why does it matter?**
> [!success]- Answer
> Repeating an operation has the same effect as doing it once. Needed because retries are inevitable — prevents double charges/orders. → [[Idempotency]]

**23. WebSocket vs SSE vs long polling?**
> [!success]- Answer
> WebSocket: persistent bi-directional (chat, games). SSE: server→client stream over HTTP with auto-reconnect (feeds, LLM tokens). Long polling: held HTTP request — fallback. → [[Real-Time-Communication]]

**24. Why use a message queue?**
> [!success]- Answer
> Decoupling, absorbing spikes, async work, fan-out, retries/failure isolation. → [[Message-Queues]]

**25. Queue vs pub/sub?**
> [!success]- Answer
> Queue: each message to one consumer (work distribution). Pub/sub: each message to every subscriber (event broadcast).

**26. What is Kafka in one line?**
> [!success]- Answer
> A distributed, partitioned, replicated append-only log where consumers track their own offsets, enabling replay and many independent consumers. → [[Kafka-Deep-Dive]]

**27. Monolith vs microservices?**
> [!success]- Answer
> Monolith: one deployable, simple, ACID joins. Microservices: independent deploy/scale per team, but network failures, distributed data and ops cost. Split for organisational scale. → [[Microservices-vs-Monolith]]

**28. What does an API gateway do?**
> [!success]- Answer
> Single entry point: routing, authentication, rate limiting, transformation, aggregation, observability. → [[Proxies-and-API-Gateways]]

## Distributed systems & production

**29. CAP theorem?**
> [!success]- Answer
> During a network partition a distributed store must choose consistency (reject/err) or availability (serve possibly stale). Partition tolerance is mandatory. → [[CAP-and-PACELC]]

**30. Strong vs eventual consistency?**
> [!success]- Answer
> Strong: every read sees the latest write. Eventual: replicas converge if writes stop; reads may be stale meanwhile.

**31. What is a saga?**
> [!success]- Answer
> A sequence of local transactions across services with compensating actions on failure — replaces distributed 2PC. → [[Distributed-Transactions]]

**32. Circuit breaker?**
> [!success]- Answer
> Stops calling a failing dependency (open), fails fast/falls back, then probes (half-open) before resuming. → [[Resilience-Patterns]]

**33. SLA vs SLO vs SLI?**
> [!success]- Answer
> SLI: measured indicator; SLO: internal target; SLA: external contract with penalties. → [[Observability-and-SLOs]]

**34. Authentication vs authorization? JWT?**
> [!success]- Answer
> Who you are vs what you can do. JWT = signed token carrying claims, verified statelessly; hard to revoke early. → [[Authentication-and-Authorization]]

**35. RPO vs RTO?**
> [!success]- Answer
> RPO: max data loss tolerated; RTO: max downtime tolerated. → [[Multi-Region-and-Disaster-Recovery]]

**36. What is RAG?**
> [!success]- Answer
> Retrieval-Augmented Generation: retrieve relevant chunks (vector + keyword search) and add them to the LLM prompt for grounded, cited, fresh answers. → [[RAG-and-Vector-Databases]]

---
**Next:** [[Tricky-Questions]] · **Up:** [[Interview-Questions-MOC]]
