<!-- openspec-refs: allow-unresolved -->
# Claude Platform — capability adoption spec

> **Date:** 2026-05-16
> **Author:** Pavel + Claude (analysis session)
> **Status:** Recommendation — not yet ratified into a capability spec
> **Scope:** Which platform.claude.com capabilities Rapoport Studio should connect, where, and in what order.
> **Source:** `platform.claude.com/docs` (features overview, 2026-05-16) cross-referenced with the current studio stack.

---

## Purpose

The Claude Platform shipped a large surface of agent/infra capabilities (effort,
adaptive thinking, structured outputs, server-side tools, MCP connector, Agent
Skills, Managed Agents, Files API, Citations, compaction). This document maps that
surface against the studio's own components — **Forge**, **Muse**, **Studio app /
Canvas**, **Maestro** — and produces a prioritised adoption plan.

This is a recommendation artifact. Each "adopt" item becomes a Linear issue + an
OpenSpec change before any code lands. Nothing here modifies `openspec/current/`.

---

## 1. Current stack — what is already wired

Established from `upgrade-forge-ai-stack-audit.md` (2026-05-12),
`upgrade-forge-baseline-2026-05-13.md`, and a fresh read of `packages/forge/`.

| Capability | State today | Where |
|---|---|---|
| Messages API (raw `fetch`) | ✅ Live | `core/anthropic.ts` `callClaude()` — custom SSE/retry client |
| Vercel AI SDK inference adapter | ✅ Live (migration done) | `inference/index.ts` `callForge()` — `ai@4` + `@ai-sdk/anthropic@1.2` |
| Per-role model tiering | ✅ Live | `ROLE_DEFAULTS` — Opus 4.7 architect/planner, Sonnet 4.6 builder, Haiku 4.5 verifier |
| Prompt caching (5m + 1h `SystemBlock`) | ✅ Implemented | `core/anthropic.ts` `SystemBlock.cache_control`, 4-layer breakpoints in `cache/` |
| Batch API (50% discount) | ✅ Live | `commands/audit/batch.ts` via `@anthropic-ai/sdk` (`--batch` flag) |
| Claude Code Agent SDK (builder subprocess) | ✅ Live | `commands/build/subprocess.ts` `query()` |
| In-process MCP server | ✅ Live | `conventions/mcp-tools.ts`, `mcp-server/` dir, `@ai-sdk/mcp` |
| JSON-retry wrapper | ✅ Live | `core/json-retry.ts` — retries on unparseable JSON |
| Cross-build "lessons" mechanism | ✅ Live (informal) | `core/lessons.ts` |
| Cost / cache telemetry | ✅ Live | `core/metrics.ts`, `computeCost()`, `observability/` |
| **Effort parameter** | ❌ Not used | — |
| **Adaptive / extended thinking** | ❌ Not used at all | grep `thinking` → 0 hits in source |
| **Structured outputs (schema-guaranteed)** | ❌ Not used | replaced by hand-rolled `json-retry.ts` |
| **Server-side web search / web fetch** | ❌ Not used | — |
| **Memory tool (platform)** | ❌ Not used | informal `lessons.ts` instead |
| **MCP connector (remote, from Messages API)** | ❌ Not used | only in-process MCP today |
| **Files API** | ❌ Not used | — |
| **Citations / Search results** | ❌ Not used | — |
| **Compaction / context editing (server-side)** | ❌ Not used | — |
| **Managed Agents** | ❌ Not used | — |
| **Computer use** | ❌ Not used | — |

**Headline gap:** Forge runs Opus 4.7 for architect/planner with **no thinking
control and no schema guarantees**. Those are the two highest-ROId, lowest-effort
fixes on the board.

---

## 2. Adoption matrix

Verdict legend: **NOW** = Phase A · **NEXT** = Phase B · **LATER** = Phase C ·
**DEFER** = not now · **SKIP** = no fit.

| Capability | Component | Verdict | Why |
|---|---|---|---|
| Effort + Adaptive thinking | Forge inference | **NOW** | No thinking control today; Opus 4.7 dropped `budget_tokens`, effort is the only knob. Direct quality + cost lever. |
| Structured outputs (JSON schema / strict tool use) | Forge, Muse | **NOW** | Eliminates `json-retry.ts` entirely; schema conformance guaranteed by the API. ZDR-eligible (qualified). |
| Prompt caching 1h tier on OpenSpec bundle | Forge | **NOW** | Infra already exists; only needs the 90k-line bundle pinned to the 1h TTL block. Config-only. |
| Data residency (`inference_geo`) | Forge, Muse, Studio | **NOW** | MD flips GDPR-grade 23 Aug 2026; studio serves government entities. Pin EU-appropriate routing as a compliance default. |
| Server-side web search + web fetch | Muse research mode | **NEXT** | Active change `add-muse-research-mode` needs grounded retrieval — server-side tools are the native fit. |
| Memory tool | Forge | **NEXT** | Formalises the informal `core/lessons.ts` into a real cross-build knowledge store. |
| Files API | Studio / Canvas, intake | **NEXT** | Client PDFs/briefs at intake; upload once, reuse across calls instead of re-embedding. |
| Citations + Search results | Studio OpenSpec viewer, Muse research | **NEXT** | Answers in Canvas cite back to `openspec/current/*` sections; research mode cites sources. |
| Compaction (server-side) | Forge long builds | **NEXT** | Architect/planner + long agentic loops approach context limit; server-side summarisation beats manual truncation. |
| Context editing | Forge builder loop | **NEXT** | Auto-clears stale tool results near token limit. |
| MCP connector (remote) | Studio → Forge | **LATER** | Lets Studio/Canvas call Forge clients without the local CLI. Depends on Forge exposing a remote MCP server. |
| Agent Skills (platform) | Studio skills library | **LATER** | Ties to active `seed-rapoport-studio-skills-library`; formalise loading via the platform Skills API. |
| Managed Agents | Maestro | **LATER** | Maestro is async/long-running/coordinating — textbook Managed Agent profile. Evaluate vs self-hosted CF Worker cron. |
| Code execution (server-side) | Canvas, Maestro | **LATER** | Sandbox for client-uploaded CSV/JSON/data analysis; free when paired with web search/fetch. |
| Admin API / Usage & Cost API | Studio dashboards | **LATER** | Per-build / per-role spend attribution beyond `computeCost()` estimates. |
| Advisor tool | Forge plan/build | **DEFER** | Beta; overlaps Forge's existing plan↔build split. Re-evaluate when GA. |
| Computer use | Maestro | **DEFER** | Only needed for UI-without-API automation; no current driver. |
| Token counting | Forge | **DEFER** | Local approximations adequate; revisit if budgeting needs precision. |
| Tool search | — | **SKIP** | Forge's toolset is small; dynamic discovery solves a problem we don't have. |
| Programmatic tool calling | — | **SKIP** | Niche latency optimisation; no multi-tool-in-code-exec workflow yet. |
| Fine-grained tool streaming | — | **SKIP** | Marginal latency gain; not worth the wiring. |
| Batch processing | Forge audit (+Muse) | **DONE → expand** | Already live in `audit/batch.ts`; extend to Muse research fan-out and periodic OpenSpec audits. |

---

## 3. Phase A — NOW (low effort, high ROI)

These four require no new dependencies and no architectural change. Each is one
OpenSpec change.

> **Implementation status (updated 2026-05-28):**
> - **A1 — ✅ shipped.** Linear RAP-926; change archived at `openspec/archive/2026-05-16-forge-effort-adaptive-thinking-control/`.
> - **A2 — ✅ shipped.** Linear RAP-927; change archived at `openspec/archive/2026-05-16-adopt-structured-outputs-retire-json-retry/`.
> - **A3 — ✅ shipped.** Linear RAP-952; change archived at `openspec/archive/2026-05-16-pin-openspec-bundle-to-1-hour/`.
> - **A4 — ✅ shipped (engineering); decision-record pending legal sign-off.** Linear RAP-928 closed; Phase 1 (decision record, RAP-950) merged via #1311; Phase 2 (`inference_geo` wiring at every call site, RAP-951) merged via #1756. Change archived at `openspec/archive/2026-05-28-inference-data-residency-routing-decision/`. Decision-record status remains `pending-legal-review` per design.md § 5 (code can ship safer-than-default `"us"` posture while legal closes the record); flip to `accepted` happens in-place in the archived `decision-record.md` after sign-off, no follow-up change needed.

### A1. Effort + Adaptive thinking in Forge inference

- **Where:** `inference/index.ts`, `core/anthropic.ts` (`ROLE_DEFAULTS` neighbourhood), `inference/personas/*`.
- **What:** Add `EffortLevel` type + `EFFORT_DEFAULTS` (`architect`/`planner` → `xhigh`, `builder` → `medium`, `verifier`/Haiku → none). Inject `output_config.effort` and `thinking: { type: "adaptive" }` via the existing `buildOmittingProvider` fetch-wrap (Vercel AI SDK 1.2.x has no first-class `output_config` support yet — same fetch-wrap pattern as the `temperature` strip).
- **Also:** raise architect `max_tokens` from 8192 → ≥64k (docs require headroom at `xhigh`).
- **Verify:** baseline epic run before/after; compare tokens + cost per role.
- **Risk:** future `@ai-sdk/anthropic` may add native `output_config` and overwrite the injected key — fetch-wrap is an explicit temporary bridge; document the removal trigger.
- **Linkage:** continuation of archived `2026-05-10-improve-forge-model-tiering-and-verifier`.

### A2. Structured outputs — retire `json-retry.ts`

- **Where:** every `callClaudeWithJsonRetry` / `callForgeWithJsonRetry` call site (planner, spec checker, audit verifier/orchestrator, build reviewer).
- **What:** Replace the retry-on-bad-JSON loop with schema-enforced structured outputs (JSON schema mode or strict tool use). The API guarantees conformance — the markdown-fence stripping and retry machinery (and the `MEMORY.md` "Claude wraps JSON in fences" gotcha) become dead code.
- **Payoff:** removes a whole class of flaky failures and the retry token cost.
- **Note:** ZDR-eligible (qualified) — only JSON schemas cached ≤24h, prompts/outputs not stored. Confirm acceptable.
- **Sequence:** do **after** A1 so the effort change is isolated in telemetry.

### A3. 1-hour cache tier on the OpenSpec bundle

- **Where:** `cache/breakpoints.ts`, the bundle-bearing `SystemBlock`.
- **What:** Pin the ~90k-line OpenSpec bundle block to `cache_control: { ttl: '1h' }` instead of the default 5m. A Forge epic run spans well beyond 5 minutes; the 1h tier keeps the bundle cached across the whole build.
- **Effort:** config-only; infra already supports both TTLs.
- **Watch:** 1h cache write surcharge is 2.0× — only worth it if the bundle is genuinely reused across the window (it is, for epics). Measure cache-read ratio after.

### A4. Data residency default

- **Where:** inference client config (Forge `callForge`, Muse client, Studio routes).
- **What:** Set `inference_geo` explicitly per the studio's compliance posture rather than relying on `"global"` default. Decide US vs global routing as a deliberate, documented choice.
- **Driver:** Moldova Law 195/2024 goes GDPR-grade 23 Aug 2026; studio targets government/corporate clients. This is a compliance decision, not a perf one — surface it to legal review.
- **Output:** a short decision record, not just code.

---

## 4. Phase B — NEXT (medium effort, clear owner)

> **Implementation status (updated 2026-05-16):**
> - **B1 — not started.** Folds into the existing `add-muse-research-mode` change; no separate issue.
> - **B2 — ✅ shipped.** Linear RAP-930; archived at `openspec/archive/2026-05-16-forge-memory-tool-for-cross-build/`.
> - **B3 — ✅ shipped.** Linear RAP-931; archived at `openspec/archive/2026-05-16-files-api-for-client-intake-canvas/`. Note: the `ai_files` Supabase migrations are in-repo but not yet applied to the live DB.
> - **B4 — ✅ shipped.** Linear RAP-932; archived at `openspec/archive/2026-05-16-spec-citations-in-canvas-via-search/`.
> - **B5 — ✅ shipped.** Linear RAP-933; archived at `openspec/archive/2026-05-16-server-side-compaction-for-long-forge/`. Compaction/context-editing flags ship **OFF**; production graduation is gated on the runbook promotion criteria.

### B1. Server-side web search + web fetch → Muse research mode
Active change `add-muse-research-mode` needs grounded retrieval. Use the
platform's server-side `web_search` / `web_fetch` tools instead of building a
scraper. Pairs naturally with B4 (Citations) so research output is sourced.
ZDR-eligible except with dynamic filtering.

### B2. Memory tool → formalise Forge lessons
`core/lessons.ts` is already an informal cross-build memory. Replace/back it with
the platform Memory tool so Forge accumulates durable knowledge (repo conventions,
recurring failure patterns like RTL split-DOM, verification-gate quirks). Decide:
wrap the existing `lessons.ts` interface over the Memory tool, or migrate fully.

### B3. Files API → intake + Canvas attachments
Client briefs and PDFs arrive at intake. Upload once via Files API, reference by
id across calls instead of re-embedding bytes every request. Also backs
Canvas file attachments.

### B4. Citations + Search results → Studio OpenSpec viewer
When Canvas answers a question about a spec, return citations that deep-link to
`openspec/current/<capability>.md` sections. Search results API gives
"web-search-quality" attribution for the custom OpenSpec knowledge base.

### B5. Compaction + context editing → long Forge builds
Architect/planner and the builder agentic loop run long. Server-side compaction
auto-summarises earlier turns near the window limit; context editing clears stale
tool results. Both are beta — gate behind a flag, measure before defaulting on.

---

## 5. Phase C — LATER (architectural, needs design)

| Item | Precondition | Decision needed |
|---|---|---|
| MCP connector — Studio → Forge | Forge exposes a remote MCP server (extends existing `mcp-server/`) | Whether Studio consumes Forge as a remote MCP or keeps the `forge/core` import carve-out |
| Agent Skills (platform API) | `seed-rapoport-studio-skills-library` lands | Whether studio skills load via the platform Skills API or stay file-based |
| Managed Agents for Maestro | Maestro substrate exists (`packages/muse/src/modes/maestro/`) | Managed Agent vs self-hosted CF Worker cron — infra ownership tradeoff |
| Code execution (server-side) | Canvas file pipeline (B3) | Whether to drop any custom sandbox in favour of the hosted one |
| Admin / Usage & Cost API | Studio cost dashboard exists | Reconcile API-reported spend with Forge's `computeCost()` estimates |

---

## 6. Phase D — DEFER / SKIP

- **Advisor tool** — beta; overlaps Forge's plan↔build split. Revisit at GA.
- **Computer use** — no UI-without-API driver yet (future Maestro only).
- **Tool search / programmatic tool calling / fine-grained streaming** — solve
  problems the studio doesn't have at current toolset size.

---

## 7. Cross-cutting risks

| Risk | Impact | Mitigation |
|---|---|---|
| Vercel AI SDK lacks native `output_config` / structured-output parity | A1, A2 rely on fetch-wrap injection | Treat fetch-wrap as a documented bridge; track `@ai-sdk/anthropic` release notes for native support |
| Beta features (compaction, context editing, MCP connector, Skills, Files API, Advisor) can break with notice | Production builds | Gate every beta behind a flag; never default-on without a measured run |
| Batch API is **not** ZDR-eligible | `audit/batch.ts` + any Muse batch fan-out | Confirm no sensitive client data flows through batch paths |
| Data-residency choice (A4) has legal weight | Compliance | Route the decision through legal review, not engineering alone |
| `@anthropic-ai/sdk` cannot be removed | Root dep stays | Muse + Studio still consume it; out of scope to remove |

---

## 8. Recommended issue cut

One Linear issue + one OpenSpec change per item. Suggested order and slugs:

| # | Linear (suggested) | OpenSpec change slug | Phase |
|---|---|---|---|
| 1 | `feat: Forge effort + adaptive thinking` | `add-forge-effort-control` | A1 |
| 2 | `feat: structured outputs — retire json-retry` | `adopt-structured-outputs` | A2 |
| 3 | `feat: 1h cache tier for OpenSpec bundle` | `pin-openspec-bundle-1h-cache` | A3 |
| 4 | `chore: data-residency routing decision` | `set-inference-data-residency` | A4 |
| 5 | `feat: Muse research mode — server-side search` | (fold into `add-muse-research-mode`) | B1 |
| 6 | `feat: Forge memory tool` | `add-forge-memory-tool` | B2 |
| 7 | `feat: Files API for intake + Canvas` | `add-files-api-intake` | B3 |
| 8 | `feat: spec citations in Canvas` | `add-spec-citations` | B4 |
| 9 | `feat: server-side compaction for long builds` | `add-forge-compaction` | B5 |

Phase C items get design docs before issues.

---

## 9. One-line recommendation

Start with **A1 + A3 in a single PR pair** — both touch `inference/`, both are
config-grade, both produce immediate measurable cost/quality telemetry that will
de-risk every later phase. A2 (structured outputs) follows once A1's effort change
is isolated in the numbers.
