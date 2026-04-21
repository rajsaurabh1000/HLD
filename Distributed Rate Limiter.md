# HLD — Distributed Rate Limiter

<a id="interview-spine-nine-steps"></a>

## 1. Clarify requirements

### 1.0 Live flow

<a id="live-flow-open"></a>

**Opening (~once):** *“I’ll align **algorithm** (token bucket vs sliding window), **scope** (user/IP/route), **sync vs local approx**, and **burst** behavior; then **data plane**, **architecture**. **Pause after the diagram**—**correctness**, **Redis**, or **edge**?”*

**Thinking transitions:** *“Distributed limiter trades **exactness** for **latency**—I’ll state **over-admission** bounds.”*

<a id="say-1-questions-human"></a>
### 1.1 Clarify

| Topic | Say it like this in the room |
|--------------------------|-------------------------------|
| **Fairness** | “**Per tenant** fairness vs **global** cap?” |
| **Burst** | “Allow **burst** of **B**?” |
| **Failure mode** | “**Fail open** vs **closed** under store outage?” |
| **Hierarchy** | “**Global** then **per-route** nested?” |

### 1.2 Functional requirements (FR)

<a id="say-fr-human"></a>

| FR area | Say it like this |
|---------|-------------------|
| **Decide** | “Given **key** + **limit rule**, return **allow** / **deny** + optional **Retry-After**.” |
| **Dynamic** | “Rules from **config** / **API**.” |
| **Obs** | “Emit **metrics** per key prefix (cardinality-safe).” |

### 1.3 Non-functional requirements (NFR)

| NFR | Say it like this |
|-----|------------------|
| **Latency** | “**Microseconds–low ms** on hot path—**local** decision preferred.” |
| **Accuracy** | “**Centralized** ‘exact’ vs **Gossip** approximate—pick and defend.” |

### 1.4 Invariants

**Invariant:** “A **deny** response is **always safe**; an **allow** may be **slightly over** quota under **distributed races**—bounded by design.”

<a id="say-voice-1"></a>

| Beat | Say it like this |
|------|------------------|
| **Bridge** | “**Token bucket** in **Redis** + **local leash**.” |
| **Core split** | “**Data plane** (fast) vs **control plane** (rules).” |

<a id="key-insight-say-early"></a>
### Key insight (say early)

**Hierarchical limits** + **token bucket** at edge with **async central reconciliation** for **abuse**—pure distributed counting is either **slow** or **fuzzy**.

#### Key anchors

1. “**Lua script** atomicity in Redis.”  
2. “**Epoch + counter** sliding window approximation.”  
3. “**Fail closed** on **payments**; **fail open** on **optional** reads if product says so.”

---

## 2. Estimate scale

<a id="say-voice-2"></a>

| Dimension | Illustrative |
|-----------|----------------|
| Checks / sec | **Same as gateway QPS** |
| Key cardinality | **Huge**—**hash** tags, **sample** metrics |

---

## 3. APIs and data model

<a id="say-voice-3"></a>

### 3.1 APIs

| API | Purpose |
|-----|---------|
| `allow(key, cost)` | Internal sync call from gateway |
| `PUT /v1/limits/{tenant}` | Control plane |

### 3.2 Rules model

- **LimitRule:** `key_template`, `rate`, `burst`, `algorithm`, `priority`.

---

## 4. High-level architecture

<a id="say-voice-4"></a>

```mermaid
flowchart TB
  GW[API Gateway]
  EDGE[Envoy / local limiter]
  RL[Rate limit svc]
  R[(Redis cluster)]
  GW --> EDGE
  EDGE --> RL
  RL --> R
```

### 4.1 Phases

| Phase | Ship |
|-------|------|
| **1** | Single-region Redis + Lua |
| **2** | **Local token bucket** + **async sync** |
| **3** | **Global** quotas with **hierarchical** aggregation |

---

## 5. Deep dive: allow check

<a id="say-voice-5"></a>

<a id="bottleneck-anchor-once"></a>
### 🎯 Bottleneck Anchor

“**Redis hot key** per **viral user**—**shard** keyspace or **local** burst absorption.”

```mermaid
sequenceDiagram
  participant GW as Gateway
  participant L as Limiter
  participant R as Redis
  GW->>L: allow(user:42, route=/search)
  L->>R: EVAL token_bucket
  R-->>L: ok / deny + ttl
  L-->>GW: 200 or 429
```

**Taking a stance:** *“**Token bucket** for **smooth bursts**; **sliding log** only when **hard** audit window needed (more storage).”*

---

## 6. Scaling and bottlenecks

| Risk | Mitigation |
|------|------------|
| **Hot key** | **Sharding** + **local** limit |
| **Cross-region** | **Sticky** routing; **CRDT-ish** counters (rare) |

---

## 7. Reliability and failure handling

- **Redis partial fail:** **degrade** to **local** only with **lower** cap.  
- **Clock skew:** rely on **Redis TIME** or **server** time authority.

---

## 8. Tradeoffs and alternatives

| Choice | Trade |
|--------|--------|
| **Central Redis** | Accuracy vs **RTT** |
| **Gossip** | No single point vs **overcount** |

---

## 9. Monitoring, observability, and security

**Metrics:** **429 rate** by route, **sync lag**, **Redis** latency.  
**Security:** **Keys** must not leak **PII** in logs; **bypass** only for **break-glass** audit.

---

## 10. Design patterns, data structures & best practices

| Pattern | Map |
|---------|-----|
| **Token bucket** | Smooth rate |
| **Leaky bucket** | Strict output rate |
| **Bulkhead** | Isolate noisy neighbors |

<a id="say-voice-10"></a>
**Live:** max **four** patterns.

---

## Closing notes

<a id="communication-do-vs-avoid"></a>

| Do | Avoid |
|----|--------|
| **State over-admission** | “Exactly once global” hand-wave |
| **Lua atomic** | Racey read-modify-write |

---

## Bar-raiser follow-ups

| They ask | Say it like this |
|----------|------------------|
| **Distributed consensus** | “Usually **overkill**—**central** store + **edge** **approx** wins.” |

---

## 60-second close

| Beat | Say it like this |
|------|------------------|
| **Recap** | “**Token bucket** + **Redis Lua**; **hierarchy** user/route/tenant; **hot key** mitigations; **fail-open/closed** explicit; **429** + **Retry-After**.” |

---
