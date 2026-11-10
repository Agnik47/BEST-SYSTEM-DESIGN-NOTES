---
title: Production MOC
topic: Production
module: 07
level: Intermediate
status: Not Started
tags: [moc, production, reliability, security]
related: [System-Design-MOC, Distributed-Systems-MOC, Modern-Topics-MOC]
---

# 🏭 07 · Running Systems in Production MOC

| # | Note | Lecture | You'll be able to |
|---|---|---|---|
| 1 | [[Resilience-Patterns]] | L19 | Timeouts, retries + jitter, circuit breakers, bulkheads, load shedding |
| 2 | [[Multi-Region-and-Disaster-Recovery]] | L19 | Active-active vs active-passive, RPO/RTO, cell architecture |
| 3 | [[Observability-and-SLOs]] | L19 | Logs/metrics/traces, SLA/SLO/SLI, error budgets |
| 4 | [[Authentication-and-Authorization]] | L20 | Sessions vs tokens, JWT, OAuth 2.0/OIDC, SSO, RBAC |
| 5 | [[Security-Essentials]] | L20 | TLS, encryption at rest, hashing, DDoS, secrets, least privilege |
| 6 | [[Search-Systems]] | L21 | Inverted index, Elasticsearch, relevance, autocomplete |
| 7 | [[Batch-and-Stream-Processing]] | L21 | MapReduce/Spark/Flink, lake vs warehouse, ETL/ELT, Lambda/Kappa |

> [!tip] The "production" signal in interviews
> After your design is drawn, volunteer 60 seconds on: **what fails first, how you'd know (metrics/alerts), and how it degrades gracefully.** This is the cheapest way to sound senior. My DevOps vault (Observability, Reliability) has the ops side of these.

**Next module:** [[Modern-Topics-MOC]] · **Up:** [[System-Design-MOC]]
