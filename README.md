# Azure OpenAI PTU + API Management — Hands-on Lab Guide

A self-study course that explains **how Azure OpenAI (AOAI) behaves when you put it behind
Azure API Management (APIM) as an AI gateway**. Every claim in this repo was **verified live**
in a Korea Central test environment — the raw command output is included under [`evidence/`](evidence/).

> **How this guide is written.** Each page starts with a plain-language analogy ("explain it
> like I'm five"), then the real concept, then commands you can run, then the result and why it
> matters. You do not need prior Azure AI knowledge to start.

---

## 🧒 The whole thing in one analogy

Imagine a company that hires a very smart assistant (a **language model**).

- The assistant lives in a **room** → an **Azure OpenAI resource**.
- Inside the room there are **desks**, one per task → **deployments** (e.g. `dep-a`, `dep-b`).
- Everyone who wants to ask the assistant something must pass a **front desk with a guard** →
  **API Management (APIM)**. The guard checks who you are, how much you've used, and whether
  you're saying anything unsafe.
- You can either **rent the assistant full-time** (reserved capacity = **PTU**) or **pay per
  question** (**PAYG**).

The whole course is about: *how are the rooms and desks organized, what can the guard do, and
when do you choose full-time vs pay-per-question?*

---

## 🗺️ Learning path

Follow the pages in order. Each lab is independent once you finish setup.

| # | Page | What you learn | Zone |
|---|------|----------------|------|
| — | [docs/ai-landing-zone.md](docs/ai-landing-zone.md) | The big picture: **AI Gateway Zone** vs **Agent Zone** | both |
| — | [docs/00-concepts.md](docs/00-concepts.md) | PTU vs PAYG, resource vs deployment, data-plane auth, APIM as AI gateway | — |
| — | [setup/prerequisites.md](setup/prerequisites.md) | Create the environment and set your shell variables | — |
| — | [apim/apim-guide.md](apim/apim-guide.md) | Put APIM in front of Azure OpenAI, step by step | Gateway |
| 01 | [labs/lab-01-rbac-boundary.md](labs/lab-01-rbac-boundary.md) | Permissions live at the **resource**, not the deployment | Agent |
| 02 | [labs/lab-02-content-filter.md](labs/lab-02-content-filter.md) | Content filters & blocklists are **per deployment** | Agent |
| 03 | [labs/lab-03-store-false-apim.md](labs/lab-03-store-false-apim.md) | APIM can **force `store=false`** so nothing is saved | Gateway |
| 04 | [labs/lab-04-prompt-shields.md](labs/lab-04-prompt-shields.md) | Detect jailbreak / prompt-injection with **Prompt Shields** | Gateway |
| 05 | [labs/lab-05-responses-crossread.md](labs/lab-05-responses-crossread.md) | Stored responses are **not isolated by caller** | Agent |
| 06 | [labs/lab-06-prompt-caching.md](labs/lab-06-prompt-caching.md) | **Prompt caching** happens automatically | Agent |
| 07 | [labs/lab-07-token-limit.md](labs/lab-07-token-limit.md) | APIM **token rate limiting** → HTTP 429 | Gateway |
| 08 | [labs/lab-08-llm-logging.md](labs/lab-08-llm-logging.md) | Log prompts, responses and **token counts** | Gateway |
| 09 | [labs/lab-09-pii-redaction.md](labs/lab-09-pii-redaction.md) | Mask Korean **PII** — and where it falls short | Gateway |
| 10 | [labs/lab-10-datazone.md](labs/lab-10-datazone.md) | **DataZone** deployments and regional availability | Agent |
| 11 | [labs/lab-11-cmk.md](labs/lab-11-cmk.md) | Customer-managed keys: **region + resource scope** | Agent |
| 12 | [labs/lab-12-ptu-quota-pricing.md](labs/lab-12-ptu-quota-pricing.md) | PTU minimums, quota units and pricing | Agent |
| — | [docs/decision-tree.md](docs/decision-tree.md) | Choose a target architecture (6 options) | both |

---

## 🧭 Part B — Consulting framework

The labs prove *how the platform behaves*. The **[consulting framework](consulting/)** turns that
proof into a repeatable customer engagement. An AI landing zone is **not** just infrastructure — its
real subject is data, models, prompts, tokens, behavior, safety and cost. The framework assesses
that across **eight dimensions**, each linked back to the labs that prove it, with ready-to-use
templates:

| Page | Purpose |
|------|---------|
| [consulting/README.md](consulting/README.md) | Why AI LZ differs from a platform LZ; how an engagement runs |
| [consulting/framework.md](consulting/framework.md) | The eight dimensions in detail (discover → decide → maturity) |
| [consulting/templates/](consulting/templates/) | Discovery questionnaire, dimension assessment, decision register, requirements matrix, RACI, findings report |

---

## 📎 Evidence

Every lab links to a raw log in [`evidence/`](evidence/). These are **unedited console
captures** from the live verification run (resource `aoai-t2v-res1`, Korea Central, 2026-10).
The verification environment has since been deleted; the logs remain as proof.

## ⚠️ A note on the test environment

The verification subscription is governed by an organization policy (MCAPS) that forces some
settings — key auth disabled on Cognitive Services, storage locked to private network, PTU
deployment creation blocked. Where a result is shaped by that policy rather than by the platform
itself, the lab says so. In a normal enterprise tenant you configure those with private
endpoints and Entra identities.

## How to use the commands

Commands are Azure CLI / `curl` / PowerShell. Set your variables once in
[setup/prerequisites.md](setup/prerequisites.md); every lab reuses them. Shell blocks use `bash`
style; on Windows PowerShell, replace `$VAR` expansion as noted in setup.

---

*This guide is a study companion. It is not official Microsoft documentation. Every external
behavior is cross-checked against Microsoft Learn, cited inline per lab.*
