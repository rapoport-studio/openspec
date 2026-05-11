# Node.js / TypeScript AI agent frameworks (2026 landscape)

> **Status:** complete (snapshot, 2026-05-11)
> **Owner:** Pavel Rapoport (founder).
> **Authors:** Pavel Rapoport <pavel@rapoport.studio> (founder, primary); Claude Opus 4.7 (Anthropic) — broad-map research across Mastra/LangGraph/Vercel AI SDK/Inngest AgentKit/Cloudflare Agents/Claude Agent SDK/VoltAgent/ElizaOS/Mem0/Letta/Cognee/Braintrust/Langfuse/MCP/Skills standards/Stagehand/etc., production-user verification + last-commit checks, ~39 search queries.
> **Method:** Single-pass — for each framework: GitHub stars + npm downloads + last commit + named production users + Anthropic Claude integration + strengths/weaknesses + when to use. Cross-referenced 2026 AI Engineer Summit talks, Anthropic engineering blog, Cloudflare Agents Week 2026 announcements, Mastra Series A coverage. Mapped recommendations to Rapoport Forge/Muse stack.
> **Reusable for:** `upgrade-forge-to-modern-ai-stack` (Epic 5) — direct input to Forge migration design (Vercel AI SDK v6 + Skills + MCP server). Also informs Epic 7 (evals + memory). Authorship: open.
> **Opened:** 2026-05-11
> **Feeds into:** Epic 5 (forge modern stack); Epic 7 (evals + memory).

---

## TL;DR

Landscape consolidated to 4 archetypes by 2026: (1) **Batteries-included** (Mastra, VoltAgent), (2) **Low-level graph orchestration** (LangGraph.js), (3) **Durable runtime** (Inngest AgentKit, Trigger.dev v3, Cloudflare Agents), (4) **Composable primitives** (Vercel AI SDK v6, Claude Agent SDK). Plus 3 stable standards: MCP v1.0 (78% enterprise AI use), Agent Skills (Anthropic Dec 2025, adopted OpenAI/Google/Cursor in weeks), OpenTelemetry GenAI semantic conventions.

For Forge specifically: migrate inference from `@anthropic-ai/sdk` to **Vercel AI SDK v6** (1.3M weekly downloads on `@ai-sdk/anthropic`), implement **Agent Skills pattern** for institutional memory between builds, **publish Forge as MCP server** so Claude Code/Cursor/IDE can dispatch builds. Defer Mastra (too opinionated for existing orchestrator), LangGraph (complexity tax), vector embeddings (premature for <500 files), Letta/Mem0 Pro (overkill for known structured state).

---

## Big picture — 2026 archetypes

| Archetype | Key player | Best fit |
|---|---|---|
| **Batteries-included** | Mastra ($35M raised, 22K stars, Replit/PayPal/Adobe) | Greenfield TS agent project |
| **Low-level graph** | LangGraph.js (Uber/LinkedIn/Klarna) | Complex multi-step workflow with branching |
| **Durable runtime** | Cloudflare Agents (Project Think Apr 2026), Inngest AgentKit, Trigger.dev v3 | Long-running agents, fault tolerance |
| **Composable primitives** | Vercel AI SDK v6 (1.3M wkly), Claude Agent SDK | Custom orchestrator (like Forge) |

Three stable standards: **MCP v1.0**, **Agent Skills**, **OpenTelemetry GenAI semantic conventions**. Trend of year: **context engineering** replacing prompt engineering.

## Top-7 frameworks deep-dive

### Mastra (mastra.ai) — DX-first favorite
22K+ stars, 1.8M monthly downloads, $35M raised (Series A Oct 2025). Production: PayPal, Adobe, Elastic, Docker, **Replit Agent 3**, Marsh McLennan, SoftBank, Factorial. Bundles agents + workflows + memory + evals + RAG. Anthropic first-class via `@ai-sdk/anthropic` (uses Vercel AI SDK underneath). TS-only.

When to use: greenfield TS agent. When NOT: existing orchestrator with custom logic (opinion lock-in).

### LangGraph.js — production-tier graph
42K+ weekly TS downloads, 438K+ on SDK package. Production: Replit, Uber, LinkedIn, GitLab, Klarna, Elastic. Low-level state machine — nodes/edges, checkpointing, time-travel debugging. Pairs with LangSmith.

Documented complexity tax: simple ReAct = 120 lines vs 40 in Smolagents. When to use: complex branching workflow + LangChain expertise. When NOT: simple agent loop.

### Vercel AI SDK v6 — fundamental primitives
**1,295,727 weekly downloads on `@ai-sdk/anthropic`** — most-used Anthropic provider in TS. Not framework, inference primitive layer. v6 added: built-in agent loop with `stopWhen: stepCountIs(20)` (default 20 steps), `prepareStep`, tool execution approval gates (`Tool.needsApproval`), OAuth for MCP clients via `@ai-sdk/mcp`.

Anthropic-specific features (extended thinking, prompt caching) first-class. Used inside Mastra/Vercel/Cloudflare Agents under the hood.

When to use: custom orchestrator (like Forge) wanting full agent loop control without writing Anthropic SDK calls by hand.

### Claude Agent SDK (Anthropic) — Claude Code engine
`@anthropic-ai/claude-agent-sdk`. **Same runtime as Claude Code** — same tool implementations, permission model, agent loop, but as TS library. Custom tools registered as in-process MCP servers. V2 preview (2026) replaced async generators with `send()`/`stream()` cycles.

Tight Anthropic coupling — plus for Anthropic-first users, minus for multi-provider. Strong candidate for parts of Forge runtime.

### Inngest AgentKit — durable agents
`@inngest/agent-kit`, Inngest sponsorship (well-funded). Two-layer: AgentKit for agents/networks (typed state, handoff), Inngest = durable runtime. `step.ai` auto-retries on failure and caches result.

When to use: long-running agents with flaky external APIs (RAP-117 style). When NOT: simple sync agent.

### Cloudflare Agents SDK — native CF stateful
"Powering thousands of production agents" (Cloudflare blog), Agents Week 2026 (April) added massive infra. Each agent = Durable Object with SQL DB, WebSocket, scheduling. **Project Think** (Apr 2026): durable execution with fibers (crash recovery, checkpointing), sub-agents (typed RPC), persistent sessions (tree-structured forking, compaction, full-text search), sandboxed code execution.

Critically aligns with Rapoport stack (already on CF Workers via @opennextjs/cloudflare).

### VoltAgent — challenger
1,500+ GitHub stars (significantly less than Mastra). `@voltagent/core` framework + VoltOps Console. Pre-Mastra-tier by adoption. Backup option only.

## Memory / Evals / Indexing — short overviews

**Memory:** Mem0 (fact-extraction, three-tier, SOC2/HIPAA), Zep (temporal knowledge graph via Graphiti, **63.8% vs Mem0's 49% on LongMemEval**), Letta (formerly MemGPT, OS-inspired), Cognee (knowledge graph, $7.5M seed, 70+ companies). For Forge institutional memory: **pgvector + custom indexing in Supabase** (we have it; `state.json` already structured), NOT Mem0 Pro ($249/mo).

**Evals:** Braintrust (CI/CD eval blocking, 1M trace spans/month free), **Langfuse** (MIT-licensed, OpenTelemetry-native, ClickHouse-acquired Jan 2026), Helicone (Apache 2.0 proxy), Phoenix (Arize, OS), Promptfoo (**OpenAI acquired $86M Mar 2026** — uncertain future). For Forge: **Langfuse self-hosted** (no lock-in).

**MCP:** 500+ public servers by 2026, registry grew 7.8× year-over-year, 78% enterprise AI teams have at least one MCP-backed agent in production. All frontier labs ship clients (Anthropic, OpenAI, Google DeepMind, Microsoft, Salesforce, Block, Cloudflare, Replit). Smithery (7300+ tools) moving to paid hosting March 2026.

**Durable execution:** Inngest (event-driven, 100M+ daily executions), Temporal (gigantic, polyglot, **known issue: workflow history saturation from large LLM payloads**), Trigger.dev v3 (built specifically for AI workflows). For Forge: Cloudflare Agents + Project Think native — no second vendor.

## Patterns — "agents become smarter"

Seven production patterns (this is **the most important** section for Pavel):

1. **Reflection / Reflexion loop.** Generate → critique → refine. Reflexion adds persistent reflection memory — failures saved as verbal reinforcement. Raised HumanEval from 80% (GPT-4) to 91%. **Returns diminish after 2-3 iterations.** Forge verifier loop already partially implements this.

2. **Progressive disclosure via Skills.** Anthropic Skills standard (Dec 2025) — folder with `SKILL.md` (name + description = ~80 tokens discovery cost), expanded when relevant. **17 official skills total ~1700 tokens discovery** — more skills than context per activated one. Adopted by OpenAI/Google/GitHub/Cursor in weeks. **This is the pattern for Forge knowledge accumulation** — each build generates "lesson learned" → new skill → next build sees it in TOC.

3. **Self-improving memory loop (AutoResearch pattern).** Scheduled process review → extract patterns → curate memories. Agents not just complete tasks but report what they learned.

4. **Ralph Loop / verifier-driven retry.** Pattern from Geoffrey Huntley (Aug 2025): agent says "done" → automatic verify → if not passed, restart with error context → loop until verified. Forge verifier gate already approximately this — but without full automation.

5. **Coordinator-Implementor-Verifier multi-agent.** Cursor 3 (Apr 2026): Planners explore + decompose, Workers execute independently, Judge agents evaluate cycle output. **Forge architecture (architect → planner → builder → verifier) — canonical variant of this pattern.**

6. **Knowledge accumulation through institutional memory.** Cursor 2.4 (Jan 2026) added subagents + skills marketplace + "Cursor Blame" linking code to issues/PRs. MemU framework — agentic memory across harnesses via hooks. **What Pavel calls "agents grow smarter" = institutional memory + progressive disclosure combined.**

7. **DSPy + BAML schema optimization.** DSPy compiles prompts (does not write), GEPA optimizer + BAML adapter gave 20+ percentage points on extraction tasks. Only if structured-output-heavy pipeline.

## Recommendations for Rapoport Studio

### Now (next sprint)

1. **Adopt Vercel AI SDK v6 as Forge inference layer.** Migrate from direct `@anthropic-ai/sdk`. Gives: native MCP, tool approval gates, cleaner agent loop control, free provider switch.
2. **Implement Agent Skills pattern in Forge.** Each "lesson learned" → `~/.forge/skills/<name>/SKILL.md`. Progressive disclosure via description-only first. **Direct implementation of "agents grow smarter" without new dependency.**
3. **Publish Forge as MCP server.** Lets Claude Code / Cursor / IDE invoke Forge build commands. Already aligns with internal architecture (custom MCP tool registration via Claude Agent SDK).

### 3–6 months out

4. **Add Langfuse self-hosted for evals + observability.** OpenTelemetry GenAI semantic conventions stable, vendor-neutral. Self-hosted = no lock-in.
5. **Memory layer = pgvector in Supabase + custom indexing.** Supabase already in stack. **Do NOT buy Mem0 Pro** ($249/mo); not needed Letta server. Forge `state.json` already structured — index it + add embedding-based retrieval.

### When durable Forge runtime is needed (potentially 6+ months out)

6. **Cloudflare Agents SDK + Project Think.** If Forge moves from local CLI to server-side (e.g., Linear webhook → automated build), this is native fit for CF stack. Durable Objects + fibers give checkpoint-resume without Temporal/Inngest overhead.

### Do NOT take

| Skip | Reason |
|---|---|
| LangGraph.js for Forge | Complexity tax not justified. Simple ReAct = 120 lines vs 40 elsewhere |
| Mastra for Forge | Opinionated framework; Forge works. **For Muse — consider** (bundled memory+evals can speed prototype) |
| Temporal / Inngest | Cloudflare Durable Objects cover, no second vendor |
| OpenRouter / multi-provider gateway | Anthropic stable; complexity without current ROI. Reconsider on $10K+/mo spend |
| DSPy / BAML | Not TS-native; mental model jump huge. Good for structured extraction, not code-gen |
| Letta / Mem0 Pro | Overkill for Forge size. Custom pgvector + state.json index works better for known use case |
| Promptfoo | OpenAI acquired $86M Mar 2026; future as independent tool uncertain |

### What custom code already does better (don't rewrite)

- Forge architect → planner → builder → verifier loop — canonical multi-agent pattern, do not migrate
- Forge verifier gate — almost Ralph Loop-style; supplement only with: automated retry with error context
- OpenSpec change proposals — **our version of institutional memory**, structured better than generic memory layers (Mem0/Letta)

## 7 principles for choosing a framework

1. Bundle vs primitives — choice of philosophy before choice of framework.
2. Vendor-neutral standards beat proprietary (MCP, Agent Skills, OpenTelemetry GenAI).
3. Durable execution = property of runtime, not separate concept.
4. Memory ≠ vector DB. Three-tier (working/session/archival) is what distinguishes Mem0/Letta/Zep from plain pgvector.
5. Evals — first-class, not afterthought.
6. Progressive disclosure — load-bearing skill 2026.
7. **Production users by name > GitHub stars.** Mastra with Replit Agent 3 / PayPal / Adobe = different signal class than Mastra with 22K stars.

---

## Sources

- [Mastra GitHub](https://github.com/mastra-ai/mastra)
- [Mastra Complete Guide 2026](https://www.generative.inc/mastra-ai-the-complete-guide-to-the-typescript-agent-framework-2026)
- [Mastra Factorial Case Study](https://mastra.ai/blog/factorial-case-study)
- [Mastra Series A](https://mastra.ai/blog/series-a)
- [LangGraph GitHub](https://github.com/langchain-ai/langgraph)
- [LangGraph 2.0 production guide 2026](https://dev.to/richard_dillon_b9c238186e/langgraph-20-the-definitive-guide-to-building-production-grade-ai-agents-in-2026-4j2b)
- [Inngest AgentKit overview](https://agentkit.inngest.com/overview)
- [Inngest vs Temporal](https://www.inngest.com/compare-to-temporal)
- [Trigger.dev product page](https://trigger.dev/product/ai-agents)
- [Cloudflare Agents docs](https://developers.cloudflare.com/agents/)
- [Cloudflare Project Think](https://blog.cloudflare.com/project-think/)
- [Cloudflare Agents Week 2026](https://blog.cloudflare.com/agents-week-in-review/)
- [VoltAgent GitHub](https://github.com/VoltAgent/voltagent)
- [Vercel AI SDK GitHub](https://github.com/vercel/ai)
- [Vercel AI SDK 6 release](https://vercel.com/blog/ai-sdk-6)
- [AI SDK loop control docs](https://ai-sdk.dev/docs/agents/loop-control)
- [@ai-sdk/anthropic on npm](https://www.npmjs.com/package/@ai-sdk/anthropic)
- [Claude Agent SDK TypeScript GitHub](https://github.com/anthropics/claude-agent-sdk-typescript)
- [Claude Agent SDK V2 preview docs](https://platform.claude.com/docs/en/agent-sdk/typescript-v2-preview)
- [Mem0 vs Zep vs Letta benchmark](https://dev.to/varun_pratapbhardwaj_b13/5-ai-agent-memory-systems-compared-mem0-zep-letta-supermemory-superlocalmemory-2026-benchmark-59p3)
- [Cognee site](https://www.cognee.ai/)
- [Braintrust observability buyer guide 2026](https://www.braintrust.dev/articles/best-ai-observability-tools-2026)
- [Langfuse alternatives 2026](https://www.braintrust.dev/articles/langfuse-alternatives-2026)
- [Top 5 eval tools after Promptfoo acquisition](https://dev.to/thedailyagent/top-5-ai-agent-eval-tools-after-promptfoos-exit-576i)
- [MCP 2026 roadmap](https://blog.modelcontextprotocol.io/posts/2026-mcp-roadmap/)
- [Smithery hosting update](https://smithery.ai/blog/updates-to-our-hosting-plan)
- [Anthropic Agent Skills engineering blog](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
- [Anthropic context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Progressive disclosure as system design](https://www.newsletter.swirlai.com/p/agent-skills-progressive-disclosure)
- [Ralph Loop coding 2026](https://www.leanware.co/insights/ralph-wiggum-ai-coding)
- [Self-improving AI agents 2026 guide](https://o-mega.ai/articles/self-improving-ai-agents-the-2026-guide)
- [Cursor 3 InfoQ](https://www.infoq.com/news/2026/04/cursor-3-agent-first-interface/)
- [Cursor 2.4 subagents skills MemU](https://memu.pro/blog/cursor-2-4-subagents-skills-memory)
- [pgvector production 2026](https://pecollective.com/tools/pgvector/)
- [LlamaIndex.ts framework docs](https://developers.llamaindex.ai/typescript/framework/)
- [OpenTelemetry GenAI semantic conventions](https://opentelemetry.io/blog/2025/ai-agent-observability/)
