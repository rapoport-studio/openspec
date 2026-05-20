# Конкурентный анализ оркестра Rapoport Studio

> **Status:** complete (single-pass research, 2026-05-20)
> **Owner:** Pavel Rapoport <hello@rapoport.studio> (founder).
> **Authors:** Pavel Rapoport <hello@rapoport.studio> (founder, primary); Claude Opus 4.7 (Anthropic) — secondary synthesis across 50+ public sources, naming-pattern survey, per-agent function-mapping tables.
> **Method:** Single-pass secondary synthesis. Six agents × ~3–4 competitors each, mapped via public comparison articles, vendor docs, pricing pages, and benchmark reports (SWE-Bench, Humanity's Last Exam, Terminal-Bench). No first-party measurement; no interviews. Source list at end of file.
> **Reusable for:** ТЗ drafting for orchestra agents; positioning copy; pitch deck competitive section; future per-agent spec updates (Maestro / Muse / Atlas / Echo / Pulse / Forge). Authorship: open.
> **Opened:** 2026-05-20
> **Companion to:** [`./competitor-landscape-synthesis.md`](./competitor-landscape-synthesis.md) (geographic competitor sweep), [`./competitive-positioning-sdd-frameworks.md`](./competitive-positioning-sdd-frameworks.md) (SDD-tooling positioning), [`./orchestra-component-stack-2026.md`](./orchestra-component-stack-2026.md), [`./ai-cli-landscape-2026-05.md`](./ai-cli-landscape-2026-05.md)
> **Feeds into:** Open strategic questions Q1–Q6 (§ 8.4 / § 9.2) — Echo feature scope, Pulse internal-vs-product positioning, Atlas beat selection, Cast-layer external visibility, public functional descriptors, Forge parallel-agents priority.

---

## TL;DR — что нашли

1. **Конкурент-на-конкурент сборки оркестра как у Rapoport — нет.** Конкурируют **сегменты по агенту**, а не оркестр целиком. Студии и AI-компании собирают похожие команды (orchestrator + coder + researcher), но никто публично не позиционирует это как «orchestra of six agents с философией Club 27 и spec-driven методологией».
2. **Naming в индустрии сходится к четырём паттернам.** У Pavel — паттерн **persona names + functional descriptors на public** (Maestro = «Conductor», Forge = «Builder»), плюс **внутренний слой Cast (Club 27)**. Это редкий гибрид: большинство публичных продуктов берут один паттерн.
3. **Функциональный gap-анализ показывает четыре категории.**
   - **Match.** Forge, Maestro, Atlas, Pulse — имеют прямых конкурентов и одинаковую базовую функциональность.
   - **Unique-shape.** Muse как multi-mode персона (одна личность, четыре режима) — у конкурентов это разбито по разным продуктам.
   - **Под-функциональность.** Echo в задуманной форме (studio-mode + client-mode под review) ближе к Jasper/Copy.ai workflows, но weaker bench без `Brand Voice IQ`-аналога.
   - **Methodology layer.** SDD (spec-driven), HitL gates, journal/audit — это **методологический слой поверх агентов**, который у Pavel жёстко прошит, а у конкурентов лежит опционально или вообще отсутствует.
4. **Главный наблюдаемый тренд (Q1 2026).** Все крупные игроки в кодинге за две недели в феврале 2026 запустили multi-agent (Grok Build, Windsurf Manager, Claude Code Agent Teams, Codex CLI Agents SDK, Devin parallel sessions). «Параллельные агенты на разных частях кодбейза — теперь table stakes». Это значит, что **архитектура оркестра у Pavel становится мейнстримом**, и окно для дифференциации сужается через **методологию и под-функциональность, а не через сам факт «у нас несколько агентов»**.

---

## 0. Индустриальный контекст

### 0.1 Размер и темп рынка

- Multi-agent AI стал доминирующей архитектурой за 2025: 72% корпоративных AI-проектов теперь multi-agent (против 23% в 2024).
- LangGraph — 47M downloads/мес; CrewAI — заявляет 60%+ Fortune 500 в late 2025 (с учётом free и POC); AutoGen → Microsoft Agent Framework (GA Q1 2026).
- AI coding agents: Cursor — 360K paying users; Devin — $2B valuation; OpenHands — 72K GitHub stars, $18.8M Series A; Aider — 4M+ developers; Claude Code — топ-1 по SWE-Bench Verified depth (80.9%); Codex CLI — топ по Terminal-Bench (77.3%).
- AI accounting agents: Pilot запустил «полностью автономного AI Accountant» в феврале 2026. Digits использует trademark «Agentic General Ledger» (AGL).

### 0.2 Четыре паттерна naming в индустрии

| Паттерн | Что это | Примеры | Когда используют |
|---|---|---|---|
| **Persona (proper name)** | Человеческое имя — намёк на индивидуальность | Devin (Cognition), Harvey (legal), Jasper (content), Fin (Intercom), Claude (Anthropic), Eva (Engine), Olive (Williams Sonoma), Sage (Mesha) | B2C, customer-facing, когда нужен эмоциональный entry-point |
| **Brand identity** | Имя из идентичности компании | Einstein (Salesforce), Otter (Otter.ai), Linear (Linear), Microsoft Copilot | Когда продукт — extension бренда |
| **Functional title** | Роль как название | Copilot (GitHub), Agent Mode, Coding Agent, Cascade (Windsurf), Plan Mode (Cursor) | B2B, enterprise, когда важна понятная функция |
| **Generic technical** | Просто «AI» / «assistant» | Notion AI, Perplexity Comet's «assistant», Zapier «Copilot» | Когда не хотят антропоморфизировать |

**Наблюдение из 200 pitch-deck выборки (Park Rangers Capital):** 70% AI-агентов с человеческими именами получают мужские имена, 20% — женские, 10% — gender-neutral. **Шаблон ярко выражен в knowledge-work агентах** (Harvey, Devin, Jasper).

**Salesforce design-guideline:** для длинных интерфейсов агент-имя должно быть **под 10 символов, 1–2 слова**, паттерн `Name + Agent`.

**Гибридная стратегия:** некоторые компании дают агенту имя+аббревиатуру (Eva = «Engine Virtual Agent», Mae = «Multi-purpose Admin Expert»). Это даёт и persona, и semantic-anchor.

### 0.3 Где сидит Rapoport

Pavel использует **трёхслойный naming**, который я ни у одного другого вендора не видел в одном продукте:

| Слой | Пример | Кто видит |
|---|---|---|
| **Functional descriptor** (public-safe) | «Conductor», «Architect», «Builder», «Research», «Voice», «Finance» | Клиенты на сайте |
| **Persona name** (internal + public) | Maestro, Muse, Atlas, Echo, Pulse, Forge | Студия + спецификации |
| **Cast layer** (internal only) | Brian Jones, Kurt Cobain, Pigpen, Jim Morrison, D. Boon, Jimi Hendrix | Только команда |

Это **уникальный гибрид**: «persona + brand identity + culture reference», где последний слой невидим публично. Indie precedent: я нашёл, что Pixar/Disney дают похожие внутренние codenames (например, ранние Cars-проекты), но не как агентский паттерн.

---

## 1. Maestro — Координатор (Conductor)

### 1.1 Industry equivalent

В индустрии эта роль называется тремя именами:
- **Supervisor agent** (LangGraph, LangChain docs)
- **Orchestrator agent** (Microsoft Azure docs, AutoGen)
- **Manager agent** (CrewAI, Microsoft Magentic-One)

Также появилось четвёртое — **Magentic** (от Microsoft, специфический паттерн с task-ledger).

### 1.2 Прямые конкуренты (4)

| Продукт | Что делает | Naming | Forks of architecture |
|---|---|---|---|
| **LangGraph Supervisor** | Центральный координатор делегирует work специалистам через directed graph. State persistence via checkpointing. Time-travel debugging. | `supervisor_node`, generic | Graph-based |
| **CrewAI Manager** | Role-based crews: roles defined as `Agent(role='CEO', goal='increase revenue')`. Human-in-the-loop checkpoints в task execution. | Functional ('Manager', 'CEO') | Role-based |
| **Microsoft Magentic-One / Agent Framework** | Manager agent + Task and Progress Ledger. Loops: invoke → evaluate goal → terminate. HITL опционален. AutoGen GroupChat преемник. | 'Manager', 'Magentic' | Conversational + ledger |
| **OpenAI Agents SDK** | Explicit handoffs между агентами. Context variables ephemeral. Built-in evaluations и guardrails. | Generic ('handoff') | Handoff-based |

### 1.3 Функциональный inventory (что считается стандартом)

1. Routing (decide which agent gets the work)
2. State persistence / checkpointing (recover from crash)
3. Human-in-the-loop pauses (decision gates, approvals)
4. Audit trail / journal (who decided what, when)
5. Cost envelopes (per-session budgets)
6. Idempotent execution (replay-safe)
7. Failure handling (retry, escalation, isolation)
8. Plan-mode (read-only planning before execution)
9. Sub-agent delegation (hierarchical levels)
10. Observability (tracing, metrics)

### 1.4 Mapping: Maestro vs industry

| Function | Maestro (spec) | LangGraph Supervisor | CrewAI Manager | Magentic-One | OpenAI Agents SDK |
|---|---|---|---|---|---|
| Routing | ✅ HitL gate per non-trivial route | ✅ Conditional edges | ✅ Process types | ✅ Adaptive planning | ✅ Handoffs |
| State persistence | ✅ `orchestra_journal` (append-only Supabase) | ✅ MemorySaver / checkpointing | ⚠ Task outputs sequential | ⚠ Ledger-based | ⚠ Context variables ephemeral |
| HitL gates | ✅ **mandatory for non-trivial routes** | ⚠ Optional integration | ✅ Built-in checkpoints | ✅ Optional human-as-manager | ⚠ Manual |
| Audit trail | ✅ Dual sink: Supabase + filesystem mirror | ⚠ via LangSmith add-on | ⚠ Limited | ⚠ via observability tools | ⚠ Runtime events |
| Cost envelopes | ✅ Per-session, surface as decision | ❌ Manual | ❌ Manual | ❌ Manual | ⚠ Manual |
| Idempotent routing | ✅ Pure function `route(snapshot, request)` | ⚠ Depends on impl | ❌ | ❌ | ❌ |
| Decision-laundering protection | ✅ `originating_agent_id` field | ❌ | ❌ | ❌ | ❌ |
| Plan-mode | ✅ Plan before handoff | ⚠ Custom | ⚠ Custom | ✅ Task ledger | ❌ |
| Sub-agent delegation | ✅ Bilateral edges per agent | ✅ Sub-graphs | ✅ Hierarchy | ✅ | ✅ |

### 1.5 Verdict

**Maestro покрывает базовый industry standard плюс три уникальные вещи:**

1. **Decision-laundering protection** через `originating_agent_id` — я не нашёл этого ни у одного конкурента в явном виде. Это решает реальную проблему («supervisor стал attribution layer для решений, которые на самом деле принимал другой агент»), но индустрия её обычно игнорирует.
2. **Filesystem rationale mirror** (`~/.maestro/journal/<date>.md`) — двойная sink (DB + readable markdown) — редкий паттерн. Конкуренты используют один (либо БД, либо лог-файлы).
3. **Cost guard сюрфэйсится как decision** — Maestro не молча останавливается на budget cap, а surface это Pavel'у как «extend session?». Большинство конкурентов — silent termination.

**Где Maestro слабее (или не определён):**

- **Нет встроенной observability stack** — LangGraph имеет LangSmith, OpenAI имеет own runtime events. Pavel пока имеет `orchestra_journal`, но grafical surface не описан.
- **Substrate-pending status** — Maestro в спеке, но runtime ещё не написан. Это не функциональный gap, но временной.

---

## 2. Muse — AI substrate (multi-mode)

### 2.1 Industry equivalent — **редкая категория**

Muse в нынешней форме — **«одна персона, четыре режима»** (Architect / Builder / Scout / Canvas) — **не имеет точного прямого конкурента**. Индустрия обычно разбивает эти роли на отдельные продукты.

Ближайшие аналоги по отдельным режимам:

| Режим Muse | Industry equivalent | Конкретные продукты |
|---|---|---|
| **Architect** (spec authoring) | Spec-driven development (SDD) tooling | GitHub Spec Kit, AWS Kiro, Claude Code Skills (`/sdd:specify`), Tessl, BMAD, OpenSpec |
| **Builder** (executes plans) | AI coding agent (cloud / CLI) | Devin, OpenHands, Codex CLI, Claude Code (build mode) |
| **Scout** (reads existing code) | Codebase intelligence | Sourcegraph Cody, Augment Code, GitHub Repository chat, Greptile |
| **Canvas** (public conversation) | Customer-facing AI agent | ChatPRD-style intake, Intercom Fin, Salesforce Agentforce |

### 2.2 Конкуренты по «multi-mode persona» концепту

Я нашёл **только два** продукта с похожей multi-mode персоной (один продукт, несколько режимов, единая личность):

1. **Roo Code** (открытый) — 4 структурированных режима: **Architect / Code / Debug / Orchestrator**. Это самый близкий аналог Muse по концепту. Раунд $8M в декабре 2025; 1.5M users; «superset of Cline» по фидбеку сообщества.
2. **Cursor** — Plan Mode + Agent Mode + Tab — три режима, но это **режимы одной утилиты, не персоны** (нет личностного через-режимного консистентного голоса).

### 2.3 Прямые конкуренты по Architect mode (SDD authoring)

| Продукт | Что делает | Naming |
|---|---|---|
| **GitHub Spec Kit** | CLI (`specify`) с slash-commands: `/constitution`, `/specify`, `/clarify`, `/plan`, `/tasks`, `/analyze`, `/implement`, `/checklist`. Model-agnostic. | Functional ('Spec Kit') |
| **AWS Kiro** | Standalone IDE (VS Code fork). Spec + plan + tasks + code в одном workspace. Глубокая AWS-интеграция. | Brand identity ('Kiro' = company brand) |
| **ChatPRD** | AI PM agent: пишет PRD, генерирует requirements, спецификации, интегрируется в Linear. 100K+ PMs. | Functional ('PRD' = Product Requirements Doc) |
| **Claude Code Skills** | `/sdd:specify`, `/sdd:plan` — SDD workflow в терминале без выхода из Claude Code. | Functional |

### 2.4 Mapping: Muse Architect vs SDD competitors

| Function | Muse Architect | Spec Kit | Kiro | ChatPRD | Claude Code Skills |
|---|---|---|---|---|---|
| Spec authoring (proposal/design/tasks) | ✅ 3-file output | ✅ Multi-file | ✅ Multi-file | ✅ PRD-shaped | ✅ Phase-based |
| Linear integration | ✅ Native | ⚠ Manual | ❌ | ✅ Native | ⚠ Manual |
| Multi-mode persona | ✅ Architect+Builder+Scout+Canvas plan | ❌ Single-mode CLI | ❌ Single product | ❌ Single mode | ⚠ Plan vs Agent |
| Project-bound canvas storage | ✅ Supabase `canvas_sessions` | ❌ Filesystem | ✅ IDE workspace | ✅ Cloud projects | ❌ Filesystem |
| HitL on spec promotion | ✅ Pavel approves promote-to-disk | ⚠ Manual | ✅ IDE diff | ✅ Review step | ⚠ Plan mode |
| Telegram surface | ✅ `@rs_canvas_bot` | ❌ | ❌ | ❌ | ❌ |
| Underlying SDD methodology | ✅ OpenSpec (custom) | ✅ EARS notation | ✅ Custom | ⚠ PRD only | ✅ Skills-defined |

### 2.5 Verdict

**Muse — самая дифференцированная часть оркестра Pavel.** Три вещи, которых нет у конкурентов:

1. **Single-personality, multiple-modes design.** Roo Code ближе всех, но они называют это режимами утилиты, не персонажа. У Pavel — это явно «одна личность, разные системные промпты», что философски и операционно отличается.
2. **Project-bound canvas как primary surface.** Конкуренты используют либо файловую систему (Spec Kit, Claude Code), либо IDE workspace (Kiro). У Pavel — Supabase-bound canvas с RLS — это шаг в сторону **multi-user spec authoring**, который ни Spec Kit, ни Kiro не поддерживают.
3. **OpenSpec как собственная методология.** SDD instruments в индустрии обычно адаптируют чужой стандард (EARS, Constitutional SDD). У Pavel — кастомный шаблон proposal/design/tasks/verification, привязанный к доменной модели.

**Слабые места:**

- **Scout и Canvas deferred.** В текущей реальности (2026 май) запущены только Architect и Builder. Эта дискретность — gap vs Claude Code, который имеет аналоги всех четырёх через skills.
- **Brand voice IQ-эквивалент отсутствует.** Jasper IQ запоминает tone и стиль клиента; у Muse пока нет аналога для Canvas mode.
- **Telegram surface слабее ChatPRD-Linear pipeline.** ChatPRD имеет более полную интеграцию с Linear/Slack/Jira.

---

## 3. Atlas — World research

### 3.1 Industry equivalent

В индустрии это называется **Deep Research agents** — категория, которая взорвалась между концом 2024 и началом 2025.

### 3.2 Прямые конкуренты (4)

| Продукт | Что делает | Naming | Speciality |
|---|---|---|---|
| **OpenAI Deep Research** | Agent — поиск + reasoning, генерирует длинные reports. Built на o3, end-to-end RL trained. ~3 min execution. | Functional + brand | General web + academic verticals |
| **Perplexity Deep Research** | Iterative search, citation-rich reports. 21.1% на Humanity's Last Exam (бенчмарк по 100 областям). | Brand + functional | Real-time web с inline citations |
| **Gemini Deep Research** | Mixture-of-Experts на Gemini 2.5, интегрируется с Google Search/Drive/Docs ecosystem. | Brand + functional | Ecosystem integration |
| **Elicit Research Reports** | Academic-focused: peer-reviewed literature, паттерны извлечения данных, one-click reports. | Brand (functional) | Academic / scientific |

### 3.3 Функциональный inventory (стандарт для Deep Research)

1. Multi-step search planning (1–2 минуты разбора задачи)
2. Iterative web/corpus retrieval
3. Citation tracking inline
4. Long-form report generation (5–50 страниц)
5. Domain-vertical support (academic / web / business)
6. Source-quality grading
7. Approval-before-execution (plan first, then search)
8. Refresh / re-query patterns

### 3.4 Mapping: Atlas (planned) vs Deep Research

| Function | Atlas (spec) | OpenAI DR | Perplexity DR | Gemini DR | Elicit |
|---|---|---|---|---|---|
| Multi-step search | ✅ Plan + execute | ✅ End-to-end RL | ✅ Iterative | ✅ MoE search | ✅ Iterative |
| Citation tracking | ✅ TBD format | ✅ inline | ✅ inline | ✅ inline | ✅ inline |
| Long-form output | ✅ `_research/world/<beat>/` | ✅ Reports | ✅ Reports | ✅ Reports | ✅ Reports |
| **Beat-style continuous coverage** | ✅ Tech beats, regional reports, society reads, cinema, philosophy | ⚠ One-shot reports | ⚠ One-shot | ⚠ One-shot | ⚠ One-shot |
| Domain vertical | ⚠ Multi-domain (tech+region+society+culture) | ✅ General | ✅ General | ✅ General | ✅ Academic |
| Filesystem storage | ✅ `_research/world/` markdown | ❌ Cloud only | ❌ Cloud only | ❌ Cloud only | ❌ Cloud only |
| Studio-context grounding | ✅ TBD | ❌ | ❌ | ❌ | ❌ |
| Distinction from code research | ✅ Explicit (different agent: Scout = code, Atlas = world) | n/a | n/a | n/a | n/a |

### 3.5 Verdict

**Atlas (планируется, не запущен) — концептуально отличается от Deep Research в одной важной точке:** **beat-style continuous coverage** vs **one-shot reports**.

OpenAI / Perplexity / Gemini Deep Research — это **on-demand reports**: задал запрос, получил отчёт. Atlas задуман как **agent с регулярными beats** (tech beats, regional reports, society reads), которые наполняют `_research/world/` непрерывно. Это ближе к **журналистскому beat-reporter** паттерну, чем к утилитному «research as a service».

**Сходство с Elicit** — она тоже про continuous literature watch, но только для академики.

**Сходство с newer experiments** — Anthropic «Project Glasswing», DeepMind Glasswing-equivalents — но это всё ранние beta.

**Gap vs конкуренты:**
- Atlas planned, не запущен — реальная функциональность не валидирована.
- **Citation/source-quality discipline** не описана в спеке (у Perplexity и OpenAI DR это central feature).
- Storage в filesystem (`_research/world/`) — плюс для repro, минус для discoverability (у конкурентов есть search UI).

**Unique positioning:**
- **Atlas + Echo edge** (`Atlas → Echo` — world research feeds market positioning) — это **продуктовая связка**, которой у standalone Deep Research нет.
- **Multi-domain coverage** (tech + region + society + cinema + philosophy) — конкуренты обычно либо «всё» (web), либо «академика». Atlas — кастомный набор beat'ов.

---

## 4. Echo — Market voice

### 4.1 Industry equivalent

**AI marketing / content agents** — зрелая категория, мощные продукты, миллиарды в revenue.

### 4.2 Прямые конкуренты (4)

| Продукт | Что делает | Naming | Revenue / scale |
|---|---|---|---|
| **Jasper** | «Always-on content engine» с Brand Voice IQ. 100+ marketing «Apps» (templates). Agentic AI с 2025 — кампания planning, content creation, SEO. Canvas (long-form), Grid, Studio. | Persona ('Jasper') | $88M revenue в 2025 (down from $120M в 2023), 100K+ customers |
| **Copy.ai** | Multi-agent **Workflows** — chain content steps + research + brand-voice + human-approval gates. Лидер по workflow-builder. | Brand + functional | $36–186/seat |
| **Writer** | Enterprise content platform с Knowledge Graph, agent governance, custom agents. IT-led procurement. | Brand identity | $200M+ revenue est. |
| **MarketMuse / Anyword / Frase** | Specialists: MarketMuse SEO, Anyword performance-prediction (CTR/conversion), Frase SERP analysis. | Functional + brand | Niche tools, $14–114/seat |

### 4.3 Функциональный inventory

1. **Brand voice modeling** (учит tone, style guides, audience)
2. Multi-channel content (blog / social / email / ad copy / product descriptions)
3. **Workflow chaining** (research → draft → review → publish)
4. **Human-approval gates** (review before publish)
5. SEO optimization (Frase-style SERP analysis)
6. Performance prediction (Anyword-style CTR/conversion)
7. Knowledge base / context layer (Jasper IQ, Writer Knowledge Graph)
8. Multi-brand / multi-property support (отдельные voices для разных property)
9. Compliance / governance (enterprise audit trails)
10. Agentic capabilities (autonomous campaign planning)

### 4.4 Mapping: Echo (planned) vs marketing platforms

| Function | Echo (spec) | Jasper | Copy.ai | Writer |
|---|---|---|---|---|
| Brand voice modeling | ⚠ TBD ('voice' implied) | ✅ Brand IQ | ⚠ Shallower | ✅ Writer style guide |
| Multi-channel content | ⚠ TBD | ✅ Apps for all channels | ✅ Workflows | ✅ Multi-format |
| Workflow chaining | ⚠ TBD | ✅ Canvas+Grid+Studio | ✅ **Best-in-class** | ✅ Enterprise workflows |
| **Always under review (no auto-publish)** | ✅ Explicit policy | ⚠ Optional | ⚠ Optional | ✅ Governance layer |
| Studio-mode vs Client-mode separation | ✅ Two modes by spec | ❌ Same surface | ❌ Same surface | ⚠ Multi-tenant |
| Performance prediction | ❌ | ⚠ Limited | ❌ | ⚠ Limited |
| SEO optimization | ❌ | ✅ Semrush integration | ⚠ Limited | ✅ Custom |
| Multi-brand voices | ✅ Per-client | ✅ Per-property | ✅ Per-workspace | ✅ Per-brand |
| Knowledge graph / context | ⚠ TBD | ✅ Jasper IQ | ⚠ Limited | ✅ **Best-in-class** |

### 4.5 Verdict

**Echo — самый weakly-specified агент в оркестре.** Spec говорит «studio-mode и client-mode, всегда под review», но **функциональный inventory не детализирован**. Это значит:

- Конкуренты (Jasper, Copy.ai, Writer) имеют 5–10 лет head-start и **гораздо более развитую функциональность** (Brand Voice IQ, Knowledge Graph, performance prediction).
- Echo в текущей форме — **defensive minimum**: «не auto-publish, всегда review». Это правильная политика, но **не отличительный feature**.

**Что у Pavel есть, чего нет у конкурентов:**

- **Atlas → Echo edge.** World research как input в drafts — Jasper и Copy.ai полагаются на user-provided context. Если Atlas доставляет beat-ready context, Echo стартует с **более глубокой основой**, чем «здесь brand voice, пиши копию».
- **Studio-mode vs client-mode separation.** Jasper-аналог — это per-workspace. У Pavel это **онтологическое** разделение: studio-mode под review Pavel-ом для собственной поверхности, client-mode под review для клиентских engagements. Разные политики, разные approval flows.

**Gaps vs конкуренты:**

- Brand Voice IQ analog отсутствует
- SEO/performance optimization не упомянуто
- Knowledge Graph не описано
- Multi-channel inventory не определён

**Recommendation для ТЗ:** Echo требует **detailed function spec** до запуска, иначе будет «AI writes drafts, Pavel reviews» — что и Claude/ChatGPT уже делают free.

---

## 5. Pulse — Books / Finance

### 5.1 Industry equivalent

**AI accounting agents / autonomous bookkeeping** — категория взлетела в 2025–2026. **Pilot** в феврале 2026 объявил «первого полностью автономного AI Accountant для SMBs».

### 5.2 Прямые конкуренты (4)

| Продукт | Что делает | Naming | Pricing |
|---|---|---|---|
| **Pilot AI Accountant** | Полностью автономный bookkeeping — onboarding до monthly close без human intervention. Built на Pilot's многолетнем human-bookkeeping опыте. Plus 24/7 AI advisor для KPI/trends. | Brand identity | Custom; от $499/mo bookkeeping plans |
| **Digits AGL (Agentic General Ledger)** | Trademark «Agentic General Ledger» — multi-step accounting workflows автономно на data layer. NLP + vector similarity + layout-aware LLMs для chart of accounts. Live financial dashboards. | Functional ('AGL') + brand | Free trial; pricing on request |
| **Vic.ai** | AP-specific automation: 85% zero-touch invoice processing в 6 months. VicInbox / VicPay / VicAnalytics. SOC 2. | Brand + functional ('Vic'+role) | Custom; $500–$800/mo small clients |
| **Docyt** | Multi-entity bookkeeping. Агенты GARY (Generative Accounting Retrieval) и Docyt Copilot. P&L variance, KPI dashboards. 30+ POS integrations. | Brand + persona ('GARY') | $699/mo |

### 5.3 Функциональный inventory

1. Transaction categorization (bank/credit card feeds)
2. Account reconciliation
3. Month-end close automation
4. AP automation (invoice extraction + GL coding + approval routing)
5. P&L variance analysis
6. KPI dashboards
7. Audit trail / compliance (SOC 2)
8. Multi-entity support
9. Cash flow forecasting
10. **Strategic / advisory layer** (CFO-equivalent): pricing, fundraising, runway
11. Tax preparation
12. Bill pay / invoicing
13. Real-time financial reporting

### 5.4 Mapping: Pulse (planned) vs accounting agents

| Function | Pulse (spec) | Pilot | Digits AGL | Vic.ai | Docyt |
|---|---|---|---|---|---|
| Transaction categorization | ⚠ TBD | ✅ Autonomous | ✅ NLP-based | ⚠ AP-focused | ✅ |
| Reconciliation | ⚠ TBD | ✅ | ✅ | ⚠ Limited | ✅ |
| Month-end close | ⚠ TBD | ✅ | ✅ | ⚠ Limited | ✅ Under-hour |
| **Cost / P&L / pricing** | ✅ Explicit | ✅ | ✅ | ⚠ AP-only | ✅ |
| **Monetization paths** | ✅ Explicit (unique vs accounting) | ⚠ CFO tier | ⚠ Limited | ❌ | ⚠ Limited |
| **Decision anchoring via numbers** | ✅ Edge `Pulse → Maestro` | ❌ | ❌ | ❌ | ❌ |
| Integration with own studio operations | ✅ Native | ❌ Standalone | ❌ Standalone | ❌ Standalone | ❌ Standalone |
| Multi-entity | ⚠ TBD | ✅ | ✅ | ✅ | ✅ Multi-entity |
| Compliance / audit | ⚠ TBD | ✅ | ✅ SOC 2 II | ✅ SOC 2 | ✅ |
| Strategic / CFO advisory | ⚠ TBD | ✅ Higher tier | ⚠ Limited | ❌ | ✅ Copilot |

### 5.5 Verdict

**Pulse — отличается от accounting-агентов одной важной вещью: позиционируется не как standalone bookkeeping product, а как «numbers anchor for orchestra decisions».**

Spec говорит: «Anchors decisions in numbers. Conceptually owns ledger functionality (implementation home decided in Layer 1)». То есть Pulse — это **internal CFO/financial-controller для самой студии**, а не product for sale.

**Что это даёт:**

- **Edge `Pulse → Maestro`** — финансовые числа cited в routing decisions. Конкуренты предоставляют financial data в дашборды для людей; Pulse — для других агентов как input.
- **Monetization paths как часть scope** — не bookkeeping, а **pricing/monetization analysis**. Этого у Pilot/Digits/Vic.ai в явном виде нет (есть CFO tier, но это human, не agent).
- **`_finance/` storage в repo** — financial state living alongside specs. Конкуренты — облачные платформы.

**Gaps:**

- **Pulse — самая поздняя phase (v1.3).** Pilot, Digits, Vic.ai в production. Pulse не запущен.
- **Compliance/audit layer не описан.** SOC 2 как table stakes для accounting — конкуренты все имеют. Pulse spec говорит «TBD».
- **Если Pulse будет масштабироваться, нужна интеграция с реальными bank feeds / GL backends** — это огромная работа, которую Pilot/Digits сделали за годы.

**Recommendation для ТЗ:** Решить — Pulse это (a) **internal-only financial controller для студии** или (b) **client-facing product** в будущем. Это влияет на scope: (a) можно ограничить P&L + pricing models на markdown в `_finance/`; (b) — нужна интеграция с QuickBooks/Xero и SOC 2.

---

## 6. Forge — Engineering CLI

### 6.1 Industry equivalent

**AI coding agents** — самый перенаселённый сегмент. **Четыре под-категории:** IDE extensions / dedicated IDEs / CLI tools / cloud platforms.

Forge — это **CLI tool category** (как Claude Code, Aider, Codex CLI, Gemini CLI).

### 6.2 Прямые конкуренты (4)

| Продукт | Что делает | Naming | Scale |
|---|---|---|---|
| **Claude Code** | Anthropic terminal agent. SWE-Bench Verified 80.9% (top depth). MCP integration. Skills system для SDD. | Brand + functional | $20+/mo, тop tier |
| **Aider** | Open-source git-native terminal agent. 70% of Aider's code written by Aider itself. 4M+ devs. Python. BYOK. | Functional ('aider' = helper) | Free, Apache 2.0 |
| **OpenHands (formerly OpenDevin)** | Open-source cloud agent platform. 72K+ GitHub stars. Multi-agent delegation, CodeAct architecture. 72%+ SWE-bench Verified. Docker sandbox. | Brand ('OpenHands' is hands metaphor) | Free self-hosted; $18.8M Series A |
| **Devin** | Cognition. Fully autonomous AI engineer — own cloud, browser, terminal, editor. 67% PR merge rate. ACU-based pricing. | Persona ('Devin') | $20+/mo, $2B valuation |

### 6.3 Функциональный inventory

Это **зрелая категория**, у standard агента 15+ функций:

1. **Issue → PR pipeline** (read issue, plan, code, test, commit, push, PR)
2. Multi-file refactor
3. Test execution + failure recovery
4. Code review (analyze diffs, suggest changes)
5. Migration scripts
6. Documentation generation
7. **Verification gates** (typecheck, lint, test before commit)
8. Plan mode (read-only planning before edits)
9. Worktree isolation (parallel work without conflicts)
10. **Checkpoint model** (stop-and-acknowledge gates)
11. **Cost tracking** (per-role budgets)
12. Linear / GitHub / Jira integration
13. Multi-project routing
14. MCP integration
15. **Parallel agents** (multiple simultaneous workers on different parts) — table stakes в Q1 2026
16. Cancel / status / resume operations
17. Sandbox execution (Docker / host)

### 6.4 Mapping: Forge (shipped) vs CLI competitors

| Function | Forge | Claude Code | Aider | OpenHands | Devin |
|---|---|---|---|---|---|
| Issue → PR pipeline | ✅ Linear-driven | ✅ Manual issue | ⚠ Manual | ✅ Issue-driven | ✅ Issue-driven |
| Plan mode | ✅ `forge plan` | ✅ Plan Mode | ⚠ Limited | ✅ Planning Mode (v1.6) | ✅ Built-in |
| Build with checkpoints | ✅ 5 gates | ⚠ Custom | ⚠ Manual | ✅ Sandbox | ✅ |
| Verification (typecheck/lint/test) | ✅ `Plan.verificationGates[]` | ⚠ Custom | ⚠ Manual | ✅ | ✅ |
| Worktree isolation | ✅ `~/.forge/worktrees/<key>` | ❌ | ❌ | ✅ Docker | ✅ Cloud |
| Cost tracking (per-role) | ✅ `state.costByRole` | ⚠ Aggregate | ⚠ Manual | ⚠ Limited | ⚠ ACU |
| Audit pipeline | ✅ `forge audit` per-module | ❌ | ❌ | ❌ | ❌ |
| **Spec coverage check** | ✅ `forge spec` (bidirectional code-vs-spec) | ❌ | ❌ | ❌ | ❌ |
| Code review (orchestrated) | ✅ `forge review --orchestrate` | ⚠ Single agent | ❌ | ⚠ Limited | ⚠ Limited |
| Linear estimation | ✅ `forge estimate` | ❌ | ❌ | ❌ | ❌ |
| Epic conductor (multi-issue) | ✅ `forge epic` | ❌ | ❌ | ❌ | ⚠ Limited |
| Programmatic API | ✅ `Forge` class | ⚠ Skills | ✅ Python module | ✅ SDK | ❌ |
| Multi-project routing | ✅ `--project=<slug>` | ⚠ Manual | ⚠ Manual | ⚠ Manual | ⚠ Manual |
| Parallel agents | ⚠ Single-issue lock | ✅ Agent Teams (Feb 2026) | ❌ | ✅ Multi-agent | ✅ Parallel sessions |
| MCP server | ✅ `mcp-server` command | ✅ | ⚠ Limited | ✅ | ⚠ Limited |

### 6.5 Verdict

**Forge — самый shipped и фичерный агент в оркестре**, и в нескольких функциональных областях **обходит публичных конкурентов**:

**Уникальные функции, которых у конкурентов нет в явном виде:**

1. **`forge audit <module>`** — параллельные agents (pm/architect/security/qa/seo/growth/perf/a11y) per module, aggregated через Opus verifier. Конкуренты делают audit как ad-hoc prompt; у Forge это formalized command с persistent output.
2. **`forge spec <module>`** — **bidirectional spec-vs-code coverage check**. Этого вообще нет в индустрии — Spec Kit и Kiro считают spec source-of-truth, но не проверяют code-vs-spec соответствие как explicit command.
3. **`forge estimate <issue-key>`** — Linear-bound Story Point / complexity / risk estimation. ChatPRD есть, но без Linear-native integration на этом уровне.
4. **`forge review --orchestrate`** — N parallel specialist agents + Opus verifier для PR review. Большинство конкурентов — single-agent review.
5. **Per-role model tiering** (architect = Opus, builder = Sonnet) — у Pavel описан как explicit per-role binding с fallback. Конкуренты обычно либо single-model, либо user-managed BYOM.
6. **`forge epic`** — epic conductor с топологическим dispatch sub-issues, merge-advance daemon, archive. Devin parallel sessions ближе всего, но без topo-sort + auto-close дисциплины.

**Где Forge отстаёт:**

1. **Parallel agents = table stakes.** В феврале 2026 все ключевые игроки запустили multi-agent в одну неделю. Forge — single-issue lock; nor multi-agent на одном issue. Это **главный gap** на 2026.
2. **Нет IDE integration.** Cursor / Windsurf / Antigravity дают visual feedback; Forge — pure CLI. Это OK для Pavel, но узкое позиционирование для potential adoption.
3. **MCP — есть, но ограниченный набор серверов.** OpenHands и Claude Code имеют broader MCP ecosystem.

**Recommendation для ТЗ:** Forge — **самая зрелая часть оркестра, можно использовать как proof point для продаж**. Audit + spec coverage check + estimate — features, которые **никто из конкурентов не сделал** и которые отвечают на реальную проблему (drift между specs и кодом, оценка Linear issues).

---

## 7. Сводная сравнительная таблица

| Агент | Industry sector | Industry naming pattern | Конкуренты (top 3) | Уникально у Pavel | Gaps |
|---|---|---|---|---|---|
| **Maestro** | Multi-agent orchestration | Generic ('supervisor', 'manager', 'orchestrator') | LangGraph Supervisor, CrewAI Manager, Magentic-One | Decision-laundering protection, FS rationale mirror, cost-as-decision | Substrate-pending, нет observability stack |
| **Muse** | Multi-mode AI substrate (rare) | Mixed (Roo Code = brand+functional) | Roo Code, GitHub Spec Kit, AWS Kiro, ChatPRD | One personality + 4 modes, project-bound canvas, OpenSpec methodology | Scout/Canvas deferred, нет Brand Voice IQ |
| **Atlas** | Deep research | Brand+functional ('Deep Research') | OpenAI DR, Perplexity DR, Gemini DR, Elicit | Beat-style continuous coverage, multi-domain (tech+region+society+culture), edge → Echo | Не запущен, citation discipline не описана |
| **Echo** | AI marketing/content | Persona ('Jasper') / brand | Jasper, Copy.ai, Writer | Studio-mode vs client-mode separation, always-under-review policy, Atlas-fed context | Самая weakly-specified, нет Brand Voice IQ, нет SEO/performance |
| **Pulse** | AI accounting / autonomous bookkeeping | Brand + functional ('AGL', 'Pilot AI') | Pilot AI Accountant, Digits AGL, Vic.ai, Docyt | Internal-CFO для studio, ledger anchors agent decisions, `_finance/` in-repo | Не запущен, compliance layer не описан, scope ambiguity (internal vs client-product) |
| **Forge** | AI coding agents (CLI category) | Mixed | Claude Code, Aider, OpenHands, Devin | `audit / spec / review / estimate / epic` commands — нигде нет, per-role model tiering, bidirectional spec coverage | Single-issue lock vs Feb 2026 parallel-agents trend, нет IDE surface |

---

## 8. Cross-cutting findings

### 8.1 Naming conventions

**Что Pavel сделал хорошо:**

- **Тройной слой (Cast / Persona / Functional)** — это **рабочий compromise** между разными конвенциями. Public surface использует functional descriptors, что Salesforce и B2B-индустрия рекомендуют. Persona names live in spec / studio surface, что даёт identity без брендового перегруза.
- **Один-два слога, под 10 символов** (Maestro / Muse / Atlas / Echo / Pulse / Forge) — точно по Salesforce design guideline.
- **Mythological + musical reference (Muse, Maestro)** работает: persona without anthropomorphism, что снижает gender-bias риск.

**Что стоит обсудить:**

- **Cast layer (Brian Jones / Kurt Cobain / etc.) — внутренний.** OK для studio, но может **создавать confusion** при scaling команды. Если новый partner спрашивает «who's Pigpen», нужен onboarding overhead. У Pixar codenames живут, но Pixar — закрытая компания. Rapoport может расти.
- **Public functional descriptors** (`Conductor / Architect / Builder / Research / Voice / Finance` per cast-naming-role-descriptors change) — **читаются как роли реальных людей**. Это **двусмысленно**: если Pavel говорит клиенту «Architect drafts the spec», клиент может ожидать человека-архитектора. Стоит уточнить позиционирование (это agent? human? hybrid?).

### 8.2 Function patterns Pavel не учитывает

Из конкурентного inventory **отсутствуют в spec / не определены**:

| Функция | Где она у конкурентов | Применимо к Pavel? |
|---|---|---|
| **Observability stack** (tracing, metrics dashboard) | LangSmith, Langfuse, OpenTelemetry GenAI | Да — для Maestro и Forge как минимум |
| **Brand Voice IQ analog** | Jasper IQ, Writer Knowledge Graph | Да — для Echo и Canvas mode (v2) |
| **Performance prediction** | Anyword, Frase | Возможно — для Echo client-mode |
| **SOC 2 compliance** | Pilot, Digits, Vic.ai | Да — для Pulse при client-facing использовании |
| **Visual diff / approval UI** | Cursor, Windsurf, Devin Spaces | Возможно — для Maestro decision-surface |
| **Citation / source-quality grading** | Perplexity, OpenAI Deep Research | Да — для Atlas |
| **Plan-mode autonomy levels** | Devin parallel sessions, Claude Code Agent Teams | Возможно — для Forge при scaling |

### 8.3 Уникальная позиция Rapoport

**То, что я НЕ нашёл у конкурентов:**

1. **Целостный orchestra с philosophy layer (Manifesto + Cast + Charter).** Все конкуренты — либо single-purpose продукты (Pilot, Jasper, Devin), либо frameworks (LangGraph, CrewAI). **Operating philosophy + cast lineage + studio charter — нет аналогов.**
2. **OpenSpec как методологический фундамент.** GitHub Spec Kit и AWS Kiro близко, но они tools, не методология studio operations. У Pavel — OpenSpec applies к specs, decisions, change folders, archive — это **operating system, не feature**.
3. **Журнал между агентами как append-only audit trail.** `orchestra_journal` + `originating_agent_id` — это **distributed-systems thinking** в AI orchestration, который индустрия только начинает frame'ить (Hermes Agent profile system делает похожее, но менее формально).
4. **Spec-driven AI agents = operating model.** Pavel применяет SDD не только для кода (как все 2026 SDD tools), а **для всего, что agents делают**. Echo drafts under review = spec'ed. Pulse decisions = ledger anchored = spec'ed. Это **расширение SDD за пределы coding**, что я не нашёл у других студий.

### 8.4 Главные стратегические вопросы

После research стало видно три открытых стратегических вопроса:

1. **Echo: defensive policy vs differentiated feature.** «Always under review» — это политика, не feature. Что Echo **делает**, чтобы быть лучше Jasper? Без ответа — Echo будет looking-like-anyone-else AI writer.
2. **Pulse: internal CFO vs client-product.** Решение определяет 12 месяцев работы (компетенция в bank-feeds / GL / SOC 2). Сейчас spec неоднозначен.
3. **Atlas: beats vs on-demand.** Если beats — какие именно? Если они описаны, Atlas становится **journalistic engine** с уникальным positioning. Если не описаны, Atlas скатится в «yet-another-Deep-Research».

---

## 9. Что делать с этими findings — план следующего шага

### 9.1 Готовность к ТЗ

После research я могу написать **техническое предложение** в одном из двух форматов:

**Вариант A — Per-agent тех-документация (compact, 1 page per agent):**
- Identity / role / runtime
- Function surface (что делает)
- Inputs / outputs
- Integration points
- Risk matrix
- Cross-cutting defenses
- Status / phase

Это формат уже существующих specs у Pavel (`agents/maestro.md` имеет 7 sections). Можно сгенерировать аналог для всех шести агентов как пакет «технического предложения», который читается клиенту или новому partner.

**Вариант B — Single «orchestra capability sheet» (presentation-style):**
- Что такое оркестр
- Шесть агентов + одна страница на каждого
- Как они общаются между собой
- Когда что активируется
- ROI / business value
- Comparison snapshot

Это формат для outreach / pitch, не для engineer onboarding.

### 9.2 Открытые вопросы

Перед написанием ТЗ нужно решить:

| # | Вопрос | Влияние |
|---|---|---|
| Q1 | Echo: studio-mode и client-mode уже определены, но какие конкретные features? Brand voice modeling, SEO, performance prediction — что включаем? | Определяет 6–12 месяцев работы |
| Q2 | Pulse: internal CFO для студии, или будущий client-facing product? | Compliance scope, integration scope |
| Q3 | Atlas: какие конкретные beats (tech / region / society / cinema / philosophy) запускаются первыми? | Storage layout в `_research/world/` |
| Q4 | Cast layer — остаётся internal-only или появится в маркетинге? | Brand strategy |
| Q5 | Public functional descriptors (Conductor / Architect / etc.) — agents or humans? | Sales positioning |
| Q6 | Parallel agents (per Q1 2026 trend) — приоритет на 2026, или wait-and-see? | Forge roadmap |

---

## 10. Источники research

**Multi-agent orchestration:**
- DataCamp: CrewAI vs LangGraph vs AutoGen (2025–2026)
- Gurusup: Best Multi-Agent Frameworks 2026
- Langfuse: AI Agent Comparison (2025)
- LangChain Docs: Supervisor pattern, personal-assistant subagents
- Microsoft Azure Architecture Center: AI Agent Orchestration Patterns
- ProductSchool: AI Agent Orchestration Patterns
- Redis: AI Agent Architecture Patterns
- LushBinary: Multi-Agent AI Orchestration Patterns

**AI coding agents:**
- MorphLLM: «We Tested 15 AI Coding Agents (2026)»
- ArtificialAnalysis: Coding Agents Comparison table
- ToolHalla: Devin vs OpenHands vs SWE-agent 2026
- LocalAIMaster: OpenHands vs SWE-Agent
- MarkTechPost: Best AI Agents for Software Development Ranked (May 2026)
- Vellum: 10 Best AI Coding Agents 2026
- MightyBot: Best AI Coding Agents 2026
- Frontman: Best Open-Source AI Coding Tools 2026
- OpenSourceAlternatives: 9 Best Open Source AI Coding Assistants
- SimilarLabs: 12 Best Claude AI Alternatives

**Deep research:**
- Aaron Tay's Musings: Rise of Agent-Based Deep Research
- AnalyticsVidhya: Perplexity Deep Research vs OpenAI vs Gemini
- AryabhConsulting: Deep Research AI Tools Comparison 2025
- TheAIRankings: Best AI for Research 2025
- Pinggy: Top 10 AI Models for Scientific Research 2026

**Marketing / content agents:**
- Copy.ai: 7 Best AI Content Generators
- DeeperInsights: Jasper AI Review 2025
- eesel AI: Jasper AI review
- AgentWiki: Jasper AI Knowledge Base
- DigitalApplied: Agentic Content Tools 2026 Matrix
- Jasper docs

**AI accounting agents:**
- AIMultiple: Top 14 Accounting AI Agents
- DesignRush: 5 Best Accounting AI Agents 2026
- SaaSrat: Best AI Accounting Software 2026
- DualEntry: Best AI Accounting Software 2026
- AgentMelt: AI Agents for Accounting Firms
- AccountingToday: Pilot launches fully autonomous AI bookkeeper
- RelayFi: 9 Best AI Tools for Accounting 2025
- Pilot.com pricing page

**Naming & SDD:**
- Park Rangers Capital: Hidden Gender Rules of AI Agents
- Salesforce: Naming an AI Agent
- ShapeOfAI: Name patterns
- AddyOsmani / O'Reilly: How to write a good spec for AI agents
- Thoughtworks: Spec-driven development
- Augment Code: What is Spec-Driven Development
- BCMS: SDD Definitive 2026 Guide
- TowardsDataScience: From Vibe Coding to Spec-Driven Development
- DeepLearning.AI: Spec-Driven Development with Coding Agents (course, late 2025)

---

## Triage

- **Feeds into**: `openspec/current/platform.md` (orchestra positioning) and per-agent specs under `openspec/current/agents/` (Maestro, Muse, Atlas, Echo, Pulse, Forge) once Q1–Q6 are resolved.
- **Status**: open — six strategic questions (Q1–Q6 in § 9.2) require founder ratification before findings can be applied.
- **Decision or finding**: Orchestra has no direct competitor as a unit; differentiation lives in (a) Forge's audit/spec/estimate command surface, (b) Muse's single-personality multi-mode design, (c) Maestro's decision-laundering protection + dual-sink journal, and (d) OpenSpec as operating-model methodology (not just SDD tooling). Echo and Pulse are under-specified relative to mature competitors and need feature-scope decisions before launch. February 2026 industry pivot to parallel-agents makes Forge's single-issue lock the most time-sensitive gap.
- **Applied**: pending founder ratification of Q1–Q6.
