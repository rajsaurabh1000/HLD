# HLD — Restaurant Recommendation System Based on User Location

<a id="interview-spine-nine-steps"></a>

## 1. Clarify requirements

### 1.0 Live flow

<a id="live-flow-open"></a>

**Opening (~once):** *“I’ll treat this as **ranking under latency**: **geo eligibility**, **personalization**, **cold start**, **experiments**; then **features**, **serving**, **architecture**. **Pause after the diagram**—**features**, **fairness**, or **infra**?”*

**Thinking transitions:** *“Same **read funnel** as homepage—**candidates** capped before **model**.”* (See [11-hld-uber-eats-homepage.md](./11-hld-uber-eats-homepage.md).)

<a id="say-1-questions-human"></a>
### 1.1 Clarify

| Topic | Say it like this in the room |
|--------------------------|-------------------------------|
| **Surface** | “**Home rail**, **search zero-query**, **email**—which?” |
| **Objective** | “**GMV**, **CTR**, **diversity**?” |
| **Privacy** | “**Precise** lat vs **neighborhood** features?” |
| **Sponsored** | “**Mixer** slot?” |

### 1.2 Functional requirements (FR)

<a id="say-fr-human"></a>

| FR area | Say it like this |
|---------|-------------------|
| **Candidates** | “From **geo index**, pull **eligible** restaurants in **radius/cells**.” |
| **Features** | “User history, **context** (time, weather), **distance**, **popularity**, **merchant** quality.” |
| **Rank** | “Score + **mixer** + **diversity** constraints.” |
| **Log** | “**Impression/click** for **training** and **fairness** audits.” |

### 1.3 Non-functional requirements (NFR)

| NFR | Say it like this |
|-----|------------------|
| **Latency** | “**Time-box** rank; **fallback** to **distance + popularity**.” |
| **Freshness** | “**Feature** lag **seconds–minutes** acceptable for many signals.” |

### 1.4 Invariants

**Invariant:** “**Recommended** restaurants are always **hard-eligible** to serve the user’s **delivery context**; **rank order** may degrade.”

<a id="say-voice-1"></a>

| Beat | Say it like this |
|------|------------------|
| **Bridge** | “**Two-tower** or **GBDT**—interviewer picks depth; **serving** shape same.” |
| **Core split** | “**Offline training** vs **online feature log** vs **real-time** retrieval.” |

<a id="key-insight-say-early"></a>
### Key insight (say early)

**Location** enters as **both hard filter** and **soft feature**—never let model **override** **ineligibility**.

#### Key anchors

1. “**Cap K**.”  
2. “**Feature store** with **point-in-time** correctness story.”  
3. “**Exploration** slots.”

---

## 2. Estimate scale

<a id="say-voice-2"></a>

| Dimension | Illustrative |
|-----------|----------------|
| Rank QPS | Same order as **homepage** reads |
| Feature cardinality | **High**—**sparse** embeddings |

---

## 3. APIs and data model

<a id="say-voice-3"></a>

### 3.1 APIs

| API | Purpose |
|-----|---------|
| `GET /v1/reco/restaurants?lat=&lng=&user_id=` | Ranked list |
| `POST /v1/reco/events` | Impression/click (async) |

### 3.2 Features (examples)

- **Geo:** distance, cell id, **traffic** band.  
- **User:** **order history**, dietary, **LTV**.  
- **Item:** rating, **prep time**, **popularity**, **new** flag.

---

## 4. High-level architecture

<a id="say-voice-4"></a>

```mermaid
flowchart LR
  GW[Gateway]
  REC[Reco svc]
  FS[Feature store]
  GEO[Geo candidates]
  MX[Mixer]
  LOG[Kafka]
  GW --> REC
  REC --> GEO
  REC --> FS
  REC --> MX
  REC --> LOG
```

### 4.1 Phases

| Phase | Ship |
|-------|------|
| **1** | Heuristic rank |
| **2** | Logged **features** + **batch** model |
| **3** | **Two-tower** + **online** learning |

---

## 5. Deep dive: rank request

<a id="say-voice-5"></a>

<a id="bottleneck-anchor-once"></a>
### 🎯 Bottleneck Anchor

“**Feature fetch fan-out** (K × features) and **model p99**—**batch** embeddings, **deadline** + **fallback**.”

```mermaid
sequenceDiagram
  participant REC
  participant GEO
  participant FS as Feature store
  participant M as Model
  REC->>GEO: candidates (capped K)
  REC->>FS: batch user + item features
  FS-->>REC: tensors
  REC->>M: score under deadline
  REC-->>Client: ranked ids + why-id optional
```

---

## 6. Scaling and bottlenecks

| Risk | Mitigation |
|------|------------|
| **Hot user** | **Cache** user embedding |
| **Model OOM** | **Bulkhead** + **fallback** ranker |

---

## 7. Reliability and failure handling

- **Feature store miss:** **default** features; **down-rank** unknowns.  
- **Model timeout:** **heuristic** sort.

---

## 8. Tradeoffs and alternatives

| Choice | Trade |
|--------|--------|
| **Heavy personalization** | Engagement vs **filter bubble** |
| **Two-tower** | Scale vs **freshness** of **cross** terms |

---

## 9. Monitoring, observability, and security

**Metrics:** **p99** rank latency, **fallback** rate, **new merchant** impression share, **CTR** slice by **distance decile**.  
**Security:** **No** cross-user **feature** leaks; **PII** in **feature store** encrypted.

---

## 10. Design patterns, data structures & best practices

| Pattern | Map |
|---------|-----|
| **Mixer** | Sponsored + organic |
| **Bandits** | Exploration |
| **Cache-aside** | Hot features |

<a id="say-voice-10"></a>
**Live:** max **four** patterns.

---

## Closing notes

<a id="communication-do-vs-avoid"></a>

| Do | Avoid |
|----|--------|
| **Eligibility before model** | “ML fixes bad zone” |
| **Time-box** | 10 min on **embedding dim** |

---

## Bar-raiser follow-ups

| They ask | Say it like this |
|----------|------------------|
| **Counterfactual** | “**Logging policy** bias—**IPS** corrections / **randomized** buckets.” |

---

## 60-second close

| Beat | Say it like this |
|------|------------------|
| **Recap** | “**Geo cap K** → **features** → **model under deadline** → **mixer**; **eligibility** hard; **offline/online** split; **obs** on **tail** + **fairness**.” |

---
