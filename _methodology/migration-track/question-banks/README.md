# Migration Track — question banks

The comprehension gate (`migration-track.md` § 7.2) draws three questions per
learner per phase — one structural, one application, one synthesis. Those
questions are pulled from the banks held in this directory.

## Structure (planned)

One file per phase: `P0.md`, `P1.md`, … `P6.md`. P3 splits per entity cluster
(`P3-<cluster>.md`). Each file holds questions grouped by level — structural,
application, synthesis — with enough variants that the two learners on a run
never receive the same question, and a retake never repeats a prior attempt.

## Generic vs run-specific (open question Q1)

Two kinds of question coexist:

- **Structural questions** can be largely **generic** — reusable across runs,
  because "name the entities in this cluster and the role of each" is
  product-agnostic in shape.
- **Application and synthesis questions** are **run-specific** — they reference a
  fresh code snippet or a hypothetical extension of *this* product, so they are
  regenerated per run by the studio architect (the `populate-vvd1-question-banks`
  work, for run #1).

This split is the working default, not a ratified decision — see
`migration-track.md` § 13 Q1. Revisit after the vivod pilot.

## Status

Stub. The banks themselves are populated per run before P1 starts. For run #1
(vivod / VVD-1) this is tracked by the `populate-vvd1-question-banks` Linear
issue.
