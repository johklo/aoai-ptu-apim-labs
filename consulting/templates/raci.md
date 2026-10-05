# RACI — Platform team vs App team vs Customer

AI landing zones have **split ownership**: a central platform/gateway team provides shared
guardrails; app teams build agents behind them; the customer owns policy and risk decisions. Fill
in names. R = Responsible (does the work), A = Accountable (owns the outcome), C = Consulted, I =
Informed.

| Activity | Platform / Gateway team | App / Workload team | Customer (business/security) | Microsoft consultant |
|----------|:---:|:---:|:---:|:---:|
| Data classification & regulatory boundary (D1) | C | C | **A/R** | C |
| Model & capacity choice, PTU/PAYG (D2) | C | R | **A** | C |
| Resource isolation design (D3) | **A/R** | C | I | C |
| AI gateway policy — token limit, routing, logging (D4) | **A/R** | C | I | C |
| Content safety & PII policy (D5) | R | C | **A** | C |
| Storage / CMK / retention (D6) | R | C | **A** | C |
| Observability & cost reporting (D7) | **A/R** | C | I | C |
| Agent build — orchestration, memory, tools (D8) | I | **A/R** | I | C |
| Model-version upgrade cadence | C | R | **A** | C |
| Incident response for unsafe output | R | R | **A** | C |

**Key principle.** The gateway team owns *shared controls*; app teams own *agent behavior*; the
customer owns *risk and policy decisions*. The consultant advises and verifies — never owns the
customer's risk decision.

**Named owners:**

| Role | Name | Contact |
|------|------|---------|
| Platform / Gateway lead | | |
| App team lead(s) | | |
| Customer security owner | | |
| Customer data owner | | |
