# Discovery Questionnaire — AI Landing Zone

Run this in the first workshop. Fill the right column with the customer's answers. Leave unknowns
blank and track them as open items. Grouped by the eight framework dimensions.

**Engagement:** _______________  **Date:** _______________  **Attendees:** _______________

---

## 0. Context

| Question | Answer |
|----------|--------|
| What business problem is the AI solving? | |
| How many distinct services/use cases in scope? | |
| Target go-live date? | |
| Existing Azure landing zone (platform LZ) in place? | |

## D1 — Data & regulatory boundary

| Question | Answer |
|----------|--------|
| What data classes enter prompts? (public / internal / personal / regulated) | |
| Any regulation pinning processing to a region or sovereign boundary? | |
| Is de-identified data still in scope per customer policy? | |
| Who owns the data-classification decision? | |

## D2 — Model & capacity

| Question | Answer |
|----------|--------|
| Peak tokens/min per service? Latency target? | |
| Traffic shape — steady or spiky? | |
| Which models? Who owns version upgrades? | |
| Budget owner and cost-allocation model? | |

## D3 — Isolation & multitenancy

| Question | Answer |
|----------|--------|
| How many teams share the platform? Do policies differ? | |
| Any service needing its own permissions / keys / failure domain? | |
| Who manages resource settings — central or per app team? | |

## D4 — AI gateway governance

| Question | Answer |
|----------|--------|
| Need central token limits / routing / logging / safety? | |
| Shared gateway team, or per-app access management? | |
| APIM tier preference / constraint? | |

## D5 — Responsible AI & safety

| Question | Answer |
|----------|--------|
| Harmful-content categories to block? Sector rules? | |
| PII masking required before the model? Which languages? | |
| Abuse monitoring required, or must it be opted out? | |

## D6 — Data protection & keys

| Question | Answer |
|----------|--------|
| Must prompts/responses be persisted, or prefer no storage? | |
| CMK required? By which regulation? | |
| Retention period and access-control needs? | |

## D7 — Observability & audit

| Question | Answer |
|----------|--------|
| Body logging required / allowed / forbidden? | |
| Cost reporting granularity? | |
| Answer-quality evaluation in scope? | |

## D8 — Agent architecture & identity

| Question | Answer |
|----------|--------|
| Simple prompt I/O, or multi-step agents with tools/memory? | |
| How do agents authenticate (managed identity / OBO)? | |
| Where does grounding data live, and its boundary? | |

---

**Open items after this workshop:**

1.
2.
3.
