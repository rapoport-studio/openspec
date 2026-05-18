---
title: SDD methodology positioning — where Rapoport Studio sits
status: ratified
owner: pavel
audience: [pavel, network-specialists, methodology-reviewers]
upstream:
  - _methodology/studio-charter.md
  - _methodology/positioning.md
  - AGENTS.md
companion:
  - changes/add-sdd-methodology-positioning/
  - notes/community-pointers.md
---

# SDD methodology positioning — where Rapoport Studio sits

> Internal methodology doc. **Not published publicly** — no `/lab` page, no
> marketing surface. It fixes how Rapoport Studio describes its own
> spec-driven-development (SDD) practice relative to the wider SDD landscape,
> so that the studio neither overstates its differentiator nor mistakes
> upstream methodology for its own invention.
>
> Companion: [`changes/add-sdd-methodology-positioning/`](../changes/add-sdd-methodology-positioning/) — the change that authored this file. Sister watchlist: [`notes/community-pointers.md`](../../notes/community-pointers.md).
>
> Distinct from [`positioning.md`](./positioning.md) (the Canvas Studio *commercial* thesis). That file answers "what do we sell"; this file answers "where does our *method* sit in the field".

---

## 1. Where we sit in the SDD landscape

Spec-driven development has, in 2025–2026, gone from a fringe practice to a
crowded field. Several tools and frameworks now compete to define what "specs
drive the work" means. The studio adopted **OpenSpec** before that field
crystallised; this section maps the landscape so the studio's relation to each
neighbour is explicit rather than assumed.

| Tool | What | Our relation |
|---|---|---|
| OpenSpec (Fission AI / @0xTab) | SDD framework — source-of-truth + delta specs | Used + extended (our upstream/parent methodology) |
| Beads (Steve Yegge) | Graph task-tracker as agent memory | Linear plays the equivalent role; Beads-pattern relevant only for a future offline mode |
| Gas Town (Steve Yegge) | Multi-agent orchestrator, 20–30 parallel agents | Reference point for the Maestro pattern; deliberately not our scale |
| Augment Intent | Coordinator + Implementor + Verifier (enterprise SaaS) | Direct architectural analog of our Maestro / Forge / Muse split |
| Spec-Kit (GitHub) | Heavyweight, phase-gated SDD | Rigid; the lighter OpenSpec philosophy is the one we adopted |
| Kiro (AWS) | IDE-locked SDD | Vendor-lock anti-pattern |
| BMAD | Persona-driven SDLC simulation | Anti-pattern — the "19-agent trap" |

### Key findings

- **Our differentiator is narrower than "we have target-state specs".**
  Vanilla OpenSpec 1.0 already ships `openspec/specs/` as a living
  source-of-truth directory. Having target-state specs is therefore *table
  stakes*, not a moat. What is actually distinctive about the studio's practice
  is three things, none of which is "we wrote specs":
  1. **Capability-split** — the corpus is decomposed into per-capability files
     (`openspec/current/<capability>.md`), each carrying its own status and
     lifecycle, rather than one monolithic spec.
  2. **`verification.md` as a first-class change artifact** — every change
     triplet is in fact a *quadruplet*; deterministic acceptance assertions
     are authored alongside `proposal.md` / `design.md` / `tasks.md`, not
     improvised at review time.
  3. **Forge as an execution adapter** — the spec corpus is not just
     human-readable documentation; it is the input contract for an automated
     build executor that turns a change into a PR.
- **"Living specs" is the industry consensus.** Augment's Intent product,
  Steve Yegge's writing, and the recent academic work on SDD all converge on
  the same claim: the spec must be a maintained, queryable artifact, not a
  write-once requirements doc. The studio is *aligned with* this consensus, not
  ahead of it. Claiming "living specs" as a unique selling point would be a
  positioning error.
- **`/loop` traces to Beads.** Anthropic has publicly credited Steve Yegge's
  Beads as the inspiration for Claude Code's `/loop` primitive. The studio uses
  `/loop`; the lineage is worth knowing because it makes the Beads task-graph
  pattern a *named upstream*, not an exotic idea.
- **Multi-repo spec management is an open OpenSpec problem.** OpenSpec issue
  #725 tracks the unsolved question of managing one spec corpus across multiple
  repositories. The studio already side-steps it: a single monorepo plus
  `forge.config.mjs` issue-prefix routing means one corpus serves every app and
  package without cross-repo synchronisation. This is a solved problem for us
  by construction, not by feature work.

---

## 2. What we share and where we diverge

### Shared

With OpenSpec, Augment Intent, and Gas Town the studio shares four practices:

- **Living specs** — the spec corpus is maintained as the work proceeds, not
  frozen at kickoff.
- **Capability decomposition** — the corpus is split along capability lines so
  each concern evolves independently.
- **Verification discipline** — "done" is defined by deterministic acceptance
  assertions, not by reviewer impression.
- **An execution-adapter layer** — a component that consumes the spec and
  produces code (Forge, for the studio; the Implementor, for Augment).

### Diverge

The studio keeps `openspec/current/` as its capability-spec directory rather
than adopting vanilla OpenSpec 1.0's `openspec/specs/`. This is a deliberate,
ratified divergence — decision **D-MTH-1** in
[`decisions.md`](../decisions.md). The rationale (rename churn, symlink
fragility under the Cloudflare Workers + OpenNext build, and `current/` being
semantically more precise for an internal-first monorepo) is recorded in full
on that decision; it is not restated here.

### Terminology: "spec gap"

The studio adopts the industry-standard term **"spec gap"** for the divergence
between what a spec says and what the code actually does. This term replaces any
bespoke internal language for the same concept. "Spec gap" is the default
vocabulary across the methodology corpus, change proposals, and review notes
from this point forward — drift, divergence, and ad-hoc phrasings for spec↔code
mismatch are normalised to it.

---

## 3. Anti-patterns we explicitly reject

The SDD field also contains practices the studio has examined and deliberately
rejected. Naming them here prevents their quiet re-introduction.

- **BMAD-style persona simulation — the "19-agent trap".** BMAD models the
  software development lifecycle by instantiating many fake SDLC-role personas
  (a "product manager" agent, a "QA" agent, a "scrum master" agent, and so on).
  The studio rejects this. Personas multiply coordination overhead and
  token cost without adding decision quality; the studio's agent roster
  (Maestro / Muse / Forge and the rest) is sized to *real* distinct functions,
  not to a simulated org chart.
- **Gas Town's industrial-scale agent burn.** Gas Town orchestrates 20–30
  agents in parallel. That scale is incompatible with the studio's cost
  discipline and with its hard concurrency cap of two parallel Forge builds.
  Gas Town is a useful *reference point* for the Maestro orchestration pattern,
  but its scale is explicitly not a target.
- **Kiro's IDE/vendor lock.** Kiro binds the SDD workflow to a specific IDE.
  The studio's corpus is plain markdown under `openspec/`, readable and
  editable by any tool; binding the methodology to a single vendor's editor is
  an anti-pattern the studio will not adopt.

---

## Change log

- `2026-05-18` — initial draft, ratified status, authored as part of change
  [`add-sdd-methodology-positioning`](../changes/add-sdd-methodology-positioning/) (RAP-1078). Ratifies decisions D-MTH-1 and D-MTH-2; seeds the community watchlist at `notes/community-pointers.md`.
