---
title: REST vs gRPC vs GraphQL
module: 05 · Communication
lecture: L13
level: Intermediate
priority: ⭐ Should
status: Not Started
prerequisites: [API-Design, Network-Protocols]
related: [Microservices-vs-Monolith, Proxies-and-API-Gateways, Real-Time-Communication]
tags: [communication, rest, grpc, graphql, api-styles]
---

# REST vs gRPC vs GraphQL

## The one-line version

- **REST:** resources over HTTP + JSON. Universal, cacheable, simple. **Default for public APIs.**
- **gRPC:** strongly-typed RPC over HTTP/2 with Protocol Buffers. Fast, streaming, codegen. **Default for internal service-to-service.**
- **GraphQL:** one endpoint, client specifies exactly the fields it needs. **Great for aggregating many backends for varied UIs (BFF).**

## Comparison

| | REST | gRPC | GraphQL |
|---|---|---|---|
| Transport | HTTP/1.1 or 2 | **HTTP/2** | HTTP (usually POST) |
| Payload | JSON (text) | **Protobuf (binary)** — smaller, faster to parse | JSON |
| Contract | OpenAPI (optional) | `.proto` (required, codegen) | Schema (required) |
| Streaming | No (use SSE/WS) | **Uni + bi-directional streaming** | Subscriptions (via WS) |
| Browser support | Native | Needs gRPC-Web proxy | Native |
| HTTP caching | **Easy (GET + URLs)** | Hard | Hard (POST, one URL) — needs persisted queries |
| Over/under-fetching | Common | Defined per method | **Solved** — ask for exact fields |
| Learning/tooling | Easiest | Moderate | Moderate; server complexity (N+1, query cost) |
| Best for | Public APIs, CRUD, web | Internal microservices, low latency, polyglot, mobile↔backend at scale | Frontends with many screens/devices, aggregating services |

## Over-fetching & under-fetching (why GraphQL exists)

REST mobile home screen might call `/user`, `/user/orders`, `/recommendations`, `/notifications` (under-fetching → 4 round trips) and get 50 fields when it needs 5 (over-fetching).

```graphql
query {
  me { name avatarUrl
    orders(first: 3) { id status total }
    notifications(unread: true) { count }
  }
}
```

One round trip, exact fields. Cost: the server must guard against expensive queries (depth/complexity limits) and the **N+1 problem** (use DataLoader batching).

## gRPC example

```proto
service RideService {
  rpc GetEta (EtaRequest) returns (EtaResponse);
  rpc StreamDriverLocation (DriverId) returns (stream Location);   // server streaming
}
message EtaRequest { string ride_id = 1; }
```

Deadlines/timeouts and cancellation propagate across calls — great for microservice chains.

## Architecture: who uses what

```mermaid
flowchart LR
    Web["Web / mobile"] -->|"REST or GraphQL"| GW["API Gateway / BFF"]
    Partner["Third-party devs"] -->|"REST + webhooks"| GW
    GW -->|"gRPC"| S1["Orders svc"]
    GW -->|"gRPC"| S2["Users svc"]
    S1 -->|"gRPC"| S3["Payments svc"]
```

Typical real-world split: **REST outside, gRPC inside, GraphQL as a BFF** when frontends are diverse (Netflix, Shopify, GitHub offer GraphQL; Google, Netflix, Uber use gRPC internally).

## 📊 Visual model

```mermaid
quadrantChart
    title API styles
    x-axis Simple and universal --> Typed and high performance
    y-axis Server decides shape --> Client decides shape
    quadrant-1 Typed and flexible
    quadrant-2 Client-shaped queries
    quadrant-3 Default web APIs
    quadrant-4 Internal RPC
    REST: [0.2, 0.25]
    gRPC: [0.85, 0.2]
    GraphQL: [0.45, 0.85]
    Webhooks: [0.15, 0.1]
    tRPC: [0.7, 0.45]
```

**One screen, three styles:**

```mermaid
sequenceDiagram
    participant M as Mobile app
    participant R as REST API
    participant G as GraphQL BFF
    Note over M,R: REST — 3 round trips, over-fetching
    M->>R: GET /me
    M->>R: GET /me/orders
    M->>R: GET /me/notifications
    Note over M,G: GraphQL — 1 round trip, exact fields
    M->>G: query { me { name orders(first:3) notifications } }
    G-->>M: exactly those fields
```

## Common mistakes

> [!warning] Traps to avoid
> 1. "GraphQL is faster" — not inherently; it saves round trips, but resolvers can be slower.
> 2. gRPC for a public browser API without considering gRPC-Web/proxy needs.
> 3. GraphQL without query cost limits (DoS by deeply nested query).
> 4. Ignoring caching — REST GETs + CDN are hard to beat for public read traffic.

## Interview understanding

> [!question] What the interviewer is really testing
> - *"Which API style for X?"* — Who's the client (public/partner/internal/browser), need for streaming, caching, payload efficiency.
> - *Tricky:* "How do you load-balance gRPC?" — L7 (request-level) balancing or client-side LB; L4 balances long-lived connections unevenly. → [[Load-Balancing]]
> - *Tricky:* "How do you cache GraphQL?" — Persisted queries via GET + CDN, response caching per field/entity, client-side normalised cache (Apollo).
> - *Tricky:* "Why Protobuf over JSON?" — Smaller binary, faster serialisation, schema evolution with field numbers.

## Mini quiz
1. gRPC runs on which HTTP version?
2. What problem does GraphQL solve?
3. What's the N+1 problem in GraphQL and the fix?
4. Best style for a public partner API?
5. Why is REST easier to cache?

> [!success]- Answers (click to reveal)
> 1. HTTP/2.
> 2. Over- and under-fetching; multiple round trips for varied UIs.
> 3. Resolver per item triggers one DB query each; batch with DataLoader.
> 4. REST (+ webhooks), documented with OpenAPI.
> 5. GET requests with unique URLs map directly onto HTTP/CDN caches.

---
**Next:** [[Real-Time-Communication]] · **Up:** [[Communication-MOC]]

## 📚 Resources
- 📄 [gRPC — Introduction](https://grpc.io/docs/what-is-grpc/introduction/)
- 📄 [GraphQL — Learn](https://graphql.org/learn/)

## My notes
-
