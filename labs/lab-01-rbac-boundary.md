# Lab 01 — Permissions live at the Resource, not the Deployment

**Zone:** 🤖 Agent · **Dimension:** D3 Isolation · **Evidence:** [evidence/13-rbac-scope.txt](../evidence/13-rbac-scope.txt)

---

## 🧒 In plain words

A **resource** is a room; **deployments** are desks inside it. You might hope to give someone a key
to *one desk only* — but Azure OpenAI only hands out keys to the **whole room**. Give access to the
room and they can use every desk. There is no "one desk" key.

## 🧠 The idea

Azure OpenAI **data-plane authorization is at the resource (account) scope.** A role assignment
scoped to a single deployment is *accepted by ARM* (the control plane) but the *data plane ignores
it* — the caller gets 401 on every deployment. Grant the role at the resource and the caller gets
200 on all deployments.

This is the single most important fact for multitenancy: **you cannot isolate services by giving
each access to a different deployment of a shared resource.** That is why a gateway (APIM) is used
to pin services to deployments — see [apim-guide](../apim/apim-guide.md).

## 🔬 Try it

Prereq: [setup](../setup/prerequisites.md) done (`$AOAI` with `$DEP_A`, `$DEP_B`).

**1. Create a service principal** (a stand-in "service identity"):

```bash
export SP=$(az ad sp create-for-rbac -n sp-aoai-lab01 -o json)
export SP_APP=$(echo "$SP" | jq -r .appId)
export SP_PW=$(echo "$SP"  | jq -r .password)
export TENANT=$(az account show --query tenantId -o tsv)
export AOAI_ID=$(az cognitiveservices account show -n "$AOAI" -g "$RG" --query id -o tsv)
```

**2. Phase 1 — assign the role scoped to ONE deployment:**

```bash
az role assignment create --assignee "$SP_APP" \
  --role "Cognitive Services OpenAI User" \
  --scope "$AOAI_ID/deployments/$DEP_A"
```

**3. Get an SP data-plane token and call each deployment:**

```bash
get_sp_token () {
  curl -sS -X POST "https://login.microsoftonline.com/$TENANT/oauth2/v2.0/token" \
    -d "grant_type=client_credentials&client_id=$SP_APP&client_secret=$SP_PW&scope=https://cognitiveservices.azure.com/.default" \
    | jq -r .access_token
}
export TOK=$(get_sp_token)

for D in "$DEP_A" "$DEP_B"; do
  code=$(curl -s -o /dev/null -w '%{http_code}' \
    "$AOAI_EP/openai/deployments/$D/chat/completions?api-version=2024-10-21" \
    -H "Authorization: Bearer $TOK" -H "Content-Type: application/json" \
    -d '{"messages":[{"role":"user","content":"hi"}]}')
  echo "$D -> HTTP $code"
done
```

**4. Phase 2 — add the role at the RESOURCE scope and repeat:**

```bash
az role assignment create --assignee "$SP_APP" \
  --role "Cognitive Services OpenAI User" --scope "$AOAI_ID"
sleep 300                      # role propagation can take minutes
export TOK=$(get_sp_token)     # refresh token
# re-run the for-loop from step 3
```

## 📊 What you'll see

```
[Phase 1]  deployment-scoped role
  dep-a -> HTTP 401    dep-b -> HTTP 401     (ARM accepted it; data plane IGNORED it)

[Phase 2]  resource-scoped role added
  dep-a -> HTTP 200 (ALLOWED)
  dep-b -> HTTP 200 (ALLOWED)
```

## ✅ Why it matters

- A **per-deployment role grant gives no data-plane access** — it looks assigned but does nothing.
- **Resource-scope grants access to *all* deployments** — all or nothing.
- Therefore **service isolation needs separate resources**, or a **gateway** that pins each service
  to a deployment and enforces its own auth. This drives decision-tree Q2
  ([decision-tree](../docs/decision-tree.md)).

## 📎 Evidence

[evidence/13-rbac-scope.txt](../evidence/13-rbac-scope.txt) — live capture (2026-10-05): deployment
scope → 401/401, resource scope → 200/200.

## 🧹 Cleanup

```bash
az role assignment delete --assignee "$SP_APP" --scope "$AOAI_ID"
az role assignment delete --assignee "$SP_APP" --scope "$AOAI_ID/deployments/$DEP_A"
az ad app delete --id "$SP_APP"
```

## 📚 References

- [How to configure Azure OpenAI with Entra ID RBAC](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/how-to/role-based-access-control)
- [Cognitive Services OpenAI User role](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/role-based-access-control)

*Confirmed 2026-10. Behavior verified live; cite Microsoft Learn for the role model.*
