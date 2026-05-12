---
title: Audience Policy — Diataxis tags per artifact type
status: stable
owner: pavel
audience: [founder, architect, ai]
date: 2026-05-12
tldr: Declares which Diataxis quadrant and audience tags apply to each OpenSpec artifact type (proposal, design, tasks, capability, research, methodology, README). Muse reads this file at startup to set audience tags in generated proposal/design/tasks frontmatter.
---

# Audience Policy — Diataxis tags per artifact type

> Source of truth for audience tag assignments across the studio's OpenSpec corpus. Every new artifact type must be listed here before use.
>
> Research backing: [`_methodology/research/multi-audience-documentation.md`](research/multi-audience-documentation.md)

---

## Diataxis quadrants

[Diataxis](https://diataxis.fr/start-here/) splits documentation by **reader intent**, not topic. Mixing quadrants in one document is the leading cause of confusing specs.

**Tutorial** — Learning by doing. Leads the reader through steps to complete a project. The reader is a beginner; their goal is to build confidence. Outcome: the reader can say "I did it." Studio example: [`README.md`](../../../README.md) — first-time setup for a new engineer or founder.

**How-to** — Solving a specific task. Assumes the reader knows what they want; guides them through the steps to do it. Outcome: the problem is solved. Studio example: [`openspec/changes/bootstrap-cli-orchestra-mvp/tasks.md`](../changes/bootstrap-cli-orchestra-mvp/tasks.md) — executor steps to ship a change.

**Reference** — Information lookup. Dry, precise, complete. The reader already knows their goal; they need the facts. Outcome: the reader found what they needed. Studio example: [`openspec/current/forge/README.md`](../current/forge/README.md) — canonical Forge capability description.

**Explanation** — Building understanding. Discusses context, rationale, and trade-offs. No action items. Outcome: the reader understands why. Studio example: [`openspec/changes/bootstrap-cli-orchestra-mvp/proposal.md`](../changes/bootstrap-cli-orchestra-mvp/proposal.md) — motivation and framing for a change.

---

## Per-artifact assignment

| Artifact | Diataxis quadrant | Audience tags | Canonical example |
|---|---|---|---|
| `proposal.md` | Explanation | `[founder, architect, executor]` | [`openspec/changes/bootstrap-cli-orchestra-mvp/proposal.md`](../changes/bootstrap-cli-orchestra-mvp/proposal.md) |
| `design.md` | Reference | `[architect, executor]` | [`openspec/changes/bootstrap-cli-orchestra-mvp/design.md`](../changes/bootstrap-cli-orchestra-mvp/design.md) |
| `tasks.md` | How-to | `[executor]` | [`openspec/changes/bootstrap-cli-orchestra-mvp/tasks.md`](../changes/bootstrap-cli-orchestra-mvp/tasks.md) |
| capability spec (`current/*.md`) | Reference | `[architect, executor, ai]` | [`openspec/current/forge/README.md`](../current/forge/README.md) |
| research (`_methodology/research/*.md`) | Explanation | `[founder, architect, ai-engineer]` | [`openspec/_methodology/research/multi-audience-documentation.md`](research/multi-audience-documentation.md) |
| methodology (`_methodology/*.md`) | Reference | `[ai, architect, executor]` | [`openspec/_methodology/risk-taxonomy.md`](risk-taxonomy.md) |
| `README.md` | Tutorial | `[founder, executor]` | [`README.md`](../../../README.md) |

---

## How Muse uses this

Muse reads `openspec/_methodology/audience-policy.md` at Architect-mode startup to pre-populate `audience:` tags in generated frontmatter. When Muse drafts a `proposal.md`, it emits `audience: [founder, architect, executor]`; when it drafts `design.md`, it emits `audience: [architect, executor]`; and so on per the table above. This eliminates per-artifact guessing and keeps all generated docs consistent with the studio's Diataxis policy.
