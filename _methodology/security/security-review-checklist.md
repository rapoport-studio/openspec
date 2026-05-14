---
title: Security review checklist (OWASP LLM Top 10 2025)
status: stable
owner: pavel
audience: [reviewer, pavel, ai]
tldr: PR review checklist. Tick each row applicable to the PR. Required for any PR touching CLI/Worker code, AI prompts, auth flows, or secrets.
---

# Security review checklist (OWASP LLM Top 10 2025)

> **When to use:** Required for any PR that touches Forge/Muse CLI code, Cloudflare Worker code, AI prompts or system messages, Supabase RLS policies, auth flows, or secrets handling. For other PRs, use judgment — if in doubt, run it.
>
> **How to use:** Copy this checklist into your PR description. For each row: mark **y** (applies and mitigated), **n** (does not apply), or **n-a** (not applicable — add one-line justification).
>
> Source: [OWASP Top 10 for LLM Applications 2025](https://genai.owasp.org/resource/owasp-top-10-for-llm-applications-2025/). Per-item detail at [genai.owasp.org/llm-top-10/](https://genai.owasp.org/llm-top-10/).

---

## Checklist

| ID | Category | Applies to this PR? | Notes / Mitigation |
|---|---|---|---|
| [LLM01](https://genai.owasp.org/llmrisk/llm01-prompt-injection/) | **Prompt Injection** — Attacker manipulates LLM via crafted input (direct: user prompt; indirect: data retrieved by the model from external sources such as repo files, issues, emails). | y / n / n-a | |
| [LLM02](https://genai.owasp.org/llmrisk/llm02-sensitive-information-disclosure/) | **Sensitive Information Disclosure** — LLM inadvertently reveals secrets, PII, or system-prompt content in its responses or logs. | y / n / n-a | |
| [LLM03](https://genai.owasp.org/llmrisk/llm03-supply-chain/) | **Supply Chain** — Vulnerable third-party models, datasets, fine-tuning data, or plugins introduced via dependencies. | y / n / n-a | |
| [LLM04](https://genai.owasp.org/llmrisk/llm04-data-and-model-poisoning/) | **Data and Model Poisoning** — Training or fine-tuning data tampered with to introduce backdoors or biases; adversarial examples in RAG retrieval sets. | y / n / n-a | |
| [LLM05](https://genai.owasp.org/llmrisk/llm05-improper-output-handling/) | **Improper Output Handling** — LLM output passed to downstream components (shell, SQL, HTML renderer) without sanitisation; can enable XSS, SSRF, RCE. | y / n / n-a | |
| [LLM06](https://genai.owasp.org/llmrisk/llm06-excessive-agency/) | **Excessive Agency** — LLM granted more permissions, tools, or autonomy than required; can take unintended consequential actions (file writes, API calls, deployments). | y / n / n-a | |
| [LLM07](https://genai.owasp.org/llmrisk/llm07-system-prompt-leakage/) | **System Prompt Leakage** — System prompt (containing business logic, personas, or secret instructions) exposed to users or extracted by adversarial prompts. | y / n / n-a | |
| [LLM08](https://genai.owasp.org/llmrisk/llm08-vector-and-embedding-weaknesses/) | **Vector and Embedding Weaknesses** — Flaws in RAG pipelines: poisoned documents, cross-tenant leakage in shared embedding stores, inadequate access controls on retrieved content. | y / n / n-a | |
| [LLM09](https://genai.owasp.org/llmrisk/llm09-misinformation/) | **Misinformation** — LLM produces plausible but factually incorrect output treated as authoritative (hallucination in specs, code comments, generated docs). | y / n / n-a | |
| [LLM10](https://genai.owasp.org/llmrisk/llm10-unbounded-consumption/) | **Unbounded Consumption** — Excessive resource use (tokens, compute, API spend) through prompt flooding, recursive agent loops, or unrestricted context growth. | y / n / n-a | |

---

## Quick-reference: which OWASP items matter for which PR type

| PR touches… | Must check |
|---|---|
| Forge / Muse system prompt or prompt templates | LLM01, LLM06, LLM07, LLM09 |
| RAG / retrieval pipeline (future) | LLM01, LLM04, LLM08 |
| Cloudflare Worker or Supabase Edge Function | LLM02, LLM05, LLM10 |
| Supabase RLS or auth flows | LLM02, LLM06 |
| Secrets handling (`wrangler secret put`, `.env`, Infisical) | LLM02, LLM07 |
| AI agent autonomy / tool calls / dispatch | LLM06, LLM10 |
| Output rendered in UI (AI-generated HTML/markdown) | LLM05, LLM09 |
| Third-party model or plugin dependency | LLM03, LLM04 |

---

## Checklist template (copy into PR description)

```markdown
### Security checklist (OWASP LLM Top 10 2025)

| ID | Category | Applies? | Notes |
|---|---|---|---|
| LLM01 | Prompt Injection | y / n / n-a | |
| LLM02 | Sensitive Information Disclosure | y / n / n-a | |
| LLM03 | Supply Chain | y / n / n-a | |
| LLM04 | Data and Model Poisoning | y / n / n-a | |
| LLM05 | Improper Output Handling | y / n / n-a | |
| LLM06 | Excessive Agency | y / n / n-a | |
| LLM07 | System Prompt Leakage | y / n / n-a | |
| LLM08 | Vector and Embedding Weaknesses | y / n / n-a | |
| LLM09 | Misinformation | y / n / n-a | |
| LLM10 | Unbounded Consumption | y / n / n-a | |
```

---

## Additional rows (applied 2026-05-13 from change `validate-personal-data-security`)

The following rows supplement the OWASP LLM Top 10 checklist for PRs that touch data surfaces, content authoring, or RLS policies.

| ID | Category | Applies to this PR? | Notes / Mitigation |
|---|---|---|---|
| DSAR | **DSAR runbook applicable** — PR introduces or modifies a table/field containing user-identifiable data. If yes, verify the DSAR extraction query in `_methodology/security/dsar-runbook.md § 3` covers the new surface; add a query template if not. | y / n / n-a | |
| AUD-TAG | **Audience tags set on new content surfaces** — PR adds a new content type, Mirror section, or document-zone surface. If yes, verify per-section audience classification is documented (client / collaborator / internal-only) per W3+W4 analysis in `archive/2026-05-13-validate-personal-data-security/`. Enforcement deferred to `add-document-zone-permissions`; classification is required now. | y / n / n-a | |
| RLS-AUDIT | **RLS adversarial audit re-run** — PR touches `supabase/migrations/**` with RLS policy changes. If yes, run `tools/security/rls-redteam.sh` against preview Supabase and attach `PASS/FAIL` summary table. Zero critical/major findings required. See `db/conventions.md § RLS adversarial-audit convention`. | y / n / n-a | |

---

## Related

- [`threat-model-template.md`](threat-model-template.md) — Shostack 4-Question Frame for per-capability threat modelling
- [`../risk-taxonomy.md`](../risk-taxonomy.md) — L2 Security level: where this checklist lives and who owns it
- [`../../_root/risk-register.md`](../../_root/risk-register.md) — R7: security policies generated, not reviewed (HIGH)
- [OWASP LLM Top 10 2025 full reference](https://genai.owasp.org/resource/owasp-top-10-for-llm-applications-2025/)
