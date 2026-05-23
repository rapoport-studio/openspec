---
title: Atlas — entity surface review (Round 1, 2026-05)
status: stable
owner: pavel
audience: [pavel, architect, executor]
date: 2026-05-22
ratifies: D-ATLAS-1..8
artifact: openspec/_atlas/2026-05/atlas.html
---

# Atlas — entity surface review (2026-05)

Retroactive AI-as-reviewer pass over the studio's entity surface, producing the
first ratified **Entities Atlas**. This document is the prose record; the visual
artifact lives at `openspec/_atlas/2026-05/atlas.html`; the machine-readable
decisions live at `openspec/_atlas/2026-05/decisions.yaml`.

## Thesis

Three drift axes destabilise the studio's entity surface. The fix is one canonical
visual reference (the Atlas) plus eight binding decisions that resolve specific
contradictions in code, tokens, and vocabulary.

## Drift findings (12)

| # | Severity | Finding |
|---|---|---|
| 1 | high | `--status-live` and `--status-embargo` both alias `var(--color-accent)`. Semantically opposite states render identically. |
| 2 | high | `featured-work-tile.tsx` hand-rolls a parallel renderer for the same entity that `work-card.tsx` renders through `<EntityCard>`. |
| 3 | high | Three uncoordinated status palettes (`StatusKind`, Forge phase palette, raw DB CHECK strings) with no resolver between them. |
| 4 | medium | `ideas` (top-level work session) shares a word with `canvas_ideas` (sticky notes). Two concepts, one name. |
| 5 | medium | 13 application sites define a private component also named `StatusPill` that does not use the design-system primitive. Grep overstates DS adoption by ~40%. |
| 6 | medium | `<SpecNavigator>` ships with the same `inline | row | card | detail` density grammar as `<EntityCard>` but has no production consumer. |
| 7 | medium | The `canvas` vocabulary persists in app code despite the 2026-05-22 schema migration to `ideas/specs/projects`. |
| 8 | low | `canvas` registry entry uses icon `Layers` while the catalog claims `MessagesSquare`. Iconography drift. |
| 9 | low | Forge phase palette (`--color-done/running/...`) is undocumented in the design-system spec but consumed by `<PhasePipeline>`. |
| 10 | low | Idea lifecycle stages and status pills coexist visually with no rule for which axis a UI element belongs to. |
| 11 | low | `apps/web` and `apps/studio` use different StatusPill geometries (border-radius, padding) for the same domain meaning. |
| 12 | observation | `canvas_invitees` should be `spec_invitees` — invites are issued against a Spec, not a Canvas/Idea. |

## Cohort

Source extraction prompt and full report archived at
`_methodology/research/atlas-entities-2026-05/extraction-report.md`
(if/when added). Cohort baseline:

- 22 `<EntityCard>` usages in the codebase (true DS adoption).
- 13 `StatusPill` shadow definitions outside the DS.
- 43 entity-id values in `packages/ui/src/entities/registry.ts`, of which 30
  are user-facing, 14 are Tier-1, 6 are Tier-2.

## Method

Retroactive: code-first reading of the current state of `apps/`, `packages/ui`,
`openspec/current/`, `openspec/brands/studio/`, and the migrations folder. No
forward-looking research — strictly documenting what exists and what
contradicts what.

## Findings ratified as decisions

See `openspec/_atlas/2026-05/decisions.yaml` for the canonical D-ATLAS-1..8.
Each carries one or more CI guards consumed by `scripts/atlas-verify.ts`.

## Open questions

1. Should the snapshot surface (frozen, read-only) get its own status palette
   overlay, or just inherit light surface with `opacity: 0.85` on non-tactical
   content?
2. The embargo plum at `oklch(0.520 0.085 305)` is on the violet side of plum;
   consider `320` (warmer) if it reads too cold next to terracotta in side-by-
   side previews.
3. Atlas v2 should evaluate whether `forge-run` belongs in Tier-1 (currently
   Tier-2). It carries a status palette, a lifecycle, and a renderer of its own
   — properties consistent with Tier-1 promotion.

## Source artifacts

- Visual: `openspec/_atlas/2026-05/atlas.html` (frozen, single-file bundle).
- Decisions: `openspec/_atlas/2026-05/decisions.yaml`.
- Notes: `openspec/_atlas/2026-05/notes.md`.
- Capabilities: `openspec/current/entities.md`,
  `openspec/current/entity-densities.md`,
  `openspec/current/status-resolver.md`.
- Entity portraits: `openspec/brands/studio/entities/*.md`.
- Change vehicle: `openspec/archive/<YYYY-MM-DD>-ratify-entities-atlas/`.
