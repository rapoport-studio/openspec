# Per-agent model bindings — methodology

> Single source of truth for agent → default model → fallback chain → reasoning across the orchestra. Per-agent capability specs forward-reference this document by row label, not by provider ID.

---

## Status

**Layer 0.5 of the philosophy foundation arc (applied 2026-05-09 from change `add-tools-methodology`, [RAP-174](https://linear.app/rapoport-studio-slr/issue/RAP-174)).** Sits between Layer 0 (philosophy + roster — `agents-overview.md`, applied 2026-05-08 from `add-philosophy-foundation`) and Layer 1 (per-agent capability specs — `add-agents-specs` Phase A merged 2026-05-09 via [PR #428](https://github.com/rapoport-studio/rapoport.studio/pull/428), `amend-muse-forge-for-maestro` merged via [PR #432](https://github.com/rapoport-studio/rapoport.studio/pull/432)).

This document is canonical for per-agent model bindings going forward. Per-agent capability specs that ship after this document forward-reference the rows in [§ Per-agent default + fallback](#per-agent-default--fallback) instead of inline-declaring their bindings. Maestro's spec ([`agents/maestro.md § Model binding`](../../current/agents/maestro.md)) carries the historical Phase A inline declaration as a preserved-prose anchor; a dated section at the end of that file declares the inline table superseded by this methodology doc per the `openspec/AGENTS.md` append-at-end convention.

The orchestra context lives in [`openspec/current/agents-overview.md § Roster`](../../current/agents-overview.md). Per-agent specs (`agents/maestro.md` today; Phase B `agents/atlas.md` / `agents/echo.md` / `agents/pulse.md` when they land) are the operational source of truth for each agent's surface; this document is the methodology that informs the model-binding rows of those specs.

---

## Selection criteria

Three principles govern model selection across the orchestra. Each per-agent row in [§ Per-agent default + fallback](#per-agent-default--fallback) traces back to one of them.

**Reliability for coordination and code.** Maestro and Forge run with the highest-tier model because failure compounds across the orchestra. Maestro's bottleneck-cascade risk (per [`agents/maestro.md § Risk matrix`](../../current/agents/maestro.md)) means a single hallucinated routing decision stalls every downstream agent; Forge's reliability-compound risk (per [`forge/verifier.md § Known gaps`](../../current/forge/verifier.md)) means a single hallucinated build step compounds at 95%^N over multi-step chains. For both, Sonnet fallback is acceptable only on availability degradation (rate limit / quota exhaustion), never as a cost-conscious downgrade — Forge in fact has no fallback at all.

**Cost-consciousness for routine.** Muse Builder mode runs on Sonnet by default. Routine spec authoring, code-generation planning, and review work go through Sonnet first; Opus is the escalation path when Sonnet plan output is rejected at a checkpoint review. The 98% Opus quality at fraction of cost ratio (per the original RAP-150 issue body and `forge.md § Builder mode`) is the canonical justification — Sonnet is not a degradation, it is the default for high-volume work.

**Specialised models for specialised work.** Pulse runs on GPT-5.5 Pro (math reasoning leader, FrontierMath benchmark). Anthropic's Claude family is general-purpose strong but does not lead math-heavy financial reasoning. Atlas pairs Perplexity (continuous-research discovery) with Claude Opus (synthesis) — two-stage research where each stage uses the canonical leader for its concern. Echo runs on Claude Opus for brand voice quality on every draft because Pavel-review is a hard gate per [`agents-overview.md § Roster`](../../current/agents-overview.md) — Sonnet-grade voice fidelity would generate more rejections at the gate than the cost savings justify.

---

## Provider catalogue

The orchestra speaks to three providers. Each has a documented purpose and runtime boundary.

| Provider | Purpose | Boundary |
|---|---|---|
| **Anthropic** | Default model calls for general agents (Maestro, Muse Architect, Muse Builder, Muse Scout, Muse Canvas, Echo, Atlas synthesis, Forge) | Wrapped today by [`packages/forge/src/core/anthropic.ts`](../../../packages/forge/src/core/anthropic.ts) and consumed indirectly by Muse runtime via the same client. Direct `fetch('https://api.anthropic.com/v1/messages')`; no SDK. |
| **OpenAI** | Math reasoning leader (Pulse) | Future runtime — provider abstraction lands in Layer 1.5 (`add-provider-abstraction`); Pulse runtime code consumes it. No OpenAI integration exists in code today; the binding is forward-looking. |
| **Perplexity** | Continuous world-research discovery (Atlas first stage) | Future runtime — Atlas Durable Object consumes it. The synthesis stage still routes through Anthropic. No Perplexity integration exists in code today. |

The catalogue declares orchestra-level provider boundaries even before all consumers exist. Phase B agents (Atlas / Echo / Pulse) activate cleanly when their specs and runtimes land — the provider catalogue is in place when they need it.

---

## Per-agent default + fallback

Main table. Includes Phase A (Maestro) and Phase B candidates (Muse modes, Atlas, Echo, Pulse, Forge). Each row corresponds to a capability spec in `openspec/current/agents/` (Maestro) or at parent level (`muse.md`, `forge.md`).

| Agent / mode | Default model | Fallback chain | Reasoning |
|---|---|---|---|
| **Maestro** | `claude-opus-4-8` | `claude-sonnet-4-6` | Coordinator quality compounds across the orchestra (bottleneck cascade per [`agents/maestro.md § Risk matrix`](../../current/agents/maestro.md)). The highest-tier model is required. Sonnet fallback only on availability degradation (Anthropic API rate-limit / quota exhaustion), not cost-conscious downgrade. |
| **Muse — Architect** | `claude-opus-4-8` | `claude-sonnet-4-6` | Spec authoring requires structured-output reliability; the v1 active mode generating `proposal.md` / `design.md` / `tasks.md` triplets in `apps/studio` canvas (per [`muse.md § Architect mode (v1)`](../../current/muse.md)). Sonnet fallback only on availability degradation. |
| **Muse — Builder** | `claude-sonnet-4-6` | `claude-opus-4-8` | 98% Opus quality at fraction of cost (per RAP-150 issue body); cost-conscious by default for routine plan execution via `forge build`. Opus escalation when Sonnet plan output is rejected at a checkpoint review (per [`forge/entities.md § Builder mode`](../../current/forge/entities.md)). |
| **Muse — Scout** | `claude-sonnet-4-6` | `claude-opus-4-8` | Codebase research mode (deferred to v1.3 per [`muse.md § Why other modes are deferred`](../../current/muse.md)); cost-conscious by default — research work is high-volume by nature. Opus escalation for synthesis tasks that require deeper reasoning. |
| **Muse — Canvas** | `claude-opus-4-8` | `claude-sonnet-4-6` | Public-facing client conversations (deferred to v2). Voice quality matters at the public surface; Opus default. Sonnet fallback only on availability degradation. |
| **Maestro mode (Muse-internal)** | (same as Maestro row above) | (same) | Maestro's runtime is implemented as the fifth Muse mode at `packages/muse/src/modes/maestro/` per [`muse.md § Maestro added as fifth mode`](../../current/muse.md). The binding is shared with the agent identity — one row, two reader contexts (the agent spec + the Muse-mode entry). |
| **Atlas — discovery** | Perplexity API | (none) | Continuous-research surveys against the open web; Perplexity is the canonical discovery substrate (web-aware, citation-grounded). No Anthropic fallback because Claude does not have native continuous web access. |
| **Atlas — synthesis** | `claude-opus-4-8` | `claude-sonnet-4-6` | Synthesis of Perplexity discovery findings into orchestra-readable artifacts (`_research/world/<beat>/<date>.md`). Quality matters because synthesis output is what other agents consume; Opus default. |
| **Echo** | `claude-opus-4-8` | `claude-sonnet-4-6` | Brand voice quality on every draft. Pavel-review is hard gate per [`agents-overview.md § Roster`](../../current/agents-overview.md) — voice fidelity must be Opus-grade or rejection rate erodes the cost savings of Sonnet. Sonnet fallback only on availability degradation. |
| **Pulse** | `gpt-5.5-pro` | `claude-opus-4-8` | Math reasoning leader (FrontierMath benchmark per RAP-150 issue body). Anthropic's Claude family does not lead this domain. Claude Opus fallback only on OpenAI availability degradation — Pulse degrades gracefully but never silently (a fallback to Claude is recorded as a degradation event in the orchestra journal once Layer 2 ships). |
| **Forge** | `claude-opus-4-8` | (none) | Coding leader, agent reliability. **No fallback** because cost savings of Sonnet do not justify reliability loss in code execution. The existing default is `MODELS.opus` per [`forge/entities.md § Anthropic API integration`](../../current/forge/entities.md). Sonnet escalation in either direction is a separate decision (e.g. some sub-pipelines like SEO audit could use Sonnet) and lands as a per-pipeline override in `forge.config.{mjs,js,json}`, not as a Forge-default fallback. |

A reader resolving "which model does <agent> use?" follows from this table; per-agent capability specs forward-reference the row by agent name, not by provider ID.

### Forge audit pipeline — per-agent bindings (D-9)

The Forge agent table above declares Forge core's default as `claude-opus-4-8` with no fallback. Inside Forge, the audit pipeline (`forge audit <module>`) fans out 12 specialised agents per run. D-9 permits a Haiku 4.5 carve-out for the explicitly-named low-stakes sub-pipelines below. Forge core, security, risk, and reasoning-heavy audit agents remain on Sonnet/Opus tier.

| Audit agent | Default model | Weight | Reasoning |
|---|---|:-:|---|
| `security` | `claude-sonnet-4-6` | 2.0 | False negative = production security hole; reasoning over diff context |
| `risk` | `claude-sonnet-4-6` | 2.0 | High cost of miss on operational risk; reasoning |
| `architect` | `claude-sonnet-4-6` | 1.5 | Architectural reasoning; long-context judgment |
| `tech-lead` | `claude-sonnet-4-6` | 1.5 | Velocity vs correctness tradeoff judgment |
| `performance` | `claude-sonnet-4-6` | 1.5 | Runtime reasoning, not pattern matching |
| `component-api` | `claude-sonnet-4-6` | 1.5 | API contract analysis; reasoning |
| `qa` | `claude-sonnet-4-6` | 1.0 | Test-coverage reasoning |
| `seo` | `claude-haiku-4-5` | 1.5 | Pattern matching: readability, keywords (D-9) |
| `growth` | `claude-haiku-4-5` | 1.0 | Analytics pattern checks (D-9) |
| `pm` | `claude-haiku-4-5` | 1.0 | Product lens; heuristic classification (D-9) |
| `ui-designer` | `claude-haiku-4-5` | 1.0 | Visual pattern checks (D-9) |
| `a11y` | `claude-sonnet-4-6` (staged Haiku candidate) | 1.5 | WCAG rule-checking; promoted to Haiku after one studio audit cycle confirms no regression. C-2 in `improve-forge-throughput` keeps a11y on Sonnet for v1 (D-9, staged) |

Per-pipeline override at consumer site: `ForgeConfig.models.auditAgents?: Partial<Record<AgentName, ModelAlias>>` takes precedence over module-registry defaults. Forge core (Builder, plan generation) is unaffected — D-7 unchanged.

---

## Model identifier table

Maps the row labels in [§ Per-agent default + fallback](#per-agent-default--fallback) to canonical provider IDs and pricing sources. Identifier strings are not stable across model generations — when a new identifier is released, this table updates; the row labels in § Per-agent default + fallback stay stable; per-agent specs continue to forward-ref the row label.

| Row label | Provider | Provider ID | Pricing source |
|---|---|---|---|
| `claude-opus-4-8` | Anthropic | TBD per Anthropic identifier registry (currently the `MODELS.opus` constant in `packages/forge/src/core/anthropic.ts`) | [`MODEL_PRICING`](../../../packages/forge/src/core/anthropic.ts) in `packages/forge/src/core/anthropic.ts § initModelPricing` (extensible per RAP-166 / [`forge/verifier.md § Known gaps #6`](../../current/forge/verifier.md)) |
| `claude-sonnet-4-6` | Anthropic | TBD per Anthropic identifier registry (currently the `MODELS.sonnet` constant in `packages/forge/src/core/anthropic.ts`) | same |
| `claude-haiku-4-5` | Anthropic | `claude-haiku-4-5-20251001` (the `MODELS.haiku` constant in `packages/forge/src/core/anthropic.ts`) | [`MODEL_PRICING`](../../../packages/forge/src/core/anthropic.ts) in `packages/forge/src/core/anthropic.ts § initModelPricing` — entry `'claude-haiku-4-5-20251001': { input: 0.5, output: 2.5 }` |
| `gpt-5.5-pro` | OpenAI | TBD per OpenAI identifier registry | TBD when provider abstraction (`add-provider-abstraction` Layer 1.5) ships an OpenAI client |
| Perplexity API | Perplexity | TBD (Perplexity Sonar or equivalent for continuous-research mode) | TBD when Atlas runtime ships and the Perplexity client is wired |

The pricing column references existing constants where they exist (`MODEL_PRICING` for Anthropic) and declares "TBD" for providers that don't have runtime yet. Per [`add-tools-methodology` design.md § D-6](../../archive/2026-05-09-add-tools-methodology/design.md), pricing details are operational state that lives with the runtime constant — this methodology doc cross-references, it does not duplicate.

---

## Update cadence

Discipline for keeping this document current.

**New model identifier release.** When Anthropic / OpenAI / Perplexity release a new identifier, update [§ Model identifier table](#model-identifier-table) Provider ID column to reflect the new ID. Do **not** change row labels in [§ Per-agent default + fallback](#per-agent-default--fallback) — those are stable cross-references that per-agent specs forward-ref. The row label is the methodology contract; the identifier is the runtime detail.

**New agent activates.** When a new agent ships a capability spec (e.g. Phase B `agents/atlas.md` or a future seventh agent), add a row to [§ Per-agent default + fallback](#per-agent-default--fallback) with default + fallback + reasoning. The per-agent spec's `## Model binding` section forward-refs the new row; no inline declaration.

**Provider boundary changes.** When the orchestra adopts a new provider (e.g. Cohere, Mistral) or retires an existing one, update [§ Provider catalogue](#provider-catalogue). The provider catalogue is the orchestra-level statement; per-agent rows in § Per-agent default + fallback follow.

**Reasoning becomes outdated.** If the underlying selection criteria change (e.g. Sonnet matches Opus quality on coordination, or a new model class supersedes the current leader), propose a separate change with the binding update. The binding does not change silently — every per-agent row carries its own reasoning, and changes to that reasoning are reviewed via the standard prose-style proposal.

**Cross-reference drift.** If a per-agent capability spec amends its `## Model binding` forward-ref to point at a renamed row (e.g. row label change due to provider rebrand), audit other forward-refs at the same time. The `node tools/check-openspec-refs.mjs` guard (see [`openspec/AGENTS.md`](../../AGENTS.md)) does not check methodology-doc anchors, only `openspec/{changes,archive}/<slug>/...` paths — methodology-doc anchor health is a manual verification step at amendment time.

---

## Decisions captured

| # | Decision | Rationale |
|---|---|---|
| D-1 | Single source of truth at `_methodology/tools/agent-model-mapping.md` | One place to update on new model releases; per-agent specs forward-ref by row label; eliminates drift across N per-agent files |
| D-2 | Row labels in § Per-agent default + fallback are stable cross-references; provider IDs in § Model identifier table are mutable | Per-agent specs forward-ref the row label; identifier strings update independently as Anthropic / OpenAI release new model versions |
| D-3 | Provider catalogue declares orchestra-level boundaries even before consumers exist | Phase B agents activate cleanly when their specs and runtimes land; OpenAI and Perplexity rows present today even though no agent uses them yet |
| D-4 | Forge has no fallback (reliability compound dominates cost) | Sonnet's cost savings do not justify reliability loss in code execution; per `forge.md § Known gaps` reliability-compound failure mode |
| D-5 | Pulse uses GPT-5.5 Pro by default, not Claude | Math reasoning leader (FrontierMath benchmark); Anthropic's Claude family does not lead this domain; degradation to Claude Opus only on OpenAI availability failure |
| D-6 | Atlas uses two-stage pipeline (Perplexity discovery + Claude synthesis) | Each stage uses the canonical leader for its concern; Perplexity is web-aware, Claude is synthesis-strong; no single-provider single-call alternative ships the same quality |
| D-7 | Cost-conscious models acceptable for Builder / Scout (Sonnet default), not for Echo / Architect / Maestro / Forge | Cost-consciousness applies to high-volume routine work; agents whose output enters a Pavel-review hard gate (Echo) or compounds across the orchestra (Maestro, Forge) demand Opus-grade quality |
| D-8 | Methodology doc out of bundle scope | Per `add-philosophy-foundation § D-12`; runtime UI does not need methodology content in v1; bundle generator unmodified by this change |
| D-9 | Haiku 4.5 acceptable for explicitly-named low-stakes Forge audit sub-pipelines | Forge audit pipeline fans out 12 agents per run; not all 12 require Opus/Sonnet reasoning depth. SEO, growth, PM, UI-designer, and (staged) A11Y are pattern-matching tasks where Haiku 4.5 (73.3% SWE-bench, 1/3 cost vs Sonnet) is sufficient. Forge core (Builder, plan generation, security/risk audit agents) remains Sonnet/Opus tier per D-7. Carve-out is enumerated, not blanket — new Haiku transitions require their own decision row. |
| D-10 | Top-tier identifier bumped to `claude-opus-4-8` (2026-05-28, RAP-1253) | Anthropic released Opus 4.8 as the new top-tier GA model with no breaking API changes from 4.7. Per-agent row labels (left column) unchanged; identifier strings updated per D-2. Historical `claude-opus-4-7` rows retained in `MODEL_PRICING`, `MODEL_ID_TO_ALIAS`, capability regexes, and `CONTEXT_LIMITS` so prior transcripts continue to resolve. Authoritative proposal/design/tasks: `openspec/archive/2026-05-28-upgrade-top-tier-to-opus-4-8/`. |

---

## Related capabilities

- [`openspec/current/agents-overview.md`](../../current/agents-overview.md) — orchestra-level facts (roster, communication graph, storage ownership). The methodology doc is the model-binding layer of the same orchestra context.
- [`openspec/current/agents/maestro.md`](../../current/agents/maestro.md) — Maestro capability spec; the Phase A inline `§ Model binding` table is preserved as historical anchor; the dated `§ Model binding superseded by methodology doc` section at end of file declares this methodology doc canonical going forward.
- [`openspec/current/muse.md`](../../current/muse.md) — Muse capability spec; the per-mode bindings (Architect / Builder / Scout / Canvas) live in the Per-agent default + fallback table here; muse.md describes mode behaviour, this doc describes mode bindings.
- [`openspec/current/forge/README.md`](../../current/forge/README.md) — Forge capability spec; the Forge row above forward-refs to the existing `MODELS.opus` constant and the `MODEL_PRICING` table here references it.
- [`packages/forge/src/core/anthropic.ts`](../../../packages/forge/src/core/anthropic.ts) — runtime constants (`MODELS`, `MODEL_PRICING`, `initModelPricing`); single source of truth for pricing rates per RAP-166.
- [`openspec/_methodology/studio-charter.md`](../studio-charter.md) — operating-model charter; the methodology doc is consumed by spec authors who reference it from per-agent specs.
- Future: `_methodology/tools/<future-doc>.md` — additional tool / capability methodology docs land in this same subdirectory as the orchestra grows.

---

## Haiku audit carve-out (applied 2026-05-11 from change `add-forge-haiku-audit-tier`)

D-9 carve-out for low-stakes Forge audit sub-pipelines (`seo`, `growth`, `pm`, `ui-designer`; `a11y` staged). Methodology decision row and Forge audit sub-table added. Authoritative proposal/design/tasks: `openspec/archive/2026-05-11-add-forge-haiku-audit-tier/` — this marker is pointer-only.
