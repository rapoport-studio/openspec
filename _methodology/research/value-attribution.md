# Research thread — value attribution in AI-augmented work

> **Status:** open research, no commitments attached.
> **Owner:** Pavel (founder).
> **Opened:** 2026-05-07.
> **Feeds into:** [`specialist-network.md § 8`](../specialist-network.md) (compensation, currently placeholder), [`studio-charter.md`](../studio-charter.md) (product philosophy), eventual `openspec/current/network.md § Compensation` after the network's first three certified specialists.

---

## The thesis

The studio is a multi-specialist environment where AI is the orchestration substrate, not a feature. In that environment, "what is a person's work worth?" is not a question that hourly rate alone answers.

Money is one signal, and a strong one. When a specialist knows their hour clears at €X, they orient — they decide whether to take the next engagement, whether to deepen a discipline, whether to stay in the network at all. The studio cannot pretend money is optional.

But money alone doesn't capture:

- Whether the work is **felt as valued** in this new environment — recognition, status in the network, durability of the seat.
- The worth of **new professions** the AI substrate is creating — prompt curators, canvas hosts, contribution reviewers, muse trainers — for which no market rate exists yet, and against which "freelancer hourly" anchors badly.
- How value is **distributed** across the network so it doesn't concentrate in one node (founder, top specialist, single discipline). The network is the studio's compounding asset; concentration breaks the asset.

This is product philosophy, not an operational policy. The studio's commercial thesis (per [`specialist-network.md § 0`](../specialist-network.md)) depends on a network that compounds — and a network only compounds if its members feel correctly valued along axes the network itself surfaces.

---

## The questions

### Q1 — What dimensions of human contribution should the studio measure?

Money is one axis. Candidates for others, named for orientation only — not committed:

- **Reuse** — was an idea, prompt, or pattern from this person picked up by another engagement?
- **Depth** — did this person's contribution change the *shape* of an engagement, or refine an existing shape?
- **Mentorship** — did certified-author work by this person help an apprentice cross to certified?
- **Domain rarity** — is this person filling a discipline the network has few members in?
- **Outcome-linked** — did engagements where this person contributed produce measurable outcomes for the client?

Research goal: which of these are **observable in muse / canvas runtime data** without instrumentation theater, and which are real signals vs. vanity? Cheap-to-observe + hard-to-game is the bar.

**Mechanism candidates under investigation:**
- [`openspec-block-authorship.md`](./openspec-block-authorship.md) — block-level attribution inside OpenSpec markdown. Investigates whether git blame already covers it, what granularity survives reformatting, and how to handle the idea-vs-text asymmetry common in studio drafting.

### Q2 — How are new professions valued before market data exists?

The studio creates roles that don't appear in any payroll database — *prompt curator*, *muse trainer*, *canvas host*, *contribution reviewer*. There is no benchmark hourly rate to anchor on.

Two known failure modes:

- **Anchor to nearest existing profession** ("a copywriter charges $X, so a prompt curator should charge close to that") — under-prices the new work and signals to the specialist that the role isn't real.
- **Pick a number that feels right** — drifts under negotiation pressure; no defensibility either to the specialist or to the engagement client.

Research goal: what is the *mechanism* by which a new profession gets its first defensible price in the studio? Candidates: outcome-share at first and conversion to fixed once a range is observed; founder-set with publicly stated reasoning and a revisit cadence; collective-set among certified specialists. Which mechanism is administrable by a small studio without becoming a wage-board?

**Adjacent thread:** [`open-source-positioning.md`](./open-source-positioning.md) treats the **publication vehicle** for new professions (public charter pages under CC-BY-SA) as a positioning question. The naming dimension upstream of pricing is referenced there as a forward "Q5" gap — addressing it may warrant a fifth question in this file.

### Q3 — How does the studio prevent value concentration?

The default failure mode of curated networks is winner-take-most: one or two specialists capture most engagement flow, the rest atrophy. The studio's pitch is that the **network itself** is the asset — concentration breaks the pitch.

Possible regulation mechanisms, none committed:

- **Engagement diversity floor** — no specialist takes more than N% of any one quarter's roster slots.
- **Contribution-to-library royalty** — promoted prompts earn recurring revenue independent of engagement load (gestured at in [`specialist-network.md § 8`](../specialist-network.md)).
- **New-discipline incubation budget** — fixed studio spend per quarter on disciplines with one or zero specialists, to seed redundancy.
- **Founder-time obligation** — Pavel's hours partially committed to apprentice mentorship, not just production work.

Research goal: which mechanisms are administrable by a studio of this size without becoming a bureaucracy? Which are felt as fair by specialists vs. felt as redistribution-by-fiat?

**Evidence base added 2026-05-12.** [`social-positioning-analogs.md`](./social-positioning-analogs.md) § 8 catalogues 8 organisations strong on both transparency and equitable distribution (Stocksy patronage, Outlandish proportional-to-hours, Hypha 35%-clip-breakdown, Loomio equal-shares, Up & Go 95/5, Nava equity-for-all, Resonate 70% to artist, Buffer COL-formula). The single load-bearing observation: *transparency is structural, not rhetorical* — anchored in legal entity, public document, or operational tooling. § 10 distills the implication for Q3: three concrete mechanism shapes — patronage / worker-owner-core / PBC-universal-equity. **Pick one; do not blend.** Adjacent thinking in `social-positioning-analogs.md § 9` (Benkler peer production triad + Ostrom 8 commons design principles + Mazzucato value-extraction frame) supplies the academic fundament.

### Q4 — What does the specialist need to *see* to feel valued?

Even if the studio measures correctly, value-felt depends on what's surfaced back to the specialist. Hourly earnings alone — even when accurate — don't communicate "your work compounded." Reuse counts, library-promotion notifications, "your prompt ran in 3 engagements this quarter" — these may be cheaper to deliver and more durable to receive.

Research goal: identify the smallest set of dashboards or digests a specialist sees that produce the felt-valued effect, without becoming gamified.

**First live instance (added 2026-05-07).** [RAP-119](../../archive/2026-05-08-add-team-onboarding-canvas/) `design.md` § Project instructions, **Block B Course-meta directive**: *"When a methodology decision happens — creating a new triplet, naming a capability, choosing where a paragraph goes — Muse briefly explains what it is doing and why. Concise. Two sentences max. Not lecturing."* This is a different shape of answer than dashboards: visibility built into the Muse turn at the moment of the decision, not surfaced retrospectively in a digest. Sessions 1–4 will show whether members report this as helpful or condescending, whether Muse stays terse or drifts. Empirical wiring lives in [`rap-119-as-research-bed.md`](./rap-119-as-research-bed.md) § 3.1.

> **Update 2026-05-08 — RAP-119 cancelled.** Block B was specified but
> never deployed; helpful-vs-condescending question never tested.
> Directive verbatim text preserved in
> [`multi-author-canvas-instructions-template.md`](./multi-author-canvas-instructions-template.md)
> § Block B. First live instance now resets to "first time the
> multi-author template runs" — pending the next multi-author
> engagement.

---

## Things this thread is *not*

- It is **not** a compensation proposal. [`specialist-network.md § 8`](../specialist-network.md) remains the place where commitments live; this file is upstream input.
- It is **not** a tracking system spec. Designing schemas for contribution ledgers or decision provenance before Q1 is answered is premature.
- It is **not** founder-philosophy in isolation. Anything this thread concludes needs validation against actual specialist behavior before it becomes policy.

---

## How this thread closes

Either:

1. The questions get answered enough that a `proposal.md` for `define-network-compensation-v1` can be drafted under `openspec/changes/`, and this file links to it with `Status: superseded`.
2. The questions resolve into "not enough data yet; revisit when N specialists have shipped M engagements" — this file becomes a watch-item with an explicit re-open trigger.

Either way, **no commitments are made from this file**. It is a research thread, not a spec.

---

## Adjacent prior thinking

| Source | Relationship |
|---|---|
| [`studio-charter.md`](../studio-charter.md) | Bootstrap charter. Claims about how studio-led platforms create value sit upstream of this thread. |
| [`specialist-network.md § 0`](../specialist-network.md) | The multiplier thesis (OpenSpec + Muse + contributed prompts) is what makes value measurable on axes other than time. |
| [`specialist-network.md § 8`](../specialist-network.md) | The compensation placeholder this thread is meant to inform. |
| [`specialist-network.md § 11`](../specialist-network.md) | `Q-NW-5` (library promotion threshold) is the operational shadow of Q1 here. |
| [`rap-119-as-research-bed.md`](./rap-119-as-research-bed.md) | Observation protocol that wires Q1–Q4 to RAP-119 surfaces (D7 triplets, Block B course-meta, D4 soft-enforcement, three named domains). First empirical test of all four questions; conclusions fold back here post-Review-week. |
| [`social-positioning-analogs.md`](./social-positioning-analogs.md) | 21-organisation evidence base for Q3 mechanism candidates. Surfaces three structural shapes (patronage / worker-owner-core / PBC-universal-equity) and the seven-trust-register framework adjacent to Q4. Academic fundament (Benkler / Ostrom / Mazzucato) in § 9. |

---

## Change log

| Date | Author | Change |
|---|---|---|
| 2026-05-07 | Pavel (founder) | Opened thread. Captured Q1–Q4. No commitments. |
