# HLD — URL Shortener (TinyURL)

## Live interview opening (clarify first — bar raiser order)

*“I’ll **start by clarifying requirements**—scope, ambiguity, latency and scale expectations—then lock **FR/NFR**. **After** that, I’ll ground **user journey**, **consistency**, **commit/decision**, and **risks** so it’s clearly **derived from what we agreed**, then **scale** and **architecture**—and I’ll **pause after the diagram** for where you want depth.”*

<a id="interview-spine-nine-steps"></a>

> **Uber SDE-2 HLD — drive order in this doc:** **§1** clarify → FR → NFR → **Framing after requirements** (user journey, consistency, commit/decision anchors) → **§2** scale → **§3** core entities + APIs → **§4** architecture → **§5** deep dive and evolution → **§6** scaling → **§7** reliability → **§8** tradeoffs → **§9** observability and security → **§10** patterns → **Closing**. Treat **Human interaction** cue blocks (headings in this doc) as *spoken* cues—**paraphrase**; do not read every row. **Bar raiser** listens for **ownership**, **failure modes**, and **honest tradeoffs**. Canonical spine: [HLD-UBER-SDE2-INTERVIEW-SPINE.md](./HLD-UBER-SDE2-INTERVIEW-SPINE.md).

## Interview delivery (golden thread — live thinking)

Bar-raiser polish: **user-first**, **explicit consistency**, **bottleneck**, **evolution**, **UX trust**, **default opinion** (not endless “A or B”). Full template + habits: **[HLD-BAR-RAISER-PERFORMANCE-PACK.md](./HLD-BAR-RAISER-PERFORMANCE-PACK.md)** · **[HLD-MASTER-DELIVERY-GOLDEN-FLOW.md](./HLD-MASTER-DELIVERY-GOLDEN-FLOW.md)**.

| Say at the right time | What interviewers grade | In this guide |
|----------------------|---------------------------|---------------|
| **Opening** | Clarify before solution | **Above** — you **do not** assume requirements. |
| **User journey + consistency + decisions** | Derived, not memorized | **After Section 1**, block **Framing after requirements** — **before Section 2** (not before clarify). |
| **Bottleneck / evolution / UX** | Ops + trust | Same **Framing** block; deep numbers still in **Sections 5–7**. |
| **Strong opinion** | Defaults | **Section 8** tradeoffs — always *“I’d start with X because…”*. |

**Do not:** read tables line-by-line · put user journey **before** clarify · fence-sit.  
**Do:** clarify → FR/NFR → **then** grounded journey → diagram → deep dive where steered.

---

## 1. Clarify requirements

### 1.0 Live flow

<a id="live-flow-open"></a>

#### Live voice (real interviewer room)

**Sound like you’re *deciding*, not reciting:** one idea per breath, then **pause**. Tables here are **backup**—if your eyes are down for more than a couple of seconds, you’ve slipped into reading the doc.

**Bridge phrases (mix naturally):** *“Let me **name the fork** first…”* · *“I’ll **default to X**—tell me if your bar is stricter.”* · *“The reason I ask is it changes **who owns the commit** / **what’s on the hot path**.”* · *“I’ll **over-answer** one layer, then stop—**where should I zoom**?”*

**Ping them (conversation, not monologue):** *“Does that match how you’d scope it?”* · *“If we only deep-dive one thing, is it **A** or **B**?”* (swap **A/B** for two tensions from *your* opening paragraph above.)

**This topic in one breath:** “Short links are **read-heavy, write-rare**—ID generation and **abuse** are the design.”

**`Verbatim` / `Live` cues:** say a line **once**, then **rephrase** the next time—verbatim twice in a row reads *canned*.

**Opening (~once):** *“I’ll align **custom vs random** slugs, **TTL**, **analytics**, **abuse**, and **read:write ratio**; then **ID generation**, **storage**, **redirect path**. **Pause after the diagram**—**collision**, **scale**, or **security**?”*

**Thinking transitions:** *“**Reads dominate**—**cache** + **edge**; **writes** need **unique** id strategy.”*

**Live rule:** **Paraphrase** §1 tables; go deep **only if probed**.

**Micro-pauses:** *“So redirect is **read-heavy** with **immutable** slug→URL—I'll cache at edge and keep **ID generation** off the hot path.”*

<a id="say-1-questions-human"></a>
### 1.1 Clarify

#### Human interaction (clarify requirements — think out loud & evolve scope)

**Verbatim:** *“I'm aligning **custom vs random** slugs, **TTL**, **analytics**, **abuse**, and **read:write ratio**—that drives **Snowflake/Base62** vs preallocated blocks and **edge** caching.”*

**Verbatim (evolution):** *“**v1** single DB; **v2** Redis cache plus sharded KV; **v3** geo edge redirects with **consistent hash**.”*

| Topic | Say it like this in the room |
|--------------------------|-------------------------------|
| **Length** | “**7-character** Base62 ok?” |
| **Private** | “**Auth** links?” |
| **Preview** | “**Unfurl** safety / malware scan?” |
| **Deletion** | “**GDPR** erase mapping?” |

### 1.2 Functional requirements (FR)

<a id="say-fr-human"></a>

#### Human interaction (FR — after alignment)

**Verbatim:** *“Create returns a short URL, resolve does **302** to long URL, and click analytics can be **async** off the critical path.”*

| FR area | Say it like this |
|---------|-------------------|
| **Create** | “`POST /shorten` → **short URL**.” |
| **Resolve** | “`GET /{slug}` → **302** to long URL.” |
| **Optional** | “Click analytics async.” |

### 1.3 Non-functional requirements (NFR)

<a id="say-nfr-human"></a>

#### Human interaction (NFR — how it must behave)

**Verbatim:** *“Redirect p99 in milliseconds at the edge; billions of mappings mean **shard by slug** and **no** DB uniqueness retry storm on the happy path.”*

| NFR | Say it like this |
|-----|------------------|
| **Latency** | “Redirect **p99** **ms** at edge.” |
| **Scale** | “**Billions** mappings; **400M+** reads/day classic interview number.” |

### 1.4 Invariants

**Invariant:** “A published **slug** maps to **at most one** **active** long URL at a time; **redirect** is **idempotent** for readers (until **revoked**).”

<a id="say-voice-1"></a>

| Beat | Say it like this |
|------|------------------|
| **Bridge** | “**Counter / snowflake** → **Base62** → **KV**.” |
| **Core split** | “**Generate** separate from **serve**.” |

<a id="key-insight-say-early"></a>
### Key insight (say early)

**Pre-generated** blocks of random IDs from a **central allocator** OR **base62 hash** of **snowflake**—avoid **DB uniqueness retry storm**.

#### Key anchors

1. “**302** vs **301**—**analytics** vs **permanent** semantics.”  
2. “**Bloom** optional for **abuse**.”  
3. “**CDN** for **redirect** at scale.”

---

## Framing after requirements (before scale + architecture)

**Placement in the room:** this block is **not** “right after clarify questions.” You earn it **after** you’ve locked **FR + NFR** (spoken summary is fine)—otherwise journey / consistency / commit boundaries read as **assuming** product and SLOs.

**Out loud:** *“We’ve **clarified** scope and I’ve stated **FR/NFR** from that—**based on that**, here’s the **user-visible path**, **consistency**, and **commit** I’ll hold before I size **Section 2** and draw **Section 4**.”*

### Thinking transitions (use during interview)

- *“Let me think through this…”*
- *“One tradeoff here is…”*
- *“If I optimize for latency…”*
- *“This might become a bottleneck because…”*
- *“I’d start simple here and evolve later…”*

## User journey (say once—**after** FR/NFR, not after clarify alone)

From the **clicker** perspective: open short URL → **302/301** redirect to long URL (or interstitial for **safe browsing** if product requires).

From the **creator** perspective: **create** mapping → share link → later **analytics** on clicks.

So:

- **read path** = **GET** `/{id}`—**massively** dominant QPS; must be **cacheable** / edge-friendly.
- **write path** = **create** short code + store mapping (**idempotent** keys if needed).
- **async path** = **click logging**, **abuse** scanning, **GC** of expired links.

## Consistency model

**Create** must avoid **collisions** on short codes—**strong** uniqueness in the **generator + DB unique constraint** story.

**Redirect target** updates: define **eventual** vs **immediate** consistency per product (**301** vs **302** semantics matter).

## Commit boundary

Mapping is **public** after **durable** insert (and any **malware** scan policy completes—or you explicitly redirect to “pending” interstitial).

**Click** analytics are **async**—must not block redirect **p99**.

## Decision (strong opinion)

I’d start with:

- **base62** IDs + **SQL/NoSQL** row + **Redis/CDN** cache for hot slugs + **precomputed** redirect responses.

because workload is **read-heavy, write-rare**; **abuse** + **collision** dominate design time.

## Evolution

| Phase | Say it like this |
|-------|------------------|
| **1** | Simple implementation that ships. |
| **2** | Scaling: partitions, caches, queues, backpressure, observability. |
| **3** | Advanced / ML / global—only when metrics or product force it. |

Details: **Section 4.1 (phases)** and **Section 5** in this file.

## Bottleneck anchor

Watch first:

- **read QPS** vs **edge cache** hit ratio.
- **hot links** (viral) and **scanner** backlog.

## Backpressure handling

Under load:

- **serve** redirect from **edge** even if analytics lags; **sample** logs if needed.
- **block** abusive targets at **write** and **read** paths.

Goal: **fast safe redirect** over **perfect** click counts in real time.

## UX awareness

Bad outcomes:

- **phishing** links using your domain—**safe browsing** + takedown flows.
- **wrong** long URL after edit—**301** caching surprises.

### Driving the conversation

- *“Does this direction make sense?”*
- *“Should I go deeper on **A** or **B**?”*
- *“Would you like failure scenarios next?”*

### Mindset

*“I’m not presenting a solution—I’m **designing with a teammate**.”*

**Playbook:** [HLD-BAR-RAISER-PERFORMANCE-PACK.md](./HLD-BAR-RAISER-PERFORMANCE-PACK.md).

---

## 2. Estimate scale

<a id="say-voice-2"></a>

#### Human interaction (estimate scale)

**Verbatim:** *“Writes are thousands per second, reads hundreds of thousands to millions—**CDN** and **cache-aside** dominate the design.”*

| Dimension | Illustrative |
|-----------|----------------|
| Writes / sec | **1k–10k** |
| Reads / sec | **100k–1M+** |
| Storage | **Billions** rows—**sharding** by **slug** |

---

## 3. APIs and data model

<a id="say-voice-3"></a>

### 3.0 Core entities (who owns what — say before API tables)

| Entity | Owns / lifecycle (one line) |
|--------|-----------------------------|
| **UrlMapping** | `slug` → `long_url`—**immutable** while active. |
| **SlugAllocator** | Unique id generation—**single-writer** or **snowflake**. |
| **ClickEvent** | Optional async analytics—**append-only**. |

#### Human interaction (API design)

**Verbatim:** *“POST creates with optional custom slug—**409** on collision; GET resolve is **cache-first** then origin with **302** semantics explicit.”*

### 3.1 APIs

| API | Purpose |
|-----|---------|
| `POST /v1/urls` | Create `{long_url}` → `{slug}` |
| `GET /{slug}` | Redirect |

### 3.2 Model

- **UrlMapping:** `slug (PK)`, `long_url`, `owner_id?`, `created_at`, `expires_at?`, `revoked`.

---

## 4. High-level architecture

<a id="say-voice-4"></a>

#### Human interaction (high-level architecture / HLD)

**Verbatim:** *“Users hit **CDN** for redirect reads; write path goes to API into sharded **KV**—**generate** and **serve** are intentionally separate scaling knobs.”*

```mermaid
flowchart LR
  U[Users]
  CDN[CDN / Edge]
  API[Write API]
  R[Read svc]
  DB[(KV / SQL)]
  U --> CDN --> R --> DB
  U --> API --> DB
```

### 4.1 Phases

| Phase | Ship |
|-------|------|
| **1** | Single DB + app |
| **2** | **Redis** cache + **sharded** KV |
| **3** | **Geo** **edge redirects** |

---

## 5. Deep dive: redirect

<a id="say-voice-5"></a>

#### Human interaction (deep dive — critical flow, optimizations & evolution)

**Verbatim:** *“Browser hits edge, edge checks cache—on miss fetch from store, populate cache with **long TTL** because mapping is immutable, return **302**—hot slug gets **request coalescing** to avoid stampede.”*

**Verbatim (evolution):** *“If you need **301** for SEO permanence, call out **analytics** tradeoff explicitly.”*

<a id="bottleneck-anchor-once"></a>
### 🎯 Bottleneck Anchor

“**Cache stampede** on **hot slug**—**coalesce** + **long TTL** for **immutable** mappings.”

```mermaid
sequenceDiagram
  participant B as Browser
  participant E as Edge
  participant C as Cache
  participant DB as Store
  B->>E: GET /abc123
  E->>C: lookup
  alt hit
    C-->>E: long_url
  else miss
    E->>DB: get
    DB-->>E: long_url
    E->>C: populate
  end
  E-->>B: 302 Location
```

**Taking a stance:** *“**Base62** of **64-bit id** from **Snowflake**—**no** collision **check** on happy path.”*

---

## 6. Scaling and bottlenecks

#### Human interaction (scaling & bottlenecks)

**Verbatim:** *“Hot slug is an edge cache problem; skew across shards is **consistent hashing**—I'm not pretending uniform random slugs always balance.”*

| Risk | Mitigation |
|------|------------|
| **Hot key** | **Edge cache** |
| **Skew** | **Consistent hash** shards |

---

## 7. Reliability and failure handling

#### Human interaction (reliability & failure handling)

**Verbatim:** *“Missed slug is **404** or branded fallback; phishing means **blocklist** and optional **Safe Browsing** hook—**rate limit** creates.”*

- **Origin miss:** **404** vs **fallback** page.  
- **Phishing:** **blocklist** + **Safe Browsing** API hook.

---

## 8. Tradeoffs and alternatives

#### Human interaction (tradeoffs & alternatives)

**Verbatim:** *“Random slugs are unguessable; vanity is branded but races on **unique index**; hash-of-URL is deterministic but can leak **enumeration**—I'd pick **snowflake + Base62** for interviews.”*

| Choice | Trade |
|--------|--------|
| **Random slug** | Unguessable vs **custom** vanity |
| **Hash of URL** | Deterministic vs **enumeration** risk |

---

## 9. Monitoring, observability, and security

#### Human interaction (monitoring, observability & security)

**Verbatim:** *“Track redirect p99, cache hit, create conflicts; security is **SSRF** guard on long_url, malware scan if we unfurl, and abuse rate limits.”*

**Metrics:** **redirect p99**, **cache hit**, **create** conflict rate.  
**Security:** **Rate limit** creates; **malware** scan on **long_url**; **SSRF** guard for internal URLs.

---

## 10. Design patterns, data structures & best practices

<a id="say-voice-10"></a>

#### Human interaction (design patterns, data structures & best practices)

**Verbatim (say 5–6 on the board):** *“**Snowflake + Base62** or **preallocated id blocks**, **sharded KV** by slug, **cache-aside** at edge, **single-flight** against stampede, optional **Bloom** for abuse, **rate limit** on create, and **unique index** for custom slugs.”*

| Pattern / DS | Where | One interview line |
|----------------|------|----------------------|
| **Time-ordered ID (Snowflake)** | Allocator | “No collision check on the happy path.” |
| **Base62 encoding** | API response | “Short URLs without guessing the whole space.” |
| **Sharded KV / consistent hash** | Store | “Scale reads with bounded blast radius.” |
| **Cache-aside + long TTL** | Edge | “Immutable mapping = cheap caching.” |
| **Single-flight / coalescing** | Hot slug | “One origin fetch for a thundering herd.” |
| **Bloom filter (optional)** | Abuse | “Cheap negative check before expensive work.” |

---

## Closing notes

<a id="communication-do-vs-avoid"></a>

#### Human interaction (closing notes)

**Verbatim:** *“**Read-heavy** redirect at edge, **write-light** create with a real **ID story**, **302** semantics explicit, **abuse** and **SSRF** called out.”*

| Do | Avoid |
|----|--------|
| **ID generation story** | Random until DB unique works |
| **Abuse** | Ignoring phishing use case |

---

## Bar-raiser follow-ups

#### Human interaction (bar-raiser follow-ups)

**Verbatim:** *“Want **collision math**, **GDPR erase**, or **geo routing**—I'll zoom on one.”*

| They ask | Say it like this |
|----------|------------------|
| **Custom slug race** | “**Unique index** + **409** or **retry**.” |

---

## 60-second close

#### Human interaction (60-second close)

**Verbatim:** *“**Write-light**, **read-heavy**; **Snowflake/Base62** or preallocated blocks; **sharded KV**; **edge cache**; **302** semantics; **abuse** controls.”*

| Beat | Say it like this |
|------|------------------|
| **Recap** | “**Write-light**, **read-heavy**; **Snowflake/Base62** or **preallocated** blocks; **KV** sharded by **slug**; **edge cache**; **302** semantics + **abuse** controls.” |

---
