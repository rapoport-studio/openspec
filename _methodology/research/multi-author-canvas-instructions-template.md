# Multi-author canvas — project_instructions template

> **Status: template, never deployed.** Distilled from cancelled
> [RAP-119](https://linear.app/rapoport-studio-slr/issue/RAP-119)
> `add-team-onboarding-canvas` § Project instructions (see
> [`openspec/archive/2026-05-08-add-team-onboarding-canvas/design.md`](../../archive/2026-05-08-add-team-onboarding-canvas/design.md)).
> Six-block shape (A identity / B course-meta / C authority /
> D intake-derived / E hard rules / F open questions registry) is
> reusable for any future multi-author canvas. Untested in production —
> first deployment validates it.

---

## When to use this template

When a canvas has more than one authoritative author and the canonical
single-`platform_owner` model does not fit. Examples:

- Co-authored product engagements (lead + N domain owners).
- External-collaborator engagements where authorship rights are shared.
- Course / onboarding canvases where the canvas doubles as a teaching
  environment.

For solo or owner-with-advisor canvases, the standard
`canvas_project_instructions` defaults are sufficient — no template
needed.

---

## Block A — Identity context

`kind=context`, `pinned=true`, `author=<lead>`.

```
This canvas is co-authored by <lead> + <domain-owner-1> + ... +
<domain-owner-N>. They are equals in business standing. They are
co-creating <artefact / product / spec corpus>. Each is authoritative
within their stated domain. When responding, attribute decisions to
their stated source.
```

Fill `<lead>`, `<domain-owner-N>` with real names. Keep the wording —
the explicit "equals in business standing" + "authoritative within
their stated domain" framing is what tells Muse to attribute, not
synthesise.

---

## Block B — Course-meta directive

`kind=context`, `pinned=true`, `author=<lead>`.

Use **only when the canvas doubles as a teaching environment**. Skip
for purely productive multi-author canvases.

```
This canvas is also a teaching environment. When a methodology decision
happens — creating a new triplet, naming a capability, choosing where a
paragraph goes — Muse briefly explains what it is doing and why.
Concise. Two sentences max. Not lecturing.
```

This is the directive referenced by
[`value-attribution.md`](./value-attribution.md) as the "visibility
built into the Muse turn at the moment of the decision" pattern. First
deployment of this directive is also the first empirical test of
whether it lands as helpful or condescending.

---

## Block C — Authority map

`kind=constraint`, `pinned=true`, `author=<lead>`.

```
| Role | Person | Authority |
|---|---|---|
| Lead | <lead> | Corpus convention, final call on architecture, Muse infrastructure |
| Domain owner — <domain-1> | <person> | <files this owner authors> |
| Domain owner — <domain-2> | <person> | <files this owner authors> |
| ...

Cross-domain edits flag for review by relevant owner. Soft-enforcement,
not RLS. Trust over policy at small team sizes.
```

Soft-enforcement = anyone can edit anything; cross-owner edits surface
as PR comment, not auto-rejection. RLS-level enforcement is a separate
decision and not part of this template.

---

## Block D — Intake-derived preferences

**Deliberately empty pre-kickoff.**

`kind=preference`, `pinned=true`, `author=<member>`. Filled live during
the kickoff session's Loop 0 (per [`loop-protocol.md`](./loop-protocol.md)
§ Session-level skeleton), one row per ratified Layer-1 quote from
the intake harvest. Format per row:

```
<member> said: "<verbatim quote>" — <source citation, e.g. chat 25.04 14:32>.
When drafting in this canvas, <behaviour shaped by quote>.
```

`kind=vocabulary`, `pinned=true`, `author=<member>`. Filled live during
Loop 0, one row per ratified Layer-2 term. Format per row:

```
<member> uses "<term>" to mean: <definition in member's own words> —
<source>. Use this term when responding to <member>; default to
<lead's term> when responding to others.
```

**Critical pattern — live ratification.** The intake harvest *file* is
prepared pre-kickoff, but the *pinned canvas rows* are produced at the
table during Loop 0, with each member confirming or correcting their
attributed quotes live. Mitigates the "pre-analyzed team" risk:
members do not walk into a canvas where their own words are already
pinned and processed without their consent. Originated in RAP-119
`design.md` § Intake stage; remains design-level until first deployed.

---

## Block E — Hard rules

`kind=constraint`, `pinned=true`, `author=<lead>`.

```
Same as platform: no direct edits to current/, three-file triplets
(proposal.md + design.md + tasks.md), archive on merge, prose-style,
no Requirement-blocks. Change folder slug = bare while active,
date-prefixed at archive.
```

Engagement-specific carve-outs (e.g. shared corpus location, custom
review cadence) go in a separate row with explicit attribution to the
decision document.

---

## Block F — Open questions registry

`kind=context`, `pinned=true`, `author=<lead>`. Seeded with known
opens; Muse refers back rather than re-deriving each turn.

```
| ID | Question | Source | Resolves at |
|---|---|---|---|
| Q-<topic-1> | <question> | <design doc § Decisions> | <when / how> |
| Q-<topic-N> | <question> | ... | ... |
```

Add new rows as `[DECISION-CANDIDATE]` markers surface in chat /
session loops.

---

## Provenance

- Originated as
  [`openspec/archive/2026-05-08-add-team-onboarding-canvas/research/project-instructions-seed.md`](../../archive/2026-05-08-add-team-onboarding-canvas/research/project-instructions-seed.md)
  on 2026-05-07 (PR #262), with hardcoded names (Pavel + Viorica +
  Alexandr + Rodion) and engagement-specific F-block rows
  (Q-RADION, Q-NAME, Q-LEGAL, Q-ONBOARDING-EXPLAIN). Archived 2026-05-08.
- Engagement cancelled 2026-05-08 without deployment. Template
  generalised here so the six-block shape survives outside the
  engagement frame.
- First deployment of this template will be the first empirical test
  of:
  - Whether Block B course-meta lands as helpful or condescending
    (per [`value-attribution.md`](./value-attribution.md)).
  - Whether Block D live-ratification mitigates the pre-analyzed-team
    risk in practice (per
    [`openspec-block-authorship.md`](./openspec-block-authorship.md)).
