# Findings Report — Template (neutral framing)

A customer-facing deliverable. Write it **neutrally**: present platform behavior, constraints and
options plus the criteria; put customer decisions in their own column; show confidence as a badge,
not fuzzy words. You provide the unbiased view and the Microsoft best-practice baseline — the
customer brings their own input to the internal discussion.

**Confidence badges:** `Verified (live)` · `Documented` · `Needs confirmation`

---

**Engagement:** _______________  **Date:** _______________  **Version:** _______________

## 1. Executive summary

- Scope assessed: _______________
- Overall maturity (current → target): _______________
- Top three decisions the customer must make: _______________

## 2. Dimension findings

For each dimension, keep statements of fact separate from decisions.

### D_ — <dimension name>

| What the platform does (fact) | Confidence | Evidence |
|-------------------------------|------------|----------|
| | | lab-__ / Learn link |

| Option | Fits when | Criteria |
|--------|-----------|----------|
| | | |

**Customer must decide:** _______________

_(repeat per dimension in scope)_

## 3. Target architecture

- Selected configuration (from [decision tree](../../docs/decision-tree.md)): **option __**
- Rationale (which answers led here): _______________
- Diagram: _______________

## 4. Requirements traceability

Attach the [requirements matrix](requirements-matrix.csv): requirement → capability → verification
→ status. Flag any `Needs confirmation` row.

## 5. Decisions & open items

Attach the [decision register](decision-register.csv). List `PENDING` rows and who owns each.

## 6. Baseline best-practice notes (Microsoft)

Top-of-list items the customer should consider and discuss internally — presented as considerations,
not directives:

1. _______________
2. _______________

## 7. Appendix — evidence

Link the relevant raw logs under [`evidence/`](../../evidence/) and the labs that produced them.

---

*This report organizes verified platform behavior and Microsoft Learn guidance. It does not
prescribe the customer's risk decisions.*
