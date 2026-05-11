# OpenSpec organization patterns in the community (2026 snapshot)

> **Status:** complete (snapshot, 2026-05-11)
> **Owner:** Pavel Rapoport (founder).
> **Authors:** Pavel Rapoport <pavel@rapoport.studio> (founder, primary); Claude Opus 4.7 (Anthropic) — web research, source synthesis, ~24 search queries across GitHub, blog posts, HN, conferences.
> **Method:** Single-pass web synthesis — Fission-AI/OpenSpec repo + docs, GitHub Spec Kit docs, AWS Kiro docs, BMad Method posts, Anthropic Skills documentation, ~10 community blog posts (redreamality, rushis, antongubarenko, intent-driven.dev), 3 HN threads (47994827, 47418296), Hackernoon comparison article. No first-party empirical measurement.
> **Reusable for:** `bridge-vocabulary-and-restructure` proposal/design context; future research on SDD tooling. Authorship: open.
> **Opened:** 2026-05-11
> **Feeds into:** Epic 1 — `bridge-vocabulary-and-restructure` (current/ layout decisions, brands/ as mirror replacement, entities/ split rationale).
> **Companion to:** [`openspec-vs-speckit-public.md`](./openspec-vs-speckit-public.md), [`competitive-positioning-sdd-frameworks.md`](./competitive-positioning-sdd-frameworks.md).

---

## TL;DR

The OpenSpec community in May 2026 has consolidated around four practical conventions: (1) `specs/` (or `current/`) is the source of truth, (2) `changes/<slug>/` holds in-flight proposals as atomic deltas, (3) `archive/YYYY-MM-DD-<slug>/` is frozen post-merge, and (4) capabilities are organized by **observable behaviour**, not by entity, layer, or technology. The Rapoport Studio variant (prose-style proposals, no `specs/` deltas) is an acknowledged local extension, documented in [`openspec/AGENTS.md`](../../AGENTS.md). Community canon does not solve "readable for non-engineers" or cross-capability linking — both are open spaces where local extensions are legitimate.

---

## Canonical layout — what every SDD tool agrees on

Fission-AI OpenSpec ([docs/getting-started.md](https://github.com/Fission-AI/OpenSpec/blob/main/docs/getting-started.md)):

```
openspec/
├── specs/                 # source of truth, by capability
│   └── <capability>/spec.md
├── changes/<change-id>/
│   ├── proposal.md
│   ├── design.md
│   ├── tasks.md
│   └── specs/<capability>/spec.md   # delta spec
├── schemas/               # optional workflow templates
└── config.yaml            # v1.0 replaced project.md
```

- **kebab-case + verb-led IDs for changes** ([rushis.com cheatsheet](https://www.rushis.com/openspec-cheatsheet/)): "kebab-case and verb-led IDs for changes".
- **Date-prefix is added only at archive**: `archive/YYYY-MM-DD-<name>/` is the output of `openspec archive`, not a manual rename. This matches Rapoport Studio's memory note that "OpenSpec change folders are bare slugs while active".
- **One `spec.md` per capability**; deltas describe ADDED/MODIFIED/REMOVED requirements ([thedocs.io spec-format](https://thedocs.io/openspec/concepts/spec-format/)). Format: `Requirement → Scenario (Given/When/Then)`.

## How the community divides "capabilities"

[redreamality OpenSpec guide](https://redreamality.com/garden/notes/openspec-guide/): *"organized by functional modules (capabilities) … e.g., `auth-login/`, `payment/`"*. [Hackernoon SDD showdown](https://hackernoon.com/the-spec-first-development-showdown-spec-kit-openspec-bmad-and-gangsta-agents-compared) confirms: *"specs organized by capability (auth-login/, checkout-cart/)"*.

Three observable splits in practice:
- **DDD-style bounded contexts** — capability ≈ subdomain.
- **User-flow style** — `checkout-cart`, `2fa-enrollment`.
- **Antipattern (avoid)** — capability named after a layer (`ui`, `api`, `db`). This is named explicitly in the [intent-driven.dev best-practices](https://intent-driven.dev/knowledge/best-practices/).

Rapoport Studio's `openspec/current/` is between the first two — `forge`, `muse`, `home`, `studio` are user-facing or functional, not layer-named. This is canonical.

## Where canon falls short

Three gaps the SDD community has not closed (and where Rapoport Studio's local extensions are defensible):

1. **Entity description.** Canon prefers behavioural requirements with Given/When/Then; it does not standardize how to describe data entities. Studio's `entities.md` (now to become `entities/`) fills a real gap.
2. **Non-engineer readability.** OpenSpec's positioning ([YC launch page](https://www.ycombinator.com/launches/Pdc-openspec-the-spec-framework-for-coding-agents)) is "engineering teams working on large or multi-repo codebases" — PMs and founders are out of scope. Studio's planned `narrative.md` plus `methodology/spec-driven-transition.md` are legitimate extensions.
3. **Cross-capability linking.** No native graph or include directive. Community workaround: numbered spec tags + Mermaid C4 diagrams in `design.md` + prose cross-links ([HN comment 47994827](https://news.ycombinator.com/item?id=47994827)).

## Antipatterns the community names

Consolidated from [intent-driven.dev best-practices](https://intent-driven.dev/knowledge/best-practices/), [Marmelab Waterfall critique](https://marmelab.com/blog/2025/11/12/spec-driven-development-waterfall-strikes-back.html), and [antongubarenko Substack](https://antongubarenko.substack.com/p/spec-driven-development-with-opensec):

1. **Specification Theater** — specs nobody reads.
2. **Premature Comprehensiveness** — specifying before understanding.
3. **Spec-Implementation Drift** — *"when you do the sync process, it just keeps drifting and drifting until you have duplication and contradictions"* (HN 47994827).
4. **AI-Generated Bloat** — accepting verbose LLM output without editing.
5. **Direct edits to `specs/` without a change** — breaks atomicity.
6. **Empty `config.yaml`** — *"AI hallucinations / references nonexistent libraries"*.
7. **Capability as a technical layer** (api / ui / db).
8. **Full-spec replacement** instead of delta — *"adds noise and makes reviews harder"*.

## Seven consolidated principles

1. Specs = source of truth, changes = atomic deltas.
2. One file per capability; capability = functional module / observable behaviour.
3. kebab-case + verb-led for changes; date-prefix only on archive.
4. Given/When/Then mandatory; SHALL/MUST as normative language.
5. Brownfield-first, delta-only.
6. Each artifact has one reader: `proposal.md` → human, `design.md` → architect, `tasks.md` → executor, `specs/*.md` → future system. Do not mix.
7. Human reviewability as quality test. If a PM cannot read the `proposal.md` without a translator, the spec failed even if it validates.

## Implications for Rapoport Studio

- **Keep `current/` (not rename to `capabilities/`)** — every SDD tool calls source-of-truth either `specs/` or `current/`; both are canonical. Renaming costs 120+ archive cross-refs without semantic gain.
- **Move `mirror/` → `brands/`** — Mirror is a Rapoport-local term with no community recognition; Brand is universal across marketing and DDD circles.
- **Split `entities.md` into `entities/` folder** — supports Skills-pattern progressive disclosure (~500-line files) and parallels the documented size convention.
- **Acknowledge `narrative.md` as extension** — community canon does not standardize it; this is studio-specific to bridge non-engineer readers (PM, founder, client). Document this in `methodology/spec-driven-transition.md` so future reviewers understand the divergence.

---

## Sources

- [Fission-AI/OpenSpec GitHub](https://github.com/Fission-AI/OpenSpec)
- [OpenSpec concepts.md](https://github.com/Fission-AI/OpenSpec/blob/main/docs/concepts.md)
- [OpenSpec getting-started.md](https://github.com/Fission-AI/OpenSpec/blob/main/docs/getting-started.md)
- [OpenSpec CLI docs](https://github.com/Fission-AI/OpenSpec/blob/main/docs/cli.md)
- [thedocs.io OpenSpec spec-format](https://thedocs.io/openspec/concepts/spec-format/)
- [OpenSpec YC Launch page](https://www.ycombinator.com/launches/Pdc-openspec-the-spec-framework-for-coding-agents)
- [redreamality OpenSpec Deep Dive](https://redreamality.com/garden/notes/openspec-guide/)
- [Rushi OpenSpec cheatsheet](https://www.rushis.com/openspec-cheatsheet/)
- [intent-driven.dev OpenSpec page](https://intent-driven.dev/knowledge/openspec/)
- [intent-driven.dev best-practices](https://intent-driven.dev/knowledge/best-practices/)
- [intent-driven.dev OpenSpec + OpenCode integration](https://intent-driven.dev/blog/2026/05/10/spec-driven-development-openspec-opencode/)
- [Hackernoon SDD Showdown (Spec Kit / OpenSpec / BMad / Gangsta Agents)](https://hackernoon.com/the-spec-first-development-showdown-spec-kit-openspec-bmad-and-gangsta-agents-compared)
- [Medium: BMAD vs Spec Kit vs OpenSpec](https://medium.com/@ap3617180/steering-the-agentic-future-a-technical-deep-dive-into-bmad-spec-kit-and-openspec-in-the-sdd-4f425f1f8d2b)
- [antongubarenko Substack on OpenSpec](https://antongubarenko.substack.com/p/spec-driven-development-with-opensec)
- [HN thread 47994827](https://news.ycombinator.com/item?id=47994827)
- [HN thread 47418296](https://news.ycombinator.com/item?id=47418296)
