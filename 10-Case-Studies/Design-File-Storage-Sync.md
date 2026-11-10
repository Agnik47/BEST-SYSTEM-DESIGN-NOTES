---
title: Design File Storage and Sync (Google Drive / Dropbox)
module: 10 · Case Studies
lecture: Practice (L12, L14, L17)
level: Advanced
priority: ⭐ Should
status: Not Started
prerequisites: [Object-Storage, Message-Queues, Consistency-Models]
related: [Real-Time-Collaboration-CRDTs, Design-Notification-System, Probabilistic-Data-Structures]
tags: [case-study, file-sync, chunking, dedup, conflicts, object-storage]
---

# 📁 Design File Storage and Sync

> [!abstract] Problem
> Users upload files from any device; files sync automatically to their other devices; they can share files/folders and see version history. Large files, flaky networks, offline edits.

## 1. Clarifying questions

- Max file size? → *up to 50 GB*
- Sync clients (desktop/mobile) + web? → *yes*
- Sharing & permissions? Version history? → *yes, 30 days of versions*
- Real-time co-editing? → *no — that's Google Docs (see CRDT note); here it's file-level sync*

## 2. Requirements

| Functional | Non-functional |
|---|---|
| Upload/download files and folders | Durability 11 nines; never lose a committed file |
| Automatic sync across devices | Sync latency: changes visible on other devices in seconds |
| Share with users/links, permissions | Efficient: only changed parts re-uploaded (delta sync) |
| Versions & restore; trash | Works offline; handles conflicts |

## 3. Estimation

```text
500M users, 100M DAU; avg 10 GB stored/user (paid skew) → 5 EB total logical; dedup saves 30–50%
Uploads: 100M DAU × 2 files × 5 MB ≈ 1 PB/day ingest
Metadata ops (list, sync polls, change notifications): ~1M/s peak → metadata DB is the hot path
Chunk size 4 MB → a 1 GB file = 256 chunks
```

```mermaid
pie showData
    title Storage before vs after chunk dedup (illustrative)
    "Unique chunks stored" : 60
    "Saved by dedup" : 40
```

## 4. API

```http
POST /v1/files/upload-session   {path, size, chunkHashes[]} → {sessionId, missingChunks[], presignedUrls[]}
PUT  <presigned S3 url>          (upload only the missing chunks)
POST /v1/files/commit            {sessionId, chunkHashes[], baseVersion} → {fileId, version}  (409 on conflict)
GET  /v1/changes?cursor=…        → {changes:[{fileId, version, op}], nextCursor}   (long-poll / push hint)
GET  /v1/files/{id}/download     → manifest of chunk URLs
POST /v1/shares                  {fileId, principal, role}
```

## 5. Data model

```mermaid
erDiagram
    USER ||--o{ FILE : owns
    FILE ||--o{ FILE_VERSION : "has versions"
    FILE_VERSION ||--|{ VERSION_CHUNK : "made of"
    CHUNK ||--o{ VERSION_CHUNK : "referenced by"
    FILE ||--o{ SHARE : "shared via"
    FILE {
        bigint file_id PK
        bigint owner_id
        bigint parent_folder_id
        string name
        bigint latest_version
        boolean is_deleted
    }
    FILE_VERSION {
        bigint file_id PK
        bigint version PK
        bigint size
        string device_id
        timestamp created_at
    }
    VERSION_CHUNK {
        bigint file_id PK
        bigint version PK
        int idx PK
        string chunk_hash FK
    }
    CHUNK {
        string chunk_hash PK "SHA-256"
        string s3_key
        int ref_count
    }
    SHARE {
        bigint file_id PK
        bigint principal_id PK
        string role "viewer editor"
    }
    USER {
        bigint user_id PK
        bigint quota_bytes
    }
```

Metadata → **sharded SQL** (by owner/namespace; strong consistency for versions). Chunks → **S3**, content-addressed by hash. Change journal per namespace for sync cursors.

## 6. High-level design

```mermaid
flowchart LR
    D1["Desktop client<br/>watcher + local DB + chunker"] --> API["Metadata API"]
    D1 -->|"presigned chunk PUT/GET"| S3[("Object storage<br/>content-addressed chunks")]
    API --> MDB[("Metadata DB (sharded)<br/>files, versions, chunks")]
    API --> J[("Change journal per namespace")]
    J -->|"events"| K[["Kafka"]]
    K --> NS["Notification service<br/>(WebSocket / long-poll hint)"]
    NS --> D2["Phone client"] & D3["Laptop 2"]
    D2 & D3 -->|"GET /changes?cursor"| API
    K --> IDX["Search / thumbnails / virus scan workers"]
    CDN["CDN for shared-link downloads"] --> S3
```

## 7. Key flows

**Delta upload with dedup:**

```mermaid
sequenceDiagram
    participant C as Client
    participant A as Metadata API
    participant S as S3
    participant N as Notifier
    C->>C: split file into 4 MB chunks, SHA-256 each
    C->>A: upload-session (chunk hashes, baseVersion=7)
    A->>A: which hashes are new? (lookup CHUNK table)
    A-->>C: missing = [c3, c9] + presigned URLs
    C->>S: PUT c3, c9 (parallel, resumable)
    C->>A: commit(version 8, base 7)
    A->>A: txn: insert FILE_VERSION 8 + VERSION_CHUNK rows, bump latest_version (if still 7)
    A-->>C: ok version 8
    A-)N: change event (namespace, file, v8)
    N-)N: notify other devices → they call GET /changes
```

**Sync conflict state:**

```mermaid
stateDiagram-v2
    [*] --> InSync
    InSync --> LocalEdit: user edits offline
    LocalEdit --> Uploading: online
    Uploading --> InSync: commit ok (base == latest)
    Uploading --> Conflict: 409 (someone committed v8 first)
    Conflict --> InSync: save as "report (Agnik's conflicted copy).docx" + download v8
    InSync --> RemoteChange: change notification
    RemoteChange --> Downloading
    Downloading --> InSync
```

## 8. Deep dives

### Chunking & dedup
Fixed-size chunks are simple; **content-defined chunking** (rolling hash, e.g., Rabin) keeps boundaries stable when bytes are inserted early in a file → far better delta sync. Content-addressed storage (key = hash) gives cross-user dedup for free; track reference counts for garbage collection (careful: delete only when ref_count = 0 and after a grace period). → [[Object-Storage]]

### Sync protocol
Client keeps a local DB of `(path → version, chunk hashes)` + a **cursor** into the server change journal. Server pushes a lightweight "something changed" hint (WebSocket/long-poll); client pulls `GET /changes?cursor`. Cursor = monotonic journal sequence per namespace. → [[Real-Time-Communication]]

### Conflicts
Optimistic concurrency on `baseVersion`. On conflict, don't merge binaries — keep both (conflicted copy). Text co-editing is a different product → [[Real-Time-Collaboration-CRDTs]].

### Metadata scale
Metadata QPS ≫ chunk QPS. Shard by namespace (user or shared folder); keep a file's versions in one shard; cache folder listings. Dropbox moved metadata to its own sharded MySQL ("Edgestore") for this reason.

### Durability & cost
S3 cross-AZ (11 nines), optional cross-region replication; cold tier for old versions; erasure coding at the storage layer (Dropbox built "Magic Pocket").

## 9. Bottlenecks & trade-offs

| Choice | Trade-off |
|---|---|
| Chunk size small vs large | Better dedup/delta vs more metadata rows & requests |
| Cross-user dedup | Storage savings vs side-channel privacy risk ("does this file exist?") — dedup per user or use convergent encryption carefully |
| Push vs pull sync | Fast hints vs connection cost; hybrid push-hint + pull-changes |
| Strong metadata consistency | Correct versions vs cross-shard moves are complex |

## 10. Follow-ups & tricky questions

- *"Moving a folder with 1M files?"* — Metadata-only operation (update parent pointer); no chunk movement; journal one move event.
- *"Shared folder edited by 50 users?"* — Namespace per shared folder; journal per namespace; all members subscribe.
- *"Upload 50 GB on mobile?"* — Resumable multipart, background upload, Wi-Fi-only option.
- *2026:* "Notion-style docs + AI Q&A over the drive" → CRDT editing + RAG with ACL-filtered retrieval → [[RAG-and-Vector-Databases]].

## 11. What each level should cover

| SDE-1 | SDE-2 | Senior |
|---|---|---|
| Metadata vs blob split, presigned upload, versions | Chunking + dedup, delta sync, change journal + cursors, conflict handling | Content-defined chunking, GC of chunks, metadata sharding, privacy of dedup, cost tiers |

## 🧠 Concepts used
[[Object-Storage]] · [[Message-Queues]] · [[Consistency-Models]] · [[Transactions-and-Isolation-Levels]] · [[Sharding-and-Partitioning]] · [[Real-Time-Communication]] · [[Real-Time-Collaboration-CRDTs]]

## My gaps
-

---
**Next:** [[Design-Key-Value-Store]] · **Up:** [[Case-Studies-MOC]]
