# Lab 09 — PII redaction with language selection

**Zone:** 🚪 Gateway · **Dimension:** D5 Responsible AI & safety · **Evidence:** [evidence/19-pii-korean.txt](../evidence/19-pii-korean.txt)

---

## 🧒 In plain words

PII masking is like a marker that blacks out private details on a form. The marker needs to know
which language the form uses. Even then, it can still miss a private detail, so someone must check
important results.

## 🧠 The idea

Azure AI Language **PiiEntityRecognition** detects and masks personal data in text. For Korean text,
the `language` value is critical. In this lab, the same Korean sample is sent once as `ko` and once
as `en`. The Korean resident registration number is found with `ko` but leaks with `en`.

This belongs before the model as a gateway pre-processing step, for example in an Azure Function
called from APIM. Azure AI Content Safety is not a PII detector — see [lab-04](lab-04-prompt-shields.md).
Detection is not 100%, so treat it as a control with review and fallback, not as a guarantee.

## 🔬 Try it

Prereq: [setup](../setup/prerequisites.md) done (`$RG`, `$LOC`, `$LANG`).

**1. Create or reuse an Azure AI Language resource:**

```bash
az cognitiveservices account create \
  -g "$RG" -n "$LANG" -l "$LOC" \
  --kind TextAnalytics --sku S --yes

export LANG_EP=$(az cognitiveservices account show -g "$RG" -n "$LANG" --query properties.endpoint -o tsv)
export LANG_KEY=$(az cognitiveservices account keys list -g "$RG" -n "$LANG" --query key1 -o tsv)
```

**2. Send the same Korean-format sample with `language=ko`:**

```bash
export SAMPLE='홍길동 고객님, 주민등록번호 900101-1234567, 전화번호 010-1234-5678, 이메일 hong@example.com, 계좌번호 110-234-567890 입니다.'

jq -n --arg text "$SAMPLE" '{
  kind: "PiiEntityRecognition",
  parameters: {modelVersion: "latest"},
  analysisInput: {documents: [{id: "1", language: "ko", text: $text}]}
}' > lab-09-ko.json

curl -sS -X POST "$LANG_EP/language/:analyze-text?api-version=2024-11-01" \
  -H "Ocp-Apim-Subscription-Key: $LANG_KEY" \
  -H "Content-Type: application/json" \
  --data @lab-09-ko.json | jq '.results.documents[0]'
```

**3. Send the same text with `language=en`:**

```bash
jq -n --arg text "$SAMPLE" '{
  kind: "PiiEntityRecognition",
  parameters: {modelVersion: "latest"},
  analysisInput: {documents: [{id: "1", language: "en", text: $text}]}
}' > lab-09-en.json

curl -sS -X POST "$LANG_EP/language/:analyze-text?api-version=2024-11-01" \
  -H "Ocp-Apim-Subscription-Key: $LANG_KEY" \
  -H "Content-Type: application/json" \
  --data @lab-09-en.json | jq '.results.documents[0]'
```

**4. Compare with an English baseline:**

```bash
jq -n '{
  kind: "PiiEntityRecognition",
  parameters: {modelVersion: "latest"},
  analysisInput: {documents: [{id: "1", language: "en", text: "Jane Doe SSN 123-45-6789"}]}
}' > lab-09-us.json

curl -sS -X POST "$LANG_EP/language/:analyze-text?api-version=2024-11-01" \
  -H "Ocp-Apim-Subscription-Key: $LANG_KEY" \
  -H "Content-Type: application/json" \
  --data @lab-09-us.json | jq '.results.documents[0]'
```

## 📊 What you'll see

```
##### language=ko  entities=5
   [Person] '홍길동' 0.96   [PersonType] '고객님' 0.83   [KRResidentRegistrationNumber] '900101-1234567' 0.85
   [PhoneNumber] '010-1234-5678' 1.0   [Email] 'hong@example.com' 0.8
   redacted: ... 주민등록번호 ****** ... 계좌번호 110-234-567890   (<-- bank account NOT redacted / MISSED)
##### same data, language=en  entities=4
   redacted: ... 주민등록번호 900101-1234567 ...   (<-- KR RRN leaked when language!=ko)
##### English baseline language=en
   [USSocialSecurityNumber] '123-45-6789' 1.0 -> redacted correctly
```

## ✅ Why it matters

- PII masking needs the correct language setting.
- Detection is not 100%; a bank account number was not redacted in the observed Korean sample.
- A gateway can reduce model exposure by redacting before the model call, but high-risk flows still
  need human review, downstream checks, or a deny/fallback path.
- This validates the deck caveat that PII detection is not 100% in preview-style control designs.

## 📎 Evidence

[evidence/19-pii-korean.txt](../evidence/19-pii-korean.txt) — Korean RRN found with `language=ko`, leaked with `language=en`; bank account number missed.

## 🧹 Cleanup

```bash
rm -f lab-09-ko.json lab-09-en.json lab-09-us.json
az cognitiveservices account delete -g "$RG" -n "$LANG"
```

## 📚 References

- [Named Entity Recognition and PII detection in Azure AI Language](https://learn.microsoft.com/en-us/azure/ai-services/language-service/personally-identifiable-information/overview)
- [Analyze text REST API](https://learn.microsoft.com/en-us/azure/ai-services/language-service/concepts/use-asynchronously)

*Confirmed 2026-10. Behavior is model- and language-dependent; re-test with current APIs.*
