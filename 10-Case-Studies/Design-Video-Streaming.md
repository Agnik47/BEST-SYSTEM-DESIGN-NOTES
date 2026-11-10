---
title: Design a Video Streaming Platform (YouTube / Netflix / Hotstar)
module: 10 · Case Studies
lecture: L27
level: Intermediate
priority: 🔥 Must
status: Not Started
prerequisites: [Object-Storage, CDN, Message-Queues]
related: [Batch-and-Stream-Processing, Recommendation-and-Ranking-Systems, Multi-Region-and-Disaster-Recovery]
tags: [case-study, video, transcoding, abr, cdn, live-streaming]
---

# 🎬 Design a Video Streaming Platform

> [!abstract] Problem
> Users upload videos; others stream them smoothly on any device and network. Handle viral spikes and **live events** (think IPL final on Hotstar — tens of millions concurrent).

## 1. Clarifying questions

- VOD, live, or both? → *VOD first, then live spikes*
- Upload volume vs views? → *views ≫ uploads*
- Devices & networks? → *mobile on 3G/4G to 4K TVs*
- Comments/recommendations? → *out of scope (mention)*

## 2. Requirements

| Functional | Non-functional |
|---|---|
| Upload video (large, resumable) | Start-up time < 2 s; rebuffering < 1% |
| Transcode to many resolutions | Massive read bandwidth, global |
| Stream with adaptive quality | Upload processing in minutes |
| View counts | Highly available playback; cost-efficient storage |

## 3. Estimation

```text
Uploads: 500 hrs of video/min (YouTube-like) — for us: 100k uploads/day × 300 MB raw = 30 TB/day raw
Transcoded renditions ≈ 1.5–2× raw → ~50 TB/day stored → object storage + lifecycle tiers
Views: 1B/day; avg watch 10 min @ 3 Mbps ≈ 225 MB/view → 225 PB/day egress → 95%+ must be CDN
Live peak: 25M concurrent × 3 Mbps ≈ 75 Tbps → multi-CDN + ISP caches
```

```mermaid
pie showData
    title Where playback bytes are served (target)
    "CDN edge / ISP caches" : 96
    "Regional shield" : 3
    "Origin (S3)" : 1
```

## 4. API

```http
POST /v1/uploads            { filename, size } → { uploadId, partUrls[] (presigned) }
POST /v1/uploads/{id}/complete
GET  /v1/videos/{id}        → { title, status, manifestUrl: ".../master.m3u8" }
GET  <CDN>/videos/{id}/720p/seg_00042.ts   (client fetches segments directly)
```

## 5. Data model

```mermaid
erDiagram
    VIDEO ||--o{ RENDITION : "has"
    VIDEO ||--o{ VIEW_AGG : "counted in"
    VIDEO {
        bigint video_id PK
        bigint owner_id
        string title
        string status "UPLOADING PROCESSING READY FAILED"
        string raw_key "s3 key"
        int duration_s
    }
    RENDITION {
        bigint video_id PK
        string profile PK "1080p_h264, 720p_av1 ..."
        string manifest_key
        int bitrate_kbps
    }
    VIEW_AGG {
        bigint video_id PK
        date day PK
        bigint views
    }
```

## 6. High-level design

```mermaid
flowchart LR
    subgraph Upload["Upload & processing"]
      C1["Creator"] -->|"multipart, presigned"| RAW[("S3 raw bucket")]
      RAW -->|"ObjectCreated"| Q[["Queue"]]
      Q --> SPLIT["Splitter: chunk into GOP-aligned segments"]
      SPLIT --> TQ[["Transcode jobs (DAG)"]]
      TQ --> W1["Transcoder fleet (spot/GPU)"] & W2["Transcoder"] & W3["Transcoder"]
      W1 & W2 & W3 --> OUT[("S3 renditions<br/>HLS/DASH segments + manifests")]
      OUT --> META[("Video metadata DB: READY")]
    end
    subgraph Play["Playback"]
      V["Viewer app"] -->|"GET manifest"| API["Video API"] --> META
      V -->|"segments"| CDN["Multi-CDN edges"] --> SH["Shield"] --> OUT
      V -->|"heartbeats, QoE"| K[["Kafka"]] --> AN["View counts, QoE analytics"]
    end
```

## 7. Key flows

**Processing DAG for one video:**

```mermaid
flowchart LR
    I["Raw upload"] --> VAL["Validate + virus scan"] --> SP["Split into 4s chunks"]
    SP --> T1["Encode 240p"] & T2["Encode 480p"] & T3["Encode 720p"] & T4["Encode 1080p"] & T5["Encode 4K (if source)"]
    SP --> AU["Audio tracks"]
    SP --> TH["Thumbnails"]
    T1 & T2 & T3 & T4 & T5 & AU --> PK["Package HLS/DASH + DRM"] --> MF["Write master manifest"] --> RD["status=READY, notify creator"]
```

**Adaptive bitrate playback:**

```mermaid
sequenceDiagram
    participant P as Player
    participant CDN as CDN edge
    P->>CDN: GET master.m3u8
    CDN-->>P: renditions: 240p/480p/720p/1080p
    P->>CDN: GET 720p/seg_1.ts
    Note over P: measured bandwidth drops (metro tunnel)
    P->>CDN: GET 480p/seg_2.ts
    Note over P: buffer healthy, bandwidth up
    P->>CDN: GET 1080p/seg_3.ts
```

**ABR ladder (typical bitrates):**

```mermaid
%%{init: {"themeVariables": {"xyChart": {"plotColorPalette": "#4E79A7, #E15759, #59A14F, #F28E2B, #B07AA1"}}}}%%
xychart-beta
    title "Rendition ladder — bitrate by resolution (Mbps, H.264 typical)"
    x-axis ["240p", "360p", "480p", "720p", "1080p", "4K"]
    y-axis "Mbps" 0 --> 18
    bar [0.4, 0.8, 1.4, 3, 5, 16]
```

## 8. Deep dives

### Transcoding at scale
Split into segments → **parallel** encode (minutes instead of hours) → workers are stateless, autoscaled on queue depth, ideally on spot instances (retry on preemption: idempotent jobs keyed by `(video, profile, segment)`). Per-title/content-aware encoding saves bandwidth. Codecs: H.264 (compat), H.265/VP9/**AV1** (30–50% smaller, costlier to encode). → [[Message-Queues]] · [[Design-Job-Scheduler]]

### Streaming protocols
**HLS / MPEG-DASH**: video split into 2–6 s segments over plain HTTP → cacheable by any CDN; manifest lists renditions; client switches quality per segment (ABR).

### CDN strategy
Popular content pre-positioned (push) to edges/ISP caches (Netflix Open Connect); long tail pulled on demand. Multi-CDN with steering by QoE. Signed URLs/tokens + DRM (Widevine/FairPlay) for paid content. → [[CDN]]

### Live streaming spikes (IPL final)
Ingest (RTMP/SRT) → live transcoder → short segments (2 s; LL-HLS for low latency) → CDN. All viewers request the **same latest segment** → perfect for caching but brutal on origin at segment boundaries → origin shield + request collapsing. Pre-scale everything ahead of known events; degrade gracefully (cap max bitrate for everyone if CDN capacity is tight). Hotstar-style "scale for the toss" planning.

### View counting
Client heartbeats → Kafka → stream aggregation (dedupe per session) → counts; approximate in real time, exact via batch. → [[Batch-and-Stream-Processing]]

## 9. Bottlenecks & trade-offs

| Choice | Trade-off |
|---|---|
| More renditions | Better QoE vs storage + encode cost |
| AV1 | Bandwidth savings vs encode compute & device support |
| Segment length 2 s vs 6 s | Lower latency/faster switching vs more requests & overhead |
| Push vs pull CDN | Predictable hits vs storage at edges |
| Storage tiers | Cold old videos → infrequent-access/archive; re-transcode on demand for rarely viewed renditions |

## 10. Follow-ups & tricky questions

- *"Resume a 5 GB upload on a flaky network?"* — Multipart with per-part retry & checksum. → [[Object-Storage]]
- *"Video in processing but user wants to share it?"* — Show status; publish lowest rendition first, add higher ones as they finish.
- *"Reduce start-up time?"* — Start at low bitrate, preload first segment, CDN edge proximity, smaller first segment.
- *2026:* auto-generated captions/chapters with speech models in the DAG; recommendations → [[Recommendation-and-Ranking-Systems]].

## 11. What each level should cover

| SDE-1 | SDE-2 | Senior |
|---|---|---|
| Upload to S3, transcode queue, CDN playback | Segmented parallel transcoding, HLS/ABR, presigned multipart, view counting | Live event capacity planning, multi-CDN steering, codec/cost trade-offs, DRM |

## 🧠 Concepts used
[[Object-Storage]] · [[CDN]] · [[Message-Queues]] · [[Batch-and-Stream-Processing]] · [[Multi-Region-and-Disaster-Recovery]] · [[Idempotency]]

## My gaps
-

---
**Next:** [[Design-Ticket-Booking]] · **Up:** [[Case-Studies-MOC]]
