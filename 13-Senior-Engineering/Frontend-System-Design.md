---
title: Frontend System Design
module: 13 · Senior Engineering
lecture: — (added)
level: Intermediate
priority: ⭐ Should
status: Not Started
prerequisites: [How-an-App-Works, CDN, API-Design, Real-Time-Communication]
related: [REST-vs-gRPC-vs-GraphQL, Caching, Real-Time-Collaboration-CRDTs, Design-News-Feed]
tags: [senior, frontend, react, nextjs, rendering, performance, micro-frontends, full-stack]
---

# Frontend System Design ➕ (for full-stack engineers)

> [!info] Why this is here
> Many companies (Flipkart, Atlassian, Microsoft, Uber, Meta, and most product startups) run a **frontend/client system design round** for UI and full-stack roles. With a React/Next.js stack, this is a round I can ace while others prepare only backend HLD.

## What is it?

Designing the **client side** of a product at scale: rendering strategy, data fetching and caching, state management, performance, offline support, real-time updates, component architecture and how frontend teams scale.

## Interview definition

> "Frontend system design covers how the UI gets data and renders it fast and reliably for many users and devices: the rendering strategy, the API contract, client caching and state, performance budgets, accessibility, and how the codebase scales across teams."

## The framework (adapted for frontend)

```mermaid
flowchart LR
    R["Requirements<br/>features, devices, network, SEO, a11y"] --> A["Architecture<br/>rendering + component tree"]
    A --> D["Data model + API<br/>shape, pagination, real-time"]
    D --> S["State + caching<br/>server vs client state"]
    S --> P["Performance<br/>Core Web Vitals budgets"]
    P --> O["Ops<br/>monitoring, flags, rollout"]
```

## Rendering strategies

| Strategy | How | Good for | Cost |
|---|---|---|---|
| **CSR** | JS renders in browser | Dashboards, logged-in apps | Slow first paint, weak SEO |
| **SSR** | Server renders HTML per request | SEO + personalised pages | Server cost, TTFB |
| **SSG** | HTML built at deploy | Docs, marketing | Rebuild on change |
| **ISR** | SSG + background re-generation | Product/catalogue pages | Staleness window |
| **Streaming SSR + React Server Components** | Server streams HTML + server components; less JS shipped | Data-heavy pages in Next.js App Router | Mental model complexity |
| **Islands / partial hydration** | Static HTML, interactive islands | Content sites (Astro) | Less suited to app-like UIs |
| **Edge rendering** | SSR at CDN edge | Global low-latency personalisation | Runtime limits → [[Edge-Wasm-and-eBPF]] |

## Core Web Vitals (performance budget)

| Metric | Measures | "Good" threshold |
|---|---|---|
| **LCP** (Largest Contentful Paint) | Loading | ≤ 2.5 s |
| **INP** (Interaction to Next Paint) | Responsiveness (replaced FID in 2024) | ≤ 200 ms |
| **CLS** (Cumulative Layout Shift) | Visual stability | ≤ 0.1 |

Levers: code splitting, lazy loading, image optimisation (AVIF/WebP, `srcset`), CDN caching, prefetching, fewer client JS bytes (RSC), virtualised long lists, web workers for heavy compute, avoiding layout shift with reserved dimensions.

## State management — the key distinction

| Kind | Examples | Tooling |
|---|---|---|
| **Server state** (cache of backend data) | Feed, orders, profile | TanStack Query, SWR, Apollo/Relay, RSC fetch cache |
| **Client/UI state** | Modal open, form drafts, theme | `useState`, Zustand, Redux Toolkit, Jotai |
| **URL state** | Filters, pagination | Search params (shareable) |
| **Persistent offline state** | Drafts, offline queue | IndexedDB, service worker |

Most "Redux everywhere" pain comes from treating server state as client state.

## 📊 Visual model

**Rendering strategy map:**

```mermaid
quadrantChart
    title Choosing a rendering strategy
    x-axis Static content --> Highly personalised
    y-axis SEO not important --> SEO critical
    quadrant-1 SSR or streaming RSC
    quadrant-2 SSG or ISR
    quadrant-3 CSR is fine
    quadrant-4 CSR with skeletons
    Marketing site: [0.1, 0.9]
    Product listing page: [0.35, 0.85]
    Product detail page: [0.45, 0.9]
    Personalised home feed: [0.85, 0.6]
    Admin dashboard: [0.6, 0.1]
    Chat app: [0.9, 0.15]
```

**Request timeline for a streamed Next.js page:**

```mermaid
sequenceDiagram
    participant B as Browser
    participant E as CDN edge
    participant S as Next.js server (RSC)
    participant API as Backend APIs
    B->>E: GET /product/42
    E->>S: miss (ISR stale) → render
    S-->>B: stream shell HTML + header (fast LCP)
    S->>API: fetch price, stock, reviews in parallel
    API-->>S: price + stock
    S-->>B: stream product section (Suspense boundary)
    API-->>S: reviews (slower)
    S-->>B: stream reviews section
    B->>B: hydrate interactive islands (add to cart)
```

**Feed page component and data architecture:**

```mermaid
flowchart TB
    APP["App shell (layout, auth)"] --> FEED["Feed page"]
    FEED --> VL["Virtualised list<br/>(only visible items in DOM)"]
    VL --> CARD["Post card"]
    CARD --> MEDIA["Lazy image / video"]
    FEED --> QC["Query cache<br/>(cursor pagination, stale-while-revalidate)"]
    QC --> API["GET /feed?cursor"]
    FEED --> WS["Real-time channel (SSE/WS)<br/>'3 new posts' banner"]
    FEED --> OPT["Optimistic like<br/>rollback on error"]
```

**Performance budget per page (illustrative):**

```mermaid
%%{init: {"themeVariables": {"xyChart": {"plotColorPalette": "#4E79A7, #E15759"}}}}%%
xychart-beta
    title "JS shipped per route (KB gzipped) vs budget"
    x-axis ["Home", "Search", "Product", "Cart", "Checkout"]
    y-axis "KB" 0 --> 400
    bar [180, 240, 310, 150, 120]
    line [200, 200, 200, 200, 200]
```

## Scaling frontend teams
- **Monorepo** (Turborepo/Nx) with shared design system + lint rules.
- **Micro-frontends** (Module Federation, route-based splits) only when many independent teams own separate areas. Costs: duplicated deps, inconsistent UX, runtime integration.
- **BFF** per client to shape APIs. → [[Proxies-and-API-Gateways]]
- Feature flags + canary for frontend deploys; RUM (real-user monitoring) + error tracking (Sentry).

## Classic frontend design prompts
News feed (infinite scroll), autocomplete/typeahead, image carousel, chat UI, Google Docs-style editor ([[Real-Time-Collaboration-CRDTs]]), e-commerce product page, video player, data table with 100k rows, notifications centre, Kanban board with drag-drop.

## Common mistakes

> [!warning] Traps to avoid
> 1. Treating server data as global client state.
> 2. Shipping huge JS bundles; no budgets.
> 3. Infinite scroll without virtualisation (DOM with 10k nodes).
> 4. Ignoring accessibility (keyboard, ARIA, contrast) and i18n.
> 5. Micro-frontends for a team of 6.

## Interview understanding

> [!question] What the interviewer is really testing
> - *"Design the frontend for an autocomplete"* — Debounce, cancel stale requests (AbortController), cache per prefix, keyboard navigation + ARIA, highlight, race conditions, empty/error states. → [[Design-Typeahead]]
> - *Tricky:* "SSR vs SSG vs ISR?" — Per-request render vs build-time vs build-time with background regeneration.
> - *Tricky:* "What replaced FID?" — INP (March 2024).
> - *Tricky:* "How do you render 100k rows?" — Virtualisation (windowing), pagination or server-side filtering, web workers for sorting.

## Mini quiz
1. Three Core Web Vitals and thresholds?
2. Server state vs client state?
3. When would you choose ISR?
4. What problem does virtualisation solve?
5. One cost of micro-frontends?

> [!success]- Answers (click to reveal)
> 1. LCP ≤ 2.5 s, INP ≤ 200 ms, CLS ≤ 0.1.
> 2. Cached copy of backend data (needs fetching, invalidation) vs UI-only local state.
> 3. Mostly static pages that change occasionally and need SEO (product/catalogue pages).
> 4. Rendering only visible items so huge lists stay fast.
> 5. Duplicated dependencies / inconsistent UX / runtime integration complexity.

---
**Next:** [[Technical-Leadership-for-Engineers]] · **Up:** [[Senior-Engineering-MOC]]

## 📚 Resources
- 📄 [web.dev — Core Web Vitals](https://web.dev/articles/vitals)
- 📄 [Next.js docs — Rendering](https://nextjs.org/docs/app/building-your-application/rendering)
- 📄 [GreatFrontEnd — Front End System Design guide](https://www.greatfrontend.com/front-end-system-design-playbook)
- 📄 [TanStack Query — Does this replace client state?](https://tanstack.com/query/latest/docs/framework/react/guides/does-this-replace-client-state)

## My notes
-
