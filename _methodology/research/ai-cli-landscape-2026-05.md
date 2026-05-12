# AI CLI & Spec-Driven Dev Landscape — May 2026

> **Status:** canonical landscape document, intended 6-month shelf life. Source-cited throughout; every concrete claim (pricing, feature, metric) carries a URL. Items I could not verify are flagged `[unverified]`.
> **Owner:** Pavel (founder).
> **Opened:** 2026-05-11.
> **Author:** drafted with Claude Opus 4.7 (Anthropic) on behalf of Pavel. Method: gap-driven web research after reading existing repo files (see § 2), 24 WebSearch queries, primary-source verification on pricing pages where possible.
> **Reusable for:** strategic decisions on Forge/Muse/Maestro positioning; pricing conversations; PR/marketing copy about the three-CLI orchestra; competitor briefs for client conversations.
> **Companion to:** [`competitive-positioning-sdd-frameworks.md`](./competitive-positioning-sdd-frameworks.md) (SDD-tool comparison, opened 2026-05-07), [`openspec-vs-speckit-public.md`](./openspec-vs-speckit-public.md) (empirical OpenSpec/Spec Kit benchmark, 2026-05-10), [`competitor-landscape-synthesis.md`](./competitor-landscape-synthesis.md) (regional MD/RO/IL/EU competitors, 2026-05-10).
> **Re-open trigger:** any of (a) major model release that changes pricing math, (b) Devin/Factory price drop ≥30%, (c) MCP/A2A/AGNTCY consolidation event, (d) Tessl GA launch, (e) Forge open-sourcing decision (pulls this from research → positioning).

---

## TL;DR (5 bullets)

1. **The "AI coding CLI" category has collapsed to table stakes.** As of May 2026, Claude Code, Codex CLI, Cline, Aider, OpenCode, Plandex, Factory droids, Cursor CLI all share the same baseline feature set: terminal-native, MCP-capable, multi-model, agent-runs-shell, parallel subagents, git-aware. The differentiator is no longer "we have a CLI"; it's **what discipline the CLI is enforcing** ([Tembo, 2026](https://www.tembo.io/blog/coding-cli-tools-comparison); [DEV, 2026](https://dev.to/soulentheo/every-ai-coding-cli-in-2026-the-complete-map-30-tools-compared-4gob)).
2. **The unit of value is converging on "engineer-hours-of-autonomous-work."** Devin sells ACUs (1 ACU ≈ 15 min autonomous work, ~$8–9/hr) ([devin.ai/pricing](https://devin.ai/pricing/)); Augment Intent sells credits routed to coordinator/implementor/verifier ([Augment, 2026](https://www.augmentcode.com/guides/intent-pricing)); Replit and Bolt sell tokens per build. The unit *implies the metaphor*: ACU → human-engineer-substitute; credits → labor pool; tokens → compute pass-through. Each metaphor has different procurement and pricing leverage.
3. **Multi-agent orchestration shipped industry-wide in February 2026 in a two-week window.** Grok Build (8 parallel agents), Windsurf (5), Claude Code (Agent Teams behind a feature flag), Codex CLI (OpenAI Agents SDK integration), Devin (fleet of Devins) all landed simultaneously ([Cognition blog](https://cognition.ai/blog/devin-2); [Claude Code docs](https://code.claude.com/docs/en/agent-teams)). "Run N agents in parallel" is no longer a differentiator — *coordination of N agents under a shared spec* is the next frontier.
4. **Spec-driven development is the dominant 2026 narrative, but "spec" means three incompatible things.** (a) Spec-first / phase-gated (Spec Kit, Kiro, BMAD); (b) spec-anchored / change-folder (OpenSpec, Augment Intent's "living spec"); (c) spec-as-source-code (Tessl — "by end of 2027 developers won't look at code most of the time") ([Tessl blog](https://tessl.io/blog/how-tessls-products-pioneer-spec-driven-development/); [Augment, 2026](https://www.augmentcode.com/tools/best-spec-driven-development-tools)). These three are NOT interchangeable; the choice is a philosophy commitment.
5. **Nobody has shipped spec-quality measurement yet.** No public tool lints specs for clarity, completeness, churn-velocity, or fidelity-to-code as a first-class product. LLM-as-judge approaches exist in research (AI-SQE workshop at ICSE 2026 — [conf.researchr.org](https://conf.researchr.org/home/icse-2026/ai-sqe-2026)), but no shipping linter at the layer where Rapoport's `forge audit` operates. **This is the single biggest underclaimed gap in the landscape as of May 2026.**

---

## 2. What was already known (cross-refs to existing repo files)

Before this file, the repo already documented:

| Topic | File | What it covers |
|---|---|---|
| **SDD framework landscape** | [`competitive-positioning-sdd-frameworks.md`](./competitive-positioning-sdd-frameworks.md) | Spec Kit, OpenSpec, Kiro, BMAD, Augment Intent, Cursor/Claude Code, plus Thoughtworks/37signals/BCG-style consultancies. User-quote-grounded. |
| **OpenSpec vs Spec Kit empirical** | [`openspec-vs-speckit-public.md`](./openspec-vs-speckit-public.md) | First-party benchmark (install timing, token cost, agent context tax), 14-category failure-mode catalogue, GitHub health-metrics audit. |
| **Regional competitor map** | [`competitor-landscape-{moldova,romania,israel,europe,synthesis}.md`](./competitor-landscape-synthesis.md) | Buyer + competitor maps in MD/RO/IL/EU. ICP locked at Muse-first founder-track. |
| **OSS licensing posture** | [`open-source-positioning.md`](./open-source-positioning.md) | FSF / OSI / source-available / Fair Source / Ethical Source spectrum. |
| **Value attribution** | [`value-attribution.md`](./value-attribution.md) | What dimensions of human contribution to measure in AI-augmented work. |

**This file's contribution** — gaps not covered above:

- **AI coding CLIs as a category** (Claude Code, Aider, Cursor CLI, Codex CLI, Cline, Plandex, OpenCode, Continue, Factory droids) — feature/price comparison.
- **SaaS autonomous builders** (Devin, Factory.ai, Codegen, Sweep, All Hands/OpenHands, Replit Agent, Lovable, v0, Bolt.new) — depth on pricing units and threat model.
- **Tessl deep-dive** — only mentioned in passing before.
- **Anthropic Skills as a meta-layer** — not covered.
- **Cursor rules / Cline rules / AGENTS.md standard** — not covered.
- **Spec quality / spec-linting tools** — not covered (gap, see § 6).
- **Multi-agent orchestration as a CLI feature** — only Devin team mode mentioned; Feb 2026 cluster missed.
- **Agent observability** (LangSmith, Langfuse, Helicone, Braintrust) — not covered.
- **A2A / AGNTCY / MCP-Apps interop standards** — not covered.

---

## 3. Category 1: AI coding CLIs

The terminal-native, file-system-touching, shell-running CLI category. Twelve tools matter as of May 2026; eleven share a converging baseline.

### 3.1 Comparison table

| Tool | Pricing (entry) | Pricing (heavy) | Models | Open source | Differentiator | Source |
|---|---|---|---|---|---|---|
| **Claude Code** | Pro $20/mo | Max 20x $200/mo or API | Claude Opus 4.6/4.7, Sonnet 4.6, Haiku 4.5 only | No (Anthropic) | 1M token context, Agent Teams beta, leads SWE-bench Verified at 80.9% (Opus 4.6); 5.5× fewer tokens than Cursor | [Anthropic](https://www.anthropic.com/news/higher-limits-spacex), [Requesty](https://www.requesty.ai/blog/agentic-coding-tools-compared-2026-claude-code-cursor-codex-aider) |
| **Codex CLI** (OpenAI) | Bundled w/ ChatGPT | API pay-per-token | GPT-5, GPT-5-mini | Apache-2.0 (CLI) | Leads Terminal-Bench 2.0 at 77.3%; auth via ChatGPT sub; OpenAI Agents SDK integration | [Builder.io](https://www.builder.io/blog/codex-vs-claude-code), [Blakecrosley](https://blakecrosley.com/blog/codex-vs-claude-code-2026) |
| **Aider** | Free (BYOK) | $30–$200/mo API | Any (Anthropic, OpenAI, DeepSeek, local) | Apache-2.0 | Oldest in category; auto-commits with semantic messages; **4.2× fewer tokens than Claude Code on same task** | [Morph LLM](https://www.morphllm.com/comparisons/morph-vs-aider-diff), [aiagentslist](https://aiagentslist.com/agents/aider) |
| **Cline** | Free | BYOK | Any | Apache-2.0 | 5M+ installs across VS Code/Cursor/JetBrains/Zed/Neovim; Feb 2026 native subagents + CLI 2.0 headless CI/CD | [Requesty](https://www.requesty.ai/blog/agentic-coding-tools-compared-2026-claude-code-cursor-codex-aider) |
| **Cursor CLI** | Pro $20/mo | Pro+ $200/mo | Any (Cursor routes) | No | IDE-pair; Cursor 3 shipped agent capabilities; high token consumption | [Requesty](https://www.requesty.ai/blog/agentic-coding-tools-compared-2026-claude-code-cursor-codex-aider) |
| **OpenCode** (sst/opencode) | Free (BYOK) | API only | 75+ providers | MIT | Go single binary, no runtime deps; built-in plan mode; MCP support | [Terminal Trove](https://terminaltrove.com/compare/ai-coding-agents/), [Plandex repo](https://github.com/plandex-ai/plandex) |
| **Plandex** | Free OSS / Cloud $45/mo | $45/mo + $20 credits | OpenAI, Anthropic, Google | MIT (OSS), commercial cloud tier | 2M token effective context; cumulative diff sandbox; designed for "large tasks" | [Plandex](https://plandex.ai/), [tooldirectory](https://tooldirectory.ai/tools/plandex) |
| **Factory CLI / Droids** | $20/mo | $200/mo Max, Teams up to 150 seats | Open-weight "Droid Core" + frontier | No | Editor-agnostic Droids (VS Code, JetBrains, Vim, terminal); $150M Series C April 2026; "agent-native development" framing | [Factory pricing](https://factory.ai/pricing), [docs.factory.ai](https://docs.factory.ai/pricing), [tech-insider](https://tech-insider.org/factory-ai-150-million-series-c-khosla-coding-droids-2026/) |
| **OpenHands** (All Hands AI) | Free (self-host) or cloud | Cloud paid tier | Any (OpenRouter, local) | MIT | v1.6.0 March 2026 with Kubernetes + Planning Mode beta; SDK lets you scale to 1000s of agents | [openhands.dev](https://www.openhands.dev/), [vibecoding](https://vibecoding.app/blog/openhands-review) |
| **Continue.dev** | Free / Enterprise | Custom | Any | Apache-2.0 | IDE-extension primary, CLI secondary; enterprise focus | [Pinggy](https://pinggy.io/blog/top_cli_based_ai_coding_agents/) [unverified — 2026 details thin] |
| **Gemini CLI** | Free quota | API | Gemini 2.5/3 | Apache-2.0 | Google's official CLI, AGENTS.md compatible | [agents.md](https://agents.md/) |
| **Forge** (Rapoport) | n/a (internal) | n/a | Anthropic-only currently | Not yet published | OpenSpec change-folder → PR; spec-driven by structural design | (internal) |

### 3.2 What's table-stakes in 2026

Every entry above ships these — they're no longer features, they're cost of admission:

1. **MCP support** (Anthropic protocol, now Linux Foundation-stewarded) — [Wikipedia](https://en.wikipedia.org/wiki/Model_Context_Protocol).
2. **AGENTS.md** as project-config file — [agents.md](https://agents.md/) is the open format used by OpenAI Codex, Amp, Jules (Google), Cursor, Factory; Linux Foundation Agentic AI Foundation governs.
3. **Git-aware operation** — auto-branch, auto-commit, PR-creation.
4. **Multi-model routing** (except Claude Code, which is Anthropic-locked by design).
5. **Headless / CI mode** — Cline shipped CLI 2.0 with this Feb 2026; Codex always had it; Forge has it.
6. **Tool-use / shell execution** — every agent can run `npm test` and read the output.
7. **Parallel subagents** — landed across the industry Feb 2026 ([MindStudio, 2026](https://www.mindstudio.ai/blog/claude-code-agent-teams-parallel-workflows)).

### 3.3 What's a real wedge

Only three structural axes still differentiate:

| Axis | Tools with the wedge | Why it's defensible |
|---|---|---|
| **Token efficiency** | Aider (4.2× cheaper than Claude Code on same task), Morph diff-only edits | Empirical, measurable, hard to copy without re-architecting prompt+diff pipeline |
| **Model lock-in (chosen)** | Claude Code (Anthropic-only) | Anthropic-quality-bar; user trades flexibility for guaranteed model quality |
| **Discipline enforcement** | Forge (OpenSpec), Spec Kit (constitution + phases), Kiro (3-doc pipeline) | Not a feature, an opinion. Hard to fork without losing the philosophy. |

### 3.4 Commentary

Of these three wedges, **discipline enforcement** is the most defensible because it's not a code-level feature. A competitor can copy "Anthropic-only" by signing a contract with Anthropic. They can copy token efficiency with engineering work. They cannot copy a methodology corpus that took 18 months to write and whose adoption requires a client to internalize its conventions.

**This is exactly Forge's wedge** vs. every entry in the table above — but the wedge only exists if the OpenSpec corpus is the artifact, not Forge itself. If the studio frames Forge as "just another CLI," it loses; if it frames Forge as "the executor for the spec methodology you bought," it wins. See § 9.1.

---

## 4. Category 2: SaaS autonomous builders

Tools that take a Linear/Jira/GitHub ticket and return a PR (or a deployed app) without you being at the keyboard. This is the category that *replaces* engineers, not augments them — and it's where Rapoport's positioning is most exposed.

### 4.1 Comparison table

| Tool | Pricing entry | Pricing scale | Unit of value | What it ships | Source |
|---|---|---|---|---|---|
| **Devin** (Cognition) | Core $20/mo | Team $500/mo (250 ACUs); Enterprise custom | ACU (~15 min autonomous work) = ~$2.00–2.25 each; ~$8–9 per "engineer-hour" | PR to GitHub from Slack/Linear ticket | [devin.ai/pricing](https://devin.ai/pricing/), [VentureBeat Apr 2026](https://venturebeat.com/programming-development/devin-2-0-is-here-cognition-slashes-price-of-ai-software-engineer-to-20-per-month-from-500) |
| **Factory.ai (Droids)** | $20/mo | Max ~$200/mo; Teams up to 150 seats | Standard usage + rollover to Droid Core (open-weight) | PR via Droid; IDE-agnostic | [Factory pricing](https://factory.ai/pricing), [Tech Insider](https://tech-insider.org/factory-ai-150-million-series-c-khosla-coding-droids-2026/) — $1.5B post-money April 2026 |
| **Codegen.com** | Custom enterprise | Custom | Per-agent / per-seat (SOC 2 Type II) | PR from ClickUp/Linear/Jira; on-prem option | [Codegen blog](https://codegen.com/blog/best-ai-coding-agents/) — pricing not public |
| **Sweep AI** | Free tier + paid | Paid scales by issues processed | Per-issue → PR | GitHub issue → PR, all in GitHub UI | [aiagentslist Sweep](https://aiagentslist.com/agents/sweep-ai) — 2026 pricing not consistently published |
| **OpenHands** (All Hands) | Free (self-host) | Cloud paid | Self-host or pay-per-task | PR via planning-mode agent | [openhands.dev](https://www.openhands.dev/) |
| **Augment Intent** | $20/mo (Augment sub) | Standard $600/mo team pool; Max ~$200/dev/mo (~450k credits) | Credits routed to Coordinator (Sonnet) / Implementor (Haiku) / Verifier | PR via multi-agent living-spec workflow | [Augment Intent pricing](https://www.augmentcode.com/guides/intent-pricing) |
| **Replit Agent** | Core $20–25/mo | Heavy users $100–300/mo on usage credits | Effort-based credits | Deployed app (cloud-hosted on Replit) | [Lovable comparison](https://lovable.dev/guides/bolt-vs-replit-vs-lovable) |
| **Lovable** | Free + Pro tiers | Per-message metering | Messages | Deployed full-stack web app | [Lovable](https://lovable.dev/) |
| **Bolt.new** | Free (~1M tokens/mo, 300k/day cap) | Pro $25–50/mo (10M–26M tokens) | Tokens | Deployed full-stack JS app | [vibecoding Bolt](https://vibecoding.app/blog/bolt-new-review) |
| **v0** (Vercel) | Free $5/mo credit | Premium $20, Team $30/seat, Business $100/seat | Tokens: $1.50/M input, $7.50/M output | React UI components / deployed Vercel app | [v0 pricing docs](https://v0.app/docs/pricing) |

### 4.2 Pricing-unit analysis: who sells *what*?

The unit-of-value choice is the most important pricing decision a SaaS builder makes — it implies the metaphor and bounds the negotiation:

| Unit | Implied metaphor | Who uses it | Pricing leverage |
|---|---|---|---|
| **ACU / engineer-hour** | "AI engineer on your team" | Devin | High — buyer compares to human salary ($100k/yr ≈ $50/hr; Devin at $8/hr is 6× cheaper) |
| **Credits routed by role** | "Pool of engineering capacity allocated by tier" | Augment Intent | Medium — buyer needs to model coordinator/implementor/verifier traffic |
| **Tokens** | "Compute pass-through, you pay for what the LLM costs" | Bolt, v0, Aider (API), Plandex | Low — buyer eventually realizes margin is in the routing, not the tokens |
| **Per-message / per-build** | "Productized output" | Lovable | High — buyer sees finished thing per unit |
| **Per-seat** | "Like Cursor / GitHub Copilot" | Claude Code, Factory, Cursor | Lowest friction, lowest ceiling |
| **Per-issue → PR** | "Outsourced junior dev" | Sweep, Codegen (enterprise) | Direct comparison to outsourcing engagement; differentiates on quality not cost |

**Devin's metaphor is the most aggressive** — selling "engineer-hours" directly anchors the customer's reference price to a human engineer's salary, not to API token cost. This is *positioning leverage*: it forces the customer to evaluate value vs. a $100k+ employee, not vs. a $20/mo subscription.

### 4.3 What these tools threaten

Forge competes most directly with **Devin**, **Factory droids**, **Augment Intent**, and **Codegen** — all four take a structured input (spec, ticket, plan) and ship a PR.

Forge wins against them when:
- The client already has a spec corpus and doesn't want a black box.
- The client cares about auditable methodology > raw throughput.
- The client wants to own the artifact (corpus + workflow) post-engagement.

Forge loses against them when:
- The client just wants the PR shipped and doesn't care how.
- The client measures value in "engineer-hours-replaced" and wants the absolute lowest $/hr.
- The client lacks the methodology maturity to author OpenSpec proposals.

**Threat severity:** Devin and Factory both shipped in 2024–25; Devin slashed pricing 25× in April 2026 ($500 → $20 entry) ([VentureBeat](https://venturebeat.com/programming-development/devin-2-0-is-here-cognition-slashes-price-of-ai-software-engineer-to-20-per-month-from-500)). The category is being commoditized rapidly. **By 2027, "agent that takes a ticket and ships a PR" will be free in most IDEs.** Forge's wedge has to be the methodology layer above, not the execution.

---

## 5. Category 3: Spec-driven dev tooling

The category that names what Rapoport is building. Three philosophical camps; not interchangeable.

### 5.1 The three camps

| Camp | Definition | Tools | Bet |
|---|---|---|---|
| **Spec-first / phase-gated** | Spec drives plan, plan drives code; sequential phases | **Spec Kit**, **Kiro**, **BMAD-Method** | Quality through discipline; greenfield-biased |
| **Spec-anchored / change-folder** | Spec lives alongside; deltas track changes | **OpenSpec** (Fission-AI), **Augment Intent** (living spec) | Brownfield reality; iteration over phases |
| **Spec-as-source** | Code regenerates from spec; spec is the maintained artifact | **Tessl** | Radical: AI is the compiler, spec is the source code |

Source: [openspec-vs-speckit-public.md](./openspec-vs-speckit-public.md) § 1 (frame); [Augment 6 SDD tools](https://www.augmentcode.com/tools/best-spec-driven-development-tools); [Tessl blog](https://tessl.io/blog/how-tessls-products-pioneer-spec-driven-development/).

### 5.2 Deep-dive: Kiro (May 2026)

Already covered in [`competitive-positioning-sdd-frameworks.md` § 2.3](./competitive-positioning-sdd-frameworks.md). Updates as of May 2026:

- Pricing: Free tier 50 agentic requests/mo; Pro $20/mo (225 vibe + 125 spec requests); Pro+ and Power $200/mo. Overage: $0.04/vibe request, $0.20/spec request ([Kiro pricing](https://kiro.dev/pricing/)).
- Three-doc convention persists: `requirements.md` / `design.md` / `tasks.md`.
- MCP support: now shipped.
- AWS lock-in: still in place (Bedrock-only model routing).
- August 2025 pricing-bug coverage and "vibe brought down AWS" incident still cited as cautionary examples ([devclass, Jul 2025](https://devclass.com/2025/07/15/hands-on-with-kiro-the-aws-preview-of-an-agentic-ai-ide-driven-by-specifications/)).

### 5.3 Deep-dive: Tessl (May 2026)

The most philosophically radical entry in the category. Worth its own section because it's the only tool betting that **spec replaces code** rather than spec produces code.

**Products as of May 2026:**

- **Tessl Framework** — "keeps agents on rails by using specifications to define what you want to build before coding" ([Tessl blog](https://tessl.io/blog/announcing-tessls-products-to-unlock-the-power-of-agents/)). Beta access.
- **Tessl Spec Registry** — over 10,000 specs explaining how to use external open-source libraries; free.

**Funding:** Series A announced for "AI Native Software Development" ([Tessl Series A](https://tessl.io/blog/announcing-our-series-a-for-ai-native-software-development/)).

**Bold claim from founder Guy Podjarny:** *"By the end of 2027, developers working with agents won't look at code most of the time, and within 2026, most development will be at least spec-assisted."* ([Tessl blog](https://tessl.io/blog/how-tessls-products-pioneer-spec-driven-development/)).

**Pricing:** Spec Registry free; Framework currently beta. No published per-seat or per-build pricing ([opentools.ai Tessl](https://opentools.ai/tools/tessl)).

**Why this matters for Rapoport:** Tessl is the only entry where the *spec corpus itself* is the product, not the tool that produces specs. If Tessl wins, Rapoport's "methodology corpus is the durable artifact" claim is *validated by the market*. If Tessl loses (or stays niche), Rapoport's positioning becomes harder to sell — "spec is the source of truth" becomes a contrarian bet rather than industry consensus.

### 5.4 Anthropic Skills as a meta-layer

Anthropic Skills ([anthropics/skills](https://github.com/anthropics/skills/)) are a new distribution primitive shipped in late 2025 / early 2026. They sit *between* spec-driven tools and agent CLIs.

**What they are:** folders containing a `SKILL.md` + scripts + resources. Loaded dynamically by Claude; can be project-scoped, plugin-scoped, or organization-scoped ([code.claude.com docs](https://code.claude.com/docs/en/skills); [Anthropic Skills PDF guide](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf?hsLang=en)).

**Cross-tool compatibility:** As of March 2026, *"the same skill files work across Claude Code, Cursor, Gemini CLI, Codex CLI, and Antigravity IDE"* ([Medium guide, Apr 2026](https://medium.com/@tort_mario/skills-for-claude-code-the-ultimate-guide-from-an-anthropic-engineer-bcd66faaa2d6)).

**Marketplace:** [SkillsMP](https://skillsmp.com/) emerged as third-party marketplace; Anthropic operates an "official marketplace" gate via PR-to-repo workflow.

**Implication for Rapoport:** OpenSpec's `_methodology/` corpus *could be* packaged as Anthropic Skills — turning the methodology into a distributable, reusable artifact across multiple CLI hosts. **This is an unexplored extension path** for Muse. See § 9.2.

### 5.5 Rules files: Cursor rules / Cline rules / AGENTS.md

Three formats compete to be "the README for AI agents":

| Format | Where it lives | Activation | Strength |
|---|---|---|---|
| **`.cursor/rules/*.mdc`** | Cursor projects | Always / Auto-attached / Agent-requested / Manual | Most expressive (4 activation modes) |
| **`CLAUDE.md`** | Claude Code projects | Always | Simple, opinionated |
| **`AGENTS.md`** | Multi-agent / multi-tool projects | Always | Open standard, Linux-Foundation-stewarded, used by OpenAI Codex, Amp, Jules, Cursor, Factory |
| **`.clinerules/`** | Cline | Folder of markdown | Project-scoped, granular |
| **`SKILL.md`** (Anthropic Skills) | Anywhere | Loaded dynamically by description match | Bundles content + executable scripts |

Source: [thepromptshelf](https://thepromptshelf.dev/blog/cursorrules-vs-claude-md/); [agents.md](https://agents.md/); [augmentcode AGENTS.md guide](https://www.augmentcode.com/guides/how-to-build-agents-md).

**The 2026 convergence:** AGENTS.md is winning as the cross-tool open format; SKILL.md is winning as the *distributable* format (works in plugins, marketplaces). The two are complementary not competitive.

**Implication for Rapoport:** Both `CLAUDE.md` (project-level) and `AGENTS.md` (cross-tool) belong in any Forge-managed repo. The studio already has [CLAUDE.md](../../../CLAUDE.md); an AGENTS.md mirror would broaden which agents can drive Forge. Worth a small change-proposal.

### 5.6 Other entries

- **kspec / ezspec** — searched, no shipping public tooling found at this name. `[unverified]`. Likely the user's recall was approximate; the established names are Spec Kit, OpenSpec, Kiro, BMAD, Tessl, Intent.
- **GenAIScript** — distinct ecosystem (Microsoft, prompt programming framework) but not spec-driven dev; out of scope.

---

## 6. Category 4: Spec quality / spec linting

**The thinnest category in the entire landscape.** Despite "spec-driven dev" being the dominant 2026 narrative, **no shipping product treats specs themselves as the artifact to lint, score, or audit** at the level Rapoport's `forge audit` operates.

### 6.1 What exists

| Tool / approach | What it does | Limitation |
|---|---|---|
| **Spectral** ([stoplight.io/spectral](https://github.com/stoplightio/spectral)) | OpenAPI / AsyncAPI / Arazzo spec linter — structural | API-spec-only, not methodology specs |
| **openapi-spec-validator** | Structural validation of OpenAPI | Same scope |
| **LLM-as-judge for specs** | Research approach: use frontier model to grade a spec against criteria | No shipping product; ICSE 2026 AI-SQE workshop is the academic venue ([conf.researchr.org](https://conf.researchr.org/home/icse-2026/ai-sqe-2026)) |
| **Augment Intent "living spec"** | Continuous update of spec from code | Updates the spec, doesn't grade it |
| **Spec Kit `/analyze`** | Phase-gate consistency check | Detects mismatch, doesn't measure quality |
| **OpenSpec `validate` (Fission-AI)** | Structural validation of change-folders | Format check, not content quality |
| **Forge audit** (Rapoport, internal) | Spec-fidelity audit against actual code | Not yet a public product |

### 6.2 What's missing

Nobody is publicly measuring:

1. **Spec churn velocity** — how often a spec is rewritten between commits. Proxy for instability.
2. **Spec → code lag** — how long between spec change and corresponding code change. Proxy for drift.
3. **Spec violations** — code changes that don't match any open spec. Proxy for unauthorized scope creep.
4. **Spec readability score** — Flesch-Kincaid-style metric for spec clarity (specs that a junior eng cannot read are specs that won't be maintained).
5. **Spec completeness** — does the spec address every requirement category (functional / non-functional / failure modes / boundary conditions)?
6. **Cost-per-spec-line shipped** — token cost of implementing each line of spec; lets you compare spec authoring efficiency across teams.

### 6.3 Commentary

This is the single biggest *underclaimed* gap in the landscape as of May 2026. Everyone agrees specs matter; nobody is shipping the measurement.

**Why nobody has shipped it yet:**
- Hard to define "good spec" without committing to a methodology (Spec Kit's "good" ≠ OpenSpec's "good" ≠ Kiro's "good"). LLM-as-judge approaches exist but lack methodology-neutral ground truth.
- Audience is small — only teams with mature spec discipline care. Same audience size that limited Spectral's TAM.
- Hard to monetize standalone; better as a feature of a bigger platform.

**Implication for Rapoport:** Forge's `forge audit` is closest to this category. Productizing it as **"spec quality CI"** — a GitHub Action that fails a PR if it violates the spec corpus, with grades and trend data — is a credible category-defining move. See § 9.1.

---

## 7. Category 5: Agent observability & metrics

The category that tells you *what your coding agents did and what it cost*. Six production-grade platforms as of May 2026.

### 7.1 Platform comparison

| Platform | Pricing entry | Pricing model | Best fit | Source |
|---|---|---|---|---|
| **LangSmith** | $39/mo team | Per-trace + per-seat | LangChain + LangGraph users (deepest framework integration) | [LangSmith via getathenic](https://getathenic.com/blog/langsmith-vs-helicone-vs-langfuse-comparison) |
| **Langfuse** | Free self-host / $50/mo cloud | Per-span; OTel-based | Open-source-friendly, framework-agnostic | [Spheron Langfuse guide](https://www.spheron.network/blog/llm-observability-gpu-cloud-langfuse-arize-phoenix-helicone/) |
| **Arize Phoenix** | Free OSS | Self-host or hosted | Eval-rigorous teams | [Spheron guide](https://www.spheron.network/blog/llm-observability-gpu-cloud-langfuse-arize-phoenix-helicone/) |
| **Helicone** | Was $20+/mo; **maintenance mode since March 2026** (founders joined Mintlify) — do not select for new deployments | API-call level | API-call-level visibility, framework-agnostic | [digitalapplied](https://www.digitalapplied.com/blog/agent-observability-platforms-langsmith-langfuse-arize-2026) |
| **Braintrust** | Custom enterprise | Brainstore OLAP DB + per-trace | Mature eval culture; CI-gating on regressions; "Loop" AI assistant for eval dev | [braintrust.dev](https://www.braintrust.dev/articles/agent-observability-complete-guide-2026) |
| **Datadog LLM Observability** | Bundled w/ Datadog | Per-host + per-trace | Existing Datadog customers | [softcery LLM obs](https://softcery.com/lab/top-8-observability-platforms-for-ai-agents-in-2025) |

### 7.2 Metrics they capture

Across the six platforms, the common metrics surfaced are:

- **Tracing:** every step the agent takes — tool calls, reasoning chains, state transitions, memory reads/writes, model responses ([Braintrust](https://www.braintrust.dev/articles/agent-observability-complete-guide-2026)).
- **Cost analytics:** $-per-trace, $-per-task, $-per-day; broken down by model and tool.
- **Latency:** p50/p95/p99 per call and end-to-end.
- **Token usage:** input/output by model.
- **Evals:** LLM-as-judge scores, rule-based assertions, custom scorers.
- **CI gating:** block deploys when eval scores regress.

### 7.3 Cost at scale

At 50 million spans per month (mid-2026 enterprise running ~10 production agents), Langfuse Cloud runs roughly **$36k/year** before retention overage ([digitalapplied](https://www.digitalapplied.com/blog/agent-observability-platforms-langsmith-langfuse-arize-2026)). This sets the floor for what production agent telemetry costs.

### 7.4 What they don't measure (yet)

None of the six publicly measures **spec adherence** as a first-class metric. Token cost, latency, eval scores are universal — but no platform tracks "did the agent's output match the spec it was given?" The closest is Braintrust's eval-as-judge, but it requires the user to author the eval against an arbitrary criterion.

**Implication for Rapoport:** there's a natural Forge integration with Langfuse or Braintrust that exposes **cost-per-spec-line-shipped** and **spec-fidelity-score** as first-class metrics. This is a 1–2 week integration, not a strategic pivot, but it produces a metric category nobody else publishes.

---

## 8. Trends 2026 (seven trends with evidence)

### 8.1 The "engineer-hour" is the emerging unit of value

Devin pioneered ACU-as-engineer-hour-substitute ([devin.ai/pricing](https://devin.ai/pricing/)); Cognition reports clients seeing "12x efficiency improvement in engineering hours saved, over 20x cost savings" ([digitalapplied Devin](https://www.digitalapplied.com/blog/devin-ai-autonomous-coding-complete-guide)). Factory raised $150M at $1.5B post-money on this framing ([tech-insider](https://tech-insider.org/factory-ai-150-million-series-c-khosla-coding-droids-2026/)). The unit-of-value choice anchors customer reference price.

### 8.2 Multi-agent shipped in a two-week window in February 2026

Grok Build (8 parallel agents), Windsurf (5 parallel), Claude Code Agent Teams (behind `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`), Codex CLI with OpenAI Agents SDK, Devin's "fleet of Devins" — all landed within ~14 days ([Cognition Devin 2.0](https://cognition.ai/blog/devin-2); [Claude Code Agent Teams](https://code.claude.com/docs/en/agent-teams)). Multi-agent is no longer a wedge; *coordination patterns* are.

### 8.3 Pricing is collapsing — Devin went from $500 → $20 entry (April 2026)

The flagship "AI software engineer" repositioned from premium to commodity in one announcement ([VentureBeat Apr 2026](https://venturebeat.com/programming-development/devin-2-0-is-here-cognition-slashes-price-of-ai-software-engineer-to-20-per-month-from-500)). Bolt.new, v0, Lovable, Replit Agent, Claude Code, Cursor, Factory, Devin all now start at $20/mo. **The price floor is set; differentiation is migrating up-stack.**

### 8.4 MCP is the de facto tool-exposure standard; A2A is the emerging agent-to-agent layer

MCP (Anthropic, Nov 2024) became cross-vendor standard within 18 months ([Wikipedia](https://en.wikipedia.org/wiki/Model_Context_Protocol)). A2A (Google, now Linux Foundation, 50+ partners including Atlassian/Salesforce/SAP/ServiceNow) is the emerging agent-to-agent protocol; v0.3 added gRPC and signed security cards ([linuxfoundation A2A](https://www.linuxfoundation.org/press/linux-foundation-launches-the-agent2agent-protocol-project-to-enable-secure-intelligent-communication-between-ai-agents); [Google A2A blog](https://developers.googleblog.com/en/a2a-a-new-era-of-agent-interoperability/)). MCP-Apps (formerly mcp-ui, SEP-1865) is the UI-from-MCP standard ([Wikipedia MCP](https://en.wikipedia.org/wiki/Model_Context_Protocol)).

**No event-bus standard yet** — cross-agent event streams remain ad-hoc.

### 8.5 The IDE wars are over; the methodology wars are starting

Cursor / Windsurf / Antigravity / VS Code-with-Copilot all converged on the same UX. The differentiation moved up: Spec Kit vs OpenSpec vs Kiro vs BMAD vs Tessl vs Intent. **The 2026 buying decision is "which methodology," not "which IDE."**

### 8.6 Anthropic Skills is becoming the cross-tool distribution primitive

Same SKILL.md works in Claude Code, Cursor, Gemini CLI, Codex CLI, Antigravity IDE ([Medium guide](https://medium.com/@tort_mario/skills-for-claude-code-the-ultimate-guide-from-an-anthropic-engineer-bcd66faaa2d6)). Third-party marketplace ([SkillsMP](https://skillsmp.com/)) emerged. This is bigger than it looks — it's the first distribution primitive that lets a *methodology* be packaged as an artifact agents pick up automatically.

### 8.7 "Living specs" are mainstream — but mean different things

Augment Intent's "living spec" updates from code-changes ([Augment Intent](https://www.augmentcode.com/product/intent)). OpenSpec's "spec-anchored" model treats archived change-folders as immutable history. Tessl treats spec as the maintained source code. Spec Kit treats spec as the originating doc that gets regenerated. **Pick your living-spec semantics — they're not compatible.**

---

## 9. Implications for Rapoport's orchestra

### 9.1 Forge (code exec) — positioning

**Threat surface:** Devin, Factory, Codegen, Augment Intent, Sweep — all do "ticket → PR." Devin's $20 entry and Factory's $1.5B war chest mean this category will be competitive on raw throughput.

**Where Forge wins:**
- **Spec is the input contract, not the prompt.** Forge takes an OpenSpec change-folder, not a Linear ticket. The spec is portable and reviewable *before* Forge runs.
- **Methodology audit is a first-class operation.** `forge audit` exists; the SaaS competitors don't have an equivalent. This is a 6-month head start that should be productized.
- **Client owns the corpus.** Per IP model B, the spec corpus is the deliverable; Forge is an implementation detail. Devin's customers own nothing; the spec they wrote is in Devin's UI.

**Where Forge is at risk of being commoditized:**
- Pure execution speed — Devin/Factory will be faster.
- Per-engineer-hour cost — they'll be cheaper at scale.
- Multi-agent parallel throughput — they shipped Feb 2026; Forge has dispatch serialization per build lock (see [`forge_global_build_lock`](../../../) memory entry).

**Recommendation:** Forge should be positioned as **"the executor for your OpenSpec methodology,"** not "an AI coder." If clients want a generic AI coder, point them at Claude Code or Codex. Forge's value is the discipline it enforces, not the code it ships.

**Concrete moves:**
1. Ship "spec quality CI" — Forge as a GitHub Action that grades PRs against the spec corpus (see § 6.3). No competitor has this.
2. Expose `cost-per-spec-line-shipped` via Langfuse/Braintrust integration (see § 7.4).
3. Publish AGENTS.md compatibility so other CLIs can drive Forge as a backend.

### 9.2 Muse (spec authoring) — positioning

**Threat surface:** Tessl's Framework + Registry, Spec Kit's `/specify`+`/clarify`, Augment Intent's spec composer, Anthropic Skills, Kiro's three-doc workflow.

**Where Muse wins:**
- **Adversarial five-persona research** (per [`rap-119-as-research-bed.md`](./rap-119-as-research-bed.md) artifacts) — no other spec-authoring tool runs adversarial review on draft specs.
- **Decision provenance:** `D-*` rows + `decisions.md` give Muse-authored specs a *why* trail. Spec Kit's constitution is project-wide; Muse's decisions are per-decision.
- **Brownfield-native** via OpenSpec; greenfield tools (Spec Kit, Tessl Framework) struggle here.

**Where Muse is at risk of being commoditized:**
- If Tessl wins, *spec authoring* becomes a free Tessl primitive and Muse's wedge collapses to "spec authoring + opinionated methodology" — which is a thinner wedge.
- Anthropic Skills + AGENTS.md may flatten the methodology layer; Muse needs to ride the wave, not fight it.

**Concrete moves:**
1. Package the studio's methodology corpus as **Anthropic Skills** — distribute via SkillsMP marketplace. Cost: weeks. Reach: every Claude Code / Cursor / Gemini CLI user.
2. Position Muse as the **multi-persona adversarial author** — explicitly cite Spec Kit's "fluent single-author" model as its opposite. This is the public differentiation copy.
3. Track Tessl's GA — if Tessl Framework ships paid, the threat is real and Muse needs a Tessl-Registry integration (consume their 10,000-spec OSS library as input).

### 9.3 Maestro (orchestrator) — positioning

**Threat surface:** thinnest of the three. Devin's "fleet of Devins" is the closest analog; Claude Code Agent Teams is the closest CLI analog. Neither has email + calendar surface.

**Where Maestro wins:**
- **Multi-channel surface** — email + calendar + Telegram, where Devin/Claude Code Agent Teams are CLI/IDE-only. This is the "administrative orchestrator" gap nobody else fills.
- **Named-agent role separation** — Forge does execution, Muse does authoring, Maestro does dispatch. Devin's "team of Devins" are all the same agent.
- **Spec-grounded** — Maestro dispatches by reading OpenSpec change-folders, not prompts.

**Where Maestro is at risk:**
- *Maestro doesn't exist yet.* It's planned ([CLAUDE.md](../../../CLAUDE.md) notes "Maestro (planned)"). By the time it ships, Cognition or Factory may have shipped equivalent multi-channel surfaces.
- Email + calendar surfaces are not technically hard; they're moats only if Maestro becomes the only place where the studio's specialist-network coordinates.

**Concrete moves:**
1. **Ship the email surface first** (hello@rapoport.studio is reserved per Pavel's memory note). That's the Maestro MVP — accept email, parse intent, dispatch to Forge or Muse. Cost: weeks. Value: founder communicates with the studio via email, agent dispatches the work.
2. Calendar surface is P2 — only useful once specialist-network is multi-person.
3. Position Maestro as **"the orchestrator that pre-existed Devin's team mode"** — the role-separation is the wedge, not the parallelism.

### 9.4 Where the wedge is (orchestra-level)

Across all three CLIs, the orchestra's wedge is:

> **A spec-first, named-agent orchestra where the spec corpus is the durable artifact the client owns, and each named agent is auditable by role.**

No competitor has all three properties:
- **Devin / Factory / Codegen:** black-box; client doesn't own a corpus.
- **Spec Kit / OpenSpec / Kiro:** spec-first but single-agent (or agent-agnostic, which means no role separation).
- **BMAD:** multi-agent personas but client never owns the methodology (it's BMAD's framework).
- **Augment Intent:** coordinator/implementor/verifier role separation BUT proprietary platform; client doesn't own the artifact.
- **Tessl:** spec-as-source but unclear on multi-agent role separation.

### 9.5 Where Rapoport is at risk of being commoditized

| Risk | Severity | Time horizon | Mitigation |
|---|---|---|---|
| **Forge as "yet another AI CLI"** | High | 12 months | Position as OpenSpec executor; ship spec-quality CI |
| **Muse vs Tessl's Framework** | Medium-High | 18 months | Distribute methodology via Anthropic Skills; track Tessl GA |
| **MCP / A2A / AGNTCY consolidation** | Medium | 24 months | Adopt early; don't build proprietary protocols |
| **Methodology corpus becomes commodity** | Medium | 24–36 months | First-mover on publishing under FSL; SkillsMP marketplace presence |
| **Pricing-floor pressure ($20/mo everywhere)** | High | 12 months | Don't compete on $/mo; sell engagement-priced methodology services (€5k Discovery floor) |

---

## 10. Open questions for Pavel

| ID | Priority | Question |
|---|---|---|
| `Q-CLI-1` | P0 | Should Forge publish AGENTS.md compatibility so other CLIs (Cursor, Codex, Claude Code) can drive it as a backend? Lowers adoption friction; risk of commoditizing the discipline layer. |
| `Q-CLI-2` | P0 | Should the studio package its methodology corpus as Anthropic Skills + publish to SkillsMP? Distribution: every Claude Code / Cursor / Codex user. Risk: methodology becomes consumable without engagement; weakens services pitch. |
| `Q-CLI-3` | P0 | Is "spec quality CI" the Forge productization play (productize `forge audit` as a GitHub Action for non-clients)? Big lift; differentiates against Devin/Factory. |
| `Q-CLI-4` | P1 | Adopt A2A protocol for cross-agent dispatch when Maestro ships? Aligns with Linux Foundation standard; risk of premature commitment if A2A doesn't win against MCP-only patterns. |
| `Q-CLI-5` | P1 | Devin slashed pricing 25× in April 2026. At what Devin-pricing point does Forge need to respond on pricing? Studio's €5k Discovery is engagement-priced not per-seat — likely insulated, but worth modeling. |
| `Q-CLI-6` | P1 | Does the studio publish a public Forge/Muse/Maestro vs Devin/Factory/Augment Intent comparison page? Mirrors §9 internally; question is whether to make it external. |
| `Q-CLI-7` | P2 | Integrate Langfuse or Braintrust for production telemetry on Forge runs? Cost: weeks. Value: cost-per-spec-line + spec-fidelity-score as first-class published metrics. |
| `Q-CLI-8` | P2 | Tessl GA tracking: when Tessl Framework ships paid, what's the studio's response? Consume Tessl Spec Registry as Muse input? Compete? Ignore? |
| `Q-CLI-9` | P2 | Should Forge support BYOK / multi-model routing (currently Anthropic-only)? Aligns with industry trend; risks losing the "Anthropic-quality-floor" wedge. |
| `Q-CLI-10` | P3 | If Maestro ships with email surface, does hello@rapoport.studio become the *primary* orchestrator address, or stays as Pavel's secondary? Memory note resolves to "Maestro's mailbox when Maestro ships" — implementation question. |

---

## 11. Honest weaknesses of this research

Not selling. Calling out so re-readers know what to verify:

1. **Pricing data ages in weeks.** Devin moved $500 → $20 in one announcement (April 2026). Treat every $-figure here as accurate-as-of-May-2026 only.
2. **Some 2026 third-party reviews are vendor-positioned.** Augment Code competitor pages (cited heavily because they're current) are vendor-positioned analyses. Treat as "one source with known bias."
3. **kspec / ezspec** — not found as shipping products. May be the user's recall approximating Spec Kit / SpecOps / similar names. Flagged `[unverified]`.
4. **Continue.dev** — 2026 CLI-specific details thin in searches; mostly an IDE extension positioning. Flagged `[unverified]` for CLI specifics.
5. **No first-party empirical benchmark.** Unlike [`openspec-vs-speckit-public.md`](./openspec-vs-speckit-public.md) which measured install + token + agent context on the studio machine, this file is a literature-synthesis pass. Real Forge-vs-Devin head-to-head on a real spec would need to be its own follow-up (see Q-CLI-7).
6. **No GitHub API audit** of contributor diversity, PR merge time, etc. for the 12+ CLIs covered. Unlike the OpenSpec-vs-Spec-Kit file which did this for two tools, doing it for twelve is out of scope here. Worth a follow-up if any specific tool becomes a critical reference.
7. **Adoption / market-share data is missing.** GitHub stars are a vanity metric; install counts (Cline's "5M+") are vendor-reported. There's no neutral source for "how many engineering teams actually use Devin in production" — every figure I'd cite would be vendor-PR'd.

---

## 12. Sources

### Primary (vendor pricing / docs)

| Source | URL |
|---|---|
| Devin pricing | [devin.ai/pricing](https://devin.ai/pricing/) |
| Cognition Devin 2.0 announcement | [cognition.ai/blog/devin-2](https://cognition.ai/blog/devin-2) |
| Factory.ai pricing | [factory.ai/pricing](https://factory.ai/pricing) + [docs.factory.ai/pricing](https://docs.factory.ai/pricing) |
| Augment Intent pricing | [augmentcode.com/guides/intent-pricing](https://www.augmentcode.com/guides/intent-pricing) |
| Augment Intent product | [augmentcode.com/product/intent](https://www.augmentcode.com/product/intent) |
| Claude Code docs | [code.claude.com/docs](https://code.claude.com/docs/en/skills) |
| Claude Code Agent Teams | [code.claude.com/docs/en/agent-teams](https://code.claude.com/docs/en/agent-teams) |
| Anthropic Skills repo | [github.com/anthropics/skills](https://github.com/anthropics/skills/) |
| Anthropic Skills complete guide PDF | [resources.anthropic.com](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf?hsLang=en) |
| Kiro pricing | [kiro.dev/pricing](https://kiro.dev/pricing/) |
| Kiro specs docs | [kiro.dev/docs/specs](https://kiro.dev/docs/specs/) |
| Tessl Framework + Registry announcement | [tessl.io/blog/announcing-tessls-products](https://tessl.io/blog/announcing-tessls-products-to-unlock-the-power-of-agents/) |
| Tessl SDD positioning | [tessl.io/blog/how-tessls-products-pioneer-spec-driven-development](https://tessl.io/blog/how-tessls-products-pioneer-spec-driven-development/) |
| Tessl Series A | [tessl.io/blog/announcing-our-series-a](https://tessl.io/blog/announcing-our-series-a-for-ai-native-software-development/) |
| Plandex | [plandex.ai](https://plandex.ai/) + [github.com/plandex-ai/plandex](https://github.com/plandex-ai/plandex) |
| OpenHands | [openhands.dev](https://www.openhands.dev/) + [github.com/OpenHands/OpenHands](https://github.com/OpenHands/OpenHands) |
| v0 pricing | [v0.app/docs/pricing](https://v0.app/docs/pricing) |
| AGENTS.md | [agents.md](https://agents.md/) |
| Cursor rules docs | [cursor.com/docs/rules](https://cursor.com/docs/rules) |
| A2A protocol | [a2a-protocol.org](https://a2a-protocol.org/latest/) + [github.com/a2aproject/A2A](https://github.com/a2aproject/A2A) |
| Linux Foundation A2A | [linuxfoundation.org](https://www.linuxfoundation.org/press/linux-foundation-launches-the-agent2agent-protocol-project-to-enable-secure-intelligent-communication-between-ai-agents) |
| MCP Wikipedia | [en.wikipedia.org/wiki/Model_Context_Protocol](https://en.wikipedia.org/wiki/Model_Context_Protocol) |
| Braintrust agent observability guide | [braintrust.dev/articles/agent-observability-complete-guide-2026](https://www.braintrust.dev/articles/agent-observability-complete-guide-2026) |
| Anthropic March 2026 usage promotion | [support.claude.com](https://support.claude.com/en/articles/14063676-claude-march-2026-usage-promotion) |
| Anthropic higher Claude Code limits | [anthropic.com/news/higher-limits-spacex](https://www.anthropic.com/news/higher-limits-spacex) |

### Secondary (analyses, comparisons)

| Source | URL |
|---|---|
| Requesty 2026 agentic coding comparison | [requesty.ai/blog/agentic-coding-tools-compared-2026](https://www.requesty.ai/blog/agentic-coding-tools-compared-2026-claude-code-cursor-codex-aider) |
| Tembo 2026 CLI tools comparison | [tembo.io/blog/coding-cli-tools-comparison](https://www.tembo.io/blog/coding-cli-tools-comparison) |
| DEV "Every AI Coding CLI in 2026" | [dev.to/soulentheo](https://dev.to/soulentheo/every-ai-coding-cli-in-2026-the-complete-map-30-tools-compared-4gob) |
| Builder.io Codex vs Claude Code | [builder.io/blog/codex-vs-claude-code](https://www.builder.io/blog/codex-vs-claude-code) |
| Blakecrosley Codex CLI vs Claude Code 2026 | [blakecrosley.com](https://blakecrosley.com/blog/codex-vs-claude-code-2026) |
| Morph LLM Aider token comparison | [morphllm.com/comparisons/morph-vs-aider-diff](https://www.morphllm.com/comparisons/morph-vs-aider-diff) |
| VentureBeat Devin 2.0 pricing slash | [venturebeat.com](https://venturebeat.com/programming-development/devin-2-0-is-here-cognition-slashes-price-of-ai-software-engineer-to-20-per-month-from-500) |
| Tech Insider Factory $150M Series C | [tech-insider.org](https://tech-insider.org/factory-ai-150-million-series-c-khosla-coding-droids-2026/) |
| Lovable Bolt/Replit comparison | [lovable.dev/guides/bolt-vs-replit-vs-lovable](https://lovable.dev/guides/bolt-vs-replit-vs-lovable) |
| Bolt.new review | [vibecoding.app/blog/bolt-new-review](https://vibecoding.app/blog/bolt-new-review) |
| Cognition Devin 18-month review | [cognition.ai/blog/devin-annual-performance-review-2025](https://cognition.ai/blog/devin-annual-performance-review-2025) |
| Augment 6 SDD tools | [augmentcode.com/tools/best-spec-driven-development-tools](https://www.augmentcode.com/tools/best-spec-driven-development-tools) |
| Augment AGENTS.md guide | [augmentcode.com/guides/how-to-build-agents-md](https://www.augmentcode.com/guides/how-to-build-agents-md) |
| Augment best Devin alternatives | [augmentcode.com/tools/best-devin-alternatives](https://www.augmentcode.com/tools/best-devin-alternatives) |
| Augment multi-agent orchestration build vs buy | [augmentcode.com/tools/multi-agent-orchestration-platforms-build-vs-buy](https://www.augmentcode.com/tools/multi-agent-orchestration-platforms-build-vs-buy) |
| Augment Intent review (Dr. Tali Rezun) | [medium.com/@talirezun](https://medium.com/@talirezun/from-writing-code-to-directing-intelligence-five-days-inside-augment-codes-intent-7b04863808bf) |
| Pinggy Top 5 CLI coding agents 2026 | [pinggy.io/blog/top_cli_based_ai_coding_agents](https://pinggy.io/blog/top_cli_based_ai_coding_agents/) |
| Terminal Trove AI coding agents | [terminaltrove.com/compare/ai-coding-agents](https://terminaltrove.com/compare/ai-coding-agents/) |
| Codegen "Best AI Coding Agents 2026" | [codegen.com/blog/best-ai-coding-agents](https://codegen.com/blog/best-ai-coding-agents/) |
| Sweep AI review | [aiagentslist.com/agents/sweep-ai](https://aiagentslist.com/agents/sweep-ai) |
| OpenAI "How to Write a Good Spec for AI Agents" | [oreilly.com/radar](https://www.oreilly.com/radar/how-to-write-a-good-spec-for-ai-agents/) + [addyosmani.com](https://addyosmani.com/blog/good-spec/) |
| AI-SQE workshop ICSE 2026 | [conf.researchr.org](https://conf.researchr.org/home/icse-2026/ai-sqe-2026) |
| SpecOps 2026 workshop | [conf.researchr.org/specops-2026](https://conf.researchr.org/home/splash-issta-2026/specops-2026) |
| AI Evaluation Metrics Reference 2026 | [digitalapplied.com](https://www.digitalapplied.com/blog/ai-evaluation-metrics-reference-guide-2026) |
| Helicone alternatives (Helicone maintenance mode mention) | [getathenic.com](https://getathenic.com/blog/langsmith-vs-helicone-vs-langfuse-comparison) |
| MindStudio Claude Code Agent Teams | [mindstudio.ai/blog/claude-code-agent-teams](https://www.mindstudio.ai/blog/claude-code-agent-teams-parallel-workflows) |
| Cursor rules vs CLAUDE.md vs AGENTS.md | [thepromptshelf.dev](https://thepromptshelf.dev/blog/cursorrules-vs-claude-md/) |
| Anthropic Skills + Codex + ChatGPT marketplace | [skillsmp.com](https://skillsmp.com/) |
| Anthropic Skills complete guide (Medium) | [medium.com/@tort_mario](https://medium.com/@tort_mario/skills-for-claude-code-the-ultimate-guide-from-an-anthropic-engineer-bcd66faaa2d6) |

---

## 13. Change log

| Date | Author | Change |
|---|---|---|
| 2026-05-11 | Pavel (founder) + Claude Opus 4.7 | Opened as canonical landscape document. Twelve AI coding CLIs, ten SaaS autonomous builders, three SDD camps (spec-first / spec-anchored / spec-as-source), Tessl deep-dive, Anthropic Skills meta-layer, AGENTS.md/Cursor rules/Cline rules format comparison, spec-quality gap mapped, six agent-observability platforms compared, seven 2026 trends with evidence, orchestra-level positioning for Forge/Muse/Maestro, ten open questions. Source-cited throughout; vendor-positioned reviews flagged; `[unverified]` markers where I couldn't confirm. AI assistance: drafted with Claude Opus 4.7 (Anthropic). |
