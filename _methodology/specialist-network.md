# Specialist Network — operational spec

> Companion to [`rapoport-studio-engagement.md`](./rapoport-studio-engagement.md). The engagement charter declares specialists as a first-class capability (§ 2 capability #1); **this** document spells out how the network is sourced, NDA'd, trained, certified, matched to engagements, augmented with their own prompts and tools, compensated, and off-boarded — i.e. how it actually works as a platform.
>
> Status: **draft v1, behind the engagement charter.** As of this writing the studio has zero specialists fully onboarded under this spec; Pavel is the only active participant, and his role is `openspec_author_lead`. Treat this document as the design contract under which the network will be built — not as a description of an already-running system.
>
> Once the network has its first three certified specialists this document graduates from `_methodology/` into a proper capability spec at `openspec/current/network.md` (via a normal change — `openspec/changes/promote-network-spec/`). Until then it lives as methodology, free to evolve without archive ceremony. <!-- openspec-refs: skip-line — `promote-network-spec` is a forward-ref to a future change-folder that opens when the third specialist onboards. -->

---

## 0. Why this is the studio's edge

The studio's commercial thesis is not "we have curated freelancers" — agencies have that and it is not a moat. The thesis is:

> **Trained specialists writing OpenSpec inside Muse, with each specialist contributing their own prompts and tools to the platform, produces deeper analysis than any individual freelancer or any agentic system can produce alone.**

Three multipliers stack:

1. **Prose-style OpenSpec discipline** — every specialist writes against the same `proposal.md` / `design.md` / `tasks.md` shape. Cross-engagement reuse becomes possible because the artifacts have the same skeleton.
2. **Muse as the substrate** — every conversation runs in the same canvas, with the same tool stack, with auditable turns. A specialist's session is reviewable, replayable, comparable across engagements.
3. **Specialist-contributed prompts and tools** — over time the network's accumulated prompt augmentations become the studio's compounding asset. A new business-model validator joining in 2027 inherits everything 2026's validators learned.

The first two are operational discipline; the third is what makes this an **engineering platform** and not a Rolodex. This document is the spec for the third.

---

## 1. Specialist domains (open list, not a catalog)

The network is intentionally not pre-bucketed into a fixed list of disciplines. Each engagement's brief generates a **demand vector** of needed disciplines; the network responds by routing specialists who match. The studio adds new disciplines as engagements demand them, not in advance.

Examples of disciplines the studio expects to see early (and currently has placeholder seats for):

| Discipline | What they do in the canvas |
|---|---|
| **OpenSpec authoring** | Write the spec triplets in studio house style. The "load-bearing" specialist on most engagements; baseline cohort. |
| **Business-model validator** | Probe monetization, pricing, unit economics. Stress-test `proposal.md` claims with numbers. |
| **Domain expert** *(per engagement: clinical, legal, finance, regulated industry, etc.)* | Anchor `design.md` claims in domain reality. Catch what generic LLMs hallucinate. |
| **Product copywriter** | Voice, tone, microcopy, marketing-page copy. Especially when the product runs in a non-English market or in a regulated emotional register. |
| **Brand / UX designer** | Tokens, components, surface composition. Often pairs with `design-system.md` work. |
| **Security reviewer** | Threat-model the spec, gate the design choices. Reads RLS and access boundaries before code is written. |
| **Accessibility reviewer** | Reads `ui.md` / `design-system.md` against WCAG. Flags issues before implementation. |
| **Data-privacy / compliance reviewer** | GDPR / HIPAA / sectoral regs. Reads spec for retention, deletion, DSAR, jurisdiction. |
| **Frontend specialist** | Implementation discipline for `ui.md` and surface specs. |
| **Mobile / native specialist** | Per-platform constraints; only when the engagement has a mobile surface in scope. |

Disciplines not in this list are **not refused** — they are added to the registry the moment an engagement legitimately needs them. The list grows with demand.

---

## 2. Lifecycle of a specialist

```
Discovery
   │
   ▼
NDA execution  ─── (with Rapoport Studio SRL, base + addenda)
   │
   ▼
Training cycle ─── (OpenSpec authoring + business-model validation)
   │
   ▼
Certification  ─── (one of three levels; § 5)
   │
   ▼
Profile creation ─ (bio, disciplines, prompts, tools, availability)
   │
   ▼
Engagement match ─ (system-suggested per brief; human-confirmed)
   │
   ▼
Active engagements
   │
   ▼
Off-boarding ─── (per-engagement or full-network exit)
```

Every transition is auditable. The studio holds a per-specialist record across all stages; the per-engagement involvement (canvas access, NDA addendum, deliverables touched) is sub-recorded under the engagement.

**Canvas role on engagement.** When a specialist is matched and confirmed for an active engagement (post-NDA addendum), they receive the `external_collaborator` project role in the canvas's project membership (per change `redesign-canvas-permission-model`, applied 2026-05-15). This role grants canvas read/write access and Mirror authoring rights while excluding the studio's internal OpenSpec (`openspec/current/**`) and methodology (`_methodology/**`) paths. Apprentice-level specialists (§ 4.3) may receive read-only canvas access for shadowing; the `external_collaborator` role is provisioned at Certified Reviewer or higher.

---

## 3. NDA model

### 3.1 Where NDAs live

Specialists sign NDAs **with Rapoport Studio SRL**, not with the engagement client. The client signs one master NDA with the studio; the studio's downstream obligations to the client flow through to each specialist via the specialist's individual NDA with the studio. This keeps the client's surface narrow (one NDA, one counterparty) and concentrates NDA-management discipline on the studio side where it scales.

### 3.2 Two-layer NDA structure

| Layer | Counterparty | Triggered by | Default content |
|---|---|---|---|
| **Base NDA** | Specialist ↔ Rapoport Studio SRL | Specialist onboarding (before any canvas access) | Confidentiality on studio internals, methodology, network composition, other engagements; non-circumvent (specialist cannot bypass studio to contract directly with engagement clients met through the studio); IP assignment of work-product to studio under specialist's compensation contract. |
| **Engagement NDA addendum** | Specialist ↔ Rapoport Studio SRL, scoped to engagement *X* | Specialist invitation to engagement *X*'s canvas | Additional confidentiality terms required by client *X* (e.g. clinical data handling, pre-acquisition due-diligence walls, regulated industry constraints). The addendum cites the master client NDA as upstream source. |

### 3.3 Drafting and templates

The base NDA template is drafted once by the studio's outside counsel (currently i-avocat.md — Andrei & Dmitry, who are also drafting the studio's incorporation). Engagement addenda are derived from the client's master NDA at kickoff; the studio's counsel reviews each addendum before specialists are routed.

The studio does not author legal text; it authors the **process and operational shape** described here.

### 3.4 Failure modes

| Failure | Consequence |
|---|---|
| Specialist receives canvas access before signing base NDA | Hard stop. Studio's internal access-grant tool refuses canvas invitation if `nda_status != signed` on the specialist record. |
| Specialist invited to engagement *X* without signing the *X*-specific addendum | Same. Canvas role for *X* is not provisioned until addendum is countersigned. |
| Specialist withdraws from network before all engagements close | Open engagements continue under the existing NDA; the specialist's profile moves to `inactive`; their attached prompts/tools remain in the studio registry per the IP-assignment clause of the base NDA. |
| Client requires direct privity with specialist (no studio middle layer) | Override recorded in *Appendix B* `D8` of the engagement charter copy; specialist signs an additional direct NDA with the client. **Default is no direct privity.** |

---

## 4. Training

Every specialist completes a two-axis training cycle before being routed to a paid engagement.

### 4.1 Axis A — OpenSpec authoring discipline

Goal: the specialist can author a `proposal.md` / `design.md` / `tasks.md` triplet that meets the studio's prose-style standard without a senior reviewer rewriting the structure.

Training surface:

- Read [`rapoport-studio-engagement.md § 5`](./rapoport-studio-engagement.md) (the OpenSpec convention).
- Read three representative archived changes from a real studio engagement — the exemplar at [`openspec/_training/exemplar/`](../openspec/_training/exemplar/) is the curated starting set (small, single-purpose, spec-only, clean `tasks.md`, no live-system requirements).
- Work through the Axis A curriculum at [`openspec/_training/openspec-authoring-axis-a.md`](../openspec/_training/openspec-authoring-axis-a.md).
- Author a "training change" against a fictional but well-shaped capability brief; the studio's senior reviewer (currently Pavel) provides line-level feedback in canvas.
- Pass criterion: training change lands in studio-internal `_training/<specialist-slug>-graduation/` with reviewer sign-off and zero outstanding structural critiques.

### 4.2 Axis B — Business-model validation methodology

Goal: the specialist can read a `proposal.md` and ask the questions that move it from "plausible" to "validated" — pricing sensitivity, unit-economics floor, retention-driver vs vanity-metric, defensibility against the named competitor in `_research/competitors/`.

Training surface:

- Read the studio's business-model validation framework at [`openspec/_training/business-model-validation.md`](../openspec/_training/business-model-validation.md).
- Run a stress-test pass on three real (anonymized) `proposal.md` files from past studio engagements.
- Pass criterion: written critique of all three reaches the bar of catching at least one structural defect per proposal that the senior reviewer agrees was material.

### 4.3 Outcome: certification levels

| Level | Meaning | Eligible engagement scope |
|---|---|---|
| **Apprentice** | Base NDA signed, training in progress | None (read-only canvas access for shadowing) |
| **Certified Reviewer** | Axis A passed | Can review specs; cannot author root drafts |
| **Certified Author** | Axis A + Axis B passed | Can author `proposal.md` / `design.md` / `tasks.md` end-to-end with senior review |
| **Senior** | Certified Author + ≥ 3 successful engagements + senior-reviewer endorsement | Can senior-review another specialist's training changes; can lead a discipline cluster on an engagement |

Certification status is part of the per-specialist profile (§ 6) and is surfaced in *Appendix B* `D7` of every engagement that uses them.

---

## 5. System-suggested matching

### 5.1 Inputs

Once Mirror produces an engagement's `perception.md` (the seven-step domain map per `_universal.yaml`), the studio's matching layer consumes it and produces a ranked candidate list.

Inputs to matching:

- **Demand vector** — disciplines named or implied by the brief.
- **Confidentiality posture** — base NDA only / clinical-data addendum / regulated-industry addendum / due-diligence walls.
- **Language requirements** — engagement may need RU + EN, or RO + RU, etc.
- **Time-zone constraints** — sync vs async tolerance, kickoff window.
- **Past-engagement quality signals** — see § 7.

### 5.2 Output

A shortlist of 2–5 candidates per discipline, each with:

- Specialist slug + bio link (studio-internal).
- Certification level.
- Past engagements relevant to the demand (anonymized if NDA prevents naming the client).
- Availability for the engagement window.
- Suggested scope (per-change or per-discipline).
- Specialist-contributed prompts and tools relevant to this engagement (§ 6).

### 5.3 Authority

The matching layer **suggests**; the human picks. Pavel and the engagement client review the shortlist together and confirm the roster. The matching layer does not unilaterally provision canvas access. v1 of the matching is allowed to be coarse (rule-based, not learned) — the human-confirm step compensates for matching imperfection.

---

## 6. Specialist-contributed prompts and tools

### 6.1 The contribution model

Every certified specialist maintains a personal **prompt set** and **tool list** inside their studio profile. When a specialist is invited into a canvas, Muse loads those augmentations alongside its base configuration — so the canvas conversation gets the specialist's domain framing without the specialist having to re-explain it each session.

A specialist's profile may contain:

- **Prompt augmentations** — system-prompt fragments that load when the specialist is `external_collaborator` on a canvas. Example: a clinical advisor's profile attaches a fragment that biases Muse to defer clinical claims and surface uncertainty rather than asserting.
- **Tool list** — a curated allowlist of tools (from the studio's master tool registry — `web_search`, `fetch_url`, `domain_research`, plus future additions) that the specialist's typical work needs. Muse loads this list per session.
- **Reference packs** — pinned URLs / file-paths to research artifacts (`_research/competitors/yuna.md`, etc.) the specialist always wants in context.
- **Refusal patterns** — the specialist's documented "won't do" list. The clinical advisor's refusal pattern includes "won't draft text that performs psychotherapy outside the agreed product surface."

### 6.2 Storage

The specialist registry lives in the `specialists` Supabase table (per `openspec/current/db/entities.md § Specialist tables` — answer to Q-NW-3 below). Specialist-specific prompt augmentations are referenced via the `prompt_augmentations_uri` column (a relative path under `packages/muse/specialists/<slug>/` — a deferred package per `platform.md`). Until that package exists, prompt files are markdown stubs under `_methodology/specialists/<slug>.md`. Either way the profile and prompts are studio-internal IP per the base NDA.

### 6.3 Promotion to common library

When a specialist's prompt augmentation has been used across **three independent engagements with positive quality signals**, the studio promotes it to the **common prompt library** (`packages/muse/prompts/library/`) where it becomes available to other specialists in the same discipline. Promotion is a deliberate step:

- The contributing specialist is named in the promoted prompt's metadata.
- The promotion is recorded in the studio's `decisions.md` as `D-NW-<n>` (network-decision).
- The contributing specialist's compensation tier may step up (per § 8).

This is how the network compounds: individual contributions become collective tools, and contribution is rewarded.

### 6.4 Hard constraints on contributed augmentations

- **No client data in prompts.** Specialist prompts must not embed engagement-specific data (names, transcripts, proprietary methodology). They embed only the specialist's own framing.
- **No safety bypass.** Specialist prompts cannot weaken Muse's safety posture (e.g. weaken the agent-vs-bot routing in conversation runtimes; bypass cost guards). The base configuration always overrides.
- **Reviewable.** Every contributed augmentation is human-reviewed by the senior reviewer before activation. Approval is logged on the specialist profile.

---

## 7. Quality and drift signals

The studio does not track specialists by hour-counts or velocity metrics. It tracks **outcome signals** and watches for drift.

### 7.1 Per-engagement signals

For every engagement a specialist participated in:

- **Senior-review delta** — number of structural critiques senior reviewer added before delivery. Trend should fall over time.
- **Client revisions requested** — number of `request revision` outcomes per § 9 of the engagement charter. Trend should fall.
- **Spec-fidelity audit score** — `forge spec` drift report against the specialist's authored capabilities, run after archive.
- **Implementation feedback** — when the engagement is in Hybrid/Full mode and code lands, did the spec hold up?

### 7.2 Per-specialist drift

If a specialist's signals degrade across two consecutive engagements:

- Status moves to `under_review` in the registry.
- A senior reviewer pairs with them on the next engagement (no solo authoring).
- If signals don't recover in the next two engagements, the specialist moves to `inactive`. Their contributed prompts/tools remain in the library; their access is revoked.

### 7.3 The studio's own drift

The studio reviews network-level signals quarterly. Targets (informational, not contractual):

| Metric | Target | Reads from |
|---|---|---|
| Median certification time (Apprentice → Certified Author) | ≤ 6 weeks | per-specialist record |
| Median engagements per certified specialist per year | ≥ 4 | engagement records |
| Specialist-contributed prompts promoted to library | ≥ 2 per year per active specialist | `D-NW-*` decisions |
| Engagement client satisfaction (informal poll, post-archive) | ≥ 8 / 10 | post-engagement note |

---

## 8. Compensation

Compensation model is **outside the scope of this document** because it touches MITP payroll discipline and Israeli/Moldovan tax interactions that are decided in coordination with i-avocat.md. Placeholders only:

- Default: per-engagement fee, with a small uplift for certified-author work over reviewer work.
- Equity in client engagements is **not** a default for specialists. If an engagement client wants to grant equity to a specific specialist, it does so directly to that specialist (with the studio's awareness) — not through the studio.
- Promoted-to-library contributions earn a one-time bonus + a recurring royalty until the prompt is superseded. (Mechanic TBD; design intent: contribution-to-library is itself a revenue line for the contributor.)

When the first three specialists are compensated end-to-end, the model graduates from this paragraph to a proper section in `network.md` (see § 0 — promotion path).

Upstream of this paragraph: [`research/value-attribution.md`](./research/value-attribution.md) is an open research thread on how the studio measures human contribution beyond hourly rate, how new professions get their first defensible price, and how value distribution avoids concentration. Conclusions from that thread feed the compensation model when it graduates.

---

## 9. Boundaries

What this network deliberately does **not** do, in any engagement:

- It does not give specialists access to the client's repo, secrets, DB, or tracker. All work happens in canvas; deliverables ship via § 9 of the engagement charter.
- It does not have specialists working under their own names with the client unless the engagement explicitly opts into that (rare; default is studio-flowed-through).
- It does not guarantee specialists will be available for any specific engagement window. Matching is best-effort against the live availability layer.
- It does not promise that every certified specialist sees every engagement opportunity. The matching layer is a filter; specialists see invites for engagements they fit.
- It does not run outside Muse. A specialist who wants to talk through Slack/Telegram/email instead of canvas is not operating under this network's contract; they are a private freelancer of their client.
- It does not address founders as buyers in this network spec. A founder who hires the studio is a Client (per `openspec/current/platform.md § Personas`); a founder who develops their own product is the buyer-counterparty for the studio's Founder Track (`openspec/archive/2026-05-08-add-founder-track/`). Specialists may be routed onto Founder Track engagements when the founder hires the studio, but the contractual shape — base NDA, certification level, training cycle — is identical to any other studio engagement. The Founder Track does not introduce a new specialist contract.

---

## 10. Relationship to other studio documents

| Document | Relationship |
|---|---|
| [`rapoport-studio-engagement.md`](./rapoport-studio-engagement.md) | The contract document. § 2 capability #1 summarizes; § 13 splits responsibilities; *Appendix B* `D7` records the per-engagement roster; this file is the operational depth behind those references. |
| [`studio-charter.md`](./studio-charter.md) | The bootstrap charter for sub-products. Sub-products inherit the same network and the same training discipline. The relationship goes both ways: a sub-product's specialists are sourced from this network, and graduates of sub-product engagements feed back into the network. |
| [`projects/<slug>.yaml`](./projects/) | Per-engagement profiles. Each engagement's `specialists:` block records the active roster, NDA status per specialist, training certifications, and contributed prompts in scope. |
| `openspec/current/muse.md` (studio repo) | The runtime that loads specialist-contributed prompts and tools per § 6. The contribution model in § 6 is the "input contract" Muse honors. |
| `openspec/current/studio-workspace/README.md` (studio repo) | The internal workspace where canvas lives. Specialist invitations, NDA status, and profile management surfaces will live in admin views of this app when the network has its first three onboarded members. |

---

## 11. Open questions

| ID | Priority | Question |
|---|---|---|
| `Q-NW-1` | P0 | What is the legal shape of the specialist relationship under MITP — contractor, employee, or hybrid? Determines payroll, NDA enforceability, IP-assignment clause. **Owner: i-avocat.md.** Open pending counsel review. Note (2026-05-15): canvas access role is now defined as `external_collaborator` per change `redesign-canvas-permission-model` independent of this legal-shape resolution. |
| `Q-NW-2` | P0 | What is the canonical base NDA template? Drafted by counsel before first non-Pavel specialist signs. Open pending i-avocat engagement. Note (2026-05-15): canvas access provisioning for hired specialists is now tied to the `external_collaborator` project role; NDA template question remains open for the legal instrument itself. |
| ~~`Q-NW-3`~~ | ~~P1~~ | ~~Where does the specialist registry live in code?~~ **Closed 2026-05-13** — registry is the `specialists` Supabase table; prompt augmentations referenced via `prompt_augmentations_uri` column. See `openspec/current/db/entities.md § Specialist tables` and `§ 6.2 Storage` above. |
| `Q-NW-4` | P1 | What is the matching layer's v1 implementation — pure rule-based filter, scored heuristic, or ML? v1 may be coarse; commit to coarse explicitly so it doesn't accrete complexity prematurely. |
| `Q-NW-5` | **resolved 2026-05-12** | What is the threshold for "promotion to common library" of a specialist-contributed prompt? **Resolution:** for studio-internal use the existing § 6.3 threshold (3 independent engagements with positive quality signals) stands. For *marketplace context* — i.e. when a published canvas exposes a specialist-contributed prompt as part of its sale — promotion additionally requires 2 senior endorsements per § 13. Two thresholds, one mechanism, distinct triggers. Resolved by [`changes/define-canvas-studio-thesis`](../changes/define-canvas-studio-thesis/) per §§ 12–15 of this document. | <!-- openspec-refs: skip-line --> <!-- TODO RAP-682: surfaced at guard-tighten; resolve in followup -->
| `Q-NW-6` | **resolved 2026-05-12** | Does the studio publish anonymized network-level metrics? **Resolution:** metrics are split into two tiers. *Per-specialist* — footprint score (§ 13) and tier badge (§ 4.3) are public-by-default for `listed` and `open` specialists; `stealth` specialists publish nothing. *Aggregate network metrics* (§ 7.3 — median certification time, median engagements per specialist per year, prompts promoted per year) remain studio-internal; selective curated quotes may appear in marketing but the live aggregates do not. This avoids leaking competitive information while still giving buyers per-specialist trust signal. Resolved by [`changes/define-canvas-studio-thesis`](../changes/define-canvas-studio-thesis/) per § 12 of this document. | <!-- openspec-refs: skip-line --> <!-- TODO RAP-682: surfaced at guard-tighten; resolve in followup -->
| `Q-NW-7` | P2 | When does this document graduate from `_methodology/` to `openspec/current/network.md`? Suggested trigger: third certified specialist + first promoted prompt. The 2026-05-12 thesis (§§ 12–15) does *not* itself trigger graduation — the trigger remains the specialist-count + promotion event. The §§ 12–15 commercial surface graduates *with* the rest of the document at that point. |

---

## 12. Public profile model — three visibility modes

> Added 2026-05-12 by change [`define-canvas-studio-thesis`](../changes/define-canvas-studio-thesis/). Status: **hypothesis** — the contract defined here ships only after the validation experiment in [`validation-plan-q3-2026.md`](./validation-plan-q3-2026.md) graduates the thesis. Until then the contract is documented (so the next agent can review it) but no UI is built. <!-- openspec-refs: skip-line --> <!-- TODO RAP-682: surfaced at guard-tighten; resolve in followup -->

The specialist registry exposes three visibility modes per specialist. Mode is set per-specialist (granularity may extend to per-discipline in v2; out of scope here). The specialist controls their own mode; the studio enforces only minimum tier requirements for `open` mode.

### 12.1 The three modes

| Mode | Profile location | Who can see it | Pricing visible | Direct contact |
|---|---|---|---|---|
| **Stealth** | Studio-internal only; the matching layer (§ 5) sees the profile, no other surface does | Pavel + matching layer | No | Studio-mediated only |
| **Listed** | Visible in the network roster at `app.rapoport.studio/network` (auth-walled — visible to clients with an active engagement) and in matching candidate lists | Active clients + Pavel + matching layer | No (cost-band may be shown per § 14, exact pricing hidden) | Inquiry routed through studio |
| **Open** | Public profile at `rapoport.studio/network/<slug>` | Anyone — including search engines | Yes — full per § 14 | Direct "request engagement" CTA on the public page |

### 12.2 What is shown in each mode

The composition of the profile differs by mode. The table below uses ✓ (shown), ◔ (shown only if specialist opts in), and ✗ (hidden).

| Profile element | Stealth | Listed | Open |
|---|---|---|---|
| Name / pseudonym | ✓ | ✓ | ✓ |
| Photo / avatar | ✓ | ✓ | ✓ |
| One-line bio | ✓ | ✓ | ✓ |
| Tier badge (Author / Senior; Apprentice / Reviewer hidden until Author) | ✓ | ✓ | ✓ |
| Discipline tags | ✓ | ✓ | ✓ |
| **Footprint score** (§ 13) — single number | ✓ (internal) | ✓ | ✓ |
| **Footprint breakdown** (component-by-component) | ✓ (internal) | ◔ | ✓ |
| Authored canvases (titles, links) | ✓ | ◔ | ✓ |
| Cited decisions (D-X-N references) | ✓ | ◔ | ✓ |
| Promoted prompts (named in library metadata) | ✓ | ✓ | ✓ |
| Drift signals (§ 7.1) | ✓ (studio-internal) | ✗ | ✗ |
| Hourly rate | ✗ | ✗ or band per § 14 | ✓ |
| Per-canvas / per-blueprint rates | ✗ | ✗ | ✓ |
| Royalty rate on published canvases | ✗ | ✗ | ✓ |
| Availability status (open / partial / closed) | ✗ | ✓ | ✓ |
| "Request engagement" CTA | ✗ | Routed via studio | Direct |

The clear principle: **footprint** is exposable in all modes (it is the *signal*); **pricing** and **direct contact** are progressively exposed as the specialist chooses to be more public.

### 12.3 Mode-change rules

- A specialist may move freely between Stealth ↔ Listed ↔ Open, with two constraints:
  - To enter `Open`, the specialist must hold tier `Certified Author` or higher (§ 4.3). `Apprentice` and `Reviewer` cannot publish public profiles (the studio does not surface a developing reputation).
  - To enter `Open`, the specialist must have non-zero footprint (i.e. ≥ 1 authored canvas *or* ≥ 1 promoted prompt). Empty `Open` profiles dilute brand.
- Mode changes are auditable — recorded with timestamp on the specialist record.
- A specialist who moves to `Stealth` after being `Open`: the public URL `rapoport.studio/network/<slug>` 404s; cached engagements remain valid.

### 12.4 Drift signals stay studio-internal regardless of mode

Drift signals from § 7.1 (senior-review delta, client-revisions trend, spec-fidelity audit, implementation feedback) are *never* exposed publicly. They are calibration data for the studio, not buyer signal. The reasoning: drift signals are leading indicators that depend on full context (client expectations, engagement scope, training stage). Exposing them invites misinterpretation.

The footprint score (§ 13) is the public-facing performance signal. Drift signals stay where they are.

---

## 13. Footprint score — the OpenSpec h-index

> Added 2026-05-12 by change [`define-canvas-studio-thesis`](../changes/define-canvas-studio-thesis/). Status: **hypothesis**. Formula and weights are first-pass; expected to evolve through Tact 1 and 2 of the canvas-studio rollout (per `positioning.md § 1.4`). <!-- openspec-refs: skip-line --> <!-- TODO RAP-682: surfaced at guard-tighten; resolve in followup -->

### 13.1 Why footprint instead of star ratings

Star ratings (Upwork, Fiverr, Yelp) average qualitative judgement into a single dimension. They are easy to read, easy to game, and converge on mediocrity (the *3.8-of-5 problem* — ratings cluster in a narrow band that doesn't discriminate). The OpenSpec corpus produces a different kind of signal: a *graph* of authored artefacts, cited decisions, promoted prompts, and downstream usage. This graph supports a footprint score analogous to the academic *h-index* — it grows over time, is hard to game, and reflects accumulated contribution rather than reported sentiment.

### 13.2 Components and weights (first-pass)

The footprint score is the weighted sum of the following components, computed from the studio's specialist-canvas-decision graph:

| Component | What it counts | Weight | Provenance |
|---|---|---|---|
| **Authored canvases** | Canvases the specialist authored (single or lead author) and that have been published in any visibility mode | 1× | Specialist record + canvas metadata |
| **Forked canvases** | Canvases that other architects forked from the specialist's published canvas as a starting point | 3× | Canvas fork-graph |
| **Purchased canvases** | Distinct paid blueprint exits where the canvas authored by this specialist was the basis | 5× | Stripe payment ↔ canvas link |
| **Cited decisions** | `D-X-N` references in *other* OpenSpec corpora to a decision authored by this specialist (counted by citing-corpus, deduplicated within a corpus) | 2× per citation | Cross-corpus citation graph (requires Forge audit / cross-repo indexer — built when Tact 2 ships) |
| **Promoted prompts** | Prompt augmentations promoted to the common library per § 6.3 | 10× | `decisions.md § D-NW-N` records (existing mechanism) |
| **Canvases in production** | Canvases the buyer self-reported (or audit-detected) as deployed to production | 25× | Buyer self-report + optional Forge audit telemetry (see § 13.5 anti-gaming) |
| **Senior endorsements** | Senior reviewer flagged the specialist's work as "best-in-category for the quarter" | 15× per endorsement | Senior reviewer log; quarterly cycle |

### 13.3 What the score looks like in display

```
Footprint: 247

Authored: 4 × 1   = 4
Forked:   2 × 3   = 6
Purchased: 8 × 5  = 40
Cited:    35 × 2  = 70
Promoted: 5 × 10  = 50
In prod:  3 × 25  = 75
Senior:   0 × 15  = 0

Total:           247
```

The breakdown is *always shown* alongside the aggregate. Single numbers without provenance are propaganda; numbers with provenance are signal. Buyers may weight the components differently than the studio defaults — the component view supports that.

### 13.4 Hypothetical archetype scores (illustrative)

To calibrate the formula's reasonableness, three illustrative archetypes:

| Archetype | Authored | Forked | Purchased | Cited | Promoted | In prod | Senior | Total |
|---|---|---|---|---|---|---|---|---|
| Newly graduated Author after 3 engagements | 3 | 0 | 0 | 0 | 0 | 0 | 0 | **3** |
| Mid-career Author, year 1 in network | 6 | 2 | 4 | 8 | 1 | 0 | 0 | **58** |
| Senior with publishing volume, year 3 | 12 | 8 | 25 | 60 | 6 | 5 | 4 | **496** |
| Domain authority (rare archetype) | 4 | 12 | 40 | 200 | 10 | 8 | 8 | **1,070** |

The wide spread is a feature, not a bug — it rewards different career strategies (volume vs. depth vs. influence) and gives buyers meaningful discrimination at the top end.

### 13.5 Anti-gaming

Six structural defences against score-gaming:

1. **No self-purchase.** Specialist purchasing their own canvas (directly or via shell account) is detected via Stripe customer ID match against specialist record; flagged for senior review and excluded from score.
2. **Citation-graph limited to ratified decisions.** A `D-X-N` only counts as "cited" if it appears in a *ratified* decisions.md entry (per the existing convention in `openspec/decisions.md`), not in a draft proposal. Forks of a specialist's own corpus do not count as citations.
3. **Production-deployment claims require attestation.** A buyer self-report of "deployed to production" requires either (a) a screenshot + URL submitted via the studio admin surface, or (b) a Forge audit identifying the specialist's canvas as the upstream of an audited production repo. Unverified claims do not count.
4. **Senior endorsements are rate-limited.** Each senior reviewer may issue at most 2 endorsements per quarter across the entire network, regardless of how many specialists they reviewed. This prevents endorsement inflation.
5. **Score is monotonic but not immutable.** Score can be *clawed back* if a component is later invalidated (e.g. a published canvas is delisted for cause; a cited decision is superseded by a contradicting one). Clawbacks are logged on the specialist record with rationale.
6. **The breakdown is always shown.** Buyers see the components, not just the aggregate. This makes manipulation visible — a specialist whose entire score comes from forks (with zero purchases) reads differently than one whose score comes from production deployments.

### 13.6 What the score does *not* measure

To avoid mis-marketing this metric internally:

- It does *not* measure talent. Talent is upstream of contribution; the score measures *contribution*. A talented specialist new to the network has low footprint; that is correct.
- It does *not* measure how pleasant the specialist is to work with. That is the engagement-feedback signal (§ 7.1, drift signals — internal).
- It does *not* measure availability. Footprint is historical; availability is current and shown separately (§ 12.2).
- It does *not* measure how much the specialist deserves to be paid. Pricing is set by the specialist within tier-band guidance (§ 14), informed by but not derived from footprint.

---

## 14. Pricing display — three disclosure levels and the pricing stack

> Added 2026-05-12 by change [`define-canvas-studio-thesis`](../changes/define-canvas-studio-thesis/). Status: **hypothesis**. Default rates are placeholders; refined through validation outcomes (per [`validation-plan-q3-2026.md`](./validation-plan-q3-2026.md)). <!-- openspec-refs: skip-line --> <!-- TODO RAP-682: surfaced at guard-tighten; resolve in followup -->

### 14.1 The pricing stack — four layers

Specialists may offer up to four pricing layers in their profile, depending on their work mode:

| Layer | What it prices | Typical buyer | Default range (illustrative) |
|---|---|---|---|
| **Hourly rate** | Ad-hoc consulting; calls; ad-hoc reviews | Architect engaging the specialist for a one-off conversation | $80–$400/hr by tier |
| **Per-canvas rate** | A packaged canvas the specialist publishes for sale on the marketplace | Architect buying a starter blueprint | $1k–$15k by canvas complexity |
| **Per-blueprint-exit rate** | Specialist's share when they participate in a buyer's blueprint exit | Architect whose canvas exit was assisted by this specialist | % of exit fee, typically 15–40% |
| **Royalty rate** | Recurring share of every sale of the specialist's published canvas | Same as per-canvas, but recurring | 60–70% to specialist; 30–40% studio |

A specialist need not offer all four. A pure marketplace publisher offers only per-canvas + royalty. A pure consultant offers only hourly. Most specialists offer some combination as their work evolves.

### 14.2 Three disclosure levels

Independently of pricing-stack composition, the specialist chooses how visibly each layer is shown.

| Level | What is shown | When to use |
|---|---|---|
| **Hidden** | No price; "ask" CTA | Specialist who wants to negotiate per-engagement; specialist who wants to discourage cold inquiries |
| **Range** | Either dollar-sign indicator ($, $$, $$$, $$$$) or numeric range ($150–$250/hr) | Specialist who wants to filter by budget without anchoring to a number |
| **Exact** | Specific value | Open mode (§ 12) only; specialist who wants maximum-friction-removal for inquiries |

A specialist may mix levels across pricing layers — e.g. hourly rate at *Range*, per-canvas rate at *Exact*, royalty rate at *Hidden*. The UI surfaces each layer separately.

### 14.3 Tier-band guidance (floor and ceiling)

Per § 4.3, the studio publishes per-tier rate guidance (not enforcement) for hourly rates:

| Tier | Suggested floor (hourly) | Suggested ceiling (hourly) |
|---|---|---|
| Apprentice | n/a — no paid engagements | n/a |
| Certified Reviewer | $60 | $120 |
| Certified Author | $120 | $300 |
| Senior | $250 | $600+ (no enforced ceiling) |

Specialists may price below or above these bands with reason recorded on their profile (visible to studio reviewers, not necessarily to buyers). Pricing dramatically below floor signals to studio that either (a) specialist is undervaluing, or (b) other rationale exists worth understanding. Pricing dramatically above ceiling is allowed without comment — high-end pricing is its own quality signal.

### 14.4 Why guidance and not enforcement

Hard floor pricing is anti-competitive in some jurisdictions and creates rigidity. The studio's interest is met by *guidance + visibility* — specialists who price below floor make a visible choice; the market's response (or lack of it) calibrates over time. The studio reserves the right to require explanation for systematic floor-undercutting; this is documented in compensation contracts when Pulse activates and § 8 is rewritten.

### 14.5 What is *not* in the pricing stack (and why)

- **Outcome-based pricing** (% of buyer's revenue / cost-saving). Tempting but creates attribution hell. Not offered as a standard pricing layer; specialists who want to negotiate outcome-based deals do so off-platform.
- **"Free first canvas" promo pricing.** Discouraged because it sets buyer expectations against the platform's positioning ("only architecture has a price"). Studio may run promotional pricing for the studio's own canvases; specialists may not advertise free canvases as marketing.

---

## 15. Race-to-bottom defenses

> Added 2026-05-12 by change [`define-canvas-studio-thesis`](../changes/define-canvas-studio-thesis/). Status: **hypothesis**. The five mechanisms below are structurally why the marketplace is expected not to degenerate the way Upwork did. If any one fails in practice, the studio reverts to a more curated mode (Tact 2 instead of Tact 3 per `positioning.md § 1.4`). <!-- openspec-refs: skip-line --> <!-- TODO RAP-682: surfaced at guard-tighten; resolve in followup -->

The dominant failure mode of open specialist marketplaces is *race-to-the-bottom*: open rating + open pricing → downward price spiral → quality drops → senior specialists leave → only newcomers remain → marketplace dies. Five mechanisms are built into the contract above to structurally prevent this:

### 15.1 Floor-price-by-tier

§ 14.3 above. Senior specialists do not compete with newcomers on price. The floor is a guidance number, but it is *publicly published* — buyers understand that pricing below floor signals something other than quality.

### 15.2 Curated entry

§§ 3 (NDA), 4 (training), 5 (matching layer human-confirm step). The barrier to becoming a network specialist is high (NDA + training + certification). Anyone can register on Upwork in five minutes; no one can become a Rapoport Studio specialist in less than approximately six weeks. This filters out price-dumping volume.

### 15.3 Footprint, not stars

§ 13. Footprint accumulates slowly and reflects actual work artefacts (canvases, citations, promoted prompts). It cannot be inflated by social games (asking buyers for stars, exchanging reviews). The signal a buyer sees grows from real contribution; manipulation is visible in the breakdown.

### 15.4 Promotion to library as career path

§ 6.3 (existing). A senior specialist has *non-financial* career paths within the network: their prompts get into the common library, their name attaches to `D-NW-N` decisions, their footprint compounds with citation. This retains top specialists who would otherwise leave when price-pressure increased.

### 15.5 Studio as quality gatekeeper

§ 5 (matching layer + human-confirm) + § 6.3 (review of contributed augmentations) + Tact 2 marketplace gate (in `positioning.md § 1.4`). The studio is willing to be the bottleneck on quality even at the cost of marketplace velocity. Tact 3 (open self-serve marketplace) is gated on automated certification *replacing* studio review — it does not arrive by removing review.

### 15.6 What if these defences fail

If race-to-the-bottom dynamics emerge despite the defences, the studio's recovery posture is:

- **First response**: tighten matching-layer filters; raise tier requirements for `Open` mode publishing; raise floor-price guidance with stronger rationale shown to underpricing specialists.
- **Second response**: revert to Tact 2 mode (no open self-serve); all marketplace listings require studio review; reduce specialist-publisher count.
- **Third response (terminal)**: close marketplace dimension; revert to studio-as-sole-publisher (Tact 1); other monetization paths (engagement modes per `rapoport-studio-engagement.md § 4`) remain.

The escalation ladder is defined here so a future agent under pressure does not improvise a worse one.

---

## Change log

- `2026-05-05` — initial draft v1, behind the engagement charter. Authored as part of the studio's specialist-network packaging discussion.
- `2026-05-12` — added §§ 12–15 (commercial / marketplace surface) via change [`define-canvas-studio-thesis`](../changes/define-canvas-studio-thesis/). Closed `Q-NW-5` and `Q-NW-6` in § 11. Status of overall doc unchanged (still draft v1, behind engagement charter). The §§ 12–15 additions are themselves **hypothesis** until the validation experiment in [`validation-plan-q3-2026.md`](./validation-plan-q3-2026.md) graduates the canvas-studio thesis; until then they document the contract without triggering implementation. <!-- openspec-refs: skip-line --> <!-- TODO RAP-682: surfaced at guard-tighten; resolve in followup -->
- `2026-05-15` — amended via change [`redesign-canvas-permission-model`](../archive/2026-05-15-redesign-canvas-permission-model/) (Phase 6 archive). Added canvas role note to § 2 lifecycle (hired specialists land as `external_collaborator`). Corrected role name in § 6.1 from `collaborator` to `external_collaborator`. Added 2026-05-15 notes to § 11 Q-NW-1 and Q-NW-2 clarifying that the canvas access role is now defined while the legal-shape and NDA-template questions remain open.
