# Maestro CLI — Operator Runbook

> **5-minute read.** Everything you need to install, verify, and operate the `maestro` binary. For deep capability context see [`openspec/current/agents/maestro.md`](../../../current/agents/maestro.md); for argv / exit-code contracts see [`openspec/_methodology/tools/cli-standards.md`](../tools/cli-standards.md).

---

## Prerequisites

| Requirement | Why |
|---|---|
| Node.js ≥ 20 | Runtime for the CLI binary |
| `pnpm` ≥ 9 | Workspace install |
| `ANTHROPIC_API_KEY` set | Required for `route` and `audit`; `doctor` checks it but degrades gracefully |
| `LINEAR_BOT_API_KEY` set | Required for `route` and `audit`; `doctor` checks it but degrades gracefully |
| Repo cloned and `pnpm install` completed | Builds `packages/muse/dist/bin/maestro/index.js` and wires the `maestro` bin entry |

Secrets are managed in **Infisical** (org `Rapoport Studio SLR`, project `rapoport-studio`). Set locally via environment variables; in CI they are injected from Infisical before any CLI invocation.

---

## Install

```bash
# From the repo root — installs all workspace packages and builds the binary
pnpm install

# Verify the binary is on PATH (pnpm workspace link)
maestro --version
# → maestro 0.0.2 (a1b2c3d)
```

The binary is declared in `packages/muse/package.json` as:

```json
"bin": { "maestro": "dist/bin/maestro/index.js" }
```

`pnpm install` links it into `.pnpm-bin/maestro` and the workspace root's `node_modules/.bin/`. After install, `pnpm maestro` or `npx maestro` (from repo root) both resolve correctly.

---

## `maestro doctor`

Run before any long session or CI pipeline to verify all credentials and state directories are healthy.

```bash
maestro doctor
```

**Expected healthy output (TTY):**

```
maestro doctor
  ✓ version              maestro 0.0.2 (a1b2c3d)
  ✓ auth.anthropic       OK (resp 142ms)
  ✓ auth.linear          OK
  ✓ state.fs             OK (~/.studio/maestro/ writable)
  ✓ cost.config          OK ($5.00 cap)

4 of 4 checks passed.
```

**Machine-readable output (for CI/scripts):**

```bash
maestro doctor --json
```

```json
{
  "ok": true,
  "data": {
    "checks": [
      { "name": "version", "status": "pass", "value": "maestro 0.0.2 (a1b2c3d)" },
      { "name": "auth.anthropic", "status": "pass", "latency_ms": 142 },
      { "name": "auth.linear", "status": "pass" },
      { "name": "state.fs", "status": "pass" },
      { "name": "cost.config", "status": "pass", "value": "$5.00 cap" }
    ],
    "passed": 4,
    "total": 4
  }
}
```

**Exit codes from `doctor`:**

| Code | Meaning |
|---|---|
| `0` | All checks passed |
| `1` | Operator action required (e.g. missing `cost.config`) |
| `2` | External dependency unreachable (API / network transient) |

**Common `doctor` errors:**

| Error | Cause | Fix |
|---|---|---|
| `auth.anthropic FAIL: ANTHROPIC_API_KEY not set` | Key missing from environment | `export ANTHROPIC_API_KEY=<key>` or pull from Infisical |
| `auth.linear FAIL: LINEAR_BOT_API_KEY not set` | Key missing | `export LINEAR_BOT_API_KEY=<key>` or pull from Infisical |
| `state.fs FAIL: ~/.studio/maestro/ not writable` | Permissions issue | `mkdir -p ~/.studio/maestro && chmod u+rwx ~/.studio/maestro` |
| `cost.config: missing` | `~/.studio/config.json` absent or has no `maestro` section | Create `~/.studio/config.json` — see § Config below |
| `auth.anthropic FAIL: 401` | Key is set but invalid | Rotate in Infisical and re-export |

---

## `maestro route <issue-key>`

_(Phase 1.4 — requires the Muse mode runtime at `packages/muse/src/modes/maestro/`. Currently exits `2` with "Maestro mode not yet active".)_

Routes a Linear issue to the correct agent (Forge, Muse Architect, etc.) based on the issue body, labels, and orchestra journal state. The routing decision is written to `~/.studio/maestro/journal/<YYYY-MM-DD>.md` and to the `orchestra_journal` Supabase table.

```bash
# Propose a route without committing (safe to run anytime)
maestro route RAP-404 --dry-run

# Propose a route and emit structured JSON (for scripts / CI)
maestro route RAP-404 --dry-run --json

# Route for real (HitL gate fires if routing target is ambiguous)
maestro route RAP-404
```

**Exit codes from `route`:**

| Code | Meaning |
|---|---|
| `0` | Routing decision made and recorded |
| `1` | Bad input (invalid issue key, unknown agent target) |
| `2` | System error (Linear unreachable, journal write failed) |
| `3` | Cost guard refused (estimated session exceeds cap) |
| `5` | Tool not in allowlist (spec violation) |

**Debugging a `route` failure:**

```bash
# Verbose stderr logs — shows which source resolved each credential
maestro route RAP-404 --dry-run -vv 2>&1 | head -40

# Inspect the most recent journal entry
cat ~/.studio/maestro/journal/$(date +%Y-%m-%d).md | tail -50
```

---

## `maestro audit`

_(Phase 1.4 — requires the Muse mode runtime. Currently exits `2` with "Maestro mode not yet active".)_

Detects drift between `agent_status` and `orchestra_journal`. Reports agents stuck on stale work, work that crosses declared boundaries, and decisions that have not been recorded.

```bash
# Audit all agents, last 24 hours
maestro audit --scope all --since 24h

# Audit only Forge, last 7 days
maestro audit --scope agent --since 7d

# JSON output for scripting
maestro audit --scope all --since 24h --json
```

**Common audit flags:**

| Flag | Meaning |
|---|---|
| `--scope agent\|repo\|project\|all` | Restrict audit surface (default: `all` from config) |
| `--since <ISO-8601 or duration>` | Start of time window (e.g. `24h`, `7d`, `2026-05-01`) |
| `--until <ISO-8601 or duration>` | End of time window (default: now) |
| `--dry-run` | Print what would be audited; skip journal write |
| `--json` | Emit structured JSON to stdout |

---

## `maestro stats`

_(Phase 1.5 — deferred until `orchestra_journal` has at least one production routing session.)_

Aggregates NDJSON logs across all agents under `~/.studio/<agent>/logs/*.ndjson`. Reports per-agent cost, log-level distribution, and time-windowed session counts.

```bash
# TTY summary table
maestro stats

# Time-windowed, all agents
maestro stats --since 7d --scope all

# JSON array of items
maestro stats --since 7d --json
```

---

## Config

Maestro reads `~/.studio/config.json` (overridable with `--config <path>`). The minimal working config:

```json
{
  "default": {
    "anthropic": {
      "cost_cap_usd_per_session": 5.00
    },
    "log_level": "warn"
  },
  "maestro": {
    "audit": {
      "default_scope": "all"
    }
  }
}
```

If the file is absent Maestro uses built-in defaults. The only hard requirement surfaced by `doctor` is that a `cost_cap_usd_per_session` value is parseable — either from the config or the environment.

**State directories** created automatically on first run:

```
~/.studio/maestro/
├── state/      per-run machine-readable state (JSON)
├── journal/    routing rationale logs (Markdown, YYYY-MM-DD.md)
├── locks/      mutex files (PID + timestamp)
├── cache/      transient; safe to delete
└── logs/       JSON-lines structured logs (rotated at 50 MB)
```

Override the root with `STUDIO_HOME=/custom/path` — useful for container runs or CI isolation.

---

## Debug tips

**Trace credential resolution:**

```bash
maestro doctor -vv 2>&1 | grep '"level":"debug"' | jq .
```

Each credential logs which source resolved it (`infisical`, `env`, or `file`). Values are never logged.

**Inspect raw structured logs:**

```bash
# Most recent 50 log entries
tail -50 ~/.studio/maestro/logs/maestro.ndjson | jq .

# Filter for errors only
cat ~/.studio/maestro/logs/maestro.ndjson | jq 'select(.level == "error")'
```

**Force JSON output in any command (useful for `jq` pipelines):**

```bash
maestro doctor --json | jq '.data.checks[] | select(.status == "fail")'
```

**Disable colour (e.g. in a log file):**

```bash
maestro doctor --no-color 2>&1 | tee doctor.log
```

**CI profile (suppresses colour, sets `info` log level):**

Add to `~/.studio/config.json`:

```json
{
  "_profiles": {
    "ci": {
      "default": { "no_color": true, "log_level": "info" }
    }
  }
}
```

Then invoke as:

```bash
maestro doctor --profile ci
```

---

## Common error quick-reference

| Exit code | Name | Action |
|---|---|---|
| `0` | OK | Continue |
| `1` | USER_ERROR | Fix input / config; do not retry |
| `2` | SYSTEM_ERROR | External dep transient; retry with backoff |
| `3` | COST_GUARD_REFUSED | Raise `cost_cap_usd_per_session` or use `--dry-run` |
| `4` | LOCK_HELD | Another instance running; wait for lock release |
| `5` | ALLOWLIST_DENIED | Spec violation; do not retry |

---

## See also

- [`openspec/current/agents/maestro.md`](../../../current/agents/maestro.md) — Maestro capability spec (function, access scope, risk matrix)
- [`openspec/archive/2026-05-12-add-maestro-cli/design.md`](../../archive/2026-05-12-add-maestro-cli/design.md) — CLI invocation architecture and phase breakdown
- [`openspec/_methodology/tools/cli-standards.md`](../tools/cli-standards.md) — argv conventions, exit codes, output modes, logging schema (canonical contract for all studio CLIs)
- [`tools/lint-maestro-cli.mjs`](../../../../tools/lint-maestro-cli.mjs) — CI lint script that verifies `--json`, exit codes, and `--help` section coverage
