---
title: Vivod Convention Adapter — Design Notes
description: Studio-side design for the Forge convention adapter that handles Vivod's REQ-numbered formal-delta OpenSpec dialect. Authored during Phase 6 of `onboard-vivod`. Implementation deferred to `add-forge-vivod-convention-adapter`.
---

# Vivod Convention Adapter — Design Notes

<!-- openspec-refs: allow-unresolved -->
<!-- This design references Vivod-internal openspec paths (`openspec/specs/<capability>/`, `openspec/changes/<slug>/specs/<cap>/spec.md`, `scripts/openspec-validate.sh`) inside `VIVOD-AI/vivod`, NOT this repo. They are not actionable refs from the studio side. Mirrors the same opt-out applied to vivod-topology.md and vivod-drift-audit.md. -->

> Studio-internal design notes for the Forge convention adapter that will land in `packages/forge/src/conventions/vivod.ts`. Authored during Phase 6 of [`onboard-vivod`](../../changes/onboard-vivod/) (T6.1). Companion to [`vivod-topology.md`](./vivod-topology.md) and [`vivod-drift-audit.md`](./vivod-drift-audit.md). Implementation tracked separately as [`add-forge-vivod-convention-adapter`](../../changes/add-forge-vivod-convention-adapter/) — opened in T6.2 as a proposal-only shell.

```yaml
design_date: 2026-05-12
vivod_commit_sha: 3f678eb88a7fc3ae97bfeae082f0b807e96a4734   # same anchor as topology / drift-audit
vivod_repo_url: https://github.com/VIVOD-AI/vivod
shareability: studio-internal-default
author: forge-subagent
companion_docs:
  - vivod-topology.md (Phase 1 — dialect grammar source)
  - vivod-drift-audit.md (Phase 2 — heading-style heterogeneity, dialect divergences)
adapter_framework: openspec/archive/2026-05-08-add-forge-convention-adapter/   # the parent that introduced OpenSpecConventionAdapter
target_implementation: openspec/changes/add-forge-vivod-convention-adapter/    # proposal-only shell opened in T6.2
target_complexity_estimate: 2-3 weeks of focused work (target 2; honest ceiling 3 — see § 7)
non_goal: do NOT invoke Vivod's own automation skills programmatically (see § 8)
```

---

## 0. Premise

Vivod ships its own OpenSpec corpus in a **formal-delta dialect** that the studio's prose convention does not match. The dialect is documented exhaustively in [`vivod-topology.md § 7`](./vivod-topology.md). The convention adapter framework introduced by [`add-forge-convention-adapter`](../../archive/2026-05-08-add-forge-convention-adapter/) (RAP-86, archived 2026-05-08) gives Forge a stable seam — `OpenSpecConventionAdapter` at [`packages/forge/src/conventions/adapter.ts`](../../../packages/forge/src/conventions/adapter.ts) — for slotting per-engagement convention rules without leaking project-slug conditionals into the build agent.

This document specifies the Vivod adapter that fills that seam for Vivod. It is **design-only**: no code is committed to `packages/forge/` as part of `onboard-vivod`. The implementation lives in a separate change folder and is run as a separate Linear issue once kickoff ratifies engagement mode (per [`onboard-vivod` design.md § Engagement mode default](../../changes/onboard-vivod/design.md)).

The design uses Vivod's own dialect format (`### REQ-VCA-NNN` requirements + Scenarios + Acceptance Criteria tables) so the doc is recognisable to Vivod reviewers if and when the studio decides to share it post-kickoff. Sharing is not assumed; the format choice is for cognitive parity with Vivod's authors, not as a promise of disclosure.

---

## 1. Vivod's spec dialect — what the adapter must read

### 1.1 Source-of-truth files

Per [`vivod-topology.md § 7.1`](./vivod-topology.md):

- Vivod has **no `openspec/current/` folder**. Capabilities live at `openspec/specs/<capability>/`.
- Each capability folder typically exposes `index.md` (Fumadocs MDX with frontmatter); a subset additionally exposes `spec.md` for the OpenSpec CLI.
- Where `spec.md` is missing, Vivod's `scripts/openspec-validate.sh` materialises it temporarily by symlinking `index.md → spec.md`, runs `npx openspec validate --all --strict`, then cleans up the symlinks. This is the **filename bridge** between Fumadocs (`index.md`) and the OpenSpec CLI (`spec.md`).

### 1.2 Frontmatter

Every `index.md` carries a Fumadocs frontmatter block (per [`vivod-topology.md § 7.4`](./vivod-topology.md)):

```markdown
---
title: <capability title>
description: <one-line summary>
---
```

The adapter MUST preserve this block when editing — both the keys and the YAML field order — because `docs.vivod.ai` renders directly from it.

### 1.3 Required structural sections

Per [`vivod-topology.md § 7.4`](./vivod-topology.md), every capability spec begins with:

- `## Purpose` — mandatory (the validate-bridge script's heuristic for "this is a spec file"). Must not be omitted.
- `## Requirements` (or equivalent — varies by spec; some use `## Architecture` + inline requirement headings) — the body containing `### REQ-<DOMAIN>-NNN` blocks.

The `## Implementation map` section name assumed in the original `onboard-vivod` design.md draft **does not exist canonically in Vivod's corpus**. The closest observed pattern is `## Cross-References` (link tables to sibling specs). The adapter MUST reuse the observed name; **inventing a section name would be a unilateral dialect change** and is forbidden.

### 1.4 Requirement headings — two variants

Per [`vivod-topology.md § 7.4`](./vivod-topology.md) and [`vivod-drift-audit.md § Capability table row 6`](./vivod-drift-audit.md):

| Variant | Used in | Example |
|---|---|---|
| `### REQ-<DOMAIN>-NNN` | most specs (auth, api, infra, facade, contracts, worker-app, design, query-layer, analytics, project-form, forge-console-ui, entities, …) | `### REQ-AUTH-013` |
| `### Requirement REQ-<DOMAIN>-NNN` | `agents/spec.md` only | `### Requirement REQ-AGT-014` |

The parser **must accept both prefixes**. A single-prefix parser would silently miss the entire agents capability (82 requirements). This is a hard parser invariant.

`<DOMAIN>` is a 2–4 character uppercase token (`AUTH`, `API`, `AGT`, `INF`, `FCD`, `CON`, `WKR`, `DSN`, `QL`, `PH`, `PF`, `P3`, `ENT`). The full registry is enumerated in [`vivod-topology.md § 7.2`](./vivod-topology.md). The adapter must not assume the registry is closed — Vivod can add new domains; the parser accepts any `[A-Z][A-Z0-9]{1,3}`.

`NNN` is a 1–3 digit positive integer (`001`, `47`, `082`). Numbering is per-capability and gap-tolerant (Vivod retires requirements without renumbering — verified by inspection of `auth/index.md` at the anchored commit).

### 1.5 Scenario blocks

Each requirement may carry zero or more Scenario blocks formatted as:

```markdown
#### Scenario: <name>

- **GIVEN** <optional preamble>
- **WHEN** <trigger>
- **THEN** <expected outcome>
```

Per [`vivod-topology.md § 7.4`](./vivod-topology.md):

- `**WHEN**` and `**THEN**` are required.
- `**GIVEN**` is optional preamble; not every Scenario carries one.
- Multi-line `**WHEN**` / `**THEN**` bodies are wrapped onto the next bullet under the same heading; the parser must collapse them into a single logical clause.

### 1.6 Acceptance Criteria tables

Optional per-spec, but when present, tables sit at the bottom of the spec body:

```markdown
## Acceptance Criteria

| REQ ID | Criterion | Pass condition |
|---|---|---|
| REQ-AUTH-001 | … | … |
| REQ-AUTH-002 | … | … |
```

Per [`vivod-topology.md § 7.4`](./vivod-topology.md):

- `auth/index.md` carries 47 rows.
- `infra/index.md` carries 24+ rows.
- Some specs (`design/index.md`, `worker-app/spec.md`) omit the table entirely.

The adapter **preserves** the table when editing but **does not require** its presence. Adding rows is allowed when the change introduces a new requirement; rewriting existing rows is gated by a Scenario-level edit.

### 1.7 Change-folder layout (formal-deltas)

Per [`vivod-topology.md § 7.4–7.5`](./vivod-topology.md) and confirmed by directory inspection of `openspec/changes/forge-console-ui/`, `worker-today-view/`, etc.:

- Active changes live at `openspec/changes/<slug>/` with the **opposite** of the studio's prose-flat layout: a sub-folder `specs/<capability>/spec.md` carrying the formal delta.
- `proposal.md`, `design.md`, `tasks.md` may also live at `openspec/changes/<slug>/` but the **delta payload** is the `specs/<cap>/spec.md` sub-folder.
- Archived changes live at `openspec/changes/archive/<YYYY-MM-DD>-<slug>/` (note: archive is a sub-folder of `changes/`, not a sibling — opposite of the studio's `openspec/archive/` placement).

The adapter writes deltas to this sub-folder shape; **never** writes to a flat `openspec/changes/<slug>/proposal.md`-only shape.

---

## 2. Adapter requirements (REQ-VCA-NNN)

The adapter conforms to the [`OpenSpecConventionAdapter`](../../../packages/forge/src/conventions/adapter.ts) interface introduced by RAP-86. Each requirement below maps to one or more methods on that interface.

### REQ-VCA-001 — Slug + name

The adapter exposes `slug = 'vivod'` and `name = 'Vivod (REQ-numbered formal-delta + Fumadocs frontmatter + filename-bridge validate)'`. Registered in [`packages/forge/src/conventions/index.ts`](../../../packages/forge/src/conventions/index.ts) alongside `studio` and `unbind`.

#### Scenario: registry resolution

- **WHEN** `resolveAdapter('vivod')` is called
- **THEN** the function returns the Vivod adapter instance with the slug `'vivod'` and the name above.

### REQ-VCA-002 — Source-of-truth file detection

The adapter recognises `openspec/specs/<capability>/index.md` as the canonical source-of-truth file. When `spec.md` is also present alongside `index.md`, the adapter treats `index.md` as authoritative (per Vivod's filename-bridge convention — `spec.md` is a build-time artefact, not the source).

#### Scenario: read capability

- **GIVEN** a capability folder `openspec/specs/auth/` containing both `index.md` and `spec.md`
- **WHEN** the adapter loads the spec for `auth`
- **THEN** it reads from `index.md` and ignores `spec.md`.

#### Scenario: read capability with no spec.md

- **GIVEN** a capability folder `openspec/specs/forge-console-ui/` containing only `index.md`
- **WHEN** the adapter loads the spec for `forge-console-ui`
- **THEN** it reads from `index.md` without surfacing a "missing spec.md" error.

### REQ-VCA-003 — Frontmatter preservation

The `capabilityFrontmatter()` method generates a Fumadocs-compatible frontmatter block when creating a new capability spec. When editing an existing spec, the adapter preserves the existing frontmatter byte-for-byte (key order, value formatting, surrounding whitespace).

#### Scenario: new capability frontmatter

- **WHEN** the build agent creates a new capability `payments` and calls `capabilityFrontmatter('payments', ctx)`
- **THEN** the adapter returns:
  ```
  ---
  title: Payments
  description: <auto-generated single-line summary or empty placeholder for the agent to fill>
  ---
  ```
- **AND** the build agent prepends the result to `openspec/specs/payments/index.md`.

#### Scenario: edit preserves frontmatter

- **GIVEN** an existing `openspec/specs/auth/index.md` with frontmatter `title: Auth\ndescription: Phone OTP / PIN / magic link`
- **WHEN** the build agent edits a Scenario block deep in the file
- **THEN** the frontmatter is unchanged (byte-for-byte) including key order and inline whitespace.

### REQ-VCA-004 — Heading-style heterogeneity (parser)

The parser accepts BOTH `### REQ-<DOMAIN>-NNN` and `### Requirement REQ-<DOMAIN>-NNN` heading prefixes. A single regex covers both:

```
/^### (?:Requirement )?REQ-([A-Z][A-Z0-9]{1,3})-(\d{1,3})\b/
```

#### Scenario: parse standard heading

- **GIVEN** a capability spec with heading `### REQ-AUTH-013 Admin subdomain rewrite`
- **WHEN** the parser tokenises the spec
- **THEN** it emits a Requirement token with `domain='AUTH'`, `number=13`, `title='Admin subdomain rewrite'`.

#### Scenario: parse agents-style heading

- **GIVEN** `agents/spec.md` with heading `### Requirement REQ-AGT-014 SCOPE photo analysis`
- **WHEN** the parser tokenises the spec
- **THEN** it emits a Requirement token with `domain='AGT'`, `number=14`, `title='SCOPE photo analysis'`.

#### Scenario: reject malformed heading

- **GIVEN** a heading `### REQ-AUTH-XYZ` (non-numeric NNN)
- **WHEN** the parser encounters it
- **THEN** the parser surfaces a structured `AdapterViolation` with `severity='warning'`, `message='Malformed REQ heading "REQ-AUTH-XYZ" — number segment must be 1–3 digits.'` and continues parsing the rest of the file.

### REQ-VCA-005 — Scenario block tokeniser

The parser recognises `#### Scenario: <name>` followed by `- **WHEN**` / `- **THEN**` (with optional `- **GIVEN**` preamble) bullets under each Requirement.

#### Scenario: parse minimal scenario

- **GIVEN** a Requirement followed by:
  ```
  #### Scenario: Manager logs in via OTP
  - **WHEN** the manager submits a valid OTP
  - **THEN** they are redirected to /platform/projects
  ```
- **WHEN** the parser tokenises
- **THEN** it emits a Scenario token attached to the parent Requirement with `name='Manager logs in via OTP'`, `when='the manager submits a valid OTP'`, `then='they are redirected to /platform/projects'`, `given=null`.

#### Scenario: parse scenario with GIVEN preamble

- **GIVEN** a Scenario block carrying `- **GIVEN** the worker has not yet completed onboarding`
- **WHEN** the parser tokenises
- **THEN** the emitted Scenario carries the preamble in the `given` field.

#### Scenario: parse multi-line clause

- **GIVEN** a `**THEN**` clause wrapped onto the next bullet:
  ```
  - **THEN** the system writes an audit row
    - including the worker_id, action='onboard', and ts
  ```
- **WHEN** the parser tokenises
- **THEN** the emitted `then` field is the collapsed single-line string `the system writes an audit row including the worker_id, action='onboard', and ts`.

### REQ-VCA-006 — Acceptance Criteria table preservation

The adapter parses `## Acceptance Criteria` tables when present, surfaces them as a structured `AcceptanceTable` node, and round-trips them losslessly when writing back. Tables are not required.

#### Scenario: round-trip preserves rows

- **GIVEN** `auth/index.md` with a 47-row Acceptance Criteria table
- **WHEN** the adapter parses the file and writes it back without edits
- **THEN** the byte-for-byte output is identical to the input (cell whitespace, column alignment, separator dashes preserved).

#### Scenario: append row for new requirement

- **WHEN** the build agent adds `REQ-AUTH-048` and the table exists
- **THEN** the adapter appends a new row with the new REQ ID; existing rows are untouched.

#### Scenario: missing table

- **GIVEN** `design/index.md` with no `## Acceptance Criteria` section
- **WHEN** the adapter parses
- **THEN** no error or warning is surfaced; the absence is allowed.

### REQ-VCA-007 — Cross-References section reuse

The adapter recognises `## Cross-References` (Vivod's observed section name) as the implementation-pointer section. When the build agent emits a code-pointer link, it is appended to `## Cross-References`, not to a fabricated `## Implementation map` section.

#### Scenario: append cross-reference

- **GIVEN** `contracts/spec.md` with an existing `## Cross-References` section
- **WHEN** the build agent surfaces a new pointer to `apps/web/src/actions/contracts.ts`
- **THEN** the adapter appends the link under the existing section.

#### Scenario: forbid invented section

- **WHEN** the build agent attempts to write a `## Implementation map` heading
- **THEN** the adapter's `validate()` step surfaces an error: `"Vivod convention uses ## Cross-References, not ## Implementation map. Move the new content under the existing section name."`

### REQ-VCA-008 — Direct edits to `specs/` (no diff-at-archive)

`allowsDirectCurrentEdits()` returns `true`. Vivod's convention applies edits directly inside the change folder's `specs/<cap>/spec.md` sub-folder, then merges to `openspec/specs/<cap>/index.md` via Vivod's own `pnpm openspec apply` flow. There is no diff-at-archive grammar (unlike Unbind).

Note: Vivod's "change folder writes to `specs/`" pattern differs from the studio's "change folder writes to `current/`" pattern only in the source-of-truth folder name. Mechanically both adapters report `allowsDirectCurrentEdits() = true`. The Forge build agent's existing direct-edit path works without modification.

### REQ-VCA-009 — Change-folder layout (formal-deltas sub-folder)

When the build agent creates a new change folder, the adapter scaffolds the sub-folder shape `openspec/changes/<slug>/specs/<capability>/spec.md` (NOT a flat `proposal.md`-only layout). The flat files (`proposal.md`, `design.md`, `tasks.md`) may also be created at `openspec/changes/<slug>/` if the change requires them — Vivod's own changes vary in whether they include all three.

#### Scenario: scaffold sub-folder delta

- **WHEN** `forge spec --fix --project vivod` is run for a change touching capability `auth`
- **THEN** the adapter creates `openspec/changes/<slug>/specs/auth/spec.md` carrying the delta in Vivod's REQ-numbered format.

#### Scenario: forbid flat-only layout

- **WHEN** the build agent attempts to create only `openspec/changes/<slug>/proposal.md` without a `specs/` sub-folder for a change that adds requirements
- **THEN** the adapter's `validate()` step surfaces an error: `"Vivod convention places spec deltas under specs/<capability>/spec.md inside the change folder. proposal.md alone is not a valid delta payload."`

### REQ-VCA-010 — `validate()` invokes `pnpm openspec validate` via the filename bridge

The adapter's `validate()` method invokes Vivod's filename-bridge script:

```
pnpm openspec validate
```

…which Vivod resolves to `scripts/openspec-validate.sh` (per Vivod's root `package.json`). The bridge symlinks `index.md → spec.md` for each spec dir matching the heuristic, runs `npx openspec validate --all --strict`, then cleans up.

The adapter SHOULD NOT replicate the filename-bridge logic in TypeScript; it shells out to Vivod's own script via `ctx.git`-rooted process spawn. This is the canonical Vivod entry point and re-implementing it would create drift.

#### Scenario: validate passes

- **GIVEN** a change folder with valid REQ-numbered deltas
- **WHEN** the adapter runs `validate()` post-build
- **THEN** the bridge script exits 0
- **AND** the adapter returns an empty `AdapterViolation[]`.

#### Scenario: validate fails

- **GIVEN** a change folder with a malformed Scenario block
- **WHEN** the adapter runs `validate()`
- **THEN** the bridge script exits non-zero with stderr describing the failure
- **AND** the adapter returns one or more `AdapterViolation` objects with `severity='error'` and the bridge's stderr passed through verbatim in the `message` field.

#### Scenario: validate not in Vivod CI — defense-in-depth only

- **NOTE** `pnpm openspec validate` is **not** run by any Vivod CI workflow at the anchored commit (per [`vivod-topology.md § 6`](./vivod-topology.md)). The adapter's `validate()` step is the **only** automated gate the studio side controls; Vivod-side merge does not enforce it.
- **THEREFORE** Forge MUST treat any `severity='error'` violation as a build-blocker (`BuildBlockedError`) and refuse to commit the change. The studio cannot rely on Vivod's CI catching the failure later.

### REQ-VCA-011 — Commit message format (commitlint)

The adapter emits commit messages conforming to Vivod's [`commitlint.config.cjs`](https://github.com/VIVOD-AI/vivod/blob/3f678eb/commitlint.config.cjs) constraints (per [`vivod-topology.md § 1`](./vivod-topology.md)):

| Rule | Severity | Constraint |
|---|---|---|
| `type-enum` | 2 (error) | One of `feat \| fix \| refactor \| chore \| perf \| docs \| test \| ci \| build \| style` |
| `scope-enum` | 1 (warn) | One of `web \| ui \| db \| domain \| auth \| agents \| config \| i18n \| facade-engine \| cloudflare \| api-gateway \| scope \| ledger \| clerk \| ci \| deps` |
| `subject-case` | 2 (error) | Subject MUST be lowercase |
| `header-max-length` | 2 (error) | Header (type + scope + subject) ≤ 100 chars |

Adapter behaviour:

- The adapter selects `type` from the change's primary intent: a new capability is `feat`, a documentation amendment is `docs`, etc. (mapped via the build agent's existing classification, not invented per-adapter).
- The adapter selects `scope` from the change folder's primary affected capability when it maps to the enum (`auth`, `api`, `web`, …). When the affected capability has no enum entry (e.g. a `forge` scope, or `guard`/`clerk` despite both being live workers — see [`vivod-topology.md § 1`](./vivod-topology.md)), the adapter emits the most accurate scope and **accepts the warn-level violation**. The kickoff conversation decides whether to register a studio-side scope upstream in Vivod's `commitlint.config.cjs` or accept the warning.
- The adapter lowercases the subject before commit. The build agent may emit any case; the adapter normalises.
- The adapter truncates the subject (with an ellipsis at character 97) if the assembled header would exceed 100 chars. This is a fallback; the build agent should already keep subjects short.

#### Scenario: well-formed commit

- **WHEN** the build agent finalises a change touching `auth/index.md` adding `REQ-AUTH-048`
- **THEN** the adapter emits a commit message matching `feat(auth): add REQ-AUTH-048 for x-vivod-subdomain dev fallback` (or similar), with all four constraints satisfied.

#### Scenario: scope outside enum

- **WHEN** the build agent finalises a change touching `apps/workers/clerk/`
- **THEN** the adapter emits `feat(clerk): …` and accepts the warn-level violation. The commit lands; commitlint surfaces a warning at pre-commit.

#### Scenario: subject too long

- **GIVEN** a build agent producing a 120-char subject
- **WHEN** the adapter assembles the commit
- **THEN** the subject is truncated to fit the 100-char header budget with a trailing `…`.

### REQ-VCA-012 — Branch alignment (Hybrid mode)

The adapter is consulted by the build agent's branch-naming step. Branch shape per [`vivod-topology.md § 1` + `vivod.yaml § engagement.handoff`](../../_methodology/projects/vivod.yaml):

- Vivod's working branch shape is `feature/VVD-<n>-slug` → `staging` → `main` (PR-only, no direct pushes; `main` auto-deploys to Railway).
- Studio bot branches MUST live under `studio/<slug>` per the engagement charter § 9.2 — never inside the `feature/*` namespace, never targeting `main` directly.
- Hybrid mode (the proposed default per [`onboard-vivod` design.md § Engagement mode default](../../changes/onboard-vivod/design.md)): the adapter's branch shape is `studio/<change-slug>` for the bot's PR; the PR targets `main` for *Vivod* to merge to `staging` → `main` themselves on their cadence. The studio bot never pushes to `main` directly even in Full mode.
- Full mode (post-kickoff escalation only): same `studio/<change-slug>` branch shape; bot may push to `main` *if* kickoff ratifies a `Contents: write` permission on the GitHub App. Default-off.

The adapter SHOULD NOT mint a `feature/VVD-<n>-…` branch. The `VVD-` prefix is Vivod's Linear; the studio's Linear prefix is whatever `engagement.linear.team_prefix` ratifies at kickoff (`VIV-` is the default candidate per [`vivod.yaml § engagement.linear`](../../_methodology/projects/vivod.yaml)). Branch names use the studio-side `<change-slug>` (e.g. `studio/add-auth-otp-resend`), not the Linear ID.

#### Scenario: scaffold branch in Hybrid mode

- **GIVEN** `engagement.mode: hybrid` and `engagement.handoff.code_branch_shape: forge/<change-slug>/<task-slug>`
- **WHEN** the adapter requests a branch name for change-slug `add-auth-otp-resend`, task-slug `wire-resend-action`
- **THEN** it returns `studio/add-auth-otp-resend` (the change-slug-only short form for the parent PR; the task-slug per-step branch shape can layer beneath it as `forge/add-auth-otp-resend/wire-resend-action` per the studio's `engagement.handoff.code_branch_shape`).

#### Scenario: refuse main push in Hybrid mode

- **WHEN** Forge attempts to open a PR targeting `main` with `engagement.mode: hybrid`
- **THEN** the adapter returns a structured error: `"Hybrid mode: PRs target studio/<slug>, not main. Vivod merges studio/<slug> → staging → main themselves."`

### REQ-VCA-013 — Registry mirror methods are no-ops

Vivod has no cross-cutting registries equivalent to Unbind's `decisions.md` / `dependencies.md` / `maturity.md` / `open-questions.md`. The adapter's `mirrorDecision`, `mirrorDependency`, `mirrorMaturity`, `mirrorOpenQuestion`, and `nextDecisionId` methods are intentionally undefined (matching the studio adapter's pattern). The MCP tools (`mcp__forge__forge_mirror_decision`, etc.) return documented no-ops when called against a Vivod project, so the build agent's existing prompt augmentation pattern continues to work.

### REQ-VCA-014 — Prompt augmentation

The adapter's `promptAugmentation()` returns a single-paragraph reminder appended to the build agent's system prompt. Suggested wording (final wording locked during implementation):

> This corpus uses Vivod's REQ-numbered formal-delta dialect. Capability source-of-truth files are `openspec/specs/<capability>/index.md` (Fumadocs frontmatter required, preserve byte-for-byte). Requirements use `### REQ-<DOMAIN>-NNN` headings (or `### Requirement REQ-AGT-NNN` for the agents capability). Scenarios use `#### Scenario: <name>` followed by `- **WHEN**` / `- **THEN**` bullets (optional `- **GIVEN**` preamble). Acceptance Criteria tables sit at the bottom of each spec. Cross-references go in `## Cross-References` (do NOT create `## Implementation map`). Change folders use the sub-folder layout `openspec/changes/<slug>/specs/<capability>/spec.md` for delta payloads. Validation runs via `pnpm openspec validate` (the adapter handles the filename-bridge symlinks). Commit messages follow Vivod's commitlint config (Conventional Commits, lowercase subject, header ≤ 100 chars). Branch shape is `studio/<change-slug>` in Hybrid mode (PRs target `main` for Vivod to merge themselves). Registry mirror tools are no-ops under this convention.

### REQ-VCA-015 — Coexistence with Vivod's own automation skills

Vivod ships its own automation skills (per [`onboard-vivod` design.md § Differences from Dentour](../../changes/onboard-vivod/design.md)):

- `vivod-openspec-workflow`
- `openspec-propose`
- `openspec-apply-change`
- `openspec-archive-change`
- `openspec-explore`
- `vivod-verify-gate`
- `vivod-ui-gatekeeper`
- `vivod-page-scaffold`
- `vivod-analytics`
- `session-analytics`

The adapter writes deltas to disk in the shape these skills expect, so a Vivod developer running `pnpm openspec apply` (or the `openspec-apply-change` skill) against a studio-authored change folder works without translation. **The adapter MUST NOT invoke these skills programmatically** — propose / apply / archive / verify is Vivod's own workflow surface, run by Vivod's developers on their cadence. See § 8 for the explicit non-goal statement.

---

## 3. Acceptance Criteria

(Vivod-style table, per the convention the adapter is being designed to handle. This is the studio's internal acceptance bar for the implementation change.)

| REQ ID | Criterion | Pass condition |
|---|---|---|
| REQ-VCA-001 | Slug + name registered | `resolveAdapter('vivod')` returns `{ slug: 'vivod', name: 'Vivod (REQ-numbered formal-delta + Fumadocs frontmatter + filename-bridge validate)' }` |
| REQ-VCA-002 | Source-of-truth detection | `index.md` is preferred over `spec.md` when both present; `spec.md`-only files load without spurious errors |
| REQ-VCA-003 | Frontmatter preservation | Round-trip read+write of every Vivod `*/index.md` at the anchored commit produces zero byte-level diff in the frontmatter region |
| REQ-VCA-004 | Heading-style heterogeneity | Parser emits Requirement tokens for both `REQ-AGT-*` (`Requirement REQ-…` form) and `REQ-AUTH-*` (`REQ-…` form); regression test fixture covers both |
| REQ-VCA-005 | Scenario tokeniser | Parser emits Scenario tokens with `name` / `when` / `then` / optional `given`; multi-line clauses collapse to single strings |
| REQ-VCA-006 | Acceptance Criteria preservation | Round-trip of `auth/index.md` (47-row table) produces zero byte-level diff |
| REQ-VCA-007 | `## Cross-References` reuse | `validate()` errors when the build agent writes `## Implementation map`; happy-path append to `## Cross-References` works |
| REQ-VCA-008 | Direct edits to `specs/` | `allowsDirectCurrentEdits()` returns `true` |
| REQ-VCA-009 | Sub-folder change layout | New change folders include `specs/<capability>/spec.md`; flat-only changes that omit it (when adding requirements) error out |
| REQ-VCA-010 | `pnpm openspec validate` | `validate()` shells out to Vivod's filename-bridge script, surfaces stderr verbatim on failure, returns empty list on success |
| REQ-VCA-011 | Commit message format | All four commitlint rules respected; out-of-enum scopes emit warn-level violations the adapter logs but does not block on |
| REQ-VCA-012 | Branch shape | Hybrid-mode branches live under `studio/<change-slug>`; PR target is `main` (Vivod merges); Full-mode same branch shape, optional `Contents: write` |
| REQ-VCA-013 | No-op registry mirrors | All four `mirror*` methods + `nextDecisionId` are intentionally undefined |
| REQ-VCA-014 | Prompt augmentation | `promptAugmentation()` emits the paragraph documented above; included in system prompt for Vivod projects only |
| REQ-VCA-015 | Skill coexistence | Adapter never imports or shells out to any of Vivod's 10 listed skills; integration test asserts no process-spawn calls match `pnpm vivod *` or `pnpm openspec (propose|apply|archive|explore)` |

---

## 4. Parser shape

A focused state machine, modelled on the existing [`packages/forge/src/conventions/diff-grammar.ts`](../../../packages/forge/src/conventions/diff-grammar.ts) parser shape but for spec-source-of-truth files rather than diff descriptions.

### 4.1 Token stream

The parser consumes a Vivod `index.md` line-by-line and emits the following token kinds:

```ts
type VivodSpecToken =
  | { kind: 'frontmatter'; raw: string }                                          // ---\n…\n--- block at the top
  | { kind: 'h1'; text: string }                                                  // # <title> — usually the Fumadocs title
  | { kind: 'h2-purpose'; raw: string }                                           // ## Purpose section body (collapsed)
  | { kind: 'h2-section'; name: string; raw: string }                             // any other ## section (e.g. ## Architecture, ## Cross-References)
  | { kind: 'requirement'; domain: string; number: number; title: string }        // ### REQ-<DOMAIN>-NNN <title>
  | { kind: 'scenario'; parentReqId: string; name: string; given: string|null; when: string; then: string }
  | { kind: 'acceptance-table'; rows: Array<{ reqId: string; criterion: string; passCondition: string }> }
  | { kind: 'cross-references-table'; rows: Array<{ from: string; to: string; note?: string }> }
  | { kind: 'prose'; raw: string };                                               // anything else; preserved verbatim on round-trip
```

### 4.2 State machine

```
START
  → frontmatter? → BODY
BODY
  → h1? → BODY
  → h2-purpose → PURPOSE_BODY → BODY
  → requirement → REQUIREMENT_BODY (scenarios accumulate under the current requirement) → BODY
  → h2-section → SECTION_BODY (acceptance-table / cross-references-table detected lazily by header pattern) → BODY
  → eof → END
```

State transitions are deterministic; the parser is a single forward pass. No backtracking. Round-trip writes the token stream back in the same order, with a per-token `raw` cache for verbatim preservation of prose / unrecognised content.

### 4.3 Implementation cost estimate

- `vivod-spec-parser.ts` (token stream + state machine + round-trip writer): **~250 LOC** (vs unbind diff-grammar at ~200 LOC; comparable scope, slightly larger because Vivod has more token kinds).
- `vivod-spec-parser.test.ts` (golden fixtures: at minimum `auth/index.md`, `agents/spec.md`, `entities/spec.md`, `design/index.md`, `forge-console-ui/index.md` — covering both heading styles, optional acceptance tables, and the variant section names): **~400 LOC** of fixture-driven assertions.

---

## 5. Validation step — `pnpm openspec validate` invocation

### 5.1 Default invocation

The adapter's `validate(changePath, ctx)` method shells out to:

```bash
cd <project.worktreeRoot> && pnpm openspec validate
```

…which Vivod's root `package.json` resolves to `bash scripts/openspec-validate.sh`. The bridge script:

1. Walks `openspec/specs/` looking for directories whose `index.md` contains both `## Purpose` and `## Requirements` (or equivalent — see [`vivod-topology.md § 7.4`](./vivod-topology.md)).
2. For each match, creates a symlink `spec.md → index.md` if `spec.md` does not already exist.
3. Runs `npx openspec validate --all --strict`.
4. Cleans up the symlinks it created (idempotent — pre-existing `spec.md` files are not touched).

### 5.2 Failure handling

- Exit 0 → `validate()` returns `[]` (empty violations).
- Exit non-zero → `validate()` returns one or more `AdapterViolation` objects:
  - `severity: 'error'`
  - `path: <changePath>` (the bridge does not always cite line/file; the adapter falls back to the change folder root)
  - `message: <bridge stderr captured verbatim>`
  - `hint: 'Run `pnpm openspec validate` locally inside the Vivod worktree to reproduce.'`

### 5.3 Why not re-implement validation in TypeScript

Two reasons:

1. **Single source of truth.** The bridge script is the canonical Vivod entry point. Re-implementing the heuristic ("does this folder have a spec?") in TS would create two definitions that drift independently. The adapter delegates.
2. **Filename-bridge is non-trivial.** The symlink-and-cleanup dance (especially the idempotency guarantee for pre-existing `spec.md` files) is correctness-critical. Re-implementing it in TS would expose the studio to subtle differences if Vivod amends the bridge.

### 5.4 Defense-in-depth posture

`pnpm openspec validate` is **not** part of Vivod's CI pipeline at the anchored commit (per [`vivod-topology.md § 6`](./vivod-topology.md)). The adapter's `validate()` step is therefore the **only** automated validation gate the studio side controls; if the studio bot lands a malformed delta, it propagates to Vivod's developers as a manual review burden. This is why REQ-VCA-010 elevates `severity='error'` violations to build-blockers (`BuildBlockedError`).

---

## 6. Commit message format — derived from `commitlint.config.cjs`

Source: [`commitlint.config.cjs`](https://github.com/VIVOD-AI/vivod/blob/3f678eb/commitlint.config.cjs) at the anchored commit, captured in [`vivod-topology.md § 1`](./vivod-topology.md).

### 6.1 Rule table (verbatim from topology)

| Rule | commitlint key | Severity | Constraint |
|---|---|---|---|
| Type allow-list | `type-enum` | 2 (error) | `feat \| fix \| refactor \| chore \| perf \| docs \| test \| ci \| build \| style` |
| Scope allow-list | `scope-enum` | 1 (warn) | `web \| ui \| db \| domain \| auth \| agents \| config \| i18n \| facade-engine \| cloudflare \| api-gateway \| scope \| ledger \| clerk \| ci \| deps` |
| Subject case | `subject-case` | 2 (error) | Subject MUST be lowercase |
| Header length | `header-max-length` | 2 (error) | `<type>(<scope>): <subject>` ≤ 100 chars |

### 6.2 Adapter-side commit assembly

```
<type>(<scope>): <subject>
```

- **`type`** — selected from the change's primary intent. Mapping table:
  - new capability spec → `feat`
  - amendment to existing spec → `docs` (Vivod treats spec-only changes as docs per the type enum's spirit; `feat` is reserved for code-bearing changes)
  - code change without spec amendment → `feat`/`fix`/`refactor`/`perf` per the build agent's existing classification
  - tooling-only → `chore` or `build`
- **`scope`** — Vivod's scope enum (15 entries). Mapping by primary affected capability:
  - `auth/` → `auth`; `api/` → `web` (closest enum entry — `api` is not in the enum); `agents/` → `agents`; `contracts/` → `web`; `worker-app/` → `web`; etc.
  - Capabilities that don't map (`forge-console-ui`, `entities`, `infra`, `design`, `ui`) get the closest enum entry the build agent can defend; if nothing fits, `chore` and an empty scope is acceptable.
  - **Out-of-enum scopes are a known warning surface.** The kickoff conversation (per [`onboard-vivod` design.md § Engagement mode default + § Differences from Dentour](../../changes/onboard-vivod/design.md) and the topology's § 9.3 flag) decides whether Vivod registers a `studio` (or `forge`) scope upstream in their `commitlint.config.cjs`. Until then, the adapter accepts the warn-level violation and logs it.
- **`subject`** — lowercased before commit. If the build agent emits `Add REQ-AUTH-048 …`, the adapter rewrites to `add REQ-AUTH-048 …`.
- **Header truncation** — if the assembled `<type>(<scope>): <subject>` exceeds 100 chars, the adapter truncates the subject at character `100 - len("<type>(<scope>): ") - 1` and appends `…`. This is a fallback; the build agent should already enforce subject brevity upstream.

### 6.3 Husky pre-commit hook coexistence

Vivod runs commitlint via a Husky pre-commit hook (per [`vivod-topology.md § 1`](./vivod-topology.md)). The studio bot's commits go through this hook on Vivod's machine when the branch is rebased / merged; the adapter does not need to install or invoke commitlint itself. The adapter's job is to **emit conformant messages** so the hook passes (or warns predictably).

### 6.4 Worked examples

| Change | Emitted commit |
|---|---|
| Add `REQ-AUTH-048` (resend OTP) | `docs(auth): add REQ-AUTH-048 OTP resend acceptance criteria` |
| Wire `useResendOtp` hook in `apps/web` | `feat(web): add useResendOtp hook for the OTP resend flow` |
| Adjust `clerk` worker prompt | `refactor(clerk): tighten contract-clause prompt` (warn: `clerk` not in enum) |
| Add a new `forge` worker handler | `feat(web): add /v1/forge/handler dispatch` (no `forge` scope; `web` is closest defensible mapping) |
| Drop a deprecated import | `chore(deps): remove unused @types/foo` |

---

## 7. Complexity estimate

**Target: 2 weeks of focused work.**
**Honest ceiling: 3 weeks** if the filename-bridge interaction with `pnpm openspec validate` proves brittle in practice (e.g., symlink races on a parallel build, Vivod amending the bridge mid-flight).

### 7.1 Drivers (vs the Unbind adapter as the comparable baseline)

The Unbind adapter (per [`openspec/archive/2026-05-08-add-forge-convention-adapter/`](../../archive/2026-05-08-add-forge-convention-adapter/)) shipped at ~330 LOC for the adapter + ~200 LOC for the diff-grammar parser + 290 tests across the convention layer, in roughly 2 weeks of Phase 2 + Phase 3 work.

Vivod's adapter is meaningfully larger because the dialect is meaningfully larger:

| Driver | Unbind | Vivod | Delta |
|---|---|---|---|
| Source-of-truth file shape | flat `current/<cap>.md` with prose body | nested `specs/<cap>/index.md` with Fumadocs frontmatter | +frontmatter preservation |
| Heading styles | plain `## <Section>` prose | TWO REQ heading variants (`### REQ-` + `### Requirement REQ-`) | +parser branch |
| Structured payload | none (prose only) | Scenarios + Acceptance Criteria tables, both round-trippable | +token kinds |
| Change-folder shape | flat 3 files | sub-folder `specs/<cap>/spec.md` payload | +scaffold branch |
| Validation | declarative TS checks (frontmatter, no-`current/`-edits) | shell-out to `pnpm openspec validate` filename bridge | +process spawn + stderr capture |
| Commit format | none (Unbind uses studio commit style) | full commitlint enum + length + case rules | +commit assembler |
| Branch shape | `studio/<slug>` (same as proposed Vivod default) | `studio/<slug>` + Hybrid/Full-mode escalation matrix | comparable |
| Registry mirror | 4 typed mutations (decisions / dependencies / maturity / open-questions) | none (intentionally undefined) | -registry surface |
| Diff-at-archive | yes (Phase 3 grammar) | no (Vivod applies edits inline) | -archive grammar |

Net: Vivod gains a parser + frontmatter handler + commit assembler; loses the registry mirror + diff-at-archive grammar. Roughly 1.5–2× the Unbind adapter's surface, depending on how comprehensive the parser fixture set is.

### 7.2 Estimated phasing for the implementation change

This is the **suggested** phasing for `add-forge-vivod-convention-adapter` (the implementation change opened as a shell in T6.2). It is not binding — the implementation change owns its own tasks.md.

| Phase | Surface | Effort estimate |
|---|---|---|
| 1 | Adapter scaffold + `slug` / `name` / `promptAugmentation` / no-op mirror methods | 0.5 day |
| 2 | Spec parser (`vivod-spec-parser.ts`) + golden-fixture round-trip tests | 4–5 days |
| 3 | `validate()` shell-out to `pnpm openspec validate` + stderr capture + `BuildBlockedError` integration | 2 days |
| 4 | Commit message assembler + lowercase / truncate + scope-mapping table | 1 day |
| 5 | Branch shape helper (Hybrid + Full) + integration with the build agent's branch-naming step | 1 day |
| 6 | `forge spec --fix --project vivod` scaffolds the sub-folder change-folder layout correctly | 1 day |
| 7 | End-to-end smoke test against a real Vivod fixture (vendored capability spec from the anchored commit) | 1 day |
| 8 | Documentation: amend `openspec/current/forge/entities.md` with the Vivod adapter; add the engagement-charter cross-reference | 0.5 day |
| | **Total** | **11 working days ≈ 2.2 weeks** |

### 7.3 Escalation triggers

If during implementation any of the following surface, the implementation owner SHOULD pause and either escalate to a standalone proposal or amend `add-forge-vivod-convention-adapter` with new design notes before continuing:

1. **Vivod amends `scripts/openspec-validate.sh`** between Phase 6 design (this doc) and the implementation start. The bridge script's heuristic for "this is a spec dir" is what the adapter delegates to; if it changes, the adapter's mental model breaks. Re-anchor the implementation against the new commit SHA before continuing.
2. **Vivod adopts a new requirement heading style** (e.g., a third variant beyond `REQ-` and `Requirement REQ-`). The parser invariant in REQ-VCA-004 is robust to the two known forms only.
3. **`commitlint.config.cjs` adds `subject-max-length` or other constraints** beyond the four documented in § 6.1. The adapter's commit assembler covers exactly the four; a fifth rule would need explicit handling.
4. **Vivod stands up a `pnpm openspec validate` CI gate.** Currently the adapter is the only enforced check; if Vivod adds CI enforcement, the studio's `BuildBlockedError` posture (REQ-VCA-010) can soften from "block on any error" to "warn on error and let CI catch it" — but the choice belongs to the implementation owner, not the design.
5. **Adapter exceeds 600 LOC or test count exceeds 60 cases.** Either signal indicates the dialect is more complex than this design captured. Pause and re-design before continuing.

The 2-week target is honest given the scope captured here. The 3-week ceiling exists because filename-bridge correctness has historically been a debugging hot spot (per [`vivod-topology.md § 6`](./vivod-topology.md) flagging the filename-bridge as a "dance" worth understanding before touching). Slipping past 3 weeks should trigger a hard escalation.

---

## 8. Explicit non-goal — do NOT invoke Vivod's own automation skills

The adapter's job is to write deltas to disk in Vivod's shape. Vivod's developers run their own propose / apply / archive flow on their cadence. The adapter MUST NOT shell out to or programmatically invoke any of Vivod's automation skills:

- `vivod-openspec-workflow`
- `openspec-propose`
- `openspec-apply-change`
- `openspec-archive-change`
- `openspec-explore`
- `vivod-verify-gate`
- `vivod-ui-gatekeeper`
- `vivod-page-scaffold`
- `vivod-analytics`
- `session-analytics`

(Source: [`onboard-vivod` design.md § Differences from Dentour](../../changes/onboard-vivod/design.md). Inventory anchored at the topology's anchor SHA; verify currency at implementation time.)

Reasons:

1. **Boundary integrity.** The studio's adapter writes deltas; Vivod's skills apply / verify / archive them. Crossing the boundary couples the studio's release cadence to Vivod's skill versioning.
2. **Dual-execution risk.** If both the adapter and `vivod-verify-gate` run `pnpm openspec validate`, the same failure surface gets reported twice with potentially different framing — confusing for whoever reads the build log.
3. **Hybrid-mode contract.** In Hybrid mode (the proposed default), Vivod-side merge happens via PR review or `pnpm openspec apply` run by Vivod's developers — the studio bot's lane stops at "deltas on a `studio/<slug>` branch." Calling Vivod's apply skill would jump that lane.

Concrete prohibitions that the integration test enforces (REQ-VCA-015 acceptance row):

- No `child_process.spawn` / `execa` call matching `pnpm openspec (propose|apply|archive|explore)`.
- No `child_process.spawn` / `execa` call matching `pnpm vivod *`.
- No import from Vivod's worktree under `apps/web/src/...` or `packages/.../skills/...`.

The single allowed shell-out is `pnpm openspec validate` (REQ-VCA-010), which is the OpenSpec CLI itself (via the filename-bridge wrapper) — not a Vivod-authored skill.

---

## 9. Cross-references

- Adapter framework (parent): [`openspec/archive/2026-05-08-add-forge-convention-adapter/`](../../archive/2026-05-08-add-forge-convention-adapter/) — interface, registry, build-agent integration shape.
- Studio adapter (back-compat baseline): [`packages/forge/src/conventions/studio.ts`](../../../packages/forge/src/conventions/studio.ts).
- Unbind adapter (closest analogue): [`packages/forge/src/conventions/unbind.ts`](../../../packages/forge/src/conventions/unbind.ts) + [`packages/forge/src/conventions/diff-grammar.ts`](../../../packages/forge/src/conventions/diff-grammar.ts).
- Companion research: [`vivod-topology.md`](./vivod-topology.md) (dialect grammar source) + [`vivod-drift-audit.md`](./vivod-drift-audit.md) (heading-style heterogeneity, dialect divergences).
- Engagement profile: [`vivod.yaml`](../../_methodology/projects/vivod.yaml) (mode, branch shape, scope policy decisions deferred to kickoff).
- Implementation shell: [`openspec/changes/add-forge-vivod-convention-adapter/`](../../changes/add-forge-vivod-convention-adapter/) — opened in T6.2 of this change.
- Engagement charter §§ 4, 7, 9.2: [`openspec/_methodology/rapoport-studio-engagement.md`](../../_methodology/rapoport-studio-engagement.md) — mode taxonomy + branch convention + bot-scope ceilings the adapter must respect.
