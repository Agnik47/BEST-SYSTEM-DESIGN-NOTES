---
title: Multi-Tenancy and SaaS Architecture
module: 13 · Senior Engineering
lecture: — (added)
level: Advanced
priority: ⭐ Should
status: Not Started
prerequisites: [Sharding-and-Partitioning, Authentication-and-Authorization, Design-Rate-Limiter]
related: [Cost-Aware-Architecture, Multi-Region-and-Disaster-Recovery, Resilience-Patterns, Privacy-and-AI-Regulation]
tags: [senior, multi-tenancy, saas, isolation, noisy-neighbor, shuffle-sharding]
---

# Multi-Tenancy and SaaS Architecture

## What is it?

One system serving many **tenants** (customer organisations), with each tenant's data and performance isolated from the others. Slack workspaces, Freshworks accounts, Razorpay merchants, Zoho orgs.

## Why it matters at SDE-2/3

Almost every B2B product and every internal platform is multi-tenant. The hard questions are senior-level: **how isolated**, **who pays for what**, **what happens when one big tenant goes wild**, and **can we delete or move one tenant's data** (compliance).

## Analogy

An apartment building. **Pool:** everyone shares the same floors and pipes (cheap, but a leak can affect neighbours). **Silo:** each tenant gets a separate house (expensive, fully isolated). **Bridge:** shared building, but big tenants get their own floor.

## Interview definition

> "Multi-tenancy trades isolation for cost efficiency. You pick a model (silo, pool, or hybrid) per layer, enforce tenant context everywhere (auth → queries → caches → queues), protect against noisy neighbours with per-tenant quotas and isolation, and keep per-tenant observability and cost attribution."

## Tenancy models

| Model | Data | Compute | Isolation | Cost / tenant | Fits |
|---|---|---|---|---|---|
| **Silo** | DB per tenant | Stack per tenant | Strongest | Highest | Enterprise, regulated (banks, govt) |
| **Pool** | Shared tables with `tenant_id` | Shared | Weakest (logical) | Lowest | SMB/self-serve, long tail |
| **Bridge / hybrid** | Schema per tenant, or pool + silo for whales | Shared + dedicated tiers | Medium | Medium | Most real SaaS at scale |
| **Cell-based** | Tenants assigned to cells (mini-stacks) | Per cell | Blast radius = one cell | Medium | Large SaaS (Slack, AWS services) |

## Enforcing isolation in a pooled DB

- **Tenant context** from the auth token, propagated through every call (header + context object); never from user input.
- Every query scoped by `tenant_id`; composite indexes start with `tenant_id`.
- **Postgres Row-Level Security (RLS):** DB enforces `tenant_id = current_setting('app.tenant_id')` as defence in depth.
- Cache keys, queue messages, object-storage prefixes, search indexes **all** include tenant ID.
- Per-tenant encryption keys (BYOK) for enterprise tiers.

```sql
ALTER TABLE invoices ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON invoices
  USING (tenant_id = current_setting('app.tenant_id')::bigint);
-- app sets per request:  SET app.tenant_id = '42';
```

## Noisy neighbours

One tenant's import job or traffic spike degrades everyone.
- **Per-tenant rate limits and quotas** (API, storage, jobs). → [[Design-Rate-Limiter]]
- **Fair scheduling**: per-tenant queues or weighted fair queuing for background work.
- **Bulkheads**: separate pools for big tenants. → [[Resilience-Patterns]]
- **Shuffle sharding**: give each tenant a random small subset of N workers; two tenants rarely share the same whole subset, so one bad tenant can't take everyone down.
- Move whales to dedicated shards/cells (tenant migration tooling!).

## 📊 Visual model

**Three models side by side:**

```mermaid
flowchart TB
    subgraph Silo["Silo"]
      TA["Tenant A"] --> SA["App A"] --> DA[("DB A")]
      TB["Tenant B"] --> SB["App B"] --> DB2[("DB B")]
    end
    subgraph Pool["Pool"]
      T1["Tenants 1..10,000"] --> SP["Shared app"] --> DP[("Shared DB<br/>tenant_id + RLS")]
    end
    subgraph Bridge["Bridge"]
      TS["Small tenants"] --> SH["Shared app"] --> DS[("Pooled DB")]
      TW["Whale tenant"] --> SH
      SH --> DW[("Dedicated DB for whale")]
    end
```

**Isolation vs cost:**

```mermaid
quadrantChart
    title Tenancy models
    x-axis Low cost per tenant --> High cost per tenant
    y-axis Weak isolation --> Strong isolation
    quadrant-1 Enterprise silo
    quadrant-2 Ideal but rare
    quadrant-3 Self-serve pool
    quadrant-4 Worst of both
    Pool with tenant_id: [0.12, 0.25]
    Pool with RLS: [0.18, 0.42]
    Schema per tenant: [0.45, 0.58]
    Cells: [0.55, 0.72]
    DB per tenant: [0.78, 0.85]
    Full stack per tenant: [0.95, 0.97]
```

**Shuffle sharding — blast radius:**

```mermaid
block-beta
  columns 8
  w1["W1"] w2["W2"] w3["W3"] w4["W4"] w5["W5"] w6["W6"] w7["W7"] w8["W8"]
  ta["Tenant A → W1, W4"]:4 tb["Tenant B → W2, W7"]:4
  tc["Tenant C → W1, W6"]:4 td["Tenant D → W3, W8"]:4
  note["If Tenant A poisons W1 and W4, C still has W6; B and D unaffected"]:8
```

$$P(\text{two tenants share the exact same shard of size } k \text{ out of } n) = \frac{1}{\binom{n}{k}}$$

> [!example] Numbers
> n = 8 workers, k = 2 per tenant → 28 combinations. n = 100, k = 5 → ~75 million combinations. That's why AWS uses shuffle sharding for Route 53.

**Tenant onboarding as a workflow:**

```mermaid
sequenceDiagram
    participant S as Signup
    participant CP as Control plane
    participant DP as Data plane
    participant B as Billing
    S->>CP: create tenant (plan = Pro)
    CP->>CP: allocate cell / shard, keys, quotas
    CP->>DP: provision schema / config
    CP->>B: create subscription + metering
    CP-->>S: tenant ready (admin user, domain)
```

## Control plane vs data plane

- **Control plane:** tenant lifecycle, plans, quotas, routing tables, provisioning. Low traffic, high correctness.
- **Data plane:** serves tenant traffic. High volume, must keep working even if the control plane is down (cache the routing config).

## Common mistakes

> [!warning] Traps to avoid
> 1. Tenant ID taken from a request parameter instead of the authenticated token.
> 2. Forgetting tenant scoping in caches/search/queues (cross-tenant data leaks).
> 3. No per-tenant metrics → can't find the noisy neighbour or price the plan.
> 4. No tooling to move a tenant between shards/regions.
> 5. One giant tenant in a pooled DB with no limits.

## Interview understanding

> [!question] What the interviewer is really testing
> - *"Design a multi-tenant SaaS"* — Choose a model per tier, tenant context propagation, RLS, quotas, noisy-neighbour defence, per-tenant observability and cost, data residency.
> - *Tricky:* "How do you delete all data of one tenant (GDPR/DPDP)?" — Tenant-scoped keys/prefixes everywhere, per-tenant encryption keys (crypto-shredding), deletion workflow across stores + backups policy.
> - *Tricky:* "What's shuffle sharding?" — Random subsets of workers per tenant; limits blast radius combinatorially.
> - *Tricky:* "Control plane down — what breaks?" — New signups/changes; data plane should keep serving from cached config.

## Mini quiz
1. Silo vs pool in one line?
2. What is RLS and why use it?
3. Three noisy-neighbour defences?
4. Control plane vs data plane?
5. What is crypto-shredding?

> [!success]- Answers (click to reveal)
> 1. Dedicated resources per tenant (isolated, costly) vs shared resources with tenant_id (cheap, logical isolation).
> 2. Postgres row-level security: the DB enforces tenant filtering as defence in depth.
> 3. Per-tenant rate limits/quotas, fair queues/bulkheads, shuffle sharding, moving whales to dedicated shards.
> 4. Management/provisioning layer vs the layer serving tenant traffic.
> 5. Deleting a tenant's encryption key, making all their encrypted data (incl. backups) unreadable.

---
**Next:** [[Capacity-Planning-and-Performance-Engineering]] · **Up:** [[Senior-Engineering-MOC]]

## 📚 Resources
- 📄 [AWS — SaaS Tenant Isolation Strategies (whitepaper)](https://docs.aws.amazon.com/whitepapers/latest/saas-tenant-isolation-strategies/saas-tenant-isolation-strategies.html)
- 📄 [AWS Builders' Library — Workload isolation using shuffle-sharding](https://aws.amazon.com/builders-library/workload-isolation-using-shuffle-sharding/)
- 📄 [PostgreSQL — Row Security Policies](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)

## My notes
-
