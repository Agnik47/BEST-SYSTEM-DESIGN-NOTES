---
title: System Design Glossary
topic: Glossary
level: All
status: Reference
tags: [glossary, reference]
related: [System-Design-MOC, Rapid-Fire-Revision]
---

# 📖 System Design Glossary

| Term | Meaning | Note |
|---|---|---|
| **A2A (Agent2Agent)** | Open protocol for agents to delegate tasks to other agents | [[Context-Engineering-and-Agent-Protocols]] |
| **ACID** | Atomicity, Consistency, Isolation, Durability | [[SQL-vs-NoSQL]] |
| **ADR** | Architecture Decision Record: short permanent record of one decision and why | [[Design-Docs-and-ADRs]] |
| **AGENTS.md** | Repo instruction file telling coding agents commands, conventions, boundaries | [[AI-Native-SDLC]] |
| **Aggregate (DDD)** | Cluster of objects changed in one transaction; root guards invariants | [[Domain-Driven-Design]] |
| **Anti-entropy** | Background repair of replica divergence (Merkle trees) | [[Design-Key-Value-Store]] |
| **Anycast** | One IP announced from many locations; BGP routes to nearest | [[CDN]] |
| **API gateway** | Single entry point: routing, auth, rate limits | [[Proxies-and-API-Gateways]] |
| **Availability** | % of time the system serves requests | [[What-is-System-Design]] |
| **Backpressure** | Slowing producers when consumers can't keep up | [[Message-Queues]] |
| **BASE** | Basically Available, Soft state, Eventually consistent | [[SQL-vs-NoSQL]] |
| **Bloom filter** | Probabilistic set membership, no false negatives | [[Probabilistic-Data-Structures]] |
| **Bounded context** | Boundary in which a domain model and its terms are consistent | [[Domain-Driven-Design]] |
| **B-tree** | Balanced tree index; fast reads, in-place updates | [[Indexing]] |
| **Bulkhead** | Isolated resource pools to contain failures | [[Resilience-Patterns]] |
| **Canary release** | Expose a new version to a small % of traffic, compare metrics, then ramp | [[Zero-Downtime-Migrations-and-Deployments]] |
| **CAP** | Choose C or A during a partition | [[CAP-and-PACELC]] |
| **CDC** | Change Data Capture — DB log → events | [[Event-Driven-Architecture]] |
| **CDN** | Edge caches near users | [[CDN]] |
| **Circuit breaker** | Stop calling a failing dependency; fail fast | [[Resilience-Patterns]] |
| **Consistent hashing** | Ring mapping that moves ~1/N keys on change | [[Consistent-Hashing]] |
| **Context engineering** | Curating what goes into a model's context at each step | [[Context-Engineering-and-Agent-Protocols]] |
| **Continuous batching** | LLM serving: refill GPU batch every decode step | [[LLM-Serving-and-Inference]] |
| **Core Web Vitals** | LCP, INP, CLS user-experience metrics | [[Frontend-System-Design]] |
| **Count-Min Sketch** | Approximate frequency counts | [[Probabilistic-Data-Structures]] |
| **CQRS** | Separate write and read models | [[Event-Driven-Architecture]] |
| **CRDT** | Data types that merge without conflicts | [[Real-Time-Collaboration-CRDTs]] |
| **Crypto-shredding** | Deleting an encryption key so all data encrypted with it becomes unreadable | [[Privacy-and-AI-Regulation]] |
| **Data contract** | Producer-owned agreement on a dataset's schema, semantics and SLAs | [[Data-Engineering-for-AI]] |
| **Dead letter queue (DLQ)** | Where repeatedly failing messages go | [[Message-Queues]] |
| **Denormalization** | Duplicating data to speed reads | [[SQL-vs-NoSQL]] |
| **DORA metrics** | Lead time, deploy frequency, change-failure rate, time to restore | [[AI-Native-SDLC]] |
| **DPDP Act** | India's Digital Personal Data Protection Act, 2023 | [[Privacy-and-AI-Regulation]] |
| **Durability** | Committed data is never lost | [[Replication]] |
| **Durable execution** | Workflow state persisted as history so code resumes after crashes | [[Workflow-Orchestration-and-Durable-Execution]] |
| **eBPF** | Verified programs running inside the Linux kernel for networking, observability, security | [[Edge-Wasm-and-eBPF]] |
| **Embedding** | Vector representing meaning of text/items | [[RAG-and-Vector-Databases]] |
| **Error budget** | 1 − SLO; allowed unreliability | [[Observability-and-SLOs]] |
| **Eval (LLM)** | Dataset + scoring that measures LLM feature quality | [[LLM-Evals-and-AI-Observability]] |
| **Event sourcing** | Store events as source of truth | [[Event-Driven-Architecture]] |
| **Eventual consistency** | Replicas converge if writes stop | [[Consistency-Models]] |
| **Expand–contract** | Additive change → dual write → backfill → switch → remove old | [[Zero-Downtime-Migrations-and-Deployments]] |
| **Fan-out** | One event delivered to many recipients | [[Design-News-Feed]] |
| **Feature flag** | Runtime switch that decouples deploy from release | [[Zero-Downtime-Migrations-and-Deployments]] |
| **Fencing token** | Monotonic number rejecting stale lock holders | [[Consensus-and-Leader-Election]] |
| **FinOps** | Shared engineering/finance ownership of cloud cost; unit economics | [[Cost-Aware-Architecture]] |
| **Geohash** | Encodes lat/lng into a prefix-sortable string | [[Geospatial-Indexing]] |
| **Golden dataset** | Versioned set of test cases for evaluating an AI system | [[LLM-Evals-and-AI-Observability]] |
| **Golden path** | Opinionated, supported self-service way to build/deploy/run a service | [[Platform-Engineering]] |
| **Gossip protocol** | Peer-to-peer spreading of cluster state | [[Design-Key-Value-Store]] |
| **H3** | Uber's hexagonal hierarchical geo index | [[Geospatial-Indexing]] |
| **Hinted handoff** | Temporary storage of writes for a down replica | [[Replication]] |
| **HLC** | Hybrid logical clock (physical + logical) | [[Clocks-and-Ordering]] |
| **HNSW** | Graph-based approximate nearest-neighbour index | [[RAG-and-Vector-Databases]] |
| **Hot key / hotspot** | One key/partition receiving disproportionate load | [[Cache-Pitfalls]] |
| **HyperLogLog** | Approximate distinct count in tiny memory | [[Probabilistic-Data-Structures]] |
| **Iceberg (Apache)** | Open table format giving ACID tables on object storage | [[Data-Engineering-for-AI]] |
| **Idempotency** | Repeating an operation has the same effect as once | [[Idempotency]] |
| **Inverted index** | Term → documents mapping | [[Search-Systems]] |
| **ISR** | Kafka in-sync replicas | [[Kafka-Deep-Dive]] |
| **JWT** | Signed token carrying claims | [[Authentication-and-Authorization]] |
| **KV cache** | Stored attention keys/values during LLM decoding | [[LLM-Serving-and-Inference]] |
| **Lamport clock** | Logical counter consistent with causality | [[Clocks-and-Ordering]] |
| **Latency** | Time for one request | [[Estimation-and-Latency-Numbers]] |
| **Leader election** | Agreeing on one coordinator | [[Consensus-and-Leader-Election]] |
| **Linearizability** | Strongest single-object consistency | [[Consistency-Models]] |
| **Little's Law** | L = λW | [[Estimation-and-Latency-Numbers]] |
| **LLM-as-judge** | Using a model with a rubric to grade another model's output | [[LLM-Evals-and-AI-Observability]] |
| **Load shedding** | Rejecting excess work early | [[Resilience-Patterns]] |
| **LSM tree** | Log-structured merge tree; write-optimised storage | [[Indexing]] |
| **MCP (Model Context Protocol)** | Open protocol connecting AI apps to tools, resources and prompts | [[Context-Engineering-and-Agent-Protocols]] |
| **Model routing / cascade** | Send requests to the cheapest model that meets the quality bar; escalate when needed | [[AI-Cost-and-Small-Models]] |
| **Mutation testing** | Inject code changes to check whether tests catch them | [[Verification-Engineering]] |
| **MVCC** | Multi-version concurrency control | [[Transactions-and-Isolation-Levels]] |
| **OLAP / OLTP** | Analytics vs transactional workloads | [[Choosing-a-Database]] |
| **Outbox pattern** | Write event to DB table in same txn, relay later | [[Event-Driven-Architecture]] |
| **PACELC** | Partition: A/C; Else: L/C | [[CAP-and-PACELC]] |
| **Partition (Kafka)** | Ordered log shard; unit of parallelism | [[Kafka-Deep-Dive]] |
| **Percentile (p99)** | 99% of requests faster than this | [[Estimation-and-Latency-Numbers]] |
| **Post-quantum cryptography (PQC)** | Algorithms resistant to quantum attacks (ML-KEM, ML-DSA, SLH-DSA) | [[Supply-Chain-Security-and-Post-Quantum]] |
| **Presigned URL** | Expiring signed URL for direct object access | [[Object-Storage]] |
| **Prompt injection** | Untrusted text in context that hijacks a model's behaviour | [[AI-Security-Engineering]] |
| **Property-based testing** | Testing invariants over many generated inputs | [[Verification-Engineering]] |
| **Quorum** | Minimum replicas for read/write success (W + R > N) | [[Replication]] |
| **Raft** | Understandable consensus algorithm | [[Consensus-and-Leader-Election]] |
| **RAG** | Retrieval-Augmented Generation | [[RAG-and-Vector-Databases]] |
| **Rate limiting** | Capping requests per client per window | [[Design-Rate-Limiter]] |
| **Read replica** | Follower serving reads | [[Replication]] |
| **Reconciliation** | Comparing records across systems to fix mismatches | [[Design-Payment-System]] |
| **Reverse proxy** | Server-side intermediary (nginx) | [[Proxies-and-API-Gateways]] |
| **Row-level security (RLS)** | DB-enforced filtering of rows per tenant/user | [[Multi-Tenancy-and-SaaS-Architecture]] |
| **RPO / RTO** | Max data loss / max downtime | [[Multi-Region-and-Disaster-Recovery]] |
| **Saga** | Local transactions + compensations | [[Distributed-Transactions]] |
| **SBOM** | Software Bill of Materials: list of components in a build | [[Supply-Chain-Security-and-Post-Quantum]] |
| **Service mesh** | Sidecar proxies handling service networking | [[Proxies-and-API-Gateways]] |
| **Sharding** | Splitting data across machines | [[Sharding-and-Partitioning]] |
| **Shuffle sharding** | Give each tenant a random small subset of workers to limit blast radius | [[Multi-Tenancy-and-SaaS-Architecture]] |
| **SLI / SLO / SLA** | Indicator / objective / agreement | [[Observability-and-SLOs]] |
| **SLSA** | Framework of levels for build integrity and provenance | [[Supply-Chain-Security-and-Post-Quantum]] |
| **Snowflake ID** | 64-bit time-sortable distributed ID | [[Unique-ID-Generation]] |
| **Split brain** | Two nodes acting as leader | [[Consensus-and-Leader-Election]] |
| **SPOF** | Single point of failure | [[Scaling-Servers-Stateless-and-SPOF]] |
| **SSE** | Server-Sent Events | [[Real-Time-Communication]] |
| **Stateless service** | Keeps no per-client state between requests | [[Scaling-Servers-Stateless-and-SPOF]] |
| **Strangler fig** | Incrementally route functionality from a legacy system to a new one | [[Large-Scale-Migrations]] |
| **Throughput** | Requests handled per second | [[Estimation-and-Latency-Numbers]] |
| **Thundering herd** | Many requests hit backend at once (e.g., on cache expiry) | [[Cache-Pitfalls]] |
| **Token bucket** | Rate-limit algorithm allowing bursts | [[Design-Rate-Limiter]] |
| **TTFT / TPOT** | Time to first token / per output token | [[LLM-Serving-and-Inference]] |
| **TTL** | Time-to-live before expiry | [[Caching]] |
| **Vector clock** | Per-node counters detecting concurrency | [[Clocks-and-Ordering]] |
| **Virtual nodes** | Many ring positions per physical node | [[Consistent-Hashing]] |
| **Watermark** | Stream progress marker for event time | [[Batch-and-Stream-Processing]] |
| **WebAssembly (Wasm)** | Portable sandboxed binary format for browser, server and edge | [[Edge-Wasm-and-eBPF]] |
| **WebSocket** | Persistent bi-directional connection | [[Real-Time-Communication]] |
| **Write skew** | Isolation anomaly breaking cross-row invariants | [[Transactions-and-Isolation-Levels]] |

---
**Up:** [[System-Design-MOC]]
