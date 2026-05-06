# openspec — agent guide

> Read this before working in `openspec/changes/<slug>/` or amending `openspec/current/*.md`.
> Companion to repo-root [CLAUDE.md](../CLAUDE.md). This file owns the OpenSpec-specific conventions; CLAUDE.md owns the broader workflow and hard rules.

---

## Prose-style proposals

Every change in `openspec/changes/<slug>/` consists of exactly three prose files:

```
openspec/changes/<slug>/
├── proposal.md   # Why + What changes
├── design.md     # How (interfaces, data shapes, boundaries)
└── tasks.md      # Phased checklist for the executor
```

There is **no `specs/` directory**, no per-capability delta files, no `Requirement` blocks. Capability specs live in `openspec/current/*.md` and are amended in place at archive time — not delta-restructured during a proposal.

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

## Exemplar: the latest archived change

When in doubt about the prose convention, **the most recent archived change is canonical**. There is intentionally no `TEMPLATE/` folder — a stale template would diverge from real practice and create the question "which version is current?". The living archive answers it instead.

Pick the newest entry under `openspec/archive/` and copy its shape:

```
ls openspec/archive/ | sort -r | head -1
```

For reference, [archive/2026-05-03-add-muse-research-tools-with-personas/](archive/2026-05-03-add-muse-research-tools-with-personas/) is a representative recent example showing all three prose files plus the eventual `current/*.md` amendment pattern.

---

## Folder naming

- **While active**: bare slug — `openspec/changes/<slug>/` (e.g. `add-client-onboarding`). No date prefix.
- **At archive**: rename to `openspec/archive/YYYY-MM-DD-<slug>/`. The date prefix is added only at archive time.

The OpenSpec CLI rejects date-prefixed change names with `--json`; manual `mv` for archive is the workaround (CLAUDE.md § Known environment gotchas).
