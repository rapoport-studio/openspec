---
title: Red lines for forge build
status: stable
owner: pavel
audience: [pavel, ai, reviewer]
tldr: Paths that must never go through forge build. Hand-coded only. Lint rule enforces with manual-review label requirement.
---

# Red lines for forge build

> These paths are off-limits to `forge build` and any auto-dispatch CI workflow. Hand-coded only.
>
> Research backing: [`research/spec-to-code-validity-2026.md`](research/spec-to-code-validity-2026.md) (8 risk entries + 7 red lines); [`research/trust-gradient-evolution-2023-2026.md`](research/trust-gradient-evolution-2023-2026.md) (structural accuracy plateau).

| Path / area | Reason | Backed by |
|---|---|---|
| `~/.forge/locks/`, `**/state.json`, process orchestration code in `packages/forge/src/orchestrator/` | Concurrent code accuracy <50% per CONCUR'25; memory: `forge_builder_orchestrator_commit_race` | [`trust-gradient-evolution-2023-2026.md`](research/trust-gradient-evolution-2023-2026.md) |
| `supabase/migrations/*RLS*`, `*rls-policy*.sql`, RLS-only migrations | Veracode 45% fail rate flat 3.5 years; Lovable CVE-2025-48757 — 70% of audited apps had RLS disabled | [`spec-to-code-validity-2026.md`](research/spec-to-code-validity-2026.md) |
| Cloudflare Worker secrets via `wrangler secret put`; `apps/*/src/lib/secrets/` | Training data thin; AI hallucinates `process.env`; memory: `worker_secrets_drift_from_infisical` | [`research/ai-risk-frameworks-and-indexing.md`](research/ai-risk-frameworks-and-indexing.md) |
| Auth flows: `apps/studio/src/app/(auth-walled)/`, `lib/auth/`, `middleware.ts` | Auth compliance <55% (Veracode); session management is L1 risk | [`research/ai-risk-frameworks-and-indexing.md`](research/ai-risk-frameworks-and-indexing.md) |
| Crypto primitives: any file importing `crypto`, `subtle`, Node `crypto` module for keys / signing / encryption | Hard red line — formal verification zone | [`spec-to-code-validity-2026.md`](research/spec-to-code-validity-2026.md) |
| Payment / billing flows (when they appear): `apps/*/src/lib/billing/`, Stripe webhook handlers | Hard red line — money + security combination | [`spec-to-code-validity-2026.md`](research/spec-to-code-validity-2026.md) |
| Migration scripts with DATA transformation (not schema-only): `INSERT`, `UPDATE`, `DELETE` in migrations | No undo; cascading risk on production data | [`spec-to-code-validity-2026.md`](research/spec-to-code-validity-2026.md) |

---

## Lint enforcement

Path-based enforcement is implemented in `tools/lint-red-lines.mjs` (Phase 6 of `openspec/changes/establish-risk-framework/`). <!-- openspec-refs: skip-line -->

PRs touching any listed path fail CI unless the `manual-review` GitHub label is set on the PR. The lint rule parses path patterns from the table above — this file is the single source of truth.

Message on failure:

```
PR touches red-line path `<path>` (reason: <reason>).
Add `manual-review` label to confirm hand-coded review was performed.
```

---

## Quarterly review

Red lines are reviewed quarterly. A path may be **removed** if measured AI accuracy on that category improves substantially (e.g., RLS accuracy crossing 70%). Removal requires a new change proposal — it cannot be done in a routine PR.

Red lines may be **added** when new structural accuracy gaps are documented in research. The bar is the same: empirical evidence of sub-55% accuracy over a sustained period.

Next scheduled review: **2026-Q3** (after trust-gradient snapshot update).

---

## Cross-references

- [`risk-taxonomy.md`](risk-taxonomy.md) — L2 Security and L4 SDD level definitions that inform these red lines
- [`_root/risk-register.md`](../_root/risk-register.md) — R2 (concurrent) and R7 (security policies) entries that back these prohibitions
- [`research/spec-to-code-validity-2026.md`](research/spec-to-code-validity-2026.md) — empirical sources for all 7 entries
- [`security/security-review-checklist.md`](security/security-review-checklist.md) — OWASP LLM Top 10 checklist used when `manual-review` label is applied
