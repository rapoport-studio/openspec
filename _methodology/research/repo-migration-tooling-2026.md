# Repo Migration & Onboarding Tooling — May 2026

> Research file for the Forge + Muse + Maestro migration/onboarding capability.
> Question: how are codebase-onboarding and repo-to-spec migrations done in 2025-2026? What tools, patterns, services, and pitfalls exist?

## TL;DR

Every serious 2025-2026 codebase-onboarding system converges on the **same three-stage pipeline**: (1) *cheap structural pass* (tree-sitter / AST-aware chunking + a Merkle-tracked file index), (2) *agentic ReAct exploration loop* (search → read → think → search, with sub-agents fanning out per concern), and (3) *retrieval-augmented synthesis* (LLM only sees curated chunks, never the raw dump). The naïve "stuff the whole repo into the context window" pattern (Repomix-only) is now considered an antipattern for anything past ~50k LOC — every production system (Cursor, Augment, Cody, Copilot Workspace, Genie) explicitly rejects it.

For code-to-spec, the bleeding edge is **SpecRover / AutoCodeRover-v2** (academic, ICSE 2025) — iterative spec inference *during* a repair workflow rather than as a one-shot pass. For SDD frameworks, **Spec Kit's "Brownfield Bootstrap" extension** (community, late 2025) and **OpenSpec's `/opsx:onboard`** are the closest public analogues to what Rapoport needs; both still leave the hard problems (capability boundaries, risk inventory, glossary mapping) to the operator. **Kiro's three-file pattern** (`product.md` / `tech.md` / `structure.md`) is the cleanest "first dump" format we've seen and is worth stealing wholesale as a Stage 0 artifact before producing OpenSpec specs.

The single biggest documented failure mode is *confident hallucination of invariants* — Daikon-style "looks-true-on-N-traces" claims that the model then promotes to spec language. Every recent postmortem (the Django monolith case, the payment API doc case, slopsquatting at large) has the same shape: model writes plausible spec → human reviews under "looks right" bias → constraint that was never true gets baked into the spec → next agent breaks something invisible.

---

## Layer 1: Bundling / flattening (raw context)

The "compile the repo into one file" layer. Cheap, dumb, and the right starting point only if the repo is small enough to fit a frontier model's context window with room to spare.

### Repomix (yamadashy/repomix)
- **What**: Node.js CLI + library. Packs a repo into one AI-friendly file (XML / Markdown / JSON / plain). Token counting per file and total. Built-in Secretlint pass to strip secrets. Respects `.gitignore` / `.repomixignore`. Available as MCP server.
- **State (May 2026)**: Active. Recent releases focus on startup speed and concurrent pipeline stages. ~30k+ GitHub stars (unverified, claim from project README rank — verify before citing).
- **Limit**: Output is monolithic; on the Next.js repo it produced ~22 minutes of work vs yek's 5 seconds — slow on large monorepos.
- **Pick for Rapoport**: Use as the *small-repo escape hatch* and for the secret-scanning side effect. Do not make it the primary pipeline.
- Source: <https://github.com/yamadashy/repomix>, <https://repomix.com/>

### gitingest (coderamp-labs/gitingest)
- **What**: Web service + CLI + Python library + browser extensions. Swap `github.com` → `gitingest.com` in any URL to get a flat digest. Zero telemetry, runs offline.
- **State**: Active. Multiple forks; `coderamp-labs/gitingest` is the most-maintained line.
- **Limit**: Web service caps at moderate repo size; private repos require self-hosting. Generates ~20% more tokens than Repomix on the same Python-docs-samples corpus (69M vs 56M on a single benchmark — see below).
- **Pick**: Useful for the *URL-pasteable demo* moment. Not the production pipeline.
- Source: <https://github.com/coderamp-labs/gitingest>, <https://gitingest.com/>

### yek (bodo-run/yek)
- **What**: Rust CLI. Fast serializer. Uses git history to *infer file importance* and auto-detects binary/large files. Streams when piped.
- **State**: Active. v0.21.0 in current line. Faster than Repomix by ~250x on the Next.js codebase per the project's own benchmark (5.19s vs 22.24min — vendor-reported, but plausible).
- **Limit**: Less ecosystem (no MCP server, no Secretlint integration). Smaller community.
- **Pick**: If we ever need Rust-speed bundling in a CI hot path. Not necessary for our scale today.
- Source: <https://github.com/bodo-run/yek>

### files-to-prompt (simonw/files-to-prompt)
- **What**: Python CLI. Concatenate a directory into one prompt. `-m/--markdown` mode fences each file. Reads file lists from stdin (pipe friendly).
- **State**: v0.6 (Feb 2025). Maintained by Simon Willison as part of his `llm` toolchain.
- **Limit**: Minimalist. No token counting, no semantic ranking, no secret scrubbing.
- **Pick**: Use inside ad-hoc shell pipelines (`fd | files-to-prompt | llm`). Not a primary pipeline component.
- Source: <https://github.com/simonw/files-to-prompt>

### code2prompt (mufeedvh/code2prompt)
- **What**: Rust CLI. Source tree + prompt templating (Handlebars) + token counting + git diff/log/branch comparison. Configurable via `.c2pconfig`. Has an MCP server.
- **State**: Active, Rust-fast. Used in agent-builder pipelines.
- **Limit**: Like the others — flat output, no structural ranking.
- **Pick**: Worth keeping for the **prompt templating** alone. The Handlebars layer is closer to what we'll want for parameterized capability extraction.
- Source: <https://github.com/mufeedvh/code2prompt>

### Benchmark snapshot (one data point, May 2025)
On `python-docs-samples`: Repomix = 56,045,437 tokens; code2prompt = 57,160,552 tokens; gitingest ≈ 69.2M tokens. Same repo, ~25% spread — **the choice of bundler materially changes context budget**. Source: 16x prompt CLI tools collection (treat as one anecdote, not gospel).

### Claude Code / Anthropic's actual startup behavior (for grounding)
On `claude` invocation in a project dir: ~2,900 base system prompt tokens, then `CLAUDE.md` (project + parent + user), then 18+ tool definitions, then the conversation. Automatic compaction triggers at ~92-95% context. The Tool Search pattern (deferred tool loading) is the official escape valve — relevant for our orchestra because **the same pattern applies to "deferred capability spec loading"**: don't dump all specs, search them. Source: <https://www.anthropic.com/engineering/claude-code-best-practices>; <https://codewithmukesh.com/blog/anatomy-claude-code-session/>.

### Layer 1 takeaway
Bundling is a tactic, not a strategy. Past ~30k LOC, every serious tool moves to Layer 2. **For Rapoport: Repomix for the secret-scan + small-repo path; code2prompt's Handlebars layer for templated prompts; everything else handled by Layer 2 + 3.**

---

## Layer 2: AST / structural understanding

Once a repo is too big to flatten, you need *structure*. Three families: tree-sitter parsers, structural search/rewrite engines, and code-intelligence indexes.

### Tree-sitter
- **What**: Parser-generator framework producing concrete syntax trees (CSTs). 130+ language grammars. Used by Cursor, Aider, ast-grep, GitHub Codespaces, every modern IDE-side index.
- **Why it won**: Incremental reparse (sub-millisecond on edit), no language-server dependency, deterministic.
- **Pick for Rapoport**: The substrate. Every other Layer 2 tool we'd use sits on it.
- Source: tree-sitter is the obvious choice; well-documented across the ecosystem.

### ast-grep (ast-grep/ast-grep)
- **What**: Rust CLI for structural search, lint, and rewrite. Patterns are "code-isomorphic" (you write the pattern as if it were code with `$VAR` metavariables). YAML rule files for lint policies. Backed by tree-sitter.
- **State**: Active. Codemod 2.0 (see Layer 5) is built on ast-grep's NAPI bindings.
- **Pick**: **Strong yes** — this is our primary structural query engine. Use it for the "find all auth boundaries / RLS policies / payment endpoints" sweep that produces the red-line inventory.
- Source: <https://github.com/ast-grep/ast-grep>, <https://ast-grep.github.io/>

### Comby (comby-tools/comby)
- **What**: OCaml-based structural search/replace, language-agnostic via parser combinators. Older (pre-tree-sitter renaissance) but elegant for cross-language patterns.
- **State**: Maintained but slower release cadence in 2025. Largely superseded by ast-grep for most workflows.
- **Pick**: Skip for greenfield work. Keep in the toolbox for languages where ast-grep's tree-sitter grammar is weak.
- Source: <https://github.com/comby-tools/comby>, <https://comby.dev/>

### Semgrep (semgrep/semgrep)
- **What**: Pattern-based static analysis with taint mode. 4,000+ rules. Now bundled with an "LLM-security" skillset that maps to OWASP LLM Top 10 (2025 edition). "Semgrep Multimodal" (March 2026) combines deterministic rules + LLM reasoning.
- **State**: Active, commercial backing, free CLI + community rules.
- **Pick**: **Strong yes** for Layer 7 (risk inventory). Also useful in Layer 2 as a "find every place that calls the database without going through the RLS-aware client" probe.
- Source: <https://semgrep.dev/p/owasp-top-ten>, <https://semgrep.dev/blog/2026/owasp-top-10-2025-whats-new/>, <https://www.helpnetsecurity.com/2026/03/20/semgrep-multimodal-code-security/>

### SCIP / Sourcegraph indexers
- **What**: SCIP (Sourcegraph Code Intelligence Protocol) — protobuf-based successor to LSIF. 8x smaller, 3x faster to process. `scip-typescript`, `scip-java`, `scip-python` all available. Powers go-to-definition / find-references at scale.
- **State**: Sourcegraph migrated entirely off LSIF as of 4.6 (2024). SCIP is the industry standard for precise code intelligence dumps in 2026.
- **Pick**: **Conditional yes.** Worth running for any repo above ~100k LOC — gives the orchestra a precise "what calls what" graph without paying LLM tokens. For smaller repos, tree-sitter + ast-grep is sufficient.
- Source: <https://github.com/sourcegraph/scip>, <https://sourcegraph.com/blog/announcing-scip>

### Aider's repomap algorithm (worth stealing)
- **What**: Tree-sitter pulls symbol defs + refs across the repo. NetworkX PageRank ranks the graph with personalization from chat context. Binary search trims to a token budget (default 1k). 130+ languages.
- **Why it matters**: This is the *cleanest open-source implementation* of "what should I send the model when it asks about file X" — and the algorithm is small enough to port. <300 lines of Python.
- **Pick**: **Steal the algorithm.** Build a Rapoport-flavored repomap as the persistent index that sits between Layer 1 and Layer 3 in our pipeline.
- Source: <https://aider.chat/2023/10/22/repomap.html>, <https://aider.chat/docs/repomap.html>, <https://github.com/Aider-AI/aider/blob/main/aider/repomap.py>

### Layer 2 takeaway
The convergent stack is **tree-sitter + ast-grep + (SCIP for big repos) + Aider-style PageRank repomap**. Comby is fading. Semgrep is for Layer 7 but doubles as a Layer 2 probe.

---

## Layer 3: LLM-based code summarization

Where structure meets prose. Three patterns in production:

### RepoAgent (OpenBMB/RepoAgent) — EMNLP 2024 demo
- **What**: LLM framework that proactively generates and maintains repo-level docs. Three stages: global structure analysis → documentation generation → documentation update. Git hook fires on commit to refresh stale docs.
- **Result**: Blind preference tests preferred RepoAgent docs over human docs at 70% (Transformers) and 91.33% (LlamaIndex). Take with salt — preference tests are easy to game and prefer the "cleaner, more structured" output regardless of accuracy.
- **Pick**: Read the paper for the *staged-update pattern*. Don't adopt the framework — it's research-grade plumbing and won't survive contact with Cloudflare Workers + Anthropic SDK.
- Source: <https://arxiv.org/abs/2402.16667>, <https://github.com/openbmb/repoagent>

### Mintlify auto-docs
- **What**: Web service. Replace `github.com` → `mintlify.com`. Scrapes README + metadata, infers entry points/exports/commands, then spins up *parallel sub-agents* per doc section. Excalidraw: ~70min → ~45min via parallelization.
- **Pattern worth stealing**: One orchestrator, N parallel section-writer sub-agents, post-hoc cross-link validation.
- **Pick**: Don't adopt the SaaS. Steal the orchestrator-fanout shape for our capability-spec generation pass.
- Source: <https://www.mintlify.com/blog/auto-generate-docs-from-repos>, <https://www.mintlify.com/blog/docs-on-autopilot>

### Augment Code Context Engine
- **What**: Real-time semantic index of the entire codebase + commit history + cross-repo deps + architectural patterns. Compresses context without losing fidelity. Claims +70% agent performance across Claude Code / Cursor / Codex.
- **Architecture**: Not "dump the codebase" — *retrieve only what matters*. Lives behind a Context Engine MCP server so any agent can consume it.
- **Pick**: This is **the architectural reference** for what Maestro should look like inside the orchestra. The MCP-server-as-context-broker pattern maps 1:1 to our "agents read specs through Maestro" model.
- Source: <https://www.augmentcode.com/context-engine>, <https://www.augmentcode.com/blog/context-engine-mcp-now-live>

### Cursor's indexing
- **What**: AST-based chunking (functions, classes, or ~500-token blocks, with sibling-merge). Merkle tree of file hashes synced to server. Embeddings via OpenAI/proprietary model into Turbopuffer vector DB. Path is obfuscated; original source never leaves the laptop.
- **Pattern**: Merkle-tree differential sync. **This is the right design** for an offline-first onboarding orchestra — only re-embed what changed.
- **Pick**: Steal the Merkle pattern; don't run a vector DB if we can avoid it (Aider-style PageRank gets us 80% of the value with no infra).
- Source: <https://cursor.com/blog/secure-codebase-indexing>, <https://read.engineerscodex.com/p/how-cursor-indexes-codebases-fast>

### Cody (Sourcegraph)
- **What**: RAG over Sourcegraph's Code Graph + multi-repo search (up to 10 repos). Three context layers: editor / local repo / remote repo. 1M-token windows on Sonnet 4.
- **State (July 2025)**: Cody Free and Pro discontinued. Enterprise-only.
- **Pick**: Reference architecture; not adoptable.
- Source: <https://aiwiki.ai/wiki/sourcegraph_cody>

### GitHub Copilot Workspace / `#codebase`
- **What**: Semantic search via a proprietary code-optimized embedding model. The agent picks its own search tools per query and chains them. CLI variant supports natural-language repo Q&A.
- **Pattern**: *Agent picks the search tool*, not a fixed pipeline. Each query may use ripgrep, semantic search, definition lookup, or all three.
- **Pick**: Confirms the multi-tool-per-query pattern is industry-wide. Our orchestra needs this too.
- Source: <https://github.com/orgs/community/discussions/142971>, <https://code.visualstudio.com/docs/copilot/reference/workspace-context>

### Cosine Genie
- **What**: Custom IDE-environment-of-one. Trained on forensically-reconstructed developer reasoning trajectories, not just code-to-code pairs. Genie 2 ships with a native execution sandbox.
- **Insight**: The hardest problem in onboarding is *prerequisite-information discovery* — what does the agent need to know *before* it can do task X? Cosine's answer: train on the search trajectories of humans doing X.
- **Pick**: Take the insight. We don't need their training pipeline, but Maestro should be designed so its **search trajectories are recorded** and can be replayed/studied.
- Source: <https://cosine.sh/blog/genie-technical-report>

### AutoCodeRover / SpecRover (academic, ICSE 2024-2025)
- **What**: AST-level code search (vs filename search). Stratified retrieval — model issues multiple search-API calls in one round. SpecRover (a.k.a. AutoCodeRover-v2) adds *iterative specification inference* during the repair loop and a reviewer agent that reconciles spec + tests + NL requirements.
- **Result**: 50%+ improvement over AutoCodeRover on full SWE-Bench (2,294 issues), $0.65/issue on SWE-Bench-lite.
- **Pick**: **This is the closest academic analog to what we want.** Read the SpecRover paper in detail before we lock the architecture — the "spec inferred as a byproduct of doing a task" pattern is more powerful than "produce spec as a pre-task pass." Strong implication: don't try to produce the spec up front; let it accrete as the orchestra does real work.
- Source: <https://arxiv.org/abs/2408.02232>, <https://arxiv.org/pdf/2404.05427> (AutoCodeRover original)

### Layer 3 takeaway
**Pattern convergence is total.** Every production system uses: (a) AST-aware chunking, (b) some form of differential sync (Merkle / git hashes), (c) retrieval (semantic + structural) instead of dump-the-repo, and (d) sub-agent fanout for parallelism. Augment + Cursor are the architecture references; SpecRover is the academic reference for inferring spec-shaped artifacts.

---

## Layer 4: Code-to-spec reverse engineering

The most specific need for Rapoport, and the layer where everyone is still wrong.

### Daikon — dynamic invariant detection (Univ. Washington)
- **What**: Runs the program, observes values across executions, reports "likely invariants." 5.8.22 released June 2025, so still maintained, but the underlying limits haven't moved in 15 years.
- **Why it's a cautionary tale**: Generates spurious invariants that fit observed traces but don't generalize. *Correct likely invariants are lost in a mass of incorrect ones.* Requires user-supplied splitting conditions for disjunctive cases.
- **Relevance**: The failure mode is exactly the failure mode of *LLM-inferred specs*. Read the Daikon postmortems before letting an LLM emit declarative spec language.
- **Pick**: Don't run Daikon. Read its known-failure cases. Use them as the test set for our spec-inference reviewer.
- Source: <https://plse.cs.washington.edu/daikon/>, <https://plse.cs.washington.edu/daikon/download/doc/daikon.pdf>

### LLM-as-spec-reverser academic landscape (2025)
- **Frontiers (July 2025) — LLM vs model-driven reverse engineering to UML**: Off-the-shelf LLMs are unreliable for OCL specs (hallucinations, simplifying assumptions, overly abstract). Fine-tuning brings them to parity with rule-based methods.
- **arXiv 2502.07835 — Reverse generation from LLM-generated code**: Defines the SBC metric (semantic-behavioral consistency between original requirement and code) to detect when reverse-generated requirements drift.
- **Takeaway**: Untuned frontier models *will* hallucinate plausible-but-wrong spec language. The state of the art is fine-tuning on (code, spec) pairs.
- Source: <https://www.frontiersin.org/journals/computer-science/articles/10.3389/fcomp.2025.1516410/full>

### OpenAPI generators from code (Swashbuckle, NestJS Swagger, fastapi-openapi)
- **What**: Reflection-driven extraction of API surface → OpenAPI spec.
- **Strength**: Deterministic. The API surface *is* the spec.
- **Weakness**: API-only. Tells you nothing about business invariants, RLS, payment flow.
- **Pick**: Run them and use the output as a *trusted bedrock layer* the LLM can't override. The OpenAPI doc is "ground truth" for endpoints; the LLM is only allowed to add prose context.

### "Prose spec from a codebase" — the gap
There is **no general-purpose AI-native tool** in 2026 that produces high-quality prose specifications from a codebase. SpecRover (above) is the closest, but it produces spec fragments *in service of a repair task*, not standalone capability docs. RepoAgent produces docs but not specs. Mintlify produces user-facing docs, not internal contracts.

**This is the gap Rapoport's orchestra is filling.** Worth noting in any external positioning.

### Layer 4 takeaway
The serious approach for Rapoport: (1) extract everything deterministically extractable first (OpenAPI, DB schema, RLS policies, env-var lists, package boundaries). (2) Run the LLM only on what's left, and (3) gate every LLM-produced spec line with a *reviewer agent that demands evidence* (file:line, test, or external doc). Adopt SpecRover's reviewer-reconciles-spec-tests-NL pattern explicitly.

---

## Layer 5: Refactoring / migration agents

### Codemod 2.0 (codemod/codemod)
- **What**: CLI for scaffolding, sharing, running multi-step transformations. First-class ast-grep support. Declarative codemods in YAML; imperative when needed; workflows orchestrate multi-step with matrix strategies and approval gates. Community Registry.
- **Pick**: Look at the workflow YAML format — it's the closest existing schema to what Forge's "wave" model wants. Don't adopt their engine, but borrow the workflow shape.
- Source: <https://github.com/codemod/codemod>, <https://codemod.com/blog/codemod2>

### OpenRewrite (Moderne)
- **What**: Java/JVM (now multi-language) refactoring engine. Lossless Semantic Tree (LST). Recipes are composable, versioned, and can be combined into "migrations." Java 8 → 11 → 17 → 21 → 25 paths all open-source.
- **State**: Moderne's "Moddy" multi-repo AI agent (2025) sits on the LST and uses it as the tool surface for an LLM.
- **Pick**: The LST + recipe model is the most mature "structured code transformation" pattern in production. Worth studying even if we don't adopt — directly informs how we should shape Forge's migration recipes.
- Source: <https://docs.openrewrite.org/>, <https://www.moderne.ai/blog/introducing-moderne-multi-repo-ai-agent-for-transforming-code-at-scale>

### jscodeshift (Facebook)
- **What**: JS/TS codemod toolkit on top of recast. Standard for React/Next/AI-SDK migrations. Recent trend: GitHub Copilot generates jscodeshift codemods on demand.
- **Pick**: Use as the JS/TS lower-level transform engine when ast-grep isn't expressive enough.
- Source: <https://github.com/facebook/jscodeshift>

### Sourcegraph Batch Changes
- **What**: Run a script across N repos → N PRs → unified dashboard tracks each to merge. Bulk rebasing in 2025. Workiva reports 80% time reduction on large-scale changes; Nine reports $276K/yr savings.
- **State**: Sourcegraph signaling that AI-driven Batch Changes is on the roadmap (Cody + agents + Batch Changes converging).
- **Pick**: Out of scope for the first onboarding — but the *fan-out → track → merge* model is exactly what Forge's epic-build flow already does. Confirms the design.
- Source: <https://sourcegraph.com/batch-changes>, <https://sourcegraph.com/blog/batch-changes-is-better-than-ever>

### Devin (Cognition)
- **What**: Autonomous SWE agent. In 2026 marketing materials, advertises COBOL/Fortran/Objective-C → Rust/Go/Python migrations. Nubank case study: 12x efficiency, 20x cost savings on Data/Collections/Risk migrations.
- **Caveats**: "Performs best on well-defined tasks, struggles with large/ambiguous systems, requires human oversight." Vendor numbers; treat as upper bound.
- **Pick**: Out of scope; reference point only.
- Source: <https://devin.ai/>, <https://www.wwt.com/blog/part-2-devin-autonomous-ai-for-modernization>

### Cosine Genie 2
- See Layer 3. Genie's onboarding-aware training is the closest thing to "agent that *learns* the codebase before doing work."

### Layer 5 takeaway
We don't need a full refactoring engine for the first migration capability. We need (a) ast-grep as the structural query tool, (b) a Codemod-style workflow YAML for multi-step transforms, (c) Forge as the orchestrator we already have. Stay away from Devin/Genie scope creep — they are *replacement IDE products*, not modules.

---

## Layer 6: SDD framework migration (between specs)

### GitHub Spec Kit — Brownfield Bootstrap
- **What**: Community extension (PR/issue 1436 + `wcpaxx/spec-kit-brownfield-extensions` + Quratulain-bilal fork) addressing the "Spec Kit's `specify init` creates generic templates that don't reflect the actual architecture" problem.
- **Approach**: Auto-analyse project, generate a Constitution from observed conventions, generate spec templates with project-specific sections, generate task templates with the real test/build commands.
- **State (May 2026)**: Still a community add-on, not first-class in `github/spec-kit`. Multiple competing forks. EPAM published a public walkthrough using it for brownfield SDD.
- **Pick**: **Read the extension's prompts** — they encode community wisdom about what to discover first (test framework, package manager, lint config, README sections). We'll want our own version. The fragmentation is a warning: this is not a solved problem in the public ecosystem.
- Source: <https://github.com/wcpaxx/spec-kit-brownfield-extensions>, <https://github.com/github/spec-kit/issues/1436>, <https://github.com/github/spec-kit/discussions/331>, <https://www.epam.com/insights/ai/blogs/using-spec-kit-for-brownfield-codebase>

### AWS Kiro — three-file "steering" pattern
- **What**: On opening a project, Kiro auto-generates three markdown files: `product.md` (what the repo does), `tech.md` (frameworks/libs/tools), `structure.md` (project organization). These act as the persistent context for every subsequent agent invocation.
- **Why it's good**: Three files. Bounded. Re-readable in <2k tokens. Solves the "where do I even start" problem for any new agent in the repo.
- **Pick**: **Steal the three-file pattern as a Stage 0 artifact** before producing OpenSpec capability specs. Map: Kiro's `product.md` → Rapoport's `openspec/current/platform.md`. Kiro's `tech.md` → an auto-generated `openspec/current/stack.md`. Kiro's `structure.md` → an auto-generated `openspec/current/repo-map.md`.
- Source: <https://kiro.dev/>, <https://aws.amazon.com/startups/prompt-library/kiro-project-init>

### BMad Method
- **What**: 21 AI personas (Analyst, Architect, Developer, etc.), 50+ workflows. Established-projects guide acknowledges "for large existing codebases, SDD is often impractical to introduce retroactively" — recommends SDD for new work, traditional for legacy maintenance.
- **Pick**: Skip the persona machinery (we have our own role split). But the *honesty about brownfield difficulty* is unusual and worth respecting. Don't promise full retroactive SDD; promise *incremental SDD on new capabilities, lightweight reverse-docs on existing ones*.
- Source: <https://docs.bmad-method.org/how-to/established-projects/>

### Tessl — spec-as-source
- **What**: Aspires to "spec is the source, generated code is marked DO-NOT-EDIT." `tessl build` regenerates code from spec. Has an `Existing code to spec` workflow.
- **Honest assessment from Martin Fowler's "Understanding SDD" piece**: "None of the tools show long-running evidence for safely introducing specs into existing, entangled codebases (how to avoid duplication, regressions, or dangling generated files); the practical migration strategy and cost are still mostly hand-wavy."
- **Pick**: Watch the space. Spec-as-source is the *next* generation; we are building for the *prior* generation (spec-as-checked-living-document). Don't try to skip a stage.
- Source: <https://docs.tessl.io/common-workflows/existing-code-to-spec>, <https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html>

### OpenSpec native — `/opsx:onboard`
- **What**: OpenSpec's own onboarding command. Walks the workflow against the user's actual codebase: pick a small improvement → create a change → implement → archive. For mature codebases, "generates initial specs from your codebase."
- **State**: Bundled with OpenSpec CLI.
- **Pick**: **Read the prompt source.** This is the closest existing pattern to what we're building, and we already use OpenSpec. Likely worth wrapping/extending rather than replacing.
- Source: <https://openspec.dev/>, <https://github.com/Fission-AI/OpenSpec>

### Layer 6 takeaway
**The Kiro three-file pattern + Spec Kit brownfield bootstrap prompts + OpenSpec's own `/opsx:onboard`** are the three reference implementations. None of them solve our problem fully. The convergent pattern is *generate the smallest useful artifact first* (Kiro's 3 files, or a one-page PRD à la BMad), then iterate.

---

## Layer 7: Risk / security scanning

For the red-line inventory step.

### Semgrep (semgrep/semgrep)
- See Layer 2. `p/owasp-top-ten` ruleset; LLM-specific skillset for OWASP LLM Top 10 2025 (prompt injection, sensitive info disclosure, RAG output filtering, etc.); MITRE ATLAS + NIST AI RMF coverage. Multimodal (rules + LLM reasoning) launched March 2026.
- **Pick**: **Mandatory**. Run as the first risk pass on every onboarded repo.
- Source: <https://semgrep.dev/p/owasp-top-ten>, <https://github.com/semgrep/semgrep-rules>

### Gitleaks + TruffleHog
- **Gitleaks**: Regex-based, fast (~ms pre-commit hook), git-focused.
- **TruffleHog**: 800+ secret types across git + S3 + Docker images + Slack + Jenkins; credential verification (actually attempts authentication to confirm a secret is live, eliminating revoked-credential false positives); slower because of the network calls.
- **Pattern in industry**: Most security teams run *both* — Gitleaks pre-commit, TruffleHog in CI.
- **Pick**: Run **both** in the onboarding pipeline. Gitleaks for speed, TruffleHog because credential-verification gives us actionable risk (vs noise).
- Source: <https://github.com/trufflesecurity/trufflehog>, <https://www.jit.io/resources/appsec-tools/trufflehog-vs-gitleaks-a-detailed-comparison-of-secret-scanning-tools>

### CodeQL
- **What**: Semantic queries over a built code DB. Cross-file, cross-function taint analysis. 400+ CVEs identified via variant analysis (GitHub Security Lab).
- **License**: Engine is **not** open-source. Free only for research / open-source. Private code requires GitHub Advanced Security.
- **Pick**: Not the first tool for us (license + setup cost). Keep in the toolbox for when an onboarded repo has a paid GHAS license and we can leverage it.
- Source: <https://codeql.github.com/>, <https://github.com/github/codeql>

### Snyk Code
- **What**: SAST + SCA CLI. ~60-90s scans on 90k-LOC services. PR-friendly. Recent: package health checks, agent-scan repo for AI-agent-specific risks.
- **Pick**: Worth running if a client already has a Snyk subscription. Not necessary for the first internal pipeline.
- Source: <https://github.com/snyk/cli>, <https://github.com/snyk/agent-scan>

### Layer 7 takeaway
**Mandatory CLI stack: Semgrep + Gitleaks + TruffleHog.** All three are free, all three run on a laptop, all three produce machine-readable output we can feed back into the spec pipeline as "things the human has to look at before we promote any of this to capability spec." CodeQL and Snyk are conditional.

---

## Patterns & pitfalls from real stories

1. **Hallucinated package names ("slopsquatting")** — A 2025 study found 20% of LLM-recommended packages don't exist; 43% of the hallucinated names repeat across queries. Threat actors register the names. *Implication*: spec-extraction must check every imported package against a registry before promoting to spec. Source: <https://snyk.io/articles/slopsquatting-mitigation-strategies/>

2. **The Django monolith case (Apr 2026, Medium / Friday Deploy)** — An engineering lead let Claude refactor an 8-year Django monolith. "Clean, confident patches that quietly broke integrations with external services." The model couldn't see the *implicit promises* (customer contracts, audit invariants, partner-system expectations). *Implication*: an onboarding agent must treat every external integration as a red-line until a human has confirmed the contract is documented. Source: <https://thefridaydeploy.substack.com/p/ai-cant-handle-your-legacy-codebase>

3. **The payment API hallucination** — Developer asked an LLM to generate docs for a payment API. Output looked perfect. The model had invented endpoints, parameters, responses. The API didn't exist. *Implication*: every endpoint claim in a generated spec must trace to a file:line. No file:line, no spec line.

4. **Apiiro (Sept 2025): AI-gen code introduced >10k new security findings/month, +322% privilege escalation paths, +153% arch design flaws over 6 months.** *Implication*: any code Forge/Muse/Maestro generates needs the same Layer 7 gate as the original repo — not just at PR time but at *spec promotion* time.

5. **Chatbot leaked 1,247 PII records (March 2024 postmortem)** — 82% of leaks came from raw, unmasked customer context injected into the LLM prompt. *Implication*: when our orchestra reads a repo, it will encounter test fixtures and seed data. We must strip PII at the bundling layer (Repomix's Secretlint pass, or a custom mask) before any agent sees them.

6. **Daikon's spurious-invariant problem** — 15+ years of research and the core failure ("plausible invariants that don't generalize, drowned out by ones that do") still hasn't been solved. LLMs *will* repeat this failure unless we explicitly guard against it.

7. **Cosine's prerequisite-information lesson** — The hardest problem is "what does the agent need to know before it can do X?" Train on *human search trajectories*, not just on (problem, solution) pairs. *Implication for Maestro*: log every search the orchestra performs, surface those logs as part of the spec ("this capability was derived after agent N read files A, B, C").

8. **Mintlify's parallel sub-agent shape works** — One orchestrator + N parallel writers + post-hoc cross-link validation. ~35% wall-clock saving on big repos. *Implication*: Forge's epic-build pattern is on the right track; extend it to spec-generation.

9. **Spec Kit brownfield fragmentation as a warning** — Three competing community forks, no first-class brownfield support from upstream. *Implication*: nobody has cracked this yet. There's room for Rapoport to publish the canonical reference if we get it right.

10. **Augment's MCP-as-context-broker** — Don't dump; broker. Every agent calls a context server, not a flat file. *Implication*: Maestro's role in the orchestra is exactly this — the context broker that all other agents go through.

---

## Recommendation: the 2026 reference migration pipeline

A 7-stage pipeline. Each stage produces a checked-in artifact. Forge orchestrates; Muse runs the per-capability passes; Maestro is the persistent context broker.

### Stage 0 — Inventory & gate (5 min)
- **Inputs**: a git URL or a local clone.
- **Tools**: `git`, `tokei` / `scc` (LOC counts), `tree -L 3`.
- **Gate**: size class (small <30k LOC / medium 30-300k / large >300k). Pipeline branches on this.
- **Output**: `migration/stage-0-inventory.md` — LOC, languages, top-10 dirs, list of detected manifests (`package.json`, `Cargo.toml`, `pyproject.toml`, `Gemfile`, etc.), git age, contributor count.

### Stage 1 — Risk red-line sweep (5-15 min)
- **Tools**: Semgrep (`p/owasp-top-ten` + `p/llm-security`) → Gitleaks → TruffleHog (with verification).
- **Why first**: We don't want to feed an LLM a repo with live credentials in it. Strip + mask before any model sees anything.
- **Output**: `migration/stage-1-risks.md` — categorized red-line inventory: RLS, auth, concurrent, locks, secrets, crypto, payments. Each item: file:line, rule that fired, severity, "blocks promotion to spec / requires human review / informational."

### Stage 2 — Structural snapshot (10-30 min)
- **Tools**: Aider-style PageRank repomap (port the algorithm; ~300 lines). For large repos, SCIP indexer for the language. ast-grep sweeps to identify capability boundaries (routes, jobs, hooks, migration files, RLS policies, env-var references).
- **Output**: `migration/stage-2-repomap.json` — the persistent index Maestro reads on every subsequent invocation. Cheap to regenerate via Merkle-tree-diff approach.

### Stage 3 — Kiro three-file scaffold (5-15 min)
- **Driver**: Muse, single agent, one model call per file, gated by Stage 1 risks.
- **Output**:
  - `openspec/current/platform.md` skeleton (what the repo does — derived from README, package.json `description`, route file names).
  - `openspec/current/stack.md` skeleton (frameworks, libraries, tooling — derived from manifests).
  - `openspec/current/repo-map.md` skeleton (file tree + top-level ownership — derived from Stage 2).
- **Constraint**: every claim must cite file:line or manifest:key. Reviewer agent rejects unsourced claims.

### Stage 4 — Glossary + canon mapping (10-20 min)
- **Driver**: Muse, single agent, two passes.
  - Pass 1: extract domain terms from identifier names, comments, README, commit messages (LLM-assisted abbreviation mining, INNOQ-style).
  - Pass 2: map each term to Rapoport canon (`openspec/_methodology/canon.md`); flag conflicts.
- **Output**: `openspec/current/glossary.md` with two columns: codebase term → canonical term. Conflicts get a flag the human resolves.

### Stage 5 — Capability spec extraction (30-90 min, parallel sub-agents)
- **Driver**: Forge, Mintlify-shape fan-out: orchestrator decomposes into N capabilities (from Stage 2's structural boundaries), spawns N parallel Muse writers, post-hoc cross-link validation.
- **Constraint per writer**: SpecRover reviewer-agent gate — every spec line must reconcile with (a) a file:line, (b) a test if available, (c) a deterministic extract (OpenAPI, schema, migration) if available. No unsourced spec lines pass.
- **Output**: `openspec/current/<capability>.md` per major behavior. One file per concern (auth, RLS, payments, jobs, etc.).

### Stage 6 — Methodology bootstrap (5 min)
- **Driver**: deterministic templating.
- **Output**:
  - `openspec/_methodology/` skeleton (research/, postmortems/, decisions/).
  - `~/.rapoport/<repo>/` setup with per-agent Infisical paths.
  - First placeholder methodology files copied from the Rapoport canon.

### Stage 7 — First OpenSpec change proposal: the migration itself (10 min)
- **Driver**: Muse, single agent.
- **Output**: `openspec/changes/<date>-<slug>-onboard/` with `proposal.md`, `design.md`, `tasks.md`. The proposal documents *what we just did* and what remaining red-lines need human review before any new feature work. Tasks.md is the human's onboarding to-do list, not Forge's.

### Pipeline summary
- **Always-run tools**: Repomix (small-repo escape), tree-sitter + ast-grep, Semgrep + Gitleaks + TruffleHog, Aider repomap port.
- **Conditional**: SCIP for large repos, OpenAPI/schema extractors per stack.
- **Orchestration**: Forge for fan-out, Muse for per-stage work, Maestro as context broker (Augment-pattern MCP server).
- **Total wall-clock target**: small repo 1 hour, medium 3-4 hours, large overnight.

---

## Risks specific to repo migration

A 5-tier register for the new capability. Tier 1 = blocks shipping; Tier 5 = informational.

| Tier | Risk | Mitigation |
|---|---|---|
| **1** | LLM hallucinates spec lines (invariants that aren't true) | Reviewer agent demands evidence per spec line; SpecRover-style reconcile pass; refuse to write a spec line without file:line + test/extract citation. |
| **1** | Onboarding writes secrets into a generated spec | Mandatory Stage 1 (Gitleaks + TruffleHog) *before* any LLM sees the repo. Mask matches before bundling. |
| **1** | Slopsquatting in extracted dependency list | Cross-check every imported package against npm/PyPI/crates registry at extract time; fail loud if a name doesn't resolve. |
| **2** | Glossary maps codebase term to wrong canonical term silently | Two-column output with explicit "no mapping yet" sentinel; human ack required before promotion. |
| **2** | Reverse-extracted RLS policies look right but are wrong (Django-monolith pattern) | Treat RLS as Tier 1 red-line; never auto-promote — human must explicitly approve each RLS policy spec line. |
| **3** | Parallel sub-agent writers produce contradictory specs (Augment "uncoordinated agents" pattern) | Post-hoc cross-link validation pass; one final reconciliation agent reads all spec files and flags contradictions. |
| **3** | Repomap grows stale; Maestro returns wrong file lists | Merkle-diff sync on every Forge dispatch; budget a re-index step in CI. |
| **3** | PII / customer data in test fixtures leaks into Repomix output | Custom mask layer on top of Secretlint; explicit fixture-directory deny-list. |
| **4** | Spec language drifts from canonical Rapoport voice over time | Style-reviewer agent in Stage 5; canon file authoritative; human-led "voice review" gate. |
| **4** | Methodology folder becomes graveyard for boilerplate | Stage 6 ships placeholders with TODOs; methodology only counts as "real" when a research/postmortem file exists. |
| **5** | Onboarding takes longer on a large repo than a human would | Document the wall-clock budgets per size class; for >1M LOC, refuse to auto-onboard — propose a smaller scope. |

---

## Open questions

1. **The big one**: Does the orchestra produce specs *before* doing real work (spec-as-precondition) or *while* doing real work (SpecRover-style, spec-as-byproduct)? Public evidence leans toward the latter being more accurate, but the former is what every SDD framework markets. We should pick a side explicitly.

2. **License gate on CodeQL** — is it worth offering a "CodeQL-augmented onboarding" tier for clients with GHAS, or do we stick to Semgrep + open tools only?

3. **Glossary canon authority** — who/what is authoritative when Rapoport canon and a client's domain language conflict? (Suggested: a `glossary-conflicts.md` file the client owner explicitly resolves; the orchestra never decides.)

4. **Does the migration capability eventually become a Tessl-style spec-as-source loop?** Tessl is honest that the migration story is "hand-wavy"; we'd be more honest still by ranking specs by *evidence strength* (deterministic-extracted / test-anchored / LLM-inferred) and never auto-promoting LLM-inferred without a human gate.

5. **Maestro identity** — is Maestro the context broker, the orchestra conductor, or both? The Augment + Cody + Cursor evidence suggests these are the same role at architecture level. (User memory confirms `hello@rapoport.studio` is Maestro's mailbox — point in favor of treating Maestro as the persistent identity, not a per-task agent.)

6. **What's the minimum viable onboarding output?** Kiro's 3 files? OpenSpec's `/opsx:onboard` output? A single one-page PRD? We should run the pipeline against our *own* repo and time-box: "this took 90 minutes, and we read X of the output before it was actionable." That's the budget for clients.

7. **No public evidence** that any 2025-2026 tool produces a *risk inventory* the way Rapoport wants it (red-line categories with severity + spec-promotion gate). Semgrep + TruffleHog get us 70% there; the curation pass is novel. Unsolved.

---

## References (selected)

- Repomix — <https://github.com/yamadashy/repomix>
- gitingest — <https://github.com/coderamp-labs/gitingest>
- yek — <https://github.com/bodo-run/yek>
- code2prompt — <https://github.com/mufeedvh/code2prompt>
- files-to-prompt — <https://github.com/simonw/files-to-prompt>
- Aider repomap algorithm — <https://aider.chat/2023/10/22/repomap.html>
- ast-grep — <https://github.com/ast-grep/ast-grep>
- Comby — <https://comby.dev/>
- SCIP — <https://github.com/sourcegraph/scip>
- Semgrep OWASP — <https://semgrep.dev/p/owasp-top-ten>
- Semgrep Multimodal — <https://www.helpnetsecurity.com/2026/03/20/semgrep-multimodal-code-security/>
- Gitleaks vs TruffleHog — <https://www.jit.io/resources/appsec-tools/trufflehog-vs-gitleaks-a-detailed-comparison-of-secret-scanning-tools>
- CodeQL — <https://codeql.github.com/>
- Snyk Code — <https://github.com/snyk/cli>
- RepoAgent — <https://arxiv.org/abs/2402.16667>
- Mintlify auto-docs — <https://www.mintlify.com/blog/auto-generate-docs-from-repos>
- Augment Context Engine — <https://www.augmentcode.com/context-engine>
- Cursor indexing — <https://cursor.com/blog/secure-codebase-indexing>
- Cody — <https://aiwiki.ai/wiki/sourcegraph_cody>
- Copilot Workspace — <https://github.com/orgs/community/discussions/142971>
- Cosine Genie — <https://cosine.sh/blog/genie-technical-report>
- AutoCodeRover — <https://github.com/AutoCodeRoverSG/auto-code-rover>
- SpecRover — <https://arxiv.org/abs/2408.02232>
- Daikon — <https://plse.cs.washington.edu/daikon/>
- LLM vs MDE reverse engineering — <https://www.frontiersin.org/journals/computer-science/articles/10.3389/fcomp.2025.1516410/full>
- Codemod — <https://codemod.com/blog/codemod2>
- OpenRewrite — <https://docs.openrewrite.org/>
- jscodeshift — <https://github.com/facebook/jscodeshift>
- Sourcegraph Batch Changes — <https://sourcegraph.com/batch-changes>
- Devin — <https://devin.ai/>
- Spec Kit brownfield — <https://github.com/github/spec-kit/issues/1436>, <https://www.epam.com/insights/ai/blogs/using-spec-kit-for-brownfield-codebase>
- Kiro project init — <https://aws.amazon.com/startups/prompt-library/kiro-project-init>
- BMad established projects — <https://docs.bmad-method.org/how-to/established-projects/>
- Tessl existing code to spec — <https://docs.tessl.io/common-workflows/existing-code-to-spec>
- Fowler "Understanding SDD" — <https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html>
- OpenSpec — <https://github.com/Fission-AI/OpenSpec>, <https://openspec.dev/>
- Slopsquatting — <https://snyk.io/articles/slopsquatting-mitigation-strategies/>
- Django-monolith failure — <https://thefridaydeploy.substack.com/p/ai-cant-handle-your-legacy-codebase>
- Apiiro Sept 2025 findings — referenced in <https://www.kusari.dev/blog/ai-coding-assistants-in-2026-4x-faster-10x-riskier-the-hidden-security-cost>
- Claude Code anatomy — <https://codewithmukesh.com/blog/anatomy-claude-code-session/>
- LLM-assisted abbreviation mining (INNOQ) — <https://www.innoq.com/en/blog/2024/11/llm-assisted-abbreviation-mining/>
