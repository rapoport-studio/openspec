# Forge Reviewer Context-Isolation Audit — May 2026

**Audit date:** 2026-05-14
**Auditor:** RAP-740 Wave 3 builder
**Scope:** `packages/forge/src/commands/build/reviewer.ts` — `buildUserMessage()` transcript-excerpt section

---

## Objective

Determine whether the Forge reviewer receives raw builder reasoning (SDK transcript) as part of its input, and if so, classify the inclusion as unconditional, degraded-mode-only, or already-absent per the decision tree in `openspec/changes/add-spec-acceptance-gate/design.md § 6`.

---

## Files audited

| File | Lines inspected |
|---|---|
| `packages/forge/src/commands/build/reviewer.ts` | `:252-276` (`buildUserMessage`) |
| `packages/forge/src/commands/build/orchestrator.ts` | `:1500-1560` (`runReviewerAndMetricsSafe`) |

---

## Findings

### Orchestrator call site (`orchestrator.ts:1502-1519`)

`runReviewerAndMetricsSafe` resolves the transcript path via `transcriptPathOrNull()` (`:1560-1565`) and passes it directly to `runReviewer()` as `inputs.transcriptPath`. The path is `null` only when the transcript file is missing or empty (dead-subprocess signal).

### Reviewer message construction (`reviewer.ts:259-276`, pre-fix)

`buildUserMessage()` branches on `inputs.transcriptPath`:

```
if transcriptPath === null  →  ## Degraded mode  (no transcript content)
else                        →  ## SDK transcript excerpt (tail)
                               ```jsonl
                               <raw JSONL tail, up to 24 000 chars>
                               ```
```

**Classification: (a) Unconditional** — the raw JSONL transcript tail was included in every non-degraded reviewer call. The reviewer received the Builder's full tool-call history (Bash invocations, file reads, edits, intermediate reasoning emitted through the SDK), not just its output artifacts.

This is the single largest source of self-bias: the reviewer could confirm the builder's own assessment rather than independently evaluating the diff and gate results.

---

## Decision tree outcome (design.md § 6, case a)

> If transcript-excerpt is ALWAYS included → remove the section in Phase 2. Replace with a one-line note: `## Builder activity: <N> tool calls, <M> file edits` (count only, no content).

---

## Fix applied

**File:** `packages/forge/src/commands/build/reviewer.ts`

Changes:
1. Removed `TRANSCRIPT_EXCERPT_CHARS` constant (was `24_000`).
2. Removed `truncateTail()` helper (no longer needed).
3. Removed `readFile` from `node:fs/promises` import (was only used by `truncateTail`).
4. Added `parseToolCallsFromTranscript` to the import from `./verification.js`.
5. Added `FILE_EDIT_TOOL_NAMES` constant and `summarizeBuilderActivity()` helper.
6. Replaced the `## SDK transcript excerpt (tail)` block with a single heading line:
   `## Builder activity: <N> tool calls, <M> file edits`

The count is derived by running `parseToolCallsFromTranscript(transcriptPath, MAX_SAFE_INTEGER)` — the same parser already used by `metrics.ts` — which parses `tool_use` events from the JSONL without exposing their content to the reviewer. File edits are counted as calls with name `Edit`, `MultiEdit`, or `Write`.

---

## Isolation guarantee

After this fix, the reviewer's input in non-degraded mode contains:

| Section | Content |
|---|---|
| `## Build context` | Issue key, wave, branch, checkpoint, cost |
| `## Builder activity: N tool calls, M file edits` | Counts only — no content, no reasoning |
| `## Verification gates` | Gate names + pass/fail status + evidence tail on failure |
| `## Baseline failures` | Pre-existing-on-main gates (when applicable) |
| `## Git diff against build base` | Full diff (up to 24 000 chars) |
| `## Pre-existing-failure heuristic` | File-intersection hint (when applicable) |

The reviewer's ground truth is the diff and gate results only. It cannot see the builder's reasoning, intermediate Bash commands, or file-read content.

In degraded mode (`transcriptPath === null`), the `## Degraded mode` section remains unchanged and explains the dead-subprocess condition. In that case the reviewer still has no builder reasoning — it has even less information.

---

## Tests

Existing reviewer unit tests (`packages/forge/src/__tests__/reviewer.test.ts`) pass without modification. The happy-path test that supplies a transcript file (`line 106`) does not assert on the transcript excerpt, so the section removal is transparent. The degraded-mode test (`line 232`) targets the `## Degraded mode` path which is unchanged.

---

## Related

- Spec: `openspec/changes/add-spec-acceptance-gate/design.md § 6`
- Issue: RAP-740 — Phase 2 context-isolation audit
- Reviewer system prompt hardening: RAP-740 parent phase (Phase 1 complete before this audit)
