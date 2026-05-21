---
title: Product anatomy — three spheres, integration, lifecycle, naming
status: stable
owner: pavel
audience: [pavel, ai, collaborator]
tldr: The studio's product anatomy across three internal spheres (Spec, Brand, Forge), how they integrate into a Product, the engagement lifecycle, the conversational naming convention, and the folder structure that follows.
---

# Product anatomy

> The studio's product is built from three internal spheres of work. This document records what each sphere covers, how they integrate into a Product per engagement, the lifecycle by which a Product is produced, and the conversational and folder conventions that follow.

Authority: `openspec/archive/2026-05-21-define-studio-product-model/design.md` ratifies decisions D-PM-1 through D-PM-7. This file is the methodology expression of those decisions.

---

## Three internal spheres

The studio operates in three named spheres. Each is a noun and a verb-stem; each has a canonical home in the repo.

| Sphere | Covers | Where it lives |
|---|---|---|
| **Spec** | architectural specifications — capabilities, behaviour, contracts | `openspec/current/` + `openspec/changes/` |
| **Brand** | identity of one engagement — voice, vocabulary, design, archetypes, surfaces | `openspec/brands/<slug>/` |
| **Forge** | the compile / build / run engine and its agents | `packages/forge/` + `openspec/current/forge/` + `openspec/current/agents-overview.md` |

The three are sphere-level, not capability-level. Each contains one or many capabilities; each is wider than any single capability.

### Forge is internal in public communication

The public-facing pitch on `/services`, `/method`, and `/` is **two-pillar**: `Spec + Brand`. Forge stays in repo, in code, in collaborator vocabulary, in pricing internals — but not in marketing copy. The cast-rename rule (`cast-naming-role-descriptors`) already operationalises this externally: when Forge appears at all on a public surface, it appears as `Builder`. The default is silence.

This is D-PM-2 in `define-studio-product-model/design.md`.

### Brand is the umbrella, not a sibling of Design

Inside Brand sit Identity (voice, vocabulary, mental model), Design System (philosophy, themes, surfaces) with its sub-files (Tokens, Components, Motion), Archetypes (per-role context-aware portraits), and Surfaces (per-rendered-target manifests). Today, `openspec/current/design-*.md` exists at the platform level because Rapoport Studio has been the only brand. The future-correct location is `openspec/brands/studio/`; the migration is `migrate-studio-design-to-brand` (deferred).

```
openspec/brands/<slug>/
├── identity.md
├── archetypes/<role>.md
├── design-system.md
├── design-tokens.md
├── design-components.md
├── design-motion.md
└── surfaces/<surface>.md
```

Rapoport Studio's own brand currently lives at `openspec/brands/studio/`. The slug `studio` predates the explicit naming of the spheres; folder rename is deferred (`rename-brand-folder-studio-to-rapoport-studio`).

### Brand Mirroring

A Brand may produce one or more **Mirrors** — sub-brands derived from it. A Mirror inherits the parent Brand's identity DNA (voice anchors, design tokens, archetypes) and overrides only the fields that diverge per locale or audience (name, vocabulary, surface inventory, copy). A Mirror is always anchored to one parent and cannot float free; it is not a peer Brand.

The folder shape, frontmatter, and inheritance rules for Mirrors are scoped to a follow-up change (`add-brand-mirroring-schema`). This document locks the vocabulary only.

---

## Product is the integration

```
Product (one engagement) = a Spec  + a Brand  + Forge run
                            (bespoke) (bespoke)  (shared service, hidden externally)
```

Spec and Brand are bespoke per engagement. Forge is shared infrastructure. Per-engagement Product manifests live in `openspec/products/<slug>/manifest.md` as thin pointer-files binding Spec(s), a Brand, and Forge access; full manifest contract is ratified in a separate change.

Rapoport Studio is its own zero-th Product: Studio's Spec lives in `openspec/current/`, Studio's Brand lives in `openspec/brands/studio/`, Forge is the engine the Studio built and runs. The studio is its own first client.

This is D-PM-3.

---

## Marketplace is orthogonal

The Marketplace is **not** part of the studio's vertical offer. It is a horizontal peer-to-peer transaction layer where users sell each other templates, re-usable specs, ideas, and knowledge across engagements. The Marketplace owns transactions; what is transacted lives in the existing three spheres.

Detail is deferred. See `_methodology/marketplace-charter.md` for the scope-deferred note.

This is D-PM-4.

---

## Engagement lifecycle

The development arc inside one engagement:

```
0. Empty                  ← nothing yet
1. Discovery              ← chat, form, intake — figuring out what
2. Inject                 ← first Spec scaffolded; entities, archetypes, competitors, problems, infrastructure accumulate
3. Spec render            ← openspec/current/<capability>.md materialises; architect voice (Rapoport's)
4. Brand render           ← openspec/brands/<client-slug>/ materialises; client's voice
5. Forge run              ← functional product compiled by Forge; deployed in apps/
6. Living                 ← all three renders co-exist and update via OpenSpec changes
```

Brand follows Spec, not the reverse. Architectural clarity comes first; brand expresses what the architecture has revealed. Forge consumes both and emits the deployed artifact.

The two audiences read different renders:

- **Architects / collaborators** read the Spec render — markdown files, the workspace UI at `/spec/[id]`, the public mirror, the OpenSpec bundle.
- **Clients** read the Brand render — Brand Book, public-facing site, identity package.
- **Users** use the Product render — the deployed app.

---

## Conversational naming convention

Across every surface where work is named — Claude Project chats, Linear issues, file paths, commit messages — the sphere word is the prefix.

| Surface | Format | Example |
|---|---|---|
| Claude Project chat (Spec sphere) | `Spec: <Title Case>` | `Spec: Background Patterns` |
| Claude Project chat (Brand sphere) | `Brand: <slug>` | `Brand: VivoD` |
| Claude Project chat (Forge sphere) | `Forge: <component>` | `Forge: build-orchestrator` |
| Claude Project chat (Mirror) | `Mirror: <parent>/<mirror-slug>` | `Mirror: vivod/vivod-fr` |
| `openspec/current/` file | `<kebab-case>.md` | `background-patterns.md` |
| `openspec/changes/` folder | `<verb-noun>-slug/` | `add-background-pattern-system/` |
| `openspec/brands/` folder | `<slug>/` | `studio/` |
| Linear epic | `Spec — <Title Case>` | `Spec — Background Patterns` |

One chat lives at one sphere level. A new chat is opened when a new capability file appears in `current/`, or when the current chat exceeds a context-budget threshold and a fresh context is needed.

---

## Notes on retired vocabulary

The word **Mirror** previously referred to a perceptual spec layer parallel to OpenSpec (`openspec/current/mirror.md`, "the client's view of their product, parallel to the OpenSpec technical layer"). That role is fully absorbed into **Brand** as defined above; the word `Mirror` is re-scoped to mean Brand Mirroring only. The current capability file remains in place pending `retire-mirror-capability`.

The word **Canvas** previously referred to the workspace UI surface at `/canvas/[id]`. It is being renamed to **Spec** at `/spec/[id]` under `rename-spec-screen` (RAP-1018). In current and future prose, use `Spec` for the screen and reserve `Canvas` only for direct references to the legacy route.

These are D-PM-5 in `define-studio-product-model/design.md`.

---

## Rendering — open

How a Spec is rendered to a reader splits into two distinct concerns.

**In-document navigation — closed.** Shipped via `add-spec-navigator` (archived `openspec/archive/2026-05-21-add-spec-navigator/`): `<SpecNavigator>` provides TOC, anchor-links, sticky-header, breadcrumb, and density variants matching the `<EntityCard>` `inline / row / card / detail` shape.

**Multi-modal rendering.** Open research area. A Spec is one markdown source; the platform may render it as document, diagram, timeline, dependency graph, state machine, comparison/diff, or other shapes per the content's structure. Existing partial precedents in repo:

- `<EntityCard>` four-variant density system (`add-entity-system`).
- Multi-file capability shape (`current/db/`, `current/forge/`, `current/agents/`) — already lets one large Spec break into navigable pieces.
- `<canvas-preview-mode>` (`add-canvas-preview-mode`) — first adaptive-rendering precedent on the workspace surface.
- `<DiffView>` + chart system (`define-chart-system`) — code-and-data render precedents.

Each is a partial answer. The full question is registered as `Q-PM-1` in `openspec/open-questions.md`; resolution is gated on user-research with real founders reading real Specs.

---

## Cross-references

- `_methodology/marketplace-charter.md` — orthogonal P2P layer (scope deferred).
- `_internal/platform/manifesto.md § Values` — the five platform values that ground the platform (anti-monopoly, conscious living, democratic accessibility, polyglot collaboration, specialist ecosystem in the chat).
- `_internal/platform/ontology.md` — Idea → Spec → Product flow, Product = integration of three spheres, Brand → Mirrors.
- `_root/glossary.md § Appendix 3` — locked vocabulary for `Spec`, `Brand`, `Forge`, `Product`, `Marketplace`, `Mirror`.
- `brands/studio/identity.md § Values` — Rapoport Studio's four brand attributes (green, human-friendly, responsibility-bearing, democratic).
- `openspec/archive/2026-05-21-define-studio-product-model/` — the ratifying change with full D-PM-N decisions.
