# Inquiry — Forge runtime audit, 2026-05-18

> **Status:** published
> **Lead:** human:pavel
> **Owner:** Pavel Rapoport <pavel@rapoport.studio> (founder, primary)
> **Authors:** Pavel Rapoport <pavel@rapoport.studio> (primary, audit author)
> **Author_ids:** human:pavel
> **Kind:** internal
> **Opened:** 2026-05-18
> **Capability_refs:** forge
> **Thesis:** Forge's 2026-05-18 wasted-run rate (76% of 17 terminal runs producing no shippable work) is dominated by input-quality and outcome-honesty defects, not engine surgery requirements.
> **Method:** Behavioural runtime audit of 19 `forge-build.yml` CI runs dispatched 2026-05-18, reconstructed from `~/.forge/state/*.json`, plan files, `logs/**`, events `*.jsonl`, and `forge-output.json` artifacts. Each run classified by reported outcome vs real outcome; defects extracted one per failure signature with severity, evidence (run id + log line), and `file:line` cause.
> **Reusable for:** `add-github-actions-control-plane` Phase 2b pre-flight design; future Forge outcome-taxonomy work; operator runbooks.
> **Companion to:** [`inquiry-drift-coverage-retrieval-spike.md`](./inquiry-drift-coverage-retrieval-spike.md) (this file is the F-12 convention-adoption test target).
> **Renaming note (2026-05-29):** previously `forge-runtime-audit-2026-05-18.md` — renamed under inquiry-* convention as the F-12 / F-3-fix test in [`inquiry-drift-coverage-retrieval-spike.md § 5.4`](./inquiry-drift-coverage-retrieval-spike.md). Content below unchanged from the legacy version; only frontmatter prepended.

---

A behavioural (not static) audit of the Forge build engine, reconstructed from
the **19 `forge-build.yml` CI runs dispatched on 2026-05-18**. This is the
runtime counterpart to the static `forge audit forge` command and to the
2026-05-10 forensic pass (`forge-audit-2026-05-10` / `forge_audit_2026_05_10`
memory note).

## How we run this analysis (methodology)

1. **Dataset = CI run logs + artifacts.** `forge-build.yml` already uploads
   `~/.forge/state/*.json`, `plans/*.md`, `logs/**`, the events `*.jsonl`, and
   `forge-output.json` as a run artifact (30-day retention). Each run is one
   forensic data point. Pull logs with `gh run view <id> --log` and artifacts
   with `gh run download <id>`.
2. **Classify every run** by *reported* outcome (`success`/`failure`/
   `cancelled`) **and** by *real* outcome (shipped real work / no-op / stale /
   crashed). The gap between the two is itself a finding.
3. **Extract distinct defects** — one per failure signature — with severity,
   evidence (run id + log line), and a `file:line` where the cause is in
   Forge source.
4. **Triage known vs new** — cross-reference the `_methodology/research/forge-*`
   files and the Forge memory notes. Known-and-unfixed defects still count.
5. **Feed back** — new HIGH/MED defects become Linear issues or fold into the
   `add-github-actions-control-plane` change.

Cadence: run this pass after every batch dispatch (≈ daily while Forge is the
primary executor).

## Dataset — 19 runs (2026-05-18)

| Run | Issue | Reported | Wall | Real outcome |
|---|---|---|---|---|
| 26011325274 | RAP-998 | failure | 1m | Sonnet planner API error |
| 26011327984 | RAP-984 | failure | 1m | Sonnet planner API error |
| 26011866617 | RAP-998 | cancelled | 90m | docker-sandbox stall → timeout |
| 26011867304 | RAP-999 | success | 11m | no-op PR (regen churn only) |
| 26011868034 | RAP-984 | success | 21m | **real PR #1402** |
| 26011868929 | RAP-985 | success | 38m | **real PR #1404** |
| 26011869661 | RAP-822 | failure | 1m | stale — change archived |
| 26011870391 | RAP-823 | failure | 1m | stale — change archived |
| 26011871194 | RAP-315 | success | 10m | **real PR #1397** |
| 26011871953 | RAP-676 | success | 13m | no-op PR (regen churn only) |
| 26011872633 | RAP-479 | failure | 1m | stale — change archived |
| 26011873459 | RAP-552 | success | 20m | no-op PR (sandbox-label path) |
| 26013853411 | RAP-319 | success | 11m | aborted-no-changes (work already in main) |
| 26014462521 | RAP-998 | success | 23m | tests-only PR #1409 (host sandbox) |
| 26015103122 | RAP-1031 | failure | 26m | spec-acceptance gate key mismatch |
| 26015103885 | RAP-1032 | success | 10m | aborted-no-changes |
| 26015104822 | RAP-1033 | running | — | — |
| 26015439045 | RAP-999 | success | 9m | aborted-no-changes |
| 26016538325 | RAP-1030 | running | — | — |

### Reported vs real

- Reported: **11 success / 6 failure / 1 cancelled / 2 running**.
- Real: of the 11 "success", only **~4 shipped real mergeable work** (RAP-984,
  RAP-985, RAP-315, and RAP-998-as-tests). The other ~7 were no-op /
  aborted-no-changes. **"success" overstates value by ~2.5×.**
- Low/zero-value runs: ~7 no-op + 3 stale + 2 Sonnet-fail + 1 timeout ≈ **13 of
  17 terminal runs (76%) produced no shippable work.** The 90-minute timeout
  run alone burned a full runner-hour.

## Defects

### D-1 — Sonnet planner path returns an API-level error · HIGH · partly-known
`models.architect: 'sonnet'` (`claude-sonnet-4-6`) makes plan generation fail
instantly: `PlanGenerationError: Plan generation stopped with reason "error"`
(runs 26011325274, 26011327984). Opus plans reliably. Origin:
`plan-builder.ts:158-159` rethrows a `StructuredOutputError` whose
`stopReason` is `"error"`. Worked around by reverting the default to Opus
(#1396); the Sonnet plan path itself is still unusable and undiagnosed —
either the model id is wrong or Sonnet rejects the structured-output request
shape.

### D-2 — Docker sandbox stalls on CI runners · HIGH · known
`sandbox.provider: 'docker'` on a GitHub runner: the container starts
(`[SANDBOX] container forge-RAP-998 started`) and then the builder agent is
never exec'd — 81 minutes of total silence until `timeout-minutes: 90` cancels
the job (run 26011866617). The `iptables` network setup also fails on every
run (`Permission denied (you must be root)`) but is non-fatal. Worked around
with `--sandbox host` in `forge-build.yml` (#1406) — the ephemeral runner VM
is itself the isolation boundary. The docker provider remains broken for
CI/ephemeral environments. Memory: `forge_docker_sandbox_stalls_2nd_build`.

### D-3 — `aborted-no-changes` is an ambiguous terminal outcome · HIGH · new
4 of 17 terminal runs ended `aborted-no-changes` (RAP-319, RAP-552, RAP-999,
RAP-1032). The outcome conflates three very different situations:
(a) the work genuinely already exists in `main`; (b) the builder failed to do
the work (e.g. a prose/copy task it cannot perform — D-8); (c) the
builder-commit race (memory `forge_builder_orchestrator_commit_race`). An
operator — or the future conductor — cannot tell which without a manual dig,
and discovering it costs a full 10–20-minute build. Forge should classify the
no-changes cause, or run a cheap pre-dispatch check (D-10).

### D-4 — Stale-change error misattributes the cause · MED · new
When a change folder has been archived, the planner aborts with
`Cannot plan RAP-NNN: no openspec/changes/<slug>/ reference in issue body`
(`plan/orchestrator.ts:102`; runs 26011869661/70391/72633). The message is
wrong — the issue body *does* contain the reference; the real cause is that
the folder lives in `openspec/archive/`, not `openspec/changes/`. The message
should say "change folder not found under openspec/changes/ — it may already
be archived" and name the archive path if present.

### D-5 — Spec-acceptance gate hardcodes the originating issue key · MED · new
`add-canvas-preview-mode/tasks.md` bakes `Done when` assertions referencing
`RAP-1006` and the branch `feat/rap-1006-canvas-preview-mode`. A build
dispatched under a *different* issue key (RAP-1031) fails the spec-acceptance
gate (`SpecAcceptanceFailedError: 6 assertion(s) failed`, run 26015103122)
purely on key mismatch — the branch is `feat/rap-1031-…` and `RAP-1031.json`,
not the spec-baked `RAP-1006`. The gate should resolve assertions against the
*current* run's issue key / branch, or `tasks.md` should template the key
rather than hardcode it.

### D-6 — `tech-debt` / audit labels silently switch to sandbox planning · MED · known
RAP-552 carries the `tech-debt` label → the `SANDBOX_LABELS` heuristic
auto-enables sandbox planning, which does not require an `openspec/changes/`
folder. So instead of failing like the other stale issues, it planned from the
issue body and built a no-op PR. Silent mode-switch on a label is a footgun.
Memory: `forge_sandbox_labels_trap`.

### D-7 — No-op builds still open PRs of pure regeneration churn · MED · new
RAP-999/676/552 produced PRs whose entire diff was regenerated generated
files (`llms-full.txt`, `INDEX.md`) — no real change. The builder commits
`pnpm install` postinstall churn as "the change". Wastes a PR, a CI run, and
review attention. (#1403 made those files deterministic so the churn no longer
*conflicts*, but the empty-PR behaviour remains.) A build whose only diff is
generated files should report `aborted-no-changes` and open no PR.

### D-8 — Forge cannot perform prose / copy-authoring tasks · LOW · new
RAP-999 ("Phase 2 — Copy authoring (per-locale)") returned `aborted-no-changes`
on **both** attempts (#1398 and run 26015439045). Forge is a code-builder;
"author per-locale marketing copy" is outside its competence. This is a
capability boundary, not a code bug — but such phases should not carry the
`auto-build` label, or Forge should refuse them explicitly rather than no-op.

### D-9 — Outcome vocabulary does not separate "shipped" from "did nothing" · MED · new
`completed-with-pr` was reported for RAP-998's tests-only PR #1409 (no
implementation, just test files) and the gate `passed`. Technically true, but
an operator scanning outcomes cannot distinguish a full implementation from a
partial or a no-op. Forge needs an outcome that reflects *delivered scope*, not
just *a PR exists*.

### D-10 — No pre-dispatch validation · MED · new
Forge will happily start a full build against an archived change, a duplicate
issue, or already-completed work, and burn the whole build to discover there
was nothing to do. A cheap pre-flight — "does `openspec/changes/<slug>/` exist;
does `tasks.md` have unchecked tasks; is there already a merged PR for this
slug" — would convert most of today's 13 wasted runs into instant skips. This
is exactly the gap the `add-github-actions-control-plane` conductor pre-check
(Phase 2b) should close.

## Scorecard

| Dimension | Reading |
|---|---|
| Plan generation (Opus) | reliable |
| Plan generation (Sonnet) | **broken** (D-1) |
| Sandbox (docker, CI) | **broken** (D-2) |
| Sandbox (host, CI) | works |
| Real-work yield | **~4/17 terminal runs (~24%)** |
| Outcome honesty | poor — "success" and `completed-with-pr` overstate (D-3, D-9) |
| Wasted-run rate | **~76%** (stale / no-op / crash) — almost entirely an *input-quality* problem, not an engine crash problem |
| Cost guard | now present — `budgets.build $15` (#1405) |

**Headline:** the engine *core* (plan→build→PR on Opus + host sandbox) works.
The failures are overwhelmingly **input-quality and outcome-honesty** problems:
Forge faithfully runs whatever issue it is handed, including archived,
duplicate, already-done, and prose-typed work, and reports a cheerful
`success` either way. The fix is a pre-dispatch validation layer (D-10) plus
honest outcome classification (D-3, D-9), not engine surgery.

## Findings

> Structured index of the defects above, added 2026-05-28 as part of
> [`inquiry-drift-coverage-retrieval-spike`](./inquiry-drift-coverage-retrieval-spike.md)
> Phase 2 substitute experiment. Grade vocabulary per `add-inquiry-entity` design.md §3
> `InquirySource.credibility` enum: `experimental` | `engineering_claim` | `marketing` | `discourse`.
>
> **Capability:** forge

| # | Claim | Grade | Falsifier |
|---|---|---|---|
| F-1 | Sonnet planner path (`models.architect: 'sonnet'` → `claude-sonnet-4-6`) returns API-level error immediately; Opus plans reliably. | engineering_claim | A Sonnet plan against an unchanged prompt-construction code path succeeds on a single fresh dispatch. |
| F-2 | Docker sandbox provider stalls indefinitely on GitHub Actions runners (container starts, agent never exec'd); 90-min timeout cancels the job. `--sandbox host` works because the runner VM itself is the isolation boundary. | engineering_claim | A `--sandbox docker` dispatch on a fresh GH Actions runner completes without timeout. |
| F-3 | `aborted-no-changes` is an ambiguous terminal outcome conflating ≥3 distinct causes (work already in main / builder declined / builder-commit race). Operators cannot disambiguate without full log dig. | engineering_claim | Forge ships a finer-grained outcome taxonomy (`shipped-real-work` / `no-op-already-done` / `no-op-builder-declined` / etc.) and runs without the ambiguous terminal state appearing. |
| F-4 | Stale-change error misattributes cause: planner says "no `openspec/changes/<slug>/` reference in issue body" when the real cause is the folder being archived. | engineering_claim | Planner runs against an archived slug and the error message names the archive path. |
| F-5 | Spec-acceptance gate hardcodes the originating issue key in `tasks.md` "Done when" assertions; dispatching under a different issue key fails purely on key mismatch. | engineering_claim | Gate resolves assertions against the *current* run's issue key/branch and the cross-key dispatch passes acceptance. |
| F-6 | `SANDBOX_LABELS = ['tech-debt', 'audit', 'bug']` heuristic silently switches OpenSpec-backed issues to sandbox planning when spec-folder detection fails. Silent mode-switch on label is a footgun. | engineering_claim | Label-AND-no-spec-folder logic gates the auto-switch; an OpenSpec-backed issue with `audit` label and visible spec folder does NOT switch to sandbox. |
| F-7 | No-op builds open PRs whose entire diff is regenerated generated files (`llms-full.txt`, `INDEX.md`); pure regen churn wastes PR + CI + review attention. | engineering_claim | A build whose only diff is generated artifacts reports `aborted-no-changes` and opens no PR. |
| F-8 | Forge has no pre-dispatch validation against archived slugs / duplicate issues / already-completed work; burns full 10-20-min builds to discover there was nothing to do. ~76% of 2026-05-18's terminal runs produced no shippable work. | engineering_claim | A pre-flight check ships and the post-deploy wasted-run rate drops below 20% over a comparable 19-run window. |
| F-9 | Outcome vocabulary does not separate "shipped" from "tests-only" from "did nothing"; `completed-with-pr` reports the same way for full implementation and tests-only PR. | engineering_claim | Forge emits distinct outcomes for delivered scope vs. PR-existence; operator dashboard surfaces the distinction. |
| F-10 | Forge cannot perform prose/copy-authoring tasks (RAP-999 returned `aborted-no-changes` twice on a "Phase 2 — Copy authoring" sub-issue). This is a capability boundary, not a code bug. | engineering_claim | Forge ships an explicit capability filter that refuses prose-typed tasks before dispatch (or auto-build label excludes prose phases). |

## Recommended actions

1. **D-10 pre-dispatch validation** — fold into `add-github-actions-control-plane`
   Phase 2b (the conductor's discovery pre-check). Highest leverage: kills the
   76% waste rate.
2. **D-3 + D-9 outcome taxonomy** — distinguish `shipped-real-work` /
   `no-op-already-done` / `no-op-builder-declined` / `partial`. Small,
   high-value engine change.
3. **D-4 error-message fix** — one-line clarity fix in `plan/orchestrator.ts`.
4. **D-5 spec-acceptance key resolution** — gate should use the run's issue
   key, not the `tasks.md`-baked one.
5. **D-1 Sonnet plan path** — diagnose separately; until then Opus stays the
   planner default.
6. **D-2 docker sandbox** — `--sandbox host` is the standing CI setting;
   docker provider needs a real fix before it is trusted on ephemeral runners.
7. **D-7 / D-8** — Forge should not open generated-files-only PRs, and
   prose-authoring phases should not be `auto-build`.
