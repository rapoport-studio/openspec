<!-- openspec-refs: allow-unresolved -->
# Forge AI stack — Phase 0 baseline capture

> **Date:** 2026-05-13
> **Branch:** `docs/upgrade-forge/phase-0-baseline`
> **Change:** `openspec/archive/2026-05-14-upgrade-forge-to-modern-ai-stack/`
> **Based on audit:** `openspec/_methodology/research/upgrade-forge-ai-stack-audit.md` (2026-05-12)
> **Baseline test run:** 905/905 passing (recorded in audit; no code changes)

---

## Purpose

This document captures the known-good Forge inference baseline immediately before Phase 1 (Vercel AI SDK migration) begins. It supplements the full audit by providing a concise regression reference: what files call the Anthropic API, which models they use, what cost/cache telemetry already exists, and what integration points will change.

A live build trace (`packages/forge/tests/fixtures/baseline-trace.json`) is deferred to Phase 1 — it requires a live Anthropic API key which is unavailable in this documentation context.

---

## 1. Anthropic API call site inventory

### 1a. Files in `packages/forge/src/` that touch the Anthropic API

Produced by:
```
grep -r "anthropic|claude|fetch.*api\.anthropic" packages/forge/src/ --include="*.ts" -l
```

**Inference layer (custom `fetch`-based — not `@anthropic-ai/sdk`):**

| File | Role |
|---|---|
| `packages/forge/src/core/anthropic.ts` | Core `callClaude()` adapter, model registry, cost/cache math |
| `packages/forge/src/core/json-retry.ts` | JSON-retry wrapper around `callClaude()` |
| `packages/forge/src/commands/plan/plan-builder.ts` | Plan builder — calls `callClaude()` directly |
| `packages/forge/src/commands/spec/fix.ts` | Spec fixer — calls `callClaude()` directly |
| `packages/forge/src/commands/spec/checker.ts` | Spec checker — calls `callClaudeWithJsonRetry()` |
| `packages/forge/src/commands/render/orchestrator.ts` | Render orchestrator — calls `callClaude()` (initial + retry) |
| `packages/forge/src/commands/audit/verifier.ts` | Audit verifier — calls `callClaudeWithJsonRetry()` |
| `packages/forge/src/commands/audit/orchestrator.ts` | Audit per-agent orchestrator — calls `callClaudeWithJsonRetry()` |
| `packages/forge/src/commands/audit/context-builder.ts` | Cache-aware system block builder for audit commands |
| `packages/forge/src/commands/review/reviewer.ts` | PR reviewer — calls `callClaudeWithJsonRetry()` |
| `packages/forge/src/commands/estimate/estimator.ts` | Effort estimator — calls `callClaude()` directly |
| `packages/forge/src/commands/build/reviewer.ts` | Build reviewer — calls `callClaude()` (primary + Sonnet escalation) |

**`@anthropic-ai/sdk` (direct SDK — 1 file only):**

| File | Lines | Purpose |
|---|---|---|
| `packages/forge/src/commands/audit/batch.ts` | 12, 58, 76, 81, 87, 95 | Anthropic Message Batches API (`forge audit --batch`) |

**`@anthropic-ai/claude-agent-sdk` (Claude Code Agent SDK — 2 files; out of scope for Phase 1):**

| File | Imported symbols | Purpose |
|---|---|---|
| `packages/forge/src/conventions/mcp-tools.ts` | `createSdkMcpServer`, `tool`, `McpSdkServerConfigWithInstance`, `SdkMcpToolDefinition` | In-process MCP server wrapping OpenSpec convention tools |
| `packages/forge/src/commands/build/subprocess.ts` | `query`, `McpServerConfig` | Builder agent subprocess (Claude Code semantics — `query()` loop) |

### 1b. `callClaude()` call graph

```
callClaude (core/anthropic.ts)
├── direct callers (10 call sites):
│   ├── commands/plan/plan-builder.ts:110
│   ├── commands/spec/fix.ts:149
│   ├── commands/render/orchestrator.ts:229, 244
│   ├── commands/estimate/estimator.ts:96
│   ├── commands/build/reviewer.ts:113, 164
│   └── core/json-retry.ts:77, 101
│
└── via callClaudeWithJsonRetry (core/json-retry.ts):
    ├── commands/spec/checker.ts:119
    ├── commands/review/reviewer.ts:107
    ├── commands/audit/verifier.ts:53
    └── commands/audit/orchestrator.ts:291
```

**Key architectural finding:** The main inference layer (`core/anthropic.ts`) calls `https://api.anthropic.com/v1/messages` via native `fetch()`, not via the `@anthropic-ai/sdk` package. The design.md assumption that these are `anthropic.messages.create()` calls is incorrect.

---

## 2. Current models used

Defined in `packages/forge/src/core/anthropic.ts`:

```typescript
export const MODELS = {
  opus: 'claude-opus-4-7',        // Architect, plan builder, build reviewer (primary)
  sonnet: 'claude-sonnet-4-6',    // Build reviewer escalation, spec fix, some estimate paths
  haiku: 'claude-haiku-4-5-20251001',  // Cost-optimized paths
} as const;
```

**Model → role mapping (from call sites):**

| Model | Command paths |
|---|---|
| `claude-opus-4-7` | Plan builder, build reviewer (primary), render orchestrator |
| `claude-sonnet-4-6` | Build reviewer (Sonnet escalation on Haiku fail), spec fix, spec checker |
| `claude-haiku-4-5-20251001` | Estimate, audit verifier, some audit orchestrator paths |

**Per-model pricing (cost table in `core/anthropic.ts`):**

| Model | Input $/MTok | Output $/MTok |
|---|---|---|
| `claude-opus-4-6` (historical) | $15 | $75 |
| `claude-opus-4-7` | $15 | $75 |
| `claude-sonnet-4-20250514` (historical) | $3 | $15 |
| `claude-sonnet-4-6` | $3 | $15 |
| `claude-haiku-4-5-20251001` | $0.50 | $2.50 |

**Notable quirk:** `claude-opus-4-7` rejects the `temperature` parameter; `shouldOmitTemperature()` in `core/anthropic.ts` detects this and omits the field for Opus 4.7 calls. Phase 1 must replicate this via `providerOptions` in the Vercel SDK.

---

## 3. Current token/cost telemetry

Prompt cache infrastructure is **already implemented** — Phase 2 extends rather than builds from scratch.

### 3a. Cache token extraction

`AnthropicResponse.usage` includes:
- `cache_creation_input_tokens` — tokens written to cache (+25% write surcharge, or +100% for 1h tier)
- `cache_read_input_tokens` — tokens served from cache (~0–10% of base price depending on model)

Extracted from SSE stream in `core/anthropic.ts`.

### 3b. Existing cache breakpoints

| File | Breakpoint |
|---|---|
| `commands/audit/context-builder.ts` | `buildAuditSystemBlocks()` — 5-minute TTL on static template block |
| `core/anthropic.ts` | `SystemBlock.cache_control?: { type: 'ephemeral'; ttl?: '5m' \| '1h' }` |

Phase 2 will extend to the full 4-layer architecture:
- Layer 1: system prompt + persona + skills index (5m TTL)
- Layer 2: tool definitions (5m TTL)
- Layer 3: OpenSpec bundle / repo-map (1h TTL)
- Layer 4: current task context (no cache)

### 3c. Cost computation

`computeCost()` and `computeCacheAwareCost()` in `core/anthropic.ts` apply:
- Base input/output rates per model
- 5m cache write surcharge: 1.25×
- 1h cache write surcharge: 2.0×
- Cache read multiplier: per-model (0.1× documented for Sonnet/Haiku; ~1.0× observed for Opus in dogfood)

Usage rolled up per build in `commands/build/usage.ts` and `commands/build/metrics.ts`.

---

## 4. Key integration points changing in Phase 1

| Integration point | Current | Phase 1 target | Risk |
|---|---|---|---|
| Core inference entry | `callClaude()` in `core/anthropic.ts` (raw `fetch`) | `callForge()` in `inference/index.ts` (Vercel `generateText`) | Medium — streaming TTY behavior must be replicated |
| Model selection | `MODELS` enum + call-site hardcoding | `anthropic('claude-opus-4-7')` via `@ai-sdk/anthropic` | Low |
| Temperature omission (Opus 4.7) | `shouldOmitTemperature()` gate | `providerOptions.anthropic` override | Medium — must verify 400 error doesn't recur |
| Cache breakpoints | `SystemBlock[].cache_control` passed to `fetch` body | `providerOptions.anthropic.cacheControl` per message | Medium — must verify cache ratio holds |
| Cache telemetry | Parsed from raw SSE `usage` field | Vercel SDK `providerMetadata.anthropic.usage` | Low |
| JSON retry | `callClaudeWithJsonRetry()` in `core/json-retry.ts` | Vercel `generateObject()` or retained wrapper | Medium — retry logic interacts with cache |
| Batches API | `@anthropic-ai/sdk` in `audit/batch.ts` | **Unchanged** — Vercel AI SDK does not expose Batches API | N/A (out of scope) |
| Claude Code subprocess | `@anthropic-ai/claude-agent-sdk` `query()` | **Unchanged** — architecturally separate | N/A (out of scope) |

---

## 5. Risks from Phase 0 audit

From `upgrade-forge-ai-stack-audit.md § 6`:

| Risk | Likelihood | Impact | Phase 1 mitigation |
|---|---|---|---|
| Vercel AI SDK v4 does not support Anthropic Message Batches API | **Confirmed** | HIGH — loses 50% batch discount on `forge audit --batch` | Keep `@anthropic-ai/sdk` narrow dep for `audit/batch.ts`; migrate inference only |
| `callClaude` → `generateText` behavior diff for Opus 4.7 temperature | LOW–MEDIUM | MEDIUM — 400 errors on architect calls | Map `shouldOmitTemperature()` → `providerOptions` override; verify before merge |
| Cache breakpoints stop working after migration | MEDIUM | HIGH — cost regression, cache ratio metrics break | Map `SystemBlock[].cache_control` → Vercel `providerOptions.anthropic.cacheControl`; re-run cache ratio tests |
| Claude Code Agent SDK (`query()`) confused with inference layer | **Confirmed** | CRITICAL if migrated incorrectly | `subprocess.ts` + `mcp-tools.ts` are explicitly out of scope for Phase 1 |
| `@anthropic-ai/sdk` removal breaks Muse or Studio | **Confirmed** | HIGH — 19 files still import it | Phase 5 blocked; root dep must stay |
| Existing 905 tests break during inference layer swap | LOW | HIGH | Run forge test suite after each file migrated |
| Streaming TTY output breaks | MEDIUM | LOW | Vercel `streamText` `onChunk` callback can replicate TTY write |

---

## 6. Files **not** migrating in Phase 1 (must remain unchanged)

| File | Reason |
|---|---|
| `commands/audit/batch.ts` | Uses Anthropic Message Batches API — not available in Vercel AI SDK |
| `commands/build/subprocess.ts` | Uses `@anthropic-ai/claude-agent-sdk` `query()` — different semantic model |
| `conventions/mcp-tools.ts` | Uses `@anthropic-ai/claude-agent-sdk` — Claude Code in-process MCP |

---

## 7. Dependencies already installed

All Phase 0 preparatory deps are present in `packages/forge/package.json`:

| Package | Version |
|---|---|
| `ai` (Vercel AI SDK) | `4.3.19` |
| `@ai-sdk/anthropic` | `1.2.12` |
| `@ai-sdk/mcp` | `1.0.42` |
| `@anthropic-ai/sdk` | `0.95.2` |
| `@anthropic-ai/claude-agent-sdk` | `0.2.128` |
| `@modelcontextprotocol/sdk` | installed |

No new `pnpm add` required before starting Phase 1.

---

## 8. Deferred: live build trace

The Phase 0 task specifies saving a prompt+response trace to
`packages/forge/tests/fixtures/baseline-trace.json`. This requires a live
Anthropic API key and a running Forge build environment. Deferred to the Phase 1
branch per the tasks.md note. The existing 905-test baseline (recorded in the
audit, 2026-05-12) serves as the regression anchor until the live trace is
captured.
