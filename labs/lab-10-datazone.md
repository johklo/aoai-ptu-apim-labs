# Lab 10 — DataZone model availability

**Zone:** 🤖 Agent · **Dimension:** D1 Data boundary / D2 capacity · **Evidence:** [evidence/10-kc-datazone.txt](../evidence/10-kc-datazone.txt)

---

## 🧒 In plain words

This is like checking the menu before ordering food. A restaurant may have some dishes and not
others. Azure regions are the restaurants, models are the dishes, and deployment types are the menu
sections.

## 🧠 The idea

Azure OpenAI model availability depends on **region** and **deployment type**. Regional, Global,
DataZoneStandard, and DataZoneProvisionedManaged do not expose the same catalog.

DataZone deployment types are a middle ground between Regional and Global for data residency and
capacity planning. The catalog changes over time, so this is a design-time check, not a value to
copy into architecture permanently.

## 🔬 Try it

Prereq: [setup](../setup/prerequisites.md) done (`$SUB`). This is a read-only catalog query.

**1. Query the Korea Central model catalog:**

```bash
az rest --method get \
  --url "https://management.azure.com/subscriptions/$SUB/providers/Microsoft.CognitiveServices/locations/koreacentral/models?api-version=2024-10-01" \
  > lab-10-models.json
```

**2. List models that expose `DataZoneStandard`:**

```bash
jq -r '
  .value[]
  | select(any(.model.skus[]?; .name == "DataZoneStandard"))
  | .model.name
' lab-10-models.json | sort -u
```

**3. List models that expose `DataZoneProvisionedManaged` PTU:**

```bash
jq -r '
  .value[]
  | select(any(.model.skus[]?; .name == "DataZoneProvisionedManaged"))
  | .model.name
' lab-10-models.json | sort -u
```

## 📊 What you'll see

```
=== Korea Central DataZone availability (api-version 2024-10-01) ===
DataZoneStandard models (10): gpt-5.2, gpt-5.3-codex, gpt-5.4-mini, gpt-5.4, gpt-5.6-sol, gpt-6.1-sol, model-router, text-embedding-3-large, text-embedding-3-small, text-embedding-ada-002
DataZoneProvisionedManaged (PTU) models (2): gpt-5.2, gpt-5.6-sol
```

## ✅ Why it matters

- Data residency choices depend on deployment type, not only on the model name.
- Capacity planning depends on which PTU-capable models are offered in the selected region.
- Model availability is time-sensitive. Re-check the catalog when the design is finalized.

## 📎 Evidence

[evidence/10-kc-datazone.txt](../evidence/10-kc-datazone.txt) — Korea Central catalog showed 10 DataZoneStandard models and 2 DataZoneProvisionedManaged models.

## 🧹 Cleanup

```bash
rm -f lab-10-models.json
```

## 📚 References

- [Azure OpenAI Service models](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/models)
- [Azure OpenAI deployment types](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/how-to/deployment-types)

*Confirmed 2026-10. Catalog availability is time-sensitive; re-query before design sign-off.*
