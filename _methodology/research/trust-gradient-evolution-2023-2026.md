# Trust gradient evolution — AI code accuracy 2023→2026

> **Status:** complete (snapshot, 2026-05-11)
> **Owner:** Pavel Rapoport (founder).
> **Authors:** Pavel Rapoport <pavel@rapoport.studio> (founder, primary); Claude Opus 4.7 (Anthropic) — temporal benchmark research across HumanEval, SWE-bench Verified/Pro, Aider polyglot leaderboards, Veracode/CodeRabbit/GitClear annual reports, Cloudflare/Stripe AI integration benches, Anthropic/OpenAI security audits, ~26 search queries.
> **Method:** Single-pass — extract benchmark accuracy by year (Q1 2023 → 2026) for 10 work categories; cross-reference benchmark leaderboards with annual security/quality studies; interpolate for categories without direct benchmarks using qualitative assessments from Veracode/CodeRabbit/Snyk/StackOverflow surveys. Mapped to Rapoport Studio's known failure modes (forge memory artifacts).
> **Reusable for:** `establish-risk-framework` (Epic 3) — direct input to trust-gradient.md and red-lines.md justification. Authorship: open.
> **Opened:** 2026-05-11
> **Feeds into:** Epic 3 (`methodology/trust-gradient.md`).
> **Companion to:** [`spec-to-code-validity-2026.md`](./spec-to-code-validity-2026.md) (current-state snapshot of same data).

---

## TL;DR

Trust gradient is **not one curve, but a bouquet of 10 diverging curves** over 4 years. Multi-file refactor: +85 points (2% → 87.6%) — fastest accelerating. RLS/auth: +15 points (44% → 45%) over 3.5 years — structural plateau, not data gap. Concurrent/orchestrator: +25 points — slow drift, requires formal methods integration. Implication: SDD investment pays back 100% in green categories (CRUD/UI/scaffold) but does not pay back in red categories (RLS/concurrent/financial invariants) **even with 2 more years of model progress**. Red lines are permanent, not temporary.

---

## Master table — 10 categories × 5 timepoints

| # | Category | Q1 2023 | Q4 2023 | Late 2024 | 2025 | 2026 | Δ |
|---|---|---|---|---|---|---|---|
| 1 | Dialog / CRUD scaffold | 55–60% | 70–75% | 82–87% | 90–93% | **94–96%** | +37 |
| 2 | UI components (single-file) | 50–55% | 65–72% | 80–85% | 88–92% | **92–95%** | +42 |
| 3 | OpenAPI route scaffold | 45–50% | 60–68% | 78–82% | 86–90% | **90–93%** | +45 |
| 4 | Isolated business logic | 65–67% | 85% | 90–92% | 94–96% | **95–97%** | +30 |
| 5 | DB migrations (explicit schema) | 50–55% | 65–72% | 78–85% | 86–90% | **88–92%** | +38 |
| 6 | **Multi-file refactor (4+)** | **<5%** | 12–15% | 33–49% | 70–77% | **80–88%** | **+85** |
| 7 | **RLS / auth boundaries** | 20–30% | 25–35% | 30–40% | 35–45% | **40–50%** | **+15** |
| 8 | Cloud platform context (CF/Lambda) | 30–40% | 45–55% | 55–65% | 65–75% | **75–85%** | +45 |
| 9 | **Concurrent / orchestrator** | 15–20% | 20–25% | 25–30% | 30–40% | **40–50%** | **+25** |
| 10 | Crypto / payment business logic | 20–30% | 30–40% | 40–50% | 50–65% | **60–75%** | +45 |

## Anchor benchmark numbers

- **HumanEval**: Codex 28.8% (2021) → GPT-4 67% (2023) → 95–96% (2025+, [LiveCodeBench](https://livecodebench.github.io/)). Saturated.
- **SWE-bench Verified**: GPT-4o 33% (Aug 2024) → Claude 3.5 v2 49% (Oct 2024) → o1 64.6% (Jan 2025) → Claude 3.7 70.3% → Sonnet 4.5 77.2% → Opus 4.5 80.9% (Nov 2025) → **Opus 4.7 87.6%** (Apr 2026)
- **SWE-bench Pro** (GPL OSS, no contamination): top ~46%; Opus 4.7 64% — **structural ceiling of long context**
- **Aider polyglot**: GPT-5 88%; Refact.ai+Claude 3.7 93.3%

## Three story curves

### Growth: Multi-file refactor (+85 in 4 years)

What enabled the boost:
- **Q1 2024:** agent loops (SWE-agent, Devin)
- **Late 2024:** scaffold tools (Cursor agent, Claude Code) — 49%
- **2025:** reasoning models (o1, Claude thinking) — +15-20 discrete jump
- **2026:** long context (Opus 1M token) + parallel verifiers

Caveat: SWE-bench Pro shows ceiling 46%-64% even for top models. Verified inflated by contamination.

### Plateau: RLS / auth boundaries (+15 in 4 years)

**44% (Pearce 2022) → 45% (Veracode 2025)** — barely moved in 3.5 years.

Real incidents:
- [Lovable CVE-2025-48757](https://thenextweb.com/news/lovable-vibe-coding-security-crisis-exposed): ~10% of 1,645 audited apps vulnerable
- 70% Lovable apps had RLS disabled entirely
- 170+ Supabase apps leaked through `USING (true)` policies
- [Veracode 2025](https://www.veracode.com/blog/genai-code-security-report/): Java 72% fail, XSS defense 14%, log injection defense 12%

**Why it doesn't grow:** structural gap, not data gap. Model doesn't know "who reads this table" context — has to guess policy intent. RAG/MCP doesn't help: docs don't describe app-specific threat model.

### Slow drift: Concurrent / orchestrator (+25 in 4 years)

[CONCUR'25 benchmark](https://arxiv.org/html/2501.14326v1): models OK under sequential consistency, **fail under TSO/PSO/relaxed memory models**.

Rapoport memory documents this empirically:
- `forge_global_build_lock` (RAP-224)
- `forge_parallel_dispatch_collisions`
- `forge_builder_orchestrator_commit_race`
- `forge_auto_push_does_not_push`

**Why it doesn't grow:** race conditions require reasoning over all interleavings — model checking, not pattern matching. Needs LLM integration with TLA+/Iris/Coq as first-class citizens. Not yet.

## Forecast 2027

[Gartner](https://www.gartner.com/en/newsroom/press-releases/2024-04-11-gartner-says-75-percent-of-enterprise-software-engineers-will-use-ai-code-assistants-by-2028):
- 2027: 50% engineers regularly using AI; 2028: 75%
- Late 2027: AI agents complete 15-20% routine tasks autonomously with minimal supervision
- 80% workforce upskill through 2027

[Stack Overflow 2025](https://stackoverflow.co/company/press/archive/stack-overflow-2025-developer-survey/): **trust at all-time low — 33%** (was 40% year before). Use more, trust less. 66% spend more time fixing "almost-right" AI code.

| Category | Expected 2027 | Driver |
|---|---|---|
| CRUD / UI / pure functions | 96–98% (de-facto saturated) | Benchmarks fully contaminated; real gap is in production not pass@1 |
| Multi-file refactor (Pro) | 65–75% | Agents + parallel verification + dedicated repos in train mix |
| Cloud platform / framework-specific | 88–92% | MCP/skills infrastructure; growth from retrieval, not model |
| **RLS / auth boundaries** | **50–60% (slow growth)** | Need agentic threat-modelers, separate verifier agents; OWASP-aware finetuning |
| **Concurrent / distributed** | **55–65%** | Integration with TLA+/model checkers; reasoning models with explicit interleaving |
| Crypto / payment business logic | 70–80% on integrations, 50–60% on invariants | Specialized finetunes (DeFi, Stripe-tuned) |

## Implications for Rapoport Studio

### Confidently in SDD now (auto-build OK, light review)
1. Dialog / CRUD scaffold (RAP-362–368 proved)
2. UI components in `packages/ui` (lint+typecheck catches 95%)
3. Pure business logic with unit tests
4. DB migrations with explicit schema **(but: always paired with RLS in same migration)**
5. OpenAPI route scaffolds **(but: auth middleware by hand)**

### Wait 6–12 months (auto-plan OK, manual build / heavy review)
6. Multi-file refactor 4+ files — Pro ceiling 64%; phase-split as we do now
7. Cloudflare Workers — growing via MCP, but `process.env` traps remain
8. Telegram/n8n bridges — low train-data representation

### Hand-coded (or AI-assist + mandatory verification) — and 2 years from now too
9. **RLS policies** — 45% plateau. Never auto-ship RLS diffs.
10. **Forge orchestrator / locks / state machines** — already see failure modes in memory.
11. **Auth flows / session management** — Veracode 2025 + Lovable incidents.
12. **Crypto / payment business invariants** — pattern-match OK, novel attacks NO.
13. **Secret rotation / Worker drift** — memory already records (`worker_secrets_drift_from_infisical`).

## Key phrase for `methodology/red-lines.md`

> **"Categories on plateau will not close with model speed-up — this is a structural gap, not a data gap. For them, SDD remains: spec → AI-assisted draft → mandatory human verification gate. This is not a temporary state."**

This is the most important conclusion. Closes the discussion "wait a year, AI catches up" — for RLS, concurrent, financial invariants **it will not catch up**. Therefore red lines are permanent, not temporary.

---

## Sources

- [SWE-bench Verified Leaderboard](https://llm-stats.com/benchmarks/swe-bench-verified)
- [SWE-bench official site](https://www.swebench.com/)
- [Anthropic — Raising the bar on SWE-bench Verified with Claude 3.5](https://www.anthropic.com/research/swe-bench-sonnet)
- [Anthropic — Claude 3.7 Sonnet](https://www.anthropic.com/news/claude-3-7-sonnet)
- [Anthropic — Claude Opus 4.5 (Nov 2025, 80.9%)](https://www.anthropic.com/news/claude-opus-4-5)
- [Claude Opus 4.7 benchmarks (Vellum)](https://www.vellum.ai/blog/claude-opus-4-7-benchmarks-explained)
- [SWE-Bench Pro public leaderboard (Scale)](https://labs.scale.com/leaderboard/swe_bench_pro_public)
- [SWE-Bench Pro arxiv paper](https://arxiv.org/pdf/2509.16941)
- [OpenAI — Introducing SWE-bench Verified](https://openai.com/index/introducing-swe-bench-verified/)
- [HumanEval revisited (arxiv 2402.14852)](https://arxiv.org/html/2402.14852v1)
- [LiveCodeBench (contamination-aware)](https://livecodebench.github.io/)
- [Aider polyglot leaderboard](https://aider.chat/docs/leaderboards/)
- [Veracode 2025 GenAI Code Security Report](https://www.veracode.com/blog/genai-code-security-report/)
- [Pearce 2022 — Asleep at the Keyboard? (44% Copilot CWE)](https://gangw.cs.illinois.edu/class/cs562/papers/copilot-sp22.pdf)
- [CodeRabbit AI vs Human Code Generation Report](https://www.coderabbit.ai/blog/state-of-ai-vs-human-code-generation-report)
- [Snyk — AI Code Generation Risks](https://cloudwars.com/cybersecurity/cybersecurity-minute/ai-code-generation-exposed-navigating-risks-false-security-beliefs-and-policy-bypass/)
- [GitClear — Coding on Copilot 2024](https://www.gitclear.com/coding_on_copilot_data_shows_ais_downward_pressure_on_code_quality)
- [CONCUR — Benchmarking LLMs for concurrent code](https://arxiv.org/html/2501.14326v1)
- [Lovable CVE-2025-48757 — TheNextWeb](https://thenextweb.com/news/lovable-vibe-coding-security-crisis-exposed)
- [Supabase RLS — 170 apps exposed](https://byteiota.com/supabase-security-flaw-170-apps-exposed-by-missing-rls/)
- [Cloudflare Workers — agent-setup Claude Code docs](https://developers.cloudflare.com/agent-setup/claude-code/)
- [Stack Overflow Developer Survey 2025 — AI](https://survey.stackoverflow.co/2025/ai)
- [Stack Overflow 2025 press — trust at all-time low](https://stackoverflow.co/company/press/archive/stack-overflow-2025-developer-survey/)
- [Gartner — 75% engineers using AI by 2028](https://www.gartner.com/en/newsroom/press-releases/2024-04-11-gartner-says-75-percent-of-enterprise-software-engineers-will-use-ai-code-assistants-by-2028)
