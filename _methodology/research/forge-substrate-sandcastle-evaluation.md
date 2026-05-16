---
title: Forge substrate evaluation — Sandcastle vs. status quo
status: research
owner: pavel
audience: [pavel, ai]
tldr: Three-outcome evaluation of whether to migrate Forge onto @ai-hero/sandcastle as a substrate, lift selected patterns (Docker sandbox + multi-provider), or reject and keep status quo. Recommendation — lift patterns. Migration cost dominates the migrate option; sandcastle solves a different problem in the reject case.
---

# Forge substrate evaluation — Sandcastle vs. status quo

> Research document. Not an OpenSpec change. Output: a decision (one of three outcomes) that produces the next OpenSpec change.
>
> **Why this exists.** Eugene Brodsky (Onyxia Cyber, CTO) recommended Matt Pocock's [`sandcastle`](https://github.com/mattpocock/sandcastle) on 2026-05-15 as a publicly available alternative to private orchestration stacks. With memory note `forge_audit_2026_05_10.md` documenting HIGH-severity bugs in Forge today (state.json lies about push/PR creation, reviewer normalizes broken behavior), Forge's substrate is genuinely up for revisit.

---

## What Sandcastle is

`@ai-hero/sandcastle` is a TypeScript library for orchestrating AI coding agents inside isolated sandboxes. 4.4k GitHub stars, MIT, actively maintained by Matt Pocock and the [aihero.dev](https://www.aihero.dev/) community.

### Core architecture

```ts
import { run, claudeCode } from "@ai-hero/sandcastle";
import { docker } from "@ai-hero/sandcastle/sandboxes/docker";

await run({
  agent: claudeCode("claude-opus-4-7"),
  sandbox: docker(),
  promptFile: ".sandcastle/prompt.md",
});
```

Returns `{ iterations, commits, branch, completionSignal, logFilePath }`.

### Architectural primitives

| Primitive | What it does | Provider options |
|---|---|---|
| `sandbox` | Per-run isolation environment | `docker()`, `podman()`, `vercel()` (Firecracker microVMs), custom via `createBindMountSandboxProvider` / `createIsolatedSandboxProvider` |
| `agent` factory | Wraps an AI coding agent | `claudeCode()`, `codex()`, `opencode()` |
| `branchStrategy` | Where commits land | `head` (bind-mount only), `merge-to-head` (temp branch → merged on completion), `branch` (named persistent branch) |
| Worktree | Git worktree per run, bind-mounted into the sandbox | Single mechanism; the bind-mount writes flow back through to host fs in real time |

### What Sandcastle does NOT include

- **No spec layer.** Sandcastle reads `.sandcastle/prompt.md` and runs an agent. There is no OpenSpec equivalent, no proposal/design/tasks triplet, no capability spec discipline.
- **No issue-tracker integration.** No Linear, no GitHub Issues, no Jira. Sandcastle is invoked programmatically; the caller resolves "what to work on".
- **No PR creation.** Sandcastle commits to a branch. PR creation, review, merge — caller's problem.
- **No cost instrumentation.** Returns iteration count; no token accounting, no per-model cost roll-up.
- **No multi-phase pipeline.** Sandcastle's loop is one phase: the agent runs until `completionSignal` fires. Forge's plan / build / verify / push split does not have an analogue.
- **No verifier.** No `forge plan`-equivalent dry run, no architect-as-reviewer role, no automated checkpoint validation.
- **No Vercel AI SDK dependency** (despite an earlier misreading on my part). Sandcastle has its own orchestration loop.

---

## What Forge does today

Per `openspec/current/forge/spec.md` + `forge/commands.md` + `forge/verifier.md` and corroborated by `_methodology/research/forge-epic-first-run-results.md`.

### Pipeline

```
Linear issue (RAP-NNN)
   ↓ fetch
Plan phase  ─→ Architect → produces plan.json (subtask list + cost estimate)
   ↓ checkpoint (auto-plan flag)
Build phase ─→ Builder (model: opus/sonnet per agent-model-mapping) → mutates worktree
   ↓ checkpoint (auto-migrations flag → apply_migration if any)
   ↓ checkpoint (auto-push flag)
Push phase  ─→ git push origin <branch>
   ↓ checkpoint (auto-pr flag)
PR creation ─→ gh pr create
   ↓ (optional: --wait-for-merge)
Auto-merge or hand off to human
```

### Forge primitives

| Primitive | Status |
|---|---|
| Worktree per build | Yes (`forge.config.mjs` `worktreeRoot: __dirname`) |
| Sandbox (Docker/etc.) | **No** — runs on host filesystem |
| Multi-provider | **No** — Anthropic-locked via `packages/forge/src/core/anthropic.ts` |
| Branch strategy | One mode — per-issue branch with auto-PR-creation |
| Issue tracker | Linear (via `@rapoport-studio/forge/core/linear-client.ts`) |
| GitHub integration | Yes — Octokit wrapper, PR creation, merge automation |
| Cost instrumentation | Yes — per-call token tracking, per-model pricing table, audit/spec/review pre-flight estimates |
| OpenSpec integration | Yes — plan phase reads `openspec/current/*` and the change's `openspec/changes/<slug>/` |
| Verifier | Yes — architect-as-reviewer role audits PRs; plan-phase dry-run via `forge plan <issue> --json` |
| Phases | Plan / Build / Verify / Push / PR — independently flaggable |
| Global build lock | Yes (`~/.forge/locks/build.lock`) |
| Per-build state | `~/.forge/state/<build-id>.json` |

### Known HIGH-severity bugs (memory `forge_audit_2026_05_10.md`)

- `state.json: failed/completed` outcome lies about real PR state — must cross-check `gh pr view`.
- Reviewer normalizes broken behavior — failed builds get marked passing if PR exists.
- Builder vs orchestrator commit race — orchestrator sees clean worktree after Builder self-commits → false `aborted-no-changes` outcome.
- F-NEW-3 (2026-05-14) — plan-phase sends empty content to Opus 4.7 → API 400. Current dist still broken.
- `--auto-push` doesn't actually push pre-RAP-392; post-RAP-392 it does (memory `forge_auto_push_does_not_push.md` was reclassified).

These are not all root-caused. The substrate decision interacts with the fix paths.

---

## Comparison matrix

| Capability | Sandcastle | Forge |
|---|---|---|
| **Sandboxing** | Docker / Podman / Firecracker microVMs (Vercel) — bind-mount isolates the agent from host fs | git worktree only — agent has full host fs access via `bash` |
| **Multi-provider** | Yes — `claudeCode()`, `codex()`, `opencode()` factories | No — Anthropic-locked |
| **Branch strategy** | Three modes (head / merge-to-head / branch) | One mode (per-issue branch) |
| **Pipeline phases** | One loop until completion-signal | Plan / Build / Verify / Push / PR — independently flaggable |
| **Issue-tracker integration** | None — caller resolves | Linear built-in |
| **PR creation** | None — caller does it | Built-in via Octokit |
| **Cost tracking** | None | Per-call token + per-model price + pre-flight estimate |
| **Spec-driven workflow** | None — `.sandcastle/prompt.md` is freeform | OpenSpec triplet (`proposal.md` / `design.md` / `tasks.md`) + capability specs |
| **Verifier / dry-run** | None | `forge plan --json`, architect-as-reviewer |
| **Lock strategy** | None visible (per-run isolation makes it less necessary) | Global build lock + `.git/config.lock` collision risk |
| **State persistence** | `logFilePath` per run | `~/.forge/state/<build-id>.json` per build |
| **Public + community** | MIT, 4.4k stars, 83 open issues, 452 forks | Private to Rapoport Studio |
| **Maturity** | Active development; v0.x | Production; daily use; known bug list |
| **License** | MIT | Internal |

---

## Three outcomes

### Outcome A — Migrate (rebuild Forge on Sandcastle)

**Definition.** Replace Forge's current loop with `sandcastle.run()`. Keep all Forge primitives that Sandcastle doesn't have (OpenSpec layer, Linear integration, PR creation, cost tracking, verifier) as wrappers around Sandcastle. The plan/build/verify/push pipeline becomes a sequence of `sandcastle.run()` calls.

**Pros**
- Free Docker / Firecracker sandboxing (mitigates AT-04 / AT-05 from `agents-threat-model.md` — RPN 15 + 10).
- Free multi-provider (closes C-4 roadmap item with zero code).
- Free community — upstream bug fixes, contributors, public scrutiny.
- Drops worktree-collision class of bugs that came from co-locating multiple Forge runs on the same fs (memory `forge_parallel_dispatch_collisions.md`).

**Cons**
- **Re-port all spec-driven primitives.** OpenSpec read + Linear fetch + PR creation + cost tracking + verifier — all currently embedded in Forge's loop. Migration means re-implementing them as Sandcastle adapters or as wrappers around it.
- **Re-port the multi-phase pipeline.** Sandcastle is single-loop; Forge's plan / build / verify / push split has to be re-expressed as a sequence of `sandcastle.run()` calls. This isn't impossible but it's not trivial — each phase has different prompts, different tool surfaces, different stop conditions.
- **Lose internal control of the orchestration code.** Once Forge depends on Sandcastle, upstream changes affect us. Mitigation by vendoring is possible but defeats half the benefit.
- **Migration risk.** Known-good (if buggy) Forge → unknown-state migrated Forge. Memory `forge_audit_2026_05_10.md` HIGH bugs need to be reproduced on the new substrate before we trust it.
- **Migration cost.** Honest estimate: 4-8 weeks of work to reach parity, then 4-8 weeks to migrate active flows (RAP-744 home-audit pilot, builder-mode RAP-48, every in-flight epic). Total: 2-4 months at solo cadence.

**Estimated cost.** 2-4 months solo. ~$2k-$5k Anthropic spend on re-validation runs (compare two substrates on the same epics).

**Verdict.** Cost dominates. Migrate is the right call only if Forge's current substrate is fundamentally broken — and the audit list, while real, is fixable in-place.

### Outcome B — Lift patterns (keep Forge, adopt selected Sandcastle ideas)

**Definition.** Forge keeps its identity, structure, and shipping path. Three Sandcastle primitives ported over time as separate OpenSpec changes:

1. **Docker sandbox provider for Forge bash tool** — analog of Sandcastle's `docker()` provider. Forge bash tool runs in a container with read-only host mount; the worktree is bind-mounted read-write. Mitigates AT-04 / AT-05 from `agents-threat-model.md`. Separate OpenSpec change `add-forge-docker-sandbox`.
2. **Multi-provider model abstraction** — analog of Sandcastle's `agent` factory. Forge's `anthropic.ts` becomes one of several providers; per-role model selection (already in `_methodology/tools/agent-model-mapping.md`) can target different providers per role. Aligns with roadmap C-4. Separate OpenSpec change `add-forge-provider-abstraction`.
3. **Three branch strategies** — analog of Sandcastle's `branchStrategy: { type: ... }`. Forge gains the same head / merge-to-head / branch options. Useful for builder-mode and audit-mode where we sometimes want commits to land on a temporary branch and merge back, vs always producing a PR. Separate OpenSpec change `add-forge-branch-strategies`.

**Pros**
- Lowest risk path — every change is incremental, independently shippable, fully reversible.
- Pavel's mental model stays current; no re-learning.
- OpenSpec discipline gives clean audit trail per pattern lift.
- Each pattern lift can be cancelled or revised mid-flight without affecting the others.

**Cons**
- Slowest path to the security benefit — Docker isolation is the highest-value pattern (AT-04 RPN 15) and would land alone, not part of a larger migration.
- We continue to maintain the orchestration code internally; no community offload.
- Multi-provider abstraction requires real design work; can't just import `claudeCode()` factory shape.

**Estimated cost.** Per-pattern estimates (rough):

| Pattern | Cost | Priority |
|---|---|---|
| Docker sandbox | 1-2 weeks | High (security gating from `agents-threat-model.md`) |
| Provider abstraction | 2-4 weeks | Medium (roadmap C-4) |
| Branch strategies | 1 week | Low (nice-to-have) |

**Verdict.** Best risk/reward. Docker sandbox is independently worth doing. Provider abstraction is independently scheduled. Branch strategies is optional.

### Outcome C — Reject (Sandcastle solves a different problem; keep status quo)

**Definition.** Acknowledge Sandcastle as a public reference, but don't adopt anything. Forge continues as-is. Threat-model mitigations from `agents-threat-model.md` get bespoke implementations (not Sandcastle-shaped).

**Pros**
- No new code to write or integrate.
- No regression risk.
- Existing Forge mental model preserved.

**Cons**
- We re-invent Docker sandboxing if we want it (we want it — AT-04 alone justifies it).
- We re-invent provider abstraction if we want it (roadmap says yes — C-4).
- Eugene's pointer goes unused; the deconstruction back to him is "we looked and rejected" rather than "we looked and lifted X, Y."
- Forge's HIGH-severity bug list grows; no external pressure to address them.

**Verdict.** Defensible only if Forge's current architecture is genuinely incompatible with Sandcastle's primitives. It isn't — Forge's worktree-per-build is the same primitive as Sandcastle's. Reject is the lazy answer, not the right one.

---

## Recommendation — Outcome B (Lift patterns)

Best risk/reward. Each lift is an independently-shippable OpenSpec change:

**Priority order:**

~~1. **`add-forge-docker-sandbox`** — Pattern: Sandcastle's `docker()` provider. Mitigates `agents-threat-model.md` AT-04 (RPN 15) and AT-05. Estimated 1-2 weeks.~~ **Shipped 2026-05-16** — archived at [`openspec/archive/2026-05-16-add-forge-docker-sandbox/`](../../archive/2026-05-16-add-forge-docker-sandbox/). AT-04/AT-05 post-mitigation RPNs: 5/5 (down from 15/10).

1. **`add-forge-provider-abstraction`** — Pattern: Sandcastle's `agent` factory shape. Closes roadmap C-4. Acceptance: `agent-model-mapping.md` rows can target a provider key (`anthropic` / `openai` / `bedrock`) per role; runner dispatches to the right adapter. Estimated 2-4 weeks.
2. **`add-forge-branch-strategies`** — Optional. Lowest priority. Useful only after we have a Builder-mode use case that benefits from non-PR commit landing. Reassess later.

**Explicitly not adopted:**

- Sandcastle's full orchestration loop. Forge's plan/build/verify/push pipeline is more sophisticated than Sandcastle's single-loop model for spec-driven work.
- Sandcastle's `claudeCode()`, `codex()`, `opencode()` factories as-is. We have our own role-model mapping in `agent-model-mapping.md`; the right move is to align our abstraction with theirs at the *concept* level, not import their code.
- Sandcastle as a runtime dependency. Pattern lift = read their code, adapt the design, ship our own. Avoids upstream-coupling risk; respects the muse → forge/core boundary discipline already in place.

---

## Decision criteria (re-evaluate quarterly)

- **Number of Forge HIGH-severity bugs unresolved.** If the audit-2 list (4 HIGH bugs as of 2026-05-10) grows past 8, re-open Outcome A (migrate); the internal substrate is unmaintainable at that point.
- **Anthropic-provider monopoly cost.** If Anthropic pricing or availability degrades such that switching matters operationally, accelerate provider-abstraction.
- **Public collaboration interest.** If a credible contributor offers to help on Forge and would rather work on `mattpocock/sandcastle` than on closed Forge, that's a real cost — re-open Outcome A.
- **Compliance pressure.** If an Unbind engagement or another client requires SOC2-style isolation, Docker sandbox is no longer optional and the patterns get pulled forward.

---

## Open questions

- **Sandcastle's session-history primitive (tree-based, branchable per Pi/`pi.dev` philosophy) — does it apply to Forge?** Forge's `~/.forge/state/<build-id>.json` is per-build flat state; no branching. Whether tree-state helps the audit/reviewer flow is worth a small spike. Out of scope for this evaluation.
- **What does the maintenance overhead of `add-forge-docker-sandbox` look like on Apple Silicon?** Docker-for-Mac performance has cost spikes; need to confirm acceptable bash latency in a Forge build before committing to the pattern. Spike: 1 day.
- **Should the Docker substrate also house Muse Architect's read-tools (RAP-830) and Maestro's session?** Today both run in-process in `apps/studio`. Docker-isolating them is a much bigger change (Cloudflare Worker compatibility) and is out of scope for this evaluation.

---

## References

- [`agents-threat-model.md`](../security/agents-threat-model.md) — companion threat model; AT-04 / AT-05 are the security driver for the Docker sandbox lift
- [`mattpocock/sandcastle` README](https://github.com/mattpocock/sandcastle)
- [aihero.dev — Matt Pocock's site](https://www.aihero.dev/)
- [`forge/spec.md`](../../current/forge/spec.md) — Forge capability spec
- [`forge/commands.md`](../../current/forge/commands.md) — Forge command reference
- [`forge/verifier.md`](../../current/forge/verifier.md) — Forge verifier role
- Memory note `forge_audit_2026_05_10.md` — HIGH bug list driving migration consideration
- Memory note `forge_parallel_dispatch_collisions.md` — worktree collision class that Docker would eliminate
