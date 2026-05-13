---
title: Funding Strategy
status: active
owner: pavel
audience: [pavel, ai, founder, future-operator]
tldr: Five-channel non-dilutive capital stack. ODA M1 primary 2026 target. OSS lever activated. Equity deferred until validated wedge.
---

# Funding Strategy

> Source of truth for the studio's capital strategy. Answers "should we apply, and on what terms?" in under five minutes for any agent or operator.
>
> This file is public methodology — strategy and decision rules are open. Specific monetary outcomes (amounts granted, internal runway numbers) live in the private `funding-pack` repo.
>
> Change origin: `openspec/archive/2026-05-13-add-funding-strategy/`.

---

## Decisions (D-FUND-1 … D-FUND-9)

| Decision | Ruling |
|---|---|
| **D-FUND-1** No equity before validated wedge | Equity instruments (SAFE, convertible, priced round) are deferred until €100K ARR + repeatable acquisition channel. See `red-lines.md § Business red lines`. |
| **D-FUND-2** Five-channel contract | Channels A–E (state grants · cloud credits · SaaS perks · OSS funds · debt/RBF) are formalised; free stacking allowed against different cost lines. |
| **D-FUND-3** Master-pack authored once | Single document set (business plan, financial model, pitch deck, one-pager, bio) lives in `funding-pack`; reused with cosmetic per-programme edits. |
| **D-FUND-4** ODA M1 primary 2026 target | Highest E[X] single application (€17.5K midpoint). Phase 1 priority: submit within two weeks of MITP residency confirmation. |
| **D-FUND-5** Forge / Muse / Maestro OSS under Apache 2.0 | No commercial-model conflict; opens Channel D (NLnet, STF, OpenAI Codex OSS); reinforces trust-gradient Pillar A. Phase 4 executes the release. |
| **D-FUND-6** Linear is the funding tracker | Each application is a Linear issue with `funding/<channel>/<programme>` label under the funding epic. Status: `draft → submitted → under-review → granted \| rejected \| withdrawn`. |
| **D-FUND-7** Quarterly review | Review at each quarter close. Off-cycle triggers: single application granted, MRR crosses €30K, validated wedge reached. |
| **D-FUND-8** Public/private contract | Strategy + eligibility matrix are public. Specific outcomes, master-pack documents, per-application correspondence are private (`funding-pack` repo). |
| **D-FUND-9** Channel E deferred | Debt + RBF (EU4Business, Levenue, Float, Round2) held until €30K MRR sustained 3 months. |

---

## Five channels

The channels are mutually compatible by construction — different cost lines, no double-funding clauses, stackable into a single 12-month plan.

| Channel | Pays for | Form | Lead time | Where it lives |
|---|---|---|---|---|
| **A — State / EU grants** | Working capital, equipment, payroll | Cash (reimbursement-after-spend MD; pre-financed EU) | 4–12 weeks | ODA, MITP fund, EDIH, USAID FTA |
| **B — Cloud + AI credits** | API/inference, hosting, compute | Service credits, time-bound | 1–4 weeks | Anthropic, AWS, GCP, Cloudflare, Microsoft |
| **C — SaaS perks** | Tooling subscriptions | Discount or free-period | 1–2 weeks | Linear, Notion, Stripe Atlas, Figma, Vercel |
| **D — OSS funds** | Open-source maintainer time | Cash + community access | 4–10 weeks | NLnet NGI, Sovereign Tech Fund, OpenAI Codex OSS |
| **E — Debt + RBF** | Growth runway against revenue | Repayable | 2–6 weeks | EU4Business credit line, Levenue, Float, Round2 |

### Stacking rules

1. Channels A, B, C, D may be stacked freely against any cost line.
2. Channel E is deferred until D-FUND-9 trigger (€30K MRR × 3 months).
3. Two channel-A grants may **not** double-fund the same line item. Financial model carries per-grant tagging.
4. Only one cloud-credit programme active per workload at a time. Primary AI pin: Anthropic Claude.

### Channel ownership

| Channel | Primary owner | Secondary owner |
|---|---|---|
| A | Pavel | accountant (when retained) |
| B | Pavel | platform agent (auto-renewal monitoring) |
| C | Pavel | — |
| D | Pavel | OSS maintainer when external co-maintainer joins |
| E | Pavel + accountant | — |

---

## Decision rules

### Apply / skip rule (§ 4.1)

Apply if and only if `E[X] > cost_of_application × 5`. Skip if the win is plausible but the application drag is disproportionate.

### Master-pack-first rule (§ 4.2)

Do not start a programme-specific submission until master-pack documents are complete. Exception: rolling-deadline programmes with `now-or-never` urgency (NLnet 2026-06-01, ODA M1 first-come-first-served) may be authored in parallel with master-pack within the same Phase 1 timebox.

### Channel-balance rule (§ 4.3)

If E[X] aggregated across channel A drops below 30% of the 12-month plan total, escalate channels D (OSS) and B (cloud credits with VC-nomination tier).

### OSS-eligibility rule (§ 4.4)

Channel D applications require concrete OSS commits visible at submission time. Phase 4 enforces: Apache 2.0 LICENSE + minimal docs + one demo case live in public mirror.

### Reject rule (§ 4.5)

Decline any grant whose conditions require:

1. Equity issue or option grant — D-FUND-1 red line.
2. IP assignment to the granting authority.
3. Geographic-presence exclusivity incompatible with EU-aligned operating model.
4. Headcount commitments without a clawback ramp.

Each reject reason is documented in the Linear issue's resolution.

---

## Application pipeline

### Lifecycle

```
draft → submitted → under-review → granted | rejected | withdrawn
```

Each transition is a Linear status change. `submitted` carries a timestamp and submission proof. `under-review` covers programme-officer iteration windows. Terminal states are final.

### Issue contract

Each application is a Linear issue:

| Field | Source |
|---|---|
| Title | `<Programme> — <amount> — <yyyy-Q?>` |
| Label | `funding/<channel>/<programme>` |
| Parent | Funding strategy epic |
| Description | Link to `funding-pack/applications/<id>/submission.md`, eligibility narrative, E[X], deadline |
| Status | per lifecycle above |
| Resolution | filled at terminal status; granted carries amount + conditions |

---

## Public / private contract

### Public

1. This file — `_methodology/funding-strategy.md`.
2. The eligibility matrix — `changes/add-funding-strategy/research/program-eligibility-matrix.md` (→ archive path after close-out).
3. The decision rules above.
4. The list of channels.
5. The OSS release of Forge / Muse (consequence of D-FUND-5) — public mirror repos.

### Private

1. Specific monetary outcomes per application (until announcement window opens).
2. Master-pack documents — `github.com/rapoport-studio/funding-pack` (private repo).
3. Per-application correspondence with programme officers.
4. Internal runway calculations and cash-on-hand snapshots.
5. Pavel's individual eligibility details (diaspora documentation, age verification artefacts).

### Boundary mechanism

Public lives in `openspec/_methodology/` and `openspec/changes/` (this repo). Private lives in `github.com/rapoport-studio/funding-pack`. No application narrative is committed to the public repo. Mirrors `_methodology/positioning.md § public-private contract`.

---

## Quarterly review

### Recalculation triggers

1. Programme officially closes or reopens.
2. Studio meets a previously-unmet eligibility criterion (revenue, OSS release, MITP residency confirmation).
3. Peer company publicly receives the same grant (adjusts probability ±10pp).

Off-cycle triggers: any single application granted; MRR crosses €30K; validated wedge reached.

### Review template

> Instantiate this template at each quarterly close. Append as a dated section at the bottom of this file.

```markdown
## Quarterly review — Q[N] YYYY

**Date:** YYYY-MM-DD
**Reviewer:** Pavel

### Open `draft` issues
<!-- List issues still in draft; should they progress or close? -->

### `under-review` issues
<!-- Any pending iterations from programme officers? -->

### Terminal outcomes since last review
<!-- Programme | Status | Amount | Calibration delta -->

### New programmes opened
<!-- Any new Linear issues to file? -->

### E[X] recalculation
<!-- Updated probability midpoints; next-quarter prioritisation table -->

### Channel balance check
<!-- A > 30% of 12-month plan? If not, escalate D + B. -->

### Action items
<!-- Bullet list with Linear issue references -->
```

### Owner and cadence

Owner: Pavel (founder). Reviewed at quarter close. Future Muse-agent automation (E[X] recalculation from public sources) is a follow-up under `_methodology/agents.md`.

---

## Cross-references

- `_methodology/red-lines.md § Business red lines` — no equity dilution; no IP assignment
- `_methodology/risk-taxonomy.md` — L1 Regulatory (grant compliance) + L3 Product (funding viability)
- `_methodology/studio-charter.md § Capital` — summary posture for sub-product agents
- `current/platform.md § Capital strategy` — gateway framing for MITP residency
- `changes/add-funding-strategy/research/program-eligibility-matrix.md` — live probability surface (→ archive after close-out)
- `changes/add-funding-strategy/research/non-dilutive-channels.md` — full channel landscape
- `changes/add-funding-strategy/design.md` — full design spec; this file is the condensed methodology surface
