# Research thread — RAP-119 as empirical research bed

> **Update 2026-05-08 — RAP-119 cancelled.** The engagement under
> observation never reached Phase 1 (intake harvest never written, no
> kickoff scheduled). Sessions 1–4 measurements will not arrive.
> Questions Q-OS-1, R-BA-1/2/3/5, B2, IP-B caveat are **reopened
> pending an alternative empirical source** (likely the first real
> multi-author engagement the studio runs, whenever that occurs).
>
> This file is **preserved** as a methodology artefact: it documents
> *how to instrument an empirical research bed* against an engagement
> — the wiring shape (questions → surfaces → observations → fold-back)
> is reusable for the next engagement that materialises. Substitute
> "RAP-119" with the next engagement's identifier and re-thread.
>
> See [`openspec/archive/2026-05-08-add-team-onboarding-canvas/`](../../archive/2026-05-08-add-team-onboarding-canvas/)
> for the original engagement design and
> [`loop-protocol.md`](./loop-protocol.md) +
> [`multi-author-canvas-instructions-template.md`](./multi-author-canvas-instructions-template.md)
> for the two artefacts that survived the cancellation.

> **Status:** open observation protocol. Wires existing research threads to one concrete engagement.
> **Owner:** Pavel (founder).
> **Opened:** 2026-05-07.
> **Companion to:** [`openspec/archive/2026-05-08-add-team-onboarding-canvas/`](../../archive/2026-05-08-add-team-onboarding-canvas/) (RAP-119, cancelled 2026-05-08) — the engagement that was under observation.
> **Feeds back into:** [`value-attribution.md`](./value-attribution.md), [`openspec-block-authorship.md`](./openspec-block-authorship.md), [`open-source-positioning.md`](./open-source-positioning.md), [`competitive-positioning-sdd-frameworks.md`](./competitive-positioning-sdd-frameworks.md).

---

## 1. Why this exists

The four research threads in `_methodology/research/` are each honest about their dependence on empirical data. Phrases like *"the studio has zero datapoints; this risk is real until first 3 engagements complete"* (`open-source-positioning.md` § 6.5) and *"specialist network has to actually work — Q-NW-1 to Q-NW-5 still open"* (`competitive-positioning-sdd-frameworks.md` § 5) appear in three of the four. Until 2026-05-07 the studio had no concrete non-Pavel engagement to measure against.

[RAP-119](../../changes/add-team-onboarding-canvas/) ([PR #262](https://github.com/rapoport-studio/rapoport.studio/pull/262)) changed that. It scaffolds the studio's first multi-author engagement: three external partners onboarded into spec-driven development across four sessions plus a review week, with a co-created product (name TBD) being authored as part of the onboarding.

This file is the **wiring document** between the four research threads and the engagement that can finally test them. It does *not* introduce new questions. It maps existing questions to RAP-119 surfaces, names what observable evidence looks like, says where observations get captured, and specifies how conclusions fold back into the source threads when Sessions 1–4 close.

The failure mode this protocol prevents: Sessions 1–4 happen, generate signal, signal gets lost in chat scrollback, and the four research files sit with their open questions intact a year later.

---

## 2. What this protocol is and is not

It **is**:

- A wiring document mapping existing research questions to RAP-119 surfaces.
- A specification of what counts as an observation, where observations live, and when they get folded back.
- An honest list of risks that could distort or invalidate observations from a single engagement.

It is **not**:

- A research thread of its own. No new questions; only wiring of existing ones.
- A substitute for the four source research files. They remain the source of truth for their respective questions.
- A guarantee of resolution. Some questions may not be answerable from one engagement (Q3 anti-concentration is hard to test at team-of-4; Q-OS-1 is a founder commitment, not a data observation).
- A research-ethics protocol over RAP-119 participants. Viorica, Alexandr, and Rodion consented to a co-authorship engagement, not to academic-style observation. "Observations" here are facilitator notes about *the methodology*, not about the people running it.

---

## 3. Mapping — research questions × RAP-119 surfaces

Each row names: which research question, which RAP-119 surface lets us see it, what an observation looks like, when in the engagement timeline.

### 3.1 `value-attribution.md` — dimensions of contribution

| Question | RAP-119 surface | What "observed" looks like | When |
|---|---|---|---|
| **Q1** — which dimensions of contribution should the studio measure? | D7: each domain owner produces one triplet by end of Session 3. Three triplets, three authors, one engagement. | For each of `Reuse / Depth / Mentorship / Domain rarity / Outcome-linked`: pick one or two paragraphs from the corpus and ask whether the dimension showed up *observably* (in canvas turns, in commit history, in cross-spec references). Cheap-to-observe + hard-to-game is the bar. | Post-Session 4 |
| **Q2** — how are new professions valued before market data exists? | The three named domains: ecosystem & funding (Viorica), operating model & metrics (Alexandr), brand & creative (Rodion). | Did the names hold from Session 1 ratification through Session 4? Did anyone re-name their domain mid-engagement? Did the corpus end up using each member's terminology consistently (Layer 2 working as designed) or did Pavel's defaults reassert? | Session 1 Loop 0 (naming), Session 4 (durability) |
| **Q3** — how does the studio prevent value concentration? | D4: soft-enforcement, cross-domain edits surface as PR-comment, not auto-rejection. | Count cross-owner edits per session. For each: was it accepted, contested, or reverted? Did Pavel mediate or stay out? Did "trust over policy at team size of 4" hold, drift, or fail? | Sessions 2–4 |
| **Q4** — what does the specialist need to *see* to feel valued? | Block B course-meta directive: Muse explains methodology choices inline, ≤2 sentences, not lecturing. | Does Muse stay terse or drift toward over-explaining? Do members report the inline explanations as helpful or condescending? Does anyone ask for the directive to be removed or amended? Does the directive survive into Session 4 unchanged? | After Session 1 (initial reception), after Session 4 (durability) |

### 3.2 `openspec-block-authorship.md` — block-level authorship mechanism

| Question | RAP-119 surface | What "observed" looks like | When |
|---|---|---|---|
| **R-BA-1** — does git blame already give a usable signal? | Three triplets, three named authors, one repo. First time this is testable on real multi-author specs. | Run `git log -p --follow` per file after the engagement archives. Manually review whether the per-paragraph author trace is coherent — or whether reformatting / merging / Pavel cleanup produced last-toucher-wins noise. | Post-Review week |
| **R-BA-2** — what block granularity survives reformatting? | Same triplets, before vs after any consistency-pass Pavel runs at archive time. | Compare paragraph-level vs section-level attributions across the reformat. Measure attrition: what % of paragraph attributions stay coherent; what % of section attributions stay coherent. | Post-Review week |
| **R-BA-3** — `canvas_message_id` ↔ `markdown_block_id` cardinality | Layer 1 and Layer 2 attribution flow: Telegram chat → harvest file (`research/intake-harvest.md`) → Loop 0 ratification → `canvas_project_instructions` row → paragraph in a triplet. | Trace **one harvest quote** through that pipeline end-to-end. What cardinality showed up — 1:1, 1:N, N:1? Run the trace on three quotes (one per member) and compare. | Continuous, captured in close-loop seeds |
| **R-BA-5** — does (originator, writer, refiners[]) shape fit real authoring? | Sessions 2–4 produce real collaborative drafts under D4 soft-enforcement. | After Session 4: pick five paragraphs across the three triplets. For each, ask the actual author (in person at Review week) — "who originated this, who wrote it, who refined it?" Does the three-field shape fit, force false simplification, or omit a real role? | Review week |
| **B2** — idea ≠ text asymmetry | Loop 0 ratification mechanism: pre-prepared harvest material is **not pinned** until live confirmation by the originator. | Did Loop 0 ratification produce edits to the harvested quotes? How many of three members corrected, expanded, or rejected their attributed Layer 1 / Layer 2 entries? This is the originator-vs-writer separation in operational form. | Session 1 Loop 0 |

`R-BA-4` (rendering surface) and `R-BA-6` (post-archive edit policy) are not testable from RAP-119 alone — they require a UI consumer and a second engagement, respectively. They stay open after Sessions 1–4.

### 3.3 `open-source-positioning.md` — what to publish

| Question / claim | RAP-119 surface | What "observed" looks like | When |
|---|---|---|---|
| **Q-OS-1** — commit to narrative posture (Phase 0)? | D7: Pavel writes a triplet on the meta-process itself (`team-onboarding.md`) inside the team's corpus. | Read the resulting `team-onboarding.md` after Session 3. Does it naturally credit OpenSpec / Spec Kit / Anthropic / Cloudflare / Supabase as upstream, or does it read as if the studio invented everything? Does the language match Phase 0's required posture (acknowledged consumer + occasional contributor) or contradict it? | Post-Session 3 |
| **Layer 1 candidate** — `loop-protocol.md` | Validate-by-use across Sessions 1–4 + Phase 8.4 corpus-promotion decision (per RAP-119 `tasks.md`). | If Phase 8.4 evaluates 4-of-5 validation criteria as "yes," `loop-protocol.md` becomes a candidate Layer 1 publication. The question for this protocol is upstream: did the loop-protocol *itself* survive the engagement, and is the surviving artifact engagement-clean (no client-specific names, no internal-only references) or does it need redaction before public release? | Post-Phase 8.4 |
| **Layer 1 candidate** — `team-onboarding.md` | D7 produces it. | Same redaction-readiness check. Bonus: is the file naturally reusable for cohort 2, or so RAP-119-specific it needs heavy generalization? | Post-Session 4 |
| **Sessions 1–4 as Phase 1 gate** | The whole engagement. | Does the engagement archive successfully (no D9 blocker, no member drop-out, no rollback)? "Successfully archives" is the precondition stated in `open-source-positioning.md` § 6.5 for risk mitigation. | Post-Review week |

### 3.4 `competitive-positioning-sdd-frameworks.md` — positioning claims

| Question / claim | RAP-119 surface | What "observed" looks like | When |
|---|---|---|---|
| **§ 5 weakness "engagement track record"** — was "Zero archived non-Pavel engagements." | RAP-119 itself. | Does it archive cleanly? Is it citable in case-study form post-engagement? At least one non-Pavel-only specialist must complete the cycle (intake → triplet → archive) for the row to flip from "true" to "partially true." | Post-Review week |
| **§ 5 weakness "multi-engagement velocity"** — "one Pavel = one canvas at a time." | RAP-119 runs one canvas with three external collaborators. | Does the canvas role model (`owner` + `collaborator` × 3) operate without breaking? Does Muse correctly route Block-B course-meta directives across three different members' terminologies (Layer 2)? Does the loop-protocol scale from 1:1 (Pavel solo) to 1:3+1 (Pavel as facilitator)? | Sessions 1–4 |
| **§ 4.1 "all three: methodology + service + artifact"** — collective-client tension. | Four equals, no studio↔client split. The artifact (the team's product corpus) is co-owned. | Does the IP-B model cleanly assign ownership at archive, or does D9 (legal entity deferral) leave artifact ownership ambiguous? Does the engagement reveal a missing "collective-client" template, or does the existing IP model handle it? | Post-D9 resolution (may be after Sessions 1–4) |
| **Q-CP-2** — public commit to the all-three claim | Same surface as above. | Decision is downstream of the § 4.1 observation. If the engagement reveals a collective-client gap, public claim needs caveat. If it doesn't, claim is defensible. | Post-Review week |

### 3.5 What this protocol does *not* test

For honesty, the questions RAP-119 cannot reach:

- `value-attribution.md` Q3 conclusively (one team-of-4 is too small for concentration patterns).
- `value-attribution.md` Q1 dimensions across multiple engagements (RAP-119 is N=1).
- `openspec-block-authorship.md` R-BA-4 (rendering surface — needs a consumer).
- `openspec-block-authorship.md` R-BA-6 (post-archive edit policy — needs a second engagement).
- `open-source-positioning.md` Q-OS-2 (license choice — counsel decision, not observation).
- `competitive-positioning-sdd-frameworks.md` § 4.3 (IP/legal architecture — gated on `legal-landscape-authorship.md`, which doesn't yet exist).

These remain open after RAP-119 closes.

---

## 4. Where observations live

Three places, in order of authority:

1. **Canvas turn record** (`canvas_messages` rows) — primary, immutable, timestamped, attributable. The truth.
2. **Per-session close-loop seeds** (per `loop-protocol.md` Close phase) — Pavel writes one per session immediately after. Rough, but timestamped against the loop they describe.
3. **Post-engagement synthesis** — single document at Review week, folding observations into per-thread updates. The most curated layer; the most lossy.

What does **not** count as a captured observation, even if the user remembers it:

- Pavel's recall after the fact (memory drift; especially after multiple sessions).
- Out-of-canvas chat (Telegram side-conversations, in-person hallway moments).
- Cross-thread inferences from one specialist's behavior alone (one person's reaction is not a pattern).

---

## 5. Fold-back protocol

After Review week, for each of the four source research files:

1. Identify which questions this protocol mapped (per § 3 tables above).
2. For each mapped question, write three lines into the source file:
   - **Observed:** what actually showed up in RAP-119.
   - **Confidence:** low / medium / high, with one-line justification.
   - **Status change:** open / partially-resolved / closed / superseded-by-D-row.
3. If a question is fully closed: amend the source file's `Status:` and link to the closing decision (a `D-*` row in `decisions.md` if one is created).
4. If partially closed: add a `post-RAP-119: ...` note inline on the question, leave it open.
5. If unmoved: leave the question, capture in this file's § 6 why RAP-119 didn't address it (so future cohorts know what to look for).

The bridge from research to policy: a question that clears post-RAP-119 with **high** confidence is a candidate for the next change-folder. For example, a high-confidence resolution of `value-attribution.md` Q4 + Q1 might trigger `openspec/changes/define-felt-value-surface-v1/`. <!-- openspec-refs: skip-line — hypothetical future change-folder cited as an illustrative example, not an existing path. -->

---

## 6. Things that will not be answered (record as observed)

When a question stays open after RAP-119, capture *why*. Possible reasons:

- N=1 limit (one engagement isn't enough to generalize).
- Wrong engagement shape (collective-client doesn't test studio↔client claims).
- Counsel-gated (license, contract template — not an observation question).
- Out-of-canvas behavior (compensation, recruitment — happens between engagements, not inside).

Logging "why not addressed" is as important as logging "what was observed." It tells cohort-2 design what to add.

---

## 7. Risks to watch

- **Observer effect.** Sessions 1–4 happening with awareness that this protocol exists may distort behavior. Mitigation: protocol stays Pavel-internal until Session 1 starts; participants are not briefed that they are being "observed for research." (They are participants in a real engagement, which they are. The methodology-research framing is upstream of their experience.)
- **Single-engagement generalization.** One engagement validates nothing on its own. Conclusions need at least two more cohorts before promotion to corpus-level methodology — a constraint `loop-protocol.md` itself states for its own promotion (Phase 8.4 says "validate by use," singular validation = research-only register).
- **Confirmation bias.** Pavel observes what Pavel was looking for. Mitigation: capture **surprises** explicitly and visibly. A close-loop seed that says "this happened and I did not expect it" is worth more than ten that confirm priors. Surprises feed back into question reformulation, not just question answering.
- **Recall over time.** Review week is one week after Session 4. Observations from Session 1 are a month old by then. Mitigation: the close-loop-seed convention is the durable record; synthesis at Review week is selection from that record, not recollection.

---

## 8. How this thread closes

When Review week ends and the per-thread fold-back updates are merged into the four source research files. At that point this file moves to `Status: archived observation-bed for cohort 1 (RAP-119)`. Later cohorts may use this file as a template — wiring their cohort to whichever research questions are still open at that time.

If RAP-119 doesn't archive cleanly (D9 legal entity stays blocking, a member drops out mid-engagement, the engagement scope changes materially), this protocol stays open with an explicit re-open trigger ("when RAP-119 archives"). The four source research files remain unchanged until the trigger fires.

---

## 9. Adjacent prior thinking

| Source | Relationship |
|---|---|
| [`openspec/archive/2026-05-08-add-team-onboarding-canvas/`](../../archive/2026-05-08-add-team-onboarding-canvas/) (RAP-119) | The engagement under observation. Source of all surfaces this protocol maps to. |
| [`openspec/archive/2026-05-08-add-team-onboarding-canvas/research/loop-protocol.md`](../../archive/2026-05-08-add-team-onboarding-canvas/research/loop-protocol.md) | The methodology being validated by use. Phase 8.4 corpus-promotion decision is parallel to this thread's per-question fold-back. |
| [`value-attribution.md`](./value-attribution.md) Q1–Q4 | Four questions tested; mapped in § 3.1. |
| [`openspec-block-authorship.md`](./openspec-block-authorship.md) R-BA-1, R-BA-2, R-BA-3, R-BA-5, B2 | Five surfaces empirically testable for the first time; mapped in § 3.2. |
| [`open-source-positioning.md`](./open-source-positioning.md) Q-OS-1, Phase 1 Layer 1 | Two surfaces tested; mapped in § 3.3. |
| [`competitive-positioning-sdd-frameworks.md`](./competitive-positioning-sdd-frameworks.md) § 5, § 4.1, Q-CP-2 | Three claims tested; mapped in § 3.4. |

---

## 10. Change log

| Date | Author | Change |
|---|---|---|
| 2026-05-07 | Pavel (founder) | Opened observation protocol. Mapped Q1–Q4 (value-attribution), R-BA-1/2/3/5 + B2 (block-authorship), Q-OS-1 + two Layer 1 candidates (open-source-positioning), § 5 weaknesses + § 4.1 + Q-CP-2 (competitive-positioning) to RAP-119 surfaces. Captured what RAP-119 cannot answer (§ 3.5). Fold-back protocol specified for post-Review-week update of the four source threads. AI assistance: drafted with Claude (Anthropic). |
