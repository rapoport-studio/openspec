# Design System — Spec Convention

> **Status:** ratified by `D-DS-1` via `define-design-system-spec-convention` (archived 2026-05-10).
> **Owns:** how a design-system spec is shaped — files, sections, rules, anti-patterns.
> **Does not own:** *what* the design system is (that lives in `current/design-system.md` and siblings) or *how the corpus changes* (that lives in [`design-system-operating-model.md`](design-system-operating-model.md)).
>
> Companion files: [`design-system-operating-model.md`](design-system-operating-model.md), [`design-system-audit-protocol.md`](design-system-audit-protocol.md), [`templates/design-system/`](templates/design-system/).

---

## 1. Why this exists

The studio's design system used to live in one 909-line capability file mixing intent, OKLCH values, motion choreography, props notes, and applied-deltas history. It worked as a snapshot but did not survive sustained editing — three of the eight findings in `openspec/analysis/2026-05-10-component-reuse-audit.md` traced back to convention drift, not implementation gaps.

This convention writes the rules down once. Every design-system-related change references it. R1..R13 below are the rule set; the five-file structure is the structural consequence; the migration plan that landed it is preserved in `archive/2026-05-10-define-design-system-spec-convention/`.

---

## 2. Five-file structure

The design-system area of `openspec/current/` is five files. Each owns one concern. The "Does NOT own" column is the load-bearing rule — that's where 90% of drift sneaks in.

### 2.1 `current/design-system.md` — visual intent

| Owns | Does NOT own |
|---|---|
| Philosophy ("editorial serif as reading anchor, mono for meta") | OKLCH values |
| Voice rules, casing rules, tone, error/empty/loading copy patterns | Token names |
| Anti-patterns at the brand level (no gradients, no emoji, no cockpit clichés) | Component anatomy |
| Surface mapping (which surface uses which theme + density profile) | Tailwind utilities |
| Hard constraints (numbered list at brand level) | path:line refs |

### 2.2 `current/design-tokens.md` — token registry

| Owns | Does NOT own |
|---|---|
| Token role names + semantic intent | OKLCH values (those are in `@theme`) |
| `$type` tag per W3C DTCG (see § 5) | Component anatomy |
| Tailwind utility mapping per token | Motion durations / curves (those are in `design-motion.md`) |
| Mode parity table (light value source / dark value source) | Variant rationale |
| Resolution order (when an alias delegates) | Visual intent prose |
| Cross-reference: `globals.css § <block>` and line range | Implementation overrides |

### 2.3 `current/design-motion.md` — motion

| Owns | Does NOT own |
|---|---|
| Duration scale (`--duration-instant` … `--duration-slower`) | Component-specific durations |
| Easing curves (`--ease-standard`, `--ease-out`, `--ease-in`, `--ease-spring`) | Per-component animation choreography |
| Motion modes (productive / expressive) and when each applies | Visual intent (lives in `design-system.md`) |
| Choreography rules (stagger, continuous element, minimum visibility) | Specific component animations (those reference these tokens) |
| Reduced-motion handling (`--duration-scalar`) | OKLCH (color is not motion) |
| Anti-patterns (motion as decoration, motion ≥500ms for UI events) | Token registry |

### 2.4 `current/design-components.md` — component intent

| Owns | Does NOT own |
|---|---|
| Component anatomy (rows / regions / what slots exist) | Props API (lives in code; spec links to it) |
| Usage rules ("when to use this primitive vs that one") | Tailwind class strings |
| Variants — closed list, each with rationale | path:line refs |
| A11y baseline per component | Internal markup |
| `lives_in` field (`packages/ui` / `apps/studio` / `apps/web`) | Token values |
| Component status (canonical / deferred / experimental / deprecated) | Animation timings (motion lives in `design-motion.md`) |
| Promotion criteria per R10 (when component moves into `packages/ui`) | Visual intent |

### 2.5 `current/ui.md` — implementation references

| Owns | Does NOT own |
|---|---|
| Package boundaries + ESLint enforcement | Token values |
| Exports surface | Component anatomy |
| Build / Tailwind v4 wiring + `@theme` integration | Visual intent |
| path:line anchors for every `design-components.md` entry | Variant rationale |
| Real Tailwind class strings observed in code | Motion choreography |
| Promotion criteria operationalized (where the component currently lives + migration history) | OKLCH |

---

## 3. Industry standards consulted

The convention does not invent shape from scratch. Each rule traces to an external reference whenever one exists.

| Source | Used for | URL |
|---|---|---|
| **W3C DTCG Format Module 2025.10** | Token registry `$type` field; closed list of types | https://www.designtokens.org/tr/drafts/format/ |
| **Tailwind CSS v4 — `@theme` directive** | Token source-of-truth in CSS, not JS config | https://tailwindcss.com/docs/theme |
| **shadcn/ui — CSS variables theming** | Convention for OKLCH source-of-truth + Tailwind utility names | https://ui.shadcn.com/docs/theming |
| **IBM Carbon Design System — Motion** | Two-mode philosophy (productive / expressive); duration scale shape; easing-curve names | https://carbondesignsystem.com/elements/motion/overview/ |
| **Norton Design System — duration-scalar** | `--duration-scalar` for prefers-reduced-motion + dev slow-motion | https://wwnorton.github.io/design-system/docs/foundations/motion/ |

These are referenced — not vendored. The studio's tokens, easing curves, and component contracts remain the studio's; the *shape* of how they are written down follows the references above.

---

## 4. Tailwind v4 context

Tailwind v4 introduced the `@theme` directive and CSS-first configuration. Tokens live in CSS, not in `tailwind.config.ts`. Source of truth for *values* is therefore `packages/ui/src/styles/globals.css`. Studio conventions written before v4 assumed JS-config sources; this convention re-anchors source-of-truth to CSS:

- **Values** (OKLCH, durations in ms, sizes in px) live in `@theme` blocks in `globals.css`. Never in spec files.
- **Names** (semantic role: `--rail-bg`, `--accent-on-dark`) live both in `@theme` and in `design-tokens.md`. The spec file declares the contract; the CSS implements it.
- **Tailwind utility mapping** (`bg-rail-bg` → `var(--color-rail-bg)`) lives in `@theme inline`. Spec files reference utility names but do not copy mapping logic.
- **Resolution order** (when `@theme inline` aliases delegate to other tokens) is documented once in `design-tokens.md § Resolution order`, not repeated per token.

---

## 5. Token registry shape — W3C DTCG

Each token in `design-tokens.md` is one row in a Markdown table. Row schema:

```
| Token name | $type | Semantic role | Tailwind utility | Mode parity | Source |
```

`$type` is one of the closed DTCG list:

```
color, dimension, fontFamily, fontWeight, fontSize, lineHeight,
letterSpacing, duration, cubicBezier, shadow, opacity, gradient,
transition (composite), typography (composite), border (composite)
```

Examples:

```
| --color-canvas | color | Page background; warm ivory in light, near-black in dark | bg-canvas | light + dark | globals.css § Palette |
| --space-md | dimension | Base 24px grid unit; everything snaps to 24 | gap-md | shared | globals.css § Spacing |
| --duration-fast | duration | 100ms — micro-interactions, hover transitions | — (consumed via motion utilities) | shared | globals.css § Motion |
| --ease-standard | cubicBezier | Default curve for elements visible across full motion duration | — | shared | globals.css § Motion |
| --font-serif | fontFamily | Editorial body + display; Lora variable | font-serif | shared | globals.css § Typography |
```

Composite types (`typography`, `border`, `transition`) are deferred — DTCG allows nesting, and the studio has no current consumer that needs composites. R11 only requires the 13 atomic `$type` values be used correctly. Composites are tracked as `Q-DS-2`.

---

## 6. Motion specification

### 6.1 Duration scale

| Token | Value | Use |
|---|---|---|
| `--duration-instant` | `0ms` | No-op for `prefers-reduced-motion` reductions; explicit "no animation" signal |
| `--duration-fast` | `100ms` | Hover, color shifts, focus rings |
| `--duration-normal` | `200ms` | Default for productive UI: dropdowns, toggles, micro-transitions |
| `--duration-slow` | `300ms` | Dialogs, sheets, multi-property transitions |
| `--duration-slower` | `500ms` | Largest allowed UI duration; signature moments only |

**Anti-pattern:** any UI motion > 500ms. If something needs longer, it is not motion — it is animation (decorative, optional, separately governed).

### 6.2 Easing curves

| Token | Curve | Use |
|---|---|---|
| `--ease-standard` | `cubic-bezier(0.4, 0, 0.2, 1)` | Element visible from start to end of motion (expanding tile, sorting reorder) |
| `--ease-out` | `cubic-bezier(0, 0, 0.2, 1)` | Entry — modal opens, dropdown appears, toggle on |
| `--ease-in` | `cubic-bezier(0.4, 0, 1, 1)` | Exit — modal closes, toaster leaves, item dismissed |
| `--ease-spring` | `cubic-bezier(0.34, 1.56, 0.64, 1)` | Expressive moments only; rare |

**Anti-patterns:** bounce/stretch in productive mode; linear easing in any mode (looks robotic); per-component custom curves (use the scale, do not invent).

### 6.3 Motion modes

| Mode | Curves | Durations | Use |
|---|---|---|---|
| **productive** (default) | `--ease-standard`, `--ease-out`, `--ease-in` | `--duration-fast`, `--duration-normal` | Micro-interactions; >95% of motion |
| **expressive** | may use `--ease-spring` | `--duration-normal`, `--duration-slow` | Signature moments — page transitions, success states, brand reveals |

**Anti-pattern:** expressive easing on routine interactions (hover, dropdown). Noise, not signal.

The studio's existing **Card reveal sequence** (deploy-from-center brackets → blueprint line → content fade, total ~1200ms) is `expressive` mode. The 60ms stagger between bracketed cards is a brand-rhythm grandfather (§ 6.5). Hover color-shift transitions are `productive`.

### 6.4 Reduced motion

```
:root {
  --duration-scalar: 1;
}
@media (prefers-reduced-motion: reduce) {
  :root {
    --duration-scalar: 0;
  }
}
```

All durations in code are `calc(var(--duration-fast) * var(--duration-scalar))`. When scalar is 0, all motion collapses to instant; when scalar is 4 or 8 (dev override), motion runs in slow-motion for choreography debugging.

### 6.5 Choreography rules

- **Multiple elements animating together:** simultaneously OR staggered with `--stagger-base: 50ms`. The existing `60ms` stagger between bracketed cards is grandfathered as the studio's brand rhythm; new staggers use 50ms.
- **Element appears + disappears in same gesture:** total visible time ≥ 200ms, otherwise the user does not register it.
- **Page transitions:** must include a continuous element (something present in both states). Otherwise the user loses spatial context.

### 6.6 Anti-patterns (motion-specific)

- Motion as decoration (no functional purpose).
- Motion that distracts from the primary task.
- UI event motion > 500ms.
- Bounce / overshoot in productive mode.
- Parallel unsynchronized animations in one scene.

---

## 7. Convention — R1..R13

**R1.** Semantic layer ≠ values. Specs describe roles (`--surface`, `--accent`); they never quote OKLCH. Source for values: `globals.css § @theme`. A PR that only changes a token *value* does not require an OpenSpec change. A PR that changes a *name*, *alias*, or *semantic mapping* does.

**R2.** Token map per visual-surface spec is mandatory. Three columns minimum: `Element / Token / Source`. Without it, the spec is incomplete.

**R3.** Component intent ≠ component implementation. The spec describes meaning, when used, variants with rationale, constraints (a11y, density). It does NOT describe props API, Tailwind classes, or pixel values. Implementation lives in code; the spec links via path string.

**R4.** Tailwind v4 `@theme` is the only source of truth for values. Specs do not duplicate OKLCH. Specs describe mapping (which utility corresponds to which role) and resolution order (`@theme inline` vs `@theme`).

**R5.** Variants — closed list with rationale. Every variant of a component is named explicitly with one sentence "when used". Open-ended variant systems are forbidden.

**R6.** Cross-cutting decisions (borderless treatment, density posture, motion-reduce posture) go in `decisions.md` as `D-DS-N`, not scattered through spec prose.

**R7.** Iconography — one file, closed mapping. Duplicating icon mapping in other spec files is forbidden; they reference.

**R8.** Mode parity. Every semantic variable is defined in every mode (light, dark). Specs include a parity-checklist in their Verification section.

**R9.** A11y baseline at the token level. Contrast, focus ring, touch target are described once at the token level — not repeated per component.

**R10.** Component reuse promotion criteria. A component is promoted into `packages/ui` when: ≥2 consumers; no surface-specific logic; tokens via CSS vars; a11y contract stable. These are criteria, not taste.

**R11.** Token registry follows W3C DTCG type tagging. Every entry in `design-tokens.md` carries `$type` from the closed list (§ 5). Type field is mandatory.

**R12.** Motion is a first-class category. Not a subsection inside `components`. Lives in `design-motion.md`. Minimum content: duration scale, easing curves, motion modes, `prefers-reduced-motion` handling, choreography rules. Component specs use motion tokens — they do not declare their own durations.

**R13.** Themes vs Modes — strictly distinguished. *Mode* = variant of one token's value (light/dark). *Theme* = a complete token set scoped to one app surface (studio theme; marketing theme; future client theme). Themes do not overlap; each owns its tokens. Any spec that uses "theme" or "mode" uses them strictly.

---

## 8. Anti-patterns per file

### 8.1 `design-system.md` (visual intent)

- Embedded OKLCH values
- Component anatomy or props
- path:line anchors
- Token name registry tables

### 8.2 `design-tokens.md`

- Component variant tables
- Visual intent prose ("editorial serif as reading anchor")
- Motion durations/curves (those live in `design-motion.md`)
- Token rows missing `$type`

### 8.3 `design-motion.md`

- Per-component animation choreography (component specs reference these tokens)
- Token color values
- Visual brand intent
- "Animations" framed as decoration without functional purpose

### 8.4 `design-components.md`

- Props API documentation
- Tailwind class strings
- Component anatomy with embedded OKLCH or px values
- Variants without rationale
- Components missing `status` (canonical / deferred / experimental / deprecated) or `lives_in` fields

### 8.5 `ui.md` (implementation)

- Visual intent prose
- Token name *definitions* (those live in `design-tokens.md`; `ui.md` references them)
- Variant rationale (lives in `design-components.md`)
- OKLCH or px values

---

## 9. Authoring workflow

When opening a design-system change, decide which file the new content belongs in *before* drafting it. The "Does NOT own" column in § 2 is the deciding rule. If the answer is genuinely "this content belongs in two files", the answer is wrong — split it.

A few common cases:

- **A new token** → `design-tokens.md`. Add the row with `$type`. Reference the `globals.css` block in the Source cell. If the token also implies a new component variant, that variant lives in `design-components.md`, not in the token row.
- **A new component** → `design-components.md`. Status starts as `experimental` (lives_in: `apps/<consumer>`). Promotion to `packages/ui` is gated by R10 (≥2 consumers; no surface-specific logic; tokens via CSS vars; a11y stable).
- **A motion change** → `design-motion.md`. Component specs reference motion tokens; they never declare durations.
- **A philosophy/voice change** → `design-system.md`. No tokens, no anatomy.
- **An implementation reshuffle** → `ui.md`. Visual intent untouched.

If you are unsure where a piece of content belongs, register it as **Q-DS-N** in `open-questions.md` instead of guessing. The cost of an open question is one row; the cost of mis-filed content is a future audit finding.

---

## 10. Cross-references

- [`current/design-system.md`](../current/design-system.md) — visual intent.
- [`current/design-tokens.md`](../current/design-tokens.md) — token registry (W3C DTCG `$type`).
- [`current/design-motion.md`](../current/design-motion.md) — motion as first-class.
- [`current/design-components.md`](../current/design-components.md) — component intent.
- [`current/ui.md`](../current/ui.md) — implementation references.
- [`design-system-operating-model.md`](design-system-operating-model.md) — roles, workflow, decision authority.
- [`design-system-audit-protocol.md`](design-system-audit-protocol.md) — quarterly audits, severity scale.
- [`templates/design-system/`](templates/design-system/) — proposal / design / tasks / agent-brief skeletons.
- [`studio-charter.md`](studio-charter.md) § 5 (OpenSpec methodology) and § 9 (Hard rules) — upstream governance.
