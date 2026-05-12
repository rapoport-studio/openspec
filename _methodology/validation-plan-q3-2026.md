---
title: Validation plan — Canvas Studio thesis (Q3 2026)
status: experiment-plan
opened: 2026-05-12
target_window: 2026-06 to 2026-07 (4–6 calendar weeks of execution; total 7 weeks including this artefact week)
owner: pavel
upstream:
  - _methodology/positioning.md
  - changes/define-canvas-studio-thesis/
audience: [pavel]
---

# Validation plan — Canvas Studio thesis (Q3 2026)

> **Status: experiment plan, awaiting kickoff.** This document specifies a 4–6 week customer-development experiment that tests the hypothesis declared in [`positioning.md`](./positioning.md). The plan is *not* in execution; execution opens when Pavel files a Linear issue against this plan and assigns himself.
>
> **Why this exists separately from `positioning.md`**: positioning is meant to outlive any single experiment. This plan is *time-boxed and disposable* — its content is consumed when the experiment closes. Conflating them would either bloat positioning permanently, or strip it after each experiment cycle.

---

## 0. The single hypothesis being tested

Restated for crispness. The hypothesis:

> A **product architect** (per [`positioning.md § 5.2`](./positioning.md), one of the eight archetypes) will pay between **\$3,000 and \$15,000** for a **packaged blueprint** — i.e. a complete OpenSpec corpus + repo scaffold + cost / risk / timeline document for a specific platform domain — delivered by Rapoport Studio without prior implementation work, on the basis of one landing page and one 30-minute conversation.

Three components must each be true for the hypothesis to graduate:

1. The persona exists and is reachable (≥ 10 candidate buyers respond to outreach).
2. The persona finds the value proposition compelling (≥ 3 of 10 say "yes, would pay $X").
3. The persona converts to commitment (≥ 1 actually pays within the experiment window).

If component 1 fails, the persona definition is wrong — the experiment cannot even run.
If component 2 fails, the proposition is wrong — the offer needs reformulating.
If component 3 fails, the commitment threshold is wrong — interest exists but not at this price-point.

---

## 1. Pre-flight (Week 0 — this change merging)

The artefacts produced by [`changes/define-canvas-studio-thesis`](../changes/define-canvas-studio-thesis/) are the pre-flight deliverable. Once those are in repo:

- [ ] Pavel reviews `positioning.md` and either ratifies the thesis-as-written (proceed to Week 1) or requests revisions (loop until ratified).
- [ ] Pavel files a Linear issue against this plan with target start date (and any deviations from the plan recorded in the issue description).
- [ ] Pavel sets aside ~1 day/week for the duration of the experiment (≈ 5 working days total across 4–6 weeks).

---

## 2. Week 1 — Blueprint selection and scoping

Resolve `Q-CS-1` (per `design.md § 8` of the change folder): which blueprint domain anchors the experiment?

### 2.1 Candidate blueprints

Three pre-identified candidates, each with rationale:

| Candidate | Domain | Why it might win | Why it might lose |
|---|---|---|---|
| **AI-orchestrated internal platform** | Studio's own stack — Next.js 16 + Supabase + Cloudflare + Forge + Muse + Linear, packaged as a buyable starter | Studio knows it cold (already built it once); buyer benefits from operational dogfooding; appeals to A1 (second-time founder) and A6 (refugee from big tech) | Niche — only buyers who specifically want spec-driven AI-native architecture. Small TAM but well-aligned. |
| **GDPR-aware telehealth platform** | Healthcare platform with Article 9 special-category-data discipline (drawing on existing telegram canvas surface work) | Studio has accumulated GDPR + Art. 9 discipline through `add-tg-canvas-surface`; differentiated; appeals to A2 (domain expert crossing) | Heavy regulatory scope — buyer needs to be sufficiently along to see the value of the discipline. May not match early-stage founders. |
| **B2B SaaS marketplace starter** | Two-sided marketplace with onboarding, escrow, dispute, billing — generic but well-shaped | Largest TAM (every founder building B2B marketplace); appeals to A1, A4, A5 | Most competitive (Vercel templates, ThirdwebShopify-clones). Hard to differentiate without strong narrative. |

### 2.2 Selection criteria

Choose the blueprint that maximises the *signal* the experiment produces. That means: pick the one whose buyer profile we can *most credibly identify and reach* in 4 weeks. Not the one with the largest TAM in the abstract.

Pavel's task in Week 1:

- [ ] Identify by name 10 candidate buyers from his existing network for *each* of the three candidate domains.
- [ ] Pick the candidate with the highest count of named, reachable buyers.
- [ ] Record the choice + rationale in `_methodology/research/canvas-studio-validation-2026-q3/blueprint-selection.md`.

### 2.3 Blueprint authoring

Once selected, draft the blueprint in Week 1.5–2:

- [ ] Author the OpenSpec corpus for the domain — `current/` capability specs covering 6–10 capabilities; ratified `D-X-N` decisions; risk register.
- [ ] Generate the repo scaffold — fork an existing studio scaffold and adapt for the domain; commit to a private GitHub repo named `<domain>-blueprint-v1`.
- [ ] Produce a 1-page architecture-overview document (PDF or markdown) — readable in 5 minutes, communicates the value.
- [ ] Set up the Stripe payment link — fixed price (e.g. \$7,500), product name "<Domain> Blueprint v1", description references the architecture-overview document.
- [ ] Set up the landing page on Notion or static site — sections: what's in it, who it's for, what it costs, FAQ, "buy" button (Stripe link), "talk first" button (Calendly link).

The landing page is *not* deployed under `rapoport.studio/blueprints/<slug>` — that surface does not exist yet (per `apps/web` state in `positioning.md § 1.3`). The temporary surface is intentional; production deployment is post-validation.

---

## 3. Weeks 2–6 — Customer development

The core of the experiment.

### 3.1 Buyer interviews — 10 conversations

#### 3.1.1 Sourcing

Resolve `Q-CS-3`: network-only or cold?

Default plan: **mixed** — 6 from Pavel's existing network (warm), 4 from cold sourcing (LinkedIn / Twitter / founder communities). Reasoning:

- Pure-network sourcing biases toward "people who already trust Pavel" — they may say "yes, would pay" out of friendship, not market signal.
- Pure-cold sourcing has high acquisition cost in 4 weeks (low response rate; long-form interview commitment is hard cold).
- 60/40 mix gives signal triangulation: warm conversations test the proposition; cold conversations test whether the proposition stands without the prior trust relationship.

#### 3.1.2 Interview structure (30 minutes)

Each interview follows the same script. The structure is asymmetric — front-loaded with discovery, back-loaded with the offer:

```
0-5 min   : Open. "Tell me what you're working on / thinking about building."
            Rule: do not pitch. Do not mention canvas / blueprint / OpenSpec.
            Goal: find their actual current pain.

5-15 min  : Dig. Probe specifics — team size, stage, prior attempts,
            what they tried that didn't work, where they are stuck.
            Rule: questions only. No "we have something for that" hints.

15-20 min : Pivot. "I'm working on something that might be relevant — can I
            describe it?" Pause. If yes: 2-minute description of the
            blueprint product (what's in it, what it costs, how it gets used).

20-25 min : The ask. "If this existed today, packaged as I described,
            for $X — would you buy it for your current project?"
            Wait for the answer. Do not soften the question.
            Then: "Why? What would change your answer to a clearer yes?"

25-30 min : Close. Offer to send the landing page link.
            Note follow-up commitment.
```

#### 3.1.3 What gets recorded

For each interview, a structured notes file at `_methodology/research/canvas-studio-validation-2026-q3/<interview-slug>.md`:

```markdown
---
interviewee: [name or pseudonym]
sourcing: [warm | cold]
date: YYYY-MM-DD
medium: [zoom | call | in-person]
duration_min: NN
recording: [yes | no — note consent]
---

## Their context
[3–5 bullet points: what they're building, stage, team, prior attempts]

## Pain
[their stated pain, in their words where possible]

## Their reaction to the proposition
[direct quote of their response to the "would you buy" ask]

## Would they pay
- [ ] Yes, at the asked price
- [ ] Yes, but at a lower price (specify: $)
- [ ] Maybe, with conditions (specify)
- [ ] No (specify reason)

## Follow-up
- [ ] Sent landing page
- [ ] Booked follow-up
- [ ] Left as cold

## Verbatims worth preserving
[direct quotes that surprised, struck, or contradicted the thesis]

## My read
[Pavel's interpretation — what this signal means]
```

#### 3.1.4 What we are listening for (positive and negative)

Positive surprises (raise confidence):
- Buyer describes the pain *before* the pivot, in language matching the thesis.
- Buyer's "would you pay" answer is yes *with a specific reason*, not yes-to-be-polite.
- Buyer asks unprompted about scope expansion ("can the blueprint also include X?").
- Buyer asks how soon they can have it.

Negative surprises (lower confidence):
- Buyer's pain is upstream of architecture (e.g. they don't have a co-founder; they don't know their market). Architecture is not yet their bottleneck.
- Buyer asks the price and falls silent — price discomfort.
- Buyer says they would rather hire a person than buy a document — sees value in human time, not artefact.
- Buyer says they would pay, but only after seeing the architecture work for someone else first — chicken-and-egg signal.

The negative surprises are *not* failures — they are signal about which adjacent thesis might be the real one.

### 3.2 Specialist interviews — 5 conversations

Same shape, different ask. Targets: 5 individuals who could plausibly become network specialists (architects, ex-CTOs, senior consultants in domains the studio cares about — clinical, fintech, gov, design systems).

Script difference at minute 15–25:

```
15-20 min : "I'm working on something that might be relevant — a platform
            where specialists like you can publish architectural blueprints
            as products and earn royalties on each sale, plus run direct
            engagements through the same surface. Take-rate is 30%, royalty
            split 70/30. Can I describe it?"

20-25 min : "If this existed today — would you publish? How many blueprints
            do you have in your head right now that could be packaged?
            What would have to be true for you to commit a month to
            publishing your first one?"
```

The signal we want from specialist interviews: **publishing intent** + **scope of what they'd publish**. A specialist saying "I have 3 blueprints in my head" is high-signal even without immediate commitment.

### 3.3 Cadence

5 buyer interviews + 2 specialist interviews per week, target. Realistic floor: 3+1 per week. The plan tolerates slippage as long as totals (10 + 5) close by end of Week 6.

---

## 4. Week 7 — Synthesis and decision

### 4.1 Synthesis artefact

A single synthesis document: `_methodology/research/canvas-studio-validation-2026-q3/synthesis.md`. Sections:

- **Buyer "would-pay" tally** — count of yes / yes-cheaper / maybe / no across the 10 interviews.
- **Buyer pain themes** — clustered themes from the discovery sections of the interviews; ranked by frequency.
- **Actual conversion** — how many of the 10 buyers paid (real money in Stripe) within the experiment window?
- **Specialist publish-intent tally** — count of would-publish / would-not-publish across the 5; total blueprints they'd publish if the platform existed.
- **Surprises** — what did the experiment teach that the thesis did not anticipate?
- **Recommendation** — Pavel's read of which decision-matrix outcome applies.

### 4.2 Decision matrix

The criteria for graduate / pivot / kill, applied mechanically to the synthesis:

| Outcome | Buyer "yes, would pay $X" tally | Actual paid (Stripe) | Specialist publish-intent | Action |
|---|---|---|---|---|
| **Graduate** | ≥ 3 of 10 | ≥ 1 | ≥ 2 of 5 say yes | Open follow-on change `promote-canvas-studio-to-current` (per `changes/define-canvas-studio-thesis/tasks.md § P2.9`); update `positioning.md § Hypothesis status` to *graduated YYYY-MM-DD* |
| **Pivot — price** | ≥ 5 of 10 say yes-cheaper *and* ≥ 0 paid at advertised price | 0 at advertised | Any | Re-run a 4-week iteration at the median price the "yes-cheaper" cohort named; same blueprint, same persona |
| **Pivot — persona** | ≤ 3 of 10 say yes BUT 5+ buyers describe an *adjacent* pain that the thesis could address with reformulation | Any | Any | Update `positioning.md § Buyer archetypes` based on observed pain; re-run with revised persona profile |
| **Pivot — domain** | Buyers say they would pay but for a *different* blueprint domain than the one tested | Any | Any | Re-run experiment with the domain interviewees pulled toward |
| **Kill — interest absent** | ≤ 2 of 10 say yes; no clear adjacent pain pattern | 0 | Any | Update `positioning.md § Hypothesis status` to *falsified YYYY-MM-DD* with rationale; this change folder archived as captured-idea memory; studio continues with engagement-charter operating model unchanged |
| **Kill — commitment absent** | 5+ of 10 say yes BUT 0 paid AND no follow-up commitments materialise | 0 | Any | Same as kill — interest absent; the apparent yes was social compliance, not real demand |

### 4.3 Decision authority

Pavel decides. The matrix is mechanical to *limit* improvisation, not to *replace* judgement. If the data are ambiguous, Pavel records which outcome he's choosing and why; future agents can review the call.

---

## 5. Budget

### 5.1 Time budget

| Phase | Calendar weeks | Pavel days | Cumulative days |
|---|---|---|---|
| Pre-flight (Week 0) | 1 | 1 | 1 |
| Blueprint selection + scoping (Weeks 1–2) | 2 | 2 | 3 |
| Buyer + specialist interviews (Weeks 3–6) | 4 | 4 | 7 |
| Synthesis + decision (Week 7) | 1 | 1 | 8 |
| **Total** | **7 weeks** | **~8 days** | |

This is ~1 day/week of Pavel attention for 7 weeks. The experiment is designed to *not* derail current Platform Foundation (AI-75) work.

### 5.2 Cash budget

| Line item | Cost |
|---|---|
| Notion / static-site landing page | \$0 (existing tooling) |
| Stripe payment processing | 2.9% + \$0.30 per transaction; no setup |
| Calendly / scheduling | \$0 (existing tooling) |
| Outreach (LinkedIn, founder communities) | \$0 (manual) |
| Interview recording + transcription (optional) | \$0–\$50 (Otter / Pavel's existing tooling) |
| **Total fixed** | **\< \$100** |
| **Worst case if all 10 buyers refund** | \$0 (no refunds at this scale; refund cost is operational only) |

### 5.3 What spending more would *not* buy

The experiment is deliberately scoped to be cheap so that scope-creep is visibly the wrong move. Adding more interviews, paying for cold-outreach services, or building a real `apps/web` landing page do not *improve* the signal — they only delay the decision. Resist.

---

## 6. Risks to the experiment itself

Risks specific to *running* this validation (distinct from risks to the thesis itself, which live in `positioning.md § 8`):

| Risk | Mitigation |
|---|---|
| Interview script drifts toward pitch-mode (Pavel's instinct to sell) | The script is recorded above. Pavel re-reads it before each call. After each call, self-grades whether they pitched or discovered. |
| Sample bias from network-only sourcing | 60/40 mix per § 3.1.1. If cold sourcing fails to produce 4 interviews by Week 4, document the failure and weight the synthesis accordingly — do *not* substitute warm interviews to hit the count of 10 |
| Respondents say yes to be polite | The "would you buy at $X" ask is followed by "Why? What would change your answer to a clearer yes?" — soft-yes signals show up here |
| Single big-name yes carries the count | Synthesis explicitly tags interviewees by sourcing; a graduate decision driven entirely by warm-network yes triggers an additional cold-sample requirement before commitment |
| Pavel runs out of attention mid-experiment | Hard cap at 8 days. If the budget overruns, the synthesis closes with whatever interviews are done, and the decision is made on partial data with the partial count noted |
| Blueprint draft takes longer than 1.5 weeks | The blueprint quality bar is "reviewable in 5 minutes". A 6-week canvas-grade blueprint is *over-investment* at this stage. Set a hard time-box. |

---

## 7. After the experiment

The plan is silent on what happens after the decision because the decision determines the next step:

- **Graduate** → see `changes/define-canvas-studio-thesis/tasks.md § P2.9` for the follow-on change shape.
- **Pivot** → second iteration of this same plan, with the variable identified by the synthesis.
- **Kill** → no follow-on; this plan + the change folder are archived as captured-idea memory; the studio continues per the existing engagement charter.

In all three outcomes, the synthesis document and the 15 interview notes remain in `_methodology/research/canvas-studio-validation-2026-q3/`. They are not deleted. Even the kill outcome has signal that subsequent thinking can build on.

---

## 8. Cross-references

- Thesis being tested: [`positioning.md`](./positioning.md)
- Change that authored this plan: [`changes/define-canvas-studio-thesis/`](../changes/define-canvas-studio-thesis/)
- Specialist commercial surface (sell-side validation pulls from): [`specialist-network.md §§ 12–15`](./specialist-network.md)
- Engagement charter (what the studio reverts to on kill): [`rapoport-studio-engagement.md`](./rapoport-studio-engagement.md)
- Pulse storage for ratified monetization decisions (where validated pricing eventually lands): [`_finance/README.md`](../../_finance/README.md)
- Research-folder authorship convention (frontmatter for interview notes): see [`openspec/archive/2026-05-10-add-research-authorship-convention/`](../archive/2026-05-10-add-research-authorship-convention/)

---

## Change log

- `2026-05-12` — initial draft, authored as part of change [`define-canvas-studio-thesis`](../changes/define-canvas-studio-thesis/). Awaiting kickoff.
