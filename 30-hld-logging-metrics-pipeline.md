# HLD — Logging / Metrics Pipeline

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

**This topic in one breath:** “Observability is **three signals + cardinality discipline**—I won’t block the request on Elasticsearch.”

**`Verbatim` / `Live` cues:** say a line **once**, then **rephrase** the next time—verbatim twice in a row reads *canned*.

**Opening (~once):** *“I’ll split **logs** (debugging) from **metrics** (SLIs) from **traces** (latency); align **retention**, **PII**, **cardinality**; then **ingest**, **storage**, **query**. **Pause after the diagram**—**cost**, **reliability**, or **schema**?”*

**Thinking transitions:** *“**Never** block the **request path** on **observability**—**async buffer**.”*

**Live rule:** **Paraphrase** §1 tables; go deep **only if probed**.

**Micro-pauses:** *“So logs are for **debug**, metrics are for **SLIs**, and neither may **blow cardinality** or leak **PII**—got it.”*

<a id="say-1-questions-human"></a>
### 1.1 Clarify

| Topic | Say it like this in the room |
|--------------------------|-------------------------------|
| **Volume** | “**TB/day** class?” |
| **Query** | “**Keyword** log search vs **metrics** only?” |
| **Compliance** | “**PII scrub** at edge?” |
| **SLI** | “**RED/USE** metrics—**required**?” |

#### Human interaction (clarify requirements — think out loud & evolve scope)

**Verbatim:** *“I’m going to split three signals: **logs**, **metrics**, **traces**—and I want retention, PII policy, and cardinality rules before I draw Kafka boxes, because otherwise we’ll build a beautiful pipeline that bankrupts us.”*

**Verbatim (evolution):** *“**v1** agents to collectors to Kafka to TSDB + log store; **v2** add sampling + SLO dashboards; **v3** tiered storage + federated query.”*

### 1.2 Functional requirements (FR)

<a id="say-fr-human"></a>

#### Human interaction (FR — after alignment)

**Verbatim:** *“Services emit structured telemetry; the pipeline parses, enriches, routes; operators query dashboards and logs—**never** synchronous hot-path writes to Elasticsearch.”*

| FR area | Say it like this |
|---------|-------------------|
| **Ingest** | “Agents / libs ship **logs**, **metrics**, **traces**.” |
| **Process** | “Parse, **enrich**, **sample**, **route**.” |
| **Store** | “Tiered **hot/warm/cold**.” |
| **Query** | “Dashboards + **ad-hoc** log query.” |

### 1.3 Non-functional requirements (NFR)

<a id="say-nfr-human"></a>

#### Human interaction (NFR — how it must behave)

**Verbatim:** *“At-least-once is fine if we dedupe with keys downstream; cost is controlled by **sampling**, **aggregation**, and **blocking** high-cardinality labels at ingest.”*

| NFR | Say it like this |
|-----|------------------|
| **Durability** | “**At-least-once** ingest; **dupes** ok downstream with **keys**.” |
| **Cost** | “**Sampling** + **aggregation** to control **cardinality**.” |

### 1.4 Invariants

**Invariant:** “**PII** never lands in **immutable** cold tiers **unredacted**; **high-cardinality** labels are **blocked** at **ingest**.”

<a id="say-voice-1"></a>

| Beat | Say it like this |
|------|------------------|
| **Bridge** | “**Kafka** buffer → **stream processors** → **TSDB + log store**.” |
| **Core split** | “**Hot path** **fire-and-forget** to **local agent**.” |

<a id="key-insight-say-early"></a>
### Key insight (say early)

**Unified pipeline** with **schema-on-write** for metrics and **indexed** logs—**shared** **enrichment** (service, region, trace id).

#### Key anchors

1. “**OTel**.”  
2. “**Cardinality guardrails**.”  
3. “**Tiered retention**.”

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

From the **service** perspective: emit **logs/metrics/traces** via **OTel** / agents → **non-blocking** local buffer → forward to **Kafka** → stream processors **parse/enrich/sample** → **TSDB + log store** → **dashboards / ad-hoc query**.

From the **on-call** perspective: page breaks → **query** SLIs and **trace** exemplars → drill to **logs**—**never** block the user request on Elasticsearch.

So:

- **write path** = **fire-and-forget** to local agent; **at-least-once** through the bus with **dedupe keys** downstream.
- **read path** = **Grafana/PromQL**, log UI—**cold** tiers slower.
- **async path** = aggregations, sampling decisions, **PII scrub**, tiered retention moves.

## Consistency model

**At-least-once** ingest is fine with **idempotent** sinks / keys.

**PII** never lands **unredacted** in **immutable** cold tiers; **high-cardinality** labels **blocked at ingest**—**§1.4 invariant**.

**Query** results can be **eventual**; **SLI correctness** is about **math on samples** you defend honestly.

## Commit boundary

The **service request** commits without waiting for **off-cluster** indexing; **agent ack** means buffered to **durable local** spool or forwarded—define which.

**Downstream** “log line is queryable” is **async commit** with **bounded lag** SLO.

## Decision (strong opinion)

I’d start with:

- **Kafka buffer** + **stream processors** → **TSDB** (metrics) + **log store** (debug), shared **enrichment** pipeline.

because **three signals** share **enrichment** but must not share the same **cardinality** failure mode unguarded.

## Evolution

| Phase | Say it like this |
|-------|------------------|
| **1** | Simple implementation that ships. |
| **2** | Scaling: partitions, caches, queues, backpressure, observability. |
| **3** | Advanced / ML / global—only when metrics or product force it. |

Details: **Section 4.1 (phases)** and **Section 5** in this file.

## Bottleneck anchor

Watch first:

- **Kafka** partition hot spots, **ingest** CPU on parsers.
- **cardinality** explosions (metrics bankruptcy).

## Backpressure handling

Under load:

- **drop/sample** tail logs first; **aggregate** metrics earlier.
- **block** new high-card labels at edge.

Goal: **keep the product hot path alive** over **complete** raw logs forever.

## UX awareness

Bad outcomes:

- **on-call** can’t answer “what broke?” because logs missing or **PII** redacted too aggressively without policy.
- dashboards **lie** because sampling wasn’t documented.

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

**Verbatim:** *“We’re talking **PB/day** logs and **billions** of metric samples—so partitioning, compaction, and cardinality policy are first-class design, not an afterthought.”*

| Dimension | Illustrative |
|-----------|----------------|
| Log volume | **PB-scale** discussion for big cos |
| Metric samples | **Billions**/day |

---

## 3. APIs and data model

<a id="say-voice-3"></a>

### 3.0 Core entities (who owns what — say before interfaces)

| Entity | Owns / lifecycle (one line) |
|--------|-----------------------------|
| **LogRecord / Metric sample** | Immutable event with **bounded** labels + **trace_id**. |
| **Collector** | Authenticate + batch + backpressure. |
| **Stream partition** | Kafka ordering unit—**hash** on service/hour not raw user for metrics. |

#### Human interaction (API design)

**Verbatim:** *“Push via agents, pull via Prometheus scrape for metrics—different paths, same **enrichment** layer, same **PII** rules.”*

### 3.1 Interfaces

- **Push:** agents **HTTP/gRPC** to **collector**.  
- **Pull:** Prometheus **scrape** (metrics).  

### 3.2 Schema

- **LogRecord:** `ts, service, level, trace_id, message, attrs`.  
- **Metric:** `name, labels{bounded}, value, ts`.

---

## 4. High-level architecture

<a id="say-voice-4"></a>

#### Human interaction (high-level architecture / HLD)

**Verbatim:** *“One sentence: services talk to local agents **non-blocking**, agents batch into Kafka, processors scrub and route into **TSDB for numbers** and **log index for strings**, and everything is correlated with **trace_id**.”*

```mermaid
flowchart LR
  SVC[Services]
  AG[Agents / OTel]
  COL[Collectors]
  K[Kafka]
  P[Processors]
  TSDB[(TSDB)]
  LOG[(Log store)]
  SVC --> AG --> COL --> K --> P
  P --> TSDB
  P --> LOG
```

### 4.1 Phases

| Phase | Ship |
|-------|------|
| **1** | ELK or Loki + Prometheus |
| **2** | **Sampling** + **SLO dashboards** |
| **3** | **Tiered** + **federated** query |

---

## 5. Deep dive: hot path → durable

<a id="say-voice-5"></a>

#### Human interaction (deep dive — critical flow, optimizations & evolution)

**Verbatim:** *“Hot path is: emit structured log to agent buffer, flush batch to Kafka, processor consumes, scrubs PII, writes index chunk; if Kafka lags during an incident, we shed **debug** tiers before we drop **RED** metrics.”*

**Verbatim (evolution):** *“**v1** basic pipeline; **v2** tail sampling for traces + SLO burn alerts; **v3** tiered cold storage with export controls.”*

<a id="bottleneck-anchor-once"></a>
### 🎯 Bottleneck Anchor

“**Kafka** **consumer lag** during incidents—**back-pressure** and **drop** **debug** verbosity tiers.”

```mermaid
sequenceDiagram
  participant S as Service
  participant A as Agent
  participant K as Kafka
  participant P as Processor
  S->>A: emit log (non-blocking)
  A->>K: produce batch
  P->>K: consume
  P->>P: scrub PII / parse
  P->>LOG: index chunk
```

**Taking a stance:** *“**Structured JSON** logs + **trace_id** correlation—**grep**-hostile unstructured **deprecated**.”*

---

## 6. Scaling and bottlenecks

#### Human interaction (scaling & bottlenecks)

**Verbatim:** *“Cardinality explosion is the silent killer—**allowlist** metric labels, aggregate early, and partition Kafka to avoid **hot partitions** from putting **user_id** in the metric key.”*

| Risk | Mitigation |
|------|------------|
| **Cardinality explosion** | **Allowlist** labels; **aggregate** early |
| **Hot partition** | **Hash** key on `(service, hour)` not **user_id** for metrics |

---

## 7. Reliability and failure handling

#### Human interaction (reliability & failure handling)

**Verbatim:** *“Agent disk full means **spill** and **shed** lowest priority; Kafka down means bounded local buffer—**never** infinite memory pretending to be reliability.”*

- **Agent disk full:** **spill** + **shed** lowest priority.  
- **Kafka down:** **buffer** locally with **caps**—**never** infinite.

---

## 8. Tradeoffs and alternatives

#### Human interaction (tradeoffs & alternatives)

**Verbatim:** *“Full fidelity logs are amazing until the bill arrives—**sampling** and **structured** logs are how you keep debuggability without bankrupting storage.”*

| Choice | Trade |
|--------|--------|
| **Full fidelity logs** | Debuggability vs **cost** |
| **Head sampling traces** | Cost vs **coverage** |

---

## 9. Monitoring, observability, and security

#### Human interaction (monitoring, observability & security)

**Verbatim:** *“Meta-observability: monitor **ingest lag**, **drop rate**, **parse errors**; security is **RBAC** on queries and audited exports because logs are sensitive.”*

**Meta:** monitor the **pipeline** lag, **drop rate**, **parse errors**.  
**Security:** **RBAC** on log query; **audit** exports.

---

## 10. Design patterns, data structures & best practices

<a id="say-voice-10"></a>

#### Human interaction (design patterns, data structures & best practices)

**Verbatim (say 5–6 on the board):** *“**Sidecar/agent buffering**, **Kafka as durable buffer**, **CQRS** between OLTP and observability stores, **schema-on-write** for metrics, **tail sampling** for traces, **multi-tier retention**, and **circuit breaking** noisy clients that spam logs.”*

| Pattern / DS | Where | One interview line |
|----------------|------|----------------------|
| **Async agent + ring buffer** | Node | “Never block request thread on network I/O.” |
| **Kafka partitions** | Buffer | “Durability + replay; pick keys to avoid hot partitions.” |
| **CQRS** | OLTP vs TSDB/log | “Different models for different questions.” |
| **Stream processor** | Flink/Beam | “Scrub PII + normalize schema before index.” |
| **Cardinality policy / label allowlist** | Ingest | “No `user_id` as a metric label unless you want pain.” |
| **Tiered storage (hot/warm/cold)** | Object store | “Cost control with lifecycle policies.” |

---

## Closing notes

<a id="communication-do-vs-avoid"></a>

#### Human interaction (closing notes)

**Verbatim:** *“Non-blocking emit, Kafka buffer, processors enforce **PII + cardinality**, TSDB for SLIs, log store for debug, and we monitor the **pipeline** itself.”*

| Do | Avoid |
|----|--------|
| **Structured logs** | printf in hot path sync |
| **Cardinality policy** | `user_id` as metric label |

---

## Bar-raiser follow-ups

#### Human interaction (bar-raiser follow-ups)

**Verbatim:** *“If you want depth: **eBPF** node metrics vs **app** business events, or **GDPR** delete in immutable logs—pick one.”*

| They ask | Say it like this |
|----------|------------------|
| **eBPF** | “Kernel-level **no-instrument** metrics—**great** for **node** health, not **biz** events.” |

---

## 60-second close

#### Human interaction (60-second close)

**Verbatim:** *“**OTel-shaped** ingest, **Kafka** buffer, **PII + cardinality** gates, **TSDB + logs**, monitor **lag/drops**, sample traces under cost.”*

| Beat | Say it like this |
|------|------------------|
| **Recap** | “**Non-blocking** emit → **Kafka** → **processors** → **TSDB + logs**; **PII** + **cardinality** gates; **lag** SLI on pipeline; **OTel**-shaped mental model.” |

---
