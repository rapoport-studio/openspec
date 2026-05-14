# Forge epic conductor — first-run results

> **Pilot date:** 2026-05-14.
> **Pilot epic:** [RAP-744](https://linear.app/rapoport-studio-slr/issue/RAP-744) — Home audit & mobile-first fixes (May 2026).
> **Pilot sub-issue:** [RAP-746](https://linear.app/rapoport-studio-slr/issue/RAP-746) — Implement #1 optimize-home-fonts-and-cache (HFC).
> **Outcome:** Build did not ship. Three confirmations + one previously-undocumented defect surfaced.
> **Companion:** measurement plan at [`forge-epic-first-run-plan.md`](./forge-epic-first-run-plan.md).

---

## Summary

The pilot exercised `forge epic --close RAP-744 --auto-plan --auto-migrations --auto-push --auto-pr` three times against a single Todo+auto-build child (RAP-746). All three runs failed before any build worktree was created. The first two failures were operator-corrective (label hygiene + primary-worktree branch state) and exercised the auto-sandbox fallback path that memory already documents at the conceptual level (`forge_dispatch_repo_root.md`); the third failure is a **previously-undocumented defect in Forge's plan-prompt construction** that produced a one-character (`' '`) user message and was rejected by the Anthropic API with HTTP 400.

The conductor itself behaved as designed end-to-end: topo-sort correct, status filtering correct, label filtering correct, no-state-on-early-failure semantics correct, stop-on-first-failure correct. The conductor is shippable as a coordinator; the dependent plan pipeline has a regression.

---

## What worked

**Conductor topology (H1).** Across all three retries the conductor reported the same skip line for each non-Todo sub-issue:

```
⏭️  RAP-752 — skipped (state=Backlog (only Todo dispatch); missing label `auto-build`)
... (six more, in deterministic order)
⏭️  RAP-745 — skipped (state=Done (only Todo dispatch); missing label `auto-build`)
```

Only RAP-746 (Todo + `auto-build`) was selected. The reported reason strings are clear and machine-parseable. **Hypothesis H1 confirmed.**

**Validator catches body-format drift.** `forge epic RAP-744 --validate` produced 25 warnings (0 errors) before any dispatch attempt. Three required sub-issue body sections — `## Scope`, `## Wave`, `## Spec source` — were flagged as missing across all eight sub-issues. After updating RAP-746's body to the canonical format (Phase/Wave/Spec-source sections), its warnings cleared on re-validate (22 → 22 − 3 = 19 specific to RAP-746). The other six sub-issues retain their warnings as expected since they're Backlog and won't dispatch this round. **Validator is a useful pre-dispatch gate.**

**No-state-on-early-failure semantics.** All three failures aborted before `~/.forge/states/RAP-746/` was created and before `~/.forge/worktrees/RAP-746/` was created. `forge status RAP-746` returned `(no build state for RAP-746)` after each failure. This means there is **no leaked state** to clean up between retries; the operator can simply re-dispatch. **This is the correct UX for early-failure paths.**

**Stop-on-first-failure works.** With one Todo+auto-build child, "first failure" was the only failure and the conductor exited cleanly with status 3 after the run. The summary line `Dispatched: 1, shipped: 0, failed: 1, skipped: 7` is accurate. **Hypothesis H3 not strongly exercised (only one sub dispatched) but conductor exited cleanly each time.**

---

## What broke

### F-NEW-1 — `SANDBOX_LABELS = ['tech-debt', 'audit', 'bug']` auto-enables `--sandbox` mode for OpenSpec-backed sub-issues

**Code:** [`packages/forge/src/commands/plan/orchestrator.ts:63-83`](../../../packages/forge/src/commands/plan/orchestrator.ts) — RAP-409 (B10) introduced an implicit `--sandbox` fallback for issues labelled `tech-debt`, `audit`, or `bug` **when** `inspect.specFolder` is null. The intent: these "fix-the-engine" categories typically don't have an `openspec/changes/<slug>/` folder, so sandbox mode (where the Linear issue body is the entire spec) is the right default.

**The failure:** if a real OpenSpec-backed change is labelled with any of `{tech-debt, audit, bug}` AND the spec-folder detection fails for any reason (see F-NEW-2 below), the auto-sandbox triggers and runs the OpenSpec-backed change as if it had no spec — bypassing the proposal/design/tasks artifact chain.

In the pilot run #1, RAP-746 was labelled both `audit` and `tech-debt`. Forge auto-enabled sandbox. After removing `tech-debt` (run #2), the `audit` label kept triggering the same path. The label-set is a heuristic, not a guarantee.

**Recommendation:**

- Document in the audit-batch convention that `audit` and `tech-debt` labels should only be applied to issues that are **explicitly sandbox-eligible** (no `openspec/changes/<slug>/` reference in their body).
- Alternatively, file a Forge improvement to AND the sandbox heuristic with "explicit `--sandbox` request OR no spec-source-reference in body", so the heuristic doesn't override an OpenSpec-backed issue.

**Workaround taken in pilot:** removed both labels from RAP-746. The conductor then chose the non-sandbox path on run #3. (See also F-NEW-2 — the sandbox-trigger on run #2 was further amplified by an upstream issue with spec-folder detection.)

---

### F-NEW-2 — `forge.config.mjs` resolves `repoRoot` to the primary worktree, which was on an auto-tracker branch behind main

This is **not a new finding** — memory `forge_dispatch_repo_root.md` already documents the pattern. But the pilot reproduced it cleanly:

**State observed before fix:**

- Primary worktree at `/Users/pavelrapoport/Documents/GitHub/rapoport.studio` was on branch `claude/main-tracker-2026-05-14` at commit `99df24f02` — **8 commits behind `origin/main`**.
- The scaffold PR #1036 merged at `df39581e0` (`origin/main` tip). The primary worktree did not yet have my `openspec/changes/optimize-home-fonts-and-cache/` folder.
- `findSpecFolder()` in [`commands/inspect/orchestrator.ts:61`](../../../packages/forge/src/commands/inspect/orchestrator.ts) resolves `openspec/changes/<slug>/` against `repoRoot` (= primary worktree) and returned `null` for the slug.
- `inspect.specFolder = null` combined with `audit` label triggered F-NEW-1's auto-sandbox path.

**Fix applied:** `git -C /Users/pavelrapoport/Documents/GitHub/rapoport.studio merge --ff-only origin/main` — fast-forward the primary worktree's `claude/main-tracker-2026-05-14` branch to current main. The branch is an auto-tracker (per its name) and was 0 ahead / 8 behind, so fast-forward was clean.

**Recommendation:**

- The auto-tracker branch needs an automated job to keep it on `origin/main`. Otherwise every Forge dispatch from this machine has a stale spec-folder visibility hazard.
- Pre-dispatch operator checklist for Forge runs from this repo should include: `git -C <primary-root> log --oneline -1` matches `origin/main` HEAD.

---

### F-NEW-3 — Plan-prompt construction produces a one-character user message; Anthropic API returns 400

**Reproducer:** after both F-NEW-1 and F-NEW-2 corrections were applied (no `audit`/`tech-debt` labels, primary on `origin/main`, openspec folder visible), `forge plan RAP-746 --json` and `forge epic RAP-744 --close --auto-*` both fail with:

```
APICallError [AI_APICallError]: messages: text content blocks must contain non-whitespace text
url: https://api.anthropic.com/v1/messages
requestBodyValues: {
  model: 'claude-opus-4-7',
  max_tokens: 4096,
  temperature: 0,
  system: undefined,
  messages: [
    { role: 'user', content: [ { type: 'text', text: ' ', cache_control: undefined } ] }
  ],
  tools: undefined,
  tool_choice: undefined
}
statusCode: 400
```

**Stack trace points to:** [`packages/forge/src/inference/index.ts:301`](../../../packages/forge/src/inference/index.ts) in `callForgeWithMessages`, which receives a `CoreMessage[]` already built by a caller (likely the plan orchestrator's cache-breakpoint builder at `packages/forge/src/cache/breakpoints.ts`).

**Interpretation:** the cache-breakpoint message construction is producing an empty user message (single space) for the plan-phase prompt on the `claude-opus-4-7` model. Possibilities:

- Recent regression in plan-prompt construction (last forge inference change: [`ff1108238 fix(forge): @ts-nocheck stale Anthropic-SDK types`](https://github.com/rapoport-studio/rapoport.studio/commit/ff1108238); previous: [`8dbbb1b80 forge(inference): fix Vercel-SDK temperature rejection for Claude 4.x`](https://github.com/rapoport-studio/rapoport.studio/commit/8dbbb1b80)).
- A code path that constructs cache breakpoint markers without preceding content, producing a content array of `[{ type: 'text', text: ' ' }]` as the sole user message.
- An Opus-4.7-specific prompt-template branch that wasn't populated.

**This is the previously-undocumented defect the pilot was designed to surface (Hypothesis H7 confirmed).**

**Recommendation:**

- File a Forge defect with reproducer (this report), priority High (blocks all builds using current Forge dist + Opus 4.7 against this prompt shape).
- Investigation starting point: [`packages/forge/src/cache/breakpoints.ts`](../../../packages/forge/src/cache/breakpoints.ts) and the plan orchestrator's call to `callForgeWithMessages`.
- Workaround for unblocking the audit batch: either (a) wait for the Forge fix, or (b) hand-author the implementation phases on the OpenSpec changes, bypassing Forge dispatch entirely.

---

## Hypotheses results (vs measurement plan)

| ID | Hypothesis | Result |
|---|---|---|
| H1 | `forge epic --close` correctly topo-sorts the seven sub-issues | **Confirmed.** Skip list deterministic; only Todo+auto-build dispatched. |
| H2 | state.json `outcome` lies for ≥ 1 sub-build (F1, F2 manifest) | **Not exercised.** No state.json was created — early failure path. |
| H3 | Build-lock serialization works correctly | **Not exercised.** Only one sub dispatched. No parallel attempts. |
| H4 | Verifier filter bug (F3) is no longer present | **Not exercised.** Verifier never ran — plan phase failed upstream. |
| H5 | Linear bot correctly advances sub-issue statuses | **Inconclusive.** RAP-746 stayed in `Todo` across all three failures — the failure was too early for the bot to transition. Status update is conditional on actual build start, which never happened. |
| H6 | `--auto-pr` actually opens a PR (post-RAP-392) | **Not exercised.** Build never reached PR phase. |
| H7 | The pilot reveals at least one previously-undocumented defect | **Confirmed.** F-NEW-3 (empty plan prompt to Opus 4.7) is new. |

Net: H1 + H7 confirmed; H2/H3/H4/H5/H6 require a successful build to exercise, deferred to next pilot once F-NEW-3 is resolved.

---

## Defects observed vs F1–F10 watch list

| ID | Defect | Manifested in this pilot? |
|---|---|---|
| F1 | state.json `outcome` lies | No — no state.json created |
| F2 | Builder self-commit → false `aborted-no-changes` | No — no builder ran |
| F3 | Verifier stray `--` filter bug | No — no verifier ran |
| F4 | --auto-plan skips architect role | No — no plan ran |
| F5 | Background dispatch needs all --auto-* flags | No — all flags present, no migration checkpoint hit |
| F6 | Parallel dispatch collisions | No — single dispatch |
| F7 | Forge dist staleness | **Possibly relevant** — dist was rebuilt from `df39581e0`. F-NEW-3 may be a dist regression. |
| F8 | tasks.md drift causes no-op pattern | No — tasks.md is current and validator-clean |
| F9 | Reviewer model normalizes broken behavior | No — no reviewer ran |
| F10 | Linear bot status not advancing | **Partial** — Linear bot didn't transition RAP-746 because build failed too early. Documented separately under H5 result. |

---

## Recommendations

1. **File F-NEW-3 as a Forge defect** with priority High. Reproducer: `pnpm forge plan RAP-746` on current main with primary-worktree-on-main. Investigation starts at `inference/index.ts` callers.
2. **Document the F-NEW-1 trap** in the audit-batch convention. Issue labels `audit`, `tech-debt`, `bug` are sandbox-eligibility heuristics; they should not be applied to issues whose body references `openspec/changes/<slug>/`.
3. **Add an operator pre-flight check** to the Forge first-run plan: verify primary worktree is at `origin/main` HEAD before dispatching. Memory `forge_dispatch_repo_root` already documents the principle; the procedural fix is a checklist item before any `forge epic --close`.
4. **Update repo memory** with F-NEW-3 (a new sticky defect) and F-NEW-1 (the label-trap mechanism, which is referenced in the source but wasn't in memory).
5. **The audit batch (RAP-744) is not blocked** on Forge specifically. Hand-authoring the seven implementations is viable as a fallback while F-NEW-3 is investigated. Alternatively, wait for the Forge fix; the OpenSpec scaffold itself is sound.

---

## Cost summary

- **Pilot tokens consumed:** ~0 (the Anthropic API rejected every plan request before generating content). Forge inference layer's empty-prompt request to Opus 4.7 returned 400 immediately.
- **Wall time:** ~10 minutes of operator time across three retries, plus ~7 minutes between scaffold-PR merge and first dispatch. No long-running Forge processes.
- **Net cost:** essentially zero — the pilot's empirical value (4 findings) was extracted at near-zero token spend because no successful build was generated.

---

## Artifacts

- Pilot logs: `/tmp/forge-pilot-rap746/conductor.log` + `conductor-retry-1.log` + `conductor-retry-2.log`.
- Plan-phase direct-call output (reproducer for F-NEW-3): captured in this conversation transcript; see `pnpm forge plan RAP-746 --json` invocation.
- Linear epic: [RAP-744](https://linear.app/rapoport-studio-slr/issue/RAP-744).
- Scaffold PR: [#1036](https://github.com/rapoport-studio/rapoport.studio/pull/1036).

---

## Update — Run 4 (full pipeline after F-NEW-3 fix)

> 2026-05-14, post-merge of PR #1040 (F-NEW-3 fix in `packages/forge/src/commands/plan/plan-builder.ts`).

After F-NEW-3 landed in main, the build was re-dispatched. Two more flag-related findings surfaced before the build finally proceeded, then the full pipeline (plan → build → verify → push → PR) completed.

### F-NEW-5 — Stale-plan freshness check + deprecated `--auto-plan` flag

`forge build` refuses to use a plan generated against an older Linear-issue mtime. Re-running `forge plan` is the canonical path; `--use-stale-plan` is the escape hatch. The `--auto-plan` flag (passed by the conductor's auto-flag set) does **not** auto-regenerate stale plans — the flag is marked `(deprecated)` in `forge build --help`.

**Manifestation in pilot:** `forge plan RAP-746 --json` was run during F-NEW-3 verification. The Linear issue was subsequently updated (Forge bot comments + my label edits). When `forge build RAP-746 --auto-plan` ran later, it threw `Error: Stale plan: the Linear issue was updated after this plan was generated. Re-run forge plan RAP-746 or pass --use-stale-plan to proceed.`

**Workaround:** added `--use-stale-plan` to the build invocation. The plan content was substantively correct; only the timestamp differed.

**Recommendation:** either (a) make `--auto-plan` actually regenerate stale plans (removing the "deprecated" label), or (b) update `forge epic --close` to skip-and-replan when it dispatches a sub-issue with a stale plan rather than failing the conductor.

### F-NEW-6 — `--auto-approve-plan` vs `--auto-plan` flag-name inconsistency

The `forge build` orchestrator at `checkpoint: plan-acceptance` checks for `--auto-approve-plan` to bypass interactive confirmation in non-interactive mode. But the error message recommends `--auto-plan`:

```
non-interactive mode requires --auto-approve-plan for checkpoint 'plan-acceptance'.
Re-run with --auto-plan --auto-migrations --auto-push --auto-pr for unattended dispatch.
```

The help text marks `--auto-plan` as `(deprecated)` while documenting `--auto-approve-plan` as the canonical flag, but runtime accepts both — and the error message points at the deprecated form. Operators see contradictory guidance.

**Recommendation:** align the error message to the help text (recommend `--auto-approve-plan`), or fully retire `--auto-plan` if `--auto-approve-plan` is canonical.

### F-NEW-7 — Builder rejected spec instructions; spec was technically wrong (positive finding)

**The most important finding of the pilot.** The build completed with the following deviation note in the final Forge transcript:

```
**Key deviations from plan (noted for PR body):**
1. `subsets: ['latin','cyrillic']` on `next/font/local` is not valid — the API
   only accepts `preload: boolean`. Used `preload: true` instead to achieve
   the same preload link emission.
2. `text:` on `next/font/local` is not valid (Google Fonts-only feature).
   Crimson Pro subsetting would require font-file-level tooling outside this
   change scope.
```

**Both deviations are correct.** `subsets` and `text` are `next/font/google`-only options. `next/font/local` accepts `src`, `display`, `variable`, `weight`, `preload`, `fallback`, `adjustFontFallback`, `declarations` — **not** `subsets` or `text`. The spec author (the audit's design.md drafter) modelled the local-font configuration on the google-font API and got it wrong; the Builder caught the mismatch, adapted to the correct API, and flagged the deviation in the final summary.

**This is a positive finding.** The Builder operated as a quality gate against the spec, not just a code executor against instructions. The spec drift was caught and corrected automatically with a clear human-readable note in the PR description.

**Action taken:** spec amended in this same repair PR. `design.md` D-HFC-2 rewritten to specify `preload: true` (the correct local-font API). D-HFC-4 marked as deferred — woff2-level subsetting requires a font toolchain change (`pyftsubset` / `glyphhanger`) outside the audit batch's scope. `tasks.md` Phase 3 + Phase 5 mirror the amendments.

### Full pipeline metrics (Run 4)

- **Duration:** 494 sec end-to-end (8.2 min).
- **Turns:** 73 (assistant ↔ tool roundtrips).
- **Total cost:** $2.67 USD ($0.55 plan via Opus 4.7 + $2.12 build via Sonnet 4.6).
- **Cache reads:** ~6 M tokens (excellent utilisation; ~99% of input tokens served from cache).
- **Models per role:** architect → claude-opus-4-7 (plan); builder → claude-sonnet-4-6 (build); verifier → (skipped — Builder self-committed).
- **Files changed:** 8 (additions: 94, deletions: 49). Inter font binary deleted (-344 KB).
- **PR opened:** [#1046](https://github.com/rapoport-studio/rapoport.studio/pull/1046) (subsequently merged).
- **Linear bot transitions observed:** Todo → In Progress on dispatch; PR attached to issue automatically; assignee `hello@rapoport.studio` (Maestro identity per memory `maestro_email_identity`).

### Hypothesis results (final, all exercised)

| ID | Hypothesis | Result |
|---|---|---|
| H1 | Conductor topo-sort | **Confirmed** (4× across retries; deterministic skip-reason strings). |
| H2 | state.json outcome lies | **Refuted for this build.** `Status: completed, Outcome: completed-with-pr` matched `gh pr view 1046` → OPEN, MERGEABLE. No divergence observed. May still lie in builder-self-commit-race scenarios; this build hit the self-commit path without false negative. |
| H3 | Build-lock serial | **Confirmed.** RAP-678 held the lock; Run 3 failed fast with BuildLockedError, Run 4 (after lock released) proceeded. |
| H4 | Verifier filter bug present? | **Unknown.** Builder self-committed before verifier ran (per Forge's normal flow when the builder fully resolves all checkpoints); verifier did not exercise. CI's separate test step did run and reported pre-existing failures, not Forge's verifier. |
| H5 | Linear bot transitions | **Confirmed.** Todo → In Progress on dispatch, PR auto-attached. Done transition pending merge (occurred after Pavel merged PR #1046). |
| H6 | `--auto-pr` opens PR | **Confirmed.** PR #1046 created automatically; memory `forge_auto_push_does_not_push` was already documented STALE post-RAP-392 and this run confirms. |
| H7 | New defects surfaced | **Confirmed.** F-NEW-1 (sandbox-labels trap), F-NEW-3 (plan-builder shape mismatch, now fixed), F-NEW-5 (stale-plan + deprecated flag), F-NEW-6 (flag-name inconsistency), F-NEW-7 (spec drift caught by Builder = positive). |

### Repair PR (post-pilot)

This update is committed alongside two CI-repair fixes:

1. **`tools/check-forge-spec.mjs` tripwire** was already failing on main (commands `install-skills` and `mcp-server` undocumented). Pre-existing drift from PRs #993 and #992. `openspec/current/forge/commands.md` gains a `### Utility commands` subsection.
2. **HFC spec amendment** per F-NEW-7 above.

These are bundled into one PR so the post-mortem, the spec amendment, and the tripwire fix land coherently.

---

## Net assessment

The conductor and the underlying build pipeline are **production-ready** for OpenSpec-backed sub-issues with correct labelling and a primary worktree on main. Three operator pitfalls (F-NEW-1, F-NEW-2, F-NEW-5/6) and one real Forge defect (F-NEW-3, now fixed) were surfaced. The Builder's behaviour as a spec-quality gate (F-NEW-7) is a stronger signal than expected — operators should treat Builder deviation notes as ground truth and amend specs accordingly, not vice versa.

The audit batch's remaining six sub-issues (RAP-747..752) can now dispatch via the conductor once each is promoted to Todo + `auto-build`, with the F-NEW-1 + F-NEW-2 operator checklist applied pre-dispatch. The two Q-blocked sub-issues (RAP-750 HTN, RAP-751 CCP) still wait on Pavel's open-question resolution.
