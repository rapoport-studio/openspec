# Home — Decorative-Zone Structural Audit

> **Status:** standalone structural research. NOT tied to a ratified change.
> **Audited:** `apps/web/src/app/[locale]/(marketing)/page.tsx` and every section component it renders, on `main` at the time of writing.
> **Purpose:** give a downstream design pass everything it needs to propose where (if anywhere) decorative pattern fills could sit on the marketing home page, without re-reading the component tree.
> **Author note:** this audit deliberately does **not** propose specific patterns or placements. It maps text-free structure only.

---

## 0. Read-this-first — premise correction

This audit was commissioned with a brief that assumed an in-flight OpenSpec change `add-background-pattern-system`, a ratified decision `D-DS-8` with a "narrow carve-out", and 8 ratified pattern primitives (`dots`, `graph`, `hatch`, `cad`, `iso`, `frame`, `phi`, `modulor`). **None of those exist in the repo.** Verified:

- `openspec/changes/` contains 27 changes — none about background patterns.
- `D-DS-8` appears nowhere in `openspec/`.
- No pattern primitives (`modulor` / `phi` / `iso` / etc.) exist in `packages/ui` or anywhere in the tree.
- The 8-pattern list and density budget are **not ratified** — they are reproduced in §1 only as *proposed* input, clearly marked.

What *is* ratified and binding today:

- [`openspec/current/design-tokens.md:70-74`](../../current/design-tokens.md) — colour-level hard rules:
  > - No gradients. No images behind text.
  > - No shadows. Elevation is expressed through surface contrast … and corner brackets.
  > - Background is flat warm ivory (light) or flat near-black (dark). **Always.**
- [`openspec/current/home/blocks.md:165`](../../current/home/blocks.md) — the **Hero block** spec:
  > No images. No video. **No background pattern.** The block is type + frame.

So any pattern system is, as of this audit, a **net-new proposal that must first amend `design-tokens.md` and `home/blocks.md`** via a change proposal. This audit is the structural groundwork for that proposal; it is not evidence the proposal is approved.

---

## 1. Constraints for a downstream design pass (PROPOSED — not ratified)

A design pass consuming this audit must treat the following as **proposed**, pending a real OpenSpec change:

**Hard invariant (ratified, non-negotiable):** no pattern may sit underneath any text-bearing element — `h1`/`h2`/`h3`/`p`/`li`/`dt`/`dd`/`blockquote`/`a`/`summary`. This is the conservative reading of `design-tokens.md:74` ("Background is flat … Always") and `:72` ("No images behind text"). A pattern fill is only admissible in a **provably text-free zone**.

**Proposed pattern vocabulary (NOT yet in the codebase — do not assume these exist):**
`dots`, `graph`, `hatch`, `cad`, `iso` (tiled CSS utilities) · `frame`, `phi`, `modulor` (composition components).

**Proposed density budget (NOT ratified):** `dots`/`graph`/`hatch` — 1 per section, 2 per page max. `cad`/`iso` — hero / work cards. `frame`/`phi`/`modulor` — hero only, 1 per page.

**Tone-token reality check — the brief's tone names are wrong for this repo.** The commissioning brief said "`accent` = terracotta". In this design system that is inverted:

| Token | Actual hue | Evidence |
|---|---|---|
| `--primary` | **terracotta** | `tactical-frame.tsx:30` — `primary: "… [--bracket-color:var(--primary)]"` documented "Terracotta corners"; `design-tokens.md:84` `--status-live` "(terracotta)" aliases `--primary` |
| `--accent` | **cool blue** | `tactical-frame.tsx:31` — `accent` documented "Cool-blue corners — for technical / blueprint frames" |
| `--color-blueprint` | blueprint blue | used for the Hero rule + diagram lines |

A design pass MUST use the repo's real token names. "Terracotta" = `--primary`, not `--accent`.

**Theme:** the public marketing surface renders **dark by default** — `apps/web/src/app/[locale]/layout.tsx:78` sets `<ThemeProvider attribute="class" defaultTheme="dark" enableSystem={false}>`. `bg-canvas` is therefore near-black on first paint. Any pattern contrast must be verified against the **dark** canvas first. A `class`-based theme attribute means a light mode is technically reachable; verify on both if a toggle is exposed.

---

## 2. Page shell & shared geometry

- Page composition: [`apps/web/src/app/[locale]/(marketing)/page.tsx:114-135`](../../../apps/web/src/app/[locale]/(marketing)/page.tsx). 14 blocks render as direct children of a single `<div className="flex flex-col w-full">`. No per-section wrapper, no inter-section spacer element.
- Container: [`layout.tsx:90`](../../../apps/web/src/app/[locale]/layout.tsx) — `<main className="… max-w-page mx-auto px-6 md:px-12 pt-[64px]">`. `--width-page = 75rem` = **1200px** (`globals.css:501`); horizontal padding **24px mobile / 48px desktop**; `pt-[64px]` clears the fixed `<Nav>`.
- Every `<section>` inherits the 1200px content box. Sections set their own vertical rhythm via `py-*`; none set their own `max-width`.
- **Section dividers:** all inter-section borders are `md:border-t md:border-grid` — i.e. **desktop-only**. On mobile there is no divider line between sections. The sole exception is `ForLegacy`, which uses unconditional `border-t border-grid` (`for-legacy.tsx:17`). The Hero has **no** `border-t`.
- **No `<DividerTechnical>` and no blueprint-line motif is used anywhere on the home page** (`grep` for `DividerTechnical` in `apps/web/src` → 0 hits). The only blueprint-line element on the page is the one-off `<span data-tactical-line>` directly below the Hero frame (`hero.tsx:88-102`). There is no recurring "boundary rule" motif for a pattern strip to attach to.
- **No cream / non-`canvas` surface anywhere on the home page** (`grep` for `bg-cream`/`bg-subtle`/`bg-bg-2` in `components/home` → 0 hits). Every section sits on the flat `bg-canvas`.

**Vertical-padding cheat-sheet** (Tailwind `py-*` → px): `py-8`→32 · `py-16`→64 · `py-32`→128 · `py-40`→160.

**Zone-overlap warning for the downstream pass:** a section's *top padding* and the previous section's *bottom padding* are the **same physical band**, split by the `md:border-t` line. On desktop a typical boundary = 128px (prev bottom pad) + 128px (this top pad) ≈ **256px of contiguous text-free space straddling the divider**. Treat each boundary as ONE shared band, not two independent strips, or you will double-count.

**`SectionHead` anatomy** ([`packages/ui/src/components/section-head.tsx`](../../../packages/ui/src/components/section-head.tsx)) — the shared header used by 10 of 14 sections: `<header>` → `MonoLabel` eyebrow (`mb-6` = 24px) → `<h2>` heading (clamp 32-48px, `mb-6` when a lede/children follow) → optional lede `<p>` (`mb-16` = 64px). The **eyebrow is the first text element**; the section's top padding is the only text-free band above it.

---

## 3. Section inventory

Order, ids and padding are from the live files. `[VERIFIED]` = component file opened in this audit run.

| # | Section id | Component | y-padding | Role | Audience | Density | Existing chrome | |
|---|---|---|---|---|---|---|---|---|
| 1 | `#hero` | `home/hero.tsx` | `py-8 md:py-32` | opening | mixed | medium | `TacticalFrame` (animated, `primary`/terracotta) + one-off blueprint line | `[VERIFIED]` |
| 2 | `#what` | `home/what-we-build.tsx` | `py-16 md:py-32` | mechanism | mixed | high | `OrchestraDiagram` (SVG) + `PullQuote` | `[VERIFIED]` |
| 3 | `#now` | `home/now-building.tsx` | `py-16 md:py-32` | proof | client | medium | `WorkCard`→`EntityCard` grid; featured card in `TacticalFrame` | `[VERIFIED]` |
| 4 | `#how` | `home/how-we-work.tsx` | `py-16 md:py-32` | process | client | medium-high | `StepRow` numerals + internal `border-t` rules | `[VERIFIED]` |
| 5 | `#connections` | `home/connected-stack.tsx` | `py-16 md:py-32` | mechanism | mixed | high | bordered 3-col tile grid (`gap-px border border-grid`) | `[VERIFIED]` |
| 6 | `#operating-model` | `home/operating-model.tsx` | `py-16 md:py-32` | mechanism | founder / specialist | very high | `OperatingModelDiagram` (many nested `TacticalFrame`s) + `OperatingModelProof` | `[VERIFIED]` |
| 7 | `#for-founders` | `home/for-founders.tsx` | `py-16 md:py-32` | segmentation | founder | medium | 3-col card grid with `md:border-s` rails | `[VERIFIED]` |
| 8 | `#for-legacy` | `home/for-legacy.tsx` | `py-32 md:py-40` | segmentation | legacy / founder | low | none (header + lede + 1 link) | `[VERIFIED]` |
| 9 | `#who-we-work-with` | `home/who-we-work-with.tsx` | `py-16 md:py-32` | segmentation | client | medium | 2-col card grid (`border-t` per card) | `[VERIFIED]` |
| 10 | `#network-and-community` | `home/the-network-and-community.tsx` | `py-16 md:py-32` | proof / segmentation | specialist | medium | `PersonCard` grid + `layer-divider` `border-t` (conditional) | `[VERIFIED]` |
| 11 | `#services` | `home/services-section.tsx` (+ `services-catalog.tsx`) | `py-16 md:py-32` | mechanism / offer | client | very high | `ServiceCard` grid + nested `SectionHead` | `[VERIFIED]` |
| 12 | `#build-on-platform` | `home/build-on-platform.tsx` | `py-16 md:py-32` | segmentation | specialist | low-medium | none (2-col paragraph grid) | `[VERIFIED]` |
| 13 | `#lab` | `home/lab-preview.tsx` | `py-16 md:py-32` | proof | mixed | medium | `PostCard` grid | `[VERIFIED]` |
| 14 | `#contact` | `shared/closing-cta.tsx` | `py-16 md:py-32` | closing | mixed | low | `AudienceTag` | `[VERIFIED]` |

**Conditional rendering — adjacency is NOT stable:**
- `#now` returns `null` when there is no in-progress work (`now-building.tsx:14`).
- `#lab` returns `null` when there are no posts (`lab-preview.tsx:22`).
- `#network-and-community`'s roster grid (Layer B) is conditional (`the-network-and-community.tsx:65`); `OperatingModelProof` inside `#operating-model` can return `null` (`operating-model-proof.tsx:30`).

A downstream pass must NOT assume "section N is always followed by section N+1". Any boundary-band treatment must survive `#now` and/or `#lab` being absent.

---

## 4. Per-section decorative zones

Pixel ranges: **D** = desktop (`md:`), **M** = mobile. The "above-section" band is the shared boundary band described in §2; it is listed once, on the lower section.

### Section 1 — `#hero`
**Component:** `apps/web/src/components/home/hero.tsx` · **Role:** opening · **Chrome:** `TacticalFrame animated tone="primary"` wrapping all content (`hero.tsx:50`); a one-off blueprint line below the frame (`hero.tsx:88-102`, `mt-12`, `hidden md:flex`, `h-2px max-w-720px`).
**Available decorative zones:**
- above-section: **none** — first block; sits directly under the 64px Nav offset. No `border-t`.
- top-decorative-strip: D ~128px / M ~32px — section top padding, above the `<TacticalFrame>`. Text-free, but the frame's animated terracotta brackets deploy *from frame centre outward* into this region's lower edge.
- bottom-decorative-strip: D ~128px / M ~32px — below the blueprint line. The blueprint line itself occupies a `mt-12` (48px) sub-band; genuine text-free space remains below it.
- card-edge-band: n/a — the Hero "card" is a `TacticalFrame`, not a card; its inner padding (`px-6 py-10 md:px-16 md:py-24`) holds text.
- text-free side-rail: no — `TacticalFrame` spans the full content box.
**Friction:** Hero block is **spec-locked against background pattern** by `home/blocks.md:165` ("No background pattern. The block is type + frame."). The frame + animated brackets + blueprint line already constitute a dense decorative anchor. Treat `#hero` as **excluded** from any pattern fill unless the spec is amended.

### Section 2 — `#what`
**Component:** `what-we-build.tsx` · **Role:** mechanism · **Chrome:** `OrchestraDiagram` inline SVG (`my-16`, `max-w-720px`, centred) + centred `PullQuote` (`mt-24`, `max-w-40ch`).
**Available decorative zones:**
- above-section (`#hero`↔`#what`): D ~256px shared band (Hero has no bottom border; `#what` has `md:border-t`). M: no divider.
- top-decorative-strip: D ~128px / M ~64px — above the eyebrow.
- bottom-decorative-strip: D ~128px / M ~64px — below the `PullQuote`.
- card-edge-band: no cards.
- text-free side-rail: **yes (partial)** — body paragraphs cap at `max-w-[56ch]` (`what-we-build.tsx:40,44,49`); a right rail exists beside them. The `OrchestraDiagram` and `PullQuote` are *centred* and narrower than the box, leaving symmetric left+right rails beside those two elements specifically.
**Friction:** the `OrchestraDiagram` (blueprint-stroke SVG lines) is itself a line-art motif — a tiled line pattern adjacent to it risks visual collision. The `PullQuote` is the page's first emphatic typographic moment; the band just below it is a natural pause.

### Section 3 — `#now`
**Component:** `now-building.tsx` · **Role:** proof · **Chrome:** `WorkCard` grid → `EntityCard variant="card"`; the first card may be `featured` → wrapped in `TacticalFrame`.
**Available decorative zones:**
- above-section (`#what`↔`#now`): D ~256px. M: no divider.
- top-decorative-strip: D ~128px / M ~64px.
- bottom-decorative-strip: D ~128px / M ~64px — below the `mt-16` link CTA (`now-building.tsx:49`).
- card-edge-band: **unverified** — `WorkCard` renders through `EntityCard` (`packages/ui/src/components/entity-card.tsx`), which this audit did not open. Whether `EntityCard variant="card"` exposes a text-free top/bottom band requires a separate read. **Flag for the downstream pass.**
- text-free side-rail: no — the card grid (`grid-cols-1 md:grid-cols-2`) spans the full box.
**Friction:** section is conditionally `null`. The featured card already carries a `TacticalFrame` — a second framing motif nearby competes.

### Section 4 — `#how`
**Component:** `how-we-work.tsx` (client variant) · **Role:** process · **Chrome:** `StepRow` × 4, numerals in mono terracotta-`accent`… (actually `text-accent` = cool blue, `step-row.tsx:38`); rows separated by internal `border-t border-grid` (`step-row.tsx:30`).
**Available decorative zones:**
- above-section (`#now`↔`#how`): D ~256px. M: no divider.
- top-decorative-strip: D ~128px / M ~64px.
- bottom-decorative-strip: D ~128px / M ~64px — `StepRow` last row has `last:pb-0`, so the band starts cleanly at the section's bottom padding.
- card-edge-band: no cards. The `<ol>` is capped `max-w-[820px]`.
- text-free side-rail: **yes** — the step list caps at 820px inside the 1200px box; a ~330px right rail exists. Each `StepRow` is a `grid-cols-[96px_1fr]` — the 96px numeral gutter is text-bearing (the numeral).
**Friction:** the section already has 3 internal horizontal `border-t` rules between steps. Adding a horizontal band motif risks reading as a 4th rule.

### Section 5 — `#connections`
**Component:** `connected-stack.tsx` · **Role:** mechanism · **Chrome:** 3-col tile grid, `gap-px border border-grid` (a hairline-grid block).
**Available decorative zones:**
- above-section (`#how`↔`#connections`): D ~256px. M: no divider.
- top-decorative-strip: D ~128px / M ~64px.
- bottom-decorative-strip: D ~128px / M ~64px — below two footnote `<p>` (`connected-stack.tsx:253,257`).
- card-edge-band: no — each `ConnectionTile` is `border border-grid p-5`; the 20px `p-5` inner padding is text-free but too small to host a fill, and the tile is otherwise edge-to-edge.
- text-free side-rail: footnote paragraphs cap at `max-w-[60ch]`; the tile grid spans the full box.
**Friction:** the tile grid is already a strong hairline-grid graphic (`gap-px border` reads as a `graph`-like lattice). A `graph`/`dots` tile pattern anywhere near it is redundant by construction.

### Section 6 — `#operating-model`
**Component:** `operating-model.tsx` · **Role:** mechanism · **Chrome:** `OperatingModelDiagram` (nodes rendered as nested `TacticalFrame`s, `operating-model-diagram.tsx:36`) + 4-principle `<dl>` (each `border-t border-grid pt-6`) + `OperatingModelProof` (`border-t border-grid pt-8 mt-8`).
**Available decorative zones:**
- above-section (`#connections`↔`#operating-model`): D ~256px. M: no divider.
- top-decorative-strip: D ~128px / M ~64px.
- bottom-decorative-strip: D ~128px / M ~64px — below the 2-CTA row (`operating-model.tsx:83`).
- card-edge-band: no card grid; the diagram is composed of many small `TacticalFrame` node boxes.
- text-free side-rail: principle `<dl>` and nucleus `<p>` cap at `max-w-[820px]`/`[60ch]` → right rail exists; the diagram itself is wide.
**Friction:** highest chrome density on the page — a multi-frame diagram plus three internal `border-t` rules. Strongly resists any added pattern adjacent to its content.

### Section 7 — `#for-founders`
**Component:** `for-founders.tsx` · **Role:** segmentation · **Chrome:** 3-col `<article>` grid; on desktop cards carry `md:border-s` start-rails (`for-founders.tsx:37`).
**Available decorative zones:**
- above-section (`#operating-model`↔`#for-founders`): D ~256px. M: no divider.
- top-decorative-strip: D ~128px / M ~64px.
- bottom-decorative-strip: D ~128px / M ~64px — below the `mt-16` link CTA.
- card-edge-band: **partial** — each card opens with `Chip` after a `pt-8` (mobile) / `pt-0` (desktop) top padding; on **mobile** there is a ~32px text-free band above the `Chip` under a `border-t`. On desktop the card content starts at the top edge (no band).
- text-free side-rail: card grid spans the box; no rail.
**Friction:** cards use `border-s`/`border-t` rails as their own structure; a card-edge pattern band would compete with those rails.

### Section 8 — `#for-legacy`
**Component:** `for-legacy.tsx` · **Role:** segmentation · **Chrome:** none.
**Available decorative zones:**
- above-section (`#for-founders`↔`#for-legacy`): **`border-t border-grid` is unconditional here** (`for-legacy.tsx:17`) — the divider shows on **mobile too**. D boundary ≈ 128px (prev bottom) + 160px (this `py-40` top) ≈ **288px**. M ≈ 64 + 128 = 192px.
- top-decorative-strip: D ~160px / M ~128px — the **largest single text-free strip on the page** (`py-32 md:py-40`).
- bottom-decorative-strip: D ~160px / M ~128px — below the single link CTA. Equally large.
- card-edge-band: no cards.
- text-free side-rail: **yes** — only a `SectionHead` (heading `max-w-[20ch]`, lede `max-w-[56ch]`) and one link; a wide right rail exists.
**Friction:** **lowest-density section on the page** and the **only one with an unconditional, mobile-visible divider**. It is the most spacious decorative candidate and the only boundary that behaves identically on mobile and desktop. Note its `py` is intentionally larger than its neighbours — it is already a "breathing" block.

### Section 9 — `#who-we-work-with`
**Component:** `who-we-work-with.tsx` · **Role:** segmentation · **Chrome:** 2-col `<article>` grid, each card `border-t border-grid pt-8`.
**Available decorative zones:**
- above-section (`#for-legacy`↔`#who-we-work-with`): D ≈ 160 (prev `py-40` bottom) + 128 ≈ 288px. M: no divider (this section is `md:border-t`).
- top-decorative-strip: D ~128px / M ~64px.
- bottom-decorative-strip: D ~128px / M ~64px — the last card ends with a `<p>`; no trailing CTA.
- card-edge-band: **yes (per card)** — each card has `border-t border-grid pt-8`; the ~32px `pt-8` band above the `Chip` is text-free, under the card's top border.
- text-free side-rail: card grid spans the box.
**Friction:** asymmetric boundary — its above-band is fed by `#for-legacy`'s oversized `py-40`. No trailing CTA means the bottom strip is "clean".

### Section 10 — `#network-and-community`
**Component:** `the-network-and-community.tsx` · **Role:** proof / segmentation · **Chrome:** `PersonCard` grid (Layer B, conditional); a `data-testid="layer-divider"` block with `border-t border-grid mt-16 pt-16` (`the-network-and-community.tsx:69`).
**Available decorative zones:**
- above-section (`#who-we-work-with`↔`#network-and-community`): D ~256px. M: no divider.
- top-decorative-strip: D ~128px / M ~64px.
- bottom-decorative-strip: D ~128px / M ~64px — when Layer B renders, the last element is the `PersonCard` grid; when it does not, the last element is the community-CTA cluster.
- card-edge-band: **unverified** — `PersonCard` (`apps/web/src/components/network/person-card.tsx`) not opened. Flag.
- text-free side-rail: community body caps `max-w-[56ch]` → right rail; the roster grid spans the box.
**Friction:** an internal `layer-divider` `border-t` already exists mid-section; its presence is conditional, so any treatment keyed to it is unstable.

### Section 11 — `#services`
**Component:** `services-section.tsx` (client variant) + `services-catalog.tsx` · **Role:** mechanism / offer · **Chrome:** 5× `ServiceCard` grid + a nested `SectionHead`.
**Available decorative zones:**
- above-section (`#network-and-community`↔`#services`): D ~256px. M: no divider.
- top-decorative-strip: **occupied** — `#services` does NOT open with an eyebrow. It opens with an intro `<p>` block (`max-w-prose mb-12`) + a `foundersLink`, *then* `<ServicesCatalog>` renders its own `SectionHead` (`services-section.tsx:78-93`). The text-free band above the intro paragraph is only the section's top padding (D ~128px / M ~64px); there is **no before-eyebrow strip**.
- bottom-decorative-strip: D ~128px / M ~64px — below the `mt-16` link CTA. Note `ServicesCatalog` ends with a centred italic `<p>` note (`services-catalog.tsx:67`) before that CTA.
- card-edge-band: **unverified** — `ServiceCard` (`apps/web/src/components/services/service-card.tsx`) not opened. Flag.
- text-free side-rail: intro `<p>` is `max-w-prose`; the 3-col card grid spans the box.
**Friction:** longest section on the page (intro + 5 cards + "what we don't take on" list + italic note + CTA). The double-header structure (intro paragraph, then nested `SectionHead`) means the usual "before-eyebrow" strip does not exist.

### Section 12 — `#build-on-platform`
**Component:** `build-on-platform.tsx` · **Role:** segmentation · **Chrome:** none; 2-col paragraph grid.
**Available decorative zones:**
- above-section (`#services`↔`#build-on-platform`): D ~256px. M: no divider.
- top-decorative-strip: D ~128px / M ~64px — `SectionHead` here has **no lede** (`build-on-platform.tsx:24-28`), so the header is short.
- bottom-decorative-strip: D ~128px / M ~64px — below the `mt-16` filled button CTA.
- card-edge-band: no cards.
- text-free side-rail: the 2-col paragraph grid caps at `max-w-[80ch]` — wider than a reading column; the right rail is narrow (~ box − 80ch).
**Friction:** low chrome, short header — one of the cleaner candidates. Audience is specialist (junior + senior).

### Section 13 — `#lab`
**Component:** `lab-preview.tsx` · **Role:** proof · **Chrome:** `PostCard` grid (1 featured, or 3-up).
**Available decorative zones:**
- above-section (`#build-on-platform`↔`#lab`): D ~256px. M: no divider.
- top-decorative-strip: D ~128px / M ~64px.
- bottom-decorative-strip: D ~128px / M ~64px — below the `mt-16` link CTA.
- card-edge-band: **unverified** — `PostCard` (`apps/web/src/components/lab/post-card.tsx`) not opened. Flag.
- text-free side-rail: card grid spans the box.
**Friction:** section is conditionally `null`. If absent, `#build-on-platform` borders `#contact` directly.

### Section 14 — `#contact` (ClosingCTA)
**Component:** `apps/web/src/components/shared/closing-cta.tsx` (client variant) · **Role:** closing · **Chrome:** `AudienceTag`.
**Available decorative zones:**
- above-section (`#lab`↔`#contact`): D ~256px. M: no divider.
- top-decorative-strip: D ~128px / M ~64px — opens with `AudienceTag` then `<h2>`.
- bottom-decorative-strip: D ~128px / M ~64px — below the CTA row (`Button` + Telegram link). This is the **last text-free band before `<Footer>`** (footer is outside `<main>`, rendered by `layout.tsx:96`).
- card-edge-band: no cards.
- text-free side-rail: **yes** — `<h2>` caps `max-w-[16ch]`, sub `<p>` `max-w-[56ch]`; a wide right rail exists.
**Friction:** the page's terminal block; the bottom strip abuts the global `<Footer>`. A closing decorative gesture here is "page-end", not "section-boundary".

---

## 5. Narrative-arc summary

The page runs **opening → mechanism → proof → process → mechanism → segmentation → closing**: Hero states the positioning; `#what`/`#connections`/`#operating-model` explain the machine; `#now`/`#lab` are proof; `#how` is process; the `#for-founders`/`#for-legacy`/`#who-we-work-with`/`#build-on-platform` cluster segments the audience; `#contact` closes. The natural emphasis pauses fall **after the `#what` `PullQuote`** (first typographic crescendo) and **at `#for-legacy`** (the page's quietest, most spacious block, and the only one with a mobile-visible divider).

On the terracotta question the brief raised: terracotta is `--primary`, and in the *current* page it is **not a rare anchor** — it recurs as the `TacticalFrame` bracket colour in the Hero and (via `MonoLabel tone="primary"` in `SectionHead`) as the tint of nearly every section eyebrow. Cool-blue `--accent` carries the technical motifs (`StepRow` numerals, the `OrchestraDiagram` conductor, the Hero blueprint line). So a "terracotta accent band" would **extend an already-frequent hue**, not introduce a scarce one — a downstream pass should decide deliberately whether terracotta stays ambient (eyebrows) or is escalated to a structural accent, because it cannot be both without losing its signal.

---

## 6. Flagged questions for a downstream design pass

1. **Boundary band = one shared region.** A desktop section boundary is ~256px of contiguous text-free space split by a hairline `md:border-t`. Should a pattern fill treat the boundary as a single band, or only ever the top-of-section half? (Double-counting top+bottom strips as separate zones is the most likely error.)
2. **Mobile has no dividers.** Every inter-section `border-t` is `md:`-only except `#for-legacy`. A boundary-strip treatment that depends on the divider line silently disappears on mobile. Is a mobile fallback required, or is pattern work desktop-only?
3. **`#for-legacy` is the standout candidate** — lowest density, largest padding (`py-40`), the only unconditional/mobile-visible divider. Should decorative emphasis concentrate there, or be distributed?
4. **Conditional sections break adjacency.** `#now` and `#lab` can be absent; `#operating-model`'s proof block and `#network-and-community`'s roster are conditional. Any boundary treatment must be defined on *pairs that can actually occur*. Which adjacencies are guaranteed?
5. **Card-edge bands are unverified.** `EntityCard`/`WorkCard`, `ServiceCard`, `PostCard`, `PersonCard` were not opened in this audit. A card-edge-band placement requires opening each component first to confirm a genuinely text-free top/bottom band exists. Is card-edge placement in scope, or boundary/strip-only?

---

## 7. Out of scope for a downstream design pass

- **No pattern under text.** Not under any `h*`/`p`/`li`/`dt`/`dd`/`blockquote`/`a`/`summary`, regardless of fade-mask. This is the ratified `design-tokens.md:72,74` line.
- **`#hero` is excluded.** `home/blocks.md:165` spec-locks the Hero block against background pattern. Do not propose a hero fill without first amending that spec.
- **Verify contrast on the dark canvas.** The public surface is dark by default (`layout.tsx:78`); patterns must be checked against near-black `bg-canvas`, and against light mode too if a theme toggle is exposed.
- **Do not invent primitives.** The 8 patterns named in the brief do **not exist in the codebase**. Any proposal must either (a) be scoped against patterns that get built as part of the same change, or (b) be explicit that it is contingent on net-new `packages/ui` primitives.
- **Do not present this as ratified.** There is no `add-background-pattern-system` change and no `D-DS-8`. A pattern system is a net-new proposal that must amend `design-tokens.md` and `home/blocks.md` through the normal OpenSpec change cycle.
- **Use the repo's real token names.** Terracotta = `--primary`; cool blue = `--accent`. The brief's "accent = terracotta" is inverted.

---

## 8. Provenance

Every claim above traces to a file opened in this audit run:

- `apps/web/src/app/[locale]/(marketing)/page.tsx`, `.../[locale]/layout.tsx`, `.../(marketing)/layout.tsx`
- `apps/web/src/components/home/`: `hero.tsx`, `what-we-build.tsx`, `now-building.tsx`, `how-we-work.tsx`, `connected-stack.tsx`, `operating-model.tsx`, `operating-model-diagram.tsx`, `operating-model-proof.tsx`, `for-founders.tsx`, `for-legacy.tsx`, `who-we-work-with.tsx`, `the-network-and-community.tsx`, `services-section.tsx`, `services-catalog.tsx`, `build-on-platform.tsx`, `lab-preview.tsx`, `step-row.tsx`
- `apps/web/src/components/shared/`: `closing-cta.tsx`, `orchestra-diagram.tsx`, `pull-quote.tsx`
- `apps/web/src/components/work/work-card.tsx`
- `packages/ui/src/components/section-head.tsx`, `tactical-frame.tsx`; `packages/ui/src/styles/globals.css`
- `openspec/current/design-tokens.md`, `openspec/current/home/blocks.md`

**Not opened (flagged as unverified for card-edge-band classification):** `packages/ui/src/components/entity-card.tsx`, `apps/web/src/components/services/service-card.tsx`, `apps/web/src/components/lab/post-card.tsx`, `apps/web/src/components/network/person-card.tsx`.
