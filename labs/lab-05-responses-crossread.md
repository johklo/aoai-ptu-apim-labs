# Lab 05 — Stored Responses Share a Resource Boundary

**Zone:** 🤖 Agent · **Dimension:** D6 Data protection · **Evidence:** [evidence/22-responses-crossread.txt](../evidence/22-responses-crossread.txt)

---

## 🧒 In plain words

Imagine the assistant writes answers in one shared notebook for the whole room. If someone is
allowed into the room and knows a page number, they can read that page. The notebook does not keep a
separate lock for each person.

## 🧠 The idea

When the Responses API uses `store=true`, the response is persisted under the Azure OpenAI
resource. There is no per-caller access control list on an individual response id.

So, if Identity A creates a stored response and Identity B has resource-scope data-plane access,
Identity B can read that same response by id. This connects back to
[lab-01-rbac-boundary](../labs/lab-01-rbac-boundary.md): access is resource-scoped.

```mermaid
flowchart LR
  A[Identity A: user] --> W[POST /responses store=true]
  W --> S[(Stored response id)]
  B[Identity B: service principal] --> R[GET /responses/{id}]
  S --> R
  R --> O[HTTP 200]
```

## 🔬 Try it

Prereq: [setup](../setup/prerequisites.md) done (`$AOAI`, `$AOAI_EP`, `$AOAI_KEY`, `$RG`, `$DEP_A`).

**1. Identity A creates a stored response:**

```bash
export API_VER=2025-03-01-preview
export RESP_ID=$(curl -sS "$AOAI_EP/openai/responses?api-version=$API_VER" \
  -H "api-key: $AOAI_KEY" -H "Content-Type: application/json" \
  -d "{\"model\":\"$DEP_A\",\"input\":\"cross identity read test\",\"store\":true}" \
  | jq -r .id)
echo "stored response id=$RESP_ID"
```

**2. Create Identity B and grant resource-scope data-plane access:**

```bash
export SP=$(az ad sp create-for-rbac -n sp-aoai-lab05 -o json)
export SP_APP=$(echo "$SP" | jq -r .appId)
export SP_PW=$(echo "$SP"  | jq -r .password)
export TENANT=$(az account show --query tenantId -o tsv)
export AOAI_ID=$(az cognitiveservices account show -n "$AOAI" -g "$RG" --query id -o tsv)

az role assignment create --assignee "$SP_APP" \
  --role "Cognitive Services OpenAI User" --scope "$AOAI_ID"
sleep 300
```

**3. Identity B gets a token and reads Identity A's response id:**

```bash
export TOK=$(curl -sS -X POST "https://login.microsoftonline.com/$TENANT/oauth2/v2.0/token" \
  -d "grant_type=client_credentials&client_id=$SP_APP&client_secret=$SP_PW&scope=https://cognitiveservices.azure.com/.default" \
  | jq -r .access_token)

curl -s -o crossread.json -w 'Identity B GET -> HTTP %{http_code}\n' \
  "$AOAI_EP/openai/responses/$RESP_ID?api-version=$API_VER" \
  -H "Authorization: Bearer $TOK"
rm -f crossread.json
```

## 📊 What you'll see

```
=== Responses store: cross-identity read within same resource ===
Identity A (user) created stored response id=resp_06a7...ca065c
Identity B (service principal) GET same id -> HTTP 200 (CAN READ)
RESULT: stored Responses are NOT isolated per caller identity. Governance implication for store=true.
```

## ✅ Why it matters

- Stored Responses are protected at the **resource boundary**, not by caller identity.
- If you enable `store=true`, add governance for what may be stored and who has resource access.
- If you do not want persisted responses, force `store=false`; see
  [lab-03-store-false-apim](../labs/lab-03-store-false-apim.md).

## 📎 Evidence

[evidence/22-responses-crossread.txt](../evidence/22-responses-crossread.txt) — live capture:
Identity B read Identity A's stored response by id with HTTP 200.

## 🧹 Cleanup

```bash
az role assignment delete --assignee "$SP_APP" --scope "$AOAI_ID"
az ad app delete --id "$SP_APP"
```

## 📚 References

- [Azure OpenAI and Entra ID RBAC](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/how-to/role-based-access-control)
- [Azure OpenAI Responses API](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/responses)

Confirmed 2026-10.
