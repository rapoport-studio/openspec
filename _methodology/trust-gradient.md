---
title: Trust gradient — 10-category accuracy and required verification
status: stable
owner: pavel
audience: [pavel, ai, reviewer, founder]
tldr: Decision aid for "should this work go through forge build, or hand-coded?". Updates annually based on benchmark trajectory.
---

# Trust gradient — 10-category accuracy and required verification

> Decision aid for routing a task to `forge build` vs hand-coded. Columns: measured AI accuracy in 2026, the minimum verification layer required regardless of mode, and the resulting Forge mode policy.
>
> Research basis: [`research/trust-gradient-evolution-2023-2026.md`](research/trust-gradient-evolution-2023-2026.md) (trajectory 2023–2026 across benchmarks); [`research/spec-to-code-validity-2026.md`](research/spec-to-code-validity-2026.md) (per-category failure modes and empirical sources).

---

| Category | 2026 accuracy | Required verification layer | Forge mode |
|---|---|---|---|
| Dialog / CRUD scaffold | 94–96% | auto-tests + lint + visual | **AUTO-BUILD OK** |
| UI components (single-file) | 92–95% | lint + typecheck + storybook | **AUTO-BUILD OK** |
| OpenAPI route scaffold | 90–93% | + auth middleware manual review | **AUTO-PLAN + MANUAL BUILD** for auth-bearing routes |
| Isolated business logic | 95–97% | + property-based tests on invariants | **AUTO-BUILD OK** |
| DB migrations (schema-only) | 88–92% | + PR review for RLS pairing | **AUTO-BUILD** with RLS-pairing check |
| Multi-file refactor (4+) | 80–88% Verified / **45–65% Pro** | + phase split mandatory | **AUTO-PLAN + MANUAL BUILD** per phase |
| RLS / auth boundaries | **40–50%** | **HAND-CODED + manual review** | **RED LINE** |
| Cloudflare platform context | 75–85% | + `wrangler secret put` after deploy by hand | **AUTO-BUILD** with secret-rotation check |
| Concurrent / orchestrator | **40–50%** | **HAND-CODED + differential testing** | **RED LINE** |
| Crypto / payment business logic | 60–75% | **HAND-CODED + formal verification where feasible** | **RED LINE** |

---

## Structural-gap statement

> *"Categories on plateau will not close with model speed-up. For them, SDD remains spec → AI-assisted draft → mandatory human verification gate. This is not a temporary state."*
>
> — Synthesised from [`research/trust-gradient-evolution-2023-2026.md`](research/trust-gradient-evolution-2023-2026.md) § Conclusion

The three RED LINE rows (RLS / auth boundaries; Concurrent / orchestrator; Crypto / payment business logic) gained only 15–25 accuracy points over four years while pure CRUD gained 40+. That asymmetry is structural: these categories require reasoning about properties that are not in the training signal — threat models that are app-specific, concurrency invariants that depend on runtime topology, and cryptographic guarantees that require formal proof. No benchmark corpus can close this gap at scale.

Enforcement mechanism: path-based prohibitions are declared in [`red-lines.md`](red-lines.md) and linted by `tools/lint-red-lines.mjs`.

---

## How to read the table

- **Accuracy** is the percentage of AI-generated outputs in that category that are correct without human correction, averaged across the benchmark studies cited above.
- **Required verification layer** is the minimum gate regardless of who authored the code. If a PR lacks this layer, it cannot merge.
- **Forge mode** is the policy for `forge build` dispatch:
  - **AUTO-BUILD OK** — Forge can build and dispatch automatically. Human reviews the PR but need not re-verify the implementation.
  - **AUTO-PLAN + MANUAL BUILD** — Forge plans; the executor runs the implementation steps manually, applying each phase before proceeding.
  - **AUTO-BUILD with check** — Forge builds automatically but CI enforces a specific post-build check (noted in the verification column).
  - **RED LINE** — Forge build is prohibited. Human writes the code; AI may assist with drafts but the implementation is reviewed and owned by a human. See [`red-lines.md`](red-lines.md).

---

## Update cadence

This table is reviewed annually or when a major new benchmark study changes the accuracy picture materially. Revisions go through a new change proposal — editing this file in a routine PR is not allowed.

Next scheduled review: **2027-Q1**.

---

## Cross-references

- [`red-lines.md`](red-lines.md) — path-based prohibitions for the three RED LINE categories
- [`risk-taxonomy.md`](risk-taxonomy.md) — L4 SDD-specific risks that map to accuracy tiers
- [`research/trust-gradient-evolution-2023-2026.md`](research/trust-gradient-evolution-2023-2026.md) — primary research source for accuracy trajectory
- [`research/spec-to-code-validity-2026.md`](research/spec-to-code-validity-2026.md) — per-category failure modes

---

*This table reflects the trust gradient as of 2026-05-12; next review 2027-Q1.*
