# Uber SDE-2 HLD — Interview spine (bar raiser / Strong Hire)

Use this page **once**, then drive **each** problem guide (`11-hld-*.md` … `37-hld-*.md`) in the **same order**. The guides already use **§1–§10 + Closing**; this doc maps your **spoken flow** to those headings and lists what a **senior / bar raiser** is listening for.

---

## 1. Canonical order in the room (maps to every HLD file)

| # | You say / do | Typical section in guides | Strong Hire signal |
|---|----------------|---------------------------|---------------------|
| 1 | **Clarify** — scope, ambiguity, who owns what | **§1.0–1.1** (Live flow + Clarify table) | You **pause**; you **reflect back** (“So if … then …”). |
| 2 | **FR** — what we build after alignment | **§1.2** + `#### Human interaction (FR…)` where present | One **spoken pass** (~60–90s); not a spec read. |
| 3 | **NFR** — latency, consistency, cost, safety | **§1.3** + NFR human block | You separate **hard** vs **soft** guarantees. |
| 4 | **Estimate scale** — round numbers, invite correction | **§2** | You name **shard keys**, **hot paths**, **cardinality**—not only QPS. |
| 5 | **Core entities** — nouns + ownership | **§3** (often `### 3.0` / `### 3.2` **Core entities** or first bullets under APIs) | 3–6 **entities**; who **writes** vs **reads** each. |
| 6 | **API design** — surfaces, idempotency, errors | **§3** (API tables + model) | **Idempotency**, **pagination**, **versioning** where relevant. |
| 7 | **HLD architecture** — boxes, data flow | **§4** + diagram | **One** diagram pass; then **pause** (“geo, rank, or infra?”). |
| 8 | **Deep dive + evolution** — bottleneck, sequence, phases | **§5** (+ **§4.1 Phases** for v1→v3) | You **time-box**; you say **default** then **when you’d add complexity**. |
| 9 | **Scaling & bottlenecks** | **§6** | **Hot keys**, **fan-out**, **queues**, **partitions**—linked to diagram. |
| 10 | **Reliability & failure** | **§7** | **Degrade** path; **no silent** user harm; **timeouts** / **retries** / **DLQ** as fits. |
| 11 | **Tradeoffs & alternatives** | **§8** | You **pick a default** and say **when** you’d switch—not “either works.” |
| 12 | **Monitoring, observability, security** | **§9** | **SLIs** tied to **user pain**; **PII** / **AuthZ** / **abuse**—not only CPU graphs. |
| 13 | **Patterns, DS, best practices** | **§10** | **4–6** items **time-boxed**, each **tied to a box**—depth only if they steer. |
| 14 | **Closing** — bar raiser, comms, 60s | **Closing notes**, **Bar-raiser**, **60-second close** | Short, **confident**; **two–four** sentences then **stop**. |

**Rule:** If a guide has **Human interaction** sections (level-4 headings in the markdown), treat them as your **live script** (*Habit* = default sentence in your head; *Live* = what you say in one breath; tables = optional detail if probed).

### Live voice (every guide **11–37**)

Each HLD file opens **§1.0** with **`#### Live voice (real interviewer room)`** *before* the scripted opening: read that block **once** before mock interviews. Intent: **decide out loud**, **pause**, **ping the interviewer** (“where should I zoom?”), use tables as **backup** not a teleprompter, and **rephrase** any `Verbatim` line the second time you return to the same idea so it does not sound canned.

---

## 2. Core entities (where it lives)

- In **11** and **12**, **Core entities** appears as **`### 3.2`** under **APIs and data model**.
- In other guides, cover the **same content** under **§3** before diving into every endpoint: **3–5 rows** — *entity / who owns lifecycle / store*.

If **§3** has no explicit subsection, add a short block:

```markdown
### 3.0 Core entities (say before API tables)

| Entity | Owns / lifecycle (one line) |
|--------|-----------------------------|
| … | … |
```

Fill with **this problem’s** nouns during prep.

---

## 3. What a bar raiser is grading (Uber SDE-2)

They are not checking memorization of **your** GitHub repo; they check whether you can **run** a one-hour design:

1. **Problem framing** — clarify before architecture; **user or business** entry (journey line where the guide has one).
2. **Correctness & boundaries** — invariants, **commit** boundaries, **idempotency**, **consistency** story.
3. **Operability** — metrics, **alerts**, **runbooks**, **security** basics.
4. **Pragmatism** — **defaults**, **phased** rollout, **degradation**, **cost** / cardinality awareness.
5. **Communication** — structure, **listening**, **explicit tradeoffs**, **time-boxing** yourself.

---

## 4. One Cursor prompt (per file) to deepen human interaction

Paste into Cursor with **one** guide open, e.g. `27-hld-search-autocomplete.md`:

> For this HLD markdown file only: (1) Ensure **§1** has Live flow (how to open and steer), Live rule, When (HLD clock) for user journey if present, Micro-pauses, FR/NFR title suffixes, Purpose before say-voice-1. (2) Add **Human interaction** subsection headings (same spirit as **11-hld-uber-eats-homepage.md**: Habit, Live, short tables) for FR, NFR, §2–§10, bar-raiser, 60s close. (3) Add **`### 3.0 Core entities`** under §3 with 3–5 rows specific to this system. (4) Add **Closing notes (where wrap-up human interaction lives)** + Communication table if missing. Do not change unrelated LLD files.

Repeat for each **`NN-hld-*.md`** from **11** to **37** until all match.

---

## 5. External repo

If you mirror this folder to GitHub (e.g. `rajsaurabh1000/HLD`), **push** these markdown changes from your local clone; interviewers do not read your repo during the live round—this is **your** cue sheet.

---

## 6. HM round (brief)

HLD proves **system** thinking; HM often probes **impact**, **conflict**, **prioritization**, **Uber values**. Keep one **STAR** story ready that touches **scale**, **mistake + fix**, and **stakeholder** alignment—no need to duplicate here.
