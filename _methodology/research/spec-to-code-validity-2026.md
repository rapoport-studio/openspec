# Spec-to-code validity — is "manage specs, not code" true? (2026)

> **Status:** complete (snapshot, 2026-05-11)
> **Owner:** Pavel Rapoport (founder).
> **Authors:** Pavel Rapoport <pavel@rapoport.studio> (founder, primary); Claude Opus 4.7 (Anthropic) — web research across SWE-bench Verified vs Pro analyses, CodeRabbit empirical study, Veracode 2025 GenAI security report, Verina paper, Marmelab + Scott Logic critiques, Hillel Wayne formal-methods perspective, ~25 search queries.
> **Method:** Single-pass web synthesis. Sources include benchmark leaderboards (SWE-bench Verified/Pro, HumanEval, Aider polyglot), empirical studies (CodeRabbit Dec 2025, Veracode 2025, Verina arxiv 2505.23135, Vibe Coding study arxiv 2512.11922), critic essays (Marmelab Nov 2025, Scott Logic Nov 2025, Fowler Dec 2025, Simon Willison, DHH), formal-methods perspective (Hillel Wayne TLA+), property-based testing case (Anthropic Red, Kiro). No first-party measurement.
> **Reusable for:** `establish-risk-framework` (Epic 3) — direct input to red-lines.md, trust-gradient.md, risk register top-10. Authorship: open.
> **Opened:** 2026-05-11
> **Feeds into:** Epic 3 (red lines, trust gradient, risk register); Epic 5 (verification gate design in Forge).
> **Companion to:** [`trust-gradient-evolution-2023-2026.md`](./trust-gradient-evolution-2023-2026.md) (temporal version of the same data).

---

## TL;DR

The thesis "manage specs, not code" is **half marketing, half working** — split is by category, not by methodology. CRUD/scaffold: 85–95% spec compliance. Security/concurrent/orchestrator: 30–55%. Universal claim does not survive Fowler 2026: *"neither spec-first nor spec-anchored nor spec-as-source is suitable for the majority of real coding problems."* For Rapoport Studio, risk score for "forget code, manage specs" is **7/10** — high but manageable through spec-anchored mode + mandatory verification layers + 7 hard red lines (security, locks, secrets, RLS, crypto, payments, secret rotation).

---

## Numbers that break the marketing

| Source | Number | What it means |
|---|---|---|
| [SWE-bench Verified vs Pro 2026](https://www.morphllm.com/swe-bench-pro) | Claude Opus 4.5: **80.9% Verified vs 45.9% Pro** | Marketing shows Verified (~80%); production complexity ~46% |
| [OpenAI Verified audit](https://www.morphllm.com/swe-bench-pro) | **59.4% hardest Verified problems had flawed test cases**; all frontier models reproduce verbatim gold patches | Verified-numbers contaminated by training data. Mentally divide by ~1.8× |
| [CodeRabbit State of AI Code 2025](https://www.coderabbit.ai/blog/state-of-ai-vs-human-code-generation-report) | AI PR = **10.83 issues**, human PR = **6.45** (1.7×); **+75% logic errors**; **2.74× security issues**; **8× excessive I/O** | AI makes 1.7× more defects, not fewer |
| [Veracode GenAI 2025](https://www.veracode.com/blog/genai-code-security-report/) | **45% AI samples fail OWASP tests**; Java 72%, XSS defense 14%, log injection defense 12% | Security gap **does not close with model size** |
| [Verina paper, o3](https://arxiv.org/html/2505.23135v3) | o3: 72.6% correct code, **52.3% sound specs, 4.9% proofs** | **LLMs write specs worse than code** — direct hit on "specs first" thesis |
| [Scott Logic empirical, Nov 2025](https://blog.scottlogic.com/2025/11/26/putting-spec-kit-through-its-paces-radical-idea-or-reinvented-waterfall.html) | Spec Kit: 4 hours total; iterative prompting: 32 min | On 1000-LOC feature **SDD was 10× slower** |
| [Mutation testing reveal](https://dev.to/jghiringhelli/the-ai-reported-931-coverage-it-was-34-290k) | **93% line coverage = 3.4% mutation kill rate** | AI writes tests that don't verify what they claim |

## Critique of "code is artifact" thesis — top 7 voices

1. **Marmelab (fzaninotto, "Waterfall Strikes Back", Nov 2025).** Seven failure modes: context blindness, markdown madness, systematic bureaucracy, faux agile, double code review, false security ("agent marked tasks as done without writing a single unit test"), diminishing returns in large codebases. Core: *"SDD tries to solve a faulty challenge: How do we remove developers from software development?"*
2. **Scott Logic (Colin Eberhardt, Nov 2025) — empirical 10× finding.** Same feature: Spec Kit 4h vs iterative prompting 32 min. Bug paradox: *"under SDD philosophy, spec clarification should precede fixes, yet this bug seemed too trivial to warrant rerunning the entire pipeline."* Verdict: *"a return to waterfall...Code is now cheap; we can create it quickly and throw it away just as fast. Spec Kit doesn't capitalise on this."*
3. **HN top opponents** ([47994827](https://news.ycombinator.com/item?id=47994827)). constantcrying: *"The spec locks you into a small niche and iterative changes force enormous complexity to keep spec and code consistent"* — classic spec drift hell. cushychicken: waterfall fails when *"unnecessarily rigid specifications calcify in stakeholders' minds months later."*
4. **Martin Fowler / Birgitta Böckeler (Dec 2025).** Identified three modes: spec-first → spec-anchored → spec-as-source. Direct quote: *"I'm quite sure that neither of them is suitable for the majority of real life coding problems."* Failure modes: false control, hallucination persistence, review fatigue.
5. **Simon Willison.** Despite spec-first advocacy: *"The one thing you absolutely cannot outsource to the machine is testing that the code actually works. Your responsibility as a software developer is to deliver working systems. If you haven't seen it run, it's not a working system."*
6. **DHH (37signals).** vibe coding *"is able to build a veneer...something that looks like it works, but it's flawed in all sorts of ways"*; *"its capacity to get lost in its own labyrinth is very great right now."* Plus the "fingertip rule": *"You learn with your fingers"* — abandoning typing = competence atrophy.
7. **Hillel Wayne (formal methods).** AI *"would naturally propose changes to either the invalid property or the spec"* — model **lies to spec to save code**. And: *"autonomously created properties were either trivial, uninteresting, or too coupled to implementation details."*

## Verification patterns — what actually works

**Property-based testing — strongest current technique.** [Anthropic Red](https://red.anthropic.com/2026/property-based-testing/): PBT agent on Hypothesis achieved **86% validity / 81% reportability** for top-ranked findings. [Kiro](https://kiro.dev/blog/property-based-testing/): *"properties are another representation of (parts of) your specification."*

**Differential testing — underrated.** Mokav: detect functional differences between two programs — **81.7% success rate** vs 4.9% Pynguin. Practical: run two LLMs on same spec, diff outputs — conflicts = risk hotspots.

**Formal methods + AI (TLA+, Alloy, Lean) — niche but valid.** Hillel Wayne: LLMs handle TLA+ syntax correction well. Limit: **don't trust AI proposing changes to spec in response to bugs** — path to masked errors.

**Contract testing (Pact, OpenAPI) — best near-term ROI.** PactFlow Drift: AI test maintenance reported "60% faster test creation."

**Mutation testing — must-have for AI tests.** 93% line coverage = 3.4% mutation kill rate is documented. Stryker for JS/TS, PIT for Java.

**Overhyped:** BDD/Cucumber as primary spec, "living documentation auto-generated from code", pure type-driven.
**Underrated:** differential testing two-model + diff, mutation testing on AI tests before trusting coverage, PBT on core invariants.

## Trust gradient (current state) — where SDD works, where doesn't

| Domain | Current spec compliance | Honest take for Rapoport |
|---|---|---|
| CRUD endpoints / scaffold | **85–95%** | RAP-362–368 phases proved this — Forge works |
| UI components (single-file) | 80–90% | `packages/ui` additions, lint+typecheck catches 95% |
| Pure business logic | 60–80% | Stage check functions OK |
| DB migrations (explicit schema) | 70–85% | Caveat: always pair with RLS in same migration |
| Multi-file refactor (4+) | **40–55%** | SWE-bench Pro zone; phase-split mandatory |
| RLS / auth boundaries | **<55%** | Veracode 45% fail rate; never auto-ship |
| Cloud platform context (CF Workers) | 40–60% | Training data thin; AI hallucinates `process.env` |
| Concurrent / orchestrator | **30–50%** | Hillel Wayne: AI suggests "race conditions are okay". Memory documents this in `forge_builder_orchestrator_commit_race` |
| Crypto / payment | <50% | Hard red line |

## Synthesis for Rapoport Studio

**Reformulate the thesis (drop marketing):**
*"Manage specs **and** code. Spec drives generation; code drives trust. Spec is the plan AI executes; code is the contract with production a human signs."*

This is **spec-anchored mode** (Thoughtworks/Fowler), not **spec-as-source** (Tessl marketing).

### 8 risk-register entries

| # | Risk | Mitigation |
|---|---|---|
| 1 | Spec drift between proposal and merged code | Quarterly grep-audit on 5 random archived changes |
| 2 | AI confidently writes wrong concurrent primitive | Any PR touching `~/.forge/locks/`, `state.json`, process management → mandatory manual review |
| 3 | Auto-generated tests don't verify what they say | Mutation testing (Stryker) on Forge test suites quarterly; threshold 70% kill rate for critical paths |
| 4 | Context overflow on 4+ files | If task touches >4 files → phase split mandatory in tasks.md |
| 5 | SpecFall: review fatigue | `design.md` hard cap 2000 words; longer = split change |
| 6 | AI lies to spec to save code (Wayne) | Verifier flags any tasks.md done without assertions/tests |
| 7 | Security policies generated, not reviewed | All Supabase RLS policies + CF secrets + auth code → manual review checkbox |
| 8 | Fingertip rot (DHH) | 20% rule: minimum 1 PR per week written by hand, no Forge |

### 7 hard red lines — never via `forge build`

1. Anything in `~/.forge/locks/`, `state.json`, process orchestration
2. Supabase RLS policies
3. Cloudflare Worker secrets handling
4. Auth flows / session management
5. Crypto primitives
6. Payment / billing flows (when they appear)
7. Migration scripts with data transformation (no undo)

These become `methodology/red-lines.md` in Epic 3 + lint rule in Forge: PR touching listed paths requires explicit `manual-review` label.

---

## Sources

- [Spec-Driven Development: Waterfall Strikes Back — Marmelab](https://marmelab.com/blog/2025/11/12/spec-driven-development-waterfall-strikes-back.html)
- [Putting Spec Kit Through Its Paces — Scott Logic](https://blog.scottlogic.com/2025/11/26/putting-spec-kit-through-its-paces-radical-idea-or-reinvented-waterfall.html)
- [CodeRabbit State of AI vs Human Code Generation](https://www.coderabbit.ai/blog/state-of-ai-vs-human-code-generation-report)
- [The Register — AI-authored code needs more attention (Dec 2025)](https://www.theregister.com/2025/12/17/ai_code_bugs/)
- [HN thread on Marmelab SDD article](https://news.ycombinator.com/item?id=45935763)
- [Hillel Wayne — AI is a gamechanger for TLA+ users](https://buttondown.com/hillelwayne/archive/ai-is-a-gamechanger-for-tla-users/)
- [Simon Willison — How I use LLMs to help me write code](https://simonw.substack.com/p/how-i-use-llms-to-help-me-write-code)
- [Veracode GenAI Code Security Report 2025](https://www.veracode.com/blog/genai-code-security-report/)
- [Vibe Coding in Practice (arxiv 2512.11922)](https://arxiv.org/abs/2512.11922)
- [SWE-Bench Pro Leaderboard analysis](https://www.morphllm.com/swe-bench-pro)
- [Verina: Benchmarking Verifiable Code Generation (arxiv 2505.23135)](https://arxiv.org/html/2505.23135v3)
- [Property-Based Testing with Claude — Anthropic Red](https://red.anthropic.com/2026/property-based-testing/)
- [Kiro: Does your code match your spec?](https://kiro.dev/blog/property-based-testing/)
- [Mutation Testing exposes 3.4% kill rate at 93% coverage](https://dev.to/jghiringhelli/the-ai-reported-931-coverage-it-was-34-290k)
- [Martin Fowler / Böckeler — Understanding SDD (Kiro/spec-kit/Tessl)](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html)
- [Pragmatic Engineer — DHH's new way of writing code](https://newsletter.pragmaticengineer.com/p/dhhs-new-way-of-writing-code)
- [Tech Startups — Vibe Coding Delusion](https://techstartups.com/2025/12/11/the-vibe-coding-delusion-why-thousands-of-startups-are-now-paying-the-price-for-ai-generated-technical-debt/)
