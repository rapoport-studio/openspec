<!-- openspec-refs: allow-unresolved -->
# Forge & OpenSpec Index Audit — 2026-05-17

> Global research pass: are the studio's indexes, docs, and infra "as they
> should be"? Commissioned after the Claude Platform adoption epic (Phases
> A/B) shipped, to confirm everything installed is documented and correctly
> indexed — for both human and AI operators.
>
> This file deliberately cites a few broken/legacy paths as findings; the
> `openspec-refs: allow-unresolved` marker above keeps the refs guard green.

## Scope & method

Four areas, audited 2026-05-17:

1. **OpenSpec indexes** — `openspec/_root/INDEX.md` routing index, the
   `openspec:*` regen toolchain, capability-spec coverage, broken links.
2. **Cloudflare infra** — KV / R2 / AI / D1 / Vectorize bindings and routes
   on the `rapoport-web` and `rapoport-studio` Workers.
3. **Postgres indexes** — index coverage on the Supabase database
   (`nifagnmgwoqkplegsicy`) vs. observed query patterns.
4. **Forge operating docs** — whether `openspec/current/forge/` accurately
   describes how to run Forge, which platforms/ecosystems it targets, and
   how its metrics work.

Method: three parallel read-only sub-agent audits over the repo + live
queries against Cloudflare (`wrangler.jsonc`) and Postgres (`pg_stat_*`,
`pg_constraint`, `pg_index`).

## Headline result

| Area | Verdict |
|---|---|
| OpenSpec INDEX.md | **Severely broken** — 3 of 50 capability specs indexed |
| Claude Platform doc-coverage | **Good** — all 7 shipped features documented; 1 minor gap |
| Cloudflare infra | **Healthy** — KV/R2/AI bound; no D1/Vectorize in use (none needed) |
| Postgres indexes | **Hygiene gap** — 22 foreign keys lack a covering index |
| Forge operating docs | **Drifted** — flag tables, event list, README all stale |

## 1. OpenSpec indexes — SEVERELY BROKEN

`openspec/_root/INDEX.md` is the capability routing index AI agents consult
first. It lists **3 capabilities** (Auth, cli-core, Platform). `openspec/current/`
holds **50 spec files**.

Root cause: `tools/build-openspec-index.mjs` only includes files that carry a
`title` YAML frontmatter field. **47 of 50 specs have no frontmatter at all**
and are silently skipped — 94 % of the studio's capabilities are invisible to
the routing index.

Contributing factors:

- `tools/lint-frontmatter.mjs` enforces frontmatter (`title`, `status`,
  `owner`, `audience`, `tldr ≤200 chars`) but runs **warn-only / exit 0**
  ("Phase A mode"). Its own header notes "Phase C flips to exit 1" — that flip
  never happened, so the gap accumulated unchecked.
- `INDEX.md` and `apps/web/public/llms-full.txt` are **committed but generated**.
  `postinstall` regenerates the bundle + search index but **not** the INDEX or
  llms-full — so they only refresh on a manual `pnpm openspec:regen`, and
  drifted stale.
- The refs guard (`tools/check-openspec-refs.mjs`) scans `openspec/current/`
  and `openspec/_methodology/` but **not** the root `CLAUDE.md` — the
  single most-read agent file in the repo.

Broken references found:

- `CLAUDE.md` (Package-boundaries table + "Active change" footer) cites
  `openspec/changes/2026-04-25-platform-foundation/` — that change folder no
  longer exists (the epic shipped and was archived). The file presents a
  fully-shipped epic as in-flight.

## 2. Claude Platform doc-coverage — GOOD

All 7 features from the Phase A/B adoption epic are **in code**, **archived**
as applied changes, and **documented in the correct `openspec/current/`
capability spec**:

| Feature | Current spec |
|---|---|
| Effort + adaptive thinking | `forge/spec.md` |
| Structured outputs (`callForgeStructured`) | `forge/spec.md` |
| 1-hour prompt caching | `forge/spec.md` |
| Memory tool / adapter | `forge/spec.md` |
| Files API | `canvas.md` |
| Search results / citations | `canvas.md` |
| Server-side compaction + context editing | `forge/spec.md` |

The archive→current-spec-update discipline held for every change. One minor
gap: the Files API proposal promised an update to
`openspec/current/studio-workspace/narrative.md` (intake-ingest pipeline now
uploads client briefs/PDFs once via the Files API wrapper, referenced by
`file_id`) — that spec update did not land.

## 3. Cloudflare infra — HEALTHY

- `rapoport-studio` Worker: KV `NEXT_INC_CACHE_KV`, Workers-AI binding,
  R2 bucket `rapoport-tg-audio`. `account_id` set. Custom domain
  `app.rapoport.studio`.
- `rapoport-web` Worker: KV `NEXT_INC_CACHE_KV` + `OPERATING_MODEL_STATS_KV`,
  nightly cron `0 3 * * *` with entrypoint `apps/web/workers/operating-model-stats.ts`
  (present). Custom domain `rapoport.studio`.
- **No D1 and no Vectorize** are provisioned — and none are needed. The
  studio's "search index" is `tools/openspec-index.json`, a build-time
  artifact bundled into the Worker, not a Cloudflare-side index. There is
  nothing to "set up" on the Cloudflare side.
- Minor: `apps/web/wrangler.jsonc` omits `account_id` (the studio config has
  it). Deploys succeed because `deploy-web.yml` supplies the account via env —
  but the asymmetry is a latent foot-gun. Low priority.

## 4. Postgres indexes — 22 UNINDEXED FOREIGN KEYS

`pg_stat_user_tables` shows high `seq_scan` counts on several tables, but
almost every table is tiny (0–80 live rows) — Postgres *correctly* picks a
sequential scan for tiny relations, so the raw seq-scan count is not itself a
problem.

The genuine gap is **22 foreign-key constraints with no covering index** on
the referencing column(s). Unindexed FKs slow joins, slow `ON DELETE`
cascades, and force a full child-table scan when a referenced parent row is
deleted. The tables are small today so the impact is currently negligible —
this is a **hygiene fix** to apply before they grow, not a live incident.

Affected (referencing column in parens): `ai_files` (project_id, owner_id),
`canvas_excluded_members` (excluded_by), `canvas_files`
(last_emitted_by_message_id, source_canvas_id), `canvas_ideas` (created_by),
`canvas_invitees` (invited_by), `canvas_messages` (author_id),
`canvas_proposals` (resolved_by_studio_user_id), `canvas_sessions`
(signoff_by_user_id), `canvas_stage_events` (triggered_by_user_id),
`canvas_tool_uses` (message_id), `certified_applications` (reviewed_by),
`channels` (added_by_user_id), `connection_events` (actor_id), `forge_runs`
(created_by), `inbox` (canvas_id_if_accepted, reviewed_by), `project_members`
(invited_by), `studio_prompt_templates` (created_by), `tg_invite_tokens`
(created_by_user_id), `tg_user_bindings` (bound_via_token).

## 5. Forge operating docs — DRIFTED

`openspec/current/forge/commands.md` and `spec.md` describe a Forge that no
longer matches `packages/forge/src/cli.ts`:

- **`forge build` flag table is stale** — missing `--wait-lock[=N]`,
  `--force`, `--auto-approve-plan`, `--use-stale-plan`, `--effort`. The
  `--sandbox` value is documented space-separated (`--sandbox docker`) but the
  parser only accepts `--sandbox=docker`; the documented form is silently
  ignored.
- **`forge epic` subcommands undocumented** — only `--close` is documented;
  the bare one-shot conductor, `--status`, `--validate`, `--watch` are not.
- **`forge dispatch` / `forge tail` are referenced but do not exist** in the
  CLI command whitelist (`spec.md` MCP-tool list, `README.md` routing table).
- **GitHub Actions execution is undocumented** — `.github/workflows/forge-build.yml`
  exists and works, but no `openspec/current/forge/` doc mentions headless
  Actions runs, the `FORGE_BOT_TOKEN` secret-naming gotcha, or how to onboard
  a new target repo / ecosystem.
- **Event/metrics docs incomplete** — `spec.md § Event emission` lists 4
  event kinds; code emits a 5th (`acceptance.gate`) plus `memory_tool_*`
  events. `runId` is documented as `forge-<key>-<cmd>-<ts>`; code uses a bare
  UUID. The rich `BuildMetrics` schema (`core/metrics.ts`) persisted to
  `~/.forge/builds/<RAP>/metrics.json` and `~/.forge/metrics/index.jsonl` is
  not documented at all — there is no operator guide for reading or A/B-ing
  Forge cost/token metrics.
- **Model-tiering vs. effort not reconciled** — `spec.md` still presents the
  RAP-213 manual per-role tiering as current; the newer `--effort` layer is
  documented separately and the two are never reconciled. `spec.md` also
  contradicts itself on architect `max_tokens` (8000 vs 65536).
- **`README.md` stale** — claims "Thirteen commands" (18 exist) and lists
  "orchestrated review not yet implemented" (it shipped).

## Effort A/B addendum (RAP-1009)

A side experiment during this audit — `forge plan RAP-951` at `--effort=high`
vs `--effort=xhigh` — surfaced a real bug: at `effort=high` the planner
deterministically wrapped a valid plan in a stray `$PARAMETER_NAME` tool-input
placeholder key, failing schema validation. RAP-1009's blind retry could not
clear a deterministic failure. Fixed (#1375) with `unwrapPlaceholderPayload`.
Post-fix A/B: `high` produces an equivalent 8-step plan for ~24 % fewer output
tokens (2 764 vs 3 628) and ~4.6 % lower cost ($1.33 vs $1.40). Conclusion:
`high` is a viable, slightly cheaper planner effort — *with the unwrap fix*.

## Findings

> Structured index added 2026-05-28 as part of
> [`inquiry-drift-coverage-retrieval-spike`](./inquiry-drift-coverage-retrieval-spike.md)
> Phase 2 substitute experiment. Grade vocabulary per `add-inquiry-entity` design.md §3
> `InquirySource.credibility` enum.
>
> **Capability:** forge

| # | Claim | Grade | Falsifier |
|---|---|---|---|
| F-1 | `openspec/_root/INDEX.md` lists 3 of 50 capability specs (94% invisibility). Root cause: `tools/build-openspec-index.mjs` includes only files with `title` YAML frontmatter; 47 of 50 have none and are silently skipped. | engineering_claim | Adding frontmatter to one of the 47 unindexed specs causes it to appear in the next INDEX regen. |
| F-2 | `tools/lint-frontmatter.mjs` is warn-only (`exit 0`, "Phase A mode") despite its own header noting "Phase C flips to exit 1" — the flip never happened, so the gap accumulated unchecked. | engineering_claim | Flipping the lint to `exit 1` produces a non-zero exit on the 47 unindexed specs in CI. |
| F-3 | `INDEX.md` and `apps/web/public/llms-full.txt` are committed-but-generated, and `postinstall` regenerates only the bundle + search index, not these two — they only refresh on manual `pnpm openspec:regen` and have drifted stale. | engineering_claim | A spec edit followed by `pnpm install` does NOT update INDEX.md or llms-full.txt (current behavior); adding them to postinstall would. |
| F-4 | Refs guard (`tools/check-openspec-refs.mjs`) scans `openspec/current/` and `openspec/_methodology/` but NOT `CLAUDE.md` — the single most-read agent file. CLAUDE.md's broken `platform-foundation` reference accumulated unnoticed. | engineering_claim | Extending refs guard to CLAUDE.md flags the broken ref in next CI run. |
| F-5 | Claude Platform doc-coverage is good: all 7 Phase A/B features are in code, archived, and documented in correct capability specs. Only minor gap: Files API spec update for `studio-workspace/narrative.md` (intake-ingest pipeline) did not land. | engineering_claim | Reading `studio-workspace/narrative.md` shows no Files-API-wrapper / `file_id` references; landing the missed update would close this. |
| F-6 | Cloudflare infra is healthy: KV + R2 + Workers-AI bound; no D1/Vectorize provisioned because the studio's "search index" is a build-time `tools/openspec-index.json` artifact, not a Cloudflare-side index. | engineering_claim | A future need for runtime vector search would require D1/Vectorize and falsify the "healthy without them" claim. |
| F-7 | Postgres has 22 foreign-key constraints without covering indexes on referencing columns. Hygiene-grade today (tables are small), but slows joins / cascades as they grow. | engineering_claim | A migration adding the 22 covering indexes runs cleanly and removes the gap from `pg_index` inventory. |
| F-8 | `forge/commands.md` flag table is stale (missing `--wait-lock`, `--force`, `--auto-approve-plan`, `--use-stale-plan`, `--effort`); `--sandbox` is documented space-separated but parser only accepts `--sandbox=docker`; documented form silently ignored. | engineering_claim | Running `forge build --sandbox docker` (space form) succeeds; current behavior shows it does not. |
| F-9 | `forge epic` subcommands `--status`, `--validate`, `--watch` and bare conductor are undocumented; only `--close` is. Operators have no reference for the other modes. | engineering_claim | Documentation regen surfaces the missing subcommands; current docs grep does not find them. |
| F-10 | `forge dispatch` / `forge tail` referenced in `spec.md` MCP-tool list and `README.md` routing table do not exist in the CLI command whitelist — broken documentation surface. | engineering_claim | `forge dispatch --help` returns non-zero / "command not found"; current state confirms. |
| F-11 | Effort A/B experiment: `--effort=high` produces equivalent 8-step plans for ~24% fewer output tokens (2 764 vs 3 628) and ~4.6% lower cost ($1.33 vs $1.40) — *only after* the `unwrapPlaceholderPayload` fix in PR #1375. Prior to fix, `high` deterministically wrapped plans in stray `$PARAMETER_NAME` tool-input placeholder. | engineering_claim | A `--effort=high` plan on a different epic produces non-equivalent step count or higher token cost, refuting generality. |

## Prioritized fix plan

| # | Fix | Area | Effort |
|---|---|---|---|
| 1 | Add frontmatter to 47 `openspec/current/` specs; regen INDEX (3→50) | 1 | L |
| 2 | Flip `lint-frontmatter.mjs` to exit 1; add index regen to `postinstall`; extend refs guard to `CLAUDE.md` | 1 | S |
| 3 | Fix `CLAUDE.md` broken `platform-foundation` refs + stale Infisical project ID | 1 | S |
| 4 | Rewrite `forge/commands.md` flag tables; document `forge epic` subcommands; fix `--sandbox` syntax | 5 | M |
| 5 | Update `forge/spec.md` event list (`acceptance.gate`, `memory_tool_*`), `runId` format; add a "Reading metrics" section; reconcile model-tiering vs effort | 5 | M |
| 6 | Add a GitHub Actions / ecosystem-launch operator runbook | 5 | M |
| 7 | Fix `forge/README.md` command count + stale gap line | 5 | S |
| 8 | Migration: add covering indexes for the 22 unindexed FKs | 3 | S |
| 9 | Add Files API intake section to `studio-workspace/narrative.md` | 2 | S |

## Outcome of this pass

This document is the audit artifact. Fixes #1–#9 are applied in follow-up
PRs referenced from this file's commit history. Cloudflare infra needed no
change (area 2 verdict: healthy).
