# SDD terminology scorecard — Rapoport Studio vocabulary vs industry canon (2026)

> **Status:** complete (snapshot, 2026-05-11)
> **Owner:** Pavel Rapoport (founder).
> **Authors:** Pavel Rapoport <pavel@rapoport.studio> (founder, primary); Claude Opus 4.7 (Anthropic) — collision-check research across SDD tools (Spec Kit, OpenSpec, Kiro, BMad, Skills, Cursor Rules), industry glossaries (DDD, TOGAF, NN/g, Material), scorecard application, ~31 search queries.
> **Method:** Single-pass — collision check against SDD inventory + classical product/design glossaries; 7-criteria scoring rubric applied to 30 terms; competitor analysis of naming conflicts (Forge, Muse, Mirror, Canvas).
> **Reusable for:** `bridge-vocabulary-and-restructure` (Epic 1) — glossary content, rename decisions, defined-strictly terms, retire list. Authorship: open.
> **Opened:** 2026-05-11
> **Feeds into:** Epic 1 — direct input to `_root/glossary.md` and rename phases in `tasks.md`.

---

## TL;DR

Of ~30 Rapoport Studio terms scored on 7 criteria (uniqueness, recognizability, precision, brevity, searchability, memorability, translatability), the **8 strongest** sit at 8.0+/10 (Spec, Workspace, Hypothesis, User persona, Brand, Perception, Problem statement, Acceptance criteria). The **4 weakest** at ≤5.0/10 are `Context`, `Module`, `Component`, `Domain` — all generic, multi-meaning, and grep-noisy. **Top renames recommended:** Studio (apps/studio) → Workspace; Persona (lifecycle) → Role; Story → Narrative; Tier → Class; Entity (Tier-2 Domain) → keep as Entity, rename Tier-2 group only.

---

## SDD canon — what every tool agrees on

[GitHub Spec Kit spec-driven.md](https://github.com/github/spec-kit/blob/main/spec-driven.md): **Constitution / Specify / Plan / Tasks / Implement**, plus Story, Principle, Gate, Artifact.
[Fission-AI OpenSpec concepts.md](https://github.com/Fission-AI/OpenSpec/blob/main/docs/concepts.md): Spec / Change / Delta Spec / Requirement / Scenario / Artifact / Archive.
[AWS Kiro hooks](https://kiro.dev/docs/hooks/): Spec / Agent Hooks / Agents / Steering Files.
[BMad Method](https://buildmode.dev/blog/mastering-bmad-method-2025/): PRD sharded into Epic + Story files (`{epicNum}.{storyNum}.story.md`).
[Anthropic Skills frontmatter](https://platform.claude.com/docs/en/agent-skills/best-practices): name (kebab-case, ≤64 chars), description (≤1024 chars), gerund-form recommended.

**Consensus terms:** Spec, Plan, Tasks, Change. **Local terms (do not export):** Constitution, Gate (Spec Kit), Story (collides badly — agile, BMad, Spec Kit, social media), Hook/Steering (Kiro).

## Industry-canon overlap

| Domain | Their canon | Studio overlap |
|---|---|---|
| DDD ([Fowler ubiquitous-language](https://martinfowler.com/bliki/UbiquitousLanguage.html), [dddcommunity.org/resources](https://www.dddcommunity.org/resources/ddd_terms/)) | Bounded Context, Ubiquitous Language, Entity (with identity), Aggregate, Value Object | Studio **Entity** is weaker than DDD Entity; studio's "Domain" (Tier-2) conflicts with DDD Domain |
| TOGAF Business Architecture ([opengroup TOGAF](https://pubs.opengroup.org/togaf-standard/business-architecture/business-capabilities.html)) | Capability = "particular ability that a business may possess or exchange" | Studio **Capability** aligns well — TOGAF favourable |
| Material 3 / Atlassian / Carbon ([m3.material.io/foundations/glossary](https://m3.material.io/foundations/glossary)) | Surface (tone-based depth layer), Token, Component, Elevation | Studio **Surface** conflicts — Material surface = depth, studio surface = UI place |
| Radix / shadcn ([radix-ui.com/primitives](https://www.radix-ui.com/primitives)) | Primitive = headless behaviour + accessibility | Studio **Primitive** mostly aligns; sub-meaning slightly differs |
| Marty Cagan SVPG ([svpg.com/product-discovery](https://www.svpg.com/product-discovery/)) | Problem statement, Hypothesis, Four risks (Value/Usability/Feasibility/Viability), Success metric | Studio bridge terms align fully |
| NN/g ([nngroup.com/articles/persona](https://www.nngroup.com/articles/persona/)) | Persona = user representation | Studio **Persona** (lifecycle) conflicts seriously |
| Brand strategy ([ramotion brand glossary](https://www.ramotion.com/blog/glossary-of-branding-terms/), [literalhumans brand-image vs identity](https://literalhumans.com/brand-image-vs-brand-identity-from-perception-to-recognition/)) | Brand identity = business self-presentation; Brand perception = audience view | Studio **Brand / Identity / Perception** triad maps cleanly |
| Slack/Notion/Linear multi-tenant ([workos.com](https://workos.com/blog/multi-tenant-permissions-slack-notion-linear)) | Workspace = unit of tenancy | Studio **Workspace** (replacing apps/studio meaning) maps cleanly |

## Collision matrix (HIGH severity only)

| Term | Top collisions | Severity |
|---|---|---|
| Studio (dual meaning: company + workspace) | Android Studio, Visual Studio, ChatGPT Studio, Anthropic Console Studio | HIGH — both internal dual-meaning and industry-wide |
| Muse | Microsoft Muse, Lightricks LTX-Video Muse, HuggingFace MUSE, Muse Group | HIGH — overloaded |
| Mirror | Mirror.xyz, Mirror Protocol (Terra), Mirror World, Apple AirPlay Mirror | MEDIUM-HIGH |
| Canvas | Figma Canvas, Notion Canvas, Slack Canvas, Apple Canvas, Microsoft Canvas Apps, Instructure Canvas LMS | HIGH — catastrophically overloaded in 2026 UI |
| Surface | Material 3 surface, Meta FDS surface, Microsoft Surface, Vercel surface area | HIGH — two distinct design-system meanings |
| Persona | NN/g UX persona, marketing buyer persona, brand persona — all distinct from studio lifecycle persona | HIGH |
| Domain (Tier-2 meaning) | DDD bounded context (different meaning), DNS domain | HIGH — internal dual meaning + DDD conflict |
| Story | Agile user story, BMad story file, Spec Kit story, Instagram/Snapchat Story | HIGH — too many concurrent meanings |
| Context | React context, LLM context, DDD bounded context | HIGH — every grep returns hundreds of false positives |
| Module / Component | Any JS module, any React Component | HIGH — generic mush |

## Scorecard — 7 criteria × 30 terms

Each criterion 0–10. Total = avg/7 rounded to 0.5.

| Term | Uniq | Recog | Prec | Brev | Search | Mem | Trans | Total | Verdict |
|---|---|---|---|---|---|---|---|---|---|
| Spec | 8 | 9 | 8 | 10 | 8 | 8 | 9 | **8.5** | KEEP |
| Workspace | 9 | 9 | 8 | 8 | 9 | 8 | 9 | **8.5** | ADD |
| Hypothesis | 9 | 10 | 8 | 6 | 9 | 7 | 9 | **8.5** | ADD |
| User persona | 8 | 10 | 9 | 6 | 9 | 9 | 9 | **8.5** | ADD |
| Brand | 7 | 9 | 8 | 10 | 7 | 8 | 9 | **8.0** | ADD |
| Perception | 9 | 7 | 8 | 8 | 9 | 7 | 8 | **8.0** | ADD |
| Problem statement | 9 | 10 | 9 | 4 | 9 | 7 | 8 | **8.0** | ADD |
| Success metric | 9 | 10 | 9 | 4 | 9 | 7 | 8 | **8.0** | ADD |
| User flow | 9 | 10 | 9 | 5 | 9 | 7 | 8 | **8.0** | ADD |
| Acceptance criteria | 9 | 10 | 9 | 3 | 9 | 8 | 8 | **8.0** | ADD |
| Identity | 6 | 8 | 6 | 9 | 6 | 8 | 9 | **7.5** | ADD + DEFINE-STRICTLY |
| Sub-product | 9 | 8 | 7 | 5 | 9 | 6 | 8 | **7.5** | KEEP |
| Forge | 5 | 7 | 8 | 10 | 6 | 9 | 8 | **7.5** | KEEP (aware of Atlassian/SD-WebUI Forge) |
| Capability | 8 | 8 | 8 | 5 | 9 | 7 | 8 | **7.5** | KEEP (TOGAF-aligned) |
| Change | 7 | 9 | 6 | 10 | 4 | 7 | 9 | **7.5** | KEEP (OpenSpec canon) |
| Engagement | 7 | 8 | 7 | 8 | 8 | 6 | 7 | **7.5** | KEEP |
| Curation | 9 | 6 | 6 | 7 | 9 | 8 | 7 | **7.5** | KEEP + DEFINE-STRICTLY |
| Entity (catalog item, post-rename) | 5 | 8 | 6 | 10 | 6 | 5 | 8 | **6.5→7.5** | KEEP (raised from 6.5 after Pavel feedback — ORM canon, Russian "сущность" natural) |
| Studio (company) | 5 | 9 | 4 | 10 | 4 | 8 | 9 | **7.0** | DEFINE-STRICTLY (always "Rapoport Studio") |
| Primitive | 6 | 8 | 7 | 8 | 7 | 7 | 7 | **7.0** | KEEP (Radix alignment) |
| Mirror (deprecated) | 4 | 7 | 7 | 10 | 5 | 8 | 9 | **7.0** | RETIRE (Pavel decision — Mirror name fully replaced by Brand) |
| NFR | 8 | 7 | 8 | 5 | 9 | 6 | 6 | **7.0** | ADD with full name on first mention |
| Surface | 4 | 7 | 5 | 10 | 5 | 8 | 6 | **6.5** | DEFINE-STRICTLY (distinguish from Material surface) |
| Tier | 5 | 7 | 5 | 10 | 6 | 6 | 8 | **6.5** | RENAME or DEFINE-STRICTLY (SaaS pricing clash) |
| Muse | 3 | 6 | 6 | 10 | 4 | 9 | 8 | **6.5** | KEEP-AWARE (market crowded) |
| Story (rejected for narrative file) | 4 | 8 | 4 | 10 | 4 | 8 | 9 | **6.5** | RENAME → Narrative |
| Narrative (replacement) | 7 | 7 | 8 | 7 | 8 | 7 | 7 | **7.5** | ADD (Pavel accepted) |
| Persona (agent lifecycle) | 3 | 5 | 4 | 8 | 6 | 7 | 8 | **5.5** | RENAME → Role |
| Studio (workspace meaning) | 3 | 6 | 2 | 10 | 4 | 6 | 7 | **5.5** | RENAME → Workspace |
| Domain (Tier-2 meaning) | 2 | 6 | 2 | 10 | 3 | 5 | 9 | **5.0** | RETIRE this meaning |
| Module | 3 | 6 | 2 | 9 | 2 | 4 | 9 | **5.0** | RETIRE |
| Component (in spec-prose) | 3 | 7 | 3 | 8 | 2 | 4 | 9 | **5.0** | RETIRE (keep React-component meaning only) |
| Context | 3 | 6 | 2 | 9 | 1 | 4 | 9 | **4.5** | RETIRE |

## Final decisions (post-Pavel feedback)

### Renames (3 immediate, 1 deferred to separate change)
- Studio (when meaning apps/studio) → **Workspace**
- Story (narrative file) → **Narrative**
- Tier → **Class** *(optional — Pavel did not commit; can be deferred)*
- Persona (lifecycle) → **Role** *(LATER — requires code migration in lifecycle.yaml + stage-actions.ts)*

### Define-strictly (5)
Studio (company-only meaning), Surface, Primitive, Capability, Curation, Identity.

### Add (12)
Brand, Workspace, Identity, Perception, Narrative, User persona, Problem statement, Success metric, User flow, Hypothesis, Acceptance criteria, NFR.

### Retire from spec-prose (4)
Context, Module, Component, Domain (without qualifier).

### Keep (and document with TOGAF/ORM canon alignment)
Spec, Change, Capability, Forge, Muse, Brand, Engagement, Entity, Sub-product, Curation.

## Transition narrative best practices

From [InfoQ enterprise SDD](https://www.infoq.com/articles/enterprise-spec-driven-development/), [SoftwareSeni adoption playbook](https://www.softwareseni.com/rolling-out-spec-driven-development-the-team-adoption-and-change-management-playbook/), [Marmelab SDD critique](https://marmelab.com/blog/2025/11/12/spec-driven-development-waterfall-strikes-back.html):

1. **Phased adoption (30/60/90)** — pilot → expand → org-wide. For Rapoport: not org-scaling but specialist-onboarding cadence.
2. **Avoid SpecFall antipattern** — show SDD as fluid, not rigid. Studio "Curation" already supports this; document in `methodology/spec-driven-transition.md`.
3. **Pair-training (PM + engineer + designer)** writes the first spec together — Ubiquitous Language formed in trialog, not solo.
4. **Tooling locks the spec to code** — bundle regen + tripwire = anti-drift mechanism. Studio already has this in `tools/build-openspec-bundle.mjs`.
5. **Cagan "operating model"** vocabulary maps to studio "Curation". Borrow phrasing for PM bridge.

## Implications for Rapoport Studio

- Glossary in `_root/glossary.md` is **two-column** (Product → OpenSpec) + collision matrix as third column.
- 4 renames go into `tasks.md` of Epic 1; Persona→Role deferred to separate code-touching change.
- Define-strictly entries get two lines each: "Studio = the company Rapoport Studio SRL. NOT apps/studio (which is Workspace)."
- The transition document is the primary onboarding artifact for all 13 schools (Engineering, Product, BA, etc.) — it is read first.

---

## Sources

- [GitHub Spec Kit spec-driven.md](https://github.com/github/spec-kit/blob/main/spec-driven.md)
- [Fission-AI OpenSpec concepts.md](https://github.com/Fission-AI/OpenSpec/blob/main/docs/concepts.md)
- [AWS Kiro hooks docs](https://kiro.dev/docs/hooks/)
- [BMad Method guide](https://buildmode.dev/blog/mastering-bmad-method-2025/)
- [Anthropic Skills best practices](https://platform.claude.com/docs/en/agent-skills/best-practices)
- [Anthropic Skills docs](https://code.claude.com/docs/en/skills)
- [Ubiquitous Language — Fowler](https://martinfowler.com/bliki/UbiquitousLanguage.html)
- [Bounded Context — Fowler](https://martinfowler.com/bliki/BoundedContext.html)
- [DDD Glossary — DDD Community](https://www.dddcommunity.org/resources/ddd_terms/)
- [TOGAF Business Capabilities](https://pubs.opengroup.org/togaf-standard/business-architecture/business-capabilities.html)
- [Material Design 3 Glossary](https://m3.material.io/foundations/glossary)
- [Atlassian Design Tokens](https://atlassian.design/foundations/design-tokens)
- [Radix Primitives](https://www.radix-ui.com/primitives)
- [Anatomy of a Primitive — Vercel Academy](https://vercel.com/academy/shadcn-ui/anatomy-of-a-primitive)
- [Marty Cagan SVPG Product Discovery](https://www.svpg.com/product-discovery/)
- [SVPG's Four Product Risks — RoadmapOne](https://roadmap.one/blog/posts/blog6-6-svpg-product-risks/)
- [NN/g — Personas Make Users Memorable](https://www.nngroup.com/articles/persona/)
- [Ramotion Brand Glossary](https://www.ramotion.com/blog/glossary-of-branding-terms/)
- [Literal Humans — Brand Image vs Identity](https://literalhumans.com/brand-image-vs-brand-identity-from-perception-to-recognition/)
- [WorkOS — Multi-tenant permissions: Slack, Notion, Linear](https://workos.com/blog/multi-tenant-permissions-slack-notion-linear)
- [InfoQ — Enterprise Spec-Driven Development](https://www.infoq.com/articles/enterprise-spec-driven-development/)
- [SoftwareSeni — Rolling Out SDD playbook](https://www.softwareseni.com/rolling-out-spec-driven-development-the-team-adoption-and-change-management-playbook/)
- [Marmelab — SDD Waterfall Strikes Back](https://marmelab.com/blog/2025/11/12/spec-driven-development-waterfall-strikes-back.html)
- [HuggingFace Stable Diffusion WebUI Forge](https://huggingface.co/spaces/fluxdev/stable-diffusion-webui-forge)
- [Wikipedia — Forge (software)](https://en.wikipedia.org/wiki/Forge_(software))
