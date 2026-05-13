# Positioning decision framework

> **Status:** active spec. Created from change [`lock-positioning-trajectory`](../changes/lock-positioning-trajectory/), authored May 2026 as part of Phase 1 deliverable per `design.md § 1.1`. <!-- openspec-refs: skip-line --> <!-- TODO RAP-682: surfaced at guard-tighten; resolve in followup -->
>
> **Owner:** Pavel Rapoport (founder). At Stage 4, ownership opens to candidate: operations specialist, founder-fellow (Path B), or head-of-product (Path C).
>
> **Review cadence:** quarterly — first scheduled review September 2026. Each review appends a dated section per § 5 (output convention). Off-cycle reviews are forced by drift triggers per § 4.
>
> **Companion to:** [`_methodology/research/deployment-tier-emergence-and-12-month-ladder.md`](./research/deployment-tier-emergence-and-12-month-ladder.md) (the research substrate — read it first for context on Stages and the fork space), [`_methodology/positioning.md`](./positioning.md) (public/private positioning contract), [`_methodology/rapoport-studio-engagement.md`](./rapoport-studio-engagement.md) (engagement charter).
>
> **Feeds into:** every future positioning proposal (`amend-positioning-stage-N`, `decide-positioning-stage-5-fork`). All future positioning proposals read this file before authoring.
>
> **Do not modify directly** outside an archive step. Amendments arrive via successor proposals (`amend-positioning-stage-N`). The mutation rule from `openspec/decisions.md` applies: ratified decisions are not edited; they are superseded.

---

## § 0 Status / Owner / Review cadence

| Field | Value |
|---|---|
| Status | Active |
| Owner | Pavel Rapoport |
| Current stage | Stage 0 → Stage 1 transition in progress (Q3 2026) |
| Review cadence | Quarterly (Sept 2026, Dec 2026, Mar 2027, June 2027, …) |
| Off-cycle triggers | See § 4 |
| Last review | — (no reviews yet; inaugural Sept 2026) |
| Next review | September 2026 |

---

## § 1 Ladder reference

The studio operates on a **six-stage evolution ladder** documented in full in [`_methodology/research/deployment-tier-emergence-and-12-month-ladder.md § 7`](./research/deployment-tier-emergence-and-12-month-ladder.md). Read that file for the complete narrative: stage-by-stage state descriptions, proof signals, and changes from prior stage.

**Do not duplicate the ladder narrative here.** The research file is the substrate. This framework is the normative spec — it defines *which observable events* move the studio from one stage to the next, not what each stage looks like from the outside.

Keeping them separate means the research file can be updated (or marked archived as the landscape shifts) without triggering a positioning amendment. Conversely, this framework can be tightened without touching the research narrative.

**Brief ladder summary for orientation:**

- Stage 0 — Foundation built, zero paying clients (current as of May 2026)
- Stage 1 — First paying engagements; network has real people with credits
- Stage 2 — Discovery productized; Pavel no longer the single bottleneck
- Stage 3 — Niche recognition; warm-intro inbound replaces cold outreach
- Stage 4 — Methodology beyond Studio; external adoption; delegation layer active
- Stage 5 — Fork: Path A (Boutique destination) / Path B (Productized methodology) / Path C (Platform / SaaS)

---

## § 2 Per-stage observable signals

Every stage transition is an **event**, not a date. Each signal has three components:

1. **Trigger predicate** — the condition that must be true.
2. **Verification source** — where the truth of the predicate is checked.
3. **Witness** — the human or process that observes the trigger and files the successor proposal.

The research file uses calendar-anchored language ("Q3 2026 ~3 months") as operating-rhythm guidance. That guidance is not the spec contract. If a signal fires early, transition is approved early. If it slips by a quarter, that is data — not a failure of the framework.

### § 2.1 Stage 0 → Stage 1 transition signal

> Already true as of this writing (May 2026). Captured here for completeness and to anchor what "S0" means.

| Component | Value |
|---|---|
| Trigger predicate | First paid Discovery invoice issued **AND** first specialist with a credited engagement on `/work` (case study live, not placeholder) |
| Verification source | Studio accounting / Stripe row showing first paid Discovery receipt **AND** `/work` page diff showing at least one non-placeholder case study with specialist credit |
| Witness | Pavel — files successor proposal `amend-positioning-stage-1` once both conditions are simultaneously met |

**Why both conditions?** Issuing a Discovery invoice without a public work credit means the commercial model is live but the network-as-signal is not yet visible. The `/work` credit is the trust signal that distinguishes "Studio started billing" from "Studio is operating as described."

### § 2.2 Stage 1 → Stage 2 transition signal

| Component | Value |
|---|---|
| Trigger predicate | Discovery engagements shipping on a ≤2-week cadence for 6 consecutive weeks **AND** Pavel did not lead the most recent Discovery end-to-end (another specialist carried it from kickoff to deliverable delivery) |
| Verification source | Linear `discovery/*` issue cadence query (6-week window, ≤14 days between consecutive close dates) **AND** project_member assignment audit on the most recent Discovery linear issue (Pavel is reviewer, not sole assignee) |
| Witness | Pavel — quarterly review checklist includes a cadence query; first review where both predicates are true triggers the successor proposal `amend-positioning-stage-2` |

**Why the bottleneck condition?** Discovery cadence alone does not prove it is productized — it might just mean Pavel worked harder. The bottleneck condition is the leading indicator that the *operating model* has changed, not just the pace.

### § 2.3 Stage 2 → Stage 3 transition signal

| Component | Value |
|---|---|
| Trigger predicate | ≥50% of the last 8 inbound `/contact` form submissions in any rolling 4-week window are warm intros (submitter names a specific referrer by person or by engagement) rather than cold outbound |
| Verification source | `inbox` table (Supabase `nifagnmgwoqkplegsicy`, eu-central-2) — `referrer_type` field classification on the 8 most recent rows (manual classification if `referrer_type` is not yet populated) |
| Witness | Pavel — quarterly review; first review where 4-week window satisfies the predicate triggers `amend-positioning-stage-3` |

**Why 50% warm, not 100%?** 100% warm inbound would signal no marketing surface, which creates a single-point-of-failure on word-of-mouth. 50% warm is the indicator that the referral network is *working and compounding*, while outbound can continue selectively.

### § 2.4 Stage 3 → Stage 4 transition signal

| Component | Value |
|---|---|
| Trigger predicate | First publicly-cited use of OpenSpec methodology by a non-Studio author appears — defined as a blog post, OSS repo README, conference talk, or podcast where the author (a) was not paid by the studio for the mention and (b) uses OpenSpec as the example of a methodology they adopted or are evaluating |
| Verification source | Manual quarterly review — Google/Bing alert on "OpenSpec Rapoport", plus GitHub search for `openspec` README mentions outside `rapoport-studio` org |
| Witness | Pavel — quarterly review; first instance satisfying both conditions (unpaid, adoptive framing) triggers `amend-positioning-stage-4` |

**Why unpaid and adoptive, not just mentioned?** The claim at S3 is "niche recognition". Recognition means someone picked up the methodology on their own terms. A studio-commissioned mention or a neutral citation ("some studios use OpenSpec") does not count; it must be an adoption or serious evaluation signal.

### § 2.5 Stage 4 → Stage 5 (fork open) transition signal

| Component | Value |
|---|---|
| Trigger predicate | 12 months of sustained operation past the S3 → S4 signal **AND** the Stage 5 fork indicator data (§ 3) has been actively collected for at least 2 quarterly reviews at Stage 4 |
| Verification source | Date of S3 → S4 signal from `amend-positioning-stage-4` proposal + two quarterly review records in this file (§ 5 appended sections) that include Path A / B / C indicator summaries |
| Witness | Pavel — the second qualifying quarterly review triggers `decide-positioning-stage-5-fork` proposal |

**Why 12 months + 2 reviews, not just one signal?** Stage 5 fork commits to a multi-year direction (boutique destination / productized methodology / SaaS platform). The decision requires evidence accumulated over time, not a single event. The 12-month floor ensures the studio has operated at Stage 4 long enough that its direction is visible from revenue shape, network shape, and methodology adoption shape, not from a single good quarter.

---

## § 3 Stage 5 fork criteria

Stage 5 fork (Path A, B, or C) is held open by D-POS-4 until the dedicated `decide-positioning-stage-5-fork` proposal in Q2 2027 (or whenever the § 2.5 transition signal fires). This section pre-commits the *criteria* so that Stage 4 data collection has explicit targets.

> **Hard rule (D-POS-4):** No Stage 5 fork ratification is attempted before the § 2.5 transition signal fires. Premature commitment destroys the Stage 1–4 evidence base that must inform the choice.

### § 3.1 Path A indicator set — Boutique destination

Choose Path A when **all** of the following are true at the Stage 4 review where the fork decision is made:

| # | Indicator | Threshold |
|---|---|---|
| A1 | Revenue composition | ≥80% of trailing-12-month revenue comes from custom engagements (Discovery + full engagements), not from subscription, certification, or marketplace |
| A2 | Network depth | ≥5 specialists doing repeat client work (credited on ≥2 separate `/work` entries each) |
| A3 | External methodology adoption | ≤2 publicly-cited non-Studio uses of OpenSpec methodology (Path B indicator A3 not triggered) |
| A4 | Pavel engagement energy | Pavel is still doing hands-on engagement work and rates it as energising on the 90-day self-assessment (§ 3.4) |
| A5 | Platform technical debt | No full-team platform build-out required — `apps/studio` has not accumulated technical debt that blocks engagement work without a dedicated engineering sprint |

### § 3.2 Path B indicator set — Productized methodology

Choose Path B when **all** of the following are true:

| # | Indicator | Threshold |
|---|---|---|
| B1 | External adoption | ≥3 publicly-cited uses of OpenSpec methodology by non-Studio authors (matching the S3→S4 signal criteria plus 2 more) |
| B2 | Demand signal direction | Inbound demand for "how do you do this?" (teaching, consulting, methodology licensing) outweighs "do this for us" (engagements) by ≥2:1 in the most recent quarterly contact classification |
| B3 | Pavel evangelism capacity | Pavel has demonstrated capacity to spend ≥30% of working time on evangelism (talks, blog, training, methodology publication) over a 90-day period without engagement quality dropping below client-satisfaction threshold |
| B4 | Specialist self-sufficiency | At least one specialist (not Pavel) has carried an OpenSpec-style change folder end-to-end — proposal, design, tasks, implementation, archive — without Pavel reviewing the methodology dimension |

### § 3.3 Path C indicator set — Platform / SaaS

Choose Path C when **all** of the following are true:

| # | Indicator | Threshold |
|---|---|---|
| C1 | Platform pull | Three or more peer studios have **unsolicited** (not solicited by Studio) asked for access to internal Forge / Muse / canvas-surface tooling |
| C2 | Platform stability | Pavel did not commit to `packages/forge`, `packages/muse`, or `apps/studio/src/lib/` for 90 consecutive calendar days (indicates tooling is stable enough to be a product, not perpetually under construction) |
| C3 | Paid platform access | At least one peer studio has paid — in cash or in methodology contribution recognized by Pavel — for early access to the tooling surface |
| C4 | Capital access | Studio has access to either ≥€200k runway from engagement revenue or pre-seed capital matching the ratified funding-strategy posture (per `add-funding-strategy` when archived) |

### § 3.4 Tie-breaker rule

If two paths' indicator sets are simultaneously satisfied at the Stage 4 review, the tie-breaker is **Pavel's energy allocation** measured over the prior 90 days.

**Instrument:** a simple weekly self-rating in the personal log (1–5 scale): "this week's work on [engagement / methodology publishing / platform tooling] felt energising (+) or depleting (−)." The 90-day aggregate for each work-type determines which path's daily work Pavel found more energising.

**No-fork fallback:** if no path's indicator set is fully satisfied at the Stage 4 review, the framework returns "stay at Stage 4." Collect another quarter of data and re-evaluate. The no-fork outcome is not a failure — it is the framework protecting against premature commitment.

---

## § 4 Drift triggers (off-cycle review)

Quarterly review is the standard cadence. Any of the following events forces an **off-cycle review** before the next scheduled quarter:

| # | Trigger | What to re-evaluate |
|---|---|---|
| DT-1 | DeployCo or Anthropic-fund-class deployment infrastructure reaches MD/RO/IL via local SI partners — evidence: a named Moldovan, Romanian, or Israeli SI (≥50 engineers) announces a formal DeployCo or Anthropic+GS partnership | Founder Track and Client Track positioning; evaluate whether L3 encroachment is affecting L6 pipeline (see ladder research file § 3 deployment stack) |
| DT-2 | Tessl achieves Spec Registry adoption outside their own customer base — evidence: ≥3 third-party blog posts, conference talks, or OSS repos in a single quarter reference Tessl's Spec Registry as a methodology standard they adopted or are adopting | Re-evaluate Stage 5 fork criteria C1/C2; Path C ceiling may lower; Path A and B priorities may shift; see research file § 4.3 |
| DT-3 | Two or more L5 boutique acquisitions in 12 months following Tomoro — evidence: any of Sona, Aigency, DeepKlarity cluster, or comparable (≤200-engineer applied-AI boutique) acquired by an L3/L4 player | Re-evaluate Path A revenue-ceiling assumption; if L5 boutiques are being acquired at premium, Path A exit becomes more attractive at Stage 5 |
| DT-4 | Funding posture changes — evidence: `add-funding-strategy` archives with an outcome that differs from bootstrapped baseline, OR funding posture ratified in any future `amend-funding-strategy-*` proposal | Re-evaluate Q-POS-2 pricing floor; trigger D-POS-PRICE ratification |
| DT-5 | A Q-POS-1..7 ratification turns out to be wrong based on first-engagement evidence — evidence: Pavel observes, or a specialist flags, that a ratified decision (e.g. MD/RO first, sprint-based pricing) is actively costing real engagement outcomes | Triggers `amend-positioning-stage-1-Q-POS-N-correction` proposal; do not silently edit the decision |

**Drift triggers are not reasons to re-evaluate the framework itself.** They re-evaluate Stage-N specifics within the framework. The framework changes only via successor positioning proposal with full proposal/design/tasks triplet.

---

## § 5 Quarterly review contract

Every quarterly review of this framework produces an **appended dated section** at the end of this file:

```
## Quarterly review — YYYY-MM-DD

### Signal status
- S0 → S1: [fired YYYY-MM-DD / not yet]
- S1 → S2: [fired YYYY-MM-DD / partial (describe what is met / what is missing) / not yet]
- S2 → S3: [...]
- S3 → S4: [...]
- S4 → S5: [...]

### Drift trigger surface
- DT-1: [no signal / signal (describe)]
- DT-2: [...]
- DT-3: [...]
- DT-4: [...]
- DT-5: [...]

### Stage 5 indicator data (only when at S4)
- Path A indicators: [status per row]
- Path B indicators: [status per row]
- Path C indicators: [status per row]
- 90-day energy allocation: [summary]

### Q-POS residual
- Q-POS-2 (pricing floor): [still gated / ratified — see D-POS-PRICE-N]
- Q-POS-4 (intake mechanism): [still deferred / ratified at S2 signal]
- Q-POS-6 (case study format): [still deferred / ratified at first engagement]

### Actions
- [If a transition signal fired: file amend-positioning-stage-N proposal]
- [If a drift trigger fired: note and evaluate per DT-N description]
- [Other]

### Owner note
[One paragraph from Pavel on what this quarter taught that the signal data does not capture.]
```

**Read inputs for each review:**
1. This file (signal definitions)
2. Current stage's recent Linear issue cadence (for S1→S2 signal)
3. `inbox` table query (for S2→S3 signal)
4. Google alert / GitHub search (for S3→S4 signal)
5. Drift trigger surface scan (§ 4)
6. Stage 5 indicator data accumulation (only when at S4)
7. Q-POS residual list (any opened questions still unresolved)

**Owner until Stage 4:** Pavel.

**Owner at Stage 4:** candidate-list opens — operations specialist (if Stage 4 has a non-founding operational role), founder-fellow (if Path B seems most likely), head-of-product (if Path C seems most likely). Transition of ownership is explicit, not automatic; it requires a brief handoff note added to the review section and an update to the frontmatter owner field.

If a transition signal fires during a review, the review output **triggers** a successor proposal — it does **not** ratify the transition. The review names the signal; the proposal ratifies the stage change and the Stage N+1 specifics.

---

## § 6 Decisions captured

This section is the decision register for the `lock-positioning-trajectory` change. It is append-only. Decisions are not edited; they are superseded by entries in successor proposals.

### D-POS-1 — Scope is Stage 1 + decision framework, not a 12-month plan

This proposal locks Stage 1 specifics and registers the decision framework. It does not lock Stage 2, 3, or 4 specifics. Each of those gets its own successor proposal at the time the prior stage's transition signal fires. Stage 5 fork is held open by D-POS-4 until Q2 2027.

**Rationale:** pre-commitment to Stage 2+ specifics before Stage 1 evidence exists would destroy the evidence base that makes the Stage 5 fork decision meaningful.

### D-POS-2 — Decision framework is a first-class methodology doc

`_methodology/positioning-decision-framework.md` (this file) is the source of truth for stage-transition logic. It lives next to `studio-charter.md`, `risk-taxonomy.md`, `trust-gradient.md`, and `specialist-network.md` — the operational discipline corpus.

**Rationale:** the charter is the founding document; it changes rarely. Operational discipline files (like this framework) are read at every quarterly review and amended at each stage transition. Keeping them separate avoids forcing charter mutation on every per-stage proposal.

### D-POS-3 — Stage 1 vocabulary lock follows the Founder Track precedent

The cold-pitch forbidden list from `archive/2026-05-08-add-founder-track D-F4` (`spec`, `methodology`, `process`, `review gate`, `discipline`, `framework`) is extended with a positive vocabulary lock. The 5 ratified positive tokens are: **architecture**, **artifact**, **diligence**, **agent-native**, **orchestration** (D-POS-VOCAB-1).

**Rationale:** the negative list bans vocabulary. Without a positive list, every new Hero / `/services` / `/discovery` copy exercise re-invents the vocabulary from scratch. The positive list is the symmetric completion.

### D-POS-4 — Stage 5 fork is out of scope until Q2 2027

Path A (Boutique destination), Path B (Productized methodology), Path C (Platform / SaaS) are **not chosen by this proposal**. The framework defines the criteria (§ 3) by which they will be evaluated, but ratification is held until `decide-positioning-stage-5-fork` in Q2 2027 (or when the § 2.5 signal fires, whichever is later).

**Rationale:** pre-commitment in Stage 0 destroys the Stage 1–4 evidence base. This is a hard rule, not a preference.

### D-POS-5 — Per-stage transitions are observable signals, not calendar dates

Stage transitions are triggered by **events** (§ 2), not by dates. The research file's calendar narrative is operating-rhythm guidance. If a signal fires early, transition is approved early. If it slips, that is the data.

**Rationale:** calendar-anchored transitions create pressure to advance without meeting the conditions. Signal-based transitions keep the framework honest.

### D-POS-6 — Pricing decisions in this proposal are bound to Stage 1 only

The pricing floor and engagement model ratified by this proposal (per Q-POS-2 and Q-POS-7 resolutions) apply to Stage 1 engagements. Revision happens via `amend-positioning-stage-2` when Stage 2 signal fires — not by direct edit.

**Rationale:** mirrors the mutation rule in `openspec/decisions.md`: ratified decisions are not edited; they are superseded.

### D-POS-7 — Decision framework is read by every future positioning proposal

All future positioning amendments read this file as their first input. This makes the framework the single consistency point across the trajectory.

**Rationale:** without this rule, each successor proposal may evolve the positioning logic independently, and the framework becomes stale while the stage-specific specs evolve around it.

---

### Q-POS-1 → D-POS-PRIORITY-1 — Geographic priority: MD/RO Client Track first

**Decision:** MD/RO Client Track is the primary geographic focus for Stage 1 engagements. IL Founder Track is kept warm but is not actively driven in 2026 without an inside-partner.

**Resolved by reference:** `_methodology/research/competitor-landscape-synthesis.md § 3.D` (IL = referral-or-nothing; locked) + `archive/2026-05-08-add-founder-track/proposal.md § 2` (IL is Founder Track ICP, MD/RO is Client Track ICP; two parallel funnels, not merged). No contradiction between the two; both confirm the same direction.

**Implication:** Stage 1 outreach and engagement prioritisation focuses on Moldovan and Romanian clients (Client Track). IL outreach is deferred until an inside-partner is identified or Stage 2 signal nears.

---

### Q-POS-2 → pending D-POS-PRICE — Pricing floor: gated by funding-strategy

**Status:** open. Hard sequencing dependency on `add-funding-strategy`.

The pricing floor (€30k / €50k / €80k / differentiated) is downstream of the funding posture: bootstrapped studio → different floor than investor-targeted studio. `add-funding-strategy` has not archived as of this writing. Q-POS-2 is held per the default resolution rule (option a: hold until funding-strategy archives).

**Close trigger:** `add-funding-strategy` archives with a ratified D-FUND-* entry on bootstrapped vs investor-targeted posture. At that point, D-POS-PRICE-N is filed in the next quarterly review or off-cycle if the gap is significant.

---

### Q-POS-3 → D-POS-DISCOVERY-1 — Discovery deliverable contract

**Decision:** Discovery (€5k fixed, 3–4 weeks) delivers: **OpenSpec capability corpus** + **Mirror domain map** + **first-pass render.** No sprint plan, no sprint execution, no recommendation document is included in the standard Discovery scope.

**Resolved by reference:** `archive/2026-05-04-add-client-onboarding/proposal.md` ratified the Discovery offering at €5k/3–4 weeks with these three outputs. The add-client-onboarding ratification governs the route's commercial structure; this decision confirms that structure holds for Stage 1 and is not extended.

**Implication:** clients who want a sprint plan after Discovery proceed to a separate full engagement (per D-POS-ENGAGEMENT-1). The Discovery is bounded; extensions are a new engagement, not a Discovery scope creep.

---

### Q-POS-4 → deferred — Specialist intake: invite-only default, close-trigger S2 signal

**Status:** deferred. To be registered in `openspec/open-questions.md` at archive.

Default at Stage 1 is invite-only (Pavel personally invites specialists). Rolling or cohort intake requires engagement volume and credit infrastructure that Stage 1 does not yet have.

**Close trigger:** S1 → S2 signal nears (Discovery shipping at 2-week cadence, 6-week run), at which point intake capacity must scale to match the productized cadence.

---

### Q-POS-5 → D-POS-VOCAB-1 — Positive vocabulary tokens

**Decision:** 5 ratified positive tokens for Stage 1 cold-pitch surfaces (Hero, `/services`, `/discovery`, Founder Track surfaces):

1. **architecture** — names the core output; platform-agnostic; works for both Client and Founder
2. **artifact** — names the deliverable shape; not "spec" (forbidden); permanent, ownable, transferable
3. **diligence** — signals rigour without triggering methodology-alarm; resonates with Series A founders
4. **agent-native** — names the platform orientation; differentiating vs generic "AI-powered"
5. **orchestration** — names the studio's thesis (AI as orchestration substrate); from `add-founder-track D-F4` approved list

**Cross-check:** none of these appear on the forbidden list (`spec`, `methodology`, `process`, `review gate`, `discipline`, `framework`). No collision.

**Enforcement:** same mechanism as the forbidden list — PR review. Future ESLint rule path per `add-founder-track § Cold-pitch vocabulary constraint`.

---

### Q-POS-6 → deferred — `/work` case study format: close-trigger first engagement

**Status:** deferred. To be registered in `openspec/open-questions.md` at archive.

No case studies exist yet. Ratifying format before the first one risks over-specifying the wrong shape. Default when ready: hybrid (problem → approach → outcome + 1–2 visuals + quote + named team credits, per `design.md § 6 Q-POS-6 default`).

**Close trigger:** first engagement completes and case study is drafted. At that point, the format emerges from real material and can be ratified with evidence.

---

### Q-POS-7 → D-POS-ENGAGEMENT-1 — Engagement model: sprint-based

**Decision:** Stage 1 engagement model is **sprint-based: €10–15k per 2-week sprint × N sprints**, with optional retainer for ongoing advisory. Pricing subject to update when Q-POS-2 (pricing floor) is ratified via D-POS-PRICE-N.

**Founder Track equity variant:** deferred to Stage 2. Q-POS-1 resolved to MD/RO Client Track first, so the equity-engagement variant (which was conditionally linked to IL Founder first) does not activate at Stage 1. Founder Track clients in MD/RO engage via the standard sprint model.

**Implication:** client-facing materials (engagement charter, `/services` copy) use sprint-based framing. Fixed-price engagements are scoped as N-sprint contracts with a defined sprint count, not as an open-ended fixed price.

---

*Quarterly reviews are appended below this line, oldest first.*
