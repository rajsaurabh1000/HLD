# Uber SDE-2 HLD — Interview spine (bar raiser / Strong Hire)

Use this page **once**, then drive each problem guide (`11-hld-*.md` … `37-hld-*.md`) in the **same order** as the drive-order blockquote in each file.

**Delivery (live room, not reading notes):** [HLD-BAR-RAISER-PERFORMANCE-PACK.md](./HLD-BAR-RAISER-PERFORMANCE-PACK.md) · **Golden flow + anti-doc table:** [HLD-MASTER-DELIVERY-GOLDEN-FLOW.md](./HLD-MASTER-DELIVERY-GOLDEN-FLOW.md)

Each guide: **Live interview opening** (clarify-first, under the title) → spine + **Interview delivery** → **Section 1** (incl. **Live voice** in §1.0) → **Framing after requirements** (user journey, consistency, defaults—**before Section 2**) → rest. That order avoids sounding like you assumed requirements.

---

## Canonical order in the room

| Step | You say / do | Typical section in guides |
|------|----------------|----------------------------|
| 1 | Clarify — scope, who owns commits | Section 1 |
| 2 | FR — what we build | Section 1.2 + Human interaction |
| 3 | NFR — latency, consistency, cost | Section 1.3 |
| 4 | **Framing after requirements** — user journey, consistency, strong defaults (derived from §1) | Block **before Section 2** in each guide |
| 5 | Estimate scale | Section 2 |
| 6 | Core entities + APIs | Section 3 |
| 7 | Architecture — one diagram, pause | Section 4 |
| 8 | Deep dive + evolution | Section 5 (+ phases in 4.1) |
| 9 | Scaling & bottlenecks | Section 6 |
| 10 | Reliability & failure | Section 7 |
| 11 | Tradeoffs — **default + when to switch** | Section 8 |
| 12 | Observability & security | Section 9 |
| 13 | Patterns / DS (time-boxed, tied to boxes) | Section 10 |
| 14 | Close — bar raiser, 60s | Closing |

**Rule:** `#### Human interaction` blocks are *spoken* cues—**paraphrase**; tables are backup if probed.

---

## What bar raisers listen for

1. **Clarify before solution**; **user journey** grounded **after** FR/NFR (see **Framing after requirements**), then boxes.  
2. **Correctness** — invariants, commit boundaries, idempotency, **consistency** story.  
3. **Operability** — metrics, alerts, runbooks, security basics.  
4. **Pragmatism** — defaults, phased rollout, degrade paths, cost/cardinality.  
5. **Communication** — structure, listening, **explicit tradeoffs**, time-boxing yourself.
