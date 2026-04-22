# HLD — WhatsApp Web / Messaging

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

### 1.0 Live flow (how to open and steer)

<a id="live-flow-open"></a>

#### Live voice (real interviewer room)

**Sound like you’re *deciding*, not reciting:** one idea per breath, then **pause**. Tables here are **backup**—if your eyes are down for more than a couple of seconds, you’ve slipped into reading the doc.

**Bridge phrases (mix naturally):** *“Let me **name the fork** first…”* · *“I’ll **default to X**—tell me if your bar is stricter.”* · *“The reason I ask is it changes **who owns the commit** / **what’s on the hot path**.”* · *“I’ll **over-answer** one layer, then stop—**where should I zoom**?”*

**Ping them (conversation, not monologue):** *“Does that match how you’d scope it?”* · *“If we only deep-dive one thing, is it **A** or **B**?”* (swap **A/B** for two tensions from *your* opening paragraph above.)

**This topic in one breath:** “Messaging is **per-conversation order**, **dedupe**, and **push**—E2E is a **scope fork**, I’ll ask before I pretend.”

**`Verbatim` / `Live` cues:** say a line **once**, then **rephrase** the next time—verbatim twice in a row reads *canned*.

**Opening (~once):** *“I’ll align on **E2E vs server-visible**, **1:1 vs groups**, **retention**; then **scale**, **APIs + storage**, **architecture**, and **send message** end-to-end. I’ll **pause after the diagram**—depth on **ordering**, **fan-out**, or **WS reliability**?”*

**Thinking transitions:** *“Let me think through …”* · *“The ordering contract is …”* · *“If durability is non-negotiable I’d …”* · *“Let me sanity-check …”* · *“Hot chat means …”*

**Live rule:** **Paraphrase** §1–2 tables; don’t read every row. Go deep **only if they probe**.

<a id="say-1-questions-human"></a>
### 1.1 Clarify 

| Topic | Say it like this in the room |
|--------------------------|-------------------------------|
| **Trust model** | “Are we designing **E2E**—or **TLS to server** so the server can index and moderate?” |
| **Chat shape** | “**1:1 only** or **groups**—and what’s the max **group** size for fan-out?” |
| **Retention** | “**Retention**, legal hold, **delete**—what does the product promise?” |
| **Multi-device** | “**Independent** sessions per device vs **phone-tethered** Web?” |
| **Features** | “**Read receipts**, **typing**, **presence**—in scope or nice-to-have?” |
| **Media** | “Max **payload** size, malware scan—anything that changes upload path?” |

**Micro-pauses:** *“So **ordering is per chat** via **server seq**, and **`client_msg_id`** handles dedupe.”*

#### Human interaction (clarify requirements — think out loud & evolve scope)

**Habit:** *“Messaging is **ordering + delivery + presence**—I separate **what must be linear** from what can be **best-effort**.”*

**Live:** *“Do we need **E2E encryption** in scope? **Multi-device** per user? **Groups** vs **1:1 only**? Those three change **fan-out** and **keying**.”*

| Stage | Assume | Evolve when… |
|-------|--------|----------------|
| **v1** | **Server-assigned seq**, **at-least-once** ingest, **WS** + **push** | Product needs **receipts** |
| **v2** | **Presence**, **typing**, **media** offload to blob | UX richness |
| **v3** | **Sharding** by `chat_id`, **CRDT**-style edits (if docs) | Extreme group size |

### 1.2 Functional requirements (FR) — after alignment, say this as “what we must build”

<a id="say-fr-human"></a>
#### Human interaction (FR — how to explain after alignment)

**Habit:** *“**Send**, **order**, **sync**—three beats.”*

**Live:** one **spoken** FR pass (~60–90 s); use [§1.0](#live-flow-open) when you move **FR → NFR**.

| FR area | Say it like this in the room |
|---------|-------------------------------|
| **Messaging** | “Send/receive in a **chat**; **server** assigns **monotonic `server_seq`** per chat.” |
| **History** | “**REST** (or similar) **catch-up** with `after_seq`; all devices converge on **server order**.” |
| **Media** | “**Pre-signed URL** upload; message holds **handle + metadata**.” |
| **States** | “Delivery / read (if in scope); **typing** as **ephemeral**, best-effort.” |

**Messaging**

- Send/receive **messages** in a **chat** (1:1 or group).  
- **Server-assigned** monotonic **sequence** per chat for total order of delivery.  
- **Delivery** and **read** states (optional **typing**).

**History and sync**

- **Fetch history** with cursor (`after_seq`); **multi-device** convergence to same order.  
- **Offline:** persist locally; **push** notification when offline (product dependent).

**Media**

- Upload via **pre-signed URL** to object store; message references **handle + metadata**.

**Account / safety (high level)**

- Block/report flows if asked; **delete message** / **tombstone** semantics.

### 1.3 Non-functional requirements (NFR) — say as “how it must behave”

<a id="say-nfr-human"></a>
#### Human interaction (NFR — how to say “how it must behave”)

**Habit:** *“**Durability before ACK**; **at-least-once** with a dedupe story.”*

| NFR area | Say it like this in the room |
|----------|-------------------------------|
| **Durability** | “I only **ACK** after the message is **durably** committed—or I’m explicit about WAL tradeoffs.” |
| **Ordering** | “**Total order per `chat_id`**—I don’t pretend **global** order across chats.” |
| **Scale** | “**Billions/day** ⇒ **shard by chat_id**; expect **hot chats**.” |
| **Availability** | “**Typing** can drop before **messages** under pressure.” |
| **Security** | “**AuthZ** per chat; **rate limits** on send.” |

#### UX on the wire (say with NFR)

- **Slow send:** show **pending** state; **retry** with same **`client_msg_id`**—never two bubbles for one intent.  
- **Reconnect:** **resume** from last **ACK’d seq**—user sees **gap-free** merge, not dupes.  
- **Typing / presence sick:** **drop** ephemeral signals before **message** delivery degrades.

**Latency**

- Low latency for **online** delivery; typing can be **best-effort**.

**Durability**

- Messages **durable** after ACK policy you state (commit before ACK).

**Scalability**

- **Billions** of messages/day → **shard by chat_id**; horizontal gateways and consumers.

**Ordering**

- **Total order** per `chat_id` for what users see as “the thread.”

**Availability**

- Gateways and chat service **HA**; **degrade** typing before dropping messages.

**Security**

- **AuthN** on WS; **authZ** per chat membership; **rate limits**; abuse controls.

<a id="key-insight-say-early"></a>
### 1.4 Invariants (one sentence you repeat under pressure)

**Invariant:** “For a given chat, **server-assigned order** is the **canonical** order for delivery; clients **dedupe** using **`client_msg_id`** (or equivalent).”

#### Key anchors (say these confidently—any order)

1. “**Durability before ACK** (or explicit WAL tradeoff you name).”  
2. “**Total order per `chat_id`**—not global order across chats.”  
3. “**At-least-once** fan-out + **idempotent** server + **client dedupe**.”  
4. “**Shard by `chat_id`**; **hot chat** = **partition skew** or **fan-out queue backlog** first—**materialized inbox** only if that is the proven bottleneck.”  
5. “**Catch-up** via **`after_seq`**—all devices **converge**.”  
6. “**User journey** (say once)—[types → sends → ACK → fan-out → `after_seq`](#user-journey-framing); **write** = durability + order, **read** = convergence.”

<a id="say-voice-1"></a>

**Purpose:** handoff → **persist → seq → fan-out** diagram.

| Beat | Say it like this |
|------|------------------|
| **Bridge** | “**Effectively-once**: **at-least-once** transport + **idempotent** insert on `client_msg_id` + UI dedupe on **`server_seq`**.” |
| **Hot chat** | “**Queue-based fan-out** first; **materialized inbox** only if **hot chat** or **fan-out cost** becomes the bottleneck.” |

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

From the user’s perspective: open chat → type → **send**; server assigns **`server_seq`**; **online** peers get **WS push**; **offline** gets **APNs/FCM**; reconnect → **`after_seq` catch-up**; optional **media** via pre-signed upload.

So:

- **write path** = authenticate → **dedupe** (`client_msg_id`) → **durable commit** → assign seq → **fan-out** to recipients.
- **read path** = **history pages** + live stream—**canonical order** is always **per `chat_id`** server order.
- **async path** = push delivery receipts, **typing/presence** (ephemeral), moderation hooks—**best-effort** before messages.

## Consistency model

**Non-negotiable**:

- **total order per chat** via monotonic **`server_seq`** (not global order across chats).
- **at-least-once** + **idempotent** server + **client dedupe** → **effectively-once** UX.

**Best-effort** is OK for:

- **typing**, **presence**, some delivery hints—**drop** these before you drop **message** durability.

**E2E vs server-visible** is a **scope fork**—if E2E, you still need **ordering + device sync** story without pretending the server “sees” plaintext.

## Commit boundary

You **ACK** the send only after:

- the message is **durably** stored (WAL/DB policy you name)—or you explicitly defend a riskier window.

Until then the UI stays **pending**; **`client_msg_id`** replays must return the **same** `server_seq`, never duplicate bubbles.

## Decision (strong opinion)

I’d start with:

- **shard by `chat_id`**, **queue-based fan-out** for groups, **WS + mobile push** for delivery.
- **pre-signed object store** for media; message row holds **handle + metadata** only.

because **durability + per-chat order** are the spine; fancy inbox materialization is **optional** until hot-chat metrics force it.

If **fan-out cost** or **mega-groups** bite:

- **materialized inbox**, **partition** consumers, read-path **pagination** hardening.

## Evolution

| Phase | Say it like this |
|-------|------------------|
| **1** | Simple implementation that ships. |
| **2** | Scaling: partitions, caches, queues, backpressure, observability. |
| **3** | Advanced / ML / global—only when metrics or product force it. |

Details: **Section 4.1 (phases)** and **Section 5** in this file.

## Bottleneck anchor

Watch first:

- **hot `chat_id`** (fan-out backlog, partition skew).
- **gateway + WS** connection churn and **reconnect storms**.

## Backpressure handling

Under load:

- **shed typing/presence** first; keep **message ingest** and **seq assignment** healthy.
- **throttle** sends per user/chat; **queue** fan-out rather than **OOM** the broadcaster.

Goal: **never duplicate** user intent; **degrade ephemerals** before **durability**.

## UX awareness

Bad outcomes:

- **dup messages** on retry without **`client_msg_id`** discipline.
- **gaps** or wrong order on reconnect without **`after_seq`** contract.
- push storms waking users for **low-signal** events—**rate** and batch.

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

**Habit:** *“**Billions/day** is the mental default—tune down if they want.”*

**Live:** *“Let me sanity-check scale…”* then **msgs/day**, **shard key**, **hot chat**—**invite correction** before the dimension table.

| Topic | Say it like this in the room |
|-------|-------------------------------|
| **Shard** | “Writes and storage keyed by **`chat_id`**.” |
| **Hot chat** | “Viral thread ⇒ **partition skew** or **fan-out queue backlog** first—**materialized inbox** only if metrics prove we need it.” |
| **Read path** | “History pulls plus live—**catch-up** must be **keyset** by seq.” |

| Dimension | Illustrative |
|-----------|----------------|
| Messages/day | **Billions** at WhatsApp scale (tune down if interviewer wants) |
| Shard key | **`chat_id`** for writes and storage |
| Hot chat | World Cup / viral → **rate limit**, **sub-queue**, then **materialized inbox** only if **queue** path breaks |
| Read:write | History pulls + live mix |

**Tie it in one line:** “**Partition by chat**; optimize **send + catch-up**; plan for **hot partition** explicitly.”

---

## 3. APIs and data model

<a id="say-voice-3"></a>

### 3.0 Core entities (who owns what — say before API tables)

| Entity | Owns / lifecycle (one line) |
|--------|-----------------------------|
| **User / Device** | Sessions, **WS** presence; **AuthN** at connect. |
| **Chat** | Membership roster; **shard** soul for writes. |
| **Message** | **Immutable** row keyed by **`(chat_id, server_seq)`**; body or **blob ref**. |
| **Timeline / Inbox pointer** | Per-user **read cursor** + **delivery state** (can be **separate** store). |
| **Media object** | **Pre-signed** upload; **virus scan** async; **metadata** on message. |

#### Human interaction (APIs & data model — API design + contracts)

**Habit:** *“**WS** for live; **REST** for history; **keys** are `(chat_id, server_seq)`.”*

| Topic | Say it like this in the room |
|-------|-------------------------------|
| **APIs** | “**Connect** over WS; **SendMessage** with **`client_msg_id`**; **GET** messages with **`after_seq`**.” |
| **Model** | “Message row is **`chat_id` + `server_seq`**; device tracks **last ack’d seq**.” |
| **Core split (once)** | Same as [Key insight / invariant](#key-insight-say-early)—**canonical server order** + **client dedupe**. |

### 3.1 APIs (sketch)

| API | Purpose |
|-----|---------|
| `WS /v1/connect` | Authenticate; heartbeat |
| `SendMessage(chat_id, client_msg_id, body)` | Over WS or gRPC from GW |
| `GET /v1/chats/{id}/messages?after_seq=&limit=` | History / catch-up |
| `POST /v1/media/upload-url` | Pre-signed upload |

### 3.2 Data model

**Message:** `(chat_id, server_seq, client_msg_id, sender_id, body_ref, ts, flags)` — **primary key** `(chat_id, server_seq)` or partition + ordering key.

**Chat / membership:** `chat_id`, members, roles.

**Device state:** `last_delivered_seq`, `last_read_seq` per `(user, chat, device)`.

---

## 👤 User journey (say once)

<a id="user-journey-framing"></a>

**Say it once** (near the [architecture diagram](#4-high-level-architecture))—**product**, not only boxes:

*“User **types** a message → **sends** → gets **ACK** (after **durable** persist) → recipients receive via **fan-out** → **offline** users **catch up** via **`after_seq`**.

So:
- **write path** ensures **durability** + **ordering**  
- **read path** ensures **convergence** across devices.”*

👉 **Intuitive**, **product-aware**, then point at **persist → seq → queue → push** on the board.

---

## 4. High-level architecture

<a id="say-voice-4"></a>
#### Human interaction (high-level architecture / HLD)

**Habit:** *“**Sticky gateways** (or registry), **stateless-ish chat service**, **queue** closes the loop.”*

| Moment | Say it like this in the room |
|--------|------------------------------|
| **Path** | “Client → **LB** → **WS gateway** → **chat service** → **DB** by **`chat_id`**.” |
| **User journey** | “Same line as [👤 User journey (say once)](#user-journey-framing): **ACK → fan-out → `after_seq`**.” |
| **Fan-out** | “After commit, enqueue **deliver_to_recipients**; queue pushes back to **gateways**.” |
| **Registry** | “**Redis** maps user → gateway for the right **push** box.” |
| **Steer** | “**Deeper** on **persist/ACK**, **fan-out queue**, or **reconnect + replay** next?” |

```mermaid
flowchart TB
  Web[Web / Mobile]
  LB[LB]
  GW[WebSocket Gateway]
  Chat[Chat Service]
  W[Message Writers]
  DB[(Message store by chat_id)]
  Redis[(Registry / presence)]
  Q[(Internal queue / Kafka)]
  Push[Push service]
  Web --> LB --> GW
  GW --> Chat
  Chat --> W --> DB
  Chat --> Q --> GW
  Chat --> Push
  GW --> Redis
```

**Media:** pre-signed **object storage**; message stores **pointer** only.

### 4.1 How we’d evolve this (if they ask “phases / MVP”)

| Phase | Ship | Why |
|-------|------|-----|
| **1 — MVP** | **Single-region**, **WS + REST catch-up**, **idempotent** send, **simple** fan-out queue | Prove ordering + dedupe story |
| **2 — Growth** | **Registry**, **hot-chat** limits, **media** pre-signed path, **DLQ** hygiene | Reliability at scale |
| **3 — Scale** | **Partitioned** queue fan-out; **materialized inbox** only if **hot chat** or **fan-out cost** forces it; **multi-region** **leader per chat** | Tail + DR |

**Taking a stance:** *“I’d default **commit-then-ACK** with **at-least-once** delivery and **explicit** client dedupe—**exactly-once** is a **product illusion**, not a wire guarantee.”*

---

## 5. Deep dive: critical flow

<a id="say-voice-5"></a>
#### Human interaction (deep dive — critical flow, optimizations & evolution)

**Live (evolution):** *“**Default**: **append** message → **assign seq** → **fan-out** pointers to recipients’ timelines. **Evolve**: **gossip** read receipts batched, **edge** connection coalescing, **hot chat** isolation shard.”*

**Habit:** *“Trace **SendMessage** like the sequence diagram—no ACK fairy tales.”*

| Step | Say it like this in the room |
|------|-------------------------------|
| **Persist** | “**Idempotent** insert on **`client_msg_id`**; allocate **`server_seq`**.” |
| **ACK** | “Return **ACK** only after **durable** commit (or say WAL explicitly).” |
| **Deliver** | “Enqueue fan-out; **at-least-once** to gateways—clients **dedupe**.” |
| **Anchor** | “Say **once**—[🎯 Bottleneck Anchor](#bottleneck-anchor-once).” |
| **Production voice** | “**GW crash** mid-send—client **retries** same **`client_msg_id`**; **queue backlog**—**shed typing**; **split brain** writer—**fencing** token if they push multi-region.” |

This is **step 5** of the [spine](#interview-spine-nine-steps)—where most Bar Raiser time should go.

<a id="bottleneck-anchor-once"></a>
### 🎯 Bottleneck Anchor

**Say once in the deep dive:**

The main bottleneck here is **usually**:

- **hot chat** causing **partition skew**  
- **or** **fan-out queue backlog**

*That’s what I’d **monitor first**.*

👉 **Prioritization**—then **send p99**, **duplicate rate**, and **GW disconnects** as supporting proof.

**Taking a stance:** *“**I’d start with queue-based fan-out**, and **only move to materialized inbox** if **hot chat** or **fan-out cost** becomes a **bottleneck**.”* *“With **sticky** gateways, **`chat_id`** partitioning, **idempotent** persist, and **ACK after durable commit**.”*

### 5.1 Send message (sequence)

```mermaid
sequenceDiagram
  participant C as Client
  participant G as WS Gateway
  participant S as Chat Service
  participant D as DB
  participant Q as Fan-out queue
  C->>G: SendMessage(chat_id, client_msg_id, body)
  G->>S: RPC (sticky route)
  S->>D: idempotent insert + next server_seq
  alt duplicate client_msg_id
    D-->>S: return existing
  end
  S-->>G: ACK(server_seq)
  G-->>C: ack
  S->>Q: deliver_to_recipients(chat_id, server_seq)
  Q-->>G: push to online recipients
```

**Say:** “ACK after **durable** commit (or state your WAL/fsync tradeoff).”

### 5.2 WebSockets and sessions

- **Sticky** to gateway or **Redis registry** `user → gateway`.  
- **Heartbeat**; **reconnect** with backoff; **session takeover** with resume token.

### 5.3 Delivery and multi-device

- **At-least-once** delivery; **dedupe** on `(chat_id, client_msg_id)` or `server_seq`.  
- **Catch-up:** `GET …?after_seq=`; all devices **converge** on server order.

---

## 6. Scaling and bottlenecks

<a id="say-voice-6"></a>
#### Human interaction (scaling & bottlenecks)

**Habit:** *“**Hot chat** partition is the headline risk.”*

| Topic | Say it like this in the room |
|-------|-------------------------------|
| **Hot chat** | “**Rate limit** + **sub-queue** on **queue fan-out** first; **materialized inbox** only when that path is **still** the bottleneck.” |
| **Gateways** | “**Horizontal** replicas; **connection** limits per box.” |
| **Queue** | “Scale consumers; **shed typing** before messages.” |

| Risk | Mitigation |
|------|------------|
| **Hot chat** partition | Rate limit; internal sharding of fan-out; **materialized inbox** only if **queue backlog** / cost still unacceptable |
| **Gateway** connection limit | Many nodes; **DRY** connection routing |
| Queue backlog | Scale consumers; **shed** typing |
| Storage size | **Tiering** cold chats to cheaper store; compaction |

**Optimizations:** batch small messages; **compress**; **keyset** pagination; typing **non-durable** path.

---

## 7. Reliability and failure handling

<a id="say-voice-7"></a>
#### Human interaction (reliability & failure handling)

**Habit:** *“**Reconnect + replay**; **DLQ** for poison.”*

| Topic | Say it like this in the room |
|-------|-------------------------------|
| **GW crash** | “Client **reconnects**, **replays** from last **ACK’d seq**.” |
| **Dupes** | “Server **idempotent** insert; client **dedupe** on `server_seq`.” |
| **Multi-region** | “**Leader per chat** or eat latency; **fencing** if failover writers.” |
| **Incident tone** | “**Duplicate bubbles** after reconnect—**idempotent** insert saves you; **one hot partition**—**rate limit** + **sub-queue**; **provider** push outage—**queue depth** + **DLQ**.” |

**UX tie-in (say aloud):** *“**Pending** → **sent** states; **ordering** matches server seq; **typing** drops before **messages** under load.”*

- **Gateway crash:** client **reconnect**, **replay** from last **ACK’d seq**.  
- **Duplicate delivery:** idempotent UI + server dedupe on `client_msg_id`.  
- **Partial fan-out failure:** retry with backoff; **DLQ** for poison.  
- **Multi-region (short):** **leader per chat** or higher latency global consistency; **fencing** for writer failover if needed.

---

## 8. Tradeoffs and alternatives

<a id="say-voice-8"></a>
#### Human interaction (tradeoffs & alternatives)

**Habit:** *“**Push fan-out** vs **inbox materialization**—storage vs write complexity.”*

| Topic | Say it like this in the room |
|-------|-------------------------------|
| **Ordering** | “Strong per-chat order is simple; cost is **hot shard**.” |
| **Inbox** | “**Materialized** per-user inbox—**fast read home**, **write amplification**; **I’d start** with **queue fan-out** and **only** add this if it becomes the **bottleneck**.” |
| **My default (ordering)** | “**Server seq per chat** + **client_msg_id** dedupe—simple story under pressure.” |
| **My default (fan-out)** | “**I’d start with queue-based fan-out**, and **only move to materialized inbox** if **hot chat** or **fan-out cost** becomes a **bottleneck**.” |

| Choice | Upside | Downside |
|--------|--------|----------|
| Strong per-chat order | Simple mental model | Hot shard |
| Per-user inbox materialization | Read cheap | Write amplification |
| Long poll vs WS | Simpler infra | Higher latency, more requests |

**Alternatives:** **gRPC streaming** instead of WS; **CRDT** for limited metadata (not full message body) if interviewer goes there.

---

## 9. Monitoring, observability, and security

<a id="say-voice-9"></a>
#### Human interaction (monitoring, observability & security)

**Habit:** *“Trace **send → persist → deliver**.”*

| Topic | Say it like this in the room |
|-------|-------------------------------|
| **SLIs** | “Send **p99**, delivery **lag**, **disconnect** rate, queue **depth**.” |
| **Security** | “TLS; **token** scoped to chat; **GDPR delete** = tombstone + async scrub.” |

**SLIs:** send **p99**, delivery lag **p99**, WS **disconnect rate**, queue **depth**, DB write errors.

**Security:** TLS; **token** scoped to chat; **rate limit** sends; **content abuse** pipeline (async); **GDPR delete** = tombstone + async scrub from object store.

**Tracing:** span `send → persist → enqueue → push`.

---

## 10. Design patterns, data structures & best practices

Say **where** each pattern lives—registry, outbox, idempotent consumer—not a laundry list.

### 10.1 Distributed / real-time patterns

| Pattern | Where | Why |
|---------|--------|-----|
| **Connection registry** | Gateway ↔ user session | Route push to correct box |
| **Back-pressure** | WS send path | Avoid OOM when client slow |
| **Circuit breaker** | Downstream DB / push | Fail fast; shed load |
| **Bulkhead** | Separate pools for **send** vs **presence** | One path cannot starve the other |
| **Outbox / WAL** | After DB commit enqueue delivery | No lost messages on crash |
| **Idempotent consumer** | Delivery worker | At-least-once safe |
| **Leader election** | Per-chat writer (optional) | Ordering + failover without split brain |

### 10.2 Classic patterns

| Pattern | Map |
|---------|-----|
| **State machine** | Message: accepted → persisted → delivered → read (optional) |
| **Command** | `SendMessage` handler isolates validation + persist |
| **Observer** | Presence / typing fan-out (careful: rate limit) |
| **Strategy** | Long-poll vs WS vs SSE per client capability |

### 10.3 Data structures

| Need | Structure |
|------|-----------|
| Per-chat order | Monotonic **server_seq** (bigint) |
| Dedupe | **Set** or DB unique on `(chat_id, client_msg_id)` |
| Unread counts | **Counter** per user+chat or materialized inbox row |
| Hot fan-out | **Partitioned** outbox queue by `chat_id` |

### 10.4 Best practices

- **Persist before push**; ack only after durable write.  
- **Client idempotency** + server sequence for ordering UI.  
- **Pagination** keyset on `(server_seq)` not offset.

### 10.5 Trade-offs

| Pick | Trade |
|------|--------|
| Per-user inbox materialization | Fast read home vs **write amplification** |
| Pull (catch-up API) | Simple recovery vs **latency** vs push |

<a id="say-voice-10"></a>
#### Human interaction (design patterns, data structures & best practices)

**Habit:** *“**Registry**, **outbox/WAL**, **idempotent consumer**, **circuit breaker**—one line each.”*

**Verbatim (drive the room in ~40s):** *“**Connection registry** maps user to gateway for push; **outbox or WAL** after DB commit so we never enqueue a message we didn’t persist; **idempotent delivery workers** for at-least-once; **circuit breaker** and **bulkhead** on DB vs push path; **state machine** on message lifecycle; **Strategy** for WS vs long-poll; **`(chat_id, server_seq)`** ordering with **`client_msg_id`** dedupe; **keyset** pagination for history catch-up.”*

**Live:** **at most four** patterns tied to boxes; then stop.

| You mean… | Say it like this in the room |
|-----------|-------------------------------|
| **Patterns** | “**Connection registry** routes push; **outbox** after commit; **breaker** on flaky deps; **bulkhead** typing vs send.” |
| **DS** | “**`(chat_id, server_seq)`** key; dedupe set on **`client_msg_id`**; **keyset** history.” |

---

## Closing notes (where wrap-up human interaction lives)

Use **`#### Human interaction`** under [Bar-raiser](#bar-raiser-follow-ups), [Communication (do vs avoid)](#communication-do-vs-avoid), and [60-second close](#60-second-close).

<a id="communication-do-vs-avoid"></a>
### Communication (do vs avoid)

| Do (sounds senior) | Avoid (sounds rehearsed) |
|--------------------|---------------------------|
| **Name ACK semantics** clearly | “We’re durable” with no commit point |
| **Checkpoint** after diagram | One monologue through WS details |
| **Default + caveat** (queue fan-out first; materialized inbox if bottleneck) | Listing every transport |
| **Invite** hot-chat depth | Assuming groups are always small |
| **Time-box** | Reading the whole doc aloud |

**60-minute sketch (flex):** clarify+FR+NFR ~8–12 · scale+APIs ~8–12 · architecture ~8–12 · **deep dive ~15–22** · scale→monitoring ~10–15 · patterns+close ~5–8.

---

## Bar-raiser follow-ups

<a id="say-voice-bar"></a>
#### Human interaction (bar-raiser)

**Habit:** two–four sentences, then **stop**.

| They ask | Say it like this |
|----------|------------------|
| **Exactly-once?** | “**Effectively-once**: at-least-once + **idempotent** writes + client **dedupe** on **`server_seq`**.” |
| **Cross-region?** | “**Leader region per chat** or accept latency; **consistent routing**.” |

---

## 60-second close

<a id="say-voice-close"></a>
#### Human interaction (60-second close)

**Habit:** one **net-net** pass.

| Beat | Say it like this in the room |
|------|------------------------------|
| **Recap** | “**WS gateways** + **registry**; **durable** write then **`server_seq`**; **ACK**; **queue-based fan-out** first; **at-least-once** + **`client_msg_id`** dedupe; **catch-up** by **`after_seq`**; **hot chat** → **skew** or **queue backlog** first—**materialized inbox** only if that’s the bottleneck.” |

---
