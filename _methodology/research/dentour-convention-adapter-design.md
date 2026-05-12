---
title: Dentour Convention Adapter — Design Notes
description: Studio-side design for the Forge convention adapter that handles Dentour's statement-style formal-spec OpenSpec dialect with commitlint + Changesets. Authored during Phase 6 of `onboard-dentour`. Implementation deferred to `add-forge-dentour-convention-adapter`.
---

# Dentour Convention Adapter — Design Notes

<!-- openspec-refs: allow-unresolved -->
<!-- This design references Dentour-internal openspec paths (`openspec/specs/<capability>/spec.md`, `openspec/changes/<slug>/specs/<cap>/spec.md`, `BRANCHING_STRATEGY.md`, `commitlint.config.js`, `version-check.yml`) inside `dentour-org/dentour`, NOT this repo. They are not actionable refs from the studio side. Mirrors the same opt-out applied to dentour-topology.md. -->

> Studio-internal design notes for the Forge convention adapter that will land in `packages/forge/src/conventions/dentour.ts`. Authored during Phase 6 of [`onboard-dentour`](../../changes/onboard-dentour/) (T6.1). Companion to [`dentour-topology.md`](./dentour-topology.md) and [`dentour-drift-audit.md`](./dentour-drift-audit.md). Implementation tracked separately as [`add-forge-dentour-convention-adapter`](../../changes/add-forge-dentour-convention-adapter/) — opened in T6.2 as a proposal-only shell.

```yaml
design_date: 2026-05-12
dentour_commit_sha: 2e96f677b7557b59530fbae87387b28e088defe3   # same anchor as topology
dentour_repo_url: https://github.com/dentour-org/dentour
shareability: studio-internal-default
author: forge-subagent
companion_docs:
  - dentour-topology.md (Phase 1 — dialect grammar source)
  - dentour-drift-audit.md (Phase 2 — capability table, Phase-0 skeleton notes)
adapter_framework: openspec/archive/2026-05-08-add-forge-convention-adapter/   # the parent that introduced OpenSpecConventionAdapter
target_implementation: openspec/changes/add-forge-dentour-convention-adapter/  # proposal-only shell opened in T6.2
target_complexity_estimate: 2 weeks of focused work (honest ceiling 3 — see § 7)
non_goal: do NOT invoke Dentour's own automation skills programmatically (see § 8)
```

---

## 0. Premise

Dentour ships its own OpenSpec corpus in a **statement-style formal-spec dialect** that the studio's prose convention does not match. The dialect is documented exhaustively in [`dentour-topology.md § 7`](./dentour-topology.md). The convention adapter framework introduced by [`add-forge-convention-adapter`](../../archive/2026-05-08-add-forge-convention-adapter/) (RAP-86, archived 2026-05-08) gives Forge a stable seam — `OpenSpecConventionAdapter` at [`packages/forge/src/conventions/adapter.ts`](../../../packages/forge/src/conventions/adapter.ts) — for slotting per-engagement convention rules without leaking project-slug conditionals into the build agent.

This document specifies the Dentour adapter that fills that seam for Dentour. It is **design-only**: no code is committed to `packages/forge/` as part of `onboard-dentour`. The implementation lives in a separate change folder and is run as a separate Linear issue once kickoff ratifies engagement mode (per [`onboard-dentour` design.md § Engagement mode default](../../changes/onboard-dentour/design.md)).

The Dentour adapter differs from both the studio adapter and the Vivod adapter in one structurally important dimension: **commit message governance uses Changesets alongside commitlint**, and **validation is a local-only step (not a CI gate)**. These two facts drive several design decisions that have no equivalent in the Vivod adapter.

---

## 1. Dentour's spec dialect — what the adapter must read

### 1.1 Source-of-truth files

Per [`dentour-topology.md § 7.1`](./dentour-topology.md):

- Dentour has **no `openspec/current/` folder**. Capabilities live at `openspec/specs/<capability>/spec.md`.
- All twelve capability specs use a single `spec.md` file (no `index.md` / `spec.md` duality, no Fumadocs frontmatter layer). No filename-bridge script is needed.
- **Seven of twelve specs are stable; five are Phase-0 skeletons** (per [`dentour-topology.md § 7.2`](./dentour-topology.md)). The adapter must parse all twelve but should treat the five skeletons as "valid but requirement-empty" — no special error.

### 1.2 Frontmatter

Dentour's `spec.md` files do **not** carry YAML frontmatter. Per [`dentour-topology.md § 7.3`](./dentour-topology.md), the OpenSpec config (`openspec/config.yaml`) governs authoring rules, not the file header. The adapter MUST NOT inject or require a frontmatter block when reading or writing Dentour spec files — doing so would break Dentour's `pnpm openspec:validate --strict` run.

### 1.3 Required structural sections

Per [`dentour-topology.md § 7.3`](./dentour-topology.md) and confirmed by reading stable specs at the anchored commit:

- `## Purpose` — mandatory. Every spec (stable + skeleton) opens with this section.
- `### Requirement: <statement>` — H3 headings forming the requirement blocks (full sentence in the heading, no `REQ-XXX-NNN` ID; see § 1.4).
- `#### Scenario: <name>` — H4 blocks under each requirement (see § 1.5).
- `## Implementation map` — mandatory per `openspec/config.yaml § rules.spec`. Present in all seven stable specs, pointing to `apps/<app>/src/...` and `packages/<pkg>/...` source files.
- `## Known gaps / Acknowledged risks` — canonical section name per `openspec/config.yaml`. Three stable specs use it; two use `## Status` for skeleton lifecycle metadata. The parser accepts both; the adapter preserves whichever name is already in the file.

### 1.4 Requirement headings — single variant

Dentour uses a single heading pattern across all stable specs (per [`dentour-topology.md § 7.3`](./dentour-topology.md)):

```
### Requirement: <statement>
```

Unlike Vivod (which has two heading variants), Dentour has one canonical form. The full requirement statement appears in the heading; there is no numeric `REQ-XXX-NNN` identifier. The adapter parser therefore:

- Matches `/^### Requirement: (.+)$/m` (exact prefix `### Requirement: `).
- Extracts the full statement string as the requirement body.
- Does **not** expect or assign numeric IDs; requirement identity is the statement text.

**Implication for this design doc:** Requirements in this document follow Dentour's own convention (`REQ-DCA-NNN`) for studio-internal accounting, while noting that the runtime format Dentour uses is statement-text, not numeric ID. See § 2 for the `REQ-DCA-NNN` requirement table.

### 1.5 Scenario blocks

Each requirement may carry zero or more Scenario blocks:

```markdown
#### Scenario: <name>

- **WHEN** <trigger>
- **THEN** <expected outcome>
```

Optional GIVEN preamble (observed in `clinic-profile/spec.md` and `quote-workflow/spec.md`):

```markdown
#### Scenario: <name>

- **GIVEN** <preamble>
- **WHEN** <trigger>
- **THEN** <expected outcome>
```

Per [`dentour-topology.md § 7.3`](./dentour-topology.md):

- `**WHEN**` and `**THEN**` are required.
- `**GIVEN**` is optional; the parser emits `given=null` when absent.
- Multi-line clauses are wrapped onto the next bullet; the parser collapses them into a single logical string.

### 1.6 No Acceptance Criteria tables

Dentour's stable specs do **not** carry `## Acceptance Criteria` tables (unlike Vivod). The parser need not recognise this section; if it is encountered, it is treated as a generic `## prose-section` and round-tripped verbatim.

### 1.7 Change-folder layout (formal-deltas sub-folder)

Per [`dentour-topology.md § 7.3`](./dentour-topology.md) and the `openspec/config.yaml § rules.proposal` authoring conventions:

- Active changes live at `openspec/changes/<slug>/` with a sub-folder `specs/<capability>/spec.md` carrying the formal delta.
- Dentour's `openspec/changes/` directory is **empty at HEAD** (only an empty `archive/` subtree). No worked example is in-tree; the studio's first Forge run will be the first delta authored on this corpus.
- The adapter writes deltas to `openspec/changes/<slug>/specs/<capability>/spec.md` — the formal-deltas sub-folder shape — which matches both Vivod's pattern and Dentour's documented convention. `proposal.md`, `design.md`, `tasks.md` may also appear at the root of the change folder when the change requires them.

---

## 2. Adapter requirements (REQ-DCA-NNN)

The adapter conforms to the [`OpenSpecConventionAdapter`](../../../packages/forge/src/conventions/adapter.ts) interface introduced by RAP-86. Each requirement below maps to one or more methods on that interface.

### REQ-DCA-001 — Slug + name

The adapter exposes `slug = 'dentour'` and `name = 'Dentour (statement-style formal-spec + commitlint + Changesets)'`. Registered in [`packages/forge/src/conventions/index.ts`](../../../packages/forge/src/conventions/index.ts) alongside `studio`, `unbind`, and `vivod`.

#### Scenario: registry resolution

- **WHEN** `resolveAdapter('dentour')` is called
- **THEN** the function returns the Dentour adapter instance with the slug `'dentour'` and the name above.

### REQ-DCA-002 — Source-of-truth file detection

The adapter recognises `openspec/specs/<capability>/spec.md` as the canonical source-of-truth file. There is no filename bridge; `spec.md` is always the direct source. The adapter does NOT look for `index.md` in Dentour spec directories.

#### Scenario: read stable capability

- **GIVEN** a capability folder `openspec/specs/auth/` containing `spec.md`
- **WHEN** the adapter loads the spec for `auth`
- **THEN** it reads from `spec.md` without error.

#### Scenario: read skeleton capability

- **GIVEN** `openspec/specs/dental-chart/spec.md` containing only `## Purpose` and a skeleton `### Requirement: skeleton — to be filled in Phase 1` placeholder
- **WHEN** the adapter loads the spec
- **THEN** it loads successfully and emits zero requirements (skeleton is valid but empty).

#### Scenario: refuse to load missing spec

- **GIVEN** no `spec.md` in a supposed capability folder
- **WHEN** the adapter attempts to load
- **THEN** it surfaces a structured `AdapterViolation` with `severity='error'`, `message='No spec.md found in openspec/specs/<capability>/ — Dentour specs must reside at spec.md (not index.md).'`

### REQ-DCA-003 — No-frontmatter posture

The adapter's `capabilityFrontmatter()` method returns an empty string (no YAML block injected). When editing an existing spec, the adapter MUST NOT add a frontmatter block. Any upstream assertion that the file begins with `---\n` is a bug — the adapter flags it as an `AdapterViolation`.

#### Scenario: no frontmatter on new capability

- **WHEN** the build agent creates a new capability `messaging-ai` and calls `capabilityFrontmatter('messaging-ai', ctx)`
- **THEN** the adapter returns `''` (empty string).
- **AND** the build agent writes `## Purpose\n\n…` as the first line of the new `spec.md`.

#### Scenario: detect spurious frontmatter

- **GIVEN** a `spec.md` that begins with `---\ntitle: Auth\n---\n`
- **WHEN** the adapter parses
- **THEN** it surfaces `AdapterViolation` with `severity='warning'`, `message='Dentour spec.md begins with a YAML frontmatter block — this is not part of the Dentour convention and will break pnpm openspec:validate.'`

### REQ-DCA-004 — Statement-style requirement heading parser

The parser recognises `### Requirement: <statement>` H3 headings (exact prefix, case-sensitive). The full statement after the colon and space is the requirement identifier at authoring time.

```
/^### Requirement: (.+)$/m
```

#### Scenario: parse stable requirement

- **GIVEN** `openspec/specs/auth/spec.md` with heading `### Requirement: A Supabase magic-link session allows any browser without prior account`
- **WHEN** the parser tokenises the spec
- **THEN** it emits a Requirement token with `statement='A Supabase magic-link session allows any browser without prior account'`.

#### Scenario: parse skeleton placeholder

- **GIVEN** `### Requirement: skeleton — to be filled in Phase 1`
- **WHEN** the parser tokenises
- **THEN** it emits a Requirement token with `statement='skeleton — to be filled in Phase 1'` and a `isSkeleton=true` flag (for downstream filtering).

#### Scenario: reject malformed heading (wrong level)

- **GIVEN** a heading `## Requirement: something` (H2 instead of H3)
- **WHEN** the parser encounters it
- **THEN** the parser emits a `prose` token (treats it as a generic H2 section body) — NOT a Requirement token — and does not surface a violation. The validate step (§ 5) catches structural errors; the parser is permissive.

#### Scenario: reject stripped colon

- **GIVEN** a heading `### Requirement something` (missing colon-space)
- **WHEN** the parser encounters it
- **THEN** the parser surfaces an `AdapterViolation` with `severity='warning'`, `message='Possible malformed Requirement heading "### Requirement something" — expected "### Requirement: <statement>". Did you omit the colon?'` and continues.

### REQ-DCA-005 — Scenario block tokeniser

The parser recognises `#### Scenario: <name>` followed by `- **WHEN**` / `- **THEN**` (with optional `- **GIVEN**` preamble) under each Requirement.

#### Scenario: parse minimal scenario

- **GIVEN** a Requirement followed by:
  ```
  #### Scenario: Magic-link email delivery succeeds

  - **WHEN** the patient submits a valid email
  - **THEN** Supabase sends the magic-link and redirects to /check-email
  ```
- **WHEN** the parser tokenises
- **THEN** it emits a Scenario token with `name='Magic-link email delivery succeeds'`, `when='the patient submits a valid email'`, `then='Supabase sends the magic-link and redirects to /check-email'`, `given=null`.

#### Scenario: parse scenario with GIVEN preamble

- **GIVEN** a Scenario block carrying `- **GIVEN** the clinic has completed onboarding`
- **WHEN** the parser tokenises
- **THEN** the emitted Scenario carries the preamble in the `given` field.

#### Scenario: parse multi-line clause

- **GIVEN** a `**THEN**` clause wrapped onto the next bullet
- **WHEN** the parser tokenises
- **THEN** the emitted `then` field is collapsed to a single string.

### REQ-DCA-006 — `## Implementation map` section handling

The adapter recognises `## Implementation map` as a mandatory section in stable specs. When the build agent surfaces a new code-pointer link, it appends under this section. When writing a new capability spec, the adapter scaffolds the section with a placeholder.

#### Scenario: append code pointer

- **GIVEN** `auth/spec.md` with an existing `## Implementation map` section
- **WHEN** the build agent surfaces a new pointer to `packages/auth/src/supabase-client.ts`
- **THEN** the adapter appends the link under `## Implementation map`.

#### Scenario: scaffold for new capability

- **WHEN** the build agent creates a new capability `messaging-ai`
- **THEN** the adapter scaffolds `## Implementation map\n\n<!-- TODO: add implementation pointers -->\n` in the new `spec.md`.

#### Scenario: warn on absent Implementation map in stable spec

- **GIVEN** a stable spec (non-skeleton) missing `## Implementation map`
- **WHEN** `validate()` runs
- **THEN** `validate()` surfaces `AdapterViolation` with `severity='warning'`, `message='Stable spec openspec/specs/<capability>/spec.md is missing ## Implementation map — required by openspec/config.yaml § rules.spec.'`

### REQ-DCA-007 — Direct edits to `specs/` (no diff-at-archive)

`allowsDirectCurrentEdits()` returns `true`. Dentour's convention applies edits directly inside the change folder's `specs/<cap>/spec.md` sub-folder, then applies via Dentour's `/opsx:apply` skill. There is no diff-at-archive grammar.

### REQ-DCA-008 — Change-folder layout (formal-deltas sub-folder)

When the build agent creates a new change folder, the adapter scaffolds the sub-folder shape `openspec/changes/<slug>/specs/<capability>/spec.md`. The flat files (`proposal.md`, `design.md`, `tasks.md`) may also be created at `openspec/changes/<slug>/`.

#### Scenario: scaffold sub-folder delta

- **WHEN** `forge spec --fix --project dentour` is run for a change touching capability `auth`
- **THEN** the adapter creates `openspec/changes/<slug>/specs/auth/spec.md` carrying the delta in Dentour's statement-heading format.

#### Scenario: forbid flat-only layout for requirement-bearing changes

- **WHEN** the build agent attempts to create only `openspec/changes/<slug>/proposal.md` without a `specs/` sub-folder for a change that adds requirements
- **THEN** `validate()` surfaces `AdapterViolation` with `severity='error'`, `message='Dentour convention places spec deltas under specs/<capability>/spec.md inside the change folder. proposal.md alone is not a valid delta payload for a requirement-bearing change.'`

### REQ-DCA-009 — `validate()` invokes `pnpm openspec:validate --strict`

The adapter's `validate()` method shells out to:

```bash
cd <project.worktreeRoot> && pnpm openspec:validate --strict
```

Per [`dentour-topology.md § 1`](./dentour-topology.md), `openspec:validate --strict` invokes `openspec validate --all --strict` via Dentour's root `package.json`. This is Dentour's local validation step (not in CI — see § 5 for the defense-in-depth posture).

#### Scenario: validate passes

- **GIVEN** a change folder with valid statement-heading deltas
- **WHEN** the adapter runs `validate()` post-build
- **THEN** `pnpm openspec:validate --strict` exits 0
- **AND** the adapter returns an empty `AdapterViolation[]`.

#### Scenario: validate fails

- **GIVEN** a delta with a malformed Scenario block (missing `**THEN**`)
- **WHEN** the adapter runs `validate()`
- **THEN** `pnpm openspec:validate --strict` exits non-zero
- **AND** the adapter returns one or more `AdapterViolation` objects with `severity='error'` and stderr passed through verbatim.

#### Scenario: validate not in Dentour CI — defense-in-depth only

- **NOTE** `pnpm openspec:validate` is **not** run by any Dentour CI workflow at the anchored commit (per [`dentour-topology.md § 6`](./dentour-topology.md) flag 4). The adapter's `validate()` step is the only automated gate the studio controls.
- **THEREFORE** Forge MUST treat any `severity='error'` violation as a build-blocker (`BuildBlockedError`).

### REQ-DCA-010 — Commit message format (commitlint + Changesets, the key Dentour difference)

**This is the structurally unique requirement for Dentour vs Vivod.** Dentour uses **both** commitlint (enforced at severity 2 on scope-enum) and Changesets (for versioned releases). The two systems have different audiences and different failure modes.

Per [`dentour-topology.md § 1`](./dentour-topology.md):

| Rule | commitlint key | Severity | Constraint |
|---|---|---|---|
| Type allow-list | `type-enum` | 2 (error) | `feat \| fix \| docs \| style \| refactor \| perf \| test \| build \| ci \| chore \| revert` |
| Scope allow-list | `scope-enum` | **2 (error)** | 30 allowed values (apps: `patient-web`, `clinic-portal`, `admin`, `api`, `docs`, `e2e-tests`, `mcp-server`, `ai-team`; packages: `ui`, `auth`, `database`, `api-client`, `types`, `routes`, `env`, `config`, `features`, `i18n`, `maps`, `messaging`, `analytics`, `observability`, `utils`, `icons`, `design-tokens`, `cli`, `query-keys`; cross-cutting: `workspace`, `tooling`, `deps`) |
| Subject case | `subject-case` | 2 (error) | Subject MUST be lowercase, imperative |
| Subject trailing period | `subject-full-stop` | 2 (error) | No trailing period |
| Header length | `header-max-length` | 2 (error) | Header ≤ 100 chars |

**Critical difference vs Vivod:** Dentour's `scope-enum` is severity **2** (hard error, not warn-but-pass). Any commit scoped outside the 30-value enum will be **rejected** by commitlint. `(onboard-dentour)` is not in the enum and would hard-fail. This is a hard constraint that the adapter must resolve at commit time, not warn about.

**Adapter resolution:** The adapter selects scope from the 30-value enum, mapping by primary affected surface:

- Spec-only change touching a capability that maps to a known package → use that package scope (e.g., `auth`, `messaging`, `ui`).
- Cross-cutting spec changes with no single clear scope → use `workspace` (always valid per the enum).
- Changes to `openspec/changes/` content that don't map to a specific package → use `docs` (type `docs`, scope `workspace`).
- **No studio-side scope is registered upstream.** Instead the adapter routes all OpenSpec-only changes through `docs(workspace):` which satisfies all four rules without requiring a pre-engagement PR to Dentour.

**Changesets interaction — the second Dentour-specific concern:**

Per [`dentour-topology.md § 6`](./dentour-topology.md), `version-check.yml` runs on every PR and asserts that PRs touching publishable packages include a changeset file (`.changeset/*.md`). The convention adapter must decide:

1. **Spec-only changes** (changes only under `openspec/`, touching no `packages/*/src/` or `apps/*/src/` files) are **exempt from Changesets**. The `version-check.yml` workflow is path-filtered to publishable packages; a PR that touches only `openspec/changes/<slug>/specs/<cap>/spec.md` does not trigger the changeset requirement.
2. **Code-bearing changes** (changes that touch `packages/<pkg>/src/` in addition to the spec delta) MUST include a changeset. The adapter's `requiresChangeset(change, ctx)` method returns `true` when the change adds, removes, or modifies any `packages/<pkg>/src/` file. When `true`, the adapter scaffolds a `.changeset/<slug>-<hash>.md` stub with a `patch` bump and the Conventional Commit subject as the changeset summary.

#### Scenario: spec-only commit — no changeset needed

- **GIVEN** a change touching only `openspec/changes/<slug>/specs/auth/spec.md`
- **WHEN** the adapter evaluates `requiresChangeset(change, ctx)`
- **THEN** it returns `false`.
- **AND** the adapter does NOT generate a `.changeset/` file.
- **AND** the emitted commit message is `docs(workspace): add <spec summary>` (no trailing period, lowercase, ≤ 100 chars).

#### Scenario: code-bearing change — changeset required

- **GIVEN** a change touching both `openspec/changes/<slug>/specs/messaging/spec.md` AND `packages/messaging/src/resend.ts`
- **WHEN** the adapter evaluates `requiresChangeset(change, ctx)`
- **THEN** it returns `true`.
- **AND** the adapter scaffolds `.changeset/<slug>-<hash>.md` with `@dentour/messaging` as the affected package, bump type `patch`, and the commit subject as the summary.
- **AND** the emitted commit message is `feat(messaging): <subject>` (type matches change intent, scope matches the affected package).

#### Scenario: scope outside enum — adapter blocks, does not warn

- **WHEN** the build agent's type/scope assembly would produce a scope not in the 30-value enum
- **THEN** the adapter surfaces `AdapterViolation` with `severity='error'`, `message='Scope "<scope>" is not in Dentour's commitlint scope-enum (30 values, severity 2 — hard error). Remapping to "workspace". If the primary affected surface is a specific Dentour package, set the scope explicitly.'`
- **AND** the adapter substitutes `workspace` as the fallback scope.
- **AND** the commit is not blocked by this substitution (the fallback satisfies the enum).

#### Scenario: commit subject normalisation

- **GIVEN** a build agent producing subject `Add REQ for magic-link resend.`
- **WHEN** the adapter assembles the commit
- **THEN** the subject is lowercased to `add REQ for magic-link resend`, trailing period stripped → `add REQ for magic-link resend`, producing `docs(workspace): add REQ for magic-link resend`.

#### Scenario: header truncation

- **GIVEN** a build agent producing a subject that would push the header past 100 chars
- **WHEN** the adapter assembles the commit
- **THEN** the subject is truncated at character `100 - len("<type>(<scope>): ") - 1` with trailing `…`.

### REQ-DCA-011 — Branch alignment (Hybrid mode + Changesets)

Per [`dentour-topology.md § 1`](./dentour-topology.md) and `dentour.yaml § engagement.handoff`:

- Dentour's working branch shape: `feat/* | fix/* | chore/* | docs/* → develop → main`. `develop` is the integration branch; `main` triggers the Changesets release flow.
- Studio bot branches MUST live under `studio/<slug>` per the engagement charter § 9.2 — never inside `feat/*`, never targeting `main` directly.
- **Hybrid mode** (the proposed default per [`onboard-dentour` design.md § Engagement mode default](../../changes/onboard-dentour/design.md)): the adapter's branch shape is `studio/<change-slug>`; the PR targets **`develop`** (not `main`) — because `develop → main` is Dentour's integration gating path. Studio PRs go into `develop`; Dentour does the `develop → main` merge on their cadence, triggering the Changesets release.
- **This is different from the Vivod adapter**, where PRs target `main`. Dentour's branching model interposes `develop` as the integration branch.

#### Scenario: scaffold branch in Hybrid mode

- **GIVEN** `engagement.mode: hybrid`
- **WHEN** the adapter requests a branch name for change-slug `add-auth-otp-resend`
- **THEN** it returns `studio/add-auth-otp-resend`, PR target `develop`.

#### Scenario: refuse main push in Hybrid mode

- **WHEN** Forge attempts to open a PR targeting `main` with `engagement.mode: hybrid`
- **THEN** the adapter returns `AdapterViolation` with `severity='error'`, `message='Hybrid mode: PRs must target develop (Dentour's integration branch), not main. main triggers the Changesets release flow and must never be the direct studio PR target.'`

#### Scenario: Changesets release flow isolation

- **GIVEN** the studio bot opens a PR `studio/add-auth-otp-resend` targeting `develop`
- **WHEN** Dentour merges `develop → main` and `version-and-release.yml` runs
- **THEN** the adapter's optional `.changeset/` file (if present) is consumed by the Changesets flow without manual intervention.
- **AND** if no `.changeset/` file is present (spec-only PR), the release workflow emits no version bump for this PR.

### REQ-DCA-012 — Registry mirror methods are no-ops

Dentour has no cross-cutting registries equivalent to Unbind's `decisions.md` / `dependencies.md` / `maturity.md` / `open-questions.md`. The adapter's `mirrorDecision`, `mirrorDependency`, `mirrorMaturity`, `mirrorOpenQuestion`, and `nextDecisionId` methods are intentionally undefined. Registry-mirror MCP tools return documented no-ops when called against a Dentour project.

### REQ-DCA-013 — Skeleton-aware delta generation

When the build agent generates a delta for a skeleton capability (e.g., `dental-chart`, `clinic-portal`, `patient-account`, `admin`, `messaging`), the adapter's `validate()` step surfaces an additional warning:

> `'openspec/specs/<capability>/spec.md is a Phase-0 skeleton. The studio's delta will be the first substantive content for this capability. Confirm with Dentour at kickoff that filling skeleton specs is in scope before committing.'`

This is a warning (not an error); the build agent may proceed after surfacing it. The intent is to prevent an unintentional first-authorship of Dentour's own undocumented capabilities.

#### Scenario: delta on skeleton capability — warning surfaces

- **GIVEN** a change touching `openspec/changes/<slug>/specs/dental-chart/spec.md`
- **WHEN** `validate()` runs
- **THEN** one `AdapterViolation` with `severity='warning'` and the skeleton-first-authorship message above is included in the result.
- **AND** `validate()` does not return a `BuildBlockedError`.

### REQ-DCA-014 — `auth-portal` blacklist

The adapter MUST NOT generate deltas for or targeting `apps/auth-portal/`. Per [`dentour-topology.md § 2.9`](./dentour-topology.md), `auth-portal` is vestigial — it contains only a `tsconfig.tsbuildinfo` file with no `package.json`. Any build-agent path referring to `auth-portal` is surfaced as an `AdapterViolation` with `severity='error'`.

#### Scenario: detect auth-portal reference

- **WHEN** the build agent emits a code pointer to `apps/auth-portal/`
- **THEN** the adapter surfaces `AdapterViolation` with `severity='error'`, `message='apps/auth-portal/ is a vestigial folder with no package.json (see dentour-topology.md § 2.9). Do not generate deltas or code pointers targeting this path.'`

### REQ-DCA-015 — Prompt augmentation

The adapter's `promptAugmentation()` returns a single-paragraph reminder appended to the build agent's system prompt. Suggested wording (final wording locked during implementation):

> This corpus uses Dentour's statement-style formal-spec dialect. Capability source-of-truth files are `openspec/specs/<capability>/spec.md` (no YAML frontmatter — do NOT add a frontmatter block). Requirements use `### Requirement: <statement>` headings (full sentence, no REQ-XXX-NNN IDs). Scenarios use `#### Scenario: <name>` followed by `- **WHEN**` / `- **THEN**` bullets (optional `- **GIVEN**` preamble). Implementation pointers go in `## Implementation map` (do NOT invent `## Cross-References` or other section names). Change folders use `openspec/changes/<slug>/specs/<capability>/spec.md` for delta payloads. Validation runs via `pnpm openspec:validate --strict` (local-only — NOT in Dentour CI; treat all errors as build-blockers). Commit messages follow Dentour's commitlint config: scope-enum is severity-2 (hard error, 30 values); fallback to `workspace` scope for spec-only changes. Spec-only PRs do NOT need a Changesets file; code-bearing PRs do — the adapter scaffolds it. Branch shape is `studio/<change-slug>` in Hybrid mode; PRs target `develop` (NOT `main`). Do NOT generate deltas for `apps/auth-portal/` (vestigial). Registry mirror tools are no-ops under this convention.

---

## 3. Acceptance Criteria

| REQ ID | Criterion | Pass condition |
|---|---|---|
| REQ-DCA-001 | Slug + name registered | `resolveAdapter('dentour')` returns `{ slug: 'dentour', name: 'Dentour (statement-style formal-spec + commitlint + Changesets)' }` |
| REQ-DCA-002 | Source-of-truth detection | `spec.md` loaded directly; skeleton specs load without error and emit zero requirements with `isSkeleton=true` flag |
| REQ-DCA-003 | No-frontmatter posture | `capabilityFrontmatter()` returns `''`; spurious frontmatter detected at parse time with warning |
| REQ-DCA-004 | Statement-style heading parser | Parser emits Requirement tokens for `### Requirement: <statement>` headings; skeleton placeholder emits `isSkeleton=true`; stripped-colon variant surfaced as warning |
| REQ-DCA-005 | Scenario tokeniser | Parser emits Scenario tokens with `name` / `when` / `then` / optional `given`; multi-line clauses collapse to single strings |
| REQ-DCA-006 | `## Implementation map` | Pointers append under existing section; new specs scaffold placeholder; absence in stable spec surfaces warning |
| REQ-DCA-007 | Direct edits to `specs/` | `allowsDirectCurrentEdits()` returns `true` |
| REQ-DCA-008 | Sub-folder change layout | New change folders include `specs/<capability>/spec.md`; flat-only requirement-bearing changes error |
| REQ-DCA-009 | `pnpm openspec:validate --strict` | `validate()` shells out to Dentour's validate command, surfaces stderr verbatim on failure, returns empty list on success; errors are build-blockers |
| REQ-DCA-010 | Commit format — commitlint + Changesets | scope-enum hard-error resolved via `workspace` fallback; spec-only PRs emit no changeset; code-bearing PRs scaffold `.changeset/*.md`; subject lowercased + period stripped + header ≤ 100 chars |
| REQ-DCA-011 | Branch shape + PR target | Hybrid-mode branches under `studio/<change-slug>`; PR target is `develop` (NOT `main`); attempt to target `main` returns error |
| REQ-DCA-012 | No-op registry mirrors | All four `mirror*` methods + `nextDecisionId` are intentionally undefined |
| REQ-DCA-013 | Skeleton-aware delta warning | Delta against a Phase-0 skeleton capability surfaces a first-authorship warning (non-blocking) |
| REQ-DCA-014 | `auth-portal` blacklist | Any code pointer to `apps/auth-portal/` surfaces a blocking error |
| REQ-DCA-015 | Prompt augmentation | `promptAugmentation()` emits the paragraph documented in § 2; included in system prompt for Dentour projects only |

---

## 4. Parser shape

A focused state machine, modelled on the existing [`packages/forge/src/conventions/diff-grammar.ts`](../../../packages/forge/src/conventions/diff-grammar.ts) parser shape but for spec-source-of-truth files.

### 4.1 Token stream

The parser consumes a Dentour `spec.md` line-by-line and emits the following token kinds:

```ts
type DentourSpecToken =
  | { kind: 'h1'; text: string }                                                        // # <title> — rare; some stable specs carry it
  | { kind: 'h2-purpose'; raw: string }                                                 // ## Purpose section body
  | { kind: 'h2-implementation-map'; raw: string }                                      // ## Implementation map body
  | { kind: 'h2-known-gaps'; name: string; raw: string }                               // ## Known gaps / Acknowledged risks (or ## Status for skeletons)
  | { kind: 'h2-section'; name: string; raw: string }                                   // any other ## section (preserved verbatim)
  | { kind: 'requirement'; statement: string; isSkeleton: boolean }                     // ### Requirement: <statement>
  | { kind: 'scenario'; parentStmt: string; name: string; given: string|null; when: string; then: string }
  | { kind: 'prose'; raw: string };                                                      // anything else; preserved verbatim on round-trip
```

Note the absence of:
- `frontmatter` token (Dentour has none)
- `acceptance-table` token (Dentour has none)
- `cross-references-table` token (Dentour uses `## Implementation map`, not `## Cross-References`)

This reduces the token set compared to the Vivod adapter.

### 4.2 State machine

```
START
  → h1? → BODY
BODY
  → h2-purpose → PURPOSE_BODY → BODY
  → h2-implementation-map → IMPL_MAP_BODY → BODY
  → h2-known-gaps → GAPS_BODY → BODY
  → requirement → REQUIREMENT_BODY (scenarios accumulate under current requirement) → BODY
  → h2-section → SECTION_BODY → BODY
  → eof → END
```

State transitions are deterministic; the parser is a single forward pass with no backtracking. Round-trip writes the token stream back in insertion order, with a per-token `raw` cache for verbatim preservation.

### 4.3 Implementation cost estimate

- `dentour-spec-parser.ts` (token stream + state machine + round-trip writer): **~200 LOC** (comparable to the unbind diff-grammar at ~200 LOC; slightly simpler because no frontmatter or acceptance tables).
- `dentour-spec-parser.test.ts` (golden fixtures: at minimum `auth/spec.md`, `multi-currency/spec.md`, `quote-workflow/spec.md` for stable specs; `dental-chart/spec.md` for skeleton; `clinic-onboarding/spec.md` for the GIVEN-preamble Scenario variant): **~350 LOC** of fixture-driven assertions.

---

## 5. Validation step — `pnpm openspec:validate --strict` invocation

### 5.1 Default invocation

The adapter's `validate(changePath, ctx)` method shells out to:

```bash
cd <project.worktreeRoot> && pnpm openspec:validate --strict
```

…which Dentour's root `package.json` resolves to `openspec validate --all --strict`. Unlike Vivod's filename-bridge script, Dentour's validate command runs directly against `openspec/specs/*/spec.md` without any symlink dance. The adapter SHOULD NOT replicate the validate logic in TypeScript; it shells out to Dentour's canonical entry point.

### 5.2 Failure handling

- Exit 0 → `validate()` returns `[]` (empty violations).
- Exit non-zero → `validate()` returns one or more `AdapterViolation` objects:
  - `severity: 'error'`
  - `path: <changePath>`
  - `message: <stderr captured verbatim>`
  - `hint: 'Run `pnpm openspec:validate --strict` locally inside the Dentour worktree to reproduce.'`

### 5.3 Why not run `pnpm openspec:list` as a pre-check

[`dentour-topology.md § 7.3`](./dentour-topology.md) notes that `pnpm openspec:list` is the intended sanity check before emitting a delta. The adapter MAY run `openspec:list` before `openspec:validate --strict` as a cheap pre-flight to confirm the capability is registered; however, this is optional and deferred to the implementation owner.

### 5.4 Defense-in-depth posture

`pnpm openspec:validate` is **not** part of Dentour's CI pipeline at the anchored commit (per [`dentour-topology.md § 6`](./dentour-topology.md), flag 4). The adapter's `validate()` step is the only automated gate the studio controls. Wiring `pnpm openspec:validate --strict` into Dentour's `ci.yml` could be the studio's first suggested contribution at kickoff — a low-risk CI enhancement that benefits both parties.

---

## 6. Commit message format — Changesets + commitlint interaction (the key Dentour difference)

This section expands § REQ-DCA-010 with worked examples and the decision tree for changeset generation.

### 6.1 Rule table (verbatim from dentour-topology.md)

| Rule | commitlint key | Severity | Constraint |
|---|---|---|---|
| Type allow-list | `type-enum` | 2 (error) | `feat \| fix \| docs \| style \| refactor \| perf \| test \| build \| ci \| chore \| revert` |
| Scope allow-list | `scope-enum` | **2 (error)** | 30 values (see REQ-DCA-010 table) — **severity 2 is the critical Dentour difference vs Vivod's severity 1** |
| Subject case | `subject-case` | 2 (error) | lowercase, imperative |
| Subject period | `subject-full-stop` | 2 (error) | no trailing period |
| Header length | `header-max-length` | 2 (error) | ≤ 100 chars |

### 6.2 Changeset decision tree

```
Is the change code-bearing?
├── YES (any package/app src/ file touched)
│   ├── Which package(s) are affected?
│   │   └── Map to `@dentour/<pkg>` names from the 20-package inventory
│   ├── What bump type? (patch default; minor for new capabilities; major for breaking changes)
│   └── Scaffold .changeset/<slug>-<hash>.md
│       content: ---\n"@dentour/<pkg>": patch\n---\n<commit subject>\n
└── NO (spec-only, openspec/ paths only)
    └── No changeset scaffolded; PR passes version-check.yml (path-filtered, skips openspec/)
```

### 6.3 Scope mapping table

| Primary affected surface | Recommended scope | Type |
|---|---|---|
| `openspec/specs/<any>/` only | `workspace` | `docs` |
| `packages/auth/` | `auth` | `feat`/`fix`/`docs` |
| `packages/messaging/` | `messaging` | `feat`/`fix`/`docs` |
| `packages/ui/` | `ui` | `feat`/`fix`/`style` |
| `packages/database/` | `database` | `feat`/`fix` |
| `packages/api-client/` | `api-client` | `feat`/`chore` |
| `packages/types/` | `types` | `feat`/`fix` |
| `packages/routes/` | `routes` | `feat`/`fix` |
| `packages/env/` | `env` | `fix`/`chore` |
| `packages/config/` | `config` | `feat`/`fix` |
| `packages/features/patient-onboarding/` | `features` | `feat`/`fix` |
| `packages/features/clinic-onboarding/` | `features` | `feat`/`fix` |
| `packages/i18n/` | `i18n` | `feat`/`fix` |
| `packages/maps/` | `maps` | `feat`/`fix` |
| `packages/analytics/` | `analytics` | `feat`/`fix` |
| `packages/observability/` | `observability` | `feat`/`fix` |
| `apps/patient-web/` | `patient-web` | `feat`/`fix`/`style` |
| `apps/clinic-portal/` | `clinic-portal` | `feat`/`fix` |
| `apps/admin/` | `admin` | `feat`/`fix` |
| `apps/api/` | `api` | `feat`/`fix` |
| `apps/docs/` | `docs` | `docs` |
| `tooling/` | `tooling` | `chore`/`build` |
| deps updates | `deps` | `chore` |
| multi-surface / root config | `workspace` | `chore` |

**Nothing maps to `openspec-only` or `onboard-dentour`** — those are not in the enum and would hard-fail commitlint. Always route spec-only changes to `docs(workspace):`.

### 6.4 Why Vivod and Dentour differ here

Vivod uses `scope-enum` at severity **1** (warn-but-pass). The adapter emits the scope it can best defend and accepts the warning. Dentour uses severity **2** (hard error). The adapter MUST NOT emit an out-of-enum scope — it must either find a valid mapping or substitute `workspace`. This constraint changes the adapter's commit assembler from "best-effort + warn" to "enum-guaranteed + fallback".

### 6.5 Worked examples

| Change | Changeset? | Emitted commit |
|---|---|---|
| Add `### Requirement:` row to `auth/spec.md` | No | `docs(workspace): add magic-link session requirement to auth spec` |
| Wire `pnpm resend` hook in `packages/messaging/` + spec delta | Yes (`@dentour/messaging`: patch) | `feat(messaging): add transactional email resend capability to messaging spec` |
| Fix typo in `multi-currency/spec.md` | No | `docs(workspace): fix typo in multi-currency spec` |
| Propose skeleton fill-in for `dental-chart/spec.md` | No | `docs(workspace): add initial dental-chart spec requirements (Phase-0 fill-in)` |
| New patient onboarding path in XState + spec | Yes (`@dentour/features`: minor) | `feat(features): add path-B patient onboarding flow with spec delta` |

---

## 7. Complexity estimate

**Target: 2 weeks of focused work.**
**Honest ceiling: 3 weeks** if the commitlint scope-enum validation interaction proves more brittle than expected in the test harness, or if the Changesets scaffolding decision tree needs more edge-case coverage than anticipated.

### 7.1 Drivers (vs Vivod adapter as the comparable baseline)

| Driver | Vivod | Dentour | Delta |
|---|---|---|---|
| Source-of-truth file shape | `specs/<cap>/index.md` with Fumadocs frontmatter | `specs/<cap>/spec.md` with no frontmatter | simpler (no frontmatter) |
| Heading style | TWO REQ heading variants | ONE statement-heading variant | simpler (single parser branch) |
| Structured payload | Scenarios + Acceptance Criteria tables | Scenarios only (no AC tables) | slightly simpler |
| Change-folder shape | `specs/<cap>/spec.md` sub-folder | same | comparable |
| Validation | shell-out to filename-bridge script | direct `pnpm openspec:validate --strict` | slightly simpler (no symlink dance) |
| Commit format — scope | severity 1 (warn) | severity **2** (hard error) — must guarantee enum membership | harder |
| Changeset scaffolding | not applicable (Vivod does not use Changesets) | required for code-bearing changes | net new surface |
| Branch / PR target | `studio/<slug>` → PR targets `main` | `studio/<slug>` → PR targets `develop` | minor difference; needs adapter constant |
| Skeleton detection | not applicable (all Vivod specs are stable) | 5 Phase-0 skeletons need warning surface | net new, lightweight |
| `auth-portal` blacklist | not applicable | vestigial-folder guard needed | net new, trivial |
| Registry mirror | no-ops (same as Dentour) | no-ops | comparable |

Net: Dentour's parser is **simpler** than Vivod's (no frontmatter, no AC tables, single heading variant), but the **commit assembler is harder** (severity-2 scope enum, changeset scaffolding). The two cross out roughly to comparable overall surface, skewed slightly simpler on the parser side and slightly harder on the commit side.

### 7.2 Estimated phasing for the implementation change

This is the **suggested** phasing for `add-forge-dentour-convention-adapter` (the shell opened in T6.2). It is not binding — the implementation change owns its own tasks.md.

| Phase | Surface | Effort estimate |
|---|---|---|
| 1 | Adapter scaffold + `slug` / `name` / `promptAugmentation` / no-op mirror methods | 0.5 day |
| 2 | Spec parser (`dentour-spec-parser.ts`) + golden-fixture round-trip tests | 3–4 days |
| 3 | `validate()` shell-out to `pnpm openspec:validate --strict` + stderr capture + `BuildBlockedError` integration | 1.5 days |
| 4 | Commit message assembler — scope lookup table + lowercase + period strip + header truncation + severity-2 guarantee | 1.5 days |
| 5 | Changeset scaffolding decision tree + `.changeset/` file generation + `requiresChangeset()` logic | 1.5 days |
| 6 | Branch shape helper (Hybrid mode; PR target `develop`) + `auth-portal` blacklist + skeleton-aware warning | 1 day |
| 7 | `forge spec --fix --project dentour` sub-folder change layout scaffold | 1 day |
| 8 | End-to-end smoke test against real Dentour fixtures (vendored from anchored commit) | 1 day |
| 9 | Documentation: amend `openspec/current/forge/entities.md` with the Dentour adapter entry | 0.5 day |
| | **Total** | **11.5 working days ≈ 2.3 weeks** |

### 7.3 Escalation triggers

If during implementation any of the following surface, pause and either escalate to a standalone proposal or amend `add-forge-dentour-convention-adapter`:

1. **Changesets schema changes.** If Dentour upgrades `@changesets/cli` past `^2.29.7` and the changeset file format changes, the adapter's scaffolding output needs re-derivation. Re-anchor the implementation against the new commit SHA before continuing.
2. **`commitlint.config.js` scope enum grows.** The 30-value enum is captured at the anchored commit; new scopes added by Dentour must be absorbed. The adapter's scope-lookup table is a constant; flag it for review at each major Dentour commit anchor update.
3. **`version-check.yml` path filter changes.** If Dentour expands the path filter to include `openspec/`, spec-only PRs would begin requiring changesets. Update `requiresChangeset()` logic accordingly.
4. **Adapter exceeds 600 LOC or test count exceeds 70 cases.** Either is an indicator the dialect is more complex than this design captured. Pause and re-design.

The 2-week target is honest. The 3-week ceiling is a function of Changesets + commitlint interaction correctness — not parser complexity (the parser is simpler than Vivod's). Slipping past 3 weeks should trigger a hard escalation.

---

## 8. Explicit non-goal — do NOT invoke Dentour's own automation skills

The adapter writes deltas to disk in Dentour's shape. Dentour's developers run their own propose / apply / archive flow on their cadence. The adapter MUST NOT shell out to or invoke any of Dentour's automation skills:

- `dentour-openspec-workflow` (per `openspec/config.yaml § rules` and `.claude/skills/dentour-openspec-workflow/SKILL.md` at the anchored commit)
- `/opsx:apply` (the Dentour-side apply skill)
- `/opsx:archive` (the Dentour-side archive skill)

(Inventory anchored at `2e96f677b7557b59530fbae87387b28e088defe3`. Verify currency at implementation time.)

Reasons:

1. **Boundary integrity.** The studio's adapter writes deltas; Dentour's skills apply / verify / archive them. Coupling the adapter to Dentour's skill versioning would bind the studio's release cadence to Dentour's internal tooling.
2. **Dual-execution risk.** If both the adapter and Dentour's `pnpm openspec:validate --strict` run via different entry points, the same failure may surface twice with different framing.
3. **Hybrid-mode contract.** In Hybrid mode, Dentour-side merge happens via PR review or `/opsx:apply` run by Dentour's developers on their cadence — the studio bot's lane stops at "deltas on a `studio/<slug>` branch targeting `develop`." Calling Dentour's apply skill would jump that lane.

The single allowed shell-out is `pnpm openspec:validate --strict` (REQ-DCA-009) — the OpenSpec CLI itself, not a Dentour-authored skill.

---

## 9. Cross-references

- Adapter framework (parent): [`openspec/archive/2026-05-08-add-forge-convention-adapter/`](../../archive/2026-05-08-add-forge-convention-adapter/) — interface, registry, build-agent integration shape.
- Studio adapter (back-compat baseline): [`packages/forge/src/conventions/studio.ts`](../../../packages/forge/src/conventions/studio.ts).
- Unbind adapter (closest code analogue): [`packages/forge/src/conventions/unbind.ts`](../../../packages/forge/src/conventions/unbind.ts) + [`packages/forge/src/conventions/diff-grammar.ts`](../../../packages/forge/src/conventions/diff-grammar.ts).
- Vivod adapter (structural template for this document): [`openspec/_methodology/research/vivod-convention-adapter-design.md`](./vivod-convention-adapter-design.md).
- Companion research: [`dentour-topology.md`](./dentour-topology.md) (dialect grammar source, topology scan) + [`dentour-drift-audit.md`](./dentour-drift-audit.md) (Phase-0 skeleton inventory, capability table).
- Engagement profile: [`dentour.yaml`](../../_methodology/projects/dentour.yaml) (mode, branch shape, Changesets workflow, scope policy decisions deferred to kickoff).
- Implementation shell: [`openspec/changes/add-forge-dentour-convention-adapter/`](../../changes/add-forge-dentour-convention-adapter/) — opened in T6.2 of this change.
- Engagement charter §§ 4, 7, 9.2: [`openspec/_methodology/rapoport-studio-engagement.md`](../../_methodology/rapoport-studio-engagement.md) — mode taxonomy + branch convention + bot-scope ceilings the adapter must respect.
