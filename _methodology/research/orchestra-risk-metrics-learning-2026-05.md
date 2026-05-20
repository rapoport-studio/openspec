# Research Pass 2 — Risk audit, metrics, learning loops, MVP packaging

> **Status:** complete (single-pass research, 2026-05-20)
> **Owner:** Pavel Rapoport <hello@rapoport.studio> (founder).
> **Authors:** Pavel Rapoport <hello@rapoport.studio> (founder, primary); Claude Opus 4.7 (Anthropic) — secondary synthesis across MAST/OWASP/EU-AI-Act/observability literature, Top-12 risk register drafting, v1.0 MVP phase plan formalization.
> **Method:** Single-pass secondary synthesis across four streams (risk taxonomy, metrics/observability, learning loops, MVP packaging). Empirical anchors include MAST NeurIPS 2025 (1,642 traces, 7 SOTA MAS frameworks), OWASP LLM/ASI Top 10 2025/2026, EU AI Act primary text + Digital Omnibus VII (May 2026), LangChain State of Agent Engineering survey (n=1,340), 15-platform observability benchmark. No first-party measurement; no interviews. Vendor-marketing numbers excluded from main text. Source list at end of file.
> **Reusable for:** Drafting `openspec/changes/add-orchestra-mvp/` package (proposal / design / tasks / verification); Maestro v1.0 scope ratification; orchestra-risk-register.md initialization; Langfuse self-host setup change folder; Phase A/A'/B/C/D/E/F planning. Authorship: open. <!-- openspec-refs: skip-line -->
> **Opened:** 2026-05-20
> **Companion to:** [`./orchestra-competitive-landscape-2026-05.md`](./orchestra-competitive-landscape-2026-05.md) (Pass 1 — competitor and naming landscape), [`./orchestra-component-stack-2026.md`](./orchestra-component-stack-2026.md), [`./orchestra-training-architecture-2026.md`](./orchestra-training-architecture-2026.md), [`./forge-epic-conductor-runbook.md`](./forge-epic-conductor-runbook.md)
> **Feeds into:** v1.0 MVP package decisions Δ1–Δ6 (§ TL;DR); open questions Q-OBS-1, Q-OBS-2, Q-LEARN-1 (§ 4.5); future amendments to `openspec/current/agents/maestro.md` and a new `openspec/_methodology/orchestra-risk-register.md`.

---

## TL;DR — пять findings + дельта в MVP

1. **MAST taxonomy (NeurIPS 2025, 1,642 traces)** говорит: 41.77% multi-agent failures — **specification problems** (роли, задачи, ограничения недоопределены), 36.94% — **coordination breakdowns**, 21.30% — **verification gaps**. Failure rate 7 SOTA open-source MAS — 41% до 86.7%. Это значит: **самый большой риск Rapoport — не security, а недоспецифицированность ролей и handoff'ов.** Защита уже частично встроена в OpenSpec, но требует журнала + еженедельного review.
2. **OWASP ASI Top 10 2026** — agentic security риски. Для studio с одним principal (Pavel) и no client agents большая часть критических atak inapplicable. Релевантные: **agent goal hijack через indirect prompt injection** (Linear issue body, repo files), **tool misuse via excessive permissions**, **cost runaway loops**. Mitigations простые — allowlists, circuit breakers, signed intent на handoff.
3. **EU AI Act enforcement Aug 2, 2026** (с возможной отсрочкой до Dec 2027 через Digital Omnibus, agreement 7 May 2026). **Rapoport MVP — НЕ high-risk system** (internal dev tooling, не Annex III). НО три principles из Act применяются как good engineering anyway: **log retention ≥6 months, human oversight by design, structured incident response**. Maestro `orchestra_journal` уже это решает; нужна explicit 6-month retention policy.
4. **Observability landscape сошёлся.** Для Rapoport — **Langfuse self-host (MIT, Postgres + ClickHouse)** — winner: open-source, OTel-native, free, self-hostable in studio's existing infra. LangSmith deeper для LangChain (Pavel не на LangChain). Phoenix — для ML-heavy. OpenTelemetry GenAI conventions всё ещё в **Experimental status** в начале 2026 — instrumentировать сейчас «forward-compatibly», но не ожидать stable schema.
5. **Learning loops — три уровня.** Heavyweight (RLHF/RLAIF fine-tuning) — **out of scope для studio MVP**, требует preference labelling и training infra. Mediumweight (memory systems Mem0/Letta/Zep) — **тоже out of scope** на v1.0, но Mem0-pattern (extract→store→retrieve memories из interactions) станет relevant для Echo/Atlas в Phase B. Lightweight — **lessons-learned writeback файлы**, уже работает в Forge (`packages/forge/.../lessons/`), нужно extend на Maestro в v1.0.

**Delta v1.0 MVP package** (что добавляется к плану из предыдущей итерации):

| # | Что добавляется | Phase |
|---|---|---|
| Δ1 | **Langfuse self-host** в studio infra + OTel GenAI instrumentation на Muse/Forge/Maestro LLM calls | A' (между A и B) |
| Δ2 | **Risk register markdown** в `openspec/_methodology/orchestra-risk-register.md` с 12 risks → defenses map | A |
| Δ3 | **Lessons-learned writeback** для Maestro — `~/.maestro/lessons/<RAP-N>.md` per-issue post-mortem, аналогично Forge | E |
| Δ4 | **6-month retention policy** на `orchestra_journal` (no auto-delete до v1.x) | A |
| Δ5 | **Cost circuit breaker** в Maestro session — per-session cap = $X, hard stop с surface к Pavel | B |
| Δ6 | **Indirect prompt injection guard** на Linear issue body parser — strip suspicious instruction patterns before feeding Maestro | D |

Шесть concrete additions. Каждое — несколько часов работы, не недели. Phases A/A'/B/C/D/E остаются, добавляется F (lessons writeback) опционально.

---

## Stream 1 — Risk taxonomy

### 1.1 MAST — Multi-Agent System Failure Taxonomy (NeurIPS 2025)

Самый эмпирически обоснованный source. Cemri et al. проанализировали 1,642 execution traces из 7 SOTA open-source MAS frameworks. Три category-level findings:

| Category | Prevalence | Что входит |
|---|---|---|
| **Specification problems** | 41.77% | Role ambiguity, unclear task definitions, missing constraints, prompt brittleness, system design issues |
| **Coordination failures** | 36.94% | Communication breakdowns, state synchronization issues, conflicting objectives, inter-agent misalignment |
| **Verification gaps** | 21.30% | Inadequate testing, missing validation mechanisms, absent output quality checks, no task verification |

Плюс **четвёртая практическая категория вне MAST**: **infrastructure issues** — rate limits, context overflows, cascading timeouts.

Failure rate на 7 SOTA MAS — **41% до 86.7%**. То есть из любых десяти многоагентных workflow от 4 до 9 ломаются в той или иной форме. Это базовый rate без специальных mitigations.

**Применимость к Rapoport:**

- **Specification (41.77%)** — это **главный риск** Rapoport. OpenSpec методология уже частично его адресует (явные роли в `agents-overview.md`, явные boundaries в per-agent specs). Но MVP добавляет Maestro routing и handoff contracts — если они недоспецифицированы, упадёт половина invokes. **Mitigation: каждый event_type в `orchestra_journal` должен иметь explicit JSON schema; каждый handoff — signed intent.**
- **Coordination (36.94%)** — Maestro + journal + originating_agent_id уже спроектированы против этого. Но **MVP single-agent-per-issue lock** ограничивает; когда захотим parallel agents (per Feb 2026 trend), coordination риск вырастет резко.
- **Verification (21.30%)** — Forge уже имеет `verificationGates[]`, Maestro v1.0 — нет проверки своих routing decisions. **Mitigation: LLM-as-judge eval scoring через Langfuse поверх journal entries (см. Stream 2).**

### 1.2 OWASP ASI Top 10 (2026) — Agentic Skills Top 10

OWASP в 2025–2026 расширил LLM Top 10 (LLM01–LLM10) до отдельного **agentic-specific** списка (ASI01–ASI10). Из десяти позиций для Rapoport MVP релевантны четыре:

| ASI | Что это | Релевантность Rapoport |
|---|---|---|
| **ASI01 Agent Goal Hijack** | Indirect prompt injection через documents, RAG content, tool outputs — атакующий меняет цель агента. OWASP LLM01:2025 — #1 vulnerability vector. | **Высокая.** Linear issue body может содержать injected instructions. Maestro feeds issue body в routing prompt → potential hijack. **Mitigation:** strip-and-quote pattern: issue body не идёт raw в prompt, оборачивается в `<untrusted_user_content>` теги, инструкции игнорируются. |
| **ASI02 Tool Misuse** | Agent имеет excessive tool permissions, может legitimate tool использовать unsafely. | **Средняя.** Forge имеет write access к repo и GitHub. Maestro в spec'е — read-only через access scope allowlist. **Mitigation: scope allowlist уже в spec'е, нужен runtime enforcement через `getToolsForAnthropicAPI(role)`.** |
| **ASI07 Inter-Agent Communication Tampering** | Сообщения между агентами перехватываются/модифицируются. | **Низкая.** MVP — single principal, single-host деплой. Все handoffs внутри одного процесса или Supabase row. **Mitigation: content_hash на journal entries, который уже в спеке.** |
| **ASI08 Cost Runaway / Denial of Wallet** | Loops, retry chains, runaway agents тратят неограниченный budget. | **Высокая.** Maestro может уйти в петлю — route to Muse → no response → re-route to Forge → ... **Mitigation: per-session cap (Δ5), circuit breaker.** |

ASI03–06, ASI09–10 — privilege escalation, memory poisoning attacks, etc. — applicable, но не critical на MVP с одним principal'ом и no public surface.

**Industry baseline:** 88% organizations deploying AI agents в 2025 reported at least one security incident. IBM 2025 breach report: организации с comprehensive AI security controls economy $1.9M per breach.

### 1.3 EU AI Act compliance posture для Rapoport

Главный вопрос: **является ли Rapoport orchestra «high-risk AI system» по AI Act?**

**Ответ: нет, на v1.0 — не high-risk.** Annex III high-risk use cases — это credit scoring, employment decisions, biometric identification, education access. Rapoport MVP — **internal developer tooling для одной студии**, без impact на fundamental rights третьих лиц.

Но есть три principles из Act, которые применяются как **good engineering practice независимо от классификации**:

1. **Article 26.6 — Log retention.** High-risk systems должны хранить logs **минимум 6 месяцев**. Для Rapoport: `orchestra_journal` no-auto-delete до v1.x — **6-month retention policy explicit в spec'е** (Δ4).
2. **Article 14 — Human oversight by design.** Системы должны позволять human overseers «understand capabilities and limitations, detect anomalies and unexpected performance». Maestro decision-on-ambiguity surface — уже это делает. **Дополнительно: weekly review ритуал** (Pavel grep'ает journal + lessons файлы).
3. **Article 17 — Quality management system.** Risk management process, incident response с 72-hour reporting. Для Rapoport: **risk register markdown** (Δ2) — это minimum viable QMS-аналог. Если в будущем Rapoport начнёт shipping для клиентов в EU, потребуется ISO 42001 certification — но это v2+.

**Прим. о timeline:** Digital Omnibus VII (political agreement 7 May 2026) может **отложить high-risk obligations с Aug 2026 до Dec 2027**, но формальное adoption ещё впереди. Не существенно для Rapoport MVP. SME simplifications extended до 750 сотрудников и €150M revenue (Rapoport phenomenally below this threshold).

### 1.4 Cooperative AI risks (Hammond et al., 2025)

Cooperative AI Foundation определила три primary failure modes:

- **Miscoordination** (failure to cooperate despite shared goals) — в Rapoport применимо, mitigation = journal + Maestro
- **Conflict** (differing goals) — в Rapoport **inapplicable**: все агенты — один principal (Pavel)
- **Collusion** (undesirable cooperation) — **inapplicable** по тем же причинам

Из семи key risk factors (information asymmetries, network effects, selection pressures, destabilising dynamics, commitment problems, emergent agency, multi-agent security) для Rapoport MVP релевантны два:

- **Commitment problems** — Maestro routing decision должен быть irrevocable после handoff (mitigation: append-only journal)
- **Emergent agency** — Maestro reroute'инг может создавать loops (mitigation: cost circuit breaker Δ5)

### 1.5 Consolidated risk register для Rapoport MVP — Top 12

Top 12 рисков с mitigation, отранжированные по probability × impact:

| # | Risk | Source | Probability | Impact | MVP mitigation |
|---|---|---|---|---|---|
| R1 | **Routing decision undefined** (Maestro не знает куда отправить) | MAST Spec | High | Med | Decision-on-ambiguity surface to Pavel; default safe = ambiguity |
| R2 | **Handoff schema drift** (Muse expects X, Forge gives Y) | MAST Spec | Med | High | JSON schema validation на каждом event_type, contract test в `__tests__` |
| R3 | **Cost runaway loop** (Maestro reroutes infinitely) | OWASP ASI08 | Med | High | Per-session cap $X (Δ5), per-issue cap, hard stop с journal entry |
| R4 | **Indirect prompt injection** через Linear issue body | OWASP ASI01 | Med | Med | Strip-and-quote (Δ6); issue body wrapped в `<untrusted_user_content>` |
| R5 | **Journal write race / inconsistent seq** | MAST Coord | Low | High | `seq monotonic per work_item_key`, content_hash на read |
| R6 | **Agent_status stale** (Muse crashed, не обновил) | MAST Verif | Med | Med | Stale-detect heuristic — если no journal entry от агента > N min, surface drift |
| R7 | **Forge worktree lock contention** (RAP-405 known issue) | Infra | Low | Low | `forge cancel <other-key>` mechanism уже есть, document как часть Maestro playbook |
| R8 | **Decision laundering** (Pavel думает что Maestro решила, на самом деле Muse в prompt'e подсказала) | Maestro spec | Med | Med | `originating_agent_id` + content_hash в каждом journal entry (уже в spec'е) |
| R9 | **Cost guard miss** (Anthropic dashboard latency, Pavel узнаёт пост-фактум) | Infra | High | Low | Langfuse cost tracking real-time + alert threshold (Δ1) |
| R10 | **Tool misuse** (Forge получает не тот repo, пишет в wrong worktree) | OWASP ASI02 | Low | High | `--project=<slug>` mandatory на multi-project; sandbox=docker default |
| R11 | **Knowledge fragmentation** (один lesson написан в Forge, другой in Maestro, никто не видит вместе) | Learning | Med | Low | `_methodology/lessons/` aggregator script (post-MVP) |
| R12 | **EU AI Act classification drift** (Rapoport случайно становится «high-risk» через client engagement) | Compliance | Low | High | Risk register quarterly review, classification re-check на каждый new client mode |

R1, R2, R3, R4 — **must-fix перед v1.0 GA**. R5–R8 — addressed by existing spec'ом design. R9–R12 — monitor, не блокеры.

---

## Stream 2 — Metrics & observability

### 2.1 Landscape (как состояние Q2 2026)

Observability tools для AI agents сошлись в шесть категорий:

| Tool | License | Best for | Pricing |
|---|---|---|---|
| **Langfuse** | MIT, self-host | OS leader, framework-agnostic, OTel-native, prompt management + eval harness. 50M+ SDK installs/мес. | Free self-host, cloud $59+/seat |
| **LangSmith** | Closed source (self-host Enterprise-only) | Deepest LangChain/LangGraph integration. LangGraph Studio. | Free tier, usage-priced |
| **Arize Phoenix** | Elastic 2.0 | ML-grade rigor, OpenInference (10 span kinds vs OTel's 2), eval-heavy teams | Free OSS, cloud paid |
| **Laminar** | Apache 2.0, self-host | Built specifically для long-running agents, transcript view, Signals, browser-agent replay | Free OSS |
| **Datadog LLM Observability** | Commercial | Enterprise-default для Datadog shops | Per-host |
| **OpenTelemetry GenAI** | Apache 2.0 (vendor-neutral standard) | Forward-compatible instrumentation | Free, status Experimental в начале 2026 |

**Industry adoption (LangChain State of Agent Engineering survey Nov–Dec 2025, n=1,340):** 62% production agent teams имеют детальное tracing для individual agent steps и tool calls. LangSmith appears в plurality of production deployments. 89% organizations running AI agents in production имеют some form of observability.

**Performance benchmark (15 platforms, 100 identical queries, multi-agent travel planning):**

- LangSmith: ~no measurable overhead
- Langfuse: 15% overhead (highest detail capture)
- Phoenix / Laminar: ~5%
- Tradeoff: detail vs latency

### 2.2 Рекомендация для Rapoport — Langfuse self-host

**Почему Langfuse:**

1. **MIT, self-host без restrictions.** Pavel может развернуть в studio infra без vendor risk.
2. **OpenTelemetry-native.** Если OTel GenAI conventions стабилизируются в 2026–2027, Langfuse мигрирует без re-instrumentation.
3. **Framework-agnostic.** Pavel не на LangChain → LangSmith не даёт depth advantage. Langfuse поддерживает любые SDK (Anthropic native, custom).
4. **Postgres + ClickHouse backend.** Pavel уже имеет Supabase Postgres; ClickHouse — добавляется как single container.
5. **Free.** $0 platform cost (только compute+storage).
6. **Eval harness built-in.** LLM-as-judge scoring через Langfuse без интеграции дополнительных tools.

**Trade-off:** 15% overhead в multi-step workflows. Для MVP — приемлемо, Maestro invocations не latency-critical.

**Alternative consideration — Laminar:** Apache 2.0, agent-first transcript view, Helm chart self-host. Лучше для **debugging long-running agents** (Forge build sessions могут идти часами). Если Pavel хочет «открыл UI, увидел полный transcript Forge build session с decisions и tool calls», Laminar лучше. **Можно рассмотреть как замену Langfuse, если debugging Forge sessions станет primary pain point.**

### 2.3 OpenTelemetry GenAI semantic conventions

**Текущий статус (начало 2026):** все категории GenAI conventions в **Experimental status**. Span kinds для агентов покрывают только `create_agent` и `invoke_agent` — недостаточно для multi-step agentic traces.

**OpenInference (Arize)** определяет 10 span kinds — CHAIN, LLM, TOOL, RETRIEVER, EMBEDDING, AGENT, RERANKER, GUARDRAIL, EVALUATOR, PROMPT — это более полная taxonomy. Phoenix использует OpenInference.

**Что делать Rapoport:** instrumentировать через **OpenLLMetry** (Traceloop, pending donation to OTel), который абстрагирует над provider-specific SDKs (Anthropic, OpenAI, etc.). Span attributes — name, model, prompt, response, tokens, cost, latency. Через год при стабилизации OTel GenAI — миграция automatic via dual-emission (`OTEL_SEMCONV_STABILITY_OPT_IN`).

### 2.4 Что измерять — три слоя metrics

**Layer 1 — System (per-LLM-call):**
- Latency p50/p95/p99 per agent role (architect, builder, planner, maestro)
- Cost per call ($)
- Tokens in/out
- Model used (per agent-model-mapping)
- Error rate (Anthropic API errors, rate limits)

**Layer 2 — Decision (per Maestro routing):**
- Routing accuracy: для скольких routes Pavel override'нул через decision-on-ambiguity (lower = Maestro лучше угадывает)
- Drift signal frequency: сколько `time_in_stage > 95th percentile` events per week
- Journal coherence: % entries with all required fields populated
- Handoff success rate: handoff issued → agent acknowledged % within N min

**Layer 3 — Outcome (per work item):**
- **Cycle time** stage → stage и end-to-end (issue → PR)
- **PR merge rate** (out of all issues that reached Stage 7)
- **Spec → PR success rate** (out of all promoted specs, how many produced merged PR)
- **Issue abandonment rate** (issues stuck в `frame` / `spec` stage > N days)

### 2.5 Critical metric для MVP success criterion

**End-to-end success rate на «happy path»:**

> `(# issues that went от inbox → spec → plan → build → PR без manual intervention) / (# total issues routed by Maestro)`

Target после первого месяца — **≥ 50%**. MAST baseline для 7 SOTA MAS — failure rate 41% до 86.7%. То есть «50% success» уже соответствует mid-tier industry baseline; всё что выше — improvement.

### 2.6 Goodhart's Law mitigation

Из `workflow.md`: «вы оптимизируете cycle time → агенты начинают делать smaller-than-meaningful PRs».

Mitigation встроен в Rapoport через:

- **Cycle time** агрегируется ПО issue, не по PR (нельзя cheat'ить через split)
- **PR merge rate** не tracks closed-without-merge (нельзя бросать)
- **Forge Productivity Index** (`add-forge-productivity-measurement` change) — multi-signal score, не single metric

### 2.7 Langfuse setup на v1.0

| Step | Что делается |
|---|---|
| 1 | Развернуть Langfuse self-host в studio infra (docker-compose: langfuse-server + postgres + clickhouse + redis) |
| 2 | Создать project `rapoport-orchestra-mvp`, API keys |
| 3 | Установить `langfuse` SDK в `packages/muse/` и `packages/forge/` |
| 4 | Wrap Anthropic API calls через Langfuse decorator (один-два import-statement в core) |
| 5 | Trace propagation: Maestro session id → child Muse/Forge calls. `span_context` field в каждом journal entry для correlation |
| 6 | Alert thresholds: cost-per-session > $X, latency p99 > Y |
| 7 | LLM-as-judge eval: каждые 100 traces, sample 5 → score через Opus 4.7 «was this routing correct?» |

---

## Stream 3 — Learning loops

### 3.1 Memory systems landscape

Memory products в 2026 разделились на категории по storage pattern:

| Product | License | Pattern | Best for |
|---|---|---|---|
| **Mem0** | Open + cloud | Extract→store→retrieve memories. Three-tier (user/session/agent). Hybrid store: vector + graph + KV. ~48K stars, $24M funding (PR Newswire/Morningstar, Oct 2025). | Personalized assistants, customer-support, B2B copilots. ECAI 2025 paper. |
| **Letta (formerly MemGPT)** | Open | OS-inspired tiered memory. Core (RAM-like, in-context), archival (vector store), recall (full history). | Long-running agents that need fine-grained control over context |
| **Zep** | Open + cloud | Temporal Knowledge Graph. Tracks evolving facts and entity relationships. | Agents tracking entities and relationships over time |
| **LangMem** | Open (LangGraph SDK) | Three types: episodic / semantic / procedural (agents rewrite own instructions). | LangGraph-based stacks |
| **Cognee** | Open | Local-first, graph reasoning, privacy-critical | Air-gap, regulated environments |
| **Supermemory** | Open | Coding agent integrations (Claude Code, OpenCode) | Coding-agent memory layers |

### 3.2 Recommendation для Rapoport MVP — НЕ внедрять memory system

**Reasons:**

1. **MVP scope не требует.** Маestro routes — это **state machine**, не memory-driven decision. Журнал даёт всю нужную state.
2. **Heavyweight infra.** Mem0/Letta/Zep требуют отдельных stores (vector DB + optionally graph DB), extract-memory pipelines, retrieval tuning.
3. **Echo / Atlas — кандидаты на v1.2+, не v1.0.** Echo нужна memory для brand voice consistency (Jasper IQ analog), Atlas — для continuous beat coverage. На MVP они out of scope.

**Когда memory system войдёт:** v1.2 при запуске Echo, когда понадобится per-client brand voice memory. Шансы — Mem0 (наиболее зрелый, ECAI paper baseline, $24M backing).

### 3.3 RLHF / RLAIF для AI agents — feasibility study

**State of the art:**

- RLHF (Christiano 2017, OpenAI InstructGPT 2022) — small amounts of preference data могут give comparable results to large dataset. Wikipedia: «one initial motivation of RLHF was that it requires relatively small amounts of comparison data to be effective».
- RLAIF (Constitutional AI, Anthropic 2022) — AI feedback может match or exceed human feedback in training harmless systems.
- PAHF (Meta, Feb 2026) — Personalized AI from Human Feedback: explicit memory + dual feedback channels (pre-action clarification + post-action feedback). Beats no-memory и single-channel baselines.
- Training cost: PPO с trlx (HuggingFace) или Anthropic rlhf fork — но требует separate reward model training step.

**Recommendation для Rapoport: НЕ делать RLHF на v1.0.**

**Reasons:**

1. **Anthropic API доступ к Claude — без fine-tuning.** Pavel может only делать prompt engineering и tool design, не model weights tuning.
2. **Training infra нужна:** GPU, reward model, PPO loop — несоразмерно с studio scale.
3. **Альтернатива дешевле и эффективнее:** prompt iteration через **Langfuse prompt management** + **LLM-as-judge eval** + **lessons-learned writeback**. Это даёт continuous improvement без training.

**Pattern для Rapoport — «prompt-as-policy» improvement loop:**

```
1. Journal entries → Langfuse traces
2. LLM-as-judge scores routes (correct? optimal?)
3. Low-scoring routes → human review (Pavel)
4. Pattern emerges → update Maestro system prompt or routing function
5. Versioned prompt → A/B compare через Langfuse
```

Это **дешевле** RLHF (no training), **быстрее** (next-day iteration), и **полностью под контролем Pavel** (no opaque weights).

### 3.4 Lessons-learned pattern — concrete mechanism for v1.0

**Forge уже имеет lessons-learned writeback** через `packages/forge/src/commands/build/lessons.ts` (упоминается в `forge.md § Builder mode`). Pattern: после каждого build, builder agent эмитит structured `lessons.md` файл, который читается следующим build session как context.

**Extend на Maestro** — Δ3 в delta list:

| File | Content | Read by | Write by |
|---|---|---|---|
| `~/.maestro/lessons/<RAP-N>.md` | Per-issue post-mortem: «routed to X, agent did Y, outcome Z, what would've been better». 200-500 words. | Next Maestro invocation для **same issue** (resume) или похожих по pattern | Maestro после `completed` или `cancel` event |
| `~/.maestro/patterns.md` | Cross-issue aggregator: «across N issues, pattern P emerges». Manually curated by Pavel еженедельно. | Maestro system prompt как stable context | Pavel manual |
| `openspec/_methodology/orchestra-lessons.md` | Архив куратированных patterns, promoted to spec level | Все агенты | Pavel (через PR review) |

**Three-tier flow:** ephemeral (per-issue) → curated (cross-issue, weekly) → spec (orchestral methodology).

Это **identical pattern с Forge**, scaled to Maestro level. Не новая infra, не новая методология — extension того, что уже работает.

### 3.5 ALMA — meta-learning of memory designs (research frontier, не для MVP)

Для context: ALMA (arXiv:2602.07755, Feb 2026) — Meta Agent searches over memory designs expressed as executable code. Discovered designs outperform hand-crafted memory architectures on all benchmarks tested.

**Implication для Rapoport:** в long term (v2+) memory architecture может быть **agent-designed**, не human-designed. Сейчас это frontier research, не production-ready. Mention as direction; не для MVP.

---

## Stream 4 — Final v1.0 MVP package

### 4.1 Что входит в v1.0 — financial summary

| Категория | Что строится | Phase | Effort estimate |
|---|---|---|---|
| **Core** | Maestro mode runtime (routing + journal write + decision-on-ambiguity) | B | M (2-4 weeks) |
| **Substrate** | `orchestra_journal` + `agent_status` Supabase tables, FS mirror writer, retention policy | A | S (1 week) |
| **Observability** (Δ1) | Langfuse self-host + OTel-GenAI instrumentation on Muse/Forge/Maestro LLM calls | A' | S (1 week) |
| **Risk register** (Δ2) | `openspec/_methodology/orchestra-risk-register.md` — 12 risks → defenses map | A | XS (1-2 days) |
| **Agent integration** | Muse emits `spec_promoted` events; Forge emits `BuildOutcome` events to journal | C | S (1 week) |
| **Triggers** | `forge maestro <subcommand>` CLI; Linear webhook endpoint в `apps/studio` | D | S-M (1-2 weeks) |
| **Cost circuit breaker** (Δ5) | Per-session cap в Maestro session loop | B | XS (1 day, в составе Phase B) |
| **Prompt injection guard** (Δ6) | Strip-and-quote pattern on Linear issue body parser | D | XS (1 day, в составе Phase D) |
| **End-to-end** | Test scenario RAP-MVP-TEST через полный pipeline | E | M (1 week, validation) |
| **Lessons writeback** (Δ3) | Maestro post-mortem `~/.maestro/lessons/<RAP-N>.md` | F (optional) | S (1 week) |

**Total estimate:** ~8–10 weeks of focused work, без Phase F. С Phase F — 9–11 недель.

### 4.2 Что НЕ входит в v1.0 — explicit out-of-scope

| Категория | Reason | Будет в |
|---|---|---|
| **Phase B agents** (Atlas, Echo, Pulse) | Spec'и не написаны; runtime infra потребует Echo brand voice IQ, Atlas beat configurations, Pulse compliance | v1.1 (Atlas) / v1.2 (Echo) / v1.3 (Pulse) per ROADMAP |
| **Memory system** (Mem0 / Letta) | Не нужно для state-machine routing; станет relevant при Echo/Atlas | v1.2+ при необходимости |
| **RLHF / fine-tuning** | Anthropic API не позволяет fine-tuning Claude. Альтернатива — prompt iteration через Langfuse — дешевле и эффективнее. | Out of scope long-term без model access |
| **Parallel agents on one issue** (Feb 2026 trend) | Single-issue lock на MVP. Multi-agent на одной задаче — separate epic. | v1.1+ |
| **Control-room UI** (`/control/*` routes per Maestro spec) | Terminal first, UI deferred per Q-SURFACE | v1.x |
| **EU AI Act high-risk compliance** (Article 17 QMS, ISO 42001) | Rapoport MVP — не high-risk. Risk register (Δ2) — minimum viable QMS-аналог. | При first EU client engagement |
| **Cron daemon trigger** | Manual + webhook покрывают v1.0 | v1.1 если manual+webhook не хватит |
| **Drift detection / audit pass в Maestro** | Только routing + decision on ambiguity per Maestro v1.0 scope | v1.4 (Maestro Watch / Risk Lens / Compass) |
| **Visual diff UI для decision surface** | Terminal output ASCII format достаточно | v1.x |

### 4.3 Updated Phase plan — 6 phases

```
Phase A — Substrate (1 week)
  ├─ orchestra_journal Supabase table + RLS
  ├─ agent_status Supabase table
  ├─ FS mirror writer ~/.maestro/journal/
  ├─ TypeScript journal client в packages/muse/src/orchestra/journal.ts
  ├─ Risk register openspec/_methodology/orchestra-risk-register.md (Δ2)
  ├─ Retention policy: 6-month no-auto-delete (Δ4)
  └─ Checkpoint: `pnpm test orchestra/journal` зелёный

Phase A' — Observability (1 week, parallel with B start)
  ├─ Langfuse self-host docker-compose в studio infra (Δ1)
  ├─ OpenLLMetry SDK install в packages/muse + packages/forge
  ├─ Anthropic API wrapper через Langfuse decorator
  ├─ Cost alert thresholds, eval-as-judge sampling pipeline
  └─ Checkpoint: тестовый trace из Forge build виден в Langfuse UI

Phase B — Maestro mode (3 weeks)
  ├─ packages/muse/src/modes/maestro/ system prompt + runner
  ├─ Routing function pure: route(snapshot, request) → decision
  ├─ Decision-on-ambiguity formatter
  ├─ Cost circuit breaker per-session cap (Δ5)
  ├─ Journal + FS mirror writes
  ├─ Tool registry filter (allowlist enforcement)
  └─ Checkpoint: 5 fixture routing cases проходят unit tests

Phase C — Agent integration (1 week)
  ├─ Muse Architect emits `spec_promoted` после promote-to-disk
  ├─ Forge orchestrator emits per-stage events (plan_written, build_started, build_completed, pr_opened, etc.)
  └─ Checkpoint: `forge build RAP-TEST` пишет ≥5 entries в orchestra_journal

Phase D — Triggers (1-2 weeks)
  ├─ `forge maestro route <key>` / `status` / `tail` / `decide` CLI subcommands
  ├─ Linear webhook endpoint `apps/studio/api/maestro/webhook`
  ├─ Indirect prompt injection guard на issue body parser (Δ6)
  └─ Checkpoint: webhook test payload → journal entry created

Phase E — End-to-end (1 week)
  ├─ Test scenario RAP-MVP-TEST run
  ├─ Bug fix, polish
  ├─ Documentation: openspec/current/agents/maestro.md amendment (applied)
  └─ Checkpoint: success criterion (см. 4.4) проходит

Phase F — Lessons writeback (1 week, optional MVP+)
  ├─ Maestro post-mortem ~/.maestro/lessons/<RAP-N>.md generator (Δ3)
  ├─ Cross-issue patterns aggregator
  └─ Checkpoint: post 5 completed issues, lessons file generated и читается next Maestro invocation
```

### 4.4 Success criterion — updated

**v1.0 MVP считается shipped когда все четыре проходят:**

1. **End-to-end happy path:** Linear issue `RAP-MVP-TEST` → Maestro routes → Muse Architect → spec promoted → Maestro re-routes → Forge plan → Forge build → PR opened. Без manual intervention в шагах routing.
2. **Journal coherence:** `SELECT * FROM orchestra_journal WHERE work_item_key='RAP-MVP-TEST' ORDER BY seq` — ≥8 entries, все с populated required fields, `seq` monotonic, content_hash valid.
3. **Observability:** В Langfuse UI виден полный trace tree от Maestro routing through Muse session through Forge build with нестед spans, cost aggregated, latency measured.
4. **Risk register active:** `openspec/_methodology/orchestra-risk-register.md` существует с 12 рисками + defenses map; ни один из top 4 (R1–R4) не unmitigated.

### 4.5 Decision points oставшиеся

Все мои предыдущие recommendations Pavel принял. После этого research-pass'а остаются три новых open questions:

| # | Вопрос | Default | Override candidate |
|---|---|---|---|
| Q-OBS-1 | Langfuse vs Laminar для observability | **Langfuse** (prompt mgmt + eval harness + free MIT) | Laminar (если debugging Forge sessions станет primary pain) |
| Q-OBS-2 | Cost cap per Maestro session — какой $ | **$5** (~250K tokens at Opus 4.7 input pricing) | Любой — Pavel решает по budget tolerance |
| Q-LEARN-1 | Phase F (lessons writeback) — в v1.0 или v1.1? | **v1.0 optional, ship if Phase E финиширует с запасом времени** | v1.1 (если v1.0 уже tight) |

Все три могут быть decided на старте Phase A. Не блокеры.

---

## Stream 5 — Sources & methodology

### Risks

- Cemri et al. *Why Do Multi-Agent LLM Systems Fail?* MAST taxonomy, NeurIPS 2025 Datasets and Benchmarks Track (arXiv:2503.13657, 1,642 traces analysed)
- Vadlamudi *Why AI Agents Fail* — 4-dimensional taxonomy of failure modes (SSRN, April 2026)
- Hammond et al. *Multi-Agent Risks from Advanced AI* — Cooperative AI Foundation taxonomy (arXiv:2502.14143)
- Microsoft Security Response Centre AI failure-mode taxonomy (already cited в Maestro spec)
- OWASP Top 10 for Agentic Applications 2026 (ASI01–ASI10), OWASP Agentic Skills Top 10
- OWASP LLM Top 10 2025 (LLM01:2025 prompt injection)
- Galileo *7 AI Agent Failure Modes* (April 2026)
- Augment Code *Why Multi-Agent LLM Systems Fail* (April 2026)
- Gartner March 2026 data and analytics predictions

### EU AI Act

- Regulation (EU) 2024/1689 — official AI Act text
- Article 14 (human oversight), Article 17 (QMS), Article 26 (deployer obligations including 6-month log retention)
- Digital Omnibus VII (political agreement 7 May 2026)
- Council & Parliament press release on simplification, 7 May 2026
- McKenna Consultants *EU AI Act High-Risk Compliance Technical Readiness Guide* (Feb 2026)
- Secure Privacy *EU AI Act 2026 Key Compliance Requirements* (May 2026)
- Latham & Watkins *AI Act Update* (May 2026)

### Observability

- LangSmith documentation (langchain.com/langsmith/observability)
- Langfuse documentation (langfuse.com)
- Laminar comparison (laminar.sh/article/2026-04-23-top-6-agent-observability-platforms)
- AgentMarketCap *OpenTelemetry for AI Agents* (April 2026)
- AIMultiple *15 AI Agent Observability Tools 2026* benchmark
- LangChain State of Agent Engineering Survey Nov–Dec 2025 (n=1,340)
- OpenTelemetry GenAI semantic conventions (status Experimental)
- OpenInference (Arize) span kinds taxonomy
- DigitalApplied *Agent Observability Platforms 2026*

### Memory & learning

- Chhikara et al. *Mem0: Building Production-Ready AI Agents with Scalable Long-Term Memory* — ECAI 2025 (arXiv:2504.19413)
- Mem0 *State of AI Agent Memory 2026* (mem0.ai, May 2026)
- Atlan *Best AI Agent Memory Frameworks 2026*
- Hermes OS *AI Agent Memory Systems 2026*
- ALMA *Learning to Continually Learn via Meta-learning Agentic Memory Designs* (arXiv:2602.07755, Feb 2026)
- PAHF — Meta *Learning Personalized Agents from Human Feedback* (Feb 2026)
- Christiano et al. *Deep Reinforcement Learning from Human Preferences* (2017, RLHF foundation)
- Anthropic Constitutional AI paper (2022, RLAIF)
- TechTarget RLHF definition
- GoCodeo *Training Agentic AI with Human Feedback Best Practices*
- Wikipedia *RLHF* article

### Methodology note

Все findings cited только если voor empirically validated (n provided, benchmark named, или paper peer-reviewed). Vendor marketing claims (e.g. «99% accuracy», «10x productivity») в основной текст не пошли — отмечены как vendor-claim в notes. EU AI Act regulatory text — primary source via artificialintelligenceact.eu и EC digital-strategy.

---

## Triage

- **Feeds into**: future `openspec/changes/add-orchestra-mvp/` package (proposal / design / tasks / verification); new `openspec/_methodology/orchestra-risk-register.md` (R1–R12 table); amendment to `openspec/current/agents/maestro.md` for Δ4 retention policy and Δ5 cost circuit breaker; new change folder for Langfuse self-host setup (Δ1). <!-- openspec-refs: skip-line -->
- **Status**: open — Δ1–Δ6 additions require founder ratification before MVP change package can be drafted; three new open questions (Q-OBS-1, Q-OBS-2, Q-LEARN-1) need defaults confirmed.
- **Decision or finding**: MAST data identifies specification undefinedness (41.77% of multi-agent failures) as the dominant risk class — already partially mitigated by OpenSpec methodology but requires JSON-schema-per-event-type and content-hash discipline at the journal layer. OWASP ASI01 (indirect prompt injection) and ASI08 (cost runaway) are the only critical agentic-security risks for the studio's single-principal MVP topology; both have lightweight mitigations (strip-and-quote + per-session cap). Rapoport is **not** a high-risk system under EU AI Act Annex III, but three Act principles (6-month log retention, human-oversight-by-design, structured incident response) apply as good engineering and are inexpensive to satisfy. Langfuse self-host wins the observability bake-off for Rapoport's framework-agnostic, single-principal, free-platform-preferred constraints. RLHF/fine-tuning is out of scope (Anthropic API gates it); prompt-as-policy iteration loop via Langfuse + lessons-learned writeback is the v1.0 learning mechanism.
- **Applied**: pending founder ratification of Δ1–Δ6 and Q-OBS-1/Q-OBS-2/Q-LEARN-1.
