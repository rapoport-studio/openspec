---
title: Glossary — Rapoport Studio OpenSpec Vocabulary
status: stable
owner: pavel
audience: [ai, architect, executor]
date: 2026-05-12
tldr: Defines the 10 highest collision-risk terms in the Rapoport Studio vocabulary. Each entry states the studio canon to resolve ambiguity with industry and tool-ecosystem overloads.
---

# Glossary — Rapoport Studio OpenSpec Vocabulary

> Source of truth for the 10 terms most likely to cause confusion between Rapoport Studio's vocabulary and industry canon, LLM context windows, or competing toolchains. The collision list is sourced from [`_methodology/research/sdd-terminology-scorecard.md`](research/sdd-terminology-scorecard.md). Each entry defines what the term means inside this repo, why it collides, and the single canonical sentence that resolves ambiguity.

---

### Studio

**Studio** has two incompatible meanings in this repository. In company context it refers to Rapoport Studio SRL — the legal entity and brand. In the codebase it was also the name of `apps/studio`, the internal workspace application. The collision maps onto a crowded external namespace (Android Studio, Visual Studio, ChatGPT Studio, Anthropic Console Studio) and creates grep-noise: every appearance of `studio` in specs and code must be read with its scope manually resolved.

**Canon:** In all OpenSpec prose, **Studio** refers exclusively to the company Rapoport Studio SRL; the workspace application is **Workspace** (`apps/studio` is being renamed per the sdd-terminology-scorecard final decisions).

---

### Muse

**Muse** is the internal name for the AI planning agent that drafts change proposals, designs, and tasks for review by Forge Builder. The name collides with Microsoft Muse (generative AI sound tool), Lightricks LTX-Video Muse, HuggingFace MUSE, and Muse Group, among others. In LLM context windows the word activates associations with creative or generative AI tools unrelated to spec-drafting.

**Canon:** **Muse** is the Rapoport Studio AI planner (Architect mode); it is not a creative-content model, a music tool, or any third-party product.

---

### Mirror

**Mirror** is a Tier-1 Brand-sphere entity (Atlas D-ATLAS-6 ratification, 2026-05-22): a parent-anchored sub-brand produced by the Brand Mirroring operation — a country- or audience-localised derivative that inherits a parent Brand's identity DNA. See `_root/glossary.md § Mirror` for the live definition, `openspec/brands/studio/entities/mirror.md` for the entity portrait, and `_methodology/product-anatomy.md § Brand Mirroring` for the operation.

The 2026 sdd-terminology-scorecard retired the *brand-intelligence feature* naming (which collided with Mirror.xyz, Mirror Protocol, and Apple AirPlay Mirror) — that brand-intelligence agent is now **Brand**. The old perceptual-spec-layer capability was retired 2026-05-20 by RAP-1125 and fully absorbed into Brand; per Atlas D-ATLAS-10 there is no longer a tombstone file at `openspec/current/mirror.md`.

**Canon:** `Mirror` is the live Tier-1 Brand-sphere entity for a parent-anchored sub-brand — see `_root/glossary.md § Mirror`.

---

### Canvas

**Canvas** is the planned collaborative editing surface for multi-author studio sessions (package `@rapoport-studio/canvas`, currently deferred). The name collides catastrophically in 2026: Figma Canvas, Notion Canvas, Slack Canvas, Apple Canvas, and Instructure Canvas LMS all use the term for structurally different concepts, and the collision is active in every major design, collaboration, and LMS ecosystem simultaneously.

**Canon:** **Canvas** in Rapoport Studio refers only to the deferred real-time collaboration layer (`packages/canvas/`); every other product that uses "canvas" is out of scope and out of namespace.

---

### Surface

**Surface** is a UI placement term in Rapoport Studio specs: it denotes a discrete viewport or rendering channel where a capability is exposed (e.g., the Telegram surface, the Workspace surface). This clashes with Material Design 3's *surface* (tone-based depth layer controlling elevation and color), Meta Foundational Design System's surface semantics, and Microsoft Surface (hardware). Readers coming from any of these contexts will import the wrong mental model.

**Canon:** In Rapoport Studio OpenSpec, **Surface** means a discrete UI host or integration channel; it carries no elevation, depth, or hardware semantics.

---

### Persona

**Persona** appears in two unrelated contexts inside the studio. In UX research it refers to a user archetype (NN/g canonical meaning: a research-based representation of key user segments). In the studio's agent lifecycle, *Persona* was used for the role a Forge agent adopts during a build phase (Architect, Builder, Verifier). The lifecycle usage clashes with NN/g UX personas, marketing buyer-personas, and brand persona frameworks.

**Canon:** **Persona** in Rapoport Studio now refers only to user personas (NN/g canon); agent lifecycle roles are called **Roles** (Architect, Builder, Verifier) — the rename is in progress per the sdd-terminology-scorecard final decisions.

---

### Domain

**Domain** carries two colliding meanings inside the studio. In DNS/infrastructure context it means a hostname or namespace. In early spec-prose it was borrowed from Domain-Driven Design to group related entities at the Tier-2 level — but DDD's Domain has a stricter meaning (a sphere of knowledge modelled with Ubiquitous Language and bounded contexts), and the bare word triggers grep hits from DNS configuration files alongside architectural ones.

**Canon:** In Rapoport Studio OpenSpec, **Domain** without qualification is retired; use **bounded context** (DDD sense) or an explicit infrastructure qualifier (`apex domain`, `subdomain`) depending on the intended meaning.

---

### Story

**Story** appeared in early Rapoport Studio specs for a narrative description of a user journey — roughly equivalent to an agile user story or a Spec Kit Story artifact. The term collides with agile user stories, BMad Method story files (`{epicNum}.{storyNum}.story.md`), Spec Kit Story artifacts, and social-media Stories (Instagram, Snapchat), making its meaning entirely context-dependent and stripping it of precision.

**Canon:** In Rapoport Studio OpenSpec, **Story** is retired; use **Narrative** for user-journey descriptions (the rename was accepted per the sdd-terminology-scorecard final decisions).

---

### Context

**Context** is among the most overloaded terms in the stack: React context (`React.createContext`), LLM context window, DDD bounded context, and generic English "the situation." In spec-prose it generates grep noise at scale (thousands of false positives across source and config files) and forces every reader to re-derive the intended meaning from surrounding text on each occurrence.

**Canon:** **Context** is retired from Rapoport Studio OpenSpec prose; qualify always — write **LLM context window**, **React context**, or **bounded context** depending on the intended meaning.

---

### Module / Component

**Module** and **Component** are both reserved in the JavaScript/TypeScript ecosystem for distinct, well-established concepts: ES modules, React components, Angular components, CSS modules. Using either term in OpenSpec prose to describe higher-level architectural units or capability groupings imports that ecosystem ambiguity into spec language, making every mention require manual disambiguation by the reader.

**Canon:** **Module** and **Component** are retired from Rapoport Studio OpenSpec prose as architectural grouping terms; use **Capability**, **Package**, or **Sub-product** instead.
