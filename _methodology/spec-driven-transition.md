# Spec-Driven Transition

> The primary onboarding artifact for anyone joining Rapoport Studio mid-flight.
> Read this once; keep it as a reference.

---

## 1. What this is and why

Rapoport Studio shifted from ad-hoc product documents to a spec-first operating model. This file explains what that means in plain language and why the shift happened. It is not a process manual — it is a translation layer: if you know how a design agency, a product company, or an AI lab operates, this document tells you which words map where and what stays familiar. The goal is that you leave this file oriented, not overwhelmed.

The short version: **every change starts with a file in `openspec/`**. That file is the single source of truth for what we're building, why, and how. Code, design, and copy follow; they do not lead.

Why did the shift happen? The studio runs on AI execution. When an AI agent picks up a task, it can only follow what is written down — it cannot ask a colleague, scan a Notion page, or attend a stand-up. The spec-driven model is not bureaucratic overhead; it is the interface between human intention and machine execution. Writing a clear proposal is not slower than a verbal brief — it is faster, because the AI does not ask clarifying questions that were never answered in writing.

Humans benefit too. When every change has a written proposal, design, and tasks file, any collaborator can join mid-flight without a dedicated handoff meeting. The document is the handoff.

---

## 2. Where we came from / where we are going

The table below maps common artifacts from product / design / engineering practice to their equivalents in the studio's spec-driven model.

| Before (typical practice) | After (studio model) | Notes |
|---|---|---|
| Requirements in a Notion PRD | `openspec/changes/<slug>/proposal.md` | Same intent; explicit format |
| Architecture decision record or design doc | `openspec/changes/<slug>/design.md` | Same intent; lives in the repo |
| Task list in Notion / Jira / Linear (ad-hoc) | `openspec/changes/<slug>/tasks.md` + Linear issue | Phased checklist with verification steps |
| Brand guidelines in a PDF or Notion page | `openspec/brands/<slug>/identity.md` | Markdown; versioned; linkable |
| Project tracking in Notion database | Linear (AI- prefix, synced with Forge) | Linear is the execution layer |
| Entity definitions scattered in product docs | `openspec/entities/<tier>.md` | Canonical; single source of truth |
| Onboarding = "read these 40 Notion pages" | `openspec/_root/README.md` + this file | Scoped and navigable |
| "Who maintains this?" — unclear | Every file was created by a change with an owner | Traceable via `openspec/archive/` |
| Figma link shared in Slack, no context | `openspec/current/design-system.md` links the canonical Figma file | The spec is the stable reference |
| Mirror / perception layer for brands | `openspec/brands/<slug>/perception.md` | Renamed from `mirror/`; same content |

**What stays the same:**

- Design work happens in Figma. The spec references the Figma file; it does not replace it.
- Client conversations happen in Telegram and WhatsApp.
- Code reviews happen in GitHub PRs.
- Strategic decisions are made by Pavel. The operating model amplifies execution; it does not replace judgment.
- The three-part change structure (proposal → design → tasks) maps directly to how thoughtful product work already happens — it just makes the implicit explicit.

---

## 3. Bridge glossary (quick reference)

The full glossary lives in [`openspec/_root/glossary.md`](../_root/glossary.md). The rows below are the most-used bridging terms for anyone arriving from a product, design, or engineering background.

| Studio canonical | Product / Design / Eng equivalent | Lives at |
|---|---|---|
| **Proposal** | PRD, brief, initiative spec, RFC | `openspec/changes/<slug>/proposal.md` |
| **Design** (OpenSpec doc) | Architecture decision record, detailed spec, solution design | `openspec/changes/<slug>/design.md` |
| **Tasks** | Sprint, execution plan, implementation ticket list | `openspec/changes/<slug>/tasks.md` |
| **Capability** | Feature, service, domain area, module | `openspec/current/<capability>.md` |
| **Change** | Epic, initiative, RFC, change request | `openspec/changes/<slug>/` |
| **Archive** | Done / closed / shipped, historical record | `openspec/archive/YYYY-MM-DD-<slug>/` |
| **Brand** | Product brand, brand identity, brand guidelines | `openspec/brands/<slug>/identity.md` |
| **Workspace** | Internal studio app, back-office, dashboard | `apps/studio` at `app.rapoport.studio` |
| **Forge** | CI robot, automation agent, coding assistant CLI | `packages/forge/` |
| **Curation** | Operating model, staffing approach, fractional / gig model | `openspec/_methodology/studio-charter.md` |
| **Studio** (as company) | Agency, SRL, the organization | Rapoport Studio SRL |
| **Narrative** | Story, case study, brand story, project recap | `openspec/brands/<slug>/` (planned per brand) |
| **Surface** | Screen, page, view, experience, touchpoint | Defined strictly in `_root/glossary.md § Appendix 3` |
| **Primitive** | Component (UI), atom, base pattern | Code-level; spec-prose uses "Primitive" not "Component" |
| **Identity** (brand) | Brand identity, brand guidelines, brand brief | `openspec/brands/<slug>/identity.md` |
| **Perception** (brand) | Perception layer, mirror, how the brand appears to outsiders | `openspec/brands/<slug>/perception.md` |

> For collision terms — words that mean different things in different schools (e.g. "Persona", "Capability", "Domain") — see the collision matrix in [`_root/glossary.md § Appendix 1`](../_root/glossary.md). For words that are banned in spec-prose with their replacements, see `_root/glossary.md § Appendix 2`.

---

## 4. Top renames and why

These are the vocabulary changes that affect the most files and cause the most confusion on first encounter. Each entry explains the problem the rename solves, not just what changed.

### Studio → Workspace (for the app)

**Problem:** The word "Studio" has two legitimate meanings: the company (Rapoport Studio SRL) and the internal workspace app (`app.rapoport.studio`). When both appear in the same spec sentence, readers don't know which is being referenced — and the ambiguity compounds over time.

**Fix:** When referring to the app, say **Workspace**. When referring to the company, say **Studio**.

**Heuristic:** If you could substitute "our company", use Studio. If you could substitute "the internal tool we log into at `app.rapoport.studio`", use Workspace. In spec-prose, the full phrase "Workspace app" is the safest version. In code, the package is `apps/studio` and the app is at `app.rapoport.studio` — the code identifiers don't change; only the prose vocabulary does.

### Story → Narrative

**Problem:** "Story" is overloaded. Agile user stories, brand stories, short-form editorial pieces, and narrative case studies all compete for the same word. In context, the reader frequently cannot tell which meaning is intended.

**Fix:** For the genre we care about — a curated, authorial account of a project, brand, or design decision — the canonical term is **Narrative** (file: `narrative.md`). User stories as a technique (in tasks files or Linear issues) are fine; they just do not name a genre of spec file.

### `mirror/` → `brands/`

**Problem:** The folder `openspec/mirror/` was named for the "Mirror" concept — a perception layer reflecting each brand outward. "Mirror" as a studio concept name was borrowed from an internal metaphor that made sense in the founding team but was opaque to collaborators and AI agents.

**Fix:** The folder is now `openspec/brands/`; the concept is **Brand perception spec**. The word "Mirror" as a concept name is retired from spec-prose. Archive entries referencing the old `mirror/` path are historical, not broken; they are intentionally not updated (per the policy that archive entries are frozen).

### `_methodology/` path stays (for now)

**Conditional:** The rename from `openspec/_methodology/` to `openspec/methodology/` (removing the leading underscore) is deferred pending bundle size verification in Phase 9 of the parent change. The underscore prefix was a "looks system-y" convention with no enforced meaning. When the rename ships, all `_methodology/` references in this file will be updated. Until then, use `_methodology/` as the canonical path.

### Banned words in spec-prose

Four words are forbidden as bare terms in `openspec/current/**/*.md` and `openspec/brands/**/*.md`. The CI lint rule (`tools/lint-spec-prose.mjs`) catches violations.

| Banned (bare) | Problem | Replace with |
|---|---|---|
| `Context` | Means Brand context, user context, conversation context, JavaScript execution context — pick the precise word | Brand, Engagement, Bounded scope, Capability scope |
| `Module` | In spec-prose this usually means Capability; in code it means JS module; the collision causes interpretation errors | Package, Capability (depending on meaning) |
| `Component` | Valid only for React components in code; in spec-prose it is unacceptably vague | Use "Primitive" for design-system atoms; use "Capability" for system concerns |
| `Domain` | Means brand domain, database domain, Tier-2 entity class, and "domain expert" all at once | Brand (for brand meaning), named entity tier for data meaning, "field" or "area of expertise" for the human meaning |

**Exceptions (always allowed):** The literal phrases "React Context", "React Component", "JS module", "TypeScript module" are allowlisted because they refer unambiguously to code artifacts. Occurrences inside fenced code blocks and inline code spans are also excluded from the check.

---

## 5. Pavel's twist for Product Managers

If you've worked in product before — at a startup, an agency, or in-house — you already know this loop:

1. You spot a **Problem**.
2. You write a **Solution** (or a brief, or a PRD, or a spec).
3. You hand it off to engineering or design, who figure out the execution implicitly.

The studio uses the same loop, but makes all three steps explicit and adds a fourth:

| What you knew | What we call it | Where it lives | What's new |
|---|---|---|---|
| Problem statement | **Proposal** (`proposal.md`) | `openspec/changes/<slug>/proposal.md` | Nothing new in substance — same intent, named plainly, lives in the repo |
| Solution / how to address the problem | **Design** (`design.md`) | `openspec/changes/<slug>/design.md` | Explicitly separated from the "why"; can be authored by PM, designer, or engineer depending on the change; contains interfaces, data shapes, boundary contracts |
| *(the implicit execution plan that lived in your head or a Jira backlog)* | **Tasks** (`tasks.md`) | `openspec/changes/<slug>/tasks.md` | **This is the genuinely new part.** The phased, reviewable execution checklist — with verification steps and acceptance criteria — that was previously implicit is now a first-class artifact alongside proposal and design |

**Why the mapping matters for PMs specifically:**

The proposal → design → tasks split resolves a common product handoff failure mode: the PM writes a clear problem and solution statement, but the executable plan lives only in the PM's mental model and gets reconstructed (with errors) by engineering in sprint planning. In the studio model, the PM either authors or reviews `tasks.md` before the change moves to execution — which forces the implicit plan to be made explicit while the context is still fresh.

The tasks file is not a ticket dump. It is a phased checklist that can be read sequentially from top to bottom to understand the shape of the work. Each phase has a name; each checkbox is actionable; each phase ends with a verification step. When an AI agent executes the tasks, it follows this file literally.

**The expanded PM role:** You will write slightly more documentation per change — specifically, you now own or review the tasks file as a first-class artifact. In exchange, status-meeting overhead drops significantly, because the artifacts replace the verbal syncs. The documents are the communication.

**For non-PMs:** The same logic applies. An engineer writing a change does all three files. A designer writing a brand change does all three. The distinction is not which role authors each file — it is that all three files must exist before execution begins. If you are mid-execution and the design or tasks file is missing, stop and write it. The cost of backfilling is much lower than the cost of executing without one.

---

## 6. Onboarding by role

These onboarding cheat sheets will be created lazily — each file appears when the corresponding role joins or when sufficient common questions accumulate. They expand the quick-reference below into a full orientation specific to your context, tooling, and first-week priorities.

| Role | Onboarding doc | Status |
|---|---|---|
| Product Manager | `openspec/_methodology/onboarding/for-product-manager.md` | Coming in Epic 6 |
| Designer | `openspec/_methodology/onboarding/for-designer.md` | Coming in Epic 6 |
| Engineer | `openspec/_methodology/onboarding/for-engineer.md` | Coming in Epic 6 |
| AI agent (Forge, Muse, etc.) | `openspec/_methodology/onboarding/for-ai-agent.md` | Coming in Epic 6 |
| Specialist (network contractor) | `openspec/_methodology/onboarding/for-specialist.md` | Coming in Epic 6 |
| Founder / investor | `openspec/_methodology/onboarding/for-founder.md` | Coming in Epic 6 |
| Client | `openspec/_methodology/onboarding/for-client.md` | Coming in Epic 6 |

Until those files exist, the fastest orientation per role is:

**Product Manager:**
Read this file → [`openspec/_root/glossary.md`](../_root/glossary.md) → skim `openspec/current/platform.md § Capability map`. Then look at one recently archived change (run `ls openspec/archive/ | sort -r | head -1` to find the most recent) to see the three-file pattern in practice.

**Designer:**
Read this file → `openspec/brands/<brand>/identity.md` for the brand you're working on → `openspec/current/design-system.md`. Figma files are linked from identity.md § Visual identity and from the design-system capability spec.

**Engineer:**
Read `CLAUDE.md` (repo root) → `openspec/AGENTS.md` → your assigned Linear issue and the linked change folder. The stack quick-reference and hard rules in CLAUDE.md are the fastest path to being unblocked on day one.

**AI agent (Forge, Muse, etc.):**
Read `CLAUDE.md` → `openspec/AGENTS.md` → `openspec/current/platform.md` → your assigned Linear issue → the linked `openspec/changes/<slug>/` folder. Execute `tasks.md` top-to-bottom. Surface conflicts between spec and task in the PR description; do not resolve them silently.

**Specialist (network contractor):**
Read this file → `openspec/_methodology/specialist-network.md` → `openspec/_methodology/rapoport-studio-engagement.md` → your assigned project folder under `openspec/_methodology/projects/`.

**Client:**
Not expected to read this file. Clients interact via Telegram / WhatsApp and receive curated deliverables via the Workspace. If a client is technically oriented and curious about the spec system, point them to `openspec/_root/README.md` first, then this file.

---

## 7. What does not change

The spec-driven model is a layer on top of existing practices — it does not replace them. These things are explicitly out of scope for the vocabulary shift.

### `openspec/current/` is read-only via change proposals

Capability specs in `openspec/current/` are modified only by opening a change in `openspec/changes/<slug>/` and going through the proposal → design → tasks → PR → archive cycle. Direct edits to `current/` are not allowed (CLAUDE.md § Hard rule 7).

Exception: typo fixes and broken-link fixes. These can go in any PR but must be called out separately in the PR description. They do not require a full change proposal.

### `openspec/archive/` is frozen

Once a change folder moves from `changes/` to `archive/` (at merge + archive time), it is never edited again. Archive entries describe the historical intent of the change, not the current state of the codebase. If an archived entry references a path that has since been renamed (`mirror/<slug>/perception.md` → `brands/<slug>/perception.md`), the archive entry is correct as a historical reference and is intentionally not updated.

### The three-file proposal contract is unchanged

Every change under `openspec/changes/<slug>/` has exactly three files: `proposal.md`, `design.md`, `tasks.md`. There is no `specs/` subdirectory, no formal delta files, no YAML schemas. The OpenSpec CLI's strict-deltas validator is expected to fail on this convention — that failure is intentional and documented. See `openspec/AGENTS.md` for why.

### `lifecycle.yaml` persona_hint values are unchanged

The word "domain" inside `lifecycle.yaml` refers to a conversation persona (architect / domain / tech / business / ops). It is distinct from the "Domain entities" tier, which is retired in the vocabulary shift. A rename from `persona_hint` to `role_hint` is a separate code-touching change; it does not ship as part of this vocabulary transition.

### Tools, platforms, and runtime

Figma, Telegram, WhatsApp, GitHub, Linear — unchanged. The tools we use for design, client communication, code review, and project tracking continue as-is. The spec-driven model augments the workflow between tools; it does not replace them.

The tech stack (Next.js, Supabase, Cloudflare Workers, Anthropic), the package boundaries, the auth model, the DB schema — none of this changes as part of the vocabulary shift. Those concerns evolve through their own capability specs and changes.

---

## 8. History of the transition

*Append-only. Add an entry when a significant vocabulary or structural change ships.*

| Date | Change | PR | Summary |
|---|---|---|---|
| 2026-05-12 | `bridge-vocabulary-and-restructure` Phases 0+1 | #811 | Created `openspec/_root/README.md` (orientation map) and `openspec/_root/glossary.md` (three-column bridge glossary with collision matrix, banned-words appendix, and defined-strictly appendix). |
| 2026-05-12 | `bridge-vocabulary-and-restructure` Phase 2 | #812 (pending) | Created this file (`spec-driven-transition.md`). Added cross-link from `studio-charter.md § 12 Glossary` to `_root/glossary.md`. Documented the full vocabulary shift, top renames, and PM onboarding hook. |
