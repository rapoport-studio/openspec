# openspec — agent guide

> Read this before working in `openspec/changes/<slug>/` or amending `openspec/current/*.md`.
> Companion to repo-root [CLAUDE.md](../CLAUDE.md). This file owns the OpenSpec-specific conventions; CLAUDE.md owns the broader workflow and hard rules.

---

## Prose-style proposals

Every change in `openspec/changes/<slug>/` consists of **four** prose files:

```
openspec/changes/<slug>/
├── proposal.md       # Why + What changes
├── design.md         # How (interfaces, data shapes, boundaries)
├── tasks.md          # Phased checklist for the executor
└── verification.md   # Deterministic acceptance assertions (NEW — produced by `muse draft`)
```

There is **no `specs/` directory**, no per-capability delta files, no `Requirement` blocks. Capability specs live in `openspec/current/*.md` and are amended in place at archive time — not delta-restructured during a proposal.

### `verification.md` canonical shape

Four sections, in this order:

```markdown
# Verification — <change-slug>

> Companion to proposal.md / design.md / tasks.md. Lists the deterministic
> acceptance assertions Forge runs before opening the PR.

## Spec alignment
Prose (1-2 paragraphs). What "done" means at the capability-spec level.

## Hallucination risks
Bulleted list: MUST appear / MUST NOT appear paths, packages, function names.

## Pattern references
Bulleted list: existing files the diff should mimic in shape/style.

## Acceptance
One deterministic assertion per line. Six allowed shapes:
- Done when: [ -f <path> ]
- Done when: grep -c '<marker>' <path> returns <N>
- Done when: pnpm <cmd>
- Done when: gh pr list --head <branch> returns ≥<N>
- Done when: jq -r '<path>' <state.json> returns '<value>'
- Reviewer confirms: <semantic claim — LLM reviewer carve-out>
```

**Two mandatory boilerplate gates** (always the last two lines in `## Acceptance`, injected by `muse draft`):

```
- Done when: gh pr list --head <branch-for-this-change> returns ≥1
- Done when: jq -r '.outcome' ~/.forge/state/<RAP-NN>.json returns 'completed-with-pr'
```

These cover the two most common silent-failure modes: PR not created and outcome field not yet populated.

`forge build` runs the `## Acceptance` section at the spec-acceptance checkpoint (checkpoint #2.5, between `migrations` and `code-review`). Missing `verification.md` is legacy-mode — the checkpoint is skipped for changes that pre-date `add-spec-acceptance-gate`.

### What gets amended where, and when

- **During a proposal/implementation** (Phase 2 → Phase 10): files only exist under `openspec/changes/<slug>/`. `current/` is untouched.
- **At archive** (Phase 11): the executor appends a dated section to the relevant `openspec/current/<capability>.md` file using the live amendment pattern:

  ```
  ## <Title> (applied YYYY-MM-DD from change <slug>)
  ```

  See [current/platform.md](current/platform.md) for live examples.

- **`openspec/current/` is otherwise read-only.** Direct edits outside an archive step are not allowed (CLAUDE.md § Hard rule 7). Exception: typo and broken-link fixes, called out separately in the PR description.

---

## Why we don't run `openspec validate`

The OpenSpec CLI's `openspec validate <change>` exits 1 on every studio change with:

> Change must have at least one delta… ensure your change has a `specs/` directory.

This is the strict-deltas validator expecting a formal `specs/<capability>/spec.md` structure that the studio convention deliberately doesn't use. **The failure is expected.** Specifically:

- Nothing in CI runs `openspec validate` (verified: no references in `.github/`, `packages/forge/`, or any pre-commit hook).
- The forge CLI does not invoke it.
- Reviewers who run it locally will hit the error — that does not mean the proposal is broken.

**Skip the command.** If a contributor or tool reports the strict-deltas error, the correct response is to point them at this file, not to restructure the proposal.

This was tracked in [RAP-18](https://linear.app/rapoport-studio-slr/issue/RAP-18) — Option A (accept and document) was chosen over Option B (adopt formal deltas) and Option C (fork the validator). Revisit if the contributor base scales and machine-checkable specs become valuable.

---

## Schema bundle

As of [RAP-889](https://linear.app/rapoport-studio-slr/issue/RAP-889) the prose convention is also declared machine-readably at [`openspec/schemas/rapoport-prose/`](schemas/rapoport-prose/) — a four-artifact schema (`proposal → design → tasks → verification`) auto-discovered by the OpenSpec CLI 1.3.x without an `openspec/config.yaml`.

What the bundle gives us today (verified against CLI 1.3.1):

- `npx -y @fission-ai/openspec@latest schemas` lists `rapoport-prose (project)` as the active schema with its full artifact chain.
- `npx -y @fission-ai/openspec@latest list` shows tasks-completion counts parsed from each change's `tasks.md` per the bundle.
- `npx -y @fission-ai/openspec@latest schema validate rapoport-prose` exits 0 (schema structure + template references are internally consistent).
- The bundle is publishable to [`wearetechnative/awesome-openspec`](https://github.com/wearetechnative/awesome-openspec) — Phase 3 of the RAP-887 epic ([RAP-891](https://linear.app/rapoport-studio-slr/issue/RAP-891)).

What the bundle does **not** yet do:

- `npx -y @fission-ai/openspec@latest validate <change>` still emits the strict-deltas error. The validator hardcodes the spec-driven delta requirement regardless of which schema is project-active — this reproduces on `add-rapoport-prose-schema-bundle` and on existing prose changes like `wire-canvas-to-repo-binding-into`. The `schema` command tree is marked `[experimental]` in `openspec --help`; per-schema validation rules are not yet implemented upstream. **The workaround in the section above therefore stays in force.** Track [`Fission-AI/OpenSpec` releases](https://github.com/Fission-AI/OpenSpec/releases) for schema-aware validation.
- `npx -y @fission-ai/openspec@latest show <change>` requires a `## Why` section header that our convention does not use. Cosmetic; address if/when an upstream PR lands schema-aware section validation.

---

## Skills registry

The studio publishes two kinds of skills, each loaded by a different consumer:

| Surface | Path | Audience | Lifecycle |
|---|---|---|---|
| **Claude Code skills** | `.claude/skills/<name>/SKILL.md` | Claude Code working in this repo. Files are loaded at session start; matched against task context via the frontmatter `description`. | Hand-authored. Committed to the repo. `.gitignore` only excludes `.claude/worktrees/` and `.claude/scratch/`. |
| **Forge engine seed-skills** | `packages/forge/src/seed-skills/<name>/SKILL.md` | Forge build executor (consumes them while running `forge build`). | Hand-authored. Bundled into `@rapoport-studio/forge` at build time. |

### Canonical SKILL.md shape

Minimal frontmatter (`name` + `description`) + procedural markdown body. End the body with a `Memory anchors:` line citing the auto-memory keys the skill formalizes — preserves the audit trail from personal memory to shared skill.

```markdown
---
name: <kebab-case-name>
description: <one-sentence trigger condition; under ~30 words; specific enough that Claude Code recognizes the situation>
---

<Procedural markdown body. Numbered steps for procedures, fenced code blocks for commands, bullets for constraints. Cross-reference other skills by name.>

Memory anchors: `<memory_key_1>`, `<memory_key_2>`.
```

We do **not** use the verbose `openspec init`–generated shape (extra `license`, `compatibility`, `metadata.{author,version,generatedBy}` fields). The minimal shape matches the in-repo Forge precedent at `packages/forge/src/seed-skills/`.

### Promotion rule: memory → skill

A procedural fact in auto-memory (`~/.claude/projects/.../memory/`) is a candidate for promotion to a shared skill when **the same fact applies to all agents in this repo, not just one Claude session**. Personal preferences (branch naming, parallel-vs-sequential, slack-vs-email) stay in memory. Operational procedures (archive workflow, migration reconcile, PR-body conventions) graduate to skills.

Canonical examples (PR landing RAP-890): `.claude/skills/{archive-openspec-change,pr-body-no-closes-keyword,supabase-migration-reconcile}/SKILL.md`.

---

## Exemplar: the latest archived change

When in doubt about the prose convention, **the most recent archived change is canonical**. There is intentionally no `TEMPLATE/` folder — a stale template would diverge from real practice and create the question "which version is current?". The living archive answers it instead.

Pick the newest entry under `openspec/archive/` and copy its shape:

```
ls openspec/archive/ | sort -r | head -1
```

For reference, [archive/2026-05-03-add-muse-research-tools-with-personas/](archive/2026-05-03-add-muse-research-tools-with-personas/) is a representative recent example showing all three prose files plus the eventual `current/*.md` amendment pattern.

---

## MVP CLI orchestra

Three CLIs collaborate to author, build, and observe changes:

| CLI | Package | Role |
|---|---|---|
| `muse` | `packages/muse/bin/muse/` | Spec-authoring agent. `muse draft RAP-NNN` turns a Linear issue into an OpenSpec change triplet. |
| `forge` | `packages/forge/` | Build executor. `forge build RAP-NNN` implements the tasks from an OpenSpec change and opens a PR. |
| `maestro` | `packages/muse/bin/maestro/` | Telemetry surface. `maestro lineage RAP-NNN` shows the full event trace; `maestro metrics` aggregates cost. |

All three emit structured events to `~/.rapoport/events/YYYY-MM-DD.jsonl` via `@rapoport-studio/cli-core/events`. The shared event library lives at `packages/cli-core/`.

**End-to-end MVP flow:**

```
muse draft RAP-456           # draft the OpenSpec change
# review + edit openspec/changes/<slug>/
# open PR manually, get it merged
forge build RAP-456          # implement the tasks (opens code PR)
maestro lineage RAP-456      # see the full trace of both runs
```

**Relevant specs:**

- `openspec/current/cli-core.md` — shared event library
- `openspec/current/agents/muse.md` — Muse CLI spec
- `openspec/current/agents/maestro.md` — Maestro CLI spec
- `openspec/current/forge/README.md` — Forge CLI spec
- `openspec/archive/2026-05-12-bootstrap-cli-orchestra-mvp/` — the change that shipped this

---

## Folder naming

- **While active**: bare slug — `openspec/changes/<slug>/` (e.g. `add-client-onboarding`). No date prefix.
- **At archive**: rename to `openspec/archive/YYYY-MM-DD-<slug>/`. The date prefix is added only at archive time.

The OpenSpec CLI rejects date-prefixed change names with `--json`; manual `mv` for archive is the workaround (CLAUDE.md § Known environment gotchas).
