# Bar-raiser performance pack — live design, not doc recital

**Mindset:** *“I’m not presenting a finished solution—I’m **designing with a teammate**.”*

In each `NN-hld-*.md`: **(1)** **Live interview opening** at the top = **clarify first** (do not assume requirements). **(2)** Do **Section 1** (clarify → FR → NFR). **(3)** Then say the **Framing after requirements** block (**before Section 2**)—user journey, consistency, decision, etc.—so it sounds **derived**, not memorized. **(4)** Scale, architecture, deep dive. Use **this file** as the printable deep reference; in mocks **speak out loud**, **pause**, simulate **interruptions**.

---

## 1. Live interview opening (clarify first — matches the guides)

*“I’ll **start by clarifying requirements**—scope, ambiguity, latency and scale—then lock **FR/NFR**. **After** that I’ll ground user journey, consistency, and defaults **before** scale and architecture, and I’ll **pause after the diagram** for where you want depth.”*

---

## 2. User journey (say once early — after Section 1, not before)

*“From the user perspective: **(1–2 lines for this system)**.”*

*“So: **write path** = … ; **read path** = … ; **async path** = ….”*

Fill the blanks per problem **before** the mock. In each guide, cross-check Section 1 and your diagram.

---

## 3. Thinking transitions (game changer)

Sprinkle these—**do not** read as a checklist:

- *“Let me think through this…”*
- *“One tradeoff here is…”*
- *“If I optimize for latency…”*
- *“This might become a bottleneck because…”*
- *“I’d start simple here and evolve later…”*

---

## 4. Consistency model (bar raiser magnet)

*“This system uses **strong** consistency for **(critical part)** because **(reason)** ; **eventual** consistency for **(others)** because **(reason)** . If we have to prioritize under load, we choose **(latency / correctness / availability)** for **(surface)** .”*

Say **where** each guarantee lives (which store, which path)—not vague “eventually consistent.”

---

## 5. Decision (strong opinion)

**Never:** *“We can do A or B.”*  
**Always:** *“I’d start with **X** because **(reason)** . If **(scale / requirements / signals)** change, I’d evolve to **Y**.”*

---

## 6. Evolution

| Phase | Say it like this |
|-------|------------------|
| **1** | Simple implementation that ships and debugs. |
| **2** | Scaling: partitions, caches, queues, backpressure, observability. |
| **3** | Advanced: ML, global, finer optimization—**only when earned**. |

Tie each phase to **what broke** or **what metric** forced the upgrade.

---

## 7. Bottleneck anchor

*“The main bottlenecks I expect here are **(1)** and **(2)** —that’s what I’d monitor first.”*

---

## 8. UX awareness

*“If this system behaves poorly, users see **(impact)** —so we prioritize **(trust lever)** over **(nice-to-have)** .”*

---

## 9. Driving the conversation

- *“Does this direction make sense?”*
- *“Should I go deeper into **geo** or **ranking**?”* (swap for your two tensions)
- *“Would you like me to explore failure scenarios next?”*

---

## 10. In the room — do this

1. **Start:** *“Let me start from user flow…”*  
2. **Insight:** *“One key thing here is…”*  
3. **Draw** one pipeline / architecture.  
4. **Pause:** *“Does this make sense so far?”*  
5. **Deep dive** only **one** area unless they redirect.  
6. **Bottlenecks** + what you’d graph.  
7. **Close** in 60 seconds, then **stop**.

---

## 11. Avoid

- Explaining the **entire** doc.  
- Rushing with no pause.  
- Sounding **memorized** (rephrase any `Verbatim` line the second time).

---

## Related

- [HLD-MASTER-DELIVERY-GOLDEN-FLOW.md](./HLD-MASTER-DELIVERY-GOLDEN-FLOW.md) — golden flow + anti–document-mode table (short companion).
