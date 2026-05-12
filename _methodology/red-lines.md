---
title: Red Lines — Categories Requiring Mandatory Human Verification
status: stable
owner: pavel
audience: [author, executor, reviewer]
date: 2026-05-12
tldr: Seven task categories where AI autonomous output is permanently prohibited — structural accuracy gaps, not temporary model limits. Each entry carries a one-line What, one-line Why never autonomous, and an explicit human verifier pointer. Consumed as a hard-disclosure header by the Muse system prompt.
red-line-categories:
  - auth-rls
  - payments
  - migrations
  - secrets
  - public-copy-at-scale
  - architectural-boundaries
  - external-api-contracts
---

# Red Lines — Categories Requiring Mandatory Human Verification

> Operational extract of [`risk-taxonomy.md`](risk-taxonomy.md) § L2 and L4. This document lists the categories where SDD retains the **spec → AI-assisted draft → mandatory human verification gate** pattern — permanently, not pending model progress. Research backing: [`research/trust-gradient-evolution-2023-2026.md`](research/trust-gradient-evolution-2023-2026.md).
>
> The seven categories below reflect structural accuracy gaps, not data gaps. The trust-gradient research documents that RLS/auth and concurrent-orchestrator categories gained only 15–25 accuracy points in 4 years, while pure CRUD gained 40+. That asymmetry is expected to persist through 2027.

---

### Auth / RLS boundaries

**What.** Any Supabase Row Level Security policy, `USING`/`WITH CHECK` clause, or authentication flow (magic-link routes, session token handling, middleware guard).

**Why never autonomous.** Accuracy plateau at 40–50% through 2026 (Veracode 2025; Lovable CVE-2025-48757 — 70% of audited apps had RLS disabled). The model cannot infer "who reads this table" from schema alone; app-specific threat models are not in training data.

**Verifier.** L2 security review by Pavel (or designated AppSec) on every PR touching RLS or auth middleware. Checklist: OWASP LLM Top 10 items LLM01, LLM02, LLM06; Supabase RLS — no `USING (true)` policies without explicit justification.

---

### Payments / crypto business invariants

**What.** Stripe webhook handlers, payment-intent state machines, refund logic, idempotency keys, any code asserting financial invariants (e.g., `amount > 0`, double-charge guards).

**Why never autonomous.** Pattern-match accuracy 60–75% (2026); novel attack surface (replay attacks, race conditions on charge) not covered by benchmark corpora. Money is irreversible; a 30% error rate is not tolerable.

**Verifier.** Pavel review + payment specialist sign-off before shipping. Automated: invariant unit tests with property-based cases (fast-check or equivalent) required in the same PR.

---

### DB migrations

**What.** Any Supabase migration file (`supabase/migrations/*.sql`) that adds or drops columns, changes nullability, creates or drops indexes, or modifies foreign key constraints.

**Why never autonomous.** Migrations are irreversible once applied to production. AI-generated schema changes silently omit `NOT NULL` defaults, misread column ordering, or drop indexes without recognising downstream query plans. Risk multiplies when the migration also changes an RLS-governed table.

**Verifier.** Human reviewer confirms: (1) migration is additive or includes a rollback path; (2) all `NOT NULL` columns have a backfill or default; (3) if RLS-governed table is touched, escalate to the Auth/RLS verifier above. Supabase MCP `apply_migration` checkpoint emits row counts — Pavel acknowledges before merge.

---

### Secrets

**What.** Infisical secret creation/rotation, Cloudflare Worker `wrangler secret put` invocations, `.env` file changes, or any code path that reads, writes, or logs a secret value.

**Why never autonomous.** Secrets are not code-reviewable after commit (they leave traces in history, logs, CI output). The studio's memory records two incidents of Worker drift from Infisical rotation (`worker_secrets_drift_from_infisical`). An AI agent misrouting a key name causes silent auth failures that are hard to diagnose.

**Verifier.** Pavel only. No AI agent may execute secret-write operations. Secret names (not values) may appear in code. Every rotation is logged in the Infisical audit trail and cross-checked against Worker bindings.

---

### Public-facing copy at scale

**What.** Marketing copy, legal notices, terms of service, pricing text, or any user-visible string rendered on `rapoport.studio` or `app.rapoport.studio` that reaches a wide audience without per-user customisation.

**Why never autonomous.** Legal exposure (misleading claims, GDPR consent language) and brand voice. AI-generated copy tends to be fluent but factually imprecise ("unlimited" when limits exist; compliance language that doesn't match the actual data-processing terms). At scale, corrections require redeployment and may trigger regulatory scrutiny.

**Verifier.** Founder review (Pavel) for tone and legal accuracy. For compliance-adjacent strings (cookie banners, data-processing notices), legal counsel sign-off is required before deploy.

---

### Architectural-class boundaries

**What.** Changes to the package-boundary table in `CLAUDE.md` (which package may import which), ESLint `no-restricted-imports` rule additions, monorepo workspace splits or merges, or new cross-package import paths not yet sanctioned in the boundary table.

**Why never autonomous.** Package boundaries encode long-lived architectural decisions. An incorrect import permitted by an AI agent cascades into circular dependencies, bundle bloat, or broken isolation contracts that require expensive refactors. The boundary table exists precisely because these decisions are non-local.

**Verifier.** Architect (Pavel) review of the diff against `CLAUDE.md § Package boundaries`. ESLint boundary guard must pass (`pnpm lint`) — but green lint is necessary, not sufficient; human must confirm the new path is intentional.

---

### External-API contracts

**What.** Changes to any public-facing API route (path, method, response shape), webhook payload format, or client-exposed TypeScript type that downstream consumers (n8n, Telegram bridge, future partners) depend on.

**Why never autonomous.** Breaking a public contract cannot be fixed by redeployment alone — every consumer must update. AI agents optimise for internal consistency and routinely drop or rename fields without flagging the breaking-change risk to external callers.

**Verifier.** Human changelog entry required in the PR description: list every removed/renamed field and which consumers are affected. For Telegram/n8n bridges: smoke-test the integration before merge. For partner-facing APIs: version bump required; breaking change must be communicated.
