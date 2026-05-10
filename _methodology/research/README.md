# `_methodology/research/` — research artifact convention

Research files live here. Each file is a self-contained corpus that backs one or more downstream artifacts (Lab articles, positioning copy, future research). The convention below ensures any specialist who reads a research file can understand its provenance, method, and reusability without asking the original author.

> **Convention introduced:** `openspec/archive/2026-05-10-add-research-authorship-convention/` (2026-05-10, archived 2026-05-11).

---

## Frontmatter block — required shape

Every research file SHOULD open with a bold-prefix blockquote like this:

```markdown
# <Research title>

> **Status:** open research | draft | complete | archived
> **Owner:** <name> (role).
> **Authors:** <Primary Name> <email> (role, primary); <Collaborator 1> (role); <AI Collaborator> (role) — <what they did>
> **Method:** <one paragraph: literature synthesized, measured first-party, interviewed, audited>
> **Reusable for:** <comma-separated downstream slots>; authorship: <open | named-successor-only>
> **Opened:** YYYY-MM-DD
> **Companion to:** [`./<related-file>.md`](./<related-file>.md)
> **Feeds into:** <downstream consumer>
> **Path note:** <if file moved or path is non-obvious>
```

---

## Field-by-field

### `Status` (mandatory)
One of `open research` (active intake), `draft` (in progress), `complete` (final), `archived` (frozen, do not edit per memory note). Free-form one-line description of state is also acceptable.

### `Owner` (mandatory)
Who is accountable for shipping or archiving the file. Usually but not always the same as the primary Author. Owner answers reviewer questions about the artifact's current state and lifecycle.

### `Authors` (mandatory on new files)
Primary human author first with email, then any collaborators. Format examples:

- `Pavel Rapoport <pavel@rapoport.studio> (founder, primary)`
- `Pavel <pavel@rapoport.studio> (primary); Andrei <andrei@rapoport.studio> (specialist, contributed sections 3-5)`
- `Pavel <pavel@rapoport.studio> (primary); Claude Opus 4.7 (Anthropic) — research, drafting, GitHub API audits`

**AI collaborators must be named.** The studio's stance is that AI is a collaborator, not an author; but its contribution must be visible. Include what the AI specifically did.

### `Method` (mandatory on new files)
One scannable paragraph describing how the research was conducted. The method statement is what a downstream author uses to write honestly about the research's limits. Examples:

- *«Multi-pass — literature synthesis from 15+ public comparisons; failure-mode catalogue from 30+ GitHub issues; first-party empirical benchmark; GitHub API health-metrics audit.»*
- *«Single-pass — secondary synthesis of public Tech Radar releases; no first-party data.»*
- *«Field — three semi-structured interviews with Series A founders in Berlin; transcripts not retained per agreed confidentiality.»*

### `Reusable for` (mandatory if public-facing)
Comma-separated downstream artifact slots, plus authorship policy. Examples:

- *«Lab MDX article authorship (any specialist); positioning copy; future SDD-track research. Authorship: open.»*
- *«Internal positioning copy only. Authorship: named-successor-only — Pavel.»*
- *«Citable but not redistributable; sources include confidential interviews. Authorship: named-successor-only.»*

If the field is omitted, the default is conservative: **named-successor-only**, internal-citable, not redistributable.

### `Opened` (mandatory)
ISO date `YYYY-MM-DD` of when the file was created.

### `Companion to` / `Feeds into` / `Path note` (optional)
Cross-references. Keep as markdown links.

---

## When is each field mandatory?

| Field | New file | Existing file (pre-convention) |
|---|---|---|
| Status | required | required (already convention) |
| Owner | required | required (already convention) |
| Opened | required | required (already convention) |
| **Authors** | **required** | not retroactively required |
| **Method** | **required** | not retroactively required |
| **Reusable for** | required if public-facing | not retroactively required |
| Companion to / Feeds into / Path note | as applicable | as applicable |

A separate change folder may backfill `Authors` / `Method` / `Reusable for` for high-leverage older files. Not enforced.

---

## Why not YAML frontmatter?

Continuity with the 13 existing research files. Bold-prefix is greppable for tooling (`grep -E '^> \*\*Author' *.md`), readable for humans, and doesn't force a one-shot migration. YAML is a candidate for a future migration if needed.

## How does this interact with block-level authorship?

[`openspec-block-authorship.md`](./openspec-block-authorship.md) explores **block-level** attribution — paragraph or section granularity. This file-level convention is the lighter, immediately-shippable baseline. Block-level layered on top later if/when the studio needs that depth.

## Examples in this directory

- [`openspec-vs-speckit-public.md`](./openspec-vs-speckit-public.md) — first file written under the new convention (2026-05-10). All three new fields present.
- Other files predate the convention; metadata backfill is deferred to a separate change folder.

---

## Reference

- Convention introduced: [`openspec/archive/2026-05-10-add-research-authorship-convention/`](../../archive/2026-05-10-add-research-authorship-convention/)
- Block-level authorship research (open): [`openspec-block-authorship.md`](./openspec-block-authorship.md)
- Studio AGENTS workflow: [`openspec/AGENTS.md`](../../AGENTS.md)
- Project CLAUDE.md (hard rules): [`/CLAUDE.md`](../../../CLAUDE.md)
