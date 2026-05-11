# AI risk frameworks + document indexing for AI agents (2026)

> **Status:** complete (snapshot, 2026-05-11)
> **Owner:** Pavel Rapoport (founder).
> **Authors:** Pavel Rapoport <pavel@rapoport.studio> (founder, primary); Claude Opus 4.7 (Anthropic) — web research across NIST AI RMF, MITRE ATLAS v5.1.0, OWASP LLM Top 10 2025, EU AI Act, Anthropic RSP, llms.txt standard, AGENTS.md, prompt-caching docs, vector-vs-file retrieval analysis, ~27 search queries.
> **Method:** Single-pass synthesis of two adjacent topics — (1) AI risk taxonomies that apply to small studio context, (2) document indexing/caching patterns that the same risk register can leverage. Sources include NIST publications, MITRE ATLAS factsheet, Anthropic engineering blog, GitHub blog 2500-repo AGENTS.md study, Cursor/Aider/Cody architecture posts.
> **Reusable for:** `establish-risk-framework` (Epic 3) — risk register taxonomy, OWASP checklist, red lines source; `introduce-indexing-and-llms-standards` (Epic 4) — INDEX.md pattern, llms.txt adoption, prompt cache architecture. Authorship: open.
> **Opened:** 2026-05-11
> **Feeds into:** Epic 3 (risk register, red lines, trust gradient template); Epic 4 (INDEX.md, llms.txt, frontmatter discipline, prompt cache).

---

## TL;DR

Five industry risk frameworks (NIST AI RMF, MITRE ATLAS, OWASP LLM Top 10, EU AI Act, Anthropic RSP) layer rather than compete — each owns a different zone. For Rapoport Studio: adopt OWASP LLM Top 10 fully (PR-checklist), Cagan Four Risks fully (proposal.md template), NIST AI RMF as mental model only, ATLAS partially (threat-model for Forge), FMEA only for large epics. Indexing: progressive-disclosure pattern (Anthropic Skills) + llms.txt for public site + 4-layer prompt cache (system 5m / tools 5m / openspec bundle 1h / dynamic no-cache) gives ~85% latency reduction on long prompts plus reduced hallucination from irrelevant context.

---

## Part 1 — AI risk frameworks

### Industry frameworks at a glance

| Framework | Year | Structure | Applicability to small studio |
|---|---|---|---|
| **NIST AI RMF 1.0 + GenAI Profile (AI 600-1)** | 2023 + Jul 2024 | 4 functions: Govern, Map, Measure, Manage. GenAI Profile adds 12 GAI-specific risks (CBRN, confabulation, info integrity, harmful bias) | HIGH — Govern as cultural frame; Map/Measure/Manage as operating cycle. Adopt without certification |
| **MITRE ATLAS v5.1.0** | Nov 2025 | 16 tactics × 84 techniques. Nov 2025 added Command and Control (AML.TA0015) + 18 agent-security techniques | MEDIUM — useful checklist for Forge threat-model (esp. prompt-injection from repo files) |
| **OWASP Top 10 for LLM Apps 2025** | Nov 2024 (v2025) | LLM01 Prompt Injection, LLM02 Sensitive Info, LLM03 Supply Chain, LLM04 Data Poisoning, LLM05 Improper Output Handling, LLM06 Excessive Agency, LLM07 System Prompt Leakage, LLM08 Vector Weaknesses, LLM09 Misinformation, LLM10 Unbounded Consumption | HIGH — direct PR checklist for CLI/Worker code |
| **EU AI Act** | Aug 2024 in force; Feb 2025 prohibitions; Aug 2025 GPAI rules | 4 tiers: unacceptable / high / limited / minimal. Studio AI surfaces = "limited" (transparency required) | LOW-MEDIUM — mostly transparency labeling |
| **Anthropic RSP v3.0 + ASL** | 2025; ASL-3 activated May 2025 with Opus 4 | ASL-1 (no risk) → ASL-3 (catastrophic misuse uplift) → ASL-4+ TBD | LOW — frontier-lab concept; useful as worldview only |
| **ISO/IEC 42001** | Dec 2023 | AIMS by PDCA cycle. Governance, risk, transparency, lifecycle | LOW for certification, MEDIUM as structural template |

### Classical product risks

- [Cagan SVPG Four Risks](https://www.svpg.com/four-big-risks/): **Value** (will they buy?), **Usability** (can they figure it out?), **Feasibility** (can we build it?), **Business Viability** (does it work for legal/finance/sales/brand?). Each owned by a different person.
- [Asana RAID log](https://asana.com/resources/raid-log): Risks / Assumptions / Issues / Dependencies. Fields: ID, description, priority, owner, mitigation, target date. For long-running epics, not single PRs.
- [ASQ FMEA](https://asq.org/quality-resources/fmea): RPN = Severity × Occurrence × Detection (each 1–10). RPN > 200 = blocking, must have automated mitigation (test/type-check/lint).

### Top-10 SDD-specific risks (with industry mitigation)

| # | Risk | Industry mitigation |
|---|---|---|
| 1 | Spec-implementation drift | Spec-anchored mode (Thoughtworks): spec update in same PR as code |
| 2 | Hallucination in spec-generation | Grounding with `<read_first>` files; ban new names without source |
| 3 | Stale documentation cascade | Refs guard in CI; status lifecycle |
| 4 | Cascading errors | Pre-merge verifier gate; fail-fast in epics. [CodeRabbit 2025](https://www.coderabbit.ai/blog/state-of-ai-vs-human-code-generation-report): AI PRs = 10.83 issues vs 6.45 human (1.7×); +75% logic errors |
| 5 | Ambiguity in spec language | Ubiquitous Language; banned subjective qualifiers (fast/robust/user-friendly) |
| 6 | Lost context / context window | Bundle pattern; progressive disclosure |
| 7 | Prompt injection via repo files | CVE-2025-53773 (Copilot RCE via repo comments). [Unit42 indirect prompt injection](https://unit42.paloaltonetworks.com/ai-agent-prompt-injection/) — sandbox exec; whitelist file reading |
| 8 | Bias amplification through templates | Periodic template audit; diverse few-shot exemplars |
| 9 | Tooling single-point-of-failure | Multi-provider failover: [Universal.cloud](https://universal.cloud/en/blog/ai-uptime-vergeten-risico/) — "Two independent LLM providers each 99.3% uptime → simultaneous-down 0.0049%, effective 99.995%" |
| 10 | SpecFall antipattern | Time-box proposal phase; thin slices; ban specs > N words without explicit epic |

### Risk taxonomy recommendation (5 levels, external→internal)

| Level | What | Where it lives in repo |
|---|---|---|
| L1 — Regulatory | EU AI Act tier; GDPR | `methodology/studio-charter.md` section |
| L2 — Security | OWASP LLM Top 10 | `methodology/security/security-review-checklist.md` |
| L3 — Product | Cagan Four Risks | `proposal.md § Risks` (mandatory) |
| L4 — SDD-specific | Drift, hallucination, ambiguity, etc. | `proposal.md § Risks` + Forge verifier |
| L5 — Implementation | Per-change technical risks | `design.md § Risks` |

### Adoption recommendation (small studio context)

**Adopt fully:** OWASP LLM Top 10 (PR checklist), Cagan Four Risks (proposal template), NIST AI RMF as mental model only.
**Adopt partially:** MITRE ATLAS Initial Access + Execution for Forge threat-model; FMEA for epics > 5 PRs; EU AI Act limited-tier labeling ("Drafted by Maestro").
**Skip:** ISO/IEC 42001 certification (bureaucratic overhead for 1-3 person studio); Anthropic RSP/ASL as rulebook (not applicable); EU AI Act high-risk requirements (not credit-scoring or biometric); full RAID log (Linear already serves this purpose).

---

## Part 2 — Document indexing for AI agents

### Standards landscape

| Standard | Author / Source | Structure | Adopted by |
|---|---|---|---|
| **llms.txt + llms-full.txt** | Jeremy Howard, Answer.AI (Sep 2024) | `/llms.txt` H1 name + blockquote + H2 sections with markdown links. `llms-full.txt` = full content single file | Anthropic, Cursor, Cloudflare, Vercel, Mintlify; 844K sites (BuiltWith Oct 2025) |
| **AGENTS.md** | OpenAI (Aug 2025) | Markdown in repo root + nested AGENTS.md (closest-wins) | OpenAI Codex, GitHub Copilot, Cursor, Google Jules/Gemini, Factory, Amp, Windsurf, Zed, RooCode; 60K+ projects |
| **Anthropic Skills frontmatter** | Anthropic (2025) | YAML: name + description (≤1024 chars), optional license + allowed-tools. ~100 tokens per skill in system prompt; SKILL.md body loaded on-demand | Claude Code, Claude API |
| **Cursor Rules** (`.cursor/rules/*.mdc`) | Cursor | MDC with frontmatter, glob-scoped | Cursor |
| **CLAUDE.md hierarchy** | Anthropic | `/etc/claude-code/CLAUDE.md` → `~/.claude/CLAUDE.md` → project root → walked-up. Later-loaded wins | Claude Code |

Consolidated conventions:
- Frontmatter mandatory; description ≤ 1024 chars.
- SKILL.md ≤ 500 lines (Anthropic).
- kebab-case directory names; gerund-form description.
- Closest-wins for AGENTS.md; later-loaded-wins for CLAUDE.md — **note the difference**.

### Prompt caching strategy (Anthropic-specific)

[platform.claude.com/docs/build-with-claude/prompt-caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching):
- `cache_control: { type: "ephemeral" }` creates breakpoint.
- TTL: **5m** default vs **1h** (`ttl: "1h"`, 2× write cost).
- Min cacheable: 1024 tokens (Sonnet), 4096 (Opus, Haiku 4.5).
- Max 4 breakpoints per request.
- Pricing: cache write 1.25× base (5m) / 2× (1h); cache read **0.1× base** (90% discount).

Recommended 4-layer hierarchy:
```
L1 — system prompt + role           5m   (always hit)
L2 — tools + skills index           5m
L3 — openspec bundle / repo-map     1h   (read ≥2x in epic)
L4 — dynamic context (task + diff)  no cache
```

Anthropic claim: "up to 85% latency reduction on long prompts. 100K-token book: 11.5s → 2.4s."

### Retrieval patterns — vector vs file vs bundle

**Vector (Cursor, Sourcegraph Cody)** — best for >1000-file repos with semantic queries.
**File-based (Claude Code, Aider, AGENTS.md)** — works for monorepo of medium size with structured docs.
**Bundle (Rapoport Forge / openspec-bundle.ts)** — best for <500 files; one JS module embeds all specs, zero I/O latency.

Studio is in bundle range. Vector embeddings premature until >2000 files or symbol-level nav becomes bottleneck.

### Recommendations for Rapoport Studio

1. **`apps/web/public/llms.txt` + `llms-full.txt`** — entry point for external AI crawlers (Anthropic, Cursor, Vercel, Cloudflare all consume).
2. **`_root/INDEX.md`** as Skills-pattern routing — H1 list of capabilities with ~100-token description each; always-in-context.
3. **Cache breakpoints in Forge architect/builder** — 4-layer architecture above. This is a code task (packages/forge), part of Epic 5.
4. **Incremental indexing** for bundle build — content hash check; reuse previous bundle if openspec/ unchanged.
5. **Sandbox/whitelist** for prompt-injection mitigation — Forge never auto-executes instructions from non-trusted markdown.
6. **No vector embeddings yet** — premature for <500 files.

### Seven principles bridging risk + indexing

1. **Path = state of context.** Knowledge-graph URL routing is both risk control (auditability) and indexing primitive.
2. **Progressive disclosure reduces risk AND optimizes cache simultaneously.** Skills pattern = less hallucination (irrelevant context filtered) + cache-friendly (stable index, dynamic body).
3. **Detection > Prevention for AI code.** FMEA: severity × occurrence × **detection**. Can't fully prevent hallucination — make it cheaply detectable.
4. **Time-decay = caching + risk register.** TTL on cache + freshness on docs; stale ≥ N months → flag in CI.
5. **Multi-provider = uptime AND spec sanity-check.** Two providers via gateway = different code from same spec → divergence is risk signal.
6. **Closest-wins for overrides, later-wins for precedence.** AGENTS.md vs CLAUDE.md — two different principles. Standardize one in repo.
7. **Bundle <1MB, RAG >1GB, vector between.** Size thresholds drive architecture. Studio stays in bundle range; defer vector embedding.

---

## Sources

### Risk frameworks
- [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework)
- [NIST AI 600-1 Generative AI Profile](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf)
- [MITRE ATLAS](https://atlas.mitre.org/)
- [MITRE ATLAS Fact Sheet](https://atlas.mitre.org/pdf-files/MITRE_ATLAS_Fact_Sheet.pdf)
- [OWASP Top 10 for LLM Apps 2025](https://genai.owasp.org/resource/owasp-top-10-for-llm-applications-2025/)
- [OWASP LLM 2025 PDF](https://owasp.org/www-project-top-10-for-large-language-model-applications/assets/PDF/OWASP-Top-10-for-LLMs-v2025.pdf)
- [EU AI Act high-level summary](https://artificialintelligenceact.eu/high-level-summary/)
- [Anthropic Responsible Scaling Policy v3](https://www.anthropic.com/responsible-scaling-policy)
- [Anthropic ASL-3 activation](https://www.anthropic.com/news/activating-asl3-protections)
- [ISO/IEC 42001:2023](https://www.iso.org/standard/42001)
- [Marty Cagan — The Four Big Risks](https://www.svpg.com/four-big-risks/)
- [Asana — RAID Log Guide](https://asana.com/resources/raid-log)
- [ASQ — FMEA Guide](https://asq.org/quality-resources/fmea)

### SDD-specific risks
- [Thoughtworks — Spec-driven development 2025](https://www.thoughtworks.com/en-us/insights/blog/agile-engineering-practices/spec-driven-development-unpacking-2025-new-engineering-practices)
- [Marmelab — SDD: Waterfall Strikes Back](https://marmelab.com/blog/2025/11/12/spec-driven-development-waterfall-strikes-back.html)
- [Unit42 — Indirect Prompt Injection in the Wild](https://unit42.paloaltonetworks.com/ai-agent-prompt-injection/)
- [CodeRabbit — State of AI vs Human Code Generation](https://www.coderabbit.ai/blog/state-of-ai-vs-human-code-generation-report)
- [Universal.cloud — Multi-model fallback strategy](https://universal.cloud/en/blog/ai-uptime-vergeten-risico/)

### Indexing / caching
- [llms.txt official](https://llmstxt.org/)
- [Answer.AI llms.txt proposal](https://www.answer.ai/posts/2024-09-03-llmstxt.html)
- [AGENTS.md spec](https://agents.md/)
- [InfoQ — AGENTS.md emerges as standard](https://www.infoq.com/news/2025/08/agents-md/)
- [Anthropic Skills repo](https://github.com/anthropics/skills/)
- [Anthropic — Equipping agents with Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
- [Anthropic Prompt Caching docs](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)
- [Anthropic Cookbook — Prompt Caching](https://github.com/anthropics/anthropic-cookbook/blob/main/misc/prompt_caching.ipynb)
- [Cursor — Securely indexing large codebases](https://cursor.com/blog/secure-codebase-indexing)
- [Aider — Repository map](https://aider.chat/docs/repomap.html)
- [Sourcegraph — How Cody understands your codebase](https://sourcegraph.com/blog/how-cody-understands-your-codebase)
- [CocoIndex — Real-time codebase indexing](https://cocoindex.io/blogs/index-code-base-for-rag/)
