---
title: Orchestra Component Stack — Layers, Picks, Access, Risks
status: research
audience: [architect, founder, ai-engineer]
date: 2026-05-12
inputs:
  - openspec/_methodology/research/orchestra-training-architecture-2026.md
  - openspec/_methodology/research/training-orchestra-on-openspec-format-2026.md
  - openspec/_methodology/research/cli-tech-stack-comparison-2026.md
  - openspec/_methodology/research/access-and-context-hierarchy-2026.md
  - openspec/_methodology/research/ai-cli-landscape-2026-05.md
  - openspec/_methodology/research/nodejs-ai-frameworks-landscape-2026.md
  - openspec/_methodology/research/ai-risk-frameworks-and-indexing.md
  - openspec/_methodology/research/sdd-terminology-scorecard.md
  - openspec/_methodology/research/13-schools-of-thought.md
---

# Orchestra Component Stack — Layers, Picks, Access, Risks

## TL;DR

- The orchestra is **8 layers × ~30 components**. This document is the parts list — every component, its role, options, risks, and pick.
- **Information hierarchy is universal in 2026**: descriptions hot, bodies warm, references cold (the Anthropic Skills pattern, now convergent across Claude Code / Cursor / Codex / Continue / Windsurf via AGENTS.md).
- **Access model**: capability-based per CLI, deny-by-default, role-specific prompt prefixes above shared corpus, per-agent secret paths in Infisical. Generalize Maestro's existing `Access scope` allowlist pattern as methodology contract for all three CLIs.
- **Biggest stack risk**: UnJS ecosystem bus factor (citty/c12/consola cluster). Mitigation: wrap in `packages/cli-core/` so call sites see *our* API.
- **Biggest security threat**: confused-deputy via shared `GITHUB_BOT_TOKEN`/`LINEAR_BOT_API_KEY` (already the shape that produced Meta Sev-1 March 2026 and Vercel OAuth incident April 2026). Per-agent path-scoped tokens, audience-bound per RFC 8707.

---

## The full architectural picture

```
┌───────────────────────────────────────────────────────────────────────────┐
│                         L7 — GOVERNANCE                                   │
│   glossary.md │ red-lines.md │ risk-register.md │ audience-policy.md     │
│   ─ canon ─    ─ classifier ─  ─ matrix ─        ─ Diataxis tags ─       │
└──────────────────────────────────┬────────────────────────────────────────┘
                                   │ informs all below
┌──────────────────────────────────┴────────────────────────────────────────┐
│                         L6 — TELEMETRY & LEARNING                         │
│  events.jsonl  │  lineage registry  │  Langfuse  │  DSPy+GEPA+BAML loop  │
│  (pino JSONL)    (per-Linear-id)      (self-hosted) (compile, not RL)    │
└──────────────────────────────────┬────────────────────────────────────────┘
                                   │ measurement substrate
┌──────────────────────────────────┴────────────────────────────────────────┐
│                         L5 — EXECUTION (Forge)                            │
│  Coordinator → Implementor → Verifier │ build-lock │ worktree manager    │
│  Octokit (Forge-core)                  │ exec via execa                  │
└──────────────────────────────────┬────────────────────────────────────────┘
                                   │ artifacts
┌──────────────────────────────────┴────────────────────────────────────────┐
│                         L4 — VALIDATION                                   │
│  Vale (prose)  │  cross-family LLM judge  │  promptfoo (CI)  │ golden set│
│  ─ deterministic ─ ─ Gemini/GPT-5 judge ─  ─ regression ─    ─ 30-50 ─   │
└──────────────────────────────────┬────────────────────────────────────────┘
                                   │ gate
┌──────────────────────────────────┴────────────────────────────────────────┐
│                         L3 — GENERATION (Muse)                            │
│  Vercel AI SDK v6  │  Zod / BAML(pilot)  │  persona templates            │
│  Opus 4.7 architect │ Sonnet planner     │  Haiku 4.5 low-stakes audit   │
└──────────────────────────────────┬────────────────────────────────────────┘
                                   │ assembled context
┌──────────────────────────────────┴────────────────────────────────────────┐
│                         L2 — CONTEXT ASSEMBLY                             │
│  L1 system (5m) │ L2 tools/skills (5m) │ L3 corpus (1h) │ L4 dynamic     │
│  ─ hot ──────────  warm ─────────────── cool ──────────── ephemeral ─    │
└──────────────────────────────────┬────────────────────────────────────────┘
                                   │ assembled by
┌──────────────────────────────────┴────────────────────────────────────────┐
│                         L1 — INDEXING                                     │
│  bundle (gzip ≤6MB) │ repo-map.json │ SKILL.md descriptions │ INDEX.md   │
│  ─ build-openspec-bundle.mjs ─       ─ hash-pinned ─        ─ ≤1KB ─     │
└──────────────────────────────────┬────────────────────────────────────────┘
                                   │ reads
┌──────────────────────────────────┴────────────────────────────────────────┐
│                         L0 — STORAGE                                      │
│  openspec/{current,changes,archive,_methodology} │ git │ R2 (cold)        │
│  pre-commit hook: archive frozen, current edit-via-change-only           │
└───────────────────────────────────────────────────────────────────────────┘

      ───────── CROSS-CUTTING ──────────
      Secrets: Infisical (per-agent paths)
      Auth: Supabase / Anthropic / GitHub PAT / Linear API
      Distribution: npm (today) → Node SEA (v2)
      Inter-CLI: filesystem + events.jsonl + capability registry
```

This is the contract. Every component below fits into exactly one of these slots.

---

## Component inventory by layer

For each component: **role / top options / pick / risks**. Full comparison tables and verified numbers live in [cli-tech-stack-comparison-2026.md](cli-tech-stack-comparison-2026.md).

### L0 — Storage

| # | Component | Role | Pick | Why | Top risk |
|---|---|---|---|---|---|
| 0.1 | File store | Authoritative spec storage | local fs + git | already in place | accidental edits to archive |
| 0.2 | Version control | Audit log + branching | git + GitHub | already in place | force-push to main (mitigated by branch protection) |
| 0.3 | Archive enforcement | Make `openspec/archive/` immutable | pre-commit hook (lefthook) | deterministic, fast | bypass via `--no-verify` (mitigate: require server-side rejection too) |
| 0.4 | Cold storage | Old archives, large research | Cloudflare R2 | already on CF, cheap | none material |

### L1 — Indexing

| # | Component | Role | Pick | Top alternatives | Top risk |
|---|---|---|---|---|---|
| 1.1 | Spec bundle | Single artifact loaded into prompt | existing `tools/build-openspec-bundle.mjs` | none needed | stale (mitigate: regen hook) |
| 1.2 | Repo map | Routing index: path → capability/audience/hash | new — write as `tools/build-repo-map.mjs` | none | drift from filesystem |
| 1.3 | Skills index | `≤1024-char description` per skill | Anthropic SKILL.md format | none — this is *the* standard | description over-fits to single use case |
| 1.4 | INDEX.md per capability | Progressive disclosure entry point | author manually per capability | none | drift; mitigation: Vale rule |
| 1.5 | Vector store (deferred) | Semantic retrieval when bundle exceeds 200K tokens | pgvector on Supabase when triggered | sqlite-vec for local-only, LanceDB | premature optimization |

### L2 — Context Assembly

| # | Component | Role | Pick | Risks |
|---|---|---|---|---|
| 2.1 | Prompt cache | 4-layer Anthropic cache | Anthropic native (5m/5m/1h) | TTL changes (silent regression Mar 2026 — track changelog) |
| 2.2 | Stable-prefix manager | Ensure cache-friendly ordering | implement in `cli-core/context/` | edge cases break cache invisibly |
| 2.3 | Skills loader | Read SKILL.md, expose description, lazy body load | implement in `cli-core/context/` | unbounded body size; cap at 500 lines per Anthropic guidance |
| 2.4 | MCP client | Tool exposure to other agents | `@modelcontextprotocol/sdk` (official TS) | protocol velocity (track spec) |

### L3 — Generation (Muse)

| # | Component | Role | Pick | Risks |
|---|---|---|---|---|
| 3.1 | Inference SDK | LLM call primitive | **Vercel AI SDK v6** (`@ai-sdk/anthropic`) | API surface churn; mitigate via `cli-core/llm/` wrapper |
| 3.2 | Model router | Pick model per role | implement in `cli-core/llm/router.ts` | hard-coded names break on rotation |
| 3.3 | Schema validation | Runtime types | **Zod v3** (full) | bundle size (acceptable for CLI; matters less for Workers) |
| 3.4 | LLM-output schema | Constrain LLM JSON output | Zod for v1; **BAML pilot** on Forge plan-bundle | BAML lock-in if it works; mitigate with adapter layer |
| 3.5 | Persona templates | Architect / Storyteller / Planner roles | author manually under `packages/muse/prompts/` | persona drift; mitigate with prompt versioning |
| 3.6 | Prompt versioning | Track which prompt produced which output | hash-stamp in events.jsonl + Langfuse | un-reproducible reruns |

### L4 — Validation

| # | Component | Role | Pick | Risks |
|---|---|---|---|---|
| 4.1 | Prose linter | Deterministic style + terminology gate | **Vale** with custom YAML rules | rule maintenance overhead |
| 4.2 | LLM judge | Probabilistic quality gate, cross-family | Gemini 2.5 Pro **or** GPT-5 (NOT same-family) | preference leakage (ICLR-2026); cross-family mitigates |
| 4.3 | Eval harness | Regression suite over golden set | **promptfoo** (TS-native CLI) | TS-only ecosystem; DeepEval is Python-only (block) |
| 4.4 | Annotation/trace UI | Heavy eval + manual review | **Langfuse** (self-hosted) | maintenance overhead |
| 4.5 | Golden set | Graded examples for regression | author manually as `tools/evals/golden/*.json` | staleness (refresh quarterly) |
| 4.6 | Red-line classifier | Pre-generation routing | small Haiku 4.5 call with explicit class list | classifier drift; track FNs |

### L5 — Execution (Forge)

| # | Component | Role | Pick | Risks |
|---|---|---|---|---|
| 5.1 | CLI framework | Argument parsing, sub-commands | **citty** (UnJS, lazy sub-command load) | UnJS bus factor (mitigate: wrap in cli-core) |
| 5.2 | Config loader | Layered flags > env > file > defaults | **c12** (UnJS) | same — wrap |
| 5.3 | Logger | Human (TTY) + structured (pipe) | **consola** for UX, **pino** under hood for events.jsonl | dual-stack complexity |
| 5.4 | Process orchestration | Spawn subprocesses, stream output | **execa v9** (already in Forge) | none material; reject zx (shell injection risk) |
| 5.5 | File locks | Serialize build operations | **proper-lockfile** | unmaintained but tiny API; fork is 1-engineer-week if breaks |
| 5.6 | Worktree manager | Per-build isolated checkout | `git worktree` + thin wrapper in cli-core | bare `git worktree add` collisions (already documented) |
| 5.7 | Builder/orchestrator | Coordinator-Implementor-Verifier | existing Forge pattern | builder/orchestrator commit race (documented memory) |
| 5.8 | GitHub client | PRs, issues, reviews | **Octokit** (wrapped in `forge/core`) | rate limits (Octokit handles) |
| 5.9 | Linear client | Issue tracking | **Linear SDK** (only one) | webhook handling |

### L6 — Telemetry & Learning

| # | Component | Role | Pick | Risks |
|---|---|---|---|---|
| 6.1 | Event log | Append-only event stream | **JSONL on disk + pino** | log rotation; mitigate with daily files |
| 6.2 | Schema versioning | Future-proof event types | manual `schemaVersion: N` per event | drift; mitigate via Zod parse on read |
| 6.3 | Lineage registry | Per-idea trace aggregation | new `~/.rapoport/lineage/<RAP-NN>.jsonl` | duplicate data with events.jsonl (acceptable) |
| 6.4 | LLM tracing | Prompt/response/cost capture | **Langfuse** (self-hosted) | infra burden |
| 6.5 | Metrics aggregation | Derive cost/speed/quality | command in cli-core: `rapoport metrics` | none material |
| 6.6 | Training loop | Compile prompts from examples | **DSPy + GEPA + BAML** (when golden set exists) | over-fitting to judge; split corpora |
| 6.7 | OTel exporter | Optional GenAI semconv | OpenTelemetry SDK JS | premature; wait for stability |

### L7 — Governance

| # | Component | Role | Pick | Risks |
|---|---|---|---|---|
| 7.1 | Glossary | Bridge terminology canon | author + maintain `openspec/_methodology/glossary.md` | drift from actual usage |
| 7.2 | Red-lines list | 7 hard categories | author `openspec/_methodology/red-lines.md` | classifier accuracy |
| 7.3 | Risk register | 5-tier consolidated matrix | author `openspec/_methodology/risk-register.md` | becomes theatre without telemetry tie-in |
| 7.4 | Audience policy | Diataxis assignment per artifact | author `openspec/_methodology/audience-policy.md` | none material |
| 7.5 | Access scope contract | Per-agent allowlists | author `openspec/_methodology/agent-access-scope.md` (generalize Maestro pattern) | adoption friction; mitigate by making it template-driven |

### Cross-cutting

| # | Component | Role | Pick | Risks |
|---|---|---|---|---|
| C.1 | Secrets | Per-agent token paths | **Infisical** with paths `/forge/`, `/muse/`, `/maestro/` | one-token-rules-all anti-pattern |
| C.2 | Auth (user) | Magic-link login for studio | **Supabase Auth** (already in place) | none material |
| C.3 | Pre-commit | Enforce L0 invariants + Vale gate | **lefthook** | bypass via `--no-verify`; mitigate server-side |
| C.4 | Distribution v1 | Internal use | **npm** under `@rapoport-studio/*` | install friction for external users |
| C.5 | Distribution v2 | External binaries | **Node SEA** (`--build-sea`, Node 25.5+) | bundle size; cold start |
| C.6 | Skills exporter | Methodology → SKILL.md | new tool in `tools/skills-export.mjs` | format spec velocity |
| C.7 | MCP server | Expose orchestra to Claude Code/Cursor | `@modelcontextprotocol/sdk` | spec version churn |
| C.8 | Inter-CLI capability registry | `--capabilities` discovery | implement in cli-core | schema drift |

---

## Information hierarchy — "what's on top"

**The convergent 2026 pattern**: descriptions hot, bodies warm, references cold. Anthropic Skills lock this in, Claude Code adopts it via CLAUDE.md walk, Cursor / Codex / Continue / Aider / OpenHands / Windsurf all converge on AGENTS.md "closest wins".

For Rapoport, this means **three context tiers per CLI**:

### Tier 1 — HOT (in every prompt, system role)

Loaded into the L1 cache layer (system + role, 5-min TTL). Same across the session.

| Content | Size budget | Source |
|---|---|---|
| Role definition + system prompt | ~1–3K tokens | `packages/<cli>/prompts/system.md` |
| Glossary (10 collision terms only) | ~500 tokens | `openspec/_methodology/glossary.md` excerpt |
| Red-lines categories (names only) | ~200 tokens | `openspec/_methodology/red-lines.md` headers |
| Skill descriptions (`≤1024 chars` each) | ~2K tokens | `<cli>/skills/*/SKILL.md` frontmatter |
| Capability inventory (names + audiences) | ~500 tokens | `openspec/current/INDEX.md` |
| Active change list (slugs + titles) | ~300 tokens | derived from `openspec/changes/` |

**Total hot ≤ 5K tokens.** This is what the CLI "knows by heart."

### Tier 2 — WARM (loaded on intent match, L2-L3 cache)

| Content | When loaded | Cache tier |
|---|---|---|
| Full SKILL.md body (≤500 lines) | When skill triggered | L2 (5m) |
| Specific capability spec | When change touches it | L3 (1h) |
| Specific archived change | When pattern-matched | L3 (1h) |
| Research file excerpt | When research-relevant intent | L3 (1h) |
| Sibling code context | When implementing | L3 (1h) |

### Tier 3 — COLD (loaded only on explicit reference)

| Content | When loaded | Notes |
|---|---|---|
| Full research file (10K+ tokens) | Explicit URL/path mention | No cache — load fresh |
| Full git history of a file | Explicit `git log` request | No cache |
| External docs (Anthropic, Linear) | Explicit external lookup | No cache |
| Old archived changes (>90d) | Explicit retrieval | No cache |

### Per-CLI specialization of the same hierarchy

The shape is identical for all three CLIs; **what fills it differs by role**.

| Slot | Forge (executor) | Muse (author) | Maestro (orchestrator) |
|---|---|---|---|
| Role prompt | "You write code from a spec" | "You draft specs from intent" | "You observe events, surface trends, escalate" |
| Hot skills | `forge build`, `forge plan`, `forge verify` | `muse draft`, `muse refine`, `muse persona` | `maestro lineage`, `maestro metrics`, `maestro alert` |
| Hot capabilities | All active capabilities (read) | Same | Same |
| Warm research | code-validity, trust-gradient, frameworks | terminology, organization patterns, multi-audience | risk frameworks, ai-cli landscape |
| Cold | full archives, external API docs | full methodology research | external observability docs |

**Critical:** the L3 corpus cache (1h) is **shared across all three CLIs**, because the spec content is the same. Only the role prefix (Tier 1) differs. This is what makes the cache strategy work — shared L3, separate L1.

---

## Cache hierarchy alignment

| Cache tier | TTL | Content | Cardinality | Who owns it |
|---|---|---|---|---|
| **L1 system** | 5 min | Role prompt + glossary + red-lines headers | **3** (one per CLI) | each CLI |
| **L2 tools/skills** | 5 min | Tool descriptions + skill frontmatter | **~50 entries** | cli-core |
| **L3 corpus** | 1 hour | OpenSpec bundle (current + changes + recent archive) | **1 shared** | cli-core |
| **L4 dynamic** | none | Current task, live state, recent events | per-request | each CLI |

**Invalidation rules:**

- L1 invalidates only when role prompt changes (rare; treat as breaking change)
- L2 invalidates when skill manifest hash changes (per-skill granularity, not whole-corpus)
- L3 invalidates when bundle hash changes (whole-bundle today; **research opportunity**: Merkle-tree per-file invalidation, see [orchestra-training-architecture-2026.md § L1 research needed](orchestra-training-architecture-2026.md))
- L4 never caches

**Why bundle-ordering matters** (Lost-in-the-Middle, Liu TACL 2024): same content recalled 30+ pp worse at position 10 vs position 1 in a 20-doc context. Bundle should order: **most-stable first, most-relevant last** — stable prefix maximizes cache reuse, relevant suffix maximizes attention.

---

## Per-CLI access matrix

Generalizing the [Maestro spec § Access scope](openspec/current/agents/maestro.md) pattern across all three CLIs.

| Resource | Forge | Muse | Maestro |
|---|---|---|---|
| `openspec/current/` (read) | ✅ all | ✅ all | ✅ all |
| `openspec/current/` (write) | ❌ never | ❌ never (only via change merge) | ❌ never |
| `openspec/changes/<slug>/` (read) | ✅ assigned only | ✅ assigned + drafting | ✅ all (telemetry) |
| `openspec/changes/<slug>/` (write proposal/design/tasks) | ❌ never | ✅ assigned | ❌ never |
| `openspec/changes/<slug>/` (write code-side artifacts) | ✅ assigned | ❌ never | ❌ never |
| `openspec/archive/` (read) | ✅ all | ✅ all | ✅ all |
| `openspec/archive/` (write) | ❌ never | ❌ never | ❌ never |
| `openspec/_methodology/` (read) | ✅ relevant subset | ✅ all | ✅ all |
| `openspec/_methodology/research/` (read) | ✅ by capability tag | ✅ all | ✅ all |
| `openspec/_methodology/` (write) | ❌ never | ❌ never (via change) | ❌ never |
| Application code (`apps/`, `packages/`) read | ✅ all | ⚠️ read for context only | ⚠️ read for context only |
| Application code write | ✅ via PR | ❌ never | ❌ never |
| `events.jsonl` (write) | ✅ emit | ✅ emit | ✅ emit |
| `events.jsonl` (read) | ⚠️ own only | ⚠️ own only | ✅ all |
| `~/.rapoport/lineage/` (write) | ⚠️ append to existing | ⚠️ append to existing | ✅ create + append |
| GitHub: open PR | ✅ | ❌ | ❌ |
| GitHub: comment | ✅ | ⚠️ on own change PRs | ✅ on telemetry alerts |
| Linear: read issue | ✅ assigned | ✅ assigned | ✅ all |
| Linear: write status/comment | ✅ on own work | ✅ on own work | ✅ on alerts |
| Anthropic API | ✅ scoped key | ✅ scoped key | ✅ scoped key |
| Infisical (secrets) | `/forge/*` only | `/muse/*` only | `/maestro/*` only |
| Supabase | ❌ never directly | ❌ never directly | ⚠️ read-only via service role for analytics |

Legend: ✅ allowed by default · ⚠️ allowed with constraint · ❌ deny by default.

**Why per-agent Infisical paths matter**: this is the structural fix for the **confused-deputy** class of incidents (Meta Sev-1 March 2026, Vercel OAuth April 2026, Comment-and-Control attacks on Claude Code via GitHub Issues). One token scoped to the *union* of what any agent might need = blast radius of the worst agent. Per-path tokens = blast radius of one agent.

---

## Risk matrix (consolidated, 5-tier)

| Tier | Risk class | Specific risk | Source | Mitigation layer |
|---|---|---|---|---|
| **L1 Regulatory** | EU AI Act transparency | Auto-generated spec labeled as human-authored | EU AI Act §52 | L7 policy + L6 metadata stamp on every artifact |
| **L1 Regulatory** | GDPR data flows | Anthropic processing client data without DPA | EU GDPR | L7 DPA inventory; no PII in prompts |
| **L2 Security** | Confused deputy via shared token | One `GITHUB_BOT_TOKEN` used by all CLIs | Meta Sev-1 Mar 2026 | C.1 per-agent Infisical paths |
| **L2 Security** | Tool poisoning via MCP description | Untrusted SKILL.md instructions injected | Postmark-MCP rug-pull Sep 2025; Cursor CVE-2025-54136 | L7 review-gate; never auto-load external skills |
| **L2 Security** | Comment-and-control | GitHub Issue body contains hidden instructions, agent acts on it | Claude Code / Gemini CLI exploits 2025 | L4 prompt injection filter on issue content |
| **L2 Security** | Cache poisoning of L3 corpus | Poisoned research file affects all three CLIs | hypothetical, but template matches | L0 archive-frozen hook; L7 OpenSpec change gate on research files |
| **L3 SDD red lines** | Auto-build of RLS / auth / concurrent | Forge writes broken security code from incomplete spec | trust-gradient plateau 45% | L7 red-line list + L3 routing |
| **L3 SDD red lines** | Spec poisoning | Adversarial spec edit changes generated code semantics | hypothetical | L4 diff review; L6 lineage detects unusual spec edits |
| **L4 Methodology** | Terminology drift | "Studio" used ambiguously; "Eval" cross-school collision | sdd-terminology-scorecard | L4 Vale rules from L7 glossary |
| **L4 Methodology** | Judge sycophancy / preference leakage | Same-family judge inflates author scores by 5-15pp | SycEval, ICLR-2026 | L4 cross-family judge |
| **L4 Methodology** | Golden-set staleness | Examples trivial for new model | empirical | quarterly refresh ritual |
| **L4 Methodology** | UnJS ecosystem bus factor | citty + c12 + consola cluster on one org | maintenance risk | wrap in `cli-core/` so call sites see Rapoport API |
| **L5 Implementation** | Builder/orchestrator race | `aborted-no-changes` outcome despite real PR | existing Forge memory | cross-check `gh pr view`; refactor outcome source |
| **L5 Implementation** | Stale bundle | events.jsonl logs against last-build's corpus | existing memory | L0 regen hook |
| **L5 Implementation** | Lock file orphaned | Ctrl-C leaves lock; next build fails | existing Forge memory | SIGINT cleanup; PID validation in proper-lockfile |

The matrix is read **bottom-up for engineering** (fix L5 first) and **top-down for governance** (L1 sets the priority).

---

## Findings

> Structured index added 2026-05-29 as part of
> [`inquiry-drift-coverage-retrieval-spike`](./inquiry-drift-coverage-retrieval-spike.md)
> F-7 fix extension. Grade vocabulary per `add-inquiry-entity` design.md §3
> `InquirySource.credibility` enum.
>
> **Capability:** forge, muse, maestro, cli-core

| # | Claim | Grade | Falsifier |
|---|---|---|---|
| F-1 | Per-agent Infisical paths are the structural fix for confused-deputy attacks (Meta Sev-1 March 2026, Vercel OAuth April 2026, Comment-and-Control attacks on Claude Code via GitHub Issues). One token scoped to the union of any agent's needs = blast radius of worst agent. Per-path tokens = blast radius of one agent. | engineering_claim | A confused-deputy incident occurring after per-path tokens deploy would falsify the structural-fix claim. |
| F-2 | Forge's L5 implementation has documented race: `aborted-no-changes` outcome despite real PR (builder/orchestrator commit race in existing Forge memory). Mitigation: cross-check `gh pr view`, refactor outcome source. | engineering_claim | A clean run sequence where outcome source is `state.json` AND matches `gh pr view` for ≥10 consecutive builds would falsify the persistence of the race. |
| F-3 | Stack picks at L5 are bus-factor-risky: citty + c12 + consola cluster on the UnJS org. Wrap in `cli-core/` so call sites see Rapoport API, not direct UnJS imports — insulates against UnJS maintainer disappearance. | engineering_claim | A measurable shift away from UnJS maintenance velocity (e.g., 0 commits over 90 days across the cluster) would activate the falsifier and force migration off the picks. |
| F-4 | Vector store (pgvector on Supabase) is correctly deferred until corpus > 200K tokens. Current corpus (~80 research files + openspec bundle) sits well below; embedded fallback (LanceDB) would be overkill. | engineering_claim | Corpus growing past 200K tokens AND grep-based retrieval producing >50% false-positive rate on capability-scoped queries would force the vector-store pull-forward. |
| F-5 | Judge sycophancy / preference leakage: same-family judge (e.g., Claude-judges-Claude) inflates author scores by 5-15pp (SycEval, ICLR-2026). Mitigation: cross-family judge (e.g., Gemini judges Claude output). | experimental | A controlled run where same-family judging produces scores within ±2pp of cross-family judging across ≥50 evaluations would falsify the sycophancy magnitude. |
| F-6 | Vale prose linter is the right L4 pick but **mandatory enforcement creates reviewer friction without first measuring false-positive rate**. Advisory mode (warn, don't fail) for first month, then enforcing only if FPR <10%. | engineering_claim | A Vale-mandatory rollout with FPR ≥20% would be the evidence that pre-measurement advisory mode was the correct staging. |
| F-7 | LLM-output schema constraint: BAML pilot makes sense for one hot path (e.g., Forge plan-builder) before broader adoption. Zod alone covers v1; BAML's schema-aligned parsing is a measurable upgrade only when the cost of schema-rejection is high. | engineering_claim | BAML pilot on Forge plan-builder yielding ≥10% reduction in plan-validation errors vs Zod-only baseline would justify broader adoption. Sub-10% would refute. |
| F-8 | Local-only `lineage/` (under `~/.rapoport/lineage/`) is correct until a second contributor appears. Central sync (e.g., Supabase) adds attack surface without clear use case at solo-cadence. | engineering_claim | A second contributor onboarding within 90 days AND reporting difficulty querying their counterpart's lineage would trigger pull-forward. |
| F-9 | A2A protocol adoption deferred until a concrete inter-vendor scenario exists (e.g., Devin/Factory/Conductor integration). MCP is the proven distribution channel; A2A's ecosystem is thin in 2026. | engineering_claim | A vendor-adoption event making A2A the primary protocol for Anthropic / Google / OpenAI compatibility within 6 months would falsify the deferral. |

## Consolidated stack recommendation

| Layer | Component | Pick | Alternative if pick fails | Defer until |
|---|---|---|---|---|
| L0 | File store | local fs + git | — | — |
| L0 | Archive enforcement | lefthook pre-commit | husky | — |
| L1 | Bundle | existing tool | — | — |
| L1 | Repo map | custom `tools/build-repo-map.mjs` | — | — |
| L1 | Vector store | — (defer) | pgvector on Supabase | corpus > 200K tokens |
| L2 | Prompt cache | Anthropic native | — | — |
| L2 | MCP client | `@modelcontextprotocol/sdk` | — | — |
| L3 | Inference SDK | Vercel AI SDK v6 | Claude Agent SDK direct | — |
| L3 | Schema | Zod v3 | — | — |
| L3 | LLM-output schema | Zod (v1) → BAML pilot | Instructor | golden set ships |
| L4 | Prose linter | Vale | textlint | — |
| L4 | LLM judge | Gemini 2.5 Pro / GPT-5 | — | — |
| L4 | Eval harness | promptfoo | Braintrust evals | — |
| L4 | Trace UI | Langfuse (self-hosted) | Braintrust | — |
| L5 | CLI framework | citty | commander | UnJS contracts |
| L5 | Config | c12 | cosmiconfig | UnJS contracts |
| L5 | Logger | consola + pino | — | — |
| L5 | Process | execa v9 | — | — |
| L5 | Locks | proper-lockfile | custom O_EXCL | maintainer disappears |
| L5 | GitHub | Octokit | gh CLI | — |
| L5 | Linear | Linear SDK | GraphQL direct | — |
| L6 | Event log | JSONL + pino | — | — |
| L6 | Lineage | custom under `~/.rapoport/lineage/` | — | — |
| L6 | LLM trace | Langfuse | Helicone | — |
| L6 | Training | DSPy + GEPA + BAML | — | golden set ships |
| L6 | OTel export | — (defer) | OTel JS SDK | semconv stable |
| L7 | Glossary | `_methodology/glossary.md` | — | — |
| L7 | Red-lines | `_methodology/red-lines.md` | — | — |
| L7 | Risk register | `_methodology/risk-register.md` | — | — |
| L7 | Audience policy | `_methodology/audience-policy.md` | — | — |
| L7 | Access scope | `_methodology/agent-access-scope.md` | — | — |
| C | Secrets | Infisical per-agent paths | — | — |
| C | Auth user | Supabase Auth | — | — |
| C | Distribution v1 | npm scope | — | — |
| C | Distribution v2 | Node SEA | — | external users matter |
| C | Skills export | custom `tools/skills-export.mjs` | — | — |

---

## Open questions

1. **One MCP server or three?** Each CLI could expose its own MCP server (Forge exposes `forge build`, Muse exposes `muse draft`, Maestro exposes `maestro lineage`). Alternative: one unified MCP server with namespaced tools. Tradeoff: discoverability (one) vs blast radius (three).

2. **Where does the golden set live — `_methodology/evals/golden/` or `tools/evals/golden/`?** First is discoverable but couples evals to bundle freshness. Second is cleanly separated but invisible to drive-by readers. Lean: `tools/evals/golden/` (clean separation), with pointer from `_methodology/`.

3. **Does Maestro have write access to `_methodology/`?** If Maestro detects drift (e.g., terminology collision spiking), should it open a PR to update the glossary, or only file a Linear issue? The first is faster; the second preserves human authority. Lean: Linear issue only — Maestro suggests, Pavel decides.

4. **When to mandate Vale?** As advisory (warn, don't fail) for first month, then enforcing (fail PR). Without measurement of false-positive rate first, mandatory Vale will create reviewer friction.

5. **Should `lineage/` be local-only or sync to a central store?** Local-only = simple, no infra. Central (e.g., Supabase Postgres) = queryable, multi-machine, but adds attack surface. Lean: local-only until second contributor appears.

6. **DSPy vs hand-tuning for the first 3 months.** DSPy is production-validated but adds complexity. Hand-tuning prompts with promptfoo regression suite is simpler and gives the same +5-10pp on small golden sets. Lean: hand-tune for 3 months, switch to DSPy once golden set has 100+ examples.

7. **A2A protocol — adopt now or wait?** Compatibility with Devin/Factory/Conductor through A2A would be strategic, but the protocol ecosystem is thin. Lean: build to MCP first (proven, distribution channel exists), add A2A as second-pass when there's a concrete inter-vendor scenario.

---

**End of stack reference.** This document is the parts list. The architectural design lives in [orchestra-training-architecture-2026.md](orchestra-training-architecture-2026.md); the empirical foundation in [training-orchestra-on-openspec-format-2026.md](training-orchestra-on-openspec-format-2026.md); the comparison deep-dives in [cli-tech-stack-comparison-2026.md](cli-tech-stack-comparison-2026.md) and [access-and-context-hierarchy-2026.md](access-and-context-hierarchy-2026.md).

Updates to this document follow L7 governance: propose a change, archive the old version.
