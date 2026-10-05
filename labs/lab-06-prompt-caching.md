# Lab 06 — Automatic Prompt Caching

**Zone:** 🤖 Agent · **Dimension:** D2 Cost/capacity · **Evidence:** [evidence/25-prompt-cache.txt](../evidence/25-prompt-cache.txt)

---

## 🧒 In plain words

If you ask the same very long question twice, the assistant can remember the beginning. The second
time, it does not have to read every word from scratch.

## 🧠 The idea

Prompt caching is automatic for identical long prompt prefixes. When the prefix is long enough
(about 1024 tokens or more) and repeated, Azure OpenAI can reuse cached prefix tokens.

You do not configure this in the deployment. The signal appears in the response usage field:
`usage.prompt_tokens_details.cached_tokens`. On the first call it is zero. On the second identical
call it becomes greater than zero.

The cache benefit reduces cost and latency. The cache scope follows the resource boundary.

## 🔬 Try it

Prereq: [setup](../setup/prerequisites.md) done (`$AOAI_EP`, `$AOAI_KEY`, `$DEP_A`).

**1. Build a long prompt with an identical prefix:**

```bash
export LONG_PREFIX=$(python - <<'PY'
print(' '.join(['This is a repeated prefix for prompt caching.'] * 260))
PY
)
```

**2. Send the same prompt twice:**

```bash
for I in 1 2; do
  curl -sS "$AOAI_EP/openai/deployments/$DEP_A/chat/completions?api-version=2024-10-21" \
    -H "api-key: $AOAI_KEY" -H "Content-Type: application/json" \
    -d "$(jq -n --arg p "$LONG_PREFIX" '{messages:[{role:"user",content:($p + " Answer in one sentence.")}]}')" \
    > "cache-$I.json"

  prompt_tokens=$(jq -r '.usage.prompt_tokens' "cache-$I.json")
  cached_tokens=$(jq -r '.usage.prompt_tokens_details.cached_tokens // 0' "cache-$I.json")
  echo "call $I: prompt_tokens=$prompt_tokens cached_tokens=$cached_tokens"
done
```

**3. Remove local response files:**

```bash
rm -f cache-1.json cache-2.json
```

## 📊 What you'll see

```
=== Prompt caching (automatic, >1024 token identical prefix) ===
call 1: prompt_tokens=2027  cached_tokens=0
call 2: prompt_tokens=2027  cached_tokens=1920
RESULT: 2nd identical call reuses cached prefix tokens (cached_tokens > 0). No config required.
```

## ✅ Why it matters

- Prompt caching is a **free cost and latency win** for repeated long prefixes.
- No per-deployment switch is needed for the behavior shown here.
- Cache scope is the **resource boundary**, so shared-resource designs should treat repeated
  prefixes as resource-local optimization.

## 📎 Evidence

[evidence/25-prompt-cache.txt](../evidence/25-prompt-cache.txt) — live capture: second identical
call reported `cached_tokens=1920`.

## 🧹 Cleanup

```bash
rm -f cache-1.json cache-2.json
```

## 📚 References

- [Azure OpenAI prompt caching](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/prompt-caching)
- [Azure OpenAI monitoring and usage](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/monitoring)

Confirmed 2026-10.
