---
title: Training the Orchestra to Author OpenSpec — Layered Architecture
status: research
audience: [architect, founder, ai-engineer]
date: 2026-05-11
inputs:
  - openspec/_methodology/research/openspec-organization-patterns-2026.md
  - openspec/_methodology/research/multi-audience-documentation.md
  - openspec/_methodology/research/sdd-terminology-scorecard.md
  - openspec/_methodology/research/ai-risk-frameworks-and-indexing.md
  - openspec/_methodology/research/spec-to-code-validity-2026.md
  - openspec/_methodology/research/trust-gradient-evolution-2023-2026.md
  - openspec/_methodology/research/13-schools-of-thought.md
  - openspec/_methodology/research/nodejs-ai-frameworks-landscape-2026.md
  - openspec/_methodology/research/training-orchestra-on-openspec-format-2026.md
---

# Training the Orchestra to Author OpenSpec — Layered Architecture

## TL;DR

The question "how do we train the orchestra to write better OpenSpec specs" is the wrong frame. The right frame: **build a system where every authored spec is gated, traced, and improvable** — so improvement is a property of the pipeline, not of any single model run.

Eight architectural layers, each with one responsibility, one technology pick, one risk class, and one open research question. The system trains itself only when **all eight close the loop**: corpus → retrieval → generation → validation → execution → telemetry → governance feeds back to corpus.

- **3 cheapest wins this quarter:** Vale linting in CI, cross-family LLM-judge on every spec PR, 30–50 example golden set.
- **3 medium investments (next 6mo):** DSPy + GEPA compile loop for the Muse author role, lineage registry as first-class artifact, OpenSpec → Anthropic Skills exporter.
- **3 long bets (12-18mo):** publish prose-spec quality benchmark, vector retrieval at 1000+ doc threshold, multi-agent self-improvement with auditable feedback.

---

## The problem, re-framed

Two failure modes are visible in the existing research library:

1. **Spec drift** — terminology collisions ([sdd-terminology-scorecard.md](sdd-terminology-scorecard.md)), audience confusion ([multi-audience-documentation.md](multi-audience-documentation.md)), and structural gaps the community hasn't closed ([openspec-organization-patterns-2026.md](openspec-organization-patterns-2026.md)) compound over months.
2. **Trust ceiling** — 7 hard red-line categories (RLS, auth, concurrent, locks, secrets, crypto, payments) plateau at ~45% accuracy across three years of model improvements ([trust-gradient-evolution-2023-2026.md](trust-gradient-evolution-2023-2026.md), [spec-to-code-validity-2026.md](spec-to-code-validity-2026.md)). Bigger models do not close this.

A naive "let's prompt-engineer Muse better" addresses neither. The right architecture treats spec authoring as a **pipeline with measurement at every joint** — same discipline that turns ML experiments into production systems.

This document is the layered blueprint for that pipeline.

---

## OpenSpec format snapshot (recap)

Triplet per change: `proposal.md` (why) + `design.md` (how) + `tasks.md` (checklist). Capability specs in `openspec/current/<cap>.md`. Archive frozen at `openspec/archive/YYYY-MM-DD-<slug>/`. Prose-first, not formal Requirement deltas (intentional studio extension).

Key invariants from the digest:
- **Capability = behaviour, not technical layer.** Antipattern: `api.md`, `ui.md`, `db.md`.
- **One reader per artifact.** `proposal.md` → stakeholder, `design.md` → architect, `tasks.md` → executor.
- **Archive is immutable.** Edits to applied specs only via new change proposals.
- **Bundle <1MB, RAG >1GB, vector embedding between.** Studio is in bundle range today.

The pipeline below is what wraps this format with measurement and feedback.

---

## Architecture — eight layers

```
┌──────────────────────────────────────────────────────────────┐
│ L7  Governance       canon, red lines, risk register         │
├──────────────────────────────────────────────────────────────┤
│ L6  Telemetry        events.jsonl, lineage, metrics, evals   │
├──────────────────────────────────────────────────────────────┤
│ L5  Execution        Forge builder + verifier → PR           │
├──────────────────────────────────────────────────────────────┤
│ L4  Validation       Vale + LLM-judge + golden-set evals     │
├──────────────────────────────────────────────────────────────┤
│ L3  Generation       Muse author role: idea → spec triplet   │
├──────────────────────────────────────────────────────────────┤
│ L2  Context assembly 4-layer prompt cache + Skills loader    │
├──────────────────────────────────────────────────────────────┤
│ L1  Indexing         bundle, repo-map, file fingerprints     │
├──────────────────────────────────────────────────────────────┤
│ L0  Storage          openspec/ filesystem, git, archive      │
└──────────────────────────────────────────────────────────────┘
                  ↑                              │
                  └────── feedback loop ─────────┘
```

Each layer below: **what / tech / risk / research needed.**

### L0 — Storage & Versioning

**Responsibility.** Authoritative file storage of `openspec/`, change proposals, archive. Immutability of applied specs.

**Tech (already in place).**
- Filesystem under `openspec/{current,changes,archive,_methodology}`
- Git as audit log
- `openspec/AGENTS.md` as format contract
- `openspec validate` skipped (prose style, strict-deltas false-positive)

**Risk.** Archive folder edits ([feedback_archive_frozen memory](../../../../.claude/projects/-Users-pavelrapoport-Documents-GitHub-rapoport-studio/memory/feedback_archive_frozen.md)) — frozen contract not technically enforced; relies on convention + reviewer discipline.

**Research needed.**
- ADR-style enforcement: pre-commit hook that rejects writes to `openspec/archive/*` and to `openspec/current/*` outside an active change branch. Cheap, deterministic, prevents the most common drift class. Closes [ai-risk-frameworks-and-indexing.md § L1 cache invalidation](ai-risk-frameworks-and-indexing.md) at storage boundary.

### L1 — Indexing & Retrieval

**Responsibility.** Make any spec retrievable by capability, by audience, by keyword, by file-hash. Today this means: bundle the corpus into a single object that downstream prompts can reference.

**Tech (current).**
- `tools/build-openspec-bundle.mjs` produces `apps/studio/src/generated/openspec-bundle.ts`
- Tripwire: `gzip(module) ≤ 6 MB` ([openspec_bundle_tripwire_gzipped memory](../../../../.claude/projects/-Users-pavelrapoport-Documents-GitHub-rapoport-studio/memory/openspec_bundle_tripwire_gzipped.md))
- Memory note: regenerate after any `openspec/` edit ([feedback_openspec_bundle_regen](../../../../.claude/projects/-Users-pavelrapoport-Documents-GitHub-rapoport-studio/memory/feedback_openspec_bundle_regen.md))

**Tech (recommended additions).**
- `repo-map.json` — flat index `{ path, capability, audience, sha256, lastEditedAt, sizeTokens }` regenerated alongside bundle. Routes queries before loading full files.
- INDEX.md per capability folder ≤ 1KB description (Anthropic Skills frontmatter pattern from [ai-risk-frameworks-and-indexing.md § progressive disclosure](ai-risk-frameworks-and-indexing.md)).
- **No vector embeddings yet.** [training-orchestra-on-openspec-format-2026.md § Layer 2](training-orchestra-on-openspec-format-2026.md) — 2026 consensus: long-context wins at <200K tokens; RAG only pays off above. Studio is below threshold.

**Risk.**
- **Lost-in-the-Middle** (Liu TACL 2024): same spec content recalled 30+ pp worse at position 10 vs position 1 in a 20-doc context. Bundle ordering matters.
- Stale bundle = silent wrong answers. Mitigation: regen hook on `openspec/` write.

**Research needed.**
- Empirical: at what `openspec/` size does long-context degrade enough that vector retrieval pays off? Need to instrument retrieval quality on golden Q&A pairs as corpus grows. Until then, the rule "stay below 200K tokens" is heuristic.
- Bundle-ordering optimization: should most-relevant capability appear first? Last? Repeated head+tail? Untested.

### L2 — Context Assembly

**Responsibility.** Take a user/agent intent ("draft a spec for X"), assemble the prompt by progressively loading: system rules → role definition → relevant capability → relevant change history → live state. Optimize for prompt cache hit rate.

**Tech (target).**
- 4-layer Anthropic prompt cache from [ai-risk-frameworks-and-indexing.md § prompt caching](ai-risk-frameworks-and-indexing.md):
  - **L1** system + role prompts (5-min TTL, stable)
  - **L2** tools + skill catalog (5-min TTL, semi-stable)
  - **L3** openspec bundle / repo-map (1h TTL, stable across edits with extended cache)
  - **L4** dynamic — current task, recent events (no cache)
- Skills loader: Anthropic SKILL.md format — each skill ≤100-token description, body loaded only on hit
- Stable-prefix discipline: never edit prompts above a cache breakpoint mid-session

**Risk.**
- Cache invalidation on any `openspec/` write thrashes L3. Mitigation: chunk the bundle by stability tier (archive = ultra-stable, current = stable, changes = volatile). Cache them as separate prefixes.
- Cache poisoning: stale skill descriptions tell the agent a skill exists when it doesn't. Mitigation: hash-pin skill manifests.

**Research needed.**
- File-level fingerprinting: which files actually changed since last bundle build? Merkle-tree-style invalidation would let L3 keep most-of-bundle cached while only invalidating changed leaves. Untested empirically.
- "Stable ordering" heuristic: ordering bundle by `lastEditedAt` ascending should maximize cache reuse. Needs validation.

### L3 — Generation

**Responsibility.** Take assembled context + intent, produce a spec triplet (proposal.md + design.md + tasks.md) that conforms to OpenSpec format. This is the Muse author role.

**Tech (recommended).**
- **Inference layer:** Vercel AI SDK v6 ([nodejs-ai-frameworks-landscape-2026.md § recommendations](nodejs-ai-frameworks-landscape-2026.md)) — `@ai-sdk/anthropic` is 1.29M weekly downloads, first-class agent loop, MCP clients, tool approval gates
- **Model routing:** Opus 4.7 for architect role (high judgment), Haiku 4.5 carve-out for low-stakes audit sub-pipelines (already established in [RAP-448 (#715)](https://github.com/rapoport-studio/rapoport.studio/pull/715))
- **Persona templates:** Three roles per [openspec-organization-patterns-2026.md § one reader per artifact](openspec-organization-patterns-2026.md):
  - Architect (drafts design.md) — Opus
  - Storyteller (drafts proposal.md narrative) — Opus
  - Planner (drafts tasks.md checklist) — Sonnet or Haiku
- **BAML for schema constraints** — 84% schema-token reduction at 100% quality parity, per [training-orchestra-on-openspec-format-2026.md § DSPy+BAML](training-orchestra-on-openspec-format-2026.md). Saves cost on every generation.

**Risk.**
- **Sycophancy** (SycEval, arxiv 2502.08177 — 58% sycophancy rate in coding LLMs). Author agent agrees with bad input prompts because of how human framed the request. Cross-family judge in L4 is the only known mitigation.
- **Red-line ignorance.** Generation cheerfully writes specs for RLS/auth/concurrent code that the trust gradient says shouldn't be auto-built. Mitigation: red-line classifier as preflight gate before generation. If the proposed change touches red-line categories, switch role to "draft, flag for human review" not "draft, hand to Forge".
- **Authoring vs reading asymmetry** — open empirical question (no public benchmark distinguishes hallucination rates by task direction).

**Research needed.**
- Build the 30–50 example golden set (single highest-leverage artifact, ~1 engineer-day investment).
- Decide: does Muse author the triplet in three serial calls (one per role) or one call with three sections? Three serial = better cache reuse + smaller failure surface; one call = better internal coherence. Untested.

### L4 — Validation

**Responsibility.** Reject or escalate generated specs that fail quality gates **before they reach humans for review**. Specifically: prose lint (deterministic), LLM-judge (probabilistic), golden-set regression (empirical).

**Tech (recommended).**
- **Vale** ([vale.sh](https://vale.sh)) — deterministic prose linter. <2s on 1.5k Markdown files. Datadog/Grafana/Spectro use it in CI. Custom YAML rules per OpenSpec:
  - Terminology canon enforcement (no "Studio" where "Workspace" is meant, etc. — see [sdd-terminology-scorecard.md § recommended renames](sdd-terminology-scorecard.md))
  - Sentence-length and Flesch-Kincaid floors
  - Audience-tag presence check
- **Cross-family LLM-judge.** If Sonnet authors → Gemini or GPT-5 judges. Mitigates sycophancy and preference-leakage (ICLR-2026 finding: judges prefer outputs from same family).
- **Golden-set regression.** 30–50 graded examples covering: trivial (5), red-line (5), CRUD (5), architectural (5), ambiguous (5). Every prompt/model change re-runs evals. **Braintrust** (free tier: 1M trace spans, 10K runs, CI-native) + **DeepEval** (TS, fits stack) — both from [training-orchestra-on-openspec-format-2026.md § Layer 5](training-orchestra-on-openspec-format-2026.md).

**Risk.**
- **Goodhart on the rubric.** Author optimizes for what judge measures; judge measures a proxy for quality. Mitigation: rotate rubric phrasing periodically; sample human review of judge agreements.
- **Golden set staleness.** Examples become trivial as model improves. Refresh quarterly with new "hard" examples.
- **No public prose-spec benchmark** to compare against — Rapoport's golden set is internal. Publishing it would be a publication-grade artifact.

**Research needed.**
- Empirical sycophancy delta: when same family judges, does Muse score 5-15 pp higher? Quantifies the cross-family-judge investment.
- LLM-judge rubric calibration — Anthropic and Stanford have public methodologies; how many human-graded samples are needed to align the judge?

### L5 — Execution

**Responsibility.** Take an approved spec, produce code, open PR. This is Forge today.

**Tech (in place, evolving).**
- Coordinator-Implementor-Verifier pattern ([nodejs-ai-frameworks-landscape-2026.md § pattern 5](nodejs-ai-frameworks-landscape-2026.md)) — already Forge's architecture
- Reflexion loop with 2-3 iterations cap (empirical limit from same source)
- Global build lock at `~/.forge/locks/build.lock` ([forge_global_build_lock memory](../../../../.claude/projects/-Users-pavelrapoport-Documents-GitHub-rapoport-studio/memory/forge_global_build_lock.md))
- Per-role model tiering ([RAP-213](../../../../.claude/projects/-Users-pavelrapoport-Documents-GitHub-rapoport-studio/memory/rap_213_complete.md))

**Risk.**
- **Builder/orchestrator commit race** ([forge_builder_orchestrator_commit_race memory](../../../../.claude/projects/-Users-pavelrapoport-Documents-GitHub-rapoport-studio/memory/forge_builder_orchestrator_commit_race.md)) — false `aborted-no-changes` outcome despite real PR. Known. Cross-check `gh pr list` before believing outcome.
- **Verification gate filter bug** ([forge_verification_gate_filter_bug memory](../../../../.claude/projects/-Users-pavelrapoport-Documents-GitHub-rapoport-studio/memory/forge_verification_gate_filter_bug.md)) — stray `--` runs full test suite. Known and documented.
- **Auto-flags trust gap** ([forge_dist_staleness_and_auto_flags memory](../../../../.claude/projects/-Users-pavelrapoport-Documents-GitHub-rapoport-studio/memory/forge_dist_staleness_and_auto_flags.md)) — `state.json: completed` does not mean PR exists. The state.json is not authoritative.

**Research needed.**
- L5 is the **most-documented layer in our memory**. Almost every Forge memory note is a layer-5 reliability issue. Pattern is that ground-truth lives in GitHub/CI, not in Forge's local state. Sourcing truth from `gh pr view` not `state.json` is the structural fix; gradual migration in progress.

### L6 — Telemetry & Learning

**Responsibility.** Capture what happened (events, traces, costs, outcomes), aggregate by lineage (one idea → its spec → its PR → its measurement), feed it back to L3 generation as either context (in-context examples) or training signal (DSPy/GEPA compile).

**Tech (recommended).**
- **events.jsonl** at `~/.rapoport/events/YYYY-MM-DD.jsonl` — typed kinds (RUN_STARTED, SPEC_DRAFTED, MODEL_CALL, PR_OPENED, JUDGE_VERDICT, …). Schema-versioned.
- **Lineage registry** — `~/.rapoport/lineage/<linear-id>.jsonl` aggregates from events.jsonl + GitHub + Linear. **This is the gap-filling artifact** ([digest § gap 1: spec quality scoring](orchestra-training-architecture-2026.md)). Cost-per-idea, time-to-spec, spec→merge lag, revert rate — all queryable from one source.
- **Langfuse** as optional sink for MODEL_CALL events. Self-hostable. Captures prompt versioning, token tracking, eval dashboards.
- **DSPy + GEPA + BAML compile loop** ([training-orchestra-on-openspec-format-2026.md § Layer 4](training-orchestra-on-openspec-format-2026.md)) — ICLR-2026 oral (arxiv 2507.19457), beats GRPO with up to 35× fewer rollouts. Production case studies: +20pp on structured tasks at ~$3 / ~1200 rollouts. **This is how the orchestra trains without RL.**

**Risk.**
- **Cost-per-iteration trap.** Compile loops are cheap individually but expensive in aggregate. Budget caps required.
- **Feedback leakage.** If judge verdicts feed back into author training, the system optimizes for the judge rather than reality. Mitigation: split into two corpora — one for author training (judge verdicts excluded), one for judge calibration (human-graded only).

**Research needed.**
- **The biggest open empirical question for Rapoport.** How fast does the author actually improve given (a) increasing golden set size, (b) increasing examples in-context, (c) GEPA compile rounds? Three-variable experiment. Publishable.

### L7 — Governance

**Responsibility.** Define what counts as "good" before any of the layers below can measure or generate. Includes: terminology canon, red lines, risk register, audience policy, escalation paths.

**Tech (mostly documents, not code).**
- `openspec/_methodology/glossary.md` — Bridge Glossary per [13-schools-of-thought.md](13-schools-of-thought.md) and [sdd-terminology-scorecard.md](sdd-terminology-scorecard.md). 10 high-collision terms + Rapoport canon.
- `openspec/_methodology/red-lines.md` — 7 categories from [trust-gradient-evolution-2023-2026.md](trust-gradient-evolution-2023-2026.md). Encoded as L4 Vale rules + L3 preflight classifier.
- `openspec/_methodology/risk-register.md` — consolidated 5-tier matrix:
  - L1 regulatory (EU AI Act transparency)
  - L2 security (OWASP LLM Top 10)
  - L3 product (SDD red lines)
  - L4 methodology (terminology drift, audience leak)
  - L5 implementation (each layer's specific risks)
- `openspec/_methodology/audience-policy.md` — Diataxis assignment per artifact type, frontmatter contract.

**Risk.**
- Governance docs **drift** from operational reality. The bridge glossary the orchestra was trained on lags behind PR-time terminology decisions. Mitigation: governance review as a quarterly ritual, **not** as a one-time write.
- Risk register **theatre**. Listing risks ≠ mitigating them. Each entry needs an owner + a recurring eval.

**Research needed.**
- Adoption empirics on ADR-style governance: does writing red lines into a doc actually change author behaviour, or is it ceremonial? [training-orchestra-on-openspec-format-2026.md § Layer 7](training-orchestra-on-openspec-format-2026.md) cites that the empirical literature on ADR adoption is weak. Real measurement requires layer 6 telemetry: count red-line violations per quarter, see if they trend down.

---

## Cross-layer concerns

### Caching strategy as architecture

Caching is not a layer — it's a **discipline that touches L1 (corpus stability), L2 (prefix ordering), L3 (deterministic prompts), L6 (cache-hit telemetry)**.

The 4-layer Anthropic strategy gives 85% latency reduction **only if** the corpus structure is cache-friendly. That means:

- Archive prefix never changes → L3 ultra-stable
- Current prefix changes per applied change → L3 stable-with-hash
- Changes folder is volatile → L4 only
- Bundle regeneration aligned with cache TTLs (1h match)

This is **why bundle structure is a layer-1 concern, not a layer-2 concern.** Get L1 right and L2 inherits cache friendliness for free.

### Terminology canon as compiler

The Bridge Glossary functions as a **compiler step at L3 generation**: every ambiguous term in the output is resolved to its Rapoport-canon meaning. Without this step, L4 validation has to do hard semantic work; with it, L4 can do mostly deterministic checks.

This is why [sdd-terminology-scorecard.md](sdd-terminology-scorecard.md) is more load-bearing than it looks. **It's not documentation — it's a constraint on the generation prompt.**

### Audience routing as access control

Diataxis assignment is **frontmatter-enforced** ([multi-audience-documentation.md § ten consolidated principles](multi-audience-documentation.md)):
- proposal.md → `audience: [stakeholder]`
- design.md → `audience: [architect]`
- tasks.md → `audience: [executor]`

L4 Vale rules can enforce: a proposal.md that contains code blocks longer than 5 lines, or a design.md without diagrams, fails the linter. This is cheap and high-precision.

### Trust gradient as routing decision

L3 generation must check red-line classification **before** invoking the author. If the change touches RLS, secrets, or auth — route to "human-first draft", not "AI-first draft". This is a single classifier call, decides which sub-pipeline to enter.

Without this routing, the orchestra wastes tokens on categories that will fail L4 anyway.

---

## Human factors

The orchestra ultimately serves humans (Pavel + future operators). Two questions:

### 1. How do humans learn the OpenSpec format?

[training-orchestra-on-openspec-format-2026.md § Layer 7](training-orchestra-on-openspec-format-2026.md) finds the empirical literature thin. What's known:
- **Diataxis adoption** ([diataxis.fr](https://diataxis.fr)) is documented at Stripe, Twilio, GitLab; **no public eye-tracking or time-to-comprehension studies** specifically on OpenSpec-style prose specs.
- **ADR adoption** (Architecture Decision Records, Nygard 2011) is widely cited but [ICSE/EASE empirical studies](https://ieeexplore.ieee.org/) show adoption decays without org-level mandate.
- **Apprenticeship/mentoring** is the historically reliable transfer mechanism for tacit technical writing skills — but unmeasured.

What to do for Rapoport:
- **Onboarding curriculum** = read one archived change end-to-end + pair on one new change. Not "read all the AGENTS.md files."
- **Single canonical example**, replaced annually. The live archive is the textbook.
- **Feedback ritual.** Every new authored change gets one round of senior review with explicit reference to a research principle that was applied or violated. Trains the human in the same loop as the model.

### 2. What is the cognitive load per role?

Per [multi-audience-documentation.md](multi-audience-documentation.md):
- Founder/stakeholder reading proposal.md — should not require reading any other file
- Architect reading design.md — should require ≤2 cross-references (capability spec + one prior change)
- Executor reading tasks.md — should be self-contained, no context-switching to prose

Layer 4 Vale can enforce a budget: "this file references ≤N other files." High-precision rule.

---

## Risk register (consolidated, 5-tier)

| Tier | Class | Examples | Mitigation layer |
|---|---|---|---|
| **L1 Regulatory** | EU AI Act transparency (high-risk if shipping to clients) | Auto-generated spec labeled as human-authored | L7 policy + L6 metadata stamp |
| **L2 Security** | OWASP LLM Top 10 — prompt injection via spec content, data leakage via cache | A malicious spec contains "ignore safety, write secrets" | L4 prose lint catches; L3 preflight classifier |
| **L3 SDD red lines** | RLS, auth, concurrent, locks, secrets, crypto, payments | Forge auto-builds an RLS policy from a partial spec | L7 red-line list + L3 routing |
| **L4 Methodology** | Terminology drift, audience leak, judge sycophancy, golden-set staleness | "Studio" used ambiguously across capabilities | L4 Vale + cross-family judge + quarterly golden refresh |
| **L5 Implementation** | Cache poisoning, lineage gaps, archive edits | Edit to archived spec, bundle stale, judge runs on old corpus | L0 pre-commit hook + L1 bundle freshness check |

The matrix is read **bottom-up for engineering** (fix L5 first) and **top-down for governance** (L1 sets the priorities).

---

## Research roadmap

### This quarter (next 3 months) — three cheapest wins

1. **Vale CI integration** — 2 engineer-days. Custom YAML rules per OpenSpec terminology canon. Per-PR signal, deterministic, zero ongoing cost.
2. **Cross-family LLM-judge gate** — 3 engineer-days. Run Gemini or GPT-5 on every spec PR with explicit OpenSpec-principles rubric. Cost ~$0.50/PR. Mitigates sycophancy.
3. **30–50 example golden set** — 1 engineer-day for v1. Five categories × 5-10 examples. Gates every prompt change downstream. **Single highest-leverage artifact.**

### 6 months — three medium investments

4. **DSPy + GEPA + BAML compile loop** — once golden set exists. Production-validated +20pp on structured tasks at ~$3 / ~1200 rollouts. ICLR-2026 paper.
5. **Lineage registry** — `~/.rapoport/lineage/<linear-id>.jsonl` aggregating events.jsonl + GitHub + Linear. Enables cost-per-idea and time-to-spec metrics. Makes L6 telemetry actually queryable.
6. **OpenSpec → Anthropic Skills exporter** — converts capability specs into SKILL.md format. Distribution channel: every Claude Code/Cursor/Codex user can install Rapoport methodology. Aligns with [ai-cli-landscape-2026-05.md § distribution primitive](ai-cli-landscape-2026-05.md).

### 12-18 months — three long bets

7. **Publish prose-spec quality benchmark.** No public benchmark exists ([training-orchestra-on-openspec-format-2026.md § biggest open question 1](training-orchestra-on-openspec-format-2026.md)). 200-500 graded examples + judge harness. Hiring signal + category-defining artifact.
8. **Vector retrieval at 1000+ docs.** When `openspec/` exceeds long-context threshold. pgvector + structured retrieval (not Mem0 Pro). Wait for empirical trigger.
9. **Multi-agent self-improvement with auditable feedback.** The architect → builder → verifier loop already exists (L5). Extending to "verifier feedback updates architect prompt corpus" requires solving the feedback leakage problem (governance L7). Speculative.

---

## First three moves (this week)

To not pile any more design on top of unfunded plans:

1. **Draft OpenSpec change proposal `extract-cli-core-with-event-bus`** — formalizes L0, L1, L6 contracts. Not code — schemas. Lets all subsequent work share vocabulary.
2. **Stand up Vale on one capability** — pick `openspec/current/muse.md`. Get one PR through Vale CI. Proves L4 feasibility before scaling rules.
3. **Hand-author 5 golden-set examples** — pick easiest category (CRUD spec). Two correct, two with one issue each, one with structural problem. Establish the rubric format. Future expansion is 90% mechanical once shape exists.

These three moves cost <1 week, deliver:
- A schema others can build to (#1)
- A working L4 gate, even minimal (#2)
- The seed of an eval corpus (#3)

Everything in the 6mo and 12-18mo lists builds on these three.

---

## Open questions

For the next conversation:

1. **Does Muse author proposal/design/tasks in three serial calls or one combined call?** Three serial = better cache reuse + tight failure surface; one combined = better internal coherence. No data either way. Choose by experiment, not deduction.

2. **Where does the golden set live?** `openspec/_methodology/evals/golden/` is the obvious answer, but evals interact differently with bundle/cache than docs do. Possibly `tools/evals/` is better. Tradeoff: discoverability (docs) vs. cache-hygiene (tools).

3. **What's the right cadence for L7 governance review?** Quarterly is plausible but unmeasured. Monthly catches drift faster but costs senior time. Could be tied to bundle size growth (every 10% bundle growth = governance review).

4. **Maestro's role in this architecture.** L6 telemetry is currently described as passive — events.jsonl appends, lineage aggregates. Maestro could be the **active L6** — reading telemetry, surfacing trends, proposing governance changes, escalating red-line violations. Is that the right scope for Maestro, or is it scope-creep?

5. **When does the studio open-source any of this?** [open-source-positioning.md](open-source-positioning.md) covers positioning. Specific question: does the golden set need to be public for credibility, or does keeping it private protect the wedge? No clear answer.

---

**End of document.** This is the structural blueprint. The 8-layer architecture is the contract; the research roadmap is the funding plan; the first-three-moves list is the next-week deliverable. Updates to this document should themselves go through L7 governance — propose a change, archive the old version.
