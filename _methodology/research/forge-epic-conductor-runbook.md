# Forge Epic + Conductor — Operator Runbook

> **5-minute read.** Everything you need to set up a Linear epic for the conductor, drive it with `forge epic`, and operate the long-running `forge conductor` daemon. For deep design context see [`openspec/changes/add-forge-epic-conductor/design.md`](../../changes/add-forge-epic-conductor/design.md); for the full command reference see [`openspec/current/forge/commands.md`](../../current/forge/commands.md).

---

## Prerequisites

| Requirement | Why |
|---|---|
| Node.js ≥ 20 | Runtime |
| `pnpm` ≥ 9 | Workspace install |
| `ANTHROPIC_API_KEY` set | Required by `forge build` (the sub-issue builder) |
| `LINEAR_BOT_API_KEY` set | Required for all Linear reads/writes |
| `GITHUB_TOKEN` set | Required for `--auto-push`, `--auto-pr`, `--auto-merge` |
| Repo cloned + `pnpm install` completed | Builds `packages/forge/dist/bin/forge/index.js` |

Secrets are managed in **Infisical** (org `Rapoport Studio SLR`, project `rapoport-studio`). Set locally via environment variables.

---

## Epic body conventions

The conductor reads specific Markdown sections from the **Linear epic body**. These sections are parsed by `packages/forge/src/epic/parse-epic.ts`.

### Required sections

```markdown
## Sub-issues

| Key | Title | Wave | Depends on |
|---|---|---|---|
| RAP-401 | W1 — Add data model | W1 | |
| RAP-402 | W2 — Add API routes | W2 | RAP-401 |
| RAP-403 | W3 — Add UI form  | W3 | RAP-402 |
```

- **Wave** (`W1`, `W2`, …) — dispatch order. Lower wave = earlier.
- **Depends on** — `blockedBy` relation keys. Conductor reads these from Linear; the table is used for display only (Linear is authoritative).

### Optional sections

```markdown
## OpenSpec change

add-forge-epic-conductor
```

When set, the conductor can open an archive PR (`docs/<epic>/archive-<slug>`) after all waves merge.

```markdown
## Auto-merge policy

trivial
```

When set to `trivial`, sub-issues with the `auto-merge-safe` Linear label are merged automatically once CI passes.

---

## Sub-issue body conventions

Each sub-issue body is parsed by `packages/forge/src/epic/parse-issue.ts`. Sections:

```markdown
## Scope

One paragraph describing what this sub-issue implements.

## Out of scope

What is explicitly not covered.

## Wave

W2

## Depends on

RAP-401

## Spec source

openspec/changes/add-forge-epic-conductor/design.md § Phase 3
```

Run `forge epic <parent-key> --validate` to lint all sub-issue bodies before dispatching.

---

## One-shot: `forge epic`

Use `forge epic` to drive a single epic from the command line (no daemon required).

### Validate epic structure

```bash
forge epic RAP-359 --validate
```

Checks the epic body + all sub-issue bodies. Exits non-zero on errors.

### Dry-run (plan only, no builds)

```bash
forge epic RAP-359 --dry-run
```

Prints the dispatch plan (sub-issues in topo order) without running any builds.

### Run the epic

```bash
forge epic RAP-359 --auto-push --auto-pr
```

Dispatches each sub-issue in wave order. For each child:
1. Creates a plan (`forge plan <child>`) if none exists.
2. Runs `forge build <child> --auto-push --auto-pr`.
3. Records outcome in `~/.forge/epics/RAP-359.json`.

### Check status

```bash
forge epic RAP-359 --status
```

Prints the current state from `~/.forge/epics/RAP-359.json`.

### Wait for merge + auto-archive

```bash
forge epic RAP-359 --auto-push --auto-pr --wait-for-merge
```

After each child's PR is opened, polls `gh pr view --json mergedAt` until the PR merges (default timeout: 4 h, poll interval: 30 s). Only advances to the next wave after the upstream merge is confirmed. On full epic completion, opens an archive PR automatically.

### Auto-merge (for eligible sub-issues)

```bash
forge epic RAP-359 --auto-push --auto-pr --wait-for-merge --auto-merge
```

Sub-issues carrying the `auto-merge-safe` Linear label have GitHub auto-merge enabled via `gh pr merge --auto --squash` as soon as required CI passes.

### Useful flags

| Flag | Default | Meaning |
|---|---|---|
| `--auto-push` | off | Push branch after build |
| `--auto-pr` | off | Open PR after push |
| `--wait-for-merge` | off | Poll until PR merges before next wave |
| `--auto-merge` | off | Enable GitHub auto-merge for `auto-merge-safe` sub-issues |
| `--wait-lock[=N]` | off | Wait up to N seconds for the build lock (or indefinitely if bare `--wait-lock`) |
| `--limit N` | all | Cap the number of sub-issues dispatched |
| `--include-all` | off | Dispatch all Todo sub-issues, not just `auto-build`-labelled ones |
| `--continue-on-error` | auto | Keep dispatching siblings after a failure (default: auto-detected from graph) |
| `--dry-run` | off | Print dispatch plan without running builds |
| `--sandbox` | off | Skip OpenSpec change validation (useful for tech-debt epics) |
| `--verbose` | off | Extra logging |

---

## Conductor daemon: `forge conductor`

The conductor daemon polls Linear every 5 minutes (configurable) for epics labelled `forge-conductor` in `Todo` or `In Progress` state, and drives each through `forge epic` automatically.

### Label an epic for the conductor

In Linear, add the label **`forge-conductor`** to the epic issue. The conductor discovers it on the next poll cycle.

### Start the daemon

```bash
forge conductor
```

Runs in the foreground. Logs each poll cycle to stdout. State persisted to `~/.forge/conductor.json`. SIGINT/SIGTERM triggers a graceful shutdown (finishes the current poll cycle, then exits 0).

### Custom poll interval

```bash
forge conductor --poll-interval-seconds 120
```

### Check daemon status

```bash
forge conductor --status
```

Prints `~/.forge/conductor.json` (running/stopped, last poll timestamp, active and paused epics).

### Pause a single epic

```bash
forge conductor --pause RAP-359
```

Creates `~/.forge/conductor-pause/RAP-359`. The conductor skips this epic on subsequent poll cycles. The epic stays in `activeEpics` in the state file so you can see it is paused.

### Pause all epics

```bash
forge conductor --pause-all
```

Creates `~/.forge/conductor.paused`. All epics are skipped until you resume.

### Resume a paused epic

```bash
forge conductor --resume RAP-359
```

Removes `~/.forge/conductor-pause/RAP-359`.

---

## Build lock (concurrent session safety)

The conductor and ad-hoc `forge build` sessions share `~/.forge/locks/build.lock`. The lock is:

- **Acquired** at the start of each `forge build` (globally, or per-repo with `--repo-root`).
- **Released** on success, failure, or SIGINT.
- **Auto-recovered** if the holding PID is dead (stale lock).

If you start a manual `forge build` while the conductor is running, use `--wait-lock` to queue instead of failing immediately:

```bash
forge build RAP-404 --auto-push --auto-pr --wait-lock
# → waits for the conductor's current build to finish, then proceeds
```

To see what holds the lock:

```bash
cat ~/.forge/locks/build.lock
# → { "pid": 12345, "issueKey": "RAP-401", "startedAt": "2026-05-12T10:00:00Z" }
```

---

## State files

| Path | Contents |
|---|---|
| `~/.forge/conductor.json` | Daemon state: running, last poll, active epics |
| `~/.forge/epics/<KEY>.json` | Per-epic phase + wave statuses + PR URLs |
| `~/.forge/locks/build.lock` | Global build lock (pid + issueKey + startedAt) |
| `~/.forge/conductor-pause/<KEY>` | Per-epic pause sentinel (empty file) |
| `~/.forge/conductor.paused` | Global pause sentinel (empty file) |

---

## Triage checklist

| Symptom | Where to look | Fix |
|---|---|---|
| Conductor skips an epic silently | `~/.forge/epics/<KEY>.json` — check `phase` | If `error`, delete the file or reset `phase` to `discover` |
| Build lock stuck after crash | `~/.forge/locks/build.lock` — PID dead? | `rm ~/.forge/locks/build.lock` (safe if PID is not running) |
| Epic paused unintentionally | `~/.forge/conductor-pause/<KEY>` present | `forge conductor --resume <KEY>` |
| All epics paused | `~/.forge/conductor.paused` present | `rm ~/.forge/conductor.paused` |
| Sub-issue dispatched but no PR | `~/.forge/epics/<KEY>.json` — `wave.status = "dispatched"` | Re-run `forge epic <KEY>` — finalize phase will commit/push/PR |
| Auto-rebase ambiguous (conflict) | Wave status `conflict-rebase-ambiguous` | Resolve manually, push, then re-run `forge epic <KEY>` |

---

## Quick reference

```bash
# Validate epic before first run
forge epic RAP-359 --validate

# Standard one-shot (operator merges PRs manually)
forge epic RAP-359 --auto-push --auto-pr

# Full auto (merges happen automatically for auto-merge-safe children)
forge epic RAP-359 --auto-push --auto-pr --wait-for-merge --auto-merge

# Long-running daemon (picks up all forge-conductor-labelled epics)
forge conductor

# Status checks
forge epic RAP-359 --status
forge conductor --status
```
