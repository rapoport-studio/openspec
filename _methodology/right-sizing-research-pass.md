# Right-sizing research-pass framework

> Authored 2026-05-20 from `openspec/archive/2026-05-20-right-size-research-pass-framework/` Phase 2 synthesis (Refs RAP-734).
> Default decision-tool for sizing OpenSpec change research-passes (0 / 1 / 3 / 5+ streams).
> Override is always allowed; the tree's job is to make the override visible.

---

## 1. The decision-tree

| # | Question | If YES | If NO |
|---|---|---|---|
| **Q1** | Can you name **one specific product decision** the research must inform? | → Q2 | **SKIP** research. Come back when the decision is named. |
| **Q2** | Cost-of-wrong-decision ≤ 1 Agent-day to correct (cheap reversal: copy, UI tweak, default value, single-file feature)? | **PASS 0** (ship-and-measure) | → Q3 |
| **Q3** | Is the decision **reversible** after ship (positioning can pivot, channel mix can rotate, default can flip)? | **PASS 1** (one focused stream) | → Q4 |
| **Q4** | Does **existing internal knowledge** (prior archive, locked positioning, published competitor data, locked Q-acks) already answer the question? | **PASS 1** (validation-only stream — confirms what's already named) | → Q5 |
| **Q5** | Does the decision domain fall in **positioning · pricing · ICP · trust · regulatory · architecture**? | **PASS 3** (parallel structured streams) | **PASS 1** (one focused stream is sufficient even for irreversible code/feature work) |
| **Q6** | Are **multiple downstream changes** (>3 successor PRs) gated on this decision AND insufficient external precedent exists? | **PASS 5+** (e.g., 4-agent specialist commission) | Stop at the smaller pass from Q3-Q5. |

**Default at every node: choose the smaller pass.** The tree's bias is toward less research, not more — Stream #3 found that 55% of this studio's archive shipped with pass 0 successfully, and pass-5+ changes cost 4–6× Pavel-days with currently unmeasurable ROI.

---

## 2. Per-leaf rationale

| Leaf | Rationale | Source citation |
|---|---|---|
| **Q1 SKIP** | Naming the decision is the cheapest filter — Stream #1 Pattern 3 ("Decision-first gate") found that named decisions resolve in 5–8 interviews ≥85% of the time, while un-named decisions consume unbounded research. | Lit-review § Pattern 3 (Rachitsky wiki; Nielsen NN/g 5-user rule) |
| **Q2 PASS 0** | Stream #3 cross-tab: 17/31 sampled archives (55%) shipped pass-0 successfully. Stream #1 Pattern 4 ("MVP-as-research"): when cost-of-wrong is below the cost of one Agent-day, build the thinnest testable version and measure. For this studio: one Forge build costs ~$2–6 and 30 min — break-even research budget is near zero for cheap-to-correct decisions. | Stream #1 § Silence (one-person Stage-0 AI economics); Stream #3 § Learning 1 |
| **Q3 PASS 1** | Reversibility is the discriminator most absent from the literature but most predictive in this studio's archive. Stream #1 Pattern 1 (Switch-to-saturation): 7–10 interviews surface the causal mechanism; one targeted stream is enough when reversal cost is bounded. Stream #2 outlier (Equal Experts): the only EU studio with a detailed methodology that explicitly **minimises** pre-engagement research — they hit a velocity threshold where more research returns diminishing yields. | Stream #1 § Patterns 1, 3; Stream #2 § Outliers (Equal Experts) |
| **Q4 PASS 1 (validation)** | When the question is already answered by internal precedent or locked Q-acks, the research's job collapses to validating the prior answer in new context. One focused stream — typically Dunford's "best-customer lens" (5–20 interviews, count less important than customer-type fit) — is sufficient. | Stream #1 § Pattern 2 (Dunford); Stream #3 § Learning 2 (research-pass usage concentrates in 3 domains — implies validation pattern for repeat queries in same domain) |
| **Q5 PASS 3 (positioning/pricing/ICP/trust/regulatory/architecture)** | Stream #3 § Learning 2: **all pass-3+ entries fall in these clusters** (founder-track, entity-system, home-expansion, funding-strategy, trust-research, validate-personal-data). No code change or spec-sync change exceeded pass 1. Domain predicts pass size more reliably than reversibility alone in this studio's archive. | Stream #3 § Learning 2 (load-bearing); Stream #2 § Learning 3 (research scope correlates with decision reversibility, not engagement size — but Stream #3's domain-specific finding supersedes for this studio). |
| **Q5 PASS 1 (code/feature)** | Stream #3 confirmation: zero non-positioning/non-trust changes in the sample exceeded pass 1 and remained right-sized. Code and feature decisions surface their own signal via build transcripts, PR reviews, and test failures — these are the studio's "in-flight measurement loop" that substitutes for pre-decision research. | Stream #3 § Learning 1; Stream #1 § Pattern 4 (Ries — measurement replaces research) |
| **Q6 PASS 5+** | The literature is mostly silent on Stage-0 multi-stream commissions (Stream #1 explicit Silence #4: "Research for internal methodology decisions" goes unaddressed). Stream #3 found exactly two pass-5+ archives in the sample (`add-funding-strategy`, `apply-trust-research-p0`) costing 6 and 2.5 Pavel-days respectively. Reserve pass 5+ for the rare case where multiple downstream changes are gated AND no external precedent exists — e.g., novel regulatory pattern, first-of-its-kind architecture, founder identity question with no prior anchor. | Stream #1 § Silence 1, 4; Stream #3 § Learning 5; Stream #2 § ThoughtWorks 8-week-discovery model (irreversible legacy modernisation) |

---

## 3. Cost-per-stream reference table

| Pass size | Pavel-days | Agent-days | € (synthetic-data only) | € (with paid panel) | When the cost pays back |
|---|---|---|---|---|---|
| **PASS 0** (skip) | 0 | 0 | 0 | n/a | Always, when Q2 holds. Default mode of this studio. |
| **PASS 1** (one stream) | 0.5–1.5 | 0.5 | ~0 | €100–500 (interview incentives if real users) | Within 1 successor change (research output cited as load-bearing in next 1–3 weeks) |
| **PASS 3** (parallel structured) | 1.5–3 | 1.5 (Agent dispatch wall-clock ≈ 30 min; review + reconcile = bulk of Pavel-time) | ~0 | €500–3000 (paid PSM panel per Stream #2 / RAP-661 Q-VM-2 default) | Within 2–3 successor changes; positioning/trust/pricing decisions held for ≥6 months |
| **PASS 5+** (specialist commission) | 4–6 | 4–6 | ~0 | €2000–8000+ (multi-specialist panels + interview incentives) | Within 6–12 months; only when downstream gates ≥3 PRs AND external precedent absent |

**Stream #3 calibration:** the two pass-5+ archives sampled (`add-funding-strategy` 6 Pavel-days, `apply-trust-research-p0` 2.5 Pavel-days) confirm the 4–6× cost differential vs pass-0. ROI on those two not yet measurable (14-day evaluation floor neutralises). Re-calibrate this table after Stream #3's re-extraction in 2–3 weeks (per Stream #3 § Caveats).

---

## 4. When to override the tree (escape hatches)

The tree is the **default**; these conditions justify an explicit override. Override must be named in the change's `proposal.md § Conflicts surfaced`.

| # | Escape hatch | Direction | Why |
|---|---|---|---|
| 1 | **Irreversible regulatory or legal obligation** (publication of legal bundle, data-processing terms, signed-into-effect contracts) | Override **UP** to pass 3-5 even if Q3 says reversible | Wrong decision crystallises into compliance debt; literature consensus across all 3 streams. |
| 2 | **First-of-its-kind architecture choice** (new state-management substrate, new package boundary, new external auth provider) | Override **UP** by one pass size | Stream #1 silence on AI-architecture decisions; Stream #2 ThoughtWorks pattern; Stream #3 only one such archive sampled (insufficient precedent). |
| 3 | **External precedent abundant and tight** (5+ public competitor examples solving the exact same problem with consistent answers) | Override **DOWN** to pass 0–1 even when Q5 domain says PASS 3 | Stream #2 finding: research scope correlates with **decision reversibility**, not engagement size — and AI-consultancy tier pattern shows 2-hour entries are valid for well-precedented decisions. |
| 4 | **Velocity threshold reached on this specific decision domain** (≥3 archives in same domain ratified ≥30 days ago without successor reversals) | Override **DOWN** by one pass size | Stream #2 Equal Experts outlier: "more research yields diminishing returns above a velocity threshold." Self-correcting once internal track-record accumulates. |
| 5 | **Pavel-day budget critical** (current sprint's Pavel-day count >10 + other gates pending) | Override **DOWN** to pass 0 with explicit "ship-and-measure; revisit after data" addendum in proposal | Stream #1 Pattern 4 (Ries) + Stream #3 § Learning 1 (55% pass-0 default). Operating-model reality: the studio is currently capacity-constrained on Pavel-days, not on Agent-days. |

---

## 5. Retroactive application to `validate-marketing-surface-stage-1` (RAP-661)

**Recommendation: Outcome 2 — right-size 5 → 2 streams + carve-outs.**

Per `archive/2026-05-26-validate-marketing-surface-stage-1-cancelled/design.md § 3` (target was cancelled 2026-05-26 per its `CANCELLED.md` — three of four successor changes of the parent umbrella shipped before any stream dispatched), the original 5 streams are:

| # | Stream | Decision unlocked | Domain | Reversibility |
|---|---|---|---|---|
| 1 | Qualitative Buyer Researcher (JTBD) | Q-EM-1 (Hero direction) + buyer-language list | Positioning | Partly reversible (positioning can pivot) |
| 2 | Pricing Strategist | **Q-POS-2** pricing floors × 5 services | Pricing | **Hard to walk back once first quotes go out** |
| 3 | Category-Design Architect | `/method` + `/openspec` framing | Positioning | Reversible (re-framing possible) |
| 4 | DevRel Distribution Strategist | Q-EM-7 + 12-week channel calendar | Channel | Reversible (channels can rotate) |
| 5 | Marketing Measurement Designer | Phase 6 archive "did marketing work" check | Internal infra | Reversible (instrumentation can be re-cut) |

Applying the tree to each stream's lead question:

| # | Stream | Q1 named? | Q2 cheap? | Q3 reversible? | Q5 domain? | **Tree verdict** |
|---|---|---|---|---|---|---|
| 1 | Buyer JTBD | YES (Q-EM-1) | NO (positioning rework ≥ 2 weeks) | Partly | Positioning → PASS 3 | **COMMISSION** |
| 2 | Pricing | YES (Q-POS-2) | NO (signed contracts) | NO | Pricing → PASS 3 | **COMMISSION** |
| 3 | Category-Design | YES (`/method` framing) | Yes-ish (re-framing reversible) | YES | Positioning, but Q3=yes → PASS 1 | **BACKLOG** (Dunford 5–20 best-fit interviews sufficient post-launch) |
| 4 | DevRel Distribution | YES (Q-EM-7) | YES (channels rotate cheaply) | YES | Channel — falls outside Q5 cluster → PASS 1 fallback, but Q2=YES → **PASS 0** | **SHIP-AND-MEASURE** |
| 5 | Measurement | YES (Phase 6 check) | YES (re-instrumentation cheap) | YES | Internal infra — outside Q5 cluster → PASS 0 | **SHIP-AND-MEASURE** |

**Net carve-out actions:**

1. **COMMISSION** Streams #1 (Buyer JTBD) + #2 (Pricing) — both gate decisions whose wrong-cost dominates 2–4 weeks of downstream work and whose domain (positioning + pricing) sits squarely in the Q5 PASS-3 cluster. Stream #3 § Learning 2 corroborates: all pass-3+ archives in this studio fall in positioning / trust / platform-strategy. Pricing especially: irreversibility = signed contracts.
2. **BACKLOG** Stream #3 (Category-Design) — record as a follow-up issue; commission only if usage signal post-launch shows the `/method` framing is misreading. Tree verdict PASS 1 is satisfied by reusing already-locked Q-acks from `lock-positioning-trajectory` archive + the competitor-landscape research from `_methodology/research/competitor-landscape-synthesis.md`.
3. **SHIP-AND-MEASURE** Streams #4 (Distribution) + #5 (Measurement) — both Q2=YES (cheap reversal). Channel mix and measurement instrumentation are exactly the decisions the literature (Stream #1 Pattern 4 — Ries) and this studio's archive (Stream #3 § Learning 1 — 55% pass-0) say should be shipped and adjusted from live signal. **Recommend cancelling these two streams outright** and folding their decisions into the Phase 1 expand-marketing-surface-stage-1 ship.

**Outcome 2 cost delta** vs the originally-scoped 5 streams:

| Plan | Streams commissioned | Pavel-days | € (paid panel for pricing per Q-VM-2 default) |
|---|---|---|---|
| Original (5 streams, RAP-661 as scaffolded) | 5 | 3.5 | €500–1500 |
| **Recommended (Outcome 2, this synthesis)** | **2** | **1.5–2** | **€500–1500** (concentrated on pricing) |
| Savings | 3 streams (60%) | 1.5–2 Pavel-days | €0 (cost stays in pricing where it pays back) |

Pavel's call to ratify or override; recommendation is published per D-RS-4. If ratified, the next action is a successor PR against `validate-marketing-surface-stage-1/proposal.md` (now `archive/2026-05-26-validate-marketing-surface-stage-1-cancelled/proposal.md`) that:

- Strikes Streams #3, #4, #5 from § 3 The five streams.
- Adds § "Right-sizing recommendation accepted (2026-05-20)" citing this methodology doc.
- Files a fresh RAP issue for the Stream #3 (Category-Design) backlog placeholder.
- Folds Stream #4 + #5 decisions into `expand-marketing-surface-stage-1/tasks.md` as "ship-and-measure" instrumentation items.

---

## 6. How to apply on next scaffold

Future `openspec/changes/<slug>/tasks.md` files include this single line at the head of their first phase:

```markdown
- [ ] **Sizing call.** Apply `openspec/_methodology/right-sizing-research-pass.md`
      decision-tree to this change's lead decision; record the tree's verdict
      and any override in `proposal.md § Conflicts surfaced`.
```

The verdict format in proposal.md:

```markdown
## Conflicts surfaced

### Sizing call (per `_methodology/right-sizing-research-pass.md`)

- **Lead decision:** <one-sentence decision name>
- **Tree verdict:** PASS <0 / 1 / 3 / 5+>
- **Reasoning trace:** Q1 <YES/NO> → Q<2..6> <YES/NO> → PASS <N>
- **Override (if any):** <escape-hatch # from § 4> — <one-line rationale>
- **Final scoping decision:** <PASS N>
```

Two-line addition; surfaces the sizing decision to every reviewer.

---

## 7. Calibration & re-evaluation

This methodology was authored from a **single synthesis pass over three streams** completed 2026-05-14. Two known calibration gaps:

1. **Stream #3 outcome-data is 14-day-floored**: 30 of 31 sampled archives are too recent (< 2 weeks) to evaluate for decision-quality outcome. Re-run Stream #3 extraction in 2–3 weeks (target: ≥ 2026-06-04) to backfill outcome distribution and re-validate § 3 cost-per-stream calibration.
2. **Founder-track diagnostic shops** (Stream #2 § Gap note): 4 target studios returned zero public methodology; sample is 12/15 vs target. If this category becomes load-bearing, re-commission Stream #2 with private-source supplementation.

Both gaps are acknowledged; the methodology is shippable today and re-callibratable on a 2–3 week cadence.

---

## 8. References

- Stream outputs (archived):
  - `openspec/archive/2026-05-20-right-size-research-pass-framework/research/stream-1-lit-review.md`
  - `openspec/archive/2026-05-20-right-size-research-pass-framework/research/stream-2-competitor-benchmark.md`
  - `openspec/archive/2026-05-20-right-size-research-pass-framework/research/stream-3-archive-extraction.md`
- Originating change: `openspec/archive/2026-05-20-right-size-research-pass-framework/` (archived 2026-05-20 after Pavel ratified Outcome 2 in PR #1545).
- Retroactive target: `openspec/archive/2026-05-26-validate-marketing-surface-stage-1-cancelled/` (RAP-661 — cancelled 2026-05-26; see folder's `CANCELLED.md`).
- Linear: RAP-734 (this methodology); RAP-735/736/737 (the 3 streams, all Done).
