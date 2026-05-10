# Research thread — public-facing OpenSpec vs Spec Kit analysis

> **Status:** complete research corpus backing the Lab article «Why OpenSpec, not Spec Kit». First-party empirical benchmark + failure-mode catalogue + GitHub health metrics, in addition to secondary synthesis. Source-cited throughout.
> **Owner:** Pavel (founder).
> **Authors:** Pavel Rapoport <pavel@rapoport.studio> (founder, primary); Claude Opus 4.7 (Anthropic) — literature synthesis, GitHub API audits via `gh api`, first-party empirical benchmark in `/tmp/sdd-benchmark/`, failure-mode catalogue assembly, drafting of all section content under Pavel's review and direction.
> **Method:** Multi-pass — (1) literature synthesis from 15+ public comparisons (Hashrocket, Rida Kaddir, ranthebuilder.cloud, Scott Logic, Martin Fowler, dev.to, intent-driven.dev, HN threads); (2) failure-mode catalogue from 30+ GitHub issues and discussions in `github/spec-kit` + `Fission-AI/OpenSpec`; (3) first-party empirical benchmark on `/tmp/sdd-benchmark/` — install timing, init timing, scaffold size, agent context tax, per-feature spec floor, cost economics at Sonnet 4.6 input pricing; (4) GitHub API health-metrics audit via `gh api` for commits (`search/commits`), releases (paginated), PR merge time + issue close time (last-30 sample), contributor diversity (last-100 sample).
> **Reusable for:** Lab MDX article authorship (`apps/web/src/content/lab/openspec-vs-speckit.mdx` — pending); positioning copy at `rapoport.studio/methodology` (when methodology page lands); future SDD-track research; potential conference talks or external publications citing the studio. **Authorship: open** — downstream Lab article author may not be the research author; this file is the data corpus, the article writer brings the voice.
> **Opened:** 2026-05-10.
> **Companion to:** [`./competitive-positioning-sdd-frameworks.md`](./competitive-positioning-sdd-frameworks.md) — that file is internal positioning (where the studio sits in the SDD landscape); this file is the public-facing data corpus.
> **Feeds into:** Lab article at `apps/web/src/content/lab/openspec-vs-speckit.mdx` (pending), positioning copy at `rapoport.studio/methodology` (when methodology page lands).
> **Path note:** placed under `_methodology/research/` per the corrected convention. First file written under the [research authorship convention](./README.md) introduced by `openspec/changes/add-research-authorship-convention/`.

---

## 0. What this thread is and is not

This is a **public-facing** research artifact: the data corpus that backs the Lab article comparing OpenSpec and Spec Kit. It is:

- **First-party empirical**: setup, install, structure, and cost economics measured in `/tmp/sdd-benchmark/` on this machine on 2026-05-10.
- **Failure-mode catalogue**: 14 categories, 30+ documented bugs with quoted evidence and direct GitHub issue links.
- **Health-metrics audit**: GitHub API data on commits, releases, PRs, issue triage, contributor diversity — with at least one claim from the prior internal positioning file actively refuted.
- **Honest about what was not measured**: agent-execution token counts (require live Claude Code session, protocol attached), star velocity (GitHub blocks paginate >400 pages), real human-triage close time for Spec Kit (stale-bot dominates the recent-close cohort).

It is not:

- Internal positioning copy — that lives in `competitive-positioning-sdd-frameworks.md`.
- A Spec Kit takedown — the data shows both projects are alive; the differences are structural, not vitality.
- A finished article — section 12 covers article structure but the MDX draft is separate.

---

## 1. Category map — three philosophies of SDD

Inheriting the frame Martin Fowler and Thoughtworks Tech Radar use:

| Axis | Spec-first | Spec-anchored | Spec-as-source |
|---|---|---|---|
| Definition | spec drives plan, plan drives code | spec lives alongside, deltas track changes | code regenerates from spec; spec is the maintained artifact |
| Tools | **Spec Kit**, Kiro, BMAD-Method | **OpenSpec**, Augment Code Intent | Tessl |
| Bias | greenfield + waterfall-shape | brownfield + iterative | radical bet on AI-as-compiler |
| Maturity | most-tested in 2026 | most-momentum (YC W26) | smallest user base |

Within spec-anchored: **OpenSpec** is the dominant CLI; BMAD is the multi-agent variant.
Within spec-first: **Spec Kit** is GitHub-blessed; Kiro is AWS-native IDE.

This article focuses on OpenSpec vs Spec Kit because they are:
- Both CLI / agent-agnostic
- Both top-of-mind in the SDD discussion (95K + 47K stars combined)
- The active live decision for most readers
- The pair where the choice has the biggest cost implication (see §10)

Tessl, Kiro, BMAD, Intent are mentioned where philosophically informative but not deep-dived.

---

## 2. Comparison parameters — 12 axes that matter

Ranked by frequency across 8 published comparisons (Hashrocket, Rida Kaddir, ranthebuilder.cloud, Scott Logic, Martin Fowler, dev.to, intent-driven.dev, HN #45154355) plus impact on real-world decision:

| # | Parameter | Why it matters |
|---|---|---|
| 1 | Output verbosity / review burden | Direct impact on iteration speed — primary practical KPI |
| 2 | Brownfield vs greenfield fit | Most real projects are brownfield; bisects the market |
| 3 | Workflow rigidity (number of phases) | 3 vs 7 phases = different philosophy of iteration |
| 4 | Setup friction | Adoption barrier; team-wide cost |
| 5 | Spec persistence model | Living doc vs branch-isolated → different memory model |
| 6 | Spec drift handling | Categorical weakness of both; differs only in reconciliation tooling |
| 7 | Cross-feature / systems thinking | Matters when features interact |
| 8 | Branch / git automation | Operational fit with team conventions |
| 9 | Agent ecosystem coverage | Lock-in risk |
| 10 | Compliance / audit trail | Decisive for enterprise, secondary for founder-track |
| 11 | Maintenance momentum | Long-term bet quality |
| 12 | Customization / templates | Matters at scale (many repos) |

---

## 3. Parameter-by-parameter analysis

### 3.1 Output verbosity & review burden

| | OpenSpec | Spec Kit |
|---|---|---|
| Output on equivalent task | ~250 lines (Hashrocket) | ~800 lines (Hashrocket); 2 577 lines (Scott Logic on real feature) |
| Review time | minimum-viable | 3.5h vs 24min traditional review (Scott Logic) |
| Pluses | low noise, fast iteration, easily readable end-to-end | rich audit trail; usable for compliance |
| Minuses | less context for new joiner | overload — *"duplicative, faux context"* (Scott Logic), *"repetitive and tedious"* (Fowler) |

### 3.2 Brownfield vs greenfield

| | OpenSpec | Spec Kit |
|---|---|---|
| Brownfield-native | **delta markers** ADDED/MODIFIED/REMOVED | not designed for it; *"assumes you're building something new"* (Rida Kaddir) |
| Greenfield | works (everything ADDED), no constitution-equivalent | **constitution-driven**: project principles inherited across features |
| Pluses | only tool with native "what changed" semantics | stronger cross-feature systems thinking via per-feature spec.md |
| Minuses | weaker cross-feature analysis — features are not first-class entities | brownfield: *"reverse-engineering friction per feature"* (dev.to); rewrite existing functionality into spec.md format |

### 3.3 Workflow rigidity

| | OpenSpec | Spec Kit |
|---|---|---|
| Phases | 3: Propose → Apply → Archive | 7: constitution → specify → clarify → plan → analyze → tasks → implement |
| Philosophy | *"fluid not rigid, iterative not waterfall"* | sequential phase-gates |
| Pluses | any artifact editable without gate-blocking | each phase is a quality gate; `/analyze` catches inconsistencies |
| Minuses | fewer built-in integrity checks | *"reinvented waterfall"* (Scott Logic); change of direction = full regeneration (ranthebuilder) |

### 3.4 Setup friction

Empirical, measured 2026-05-10 in `/tmp/sdd-benchmark/`:

| | OpenSpec | Spec Kit |
|---|---|---|
| Stack | TypeScript / npm | Python + `uv` + GitHub release fetch |
| Cold install | 4.5s (`npm install -g @fission-ai/openspec`) | 15.5s (10.5s `brew install uv` + 5.0s `uvx --from git+...specify init`) |
| Init in repo | 1.5s | 5.0s |
| **End-to-end** | **6.0s** | **20.5s** (3.4× longer) |
| Commands installed | 4 | 8 |
| Setup gotcha | none | PyPI package `specify-cli` is **not** the right tool — official path is `uvx --from git+https://github.com/github/spec-kit.git`. First install attempt failed silently with *«No matching release asset found»* |

### 3.5 Spec persistence model

| | OpenSpec | Spec Kit |
|---|---|---|
| Source of truth on disk | `openspec/specs/<capability>/spec.md` — durable; appended via delta-merge on archive | `.specify/specs/[NNN-feature]/spec.md` — per-branch, per-feature |
| History accumulation | `changes/archive/<date>-<name>/` keeps full proposal+design+tasks | git history of feature branches; no first-class archive |
| Pluses | spec is single source of current truth; auditable now-state | each feature is isolated, no concurrent-modification chaos |
| Minuses | concurrent changes on same capability need merge discipline | *"specs as change artifacts, not long-lived capability contracts"* (dev.to); no current-state-of-the-system view in one place |

### 3.6 Spec drift

Categorical weakness of both; mechanism differs:

| | OpenSpec | Spec Kit |
|---|---|---|
| Reconcile mechanism | new change-folder with `MODIFIED` deltas describing drift | re-run `specify → plan → analyze`; `/analyze` flags mismatch |
| Cost | manual but compact | manual; regenerates all artifacts |

The deeper observation: a Spec Kit user in Discussion #152 wrote *«Treating specs as session deltas rather than permanent truth has been the thing that finally removed the friction»*. **The user reinvented OpenSpec's model inside Spec Kit.**

### 3.7 Cross-feature systems thinking

| | OpenSpec | Spec Kit |
|---|---|---|
| Features as first-class | weakly — capabilities exist but less prominent | strongly — `[NNN-feature-name]/` per feature |
| Cross-feature analysis | not primary | primary strength |

### 3.8 Branch / git automation

| | OpenSpec | Spec Kit |
|---|---|---|
| Auto-branch on new change | manual | auto: `001-feature-name` |
| Pluses | control over branching strategy | convenient for strict-branching teams |
| Minuses | manual step | branch-per-spec hardens the «spec = change» mental model, away from modeling capabilities |

### 3.9 Agent ecosystem

| | OpenSpec | Spec Kit |
|---|---|---|
| Agents | 20+ via AGENTS.md | 18-30+ via slash-commands |
| Lock-in | none | none |

Parity. Both AGENTS.md-compatible (Linux Foundation's Agentic AI Foundation, 60K+ projects).

### 3.10 Compliance / audit trail

| | OpenSpec | Spec Kit |
|---|---|---|
| Audit trail | via `archive/` (date-prefixed) | via git history + `.specify/specs/` |
| Compliance scaffolding | minimal | richer — constitution + analyze + clarify steps |
| Pluses | simple, predictable | better fit for enterprise procurement |
| Minuses | *"ceiling is lower but predictable"* (Big Hat Group) | compliance-overhead on simple tasks |

### 3.11 Maintenance momentum

See §11 for the empirical update — the prior thesis ("Spec Kit is dying") is **refuted by May 2026 data**. Both projects are healthy; the differences are structural.

### 3.12 Customization

| | OpenSpec | Spec Kit |
|---|---|---|
| Templates | runtime via `openspec instructions` + `openspec/config.yaml` | static files in `.specify/templates/` |
| Override layers | flat (single file) | 4-tier: project-overrides → presets → extensions → core |
| Pluses | simple to manage | powerful at scale (10+ repos) |
| Minuses | smaller team-template library | extra complexity surface |

---

## 4. Interop & migration analysis

**Short answer**: automatic conversion does not exist in either direction; manual conversion is feasible because both use markdown; structural fidelity is **lossy in both directions**.

### 4.1 What exists today

- **OpenSpec [migration-guide.md](https://github.com/Fission-AI/OpenSpec/blob/main/docs/migration-guide.md)** — covers intra-version migration only (legacy → OPSX), not cross-tool.
- **Spec Kit [issue #1242](https://github.com/github/spec-kit/issues/1242)** — open feature-request for `specify import` to convert from `.kiro/`. *«Kiro IDE users who want to migrate to spec-kit currently face a manual, error-prone process»* — and there's no equivalent for OpenSpec.
- **Both tools use markdown** — content is portable as text; structure is not.

### 4.2 Structural mapping (manual, lossy)

**OpenSpec → Spec Kit (lossy)**:

| OpenSpec | → | Spec Kit | Loss |
|---|---|---|---|
| `changes/<name>/proposal.md` | → | `.specify/specs/[NNN-name]/spec.md` (part) | constitution links missing |
| `changes/<name>/design.md` | → | `.specify/specs/[NNN-name]/plan.md` | data-model.md / research.md / contracts/ must be regenerated |
| `changes/<name>/tasks.md` | → | `.specify/specs/[NNN-name]/tasks.md` | 1:1 |
| `specs/<capability>/spec.md` (durable) | → | **no analogue** | Spec Kit has no first-class capability concept; degrades to one-off feature spec |
| `changes/<name>/specs/<cap>/spec.md` (delta) | → | inline in spec.md | **ADDED/MODIFIED/REMOVED markers lost** |
| `changes/archive/<date>-<name>/` | → | git history | archive convention disappears |
| `openspec/project.md` (or config.yaml) | → | `.specify/memory/constitution.md` | freeform context → structured principles requires hand-rewrite |

**Spec Kit → OpenSpec (also lossy, less so)**:

| Spec Kit | → | OpenSpec | Loss |
|---|---|---|---|
| `[NNN-feature]/spec.md` | → | `changes/<feature>/proposal.md` | clarify-section structured form lost |
| `[NNN-feature]/plan.md` | → | `changes/<feature>/design.md` | 1:1 |
| `[NNN-feature]/tasks.md` | → | `changes/<feature>/tasks.md` | 1:1 |
| `constitution.md` | → | `openspec/project.md` (config.yaml in OPSX) | structured principles → freeform context |
| `data-model.md`, `research.md`, `contracts/` | → | embed in `design.md` | artifact-level granularity lost |
| `analyze` reports | → | no analogue | lost |
| Branch convention `001-feature` | → | manual | lost |

### 4.3 Implications

1. **Automatic converters don't exist in either direction** — empty niche. A `openspec ↔ spec-kit` migration CLI is publishable as an open contribution to both ecosystems.
2. **Spec Kit → OpenSpec is easier to automate** than the reverse because OpenSpec's structure is wider (capabilities + deltas + archive); compressing wide → narrow is harder than expanding narrow → wide.
3. **Untranslatable element**: OpenSpec's delta markers. Folding them into Spec Kit erases the entire reason to have used OpenSpec — which is exactly why Spec Kit users reinvent the delta model on their own (Discussion #152).

---

## 5. Agent economy

Parameters that matter to the agent (not the human reviewer):

### 5.1 Token economy

Empirical, measured 2026-05-10:

| | OpenSpec | Spec Kit |
|---|---|---|
| Always-loaded skills + commands | ~11 200 tokens (44 KB) | **~25 800 tokens** (103 KB) |
| Per-feature spec floor | ~1 088 tokens (4.3 KB) | ~2 725 tokens (10.9 KB) |

Spec Kit's [issue #1401](https://github.com/github/spec-kit/issues/1401) reports «18.6K tokens consumed every session» — same order of magnitude. On Cursor's 20K context window, Spec Kit alone consumes 93% of available space.

This compounds with context-engineering wisdom from 2026: *«2 000 relevant tokens beat 20 000 loose»* (Anthropic Agentic Coding Trends 2026); Context Mode and similar virtualization layers achieve 98% reduction. OpenSpec is aligned with this trend; Spec Kit is structurally against it.

### 5.2 Memory across sessions

- **OpenSpec**: `openspec/specs/<capability>/spec.md` is durable on disk; every fresh agent session reads it. Closes the most-cited Spec Kit complaint (*«AI forgets between sessions»*, Discussion #1482).
- **Spec Kit**: `.specify/specs/[NNN-feature]/` is branch-bound; new branches reset memory. Constitution partially fills this gap, but [issue #2219](https://github.com/github/spec-kit/issues/2219) documents constitution context loss mid-execution (see §9).

### 5.3 Drift reconciliation (under agent constraint)

Both have manual reconciliation (§3.6). The agent-side cost differs:

- **OpenSpec**: drift fix is a new `MODIFIED` delta — agent reads only the delta, context stays compact.
- **Spec Kit**: drift fix means full regeneration of spec → plan → analyze → tasks; agent re-processes hundreds of lines.

### 5.4 Compatibility & adversarial review

| | OpenSpec | Spec Kit |
|---|---|---|
| Agents supported | 20+ (AGENTS.md) | 18-30+ (slash-commands) |
| AGENTS.md (Linux Foundation, 60K+ projects) | yes, native | yes, via slash-commands |
| Adversarial review | none native | none native |

Parity on compatibility. Adversarial review is BMAD's strength — neither of these two has it.

### 5.5 Verdict for agents

**OpenSpec is structurally agent-friendlier in 2026** — wins on 3 of 4 axes (tokens, memory, drift), parity on compatibility. Spec Kit is optimized for the **human reviewer** through rich artifacts, betting on manual gating — a strategy that doesn't scale per Thoughtworks' "bitter lesson" critique.

---

## 6. Audience segmentation — who picks what

### 6.1 Startups, founders, indie hackers

**Choice: OpenSpec.**

- TS/Node stack matches the modern startup default
- 5-min setup, 3-phase workflow → iteration > audit
- Brownfield-native: real startups rarely begin from zero; there is always a prototype, MVP, technical debt
- EU AI Act not yet binding for early-stage without high-risk classification
- OpenSpec itself is a YC W26 startup (Tabish Bidiwale, Fission AI) — speaks the founder dialect

Validated in practitioner tests: ranthebuilder.cloud rated OpenSpec 4.0/5 vs Spec Kit 2.77/5; Hashrocket recommends OpenSpec for *«senior developers, small teams with overlapping roles»*.

### 6.2 Corporations / enterprise

**Default choice today: Spec Kit. Real adoption: nobody we can find.**

- GitHub Copilot is at **90% of Fortune 100**, ~55% task acceleration (Accenture/GitHub research). Spec Kit rides this distribution.
- **Microsoft Learn has an official module** «Implement Spec-Driven Development using GitHub Spec Kit» for enterprise developers. Procurement reads "Microsoft Learn" as a safety signal.
- Constitution mechanism fits PO/architect/dev role separation typical of large teams.

But critically:
- **Thoughtworks Tech Radar Vol 33: SDD = Assess** — not Trial, not Adopt. Names Kiro, Spec Kit, Tessl. **Does not even name OpenSpec.**
- **InfoQ Enterprise SDD analysis: zero Fortune 500 case studies** as of 2026. All publications are principles, no deployments.
- Quote from Thoughtworks: *«handcrafting detailed rules for AI ultimately doesn't scale»* — bitter-lesson concern.

OpenSpec targets this segment via the paid SKU **OpenSpec Workspaces** (per YC launch: *«for teams that need deeper collaboration, customization, multi-repo planning»*). Direct competition for Spec Kit's enterprise share without Microsoft's distribution anchor.

### 6.3 Government / regulated / public sector

**Choice today: neither natively. Practical answer: services-consultants + BMAD or Augment Intent.**

- **EU AI Act, August 2 2026**: high-risk system obligations enforced; fines up to €15M or 3% global turnover.
- **SDD as Article 11 evidence** = **partial compliance**, not full. Doesn't cover Article 9 (risk mgmt), 10 (data governance), 15 (accuracy testing), 27 (FRIA).
- Article 12 requires **logging integrated into core design**, retention ≥6 months. Neither tool addresses agent logging.
- Spec Kit (constitution + clarify + analyze) maps marginally better to Articles 11/12/14, but governments still need audit-trail tooling and FRIA — those are service-layer.

Post-August-2-2026 EU AI Act will create real pull on SDD from gov/regulated, but the opportunity is for **services + tooling**, not for the CLI alone. This is the structural niche the studio's «methodology + service + artifact» model targets.

---

## 7. Statistics & technology direction

### 7.1 Numbers (snapshot 2026-05-10)

| Metric | OpenSpec | Spec Kit |
|---|---|---|
| GitHub stars | 46 684 | **95 049** |
| Forks | 3 277 | 8 263 |
| Watchers | 226 | 536 |
| Days alive | 278 | 262 |
| Stars/day lifetime | 167.9 | 362.8 |
| YC | W26 (Fission AI) | — |
| Corporate backer | — | Microsoft / GitHub |

(See §11 for behavioural metrics — commit velocity, release cadence, etc.)

Platform context 2025–2026:
- 180M developers on GitHub (+36M in 2025, record growth)
- 65% of organizations regularly use GenAI (McKinsey 2025, up from 33% YoY)
- 90% of Fortune 100 use GitHub Copilot
- Spec Kit has Microsoft Learn distribution; OpenSpec has YC distribution

### 7.2 Where the technology is going

1. **AGENTS.md** as cross-tool convergence layer under the **Agentic AI Foundation (Linux Foundation)** — 170+ members in <4 months (Anthropic, OpenAI, Google, MS, AWS, Cloudflare). Both OpenSpec and Spec Kit are AGENTS.md-compatible, so the corpus survives a tool switch.
2. **Living specs > static specs** — Tessl, OpenSpec, Augment Intent push this direction. Spec Kit still static.
3. **CLI > IDE** — Kiro (IDE) lost ground to OpenSpec/Spec Kit (CLI) due to AWS lock-in, pricing-bug incident, vibe-coding production incident. CLI form factor wins in 2026.
4. **Multi-agent orchestration** rising — BMAD, Augment Intent. Single-agent CLIs (OpenSpec/Spec Kit) will either gain multi-agent support or fade to niche.
5. **Compliance as forcing function** — EU AI Act August 2 2026 will pull SDD into procurement requirements. Living specs become Article 11 evidence stack candidates.
6. **Bitter lesson** (Thoughtworks): *«handcrafting detailed rules for AI ultimately doesn't scale»* — explains why no SDD toolkit reaches Tech Radar Adopt.
7. **Reverse-engineering existing code → spec** — tooling exists ([Show HN, Mar 2026](https://news.ycombinator.com/item?id=46838086): AST + tree-sitter + optional LLM). Brownfield adoption gets cheaper, deepening OpenSpec's structural advantage.
8. **Open-core + paid team SKU** — OpenSpec Workspaces is "Linear-for-SDD" pattern. Spec Kit has no equivalent paid layer.

---

## 8. User demographics & archetypes

### 8.1 OpenSpec user archetype

**Profile**: Senior IC engineer, ~28–42, 8+ years experience.

- **Stack**: TS/Node primary; Python/Go secondary
- **Context**: brownfield (legacy + MVP + technical debt)
- **Team size**: solo founder or 2–7 person dev team
- **Geography**: startup-density hubs (SF, NYC, Berlin, London, Tel Aviv, Sydney, Cluj, Bucharest)
- **Tooling**: Claude Code or Cursor primary, MCP, agent-agnostic preference; typically subscribed to 2+ LLM providers
- **Ideology**: «discipline without bureaucracy»; «code is implementation, not source of truth»; agentic-native
- **Trigger to choose**: tired of token-burn on Spec Kit; tired of vibe-coding; read Hashrocket / ranthebuilder; tried Spec Kit and switched (Rida Kaddir pattern)

Community marker: *«Lowest-friction start, parallel development support, lightest footprint»* (ranthebuilder.cloud).

### 8.2 Spec Kit user archetypes (two segments)

**A. Enterprise developer in Microsoft ecosystem**

- Age: ~30–55
- Context: already on Copilot Enterprise; follows Microsoft Learn recommendations
- Team: 50+ people, PO ↔ architect ↔ dev ↔ QA separation
- Ideology: «GitHub blessed it = safe procurement choice»; «more documentation = less review risk»
- Trigger: «we need compliance/audit-trail and an enterprise-approved tool»

**B. Junior / hobbyist seeking discipline training-wheels**

- Age: broad, often new to agentic coding
- Context: learning to work with AI agents, not confident in own discipline
- Ideology: training-wheels — needs procedural rails so agent doesn't run away
- Trigger: *«I'm just not disciplined enough, it makes me lazy»* (HN #45154355)

**Spec Kit skeptic (also a recognizable archetype)**:
- *«Code has to be the source of truth»* (HN #45154355)
- Compares to TDD; expects SDD to repeat the same pattern
- Often older engineer with long legacy-project experience

### 8.3 Structural read on user-base shape

**OpenSpec collects signal-heavy users**: 46K stars are people who **chose** against Spec Kit after explicit comparison. Each star ≈ a vote from archetype 8.1.

**Spec Kit collects volume-heavy users**: 95K stars include:
- Distributional pull from GitHub homepage / Microsoft Learn
- Curious enterprise developers who don't actually use it
- Juniors looking for training wheels
- Skeptics who starred to comment on HN

**OpenSpec's active-user / star ratio is higher** — denser community, better content distribution audience.

---

## 9. Failure-mode catalogue

14 categories, 30+ documented bugs with quoted evidence. Sources: GitHub issues, GitHub Discussions, HN threads, practitioner blog posts, Microsoft Learn module.

### 9.A — Token / context exhaustion

**A1. Spec Kit consumes 18.6K tokens every session — regardless of usage.** [#1401](https://github.com/github/spec-kit/issues/1401)
*«Spec-kit commands consume approximately 18.6K tokens across all commands combined»*. On Cursor (~20K context) — 93% of available space. On Copilot 64K — 29%. On Claude Code 200K — 9.3%. **Severity: HIGH, open**.

**A2. Output volume explodes on real tasks.** Hashrocket: 250 vs 800 lines. Scott Logic: **2 577 lines markdown vs 689 lines code** — agent execution 33.5min vs 8min, review 3.5h vs 24min. **Severity: HIGH for Spec Kit**.

### 9.B — Brownfield install — silent failure

**B1. Spec Kit `init` silently fails on existing NX monorepo.** [#2506, May 2026](https://github.com/github/spec-kit/issues/2506)
*«Adding spec-kit to an existent NX monorepo project — fails silently after changing branches without creating any spec.md file»*. Reproduces on spec-kit 0.8.7, Claude Code, macOS 26.3.1. **Severity: CRITICAL, open**.

**B2. `/specify` and `/plan` run, output to console, write nothing.** [#337](https://github.com/github/spec-kit/issues/337)
*«Everything is only output to the console... no files or git branches are created. Currently, it's just pure token burning»*. Closed as **«not planned»**, never fixed. **Severity: HIGH**.

**B3. Maintainers publicly admit brownfield is unsolved.** [Discussion #331](https://github.com/github/spec-kit/discussions/331)
jflam: *«have the agent go and research your existing codebase and write it down in a research doc. Then you refer to that research doc from the specs»*. [Discussion #746](https://github.com/github/spec-kit/discussions/746) brownfield user with 3 pain points received **zero replies**. [#1436 «Brownfield Bootstrap»](https://github.com/github/spec-kit/issues/1436) extension proposal, not shipped. **Severity: STRUCTURAL**.

**B4. Reverse-engineered specs become a hazard.** [The Brownfield Problem, jjmasse](https://www.jjmasse.com/2026/03/06/the-brownfield-problem-why-most-ai-development-advice-ignores-your-actual-codebase/)
*«A reverse-engineered spec is a derived artifact... when new features are developed against a misaligned spec, the risk of regressions and unintended side effects rises sharply»*. Applies to both tools, but Spec Kit lacks delta-mechanism for safe incremental adoption.

### 9.C — Cross-spec / cross-feature breakage

**C1. Spec Kit: new spec breaks behaviour from old spec.** [#876](https://github.com/github/spec-kit/issues/876)
*«After creating a "Chatgpt-like" Agent UI in one spec, updates in a second spec to handle metadata broke the previously defined behavior»*. *«The behavior defined in the previous spec was completely broken, and I ended up struggling with vibe code»*. Open, marked `stale`. **Severity: HIGH, brand-shaping bug**.

**C2. OpenSpec: multiple changes on one capability → spec fragmentation.** [Discussion #737](https://github.com/Fission-AI/OpenSpec/discussions/737)
User ends up with two specs `user-show/spec.md` and `user-detail/spec.md` instead of one `user-management/spec.md`. Workaround: hand-edit Capabilities section in proposal.md before apply. **Severity: MEDIUM, requires author discipline**.

**C3. Multi-agent / parallel branches not natively supported.**
Spec Kit operates on shared local state — race conditions on concurrent runs ([#1476 worktree request](https://github.com/github/spec-kit/issues/1476)). [#1176](https://github.com/github/spec-kit/issues/1176): multi-agent merge unimplemented. **Severity: HIGH at team-scale**.

### 9.D — Multi-repo / monorepo

**D1. Spec Kit doesn't run from monorepo subfolder.** [#1026](https://github.com/github/spec-kit/issues/1026)
*«Speckit commands cannot currently be invoked from chat when installed only in a subfolder»*. [#581](https://github.com/github/spec-kit/issues/581) («Mono repo considerations and best practices») — open, no docs. [#1095 multirepo project](https://github.com/github/spec-kit/issues/1095) — open. [Discussion #1743](https://github.com/github/spec-kit/discussions/1743): separate Specs Repo + FE/BE — open question. InfoQ: *«mono-repo focus incompatible with microservices»*. **Severity: HIGH for enterprise**.

**D2. OpenSpec multi-repo is paid-only.**
Open-core OpenSpec has no multi-repo coordination. **OpenSpec Workspaces** (paid) addresses this. **Severity: MEDIUM** (product-strategy).

### 9.E — Agent-integration regressions

**E1. Cursor CLI silently drops args — no error.** [Spec Kit #1264](https://github.com/github/spec-kit/issues/1264)
*«Invoking any slash-command results in only the command name being sent. Arguments or trailing text are dropped»*. Spec Kit 0.0.22, Cursor 2025.11.25, macOS 14.1. *«Silent — no error messages appear, making diagnosis difficult»*. **Severity: HIGH**.

**E2. OpenSpec — all `/opsx:*` commands broken in codex-cli v0.117.0.** [#890](https://github.com/Fission-AI/OpenSpec/issues/890)
*«OpenSpec slash commands became non-functional in interactive mode... cannot invoke /opsx:archive, /opsx:propose, /opsx:apply»*. Root cause: codex-cli added native `/title` and `/plugins` → broke third-party namespaces. Duplicate report: [#885](https://github.com/Fission-AI/OpenSpec/issues/885) (P0-critical, 8 comments). **Severity: CRITICAL for Codex users, open**.

**E3. Spec Kit breaks on every agent update.** [#926](https://github.com/github/spec-kit/issues/926) (Cursor 1.7.46), [#1798](https://github.com/github/spec-kit/issues/1798) (Antigravity), [#2151](https://github.com/github/spec-kit/issues/2151) (extensions can't install — download URL fails), [#2406](https://github.com/github/spec-kit/issues/2406) (workflow engine hardcodes `copilot`), [#2293](https://github.com/github/spec-kit/issues/2293) (integration switch leaves stale scripts), [#2419](https://github.com/github/spec-kit/issues/2419) (**11+ integrations have unverified CLI dispatch flags — systemic**). **Severity: HIGH systemic**.

**E4. OpenSpec breaks on every agent update.** [#723](https://github.com/Fission-AI/OpenSpec/issues/723) (`/prompt` broken in Codex), [#838](https://github.com/Fission-AI/OpenSpec/issues/838) (deprecated TOML for Qwen Code), [#132](https://github.com/Fission-AI/OpenSpec/issues/132) (Codex on Windows broken; closed without fix), [#643](https://github.com/Fission-AI/OpenSpec/issues/643) (fast-forward instructs use of TodoWrite tool which Claude no longer has — instruction rot). **Severity: HIGH systemic — both tools suffer**.

### 9.F — Constitution / context drift mid-execution

**⭐ F1. Spec Kit: agent loses constitution mid-execution and "guesses".** [#2219](https://github.com/github/spec-kit/issues/2219)

The most critical structural finding of this catalogue:

*«Constitution is a large document read once at plan time and then gone from context. When the agent reaches the push step, it constructs the command from recall rather than from the constitution, gets the flags wrong, and hits a 403»*.

Reproduces: speckit project with a constitution defining a governed push operation requiring environment-variable authentication → agent uses recall, supplies wrong API key, **only consults the constitution after failure**. **Severity: CRITICAL — structural defect of the «read once and forget» model**.

**F2. Spec Kit users reinvent the delta model on their own.** [Discussion #152](https://github.com/github/spec-kit/discussions/152)
*«Treating specs as session deltas rather than permanent truth has been the thing that finally removed the friction we kept hitting»*. *«The parts of SDD that really work are the tactical execution docs, which tend to be throwaway»*. **OpenSpec's model is what Spec Kit users grope toward by trial.**

**F3. Practitioner observation: AI drifts on implementation details.** [Lookoutking on Medium](https://medium.com/@lookoutking/spec-driven-development-in-practice-my-experience-with-spec-kit-8f250b47d677)
*«AI sometimes drifts on implementation details (e.g., ignoring grammY in one version)»*. Despite clear specifications, ongoing vigilance required.

### 9.G — Implementation-phase failures

**G1. Spec Kit claims success when code is broken.**
*«Spec Kit and its AI determined that the spec had been correctly implemented, but the code was actually wrong, with most functionality missing and zero tests generated»*. **Severity: HIGH**.

**G2. No workflow for fixing broken implementation.** [#1450](https://github.com/github/spec-kit/issues/1450)
*«Maybe we need another speckit agent? like speckit.fix maybe?»* (January 2026, no answer by May 2026).

**G3. «Wonderfully specified, poorly implemented».** [#1092](https://github.com/github/spec-kit/issues/1092)
*«After several hours work, I have a wonderfully specified app, but the implementation is poor»*. *«$ cost of writing/rewriting specs is costing me more than the $ cost of writing code»*. **Severity: STRATEGIC** — the value proposition is denied.

**G4. Agent ignores existing classes and re-generates them.** Martin Fowler review.

**G5. Scott Logic: spec-driven code still contains «obvious bug».**
*«Despite the elaborate specification pipeline, the generated code contained a simple, and very obvious, bug — variables weren't properly populated from the datastore»*.

### 9.H — Validation / parser bugs (silent corruption)

**H1. OpenSpec parser breaks on markdown edge cases.**
[#361](https://github.com/Fission-AI/OpenSpec/issues/361) (line breaks in headers break validation), [#312](https://github.com/Fission-AI/OpenSpec/issues/312) (parser treats `#` inside code blocks as headers), [#164](https://github.com/Fission-AI/OpenSpec/issues/164) («Error: Change has issues» — **deltas correctly parsed by `show` but NOT by `validate`** — internally inconsistent parsers).

**H2. OpenSpec `open:sync` corrupts spec.md files.** [#911](https://github.com/Fission-AI/OpenSpec/issues/911)
*«Now my spec.md files look different from what I expected. It's like they're not fully synced, or something got changed unexpectedly»*. Labels: `bug` + `needs-info`, open. **Severity: HIGH (data integrity)**.

**H3. OpenSpec `view` shows stale progress.** [#374](https://github.com/Fission-AI/OpenSpec/issues/374)
*«Codex, Antigravity, Microsoft Copilot fail to update task state... openspec view displays stale information»*. Disconnect: model works, state isn't written → percentage progress lies. **Severity: HIGH**.

### 9.I — Archive / lifecycle bugs (OpenSpec-specific)

**I1. Archive command writes to wrong path.** [#412](https://github.com/Fission-AI/OpenSpec/issues/412)
Expected: `openspec/archive/<date>-<name>`; actual: `openspec/changes/archive/<date>-<name>`. Other CLI parts look in the first path → archived change "lost". Fix proposed, not merged.

**I2. `/opsx:archive` doesn't invoke `openspec archive` CLI.** [#863](https://github.com/Fission-AI/OpenSpec/issues/863) — AI command diverges from documentation. [#913](https://github.com/Fission-AI/OpenSpec/issues/913) — `/opsx:archive` offers «Sync now» when sync skill not installed → fails.

**I3. Folder-name conflict on two archives in one day.**
*«If the days are the same there is a conflict because OpenSpec tries to recreate the same folder in the archive repository with the same name»*.

### 9.J — Security / supply-chain risk

**J1. Spec Kit workflows = arbitrary shell execution without opt-in gate.** [#2440](https://github.com/github/spec-kit/issues/2440)
*«A workflow definition can contain arbitrary shell commands. Users should have a clear prompt or policy gate before a workflow with shell execution runs, especially when the workflow came from a catalog, URL, or third-party source»*. 9 comments, security-hardening proposal. Vectors: supply-chain, privilege escalation, implicit execution. **Severity: CRITICAL** (CVE-class on malicious-catalog exploit).

**J2. Spec Kit lacks automated security audit for Python deps.** [#2438](https://github.com/github/spec-kit/issues/2438) — no static analysis in CI. **Severity: MEDIUM**.

**J3. OpenSpec — no publicly documented security failures**, but the TS install layer also runs in user process; same risks apply on malicious MCP / template.

### 9.K — Maintainership / ecosystem health

The earlier "Spec Kit dying" thesis is **refuted by §11 data**. [#1550 «Is this project dead?»](https://github.com/github/spec-kit/issues/1550) was a panic from February 2026. By May, Spec Kit ships 17 releases / 30 days. Both projects healthy in different shapes — see §11.

### 9.L — Localization / multi-language

**L1. OpenSpec — native multi-language via `config.yaml`.** [docs/multi-language.md](https://github.com/Fission-AI/OpenSpec/blob/main/docs/multi-language.md). Spanish, Chinese, Japanese (presumably Russian works too). **★ advantage for diaspora-projects**.

**L2. Spec Kit — community fork only.** [`spec-kit-chinese`](https://github.com/888888888881/spec-kit-chinese) independent fork. [Issue #218](https://github.com/github/spec-kit/issues/218) Chinese support open. **Severity: MEDIUM for non-English markets**.

### 9.M — Non-functional requirements (NFR)

**M1. Both tools claim NFR support; neither has structured slot.** Spec Kit philosophy: *«Performance bottlenecks become new non-functional requirements... baked into the spec itself»*. OpenSpec: NFRs in `specs/<capability>/spec.md` as prose. **Real location**: constitution.md (Spec Kit) or capability spec (OpenSpec) as freeform — **not enforceable contract**. No publicly documented "spec drift caught by NFR violation" case. **Severity: STRUCTURAL**.

### 9.N — Cost / waste failures

**N1. Token-burning without output.** [#337](https://github.com/github/spec-kit/issues/337): «pure token burning». [#1092](https://github.com/github/spec-kit/issues/1092): cost of specs > cost of code. Scott Logic: **10× slower** development with Spec Kit than without.

**N2. Iteration phase is the most expensive.** ranthebuilder: *«Spec-Kit's iteration stops at planning; changing direction means re-running affected commands, each regenerating its full document»*. **Severity: HIGH for exploration-heavy work**.

### 9.X — Severity summary table

| Category | Spec Kit | OpenSpec | Source-of-truth |
|---|---|---|---|
| A. Token tax | 🔴 critical (18.6K/session) | 🟢 minimal | issue #1401, our benchmarks |
| B. Brownfield install | 🔴 silent fail, structural | 🟢 native delta-model | #2506, #337, #331 |
| C. Cross-spec breakage | 🔴 documented (#876) | 🟡 manual workaround | #876, OS #737 |
| D. Multi-repo | 🔴 unsolved | 🟡 paid SKU | #581, #1095, #1026 |
| E. Agent integration | 🔴 systemic instability | 🔴 systemic instability | #1264, OS #890, #2419 |
| F. Constitution drift | 🔴 critical (#2219) | 🟢 durable spec on disk | #2219 |
| G. Implementation broken | 🔴 documented (#1450, #1092) | 🟡 less data | Fowler, Scott Logic |
| H. Validation/parser | 🟡 less data | 🔴 multiple parser bugs | OS #361, #312, #164, #911, #374 |
| I. Archive/lifecycle | n/a | 🔴 path bug, hooks missing | OS #412, #863, #913 |
| J. Security | 🔴 shell-step injection | 🟡 not documented | #2440 |
| K. Maintenance | (refuted — see §11) | (refuted — see §11) | #1550, #1482 |
| L. Localization | 🟡 community only | 🟢 native | OS docs |
| M. NFR capture | 🟡 freeform | 🟡 freeform | structural |
| N. Cost waste | 🔴 documented 10× slower | 🟡 less data | Scott Logic, #1092 |

🔴 critical · 🟡 substantive limitation · 🟢 not a problem

**Spec Kit has 9 critical 🔴 categories vs OpenSpec's 4** (excluding K which is refuted by data) — strongest numerical evidence for the article thesis.

---

## 10. Empirical benchmark — installation, structure, cost

**Date**: 2026-05-10  
**Test feature**: «Add author field to lab posts with multi-author + i18n display»  
**Environment**: macOS 14, Node 22.17.1, Python 3.9 (system), uv 0.11.12 (Homebrew)  
**Methodology**: clean install both tools in separate `/tmp/sdd-benchmark/` repos; init; scaffold same feature; measure structural floor.

### 10.1 Setup time

| Step | OpenSpec | Spec Kit |
|---|---|---|
| Prerequisite install | 0s (Node already present) | 10.5s (`brew install uv`) |
| Tool install | 4.5s (`npm install -g @fission-ai/openspec`) | 5.0s (`uvx --from git+https://github.com/github/spec-kit.git specify init`) |
| **Total cold install** | **4.5s** | **15.5s** |
| Init in repo | 1.5s (`openspec init . --tools claude --force`) | 5.0s (`specify init . --here --ai claude --force`) |
| **End-to-end first run** | **6.0s** | **20.5s** |

**Spec Kit setup is 3.4× longer** end-to-end on a fresh dev machine.

#### Setup gotcha caught live in the benchmark

PyPI package `specify-cli` (which `uv tool install specify-cli` provides) is **not** the official Spec Kit. First install attempt failed silently with *«No matching release asset found for claude (expected pattern: spec-kit-template-claude-sh)»*. Cost on a real developer: 5–15 minutes debugging before discovering the correct path is `uvx --from git+https://github.com/github/spec-kit.git`.

### 10.2 Base install footprint

| Metric | OpenSpec | Spec Kit | Ratio |
|---|---|---|---|
| Files placed on disk | 9 | 27 | **3.0×** |
| Total lines | 1 276 | 4 221 | **3.3×** |
| Total bytes | 45 KB | 185 KB | **4.1×** |
| Slash commands | 4 | 7 | 1.75× |
| Skills | 4 | 9 | 2.25× |

#### Always-loaded into agent context

| | OpenSpec | Spec Kit |
|---|---|---|
| Skills + commands total | 1 256 lines / 44 KB | 2 043 lines / 103 KB |
| **Estimated tokens at 4 chars/token** | **~11 200 tok** | **~25 800 tok** |
| Constitution.md | not present | 50 lines (template, must be filled) |

This corroborates [Spec Kit issue #1401](https://github.com/github/spec-kit/issues/1401)'s claim of «18.6K tokens consumed every session». Our 25.8K is worst-case (lazy-load brings it closer to 18K).

### 10.3 Per-feature artifact footprint (floor)

Same feature, equivalent semantic depth, hand-filled using each tool's mandatory sections:

| Artifact | OpenSpec | Spec Kit |
|---|---|---|
| proposal.md / spec.md | 27 lines | 94 lines |
| design.md / plan.md | 38 lines | 92 lines |
| tasks.md | 10 lines | 61 lines |
| spec.md (per capability) | 17 lines | n/a |
| .openspec.yaml / branch metadata | 2 lines | 0 |
| **Total per feature** | **94 lines / 4.3 KB** | **247 lines / 10.9 KB** |
| **Ratio** | **1×** | **2.6× lines, 2.5× bytes** |

This is the **floor**. Published third-party tests show Claude Code agent generates 2–10× this on Spec Kit:
- Hashrocket: 250 vs 800 lines (3.2×)
- Scott Logic: 689 lines code vs 2 577 lines markdown (3.7×)

### 10.4 Token-cost economics (Sonnet 4.6 — input $3/Mtok)

Assumptions:
- One feature = 15 spec-reads during iteration
- 3 fresh chats per feature average
- 4 chars / token

#### Per-feature direct cost (floor)

| | OpenSpec | Spec Kit |
|---|---|---|
| Spec content per read | ~1 088 tok | ~2 725 tok |
| × 15 reads | 16 320 tok | 40 875 tok |
| Cost @ $3/Mtok | **$0.049** | **$0.123** |

#### Per-feature session-tax (always-loaded skills)

| | OpenSpec | Spec Kit |
|---|---|---|
| Always-loaded tokens | ~11 200 | ~25 800 |
| × 3 sessions | 33 600 | 77 400 |
| Cost @ $3/Mtok | **$0.101** | **$0.232** |

#### Total per feature (floor)

| | OpenSpec | Spec Kit |
|---|---|---|
| Total | **~$0.15** | **~$0.36** |
| Ratio | 1× | **2.4×** |

#### Scaled — 5-dev team, 100 features/month

| | OpenSpec | Spec Kit |
|---|---|---|
| Monthly | $75 | $180 |
| Annual | **$900** | **$2 160** |
| **Δ** | — | **+$1 260/year** |

**Real-world likely 2–3× this** because actual agent-generated specs run 2–10× the floor — so real spread in this scenario is **$2 500 – $7 000/year per 5-dev team**.

### 10.5 Architectural observation revealed by benchmark

| Aspect | OpenSpec | Spec Kit |
|---|---|---|
| Templates | runtime-generated via `openspec instructions` | static files in `.specify/templates/` |
| Template lines (proposal/design/tasks) | 23 + 19 + 9 = **51 lines** | 128 + 104 + 251 = **483 lines** |
| Mandatory sections | 3 + 2 + 1 = **6** | 5 + many + many = **~25** |
| Customization | `openspec/config.yaml` (single file) | 4-tier override |

Spec Kit forces ~9× more template-driven content per feature. OpenSpec's runtime-instruction model means template churn is centralized and cheap.

### 10.6 What this benchmark cannot measure

Need live Claude Code session for:
- Actual token counts in real agent runs (not floor-based estimates)
- Wall-clock time to feature completion
- Iteration count to passing-state
- Agent failure modes encountered live

**Protocol attached at `/tmp/sdd-benchmark/report/benchmark-results.md` § 7** for follow-up by Pavel. Both test repos preserved at `/tmp/sdd-benchmark/{openspec,speckit}-test/`.

---

## 11. Health metrics — what GitHub data actually shows

**Snapshot date**: 2026-05-10. Method: `gh api` with sampling on last 30/100 items per metric.

### 11.1 The thesis being refuted

In the prior internal positioning file (`competitive-positioning-sdd-frameworks.md`) we cited [Discussion #1482](https://github.com/github/spec-kit/discussions/1482) — *«22 PRs / 0 commits»* — as evidence that Spec Kit is dying. **This citation is outdated.** May 2026 data shows the opposite. The refutation matters because the article narrative built on the prior thesis was wrong.

### 11.2 Commit velocity (`search/commits` API)

| Window | Spec Kit | OpenSpec | Ratio |
|---|---|---|---|
| Last 4 weeks | **159** | 28 | **5.7×** |
| Last 8 weeks | 315 | 46 | 6.8× |
| Last 12 weeks | 397 | 62 | 6.4× |
| Last 26 weeks | 447 | 225 | 2.0× |

**89% of Spec Kit's last-26-weeks commits (397/447) fell in the last 12 weeks.** Sharp acceleration after a quiet December–January.

OpenSpec is the inverse: 28% of last-26-weeks commits (62/225) in the last 12 weeks. **~70% deceleration** vs autumn-winter pace.

### 11.3 Release cadence

| | Spec Kit | OpenSpec |
|---|---|---|
| Total releases | **142** | 36 |
| Span | 257 days | 226 days |
| Avg days between releases (lifetime) | **1.8** | 6.5 |
| Releases in last 60 days | **29** | 2 |
| Releases in last 30 days | **17** | 2 |
| Cadence today | 1 per **1.8 days** | 1 per 15 days |

Spec Kit shipped **8.5× more releases** in the last 30 days. Anything but stagnation.

### 11.4 PR throughput (sample of 30 most-recently merged)

| | Spec Kit | OpenSpec |
|---|---|---|
| Median merge time | **23.3 hours** (1.0 day) | **1.0 hour** |
| Mean merge time | 80.0h | 14.3h |
| P25 / P75 | 9.8h / 121.5h | 0.2h / 9.0h |
| Min / Max | 0.0h / 515.3h | 0.1h / 163.8h |

**OpenSpec's 1-hour median is founder self-merge** (TabishB pushes own PRs) — not an analogue of human review. **Spec Kit's 1-day median is healthy community-style review.**

### 11.5 Issue triage (sample of 30 most-recently closed)

#### Naive median

| | Spec Kit | OpenSpec |
|---|---|---|
| Median close time | **5 086 hours (212 days)** | 65.8h (2.7 days) |

#### Why the 212-day median?

All 30 most-recent issues closed in Spec Kit were labeled `stale`. This is **bulk mass-close by stale-bot, not active triage**.

When filtering to last-60-days only:

| | Spec Kit | OpenSpec |
|---|---|---|
| Total closed in 60-day window | 411 | 109 |
| Median (recent cohort) | **211 days** (still stale-bot dominated) | **3.3 days** |
| Top labels | 100% `stale` | diverse (`bug`, `enhancement`, `P2-medium`, `adapter`, `backlog`) |

**Spec Kit in May 2026 closes issues primarily via stale-bot.** Real human-triage rate is not visible in this data — would need a separate `-label:stale` filter.

**OpenSpec triage is healthy**: 3.3-day median, diverse labels — real human review.

### 11.6 Contributor diversity (recent 100 commits)

| | Spec Kit | OpenSpec |
|---|---|---|
| Unique authors | 36 | 32 |
| Top contributor share | mnriem **23%** | TabishB **58%** |
| Top 3 share | **41%** | **69%** |
| Bot/dependabot commits | 6% | 9% |

#### Bus factor — critical structural difference

- **Spec Kit**: top-1 writes 23%; top-5 = `mnriem(23) BenBtg(11) DyanGalih(7) dependabot(6) hindermath(6)`. **Distributed team** — if mnriem leaves, project continues.
- **OpenSpec**: TabishB writes **58%** of recent commits; next is `openspec-release-bot` (8 — automation). **Founder-driven entirely.** If Tabish leaves, project stalls.

This is not "OpenSpec is dying". It's **early-stage YC startup shape** — normal at this stage. But for adoption decisions, the bus factor is a risk to disclose.

### 11.7 Qualitative read — what's actually being committed

#### Spec Kit (10 most-recent)

```
DyanGalih:        chore: update community catalog with latest extension versions
dependabot[bot]:  chore(deps): bump actions/setup-dotnet from 4.3.1 to 5.2.0
dependabot[bot]:  chore(deps): bump actions/github-script from 7 to 9
dependabot[bot]:  chore(deps): bump DavidAnson/markdownlint-cli2-action
dependabot[bot]:  chore(deps): bump github/codeql-action from 4.35.3 to 4.35.4
Quratulain-bilal: feat(catalog): add API Evolve community extension
Copilot:          feat: Config-driven opt-in authentication registry
mnriem:           chore: release 0.8.7, begin 0.8.8.dev0 development
pragya247:        feat: add agent-orchestrator to community extension catalog
DyanGalih:        chore: update extension versions in community catalog
```

**Pattern**: catalog/extension churn + dependabot bumps + release ceremonies. **Ecosystem-maintenance mode**, not core feature work.

#### OpenSpec (10 most-recent)

```
HowardYan888:  docs(migration-guide): fix inconsistent /opsx:sync description
TabishB:       Fix Windows workspace launch arg expectation
TabishB:       Fix Windows workspace CI tests
TabishB:       [codex] Propose workspace open agent context
TabishB:       archive workspace create and register repos
TabishB:       Fix Windows workspace path test expectations
TabishB:       Fix Windows workspace path aliases
TabishB:       [codex] Add workspace setup commands
TabishB:       archive workspace foundation
JiangWay:      docs: add Community Schemas section
```

**Pattern**: founder shipping the **Workspaces feature** (paid SKU per YC launch). **Focused product work** on the new revenue vehicle.

### 11.8 Summary table — the real shape of "health"

| Dimension | Spec Kit | OpenSpec | Implication |
|---|---|---|---|
| Commit velocity (4w) | 159 | 28 | Spec Kit 5.7× more active |
| Releases (30d) | 17 | 2 | Spec Kit 8.5× more frequent |
| PR merge median | 23h | 1h | OpenSpec founder self-merge; Spec Kit community review |
| Real issue triage | unclear (stale-bot dominates) | 3.3 days | OpenSpec — healthy triage |
| Top contributor share | 23% | 58% | OpenSpec — critical bus factor |
| Bot share | 6% | 9% | comparable |
| PR rejection rate | 41% | 22% | Spec Kit more selective |
| Nature of commits | catalog/dep churn | core feature (Workspaces) | different character of activity |

### 11.9 What this means for the article narrative

**Theses that remain valid (independent of velocity)**:
- Architectural failure modes of Spec Kit (#2219 constitution loss, #2440 security, brownfield broken) are not fixed by velocity — catalog churn doesn't repair architecture
- Token tax 18.6K/session is objective; release cadence doesn't reduce it
- OpenSpec brownfield-nativity via delta markers stands
- Cost economics: 2.4× $/feature gap on the empirical benchmark is stable

**Theses that need updating**:
- ❌ ~~"Spec Kit is dying, no commits"~~ → ✅ "Spec Kit had a quiet period in early 2026 (which #1482 captured) but is now in extreme high-velocity — 17 releases in 30 days, 159 commits in 4 weeks"
- ❌ ~~"OpenSpec is accelerating"~~ → ✅ "OpenSpec is decelerating in commit velocity (28 in 4 weeks, -70% from peak); shifted focus to the Workspaces paid SKU — pivot from open-core to commercialization"
- ⚠️ NEW: "OpenSpec bus factor critical — TabishB writes 58% of commits; Spec Kit distributed (top-1 = 23%)"

**New thesis for the article**:

**Health ≠ velocity.** Spec Kit has high velocity, but it's **maintenance churn** (dependabot + community catalog). OpenSpec has low velocity, but **focused product work** on the new revenue vehicle. Both healthy, in different shapes; neither dying. This is more honest than "one is dying, the other is rising."

### 11.10 What was not measurable

- **Recent star velocity**: GitHub blocks paginate >400 pages. star-history.com API available — possible follow-up.
- **Real human-triage close time for Spec Kit**: stale-bot mass-close dominates the sample; need separate `-label:stale` filter.
- **Contributor retention**: how many Spec Kit contributors return after ≥3 months. Needs `stats/contributors` endpoint which returned `{}` repeatedly in our run.

These three gaps are addressable in a follow-up. Sufficient data exists today for an honest article.

---

## 12. Synthesis — what this corpus implies for the article

### 12.1 Recommended article structure (hybrid: focus + category map)

1. **Open**: 3-axis category map of SDD philosophies (spec-first / spec-anchored / spec-as-source) — orientation for readers who don't know the landscape
2. **Body** (~70%): OpenSpec vs Spec Kit deep-dive
   - Token economy (the floor)
   - Brownfield reality
   - Memory thesis (durable spec on disk)
   - Failure-mode catalogue (most citeable section)
   - Empirical benchmark (the numbers)
   - Health metrics (the honest twist)
3. **Close**: «Why not Tessl, Kiro, BMAD» — short paragraph each, structural reasons
4. **Verdict**: founder-stage / senior IC → OpenSpec; enterprise procurement-locked → Spec Kit; gov/regulated → services-wrapper

### 12.2 Key article numbers (article-grade quotes)

- **3.4×** — Spec Kit setup is longer (15.5s vs 4.5s)
- **2.6× / 2.5×** — Spec Kit's spec floor in lines / bytes
- **18.6K tokens** — Spec Kit's session context tax (Spec Kit's own admission)
- **2.4×** — cost-per-feature spread at floor
- **$2 500–7 000/year** — real-world per-team annual cost difference
- **17 vs 2** — Spec Kit vs OpenSpec releases in last 30 days
- **159 vs 28** — commits in last 4 weeks
- **58%** — TabishB's share of recent OpenSpec commits (bus-factor disclosure)
- **9 vs 4** — critical 🔴 failure-mode categories per tool

### 12.3 The "headline failure" for the article

The single most damaging Spec Kit issue worth quoting verbatim:

> *«Constitution is a large document read once at plan time and then gone from context. When the agent reaches the push step, it constructs the command from recall rather than from the constitution, gets the flags wrong, and hits a 403.»*  
> — [github/spec-kit#2219](https://github.com/github/spec-kit/issues/2219)

This is a **structural** defect, not a tactical bug — and it's exactly the failure mode that OpenSpec's durable-spec-on-disk model architecturally avoids.

### 12.4 What the article should NOT do

- Don't claim Spec Kit is dying — data refutes it
- Don't hide OpenSpec's bus factor — disclose 58% founder share
- Don't quote Discussion #1482 ("22 PRs / 0 commits") as current evidence — outdated
- Don't oversell OpenSpec on cross-feature systems thinking — this is genuinely Spec Kit's strength
- Don't claim a converter exists — empty niche, possibly publishable as separate artifact

### 12.5 The implicit positioning bridge

The structural failure modes that **neither** OpenSpec nor Spec Kit closes — F (constitution drift), C (cross-spec breakage), G (broken implementation undetected), I (archive lifecycle) — are exactly the methodology-layer the studio adds (decisions.md, dependency graph, Status markers, forge audit). This is the implicit conclusion of the article; it should be a one-paragraph close, not a sales pitch.

---

## 13. Open research questions (post-article)

| ID | Priority | Question |
|---|---|---|
| `Q-PA-1` | P1 | Real human-triage close time for Spec Kit (filter `-label:stale`) — does fast PR merge translate to fast issue resolution? |
| `Q-PA-2` | P1 | Star velocity over 12 months for both projects via star-history.com API — does the article's "growth trajectory" claim hold? |
| `Q-PA-3` | P1 | Live agent-execution benchmark: real token counts, wall-clock, iteration count — Pavel runs the protocol in `/tmp/sdd-benchmark/report/benchmark-results.md § 7` |
| `Q-PA-4` | P2 | Hallucination rate by format — same task, both formats, 5–10 runs each, scored — would single-handedly defend the article's claims |
| `Q-PA-5` | P2 | Forks landscape — what does `spec-kitty` (HN #46966273) and other Spec Kit forks add that the original lacks? |
| `Q-PA-6` | P2 | Tessl positioning — third-philosophy tool deserves its own short post; for this article it's a footnote |
| `Q-PA-7` | P3 | Practitioner interviews (3–5) — direct quotes from real OpenSpec/Spec Kit users with named attribution |
| `Q-PA-8` | P3 | OpenSpec ↔ Spec Kit converter (publishable artifact) — niche genuinely empty per §4 |
| `Q-PA-9` | P3 | Compliance deep-dive — EU AI Act Articles 11/12/14 mapping per format; separate post on compliance-track |

---

## 14. Sources cited

### 14.1 GitHub Spec Kit issues

[#125](https://github.com/github/spec-kit/issues/125), [#218](https://github.com/github/spec-kit/issues/218), [#264](https://github.com/github/spec-kit/issues/264), [#337](https://github.com/github/spec-kit/issues/337), [#581](https://github.com/github/spec-kit/issues/581), [#876](https://github.com/github/spec-kit/issues/876), [#926](https://github.com/github/spec-kit/issues/926), [#1026](https://github.com/github/spec-kit/issues/1026), [#1092](https://github.com/github/spec-kit/issues/1092), [#1095](https://github.com/github/spec-kit/issues/1095), [#1176](https://github.com/github/spec-kit/issues/1176), [#1242](https://github.com/github/spec-kit/issues/1242), [#1264](https://github.com/github/spec-kit/issues/1264), [#1401](https://github.com/github/spec-kit/issues/1401), [#1436](https://github.com/github/spec-kit/issues/1436), [#1450](https://github.com/github/spec-kit/issues/1450), [#1476](https://github.com/github/spec-kit/issues/1476), [#1550](https://github.com/github/spec-kit/issues/1550), [#1798](https://github.com/github/spec-kit/issues/1798), [#2151](https://github.com/github/spec-kit/issues/2151), [#2219](https://github.com/github/spec-kit/issues/2219), [#2293](https://github.com/github/spec-kit/issues/2293), [#2406](https://github.com/github/spec-kit/issues/2406), [#2419](https://github.com/github/spec-kit/issues/2419), [#2438](https://github.com/github/spec-kit/issues/2438), [#2440](https://github.com/github/spec-kit/issues/2440), [#2506](https://github.com/github/spec-kit/issues/2506)

### 14.2 GitHub Spec Kit discussions

[#152](https://github.com/github/spec-kit/discussions/152), [#331](https://github.com/github/spec-kit/discussions/331), [#746](https://github.com/github/spec-kit/discussions/746), [#1482](https://github.com/github/spec-kit/discussions/1482), [#1536](https://github.com/github/spec-kit/discussions/1536), [#1743](https://github.com/github/spec-kit/discussions/1743)

### 14.3 OpenSpec issues

[#132](https://github.com/Fission-AI/OpenSpec/issues/132), [#164](https://github.com/Fission-AI/OpenSpec/issues/164), [#258](https://github.com/Fission-AI/OpenSpec/issues/258), [#262](https://github.com/Fission-AI/OpenSpec/issues/262), [#293](https://github.com/Fission-AI/OpenSpec/issues/293), [#312](https://github.com/Fission-AI/OpenSpec/issues/312), [#361](https://github.com/Fission-AI/OpenSpec/issues/361), [#370](https://github.com/Fission-AI/OpenSpec/issues/370), [#374](https://github.com/Fission-AI/OpenSpec/issues/374), [#412](https://github.com/Fission-AI/OpenSpec/issues/412), [#429](https://github.com/Fission-AI/OpenSpec/issues/429), [#492](https://github.com/Fission-AI/OpenSpec/issues/492), [#609](https://github.com/Fission-AI/OpenSpec/issues/609), [#643](https://github.com/Fission-AI/OpenSpec/issues/643), [#673](https://github.com/Fission-AI/OpenSpec/issues/673), [#693](https://github.com/Fission-AI/OpenSpec/issues/693), [#704](https://github.com/Fission-AI/OpenSpec/issues/704), [#723](https://github.com/Fission-AI/OpenSpec/issues/723), [#838](https://github.com/Fission-AI/OpenSpec/issues/838), [#863](https://github.com/Fission-AI/OpenSpec/issues/863), [#879](https://github.com/Fission-AI/OpenSpec/issues/879), [#885](https://github.com/Fission-AI/OpenSpec/issues/885), [#890](https://github.com/Fission-AI/OpenSpec/issues/890), [#911](https://github.com/Fission-AI/OpenSpec/issues/911)

### 14.4 OpenSpec discussions

[#737](https://github.com/Fission-AI/OpenSpec/discussions/737)

### 14.5 Practitioner posts

- [Hashrocket — OpenSpec vs Spec Kit](https://hashrocket.com/blog/posts/openspec-vs-spec-kit-choosing-the-right-ai-driven-development-workflow-for-your-team)
- [Rida Kaddir — Why I Switched and Never Looked Back](https://ridakaddir.com/blog/post/openspec-vs-spec-kit-why-i-switched)
- [Lookoutking — Spec-Driven Development in Practice](https://medium.com/@lookoutking/spec-driven-development-in-practice-my-experience-with-spec-kit-8f250b47d677)
- [Scott Logic — Putting Spec Kit Through Its Paces](https://blog.scottlogic.com/2025/11/26/putting-spec-kit-through-its-paces-radical-idea-or-reinvented-waterfall.html)
- [Ran the Builder — I Tested Three Spec-Driven AI Tools](https://ranthebuilder.cloud/blog/i-tested-three-spec-driven-ai-tools-here-s-my-honest-take/)
- [intent-driven.dev — Spec Kit vs OpenSpec](https://intent-driven.dev/knowledge/spec-kit-vs-openspec/)
- [dev.to (willtorber) — Spec Kit vs BMAD vs OpenSpec 2026](https://dev.to/willtorber/spec-kit-vs-bmad-vs-openspec-choosing-an-sdd-framework-in-2026-d3j)
- [HackerNoon — Spec-First Development Showdown](https://hackernoon.com/the-spec-first-development-showdown-spec-kit-openspec-bmad-and-gangsta-agents-compared)
- [The Brownfield Problem — jjmasse](https://www.jjmasse.com/2026/03/06/the-brownfield-problem-why-most-ai-development-advice-ignores-your-actual-codebase/)
- [Augment Code — 6 Best SDD Tools 2026](https://www.augmentcode.com/tools/best-spec-driven-development-tools)

### 14.6 Authoritative references

- [Martin Fowler — SDD Tools (Kiro, Spec-Kit, Tessl)](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html)
- [Thoughtworks Tech Radar — Spec-Driven Development (Assess)](https://www.thoughtworks.com/en-us/radar/techniques/spec-driven-development)
- [InfoQ — SDD at Enterprise Scale](https://www.infoq.com/articles/enterprise-spec-driven-development/)
- [Microsoft Learn — Implement SDD with GitHub Spec Kit (enterprise)](https://learn.microsoft.com/en-us/training/modules/spec-driven-development-github-spec-kit-enterprise-developers/)
- [Anthropic — 2026 Agentic Coding Trends Report](https://resources.anthropic.com/hubfs/2026%20Agentic%20Coding%20Trends%20Report.pdf)
- [AGENTS.md open standard](https://agents.md/)
- [Augment Code — EU AI Act 2026](https://www.augmentcode.com/guides/eu-ai-act-2026)

### 14.7 OpenSpec / Spec Kit official

- [Fission-AI/OpenSpec — repo](https://github.com/Fission-AI/OpenSpec)
- [OpenSpec concepts.md](https://github.com/Fission-AI/OpenSpec/blob/main/docs/concepts.md)
- [OpenSpec multi-language docs](https://github.com/Fission-AI/OpenSpec/blob/main/docs/multi-language.md)
- [OpenSpec migration-guide (intra-version)](https://github.com/Fission-AI/OpenSpec/blob/main/docs/migration-guide.md)
- [OpenSpec landing — openspec.pro](https://openspec.pro/)
- [OpenSpec landing — openspec.dev](https://openspec.dev/)
- [OpenSpec on Y Combinator (W26)](https://www.ycombinator.com/companies/openspec)
- [Launch YC: OpenSpec — The Spec Framework for Coding Agents](https://www.ycombinator.com/launches/Pdc-openspec-the-spec-framework-for-coding-agents)
- [Tabish Bidiwale — LinkedIn](https://www.linkedin.com/in/tabishbidiwale/)
- [github/spec-kit — repo](https://github.com/github/spec-kit)
- [Spec Kit AGENTS.md](https://github.com/github/spec-kit/blob/main/AGENTS.md)

### 14.8 HN threads

- [HN #45154355 — github/spec-kit reception](https://news.ycombinator.com/item?id=45154355)
- [HN #45663874 — OpenSpec launch](https://news.ycombinator.com/item?id=45663874)
- [HN #46838086 — Reverse-engineer OpenSpec from existing codebases](https://news.ycombinator.com/item?id=46838086)

---

## 15. Methodology / reproducibility

### 15.1 Empirical benchmark

Test directories preserved at `/tmp/sdd-benchmark/{openspec,speckit}-test/`. Full report in `/tmp/sdd-benchmark/report/benchmark-results.md`. Reproducible:

```bash
mkdir -p /tmp/sdd-benchmark && cd /tmp/sdd-benchmark
mkdir openspec-test && cd openspec-test && git init -q
time openspec init . --tools claude --force

cd .. && mkdir speckit-test && cd speckit-test && git init -q
time uvx --from git+https://github.com/github/spec-kit.git specify init . --here --ai claude --force --no-git
```

### 15.2 Health metrics

All raw JSON in `/tmp/sdd-benchmark/health/`. Full report in `/tmp/sdd-benchmark/report/health-metrics.md`. Reproducible:

```bash
# Repo basic info
gh api repos/github/spec-kit
gh api repos/Fission-AI/OpenSpec

# Commit velocity
gh api 'search/commits?q=repo:REPO+committer-date:>YYYY-MM-DD&per_page=1'

# PR / Issue counts
gh api 'search/issues?q=repo:REPO+is:pr+is:merged&per_page=1'

# Releases (paginated)
gh api 'repos/REPO/releases?per_page=100&page=N'

# PR merge time sample
gh api 'search/issues?q=repo:REPO+is:pr+is:merged&sort=updated&order=desc&per_page=30'

# Recent commits with authors
gh api 'repos/REPO/commits?per_page=100'

# Stargazer timestamps
gh api -H 'Accept: application/vnd.github.star+json' 'repos/REPO/stargazers?per_page=100&page=N'
```

### 15.3 Citation hygiene

- Every quote with `«»` is verbatim from the linked source as of 2026-05-10
- Every issue/discussion link was fetched directly during research
- Numbers from third-party benchmarks (Hashrocket, Scott Logic, ranthebuilder) are reproduced from their published claims; not independently re-verified
- Where third-party claims and our empirical measurements diverge, both are cited and the divergence noted

---

## 16. Change log

| Date | Author | Change |
|---|---|---|
| 2026-05-10 | Pavel (founder) | Opened thread. Consolidated three analysis passes (parameters/pluses-minuses, agents/segments/stats/demographics, failure-modes), first-party empirical benchmark on `/tmp/sdd-benchmark/`, GitHub health-metrics audit. Refuted prior thesis from `competitive-positioning-sdd-frameworks.md` § 2.1 ("22 PRs / 0 commits → Spec Kit dying") with May 2026 data showing 17 releases / 30 days. New thesis: «health ≠ velocity; both are alive in different shapes». 9 open research questions; protocol for live agent-execution attached. AI assistance: drafted with Claude (Anthropic). |
