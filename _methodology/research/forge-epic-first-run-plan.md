# Forge epic conductor — first-run measurement plan

> **Status:** planning (pre-pilot). Results document follows after the pilot Forge build completes.
> **Opened:** 2026-05-14.
> **Owner:** Pavel Rapoport.
> **Context:** May 2026 home audit batch — seven OpenSpec changes packaged as one Linear epic with seven implement sub-issues. This is the first time `forge epic --close` is exercised on a real multi-sub-issue topology.

---

## Why

`forge epic --close` is the conductor command that picks `Todo + auto-build`-flagged sub-issues from a parent Linear epic, topo-sorts them, and dispatches serially, stopping at the first non-shipping outcome (per repo memory `forge_epic_close_command.md`). It has never been exercised against a real multi-sub-issue epic in this repo. Several known defects in single-build Forge (state.json divergence from reality, builder-orchestrator commit race, verification-gate filter bug) may compound or interact at epic scale. This document defines what we measure, how we collect data, and what the post-run report must contain — authored *before* the run so the measurement plan is honest.

---

## Pilot scope

**Pilot epic:** the audit-batch epic `RAP-<NNN>` (TBD after Linear creation), parent of seven implement sub-issues.

**Pilot sub-issue dispatched first:** `optimize-home-fonts-and-cache` (HFC). Reasons:

- Smallest blast radius (no home block contract changes — infrastructure only).
- No `Q-*` dependencies (no Pavel-input blockers).
- Clear pre/post measurement (asset graph byte delta, Lighthouse LCP, HTTP cache header).
- Verifiable outcome via `curl -I`, no subjective UI judgment.

**Pilot ordering** (if expanded beyond HFC during the first run):

1. HFC — fonts & cache.
2. HSS — SEO baseline.
3. HMA — mobile a11y.
4. HAF — above-fold tighten.
5. HMR — rhythm & density (largest single change).

HMR + HAF share file surface (`hero.tsx`, `closing-cta.tsx`) — Forge's global build-lock serializes them automatically (per `forge_global_build_lock.md`). If parallel dispatch is attempted, the second build fails fast with `BuildLockedError`; expected. HTN + CCP are **blocked** (Q-*) and not dispatched in this pilot.

---

## What we measure

### Per sub-build (each Forge run = one Linear sub-issue)

| Metric | Source | Notes |
|---|---|---|
| Started timestamp | `~/.forge/states/<id>/state.json` `startedAt` | ISO-8601 |
| Ended timestamp | `state.json` `endedAt` | |
| Wall time | computed | total + per-phase |
| Outcome (claimed) | `state.json` `outcome` field | per memory `forge_audit_2026_05_10`, may not match reality |
| Outcome (verified) | `gh pr view <num> --json state,merged,mergedAt,isDraft` | cross-check |
| Outcome divergence | computed | claimed ≠ verified is the critical defect signal |
| Phases run | `state.json` `phases[]` | architect / plan / build / verify / reviewer |
| Token cost per phase | `state.json` `usage[]` | input + output + cache hits |
| Model per phase | `state.json` `model[]` | per-role tier (Opus / Sonnet / Haiku) |
| # replan cycles | `state.json` `replanCount` | architect replanning during build |
| # tool errors | logs grep `tool.*error` | |
| `--no-verify` triggered? | git log of the branch `git log --format=%s --grep='no-verify'` | must be 0 |
| `--auto-*` flags fired correctly | logs grep | --auto-plan, --auto-push, --auto-pr, --auto-migrations |
| `architect` budget cap hit | state.json | per memory `forge_architect_budget_cap`, --auto-plan skips architect; budget unused |
| Builder self-commit detected | `git log --author=Forge` on branch | per memory `forge_builder_orchestrator_commit_race` |
| `aborted-no-changes` outcome despite real PR | computed | the canonical false-positive pattern |
| Verifier failure with stray `--` in filter | logs grep `pnpm.*test.*--.*--` | per memory `forge_verification_gate_filter_bug` |

### Per epic (the `forge epic --close` invocation)

| Metric | Source | Notes |
|---|---|---|
| Topo-sort order observed | logs | does it match the `blocks/blocked-by` graph in Linear? |
| Sub-issue dispatch order | logs | |
| Stop-on-first-non-shipping triggered? | logs + state.json per sub | which sub-issue stopped the train? |
| Build-lock contention events | logs grep `BuildLockedError` | should be 0 if conductor is serial |
| `.git/config.lock` collisions | logs grep `config.lock` | per memory `forge_parallel_dispatch_collisions` |
| Linear bot status updates | Linear MCP `list_comments` on each sub | did Forge correctly transition Todo→In Progress→In Review→Done? |
| Total wall time | epic start → conductor exit | |
| Total token cost | summed across sub-builds | |
| Forge dist build freshness | `.forge/build/` mtime vs `git log -1 HEAD` | per memory `forge_dist_staleness_and_auto_flags` |
| Worktree root vs cwd | `forge.config.mjs` `worktreeRoot` resolved value | per memory `forge_dispatch_repo_root` |

---

## Known defects to watch for

Cited from repo memory (each is a confirmed prior incident):

| ID | Defect | Memory reference | Manifestation to watch |
|---|---|---|---|
| F1 | state.json `outcome` lies about `pr-opened` and `aborted-no-changes` | `forge_audit_2026_05_10.md` | claimed outcome ≠ `gh pr view` reality |
| F2 | Builder self-commit → orchestrator sees clean worktree → false `aborted-no-changes` | `forge_builder_orchestrator_commit_race.md` | state.json says aborted, PR exists |
| F3 | Verification gate orchestrator.ts:354 stray `--` makes vitest run full suite | `forge_verification_gate_filter_bug.md` | verify phase token cost ≫ expected; run time ≫ expected |
| F4 | --auto-plan skips architect role entirely | `forge_architect_budget_cap.md` | architect phase shows 0 cost / time |
| F5 | Background dispatch needs ALL --auto-* flags | `forge_background_dispatch_flags.md` | migrations checkpoint fires → exit 13 on EOF without --auto-migrations |
| F6 | Parallel dispatches collide on build.lock AND .git/config.lock | `forge_parallel_dispatch_collisions.md` | second dispatch fails fast |
| F7 | Forge dist staleness — rebuild before parallel runs | `forge_dist_staleness_and_auto_flags.md` | conductor uses stale Forge binary |
| F8 | tasks.md drift causes RAP-148/RAP-171 no-op pattern | `forge_dispatch_check_actual_state.md` | claimed "task done" but no file changed |
| F9 | Reviewer model normalizes broken behavior | `forge_audit_2026_05_10.md` (Run 2) | reviewer pushes back on intentional fixes |
| F10 | Linear bot may not advance sub-issue status correctly | observed in earlier forensics | sub-issue stuck in In Progress despite merged PR |

For each defect, the post-run report records: did it manifest in this pilot? How was it detected? Was it auto-corrected by Forge or did it require manual intervention?

---

## Data collection plan

### Before dispatch

1. Confirm Forge dist is fresh: `cd packages/forge && pnpm build && git log -1 dist/`.
2. Confirm Linear epic + sub-issues exist with correct `blocks/blocked-by` graph.
3. Confirm scaffold PR merged (so OpenSpec proposals + designs + tasks are on `main`).
4. Pre-pilot baseline measurement (for HFC pilot specifically):
   - `du -sh apps/web/src/app/fonts/` — current asset weight.
   - `curl -I https://rapoport.studio/en` — current cache header.
   - Lighthouse Mobile current production score (capture screenshot).
5. Snapshot `~/.forge/states/` to a baseline (empty or pre-pilot state) for diffing.

### During dispatch

The Forge conductor runs detached in the foreground or background. Logs stream to `~/.forge/logs/<epic-id>/` per the standard pattern.

- Open a second terminal: `tail -F ~/.forge/logs/<epic-id>/conductor.log`.
- Open `gh run watch` if a CI workflow fires.
- Do NOT manually intervene unless a defect blocks indefinitely; the point of the pilot is to observe Forge's behavior, not patch around it.

### After each sub-build completes

For each completed sub-build, capture into the post-run report:

- `cat ~/.forge/states/<build-id>/state.json | jq .` — full state.
- `gh pr view <num> --json title,state,merged,mergedAt,headRefName,baseRefName,labels,reviewDecision`
- `git log --oneline <head>..main` on the build's branch.
- Linear MCP: `list_comments` on the sub-issue + parent epic.

### After conductor exits

- `~/.forge/states/` diff against baseline.
- All conductor logs gzipped to `_methodology/research/forge-epic-first-run-data/<date>/`.
- Linear epic + sub-issue final status snapshot.

---

## Hypotheses to test

H1: **`forge epic --close` correctly topo-sorts the seven sub-issues by `blocks/blocked-by` graph.**

- Expected order: HFC, HSS, HMA in any order (independent) → HAF → HMR (HMR has soft dependency on HAF) → HTN + CCP skipped (Blocked status).
- Validation: log order matches expected partial order.

H2: **state.json `outcome` lies for at least one sub-build (F1, F2 manifest).**

- Expected: high probability per memory.
- Validation: outcome ≠ `gh pr view` for ≥ 1 sub-build.

H3: **Build-lock serialization works correctly across sub-builds.**

- Expected: no parallel-dispatch collisions; each build completes before next starts.
- Validation: build start times don't overlap.

H4: **Verifier filter bug (F3) is no longer present in current Forge build.**

- Expected: unknown — depends on whether RAP-XXX (the verifier fix) has shipped since the last memory update.
- Validation: verify phase token cost and wall time within expected range for the test surface size.

H5: **Linear bot correctly advances sub-issue statuses.**

- Expected: Todo → In Progress (at dispatch) → In Review (at PR open) → Done (at merge).
- Validation: Linear timeline per sub matches.

H6: **`--auto-pr` actually opens a PR** (per memory `forge_auto_push_does_not_push.md` which is STALE pre-RAP-392; auto-push is now real).

- Expected: PR opens.
- Validation: `gh pr view` returns the PR after Forge completes.

H7: **The pilot reveals at least one previously-undocumented defect.**

- This is the qualitative test — we expect the pilot to surface something unexpected. The value of running the pilot is partly in finding what the memory doesn't already say.

---

## Schema for the post-run report

The post-run report lives at `openspec/_methodology/research/forge-epic-first-run-results.md` (new file, written after the pilot completes). Structure:

```
# Forge epic conductor — first-run results

> Pilot date: <YYYY-MM-DD>
> Pilot epic: RAP-<NNN>
> Sub-issues attempted: <list>
> Sub-issues completed successfully: <list>
> Sub-issues failed / blocked: <list with reasons>

## Summary

<2-3 paragraphs: what shipped, what broke, headline numbers>

## Per-sub-build details

[One section per sub-issue, with the metrics table populated]

## Defects observed (vs the F1–F10 watch list)

| Defect | Manifested? | Detection method | Auto-corrected? | Filed as Linear issue? |
|---|---|---|---|---|

## New defects (not in F1–F10)

[Anything previously undocumented]

## Hypotheses test results

[H1–H7 with confirmed / refuted / inconclusive]

## Recommendations

[What to fix in Forge before next epic dispatch]

[What to fix in the OpenSpec tasks.md template based on what Forge struggled with]

[What to document in repo memory based on new findings]

## Artifact links

- Scaffold PR
- Per-sub implementation PRs
- Linear epic
- Conductor logs (gzipped, in this folder)
- state.json snapshots
```

---

## Out of scope

- **Fixing Forge defects discovered during the pilot.** Forge improvements are separate changes; this run documents, doesn't patch.
- **Comparing Forge epic mode vs single-build mode performance.** That's a follow-up benchmark.
- **Cost / quality benchmarking across Anthropic model tiers.** Out of scope; existing tiering decisions (RAP-213 per memory) stand.
- **A11y / perf / SEO measurement of the resulting PRs.** The home-audit batch has its own per-change verification; the Forge run produces those PRs. The methodology here measures *Forge's behavior*, not the home's quality.
- **Cross-vendor benchmarking** (Forge vs Cursor vs Claude Code direct vs other agentic-build tools). Out of scope.
- **Statistical significance.** N=1 pilot; this is exploratory, not hypothesis-testing in a statistical sense. The hypotheses above are qualitative pass/fail.

---

## Update protocol

After the pilot completes, the post-run results document gets written. This planning doc may be updated to reflect:

- Defects that did manifest get added to repo memory if they weren't already there.
- Hypotheses that were refuted go into Forge-improvement Linear issues.
- Schema changes for the next epic measurement get recorded as `Q-FE-N` open questions for the next iteration's plan.

The planning doc itself is **not** edited retroactively to look like the results — both files coexist in `_methodology/research/`.
