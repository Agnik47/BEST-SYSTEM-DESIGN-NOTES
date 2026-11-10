---
title: Design a Distributed Job Scheduler
module: 10 · Case Studies
lecture: — (added, high frequency)
level: Advanced
priority: ⭐ Should
status: Not Started
prerequisites: [Message-Queues, Consensus-and-Leader-Election, Idempotency]
related: [Design-Notification-System, Resilience-Patterns, AI-Agent-Systems]
tags: [case-study, scheduler, cron, delayed-jobs, leader-election, at-least-once]
---

# ⏰ Design a Distributed Job Scheduler

> [!abstract] Problem
> Let services schedule jobs: one-off at a time ("send reminder at 9 am"), recurring (cron: "every night at 2 am"), or delayed ("retry in 5 min"). Jobs must run **on time** and **at least once** (ideally effectively once), at scale.

## 1. Clarifying questions

- One-off + cron + delayed? → *all three*
- Scale? → *100M scheduled jobs/day, peaks at round times (9:00:00 sharp!)*
- Precision? → *within ~1–2 seconds*
- Job execution: we run code or call webhooks/queues? → *we dispatch to queues/webhooks; workers belong to teams*

## 2. Requirements

| Functional | Non-functional |
|---|---|
| Create / cancel / update jobs | Fire within 2 s of scheduled time |
| One-off, cron, delayed | No job lost; at-least-once execution |
| Retries with backoff, DLQ | No double-firing in normal operation (dedupe) |
| History & status | Horizontally scalable, HA scheduler (no single leader bottleneck) |

## 3. Estimation

```text
100M executions/day ≈ 1.2k/s avg, but massive spikes at :00 (e.g., 2M jobs at 09:00:00)
Job record ~500 B → 100M/day × 30 days retention = 1.5 TB → partitioned table
Spike handling: spread the 2M over a few seconds via dispatch fleet → ~500k/s dispatch capacity or jitter
```

```mermaid
%%{init: {"themeVariables": {"xyChart": {"plotColorPalette": "#4E79A7, #E15759, #59A14F, #F28E2B, #B07AA1"}}}}%%
xychart-beta
    title "Jobs due per minute around 9 AM (illustrative, thousands)"
    x-axis ["08:57", "08:58", "08:59", "09:00", "09:01", "09:02", "09:03"]
    y-axis "k jobs" 0 --> 2100
    bar [20, 25, 40, 2000, 60, 30, 25]
```

## 4. API

```http
POST /v1/jobs  { "type": "webhook" | "queue", "target": "...", "payload": {...},
                 "runAt": "2026-11-07T09:00:00+05:30" | "cron": "0 2 * * *",
                 "idempotencyKey": "...", "retry": {"max": 5, "backoff": "exp"} } → {jobId}
DELETE /v1/jobs/{id}
GET    /v1/jobs/{id}/executions
```

## 5. Data model

```mermaid
erDiagram
    JOB ||--o{ EXECUTION : "runs as"
    JOB {
        bigint job_id PK
        string type
        string target
        string payload
        string cron_expr
        bigint next_run_bucket "epoch minute"
        timestamp next_run_at
        string status "ACTIVE PAUSED DELETED"
        string idempotency_key UK
    }
    EXECUTION {
        bigint job_id PK
        timestamp scheduled_for PK
        string state "PENDING DISPATCHED RUNNING SUCCEEDED FAILED"
        int attempt
        string worker_id
        timestamp lease_until
    }
```

Partition `JOB` by `next_run_bucket` (time bucket) + shard → scanning "due in the next minute" touches only one bucket.

## 6. High-level design

```mermaid
flowchart LR
    C["Client services"] --> API["Scheduler API"] --> DB[("Jobs DB<br/>partitioned by time bucket + shard")]
    subgraph Sched["Scheduler fleet (sharded, leader per shard)"]
      S1["Poller shard 0"] & S2["Poller shard 1"] & S3["Poller shard N"]
    end
    LE[("etcd / ZooKeeper<br/>shard ownership leases")] -.-> S1 & S2 & S3
    S1 & S2 & S3 -->|"due in next 60s → in-memory timing wheel"| TW["Timing wheel / delay heap"]
    TW -->|"at fire time"| Q[["Kafka / SQS dispatch queues<br/>(per priority / tenant)"]]
    Q --> W["Team workers / webhook dispatcher"]
    W -->|"ack / fail"| RES["Result handler"] --> DB
    RES -->|"retry with backoff"| DB
    RES --> DLQ[["DLQ"]]
```

## 7. Key flows

```mermaid
sequenceDiagram
    participant P as Poller (owns shard 3)
    participant DB as Jobs DB
    participant TW as Timing wheel
    participant Q as Dispatch queue
    participant W as Worker
    loop every 10 s
      P->>DB: SELECT jobs WHERE shard=3 AND next_run_at < now+60s AND not claimed
      P->>DB: claim (UPDATE … SET claimed_by=me, lease=+90s WHERE claimed_by IS NULL)
      P->>TW: schedule in memory
    end
    TW->>Q: fire job 77 (execution id = 77@09:00)
    Q->>W: deliver
    W->>W: dedupe by execution id, do work
    W-->>P: success
    P->>DB: mark SUCCEEDED, compute next_run_at (cron) → new bucket
```

**Execution state machine:**

```mermaid
stateDiagram-v2
    [*] --> SCHEDULED
    SCHEDULED --> CLAIMED: poller leases it
    CLAIMED --> DISPATCHED: fire time reached
    CLAIMED --> SCHEDULED: lease expired (poller died)
    DISPATCHED --> RUNNING: worker picked up
    RUNNING --> SUCCEEDED
    RUNNING --> RETRY_WAIT: failure (attempt < max)
    RETRY_WAIT --> DISPATCHED: backoff elapsed
    RUNNING --> DEAD: attempts exhausted → DLQ
    SUCCEEDED --> [*]
    DEAD --> [*]
```

## 8. Deep dives

### Avoiding double-firing
Shard ownership via leases in etcd/ZooKeeper (one poller per shard) + DB claim with lease + **fencing token** → even after a paused poller wakes up, its stale claim is rejected. Execution ID `(job_id, scheduled_for)` makes workers idempotent. → [[Consensus-and-Leader-Election]] · [[Idempotency]]

### Precision without hammering the DB
Two-level design: coarse DB scan for "due within the next minute" + an in-memory **timing wheel / min-heap** for second-level precision. Redis ZSET (`score = runAt`) with `ZRANGEBYSCORE` is a common simpler alternative for delayed jobs.

### The :00 thundering herd
Encourage jitter for non-critical jobs (`runAt ± random(0, 30s)`), pre-load buckets early, autoscale dispatchers before known peaks, and per-tenant rate limits so one team's 2M jobs don't delay others' critical ones. → [[Resilience-Patterns]]

### Cron semantics
After each run compute `next_run_at` from the cron expression in the job's **time zone** (DST!); define misfire policy (run once now vs skip) if the system was down.

### Long-running jobs
Heartbeat to extend the lease; if heartbeats stop, the job is retried elsewhere — so job logic must be idempotent or checkpointed. For multi-step workflows → durable workflow engines (Temporal). → [[AI-Agent-Systems]]

## 9. Bottlenecks & trade-offs

| Choice | Trade-off |
|---|---|
| DB polling vs Redis ZSET vs Kafka delay topics | Durability/queryability vs simplicity/speed |
| Single leader vs sharded leaders | Simplicity vs throughput & blast radius |
| At-least-once | Simple & safe vs duplicates (needs idempotent workers) |
| Lease length | Fast failover vs false re-dispatch of slow jobs |

## 10. Follow-ups & tricky questions

- *"Exactly-once?"* — Not end to end; at-least-once + idempotent execution IDs.
- *"Cancel a job that's already dispatched?"* — Best effort: mark cancelled; workers check status before side effects.
- *"Priorities?"* — Separate dispatch queues & worker pools (bulkheads).
- *"How does Kubernetes CronJob / Airflow compare?"* — K8s CronJob: per-cluster, controller-based; Airflow: DAG orchestration for data pipelines, not millions of tiny per-user jobs.

## 11. What each level should cover

| SDE-1 | SDE-2 | Senior |
|---|---|---|
| Jobs table + poller + queue + workers + retries | Time-bucket partitioning, leases, idempotent executions, timing wheel, DLQ | Sharded leadership with fencing, :00 spikes, multi-tenant fairness, cron/DST semantics |

## 🧠 Concepts used
[[Message-Queues]] · [[Consensus-and-Leader-Election]] · [[Idempotency]] · [[Resilience-Patterns]] · [[Redis-Deep-Dive]] · [[Sharding-and-Partitioning]]

## My gaps
-

---
**Next:** [[Design-Leaderboard-Top-K]] · **Up:** [[Case-Studies-MOC]]
