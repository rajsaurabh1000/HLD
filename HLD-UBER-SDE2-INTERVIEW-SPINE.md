# Uber SDE-2 HLD — Interview spine (bar raiser / Strong Hire)

Use this page **once**, then drive each problem guide (`11-hld-*.md` … `37-hld-*.md`) in the **same order** as the drive-order blockquote in each file.

**Delivery (live room, not reading notes):** [HLD-BAR-RAISER-PERFORMANCE-PACK.md](./HLD-BAR-RAISER-PERFORMANCE-PACK.md) · **Golden flow + anti-doc table:** [HLD-MASTER-DELIVERY-GOLDEN-FLOW.md](./HLD-MASTER-DELIVERY-GOLDEN-FLOW.md)

Each guide starts with a **Live interview opening** strip (under the title), then **Interview delivery**, then Section 1 **Live voice**—use those before you dive into tables.

---

## Canonical order in the room

| Step | You say / do | Typical section in guides |
|------|----------------|----------------------------|
| 1 | Clarify — scope, who owns commits | Section 1 |
| 2 | FR — what we build | Section 1.2 + Human interaction |
| 3 | NFR — latency, consistency, cost | Section 1.3 |
| 4 | Estimate scale | Section 2 |
| 5 | Core entities + APIs | Section 3 |
| 6 | Architecture — one diagram, pause | Section 4 |
| 7 | Deep dive + evolution | Section 5 (+ phases in 4.1) |
| 8 | Scaling & bottlenecks | Section 6 |
| 9 | Reliability & failure | Section 7 |
| 10 | Tradeoffs — **default + when to switch** | Section 8 |
| 11 | Observability & security | Section 9 |
| 12 | Patterns / DS (time-boxed, tied to boxes) | Section 10 |
| 13 | Close — bar raiser, 60s | Closing |

**Rule:** `#### Human interaction` blocks are *spoken* cues—**paraphrase**; tables are backup if probed.

---

## What bar raisers listen for

1. Problem framing and **user journey** before boxes.  
2. **Correctness** — invariants, commit boundaries, idempotency, **consistency** story.  
3. **Operability** — metrics, alerts, runbooks, security basics.  
4. **Pragmatism** — defaults, phased rollout, degrade paths, cost/cardinality.  
5. **Communication** — structure, listening, **explicit tradeoffs**, time-boxing yourself.
