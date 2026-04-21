# HLD — Push Notification Service

<a id="interview-spine-nine-steps"></a>

## 1. Clarify requirements

### 1.0 Live flow

<a id="live-flow-open"></a>

**Opening (~once):** *“I’ll align **transactional vs marketing**, **device tokens**, **priority**, and **delivery guarantees**; then **API**, **queue**, **provider adapters** (APNs/FCM). **Pause after the diagram**—**fan-out**, **privacy**, or **throttling**?”*

**Thinking transitions:** *“Push is **at-most-once** from the provider’s POV—**idempotency** at **our** boundary for **transactional**.”*

<a id="say-1-questions-human"></a>
### 1.1 Clarify

| Topic | Say it like this in the room |
|--------------------------|-------------------------------|
| **Volume** | “**Marketing blast** vs **trip updates**?” |
| **Personalization** | “Deep links + **payload** size limits?” |
| **Quiet hours** | “**Compliance** by locale?” |
| **Web** | “**Web Push** in scope?” |

### 1.2 Functional requirements (FR)

<a id="say-fr-human"></a>

| FR area | Say it like this |
|---------|-------------------|
| **Register** | “Device **token** per **user** + **platform**.” |
| **Send** | “Single user, **topic**, or **segment**.” |
| **Track** | “Delivered/opened **if** OS provides callbacks.” |

**Cross-ref:** Stock-style alerts — [16-hld-notification-stock-alerts.md](./16-hld-notification-stock-alerts.md); email/SMS — [35-hld-email-sms-notification.md](./35-hld-email-sms-notification.md).

### 1.3 Non-functional requirements (NFR)

| NFR | Say it like this |
|-----|------------------|
| **Latency** | “**Transactional** **seconds** class; **marketing** **minutes** ok.” |
| **Reliability** | “**DLQ** + **retry** with backoff for provider **5xx**.” |

### 1.4 Invariants

**Invariant:** “A **transactional** notification (e.g. **trip assigned**) is **deduped** by **`(user_id, event_id)`** within a **TTL window** so **retries** don’t **spam**.”

<a id="say-voice-1"></a>

| Beat | Say it like this |
|------|------------------|
| **Bridge** | “**API** → **Kafka** → **workers** → **APNs/FCM**.” |
| **Core split** | “**Token registry** vs **send pipeline**.” |

<a id="key-insight-say-early"></a>
### Key insight (say early)

**Adapter layer** per provider + **central policy** (rate, consent, quiet hours) + **fan-out workers** for **segments**.

#### Key anchors

1. “**Token invalidation** on **410** / **Unregistered**.”  
2. “**Collapse_key** (Android) / **apns-id** (iOS) for **dedupe**.”  
3. “**Consent** gate.”

---

## 2. Estimate scale

<a id="say-voice-2"></a>

| Dimension | Illustrative |
|-----------|----------------|
| Sends / day | **Billions** at mega-scale |
| Fan-out | **Millions** for rare blasts—**partition** topic |

---

## 3. APIs and data model

<a id="say-voice-3"></a>

### 3.1 APIs

| API | Purpose |
|-----|---------|
| `POST /v1/devices` | Register token |
| `POST /v1/notifications` | Enqueue send |
| `POST /v1/topics/{id}/subscribe` | Topic fan-out |

### 3.2 Model

- **Device:** `user_id`, `token`, `platform`, `app_version`, `last_seen`.  
- **NotificationJob:** `id`, `payload_ref`, `priority`, `dedupe_key`.

---

## 4. High-level architecture

<a id="say-voice-4"></a>

```mermaid
flowchart TB
  API[Notification API]
  K[Kafka]
  W[Send workers]
  APNs[APNs]
  FCM[FCM]
  REG[(Token store)]
  API --> K --> W
  W --> APNs
  W --> FCM
  W --> REG
```

### 4.1 Phases

| Phase | Ship |
|-------|------|
| **1** | Single provider + DB tokens |
| **2** | **Priority queues** + **per-tenant** rate |
| **3** | **Geo** routing + **HA** provider pools |

---

## 5. Deep dive: enqueue → provider

<a id="say-voice-5"></a>

<a id="bottleneck-anchor-once"></a>
### 🎯 Bottleneck Anchor

“**Provider rate limits** + **invalid tokens**—**adaptive** concurrency and **token hygiene**.”

```mermaid
sequenceDiagram
  participant S as Trip svc
  participant N as Notify API
  participant K as Kafka
  participant W as Worker
  participant FCM
  S->>N: send(user, template, event_id)
  N->>N: dedupe check
  N->>K: enqueue
  W->>W: load devices
  W->>FCM: multicast chunk
  FCM-->>W: partial invalid tokens
  W->>W: tombstone bad devices
```

**Taking a stance:** *“**Transactional** path **small** Kafka **priority** topic; **marketing** **separate** cluster/pool so **blast** doesn’t **starve** **trip** pushes.”*

---

## 6. Scaling and bottlenecks

| Risk | Mitigation |
|------|------------|
| **Segment fan-out** | **Pre-materialize** audiences in **batch** |
| **Hot user** many devices | **Cap** devices per send |

---

## 7. Reliability and failure handling

- **Provider outage:** **exponential backoff**; **DLQ** for manual replay.  
- **Duplicate API:** **idempotency key** on `POST /notifications`.

---

## 8. Tradeoffs and alternatives

| Choice | Trade |
|--------|--------|
| **Data in payload** | Rich UX vs **privacy** + size |
| **Pull inbox** | Reliable vs **not native push** |

---

## 9. Monitoring, observability, and security

**Metrics:** **provider** error codes, **latency**, **queue depth**, **invalid token** rate.  
**Security:** **Encrypt** tokens at rest; **no PII** in payload if avoidable; **signed** deep links.

---

## 10. Design patterns, data structures & best practices

| Pattern | Map |
|---------|-----|
| **Adapter** | APNs vs FCM |
| **Outbox** | Trip → notify durable |
| **Bulkhead** | Marketing vs transactional pools |

<a id="say-voice-10"></a>
**Live:** max **four** patterns.

---

## Closing notes

<a id="communication-do-vs-avoid"></a>

| Do | Avoid |
|----|--------|
| **Dedupe + priority split** | One queue for everything |
| **Token tombstone** | Retry forever on dead token |

---

## Bar-raiser follow-ups

| They ask | Say it like this |
|----------|------------------|
| **Rich media** | “**Attachment** URLs **signed**; **download** on display.” |

---

## 60-second close

| Beat | Say it like this |
|------|------------------|
| **Recap** | “**Token registry**; **Kafka** + **workers**; **provider adapters**; **dedupe** for **transactional**; **separate** pools for **marketing**; **metrics** on **provider errors**.” |

---
