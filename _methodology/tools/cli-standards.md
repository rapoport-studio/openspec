# Studio CLI standards — methodology

> Single source of truth for argv conventions, exit codes, output modes, state paths, config, logging, `doctor` command, `--help` format, and auth across studio CLIs. Per-CLI capability specs forward-reference rows in this document by section anchor; they do not inline-replicate the contract.

---

## Status

**Applied 2026-05-10 from change `add-cli-standards` ([RAP-228](https://linear.app/rapoport-studio-slr/issue/RAP-228)).** Companion to [`agent-model-mapping.md`](./agent-model-mapping.md) — both live under `openspec/_methodology/tools/`. This document is canonical for CLI behaviour across the studio's executor surface.

In-scope CLIs at authoring time:

| CLI | Status | Notes |
|---|---|---|
| `forge` ([`packages/forge/`](../../../packages/forge/)) | Mature, predates this contract | Catalogued exceptions in [§ Known gaps](#known-gaps); migration tracked separately |
| `maestro` ([`packages/muse/bin/maestro`](../../current/agents/maestro.md)) | Active | First CLI built to spec from day 1; CI lint enforced via `tools/lint-maestro-cli.mjs` |
| `muse` (future, headless invocation surface) | Will trigger rule-of-three for `packages/cli-core/` extraction | Migration plan lands at extract time |

Code extraction (`packages/cli-core/`) is **deferred** to the Muse CLI milestone per `add-cli-standards § Decisions D-1`. Until then, this document is the contract; enforcement is by review and (planned) lint rather than by shared library.

---

## Selection criteria

Three principles govern every choice in this document. Each section in the contract traces back to one of them.

**Principle A — Uniformity for scripting.** Operators write shell scripts, cron jobs, CI workflows, and Maestro `route` chains that compose multiple studio CLIs. Argv flags, exit codes, and output modes must be identical so scripts compose without per-CLI special-casing. Divergence multiplies operator cognitive load and breaks scripts at the worst time.

**Principle B — Observability for orchestration.** Maestro's future `stats` command aggregates metrics across the orchestra (cost per route, build success rate, drift observations, cost-guard fires, doctor failures over time). Aggregation works only if every CLI emits structured logs in the same shape. JSON-lines with canonical fields is the cheapest format that satisfies both human readability (with `jq`) and programmatic consumption.

**Principle C — Honesty about exceptions.** Where a CLI deviates from the contract today (e.g., Forge's `~/.forge/` state path), the contract documents the deviation as a known gap with an owner and a remediation path. Hidden deviations are surprise debt; documented deviations are visible technical debt that gets addressed deliberately.

---

## §1 Argv flag conventions

Tag in column "Principle" indicates which of A/B/C the row primarily serves.

### Universal flags (every CLI)

| Flag | Short | Type | Semantics | Principle |
|---|---|---|---|---|
| `--help` | `-h` | bool | Print help for current command and exit `0`. | A |
| `--version` | — | bool | Print `<cli> <version> (<git-sha>)` and exit `0`. | A |
| `--json` | — | bool | Switch output to JSON mode (see [§3](#§3-output-modes)). Mutually exclusive with `--quiet` for output-emitting commands. | B |
| `--no-color` | — | bool | Disable ANSI colour in TTY mode. (Pipe mode and JSON mode never emit colour.) | A |
| `--quiet` | `-q` | bool | Suppress all non-essential output. Errors still print to stderr. | A |
| `--verbose` | `-v` | bool | Emit `info`-level logs to stderr in TTY mode (default is `warn`+). Repeatable: `-vv` enables `debug`. | A |
| `--config <path>` | — | string | Override `~/.studio/config.json` location. | A |
| `--profile <name>` | — | string | Select named profile within config file (default: `"default"`). | A |

### Action-modifying flags (when applicable to the command)

| Flag | Short | Type | Semantics | Principle |
|---|---|---|---|---|
| `--dry-run` | — | bool | Compute and display what *would* happen; do not execute side effects. Default for read-mostly commands (e.g., `maestro route`). | A |
| `--yes` | `-y` | bool | Auto-approve interactive prompts. Required for cron / CI use. | A |
| `--force` | `-f` | bool | Bypass safety prompts (use with care; documented per command). Never auto-implied by `--yes`. | A |

### Time / scope flags (when applicable)

| Flag | Short | Type | Semantics | Principle |
|---|---|---|---|---|
| `--since <when>` | — | string | Time-window start. Accepts ISO-8601 (`2026-05-01`) or duration (`7d`, `24h`, `30m`). Inclusive. | A |
| `--until <when>` | — | string | Time-window end. Same format as `--since`. Exclusive. Default: now. | A |
| `--limit <N>` | `-n` | int | Cap on rows / records returned. Default per command. | A |
| `--scope <s>` | — | string | Restrict scope: `agent`, `repo`, `project`, `all`. Per-command meaningful values. | A |

### Naming rules

* Long flags use **kebab-case**: `--dry-run`, `--no-color`, not `--dryRun` / `--dry_run`.
* Boolean flags take no value: `--dry-run`, never `--dry-run=true`. Use `--no-<flag>` to negate a default-true boolean.
* Short flags are reserved for the universal table above + per-command judgment. Avoid coining new short flags for ergonomics-only.
* Plural is preferred where applicable: `--labels foo,bar` not `--label foo --label bar` (unless repetition is semantically meaningful).
* Argument-taking flags use `<placeholder>` in help, not `[arg]` or `=VALUE`.

### Conflicts and precedence

* `--json` overrides `--no-color` (JSON never has colour anyway).
* `--quiet` and `--verbose` together: `--verbose` wins (verbosity is a stronger declaration).
* `--config` + `--profile`: profile is selected within the config file pointed at by `--config`.
* CLI flags override config-file values; config-file values override built-in defaults.

---

## §2 Exit codes

Aligned with `sysexits.h` spirit but specialised for studio operations. Reserved range `0`–`9`; `64`+ reserved for future expansion.

| Code | Name | Meaning | Typical caller behaviour |
|---|---|---|---|
| `0` | `OK` | Command succeeded; output is canonical. | Continue. |
| `1` | `USER_ERROR` | Bad input from the operator: missing required arg, invalid flag value, unknown subcommand, validation failure on supplied data. | Show error and stop; do not retry. |
| `2` | `SYSTEM_ERROR` | External dependency unavailable: network unreachable, DB connection refused, Anthropic / Linear / GitHub API 5xx, FS write denied. | Retry with backoff is acceptable; treat as transient. |
| `3` | `COST_GUARD_REFUSED` | Pre-flight estimation exceeded budget cap, or in-flight cost ceiling tripped. Operator must raise the cap explicitly. | Do not retry; surface to operator. |
| `4` | `LOCK_HELD` | Another instance is holding the operation lock (e.g., Forge global build lock per [RAP-224](https://linear.app/rapoport-studio-slr/issue/RAP-224)). | Retry after lock release; safe to schedule. |
| `5` | `ALLOWLIST_DENIED` | Operation requested a tool, path, or external API not declared in the agent's allowlist (per `agents/<agent>.md § Access scope`). | Do not retry; this is a spec violation, not a transient failure. |
| `6`–`9` | reserved | Future expansion. | — |
| `64`–`78` | (reserved) | Aligned with `sysexits.h` if a CLI ever needs it. | — |

**Rules:**

* Exit code `0` is exclusive to success. Partial success (e.g., 3 of 4 records processed) → exit `1` if any item failed; the JSON output should report per-item status.
* CLIs **must** exit `0` only when stdout output is canonical and complete.
* CLIs **must not** exit `2` for user errors (missing arg, etc.); `1` is correct.
* Cron jobs and Maestro routing rely on `2` being retryable and `3`/`5` being non-retryable. Misclassifying a system error as a user error breaks retry logic.

---

## §3 Output modes

Three modes. Detection rules first; rendering rules second.

### Detection rules

| Condition | Mode |
|---|---|
| `--json` flag present | **JSON** |
| stdout is a TTY (interactive terminal) and `--json` not set | **TTY** |
| stdout is not a TTY (pipe / file redirect / cron) and `--json` not set | **PIPE** |

`--no-color` forces colour off but does not change which mode is active.

### Rendering rules

| Mode | Colour | Spinners / progress | Carriage returns | Log destination |
|---|---|---|---|---|
| **TTY** | yes (unless `--no-color`) | yes (real-time progress, transient) | yes | stderr |
| **PIPE** | no | no (status lines as plain text, line-terminated) | no | stderr |
| **JSON** | no | no | no | stderr (must remain JSON-lines per §6) |

### JSON output shape

Every JSON-mode invocation emits **one root object per command** to stdout, terminated with `\n`:

```json
{
  "ok": true,
  "data": { ... command-specific payload ... },
  "warnings": []
}
```

On error:

```json
{
  "ok": false,
  "error": {
    "code": "COST_GUARD_REFUSED",
    "exit": 3,
    "message": "Estimated $4.20 exceeds session cap $3.00",
    "hint": "Raise --cost-cap or use --dry-run to inspect"
  }
}
```

* `ok` is the canonical success bit; callers should check `ok`, not just `exit`.
* `data` is per-command; documented in `<cli> <cmd> --help`.
* `error.code` matches the exit-code name in [§2](#§2-exit-codes).
* Multi-record commands return `data: { items: [...], cursor: "..." }` for pagination.

### Stream commands

A small number of commands stream multiple events (e.g., `forge build` log tail, future `maestro journal --follow`). These emit **JSON-lines on stdout** (one object per line) and **must** declare `--json` semantics in `--help`. The non-`--json` streaming form uses TTY-style line-by-line text. Streaming commands MUST set the `Content-Type: application/x-ndjson` semantic in any documented HTTP wrapping (out of scope today).

---

## §4 State path convention

### Canonical layout

New CLIs settle under `~/.studio/<agent>/`:

```
~/.studio/
├── config.json                      # shared config (see §5)
├── <agent>/                         # per-agent state root
│   ├── state/                       # per-run / per-session state files
│   ├── journal/                     # human-readable rationale logs (markdown)
│   ├── locks/                       # mutex / lock files
│   ├── cache/                       # transient, safe to delete
│   └── logs/                        # JSON-lines structured logs (rotated)
└── shared/                          # cross-agent shared state (rare; documented per use)
```

### Per-subdirectory rules

* **`state/`** — durable, machine-readable (JSON). Each artefact has a stable id; never overwrite, only append or replace atomically (write-temp-then-rename).
* **`journal/`** — markdown, append-only, human-readable. Mirror of any DB-backed journal for offline review (per `agents/maestro.md § Access scope`).
* **`locks/`** — file-based locks. PID + timestamp inside the file for forensic recovery. Stale locks (> N hours, no live PID) are reapable.
* **`cache/`** — anything reproducible from upstream sources. CI sometimes wipes this between runs; CLIs must tolerate cold cache.
* **`logs/`** — JSON-lines per [§6](#§6-logging-format). Rotated by size (e.g., 50 MB) with `<file>.1`, `<file>.2`, etc. retention configurable per CLI.

### Environment override

`STUDIO_HOME` overrides `~/.studio`. Use case: containerised runs, multi-tenant test isolation, CI ephemeral homes. CLIs **must** honour this env var.

### Exception: Forge

Forge's existing state lives at `~/.forge/{builds,lessons-learned.md,locks/}`. Documented exception per `add-cli-standards § Decisions D-2`. Migration to `~/.studio/forge/` is deferred to the future `cli-core` extract change. Until then, Forge reads/writes from the legacy path. New CLIs **must not** copy this pattern.

---

## §5 Config file

### Location

`~/.studio/config.json` — single shared file with per-agent sections. Detected automatically; overridable with `--config <path>`.

If the file is missing, CLIs use built-in defaults; no error.

### Schema

```json
{
  "default": {
    "anthropic": {
      "model_override": null,
      "cost_cap_usd_per_session": 5.00
    },
    "log_level": "warn",
    "no_color": false
  },
  "forge": {
    "build": {
      "auto_approve_plan": false,
      "verifier_strict": true
    }
  },
  "maestro": {
    "audit": {
      "default_scope": "all"
    }
  },
  "muse": {
    "_reserved": true
  },
  "_profiles": {
    "ci": {
      "default": { "no_color": true, "log_level": "info" },
      "forge": { "build": { "auto_approve_plan": true } }
    }
  }
}
```

### Rules

* Top-level keys are agent IDs (`forge`, `maestro`, `muse`, ...) plus `default` (universal) and `_profiles` (named overrides).
* `--profile <name>` selects a profile from `_profiles.<name>`; values shallow-merge over base.
* Schema is JSON (not JSONC, not YAML, not TOML) so every CLI can parse without a dependency.
* Unknown keys are tolerated (forward-compatibility); CLIs warn but do not error on unknown keys at `info`+ verbosity.
* CLIs **must not** write to this file from within a command run. Editing is a deliberate operator action via `<cli> config set <key> <value>` (a future, optional command per CLI; not required by this contract).

### Precedence (highest wins)

1. CLI flag value
2. Environment variable (when documented per command)
3. Profile section (`_profiles.<name>.<agent>.<key>` → `_profiles.<name>.default.<key>`)
4. Agent section (`<agent>.<key>`)
5. Default section (`default.<key>`)
6. Built-in default

---

## §6 Logging format

### Destination

All structured logs go to **stderr**. stdout is reserved for command output (see [§3](#§3-output-modes)).

### Format

JSON-lines: one JSON object per line, `\n`-terminated. No leading whitespace, no pretty-printing.

### Required fields

```json
{ "ts": "2026-05-10T12:34:56.789Z", "agent": "forge", "level": "info", "msg": "Build accepted plan" }
```

| Field | Type | Notes |
|---|---|---|
| `ts` | string | ISO-8601 UTC with millisecond precision and trailing `Z`. |
| `agent` | string | Lowercase agent ID — `forge`, `maestro`, `muse`. Matches the CLI binary name. |
| `level` | string | One of `debug`, `info`, `warn`, `error`. |
| `msg` | string | Human-readable summary. Stable across runs (do not embed timestamps or unique ids in `msg`; put those in fields). |

### Optional fields (when applicable)

| Field | Type | Notes |
|---|---|---|
| `session_id` | string | Studio-assigned session UUID for cross-CLI correlation. |
| `run_id` | string | Per-invocation UUID (uniquely identifies one CLI run). |
| `cost_usd` | number | Cumulative cost so far this run. |
| `tokens_in` / `tokens_out` | number | Anthropic token counters. |
| `model` | string | Anthropic model id (e.g., `claude-opus-4-7`). |
| `tool` | string | Tool name when emitting a tool-use log. |
| `err` | object | Structured error with `{ code, message, stack? }`. Stack only at `debug` level. |

### Verbosity behaviour

* **TTY mode + no flags**: `warn`+ to stderr, formatted human-readably (e.g., yellow `⚠` for warn, red `✕` for error).
* **TTY mode + `-v`**: `info`+ to stderr.
* **TTY mode + `-vv`**: `debug`+ to stderr.
* **PIPE mode**: same threshold as TTY but no colour; logs are JSON-lines (always — pipe consumers can `jq`).
* **JSON mode**: same threshold as PIPE; logs are JSON-lines on stderr; stdout reserved for the single root JSON object.

### Aggregation

Maestro's future `stats` command (planned for `add-maestro-cli` Phase 5) reads `~/.studio/<agent>/logs/*.ndjson` from every agent and aggregates by `agent`, `level`, time window, and cost. The contract above is the schema this aggregation depends on.

---

## §7 `doctor` command

Every studio CLI **must** implement `<cli> doctor`. Operators run `forge doctor && maestro doctor && muse doctor` to verify the executor surface is healthy before a long session.

### Required checks (minimum)

| Check | Pass criterion | Failure exit |
|---|---|---|
| `version` | Binary version + git SHA known | always pass |
| `auth.anthropic` | `ANTHROPIC_API_KEY` resolvable; lightweight model request returns 200 | `2` if unreachable |
| `auth.linear` | `LINEAR_BOT_API_KEY` resolvable (only required for CLIs that touch Linear) | `2` if unreachable |
| `auth.github` | `GITHUB_BOT_TOKEN` resolvable (only required for CLIs that touch GitHub) | `2` if unreachable |
| `auth.supabase` | DB connection succeeds (only required for CLIs that touch Supabase) | `2` if unreachable |
| `state.fs` | State directory writable (`~/.studio/<agent>/` or documented exception) | `2` if read-only |
| `cost.config` | Cost cap is set and parseable | `1` if missing |

Per-CLI commands MAY add additional checks. They MUST NOT remove the minimum set above.

### Output shape

#### TTY mode

```
forge doctor
  ✓ version              forge 1.4.2 (a1b2c3d)
  ✓ auth.anthropic       OK (resp 142ms)
  ✓ auth.linear          OK
  ✓ auth.github          OK
  ✗ state.fs             FAIL: ~/.forge/builds/ not writable
                            hint: chmod u+w ~/.forge/builds/
  ✓ cost.config          OK ($5.00 cap)

5 of 6 checks passed.
```

#### JSON mode

```json
{
  "ok": false,
  "data": {
    "checks": [
      { "name": "version", "status": "pass", "value": "forge 1.4.2 (a1b2c3d)" },
      { "name": "auth.anthropic", "status": "pass", "latency_ms": 142 },
      { "name": "auth.linear", "status": "pass" },
      { "name": "auth.github", "status": "pass" },
      { "name": "state.fs", "status": "fail", "message": "~/.forge/builds/ not writable", "hint": "chmod u+w ~/.forge/builds/" },
      { "name": "cost.config", "status": "pass", "value": "$5.00 cap" }
    ],
    "passed": 5,
    "total": 6
  }
}
```

### Exit code

* All checks pass → `0`
* Any check `fail` → `1` (operator action required) or `2` (transient external dep)
* `doctor` itself errored before checks completed → `2`

### Performance budget

`<cli> doctor` should complete in **under 5 seconds** on a healthy system. Slower checks (e.g., DB roundtrip > 1s) emit a `warn`-level log noting latency.

---

## §8 `--help` template

### Section ordering

Every `<cli> --help` and `<cli> <command> --help` follows this section order:

```
USAGE
  <synopsis line(s)>

DESCRIPTION
  <one to three paragraphs explaining what the command does and when to use it>

COMMANDS                          (top-level only, omit for subcommand help)
  <name>           <one-line summary>
  ...

OPTIONS
  -<short>, --<long> <type>       <one-line description>
  ...

EXAMPLES
  $ <cli> <command> <args>
      <one-line outcome>
  ...

ENVIRONMENT
  <ENV_VAR>          <one-line description>
  ...

EXIT CODES
  0   Success
  1   User error
  2   System error
  ...

SEE ALSO
  <cli> <related-command>
  <link to spec or methodology section>
```

### Rules

* Section names in **uppercase** (Unix `man` page convention).
* Two-space indentation inside sections.
* Examples must be **runnable as written**; copy-paste should work in a shell.
* `EXAMPLES` section has at least one line. CLIs without examples are a documentation gap.
* `EXIT CODES` lists only codes the command actually emits (not the full table from [§2](#§2-exit-codes)). Reference the canonical table in `SEE ALSO`.
* `SEE ALSO` links to peer commands and to the canonical methodology document for any non-obvious behaviour.

### Length budget

* Top-level `<cli> --help`: 40–80 lines.
* Per-command `<cli> <command> --help`: 30–60 lines.
* Longer than that means the command is too complex; consider splitting.

---

## §9 Auth conventions

### Resolution chain

Every credential resolves through the same chain, in order. Stop at first hit.

1. **Infisical** (organisation `Rapoport Studio SLR`, project `rapoport-studio`, workspace `7d67e5e8-3693-487d-bb18-9fbcb288c96e`) per [`CLAUDE.md`](../../../CLAUDE.md).
2. **Environment variable** (canonical name per table below).
3. **File fallback** (only when CLI documents the file location explicitly; rare).

CLIs **must** log a `debug`-level entry naming which source resolved each credential, so `forge doctor -vv` can trace resolution. CLIs **must not** log credential values.

### Canonical environment variable names

Per [user memory](../../../CLAUDE.md):

| Credential | Env var | Used by |
|---|---|---|
| Anthropic API key | `ANTHROPIC_API_KEY` | All CLIs that call Claude |
| GitHub bot token | `GITHUB_BOT_TOKEN` | Forge (write); Maestro (no — read-only spec, but doctor checks if used) |
| Linear bot API key | `LINEAR_BOT_API_KEY` | Forge (write); Maestro (read-only) |
| Supabase service role key | `SUPABASE_SERVICE_ROLE_KEY` | CLIs that read/write `orchestra_journal`, `agent_status`, etc. |
| Supabase anon key | `SUPABASE_ANON_KEY` | Read-only DB queries from CLI when service-role isn't required |

Unconventional names (`LINEAR_API_KEY`, `GH_TOKEN`, `OPENAI_API_KEY`) are **not** read. Operators set the canonical names; CLIs read only the canonical names. This is enforced by code review and the CI lint script at `tools/lint-maestro-cli.mjs` (`add-maestro-cli` Phase 1.6).

### Sharing across CLIs

Pre-extract: every CLI imports the auth loader from `@rapoport-studio/forge/core`. The carve-out for `packages/muse → @rapoport-studio/forge/core` is amended into `CLAUDE.md` when `add-maestro-cli` lands.

Post-extract: the loader moves to `@rapoport-studio/cli-core/auth` and Forge / Maestro / Muse all import from there.

### Secret rotation

Operators rotate via Infisical and set new values into Cloudflare Workers via `wrangler secret put` (per [`CLAUDE.md § Known environment gotchas`](../../../CLAUDE.md)). CLIs are stateless about secret values — restart picks up new values from the resolution chain. CLIs **must not** cache resolved secret values across runs (per-run cache is fine and recommended).

---

## Known gaps

Documented deviations from the contract. Each has an owner and a remediation path. Hidden deviations are not allowed; if you find one, add a row.

| Gap | CLI | Remediation owner | ETA / change anchor |
|---|---|---|---|
| State path is `~/.forge/`, not `~/.studio/forge/` | `forge` | Forge maintainer | At `cli-core` extract time (rule-of-three trigger when `add-muse-cli` lands) |
| `forge build` exit code mapping not yet audited against [§2](#§2-exit-codes) | `forge` | Forge maintainer | Audit task in `add-maestro-cli` (so divergences surface during contract validation) |
| `--profile` flag not yet implemented in any CLI | all | Whoever ships the first multi-environment use case | Deferred — capture in `add-cli-config-profiles` if/when needed |
| `doctor` command not yet present in `forge` | `forge` | Forge maintainer | Tracked in [Forge known gaps](../../current/forge/verifier.md); ships when next Forge improvement wave lands |
| Pre-extract auth carve-out: `packages/muse` imports `@rapoport-studio/forge/core` directly (auth + Anthropic + Linear + GitHub clients) instead of a shared `packages/cli-core/auth` | `maestro` (via `packages/muse`) | Maestro / Muse CLI maintainer | Lifted to `@rapoport-studio/cli-core/auth` at rule-of-three trigger when Muse CLI lands as the third consumer; boundary enforced by `eslint.config.mjs` patterns rule (see `add-maestro-cli` Phase 1.1) |

This list is part of the contract. PRs that add new CLIs without compliance must add a row, not omit it.

---

## Related

* [`agent-model-mapping.md`](./agent-model-mapping.md) — sibling methodology under `_methodology/tools/`. Per-agent model bindings (Maestro / Forge / Muse / Atlas / Echo / Pulse).
* [`openspec/AGENTS.md`](../../AGENTS.md) — OpenSpec authoring conventions; explains why methodology lives at `_methodology/` and why this document amends nothing in `current/`.
* [`openspec/current/agents/maestro.md`](../../current/agents/maestro.md) — Maestro capability spec; `§ Runtime class` declares the `runMuseHeadless` headless path that the Maestro CLI binds to.
* [`openspec/current/forge/README.md`](../../current/forge/README.md) — Forge capability spec; reference implementation of many patterns this document codifies.
* [`openspec/current/muse.md`](../../current/muse.md) — Muse capability spec; future `muse` CLI is the third consumer that triggers `cli-core` extract.
* [`openspec/_methodology/studio-charter.md`](../studio-charter.md) — operating principles; `§ Tooling` carries the studio-side framing for executor surfaces.
* [`CLAUDE.md`](../../../CLAUDE.md) — repo-root guide; this document fills the previously-empty CLI-conventions slot.

---

## Change history

| Date | Change | Anchor |
|---|---|---|
| 2026-05-10 | Initial publication; nine sections + known gaps + related | `add-cli-standards` ([RAP-228](https://linear.app/rapoport-studio-slr/issue/RAP-228)) |
| 2026-05-12 | Maestro row status → Active; remove "No CI lint" Known Gaps row (shipped as `tools/lint-maestro-cli.mjs`); update §9 lint reference | `add-maestro-cli` Phase 1.6 (RAP-254) |
