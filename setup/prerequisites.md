# Prerequisites & Environment Setup

Build the environment once, set the variables, and every lab reuses them.

> **ELI5.** Before cooking, lay out your ingredients and label the jars. These are the jars
> (variables) every recipe (lab) reaches for.

---

## Tools

- **Azure CLI** `az` (with `cognitiveservices`, `apim` extensions) — `az version`
- **curl** and a shell (bash, or PowerShell — see note at bottom)
- An Azure subscription where you can create Cognitive Services + API Management
- `jq` (optional, for JSON) 

Sign in: `az login` then `az account set --subscription <SUB_ID>`.

---

## Variables (set once)

```bash
# --- core ---
export SUB=$(az account show --query id -o tsv)
export LOC=koreacentral
export RG=rg-aoai-labs

# --- Azure OpenAI ---
export AOAI=aoai-labs-res1            # the "room" (resource)
export DEP_A=dep-a                    # "desk" A
export DEP_B=dep-b                    # "desk" B
export MODEL=gpt-4.1-mini
export MODEL_VER=2025-04-14

# --- supporting services (created as labs need them) ---
export LANG=lang-labs                 # Azure AI Language (PII) — lab 09
export CS=cs-labs                     # Content Safety (Prompt Shields) — lab 04
export LAW=law-labs                   # Log Analytics — lab 08
export APIM=apim-labs                 # API Management — gateway labs
export KV=kv-labs-$RANDOM             # Key Vault (CMK) — lab 11
```

> Replace names to keep them globally unique where required (APIM, Key Vault, Storage).

---

## 1. Resource group

```bash
az group create -n "$RG" -l "$LOC"
```

## 2. Azure OpenAI resource + two deployments

```bash
# the resource (room)
az cognitiveservices account create -n "$AOAI" -g "$RG" -l "$LOC" \
  --kind OpenAI --sku S0 --custom-domain "$AOAI"

# two deployments (desks), same model — lets labs compare deployment-level behavior
for D in "$DEP_A" "$DEP_B"; do
  az cognitiveservices account deployment create -n "$AOAI" -g "$RG" \
    --deployment-name "$D" \
    --model-name "$MODEL" --model-version "$MODEL_VER" --model-format OpenAI \
    --sku-capacity 1 --sku-name GlobalStandard
done
```

Get the endpoint and (if key auth is allowed in your tenant) a key:

```bash
export AOAI_EP=$(az cognitiveservices account show -n "$AOAI" -g "$RG" --query properties.endpoint -o tsv)
export AOAI_KEY=$(az cognitiveservices account keys list -n "$AOAI" -g "$RG" --query key1 -o tsv)
```

> **Tenant note.** Some organizations force `disableLocalAuth=true` on Cognitive Services, so key
> auth fails and you must use Entra tokens / managed identity. If `AOAI_KEY` calls return 401, use:
> ```bash
> export AOAI_TOKEN=$(az account get-access-token \
>   --resource https://cognitiveservices.azure.com --query accessToken -o tsv)
> # then:  -H "Authorization: Bearer $AOAI_TOKEN"   instead of  -H "api-key: $AOAI_KEY"
> ```

## 3. Smoke test (data-plane call)

```bash
curl -sS "$AOAI_EP/openai/deployments/$DEP_A/chat/completions?api-version=2024-10-21" \
  -H "api-key: $AOAI_KEY" -H "Content-Type: application/json" \
  -d '{"messages":[{"role":"user","content":"Say hello in one word."}]}' \
  -w '\nHTTP %{http_code}\n'
```

Expected: `HTTP 200` and a short completion. If `401`, switch to the Bearer-token form above.

---

## Which services each lab needs

| Lab | Extra service to create |
|-----|-------------------------|
| 01 RBAC, 02 content filter, 05 cross-read, 06 caching, 12 PTU | none beyond AOAI |
| 03 store=false, 07 token limit, 08 logging | APIM ([apim-guide](../apim/apim-guide.md)) |
| 04 Prompt Shields | Content Safety (`$CS`) |
| 08 logging | Log Analytics (`$LAW`) + APIM |
| 09 PII | Azure AI Language (`$LANG`) |
| 10 DataZone | none (read-only catalog query) |
| 11 CMK | Key Vault (`$KV`) |

Create each as its lab instructs, so you only pay for what you use.

---

## Cleanup (end of study)

```bash
az group delete -n "$RG" --yes --no-wait
```

Also remove any service principal and role assignments created in lab 01, and delete local files
that hold keys/tokens.

---

## PowerShell note

On Windows PowerShell, `export VAR=value` becomes `$VAR = "value"`, and `$VAR` expands the same way
inside double-quoted strings. For `curl`, use `curl.exe` to avoid the `Invoke-WebRequest` alias.
URLs containing `&` must be quoted (`"..."`), and `az` is a batch file so wrap REST URLs in an extra
pair of quotes: `--url """$url"""`.
