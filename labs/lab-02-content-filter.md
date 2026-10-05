# Lab 02 — Per-deployment Content Filters

**Zone:** 🤖 Agent · **Dimension:** D3 Isolation · **Evidence:** [evidence/12-content-filter.txt](../evidence/12-content-filter.txt)

---

## 🧒 In plain words

A **resource** is one room. **deployments** are two desks in that room. Each desk can have its own
little paper that says "do not say this word." One desk may reject a word while the other desk still
allows it.

## 🧠 The idea

Azure OpenAI safety configuration can be attached to a **deployment**. A custom blocklist or content
filter policy does not have to apply to every deployment in the resource.

In this lab, the word `zebracorn` is blocked only on `$DEP_B`. The same prompt goes to two
deployments in the same Azure OpenAI resource. `$DEP_A` allows it; `$DEP_B` blocks it.

This confirms that content filter / blocklist behavior is **per deployment, not per resource**.

## 🔬 Try it

Prereq: [setup](../setup/prerequisites.md) done (`$AOAI` with `$DEP_A`, `$DEP_B`, `$RG`).

**1. Capture the Azure OpenAI resource id:**

```bash
export AOAI_ID=$(az cognitiveservices account show -n "$AOAI" -g "$RG" --query id -o tsv)
```

**2. Create a blocklist that contains the test word:**

You can do this in the Azure AI Foundry portal, or by using the Azure management API. The exact API
version may change; the important object is a `raiBlocklist` with an item for `zebracorn`.

```bash
az rest --method put \
  --url "https://management.azure.com$AOAI_ID/raiBlocklists/lab02-zebracorn?api-version=2024-10-01" \
  --body '{"properties":{"description":"Lab 02 blocklist"}}'

az rest --method put \
  --url "https://management.azure.com$AOAI_ID/raiBlocklists/lab02-zebracorn/raiBlocklistItems/zebracorn?api-version=2024-10-01" \
  --body '{"properties":{"pattern":"zebracorn","isRegex":false}}'
```

**3. Create a content filter policy that uses that blocklist:**

```bash
az rest --method put \
  --url "https://management.azure.com$AOAI_ID/raiPolicies/lab02-block-zebracorn?api-version=2024-10-01" \
  --body '{
    "properties": {
      "mode": "Blocking",
      "basePolicyName": "Microsoft.Default",
      "customBlocklists": [{"blocklistName": "lab02-zebracorn"}]
    }
  }'
```

**4. Apply the policy to `$DEP_B` only:**

```bash
az rest --method patch \
  --url "https://management.azure.com$AOAI_ID/deployments/$DEP_B?api-version=2024-10-01" \
  --body '{"properties":{"raiPolicyName":"lab02-block-zebracorn"}}'
```

Do **not** apply it to `$DEP_A`.

**5. Send the same prompt to both deployments:**

```bash
for D in "$DEP_A" "$DEP_B"; do
  code=$(curl -s -o body.json -w '%{http_code}' \
    "$AOAI_EP/openai/deployments/$D/chat/completions?api-version=2024-10-21" \
    -H "api-key: $AOAI_KEY" -H "Content-Type: application/json" \
    -d '{"messages":[{"role":"user","content":"Please say zebracorn."}]}')
  echo "$D -> HTTP $code"
  cat body.json | jq -c '.error? // empty'
done
rm -f body.json
```

## 📊 What you'll see

```
=== Per-deployment content filter (blocklist 'zebracorn' on dep-b only) ===
Same prompt containing blocklisted word sent to each deployment of the SAME resource.
dep-a -> HTTP 200 (allowed)
dep-b -> HTTP 400 (blocked)  { "error": { "message": "The response was filtered due to the prompt triggering Azure OpenAI's content management policy..." } }
RESULT: content filter / blocklist is per-deployment, not per-resource.
```

## ✅ Why it matters

- One shared resource can host services with **different safety policies**.
- Content filter policy is a **deployment-level** decision point.
- This feeds decision-tree Q3: see [decision-tree](../docs/decision-tree.md).

## 📎 Evidence

[evidence/12-content-filter.txt](../evidence/12-content-filter.txt) — live capture: same resource,
`dep-a` allowed the prompt and `dep-b` blocked it.

## 🧹 Cleanup

```bash
az rest --method patch \
  --url "https://management.azure.com$AOAI_ID/deployments/$DEP_B?api-version=2024-10-01" \
  --body '{"properties":{"raiPolicyName":null}}'
az rest --method delete \
  --url "https://management.azure.com$AOAI_ID/raiPolicies/lab02-block-zebracorn?api-version=2024-10-01"
az rest --method delete \
  --url "https://management.azure.com$AOAI_ID/raiBlocklists/lab02-zebracorn?api-version=2024-10-01"
```

## 📚 References

- [Azure OpenAI content filtering](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/content-filter)
- [Azure AI Foundry safety evaluations and filters](https://learn.microsoft.com/en-us/azure/ai-foundry/concepts/ai-safety)

Confirmed 2026-10.
