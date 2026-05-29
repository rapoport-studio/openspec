<!-- openspec-refs: allow-unresolved -->
# Forge AI stack audit — Phase 0 pre-flight

> **Date:** 2026-05-12
> **Branch:** `feat/upgrade-forge-to-modern-ai-stack/phase-0-audit`
> **Change:** `openspec/archive/2026-05-14-upgrade-forge-to-modern-ai-stack/`
> **Baseline test run:** 905 tests in 75 test files — all passed (no code changes made)

---

## Executive summary

The design.md for this change assumes Forge calls `@anthropic-ai/sdk` in the way
library consumers typically do — `const client = new Anthropic({ apiKey }); await
client.messages.create(...)`. **That is not what Forge does.** The main inference
layer (`packages/forge/src/core/anthropic.ts`) calls the Anthropic REST endpoint via
the native `fetch()` API, implementing streaming SSE parsing and retry logic from
scratch. The `@anthropic-ai/sdk` package is only used in one narrow place: the
async Message Batches path (`commands/audit/batch.ts`). A second distinct SDK,
`@anthropic-ai/claude-agent-sdk`, powers the Claude Code subprocess (builder agent)
and is architecturally independent of the inference layer.

This changes the Phase 1 migration scope significantly relative to the design's
assumptions, and introduces a concrete risk that the design did not anticipate:
Vercel AI SDK v4/v6 does not natively expose the Anthropic Message Batches API.
Migrating `audit/batch.ts` to Vercel AI SDK would lose that functionality or require
shimming.

**Recommendation:** Phase 1 is not safe to execute in this PR. The audit findings
below map the true scope; Section 7 describes what a safe migration requires before
implementation.

---

## 1. Installed package versions

| Package | Declared in `packages/forge/package.json` | Installed |
|---|---|---|
| `ai` (Vercel AI SDK) | `^4` | `4.3.19` |
| `@ai-sdk/anthropic` | `^1` | `1.2.12` |
| `@ai-sdk/mcp` | `^1` | `1.0.42` |
| `@anthropic-ai/sdk` | `^0.95.2` | `0.95.2` |
| `@anthropic-ai/claude-agent-sdk` | `^0.2.128` | `0.2.128` |
| `@modelcontextprotocol/sdk` | `^1` | installed |

**Key finding:** All four "Phase 0 — Install dependencies" packages (`ai`,
`@ai-sdk/anthropic`, `@ai-sdk/mcp`, `@modelcontextprotocol/sdk`) are already
present in `packages/forge/package.json`. The dependency installation task in
Phase 0 is **already complete**.

---

## 2. Anthropic SDK call site inventory

### 2a. `@anthropic-ai/sdk` (direct SDK) — 1 call site in Forge

| File | Line | Purpose |
|---|---|---|
| `packages/forge/src/commands/audit/batch.ts` | 12 | `import Anthropic from '@anthropic-ai/sdk'` |
| | 58 | `const client = new Anthropic({ apiKey })` |
| | 76 | `await client.messages.batches.create({ requests })` |
| | 81 | `await client.messages.batches.retrieve(batch.id)` — poll loop init |
| | 87 | `await client.messages.batches.retrieve(batch.id)` — poll loop body |
| | 95 | `await client.messages.batches.results(batch.id)` — result streaming |

**Purpose:** `runAuditBatch()` fans out the 12 audit agents through the Anthropic
Message Batches API for a 50% cost discount (stacking on top of prompt caching).
This is a batch-async path distinct from the sync inference path; it is gated by the
`--batch` flag in `forge audit`.

**Migration risk:** The Vercel AI SDK (`ai@4`) does not expose the Anthropic Batches
API. Migrating this file would require either keeping `@anthropic-ai/sdk` as a
narrow dep for batches, or implementing the Batches API via raw `fetch()` (matching
the pattern in `core/anthropic.ts`). The design.md does not address this; it is a
Phase 5 scope gap.

### 2b. `@anthropic-ai/claude-agent-sdk` — 2 files in Forge

| File | Imported symbols | Purpose |
|---|---|---|
| `packages/forge/src/conventions/mcp-tools.ts` | `createSdkMcpServer`, `tool`, `McpSdkServerConfigWithInstance`, `SdkMcpToolDefinition` | Builds an in-process MCP server wrapping the OpenSpec convention adapter tools |
| `packages/forge/src/commands/build/subprocess.ts` | `query`, `McpServerConfig` | Runs the builder agent subprocess via the Claude Code SDK; `query()` is the agent loop |

**Important distinction:** `@anthropic-ai/claude-agent-sdk` is the **Claude Code
Agent SDK**, not the general messages API. It wraps the Claude Code agent loop
(tool use, file edits, shell execution). This is orthogonal to the Vercel AI SDK
migration — the builder subprocess intentionally runs Claude Code semantics, not a
raw `generateText` call. The Vercel AI SDK does not provide an equivalent to
`query()`. This component must **not** be migrated in Phase 1.

### 2c. Custom `callClaude` / `core/anthropic.ts` — the main inference layer

The principal inference function is `callClaude()` in
`packages/forge/src/core/anthropic.ts`. It calls `https://api.anthropic.com/v1/messages`
directly via `fetch()`. It handles:

- Streaming SSE (raw text delta accumulation, TTY writing to `stderr`)
- Retry on 5xx and rate-limit (429) with exponential backoff
- `SystemBlock[]` for prompt cache breakpoints (already implemented: 5m + 1h TTLs)
- Temperature omission for Opus 4.7 (Anthropic API contract: the model rejects `temperature`)
- Cache token extraction (`cache_creation_input_tokens`, `cache_read_input_tokens`)

**No dependency on `@anthropic-ai/sdk` in this file.** The design.md assumption
(§ 1 "Current state") that these are `anthropic.messages.create()` calls is
incorrect; the existing code is already a raw-fetch adapter.

`callClaude` direct call sites (10 total):

| File | Line(s) | Role |
|---|---|---|
| `packages/forge/src/commands/spec/fix.ts` | 149 | Spec fixer (plain text response) |
| `packages/forge/src/commands/render/orchestrator.ts` | 229, 244 | Render orchestrator (initial + retry) |
| `packages/forge/src/commands/plan/plan-builder.ts` | ~110 | Plan builder |
| `packages/forge/src/commands/estimate/estimator.ts` | ~96 | Effort estimator |
| `packages/forge/src/commands/build/reviewer.ts` | 113, 163 | Build reviewer (primary + Sonnet escalation) |
| `packages/forge/src/core/json-retry.ts` | 77, 101 | JSON retry wrapper (indirectly for 5 callers below) |

`callClaudeWithJsonRetry` call sites (each calls `callClaude` via `json-retry.ts`):

| File | Line | Role |
|---|---|---|
| `packages/forge/src/commands/spec/checker.ts` | ~119 | Spec checker |
| `packages/forge/src/commands/review/reviewer.ts` | ~107 | PR reviewer |
| `packages/forge/src/commands/audit/verifier.ts` | ~53 | Audit verifier |
| `packages/forge/src/commands/audit/orchestrator.ts` | ~291 | Audit per-agent orchestrator |

---

## 3. Non-Forge dependents of `@anthropic-ai/sdk`

| Package / App | File count | Import type |
|---|---|---|
| `packages/muse/src/shared/client.ts` | 1 | Value import (`import Anthropic from '@anthropic-ai/sdk'`) |
| `packages/muse/src/…` | 12 | Type-only imports (`import type Anthropic`) |
| `apps/studio/src/…` | 6 | Value imports in Telegram classify, canvas muse runner, API routes |

**Conclusion:** `@anthropic-ai/sdk` must remain in the workspace `package.json`
even after Forge migration because `packages/muse` and `apps/studio` are active
consumers. Phase 5 of the task checklist (remove root dep) is blocked until Muse and
Studio are also migrated — which is out of scope for this change.

---

## 4. `ai` (Vercel AI SDK v4) — current Forge usage

A search of `packages/forge/src/**/*.ts` for `from 'ai'` or `from "@ai-sdk"` found
**zero results**. The `ai` and `@ai-sdk/anthropic` packages are installed in
`packages/forge/package.json` but are not yet imported anywhere in the forge source.
They were added as preparatory dependencies and have no active usage.

---

## 5. Prompt cache breakpoints — current state

The design.md describes implementing cache breakpoints in Phase 2 as new work.
In reality, prompt caching is **already implemented** in the current codebase:

- `packages/forge/src/core/anthropic.ts` defines `SystemBlock` with
  `cache_control?: { type: 'ephemeral'; ttl?: '5m' | '1h' }`.
- `packages/forge/src/commands/audit/context-builder.ts` exports
  `buildAuditSystemBlocks()` which places `cache_control: { type: 'ephemeral', ttl: '5m' }`
  on the static template block.
- `cache_creation_input_tokens` and `cache_read_input_tokens` are extracted from
  the SSE stream and surfaced in `AnthropicResponse.usage`.
- `computeCost()` applies the 5m (1.25×) and 1h (2.0×) write surcharges and the
  per-model cache-read multiplier (Opus: 1.0× observed; Sonnet/Haiku: 0.1×
  documented).

Phase 2 (cache breakpoint instrumentation) should be interpreted as extending this
existing infrastructure to the full 4-layer architecture described in `design.md § 4`,
not building from scratch.

---

## 6. Risk matrix

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Vercel AI SDK v4 does not support Anthropic Message Batches API | Confirmed | HIGH — loses 50% batch discount on `forge audit --batch` | Keep `@anthropic-ai/sdk` for `audit/batch.ts`; migrate inference path only |
| `callClaude` → `generateText` behavior diff for Opus 4.7 temperature | LOW–MEDIUM | MEDIUM — 400 errors on architect calls | Map `shouldOmitTemperature()` to a `providerOptions` override; verify before merging |
| Cache breakpoints stop working after migration | MEDIUM | HIGH — cost regression, cache ratio metrics break | Map `SystemBlock[].cache_control` → Vercel `providerOptions.anthropic.cacheControl`; re-run cache ratio tests |
| Claude Code Agent SDK (`query()`) confused with the inference layer | Confirmed | CRITICAL if migrated incorrectly | `subprocess.ts` and `mcp-tools.ts` are out of scope for Phase 1 |
| `@anthropic-ai/sdk` removal breaks Muse or Studio | Confirmed | HIGH | Phase 5 blocked until Muse + Studio separately migrated |
| Existing 905 tests break during inference layer swap | LOW | HIGH | Run `pnpm --filter @rapoport-studio/forge test` after each file migrated |
| Streaming TTY output breaks | MEDIUM | LOW | Vercel `streamText` `onChunk` callback can replicate TTY write |

---

## Findings

> Structured index added 2026-05-28 as part of
> [`inquiry-drift-coverage-retrieval-spike`](./inquiry-drift-coverage-retrieval-spike.md)
> Phase 2 substitute experiment. Grade vocabulary per `add-inquiry-entity` design.md §3
> `InquirySource.credibility` enum.
>
> **Capability:** forge

| # | Claim | Grade | Falsifier |
|---|---|---|---|
| F-1 | The design.md's premise — "Forge calls `@anthropic-ai/sdk` in the way library consumers typically do (`client.messages.create()`)" — is false. The main inference layer (`packages/forge/src/core/anthropic.ts`) calls the Anthropic REST endpoint via native `fetch()`, implementing SSE streaming and retry from scratch. | engineering_claim | A grep of `core/anthropic.ts` showing `import Anthropic from '@anthropic-ai/sdk'` would refute. Current audit confirms no such import. |
| F-2 | `@anthropic-ai/sdk` has exactly 1 call site in Forge: `commands/audit/batch.ts`, used solely for the async Message Batches API (50% cost discount stacking on prompt caching). | engineering_claim | A second call site in Forge would refute the "1 call site" claim. |
| F-3 | Vercel AI SDK v4 (`ai@4.3.19`) does not expose the Anthropic Message Batches API. Migrating `audit/batch.ts` to Vercel AI SDK loses functionality or requires shimming via raw fetch. **The design.md does not address this — it is a Phase 5 scope gap.** | engineering_claim | A Vercel AI SDK release shipping native Batches support would falsify. |
| F-4 | `@anthropic-ai/claude-agent-sdk` is the Claude Code Agent SDK (used in `subprocess.ts` + `mcp-tools.ts`), architecturally independent of the inference layer. Vercel AI SDK has no equivalent to `query()`. This component must NOT be migrated in Phase 1. | engineering_claim | A Vercel-AI-SDK shipping a subprocess-agent-loop primitive comparable to `query()` would partially falsify the "do not migrate" recommendation. |
| F-5 | All four "Phase 0 — Install dependencies" packages (`ai`, `@ai-sdk/anthropic`, `@ai-sdk/mcp`, `@modelcontextprotocol/sdk`) are already in `packages/forge/package.json`. The Phase 0 install task is already complete. | engineering_claim | A `pnpm install` run showing the packages absent would refute (current state confirms presence). |
| F-6 | Custom `callClaude` has 10 direct call sites + 4 indirect via `callClaudeWithJsonRetry`. Migration order should be lowest-risk first (estimator, plan-builder, spec/fix, render) then JSON-retry callers, then `build/reviewer.ts` last (highest coverage; Haiku + Sonnet escalation path is subtle). | engineering_claim | A migration attempt in a different order producing fewer regressions would partially falsify the ordering recommendation. |
| F-7 | Existing 905 tests across 75 test files pass on baseline (no code changes). Phase 1 migration risk: regression on the 905-test suite is the primary gate. | engineering_claim | Phase 1 migration that lands without a 905→905 test pass would falsify the "regression is the gate" framing. |
| F-8 | Removing `@anthropic-ai/sdk` from Forge is Phase 5 — blocked until Muse and Studio separately migrate. Cross-package dependency surface. | engineering_claim | An audit showing Muse/Studio use Vercel AI SDK already (no `@anthropic-ai/sdk` import) would unblock Phase 5 immediately. |

---

## 7. Phase 1 — Recommended safe approach (for the next executor)

The main inference layer is a custom `fetch`-based client, not `new Anthropic()`. A
safe Phase 1 migrates **only the inference call sites** and leaves `audit/batch.ts`,
`subprocess.ts`, and `mcp-tools.ts` untouched.

**Pre-conditions (must be met before starting Phase 1):**

1. Decide the Batches API fate: keep `@anthropic-ai/sdk` as a narrow batch-only dep,
   or rewrite `audit/batch.ts` via raw fetch.
2. Capture a baseline trace: run a small live build with the current `callClaude`
   implementation and save the prompt + response to
   `packages/forge/tests/fixtures/baseline-trace.json`.

**Recommended step order:**

1. Create `packages/forge/src/inference/index.ts` exporting a `callForge()` function
   wrapping `generateText` from `ai` with `@ai-sdk/anthropic`. Match the
   `callClaude()` signature so the swap is mechanical.
2. Migrate lower-risk call sites first: `estimate/estimator.ts`, `plan/plan-builder.ts`,
   `spec/fix.ts`, `render/orchestrator.ts`. Run the test suite after each file.
3. Migrate JSON-retry callers: `spec/checker.ts`, `review/reviewer.ts`,
   `audit/verifier.ts`, `audit/orchestrator.ts`.
4. Migrate `build/reviewer.ts` last (highest test coverage; Haiku + Sonnet escalation
   path is subtle).
5. Delete `core/anthropic.ts` only after all call sites are migrated and tests pass.
6. Leave `commands/audit/batch.ts`, `commands/build/subprocess.ts`, and
   `conventions/mcp-tools.ts` unchanged.

---

## 8. Baseline test results

```
Test Files: 75 passed (75)
Tests:      905 passed (905)
Start at:   16:47:57
Duration:   12.64s (transform 2.26s, setup 0ms, collect 6.29s, tests 39.00s)
```

No changes were made to forge source code in this PR.

---

## 9. Phase 0 tasks checklist

| Task | Status | Notes |
|---|---|---|
| Audit `@anthropic-ai/sdk` call sites in `packages/forge/` | ✅ | 1 file (`audit/batch.ts`), 6 call sites. Main inference layer uses raw `fetch()`, not the SDK. |
| Audit non-Forge dependents (`packages/muse/`, `apps/`) | ✅ | Muse (13 files) + Studio (6 files) — root dep must stay |
| Capture known-good build trace | ⏳ | Deferred to Phase 1 — requires live API key + running build |
| Install dependencies | ✅ | Already present: `ai@4.3.19`, `@ai-sdk/anthropic@1.2.12`, `@ai-sdk/mcp@1.0.42`, `@modelcontextprotocol/sdk` |
| Author this audit document | ✅ | This file |
