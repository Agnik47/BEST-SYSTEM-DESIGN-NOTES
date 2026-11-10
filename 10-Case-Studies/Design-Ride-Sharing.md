---
title: Design a Ride-Sharing App (Uber / Ola / Rapido)
module: 10 · Case Studies
lecture: L29
level: Advanced
priority: 🔥 Must
status: Not Started
prerequisites: [Geospatial-Indexing, Real-Time-Communication, Distributed-Transactions]
related: [Design-Food-Delivery, Design-Payment-System, Batch-and-Stream-Processing]
tags: [case-study, geospatial, matching, location-updates, surge]
---

# 🚕 Design a Ride-Sharing App

> [!abstract] Problem
> Riders request a ride; the system finds a nearby available driver, the driver accepts, both see live locations, trip completes, payment happens. Handle surge pricing during demand spikes.

## 1. Clarifying questions

- Scope: request, match, track, pay? → *yes; pricing basic + surge*
- Scale? → *India: 1M active drivers at peak, 10M rides/day*
- Location update frequency? → *drivers every 4 s*
- Pooling/scheduled rides? → *out of scope*

## 2. Requirements

| Functional | Non-functional |
|---|---|
| Driver goes online, sends location | Match within ~10 s |
| Rider requests, sees ETA & fare | Location writes: very high throughput |
| Match driver (accept/decline) | A driver must never get 2 trips at once (consistency for assignment) |
| Live tracking both ways | Tracking freshness ~seconds |
| Trip state machine, payment, rating | High availability; degrade gracefully |

## 3. Estimation

```text
Location writes: 1M drivers / 4 s = 250k writes/s  (in-memory store, not a DB row update)
Ride requests: 10M/day ≈ 115/s avg, ~1k/s peak (rush hour, rain)
Each match query: geo search ~100–300 candidates → ETA ranking
Location history: 250k/s × 100 B ≈ 2 TB/day → Cassandra / lake, sampled
```

```mermaid
pie showData
    title Write traffic mix at peak
    "Driver location pings" : 250000
    "Trip state updates" : 3000
    "Ride requests" : 1000
```

## 4. API

```http
POST /v1/drivers/me/status        {online: true}
WS   driver channel: → {lat, lng, heading, ts} every 4 s;  ← offers
POST /v1/rides                     {pickup, drop, product} Idempotency-Key → {rideId, status: MATCHING}
POST /v1/rides/{id}/accept         (driver)
WS   rider channel: ← driverLocation, eta, status
POST /v1/rides/{id}/complete
```

## 5. Data model

```mermaid
erDiagram
    RIDER ||--o{ RIDE : requests
    DRIVER ||--o{ RIDE : drives
    RIDE ||--o| PAYMENT : "settled by"
    RIDE {
        bigint ride_id PK
        bigint rider_id
        bigint driver_id
        string status
        string pickup_geohash
        int fare_paise
        float surge_multiplier
        timestamp requested_at
    }
    DRIVER {
        bigint driver_id PK
        string vehicle_type
        string city
        float rating
    }
    RIDER {
        bigint rider_id PK
        string phone
    }
    PAYMENT {
        bigint payment_id PK
        bigint ride_id FK
        string status
    }
```

Live driver location & availability → **Redis GEO / in-memory geo index per city**; trips → Postgres (sharded by city/ride_id); location trail → Cassandra.

## 6. High-level design

```mermaid
flowchart LR
    DA["Driver app"] <-->|"WebSocket"| GWD["Driver gateway"]
    RA["Rider app"] <-->|"WebSocket / HTTPS"| GWR["Rider gateway"]
    GWD -->|"pings"| LOC["Location service"]
    LOC --> GEO[("Geo index (Redis GEO / H3 in-memory)<br/>sharded by city")]
    LOC -.->|"events (async)"| K[["Kafka"]]
    K --> HIST[("Cassandra: trails")]
    K --> SURGE["Surge engine (Flink)<br/>supply vs demand per H3 cell"]
    GWR --> RIDE["Ride service<br/>(trip state machine)"] --> PG[("Postgres: rides")]
    RIDE --> MATCH["Matching service"]
    MATCH --> GEO
    MATCH --> ETA["ETA / routing service<br/>(road graph + traffic)"]
    MATCH -->|"offer"| GWD
    SURGE --> PRICE["Pricing service"] --> RIDE
    RIDE --> PAY["Payment service"]
    RIDE -->|"driver location relay"| GWR
```

## 7. Key flows

**Request → match:**

```mermaid
sequenceDiagram
    participant R as Rider
    participant RS as Ride svc
    participant M as Matcher
    participant G as Geo index
    participant E as ETA svc
    participant D as Driver
    R->>RS: request ride (pickup, drop)
    RS->>RS: ride=MATCHING, quote fare × surge
    RS->>M: find driver
    M->>G: GEOSEARCH pickup radius 2km, available
    G-->>M: 180 candidates
    M->>E: ETA for top 20 by distance
    E-->>M: ranked by ETA
    M->>M: lock best driver (SET driver:lock NX PX 15000)
    M->>D: offer (15 s to accept)
    alt accepts
      D-->>M: accept
      M->>RS: assign driver (conditional update: driver not on trip)
      RS-->>R: driver assigned, ETA 4 min
    else declines / timeout
      M->>M: release lock, next candidate (expand radius)
    end
```

**Trip state machine:**

```mermaid
stateDiagram-v2
    [*] --> REQUESTED
    REQUESTED --> MATCHING
    MATCHING --> DRIVER_ASSIGNED: driver accepts
    MATCHING --> NO_DRIVERS: timeout after expanding radius
    DRIVER_ASSIGNED --> ARRIVED
    DRIVER_ASSIGNED --> CANCELLED: rider/driver cancels (fee policy)
    ARRIVED --> IN_PROGRESS: OTP verified
    IN_PROGRESS --> COMPLETED
    COMPLETED --> PAID: payment captured
    PAID --> [*]
    NO_DRIVERS --> [*]
    CANCELLED --> [*]
```

## 8. Deep dives

### Location updates at 250k/s
Don't write to a relational DB. Gateway → location service → **in-memory geo index** (Redis GEO per city or custom H3/quadtree service), TTL so stale drivers vanish; async Kafka for history/analytics. Shard by city (Bengaluru alone is big → sub-shard by H3 cell). → [[Geospatial-Indexing]]

### Matching
Candidates by geo radius → filter (available, vehicle type, rating) → **rank by ETA** (road network, not straight-line). Prevent double assignment with a short **driver lock** + conditional DB update (`UPDATE drivers SET current_ride = :r WHERE id = :d AND current_ride IS NULL`). Batch matching (collect requests for 1–2 s and solve assignment globally) improves overall ETAs at scale.

### Surge pricing
Stream supply (available drivers) and demand (requests, app opens) per **H3 hexagon** in 1-min windows → multiplier → cached per cell; quote locked at request time. → [[Batch-and-Stream-Processing]]

### Live tracking to rider
Driver pings → location service → publish to the rider's gateway via pub/sub (only for active trips). Client-side interpolation smooths the car icon between updates.

### Payments
Authorise at request/complete, capture after trip; idempotent; wallet/UPI/card; driver payouts batched. → [[Design-Payment-System]]

## 9. Bottlenecks & trade-offs

| Risk | Mitigation |
|---|---|
| Hot cells (airport, stadium exit) | Finer sub-sharding, queue-based pickup zones |
| Location service failure | Drivers re-send; short TTL; stale-but-usable data for a few seconds |
| ETA service latency | Cache ETAs between cell pairs; approximate first, refine later |
| Double assignment | Lock + conditional update + idempotent accept |
| Network flakiness in drivers' phones | Idempotent APIs, offline buffering of pings |

## 10. Follow-ups & tricky questions

- *"How do you compute ETA?"* — Road graph (contraction hierarchies), live traffic speeds per segment, ML correction; cache.
- *"Why not query PostGIS for nearby drivers?"* — 250k updates/s on indexed rows would crush it; in-memory index fits the access pattern.
- *"Region-level outage?"* — Cities are natural cells — failover per city/region; trips in progress continue with cached state on devices.
- *"Pool rides?"* — Matching becomes a routing optimisation (insertion heuristics) with detour constraints.

## 11. What each level should cover

| SDE-1 | SDE-2 | Senior |
|---|---|---|
| Location updates + geo search + trip states | 250k/s numbers, in-memory geo sharding, driver locking, ETA ranking, tracking relay | Batch matching, surge streaming with H3, city cells, failure/degradation strategy |

## 🧠 Concepts used
[[Geospatial-Indexing]] · [[Real-Time-Communication]] · [[Redis-Deep-Dive]] · [[Kafka-Deep-Dive]] · [[Batch-and-Stream-Processing]] · [[Distributed-Transactions]] · [[Idempotency]] · [[Multi-Region-and-Disaster-Recovery]]

## My gaps
-

---
**Next:** [[Design-Payment-System]] · **Up:** [[Case-Studies-MOC]]
