# Lab 12 — PTU quota and pricing checks

**Zone:** 🤖 Agent · **Dimension:** D2 Model & capacity strategy · **Evidence:** [evidence/10-kc-datazone.txt](../evidence/10-kc-datazone.txt), [evidence/50-contentlogging.txt](../evidence/50-contentlogging.txt)

---

## 🧒 In plain words

PTU is like renting a private lane on a highway. The smallest lane you can rent depends on the type
of vehicle. The city also limits how many private lanes you can rent, and the price depends on the
vehicle type.

## 🧠 The idea

PTU minimums and increments are **per model**. Quota is tracked by **subscription × region ×
deployment type**. Pricing is also per model, so a design needs all three checks: model capacity,
regional quota, and current price.

In the verification environment, organization policy blocked creating a Provisioned deployment with
`RequestDisallowedByPolicy`. The PTU minimums were therefore confirmed by the `modelCapacities` API,
not by deployment creation. Treat this lab as the method to verify current values in your tenant.

## 🔬 Try it

Prereq: [setup](../setup/prerequisites.md) done (`$SUB`, `$LOC`).

**1. Read available PTU models for the region:**

```bash
az rest --method get \
  --url "https://management.azure.com/subscriptions/$SUB/providers/Microsoft.CognitiveServices/locations/$LOC/models?api-version=2024-10-01" \
  > lab-12-models.json

jq -r '
  .value[]
  | select(any(.model.skus[]?; .name == "DataZoneProvisionedManaged" or .name == "ProvisionedManaged"))
  | [.model.name, (.model.version // "")]
  | @tsv
' lab-12-models.json | sort -u
```

**2. Read PTU capacity and minimums for one model/version:**

```bash
export MODEL_NAME="gpt-5.6-sol"
export MODEL_VERSION="<version-from-catalog>"

az rest --method get \
  --url "https://management.azure.com/subscriptions/$SUB/providers/Microsoft.CognitiveServices/locations/$LOC/modelCapacities?api-version=2024-10-01&modelFormat=OpenAI&modelName=$MODEL_NAME&modelVersion=$MODEL_VERSION" \
  > lab-12-capacity.json

jq '.value[] | select(.skuName == "ProvisionedManaged" or .skuName == "DataZoneProvisionedManaged")' lab-12-capacity.json
```

**3. Check regional usage and quota:**

```bash
az cognitiveservices usage list -l "$LOC" -o table
```

**4. Read current retail price for the region and filter for the model PTU meter:**

```bash
curl -sS "https://prices.azure.com/api/retail/prices?\$filter=serviceName eq 'Cognitive Services' and armRegionName eq 'koreacentral'" \
  > lab-12-prices.json

jq -r --arg model "$MODEL_NAME" '
  .Items[]
  | select((.meterName // "" | test($model; "i")) and (.meterName // "" | test("PTU|Provisioned"; "i")))
  | [.meterName, .skuName, .retailPrice, .unitOfMeasure]
  | @tsv
' lab-12-prices.json
```

## 📊 What you'll see

```
- DataZoneProvisionedManaged (PTU) models available in KC: gpt-5.2, gpt-5.6-sol (from 10-kc-datazone.txt)
- Resource-level limits sample (from 50-contentlogging.txt): MaxFineTuneCount=500, MaxRunningFineTuneCount=3, useEndpointRateLimiter=Enabled, quotaRefundsEnabled=false
- CAVEAT: creating a Provisioned deployment returned RequestDisallowedByPolicy in the test tenant; PTU minimums were therefore confirmed via the modelCapacities API, not by deployment creation.
```

## ✅ Why it matters

- Capacity planning must be per model because PTU minimums and increments differ by model.
- Quota must be checked in the target subscription, region, and deployment type.
- Cost planning must use current model-specific retail pricing.
- All numbers are time-sensitive, so verify them at design time instead of copying old values.

## 📎 Evidence

[evidence/10-kc-datazone.txt](../evidence/10-kc-datazone.txt) — PTU-capable DataZone models in Korea Central; [evidence/50-contentlogging.txt](../evidence/50-contentlogging.txt) — resource-level limits sample and tenant caveat.

## 🧹 Cleanup

```bash
rm -f lab-12-models.json lab-12-capacity.json lab-12-prices.json
```

## 📚 References

- [Azure OpenAI Service quotas and limits](https://learn.microsoft.com/en-us/azure/ai-services/openai/quotas-limits)
- [Azure Retail Prices API overview](https://learn.microsoft.com/en-us/rest/api/cost-management/retail-prices/azure-retail-prices)

*Confirmed 2026-10. PTU availability, quota, and price are time-sensitive; re-check with live APIs.*
