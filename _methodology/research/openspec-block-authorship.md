# Research thread — block-level authorship in OpenSpec documents

> **Status:** open research. Mechanism candidate under [`value-attribution.md`](./value-attribution.md) Q1.
> **Owner:** Pavel (founder).
> **Opened:** 2026-05-07.

---

## The idea

OpenSpec proposals (`proposal.md`, `design.md`, `tasks.md`) and current capability files are written collaboratively in canvas. Today's authorship convention is coarse: the `changes/<slug>/` folder is "owned" by whoever opened it, and `D-*` rows in `decisions.md` are tagged by hand.

The proposed mechanism: attribute **at the block level**. A specific paragraph, list item, or sub-section is recorded as having an originator (and possibly co-refiners). When the change archives, the block-author record persists alongside the document and is queryable later.

Restated from the originator: this gives the studio a structural read on which specialists are *actually generating accepted thinking* — observable at the unit where thinking lives (a paragraph inside a spec), not at coarser granularity (engagement count, tier badge).

This is a candidate answer to [`value-attribution.md`](./value-attribution.md) Q1 — which axes of contribution are observable without instrumentation theater.

---

## Why it's appealing

- **Source-of-truth durable.** Markdown is checked into the repo and survives any decision the canvas/Muse runtime makes. Attribution that lives on markdown is more durable than attribution that lives on ephemeral chat turns.
- **Naturally cumulative.** A specialist who consistently authors blocks that survive archive — no one rewrote them, no later change deleted them — accumulates a defensible signal of contribution depth.
- **Aligns with library promotion.** [`specialist-network.md § 8`](../specialist-network.md) gestures at recurring royalty for prompts promoted to the studio's common library. Block authorship gives the same contour for spec contributions.
- **Cheap to surface.** "Your block in `add-network-entities/proposal.md § 3` was referenced by 4 later changes" is exactly the kind of digest [`value-attribution.md`](./value-attribution.md) Q4 calls for.

---

## Why it's hard

### B1 — What is a "block"?

Markdown has no formal block ID. Each granularity choice has a cost:

- **Paragraph-level.** Highest fidelity. Vulnerable to reformatting — a paragraph break shift re-segments and orphans attribution. Highest churn.
- **Section-level (heading-bounded).** Stable across reformatting. Loses too much — a single `## Section` can be hundreds of lines authored by many people.
- **Explicit marker** (`<!-- block-id: foo -->` HTML comment before each tracked block). Stable, machine-readable, but pollutes source and requires authoring discipline at draft time.
- **Sidecar file** (`proposal.authorship.json` parallel to the markdown, mapping line ranges or block IDs to authors). Keeps source clean. Goes stale on every edit unless tooled.

No option is free. The choice upstream-constrains what "block authorship" can ever mean operationally.

### B2 — Idea ≠ text

The studio's most common authoring pattern is asymmetric:

> Pavel (or another specialist) proposes a direction in a canvas turn. A writing-discipline specialist (OpenSpec authoring, product copywriter) writes it up into prose.

Both contributed; neither is "the author" in a single-tag sense.

A faithful schema needs at minimum:

- `originator` — whose idea this expresses.
- `writer` — whose words these are.
- (optional) `refiners[]` — co-shapers who edited the block in a way that materially improved it.

Single-author attribution is wrong for the studio's actual workflow. Multi-author makes the schema honest and the rendering harder.

### B3 — Edit drift

Once a change archives, future edits exist:

- Typo fixes (any maintainer, often Pavel).
- Clarifications introduced by a later change that touches the same section.
- Spec evolution that renames, renumbers, restructures.

Each edit re-opens the attribution question. Strict re-stamping on every edit is maintenance theater. Lax stamping makes the long-tail signal noisy. Neither default is obviously right.

### B4 — Goodhart

If block authorship becomes the basis for compensation or tier promotion, the incentive shifts from "write the best spec" to "write *more* attributed blocks." Specialists develop reasons to resist co-authoring, because shared attribution dilutes personal signal. The studio's pitch is **collaboration**; this metric, naively coupled to comp, would punish exactly the behavior the pitch depends on.

`value-attribution.md` Q1 flags this category of risk in general terms; here it is concrete and immediate.

### B5 — Granularity vs. authoring friction

The finer the block, the higher the cognitive load on whoever drafts. Pavel writing a `proposal.md` does not want to be reasoning about block IDs every paragraph. The mechanism therefore has to be one of:

- **Automatic** — captured from canvas → commit, no manual marking.
- **Imposed only on tracked changes** — most prose has no attribution; only blocks the author chooses to mark do.
- **Lazy** — attribute only when a block is *referenced* by something that wants attribution (e.g. a later change cites it, or a digest highlights it).

Each path has downstream consequences for tooling, schema, and review.

---

## What research would answer

| ID | Priority | Question |
|---|---|---|
| `R-BA-1` | P0 | Does `git blame` on this repo's actual editing patterns already give a usable signal? Sample three recent archived change-folders. Run `git log -p --follow <file>` and check whether the per-block author trace is coherent. **If yes, the infrastructure is already there and the open question shrinks to "is it surfaced?"** |
| `R-BA-2` | P0 | What block granularity is durable across reformatting? Take three archived proposals; reformat through `prettier --prose-wrap` (or equivalent); measure what fraction of paragraph-level vs. section-level attributions survive intact. |
| `R-BA-3` | P1 | When a canvas turn produces a markdown commit, what is the cardinality between `canvas_message_id` and `markdown_block_id`? 1:1, 1:N, or N:1 will all occur in real authoring. The schema has to admit all three. |
| `R-BA-4` | P1 | What rendering surface justifies the attribution cost? If the only consumer is a never-built future studio UI, the cost isn't justified yet. If it's a canvas annotation (hover a block, see who originated it) or a per-specialist digest, the value is immediate. |
| `R-BA-5` | P1 | Does the (originator, writer, refiners[]) shape fit three recent collaborative drafts, or does it force false simplification? Test against archive before committing. |
| `R-BA-6` | P2 | What is the studio's policy on archived-block edits? Re-stamp authorship, freeze authorship at archive, or annotate edit-drift separately? |

**Empirical update (2026-05-07).** [RAP-119](../../archive/2026-05-08-add-team-onboarding-canvas/) introduces three external authors producing one triplet each by Session 3 (`design.md` D7). For the first time, `R-BA-1`, `R-BA-2`, `R-BA-3`, and `R-BA-5` become testable on real multi-author specs. `B2` (idea ≠ text asymmetry) gets a live operational instance in the Loop 0 ratification mechanism — pre-prepared harvest material is not pinned until the originator confirms it live. Wiring of these tests to RAP-119 surfaces lives in [`rap-119-as-research-bed.md`](./rap-119-as-research-bed.md) § 3.2.

> **Update 2026-05-08 — RAP-119 cancelled.** Engagement deprioritised
> against MVP v1; Sessions 1–4 never ran. R-BA-1/2/3/5 and B2 remain
> **untested** — testability claim above stands but the empirical run
> did not happen. Reopen pending the next multi-author engagement. See
> [`openspec/archive/2026-05-08-add-team-onboarding-canvas/`](../../archive/2026-05-08-add-team-onboarding-canvas/)
> for the original engagement design.

---

## Adjacent options to compare

The studio does not have to pick one path; combinations are valid.

- **Status quo.** Change-folder-level authorship + `D-*` rows. Lossy at block grain, zero overhead.
- **Git blame, surfaced.** No new schema. Build a digest from `git blame` + `git log` and accept its limits (line-level, last-toucher-wins, no idea-vs-text distinction). Cheap. Probably 80% of the value.
- **Canvas-message provenance.** Record which canvas turn produced which markdown commit; attribute via canvas message author. Decouples from markdown internals. Depends on the canvas-capture pipeline being reliable, which is a separate research problem.
- **Block-author markup (this thread).** Explicit attribution embedded in or alongside markdown.
- **Pinned-instruction attribution.** Author-attributed `canvas_project_instructions` rows (`kind=preference` / `kind=vocabulary`) ratified live in Loop 0 before pinning. Originator named, timestamp from chat harvest, ratification is the writer-confirms-originator step. Specified by [RAP-119](../../archive/2026-05-08-add-team-onboarding-canvas/) `design.md` § Intake stage (cancelled 2026-05-08 before deployment); does not require markdown attribution at all — attribution lives on the upstream instruction, not on the downstream paragraph. Live-ratification pattern survives in [`multi-author-canvas-instructions-template.md`](./multi-author-canvas-instructions-template.md) § Block D, awaiting first deployment.

Plausible final answer: **git blame for fine grain + change-folder for coarse + selective block markup where the studio explicitly wants it (e.g. promoted-to-library prompt blocks) + pinned-instruction attribution for the upstream layer (preferences and vocabulary)**. Not one-mechanism-fits-all.

---

## Things this thread is *not*

- It is **not** a proposal to ship a `block_authorship` schema. The thread exists to decide whether a schema is justified at all.
- It is **not** a basis for compensation. Even if block authorship becomes capturable, B4 (Goodhart) is the warning against coupling it to comp directly.
- It is **not** a substitute for the research above. Picking a markup style without `R-BA-1` through `R-BA-6` produces a system that fits no real authoring pattern.

---

## How this thread closes

Either:

1. `R-BA-1` through `R-BA-6` produce enough signal that a proposal for `add-block-authorship-experiment` is drafted under `openspec/changes/`. The experiment runs on **one** engagement before any generalization.
2. Research concludes that **git blame + `D-*` rows + canvas-message provenance** together already cover the practical value, and explicit block markup is overhead without payoff. Thread closes as resolved-negative; the conclusion gets recorded as a `D-*` row.

Either is a valid closing.

---

## Adjacent prior thinking

| Source | Relationship |
|---|---|
| [`value-attribution.md`](./value-attribution.md) | Parent research thread. This is one mechanism candidate under § Q1. |
| [`specialist-network.md § 6`](../specialist-network.md) | The contribution model the studio already gestures at; block authorship would be a finer-grained version. |
| [`specialist-network.md § 8`](../specialist-network.md) | Compensation. `B4` (Goodhart) is the warning against coupling these directly. |
| [`rap-119-as-research-bed.md`](./rap-119-as-research-bed.md) | Observation protocol that wires `R-BA-1`, `R-BA-2`, `R-BA-3`, `R-BA-5`, and `B2` to RAP-119 surfaces. First empirical test of this thread. |

---

## Change log

| Date | Author | Change |
|---|---|---|
| 2026-05-07 | Pavel (founder) | Opened thread. No commitments. Captured B1–B5 problems and R-BA-1 to R-BA-6 questions. |
