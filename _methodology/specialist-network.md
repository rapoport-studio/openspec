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
- Read three representative archived changes from a real studio engagement (the most recent three under `openspec/archive/` of any active engagement).
- Author a "training change" against a fictional but well-shaped capability brief; the studio's senior reviewer (currently Pavel) provides line-level feedback in canvas.
- Pass criterion: training change lands in studio-internal `_training/<specialist-slug>-graduation/` with reviewer sign-off and zero outstanding structural critiques.

### 4.2 Axis B — Business-model validation methodology

Goal: the specialist can read a `proposal.md` and ask the questions that move it from "plausible" to "validated" — pricing sensitivity, unit-economics floor, retention-driver vs vanity-metric, defensibility against the named competitor in `_research/competitors/`.

Training surface:

- Read the studio's business-model validation framework (lives at `_training/business-model-validation.md`; to be authored as the first piece of training infrastructure when the second specialist is onboarded).
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

- **Prompt augmentations** — system-prompt fragments that load when the specialist is `collaborator` on a canvas. Example: a clinical advisor's profile attaches a fragment that biases Muse to defer clinical claims and surface uncertainty rather than asserting.
- **Tool list** — a curated allowlist of tools (from the studio's master tool registry — `web_search`, `fetch_url`, `domain_research`, plus future additions) that the specialist's typical work needs. Muse loads this list per session.
- **Reference packs** — pinned URLs / file-paths to research artifacts (`_research/competitors/yuna.md`, etc.) the specialist always wants in context.
- **Refusal patterns** — the specialist's documented "won't do" list. The clinical advisor's refusal pattern includes "won't draft text that performs psychotherapy outside the agreed product surface."

### 6.2 Storage

Specialist prompts and tools live in the studio repo under `packages/muse/specialists/<slug>/` (a deferred package per `platform.md`). Until that package exists, profiles live as markdown files under `_methodology/specialists/<slug>.md`. Either way the profile is studio-internal IP per the base NDA.

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
| `openspec/current/studio.md` (studio repo) | The internal workspace where canvas lives. Specialist invitations, NDA status, and profile management surfaces will live in admin views of this app when the network has its first three onboarded members. |

---

## 11. Open questions

| ID | Priority | Question |
|---|---|---|
| `Q-NW-1` | P0 | What is the legal shape of the specialist relationship under MITP — contractor, employee, or hybrid? Determines payroll, NDA enforceability, IP-assignment clause. **Owner: i-avocat.md.** |
| `Q-NW-2` | P0 | What is the canonical base NDA template? Drafted by counsel before first non-Pavel specialist signs. |
| `Q-NW-3` | P1 | Where does the specialist registry live in code — `packages/muse/specialists/`, a Supabase table, or both? Decision affects how Muse loads prompt augmentations at session start. |
| `Q-NW-4` | P1 | What is the matching layer's v1 implementation — pure rule-based filter, scored heuristic, or ML? v1 may be coarse; commit to coarse explicitly so it doesn't accrete complexity prematurely. |
| `Q-NW-5` | P1 | What is the threshold for "promotion to common library" of a specialist-contributed prompt? Three engagements is a starting point; needs validation when first promotion candidate appears. |
| `Q-NW-6` | P2 | Does the studio publish anonymized network-level metrics (count, disciplines, retention)? Useful for client-facing trust signals; risk of leaking competitive information. |
| `Q-NW-7` | P2 | When does this document graduate from `_methodology/` to `openspec/current/network.md`? Suggested trigger: third certified specialist + first promoted prompt. |

---

## Change log

- `2026-05-05` — initial draft v1, behind the engagement charter. Authored as part of the studio's specialist-network packaging discussion.
