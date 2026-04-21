# HLD — Distributed Cache

<a id="interview-spine-nine-steps"></a>

## 1. Clarify requirements

### 1.0 Live flow

<a id="live-flow-open"></a>

**Opening (~once):** *“I’ll align **read vs write-through**, **consistency** (eventual vs strong), **eviction**, and **stampede**; then **client library**, **cluster topology**, **failure modes**. **Pause after the diagram**—**Redis cluster**, **invalidation**, or **multi-region**?”*

**Thinking transitions:** *“Cache is **not** a **source of truth**—I’ll say what **owns** truth.”*

<a id="say-1-questions-human"></a>
### 1.1 Clarify

| Topic | Say it like this in the room |
|--------------------------|-------------------------------|
| **Workload** | “**Hot key** social graph vs **session**?” |
| **TTL** | “Default **staleness** tolerance?” |
| **Size** | “**MB** values ok?” |
| **Multi-region** | “**Active-active** writes?” |

### 1.2 Functional requirements (FR)

<a id="say-fr-human"></a>

| FR area | Say it like this |
|---------|-------------------|
| **KV** | “Get/set/del with **TTL**.” |
| **Collections** | “Sets, sorted sets, hashes if Redis-shaped.” |
| **Optional** | “Pub/sub **invalidation** hints.” |

### 1.3 Non-functional requirements (NFR)

| NFR | Say it like this |
|-----|------------------|
| **Latency** | “**Sub-ms** in-AZ typical.” |
| **Availability** | “**Degrade** to **origin** on miss/outage.” |

### 1.4 Invariants

**Invariant:** “**Mutations** to **authoritative** state go through **primary store** first (or **write-through** with **acknowledged** consistency model); **cache** never **silently** invents **commits**.”

<a id="say-voice-1"></a>

| Beat | Say it like this |
|------|------------------|
| **Bridge** | “**Cache-aside** + **TTL** + **jitter** for **most** reads.” |
| **Core split** | “**Client** owns **coalescing** / **circuit breaker**.” |

<a id="key-insight-say-early"></a>
### Key insight (say early)

**Solve thundering herd** with **singleflight / request coalescing**, **probabilistic early refresh**, and **TTL jitter**—not bigger machines alone.

#### Key anchors

1. “**Consistent hashing** + **replicas**.”  
2. “**Hot key** **split** logical keys.”  
3. “**LRU** + **maxmemory policy**.”

---

## 2. Estimate scale

<a id="say-voice-2"></a>

| Dimension | Illustrative |
|-----------|----------------|
| QPS | **Millions** aggregate |
| Memory | **TB** cluster |

---

## 3. APIs and data model

<a id="say-voice-3"></a>

### 3.1 Client API (conceptual)

- `get(key)`, `set(key, val, ttl)`, `del(key)`  
- **CAS** optional (`SET NX`, versioned **ETag** pattern)

---

## 4. High-level architecture

<a id="say-voice-4"></a>

```mermaid
flowchart TB
  APP[App servers]
  C[Cache client lib]
  R1[Redis primary]
  R2[Replica]
  DB[(Origin DB)]
  APP --> C --> R1
  R1 --> R2
  C -->|miss| DB
```

### 4.1 Phases

| Phase | Ship |
|-------|------|
| **1** | Single shard Redis + replicas |
| **2** | **Cluster** + **client-side** routing |
| **3** | **Global** **active-passive** with **conflict** rules |

---

## 5. Deep dive: cache-aside read

<a id="say-voice-5"></a>

<a id="bottleneck-anchor-once"></a>
### 🎯 Bottleneck Anchor

“**Thundering herd** on **popular key expiry**—**coalesce** + **stale-while-revalidate**.”

```mermaid
sequenceDiagram
  participant A as App
  participant C as Cache
  participant D as DB
  A->>C: get(key)
  alt miss
    C-->>A: miss
    A->>A: singleflight lock
    A->>D: load
    D-->>A: row
    A->>C: set + TTL jitter
  end
  A-->>User: response
```

**Taking a stance:** *“**Write-through** when **read-after-write** must be **fresh**; **aside** for **most** denormalized reads.”*

---

## 6. Scaling and bottlenecks

| Risk | Mitigation |
|------|------------|
| **Hot key** | **Subkey sharding** in app |
| **Big payload** | **Compression** + **chunking** policy |
| **Failover** | **Replica** promotion with **fencing** if strong consistency needed |

---

## 7. Reliability and failure handling

- **Cluster split:** prefer **availability** + **stale** vs **split-brain writes**—define.  
- **Origin slow:** **bulkhead** threads fetching DB.

---

## 8. Tradeoffs and alternatives

| Choice | Trade |
|--------|--------|
| **Redis vs Memcached** | Rich ops vs **simplicity** |
| **Local L1** | Speed vs **invalidation** complexity |

---

## 9. Monitoring, observability, and security

**Metrics:** **hit rate**, **latency**, **evictions**, **connections**.  
**Security:** **TLS**, **ACL** per prefix; **no secrets** in values.

---

## 10. Design patterns, data structures & best practices

| Pattern | Map |
|---------|-----|
| **Cache-aside** | Most web reads |
| **Write-through** | Strong read-your-writes |
| **SWR** | Stale-while-revalidate |

<a id="say-voice-10"></a>
**Live:** max **four** patterns.

---

## Closing notes

<a id="communication-do-vs-avoid"></a>

| Do | Avoid |
|----|--------|
| **Stampede story** | “We’ll set a TTL” only |
| **Ownership of truth** | Cache as database |

---

## Bar-raiser follow-ups

| They ask | Say it like this |
|----------|------------------|
| **Near-cache** | “**Per-process** L1 + **pub/sub** invalidation—**complex**.” |

---

## 60-second close

| Beat | Say it like this |
|------|------------------|
| **Recap** | “**Cache-aside** default; **TTL+jitter**; **singleflight**; **Redis cluster** + **replicas**; **hot key** patterns; **never** sole **source of truth** for **commits**.” |

---
