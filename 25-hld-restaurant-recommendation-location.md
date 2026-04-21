# HLD — Restaurant Recommendation System Based on User Location

## Live interview opening (say naturally)

*“I’ll start from the **user perspective**, clarify key requirements, then design **high-level architecture** and go deeper on the **most critical** part. I’ll **pause after the diagram** in case you want to go deeper into any section.”*

## User journey (say once early)

*“From the user perspective: **(prep: one–two lines for this system)**.”*

*“So: **write path** = … ; **read path** = … ; **async path** = ….”* — fill with concrete nouns from **Section 1** and your diagram as you speak.

## Thinking transitions (use during interview)

- *“Let me think through this…”*
- *“One tradeoff here is…”*
- *“If I optimize for latency…”*
- *“This might become a bottleneck because…”*
- *“I’d start simple here and evolve later…”*

## Consistency model

*“**Strong** consistency for **(critical part)** because **(reason)** ; **eventual** for **(non-critical)** because **(reason)** . Under load we prioritize **(latency / correctness / availability)** on **(which surface)** .”* — align with **Section 1 invariants** and any dedicated consistency blocks in this guide.

## Decision (strong opinion)

*“I’d start with **X** because **(reason)** . If **(scale / requirements / signals)** change, I’d evolve to **Y**.”* — state your real default from **Section 8** in the room.

## Evolution

| Phase | Say it like this |
|-------|------------------|
| **1** | Simple implementation that ships. |
| **2** | Scaling: partitions, caches, queues, backpressure, observability. |
| **3** | Advanced / ML / global—only when metrics or product force it. |

Details: **Section 4.1 (phases)** and **Section 5** in this file.

## Bottleneck anchor

*“The main bottlenecks I expect are **(1)** and **(2)** —that’s what I’d monitor first.”* — concrete wording lives under **Section 5 — Bottleneck** in this guide.

## UX awareness

*“If this behaves badly, users see **(impact)** —so we prioritize **(trust lever)** .”* — tie to **reliability / degrade / UX** sections later in this guide.

## Driving the conversation

- *“Does this direction make sense?”*
- *“Should I go deeper on **A** or **B**?”*
- *“Would you like failure scenarios next?”*

## Mindset (before you walk in)

*“I’m not presenting a solution—I’m **designing with a teammate**.”*

**Rehearsal beats editing:** speak aloud, practice **pauses**, simulate **interruptions**. **Playbook:** [HLD-BAR-RAISER-PERFORMANCE-PACK.md](./HLD-BAR-RAISER-PERFORMANCE-PACK.md).

---

<a id="interview-spine-nine-steps"></a>

> **Uber SDE-2 HLD — drive order in this doc:** **§1** clarify → FR → NFR → **§2** scale → **§3** core entities + APIs → **§4** architecture → **§5** deep dive and evolution → **§6** scaling → **§7** reliability → **§8** tradeoffs → **§9** observability and security → **§10** patterns → **Closing**. Treat **Human interaction** cue blocks (headings in this doc) as *spoken* cues—**paraphrase**; do not read every row. **Bar raiser** listens for **ownership**, **failure modes**, and **honest tradeoffs**. Canonical spine: [HLD-UBER-SDE2-INTERVIEW-SPINE.md](./HLD-UBER-SDE2-INTERVIEW-SPINE.md).

## Interview delivery (golden thread — live thinking)

Bar-raiser polish: **user-first**, **explicit consistency**, **bottleneck**, **evolution**, **UX trust**, **default opinion** (not endless “A or B”). Full template + anti–document-mode habits: **[HLD-BAR-RAISER-PERFORMANCE-PACK.md](./HLD-BAR-RAISER-PERFORMANCE-PACK.md)** (final lines + sections) · **[HLD-MASTER-DELIVERY-GOLDEN-FLOW.md](./HLD-MASTER-DELIVERY-GOLDEN-FLOW.md)** (golden flow + anti-doc table).

| Say early (out loud) | What interviewers grade | In this guide, nail it by… |
|---------------------|---------------------------|------------------------------|
| **User journey** | Product before boxes | Opening Section 1 with who does what; separate **read / write / async** before services. |
| **Consistency model** | Strong vs eventual, where | Stating **invariants** and what is **strict vs best-effort** before API trivia. |
| **Bottleneck anchor** | What breaks first | Naming Section 5 **Bottleneck** + Section 6 hot paths + **first SLIs**. |
| **Evolution** | MVP → scale → advanced | Using **v1 / v2 / v3** (often Section 4.1 + Section 5); say **when** complexity earns its keep. |
| **UX awareness** | Trust on degrade | Saying what the user **sees** on partial failure (honest empty vs wrong vs spin forever). |
| **Strong opinion** | Defaults | *“I’d start with **X** because …; I’d switch to **Y** if …”*—lead Section 8 with a pick. |

**Do not:** read tables line-by-line · list ten patterns before a diagram · fence-sit.  
**Do:** signpost · pause · one diagram · deep dive **only** where they steer.

---

## 1. Clarify requirements

### 1.0 Live flow (how to open and steer)

<a id="live-flow-open"></a>

#### Live voice (real interviewer room)

**Sound like you’re *deciding*, not reciting:** one idea per breath, then **pause**. Tables here are **backup**—if your eyes are down for more than a couple of seconds, you’ve slipped into reading the doc.

**Bridge phrases (mix naturally):** *“Let me **name the fork** first…”* · *“I’ll **default to X**—tell me if your bar is stricter.”* · *“The reason I ask is it changes **who owns the commit** / **what’s on the hot path**.”* · *“I’ll **over-answer** one layer, then stop—**where should I zoom**?”*

**Ping them (conversation, not monologue):** *“Does that match how you’d scope it?”* · *“If we only deep-dive one thing, is it **A** or **B**?”* (swap **A/B** for two tensions from *your* opening paragraph above.)

**This topic in one breath:** “Reco is **eligible near you first**, model second—otherwise ML hides bad geo.”

**`Verbatim` / `Live` cues:** say a line **once**, then **rephrase** the next time—verbatim twice in a row reads *canned*.

**Opening (~once):** *“I’ll treat this as **ranking under latency**: **geo eligibility**, **personalization**, **cold start**, **experiments**; then **features**, **serving**, **architecture**. **Pause after the diagram**—**features**, **fairness**, or **infra**?”*

**Thinking transitions:** *“Same **read funnel** as homepage—**candidates** capped before **model**.”* (See [11-hld-uber-eats-homepage.md](./11-hld-uber-eats-homepage.md).)

**Live rule:** **Paraphrase** §1–2 tables; don’t read every row. Go deep **only if they probe**.

**When (HLD clock):** the **user-journey script** lives **[just above §4](#user-journey-reco-25)**—say it **once** immediately **before** the architecture diagram so the feed is **user-first**. Optional: **one clause** in clarify if you opened model-first.

<a id="say-1-questions-human"></a>
### 1.1 Clarify

| Topic | Say it like this in the room |
|--------------------------|-------------------------------|
| **Surface** | “**Home rail**, **search zero-query**, **email**—which?” |
| **Objective** | “**GMV**, **CTR**, **diversity**?” |
| **Privacy** | “**Precise** lat vs **neighborhood** features?” |
| **Sponsored** | “**Mixer** slot?” |

**Micro-pauses:** *“So **retrieval** is **geo candidates**, **rank** is **model + mixer**, and **correctness** is **eligibility**—got it.”*

#### Human interaction (clarify requirements — think out loud & evolve scope)

**Habit:** *“Reco is **ranking under latency**—I pin **objective**, **privacy**, and **sponsored** before **embedding** dimensions.”*

| Stage | Default | Evolve when… |
|-------|---------|----------------|
| **v1** | Heuristic / distance | Cold start |
| **v2** | **GBDT** + feature logging | Quality + debuggability |
| **v3** | **Two-tower** + online learning | Candidate scale |

### 1.2 Functional requirements (FR) — after alignment, say this as "what we must build"

<a id="say-fr-human"></a>

#### Human interaction (FR — after alignment)

**Habit:** *“**Retrieve → feature → score → mixer → log**—say it once.”*

| FR area | Say it like this |
|---------|-------------------|
| **Candidates** | “From **geo index**, pull **eligible** restaurants in **radius/cells**.” |
| **Features** | “User history, **context** (time, weather), **distance**, **popularity**, **merchant** quality.” |
| **Rank** | “Score + **mixer** + **diversity** constraints.” |
| **Log** | “**Impression/click** for **training** and **fairness** audits.” |

### 1.3 Non-functional requirements (NFR) — say as "how it must behave"

<a id="say-nfr-human"></a>

#### Human interaction (NFR — how it must behave)

**Live:** *“**Time-box** rank; **fallback** is a **product-safe** list, not random.”*

| NFR | Say it like this |
|-----|------------------|
| **Latency** | “**Time-box** rank; **fallback** to **distance + popularity**.” |
| **Freshness** | “**Feature** lag **seconds–minutes** acceptable for many signals.” |

### 1.4 Invariants

**Invariant:** “**Recommended** restaurants are always **hard-eligible** to serve the user’s **delivery context**; **rank order** may degrade.”

<a id="consistency-model-reco-25"></a>

## ⚖️ Consistency Model

Bar-raiser thread: *“**How fresh** are recommendations?”*

Say it like this:

*“Recommendations are **eventually consistent**:

- **Eligibility** is **always correct** (hard filter—never **ML-overridden**).  
- **Features** may be **slightly stale** (**seconds–minutes**) for many signals—**bounded** and **observed**.  
- **Model outputs** are designed to **tolerate** that lag (**fallback** ranker, **time-box**, **versioned** features).”*

<a id="say-voice-1"></a>

**Purpose:** no second “clarify lecture”—only the **handoff** from answers → design.

| Beat | Say it like this |
|------|------------------|
| **Bridge** | “**I’d start with GBDT** for **interpretability** and **shipping** speed; **move to two-tower** at **scale** when **retrieval** + **embedding** efficiency dominates—**serving** shape (**cap K → features → score → mixer**) stays the same.” |
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

#### Human interaction (estimate scale)

**Live:** *“Rank **QPS** tracks **home**; **feature cardinality** drives **fan-out**—**cap K** early.”*

| Dimension | Illustrative |
|-----------|----------------|
| Rank QPS | Same order as **homepage** reads |
| Feature cardinality | **High**—**sparse** embeddings |

---

## 3. APIs and data model

<a id="say-voice-3"></a>

### 3.0 Core entities (who owns what — say before API tables)

| Entity | Owns / lifecycle (one line) |
|--------|-----------------------------|
| **UserContext** | **Privacy**-safe location bucket + session. |
| **RestaurantItem** | **Catalog** features + **eligibility** flags. |
| **CandidateSet** | **Geo retrieval** output—**capped K**. |
| **RankedList** | **Model + mixer** output + **impression id**. |
| **TrainingLog** | **Impression/click** events—**async**. |

#### Human interaction (API design — read API + async logging)

**Live:** *“**GET reco** is synchronous to a **deadline**; **`POST events`** is **async**—never block **p99** on logging.”*

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

<a id="user-journey-reco-25"></a>

### 👤 User journey (say once—before this diagram)

*“**User opens app** → system **fetches nearby** restaurants → **filters eligible** → **ranks** with **personalization** → **returns feed**.

So:

- **retrieval** = **geo** candidates  
- **ranking** = **ML** + **mixer**  
- **correctness** = **eligibility**.”*

---


<a id="say-voice-4"></a>

#### Human interaction (high-level architecture / HLD)

**Habit:** *“Say [journey](#user-journey-reco-25) then boxes: **GEO → FS → Model → Mixer → LOG**.”*

**Live:** *“**Mixer** owns **sponsored** + **diversity** without breaking **[eligibility invariant](#consistency-model-reco-25)**.”*

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
| **2** | **GBDT** + logged **features** + **batch** train (default **interpretable** path) |
| **3** | **Two-tower** + **online** learning (**after** GBDT baseline proves **traffic**) |

---

<a id="ux-awareness-reco-25"></a>

## 👤 UX Awareness

If the feed feels **repetitive** or **biased**, **trust** drops—so we **enforce** **diversity** and **exploration** slots in the **mixer** (and **watch** impression share for **cold** merchants). Pair with **transparent** “why” only if product wants it—**never** at the cost of **p99**.

---

## 5. Deep dive: rank request

<a id="say-voice-5"></a>

#### Human interaction (deep dive — critical flow, optimizations & evolution)

**Habit:** *“Sequence **GEO cap K → batch feature fetch → score under deadline → mixer**.”*

**Live (evolution):** *“**v1** heuristics. **v2** GBDT + logged features. **v3** two-tower retrieval—**serving skeleton** unchanged.”*

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

#### Human interaction (scaling & bottlenecks)

**Live:** *“**Hot user** embeddings and **model bulkhead**—**fallback ranker** is production, not shame.”*

| Risk | Mitigation |
|------|------------|
| **Hot user** | **Cache** user embedding |
| **Model OOM** | **Bulkhead** + **fallback** ranker |

---

## 7. Reliability and failure handling

#### Human interaction (reliability & failure handling)

**Live:** *“**Missed features** → **defaults** + **down-rank**; **model timeout** → **heuristic**—user still sees **eligible** food.”*

- **Feature store miss:** **default** features; **down-rank** unknowns.  
- **Model timeout:** **heuristic** sort.

---

## 8. Tradeoffs and alternatives

#### Human interaction (tradeoffs & alternatives)

**Live:** *“**Personalization** vs **bubble** is a **mixer** + **metrics** problem—call it explicitly.”*

| Choice | Trade |
|--------|--------|
| **Heavy personalization** | Engagement vs **filter bubble** |
| **GBDT → two-tower** | **GBDT** first for **debuggability** / **ops**; **two-tower** when **candidate scale** + **latency** force **approximate** retrieval—**cost** is **complexity** + **freshness** of **cross** terms |

---

## 9. Monitoring, observability, and security

#### Human interaction (monitoring, observability & security)

**Habit:** *“Slice **CTR** by **distance decile** to catch **geo bugs** early.”*

**Metrics:** **p99** rank latency, **fallback** rate, **new merchant** impression share, **CTR** slice by **distance decile**.  
**Security:** **No** cross-user **feature** leaks; **PII** in **feature store** encrypted.

---

## 10. Design patterns, data structures & best practices

#### Human interaction (design patterns, data structures & best practices)

**Verbatim (say on the board, ~30s):** *“**Eligibility before model**—geo and hours are **hard filters**; then **two-stage retrieve + rank** with **GBDT** or **two-tower** embeddings; **mixer** for sponsored vs organic; **contextual bandits** for exploration; **cache-aside** on the **feature store**; **deadline + fallback** when the ranker is slow.”*

**Live:** *“**Mixer**, **bandits**, **cache-aside**—tie to **Reco**, **FS**, **GEO** boxes.”*

| Pattern / DS | Where | One interview line |
|----------------|------|----------------------|
| **Retrieve → rank (funnel)** | Reco svc | “**Thousands → hundreds → dozens** under a **latency** budget.” |
| **Mixer / Decorator** | Feed assembly | “Organic first, **caps** on sponsored adjacency for **trust**.” |
| **Contextual bandit / ε-greedy** | Exploration | “Cold-start gets **explore** with **guardrails** on distance.” |
| **Cache-aside + TTL** | Feature store | “Features can be **seconds stale**; **eligibility** cannot.” |
| **Two-tower / ANN (later)** | Candidate gen | “When **catalog** explodes, **ANN** replaces brute force k-NN.” |
| **Deadline + fallback** | Rank path | “Miss the **SLO** → heuristic sort by distance + rating.” |

<a id="say-voice-10"></a>
**Live:** pick **five or six** rows; **eligibility-first** is the bar-raiser line.

---

## Closing notes (where wrap-up human interaction lives)

#### Human interaction (closing notes)

**Live:** *“**Eligibility** beats **model**; **GBDT→two-tower** is an **evolution**, not a rewrite.”*

Endgame is **short**, **confident**, and **conversational**: drive the wrap from [Bar-raiser](#bar-raiser-follow-ups), [Communication (do vs avoid)](#communication-do-vs-avoid), and [60-second close](#60-second-close)—not a second full design pass.

<a id="communication-do-vs-avoid"></a>

### Communication (do vs avoid)

| Do (sounds senior) | Avoid (sounds rehearsed) |
|--------------------|---------------------------|
| **Eligibility before model** | “ML fixes bad zone” |
| **Time-box** | 10 min on **embedding dim** |

---

## Bar-raiser follow-ups

#### Human interaction (bar-raiser follow-ups)

| They ask | Say it like this |
|----------|------------------|
| **Counterfactual** | “**Logging policy** bias—**IPS** corrections / **randomized** buckets.” |
| **How fresh is reco?** | “[Consistency model](#consistency-model-reco-25): **eligibility** strict; **features** **seconds–minutes** stale **bounded**; **fallback** + **deadline**.” |
| **Filter bubble** | “[UX awareness](#ux-awareness-reco-25): **diversity** + **exploration** in **mixer**, **metrics** on **share**.” |

---

## 60-second close

#### Human interaction (60-second close)

| Beat | Say it like this |
|------|------------------|
| **Recap** | “**Journey**: open → nearby → **eligible** → rank → feed. **Consistency**: **hard eligibility**; **feature lag** OK **bounded**; **tolerate** in model + **fallback**. **Model path**: **GBDT** first → **two-tower** at scale. **UX**: **diversity** / **exploration** for **trust**.” |

---
